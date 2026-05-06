---
name: database-architect
source: comprehensive-review
version: 1.2.0
snapshot_date: 2026-04-12
upstream_path: ~/.claude/plugins/comprehensive-review/agents/database-architect.md
stack_match_reason: Detected PostgreSQL + Drizzle ORM via environment.json
---

# database-architect (specialised — snapshot)

> _Snapshot pinned 2026-04-12. Refresh via `specflow:agent refresh` (drift report only)._

Specialised agent for schema design, migration review, and query analysis. Matched to this project's stack via `environment.json` (PostgreSQL 16, Drizzle ORM detected; migrations live at `apps/web/drizzle/`).

## When this agent is dispatched

- Tasks whose scope intersects `apps/web/drizzle/` or modifies any table definition file.
- Yellow-lane tasks where the user requested schema-aware review.
- Red-lane tasks touching `confidentialPaths` matched to billing or auth-token tables.

## What this agent does (summary, faithful to the upstream version pinned)

- Reviews schema changes for backwards-compatibility, locking implications, and index coverage.
- Validates migration scripts against the rollback contract: every `up` has a viable `down`; destructive operations call out the consent gate.
- Surfaces query-plan concerns when changes touch hot paths (joins on unindexed columns, N+1 patterns).
- Refuses to author schema directly — proposes via PR comments and lets the user run the migration.

## How to refresh

Same protocol as siblings.
