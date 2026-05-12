---
name: specflow:task
description: User-facing entry point for task generation. Five-phase orchestrator — A pre-flight + read PRD/interview, B task synthesis with coverage matrix, C intent summaries surfaced in chat, D override capture to task-history.json, E Gate 3 multi-agent debate manifest. Resumes intelligently if invoked on an in-flight feature.
status: v2-enhancement
phase: 1
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-interview.md
  - docs/specflow/features/{NNN-slug}/debate-log/prd-gate2/manifest.md
  - docs/specflow/admin/rules/non-negotiable.md
  - docs/specflow/admin/rules/guidelines.md
  - docs/specflow/admin/task-history.json
  - docs/specflow/admin/decision-log.md
  - docs/specflow/admin/lessons.json
produces:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md
  - docs/specflow/features/{NNN-slug}/debate-log/tasks-gate3/manifest.md
  - docs/specflow/features/{NNN-slug}/debate-log/tasks-gate3/findings/
  - docs/specflow/admin/task-history.json
eval: tasks file exists with one task per PRD requirement; coverage matrix shows 100% PRD-requirement coverage and zero orphan tasks; every task acceptance is binary; Gate 3 debate manifest closes with Orchestrator sign-off entry; any user-driven recut wrote a record to task-history.json; when {NNN-slug}-tasks.md has 3+ tasks, the closing Gate 3 manifest contains a Cross-task findings H2 section (or the literal `Cross-task review skipped:` line for sub-threshold runs); the manifest's writer_id, cross_task_reviewer_id, and applier_id fields are populated and pairwise distinct in the green path (022-cross-task-review v2.5.0); every task entry carries a `sprint-bucket: N` field with N >= 1, AND for every T_i with sprint-bucket: N every T_j in T_i.Depends on: has sprint-bucket: M with M < N (topological-floor corollary, strictly less-than per 025-sprint-task-flagging v2.5.0).
---

# specflow:task

You are the user-facing entry point for task generation. You own the full flow from "PRD signed off" to "task list reviewed, recorded, and gate-3 closed."

This is a **5-phase orchestrator**. Phase E delegates to forked sub-agents (the principle reviewers + Devil's Advocate) per the orchestrator pattern (see `templates/orchestrator-pattern.md`). Your parent context never accumulates the reviewers' raw work.

---

## Inputs

The user invokes you with a feature ID:
- `specflow:task {NNN}` or `specflow:task {NNN-slug}` — the feature must already have a PRD that closed Gate 2.
- `specflow:task --apply-cross-task-feedback {NNN-slug}` — applier-mode entry point (per 022-cross-task-review). Skips Phases A / B / C / D / E entirely and enters Phase F (cross-task feedback application). Orchestrator-internal contract; not typically invoked manually.
- `/specflow:task` with no argument — ask the user which feature.

**Resume logic.** Before starting Phase A, detect the situation:

1. Locate `docs/specflow/features/NNN-{slug}/`. If missing, refuse: *"Feature `{NNN-slug}` does not exist. Run `specflow:prd` first."*
2. Verify the PRD is gate-2-closed:
   - `features/NNN-{slug}/{NNN-slug}-prd.md` exists.
   - `features/NNN-{slug}/debate-log/prd-gate2/manifest.md` exists with a `**passed**`, `**passed-with-revisions**`, or `**passed-with-escalations**` closing decision.
   - If not closed (or status `failed`), refuse: *"PRD has not closed Gate 2 (status: `{status}`). Resolve Gate 2 before tasking. Re-run `specflow:prd {NNN-slug}` to resume."*
3. Determine the resume point:
   - **`--apply-cross-task-feedback` flag present** → skip Phases A / B / C / D / E entirely and jump to Phase F. Phase F's precondition check runs first (per `templates/task/cross-task-review.md`); on missing inputs, refuse with the canonical diagnostic.
   - **No tasks file** → start Phase A.
   - **Tasks file exists, no Gate 3 manifest** → resume Phase E.
   - **Tasks file + Gate 3 manifest exist** → ask the user: *"This feature appears tasked (Gate 3 closed). What do you want to do? (1) re-run Gate 3 with the existing tasks, (2) regenerate tasks from scratch — old tasks archived, (3) `specflow:scope-change` if the PRD itself needs revision."*

Tell the user explicitly which phase you're starting at.

---

## Phase A — Pre-flight + read PRD/interview

### A.0 Mode read (per 032-lightweight-mode v2.11.0)

Before A.1, read `features/{NNN-slug}/{NNN-slug}-feature.md` if present. Extract the `mode:` frontmatter field. Bind `MODE` for this run.

- `mode: light` → skip the heavy review chain. See per-phase skips below.
- `mode: full` or feature.md absent or `mode:` field missing → run the full review chain (legacy behaviour).

Surface a one-line chat note: *"Mode: `{MODE}`."*

When `MODE == "light"`, the following phases skip:

- **Phase B.5 (pre-Gate-3 Codex adversarial pass)** — skipped entirely. Write a one-line `pre-gate-codex.md` stating *"Skipped in light mode — no adversarial surface for trivial changes."* and proceed.
- **Phase D (per-task reviewer rounds)** — skipped. The per-task reviewer multi-agent passes (Round 1 / 2 / 3) add no signal when the task set is 1-2 small tasks.
- **Phase E.4.5 (cross-task review)** — skipped. Cross-task arrangement analysis on a 1-2 task set is pointless.
- **Phase E (Gate 3 multi-agent debate manifest)** — skipped entirely. Write a stub manifest at `debate-log/tasks-gate3/manifest.md` with closing decision **passed (light mode — no multi-agent review)** and a one-line rationale citing the mode value.

Phases B.1-B.4 (coverage matrix, self-checks for budget / duration / graph-validity), B.6 (sprint-bucket assignment), and Phase C (intent summaries to the user) still run regardless of mode — they're the actual decomposition work, not review.

### A.1 Verify the artefact chain

Use Read tool in parallel on:
- `features/NNN-{slug}/{NNN-slug}-prd.md`
- `features/NNN-{slug}/{NNN-slug}-interview.md`
- `features/NNN-{slug}/debate-log/prd-gate2/manifest.md`
- `admin/rules/non-negotiable.md`
- `admin/rules/guidelines.md`
- `admin/task-history.json` (empty array `{"tasks": []}` is fine)
- `admin/decision-log.md` (optional)
- `admin/lessons.json` (the project's self-learning corpus; missing or `[]` is fine on a fresh project)

### A.2 Surface Gate 2 block-finding resolutions

If the Gate 2 manifest's status is `passed-with-revisions`, read its "PRD revisions applied" section. Each revision listed there is a load-bearing constraint the tasks must respect. For example, if a Gate 2 `block` finding added a new requirement R5.1 (mechanical pre-Gate-4 lane recheck step), the tasks should include a distinct task for the mechanical recheck *separate from* the catch-all reviewer-driven re-lane (R5). Do not merge tasks the Gate 2 process deliberately separated.

Note any revisions that produced new R-level requirements; these inherit the forward-coverage rule (≥1 task per R) automatically when extracted in Phase A.3.

If the manifest status is `passed` (no revisions section expected) or `passed-with-escalations`, skip this sub-step.

### A.2.5 Surface design iteration-log decisions (when present)

If `features/NNN-{slug}/design/{slug}-iteration-log.md` exists, read it (010-design-readback, v2.4.0). For every entry whose timestamp is *later* than the PRD's frontmatter date, the entry's *Why* field is a post-PRD decision tasks must respect — the PRD body may not reflect it yet, but the proposed.html implementation does.

For each post-PRD iteration entry:
- If the entry maps cleanly to an existing R or AC, link it: the task derived from that R gains a `design-decision: iteration-N` field referencing the log entry.
- If the entry expresses a constraint not covered by any R, surface to the user: *"Iteration {N}'s decision ({one-line summary}) isn't covered by any PRD requirement. Proceed with the decision noted, or run `specflow:scope-change` to add a requirement?"* Default if user accepts: proceed; the decision is logged in `admin/scratch/{NNN-slug}-tasks/post-prd-design-decisions.json` and the user-facing summary lists them.

If `design/` is absent or `iteration-log.md` has no entries newer than the PRD, skip silently.

### A.3 Extract the PRD's load-bearing fields

You need clean lists for Phase B's coverage matrix:
- **Requirements** — every `R1`, `R2`, …, with their Trace + Serves-goal pairs. Include any sub-requirements (e.g. `R5.1`) introduced by Gate 2 revisions.
- **Acceptance criteria** — every `AC-1`, `AC-2`, …, with which requirements they verify.
- **Non-goals** — used in Phase E (Surgical Reviewer cross-checks).
- **Open questions** — flagged so any task that depends on an unresolved question is marked.
- **Gate 2 revisions applied** — surfaced from A.2 when present; tasks reviewing these revisions get a `gate2-revision: {finding-id}` field linking back to the manifest.

Write a small extraction file to `admin/scratch/{NNN-slug}-tasks/prd-extracts.json` (orchestrator-pattern: scratch directory per orchestration). The reviewers in Phase E read this via command substitution.

### A.4 Query the lessons registry

Read `admin/lessons.json` (or treat as `[]` if missing). Apply the query algorithm specified in `skills/test/SKILL.md` § Lessons registry → Query algorithm:

1. **Build the query tag set.** PRD frontmatter (audience / domain), `environment.json` (stack), and a keyword scan of the PRD's Requirements section against the canonical surface vocabulary (`ui`, `data-model`, `api`, `auth`, `migration`, `infra`, `cli`, `docs`).
2. **Filter.** Active lessons whose `tags` array overlaps the query tag set by ≥1 tag.
3. **Rank.** By `occurrences.length` desc, then `first_seen` desc.
4. **Cap.** At 5 surfaced lessons; write the full matched list to `admin/scratch/{NNN-slug}-tasks/matched-lessons.json` for audit.

If matches found, surface in chat:

```
Heads up — relevant lessons from prior work on this project:
- L-001 (Splash screen wrong font + wrong loading state) — UI tasks need a Playwright screenshot diff before Verifier passes.
- L-007 (Token refresh silent failure) — auth tasks must include a token-expiry test case.

Task synthesis below biases toward incorporating these.
```

Each matched lesson's `remediation` becomes a "must consider" line in B.1's task derivation. If an existing PRD requirement already covers a lesson, the task derived from that requirement gets a `lesson-anchor: L-NNN` field. If no existing requirement covers a lesson, surface to the user: *"L-{NNN}'s remediation isn't covered by any PRD requirement. Proceed with the lesson noted (no task), or run `specflow:scope-change` to add a requirement?"* Default if user accepts: proceed; the lesson is logged in `admin/scratch/{NNN-slug}-tasks/uncovered-lessons.json` and listed in the user-facing summary.

If no matches, note: *"No prior lessons match this feature's tags."* Continue.

### A.5 Tell the user what you're doing

*"Read the PRD and interview. Synthesising tasks now — every PRD requirement gets at least one task; every task anchors to a requirement and has a binary acceptance check.{If lessons matched: 'Lessons L-NNN, L-NNN are being woven into the plan.'}"*

---

## Phase B — Task synthesis with coverage matrix

### B.1 Derive tasks from requirements

For each PRD requirement, derive one or more tasks. A task is the smallest shippable unit that:
- Anchors to ≥1 PRD requirement (cite the requirement ID).
- Has a binary acceptance check (cite the AC ID it satisfies, or write a new one tied to the requirement).
- Names the surfaces it touches (files, components, modules).
- Is independently reviewable — a reviewer can read this one task and tell whether it's done without holding the rest of the task list in their head.

**Sizing heuristics:**
- A task that touches >5 files is probably two tasks. Split it.
- A task whose acceptance criterion contains the word "and" twice is probably two tasks. Split it.
- A task that depends on more than one other task should be reordered or split — long dependency chains kill parallelism.

### B.2 Build the coverage matrix

Cross-tabulate PRD requirements against derived tasks. Two checks must pass:
- **Forward coverage:** every requirement has ≥1 task.
- **Reverse traceability:** every task anchors to ≥1 requirement. **Zero orphan tasks** — a task that doesn't trace to a requirement is scope creep and must be either dropped or surfaced as a `misc-task` candidate.

If a requirement has no task, you've missed it — go back to B.1.
If a task has no requirement, drop it OR (if you believe it's load-bearing) surface to the user as an explicit ask: *"Task T{N} doesn't trace to any PRD requirement. Either: (a) it's scope creep and I'll drop it, (b) the PRD is missing a requirement I should propose. Which?"*

### B.3 Write `tasks.md`

Use Write tool to create `docs/specflow/features/NNN-{slug}/{NNN-slug}-tasks.md`:

```markdown
---
feature: NNN-slug
status: draft
created: {YYYY-MM-DD}
prd: ./{NNN-slug}-prd.md
interview: ./{NNN-slug}-interview.md
gate3: ./debate-log/tasks-gate3/manifest.md
---

# Tasks — {Feature title — derived from slug, title-cased}

## Coverage matrix

| PRD requirement | Tasks satisfying it |
|---|---|
| R1 | T1, T3 |
| R2 | T2 |
| R3 | T4, T5 |

## Tasks

### T1 — {short title, ≤8 words}
- **Anchor:** PRD R1 — *{paraphrase the requirement in one line}*
- **Scope:** {file paths, components, modules touched. Be specific.}
- **Acceptance:** {binary pass/fail check. Cite AC-N from the PRD if applicable.}
- **Depends on:** {T-id of any task that must complete first, or "none"}
- **context-budget-estimate:** {int_tokens — sum of PRD slice + task spec + matched lessons + manifest scaffold + codebase-context payload + test plan, per `templates/admin/single-context-task.md`}
- **sprint-bucket:** {positive integer ≥ 1 — derived deterministically from the dependency graph + scope-overlap per `templates/task/sprint-bucket-heuristic.md` (per 025-sprint-task-flagging v2.5.0)}
- **prior-lessons:** {array of L-NNN ids that shaped this task, per the lessons registry query at A.4 (per 018-lessons-registry v2.6.0); empty array `[]` when no lessons apply}
- **Notes:** {gotchas, rule-registry entries that apply, decision-log references; or "none"}

### T2 — {short title}
- **Anchor:** PRD R2 — *{paraphrase}*
- ...

(continue for every task — typical: 5-15 tasks per PRD)

## Open questions inherited from PRD

{If the PRD's "Open questions" section had any, list them here so any task that
 depends on an unresolved question is flagged. Format: "Q: {question} — affects: T{N}, T{M}".}

## See also

- PRD: [`./{NNN-slug}-prd.md`](./{NNN-slug}-prd.md)
- Interview: [`./{NNN-slug}-interview.md`](./{NNN-slug}-interview.md)
- Gate 3 manifest: [`./debate-log/tasks-gate3/manifest.md`](./debate-log/tasks-gate3/manifest.md) (generated by Phase E)
```

### B.4 Self-check before Phase C

Before surfacing intent summaries, verify:
1. **Forward coverage** — re-walk the matrix; every PRD requirement appears in the *Tasks satisfying it* column.
2. **Reverse traceability** — every task's *Anchor* line cites at least one valid requirement ID.
3. **Binary acceptance** — every task's *Acceptance* line passes the binary test (could a fresh agent run the check and report pass/fail unambiguously). If not, sharpen the acceptance line.
4. **No requirement contradicts the goal's Out-of-scope-at-goal-level** — re-read the interview's Goal section and cross-check.
5. **Budget self-check (per 029-single-context-task).** For each task, verify `context-budget-estimate` ≤ `config.json.task.contextBudget` (default 80,000 tokens). Tasks over budget are auto-flagged: append an inline note `> Budget overrun: estimate {N}K vs budget {M}K — split required before develop.` under the task block AND surface a chat-line prompt directing the user to `specflow:scope-change` for the recut. The over-budget task remains in the file with the warning so the recut is auditable. Citation: `templates/admin/single-context-task.md` for the estimation algorithm and the no-mid-task-compaction rationale.

6. **Graph-validity check (per 025-sprint-task-flagging).** Before bucket assignment, walk the per-task `Depends on:` lists and reject malformed graphs with deterministic `GRAPH-INVALID:` diagnostics — cycle, self-loop, duplicate task IDs, duplicate dependency edge, dangling reference. On any failure, abort synthesis before any `sprint-bucket: N` is written; point the user to `specflow:scope-change` for legitimate dependency-graph edits. Diagnostic format documented in `templates/task/sprint-bucket-heuristic.md`.

7. **Sprint-bucket assignment (per 025-sprint-task-flagging).** Apply the single-rule heuristic from `templates/task/sprint-bucket-heuristic.md`: `bucket(T) = 1` for tasks with no predecessors and no same-bucket scope conflict; otherwise `1 + max(bucket(P) for P in Depends-on(T) ∪ EarlierIDSameBucketScopeConflicts(T))`. Apply bump iteration to fixed point. Bucket assignment runs AFTER step 5 (budget self-check) — oversize tasks never reach bucketing.

Tasks are also sized at synthesis to fit within `config.task.maxDurationHours` (default `1`); when the value is `"auto"`, sizing is enforced by `contextBudget` alone.

If any check fails, fix the tasks file before proceeding.

### B.5 Pre-Gate-3 Codex adversarial pass

Before Phase C surfaces intent highlights, run a programmatic Codex adversarial pass against the tasks file and capture verbatim output as a file artefact at `features/{NNN-slug}/debate-log/tasks-gate3/pre-gate-codex.md`. The user reviews highlights against an already-vetted list; the multi-agent Gate 3 panel (Phase E) reviews a sharpened artefact.

If `admin/environment.json` has `cli.codex.available: false`, write the file with one line — *"Codex CLI not detected — pre-gate pass skipped. Install via `/codex:setup` for full coverage."* — and proceed to Phase C. The in-gate Codex reviewer at Phase E follows the same env gating.

Otherwise:

1. Pre-create the debate-log folder if it doesn't exist: `mkdir -p features/{NNN-slug}/debate-log/tasks-gate3`. (E.1 also runs `mkdir -p`; both are idempotent.)
2. Bash-invoke `codex adversarial-review` against the tasks file per the orchestrator-pattern fork convention (mirrors develop Phase E.2's in-gate `codex review` invocation). Frame the prompt to challenge whether every PRD requirement maps to a task, whether task scopes are surgical, whether dependency ordering is sound, and whether every acceptance line is binary. Capture stdout to the file path above.
3. On invocation failure (auth, network, exec error), write the error verbatim to the same path with prefix *"Codex pass failed at runtime:"* and continue to step 4.
4. Tell the user: *"Pre-gate Codex pass written to `{path}`. Reply `continue` to proceed to Phase C, `revise: <description>` to address a specific gap inline, `recut: <description>` to regenerate tasks against the gap, or `skip` to proceed without revisions."*

On `continue`: append `— User reviewed; no revisions, {YYYY-MM-DD}.` to the file. Proceed to C.1.
On `revise: <description>`: edit the tasks file inline to address the gap, re-run B.2 (rebuild coverage matrix) and B.4 (self-check), then re-prompt at B.5.
On `recut: <description>`: archive the current tasks file as `{NNN-slug}-tasks.md.pre-recut.bak`, loop back to B.1 with the user's gap as additional context, run B.2/B.3/B.4 from scratch, then re-prompt at B.5.
On `skip`: append `— User skipped without revisions, {YYYY-MM-DD}.` to the file. Proceed to C.1.

---

## Phase C — Intent summaries (chat-only)

### C.1 Pick 3-5 highlighted tasks

Choose tasks that are:
- Most user-visible (reviewer can verify by using the feature, not just reading code).
- Most architecturally load-bearing (touches a primary module or introduces a new pattern).
- Hardest to estimate (would benefit from human sanity-check before lock-in).

If the task list has fewer than 5 total, summarise all of them.

### C.2 Write the summaries (in chat, not to file)

For each highlighted task, write a 2-sentence non-technical summary:
- Sentence 1 — *what changes for the user / system when this task is done.*
- Sentence 2 — *what's the reviewable surface (where you'd look to confirm).*

Surface them in chat under a header like:

```
## Highlights

**T2 — Add notification popover trigger**
When a notification arrives, the header shows a badge with an unread count, and clicking it opens the popover with the latest notifications. Reviewable in the live header — open the dev server, fire a notification, click the badge.

**T5 — Persist read state to API**
Marking a notification as read syncs to the backend so the badge state survives page reload. Reviewable by reading a notification, refreshing, and confirming the badge stays cleared.

...
```

This is **chat-only**. Do NOT write the summaries to a file. They're a sanity-check surface, not an artefact.

### C.3 Ask for sign-off

*"Coverage matrix shows {N} tasks covering all {M} PRD requirements. Highlights above. Sign off to proceed to Gate 3 review, or call out any tasks you want recut, retitled, re-anchored, dropped, or added."*

---

## Phase D — Override capture

### D.1 Detect overrides

If the user accepts everything, skip to Phase E. If the user redirects:
- *Recut* — split or merge tasks.
- *Retitle* — rename a task.
- *Re-anchor* — change the PRD requirement(s) a task traces to.
- *Drop* — remove a task.
- *Add* — introduce a new task (must trace to a PRD requirement).

For each override:

1. Apply it to `{NNN-slug}-tasks.md` (use Edit tool for surgical changes; use Write only if regenerating the whole file).
2. Re-verify the coverage matrix (Phase B.4 checks must still pass).
3. Append a record to `admin/task-history.json` under the `tasks` array:

```json
{
  "id": "{NNN-slug}-T{N}",
  "feature": "{NNN-slug}",
  "title": "{final title}",
  "anchor": "PRD R{N}",
  "ai_proposal": {
    "title": "{what AI proposed}",
    "anchor": "PRD R{N}",
    "scope": "{AI's scope}",
    "acceptance": "{AI's acceptance line}"
  },
  "user_override": {
    "type": "recut | retitle | re-anchor | drop | add",
    "reason": "{user's stated reason, or 'unstated' if not given}",
    "applied_at": "{YYYY-MM-DD HH:MM}"
  },
  "captured_at": "{YYYY-MM-DD HH:MM}",
  "phase": "task-creation"
}
```

The other fields in the I3 schema (`linearId`, `actualHours`, `whatWorked`, `whatDidntWork`, `aiAssistance`) are populated later by `specflow:complete` — leave them off the entry at creation time.

### D.2 Confirm before Phase E

After all overrides are applied: *"{N} overrides recorded to `task-history.json`. Re-running coverage check… all checks pass. Proceeding to Gate 3."*

If a coverage check now fails (e.g. user dropped a task that was the only cover for R3), refuse to proceed: *"Override left R{N} uncovered. Either restore a covering task or run `specflow:scope-change` to revise the PRD."*

---

## Phase E — Gate 3 multi-agent debate manifest

### E.1 Set up the debate folder

Use Bash:

```bash
mkdir -p docs/specflow/features/NNN-{slug}/debate-log/tasks-gate3/findings/{round-1,round-2,round-3}
mkdir -p docs/specflow/features/NNN-{slug}/debate-log/tasks-gate3/raw
touch docs/specflow/features/NNN-{slug}/debate-log/tasks-gate3/manifest.md
```

### E.2 Identify reviewers

From `docs/specflow/admin/agents/standard/`, the standing reviewer set:
- `lifecycle/devils-advocate.md` — always present.
- `principles/simplicity-reviewer.md`
- `principles/surgical-reviewer.md`
- `principles/think-before-coding-reviewer.md`
- `principles/goal-driven-reviewer.md` — **load-bearing at Gate 3** (coverage matrix is its primary lens).

Plus, if `admin/environment.json` has `cli.codex.available: true`, include Codex as a sixth reviewer.

### E.3 Round 1 — parallel finding fire

For each reviewer, dispatch a forked sub-agent (Agent tool with the reviewer's role definition as the brief). Pass each reviewer:
- The tasks path (`features/NNN-{slug}/{NNN-slug}-tasks.md`).
- The PRD path (read second — for traceability checking).
- The interview path (read third — for goal alignment).
- The Gate 2 manifest path (read fourth — escalations from Gate 2 may indicate task-level concerns).
- Their own role definition (from `admin/agents/standard/{lifecycle,principles}/{reviewer}.md`).
- The PRD-extracts file (`admin/scratch/{NNN-slug}-tasks/prd-extracts.json`) for clean requirement/AC lists.
- Instruction: write a minimal finding JSON to `debate-log/tasks-gate3/findings/round-1/{reviewer-name}.json` (schema in the reviewer's role file) and return only the file path.

The reviewers' specific Gate-3 lenses (each role file documents these in detail):
- **Goal-Driven Reviewer:** primary lens. Verifies forward coverage (every R has a T) and binary acceptance for every task.
- **Surgical Reviewer:** verifies reverse traceability (every T anchors to an R) and flags any task whose scope drifts past the requirement it claims to satisfy.
- **Simplicity Reviewer:** flags over-decomposition (3 tasks where 1 would do), speculative tasks "for later", and acceptance criteria that overspecify implementation.
- **Think-Before-Coding Reviewer:** flags tasks whose acceptance depends on unstated assumptions (often: "the X service exists" when it doesn't).
- **Devil's Advocate:** flags cross-artefact drift between tasks and the Gate 2 manifest (escalations the tasks didn't address).

Wait for all reviewers to return their finding paths.

### E.4 Round 2 — AI responds

In your own forked context, read every Round-1 finding via command substitution (`!{cat features/NNN-{slug}/debate-log/tasks-gate3/findings/round-1/*.json}`). For each finding, write a structured response to `debate-log/tasks-gate3/findings/round-2/responses.json`:

```json
{
  "round": 2,
  "responses": {
    "{reviewer}-r1-f1": {
      "decision": "accept | push_back",
      "rationale": "...",
      "revision_applied": "if accept: brief description of the revision applied to tasks.md"
    }
  }
}
```

If accepting: actually edit `{NNN-slug}-tasks.md` to apply the revision. If revisions are substantial, re-run Phase B.4 self-check before continuing.

### E.4.5 — Cross-task review (per 022-cross-task-review v2.5.0)

This sub-phase fires only when the post-Round-2 `{NNN-slug}-tasks.md` carries 3 or more tasks (counted by `### T-` headings). When fewer than 3 tasks exist, skip this sub-phase entirely and write to the manifest's "Cross-task findings" section a single line: `Cross-task review skipped: task count {N} below threshold (3)`.

When firing, run a three-round mini-debate. The reviewer (`cross-task-reviewer`) and the applier (`specflow:task --apply-cross-task-feedback`) each run in their own forked sub-agent in a fresh context window. The original Phase E.4 orchestrator does NOT respond to or auto-apply cross-task findings — only the dedicated applier does. Full operational contract in `templates/task/cross-task-review.md`.

#### E.4.5.1 — Cross-task R1 fire

Dispatch a forked sub-agent reading `admin/agents/standard/principles/cross-task-reviewer.md`, the post-Round-2 `{NNN-slug}-tasks.md`, the PRD, the interview, and the Gate 2 manifest. The reviewer writes findings to `debate-log/tasks-gate3/findings/round-1/cross-task-reviewer.json` with per-finding `lens` field (`coherence | better-arrangement`).

#### E.4.5.2 — Applier R2 response + apply

Re-invoke `specflow:task --apply-cross-task-feedback {NNN-slug}` in a fresh context window. The applier reads the cross-task R1 findings + the post-Round-2 tasks.md + the PRD/interview, decides accept / reject / scope-change-required per finding, applies accepted changes to tasks.md, and writes its decisions to `debate-log/tasks-gate3/findings/round-2/cross-task-responses.json`.

#### E.4.5.3 — Cross-task R3 sharpen

Re-dispatch the cross-task reviewer (fresh forked sub-agent) reading its R1 findings + the applier's R2 responses + the post-applier tasks.md. May sharpen any rejection with new evidence or escalated severity. Writes to `debate-log/tasks-gate3/findings/round-3/cross-task-reviewer.json`.

#### E.4.5.4 — Applier final pass

If any sharpens, re-invoke the applier on the sharpened findings. Decisions append to `cross-task-responses.json`. Unresolved findings escalate to the manifest's Cross-task Escalations sub-section. Phase E.4.5.4 must complete writing the post-applier tasks.md before Phase E.5 starts.

**Sub-agent dispatch failure fallback.** If any sub-agent fails to return a valid finding/response JSON (network drop, harness crash, malformed JSON, missing finding file), log the failure to `debate-log/tasks-gate3/findings/round-{N}/cross-task-{role}.failure.json`. The gate escalates as `passed-with-escalations` (NOT failed); the manifest closer notes *"Cross-task review unavailable: {reason}; per-task review remains authoritative for this run."*

### E.5 Round 3 — Reviewers sharpen or accept

Phase E.5 fires AFTER Phase E.4.5 closes. Re-dispatch each per-task reviewer (fresh forked context) with their Round-1 finding + the Round-2 response, against the **post-applier** `{NNN-slug}-tasks.md`. Each writes to `debate-log/tasks-gate3/findings/round-3/{reviewer-name}.json` per the schema in their role file.

Hybrid R3 surface (per 022-cross-task-review):

- Sharpen R1 findings whose anchored T-id still exists in the post-applier tasks.md (standard R3 sharpen).
- Treat R1 findings whose T-id was merged-out / dropped as auto-resolved — recorded in manifest as `resolved-by-cross-task-merge: T{N}->T{M}` or `resolved-by-cross-task-drop: T{N}` per finding.
- Net-new findings on tasks introduced by the applier (e.g. a merged T3+T7 → new T3.5) are recorded as `round-3-net-new` per-task findings AND require a one-pass orchestrator response (no R4 sharpen — net-new findings are themselves an exit lever).

If any sharpen: re-edit the tasks file one more time and record the revision in `debate-log/tasks-gate3/findings/round-3/ai-revision.md`.

### E.6 Closer — Orchestrator collates

Now act as the Orchestrator (per `lifecycle/orchestrator.md` closer logic). Read all findings + responses across all three rounds. Write `debate-log/tasks-gate3/manifest.md`:

```markdown
# Gate 3 — tasks vs PRD review

**Feature:** NNN-slug
**Date:** {YYYY-MM-DD}
**Reviewers:** {comma-separated list, including cross-task-reviewer when 3+ tasks}
**Artefact under review:** {NNN-slug}-tasks.md
**writer_id:** {opaque value generated at synthesis dispatch time}
**cross_task_reviewer_id:** {opaque value generated at E.4.5.1 dispatch time, or empty if skipped}
**applier_id:** {opaque value generated at E.4.5.2 dispatch time, or empty if skipped}

## Accepted findings
- **{finding-id}** ({reviewer}, {severity}) — {claim}
  - Evidence: {evidence}
  - Revision applied: {description}

## Rejected findings
- **{finding-id}** ({reviewer}, {severity}) — {claim}
  - Evidence: {evidence}
  - Reason for rejection: {AI's Round-2 push-back, sharpened in Round 3 if applicable}

## Escalated to human
- **{finding-id}** ({reviewer}, {severity}) — {claim}
  - Reason: reviewers and writer did not converge in 3 rounds; surfaced for human decision.
  - Recommendation: {one-line suggested resolution}

## Cross-task findings

### Accepted findings
- **{finding-id}** (cross-task-reviewer, lens: coherence|better-arrangement, {severity}) — {claim}
  - Evidence: {evidence}
  - Revision applied: {description}

### Rejected findings
- **{finding-id}** (cross-task-reviewer, lens: ..., {severity}) — {claim}
  - Evidence: {evidence}
  - Reason for rejection: {applier rationale}

### Escalated to human
- **{finding-id}** (cross-task-reviewer, lens: ..., {severity}) — {claim}
  - Reason: {scope-change-required diagnostic, sub-agent-failure fallback, or unconverged block}

(When `Cross-task review skipped: task count {N} below threshold (3)`, the three sub-headings above are replaced by that single line.)

## Closing decision

Gate 3 status: **passed | passed-with-revisions | passed-with-escalations | failed**

{One paragraph closing rationale by the Orchestrator. Names which lens(es) drove the status — e.g. "passed-with-escalations: per-task PASSED; cross-task escalated 1 block-severity coherence finding". If passed: tasks fit to proceed to specflow:develop or specflow:test. If escalations: list items the human must resolve. If failed: list blocking findings and what must change.}

— Orchestrator, {YYYY-MM-DD}
```

Apply the orchestrator's pass/fail rules from `lifecycle/orchestrator.md` (FAIL on any unresolved `block`; HUMAN-DECISION-NEEDED on any unconverged `block`; PASS otherwise). The FAIL rule applies to the UNION of per-task and cross-task findings — a `block`-severity finding from EITHER lens triggers the same status. The closing rationale paragraph names which lens(es) drove the status.

### E.7 Final disposition

If Gate 3 status is **passed** or **passed-with-escalations**: tell the user *"Tasks synthesised and Gate 3 review complete. Status: {status}. Manifest at `debate-log/tasks-gate3/manifest.md`. Next step: `specflow:test {NNN-slug}` for verification cadence, or `specflow:develop {NNN-slug}` (Phase 2) to begin implementation."*

If escalations exist, list them in your response so the user sees them without opening the manifest.

If Gate 3 status is **failed**: tell the user *"Tasks failed Gate 3 review. Blocking findings:\n{list}\n\nReview the manifest at `debate-log/tasks-gate3/manifest.md` and either revise the tasks or run `specflow:scope-change` if the PRD itself needs revision, then re-run `specflow:task {NNN-slug}` to resume from Phase A."*

### E.8 Clean up scratch

After successful close, remove the scratch directory:

```bash
rm -rf admin/scratch/{NNN-slug}-tasks
```

(Retain on failure for debugging.)

---

## Phase F — Cross-task feedback application (per 022-cross-task-review)

This phase fires only when `specflow:task` is invoked with `--apply-cross-task-feedback {NNN-slug}`. Phases A / B / C / D / E are skipped entirely. Full operational contract in `templates/task/cross-task-review.md`.

### F.1 Precondition check

Phase F's first action is a precondition check on three inputs:

1. The post-Round-2 `{NNN-slug}-tasks.md`.
2. The cross-task findings JSON at `debate-log/tasks-gate3/findings/round-1/cross-task-reviewer.json` (or `round-3/...` for the final pass).
3. The PRD + interview for coverage-matrix validation.

On missing `cross-task-reviewer.json`, refuse with the canonical sentinel: *"Cross-task review has not fired for this feature. Run `specflow:task {NNN-slug}` from Phase E to produce the findings, then re-invoke `--apply-cross-task-feedback`."* On missing tasks.md or PRD, refuse with the analogous diagnostic. The chain-skip-Phase-A behaviour assumes the parent run guaranteed chain verification; manual standalone invocation is an unsupported entry point in v2.5.

### F.2 Per-finding decision + apply

For each finding in the input JSON, decide one of: `"decision": "accepted"`, `"decision": "rejected"`, or `"decision": "scope-change-required"`. Write decisions to `debate-log/tasks-gate3/findings/round-2/cross-task-responses.json` with the schema documented in `templates/task/cross-task-review.md`.

`accepted` decisions produce in-place edits to `{NNN-slug}-tasks.md` (merge / split / reorder / add). After each edit, re-verify the coverage matrix (Phase B.4 logic re-applied — forward + reverse coverage; binary acceptance; budget self-check per 029-single-context-task).

`rejected` decisions stand. `scope-change-required` decisions halt application for that finding and route to `specflow:scope-change`.

PRD-coverage-changing findings (add missing task with new R/AC anchor; merge-with-coverage-ambiguity; drop-with-coverage-hole) MUST be flagged `scope-change-required`; the applier does NOT modify the PRD's coverage matrix on the fly.

### F.3 Hard-cap enforcement (per 029-R4)

The applier MUST decline any merge whose combined `context-budget-estimate` would exceed the configured per-task budget cap. **Phase F reads `config.task.contextBudget` from the active config at Phase F entry** — never from a value embedded in the Round-1 finding, tasks.md, or synthesis-time snapshot. Such mergers are flagged `scope-change-required` with rationale `merge would violate 029 R4 hard-cap (combined budget {N}K > cap {C}K) — re-synthesis required`.

### F.4 Sprint-bucket recompute (per 025)

When the applier accepts a merge or split:

- **Merge** — the merged task's `sprint-bucket` is recomputed via 025's heuristic at `templates/task/sprint-bucket-heuristic.md` against the merged scope (NOT inherited). If recompute would alter buckets outside the merged component, flag `scope-change-required` with rationale `merge bucket-recompute creates graph-wide bucket drift — re-synthesis required`.
- **Split** — each child task re-runs the heuristic.
- **Audit** — pre-edit and post-edit bucket values for each affected task ID are recorded in `cross-task-responses.json` under the finding's `bucket_audit` field.

### F.5 Final disposition

Tell the user *"Cross-task feedback application complete. {N} findings accepted; {M} rejected; {K} routed to scope-change. Tasks.md updated; coverage matrix re-verified."* Phase F returns control to the parent Gate 3 orchestration (Phase E.5 / E.6 continues).

---

## What you MUST NOT do

- **Do not skip Phase A's chain check.** A PRD that didn't close Gate 2 is not a finished PRD. Tasking off it imports its problems into Phase 2.
- **Do not let orphan tasks ship.** Every task anchors to a PRD requirement. No exceptions.
- **Do not skip the override capture.** Every user redirect lands in `task-history.json`. Phase 3's self-learning depends on this corpus.
- **Do not bypass Gate 3.** A task list that hasn't been through Gate 3 is not a finished task list; downstream skills (`specflow:develop`, `specflow:test`) should reject it.
- **Do not edit PRD requirements from this skill.** If a Gate 3 finding indicates the PRD is wrong (not just the tasks), surface it as an escalation. PRD changes go through `specflow:scope-change`.
- **Do not write the intent summaries to a file.** They're chat-only by design.
- **Do not mention the underlying AI tooling or vendor** in any user-facing output, the tasks file, the debate manifest, or `task-history.json`. Per the project's CLAUDE.md, this is non-negotiable.

---

## Verify before declaring done

Before returning to the user with "tasks complete":

1. `features/NNN-{slug}/{NNN-slug}-tasks.md` exists with frontmatter, coverage matrix, and ≥1 task per PRD requirement.
2. Coverage matrix shows 100% forward coverage AND zero orphan tasks (reverse traceability holds).
3. Every task acceptance line passes the binary test.
4. `features/NNN-{slug}/debate-log/tasks-gate3/manifest.md` exists with closing decision entry signed by the Orchestrator.
5. Gate 3 status is recorded and surfaced to the user.
6. Any user-driven recut wrote a record to `admin/task-history.json` (verify by reading the latest entry).
7. Scratch directory `admin/scratch/{NNN-slug}-tasks/` retains `matched-lessons.json` (and `uncovered-lessons.json` if any) on failure; cleaned up on success.
8. Every lesson surfaced in A.4 either has a `lesson-anchor: L-NNN` field on the task that covers it, OR is recorded in `uncovered-lessons.json` with the user's accepted "proceed without task" decision.

If any verify step fails, fix it before returning.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 7 — coverage matrix + intent summary spec.
- `docs/PRD.md` Appendix N (especially N1 Gate 3 + N2 manifest spec).
- `docs/PRD.md` Appendix I3 — `task-history.json` schema.
- `templates/orchestrator-pattern.md` — fork / file handoff / command substitution conventions.
- `templates/agents/standard/lifecycle/orchestrator.md` — closer logic for Gate 3.
- `templates/agents/standard/lifecycle/devils-advocate.md` — parallel reviewer for Gate 3.
- `templates/agents/standard/principles/*.md` — principle-reviewer prompts; goal-driven-reviewer.md is load-bearing here.
- `skills/prd/SKILL.md` — sister skill, Gate 2 pattern; mirror for consistency.
