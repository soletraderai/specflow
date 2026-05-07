---
feature: 029-single-context-task
status: drafted
created: 2026-05-07
templateVersion: v2.5
interview: ./029-single-context-task-interview.md
---

# 029 — Single context window per task implementation

## Vision

Codify the rule that every task's Red → Green → Refactor cycle (per 017-tdd-discipline) runs in a **single agent context window**. The agent loads what it needs once at task entry and runs to completion — or escalates — within that window. Mid-task compaction is forbidden; cross-session resumption mid-implementation is forbidden. Tasks that cannot fit are split at synthesis time, not patched mid-execution. The rule is doctrine, not a knob: compaction during develop is treated as a defect signal that the synthesis estimate was wrong, and the recovery path is escalation + re-synthesis, never silent context shrinkage. The implementation is minimal: one doctrine doc carries the contract, `specflow:task` adds a `context-budget-estimate` field on every task, and `specflow:develop` Phase A adds a pre-flight that catches estimate-vs-actual divergence before code execution begins.

## Problem

`specflow:develop` currently has no token-budget pre-flight. Phase A reads the PRD, tasks file, and Gate 3 manifest; Phase B triages lanes; Phase D executes. Nothing measures the loaded context against a budget; nothing flags a task whose actual context will exceed the smart-zone cliff. When the cliff is hit mid-task, the default LLM behaviour is silent compaction — which preserves *bookkeeping* (the manifest still reads "in progress") while degrading the agent's reasoning into the dumb zone (per Pocock's empirical ~100K marker). The compaction is invisible at the manifest layer; only the retro catches it (often as "the task drifted at the end"), and by then the lesson is too late to feed forward.

Two converging research signals make the problem load-bearing. **Context rot is the dominant agent failure mode** (Inspiration: `knowledge/medin-wisc-framework.md` — context rot section): every additional token degrades retrieval, and raising the ceiling to 1M tokens does not fix needle-in-the-haystack collapse. **The smart-zone cliff sits at ~100K tokens regardless of context-window size** (Inspiration: `knowledge/pocock-ai-coding-real-engineers.md` § LLM Constraint 1): 1M-token windows are mostly dumb-zone for coding work; the smart zone is still ~100K. Together they imply: size every task to fit inside the smart zone, load just-in-time not just-in-case, and refuse mid-task compaction because compaction drops the agent through the cliff with the bookkeeping artefact of having "preserved context" — the worst-of-both outcome.

The fix is a doctrine-and-pre-flight pair, not a runtime token counter. Synthesis estimates the budget per task; develop pre-flights the actual load against the estimate at Phase A; the doctrine doc spells out what the agent does when context approaches the cliff (escalate, never compact). Tasks too big to fit are caught at synthesis and split via `specflow:scope-change` before code execution begins.

## Goals

- Doctrine doc at `plugins/specflow/templates/admin/single-context-task.md` defining: rationale, the verbatim rule, `context-budget-estimate` schema, no-mid-task-compaction contract, how task-synthesis applies the rule, how develop Phase A pre-flights, cross-references to 017 / 018 / 019 / 022 / 025 / 026 / 020.
- `specflow:task` Phase B writes `context-budget-estimate: <int_tokens>` on every task entry. Synthesis self-checks each estimate against `config.json.task.contextBudget` (default 80K). Tasks exceeding the budget are auto-flagged for split with an inline note in `tasks.md` and a chat-line prompt directing the user to `specflow:scope-change`.
- `specflow:develop` Phase A.6 pre-flights each in-scope task's `context-budget-estimate` against the actually-loaded context. Estimate-vs-actual divergence ≥20% triggers a developer prompt with three options (approve the over-run, drop optional context, route to `specflow:scope-change`). Actual load exceeding the configured budget routes to `specflow:scope-change` non-optionally.
- Phases D/E/F carry a one-line reminder citing the doctrine doc as the canonical reference for "no mid-task compaction." The compaction-is-a-defect-signal stance lives in the doctrine doc, not in the skill body (chain-don't-absorb).
- Worked example demonstrates a deliberately-oversized task being caught at synthesis, split via the auto-flag path, and the resulting two-task pair fitting cleanly within budget.

## Non-goals

- **Building a runtime token counter.** The pre-flight measures load at Phase A boundary; the doctrine does not introduce per-tool-call counting. Budget is a guardrail, not exact accounting.
- **Redesigning compaction itself.** This feature forbids mid-task compaction; it does not change how compaction works when it is permitted (e.g. between features, between tasks).
- **Changing the smart-zone threshold.** The 80K default is empirical (per Pocock's ~100K marker minus headroom). Tightening the threshold is a config change, not a doctrine change.
- **Unifying the budget formula across model tokenisations.** chars ÷ 4 is approximate; the ±20% tolerance absorbs tokenisation variance. Model-specific tokenisation is out of scope.
- **Enforcing a sprint-wide context budget cap (across tasks in the same `sprint-bucket: N`).** Buckets do not share context — each task runs in its own window — so a sprint-wide sum is not load-bearing. The sprint-plan gate may visualise per-task budgets (per 020-sprint-skill) but does not enforce a sprint-wide cap.
- **Auto-routing to `specflow:scope-change` without developer confirmation.** Phase A's three-option prompt always surfaces; the route-to-scope-change path is one option, never the default.

## Users

- **`specflow:task` synthesisers** at Phase B — they write `context-budget-estimate` per task using the schema from the doctrine doc, run the budget self-check, and surface auto-flag warnings when a task exceeds budget.
- **`specflow:develop` orchestrators** at Phase A — they pre-flight each task's budget against actual load, and on ≥20% divergence prompt the developer with three options (approve / drop / route-to-scope-change). Mid-task compaction is refused; the agent escalates instead.
- **Cross-task reviewers (022)** at Gate 3 — their "better arrangement" lens consults `context-budget-estimate` to detect oversized tasks the per-task lens missed, and may suggest merging two small tasks or splitting one large task on budget grounds alone.
- **Sprint-skill (020) developers** at the sprint-plan gate — they see per-task budgets visualised so they can spot a risky task before approving the sprint.
- **Retro readers (`specflow:complete`)** post-feature — they see compaction events logged in the per-task manifest (019) with `event_type: escalation` outcome `compacted-mid-task`, which feed the lesson back through 018-lessons-registry to tighten future estimates.

## Requirements

- **R1.** Doctrine doc `plugins/specflow/templates/admin/single-context-task.md` exists with: rationale, verbatim rule, `context-budget-estimate` schema, no-mid-task-compaction contract, how task-synthesis applies the rule, how develop Phase A pre-flights, cross-references.
  - Trace: templates/admin/single-context-task.md (new in v2.5.0).
  - Serves goal: doctrine is the single citation point.

- **R2.** The verbatim rule quotes locked-in decision #21 from `SESSION-HANDOFF.md` exactly. No paraphrase, no abbreviation.
  - Trace: templates/admin/single-context-task.md § The rule.
  - Serves goal: doctrine matches the locked-in architectural decision.

- **R3.** `specflow:task` Phase B writes `context-budget-estimate: <int_tokens>` on every task entry in `{NNN-slug}-tasks.md`. The skill body cites `templates/admin/single-context-task.md` for the estimation algorithm rather than inlining it.
  - Trace: skills/task/SKILL.md § Phase B (per-task fields).
  - Serves goal: every task carries its budget; skill stays lightweight (≤500 lines).

- **R4.** When a task's estimate exceeds `config.json.task.contextBudget` (default 80,000 tokens), synthesis writes an inline auto-flag note in `tasks.md` under the task block (`> Budget overrun: estimate {N}K vs budget {M}K — split required before develop.`) AND surfaces a chat-line prompt directing the user to `specflow:scope-change`.
  - Trace: skills/task/SKILL.md § Phase B; templates/admin/single-context-task.md § How task-synthesis applies the rule.
  - Serves goal: oversized tasks caught at synthesis, not patched mid-execution.

- **R5.** `specflow:develop` Phase A pre-flights each in-scope task's `context-budget-estimate` against the actually-loaded context at the boundary between A.5 (announce) and B (lane triage). On ≥20% divergence, the developer is prompted with three options (approve over-run, drop optional context, route to `specflow:scope-change`). Actual load exceeding the configured budget outright routes to `specflow:scope-change` non-optionally.
  - Trace: skills/develop/SKILL.md § Phase A pre-flight sub-step; templates/admin/single-context-task.md § How develop Phase A pre-flights.
  - Serves goal: estimate-vs-actual divergence is caught before Phase B.

- **R6.** Phases D/E/F of `specflow:develop` carry a single one-line reminder citing the doctrine doc as the canonical "no mid-task compaction" reference. The compaction-is-a-defect-signal stance is *not* duplicated in the skill body.
  - Trace: skills/develop/SKILL.md § Phase D entry (reminder line); templates/admin/single-context-task.md § No-mid-task-compaction contract.
  - Serves goal: chain-don't-absorb (per `feedback_skill_size_ceiling`); develop stays under the cap.

- **R7.** Compaction events that occur mid-task (despite the rule) are logged in the per-task manifest (019) with `event_type: escalation` and outcome `compacted-mid-task`. The doctrine doc names the manifest field for retro consumption.
  - Trace: templates/admin/single-context-task.md § No-mid-task-compaction contract.
  - Serves goal: defect signal is auditable; retro can identify the synthesis-time sizing error.

- **R8.** Worked example folder `examples/docs/specflow/features/029-single-context-task/` exists with PRD, interview, tasks file (demonstrating the meta `context-budget-estimate` field on every task and ONE deliberately-oversized task that auto-flags then splits into two successors that each fit budget), and a Gate 2 manifest closed `passed`.
  - Trace: this PRD's See also.
  - Serves goal: dogfood discipline; fresh agent reading doctrine + example folder produces the same artefact shape on a third project.

## Acceptance criteria

- **AC-1.** `plugins/specflow/templates/admin/single-context-task.md` exists with all sections from R1, ≤120 lines. Verifies R1.
- **AC-2.** The doctrine doc's "The rule" section quotes locked-in decision #21 verbatim from `SESSION-HANDOFF.md` (no paraphrase). Verifies R2.
- **AC-3.** Every task entry in a fresh `specflow:task` synthesis carries `context-budget-estimate: <int_tokens>`. Verifies R3.
- **AC-4.** A task whose estimate exceeds the configured budget produces an inline auto-flag note in `tasks.md` AND a chat-line prompt directing the user to `specflow:scope-change`. Verifies R4.
- **AC-5.** `specflow:develop` Phase A's pre-flight sub-step exists; the three-option developer prompt fires on ≥20% divergence; actual-exceeds-budget-outright routes to `specflow:scope-change` without the three-option prompt. Verifies R5.
- **AC-6.** Phases D / E / F of `specflow:develop` cite the doctrine doc once for "no mid-task compaction"; the compaction-is-a-defect-signal stance is documented in the doctrine doc, not the skill body. Verifies R6.
- **AC-7.** A compaction event mid-task produces a manifest entry in the 019-task-manifest with `event_type: escalation` and outcome `compacted-mid-task`. Verifies R7.
- **AC-8.** Worked example folder exists with PRD, interview, tasks file showing a meta `context-budget-estimate` field on every task and ONE deliberately-oversized task that splits into two successors, and a Gate 2 manifest closed `passed`. Verifies R8.

## Open questions

None — the four grilling rounds resolved scope (D/E/F coverage), default budget (80K with config knob), divergence handling (three-option prompt at ±20%), and escape-hatch absence (no mid-task compaction; escalation only).

## Topics not discussed

- Whether the `context-budget-estimate` formula should be machine-verifiable. Out of scope — synthesis-time estimation is judgement-based; over-engineering the formula multiplies surface for marginal precision gain. The ±20% pre-flight tolerance is the calibration knob.
- Model-specific tokenisation accuracy. Out of scope — chars ÷ 4 is approximate; ±20% tolerance absorbs tokenisation variance.
- Sprint-wide context budget caps. Considered, deferred — buckets do not share context. May surface as a 020-sprint-skill visualisation concern, not a hard cap here.

## See also

- `plugins/specflow/templates/admin/single-context-task.md` — the doctrine
- `plugins/specflow/skills/task/SKILL.md` § Phase B — `context-budget-estimate` field
- `plugins/specflow/skills/develop/SKILL.md` § Phase A pre-flight — estimate-vs-actual check
- `v2/docs/SESSION-HANDOFF.md` § Architectural decisions — locked in #21 — the verbatim rule
- `v2/docs/knowledge/medin-wisc-framework.md` — context rot inspiration
- `v2/docs/knowledge/pocock-ai-coding-real-engineers.md` — smart-zone cliff inspiration
- 017 — tdd-discipline (Red/Green/Refactor unit covered by the single window)
- 018 — lessons-registry (compaction events feed lessons forward)
- 019 — task-manifest (compaction events log as escalation entries)
- 022 — cross-task review (consults budget for "better arrangement" lens)
- 025 — sprint-task-flagging (bucket sizing informed by per-task budgets)
- 020 — sprint-skill (sprint-plan gate visualises budgets)
- 026 — agent-teams-per-stage (Build team operates inside the single-window contract)
