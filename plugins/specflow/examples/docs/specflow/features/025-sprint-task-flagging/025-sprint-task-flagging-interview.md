# PRD interview — features/025-sprint-task-flagging

## Goal confirmation

The user invoked `specflow:prd 025-sprint-task-flagging` with the brief: "When `specflow:task` synthesises tasks, flag which can be batched into a sprint together via a `sprint-bucket: N` field on each task. Document the bucket-assignment heuristic so the field is reproducible, not vibes."

Confirmed: the goal is to extend the per-task synthesis schema with a single integer field (`sprint-bucket: N`) plus a documented heuristic in `task/SKILL.md` capturing dependency edges, parallelism opportunity, and respect for 029's `context-budget-estimate`. Out-of-scope-at-goal-level: building the sprint-skill itself (020), enforcing a sprint-wide context budget cap (per 029 § Non-goals), or auto-promoting bucket numbers into Linear cycle assignments (deferred to 020).

## Original request

> features.md item 2 — "When tasks are generated, the AI must flag which tasks may be done within the same Sprint." Lines up with chat feedback that 020-sprint-skill needs an input format it can read; the upstream `specflow:task` synthesis is where the flag belongs because that's where the dependency graph and per-task budgets already exist.

## Codebase context (pre-grilling)

- `plugins/specflow/skills/task/SKILL.md` § Phase B writes per-task entries with `Anchor / Scope / Acceptance / Depends on / context-budget-estimate / Notes`. 029 just shipped `context-budget-estimate`; this feature appends a sibling field.
- `v2/docs/FEATURES.md` § 020 (sprint-skill) names `sprint-bucket: N` as the input contract from 025 — the field consumer is already designed.
- `v2/docs/FEATURES.md` § 029 § Implications #3 explicitly anticipates this feature: *"Tasks in the same bucket don't share context (each runs in its own window), but the bucket sizing is informed by per-task budgets so the developer doesn't approve a sprint where any single task can't fit."*
- `knowledge/medin-parallel-agentic-playbook.md` § Pillar 1 (issue-as-spec) — issues are pre-scoped so parallel agents have natural boundaries; buckets group what's safe to fan out.
- `knowledge/medin-archon-livestream.md` § Hybrid Secret — deterministic nodes pass artefacts between coding-agent nodes; the `sprint-bucket: N` field IS the deterministic artefact passed from `specflow:task` to `specflow:sprint`.
- `task/SKILL.md` is currently 486 lines after 029 shipped. The 500-line ceiling (per `feedback_skill_size_ceiling`) leaves ~14 lines headroom — additions must be tight.

## Round 1 — what does the heuristic actually decide

**Question.** A bucket number is just an int. What inputs determine which int a given task gets, and what makes the heuristic reproducible across two synthesis runs?

**Answer.** Three inputs in priority order: (1) **dependency edges** — tasks in bucket N+1 must not depend on bucket N+2; only on bucket N or N+1. This forces the topological floor: tasks land in the lowest bucket whose predecessors all sit in lower or same-numbered buckets. (2) **Parallelism opportunity** — tasks with no shared `Depends on:` edges and disjoint `Scope:` surfaces share a bucket so the sprint-skill can fan them out. (3) **Per-task budget respect** — the bucket count is not capped (a feature can have 4 buckets if its dependency graph is deep), but no single bucket may contain a task whose `context-budget-estimate` exceeds the configured budget — that case is already auto-flagged for split at synthesis (per 029 R4) and never reaches bucket assignment.

## Round 2 — does sprint-bucket compete with `Depends on:` for the same information

**Question.** `Depends on:` already encodes the edge; isn't `sprint-bucket: N` redundant?

**Answer.** No — they encode different views. `Depends on:` is an edge list (which task before which); `sprint-bucket: N` is a topological grouping derived from the edges, with the parallelism-opportunity overlay. The sprint-skill (020) reads `sprint-bucket: N` to plan a fan-out batch; it would have to recompute the topology from `Depends on:` otherwise. Caching the grouping at synthesis time keeps 020 lightweight and gives the developer a stable artefact to review at the sprint-plan gate. The heuristic explicitly states: `sprint-bucket` is computed from `Depends on:` + scope-overlap, never asserted independently.

## Round 3 — what about cross-feature buckets

**Question.** Should two features whose tasks could run in parallel share bucket numbers (e.g. feature A's bucket 1 and feature B's bucket 1 are the same sprint)?

**Answer.** No — bucket numbers are scoped to the feature. Cross-feature sprint planning is parked under the existing parked-memory entry (per FEATURES.md § Parked) and out of scope for both 025 and 020. The sprint-skill (020) maps one Linear project to one specflow feature; cross-feature batching happens at a higher orchestration layer if/when promoted. Within-feature: bucket 1 always means "no predecessors"; bucket 2 means "depends only on bucket 1"; etc.

## Sign-off

User confirmed: single integer field per task; heuristic documented in `task/SKILL.md` Phase D verify-steps; respect for 029's per-task budget; feature-scoped numbering. Proceed to PRD synthesis.

## Topics not discussed

- Whether the heuristic should produce a visualisation (DAG render). Out of scope — visualisation belongs to the sprint-plan gate (020), not the synthesis-time field.
- Whether `sprint-bucket: 0` is reserved (e.g. for "always run first, e.g. setup tasks"). Out of scope — bucket 1 is the floor; introducing a 0 layer multiplies surface for marginal value.
- Whether the heuristic should auto-detect the optimal number of buckets vs accepting whatever the topology produces. Out of scope — synthesis writes the topological grouping; tuning the bucket count is a 022-cross-task-review concern (the "better arrangement" lens).
