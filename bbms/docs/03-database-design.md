# Database Design — BBMS (PostgreSQL + Prisma)

Relational, normalized to **3NF** (with deliberate, documented denormalization for
inventory counters which are also kept consistent by a ledger). All tables carry
`id (uuid)`, `created_at`, `updated_at`, and where applicable `deleted_at` (soft delete)
and `branch_id` (multi-tenancy by branch).

---

## 1. ER Overview (text diagram)

```
ORGANIZATION 1───* BRANCH 1───* USER *───* ROLE *───* PERMISSION
                      │
   ┌──────────────────┼─────────────────────────────────────────────┐
   │                  │                                              │
 DONOR 1───* DONATION_VITALS                                     HOSPITAL 1───* HOSPITAL_DOCTOR
   │                                                                 │
   1                                                                 1
   *                                                                 *
 COLLECTION(bag) 1───1 LAB_TEST                                  PATIENT
   │  │                                                              │
   │  1───* BLOOD_COMPONENT 1───* INVENTORY_LEDGER                   │
   │                  │                                              │
   │                  *                                              │
   │              RESERVATION *───1 BLOOD_REQUEST *───1 PATIENT ─────┘
   │                                   │
   │                                   1
   │                                   *
   └──────────────────────► ISSUE *───1 BLOOD_COMPONENT
                              │
                              1
                              *
                          INVOICE 1───* INVOICE_LINE   INVOICE 1───* PAYMENT

 CAMP 1───* COLLECTION        CAMP 1───* CAMP_EXPENSE    CAMP 1───* CAMP_PHOTO
 STAFF 1───* ATTENDANCE       STAFF 1───* LEAVE
 AUDIT_LOG (polymorphic)      NOTIFICATION (per user/event)
```

---

## 2. Core Tables (selected columns + key constraints)

### organization
`id, name, gstin, address, logo_url, settings(jsonb)`

### branch
`id, organization_id→organization, name, code(unique), address, phone, license_no`

### user
`id, branch_id→branch, email(unique), username(unique), password_hash, full_name,
phone, status(enum: active|suspended), last_login_at, mfa_enabled`
- Index: `(email)`, `(branch_id, status)`.

### role  /  permission  /  role_permission  /  user_role
- `role(id, name unique, description, is_system)`
- `permission(id, module enum, action enum)` — action ∈ {view,create,update,delete,approve,export}
- Junctions `role_permission(role_id, permission_id)`, `user_role(user_id, role_id)`.
- Many-to-many. Enables granular per-module-per-action control.

### donor
```
id, branch_id, donor_code(unique), photo_url, full_name, dob, gender(enum),
blood_group(enum: A_POS..O_NEG), weight_kg, mobile, email, address, city,
occupation, govt_id_type, govt_id_number(encrypted), emergency_contact_name,
emergency_contact_phone, medical_history(jsonb), eligibility_status(enum:
eligible|temporarily_deferred|permanently_deferred), deferral_reason,
last_donation_date, next_eligible_date, donation_count, status(enum: active|inactive),
deleted_at
```
- Indexes: `(blood_group)`, `(mobile)`, `(city)`, `(eligibility_status)`, full-text on `full_name`.
- Constraint: `weight_kg >= 0`, `donation_count >= 0`.

### donation_vitals
`id, collection_id→collection, bp_systolic, bp_diastolic, pulse, hb_level, temperature,
passed(bool), notes` (pre-donation screening).

### collection  (the "bag")
```
id, branch_id, bag_number(unique), donor_id→donor, blood_group(enum), volume_ml,
collected_at, collection_staff_id→user, donation_type(enum: voluntary|replacement|camp|hospital),
camp_id→camp(nullable), hospital_id→hospital(nullable),
status(enum: collected|processing|quarantine|completed|rejected), reject_reason
```
- Indexes: `(bag_number)`, `(donor_id)`, `(status)`, `(collected_at)`, `(camp_id)`.

### lab_test
```
id, collection_id→collection(unique 1:1), hb, hiv(enum: pending|negative|reactive),
hbsag(enum), hcv(enum), malaria(enum), syphilis(enum), blood_typing,
result(enum: pending|approved|rejected), technician_id→user, doctor_id→user(nullable),
comments, tested_at, approved_at
```
- Rule (app + DB check trigger): if any TTI = reactive ⇒ `result = rejected`.

### blood_component
```
id, branch_id, collection_id→collection, component_type(enum: whole_blood|prbc|
platelets|ffp|cryoprecipitate), barcode(unique), volume_ml, storage_temp,
storage_location, produced_at, expires_at,
status(enum: available|reserved|issued|expired|discarded|quarantine), discard_reason
```
- Indexes: `(component_type, blood_group)`, `(status)`, `(expires_at)`, `(barcode)`.
- `blood_group` denormalized from collection for fast inventory queries.

### inventory_ledger  (source of truth for counts)
```
id, branch_id, component_id→blood_component, blood_group, component_type,
change(int: +1/-1), reason(enum: collected|reserved|released|issued|expired|discarded|
transferred_in|transferred_out|returned), ref_type, ref_id, balance_after, created_at
```
- Append-only. Real-time inventory = aggregation; cached snapshot in `inventory_snapshot`.

### inventory_snapshot  (denormalized cache, rebuilt from ledger)
`branch_id, blood_group, component_type, available, reserved, issued, expired, discarded`
- Unique `(branch_id, blood_group, component_type)`. Maintained transactionally + reconciled by job.

### hospital  /  hospital_doctor
- `hospital(id, branch_id, name, address, city, phone, email, outstanding_amount_paise)`
- `hospital_doctor(id, hospital_id→hospital, name, specialization, phone, reg_no)`

### patient
```
id, branch_id, patient_code, full_name, age, gender, blood_group(enum),
hospital_id→hospital(nullable), doctor_id→hospital_doctor(nullable),
medical_notes, created_at
```

### blood_request
```
id, branch_id, request_code(unique), patient_id→patient, hospital_id→hospital(nullable),
blood_group(enum), component_type(enum), units_required, priority(enum: critical|normal),
source(enum: online|hospital|emergency|walk_in),
status(enum: pending|approved|rejected|fulfilled|cancelled),
requested_at, approved_by→user(nullable), approved_at, notes
```
- Indexes: `(status)`, `(priority)`, `(requested_at)`, `(hospital_id)`.

### reservation
`id, request_id→blood_request, component_id→blood_component, reserved_at, expires_at,
status(enum: held|released|consumed)`
- Links a held unit to a request; releases on cancel/expiry.

### cross_match
`id, request_id→blood_request, component_id→blood_component, patient_sample_id,
result(enum: compatible|incompatible|pending), technician_id→user, performed_at`

### issue
```
id, branch_id, issue_code(unique), request_id→blood_request, component_id→blood_component,
patient_id→patient, hospital_id→hospital(nullable), doctor_id→hospital_doctor(nullable),
issued_by→user, cross_match_id→cross_match, issued_at,
status(enum: issued|returned|cancelled), return_reason
```
- One issue ⇒ one component (a multi-unit request creates multiple issue rows).
- Trigger: on insert ⇒ ledger `-1 issued`, component.status=issued, create invoice_line.

### camp  /  camp_expense  /  camp_photo
- `camp(id, branch_id, name, location, scheduled_date, organizer, status(enum:
  upcoming|ongoing|completed|cancelled), revenue_paise)`
- `camp_expense(id, camp_id, description, amount_paise)`
- `camp_photo(id, camp_id, url, caption)`

### staff  /  attendance  /  leave  /  department
- `staff(id, user_id→user, department_id→department, designation, joined_at)`
- `attendance(id, staff_id, date, check_in, check_out, status)`
- `leave(id, staff_id, from_date, to_date, type, status, approved_by)`

### invoice  /  invoice_line  /  payment  /  expense
```
invoice(id, branch_id, invoice_no(unique), hospital_id(nullable), patient_id(nullable),
subtotal_paise, gst_paise, total_paise, status(enum: draft|unpaid|partial|paid|void),
issued_at, due_at)
invoice_line(id, invoice_id, issue_id(nullable), description, qty, unit_price_paise,
gst_rate, amount_paise)
payment(id, invoice_id, amount_paise, method(enum: cash|card|upi|bank), paid_at, ref_no)
expense(id, branch_id, category, amount_paise, incurred_at, notes)
```

### notification
`id, user_id→user(nullable), type(enum), channel(enum: in_app|email|sms|whatsapp),
title, body, payload(jsonb), read_at, status(enum: queued|sent|failed), created_at`

### audit_log  (append-only, hash-chained)
`id, actor_id→user, action, entity_type, entity_id, before(jsonb), after(jsonb),
ip, user_agent, prev_hash, hash, created_at`

### setting  (key-value per branch/org)
`id, scope(enum: org|branch), scope_id, key, value(jsonb)`

---

## 3. Key Relationships & Cardinality

| From | To | Type | Notes |
|---|---|---|---|
| organization | branch | 1–* | |
| branch | user / donor / collection / ... | 1–* | tenancy boundary |
| donor | collection | 1–* | a donor donates many times |
| collection | lab_test | 1–1 | each bag tested once (re-test = new version) |
| collection | blood_component | 1–* | separation into components |
| blood_component | inventory_ledger | 1–* | every state change is a ledger row |
| blood_request | reservation | 1–* | held units |
| blood_request | issue | 1–* | one issue per unit |
| issue | invoice_line | 1–1 | auto-billing |
| invoice | payment | 1–* | partial payments |
| camp | collection | 1–* | camp collections |

## 4. Indexing Strategy
- B-tree on all FKs and status/date columns used in filters.
- Composite `(blood_group, component_type, status)` on `blood_component` for inventory.
- Composite `(branch_id, requested_at)` on `blood_request` for dashboards.
- Partial index `WHERE deleted_at IS NULL` on soft-deleted tables.
- GIN full-text on `donor.full_name`, `patient.full_name`, `hospital.name` for global search.
- `expires_at` index drives the expiry job.

## 5. Constraints & Integrity
- FK constraints with `ON DELETE RESTRICT` for clinical tables (no cascading data loss).
- CHECK: positive volumes/weights/amounts; enums for all controlled vocabularies.
- UNIQUE: `bag_number`, `barcode`, `*_code`, `invoice_no`, `email`, `username`.
- DB triggers backstop the two critical safety rules (reactive ⇒ rejected; issue ⇒ ledger+invoice),
  even though they're also enforced in the application service layer.

## 6. Normalization Notes
- 1NF: atomic columns; repeating data (doctors, photos, expenses) in child tables.
- 2NF/3NF: no partial/transitive dependencies; lookups (roles, permissions, departments)
  are separate tables.
- **Deliberate denormalization:** `blood_component.blood_group` (from collection) and
  `inventory_snapshot` (from ledger) for read performance — both are derived and
  reconciled, so they don't violate the single-source-of-truth principle.

## 7. Migrations & Seed
- Prisma migrations, versioned. Seed: roles, permissions, blood groups, component
  shelf-life config, one super admin, demo branch — all idempotent.
