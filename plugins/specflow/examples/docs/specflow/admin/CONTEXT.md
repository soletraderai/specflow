# Project context

> Slim live context doc for **Northwind Orders**. How this project actually works. Updated by humans, by `feedback-loop-audit`, and incidentally by other skills as they learn things worth recording.

This file is loaded into context for skills that need a high-level understanding of the project before they touch the codebase. Target 500 lines, hard limit 700. When it grows past that, distill rather than append.

---

## How this project works

Northwind Orders is an internal order-management dashboard used by ~120 operations, support, and warehouse staff across three regional offices. Customers do not log in here — this is a back-office tool. The product replaced a legacy desktop app over the last 18 months; some workflows are still being ported.

Two audiences dominate the design conversation:

- **Power users** (ops coordinators, warehouse leads) live in the app for hours a day. They've memorised URL patterns and keyboard shortcuts. UI churn costs them productivity.
- **Occasional users** (support agents, new hires, finance during close) need the app to be discoverable without training. They optimise for *recovery from mistakes* over speed.

The sharpest design tension is keeping power-user workflows fast (bulk actions, keyboard navigation, no-modal flows) without making the app illegible to occasional users. We resolve this with a layered surface: defaults are discoverable, power moves are reachable by keyboard or URL.

Engineering culture is small-team and trust-based. Five engineers, one designer, one PM. We ship daily on weekdays, never on Fridays. Decision-log entries are written by whoever made the call, not assigned out. Surgical changes is the load-bearing principle — we got burned twice by "while I was here..." refactors that turned into outages.

## Stack

- **Frontend.** Next.js 14 (App Router), TypeScript, Tailwind 3, internal component library at `packages/ui/`. No external design system; tokens live in `packages/ui/src/tokens/`.
- **Backend.** Next.js API routes, Prisma 5, Postgres 15. No microservices; one repo, one deployment unit.
- **Testing.** Vitest for unit + integration, Playwright for e2e. Tests sit alongside source in `__tests__/` directories — never in a top-level `tests/` tree (see `rules/guidelines.md`).
- **Auth.** Custom JWT-based auth in `src/auth/`. Red-lane per `config.json.confidentialPaths`.
- **Deployment.** Vercel for the dashboard, Postgres on Neon. Migrations gated behind a manual approval step in CI.
- **Tooling.** GitHub for source + reviews, Linear for tracking, Slack-free (decision logged 2026-03-22).

## Conventions

These are the patterns that surprise newcomers — the ones you only learn by stepping on a rake.

### Tests sit alongside code

Every module has its tests in a sibling `__tests__/` directory. Top-level `tests/` is reserved for cross-module integration suites that don't belong to any single module. This convention is enforced by `rules/guidelines.md` (PREFER_LOCAL_TESTS) and is the preferred shape unless a test genuinely spans modules.

### Composition over inheritance in services

The `src/services/` tree uses composition (small functions injected with their dependencies) rather than class hierarchies. We tried a `BaseService` class in 2025 Q4 and removed it three months later — see decision-log entry 2026-01-08. New services should copy the pattern in `src/services/orders/orders-service.ts`.

### One-way data flow in the orders detail page

The orders detail page is the most complex surface in the product. State flows top-down from a single `useOrderDetail` hook in `app/orders/[orderId]/page.tsx`. Sibling components do NOT cross-import each other's state — if two components need shared state, lift it to the page hook. This rule was added after a bug where two components held divergent copies of the order's status field.

### Audit-trail writes happen via the `audit` helper, not inline

Any state-changing action must call `src/audit/log.ts`'s helper, which writes the action, the actor, the timestamp, and the diff. Inline `console.log` or ad-hoc audit is rejected at code review.

### Design tokens are extracted, never invented

`packages/ui/src/tokens/` is the single source of truth for colours, typography, spacing, and component shape. Specflow's `design` skill (see `features/001-design-skill/`) extracts from this tree directly — never approximates. This convention is the reason we adopted the codebase-truth principle for design mockups in the first place.

## Confidential paths

The Red lane is rule-based, never AI-rated. The following globs (mirrored in `config.json.confidentialPaths`) require human-led work:

- `src/billing/**` — invoice generation, refund flows, ledger writes.
- `src/auth/**` — authentication, authorisation, session lifetime.
- `src/migrations/**` — Prisma migrations and data backfills.
- `infrastructure/**` — Terraform, deployment configs.
- `ops/runbooks/**` — incident response procedures.
- `.env*`, `secrets/**` — credentials and per-environment values.

Skills that touch any of these surfaces MUST escalate to the Red lane and surface the work item for human review before proceeding. See `rules/non-negotiable.md` rule `PROTECTED_PATHS_REQUIRE_RED_LANE`.

## Recent context

Recent context entries surface decisions, incidents, or pattern shifts that affect how skills should reason about this codebase right now. They expire when the underlying state changes; older entries get distilled into the conventions section.

- **2026-05-02 — `specflow:design` adopted.** The first feature shipped on the new design skill is the order-detail filter rework. The mockup loop converged in 4 iterations against `packages/ui/src/tokens/`. See `features/001-design-skill/` for the worked example.
- **2026-04-19 — Bulk actions on orders list.** New affordance pattern: floating action bar appears when one or more rows are selected. Power users adopted it within a day; first-time visitors discovered it within a week (per support tickets). Pattern is documented in `packages/ui/src/components/action-bar/README.md`.
- **2026-04-03 — `BaseService` removal completed.** All services migrated off the base class. Composition pattern is now the only sanctioned shape. See decision-log entry 2026-01-08 for the rationale.
- **2026-03-22 — Slack removed from the stack.** Decision-log entry of the same date. Skills that previously referenced Slack (e.g. release announcements) should use Linear comments + GitHub releases instead.
- **2026-03-10 — Playwright pinned to 1.43.x.** A 1.44 upgrade broke our visual diff baselines. Until we re-baseline, Playwright stays on 1.43.x.

## Where to look next

- `docs/specflow/admin/decision-log.md` — *why* this project made its non-obvious choices.
- `docs/specflow/admin/rules/non-negotiable.md` — hard rules every skill respects.
- `docs/specflow/admin/rules/guidelines.md` — soft rules tuned to this project's taste.
- `docs/specflow/admin/rules/glossary.md` — one-line summary of every rule with the *why*.
- `docs/specflow/admin/profiles.json` — the five user profiles this product is designed for.
- `docs/specflow/admin/pages.json` — the page inventory and which profiles each page targets.
- `docs/specflow/features/` — feature folders, one per work item; see `001-design-skill/` for a fully worked example.
- `docs/specflow/admin/task-history.json` — completed work with what-worked / what-didn't notes.

---

*This file is meant to be read end-to-end in under 5 minutes. If it can't be, distill it.*
