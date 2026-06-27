# Folder Structure — BBMS Monorepo

A pnpm + Turborepo monorepo: one repo, isolated apps and shared packages.

```
bbms/
├─ docs/                      # this planning package
├─ apps/
│  ├─ web/                    # Next.js 14 (App Router) frontend
│  │  ├─ src/
│  │  │  ├─ app/              # routes (App Router)
│  │  │  │  ├─ (auth)/        # login, forgot-password, otp
│  │  │  │  ├─ (dashboard)/   # authenticated shell + all modules
│  │  │  │  │  ├─ dashboard/
│  │  │  │  │  ├─ donors/
│  │  │  │  │  ├─ collection/
│  │  │  │  │  ├─ lab/
│  │  │  │  │  ├─ components/
│  │  │  │  │  ├─ inventory/
│  │  │  │  │  ├─ patients/
│  │  │  │  │  ├─ requests/
│  │  │  │  │  ├─ issue/
│  │  │  │  │  ├─ hospitals/
│  │  │  │  │  ├─ camps/
│  │  │  │  │  ├─ staff/
│  │  │  │  │  ├─ billing/
│  │  │  │  │  ├─ reports/
│  │  │  │  │  ├─ analytics/
│  │  │  │  │  └─ settings/
│  │  │  │  ├─ layout.tsx
│  │  │  │  └─ globals.css
│  │  │  ├─ components/       # ui/ (shadcn), shared widgets, charts, command-palette
│  │  │  ├─ features/         # feature modules: api hooks + components co-located
│  │  │  ├─ lib/              # api client, auth, utils, formatters
│  │  │  ├─ stores/           # Zustand stores (ui, auth-session, drafts)
│  │  │  ├─ hooks/            # shared React hooks
│  │  │  └─ styles/           # tailwind theme tokens, dark mode
│  │  ├─ public/              # icons, manifest (PWA), offline page
│  │  ├─ next.config.mjs
│  │  └─ tailwind.config.ts
│  │
│  ├─ api/                    # Express + Prisma backend
│  │  ├─ src/
│  │  │  ├─ config/           # env (zod-validated), db, redis, logger
│  │  │  ├─ middleware/       # auth, rbac, validate, rateLimit, error, audit
│  │  │  ├─ modules/          # one folder per domain module
│  │  │  │  ├─ auth/          # controller, service, routes, schema, tests
│  │  │  │  ├─ donor/
│  │  │  │  ├─ collection/
│  │  │  │  ├─ lab/
│  │  │  │  ├─ component/
│  │  │  │  ├─ inventory/
│  │  │  │  ├─ patient/
│  │  │  │  ├─ request/
│  │  │  │  ├─ issue/
│  │  │  │  ├─ hospital/
│  │  │  │  ├─ camp/
│  │  │  │  ├─ staff/
│  │  │  │  ├─ billing/
│  │  │  │  ├─ report/
│  │  │  │  ├─ notification/
│  │  │  │  ├─ analytics/
│  │  │  │  └─ settings/
│  │  │  ├─ domain/           # pure logic: eligibility, abo-compat, expiry, pricing
│  │  │  ├─ jobs/             # BullMQ workers: expiry, lowStock, reminders, reports
│  │  │  ├─ lib/              # barcode, pdf, excel, storage(s3), mailer, sms, whatsapp
│  │  │  ├─ app.ts            # express app wiring
│  │  │  └─ server.ts         # bootstrap
│  │  ├─ prisma/
│  │  │  ├─ schema.prisma
│  │  │  ├─ migrations/
│  │  │  └─ seed.ts
│  │  └─ tests/               # integration tests
│  │
│  └─ worker/                 # (optional) dedicated BullMQ worker process
│
├─ packages/
│  ├─ types/                  # shared TS types / zod schemas (API contract)
│  ├─ config/                 # eslint, tsconfig, prettier presets
│  └─ ui/                     # (optional) shared component primitives
│
├─ infra/
│  ├─ docker/                 # Dockerfiles (web, api, worker)
│  ├─ nginx/                  # reverse proxy + TLS config
│  ├─ docker-compose.yml      # local: web, api, worker, postgres, redis, minio
│  └─ docker-compose.prod.yml
│
├─ .github/workflows/         # CI: lint, typecheck, test, build, docker
├─ turbo.json
├─ package.json
└─ pnpm-workspace.yaml
```

## Module convention (backend)
Each `modules/<name>/` contains:
`<name>.routes.ts` · `<name>.controller.ts` · `<name>.service.ts` ·
`<name>.schema.ts` (zod) · `<name>.test.ts`. Controllers are thin; business logic
lives in services; cross-cutting clinical rules live in `domain/` and are unit-tested
in isolation.

## Frontend convention
Feature-first: `features/<name>/` holds React Query hooks (`api.ts`), components,
and types for that module, consumed by the route under `app/(dashboard)/<name>`.
Shared, dumb UI primitives stay in `components/ui` (shadcn).

## Why a monorepo
- Single source of truth for the API contract (`packages/types`) shared by web + api.
- Atomic PRs that change backend + frontend together.
- Turborepo caches lint/test/build across apps.
