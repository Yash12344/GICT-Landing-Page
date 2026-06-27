# Suggested Improvements & Missing Features

The brief is strong. These are gaps, safety issues, and high-leverage additions I
recommend folding into v1 (or explicitly deferring). Items marked **[Safety]** are
strongly recommended — they prevent fatal errors or regulatory breaches.

## A. Safety & Correctness gaps in the original brief

1. **[Safety] ABO/Rh compatibility engine.** The brief lists cross-match but not a
   compatibility matrix. Issuing must validate donor↔patient ABO/Rh compatibility
   (e.g. O− universal donor, AB+ universal recipient) **before** the bag list is even
   offered. Build this as a pure, unit-tested function.
2. **[Safety] Quarantine state for untested/reactive units.** A bag must sit in
   `Quarantine` from collection until lab approval. Reactive results route to
   `Discard`, never to inventory. Make this a hard state machine, not a flag.
3. **[Safety] Component-specific shelf life.** PRBC ≈ 42d, Platelets ≈ 5d, FFP ≈ 365d,
   Whole Blood ≈ 35d, Cryo ≈ 365d. Expiry must be computed per component, not per bag.
4. **[Safety] Donation deferral rules.** Capture temporary vs permanent deferrals
   (recent tattoo, travel, low Hb, medication) so an ineligible donor is auto-blocked.
5. **[Safety] Reverse traceability (lookback).** If a donor later tests positive, the
   system must instantly list every component/issue derived from their prior donations.
6. **[Safety] Maximum issue safeguards.** Warn/block on duplicate cross-match, already-issued
   units, or group mismatch even when manually overridden.

## B. Operational features worth adding

7. **Donor appointment scheduling** + reminders (reduces walk-in chaos).
8. **Sample/segment tracking** for cross-match (pilot tubes), not just the bag.
9. **Temperature-excursion logging** for storage units (cold-chain compliance).
10. **Wastage & discard reasons taxonomy** with reporting (expiry, breakage, reactive,
    QNS) — regulators ask for this.
11. **Inter-branch stock transfer** with in-transit state.
12. **Reservation expiry** — held units auto-release if a request isn't fulfilled in N hours.
13. **Issue return / re-stock workflow** within a safe time/temperature window.
14. **Patient blood-group verification** captured independently before issue.

## C. Platform & security hardening

15. **Idempotency keys** on money/issue endpoints to survive retries.
16. **Field-level encryption** for govt ID numbers and contact PII.
17. **2FA/OTP for high-risk roles** (Admin, Accountant) — brief has OTP, make it role-gated.
18. **Immutable, append-only audit log** (hash-chained) so tampering is detectable.
19. **Feature flags** to ship modules progressively and dark-launch AI.
20. **Soft-delete + undo everywhere**, with a restore screen.

## D. UX additions

21. **Command palette (⌘K)** for global search + quick actions (Linear-style).
22. **Keyboard shortcuts** map and a help overlay (`?`).
23. **Saved views / filters** per user on every list.
24. **Bulk actions** (approve, export, label-print) with progress + undo.
25. **Print-optimized layouts** for donor cards, labels, invoices, reports.
26. **Empty / loading / error states** designed from day one (skeletons, not spinners).

## E. AI features (deferred phase, behind a flag)

27. **Low-stock prediction** — time-series forecast per group/component from history.
28. **Demand forecasting** — seasonality + camp pipeline + hospital trends.
29. **Smart donor suggestions** — when stock of group X is low, surface eligible,
    nearby, recently-active donors of compatible groups to recall.
30. **Auto report summary** — natural-language synopsis on top of each report.
31. **Natural-language search** — "O− expiring this week" → structured query.

> AI is implemented as an optional service. The core system is fully functional
> without it; flags keep it from blocking the v1 critical path.

## F. Recommended de-scoping for a shippable v1

- WhatsApp can launch as a **stub provider** (interface ready, real API later).
- Native mobile → use the **PWA** for v1.
- Performance/attendance can be **basic** (clock-in/out + leave) in v1; expand later.

## Decision needed from product owner

- Confirm regulatory jurisdiction (defaults: India — GST + NACO report formats).
- Confirm component shelf-life values against local SOP.
- Confirm whether eligibility override is allowed at all, and by which role.
