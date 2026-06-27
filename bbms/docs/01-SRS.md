# Software Requirement Specification (SRS) — BBMS

Version 1.0 · Status: Draft for approval · Standard: loosely follows IEEE 830.

---

## 1. Introduction

### 1.1 Purpose
Define the complete functional and non-functional requirements for the Blood Bank
Management System so that engineering can implement, and stakeholders can verify,
a production-grade product.

### 1.2 Intended Audience
Product owner (blood bank management), engineering, QA, and auditors.

### 1.3 Definitions
- **TTI** — Transfusion Transmissible Infection (HIV, HBsAg, HCV, Syphilis, Malaria).
- **PRBC** — Packed Red Blood Cells. **FFP** — Fresh Frozen Plasma.
- **Cross-match** — compatibility test between donor unit and patient sample.
- **Component separation** — splitting whole blood into PRBC / platelets / FFP / cryo.
- **Unit / Bag** — one physical collection, identified by a unique bag number.

---

## 2. Overall Description

### 2.1 Product Perspective
A standalone, multi-branch web application (responsive + PWA). Backend exposes a
versioned REST API consumed by the web client and, later, by integrations.

### 2.2 User Classes & Privileges (RBAC summary)

| Role | Can do |
|---|---|
| **Super Admin** | Everything + org/branch/user/role config, backup/restore |
| **Admin** | All operational modules, approve requests, no destructive config |
| **Doctor** | Donor eligibility sign-off, lab approval, cross-match/issue sign-off |
| **Lab Technician** | Enter & submit lab results, component separation |
| **Reception** | Register/edit donors, schedule donations, print donor cards |
| **Data Entry** | Bulk import, data capture for donors/collection |
| **Accountant** | Billing, payments, invoices, financial reports |
| **Blood Collection Staff** | Create collection records, manage camp collections |
| **Store Manager** | Inventory, expiry/discard, storage locations |

Permissions are enforced **per module + per action** (view/create/update/delete/approve).

### 2.3 Operating Environment
Modern browsers (last 2 versions of Chrome, Edge, Firefox, Safari). Server: Linux
+ Docker. DB: PostgreSQL 15+. Cache/queues: Redis. Object storage: S3-compatible.

### 2.4 Design & Implementation Constraints
- Stack fixed by brief: Next.js + TS + Tailwind + shadcn/ui + React Query + Zustand
  + Framer Motion (frontend); Node + Express + Prisma + PostgreSQL + Redis + BullMQ
  (backend); JWT auth; Docker + Nginx for deployment.
- All money in minor units (paise) as integers. All timestamps stored UTC.
- No PHI/PII leaves the system unencrypted.

### 2.5 Assumptions
- Single organization, multiple branches.
- Indian regulatory context (GST, NACO/Drugs & Cosmetics Act report formats) but
  report templates are configurable.

---

## 3. Functional Requirements (by module)

> Notation: **FR-<module>-<n>**. "System shall…".

### 3.1 Authentication & Authorization (AUTH)
- FR-AUTH-1: Email/username + password login with bcrypt/argon2 hashing.
- FR-AUTH-2: JWT access token (short-lived) + rotating refresh token (httpOnly cookie).
- FR-AUTH-3: Forgot-password via email reset link (expiring token).
- FR-AUTH-4: OTP (email/SMS) for login MFA and sensitive actions (configurable).
- FR-AUTH-5: Role-based access enforced on every API route and UI route.
- FR-AUTH-6: Account lockout after N failed attempts; rate-limited login.
- FR-AUTH-7: Session list + remote logout; force-logout on role change.

### 3.2 Dashboard (DASH)
- FR-DASH-1: KPI cards — Today's Collection, Today's Issued, Today's Requests,
  Emergency Requests, Pending Tests, Total Inventory, Units Expiring Soon (≤7d),
  Today's Revenue, Active Camps.
- FR-DASH-2: Charts — Weekly & Monthly Collection, Inventory Trend, Demand by
  Blood Group, Component Distribution.
- FR-DASH-3: Recent Activity timeline (from audit log), Quick Actions, Tasks,
  Pinned Reports, Favorites.
- FR-DASH-4: All KPIs scoped to the selected branch and role permissions.

### 3.3 Donor Management (DON)
- FR-DON-1: CRUD donors with all fields (see DB `donors`), photo upload, govt ID.
- FR-DON-2: Auto-compute eligibility: age 18–65, weight ≥ 45kg, ≥ 90 days (men) /
  120 days (women) since last whole-blood donation; flag deferrals.
- FR-DON-3: Maintain donation count, last donation date, next eligible date.
- FR-DON-4: Search, multi-filter (group, status, eligibility, city), export/import (CSV).
- FR-DON-5: Print donor card with QR code; view full donation history.
- FR-DON-6: Soft-delete with undo; never hard-delete a donor with donations.

### 3.4 Blood Collection (COL)
- FR-COL-1: Create collection record: unique bag number, date, donor, group, volume,
  collection staff, donation type (Voluntary/Replacement/Camp/Hospital), source camp/hospital.
- FR-COL-2: Status lifecycle: Collected → Processing → Completed, or → Rejected.
- FR-COL-3: Pre-donation vitals capture (BP, pulse, Hb, temp) with pass/fail gate.
- FR-COL-4: Block collection if donor not eligible (override requires Doctor + reason).
- FR-COL-5: On creation, update donor's last/next donation dates and count.

### 3.5 Lab Testing (LAB)
- FR-LAB-1: For each bag, record HB, HIV, HBsAg, HCV, Malaria, Syphilis, blood typing.
- FR-LAB-2: Cross-match results against patient samples for issue.
- FR-LAB-3: Result = Approved / Rejected with technician, doctor sign-off, comments.
- FR-LAB-4: **Any reactive TTI → bag auto-marked Rejected and quarantined; cannot
  enter inventory.** Triggers discard workflow.
- FR-LAB-5: Only Approved bags become eligible for component separation.

### 3.6 Blood Components (CMP)
- FR-CMP-1: On lab approval, allow separation of whole blood into Whole Blood / PRBC /
  Platelets / FFP / Cryoprecipitate (configurable yield rules).
- FR-CMP-2: Each component gets component-specific expiry (e.g. PRBC 42d, Platelets 5d,
  FFP 1yr, Whole Blood 35d), storage temp, storage location, barcode/QR, status.
- FR-CMP-3: Components inherit traceability to parent bag and donor.

### 3.7 Inventory (INV)
- FR-INV-1: Real-time counts by blood group × component: Available / Reserved /
  Issued / Expired / Discarded.
- FR-INV-2: Color-coded stock levels (green/amber/red) vs configurable thresholds.
- FR-INV-3: Automatic expiry job moves expired units → Expired and alerts Store Manager.
- FR-INV-4: Low-stock alerts per group/component.
- FR-INV-5: Reservation places a hold tied to a request; release on cancel/expire.

### 3.8 Patients (PAT)
- FR-PAT-1: CRUD patient with hospital, doctor, blood required, component, units,
  request date, status, medical notes, blood group.
- FR-PAT-2: Link patient to one or more blood requests and issues (history).

### 3.9 Blood Requests (REQ)
- FR-REQ-1: Create requests: online / hospital / emergency, with priority Critical/Normal.
- FR-REQ-2: Status: Pending → Approved/Rejected → Fulfilled (Completed).
- FR-REQ-3: On approval, reserve matching units from inventory.
- FR-REQ-4: Emergency/Critical requests surface on dashboard and trigger notifications.
- FR-REQ-5: Availability check shows compatible units (incl. ABO/Rh compatibility matrix).

### 3.10 Blood Issue (ISS)
- FR-ISS-1: Issue units against an approved request via barcode/QR scan.
- FR-ISS-2: Require completed, compatible cross-match before issue.
- FR-ISS-3: Capture patient, hospital, doctor, issue date, units, component, issuing staff.
- FR-ISS-4: On issue: decrement inventory, mark unit Issued, auto-create billing line.
- FR-ISS-5: Support issue reversal/return within policy window (with reason + audit).

### 3.11 Hospital Management (HOSP)
- FR-HOSP-1: CRUD hospitals, contacts, addresses, associated doctors.
- FR-HOSP-2: Per-hospital request history, billing, and outstanding balance.

### 3.12 Blood Donation Camps (CAMP)
- FR-CAMP-1: CRUD camps (upcoming/past): location, date, organizer, volunteers.
- FR-CAMP-2: Link collections made at a camp; show collection statistics.
- FR-CAMP-3: Track camp expenses, revenue, photo gallery.

### 3.13 Staff (STAFF)
- FR-STAFF-1: Staff profiles, departments, roles.
- FR-STAFF-2: Attendance, leave requests/approval.
- FR-STAFF-3: Activity logs and basic performance metrics (collections, issues handled).

### 3.14 Billing (BILL)
- FR-BILL-1: Invoices (from issues), payments, receipts, GST computation.
- FR-BILL-2: Expenses & income, daily cash report, monthly financial report.
- FR-BILL-3: Outstanding tracking per hospital/patient; partial payments.

### 3.15 Reports (RPT)
- FR-RPT-1: Daily/weekly/monthly/yearly reports for inventory, donors, collection,
  issue, lab, finance.
- FR-RPT-2: Statutory/government report templates.
- FR-RPT-3: Export every report as PDF, Excel, CSV.

### 3.16 Notifications (NOTIF)
- FR-NOTIF-1: Channels: Email, SMS, WhatsApp (pluggable providers).
- FR-NOTIF-2: Triggers: low stock, expiry, donor eligibility reminder, camp reminder,
  emergency request.
- FR-NOTIF-3: Queued & retried via BullMQ; per-user/in-app notification center.

### 3.17 Settings (SET)
- FR-SET-1: Organization & branch management.
- FR-SET-2: Users, roles, granular permissions.
- FR-SET-3: Email/SMS/WhatsApp gateway config, themes, backup/restore.

### 3.18 Global Search (SRCH)
- FR-SRCH-1: Instant global search across donors, patients, bags, requests, hospitals.
- FR-SRCH-2: Filters available on every list view.

### 3.19 Analytics (ANL)
- FR-ANL-1: Collection & demand trends, inventory heatmap, most-requested group,
  top hospitals, top donors, revenue & growth.

### 3.20 Audit & Activity (AUD)
- FR-AUD-1: Immutable audit log of every create/update/delete/approve/issue, with
  actor, entity, before/after, IP, timestamp.
- FR-AUD-2: Soft-delete + restore (undo) for primary entities.

---

## 4. Non-Functional Requirements

### 4.1 Security
- NFR-SEC-1: Passwords hashed (argon2id). JWT signed (RS256/HS256), short TTL.
- NFR-SEC-2: Input validation (zod) on every endpoint; output encoding to prevent XSS.
- NFR-SEC-3: Parameterized queries via Prisma (no raw string SQL) → SQLi safe.
- NFR-SEC-4: CSRF protection on cookie-based flows; CORS allowlist.
- NFR-SEC-5: Rate limiting (per-IP + per-user) and brute-force lockout.
- NFR-SEC-6: Secrets in env/secret manager, never in repo. TLS everywhere.
- NFR-SEC-7: Audit log immutable; PII encrypted at rest where feasible.

### 4.2 Performance
- NFR-PERF-1: P95 API latency < 300ms for reads under nominal load.
- NFR-PERF-2: Dashboard first contentful paint < 2s on broadband.
- NFR-PERF-3: List endpoints paginated (cursor/offset), default page ≤ 25.
- NFR-PERF-4: Hot reads (inventory KPIs) cached in Redis with invalidation.

### 4.3 Reliability & Availability
- NFR-REL-1: Target 99.9% uptime; graceful degradation if queue/cache down.
- NFR-REL-2: Daily automated DB backups + tested restore.
- NFR-REL-3: Idempotent write endpoints for retries (issue, payment).

### 4.4 Scalability
- NFR-SCALE-1: Stateless API horizontally scalable behind Nginx.
- NFR-SCALE-2: Background work isolated in BullMQ workers.

### 4.5 Usability & Accessibility
- NFR-UX-1: WCAG 2.1 AA color contrast; keyboard navigable; focus states.
- NFR-UX-2: Responsive for mobile/tablet/desktop; dark mode.
- NFR-UX-3: Optimistic UI + auto-save for long forms; undo for deletes.

### 4.6 Maintainability
- NFR-MNT-1: Modular monorepo, typed end-to-end, ≥ 70% test coverage on domain logic.
- NFR-MNT-2: Lint + format + typecheck gates in CI.

### 4.7 Compliance & Data Retention
- NFR-CMP-1: Configurable retention; audit logs retained ≥ statutory minimum.
- NFR-CMP-2: Right-to-access/export of donor data.

---

## 5. Acceptance Criteria (samples)
- A bag with any reactive TTI can **never** appear as Available inventory or be issued.
- Issuing a unit always decrements inventory and creates exactly one billing line.
- An expired component is automatically excluded from availability at/after expiry.
- Every destructive action is reversible (undo) or recorded immutably in the audit log.
- Each role sees only its permitted modules in the nav and is blocked at the API.
