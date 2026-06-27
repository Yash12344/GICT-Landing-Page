# Blood Bank Management System (BBMS)

A production-grade, modular platform to run the complete daily operations of a
private blood bank — donors, collection, lab, components, inventory, requests,
issue, hospitals, camps, staff, billing, reports, analytics, and notifications —
with every entity interconnected and a full chain of custody for every unit of blood.

> **Status: Phase 0 — Planning complete. No application code yet.**
> Implementation proceeds module-by-module per the roadmap, only after the plan
> is approved.

## Planning package (read in order)

| # | Document | What it covers |
|---|----------|----------------|
| 00 | [Product Overview](docs/00-product-overview.md) | Vision, problem, personas, scope |
| 01 | [SRS](docs/01-SRS.md) | Functional + non-functional requirements |
| 02 | [Improvements](docs/02-improvements.md) | Gaps in the brief, safety fixes, AI plan |
| 03 | [Database Design](docs/03-database-design.md) | ER model, tables, indexes, normalization |
| 04 | [Folder Structure](docs/04-folder-structure.md) | Monorepo layout |
| 05 | [UI Wireframes](docs/05-ui-wireframes.md) | ASCII wireframes for key screens |
| 06 | [User Flows](docs/06-user-flows.md) | Step-by-step critical journeys |
| 07 | [API Documentation](docs/07-api-documentation.md) | REST surface, auth, conventions |
| 08 | [Roadmap](docs/08-roadmap.md) | Phased delivery plan + Definition of Done |

## Tech stack
**Frontend:** Next.js · TypeScript · TailwindCSS · shadcn/ui · React Query · Zustand · Framer Motion
**Backend:** Node.js · Express · Prisma · PostgreSQL · Redis · BullMQ
**Storage/Auth/Deploy:** S3-compatible · JWT + refresh tokens · Docker · Nginx

## Design language
White background · red accent `#E53935` · rounded cards · soft shadows · excellent
typography · fast animations · dark mode · fully responsive. Reference bar: Stripe,
Linear, Vercel, Supabase.

## The core safety contract
A unit of blood can **never** reach inventory or a patient unless it has passed all
TTI tests and an ABO/Rh-compatible cross-match. Every state change is recorded in an
immutable audit log, and inventory is driven by an append-only ledger.

## What's next
Approve the plan → **Phase 1: Foundation & Scaffolding** (monorepo, Docker, Prisma
schema + seed, authenticated app shell). See the [roadmap](docs/08-roadmap.md).
