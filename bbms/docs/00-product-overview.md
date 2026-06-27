# Blood Bank Management System (BBMS) — Product Overview

> Phase 0 deliverable. This document captures the complete product understanding
> **before any code is written**.

## 1. Vision

A single, modern web platform that lets a private blood bank run **all** daily
operations — eliminating paper registers, disconnected spreadsheets, and manual
registers. Every entity (donor → collection → lab → component → inventory →
request → issue → billing → reports) is interconnected so a unit of blood can be
traced end-to-end from the vein it was drawn from to the patient it was
transfused into.

## 2. The Core Problem

Private blood banks today juggle:

- Paper donor registers and donation cards.
- Manual stock counts that go stale within hours.
- Lab results recorded on loose sheets, disconnected from the bag.
- Expiry tracking done by eye → wastage of a scarce, life-critical resource.
- Hospital billing reconciled at month-end with errors and disputes.
- Government / regulatory reports compiled by hand under deadline pressure.

A single mistake — issuing an expired or infected unit, or mismatching a blood
group — can be fatal and legally catastrophic.

## 3. The Solution — One Connected Lifecycle

```
 DONOR ──► COLLECTION ──► LAB TESTING ──► COMPONENT SEPARATION ──► INVENTORY
                                │                                      │
                          (reject if                                   ▼
                           any test +ve)              REQUEST ──► CROSS-MATCH ──► ISSUE
                                                          │                         │
                                                       PATIENT                   BILLING
                                                       HOSPITAL                  REPORTS
```

Every arrow is enforced in software. A bag cannot reach inventory without passing
lab tests. A unit cannot be issued without a cross-match. An issue automatically
generates a billing line. Every action is written to an immutable audit log.

## 4. Primary Personas

| Persona | Goal | Most-used modules |
|---|---|---|
| Super Admin | Configure the system, manage branches & users | Settings, Staff, Analytics |
| Admin | Run day-to-day operations, approve requests | Dashboard, Requests, Inventory |
| Doctor | Approve donor eligibility, sign off lab/issue | Lab, Donors, Issue |
| Lab Technician | Run & record TTI/serology tests | Lab Testing, Components |
| Reception / Data Entry | Register donors, capture vitals | Donors, Collection |
| Blood Collection Staff | Record bag collection (incl. camps) | Collection, Camps |
| Store Manager | Manage inventory, expiry, discards | Inventory, Components |
| Accountant | Invoices, payments, daily cash, GST | Billing, Reports |

## 5. Non-Negotiable Qualities

- **Safety first** — software-enforced rules prevent unsafe blood from being issued.
- **Traceability** — full chain of custody for every unit (donor ↔ bag ↔ component ↔ patient).
- **Auditability** — who did what, when, from where; nothing silently deleted.
- **Real-time inventory** — counts are always correct, with proactive expiry/low-stock alerts.
- **Regulatory readiness** — one-click statutory reports.
- **Beautiful & fast** — a UI on par with Stripe / Linear / Vercel so staff actually enjoy using it.

## 6. Scope Boundaries (v1)

**In scope:** the 15 functional modules listed in the brief, RBAC, notifications
(email/SMS/WhatsApp), reporting, analytics, audit logs, dark mode, PWA, barcode/QR,
PDF/Excel/CSV export.

**Out of scope for v1 (flagged for later):** native mobile apps, HL7/FHIR hospital
EMR integration, multi-currency, AI features (shipped as a follow-up phase behind a
feature flag — see `02-improvements.md`).

## 7. Planning Artifacts (this folder)

| # | Document | Purpose |
|---|---|---|
| 00 | product-overview.md | This file |
| 01 | SRS.md | Full functional + non-functional requirements |
| 02 | improvements.md | Gaps in the brief, fixes, and recommendations |
| 03 | database-design.md | ER model, tables, relationships, indexes, normalization |
| 04 | folder-structure.md | Monorepo layout for frontend + backend |
| 05 | ui-wireframes.md | ASCII wireframes for every key screen |
| 06 | user-flows.md | Step-by-step flows for critical journeys |
| 07 | api-documentation.md | REST API surface, auth, conventions |
| 08 | roadmap.md | Phased delivery plan, milestones, definition of done |
