# PRD interview — features/029-single-context-task

## Goal confirmation

The user invoked `specflow:prd 029-single-context-task` with the brief: "Codify the rule that every task's implementation runs in a single agent context window, no mid-task compaction. Tasks that don't fit get split at synthesis time, not patched mid-execution. Make compaction during develop a defect signal."

Confirmed: the goal is to install a single doctrine doc + add a `context-budget-estimate` field to task synthesis + add a Phase A pre-flight to `specflow:develop` that catches estimate-vs-actual divergence before code execution begins. Out-of-scope-at-goal-level: building runtime token counters; redesigning compaction itself; changing the smart-zone threshold (kept at the empirical ~80K within 100K cliff).

## Original request

> "For the implementation of a task, it needs to be done all within one context window so it has all of the information that it needs to complete the task." — chat feedback citing the Cole Medin WISC framework video. Elevated to locked-in architectural decision #21 in `SESSION-HANDOFF.md`; no re-litigation.

## Codebase context (pre-grilling)

- `plugins/specflow/skills/develop/SKILL.md` Phases D/E/F currently have no token-budget pre-flight; Phase A (lines 67-113) reads PRD/tasks/Gate-3 manifest but does not measure load.
- `plugins/specflow/skills/task/SKILL.md` Phase B (lines 125-227) writes `tasks.md` task entries with `Anchor / Scope / Acceptance / Depends on / Notes` fields — no budget field; no split-at-synthesis path beyond the existing sizing heuristics ("touches >5 files = split").
- `plugins/specflow/templates/admin/team-review-bridge.md` is the doctrine-doc shape this feature mirrors (the v2.4 reference).
- `knowledge/medin-wisc-framework.md` § Select calls out "just in time, not just in case" — the load-discipline that prevents bloat in the first place.
- `knowledge/pocock-ai-coding-real-engineers.md` § LLM Constraint 1 calls out the smart-zone / dumb-zone cliff at ~100K tokens regardless of context-window size — the empirical anchor for the 80K default budget.
- The existing `017-tdd-discipline` (Sprint 2) defines Red → Green → Refactor as the implementation unit; this feature's "single window" is the unit that covers all three sub-phases.

## Round 1 — what does "single context window" cover

**Question.** Does the rule cover only Phase D (implementation) or all of D/E/F (including Gate 5 review and Verifier)?

**Answer.** All of D/E/F for the *same* task. Gate 5 reviewers run in fresh contexts per 027-reviewer-context-isolation, so they're not the writer's window — but the writer's window must still hold from D entry through F exit for the task. If the writer compacts between D and E to "make room for the gate", that's still mid-task compaction and still a defect signal. The rule binds the writer's continuity, not the reviewers' isolation.

## Round 2 — what's the budget number, and is it configurable

**Question.** Pocock's empirical cliff is ~100K. Should the budget default to 100K, or lower for headroom?

**Answer.** Default 80K, configurable via `config.json.task.contextBudget`. 80K leaves ~20K headroom inside the smart zone for tool-call output, scratch reasoning, and the inevitable underestimate. Projects with disciplined just-in-time loading and tight tasks can raise the default; projects burning tokens in research-heavy tasks should not. The number is empirical, not derived — the doctrine doc spells out the components but does not pretend the formula is exact.

## Round 3 — what happens when the estimate is wrong

**Question.** If a task's estimate at synthesis says 60K but the actual load at Phase A is 95K, what does develop do?

**Answer.** Pause and prompt the developer with three options: (a) approve the over-run (logged to `decision-log.md`), (b) drop optional context — the developer names which payload component to drop, typically lessons or codebase-context files, (c) route to `specflow:scope-change` to split the task. The ≥20% divergence threshold is what triggers the prompt; ±20% is treated as estimation noise and proceeds silently. If the actual load exceeds the configured budget outright (regardless of estimate), the route-to-scope-change path is non-optional.

## Round 4 — does compaction during D/E/F have any escape hatch

**Question.** Is there *any* case where mid-task compaction is acceptable?

**Answer.** No. Compaction is the failure mode this feature exists to remove. The agent's response when context approaches the cliff is to escalate to the developer, who then picks the same three options as the Phase A pre-flight (approve the over-run, drop optional context, or route to scope-change). The escalation is logged in the per-task manifest (019) with `event_type: escalation` and outcome `compacted-mid-task` — so the retro can identify which synthesis estimate was wrong and feed the lesson back via 018.

## Sign-off

User confirmed: doctrine doc + `context-budget-estimate` field on tasks + Phase A pre-flight + no-mid-task-compaction rule (with escalation, not silent compaction). Proceed to PRD synthesis.

## Topics not discussed

- Whether the `context-budget-estimate` formula should be machine-verifiable (e.g. a budget-checker skill). Out of scope — synthesis-time estimation is judgement-based; over-engineering the formula multiplies surface for marginal precision gain.
- Whether to count tokens by model-specific tokenisation (chars ÷ 4 is approximate). Out of scope — the budget is a guardrail, not an exact accounting; ±20% tolerance absorbs tokenisation variance.
- Whether sprint-bucket sizing (025) should sum per-task budgets and enforce a sprint-wide cap. Considered, deferred — buckets do not share context, so a sprint-wide sum is not load-bearing for this rule. May surface as a 020-sprint-skill UX concern (visualisation), not a hard cap here.
