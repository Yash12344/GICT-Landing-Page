# User Flows — BBMS

Step-by-step flows for the critical journeys. Each flow notes the **role**,
**guards** (rules that block progress), and **side effects** (what the system does
automatically).

## Flow 1 — End-to-end unit lifecycle (the spine)

```
Donor registered → Eligibility check → Collection → Lab test → (Approve)
→ Component separation → Inventory(Available) → Request → Reserve
→ Cross-match → Issue → Inventory(Issued) + Invoice → Payment → Reports
                         │
            (Lab Reject) └→ Quarantine → Discard → Wastage report
```
This is the contract the whole system enforces. Every other flow is a slice of it.

## Flow 2 — Donor registration & eligibility
**Role:** Reception / Data Entry
1. Donors → **+ Add Donor**.
2. Fill identity, contact, blood group, weight, govt ID, photo.
3. System computes eligibility (age 18–65, weight ≥ 45kg, days since last donation).
4. **Guard:** ineligible → status `deferred`, reason shown, donation blocked.
5. Save → donor code generated, audit logged. Option: **Print Donor Card** (QR).

## Flow 3 — Blood collection
**Role:** Collection Staff
1. Collection → **+ New Collection**; pick donor (search/scan card QR).
2. Capture pre-donation vitals → **Guard:** vitals fail ⇒ cannot proceed.
3. Enter bag number, volume, donation type, source (camp/hospital).
4. Save → bag status `Collected` → auto `Quarantine`; donor's `last_donation_date`,
   `next_eligible_date`, `donation_count` updated. Bag **not** in inventory yet.

## Flow 4 — Lab testing & approval
**Role:** Lab Technician → Doctor sign-off
1. Lab queue shows quarantined bags.
2. Enter HB + TTI panel (HIV, HBsAg, HCV, Malaria, Syphilis) + typing.
3. **Guard:** any reactive ⇒ result locks to `Rejected`; bag → discard workflow;
   **lookback** triggered for that donor's prior units.
4. All non-reactive + doctor sign-off ⇒ `Approved` ⇒ eligible for separation.

## Flow 5 — Component separation
**Role:** Lab Technician / Store Manager
1. Approved bag → **Separate Components**.
2. Choose components (Whole/PRBC/Platelets/FFP/Cryo) per yield rules.
3. System assigns each: barcode/QR, component-specific expiry, storage temp/location.
4. Save → each component → Inventory `Available`; ledger `+1 collected` rows written.

## Flow 6 — Blood request → approval → reservation
**Role:** Reception/Hospital (create) → Admin/Doctor (approve)
1. Requests → **+ New Request**: patient, hospital, doctor, group, component, units,
   priority (Critical/Normal), source.
2. **Critical** ⇒ flagged on dashboard + emergency notification fired.
3. Approver sees **availability** (ABO/Rh-compatible units).
4. **Approve** ⇒ reserve N compatible units (ledger `-available/+reserved`),
   reservation has expiry. Reject ⇒ reason + notify.

## Flow 7 — Cross-match → issue → billing
**Role:** Lab (cross-match) → Issue Staff
1. Perform cross-match between reserved unit(s) and patient sample → Compatible/Incompatible.
2. Issue → open approved request → **scan bag barcode**.
3. **Guards (all must pass):** ABO/Rh compatible · cross-match Compatible · not expired ·
   status Reserved/Available · not already issued.
4. Confirm ⇒ component `Issued`, ledger `-1 issued`, **invoice line auto-created**,
   request → `Fulfilled` when all units issued. Audit logged.
5. Optional return within policy window ⇒ restock + reverse billing (with reason).

## Flow 8 — Expiry & low-stock automation
**Role:** system (BullMQ) → Store Manager
1. Nightly job scans `expires_at`. Past expiry ⇒ component → `Expired`, ledger row,
   removed from availability.
2. Snapshot vs thresholds ⇒ low/critical groups generate alerts (in-app + email/SMS).
3. Store Manager works the **expiry queue** to discard with a reason (wastage report).

## Flow 9 — Blood camp
**Role:** Admin / Collection Staff
1. Camps → **+ Camp**: location, date, organizer, volunteers.
2. On camp day, collections are created with `donation_type=camp`, `camp_id` set.
3. Camp page aggregates collection stats, expenses, revenue, photos.

## Flow 10 — Billing & payment
**Role:** Accountant
1. Issues generate draft invoice lines per hospital/patient.
2. Accountant finalizes invoice (GST applied) → `Unpaid`.
3. Record payment(s) → `Partial`/`Paid`; hospital outstanding updated.
4. Daily cash + monthly financial report generated.

## Flow 11 — Auth & access
1. Login (email + password) → optional OTP (role-gated) → JWT + refresh cookie.
2. Forgot password → email reset link (expiring) → set new password.
3. Every route/API checks role permissions; unauthorized ⇒ 403 + hidden nav.
4. Role change ⇒ existing sessions re-evaluated/force-logout.

## Flow 12 — Reports & export
1. Reports → pick type + range + branch.
2. Preview on screen; **Export** as PDF / Excel / CSV (queued for large sets).
3. Pin frequently used reports to the dashboard.

## Cross-cutting
- **Undo:** destructive actions show an undo snackbar; restorable from a trash view.
- **Audit:** every create/update/delete/approve/issue writes an immutable log entry.
- **Notifications:** emergency request, low stock, expiry, donor & camp reminders.
