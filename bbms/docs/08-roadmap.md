# Development Roadmap — BBMS

Module-by-module delivery. Each phase is independently shippable, tested, and
demoable. "Definition of Done" (DoD) applies to every phase.

## Definition of Done (every phase)
- Typed end-to-end (zod schema in `packages/types`, Prisma model, React Query hook).
- Input validation + RBAC on every endpoint.
- Unit tests on domain logic; integration test on the happy path + key guard.
- Loading / empty / error UI states; responsive; dark-mode verified.
- Audit logging for writes; migration + idempotent seed updates.
- Lint + typecheck + tests green in CI.

---

## Phase 0 — Planning ✅ (this PR)
SRS, improvements, DB design, folder structure, wireframes, user flows, API docs,
roadmap. **No code.**

## Phase 1 — Foundation & Scaffolding
- Monorepo (pnpm + Turborepo), `apps/web`, `apps/api`, `packages/types`, `infra/`.
- Docker Compose: postgres, redis, minio, api, web, worker.
- Prisma schema (all core tables) + first migration + seed (roles, permissions,
  blood groups, component shelf-life, super admin, demo branch).
- Express app skeleton: config, logger, error handler, middleware stubs, health check.
- Next.js app shell: layout, sidebar, top bar, theme tokens, dark mode, shadcn setup.
- CI workflow (lint/typecheck/test/build).
- **Demo:** containers boot; seeded DB; empty authenticated shell renders.

## Phase 2 — Auth & RBAC
- Login, refresh, logout, forgot/reset password, OTP (role-gated MFA).
- JWT + rotating refresh (httpOnly), argon2 hashing, lockout, rate limit.
- RBAC middleware + permission-aware frontend nav and route guards.
- Settings: users, roles, permission matrix.
- **Demo:** each role logs in and sees only permitted modules.

## Phase 3 — Donor Management
- Donor CRUD, photo upload (S3/minio), eligibility engine (unit-tested), deferrals.
- Search/filters/saved views, CSV import/export, donor card PDF (QR), history.
- **Demo:** register, defer, print card, import a batch.

## Phase 4 — Collection + Lab + Components (the clinical core)
- Collection records, vitals gate, bag state machine → quarantine.
- Lab panel, reactive ⇒ auto-reject + discard + lookback, doctor sign-off.
- Component separation with per-component expiry, barcode/QR labels.
- **Demo:** bag from collection → tested → separated → components created.

## Phase 5 — Inventory
- Ledger + snapshot, real-time grid, color-coded thresholds.
- Expiry job (BullMQ), low-stock alerts, expiry queue, adjust/transfer.
- **Demo:** components appear as Available; expiry job moves a unit to Expired.

## Phase 6 — Patients, Requests, Cross-match, Issue, Billing
- ABO/Rh compatibility engine (unit-tested), reservations.
- Request lifecycle + approval + emergency surfacing.
- Cross-match, barcode-driven issue with all guards, auto-invoice.
- Billing: invoices, payments, GST, daily cash, hospital outstanding.
- **Demo:** full spine — request → approve → cross-match → issue → invoice → pay.

## Phase 7 — Hospitals, Camps, Staff
- Hospital + doctors + request history + outstanding.
- Camps: CRUD, linked collections, stats, expenses, revenue, photos.
- Staff: profiles, attendance, leave, activity logs.
- **Demo:** run a camp; hospital billing summary; staff clock-in.

## Phase 8 — Reports & Analytics
- Report engine (daily/…/government) with PDF/Excel/CSV export (queued for large).
- Analytics: trends, heatmap, top hospitals/donors, revenue, growth.
- Dashboard charts wired to real data; pinned reports & favorites.
- **Demo:** export a monthly finance PDF; dashboard charts live.

## Phase 9 — Notifications & Realtime
- BullMQ notification pipeline; email + SMS providers; WhatsApp stub.
- Triggers: low stock, expiry, donor/camp reminders, emergency request.
- SSE/WS live dashboard updates; in-app notification center.
- **Demo:** low-stock + emergency notifications delivered and shown live.

## Phase 10 — Hardening, PWA, Settings, Deploy
- Security pass (helmet, CSRF, rate limits, audit hash-chain, pen-test checklist).
- PWA (manifest, offline drafts, installable), keyboard shortcuts, command palette polish.
- Backup/restore, integration settings, themes.
- Dockerized prod compose + Nginx + TLS; load test against NFR targets.
- **Demo:** install as PWA; production stack up behind Nginx.

## Phase 11 — AI (optional, feature-flagged)
- Low-stock prediction, demand forecast, smart donor suggestions, NL search,
  auto report summaries. Shipped dark-launched; core unaffected if disabled.

---

## Sequencing rationale
Foundation → auth → donor → **clinical core** → inventory → request/issue/billing
follows the real-world unit lifecycle, so every phase produces something genuinely
usable and each builds on a working predecessor. Cross-cutting concerns (audit,
notifications, reports) are layered once the entities they observe exist.

## Risks & mitigations
- **Clinical safety bugs** → pure, unit-tested domain functions + DB-level triggers as backstop.
- **Inventory drift** → ledger as source of truth + nightly reconciliation of snapshot.
- **Scope creep** → feature flags + strict per-phase DoD; AI and WhatsApp deferred.
- **Money correctness** → integer paise + idempotency keys + integration tests.

## Suggested next action
Approve this plan, then I begin **Phase 1 (Foundation & Scaffolding)** — monorepo,
Docker, Prisma schema + seed, and the empty authenticated app shell — committed to
`claude/blood-bank-management-system-o8eor3`.
