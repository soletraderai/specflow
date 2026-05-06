---
name: frontend-developer
source: frontend-mobile-development
version: 1.4.2
snapshot_date: 2026-04-12
upstream_path: ~/.claude/plugins/frontend-mobile-development/agents/frontend-developer.md
stack_match_reason: Detected React + TypeScript stack via environment.json
---

# frontend-developer (specialised — snapshot)

> _This is a snapshot pinned in-repo on 2026-04-12. Refresh via `specflow:agent refresh` (drift report only — never auto-overwrites; user re-snapshots per agent on confirmation)._

Specialised agent for React + TypeScript frontend implementation tasks. Matched to this project's stack via `environment.json` (React 18, TypeScript 5.x, Tailwind CSS detected).

## When this agent is dispatched

`specflow:develop` Phase D (lane execution) dispatches this agent on green-lane tasks whose scope intersects `apps/web/`, `packages/ui/`, or any component-shaped path detected by the project's stack inventory. Yellow-lane tasks pair the agent with the user via the agent-teams plugin's HITL primitive.

## What this agent does (summary, faithful to the upstream version pinned)

- React component implementation following the project's existing patterns (function components + hooks; Tailwind utility classes; props interface adjacent to the component).
- TypeScript type-safety with strict mode; prefers narrow types over `any`; reuses existing type aliases where present.
- Test scaffolds via the project's detected runner (Vitest / Jest); colocates tests in `__tests__/` per the project's guideline.
- Refuses to introduce new dependencies without flagging via `specflow:misc --auto`.

## How to refresh this snapshot

`specflow:agent refresh` reports drift between this snapshot and the upstream plugin version. To accept upstream changes: `specflow:agent add frontend-developer --source frontend-mobile-development` re-snapshots verbatim. Manual edits to this file are preserved across refresh runs (drift surfaced for human review).
