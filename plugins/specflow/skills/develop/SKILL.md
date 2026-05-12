---
name: specflow:develop
description: User-facing entry point for lane-based implementation. Six-phase orchestrator — A pre-flight + read PRD/tasks/Gate 3, B lane triage with mechanical confidentiality classification, B.1 mechanical pre-Gate-4 lane recheck, C Gate 4 multi-agent debate manifest (plan vs tasks/PRD), D lane execution (green-batch / yellow-HITL / red-human-led), E Gate 5 multi-agent debate manifest (code vs plan, Codex when available), F Verifier confirmation + PR + Linear + task-history. Resumes intelligently if invoked on an in-flight feature.
status: v2-new
phase: 2
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-interview.md
  - docs/specflow/features/{NNN-slug}/debate-log/tasks-gate3/manifest.md
  - docs/specflow/admin/agents/standard/lifecycle/
  - docs/specflow/admin/agents/standard/principles/
  - docs/specflow/admin/agents/specialised/
  - docs/specflow/admin/config.json
  - docs/specflow/admin/rules/non-negotiable.md
  - docs/specflow/admin/rules/guidelines.md
  - docs/specflow/admin/environment.json
  - docs/specflow/admin/task-history.json
produces:
  - docs/specflow/features/{NNN-slug}/debate-log/develop-gate4/manifest.md
  - docs/specflow/features/{NNN-slug}/debate-log/develop-gate4/findings/
  - docs/specflow/features/{NNN-slug}/debate-log/develop-gate5/manifest.md
  - docs/specflow/features/{NNN-slug}/debate-log/develop-gate5/findings/
  - docs/specflow/admin/scratch/{NNN-slug}-develop/
  - docs/specflow/admin/task-history.json
eval: define T_run = sprint-mode ? tasks_in_scope (from the sprint result payload) : every task in the feature's tasks file; lane-assignments.json exists with one entry per task in T_run; for every task in T_run, lane-recheck-{task-id}.json exists in the scratch directory before Gate 4 opens; Gate 4 manifest closes with passed | passed-with-revisions | passed-with-escalations; Gate 5 manifest closes with passed | passed-with-revisions | passed-with-escalations; every task in T_run has a corresponding entry in admin/task-history.json with the six development-time fields populated (lane_assigned, ai_assistance_level, elapsed_minutes, what_worked, what_didnt_work, blast_radius_outcome); tasks NOT in T_run (the out-of-sprint complement) have NO lane-assignment entry, NO recheck file, NO manifest stub, and NO new task-history entry from this run — they remain untouched for the next sprint; for every Yellow-lane task in T_run and every Green-lane task in T_run with config.develop.tddRequired === true, the per-task manifest stub at admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md contains a line matching the regex ^red:\s+(passed|failed|skipped \((config|trivial|human-led Red lane)\))\s+\([0-9T:Z-]+\)\s+—\s+.+$ and equivalent regexes for green: and refactor: (017-tdd-discipline v2.5.0).
---

# specflow:develop

You are the user-facing entry point for lane-based implementation. You own the full flow from "Gate 3 closed" to "PR opened, Verifier passed, task-history updated."

This is a **6-phase orchestrator** (A → B → B.1 → C → D → E → F) that turns a closed-Gate-3 task list into shipped code. Every gate manifest, every reviewer dispatch, every plan emission, every agent execution, and every Verifier run is a forked sub-agent per the orchestrator pattern (`templates/orchestrator-pattern.md`). Your parent context never accumulates the sub-skills' raw work — target ≤30K tokens for a 10-task feature end-to-end, ≤2K growth per gate.

The four core principles bind here as everywhere: think before coding (assumptions in the artefact, not silent), simplicity first (the plan is the minimum surface that satisfies the task acceptance), surgical changes (a single task touches the smallest blast radius), goal-driven execution (every step has a binary verify check inline).

---

## Inputs

The user invokes you with one of:
- `specflow:develop {NNN-slug}` — feature-level batch run (Green batch first, Yellow sequential second, Red surfaced third).
- `specflow:develop {NNN-slug} --task T{N}` — runs exactly one task regardless of lane.
- `/specflow:develop` with no argument — ask the user which feature.
- Auto-invocation from Linear UI (Backlog/Todo → In Progress trigger) — lands at Phase A of the same flow.

**Resume logic.** Before starting Phase A, detect the situation:

1. Locate `docs/specflow/features/{NNN-slug}/`. If missing, refuse: *"Feature `{NNN-slug}` does not exist. Run `specflow:prd` and `specflow:task` first."*
2. Verify Gate 3 closed:
   - `features/{NNN-slug}/{NNN-slug}-tasks.md` exists.
   - `features/{NNN-slug}/debate-log/tasks-gate3/manifest.md` exists with a `**passed**`, `**passed-with-revisions**`, or `**passed-with-escalations**` closing decision.
   - If not closed (or status `failed`), refuse with the literal sentinel from T1's acceptance: *"Gate 3 not closed for `{NNN-slug}` (status: `{status|missing}`). Resolve Gate 3 first via `specflow:task {NNN-slug}`."*
3. **Bind `T_run` BEFORE evaluating resume artefacts** (per A.6.5 doctrine — resume predicates must be scoped to the run set, not the full tasks file):
   - **Fresh run** (no `admin/scratch/{NNN-slug}-develop/` directory) → defer T_run binding to A.6.5 (sprint-mode runs Phase A.5.5 first; feature-mode binds the full tasks file).
   - **Resuming** (scratch directory exists):
     - If `admin/scratch/{NNN-slug}-develop/t-run.json` exists → load `task_ids` and use that as `T_run` for every check below.
     - If `t-run.json` is missing → halt and ask the user: *"Resume detected for `{NNN-slug}` but `t-run.json` is missing. Choose: (1) re-bind as feature-mode (full tasks file), (2) re-bind as per-task `--task T{N}`, (3) discard scratch and start fresh."* Do NOT auto-default to feature-mode — the previous run may have been a partial sprint and silently widening the scope would invent artefact-existence requirements for tasks the user never asked to process.
4. Determine the resume point — every "task" predicate below scopes to `T_run`:
   - **No `admin/scratch/{NNN-slug}-develop/` directory** → start Phase A (fresh run); T_run binds at A.6.5.
   - **Scratch directory exists, no `lane-assignments.json`** → resume Phase B (lane triage) for `T_run`.
   - **`lane-assignments.json` exists, no `lane-recheck-{T*}.json` for some tasks in `T_run`** → resume Phase B.1 (mechanical recheck) for the missing-recheck subset of `T_run`.
   - **All `T_run` recheck files exist, no Gate 4 manifest for some tasks in `T_run`** → resume Phase C (Gate 4) for the missing-gate subset of `T_run`.
   - **Gate 4 closed for some `T_run` tasks, no code change set OR no Gate 5 manifest** → resume Phase D + E for those tasks (still inside `T_run`).
   - **All `T_run` gates closed, some Verifier rejections without user election** → resume Phase F at the rejection.
   - **All `T_run` tasks complete** → ask the user: *"Run for `{NNN-slug}` appears complete for the in-scope task set ({task_ids}). Tasks outside the run set were intentionally untouched. What do you want to do? (1) re-run a specific task with `--task T{N}`, (2) inspect the manifests, (3) start a new sprint via `specflow:sprint {NNN-slug}` for the next batch, (4) `specflow:scope-change` if the PRD itself needs revision."*

Tell the user explicitly which phase you're starting at AND which task set is in scope (cite `T_run` task IDs).

---

## Phase A — Pre-flight + read PRD/tasks/Gate 3

### A.0 Check the skill toggle

Read `admin/config.json`. If `config.skills.develop.enabled === false`, refuse with the canonical message and return:

> *"`specflow:develop` is disabled in this project (`config.skills.develop.enabled = false`). Re-enable in `docs/specflow/admin/config.json` or invoke a different skill."*

Treat a missing `config.skills.develop` field as `enabled: true` (backward-compat for v2.3 projects upgraded via `specflow:upgrade`). Resolver contract documented in `templates/admin/skill-toggles.md`. Citation: 012-config-skill-toggles v2.4.0.

### A.1 Verify the artefact chain

Use Read tool in parallel on:
- `features/{NNN-slug}/{NNN-slug}-tasks.md`
- `features/{NNN-slug}/{NNN-slug}-prd.md`
- `features/{NNN-slug}/{NNN-slug}-interview.md`
- `features/{NNN-slug}/debate-log/tasks-gate3/manifest.md`
- `admin/config.json`
- `admin/rules/non-negotiable.md`
- `admin/rules/guidelines.md`
- `admin/environment.json`
- `admin/task-history.json` (empty array `{"tasks": []}` is fine)

### A.2 Detect plugins, MCPs, and CLIs

Inspect `admin/environment.json` for the soft-dependency surface:
- **`plugins.agent-teams`** — present → Phase B can dispatch Green/Yellow batches via `team-spawn`. Absent → Phase D degrades to sequential single-specialist invocation; warn the user once at this point: *"agent-teams plugin not detected — implementation will run via sequential single-specialist invocation. Throughput is reduced; functionality is preserved."*
- **`mcp.linear.available`** — true → Phase A.4 fires "Backlog/Todo → In Progress" transition; Phase F fires "In Progress → In Review" on PR open. False → both fall back to chat-only status lines.
- **`cli.codex.available`** — true → Phase E Gate 5 invokes Codex as a sixth reviewer. False → Phase E manifest writes the literal sentinel *"Codex not detected — same-provider review only. Cross-provider findings may be missed; install Codex CLI for full Gate 5 coverage."* and runs Gate 5 with the standard five reviewers. **Lens-overlap note:** Codex's correctness-and-cross-skill lens overlaps with Goal-Driven Reviewer's reverse-traceability lens at Gate 5; treat Codex as the primary cross-provider check on schema validation, contract semantics, and cross-skill state assumptions, while Goal-Driven retains forward + reverse coverage of R-IDs to ACs to code surfaces. Findings that both fire are not duplicates — they're independent confirmations.

Record the detected surface to `admin/scratch/{NNN-slug}-develop/environment-snapshot.json` (orchestrator-pattern: scratch directory per orchestration). Phase B reads this to make the team-spawn decision; Phase E reads this to write the Codex section.

### A.3 Surface Gate 3 closing decisions

Read the Gate 3 manifest's "Closing decision" section. If status is `passed-with-revisions`, read the revisions list — every revision is a load-bearing constraint that the plan emitter (Phase D) must respect. Note the revisions in your working memory; surface them to the user only if a Gate 4 reviewer flags drift.

### A.4 Linear status: Backlog/Todo → In Progress

If `mcp.linear.available: true` AND the invocation is per-task (`--task T{N}`) AND the task's current Linear status is Backlog or Todo, fire the transition before continuing. If running per-feature (no `--task`), fire the transition for each task as Phase D enters that task's loop, not all at once.

If MCP unavailable, print the chat-only line: `[linear status: T{N} → In Progress (skipped — MCP not available)]` and continue.

### A.5 Tell the user what you're doing

*"Read the PRD, tasks file, and Gate 3 manifest. Environment: agent-teams `{present|absent}`; Linear MCP `{available|absent}`; Codex CLI `{available|absent}`. Lane-triaging {N} tasks now."*

### A.5.5 Sprint plan via specflow:sprint (per 020-sprint-skill v2.7.0)

Before lane triage, fork `specflow:sprint {NNN-slug}` as a sub-skill. Sprint pulls the mapped Linear project (when MCP available), reconciles drift, filters to the in-scope batch via `sprint-bucket: N` and `config.json.develop.maxIssuesPerSprint` (default 5), synthesises a sprint plan with per-stage team assignments per `templates/admin/stage-teams.md` (per 026-agent-teams-per-stage), presents the sprint-plan gate to the developer, and on approval creates a git work-tree at `admin/scratch/{NNN-slug}-sprint/worktree/`.

Sprint returns the structured result:

```json
{
  "feature": "{NNN-slug}",
  "manifest_path": "features/{NNN-slug}/debate-log/sprint-plan/manifest.md",
  "tasks_in_scope": ["T-1", "T-2", "T-3"],
  "worktree_path": "admin/scratch/{NNN-slug}-sprint/worktree",
  "team_assignments": { "T-1": {...}, "T-2": {...}, "T-3": {...} }
}
```

The in-scope tasks become the set Phase B (lane triage) operates over. Tasks outside the sprint stay in `tasks.md` for the next session. The team assignments inform Phase B.4's agent-composition decision per task — Phase B.4 reads the resolved team-assignment block instead of computing the team from scratch (per 026).

If the developer defers at the sprint-plan gate, develop exits without entering Phase B; nothing is written.

Skill body for sprint: `skills/sprint/SKILL.md`.

### A.6 Context-budget pre-flight (per 029-single-context-task)

For each in-scope task, read `context-budget-estimate` from the tasks file and measure the actually-loaded context size for the task's payload (PRD slice + task spec + matched lessons + per-task manifest scaffold + codebase-context files + test plan).

**Task block format detection (per 033-human-readable-tasks v2.12.0).** Two formats may exist:

- **v2.12.0+ format** — task blocks have a `**Parent PRD:**` line and an HTML-comment footer `<!-- ai-metadata: context-budget-estimate=N sprint-bucket=N prior-lessons=[...] -->`. Read `context-budget-estimate` from the HTML comment. The per-task slice loaded for execution is the `Technical Implementation` paragraph + `Technical References` list + `Files to Modify` / `Files to Create` lists + `Acceptance Criteria` bullets. `Current State` / `Expected State` / `QA Verification` / `Definition of Done` / `User Stories Addressed` are NOT loaded at execution time — they are human-reading sections.
- **Pre-2.12.0 format** — task blocks have a `**Anchor:**` line and visible `**context-budget-estimate:**` / `**sprint-bucket:**` / `**prior-lessons:**` fields. Read `context-budget-estimate` from the visible field. Per-task slice is the `Scope:` + `Acceptance:` + `Notes:` fields. Backward-compatible.

Detection: presence of `**Parent PRD:**` in the task block → new format; absence → legacy format. Mixed-format files (some tasks new, some old) are tolerated — detect per-task block.

Compare:

- **Within ±20%** → proceed silently to Phase B.
- **Actual exceeds estimate by ≥20% but stays within `config.json.task.contextBudget`** → pause and surface the three-option developer prompt: *"T{N} actual context {A}K vs estimate {E}K (≥20% divergence). Choose: (1) approve the over-run (logged to `decision-log.md`), (2) drop optional context (name the payload component to drop, typically lessons or codebase-context files), (3) route to `specflow:scope-change` to split."* No auto-default; empty input re-prompts.
- **Actual exceeds the configured budget outright** → refuse to enter Phase B for that task; route to `specflow:scope-change` non-optionally.

The estimation algorithm and the no-mid-task-compaction rationale live in `templates/admin/single-context-task.md` — cite, do not inline.

### A.6.5 Bind `T_run` (the run-scope set)

Define the set of tasks this invocation processes — every subsequent per-task loop, gate manifest, and verification check binds to `T_run`, not to the full tasks file.

- **Sprint-mode** (Phase A.5.5 returned a sprint result): `T_run = result.tasks_in_scope`.
- **Feature-mode** (no sprint result; legacy or `--feature-complete` invocation): `T_run = every task ID in {NNN-slug}-tasks.md`.
- **Per-task mode** (`--task T{N}`): `T_run = ["T{N}"]`.

Persist `T_run` to `admin/scratch/{NNN-slug}-develop/t-run.json`:

```json
{
  "feature": "{NNN-slug}",
  "mode": "sprint | feature | per-task",
  "task_ids": ["T-1", "T-2", "T-3"],
  "bound_at": "{YYYY-MM-DD HH:MM}"
}
```

**Out-of-scope guard.** Tasks in the tasks file but NOT in `T_run` (the *out-of-sprint complement*) MUST receive no lane assignment, no recheck, no manifest stub, no Gate 4/5 manifest, and no new task-history entry from this run. They remain untouched for the next sprint. Verify this absence at the final-verify checklist.

Hand off to Phase B.

---

## Phase B — Lane triage

### B.1 Compute the four-axis tuple per task

For each task in `T_run` (per A.6.5 — the set persisted to `admin/scratch/{NNN-slug}-develop/t-run.json`):

1. **Verifiability** — AI-judged from the task's `Acceptance` field. Enum: `high` (binary check, no judgement), `medium` (binary with one ambiguous edge), `low` (judgement-dependent or non-binary). Cite the exact acceptance line.
2. **Blast radius** — AI-judged from the task's `Scope` field. Enum: `low` (≤1 file, single module), `medium` (2-3 files, single module), `high` (>3 files OR cross-module). Cite the scope line.
3. **Dependency state** — mechanical. Read the task's `Depends on:` field; for each upstream task, check `admin/task-history.json` for `status: shipped`. Enum: `satisfied` (all upstream shipped, OR upstream is in the same Green batch), `blocked` (any upstream non-shipped and not in batch).
4. **Confidentiality** — rule-based via `admin/config.json.confidentialPaths` glob match against the task's `Scope` field's listed file paths. Enum: `non-confidential` (no path matches), `confidential` (any path matches). **Never AI-rated.**

### B.2 Apply monotonic-downgrade precedence

Lane assignment per the precedence rule (any axis weak → at least Yellow; any axis Red-qualifying → Red):

| Axes summary | Lane |
|---|---|
| All four Green-qualifying (verifiability=high, blast=low, deps=satisfied, conf=non-confidential) | green |
| At least one weak axis (medium verifiability/blast OR blocked deps), no Red-qualifying axis | yellow |
| Any Red-qualifying axis (verifiability=low OR blast=high OR confidentiality=confidential) | red |

Confidentiality=confidential ALWAYS forces red, regardless of other axes. A user request to downgrade a confidential-path task's lane is refused with the literal sentinel: *"Confidential-path lane is non-overridable. Resolve the path classification or run the task in Red."*

### B.3 Write `lane-assignments.json`

Use Write tool on `admin/scratch/{NNN-slug}-develop/lane-assignments.json`:

```json
{
  "feature": "{NNN-slug}",
  "computed_at": "{YYYY-MM-DD HH:MM}",
  "tasks": [
    {
      "id": "T{N}",
      "lane": "green | yellow | red",
      "axes": {
        "verifiability": {"value": "high|medium|low", "cited": "tasks.md:T{N}.Acceptance line {n}"},
        "blast_radius": {"value": "low|medium|high", "cited": "tasks.md:T{N}.Scope line {n}"},
        "dependency_state": {"value": "satisfied|blocked", "cited": "Depends on: {list}; task-history.json status check"},
        "confidentiality": {"value": "non-confidential|confidential", "cited": "config.json.confidentialPaths glob: {match|no-match}"}
      }
    },
    ...
  ]
}
```

### B.4 Agent-teams plugin consultation (per task)

For each task, decide the agent composition:

- **Red lane:** never spawn a team. Pick a single specialised agent from `admin/agents/specialised/` by stack-match (file-glob) against the task's scope. Log to the per-task plan: `single-specialist invoked: agent={agent-name}, stack-match={glob}`.
- **Green or Yellow lane:**
  - If `plugins.agent-teams` is present AND the task's scope matches a `config.json.teams[].scope` glob → consult the plugin: `team-spawn --preset {name}`. On success: log `team-spawn invoked: preset={name}, members={list}`. On failure (preset unmatched, version-skew, internal error): log `team-spawn failed: {error}; fell back to single-specialist invoked: agent={agent-name}` AND surface the chat-line warning *"[agent-teams: team-spawn failed for preset {name} — falling back to single-specialist {agent}]"*. The fallback never aborts the task.
  - Otherwise: pick a single specialised agent by stack-match. Log `single-specialist invoked: agent={agent-name}, stack-match={glob}`.

The agent decision is recorded per task in `lane-assignments.json` under a `composition:` field.

### B.5 Tell the user the lane distribution

*"Lane distribution: {G} Green / {Y} Yellow / {R} Red. Green batch will run first ({N} tasks, batch cap {cap}); Yellow tasks run sequentially with HITL pairing; Red tasks surface to you for human-led handling."*

If the distribution looks pathological (e.g. 30/40/30 instead of the target 60/30/10), append a note: *"Distribution skews toward Yellow/Red — consider whether the PRD needs re-cutting via `specflow:scope-change`. Logging distribution to `task-history.json` as a Phase 3 learning signal."*

Hand off to Phase B.1.

---

## Phase B.1 — Mechanical pre-Gate-4 lane recheck

This phase fires unconditionally for every task in `T_run` BEFORE Gate 4 opens. It is the load-bearing addition from the dogfood Gate 2 (block tbc-r1-f1) — without it, the lane-aggressive-flag mechanism would not catch the cases where the plan's emitted content reveals more than the task scope claimed.

### B.1.1 Emit the per-task plan (preliminary)

For each task, fork a plan-emitter sub-agent. Pass it:
- The task's tasks-file entry (via command substitution).
- The PRD's matching `R{N}` requirement (via command substitution).
- The lane assignment from `lane-assignments.json`.

The plan emitter writes the preliminary plan to `admin/scratch/{NNN-slug}-develop/plans/{task-id}-plan.md`. The plan's first paragraph MUST follow the R17 PRD-anchor format: *"We're doing X because of PRD requirement R{N}. This aligns with Z."* — X paraphrases the task scope, R{N} is a literal requirement reference from the feature's PRD, Z paraphrases the requirement's `Serves goal:` field. Plans whose first paragraph does not match are a failed plan emission; refuse to proceed to B.1.2 until the format is satisfied.

### B.1.2 Run the mechanical recheck

For each task, fork a recheck sub-agent. Pass it:
- The preliminary plan (via command substitution).
- The task's `Scope` field claim of file count + listed modules (via command substitution).
- The `admin/config.json.confidentialPaths` array (via command substitution).

The recheck sub-agent writes `admin/scratch/{NNN-slug}-develop/lane-recheck-{task-id}.json`:

```json
{
  "task_id": "T{N}",
  "ran_at": "{YYYY-MM-DD HH:MM}",
  "file_count_plan_vs_scope": {
    "plan_files": N,
    "scope_claimed": M,
    "ratio": "N/M",
    "downgraded": false
  },
  "modules_plan_vs_scope": {
    "plan_modules": [...],
    "scope_modules": [...],
    "new_modules": [],
    "downgraded": false
  },
  "confidential_path_match": {
    "plan_paths_matching": [],
    "forced_red": false
  },
  "lane_before": "green | yellow | red",
  "lane_after": "green | yellow | red"
}
```

### B.1.3 Apply downgrades

For each recheck file:
- If `file_count_plan_vs_scope.ratio > 1.5` → downgrade blast radius one step (low→medium→high) and re-resolve the lane via the precedence table (B.2). Set `file_count_plan_vs_scope.downgraded: true`.
- If `modules_plan_vs_scope.new_modules` is non-empty → downgrade blast radius one step. Set `modules_plan_vs_scope.downgraded: true`.
- If `confidential_path_match.plan_paths_matching` is non-empty AND those paths weren't in the original scope → force lane to `red`. Set `confidential_path_match.forced_red: true`.

If any downgrade fired, re-emit the per-task plan with the corrected lane (re-run B.1.1 for that task only) BEFORE Gate 4 opens. Update `lane-assignments.json` with the corrected lane.

### B.1.4 Verify

Before handing off to B.1.5, verify:
- Every task has a `lane-recheck-{task-id}.json` file (one per task; missing any is a failed run).
- Every plan whose lane was downgraded has been re-emitted.
- `lane-assignments.json` reflects the final post-recheck lane.

### B.1.5 Aggregate-outcome record

Write `admin/scratch/{NNN-slug}-develop/lane-assignments.json`'s `b1_recheck` field as a structured aggregate (not free-form prose). Reviewers and downstream phases consume this surface:

```json
"b1_recheck": {
  "ran_at": "{YYYY-MM-DD HH:MM}",
  "tasks_checked": N,
  "lane_changes": [
    { "task_id": "T{N}", "before": "green", "after": "yellow", "reason": "file_count_ratio | new_modules | confidential_path" }
  ],
  "batch_shape_at_default_cap": {
    "greenBatchCap": N,
    "green_task_count": N,
    "batches": [["T1","T2","T3"], ["T4","T5","T6"], ["T7"]],
    "boundary_warnings": []
  },
  "summary": "no upgrades triggered | N upgrades triggered (see lane_changes)"
}
```

The `batch_shape_at_default_cap` field surfaces the green-batch shape as it would be cut at the project's `config.json.develop.greenBatchCap`. `boundary_warnings` flags batches that split a `Depends on:` chain across batch edges (the dependency is satisfied but the reviewer sees only one half — a calibration risk worth surfacing). Empty array when no boundary issues.

Gate 4 reviewers read this aggregate (per C.2 input contract) to evaluate whether the batch cadence is healthy without re-deriving it.

---

## Phase C — Gate 4 multi-agent debate manifest (plan vs tasks/PRD)

Gate 4 reviews the per-task plan against the task's tasks-file entry, the PRD anchor, and the lane assignment. It uses the same five-reviewer pattern as Gates 2 + 3.

### C.1 Set up the debate folder

```bash
mkdir -p docs/specflow/features/{NNN-slug}/debate-log/develop-gate4/findings/{round-1,round-2,round-3}
mkdir -p docs/specflow/features/{NNN-slug}/debate-log/develop-gate4/raw
```

The Gate 4 manifest path is per-feature, not per-task — multiple Gate 4 manifests for a feature are stored sequentially with task-id suffix when needed (e.g. on R5 re-lane forcing a fresh manifest, the original is renamed to `manifest-T{N}-pre-relane.md`).

### C.2 Identify reviewers

The Gate 4 reviewer set is exactly:
- `lifecycle/devils-advocate.md`
- `principles/simplicity-reviewer.md`
- `principles/surgical-reviewer.md`
- `principles/think-before-coding-reviewer.md` — **load-bearing at Gate 4** (plan-level unstated assumptions surface here).
- `principles/goal-driven-reviewer.md` — **load-bearing at Gate 4** (cross-checks PRD anchor against cited R-ID; flags orphan ACs).
- `principles/edge-case-reviewer.md` — **load-bearing at Gate 4 for blindspot coverage** (per 028-edge-case-reviewer v2.6.0). Deliberately NOT goal-aware; runs in fresh context per 027; findings carry `recommendation` + `reasoning` and are advisory (orchestrator decides accept / reject / defer-to-misc).

Codex does NOT join Gate 4 in v1 — there is no `config.json.develop.codexAtGate4` knob. Setting the field has no observable effect.

### C.3 Round 1 — parallel finding fire

For each reviewer, dispatch a forked sub-agent. Pass each:
- The per-task plan (via command substitution).
- The task's tasks-file entry (via command substitution).
- The matching PRD requirement R{N} (via command substitution).
- The Gate 3 manifest's "PRD revisions applied" section if status was `passed-with-revisions` (via command substitution).
- Their own role definition.

Lens emphasis at Gate 4 (each reviewer's role file documents these):
- **Goal-Driven:** verifies the plan's first paragraph follows R17's PRD-anchor format; verifies the cited R-ID resolves to a real PRD requirement; flags orphan ACs the plan introduces.
- **Surgical:** flags scope drift between the task's `Scope` field and the plan's listed file paths.
- **Simplicity:** flags premature abstraction or speculative configurability in the plan.
- **Think-Before-Coding:** flags unstated implementation assumptions (often: "the X module exists" when it doesn't; "the Y service returns Z" without cited evidence).
- **Devil's Advocate:** flags cross-artefact ambiguity (plan references a config flag the PRD doesn't mention; plan's lane assignment looks too aggressive given the scope; cross-product branches the writer missed).

Each reviewer writes a minimal finding JSON to `debate-log/develop-gate4/findings/round-1/{reviewer-name}.json` and returns only the path.

### C.4 Round 2 — AI responds

In your own forked context, read every Round-1 finding via command substitution. For each finding, write to `debate-log/develop-gate4/findings/round-2/responses.json`:

```json
{
  "round": 2,
  "responses": {
    "{reviewer}-r1-f{n}": {
      "decision": "accept | push_back",
      "rationale": "...",
      "revision_applied": "if accept: brief description of the plan revision"
    }
  }
}
```

If accepting: edit the per-task plan accordingly.

**Special case — re-lane forced (R5).** If a Round-1 finding has severity `block` AND the finding's `claim` field references the lane assignment as too aggressive AND you accept in Round 2: rename the current Gate 4 manifest to `manifest-T{N}-pre-relane.md`, record `closing_decision: re-lane forced (old → new)` in it, re-emit the per-task plan with the corrected lane, open a fresh Gate 4 manifest at the canonical path, and re-fire C.3. The second Gate 4 manifest's closing decision is what counts.

### C.5 Round 3 — Reviewers sharpen or accept

Re-dispatch each reviewer (fresh forked context) with their Round-1 finding + Round-2 response. Each writes to `debate-log/develop-gate4/findings/round-3/{reviewer-name}.json` per the schema in their role file. Decisions are `accept | sharpen`.

If any sharpen: re-edit the plan one more time and record the revision in `debate-log/develop-gate4/findings/round-3/ai-revision.md`.

### C.6 Closer — Orchestrator collates

Now act as the Orchestrator. Write `debate-log/develop-gate4/manifest.md`:

```markdown
# Gate 4 — plan vs tasks/PRD review

**Feature:** {NNN-slug}
**Task:** T{N}
**Date:** {YYYY-MM-DD}
**Reviewers:** devils-advocate, simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer
**Codex:** not invoked at Gate 4 (v1 hard-coded)
**Lane (post-mechanical-recheck):** {green|yellow|red}

## Accepted findings
...

## Rejected findings
...

## Escalated to human
...

## Closing decision

Gate 4 status: **passed | passed-with-revisions | passed-with-escalations | failed | re-lane forced**

{One paragraph closing rationale.}

— Orchestrator, {YYYY-MM-DD}
```

Status determination matches the Gate 2/3 taxonomy:
- `passed` — zero `block` findings landed (or all `block`s rejected with reviewer-accepted defences); zero accepted findings forced load-bearing plan revisions.
- `passed-with-revisions` — `block` or load-bearing-`concern` findings landed, accepted, plan revised, reviewers converged in Round 3.
- `passed-with-escalations` — at least one finding did not converge in 3 rounds.
- `failed` — at least one `block` finding was not resolved.
- `re-lane forced` — only used for the original manifest when R5 re-lane fires; the fresh manifest closes with one of the four standard statuses.

Refuse to proceed to Phase D if status is `failed`. Surface escalations to the user before continuing.

---

## Phase D — Lane execution

For each task in `T_run` whose Gate 4 closed with passed/passed-with-revisions/passed-with-escalations, execute according to lane.

> **Single context window per task.** Phases D / E / F for a given task run in one agent context window — no mid-task compaction, no cross-session resumption mid-implementation. If context approaches the cliff during execution, escalate to the developer (same three-option prompt as A.6); never compact silently. Compaction during develop is a defect signal, not a recovery move. Full contract in `templates/admin/single-context-task.md` (per 029-single-context-task).

> **Red → Green → Refactor cycle.** Inside the single context window, the agent's work follows Pocock's Red → Green → Refactor cycle (per 017-tdd-discipline). The cycle is *internal* to the task (not phase-numbered). Cycle markers (`red:` / `green:` / `refactor:`) write to `admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md` for retro audit; Phase E (Gate 5) reviewers do NOT see them. Full contract in `templates/admin/tdd-discipline.md`.

**Red sub-step.** The agent invokes `specflow:test {NNN-slug} --plan-only --task T{N}` to write the per-task plan section into `{NNN-slug}-test.md` with the primary AC's case marked `Status: red (failing)`. The agent then runs the targeted test command (e.g. `vitest run path/to/test`) against the pre-implementation state and captures the failing exit + stderr to `admin/scratch/{NNN-slug}-develop/red-test-trace-{task-id}.log`. **A pre-implementation pass is an invalid Red artefact** and refuses to enter Green — the agent halts and surfaces to the developer. Manifest marker: `red: passed (...)` once the failing exit is captured.

**Green sub-step.** Yellow lane refuses to enter Green without the Red artefact (the per-task plan section in `{NNN-slug}-test.md` containing at least one case marked `Status: red (failing)` AND the captured pre-implementation failing exit). Green lane behaviour is gated on `config.develop.tddRequired` — when `tddRequired: true` (the default), Green refuses to enter without the failing test, identically to Yellow. When `tddRequired: false`, Green may skip the Red sub-step and the per-task manifest stub records `red: skipped (config) (...)` alongside the operator's strong-CI-signal attestation. The knob applies to Green only. The agent writes the simplest change that makes the failing test pass; re-runs the same targeted test command (exit 0); appends to the trace log. Manifest marker: `green: passed (...)`.

**Refactor sub-step.** Bounded structural improvement under the green test as guard. Three explicit bounds (full contract in `templates/admin/tdd-discipline.md`): (a) no new behaviour, (b) no new files, (c) no scope creep / route to `specflow:scope-change`. Refactor is optional for trivial tasks. Manifest marker: `refactor: passed (...)` or `refactor: skipped (trivial) (...)` or `refactor: failed (new-file-attempted) (...)` per the outcome enum.

### D.1 Green-lane batched execution

Green-qualifying tasks whose dependencies are all satisfied (mechanical: `Depends on:` chain resolves to either shipped tasks or other tasks in the same batch) are grouped into a batch. The batch cap is `config.json.develop.greenBatchCap` defaulting to `3` — projects with strong CI signal can raise it. Tasks beyond the cap land in the next batch.

For each Green batch:

1. **Per-task agent execution** — for each task in the batch, dispatch the chosen agent (single-specialist or team-spawn output) in a forked sub-agent context. The agent reads the per-task plan via command substitution, makes the code changes, runs the machine checks (test, typecheck, lint) on its own code change set, and writes a per-task diff to `admin/scratch/{NNN-slug}-develop/diffs/{task-id}.patch`.
2. **Per-task Gate 5** — fire Phase E for each task individually as it completes (see Phase E).
3. **Per-task Verifier** — fire Phase F.1 for each task individually.
4. **Batch assembly** — once every task in the batch has Gate 5 closed and Verifier passed (or option-4 accepted-and-shipped), assemble the per-task diffs into a single PR. The PR description begins with the R17 PRD-anchor format (mirrored from each per-task plan's first paragraph) followed by the per-task gate-outcome summary table.
5. **Single batched human sign-off** — surface the assembled PR to the user with exactly one prompt: *"Sign off the assembled batch ({N} tasks) to merge the PR."*. Do not proceed without an explicit user response.

**Per-task gate failure inside a batch:** if any per-task Gate 4, Gate 5, or Verifier fails during the batch, pause at that task's boundary, write a partial PR description containing only the tasks completed up to and including the failing task, and prompt the user with three options:
- `resume` — retry from the failing task.
- `abort` — drop the entire partial batch.
- `drop` — drop the failing task and continue with the remainder of the batch.

The skill does not auto-default; an empty input or any input not matching `resume|abort|drop` re-prompts.

**Machine-check failure mid-task:** if `test`, `typecheck`, or `lint` fails for a Green-lane task's diff, force-upgrade the lane to Yellow for that task (lane upgrades are allowed; downgrades are not), surface to the user, and continue per Yellow-lane semantics (D.2) for the upgraded task.

### D.2 Yellow-lane HITL pairing

Yellow-qualifying tasks run one at a time, with the agent and human paired in real time. The agent emits the plan (already done at B.1.1 / C), executes the change in a forked sub-agent context, and surfaces every machine-check result as it lands. The human reviews each step in real time; the agent waits for explicit go-ahead before opening a PR.

Yellow lane is mandatory tests-first — Yellow refuses to enter the Green cycle step without the Red artefact (the per-task plan section in `{NNN-slug}-test.md` containing at least one case marked `Status: red (failing)` AND the captured pre-implementation failing exit). Cycle marker contract per `templates/admin/tdd-discipline.md`.

Yellow batch size is 1 — never grouped. Yellow-lane sign-off is per-task, not batched.

### D.3 Red-lane human-led with bounded AI assistance

Red-qualifying tasks are surfaced to the user with the per-task plan and lane evidence. The skill does NOT execute the change autonomously. The user leads; the AI assists on bounded subtasks the user explicitly delegates (e.g. *"draft the test fixture for the new endpoint"*, *"sketch the regex"*).

Red-lane plans always pick a single specialised agent (per R9, T9). Multi-agent composition is short-circuited.

When AI assists by drafting or changing testable code in a Red-lane task, EITHER the human supplies a Red artefact before any AI Green-like implementation assistance OR the per-task manifest stub records `red: skipped (human-led Red lane) (...)` and **no `green:` marker is emitted by the skill** — the audit signal stays honest. The TDD discipline is not enforced by the skill in Red lane; the human leads.

The user owns the PR open + Verifier invocation manually. The skill records the lane, plan, gate manifests, and Verifier outcome in the same shape as Green/Yellow, but the agent execution step is human-driven.

### D.4 PRD-anchor preservation in PR description

Every PR (per-task for Yellow/Red, batched for Green) opens with the R17 PRD-anchor format — paragraph 1 is the literal first paragraph of the per-task plan. For batched Green PRs, paragraph 1 lists each task's PRD anchor as a bulleted sub-list.

This serves PRD goal Audience surface (tertiary stakeholders): PR readers see the PRD anchor without invoking the skill.

---

## Phase E — Gate 5 multi-agent debate manifest (code vs plan)

Gate 5 reviews the code change set against the per-task plan. It always fires — never skipped. When Codex CLI is available it joins as a sixth reviewer; when absent the manifest writes a literal degraded-coverage sentinel and runs Gate 5 with the standard five.

### E.1 Set up the debate folder

```bash
mkdir -p docs/specflow/features/{NNN-slug}/debate-log/develop-gate5/findings/{round-1,round-2,round-3}
mkdir -p docs/specflow/features/{NNN-slug}/debate-log/develop-gate5/raw
```

### E.2 Identify reviewers

Standard set (always):
- `lifecycle/devils-advocate.md`
- `principles/simplicity-reviewer.md`
- `principles/surgical-reviewer.md`
- `principles/think-before-coding-reviewer.md`
- `principles/goal-driven-reviewer.md` — **load-bearing at Gate 5 when Codex absent** (code-vs-plan acceptance is its lens).
- `principles/edge-case-reviewer.md` — **load-bearing at Gate 5 for blindspot coverage** (per 028-edge-case-reviewer v2.6.0). Deliberately NOT goal-aware; runs in fresh context per 027; findings carry `recommendation` + `reasoning` and are advisory (orchestrator decides accept / reject / defer-to-misc).

Codex (when `cli.codex.available: true`):
- Invoked via `codex review` per the orchestrator-pattern fork convention. Codex reads the diff + plan via command substitution and writes its findings to `debate-log/develop-gate5/findings/round-1/codex.json` in the same shape as the other reviewers. Codex is **load-bearing at Gate 5 when present** — it covers cross-provider concerns the same-provider DA cannot cover by definition.

### E.3 Round 1 — parallel finding fire

Dispatch each reviewer (including Codex if present) as a forked sub-agent. Pass each:
- The code diff for the task (via command substitution from the patch file).
- The per-task plan (via command substitution).
- The task's tasks-file entry (via command substitution).
- Their own role definition.

Each writes to `debate-log/develop-gate5/findings/round-1/{reviewer-name}.json` and returns only the path.

### E.4 Round 2 — AI responds

Standard pattern (same as Gate 4). Push back where the principle's evidence threshold is met; accept where it isn't. If accepting: edit the code change set (or the plan, on rare occasions where the finding reveals a plan defect — but plan revisions at Gate 5 require a re-fire of Gate 4 if they cross the lane assignment).

### E.5 Round 3 — Reviewers sharpen or accept

Standard pattern. Decisions are `accept | sharpen` per the goal-driven-reviewer template.

### E.6 Codex-only-finding auto-promotion (R13)

After Round 3, scan Codex's Round-1 findings (when present) for IDs whose `claim` field has no overlap with any other reviewer's Round-1 findings. For each such codex-only finding:

1. Check `skills/misc/SKILL.md` for the `specflow:misc --auto` payload schema.
2. If the schema accepts `manifest_path` and `gate_finding_id` as named fields → invoke `specflow:misc --auto` with payload:
   ```json
   {
     "manifest_path": "features/{NNN-slug}/debate-log/develop-gate5/manifest.md",
     "gate_finding_id": "codex-r1-f{N}",
     "proposed_rule": "{rule text derived from the finding's evidence + claim}"
   }
   ```
3. If the schema does NOT include those fields → skip the auto-promotion AND surface the chat-line warning *"[develop: codex-only finding {id} not auto-promoted — specflow:misc schema gap; see AC-10 schema-dependency clause]"*. Record in the manifest as `auto_promotion_skipped: schema_gap`.

The skill never writes `admin/rules/guidelines.md` directly. The misc-task is the human-reviewed promotion path.

### E.7 Closer — Orchestrator collates

Write `debate-log/develop-gate5/manifest.md`:

```markdown
# Gate 5 — code vs plan review

**Feature:** {NNN-slug}
**Task:** T{N}
**Date:** {YYYY-MM-DD}
**Reviewers:** devils-advocate, simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer{, codex when available}
**Codex:** {findings sub-heading | "Codex not detected — same-provider review only. Cross-provider findings may be missed; install Codex CLI for full Gate 5 coverage."}

## Accepted findings
...

## Rejected findings
...

## Codex-only auto-promotions
- {finding-id} → misc-task {misc-task-id} | auto_promotion_skipped: schema_gap

## Closing decision

Gate 5 status: **passed | passed-with-revisions | passed-with-escalations | failed**

— Orchestrator, {YYYY-MM-DD}
```

Refuse to proceed to Phase F if status is `failed`. Surface escalations to the user.

---

## Phase F — Verifier confirmation + PR + Linear + task-history

### F.1 Verifier confirmation

Fork a Verifier sub-agent. Pass it:
- The task's tasks-file entry (especially the `Acceptance` field) via command substitution.
- The code diff via command substitution.
- The Gate 5 manifest path.

The Verifier checks each acceptance clause against the diff's observable behaviour. It returns one of:
- `pass` — every acceptance clause verified.
- `conditional-pass` — every acceptance clause verified, but at least one clause cited a cross-skill or downstream-PRD dependency that the Verifier could not directly close. The condition is named in the Verifier outcome with a concrete reference to the dependency.
- `reject` — at least one acceptance clause failed; structured failure payload follows.

### F.2 On Verifier pass

Proceed to PR open (F.4).

### F.2.1 On Verifier conditional-pass — escalation contract

When Verifier returns `conditional-pass`, write the condition into `admin/scratch/{NNN-slug}-develop/verifier-outcome.json`'s `task_outcomes[].result: "conditional-pass"` with the `evidence` field naming the dependency (e.g. *"depends on `specflow:develop` Phase F.5 default-flag emission — verified at SKILL.md:B.3 prerequisite clause"*). Then surface a two-option prompt to the user:

1. **Accept and proceed.** The conditional dependency is documented and accepted as load-bearing on a future change. Phase F continues to F.4 (PR open). The PR description body MUST include a `Conditional acceptance:` section citing the dependency verbatim. Downstream `specflow:complete` reads the conditional flag and surfaces it in the retro entry's `addenda` (kind: `escaped-issue` if the dependency is bug-shaped, `note` otherwise).
2. **Defer.** The user wants the cross-skill dependency resolved before this task ships. Phase F halts; surface the dependency as a follow-up task via `specflow:misc --auto` with a structured payload citing the AC, the dependency, and the Verifier outcome path. The original task remains open in the tasks file; Linear status reverts to In Progress (or stays In Progress if it was never moved). The user picks up the dependency PR first; this task re-runs `specflow:develop` from Phase A once the dependency lands.

The two-option prompt is the only path from `conditional-pass` to either F.4 or follow-up. **No third "force-pass" option** — Goal-Driven discipline requires the dependency to be either accepted as documented technical debt OR resolved before ship.

Verify after the user responds:
- `admin/scratch/{NNN-slug}-develop/verifier-outcome.json` has the conditional-pass entry with non-empty `evidence`.
- If accepted: PR description includes the `Conditional acceptance:` section.
- If deferred: a `misc-task` entry exists in `docs/specflow/misc-task/000-tasks-misc-tasks.md` referencing the AC and dependency.

### F.3 On Verifier reject — structured failure payload + four user options

Write `admin/scratch/{NNN-slug}-develop/verifier-failure-{task-id}.json`:

```json
{
  "task_id": "T{N}",
  "rejected_at": "{YYYY-MM-DD HH:MM}",
  "failed_ac": "AC-{N}",
  "verification_check_finding": "{summary of what the Verifier saw}",
  "diff_section": "{file}:{line-range}",
  "agent_name": "{agent-id-of-the-implementing-agent}"
}
```

Surface to the user:

*"Verifier rejected T{N} on AC-{N}. The check found {summary}. Choose:*
- *(1) re-implement (loop back to Phase D agent execution with the failure payload as additional context),*
- *(2) re-plan (loop back to Phase C — re-fires Gate 4),*
- *(3) abort the task and surface to `specflow:scope-change` (the AC may itself be wrong),*
- *(4) accept the failure and ship anyway (requires explicit confirmation, recorded as a `decision-log.md` entry)."*

The skill does not auto-default. An empty input or any input not matching `1|2|3|4` re-prompts. Option 4 writes a `decision-log.md` entry capturing the AC, the failure summary, and the user's stated reason for shipping.

The skill never silently re-implements — auto-loop-on-rejection is the failure mode this skill exists to remove.

### F.4 PR open + Linear In Review

Open the PR (per-task for Yellow/Red; batched for Green). The PR description's first paragraph is the per-task plan's PRD-anchor paragraph (or, for batched Green, a bulleted list of each task's anchor).

If `mcp.linear.available: true` AND Verifier passed (or option-4 was elected): fire the Linear status transition "In Progress → In Review" for each task.

The skill NEVER fires "In Review → Done" — that is a human merge decision, never the skill's. A run that fires a Done transition is a failed run.

If MCP unavailable, print the chat-only line: `[linear status: T{N} → In Review (skipped — MCP not available)]` and continue.

### F.5 Append development-time fields to `task-history.json`

For each completed task (Verifier pass OR option-4 accept-and-ship), read `admin/task-history.json`, locate the entry with id `{NNN-slug}-T{N}` (or create one if absent), and write the six development-time fields:

```json
{
  "id": "{NNN-slug}-T{N}",
  "feature": "{NNN-slug}",
  "lane_assigned": "green | yellow | red",
  "ai_assistance_level": "full | partial | bounded-subtasks",
  "elapsed_minutes": N,
  "what_worked": "{free text}",
  "what_didnt_work": "{free text}",
  "blast_radius_outcome": N
}
```

Pre-existing entries (created at task creation by `specflow:task` Phase D for user overrides) are updated in place; pre-task-creation fields (e.g. `ai_proposal`, `user_override`) are preserved.

### F.6 Cleanup scratch on success

After all `T_run` tasks complete and the per-run closure fires (sprint-mode: end-of-sprint; feature-mode: end-of-feature):

```bash
rm -rf admin/scratch/{NNN-slug}-develop/
```

Retain the scratch directory on failure for debugging.

---

## Failure modes

The following situations are explicit failure modes the skill handles without silent retry. Each maps to a documented user-elected response.

- **Gate 4 status `failed`** — refuse to proceed to Phase D. Surface the blocking findings to the user. The user revises the plan (manual edit) or re-runs the gate (`specflow:develop {NNN-slug} --task T{N}` to resume).
- **Gate 5 status `failed`** — refuse to proceed to Phase F. Surface the blocking findings. The user revises the code change set (manual edit) or re-runs the gate.
- **Verifier rejection** — structured failure payload + four user options per F.3. Never silent re-implement.
- **Machine-check failure on a Green-lane task** — force-upgrade lane to Yellow, surface to user, continue under Yellow semantics. Lane upgrades are allowed; downgrades are not.
- **agent-teams plugin absent** — Phase A.2 warns once, Phase B.4 falls back to sequential single-specialist invocation. Functionality preserved; throughput reduced.
- **agent-teams plugin present but `team-spawn` returns an error** — loud-fallback to single-specialist + chat-line warning per R8 / T8. The fallback never aborts the task.
- **Codex CLI absent** — Phase E.2 omits Codex from the reviewer set; Phase E.7 manifest's Codex section contains the literal degraded-coverage sentinel. Gate 5 always fires.
- **Confidential-path detected mid-execution** — if a code change set's actual diff touches a path matching `config.json.confidentialPaths` that wasn't in the original task scope, force-upgrade lane to Red, pause execution, surface to user. Lane downgrade is refused regardless of user preference.
- **`specflow:misc --auto` schema gap** — Phase E.6 skips the auto-promotion and surfaces the chat-line warning per R13 / T13. The Gate 5 manifest records `auto_promotion_skipped: schema_gap`.
- **Linear MCP absent** — chat-only status lines at A.4 and F.4. Never aborts.
- **Scope change requested mid-run** — refuse the in-skill edit. Direct the user to `specflow:scope-change`. Modifying `tasks.md` mid-run is forbidden.

---

## What you MUST NOT do

- **Do not skip Phase A's chain check.** A feature whose Gate 3 didn't close is not a finished task list; running Phase B against it imports the unresolved problems into Phase D.
- **Do not let the lane downgrade.** Lanes monotonically downgrade from Green to Yellow to Red; once at Red, no path returns. Upgrades-on-recheck are allowed (B.1.3 + machine-check failure at D.1); user-requested downgrades are refused.
- **Do not override confidentiality classification.** Path-glob matches against `config.json.confidentialPaths` always force Red. User requests to "just run it as Yellow this once" are refused with the literal sentinel.
- **Do not skip Phase B.1.** The mechanical pre-Gate-4 lane recheck is the load-bearing addition from Gate 2 dogfood block tbc-r1-f1; without it, the lane-aggressive-flag mechanism does not fire reliably.
- **Do not invoke Codex at Gate 4.** v1 hard-codes Codex to Gate 5. Setting `config.json.develop.codexAtGate4` has no effect.
- **Do not skip Gate 5.** Gate 5 always fires. When Codex is absent, the standard five-reviewer set runs and the manifest records the degraded-coverage sentinel.
- **Do not write `admin/rules/guidelines.md` directly.** Codex-only findings auto-promote via `specflow:misc --auto`, which is human-reviewed before the rule lands. The skill never writes the rules file directly.
- **Do not silently re-implement after Verifier rejection.** Always surface the four-option prompt. Auto-loop is the failure mode this skill exists to remove.
- **Do not bypass `--no-verify` on commits.** Pre-commit hooks exist for a reason; bypassing is refused with a link to `CORE_PRINCIPLES.md` (specifically the goal-driven-execution principle on inline verify steps).
- **Do not "looks correct" verify.** Verification is mechanical or AC-citation-based — every Verifier check binds to a specific acceptance clause from the tasks file.
- **Do not modify `tasks.md` mid-run.** Scope changes route through `specflow:scope-change`. The tasks file is the contract; in-flight edits break it.
- **Do not fire Linear "Done" transitions.** "In Review → Done" is a human merge decision, never the skill's.
- **Do not mention the underlying AI tooling or vendor** in any user-facing output, the per-task plan, the PR description, the gate manifests, the verifier failure payloads, or `task-history.json`. Per the project's CLAUDE.md, this is non-negotiable.

---

## Verify before declaring done

Before returning to the user with "sprint complete" (sprint-mode) or "feature complete" (feature-mode):

1. `admin/scratch/{NNN-slug}-develop/lane-assignments.json` exists with one entry per task in `T_run` (and zero entries for tasks NOT in `T_run`).
2. For every task in `T_run`, `admin/scratch/{NNN-slug}-develop/lane-recheck-{task-id}.json` exists (mechanical recheck fired before Gate 4). For tasks NOT in `T_run`, no recheck file exists from this run.
3. For every task in `T_run`, a Gate 4 manifest exists under `features/{NNN-slug}/debate-log/develop-gate4/` with closing decision `passed`, `passed-with-revisions`, or `passed-with-escalations` (NEVER `failed`). No Gate 4 manifest exists from this run for tasks outside `T_run`.
4. For every task in `T_run`, a Gate 5 manifest exists under `features/{NNN-slug}/debate-log/develop-gate5/` with closing decision `passed`, `passed-with-revisions`, or `passed-with-escalations`. No Gate 5 manifest exists from this run for tasks outside `T_run`.
5. For every task in `T_run`, `admin/task-history.json` contains an entry with all six development-time fields populated (`lane_assigned`, `ai_assistance_level`, `elapsed_minutes`, `what_worked`, `what_didnt_work`, `blast_radius_outcome`). Tasks outside `T_run` have NO new task-history entry from this run (any pre-existing entries are unchanged).
6. Every per-task plan's first paragraph matches the R17 PRD-anchor format; every PR description's first paragraph mirrors it.
7. `specflow:budget --skill specflow:develop {NNN-slug}` returns a JSON report with `parent_context_tokens < 30000` AND `max_per_gate_growth_tokens < 2000`.
8. Scratch directory `admin/scratch/{NNN-slug}-develop/` is cleaned up on success (or retained on failure for debugging).
9. Linear status transitions fired correctly when MCP available; chat-only fallback lines surfaced when absent.

If any verify step fails, fix it before returning. Do NOT claim the feature is complete with missing manifests, missing recheck files, or missing task-history fields.

---

## Reference

- `docs/PRD.md` Phase 2 scope item 1 — lane-based execution.
- `docs/PRD.md` Appendix L — full skill spec (Appendix L1 step list; L2 lane semantics; L4 rule auto-flag; L5 Codex Gate 5).
- `docs/PRD.md` Appendix N — multi-agent debate manifest spec (N1 Gates 4 + 5; N2 manifest shape).
- `docs/PRD.md` Appendix I3 — `task-history.json` schema (development-time fields).
- `templates/orchestrator-pattern.md` — fork / file handoff / command substitution conventions.
- `templates/agents/standard/lifecycle/orchestrator.md` — closer logic for Gate 4 + Gate 5.
- `templates/agents/standard/lifecycle/verifier.md` — Verifier role definition.
- `templates/agents/standard/lifecycle/devils-advocate.md` — DA role definition.
- `templates/agents/standard/principles/*.md` — principle-reviewer prompts; goal-driven and think-before-coding are load-bearing at Gate 4; goal-driven is load-bearing at Gate 5 when Codex absent.
- `skills/prd/SKILL.md` — sister Phase 1 orchestrator (Gate 2 pattern); mirror for consistency.
- `skills/task/SKILL.md` — sister Phase 1 orchestrator (Gate 3 pattern); mirror for consistency.
- `skills/agent/SKILL.md` — manages the agent index this skill consumes (read-only here).
- `skills/linear/SKILL.md` — Linear MCP wrapper used by Phase A.4 + F.4.
- `skills/misc/SKILL.md` — auto-promotion target for Codex-only Gate 5 findings.
- `skills/scope-change/SKILL.md` — handoff target for Verifier-rejection option 3.
- `skills/budget/SKILL.md` — token-consumption tracker referenced by the eval block + verify step 7.
- `skills/test/SKILL.md` — downstream consumer; runs after `specflow:develop` to capture the formal test plan.
