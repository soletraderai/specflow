---
name: code-reviewer
source: comprehensive-review
version: 1.2.0
snapshot_date: 2026-04-12
upstream_path: ~/.claude/plugins/comprehensive-review/agents/code-reviewer.md
stack_match_reason: Generic code-review surface; not stack-specific
---

# code-reviewer (specialised — snapshot)

> _Snapshot pinned 2026-04-12. Refresh via `specflow:agent refresh` (drift report only)._

Specialised agent for full-pass code review on yellow-lane and red-lane tasks. Used by `specflow:develop` Phase E (Gate 5) when the standard five reviewers are insufficient and a heavier pass is warranted (high blast radius; cross-module changes).

## When this agent is dispatched

- Yellow-lane tasks where the user requested an additional reviewer beyond the standard set.
- Red-lane tasks (always; the agent runs alongside the user's own review).
- Misc-task batches where a structured second-opinion pass is requested.

## What this agent does (summary, faithful to the upstream version pinned)

- Reads the diff in full + the surrounding files for context (≤2k LOC scope cap; refuses larger).
- Surfaces concrete findings: correctness, security surfaces, performance, maintainability.
- Cites file:line for every finding; refuses vague "looks good" / "needs work" outputs.
- Flags out-of-scope observations as `misc-task` candidates rather than fixing inline (Surgical principle).

## How to refresh

Same protocol as `frontend-developer`. Drift surfaced; manual re-snapshot via `specflow:agent add code-reviewer --source comprehensive-review`.
