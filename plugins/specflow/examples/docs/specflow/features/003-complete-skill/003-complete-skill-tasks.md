---
feature: 003-complete-skill
status: draft
created: 2026-05-06
requires:
  - ./003-complete-skill-prd.md
  - ./003-complete-skill-interview.md
  - ./debate-log/prd-gate2/manifest.md
produces:
  - ../../admin/task-history.json
  - ../../admin/decision-log.md
  - ../../admin/scratch/complete-{task-id}.lock
eval: tasks file exists with one task per PRD requirement; coverage matrix shows 100% PRD-requirement coverage and zero orphan tasks; every task acceptance is binary; Gate 3 debate manifest closes with Orchestrator sign-off entry.
prd: ./003-complete-skill-prd.md
interview: ./003-complete-skill-interview.md
gate3: ./debate-log/tasks-gate3/manifest.md
---

# Tasks — specflow:complete (retro skill closing the work-done → project-memory loop)

## Coverage matrix

| PRD requirement | Tasks satisfying it |
|---|---|
| R1 — Auto-fire from `specflow:develop` Phase F (per-task; F.5 → F.6 boundary) | T1 |
| R2 — Manual `/specflow:complete {task-id}` escape-hatch path | T2 |
| R3 — Three retro-only fields (`actual_hours`, `regressions_caught`, `escaped_issues`) | T3 |
| R4 — Retro supersedes Phase F placeholders + cross-skill `superseded_by_retro` flag | T4 |
| R5 — Snake_case schema; Schema Appendix is the v1 contract | T5 |
| R6 — Two-condition decision-log elevation rule (user-flag OR escaped-issue medium+) | T6 |
| R7 — User reviews composed elevation entry before write (confirm/edit/cancel) | T7 |
| R8 — Zero webhook integration in v1 (two trigger paths only) | T8 |
| R9 — Plain-mode refusal on tasks with existing retros | T9 |
| R10 — `--amend` mode appends to `addenda` array; preserves original entry | T10 |
| R11 — Accept-and-ship retros fire with forced `accepted_with_failure: true` | T11 |
| R12 — Forced elevation prompt + Phase F.3 decision-log linkage | T12 |
| R13 — Scope-changed tasks record final definition + `scope_change_log` | T13 |
| R14 — Concurrent-trigger lock-file guard at `admin/scratch/complete-{task-id}.lock` | T14 |
| AC-15 / Goal Outcome surfaces (c)+(d) — anchored to goal Outcome (c) chat-line summary citing the new entry's id + (d) zero silent failures, not to an R; documented exception per Gate 3 finding goal-r1-f1 | T15 |

Forward coverage: 14/14 PRD requirements covered (R1-R14) plus AC-15 surfaced as T15. Reverse traceability: all 15 tasks anchor to ≥1 requirement or stated PRD acceptance.

## Tasks

### T1 — Auto-fire from `specflow:develop` Phase F at the F.5 → F.6 boundary

- **Anchor:** PRD R1 — *invoke once per task at the boundary between Phase F.5 task-history.json write and Phase F.6 scratch cleanup, AFTER Verifier passes (or option-4 elected) AND AFTER the per-task PR opens; Green-lane batched flow fires per-task, never per-batch.*
- **Scope:** `skills/complete/SKILL.md` Phase A trigger-detection step; `skills/develop/SKILL.md` Phase F.5 → F.6 boundary hook (invoke `specflow:complete` per-task before scratch-cleanup); `triggered_by` field writer on the retro entry.
- **Acceptance:** When `specflow:develop` Phase F passes Verifier (or option-4 accept-and-ship is elected) AND the per-task PR opens, `specflow:complete` is invoked exactly once per task before Phase F.6 scratch-cleanup runs. The retro entry written under `admin/task-history.json` has `id` matching the Phase F task-id (literal string `{NNN-slug}-T{N}`) AND `triggered_by` set to the literal string `specflow:develop-phase-f`. On a Green-lane batch of 3 tasks, the run produces exactly 3 retro entries (one per task-id) AND zero entries with `id` ending `-batch` or any aggregated marker. A Phase F run that completes without firing `specflow:complete` for at least one of its closed tasks, or that fires once per batch instead of per-task, is a failed run. (Verifies AC-1.)
- **Depends on:** none.
- **Notes:** The hook in `specflow:develop` Phase F.5 → F.6 is the auto-fire surface; the same `specflow:complete` SKILL.md handles both auto-fire and manual paths via `triggered_by` differentiation (T2). The per-task firing is non-negotiable — batched firing would collapse three distinct retros into one.

### T2 — Manual `/specflow:complete {task-id}` escape-hatch path

- **Anchor:** PRD R2 — *manual CLI invocation `/specflow:complete {task-id}` is the alternate path for tasks closed outside `specflow:develop` (red-lane human-led tasks, misc-tasks); same retro flow as auto-fire; differs only in trigger.*
- **Scope:** `skills/complete/SKILL.md` argument parser (CLI form `/specflow:complete {task-id}` with `--amend`, `--escaped-issue`, `--escaped-blast-radius`, `--note` flags); `triggered_by: "manual-cli"` writer; fresh-entry creator (when `task-id` is not present in `admin/task-history.json`).
- **Acceptance:** Running `/specflow:complete {task-id}` from the CLI on a task with no existing retro entry runs the same retro question flow as auto-fire AND writes the entry with `triggered_by` set to the literal string `manual-cli`. Running `/specflow:complete {task-id}` for a `task-id` not present in `admin/task-history.json` (i.e. a task not previously created by `specflow:develop` or `specflow:task` Phase D) produces a fresh entry with all required schema fields (per T3, T5) populated AND `triggered_by: "manual-cli"`. Invoking with no `task-id` argument prompts for one and refuses to proceed without an explicit value; an empty input re-prompts. (Verifies AC-2.)
- **Depends on:** T1, T3.
- **Notes:** The manual path deliberately accepts task-ids absent from `task-history.json` to support red-lane human-led tasks and misc-tasks closed outside `specflow:develop`. The retro question set is identical between paths; only `triggered_by` differs.

### T3 — Three retro-only fields with non-null defaults

- **Anchor:** PRD R3 — *retro adds three new fields beyond Phase F's development-time set: `actual_hours` (user-reported reflective hours, distinct from `elapsed_minutes`), `regressions_caught` (count + descriptions, caught before ship), `escaped_issues` (count + descriptions + blast-radii, found post-ship; default zero with AMEND as primary capture surface).*
- **Scope:** `skills/complete/SKILL.md` Phase B retro-question prompter; field-shape validator (`actual_hours: number`, `regressions_caught: {count, descriptions[]}`, `escaped_issues: {count, descriptions[], blast_radii[]}`); retro-time default writer for `escaped_issues` (`{count: 0, descriptions: [], blast_radii: []}`).
- **Acceptance:** Every retro entry written by the skill includes the three retro-only fields with non-null values: (a) `actual_hours: number` (the user-reported reflective hours; if the user declines to estimate, the field is exactly `0` per the convention `0 = declined`), (b) `regressions_caught: {count: number, descriptions: string[]}` where `count` equals `descriptions.length`, (c) `escaped_issues: {count: number, descriptions: string[], blast_radii: ("low"|"medium"|"high")[]}` where `count` equals both `descriptions.length` AND `blast_radii.length`. At retro time the default for `escaped_issues` is exactly `{count: 0, descriptions: [], blast_radii: []}` — the AMEND path (T10) is the primary capture surface for post-ship escaped issues. An entry with any of the three fields set to `null`, missing the field entirely, or whose `count` does not match the array length(s) is a failed write. (Verifies AC-3.)
- **Depends on:** T1.
- **Notes:** The retro-time `escaped_issues = 0` default plus the AMEND-as-primary-capture-surface convention is the load-bearing assumption surfaced at Gate 2 (block tbc-r1-f1) — downstream consumers should treat retro-time `0` as "unknown" until the entry has an `addenda` array entry confirming zero. The `actual_hours = 0` declined-to-estimate sentinel is documented in the PRD's Open Questions; treat any positive number (including fractional) as captured value.

### T4 — Retro supersedes Phase F placeholders for `what_worked`, `what_didnt_work`, `ai_assistance_level`

- **Anchor:** PRD R4 — *retro write SUPERSEDES the Phase F write's three placeholder values; the supersede is a write-in-place flagged by `superseded_by_retro: true`; cross-skill schema dependency on `specflow:develop` Phase F.5 emitting `superseded_by_retro: false` as the default flag in its initial write.*
- **Scope:** `skills/complete/SKILL.md` Phase B supersede-writer (write-in-place on the existing entry's three fields + flip `superseded_by_retro` from `false` to `true`); cross-skill enhancement-PRD prerequisite check (runtime detection of the absent flag).
- **Acceptance:** When the retro fires on a task whose `task-history.json` entry already has Phase F.5's `what_worked`, `what_didnt_work`, `ai_assistance_level` populated, the retro write replaces those three field values in place AND sets `superseded_by_retro` from `false` to `true` on the same entry. The original Phase F values are NOT preserved (no `phase_f_*` mirror fields in v1). A diff of `task-history.json` before vs after the retro shows exactly four fields changed for the entry: `what_worked`, `what_didnt_work`, `ai_assistance_level`, `superseded_by_retro` (plus any new retro-only fields per T3). **Schema dependency:** if `skills/develop/SKILL.md` Phase F.5's initial-write does not include `superseded_by_retro: false` as a default field at the time `specflow:complete` v1 ships, the cross-skill enhancement PRD MUST land first; a `specflow:complete` run that finds the field absent from the existing entry refuses with the literal sentinel *"Cross-skill schema dependency unmet: `specflow:develop` Phase F.5 must emit `superseded_by_retro: false` as default. See PRD R4 schema-dependency clause."* and writes zero retro fields. (Verifies AC-4.)
- **Depends on:** T1.
- **Notes:** The cross-skill schema dependency is named at Gate 2 (concern surgical-r1-f1) — explicit cross-skill dependency call-out at both R-level and AC-level mirrors the 002-develop-skill AC-10 precedent. The replace-in-place choice trades the `/insights`-cadence Phase-F-vs-retro divergence-signal for entry-shape simplicity per Gate 2 (concern tbc-r1-f2); v2 may revisit with `phase_f_*` mirror fields. The cross-skill prerequisite (adding `superseded_by_retro: false` as default in `specflow:develop` Phase F.5) is out of scope for this task and out of scope for `specflow:complete` v1; ship it via a separate `specflow:develop` enhancement PRD per PRD R4. T4's runtime-branch refusal is the surgical-changes-compliant handling of the absent-field case in the meantime.
- **gate2-revision:** surgical-r1-f1
- **gate3-revision:** surgical-r1-f1

### T5 — Snake_case schema with Schema Appendix as the v1 contract

- **Anchor:** PRD R5 — *retro fields use snake_case matching `specflow:develop` Phase F.5's working convention; Appendix I3's camelCase example is illustrative-not-normative; the PRD's Schema Appendix is the v1 contract.*
- **Scope:** `skills/complete/SKILL.md` Phase B schema validator; field-name checker (snake_case enforcement); allow-list of fields from the PRD Schema Appendix; reject-on-extraneous-field path.
- **Acceptance:** Every retro entry's field names match snake_case (regex `^[a-z][a-z0-9_]*$` for every key at the top level AND nested within `regressions_caught`, `escaped_issues`, `addenda[]`, `decision_log_links[]`, `scope_change_log[]`). No camelCase fields appear in any retro write — an entry containing a key matching the regex `^[a-z]+[A-Z]` is a failed write. The set of permitted top-level fields is exactly the field list named in the PRD's Schema Appendix (`id`, `feature`, `task_id`, `title`, `scope`, `completed_at`, `triggered_by`, `lane_assigned`, `ai_assistance_level`, `elapsed_minutes`, `what_worked`, `what_didnt_work`, `blast_radius_outcome`, `superseded_by_retro`, `actual_hours`, `regressions_caught`, `escaped_issues`, `accepted_with_failure`, `elevation_offered`, `elevation_fired_by`, `elevation_outcome`, `decision_log_links`, `scope_change_log`, `addenda`, `prd_anchor`); an entry containing a top-level field outside this list is a failed write — the entry is rejected and the user is asked to revise. (Verifies AC-5.)
- **Depends on:** T3.
- **Notes:** The reconciliation note in PRD R5 acknowledges Appendix I3's camelCase example is illustrative; the working convention in `specflow:develop`'s shipped schema is snake_case and that carries forward here. Future schema additions land via PRD revision; the allow-list refusal is the schema-stability surface.

### T6 — Two-condition decision-log elevation rule (OR; firing condition recorded)

- **Anchor:** PRD R6 — *retro elevates to `decision-log.md` when EITHER (a) user explicitly flags via the retro question, OR (b) `escaped_issues` field captures a post-ship issue with `blast_radius` medium or high; conditions OR'd; lane-vs-outcome divergence is `/insights`-cadence territory, not per-task elevation.*
- **Scope:** `skills/complete/SKILL.md` Phase C elevation-condition evaluator (reads two inputs: user retro-question answer + `escaped_issues.blast_radii`); `elevation_fired_by` writer (one of the literal strings `user-flag`, `escaped-issue-medium-or-high`, `none`); short-circuit when neither condition fires (zero `decision-log.md` writes).
- **Acceptance:** The elevation logic fires when EITHER (a) the user answers `Y` to the retro question *"Should this land in `decision-log.md`?"* AND provides a non-empty description, OR (b) the retro entry's `escaped_issues` field has at least one entry whose `blast_radius` is `medium` or `high`. The retro entry's `elevation_fired_by` field is set to the literal string `user-flag` when only condition (a) fires, `escaped-issue-medium-or-high` when only condition (b) fires (or both — condition (b) takes precedence in the recorded value when both fire), and `none` when neither fires. When `elevation_fired_by` is `none`, no `decision-log.md` write happens — a `decision-log.md` diff shows zero new entries for the retro run. Lane-vs-outcome divergence (e.g. `lane_assigned: green` paired with `blast_radius_outcome` indicating high) is NOT a firing condition; the divergence signal lives in the schema appendix's `lane_assigned` + `blast_radius_outcome` fields for `/insights`-cadence pattern detection. (Verifies AC-6.)
- **Depends on:** T3, T5.
- **Notes:** AC-6's verification surface was scoped at Gate 2 (block goal-r1-f1) — the corpus-signal capture lives in the schema appendix as the bindable surface, not in this AC. The OR semantics + precedence on the recorded value (escaped-issue wins when both fire) is the deterministic shape `/insights` consumers read.

### T7 — User reviews composed elevation entry before write (confirm / edit / cancel)

- **Anchor:** PRD R7 — *when elevation conditions fire, the skill composes the `decision-log.md` entry from retro fields and surfaces it to the user with confirm-or-edit-or-cancel; on cancel, the elevation is dropped and `elevation_outcome: cancelled` is recorded for `/insights`-cadence pattern detection.*
- **Scope:** `skills/complete/SKILL.md` Phase C elevation-entry composer (Title/Context/Decision/Rationale/Related fields per the sister-skill mirror pattern); user-prompt presenter (three options labelled `confirm-and-write`, `edit-and-write`, `cancel`); `decision-log.md` writer (delegate to `specflow:decision`'s write-helper); retro-entry flag writers (`elevation_offered`, `elevation_outcome`, `decision_log_links`).
- **Acceptance:** When elevation fires (per T6), the skill composes a candidate `decision-log.md` entry with exactly five fields: `Title: Retro: {task title}`; `Context: {task's PRD anchor}`; `Decision: "{user-stated lesson} — pattern surfaced via specflow:complete"`; `Rationale: {the retro field that fired the elevation, quoted verbatim}`; `Related: {task-history entry id} + {PRD path}`. The skill surfaces the candidate to the user with exactly three options labelled `1` (confirm-and-write), `2` (edit-and-write), `3` (cancel). On `1`: the entry is appended to `decision-log.md` AND the retro entry's `decision_log_links` field captures the new entry's `{date, title}`; `elevation_outcome: written`. On `2`: the skill accepts an edited five-field version, re-validates the field shape, writes; `elevation_outcome: written` AND `decision_log_links` captures the new entry. On `3`: no `decision-log.md` write happens; the retro entry's `elevation_offered: true` AND `elevation_outcome: cancelled` are set. The skill must not proceed without an explicit user choice; auto-default is a failed run. (Verifies AC-7.)
- **Depends on:** T6.
- **Notes:** Per Gate 2 (concern goal-r1-f2 push-back defence), the `Date` field on the composed entry is auto-populated by `specflow:decision`'s write-helper at write-time, not composed by `specflow:complete` from retro state — this AC verifies five composed fields, not six. The cancellation flag (`elevation_outcome: cancelled`) is the `/insights`-cadence pattern signal R7's last sentence names; the triple-flag (`elevation_offered`, `elevation_fired_by`, `elevation_outcome`) carries distinct semantic content per Gate 2 (concern simplicity-r1-f1 push-back defence).

### T8 — Zero webhook integration in v1; two trigger paths only

- **Anchor:** PRD R8 — *v1 ships zero webhook plumbing — no polling, no listener, no `config.json.complete.webhook*` knobs; trigger paths are exactly R1's auto-fire and R2's manual CLI; v2 enhancement PRD lands webhook polling when a documented consumer surfaces.*
- **Scope:** `skills/complete/SKILL.md` SKILL.md frontmatter (`requires:` and `produces:` lists); webhook-knob refusal (any read of `config.json.complete.webhook*` returns undefined; the keys are not documented in the SKILL.md schema); HTTP-listener absence.
- **Acceptance:** No webhook code paths exist in `skills/complete/SKILL.md`. The SKILL.md frontmatter `requires:` and `produces:` field lists do not reference any webhook config or listener path (a grep for `webhook` against the SKILL.md returns zero matches). No `config.json.complete.webhook*` keys are read or written by the skill (a grep for `config.json.complete.webhook` against the skill's source returns zero matches). No HTTP listener is started by the skill (no `http.createServer`, no `listen(`, no equivalent network-bind call appears in the skill's source). Trigger paths are exactly two: auto-fire from `specflow:develop` Phase F (per T1) and manual CLI invocation (per T2). A SKILL.md or source file containing any of these webhook surfaces is a failed run. (Verifies AC-8.)
- **Depends on:** T1, T2.
- **Notes:** Per `docs/PRD.md` Appendix I8 (Manual first); webhook plumbing lands in v2 if a documented consumer surfaces (e.g. tasks closed in Linear UI without going through `specflow:develop`). The Simplicity First sub-clause (no speculative configurability) bites hardest on this requirement — adding a webhook knob in v1 would defeat the purpose.

### T9 — Plain-mode refusal on tasks with existing retros

- **Anchor:** PRD R9 — *running `/specflow:complete {task-id}` (no `--amend` flag) on a task whose entry already has retro fields populated refuses with a structured chat line; the refusal is loud and exits without writing.*
- **Scope:** `skills/complete/SKILL.md` Phase A pre-write existence-check (reads `task-history.json` for the `task-id`; checks whether the three retro-only fields are populated per T3); refusal-message emitter; non-zero exit path.
- **Acceptance:** When `/specflow:complete {task-id}` is invoked WITHOUT the `--amend` flag AND the existing `task-history.json` entry for `{task-id}` has `superseded_by_retro: true` (the supersede flag is set only by a retro write per T4; the value provably toggles from `false` to `true` exactly once when the retro fires), the skill emits the literal sentinel chat line *"Task `{task-id}` already has a retro entry (created `{date}`). Use `--amend` to append addenda; manual edit of `task-history.json` is forbidden."* AND exits without writing to `task-history.json` or `decision-log.md`. The refusal is loud (chat line, not silent skip); a silent exit on this case is a failed run. The `{date}` token in the refusal line resolves to the existing entry's `completed_at` value. (Verifies AC-9.)
- **Depends on:** T3, T4.
- **Notes:** `superseded_by_retro: true` is the binary signal a retro fired; T4's supersede-write is the only path that flips it from `false` to `true`. This avoids the values-that-look-like-defaults boundary (per Gate 3 finding goal-r1-f2) where `actual_hours: 0`, `regressions_caught: {count:0, descriptions:[]}`, and `escaped_issues: {count:0, descriptions:[], blast_radii:[]}` could observationally match either 'populated retro with zero values' or 'never-written retro defaults'. Phase F.5 writes a non-retro entry first with `superseded_by_retro: false`; T9 distinguishes that from a retro-completed entry via the single boolean. The race-condition concern between the existence-check and the actual write is handled by T14's lock-file guard.
- **gate3-revision:** goal-r1-f2

### T10 — `--amend` mode appends to `addenda` array; preserves original entry intact

- **Anchor:** PRD R10 — *amend path appends to the existing entry's `addenda` array of shape `{date, kind: "escaped-issue" | "note", description, blast_radius?}`; original retro fields NOT modified; if `kind: escaped-issue` AND `blast_radius` medium or high, R6's elevation rule fires from the addendum.*
- **Scope:** `skills/complete/SKILL.md` Phase A `--amend` parser (recognises `--escaped-issue`, `--escaped-blast-radius`, `--note` flags); addendum builder (kind taxonomy: `escaped-issue` | `note`); `addenda` array appender; original-fields preservation guard; addendum-driven elevation invoker (delegates to T6).
- **Acceptance:** Running `/specflow:complete --amend {task-id}` with at least one of `--escaped-issue "{description}" --escaped-blast-radius {low|medium|high}` or `--note "{free text}"` appends an addendum to the existing entry's `addenda` array of exact shape `{date: "YYYY-MM-DD", kind: "escaped-issue" | "note", description: "...", blast_radius?: "low" | "medium" | "high"}` (the `blast_radius` key is present iff `kind` is `escaped-issue`). The original retro fields (the three fields per T3, plus all other top-level entry fields except `addenda`) are NOT modified by the amend path — a diff of `task-history.json` before vs after the amend shows the entry's `addenda` array grew by exactly one element AND no other field on that entry changed. When the addendum's `kind` is `escaped-issue` AND `blast_radius` is `medium` or `high`, R6's elevation rule fires from the addendum (per T6 logic) AND T7's user-review-before-write applies; the composed `decision-log.md` entry's `Rationale` cites the addendum's `description` (verbatim quote), NOT the original retro fields. **Accept-and-ship interaction:** on amend invocations where the existing entry has `accepted_with_failure: true`, T12's unconditional-elevation-prompt does NOT re-fire — T12's prompt fires once at the original retro-write time only; subsequent amends follow R6's two-condition rule (per T10 logic). The retro entry's `decision_log_links` field already captured the Phase F.3 entry at original-retro-time; an amend adds linkage only if R6 conditions fire on the addendum. Running `/specflow:complete --amend {task-id}` with neither `--escaped-issue` nor `--note` re-prompts and refuses to proceed without at least one. (Verifies AC-10.)
- **Depends on:** T3, T6, T7, T14.
- **Notes:** The `kind: "note"` taxonomy is interview-grounded per Round 5's "retroactive notes" resolution alongside escaped-issue captures (Gate 2 simplicity-r1-f1 push-back defence). Manual edit of `task-history.json` outside the skill is forbidden by convention; addenda are the only legitimate post-retro mutation surface. The accept-and-ship-interaction clause closes Gate 3 da-r1-f1 — the (accept-and-ship + amend with kind:note) cross-product is now pinned to T10's R6-conditional logic, never re-firing T12's unconditional prompt.
- **gate3-revision:** da-r1-f1

### T11 — Accept-and-ship retros fire normally with forced `accepted_with_failure: true`

- **Anchor:** PRD R11 — *tasks closed via `specflow:develop` Phase F.3 option-4 fire retro normally with `accepted_with_failure: true` set; clean-pass retros default to `false`; refusing the retro on accept-and-ship would lose the lesson.*
- **Scope:** `skills/complete/SKILL.md` Phase A Phase-F.3-marker reader (looks up the existing `task-history.json` entry's marker indicating option-4 was elected); `accepted_with_failure` flag writer; default-to-false on absent marker.
- **Acceptance:** When `specflow:develop` Phase F.3 option-4 (accept-and-ship) was elected for a task, the retro entry written by `specflow:complete` has `accepted_with_failure` set to the literal boolean `true`. When Verifier passed without rejection (clean-pass), the retro entry has `accepted_with_failure: false`. The flag is set by reading the Phase F.3 marker on the existing `task-history.json` entry; if the marker is absent the default is `false`. A retro entry with `accepted_with_failure` missing entirely (neither `true` nor `false`) is a failed write. (Verifies AC-11.)
- **Depends on:** T1.
- **Notes:** The accept-and-ship retro is the lesson the option-4 path was specifically designed to capture — refusing the retro on those tasks would defeat the purpose. The `accepted_with_failure: true` flag is the corpus-signal `/insights` reads for "calibre of failure that ships" pattern detection.

### T12 — Forced elevation prompt + Phase F.3 decision-log linkage on accept-and-ship

- **Anchor:** PRD R12 — *for accept-and-ship retros, elevation prompt fires unconditionally; the existing Phase F.3 `decision-log.md` entry's title is surfaced for confirmation/edit; never duplicated.*
- **Scope:** `skills/complete/SKILL.md` Phase C unconditional-elevation-prompt firer (overrides T6's two-condition logic when `accepted_with_failure: true`); `admin/decision-log.md` reader (locates entries whose `Related` field cites the task-id); user confirm-or-edit prompt; `decision_log_links` writer (records linked entry's `{date, title}`); duplication guard (confirm path records the link without writing a new entry; only edit-and-write produces a new entry).
- **Acceptance:** When `accepted_with_failure: true` is set on the retro entry (per T11), the elevation prompt fires unconditionally — the prompt fires regardless of whether T6's two conditions (user-flag, escaped-issue medium-or-high) fire. The skill reads `admin/decision-log.md`, locates every entry whose `Related` field cites the `task-id`, and surfaces each matching entry's title to the user with a confirm-or-edit prompt. The retro entry's `decision_log_links` field captures every confirmed/edited entry's `{date, title}` in the order shown. The skill never duplicates an existing decision-log entry — when the user confirms an existing entry, the link is recorded but no new entry is appended to `decision-log.md`; only the edit-and-write option produces a new `decision-log.md` entry. A run that writes a duplicate `decision-log.md` entry with the same title as an existing Phase F.3 entry is a failed run. The user can still answer N to drop all linkage; in that case `decision_log_links` is `[]` and `elevation_outcome: cancelled` is recorded (per T7). (Verifies AC-12.)
- **Depends on:** T7, T11.
- **Notes:** The unconditional firing serves the goal Outcome surface — accept-and-ship tasks DID close with a known failure, and the retro entry needs to point at the existing Phase F.3 decision-log entry for the audit trail to be observable. Duplicate-prevention is the corpus-quality property; never duplicate an existing entry. **The unconditional elevation prompt fires exactly once per retro entry — at original retro-write time. Subsequent amends (per T10) follow R6's two-condition rule; T12's unconditional logic does NOT re-fire on amends.** This pins the (accept-and-ship + amend) cross-product per Gate 3 da-r1-f1.
- **gate3-revision:** da-r1-f1

### T13 — Scope-changed tasks record final task definition + `scope_change_log`

- **Anchor:** PRD R13 — *tasks touched by `specflow:scope-change` mid-execution record the FINAL post-scope-change task definition; the retro entry adds `scope_change_log` listing every `decision-log.md` entry whose `Related` field references the task-id; single retro entry per task regardless of scope-change count.*
- **Scope:** `skills/complete/SKILL.md` Phase B final-definition reader (reads `tasks.md` for the current task definition, NOT a historical version); `admin/decision-log.md` Related-field parser (substring match accepting both formats per the PRD Open Questions); `scope_change_log` writer (chronological order); single-entry guarantee.
- **Acceptance:** When `admin/decision-log.md` contains at least one entry whose `Related` field cites the `task-id` (e.g. an entry written by `specflow:scope-change` mid-execution), the retro entry's `scope_change_log` field captures every such entry's `{date, title}` in chronological order (oldest first). The retro questions are asked against the final post-scope-change task definition — the skill reads the current `features/{NNN-slug}/{NNN-slug}-tasks.md` entry for `{task-id}`, NOT a historical version. The retro entry is exactly one record per task regardless of scope-change count — a run that writes two retro entries for a single `task-id` due to multiple scope-changes is a failed run. When `admin/decision-log.md` contains zero entries citing the `task-id`, `scope_change_log` is the empty array `[]`. The Related-field substring match accepts both formats `{NNN-slug}-T{N}` and prose mentions of the task-id (per PRD Open Questions). When `tasks.md` no longer contains an entry for `{task-id}` (the task was dropped during scope-change), the skill emits the structured failure line *"Scope-changed task `{task-id}` no longer present in `tasks.md`. Retro skipped; review `decision-log.md` for the dropping rationale."* and exits without writing. (Verifies AC-13.)
- **Depends on:** T1.
- **Notes:** The substring-match approach for Related-field parsing is pragmatic per the PRD Open Questions — `specflow:decision`'s SKILL.md does not currently enforce a citation format. A canonical format is a follow-up enhancement on `specflow:decision`. Single-entry-per-task is the corpus-shape invariant — `/insights` reads one record per task even across scope-changes. **Branch (c) (task dropped by scope-change) is an implementation choice not directly named in PRD R13/AC-13; the choice is `retro skipped + structured failure line` rather than `stub retro recording the drop`. Rationale: a dropped task did not close, and `specflow:complete` exists to capture closure-time lessons; a stub retro would create a corpus entry that does not represent a closed unit of work. Surface this choice to the user at implementation; if the assumption breaks (a downstream consumer wants the stub-retro shape), revise via PRD scope-change.**
- **gate3-revision:** tbc-r1-f1

### T14 — Concurrent-trigger lock-file guard at `admin/scratch/complete-{task-id}.lock`

- **Anchor:** PRD R14 — *when auto-fire and manual CLI race on the same task-id, the second-to-start path detects the in-flight retro via a per-task lock file; refuses with a structured chat line; lock files older than 30 minutes are stale and ignored.*
- **Scope:** `skills/complete/SKILL.md` Phase A lock-file creator (atomic create at `admin/scratch/complete-{task-id}.lock` with timestamp); lock-file detector (atomic check against existing lock); lock-file remover (atomic remove on successful write OR on refusal exit); stale-lock detector (compares lock timestamp to now; ignores if > 30 minutes); structured-refusal emitter.
- **Acceptance:** When `specflow:develop` Phase F auto-fires `specflow:complete` for a task AND a manual `/specflow:complete {task-id}` is invoked concurrently for the same task-id before the auto-fire completes, the second-to-start path detects the existing lock file at `admin/scratch/complete-{task-id}.lock` AND refuses with the literal sentinel chat line *"Retro for `{task-id}` is in flight (started `{timestamp}`). Wait for it to complete, then use `--amend` if you have additions."* (the `{timestamp}` token resolves to the lock file's recorded ISO-8601 timestamp). The lock file is created atomically by whichever path starts first AND removed atomically when that path completes — both successful-write exits AND refusal exits remove the lock. A second-to-start path that succeeds in writing while the lock file exists (lock not respected) is a failed run; a path that completes without removing its own lock is a failed run. Lock files older than 30 minutes are considered stale AND ignored — a path that detects a stale lock proceeds with its own retro AND overwrites the stale lock with a fresh timestamp. (Verifies AC-14.)
- **Depends on:** T1, T2.
- **Notes:** The lock-file pattern matches the Phase 1 scratch convention (`admin/scratch/{NNN-slug}-*` directory). The 30-minute stale-lock heuristic covers crashed-orchestration cleanup; the PRD Open Questions notes a `--clear-stale-locks` admin verb may land in v2 if consumers report stale-lock false-positives. Per Gate 2 (concern da-r1-f1) — the binary "skip if exists" framing was insufficient because the existence-check itself races; the lock file is the in-flight marker that closes the TOCTOU window.
- **gate2-revision:** da-r1-f1

### T15 — Chat-line summary on every successful retro write; structured failure on every refused exit

- **Anchor:** PRD AC-15 / Goal Outcome surfaces (c) chat-line summary citing the new entry's id + (d) zero silent failures — *every successful retro write emits a chat-line summary citing the `task-history.json` entry id and any `decision-log.md` entry written; failed writes emit a structured failure line citing the failure reason; no silent exits.*
- **Scope:** `skills/complete/SKILL.md` Phase D summary-emitter (success path); structured-failure emitter (refusal paths from T4 schema-dependency, T9 plain-mode existence, T13 scope-changed-task-missing, T14 lock-file race); failure-reason taxonomy.
- **Acceptance:** On every successful retro write (entry appended to `task-history.json` per T1 or T2; or `addenda` array appended per T10), the skill emits the literal chat line *"Retro captured: `task-history.json` entry `{id}`. Decision-log entry: `{title}`."* where `{id}` resolves to the retro entry's `id` and `{title}` resolves to the new `decision-log.md` entry's title (or the literal string `none` when no `decision-log.md` write happened). On every refused exit (the four refusal paths: T4 cross-skill schema dependency unmet, T9 plain-mode existing-retro, T13 scope-changed-task missing from `tasks.md`, T14 lock-file race), the skill emits a structured failure line citing the failure reason — the failure line is the sentinel string defined in the corresponding refusal task (T4, T9, T13, T14). A successful retro write that does not emit the summary line is a failed run. A refused exit that does not emit one of the four sentinel failure lines is a failed run (silent exit). (Verifies AC-15.)
- **Depends on:** T1, T2, T4, T9, T10, T13, T14.
- **Notes:** The chat-line summary is the user-visible feedback surface — every retro produces an observable trace AND every refusal surfaces an explicit reason. Silent failure is the failure mode this skill exists to remove.

## Open questions inherited from PRD

- **Q:** Lock-file cleanup on crashed orchestration — should the skill ship a `/specflow:complete --clear-stale-locks` admin verb for explicit cleanup, or is the 30-minute time-based heuristic sufficient? — affects: T14. PRD recommends time-based heuristic for v1; revisit if consumers report stale-lock false-positives.
- **Q:** `actual_hours` user-declined-to-estimate sentinel — does `0` collide with legitimate sub-hour tasks (e.g. a 20-minute typo fix where the user honestly reports `0.3`)? — affects: T3. PRD recommends documenting `0` exactly as "declined" with `null` reserved for "not yet captured" and any positive number (including fractional) as "captured value"; confirm at implementation.
- **Q:** `scope_change_log` Related-field parsing — what's the canonical task-id citation format in `decision-log.md` Related fields (e.g. `T-2026-04-T2`, `{NNN-slug}-T{N}`, prose mention)? — affects: T13. PRD recommends substring match accepting both formats; canonical format is a follow-up enhancement on `specflow:decision`.

## See also

- PRD: [`./003-complete-skill-prd.md`](./003-complete-skill-prd.md)
- Interview: [`./003-complete-skill-interview.md`](./003-complete-skill-interview.md)
- Gate 2 manifest: [`./debate-log/prd-gate2/manifest.md`](./debate-log/prd-gate2/manifest.md)
- Gate 3 manifest: [`./debate-log/tasks-gate3/manifest.md`](./debate-log/tasks-gate3/manifest.md)
- Sister Phase 3 skills: `skills/decision/SKILL.md`, `skills/scope-change/SKILL.md`
- Adjacent Phase 2 skill: `skills/develop/SKILL.md` (Phase F.5 boundary; cross-skill `superseded_by_retro` schema dependency)
- Tests: `./003-complete-skill-test.md` (generated by `specflow:test`)
