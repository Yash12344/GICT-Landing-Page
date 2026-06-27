# UI Wireframes — BBMS

Low-fidelity ASCII wireframes establishing layout, hierarchy, and component
placement. Visual language: white bg, red accent `#E53935`, rounded-2xl cards,
soft shadows, generous whitespace, dark-mode mirror. Reference quality: Stripe /
Linear / Vercel.

## Global App Shell

```
┌────────────────────────────────────────────────────────────────────────┐
│ [≡] BloodBank ▸ Branch ⌄        ⌘K Search…           🔔3  ◑ dark  ⬤ User⌄│  top bar
├──────────────┬─────────────────────────────────────────────────────────┤
│ ◆ Dashboard  │                                                          │
│ ◆ Donors     │                   MAIN CONTENT AREA                      │
│ ◆ Collection │                  (module pages render here)              │
│ ◆ Lab        │                                                          │
│ ◆ Components │                                                          │
│ ◆ Inventory  │                                                          │
│ ◆ Patients   │                                                          │
│ ◆ Requests ●3│   ● = badge (emergency/pending counts)                   │
│ ◆ Issue      │                                                          │
│ ◆ Hospitals  │                                                          │
│ ◆ Camps      │                                                          │
│ ◆ Billing    │                                                          │
│ ◆ Reports    │                                                          │
│ ◆ Analytics  │                                                          │
│ ◆ Staff      │                                                          │
│ ◆ Settings   │                                                          │
│ ────────────  │                                                          │
│ Pinned ▸     │                                                          │
└──────────────┴─────────────────────────────────────────────────────────┘
```
Sidebar collapses to icons on tablet, becomes a drawer on mobile. Nav items are
filtered by the user's role permissions.

## Dashboard

```
 Good morning, Dr. Sharma                         [+ Quick Action ⌄]
 ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
 │Today's   │ │Issued    │ │Requests  │ │Emergency │ │Expiring  │
 │Collection│ │  18      │ │  24      │ │   3 ●    │ │ ≤7d  11  │
 │  42 ▲12% │ │  ▲4%     │ │  ▼2%     │ │  red     │ │  amber   │
 └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘
 ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
 │Pending   │ │Inventory │ │Revenue   │ │Active    │
 │Tests  7  │ │ 312 units│ │ ₹84,200  │ │Camps  2  │
 └──────────┘ └──────────┘ └──────────┘ └──────────┘

 ┌───────────────────────────────┐ ┌──────────────────────────────┐
 │ Weekly Collection (bar)       │ │ Demand by Blood Group (bar)  │
 │   ▁▃▅▂▇▆▄                      │ │ O+ ████  A+ ███  B+ ██ ...   │
 └───────────────────────────────┘ └──────────────────────────────┘
 ┌───────────────────────────────┐ ┌──────────────────────────────┐
 │ Inventory Trend (line)        │ │ Component Distribution (pie) │
 └───────────────────────────────┘ └──────────────────────────────┘

 ┌───────────────────────────────┐ ┌──────────────────────────────┐
 │ Recent Activity (timeline)    │ │ Tasks / Pinned Reports       │
 │ • Bag B-1042 issued  2m       │ │ ☐ Approve 3 requests         │
 │ • Lab approved B-1039 14m     │ │ ☐ Discard 2 expired units    │
 └───────────────────────────────┘ └──────────────────────────────┘
```

## List view pattern (Donors / Patients / Requests / …)

```
 Donors                                 [Import] [Export] [+ Add Donor]
 ┌ Search… ─────────┐ [Group ⌄] [Status ⌄] [City ⌄] [Eligibility ⌄] [Saved ⌄]
 ┌─────────────────────────────────────────────────────────────────────┐
 │ ☐ │ Photo │ Donor ID │ Name      │ Group │ Last Donation │ Status   │
 │ ☐ │  🙂   │ D-00123  │ R. Mehta  │ O+    │ 2026-03-12    │ ●Eligible│
 │ ☐ │  🙂   │ D-00124  │ S. Khan   │ B−    │ 2026-05-01    │ ●Deferred│
 └─────────────────────────────────────────────────────────────────────┘
 [bulk: Export ▾  Print Cards ▾]              ◀ 1 2 3 … ▶  25 / page ⌄
```
Row click → side drawer or detail page. Sticky header, column sort, skeleton
loading rows, empty-state illustration when no data.

## Donor detail / form (drawer)

```
 ┌ Donor: R. Mehta (D-00123) ──────────────────────────── [Print Card] [✕] ┐
 │ [Photo]  O+ · 34y · Male              ●Eligible · Next: 2026-06-10        │
 │ ┌ Tabs: Overview | Donations | Medical | Documents ┐                     │
 │ Mobile 98xxxxxx · Email r@x.com · Karanpur, Sri Ganganagar               │
 │ Donations: 7   Last: 2026-03-12   Weight: 68kg                           │
 │ Donation history table…                                                  │
 │                                          [Edit]  [Schedule Donation]     │
 └──────────────────────────────────────────────────────────────────────────┘
```

## Inventory (color-coded grid)

```
 Inventory  (Branch: Sri Ganganagar)                      [Transfer] [Adjust]
        │ Whole │ PRBC │ Platelets │ FFP │ Cryo │  ← component columns
  O+    │  12 🟢│ 20 🟢│   3 🔴    │ 8 🟢│ 5 🟢 │
  O−    │   2 🔴│  4 🟡│   1 🔴    │ 3 🟡│ 2 🟡 │   🟢 ok  🟡 low  🔴 critical
  A+    │   9 🟢│ 15 🟢│   5 🟢    │ 6 🟢│ 4 🟢 │
  ...                                                expiring ≤7d highlighted
 Legend: Available / Reserved / Expiring soon          [Expiry queue →]
```

## Blood Issue (barcode-driven)

```
 Issue Blood — Request REQ-2207 (CRITICAL ●)
 Patient: A. Verma · O+ · City Hospital · Dr. Rao
 Required: PRBC × 2
 ┌ Scan or enter bag barcode ───────────────┐ [Scan 📷]
 │ ███ B-1042                                │
 └──────────────────────────────────────────┘
  Compatibility ✓ O+  · Cross-match ✓ Compatible · Expiry 2026-07-30 ✓
 ┌ Selected units ──────────────────────────────────────────┐
 │ B-1042  PRBC  O+  ✓     B-1051  PRBC  O+  ✓               │
 └──────────────────────────────────────────────────────────┘
                              [Cancel]  [Confirm Issue → invoice]
```
Guardrails surface inline: incompatible group, expired, no cross-match → blocked
with a clear red reason; safe units show green checks.

## Lab Testing

```
 Lab — Bag B-1039  (O+, collected 2026-06-26)            ●Quarantine
 HB [13.2]  HIV (○Neg ●Reactive)  HBsAg (●Neg)  HCV (●Neg)
 Malaria (●Neg)  Syphilis (●Neg)  Blood Typing [O+]
 Technician: T. Singh   Doctor sign-off: Dr. Sharma
 Comments […]
 If any reactive → result locks to REJECTED + discard.   [Reject] [Approve ✓]
```

## Auth screens

```
 ┌─────────────── Sign in ───────────────┐   ┌──── Verify OTP ────┐
 │  🩸 BloodBank                          │   │  Enter 6-digit code │
 │  Email   [__________________]          │   │  [_][_][_][_][_][_] │
 │  Password[__________________] 👁        │   │  Resend in 0:27     │
 │  [ ] Remember     Forgot password?     │   │     [Verify]        │
 │            [ Sign in ]                  │   └────────────────────┘
 └────────────────────────────────────────┘
```

## Responsive & states
- **Mobile:** single column, sticky bottom action bar, sheet dialogs.
- **States:** every screen designs skeleton (loading), empty (illustration + CTA),
  and error (retry) states. Toasts for success, inline red text for validation,
  undo snackbar for deletes.
- **Command palette (⌘K):** fuzzy search entities + run quick actions.
