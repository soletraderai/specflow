---
feature: 003-complete-skill
status: draft
created: 2026-05-06
interview: ./003-complete-skill-interview.md
---

# specflow:complete — retro skill that closes the loop between work-done and project-memory

## Vision

`specflow:complete` closes the loop between work-done and project-memory. Every closed task leaves a structured trace in `admin/task-history.json` so future tasks read the lessons; significant patterns elevate to `admin/decision-log.md` for the human-readable narrative. The skill turns task outcomes into structured project memory — without it, the self-learning loop has no input data and Phase 3's compounding signals never accumulate. Power Users running `/specflow:complete` manually at the end of a led task get a short retro that captures the after-the-fact lesson the development-time write cannot produce; Developer / API Consumer profiles whose green-lane batches close via `specflow:develop` Phase F get the retro inline as the per-task closure boundary; future-readers (the same engineer six months later, a teammate onboarding, the `/insights` cadence reading the corpus for patterns) consume `task-history.json` and `decision-log.md` without ever invoking the skill. The skill is the producer the self-learning loop depends on for input data — manual + auto-fire from `specflow:develop` are the two trigger paths, and append-only discipline plus the `--amend` escape hatch keep the corpus honest.

## Problem

Phase 1 ships the `admin/task-history.json` and `admin/decision-log.md` files as empty templates at setup; Phase 2's `specflow:develop` Phase F.5 writes six development-time fields (`lane_assigned`, `ai_assistance_level`, `elapsed_minutes`, `what_worked`, `what_didnt_work`, `blast_radius_outcome`) on Verifier pass. But the development-time write is captured at completion-of-implementation, not after-ship — three of the most lesson-bearing fields the corpus needs (`actual_hours` user-reported, `regressions_caught` during implementation, `escaped_issues` found post-ship) cannot be produced at the Phase F boundary because they require user reflection or post-ship signal that doesn't exist at the time Phase F runs. The failure modes that result are well-known: (a) closed tasks leave no retro trace and the lessons evaporate — six months later the same gotcha recurs because nobody wrote it down; (b) significant decisions made during execution are remembered inconsistently across team members because the source was a chat thread, not a versioned file; (c) `/insights` (monthly) and `specflow:task` similarity-search have no corpus to read because nothing wrote the retro fields the development-time write cannot produce; (d) when escaped issues surface a week after ship, there's no idempotent path for amending the original retro entry — the user either manually edits `task-history.json` (breaking the audit trail) or creates a duplicate entry (breaking similarity-search).

The fix is structural: a single skill that owns the retro layer end-to-end, runs once per task at the boundary between Phase F's task-history write and the cleanup step (auto-fire) or via manual `/specflow:complete {task-id}` (escape hatch), captures the three retro-only fields the development-time write cannot produce, supersedes the Phase F placeholder values for fields the retro is the source-of-truth for (`what_worked`, `what_didnt_work`, `ai_assistance_level`), elevates significant patterns to `decision-log.md` via a two-condition rule (user-flagged OR escaped-issue at medium+ blast radius), refuses plain-mode re-invocation on tasks with existing retros (forces `--amend` for follow-ups), and stays append-only by design with an `addenda` array for post-ship corrections.

## Goals

- Auto-fire from `specflow:develop` Phase F once per task at the per-task Verifier-pass boundary (Green-lane batched flows fire per-task, never per-batch — preserves the per-task signal `/insights` reads later) AND manual `/specflow:complete {task-id}` as the escape hatch for tasks closed outside `specflow:develop`.
- Capture three retro-only fields the development-time write cannot produce: `actual_hours` (user-reported reflective hours, distinct from Phase F's wall-clock `elapsed_minutes`), `regressions_caught` (count + brief description, caught before ship), `escaped_issues` (count + brief description, found post-ship; capturable at retro time with default 0 and amendable later via `--amend --escaped-issue`).
- Supersede Phase F's placeholder values for `what_worked`, `what_didnt_work`, `ai_assistance_level` — the retro is the source of truth for these fields because it runs after task closure with user reflection; the Phase F write is the placeholder.
- Two-condition decision-log elevation rule: user-flagged via the retro question "Should this land in `decision-log.md`?" OR `escaped_issues` field captures a post-ship issue with `blast_radius` medium or high. Conditions OR'd; user reviews the composed entry before write; lane-vs-outcome divergence is `/insights`-cadence territory, not per-task elevation.
- Plain-mode `/specflow:complete {task-id}` refuses on tasks with existing retros with a structured chat line; `--amend` flag is the explicit path for follow-up captures (escaped issues surfacing post-ship, retroactive notes); amend appends to an `addenda` array on the existing entry, preserving the original retro intact.
- Accept-and-ship tasks (per `specflow:develop` Phase F.3 option-4) fire retro normally with a forced `accepted_with_failure: true` flag and a forced elevation prompt; the existing Phase F.3 `decision-log.md` entry's title is surfaced for confirmation/edit, never duplicated.
- Scope-changed tasks (touched by `specflow:scope-change` mid-execution) record the FINAL post-scope-change task definition + a `scope_change_log` field listing every `decision-log.md` entry whose Related field references the task-id. Single retro entry regardless of scope-change count.
- Snake_case schema convention; field-name reconciliation note acknowledges Appendix I3's camelCase example is illustrative-not-normative; the working `task-history.json` schema documented in this PRD is the v1 contract.
- Zero silent failures: refused invocation on incomplete acceptance, schema-gap, or duplicate task-id surfaces explicitly to the user with a structured reason. Every artefact lives at `admin/` so the corpus stays per-repo and committed.

## Non-goals

- **Linear webhook auto-fire.** Out of scope per `docs/PRD.md` Appendix I8 ("Manual first") and the goal's Out-of-scope-at-goal-level. v1 ships zero webhook integration — no polling, no listener, no `config.json.complete.webhook*` knobs. v2 enhancement PRD lands webhook polling when a documented consumer surfaces (e.g. tasks closed in Linear UI without going through `specflow:develop`).
- **Cross-feature retro rollups.** That's `/insights` (monthly) Phase 3 scope item 10 — a separate skill that consumes `task-history.json` rather than producing it. `specflow:complete` is the producer; the consumer is its own PRD.
- **Similarity-search on the corpus during retro capture.** Surfacing similar past tasks during retro is `specflow:task` and `specflow:misc` Phase 3 scope item 5 (consumers, not the producer). The retro skill writes the corpus; querying it belongs to consumers.
- **Auto-promotion of patterns to the rules registry.** Phase 3 scope item 8's observation → guideline → non-negotiable promotion is `/insights`-driven, never `specflow:complete`-driven. The retro skill writes only to `task-history.json` + `decision-log.md`; `admin/rules/` is touched only by `/insights` after multiple retros surface the same pattern, with human sign-off at each promotion.
- **Editing or removing existing entries.** Append-only by design. Corrections land as new entries (linked back via `supersedes` field — convention from `decision-log.md`) or as addenda on existing entries (via `--amend`). The skill never edits a prior retro field in place; manual edits to `task-history.json` outside the skill are forbidden by convention.
- **Adversarial review gating (multi-agent debate manifest).** Phase 3 retro skills do not gate at adversarial review — the only "gate" the retro skill enforces is the schema-parse check + the binary "all required fields present" check. Consistent with sister skills `specflow:decision` (no manifest) and `specflow:misc` (no manifest). A debate manifest on every retro would be over-instrumentation for a small Phase 3 skill.

## Users

- **Power User (per `admin/profiles.json`)** — frontend / backend engineers running implementation in their own repos. They invoke `/specflow:complete {task-id}` manually at the end of tasks they led (red-lane human-led work, misc-tasks, work that closed outside `specflow:develop`). They need the retro questions short (5 questions, plain language), the elevation prompt clear (one Y/N + description), and the schema additions stable (snake_case, documented).
- **Developer / API Consumer (per `admin/profiles.json`)** — engineers supervising AI-led green-lane batches. They never invoke `/specflow:complete` directly; the skill auto-fires from `specflow:develop` Phase F as each task closes. They need the retro to fire per-task even in batched flows (so per-task signal reaches `/insights` un-collapsed), and they need the retro questions to be short enough that batched closures don't accumulate prompt fatigue.
- **Secondary: future-readers** — the same engineer six months later, a teammate onboarding to the codebase, the `/insights` (monthly) cadence reading the corpus for patterns. They consume `task-history.json` and `decision-log.md` without ever invoking the skill. They benefit from append-only discipline (the corpus stays trustworthy as audit trail), schema consistency (snake_case across all entries; field names documented), and the two-condition elevation rule (`decision-log.md` stays high-signal because low-signal divergences route to `/insights`-cadence pattern detection instead).

## Requirements

- **R1.** Auto-fire from `specflow:develop` Phase F. The retro skill is invoked once per task at the boundary between Phase F.5 (task-history.json write of development-time fields) and Phase F.6 (scratch cleanup), AFTER Verifier passes (or option-4 accept-and-ship is elected) AND AFTER the per-task PR opens. Green-lane batched flows fire per-task, never per-batch — the per-task signal is the signal `/insights` reads later, and batching across a 3-task green batch would collapse three distinct retros into one.
  - Trace: interview Round 1 — *Phase F auto-fires `specflow:complete` once per task after Verifier passes (or option-4 accept-and-ship is elected) AND after the per-task PR opens, before Phase F.6 scratch-cleanup; Green-lane batched flow fires per-task not per-batch*.
  - Serves goal: Outcome (every closed task leaves a structured trace; per-task signal preserved for `/insights` consumption).

- **R2.** Manual `/specflow:complete {task-id}` as escape hatch. The skill accepts a manual CLI invocation `/specflow:complete {task-id}` for tasks closed outside `specflow:develop` (e.g. red-lane human-led tasks where the user closed the work manually, or misc-tasks that closed without running through Phase F). The manual path runs the same retro flow as auto-fire; the trigger differs but the captured fields and write surfaces are identical.
  - Trace: interview Round 1 — *manual `/specflow:complete {task-id}` is the alternate escape-hatch path for tasks closed outside `specflow:develop`*.
  - Serves goal: Audience (Power Users running implementation in their own repos invoke `/specflow:complete` manually at the end of led tasks).

- **R3.** Three retro-only fields beyond Phase F's development-time set. The retro adds three new fields the development-time write cannot produce: (a) `actual_hours` — user-reported reflective hours-elapsed, distinct from Phase F's wall-clock `elapsed_minutes` (the retro field captures thinking/blocked/context-switch time the orchestration cannot observe); (b) `regressions_caught` — count + brief description of regressions caught during implementation (machine-check fail, gate failure, Verifier reject before final pass — anything caught before ship); (c) `escaped_issues` — count + brief description of issues found AFTER ship (production incident, code-review catch, follow-up bug). At retro time the default for `escaped_issues` is `{count: 0, descriptions: []}`; the field is amendable later via `--amend --escaped-issue` (R10).
  - Trace: interview Round 2 — *new fields beyond Phase F's development-time set are `actual_hours`, `regressions_caught`, `escaped_issues`*.
  - Serves goal: Driving value (lessons evaporate failure mode — these three fields are the lesson-bearing fields the development-time write cannot produce because Phase F runs at completion-of-implementation, not after-ship).

- **R4.** Retro supersedes Phase F placeholder values for three fields. The retro write SUPERSEDES the Phase F write's `what_worked`, `what_didnt_work`, and `ai_assistance_level` values — the retro is the source of truth for these fields because it runs after task closure with user reflection; the Phase F write is the placeholder. The supersede is a write-in-place on the existing entry (not append), explicitly recorded in the entry's `superseded_by_retro: true` flag so a downstream reader can tell the field's lineage. **Cross-skill schema dependency:** `specflow:develop` Phase F.5's task-history.json write must include the `superseded_by_retro: false` default flag in its initial write so the retro's supersede semantic has a target field. If the current `specflow:develop` SKILL.md doesn't include this default flag, ship the addition as a `specflow:develop` enhancement PRD before `specflow:complete` v1 lands.
  - Trace: interview Round 2 — *retro `what_worked` + `what_didnt_work` + `ai_assistance_level` REPLACE the Phase F write's placeholder values*. Extended at Gate 2 per surgical-r1-f1: explicit cross-skill schema dependency call-out for the `superseded_by_retro` default flag.
  - Serves goal: Outcome (structured trace with retro source-of-truth for reflective fields; lineage observable via flag).

- **R5.** Snake_case schema convention; reconciliation note for Appendix I3. The retro fields and the supersede fields use snake_case (matching `specflow:develop` Phase F.5's working convention). Appendix I3's camelCase example (`whatWorked`, `aiAssistance`) is illustrative-not-normative; the v1 working schema documented in this PRD is the contract going forward. The PRD's Schema Appendix (below) names the full field list with snake_case naming.
  - Trace: interview Round 2 — *field-name reconciliation: snake_case is the working convention in `specflow:develop`'s shipped schema; Appendix I3's camelCase is example-not-spec*.
  - Serves goal: Outcome (schema consistency for downstream consumers).

- **R6.** Two-condition decision-log elevation rule. A retro elevates to `admin/decision-log.md` when EITHER (a) the user explicitly flags the retro for elevation via the retro question *"Should this land in `decision-log.md`? Y/N + description if Y"*, OR (b) the `escaped_issues` field captures a post-ship issue with `blast_radius` medium or high. Conditions are OR'd; any one fires the elevation. Lane-vs-outcome divergence is `/insights`-cadence territory, not per-task elevation — a single divergent task is a data-point, not a decision.
  - Trace: interview Round 3 — *decision-log elevation fires when EITHER (a) the user explicitly flags the retro for elevation, OR (b) the `escaped_issues` field captures a post-ship issue with blast_radius medium or high; conditions are OR'd*.
  - Serves goal: Driving value (decisions inconsistent failure mode — per-task elevation triggers on observable shapes, no judgement-call "is this important?"; corpus stays high-signal because low-signal patterns route to `/insights` instead).

- **R7.** User reviews composed elevation entry before write. When the elevation conditions fire (per R6), the skill composes the `decision-log.md` entry from the retro fields (Title = task title prefixed with `Retro:`; Context = the task's PRD anchor; Decision = "{user-stated lesson} — pattern surfaced via specflow:complete"; Rationale = the retro field that fired the elevation; Related = the task-history entry id + PRD path) and surfaces it to the user for review-or-edit-or-cancel BEFORE writing. The user can confirm, edit (skill re-renders), or cancel (the elevation is dropped; no decision-log write; the retro entry's `elevation_offered: true; elevation_outcome: cancelled` flags are recorded for `/insights`-cadence pattern detection).
  - Trace: interview Round 3 — *user reviews the composed entry before write — explicit confirm step, not silent*.
  - Serves goal: Driving value (corpus stays high-signal — explicit confirm prevents low-signal entries; `/insights` reads cancelled-elevation flags as their own pattern signal).

- **R8.** Zero webhook integration in v1. The skill ships zero webhook plumbing — no polling, no listener, no `config.json.complete.webhook*` knobs. The two trigger paths are R1's auto-fire from `specflow:develop` Phase F and R2's manual CLI. v2 enhancement PRD lands webhook polling when a documented consumer surfaces (e.g. tasks closed in Linear UI without going through `specflow:develop`).
  - Trace: interview Round 4 — *v1 ships zero webhook integration; two trigger paths only — `specflow:develop` Phase F auto-fire + manual `/specflow:complete {task-id}`; no `config.json.complete.webhook*` knobs*.
  - Serves goal: Out-of-scope-at-goal-level (Linear webhook auto-fire is deferred per `docs/PRD.md` Appendix I8); Simplicity First (no speculative configurability).

- **R9.** Plain-mode re-invocation refuses on tasks with existing retros. Running `/specflow:complete {task-id}` (no `--amend` flag) on a task whose `task-history.json` entry already has retro fields populated refuses with a structured chat line: *"Task `{task-id}` already has a retro entry (created `{date}`). Use `--amend` to append addenda; manual edit of `task-history.json` is forbidden."* The refusal is loud (chat line, not silent skip) and exits without writing.
  - Trace: interview Round 5 — *plain-mode `/specflow:complete {task-id}` on a task with existing retro entry refuses with a structured chat line*.
  - Serves goal: Out-of-scope-at-goal-level (editing or removing existing entries forbidden; append-only by design).

- **R10.** `--amend` mode appends to `addenda` array, preserves original entry intact. The amend path `/specflow:complete --amend {task-id} [--escaped-issue "{description}" --escaped-blast-radius {low|medium|high}] [--note "{free text}"]` appends an addendum to the existing entry's `addenda` array (new field on the schema): `{date, kind: "escaped-issue" | "note", description, blast_radius?: "low"|"medium"|"high"}`. The original retro fields are NOT modified. If the addendum's `kind` is `escaped-issue` AND `blast_radius` is `medium` or `high`, R6's elevation rule fires from the addendum (R7's user-review-before-write still applies).
  - Trace: interview Round 5 (extended for kind taxonomy) + Round 2 — *amend mode appends a corrigendum-style addendum to the entry rather than creating a new entry*.
  - Serves goal: Driving value (lessons evaporate failure mode — post-ship escaped-issue capture has a legitimate idempotent path that preserves the audit trail).

- **R11.** Accept-and-ship tasks fire retro normally with forced flag. Tasks closed via `specflow:develop` Phase F.3 option-4 (accept the failure and ship) fire retro normally with `accepted_with_failure: true` set on the retro entry. The flag distinguishes accept-and-ship retros from clean-pass retros so `/insights` can pattern-match on the calibre of failure that ships. Refusing the retro on accept-and-ship would lose the lesson the option-4 path was specifically designed to capture.
  - Trace: interview Round 6 — *retros fire normally on accept-and-ship tasks with a forced `accepted_with_failure: true` flag*.
  - Serves goal: Outcome (every closed task leaves a structured trace; accept-and-ship tasks DID close).

- **R12.** Forced elevation prompt + Phase F.3 decision-log linkage on accept-and-ship. For accept-and-ship retros, the elevation prompt fires unconditionally (the user can still answer N, but the prompt fires regardless of R6's conditions because the option-4 path already wrote a `decision-log.md` entry per Phase F.3 — the retro entry needs to point at it). The skill reads `admin/decision-log.md`, locates entries whose Related field cites the task-id created by Phase F.3, and surfaces the entry's title to the user with a confirm-or-edit prompt. The retro entry's `decision_log_links` field captures the linked entry's date + title; the skill never duplicates the existing entry.
  - Trace: interview Round 6 — *the elevation prompt fires unconditionally for these tasks (user can still answer N); the existing Phase F.3 decision-log entry's title is surfaced for confirmation/edit (not duplicated)*.
  - Serves goal: Outcome (accept-and-ship audit trail observable; no duplicate decision-log entries).

- **R13.** Scope-changed tasks record final task definition + `scope_change_log`. Tasks touched by `specflow:scope-change` mid-execution record the FINAL post-scope-change task definition in the retro entry; the retro questions are asked against the final definition. The retro entry adds a `scope_change_log` field listing every `decision-log.md` entry whose Related field references the task-id (the skill reads `admin/decision-log.md` to compute this list, captures `{date, title}` per entry). Single retro entry per task regardless of scope-change count.
  - Trace: interview Round 7 — *retro records the final post-scope-change task definition; `scope_change_log` field lists every `decision-log.md` entry whose Related field references the task-id*.
  - Serves goal: Outcome (structured trace coherent — describes one piece of work; audit trail intact via cited decision-log entries).

- **R14.** Concurrent-trigger guard between auto-fire and manual paths. When `specflow:develop` Phase F auto-fires the retro for a task AND the user concurrently invokes `/specflow:complete {task-id}` for the same task before the auto-fire completes, the second invocation detects the in-flight retro via a per-task lock file at `admin/scratch/complete-{task-id}.lock` (created by whichever path starts first; removed atomically when that path completes successfully or surfaces a refusal). The second invocation refuses with the structured chat line: *"Retro for `{task-id}` is in flight (started `{timestamp}`). Wait for it to complete, then use `--amend` if you have additions."* Lock files older than 30 minutes are considered stale and ignored (covers crashed-orchestration cleanup).
  - Trace: Gate 2 review da-r1-f1 (concern) — *concurrent-trigger ambiguity: auto-fire from Phase F and manual CLI could race on the same task-id; binary "skip if exists" framing insufficient because the existence-check itself races*.
  - Serves goal: Driving value (zero silent failures — the race surfaces explicitly via the in-flight lock-file check rather than producing a silent overwrite or a duplicate-entry corruption).

## Acceptance criteria

- **AC-1.** When `specflow:develop` Phase F passes Verifier (or option-4 accept-and-ship is elected) AND the per-task PR opens, the skill auto-fires `specflow:complete` for that task before Phase F.6 scratch-cleanup. The retro entry's `id` field matches the Phase F task-id (`{NNN-slug}-T{N}`); the entry's `triggered_by` field is the literal `specflow:develop-phase-f`. (Verifies R1.)

- **AC-2.** Running `/specflow:complete {task-id}` from the CLI on a task that has no existing retro entry runs the same retro flow as auto-fire and writes the entry with `triggered_by: "manual-cli"`. Tasks `{task-id}` not present in `admin/task-history.json` (i.e. not created by `specflow:develop` or `specflow:task` Phase D) are accepted — the skill creates a fresh entry with retro fields populated and `triggered_by: "manual-cli"`. (Verifies R2.)

- **AC-3.** Every retro entry includes the three retro-only fields with non-null values: `actual_hours: number` (user-reported reflective hours; never null — if the user declines to estimate, the field is `0` with the convention that `0` means "user declined to estimate"), `regressions_caught: {count: number, descriptions: string[]}`, `escaped_issues: {count: number, descriptions: string[], blast_radii: ("low"|"medium"|"high")[]}`. At retro time the default for `escaped_issues` is `{count: 0, descriptions: [], blast_radii: []}`. (Verifies R3.)

- **AC-4.** When the retro fires on a task whose `task-history.json` entry already has Phase F.5's `what_worked`, `what_didnt_work`, `ai_assistance_level` populated, the retro write replaces those values in place AND sets `superseded_by_retro: true` on the entry. The original Phase F values are NOT preserved (the retro is the source of truth). **Schema dependency:** AC-4 depends on `specflow:develop` Phase F.5's task-history.json write including the `superseded_by_retro: false` default flag in its initial write so this AC's supersede semantic has a target field. If the current `specflow:develop` SKILL.md doesn't include this default flag, ship the addition as a `specflow:develop` enhancement PRD before `specflow:complete` v1 lands. (Verifies R4.)

- **AC-5.** Every retro entry's field names are snake_case. No camelCase fields appear in any retro write. The PRD's Schema Appendix below is the authoritative field list; entries that include fields not on that list are a failed write (entry is rejected; user is asked to revise). (Verifies R5.)

- **AC-6.** Decision-log elevation fires when EITHER (a) the user answers Y to the retro question *"Should this land in `decision-log.md`?"* AND provides a non-empty description, OR (b) the retro entry's `escaped_issues` field has at least one entry whose `blast_radius` is `medium` or `high`. The skill records the firing condition in the retro entry's `elevation_fired_by` field (`"user-flag"` | `"escaped-issue-medium-or-high"` | `"none"`). When neither condition fires, no `decision-log.md` write happens. Lane-vs-outcome divergence (e.g. lane assigned Green but blast_radius_outcome = high) is NOT a firing condition — the field's value is recorded for `/insights`-cadence pattern detection. (Verifies R6.)

- **AC-7.** When elevation fires (per AC-6), the skill composes the `decision-log.md` entry (Title = `Retro: {task title}`; Context = task's PRD anchor; Decision = "{user-stated lesson} — pattern surfaced via specflow:complete"; Rationale = the retro field that fired the elevation, quoted; Related = task-history entry id + PRD path) and surfaces it to the user with three options: confirm-and-write, edit-and-write (skill accepts an edited version, re-validates the six-field shape, writes), cancel. On cancel the retro entry's `elevation_offered: true; elevation_outcome: "cancelled"` flags are set; no `decision-log.md` write happens. On confirm-and-write or edit-and-write the entry is appended to `decision-log.md` AND the retro entry's `decision_log_links` field captures the new entry's `{date, title}`. (Verifies R7.)

- **AC-8.** No webhook code paths exist in `specflow:complete`. The skill's SKILL.md `requires:` and `produces:` field lists do not reference any webhook config or listener path; no `config.json.complete.webhook*` keys are read or written; no HTTP listener is started. Trigger paths are exactly two: auto-fire from `specflow:develop` Phase F (per R1) and manual CLI invocation (per R2). (Verifies R8.)

- **AC-9.** Running `/specflow:complete {task-id}` (no `--amend` flag) on a task whose `task-history.json` entry already has the three retro-only fields populated (per AC-3) refuses with the literal sentinel chat line: *"Task `{task-id}` already has a retro entry (created `{date}`). Use `--amend` to append addenda; manual edit of `task-history.json` is forbidden."* The skill exits without writing. (Verifies R9.)

- **AC-10.** Running `/specflow:complete --amend {task-id}` with at least one of `--escaped-issue "{description}" --escaped-blast-radius {low|medium|high}` or `--note "{free text}"` appends an addendum to the existing entry's `addenda` array of shape `{date, kind: "escaped-issue" | "note", description, blast_radius?: "low"|"medium"|"high"}`. The original retro fields are NOT modified by the amend path (`task_history.json` diff shows the entry's `addenda` array grew by one element; no other field changed). When the addendum's `kind` is `escaped-issue` AND `blast_radius` is `medium` or `high`, R6's elevation rule fires from the addendum (AC-7's user-review-before-write applies; the composed decision-log entry's Rationale cites the addendum, not the original retro fields). (Verifies R10.)

- **AC-11.** When `specflow:develop` Phase F.3 option-4 (accept-and-ship) was elected for a task, the retro entry written by `specflow:complete` has `accepted_with_failure: true`. Clean-pass retros (Verifier passed without rejection) have `accepted_with_failure: false` (default). The flag is set by reading the Phase F.3 marker in the existing task-history.json entry; if the marker is absent the default is `false`. (Verifies R11.)

- **AC-12.** When `accepted_with_failure: true` is set (per AC-11), the elevation prompt fires unconditionally (regardless of AC-6's two conditions). The skill reads `admin/decision-log.md`, locates entries whose Related field cites the task-id, and surfaces each matching entry's title to the user with a confirm-or-edit prompt. The retro entry's `decision_log_links` field captures every confirmed/edited entry's `{date, title}`. The skill never duplicates an existing decision-log entry (if the user confirms an existing entry, the link is recorded but no new entry is written; only edit-and-write produces a new decision-log entry). (Verifies R12.)

- **AC-13.** When `admin/decision-log.md` contains at least one entry whose Related field cites the task-id (e.g. an entry written by `specflow:scope-change` mid-execution), the retro entry's `scope_change_log` field captures every such entry's `{date, title}` in chronological order. The retro questions are asked against the final post-scope-change task definition (the skill reads the current `tasks.md` entry, not a historical version). The retro entry is one record regardless of scope-change count. (Verifies R13.)

- **AC-14.** When auto-fire from `specflow:develop` Phase F and manual `/specflow:complete {task-id}` race on the same task-id, the second-to-start path detects the lock file at `admin/scratch/complete-{task-id}.lock` and refuses with the literal sentinel chat line: *"Retro for `{task-id}` is in flight (started `{timestamp}`). Wait for it to complete, then use `--amend` if you have additions."* The lock file is created atomically by whichever path starts first AND removed atomically when that path completes (both successful-write and refusal exits). Lock files older than 30 minutes are ignored as stale (covers crashed-orchestration cleanup); a path that detects a stale lock proceeds with its own retro and overwrites the stale lock. (Verifies R14.)

- **AC-15.** On every successful retro write, the skill emits a chat-line summary citing the new `task-history.json` entry's id AND any `decision-log.md` entry's title written in the same run: *"Retro captured: `task-history.json` entry `{id}`. Decision-log entry: `{title}` (or `none`)."* Failed writes (schema violation, refused invocation, race-detected, scope-changed task missing from `tasks.md`) emit a structured failure line citing the failure reason; no silent exits. (Verifies the goal Outcome surface (c) — chat-line summary citing the new entry's id; goal surface (d) — zero silent failures.)

## Schema appendix — `task-history.json` retro entry shape (v1)

The authoritative field list. Snake_case. Reconciles with `specflow:develop` Phase F.5's working schema; supersedes Appendix I3's camelCase example.

```json
{
  "id": "{NNN-slug}-T{N}",
  "feature": "{NNN-slug}",
  "task_id": "T{N}",
  "title": "{from tasks.md}",
  "scope": ["{from tasks.md Scope field}"],
  "completed_at": "{YYYY-MM-DD}",
  "triggered_by": "specflow:develop-phase-f | manual-cli",

  "lane_assigned": "green | yellow | red",
  "ai_assistance_level": "full | partial | none",
  "elapsed_minutes": 0,
  "what_worked": "{retro text supersedes Phase F placeholder}",
  "what_didnt_work": "{retro text supersedes Phase F placeholder}",
  "blast_radius_outcome": 0,
  "superseded_by_retro": true,

  "actual_hours": 0,
  "regressions_caught": {"count": 0, "descriptions": []},
  "escaped_issues": {"count": 0, "descriptions": [], "blast_radii": []},

  "accepted_with_failure": false,
  "elevation_offered": false,
  "elevation_fired_by": "user-flag | escaped-issue-medium-or-high | none",
  "elevation_outcome": "written | cancelled | not-offered",
  "decision_log_links": [{"date": "YYYY-MM-DD", "title": "..."}],
  "scope_change_log": [{"date": "YYYY-MM-DD", "title": "..."}],

  "addenda": [
    {"date": "YYYY-MM-DD", "kind": "escaped-issue | note", "description": "...", "blast_radius": "low | medium | high"}
  ],

  "prd_anchor": "{features/NNN-slug/NNN-slug-prd.md#R{N}}"
}
```

## Open questions

- **Lock-file cleanup on crashed orchestration.** R14's 30-minute stale-lock convention is a heuristic. Open question: should the skill ship a `/specflow:complete --clear-stale-locks` admin verb for explicit cleanup, or is the time-based heuristic sufficient? Recommend the time-based heuristic for v1; revisit if consumers report stale-lock false-positives.
- **`actual_hours` user-declined-to-estimate sentinel.** AC-3 uses `0` with the convention that `0` means declined. Open question: does `0` collide with legitimate sub-hour tasks (e.g. a 20-minute typo fix where the user honestly reports `0.3`)? Recommend documenting `0` exactly as "declined" with `null` reserved for "not yet captured" and any positive number (including fractional) as "captured value." Confirm at implementation.
- **`scope_change_log` Related-field parsing.** R13 / AC-13 reads `admin/decision-log.md` for entries whose Related field cites the task-id. Open question: what's the canonical task-id citation format in Related fields (e.g. `T-2026-04-T2`, `{NNN-slug}-T{N}`, prose mention like "task T2 of feature 002-develop-skill")? `specflow:decision`'s SKILL.md doesn't enforce a citation format. Recommend the skill match on substring with both formats accepted; document the preferred format in `specflow:decision`'s SKILL.md as a follow-up enhancement. Confirm at implementation.

## See also

- Interview: [`./003-complete-skill-interview.md`](./003-complete-skill-interview.md)
- PRD spec: [`../../../../docs/PRD.md`](../../../../docs/PRD.md) § "Phase 3 — Memory and self-learning" + Appendix I (especially I2, I3, I4, I8)
- Sister Phase 3 skills: `skills/decision/SKILL.md` (lightweight `decision-log.md` writer; pattern this skill mirrors for elevation), `skills/scope-change/SKILL.md` (mid-development scope capture; produces decision-log entries this skill cites in `scope_change_log`)
- Adjacent Phase 2 skill: `skills/develop/SKILL.md` (Phase F.5 writes the development-time fields this skill supersedes; auto-fires this skill at the F.5 → F.6 boundary)
- Downstream consumers: `/insights` (Phase 3 scope item 10 — reads the corpus this skill writes); `specflow:task` + `specflow:misc` similarity-search (Phase 3 scope item 5)
- Tasks: `003-complete-skill-tasks.md` (generated by `specflow:task` against this PRD when Phase 3 build begins)
- Tests: `003-complete-skill-test.md` (generated by `specflow:test` once tasks close Gate 3)
