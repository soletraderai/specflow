# Single context window per task implementation

When `specflow:develop` enters a task's Red → Green → Refactor cycle (per 017-tdd-discipline), the entire cycle runs in **one agent context window**. The agent loads what it needs once at the start and runs the task to completion — or escalates — within that window. Mid-task compaction is forbidden; cross-session resumption mid-implementation is forbidden. Tasks that cannot fit are split at synthesis time, not patched mid-execution.

Introduced in v2.5.0 (`029-single-context-task`). Codifies locked-in architectural decision #21.

## Why this exists

Two converging signals from the research dataset make the rule load-bearing:

- **Context rot is the dominant agent failure mode** (`knowledge/medin-wisc-framework.md` §S — Select). Every additional token degrades retrieval; raising the ceiling to 1M tokens does not fix needle-in-the-haystack collapse — engineering the window does. The discipline is *less* context, loaded just-in-time.
- **The smart-zone cliff sits at ~100K tokens regardless of context-window size** (`knowledge/pocock-ai-coding-real-engineers.md` §LLM Constraint 1). 1M-token windows are mostly dumb-zone for coding work; the smart zone is still ~100K. Compaction during a task drops the agent through the cliff and back into dumb-zone reasoning *with the bookkeeping artefact of having "preserved context"* — the worst-of-both outcome.

Without the rule, develop drifts into the failure mode this skill exists to remove: silent mid-task compaction that degrades the agent's reasoning while the manifest reads "still in progress." With the rule, oversized tasks get caught at synthesis and split via `specflow:scope-change` *before* code execution begins.

## The rule (verbatim — locked-in decision #21)

> **Single context window per task implementation** — every task's Red/Green/Refactor runs in one agent context window (no mid-task compaction; no cross-session resumption mid-implementation). Tasks too big to fit are split at synthesis time, not patched mid-execution. Compaction during develop is a defect signal, not a recovery move (Sprint 2 — 029).

## `context-budget-estimate` schema

Every task entry in `{NNN-slug}-tasks.md` carries a `context-budget-estimate` field, in tokens.

| Field component | What it estimates | How to estimate |
|---|---|---|
| PRD slice | Tokens consumed loading the R/AC anchors the task cites | Count chars in the cited R + AC blocks ÷ 4 |
| Task spec | The task's own tasks-file entry | Count chars in the task block ÷ 4 |
| Prior lessons | Lessons surfaced via 018-lessons-registry query | Sum chars of matched lesson bodies ÷ 4 |
| Per-task manifest scaffold | The 019-task-manifest header + read-first entries | Fixed ~2K tokens reserved |
| Codebase-context payload | Files the task will read during execution | Sum file sizes ÷ 4 (use Read-tool quotas) |
| Test plan | The 017-tdd-discipline Red plan | Fixed ~1.5K tokens reserved |

Default budget: **80,000 tokens** (configurable via `config.json.task.contextBudget`). The default sits inside the smart zone with headroom for tool-call output. Tasks whose estimate exceeds the budget are auto-flagged at synthesis and cannot proceed to develop without re-synthesis.

Recompute the estimate at three checkpoints:

1. **At task synthesis** (`specflow:task` Phase B) — initial value lands on the task entry.
2. **At cross-task review** (022) — the "better arrangement" lens may merge or split tasks; estimates re-computed for affected tasks only.
3. **At develop Phase A pre-flight** — the loaded context is measured against the estimate; ≥20% divergence triggers a developer prompt (see *How develop Phase A pre-flights* below).

## No-mid-task-compaction contract

Compaction during Phases D / E / F is treated as a **defect signal**, not a recovery move. Possible interpretations of a compaction event mid-task:

- The estimate at synthesis was wrong (most common — fix at synthesis next time).
- The PRD slice loaded was over-broad (load only cited R/AC, not the whole PRD).
- The codebase-context payload pulled files the task does not actually read (just-in-time, not just-in-case).
- The task should have been split (route to `specflow:scope-change`).

The agent's response when context approaches the cliff is to **escalate to the developer**, not compact. The developer chooses: approve the over-run (logged as a `decision-log.md` entry), drop optional context (e.g. lessons that did not influence the plan), or route the task to `specflow:scope-change` for splitting. Auto-compaction is the failure mode this rule exists to remove.

A run that compacts mid-task is a failed run. The manifest entry under 019-task-manifest records the compaction event with `event_type: escalation` and outcome `compacted-mid-task` so the retro can identify the synthesis-time sizing error.

## How task-synthesis applies the rule

`specflow:task` Phase B writes `context-budget-estimate: <int_tokens>` on every task entry. After Phase B.4's coverage matrix self-check, a budget self-check runs:

- For each task, compute the estimate per the schema above.
- If estimate ≤ `config.json.task.contextBudget`, the task is fit-for-develop.
- If estimate > budget, the task is **auto-flagged for split** — an inline note appears in `tasks.md` under the task block (`> Budget overrun: estimate {N}K vs budget {M}K — split required before develop.`), and synthesis emits a chat-line prompt directing the user to `specflow:scope-change` to recut the task. The task remains in the file with the warning so the recut is auditable.

The split successors carry `context-budget-estimate` fields each fitting the budget, and inherit the original task's `prior-lessons` and `sprint-bucket` fields (re-evaluated when the new shape changes the bucket assignment per 025).

## How develop Phase A pre-flights

`specflow:develop` Phase A reads each in-scope task's `context-budget-estimate` against the actually-loaded context measured at the boundary between Phase A.5 (announce) and Phase B (lane triage). Three outcomes:

- **Estimate within ±20% of actual** → proceed silently to Phase B.
- **Actual exceeds estimate by ≥20%** → pause and prompt the developer with three options: (a) approve the over-run (logged), (b) drop optional context (the developer names which payload component to drop — typically lessons or codebase-context files), (c) route the task to `specflow:scope-change` for splitting.
- **Actual exceeds the configured budget outright** → refuse to enter Phase B for that task; route to `specflow:scope-change` non-optionally.

The pre-flight closes the loop between synthesis-time estimation and execution-time reality. Repeated divergences feed the retro (via 018-lessons-registry) and tighten future estimates.

## Cross-references

- **017 — tdd-discipline** — Red/Green/Refactor is the unit the single-context window covers.
- **019 — task-manifest** — compaction events log as escalation entries; the per-task manifest is loaded once at task entry, not re-loaded mid-task.
- **022 — cross-task review** — its "better arrangement" lens consults `context-budget-estimate` to detect oversized tasks the per-task lens missed.
- **025 — sprint-task-flagging** — `sprint-bucket: N` respects per-task budgets; tasks in the same bucket each run in their own window, but the bucket sizing is informed by per-task budgets.
- **020 — sprint-skill** — the sprint-plan gate visualises per-task budgets so the developer sees risk at-a-glance.
- **026 — agent-teams-per-stage** — the Build team operates inside the single-window contract; team-spawn does not waive the rule.

## Worked example

See `examples/docs/specflow/features/029-single-context-task/`.

## Resolution citation

`v2/docs/SESSION-HANDOFF.md` § Architectural decisions — locked in #21.
