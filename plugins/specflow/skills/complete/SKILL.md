---
name: specflow:complete
description: Retro skill — captures task outcome at completion; feeds the self-learning loop into task-history.json and decision-log.md.
status: v2-new
phase: 3
requires: [docs/specflow/features/{NNN-slug}/NNN-{slug}-tasks.md]
produces: [docs/specflow/admin/task-history.json, docs/specflow/admin/decision-log.md]
eval: entry has all required fields (id, scope, AI assistance level, what worked, what didn't, blast-radius outcome) and parses against the task-history.json schema.
---

# specflow:complete

Retro skill. Captures the outcome of a completed task and feeds it into the self-learning memory loop. Phase 3 — wires up only after the substrate (Phase 1) and the development orchestration (Phase 2) are in place to populate it.

**Triggers:**
- "/specflow:complete {task-id}" — manual invocation.
- (Future, open question O17) Linear webhook on task → Done.

**Behaviour:**
- Reads the task from `features/NNN-{slug}/tasks.md`.
- Asks a short retro: AI assistance level (`full | partial | none`), what worked, what didn't, hours estimated vs actual, regressions caught, escaped issues.
- Appends the answers to `admin/task-history.json` (schema in PRD Appendix I3).
- For significant patterns, appends an entry to `admin/decision-log.md`.

**Reuse downstream:**
- `specflow:task` and `specflow:misc` query `task-history.json` for similar past tasks during creation; surface average estimate accuracy and common gotchas.
- `/insights` (monthly) scans for recurring patterns and proposes rule-registry promotions.

**Verify steps:**
1. New entry in `task-history.json` has all required fields.
2. Entry parses against the schema.
3. If significant → corresponding entry in `decision-log.md`.

**Reference:** PRD Phase 3 scope items 1-2; Appendix I (full self-learning loop spec).
