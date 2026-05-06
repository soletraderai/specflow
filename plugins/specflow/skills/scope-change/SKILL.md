---
name: scope-change
description: Capture mid-development scope changes with audit trail (why intent changed, what the PRD now needs to say, which tasks regenerate, what in-flight work is impacted). Auto-suggested by specflow:develop on detected drift; manually invoked via /specflow:scope-change. Updates the PRD, regenerates affected tasks, surfaces an impact list, appends a decision-log entry.
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-interview.md
  - docs/specflow/features/{NNN-slug}/debate-log/prd-gate2/manifest.md
produces:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-interview.md (extended; new "Scope change" section appended)
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md (surgically updated; sibling .bak preserved)
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-brief.html (re-composed)
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md (delta-regenerated; sibling .bak preserved)
  - docs/specflow/features/{NNN-slug}/debate-log/prd-gate2/manifest-scope-change-{SC-NNN}.md
  - docs/specflow/features/{NNN-slug}/debate-log/tasks-gate3/manifest-scope-change-{SC-NNN}.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-scope-change-{date}-impact.md
  - docs/specflow/admin/decision-log.md (appended SC-{NNN} entry)
  - docs/specflow/admin/scratch/{NNN-slug}-scope-change/ (per-step status files; cleaned on success)
eval: |
  PRD .bak preserved; interview extended append-only with strikethrough-marked supersessions on prior Resolved lines;
  manifest-scope-change-{SC-NNN}.md exists at debate-log/prd-gate2/ AND debate-log/tasks-gate3/ with closing decision
  passed/passed-with-revisions/passed-with-escalations; tasks.md diff scoped to changed-R-ID set with prior
  task-history.json entries retained via superseded_by; impact list cites every artefact across four sources bounded
  to changed-R-IDs; decision-log entry SC-{NNN} written via specflow:decision; references the impact list path.
---

# scope-change

You are the orchestrator for mid-development scope changes. You own the boundary between "the user realised the PRD is wrong mid-`specflow:develop`" and "the PRD/tasks/decision-log triad is back in sync, the audit trail is preserved, and in-flight work has a concrete impact list to triage." Without this skill, mid-development drift goes silent — the PRD stales, tasks rot against intent, in-flight work gets implicitly invalidated, and the *why* of the change evaporates.

This is an **8-phase orchestrator** (A → B → C → D → E → F → G → H) mapping one-to-one onto the eight steps R11 names: pre-flight + drift articulation routing (A, B), append-only interview extension via `/grill` extend-mode (C step i), surgical PRD synthesis + Gate 2 re-fire (D step ii + iii), delta task regeneration + Gate 3 re-fire (E step iv + v), in-flight impact list across four sources (F step vi), `specflow:decision` invocation (G step vii), final disposition + scratch cleanup (H step viii). Every sub-skill invocation runs in a forked sub-agent context per `templates/orchestrator-pattern.md`; target parent-context budget is ≤25K tokens for a feature with 8 R-IDs and 12 tasks where 2 R-IDs change (AC-13).

The four core principles bind here as everywhere; **Surgical Changes is load-bearing** — this skill modifies user-edited files that already passed Gate 2 / Gate 3. Backups are non-negotiable, append-only interview edits are non-negotiable, diffs scoped to the changed-R-ID set are non-negotiable, prior `task-history.json` entries are retained (never deleted) per Phase 3 self-learning preservation. **Think before coding** — every drift trigger names the *why* before any file write. **Simplicity first** — v1 hard-codes Backlog exclusion (Gate 2 simplicity-r1-f1; no `--include-backlog` flag), forward-only (no native rollback per Non-goals), one feature per invocation. **Goal-driven execution** — every phase has an inline binary verify step; every refusal exits with a sentinel chat line; the decision-log entry only writes after step (vii) so partial scope changes never corrupt the audit trail (R11 + AC-11).

---

## Inputs

The skill is invoked via one of (per R2 + AC-2):

- **Manual:** `/specflow:scope-change {NNN-slug}` — user-elected.
- **Auto-handoff from `specflow:develop` Phase F.3** (Verifier rejection option 3) — y/n prompt *"Run `specflow:scope-change {NNN-slug}` now? (y/n)"*; payload sets `task-history.json.status = aborted_for_scope_change` for the rejected task before this skill fires (Gate 2 da-r1-f1 lifecycle handoff).
- **Auto-handoff from `specflow:develop` Phase A** (resume-prompt option 3) — same y/n prompt.
- **Auto-suggestion from `specflow:develop` Phase B.5** — informational note `[scope-change suggestion: lane distribution {G/Y/R} suggests the PRD may need re-cutting; run specflow:scope-change {NNN-slug} when ready]`. Informational only; no y/n; no auto-invocation. Confirmed against `skills/develop/SKILL.md:171`.

Gate 4 reviewer-driven re-lane (R5/R5.1) and `specflow:misc --auto` rule-violation flags do NOT count as drift triggers — those are within-skill recoveries.

Tell the user explicitly which mode you detected: *"Scope change for `{NNN-slug}` — mode: `{manual | auto-handoff-develop-f | auto-handoff-develop-a | auto-suggestion-lane-distribution}`. Starting Phase A."*

---

## Phase A — Pre-flight + invocation routing (R1)

### A.1 Resolve the feature folder + read the artefact chain

Use Read tool in parallel on:
- `features/{NNN-slug}/{NNN-slug}-prd.md`
- `features/{NNN-slug}/{NNN-slug}-tasks.md`
- `features/{NNN-slug}/{NNN-slug}-interview.md`
- `features/{NNN-slug}/debate-log/prd-gate2/manifest.md`
- `admin/environment.json`
- `admin/decision-log.md` (to compute the next `SC-{NNN}` id at G.1)

If `{NNN-slug}` was not provided, ask: *"Provide a feature id (`/specflow:scope-change NNN-slug`)."*. Empty input re-prompts; no auto-default.

### A.2 Refuse on missing chain or unclosed Gate 2 (R1, AC-1)

Apply the literal sentinel rules in this order:

- If the feature folder does not exist OR the PRD is missing OR the tasks file is missing OR the interview is missing → refuse with: *"Feature `{NNN-slug}` does not exist. Run `specflow:prd` first to create it; `specflow:scope-change` is for revising an existing PRD, not creating one."* Exit. No fall-through to `specflow:prd` (different intent — new vs revise).
- If the Gate 2 manifest's closing decision is `failed` OR missing → refuse with: *"PRD has not closed Gate 2 (status: `{status|missing}`). Resolve Gate 2 first via `specflow:prd {NNN-slug}` before running scope-change — scope-change preserves Gate 2 closure as a pre-condition for the surgical-edit pattern."* Exit.
- If closing decision is `passed`, `passed-with-revisions`, or `passed-with-escalations` → proceed.

### A.3 Detect the scratch directory (resume vs fresh)

Read `admin/scratch/{NNN-slug}-scope-change/` (Glob). Determine resume point per R11:
- **No scratch directory** → fresh run. Create the directory; allocate the next `SC-{NNN}` id by scanning `admin/decision-log.md` for the highest existing `SC-` prefix and incrementing.
- **Scratch directory exists** → read each `step-{N}-status.json` (states: `pending | in_progress | done | failed`). Resume at the first step whose state is NOT `done`. Steps with state `done` are NOT re-run. Steps with state `failed` are re-run from the same step (after the user resolves the underlying error). Tell the user: *"Resuming scope-change for `{NNN-slug}` (SC-{NNN}) at step ({N}) `{step-name}` — prior state `{state}`."*

### A.4 Detect `specflow:develop` lock + soft dependencies

Read `admin/scratch/{NNN-slug}-develop/` for an active `specflow:develop` orchestration lock. If present, surface to the user: *"`specflow:develop` is in flight for `{NNN-slug}` (lock acquired `{timestamp}`). Scope-change will surface this in the impact list (Phase F) but will not auto-resume develop."* — feeds Phase F's impact list.

Read `admin/environment.json` for soft dependencies referenced downstream:
- `mcp.linear.available` — Phase F source (b); Phase H optional Linear status updates.
- `cli.gh.available` — Phase F source (c) primary; falls back to `git branch --list` when absent.
- `cli.codex.available` — Phase D step (iii) Gate 2 re-fire reviewer slot.

Record the detected surface to `admin/scratch/{NNN-slug}-scope-change/environment-snapshot.json`.

### A.5 Verify before continuing

- Feature chain readable; Gate 2 closing decision in {`passed`, `passed-with-revisions`, `passed-with-escalations`}.
- `SC-{NNN}` id allocated; scratch directory exists.
- Mode + resume-step communicated to the user.

Hand off to Phase B.

---

## Phase B — Drift articulation (R2 trigger payload + new "why")

### B.1 Capture the *why* (interactive Q&A)

The *why* is the most important field — it justifies the change to future readers and lands verbatim in the decision-log entry's `rationale` (R10). Walk the user through four short questions:

1. *"What changed? In one short paragraph: the why behind this scope change."* — non-empty; refuse empty input with re-prompt.
2. *"Which PRD R-IDs are now wrong, missing, or superseded? List them (e.g. R3, R7)."* — non-empty list of valid R-IDs from the PRD; reject IDs that don't resolve.
3. *"What's the new scope? In one short paragraph: what the PRD should now say."* — non-empty; refuse empty input.
4. *"When did the change become apparent? Date or event (e.g. `2026-05-06`, `during T4 Verifier rejection`)."* — non-empty.

For auto-handoff modes, pre-populate from the structured payload (`specflow:develop` Phase F.3 carries the failed AC + verification check finding; Phase A resume prompt carries the user-stated reason; lane-distribution carries the G/Y/R ratio). Surface the synthesised draft with one prompt: *"Auto-handoff draft for the *why* — confirm to proceed, edit any field, or cancel."* Confirm proceeds; edit re-renders; cancel exits without scratch state changes.

### B.2 Persist the drift articulation

Write `admin/scratch/{NNN-slug}-scope-change/drift-articulation.json` capturing `sc_id`, `captured_at`, `mode`, `why` (verbatim), `claimed_affected_r_ids`, `new_scope`, `change_apparent_at`. The `claimed_affected_r_ids` is the user's claim; the **canonical** changed-R-ID set is computed in Phase E after PRD synthesis by diffing pre/post-PRD R-IDs. The two may diverge; the canonical set wins.

### B.3 Verify before continuing

- All four fields non-empty; each `claimed_affected_r_ids` entry resolves to a real PRD requirement.
- `drift-articulation.json` written; `step-1-status.json` set to `in_progress`.

Hand off to Phase C.

---

## Phase C — Step (i): Append-only interview extension via `/grill` extend-mode (R3, R4)

### C.1 Snapshot the pre-change interview

Copy the interview verbatim to `admin/scratch/{NNN-slug}-scope-change/interview-pre-change.md`. This snapshot powers the AC-3 byte-identity verification at C.4.

### C.2 Invoke `/grill` in extend-mode (forked sub-agent)

Fork a `/grill` sub-agent per the orchestrator pattern. Pass via command substitution:
- The interview file path.
- The drift articulation JSON (`!{cat admin/scratch/{NNN-slug}-scope-change/drift-articulation.json}`).
- The PRD R-ID list (so `/grill` can target the new rounds at the claimed affected R-IDs).

`/grill` extend-mode (per its SKILL.md "manual `/grill {feature-id}` to extend an existing interview file with new rounds" trigger) appends a new top-level `## Scope change — {YYYY-MM-DD}` section after `## Topics not discussed`. The new section captures:

- A `*why*` preamble paragraph (verbatim from `drift-articulation.json.why`).
- Typically 1-3 new rounds (narrower than a full PRD interview), each with Q + AI's recommended answer + user's answer + Resolved line.
- A fresh sign-off line dated to today.
- An explicit reference to the resulting decision-log entry id: *"See `admin/decision-log.md` entry SC-{NNN}"*.

The original Goal section, Original Request, Codebase context, and prior Rounds are NEVER edited — append-only.

### C.3 Mark superseded prior Resolved lines (R4)

For each prior round's Resolved line that this scope change explicitly invalidates (per the new round's content), apply the `~~text~~` strikethrough markup inline AND append the parenthetical note `(superseded by Scope change {YYYY-MM-DD})`. The strikethrough wraps only the affected text, not the whole line. The line is NEVER deleted — strikethrough preserves the audit trail.

The strikethrough markup paired with the supersession note is the **canonical machine-readable signal** (per Gate 2 tbc-r1-f1 sharpen): downstream skills (`/insights` Phase 3 cross-feature queries) parse this surface to enumerate supersession history. Orphan strikethrough runs without matching supersession notes are a failed step (i).

Capture the superseded-line set in `admin/scratch/{NNN-slug}-scope-change/superseded-resolved-lines.json` using **round + anchor-text** shape (drift-resistant per Gate 2 tbc-r1-f1; line-numbers drift if the strikethrough adds characters mid-line) — each entry: `{ "round": N, "anchor_text": "..." }`.

### C.4 Verify before continuing (AC-3, AC-4)

- The new `## Scope change — {YYYY-MM-DD}` section exists with `*why*` preamble + ≥1 round + sign-off + decision-log id reference.
- Diff the post-extension interview against `interview-pre-change.md`: pre-existing sections (Original Request, Codebase context, Goal, prior Rounds, Topics not discussed) are byte-identical EXCEPT for strikethrough+note insertions on prior Resolved lines.
- Every strikethrough run has a matching `(superseded by Scope change {YYYY-MM-DD})` note. Orphan strikethroughs fail this step.
- `superseded-resolved-lines.json` written.
- `step-1-status.json` set to `done`.

Hand off to Phase D.

---

## Phase D — Step (ii)+(iii): Surgical PRD synthesis + Gate 2 re-fire (R5, R6)

### D.1 Back up the PRD

Before any write, copy `features/{NNN-slug}/{NNN-slug}-prd.md` to `features/{NNN-slug}/{NNN-slug}-prd.md.bak`. The `.bak` is the user's reviewable diff anchor and the rollback anchor on partial-state failure.

### D.2 Re-fire `specflow:prd` Phase C synthesis with surgical-changes constraint (R5, step ii)

Fork a `specflow:prd` Phase C sub-agent (synthesis only; not the full skill). Pass via command substitution:
- The extended interview (post-Phase-C).
- The pre-change PRD (`!{cat features/{NNN-slug}/{NNN-slug}-prd.md.bak}`).
- The `superseded-resolved-lines.json` set.

Synthesis constraint (load-bearing per Surgical Changes principle): only requirements/ACs whose Trace lines reference the new scope-change rounds (or whose original Trace line is now strikethrough-superseded) are modified. Existing R/ACs whose Trace lines reference unchanged original rounds are byte-stable in the diff. Vision, Problem, Goals, Non-goals, Users, Open questions, See-also sections are byte-stable unless the scope change explicitly mandates a change (rare; surfaced in the new round's Resolved line if so).

After synthesis lands, append a `last_scope_change: SC-{NNN}` line to the PRD frontmatter (downstream skills detect the most recent revision via this field). The PRD's frontmatter `status` field stays `draft`.

### D.3 Re-fire `specflow:brief` to refresh the brief (R5)

Invoke `specflow:brief {NNN-slug}` as a forked sub-skill **after** D.4 completes (Gate 2 re-fire) so the new manifest is included in the brief composition. The sibling `{NNN-slug}-brief.html` regenerates; byte-stable except for regions corresponding to the changed R/ACs, the new frontmatter line, and the appended Gate 2 manifest entry.

### D.4 Re-fire `specflow:prd` Phase D Gate 2 manifest (R6, step iii)

Fork a Gate 2 multi-agent debate sub-orchestration per `specflow:prd` Phase E. The standard five reviewers fire (Devil's Advocate, Simplicity, Surgical, Think-Before-Coding, Goal-Driven); Codex joins as a sixth when `cli.codex.available: true`.

Write the new manifest to `features/{NNN-slug}/debate-log/prd-gate2/manifest-scope-change-{SC-NNN}.md`. The original `manifest.md` is NEVER modified — append-only across scope-change events.

If the new manifest's closing decision is `failed`: halt the flow with the same surface `specflow:prd` Phase D.7 uses (surface blocking findings; refuse to proceed to Phase E). Mark `step-3-status.json` as `failed` with the manifest path; the user resolves the findings (manual edit or another scope-change cycle) and re-invokes — this skill resumes at step (iii) on re-invocation.

### D.5 Verify before continuing (AC-5, AC-6)

- `features/{NNN-slug}/{NNN-slug}-prd.md.bak` exists; PRD diff shows changes ONLY to (a) R/ACs whose Trace line references a scope-change round OR a strikethrough'd prior Resolved line, (b) the frontmatter `last_scope_change: SC-{NNN}` line.
- R/ACs whose Trace line references unchanged original rounds are byte-identical to the `.bak`.
- `{NNN-slug}-brief.html` byte-stable except for regions corresponding to the changed R/ACs and the appended Gate 2 manifest entry.
- `manifest-scope-change-{SC-NNN}.md` exists at `debate-log/prd-gate2/`; original `manifest.md` is byte-identical to its pre-scope-change content.
- New manifest closing decision in {`passed`, `passed-with-revisions`, `passed-with-escalations`}.
- `step-2-status.json` and `step-3-status.json` both set to `done`.

Hand off to Phase E.

---

## Phase E — Step (iv)+(v): Delta task regeneration + Gate 3 re-fire (R7, R7.1)

### E.1 Compute the canonical changed-R-ID set

Diff the pre-change PRD (`.bak`) against the post-change PRD. Write `admin/scratch/{NNN-slug}-scope-change/changed-r-ids.json` with four arrays — `added`, `modified`, `deleted`, `superseded`. This canonical set replaces `drift-articulation.json.claimed_affected_r_ids` for downstream phases (diff wins).

### E.2 Locate affected tasks via the coverage matrix

For each changed R-ID, find tasks whose `Anchor: R{N}` field references it (read `tasks.md`'s coverage matrix). Capture the affected-tasks set in `affected-tasks.json`.

### E.3 Partial-completion handling (R7.1, AC-7 sub-clause)

For each affected task, read its `task-history.json` entry. If `status == "in_progress"`, the task is **NOT** auto-regenerated in step (iv) — instead it is added to Phase F's in-flight impact list with the recommended action *"abort current `specflow:develop` run and resume after scope-change completes"*. The user owns the triage decision (per Goal Out-of-scope-at-goal-level (3)).

Capture the deferred-tasks set in `deferred-in-flight-tasks.json`.

### E.4 Back up the tasks file

Copy `features/{NNN-slug}/{NNN-slug}-tasks.md` to `features/{NNN-slug}/{NNN-slug}-tasks.md.bak`.

### E.5 Re-fire `specflow:task` Phase B synthesis in delta mode (R7, step iv)

Fork a `specflow:task` Phase B sub-agent. Pass via command substitution:
- The post-change PRD.
- `changed-r-ids.json`.
- `affected-tasks.json` (minus the deferred set).
- The pre-change `tasks.md.bak` (for byte-stability check).

Phase B synthesis runs ONLY against the non-deferred affected tasks. Unchanged tasks are byte-stable. Each regenerated task gets a `regenerated_at: {timestamp}` + `superseded_by_scope_change: SC-{NNN}` + `prior_task_id: {original T-id}` triple in its `task-history.json` entry.

For each prior `task-history.json` entry whose task was regenerated: retain the entry (NOT delete) and add a `superseded_by: {new T-id}` field. The development-time fields (`lane_assigned`, `ai_assistance_level`, `elapsed_minutes`, `what_worked`, `what_didnt_work`, `blast_radius_outcome`) populated by prior `specflow:develop` runs are preserved verbatim — the Phase 3 self-learning corpus is not erased on scope change.

Update the coverage matrix in `tasks.md` to reflect the new R-ID set.

### E.6 Re-fire Gate 3 manifest against the delta only (R7, step v)

Fork a Gate 3 multi-agent debate sub-orchestration per `specflow:task` Phase E, scoped to the delta. The manifest's "Artefact under review" cites the delta (changed-R-ID set + regenerated tasks); the reviewer set is unchanged from a fresh Gate 3.

Write the new manifest to `features/{NNN-slug}/debate-log/tasks-gate3/manifest-scope-change-{SC-NNN}.md`. The original `manifest.md` is byte-stable.

If `failed`: halt; mark `step-5-status.json` as `failed`; the user resolves and re-invokes.

### E.7 Verify before continuing (AC-7)

- `tasks.md.bak` exists; diff shows changes ONLY to tasks whose Anchor R-ID is in the changed-R-ID set (or to coverage matrix entries for those R-IDs); deferred-in-flight tasks are byte-stable.
- Each regenerated task has `regenerated_at` + `superseded_by_scope_change` + `prior_task_id` in its `task-history.json` entry.
- Each prior `task-history.json` entry for a regenerated task retained with `superseded_by: {new T-id}` field.
- `manifest-scope-change-{SC-NNN}.md` exists at `debate-log/tasks-gate3/`; original `manifest.md` byte-identical.
- New manifest closing decision in {`passed`, `passed-with-revisions`, `passed-with-escalations`}.
- `step-4-status.json` and `step-5-status.json` both set to `done`.

Hand off to Phase F.

---

## Phase F — Step (vi): In-flight impact list (R8, R9)

This phase runs in the parent context (it's a small read-aggregate operation, not a sub-skill invocation per R12). Outputs are bounded; no fan-out leakage.

### F.1 Cross-reference four sources, scoped to the changed-R-ID set (R8, AC-8)

Compute the impact list across four sources. Each entry's R-ID anchor MUST be in `changed-r-ids.json`; artefacts outside the affected R-IDs are NOT listed.

- **(a) `task-history.json`** — every entry where `feature == {NNN-slug}` AND `status` ∈ {`planned`, `in_progress`, `review`} AND the entry's Anchor R-ID is in the changed-R-ID set. **Self-reference exclusion** (Gate 2 da-r1-f1): exclude entries whose `status == "aborted_for_scope_change"` AND whose abort was triggered by THIS scope-change invocation — that task surfaced this scope-change; surfacing it back would be circular.
- **(b) Linear** (when `mcp.linear.available: true`) — every issue where `feature == {NNN-slug}` AND `status` ∈ {`In Progress`, `In Review`} AND the issue's Anchor R-ID is in the changed-R-ID set. Backlog tickets are excluded; v1 ships with this hard-coded (per Gate 2 simplicity-r1-f1 — `--include-backlog` flag dropped). Custom intermediate statuses (`Blocked`, `On Hold`, etc.) are EXCLUDED with a chat-line note surfacing the skipped count (per Gate 2 goal-r1-f1 boundary clause). When MCP unavailable, this source contributes `(source unavailable: linear-mcp-not-configured)` and the chat-line `[scope-change: Linear MCP not detected — impact list source (b) skipped]` surfaces.
- **(c) Open PRs via `gh pr list --state open --label feature/{NNN-slug}`** — every PR labelled or branch-named to the feature whose touched files overlap with the changed R-IDs' scope-listed files. Draft PRs are INCLUDED with `(draft)` annotation (per Gate 2 goal-r1-f1 boundary clause). When `gh` CLI absent, falls back to `git branch --list 'feature/{NNN-slug}-*'` for branch presence with `(source unavailable: gh-cli-not-configured; branch-list fallback used)` annotation.
- **(d) Draft test plans** — `features/{NNN-slug}/{NNN-slug}-test.md` with `status: draft` frontmatter — surfaced as a single line if present.

### F.2 Write the impact list file (AC-8)

Write `features/{NNN-slug}/{NNN-slug}-scope-change-{date}-impact.md` with a header naming `SC-{NNN}` + the changed R-IDs, then four `##` sections corresponding to the four sources: `## task-history.json`, `## Linear`, `## Open PRs`, `## Draft test plans`. Each section either lists matching artefacts (one bullet per artefact, with the artefact's id, R-ID anchor, and current status) OR records `(source unavailable: {reason})`. Draft PRs are listed with `(draft)` annotation; custom-status Linear entries are summarised with a skipped-count line.

### F.3 Add the "What you should consider" section (R9, AC-9)

After the four source sections, write a `## What you should consider` section. For each artefact listed above, write a one-line recommended action (e.g. *"PR #45 (anchor R7) — files now removed from R7 scope; consider rebasing or closing"*; *"T4 in_progress (anchor R7 now superseded) — consider aborting current `specflow:develop` run and resuming after scope-change completes"*). Every artefact in the source sections has exactly one corresponding entry; an artefact without a recommendation is a failed step (vi). The recommendations leave the decision to the user — the skill never auto-resolves.

### F.4 Surface the impact list inline + verify

Surface the impact list inline in chat (full file content). Verify before continuing:
- The file exists at the expected path; four source sections present; "What you should consider" section present.
- Every artefact listed has a corresponding "What you should consider" entry.
- Every listed artefact's R-ID anchor is in `changed-r-ids.json`.
- Self-reference exclusion applied; chat-line notes surfaced for unavailable sources.
- `step-6-status.json` set to `done`.

Hand off to Phase G.

---

## Phase G — Step (vii): Decision-log entry via `specflow:decision` (R10, R10.1)

### G.1 Compose the decision-log payload

Build the payload using the standard `specflow:decision` schema (`title`, `context`, `decision`, `rationale`, `date`, `references`) PLUS the parameter `id_prefix: "SC"` (so the entry's id is `SC-{NNN}` — a separate counter from `specflow:decision`'s default, making scope-change entries filterable by id-prefix in `/insights` queries). The `references` block is the standard reference shape with three scope-change-specific keys EMBEDDED inside it (NOT as new top-level fields, to keep the schema stable for `/insights`):

- `affected_r_ids: ["R3", "R7", "R9"]` — the canonical changed-R-ID set from `changed-r-ids.json`.
- `superseded_resolved_lines: [{round, anchor_text}, ...]` — verbatim from `superseded-resolved-lines.json` (drift-resistant round+anchor-text shape per Gate 2 tbc-r1-f1).
- `impact_list_path: "features/{NNN-slug}/{NNN-slug}-scope-change-{date}-impact.md"`.

The `rationale` field carries the *why* verbatim from `drift-articulation.json.why` — the load-bearing field per Round 5's resolved decision. Title format: *"Scope change — {feature title} — {one-line summary}"*. Context = pre-scope-change state in 1-2 sentences; Decision = post-scope-change state in 1-2 sentences. Persist the composed payload to `admin/scratch/{NNN-slug}-scope-change/decision-payload.json`.

### G.2 Schema-dependency pre-check (R10.1, AC-10)

Before invoking, verify `specflow:decision` accepts (a) an arbitrary keyed-block within `references` AND (b) an `id_prefix: "SC"` parameter. Read `skills/decision/SKILL.md` for the current schema. If either is missing, refuse with the literal sentinel:

*"Cross-skill schema dependency unmet: `specflow:decision` does not accept `{missing-affordance}`. Ship a `specflow:decision` enhancement PRD before `specflow:scope-change` v1 lands. See PRD R10.1 schema-dependency clause."*

Mark `step-7-status.json` as `failed` with the dependency reason; the user resolves (typically by shipping the `specflow:decision` enhancement PRD first) and re-invokes.

### G.3 Invoke `specflow:decision` as a sub-skill

Fork a `specflow:decision` sub-agent; pass the payload via command substitution from `admin/scratch/{NNN-slug}-scope-change/decision-payload.json`. The sub-skill writes the entry to `admin/decision-log.md` per its own conventions; the parent context never touches `decision-log.md` directly (per Non-goal (1) — `specflow:scope-change` invokes the writer, never embeds it).

### G.4 Verify before continuing (AC-10, AC-11)

- New entry exists in `admin/decision-log.md` with `id: SC-{NNN}` and the standard schema fields.
- `rationale` contains the *why* verbatim from `drift-articulation.json.why`.
- `references` block contains `affected_r_ids`, `superseded_resolved_lines`, `impact_list_path`.
- Steps (i) through (vi) all have `state: done` in their status files (audit-trail integrity per AC-11 — partial scope changes never write to `decision-log.md`).
- `step-7-status.json` set to `done`.

Hand off to Phase H.

---

## Phase H — Step (viii): Final disposition + scratch cleanup (R12)

### H.1 Surface the impact list summary

Re-surface the impact list path to the user with the SC-{NNN} id and a one-paragraph summary (changed R-IDs, regenerated tasks count, in-flight artefacts count). The user already saw the full list at F.4; this is the closing reminder.

### H.2 Recommend `specflow:develop --resume` if applicable

If Phase A.4 detected a `specflow:develop` lock, surface: *"`specflow:develop` was in flight for `{NNN-slug}` when scope-change started. Once you've acknowledged the impact list, run `specflow:develop {NNN-slug}` to resume — it will re-detect the regenerated tasks and pick up at the appropriate phase."*

### H.3 Optional Linear status updates (best-effort)

If `mcp.linear.available: true` AND the impact list contains Linear issues: prompt the user *"Fire status updates on the {N} affected Linear issues? Each is confirmed individually (y/n per issue)."* For each `y`, fire the transition via `specflow:linear` (typically `In Progress → Backlog` for issues whose anchor R-ID was deleted/superseded; user-confirmed). For each `n`, skip. Best-effort; failures log a chat-line and continue.

If MCP unavailable, skip with the chat-line `[scope-change: Linear MCP not detected — Phase H status updates skipped]`.

### H.4 Cleanup scratch on success

After every prior phase reports `done` AND the user has acknowledged the impact list:

```bash
rm -rf admin/scratch/{NNN-slug}-scope-change/
```

The `.bak` files at `features/{NNN-slug}/{NNN-slug}-prd.md.bak` and `{NNN-slug}-tasks.md.bak` are RETAINED (the user's reviewable diff anchor; the user can clean them up after confirming the regeneration). Set `step-8-status.json` to `done` immediately before the directory removal so a re-invocation post-cleanup detects no scratch and starts fresh.

Retain the scratch directory on failure for debugging.

### H.5 Emit the closing chat-line summary

*"Scope change `SC-{NNN}` complete for `{NNN-slug}`. Affected R-IDs: {list}. Regenerated tasks: {count}. In-flight artefacts surfaced: {count}. Decision-log entry: `SC-{NNN}` — `{title}`. Impact list: `{impact-list-path}`."*

---

## Failure modes

Each maps to a documented user-elected response or a sentinel refusal exit; never silent retry.

- **Feature does not exist / PRD-tasks-interview missing (A.2)** — refuse with the literal sentinel; no fall-through to `specflow:prd`; no stub creation.
- **Gate 2 closing decision `failed` or missing (A.2)** — refuse with the unclosed-Gate-2 sentinel; no fall-through.
- **User aborts mid-Phase B** — remove the scratch directory; do NOT write to `decision-log.md`; exit cleanly. No `.bak` written yet so nothing to restore.
- **`/grill` extend-mode fails (C.2)** — surface the error; mark `step-1-status.json` `failed`. The user can edit the interview manually with the strikethrough convention and re-invoke (resume at step ii). Never auto-retry.
- **Gate 2 / Gate 3 re-fire returns `failed` (D.4 / E.6)** — halt; surface blocking findings; mark the relevant step `failed`; the user resolves and re-invokes.
- **`specflow:develop` lock detected (A.4)** — surface to the user; feed the in-flight task into Phase F's impact list. Force-proceed requires the `--force-on-stale-lock` flag with explicit confirmation; refusing the flag is the safer default.
- **`specflow:decision` schema gap (G.2)** — refuse with the schema-dependency sentinel; the user ships the `specflow:decision` enhancement PRD first.
- **Linear MCP unavailable (F.1, H.3) / `gh` CLI unavailable (F.1)** — sources record `(source unavailable: ...)`; chat-line notice surfaces degraded coverage. Best-effort; never blocks.

---

## Anti-patterns (refuse to do)

- **Modify the PRD without writing the `.bak`.** D.1 is non-negotiable.
- **Skip the decision-log entry.** Every scope change is an audit-trail-bearing event; a run that completes A-F without firing G is failed.
- **Auto-regenerate tasks without surfacing the diff.** Phase E's diff MUST be surfaced before Phase F proceeds.
- **Mark in-flight work `invalidated` without a documented reason.** Phase F's "What you should consider" requires a recommendation per artefact.
- **Re-grill the original interview's Goal section.** The Goal is precedent; only Resolved-line additions (new rounds + strikethrough on prior lines) are allowed.
- **Edit prior interview rounds in place.** Append-only. Strikethrough on Resolved lines is the ONLY permitted in-place edit.
- **Auto-resolve in-flight artefacts.** Closing PRs, transitioning tickets to invalid, force-pushing branches, aborting `specflow:develop` runs — out of scope per Non-goal (3).
- **Modify the original Gate 2 / Gate 3 `manifest.md`.** Append-only across scope-change events; new manifests go to `manifest-scope-change-{SC-NNN}.md`.
- **Delete prior `task-history.json` entries.** Retain with `superseded_by: {new T-id}`; the development-time fields are Phase 3 self-learning corpus.
- **Write `admin/decision-log.md` directly.** Always invoke `specflow:decision` as a sub-skill (Non-goal (1)).
- **Name any AI vendor or tooling in user-facing output.** Interview, PRD, impact list, decision-log, chat-line summaries stay vendor-neutral per the project attribution rule.

---

## Cross-skill integration

- **`specflow:develop`** Phase F.3 + Phase A + Phase B.5 emit the three drift-trigger chat lines; lifecycle handoff sets `task-history.json.status = aborted_for_scope_change` (Gate 2 da-r1-f1).
- **`specflow:prd`** Phase C synthesis re-fired with surgical constraint (D.2); Phase E Gate 2 re-fired (D.4).
- **`/grill`** extend-mode invoked in C.2 (append-only `## Scope change — {date}` section).
- **`specflow:brief`** invoked in D.3 to refresh the sibling `{NNN-slug}-brief.html`.
- **`specflow:task`** Phase B synthesis re-fired in delta mode (E.5); Phase E Gate 3 re-fired against the delta (E.6).
- **`specflow:decision`** invoked in G.3 with `id_prefix: "SC"` + arbitrary keyed-block within `references`. Cross-skill schema affordances are non-negotiable per R10.1.
- **`specflow:complete`** reads `superseded_by` chain in `task-history.json` to surface supersession history in retros.
- **`specflow:linear`** best-effort status updates in H.3 (user-confirmed per issue).
- **`/insights`** (Phase 3) reads `decision-log.md` for `SC-{NNN}` entries; parses `references.affected_r_ids` + `references.superseded_resolved_lines` for cross-feature pattern queries.

---

## Verify before declaring done

1. PRD `.bak` exists; PRD diff scoped to changed R-IDs + `last_scope_change: SC-{NNN}` frontmatter line; rest byte-stable.
2. Tasks `.bak` exists; tasks diff scoped to the changed-R-ID set; deferred in-flight tasks byte-stable.
3. Interview extended with `## Scope change — {YYYY-MM-DD}` section; pre-existing sections byte-identical except for strikethrough+supersession-note insertions; every strikethrough has a matching note.
4. `manifest-scope-change-{SC-NNN}.md` exists at both `debate-log/prd-gate2/` and `debate-log/tasks-gate3/`; original `manifest.md` files byte-identical.
5. Each regenerated task has `regenerated_at` + `superseded_by_scope_change` + `prior_task_id`; each prior `task-history.json` entry retained with `superseded_by`.
6. Impact list file exists with four source sections + "What you should consider"; every artefact has a recommendation; every listed artefact's R-ID is in the changed-R-ID set.
7. `decision-log.md` has new entry `id: SC-{NNN}`; `rationale` carries the *why* verbatim; `references` block contains `affected_r_ids` + `superseded_resolved_lines` + `impact_list_path`.
8. All eight `step-{N}-status.json` files set to `done`; scratch cleaned (or retained on failure).
9. `specflow:budget --skill scope-change {NNN-slug}` confirms `parent_context_tokens < 25000` AND `max_per_sub_skill_growth_tokens < 2000` (AC-13).

If any verify step fails, surface the failure and refuse to claim success.

---

## Reference

- `docs/specflow/features/005-scope-change-skill/{prd,interview}.md` and `debate-log/prd-gate2/manifest.md` — full R1-R12 + AC-1 to AC-13; Gate 2 closing decision.
- `templates/orchestrator-pattern.md` — three primitives load-bearing per R12.
- `CORE_PRINCIPLES.md` — Surgical Changes load-bearing here.
- `skills/develop/SKILL.md` Phase F.3 + A + B.5; `skills/prd/SKILL.md` Phase C + E; `skills/task/SKILL.md` Phase B + E; `skills/grill/SKILL.md` extend-mode; `skills/decision/SKILL.md` schema affordances; `skills/brief/SKILL.md`; `skills/linear/SKILL.md`; `skills/complete/SKILL.md`; `skills/budget/SKILL.md`.
