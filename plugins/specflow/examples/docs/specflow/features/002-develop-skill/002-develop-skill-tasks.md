---
feature: 002-develop-skill
status: draft
created: 2026-05-06
requires:
  - ./002-develop-skill-prd.md
  - ./002-develop-skill-interview.md
  - ./debate-log/prd-gate2/manifest.md
produces:
  - ./debate-log/develop-gate4/manifest.md
  - ./debate-log/develop-gate5/manifest.md
  - ../../admin/task-history.json
  - ../../admin/scratch/{NNN-slug}-develop/
prd: ./002-develop-skill-prd.md
interview: ./002-develop-skill-interview.md
gate3: ./debate-log/tasks-gate3/manifest.md
---

# Tasks — specflow:develop (lane-based implementation orchestrator)

## Coverage matrix

| PRD requirement | Tasks satisfying it |
|---|---|
| R1 — Pre-flight chain check (refuse if Gate 3 missing/failed) | T1 |
| R2 — Per-feature + per-task invocation walk | T2 |
| R3 — Four-axis lane triage tuple (AI-judged + mechanical + rule-based) | T3 |
| R4 — Monotonic-downgrade precedence; confidentiality Red is non-overridable | T4 |
| R5 — Reviewer-driven lane re-check (catch-all) at Gate 4 | T5 |
| R5.1 — Mechanical pre-Gate-4 lane recheck (file-count, modules, glob re-match) | T5b |
| R6 — Green-lane batching policy + configurable cap | T6 |
| R7 — Single batched human sign-off; per-task gate failures pause batch | T7 |
| R8 — Agent-teams plugin consultation + present-but-failing fallback | T8 |
| R9 — Red lane never spawns a team | T9 |
| R10 — Gate 4 reviewer set (standard five with lens emphasis) | T10 |
| R11 — Codex invoked at Gate 5 only (not Gate 4) | T11 |
| R12 — Gate 5 always fires; Codex-absent degraded path with manifest note | T12 |
| R13 — Codex-only findings auto-promote via specflow:misc | T13 |
| R14 — Linear status transitions (three transitions; never auto-Done) | T14 |
| R15 — Verifier rejection: structured payload + four user options | T15 |
| R16 — Orchestrator pattern compliance (forks, scratch, command sub) | T16 |
| R17 — PRD-anchor format on every per-task plan | T17 |

Forward coverage: 18/18 PRD requirements covered (R1-R17 plus R5.1). Reverse traceability: all 19 tasks anchor to ≥1 requirement.

## Tasks

### T1 — Pre-flight chain check + refuse on missing or failed Gate 3

- **Anchor:** PRD R1 — *read tasks/PRD/interview/Gate 3 manifest; refuse if Gate 3 missing or `failed`.*
- **Scope:** `skills/develop/SKILL.md` Phase A pre-flight; chain-existence check on `features/{NNN-slug}/{NNN-slug}-tasks.md`, `{NNN-slug}-prd.md`, `{NNN-slug}-interview.md`, `debate-log/tasks-gate3/manifest.md`; status parser for the manifest's closing decision.
- **Acceptance:** Invoking `specflow:develop {NNN-slug}` on a feature whose Gate 3 manifest closing decision is `passed`, `passed-with-revisions`, or `passed-with-escalations` proceeds to Phase A.2 lane triage. Invoking on a feature whose Gate 3 manifest closing decision is `failed`, OR whose manifest file does not exist, exits non-zero with the literal sentinel *"Gate 3 not closed for `{NNN-slug}` (status: `{status|missing}`). Resolve Gate 3 first via `specflow:task {NNN-slug}`."* and writes zero artefacts under `features/{NNN-slug}/debate-log/develop-gate4/` or `develop-gate5/`. (Verifies AC-1.)
- **Depends on:** none.
- **Notes:** Match the abort-message style used by `skills/task/SKILL.md` Phase A. Tasks not closed at Gate 3 are imported-problem candidates per the Phase 1 task skill's pre-flight rule.

### T2 — Wire per-feature batch walk + per-task `--task` override

- **Anchor:** PRD R2 — *`specflow:develop {NNN-slug}` walks Green batch → Yellow one-at-a-time → Red surface; `--task T{N}` runs one task regardless of lane.*
- **Scope:** `skills/develop/SKILL.md` argument parser; per-feature walk loop (Green-batch first, Yellow sequential second, Red surface third); `--task` flag short-circuit.
- **Acceptance:** Invoking `specflow:develop {NNN-slug}` on a 6-task feature (3 Green-qualifying with satisfied deps, 2 Yellow, 1 Red) executes the 3 Green tasks as one batch, then walks the 2 Yellow tasks one at a time with HITL pairing, then surfaces the 1 Red task to the user without auto-executing. Invoking `specflow:develop {NNN-slug} --task T4` on the same feature runs exactly task T4 regardless of its lane and ignores the other 5 tasks. Invoking with neither `{NNN-slug}` nor `--task` prompts for a feature ID and refuses to proceed without one. (Verifies AC-2.)
- **Depends on:** T1, T3.
- **Notes:** Argument resolution mirrors `skills/task/SKILL.md` Phase A. The walk order (Green → Yellow → Red) is fixed; reordering is a v2 concern.

### T3 — Lane triage tuple records all four axes with cited evidence

- **Anchor:** PRD R3 — *(verifiability + blast radius + dependency state + confidentiality); first two AI-judged with cited evidence, third mechanical, fourth rule-based.*
- **Scope:** `skills/develop/SKILL.md` Phase B lane-triage step; per-task plan writer (four-line tuple); citation formatter; `task-history.json` reader (for upstream task `status: shipped` check); `admin/config.json.confidentialPaths` glob matcher.
- **Acceptance:** For every per-task plan emitted, the plan body contains exactly four lines in this order: (a) `verifiability: {high|medium|low} (cited: {scope file or acceptance line})`, (b) `blast radius: {high|medium|low} (cited: {scope file or acceptance line})`, (c) `dependency state: {satisfied|blocked} (cited: tasks file Depends-on field + task-history.json status)`, (d) `confidentiality: {non-confidential|confidential} (cited: config.json.confidentialPaths match or no-match)`. Any plan missing one of the four lines, or whose verifiability/blast-radius citations do not resolve to a real scope file or acceptance line, is a failed plan emission and Gate 4 must not be invoked. The lane triage record is also written to `admin/scratch/{NNN-slug}-develop/lane-assignments.json` with one entry per task. (Verifies AC-3.)
- **Depends on:** T1.
- **Notes:** Confidentiality classification is rule-based via path-glob match — never AI-rated. Verifiability and blast-radius use a small enum (high/medium/low) so the AI judgement is structured, not free-form.

### T4 — Monotonic-downgrade precedence; confidentiality Red is non-overridable

- **Anchor:** PRD R4 — *any weak axis → at least Yellow; any Red-qualifying → Red; confidentiality Red cannot be overridden.*
- **Scope:** `skills/develop/SKILL.md` Phase B lane-resolver; precedence table; user-override refusal path.
- **Acceptance:** Given a four-axis tuple, the lane-resolver assigns Green only when all four axes are Green-qualifying; Yellow when at least one axis is weak (any axis with verifiability=medium, blast-radius=medium, dependency-state=blocked, or confidentiality=non-confidential is acceptable for Green only when no other axis is weak); Red when at least one axis is Red-qualifying (verifiability=low OR blast-radius=high OR dependency-state=blocked-on-non-shipped-upstream OR confidentiality=confidential). When confidentiality=confidential, the lane is `red` regardless of the other three axes' values; a user request mid-run to downgrade the lane to Yellow or Green on a confidential-path task is refused with the literal sentinel *"Confidential-path lane is non-overridable. Resolve the path classification or run the task in Red."* (Verifies AC-4.)
- **Depends on:** T3.
- **Notes:** Lanes monotonically downgrade — only upgrade-on-recheck is allowed (T5b), never downgrade-on-user-judgement. Per `docs/PRD.md` Phase 2 scope item 1 ("no issue escapes Red because the AI feels confident").

### T5 — Reviewer-driven lane re-check (catch-all) at Gate 4

- **Anchor:** PRD R5 — *Gate 4 reviewer fires `block` finding naming the lane assignment; AI accepts in Round 2; plan re-emitted with corrected lane and a fresh Gate 4 manifest opens before code is written.*
- **Scope:** `skills/develop/SKILL.md` Phase C Gate 4 closer; re-lane-on-block detector (matches a Round 1 `block` finding whose `claim` field references the lane assignment); plan re-emitter; second Gate 4 manifest opener; first Gate 4 manifest closing-decision recorder (status `re-lane forced`).
- **Acceptance:** When a Gate 4 round-1 finding has severity `block` AND the finding's `claim` field references the lane assignment as too aggressive AND the AI accepts the finding in Round 2, the original Gate 4 manifest's closing decision records `re-lane forced` with the old → new lane recorded as a literal pair (e.g. `green → yellow`); a fresh Gate 4 manifest at the same `develop-gate4/` path is opened against the re-emitted plan; the second Gate 4 manifest's closing decision records `passed`, `passed-with-revisions`, `passed-with-escalations`, or `failed` per the standard taxonomy. A run that produces only one Gate 4 manifest after a `block`-and-accepted lane finding is a failed run. (Verifies AC-8 part (b).)
- **Depends on:** T3, T4, T10.
- **Notes:** R5 is the catch-all path — reviewer judgement catches what the mechanical recheck T5b cannot. Per `interview Round 4` (edited): *"Gate 4 reviewers can flag and force a re-lane before code is written."* This task is intentionally separate from T5b per the Gate 2 dogfood (block tbc-r1-f1) — the mechanical and reviewer paths are not merged.
- **gate2-revision:** tbc-r1-f1

### T5b — Mechanical pre-Gate-4 lane recheck (file-count, modules, confidential-path glob)

- **Anchor:** PRD R5.1 — *before Gate 4 manifest opens, mechanically re-evaluate the four-axis tuple given the plan's emitted content; downgrade the lane on any breach; re-emit the plan before Gate 4 opens.*
- **Scope:** `skills/develop/SKILL.md` Phase B.1 (between lane triage and Gate 4); file-count comparator (plan's actual file list vs scope's claimed file count, ratio threshold 1.5x); module comparator (plan's listed modules vs task's `Scope` modules); `config.json.confidentialPaths` glob re-runner against the plan's listed file paths; lane-downgrade applier (one step per breach); plan re-emitter on downgrade.
- **Acceptance:** Before any Gate 4 manifest opens for a task, the skill writes `admin/scratch/{NNN-slug}-develop/lane-recheck-{task-id}.json` containing exactly three numeric/boolean fields: (a) `file_count_plan_vs_scope: {plan_files: N, scope_claimed: M, ratio: N/M, downgraded: bool}`, (b) `modules_plan_vs_scope: {plan_modules: [...], scope_modules: [...], new_modules: [...], downgraded: bool}`, (c) `confidential_path_match: {plan_paths_matching: [...], forced_red: bool}`. If `file_count_plan_vs_scope.ratio > 1.5` OR `modules_plan_vs_scope.new_modules` is non-empty, the lane is downgraded one step (green→yellow OR yellow→red) and the file records the literal old → new pair. If `confidential_path_match.forced_red` is true, the lane is forced to `red` and the file records `forced_red: true`. When any field's `downgraded` or `forced_red` is true, the plan is re-emitted with the corrected lane before Gate 4 opens; a run that opens Gate 4 against the original plan after a mechanical-recheck downgrade is a failed run. The recheck runs unconditionally for every task; a missing `lane-recheck-{task-id}.json` for any task is a failed run. (Verifies AC-8 part (a).)
- **Depends on:** T3, T4.
- **Notes:** The mechanical recheck is structured (file-count + glob match) — no reviewer judgement involved. This is the load-bearing addition from Gate 2 dogfood (block tbc-r1-f1) — without it, the lane-aggressive-flag mechanism R5 + AC-8 promised would not have fired in practice.
- **gate2-revision:** tbc-r1-f1

### T6 — Green-lane batching policy with configurable cap

- **Anchor:** PRD R6 — *Green batch = all green-qualifying tasks whose deps are satisfied; configurable via `config.json.develop.greenBatchCap` defaulting to 3.*
- **Scope:** `skills/develop/SKILL.md` Phase D Green-lane batcher; dependency-resolution walker (`Depends on:` chain → shipped tasks OR other tasks in same batch); `config.json.develop.greenBatchCap` reader with default `3`; batch-size enforcer.
- **Acceptance:** On a feature with 5 Green-qualifying tasks where T1's deps resolve to shipped, T2's deps resolve to T1 (in-batch), T3's deps resolve to T1, T4's deps resolve to a non-shipped Yellow task (blocked), and T5's deps resolve to shipped: the Green batch contains exactly {T1, T2, T3, T5} (not T4 — its dep is non-shipped and out of batch). With `config.json.develop.greenBatchCap = 3`, the batch caps at the first three by task-ID order ({T1, T2, T3}); T5 lands in the next batch. With the cap absent, the default `3` applies. With the cap explicitly set to `5`, the batch is {T1, T2, T3, T5}. (Verifies AC-5 in part with T7.)
- **Depends on:** T2, T3.
- **Notes:** The cap default `3` is the user-confirmed value from interview Round 2 (lowered from AI's recommended 5). Green-batch dependency resolution treats in-batch tasks as virtually-shipped for the purpose of unblocking other in-batch tasks.

### T7 — Single batched human sign-off + per-task gate failure pauses batch

- **Anchor:** PRD R7 — *single human sign-off after the assembled PR diff; per-task gate failure pauses the batch at that boundary.*
- **Scope:** `skills/develop/SKILL.md` Phase D Green-lane PR assembler; per-task gate-outcome summariser (Gate 4 closing decision, Gate 5 closing decision, Verifier pass/fail); single sign-off prompt; on-failure pause handler (resume/abort/drop options).
- **Acceptance:** A successful Green batch run produces exactly one PR description containing a per-task gate-outcome summary table for every task in the batch in task-ID order, with columns `task-id | Gate 4 | Gate 5 | Verifier`, followed by exactly one sign-off prompt: *"Sign off the assembled batch ({N} tasks) to merge the PR."*. If any per-task Gate 4, Gate 5, or Verifier check fails during the batch, the skill stops at that task's boundary, writes a partial PR description containing only the tasks completed up to and including the failing task, and prompts the user with exactly three options: `resume` (retry from the failing task), `abort` (drop the entire partial batch), or `drop` (drop the failing task and continue with the remainder). The skill must not proceed without an explicit user choice; auto-default is a failed run. (Verifies AC-5.)
- **Depends on:** T6 always; T10 and T12 invoked per-task inside the batch loop (Gate 4 fires before agent execution, Gate 5 fires after).
- **Notes:** Per-task gates fire individually inside the batch; the batched sign-off is the AFK-eligibility win. The three-option pause is the explicit-user-choice surface — silent retry is the failure mode this skill exists to remove. The failure-path pause handler must inspect three independent per-task signals (Gate 4 closing decision, Gate 5 closing decision, Verifier pass/fail) and pause on the first failure, not wait for the batch to finish. The signal source is the manifest closing-decision line at each gate — read after each gate closes, not at end-of-batch.

### T8 — Agent-teams plugin consultation + present-but-failing fallback

- **Anchor:** PRD R8 — *consult the plugin once per task at lane-assignment; team-spawn fires only when (lane Green or Yellow) AND (task scope matches `config.json.teams[].scope` glob) AND (plugin detected); present-but-failing path falls back to single-specialist with a chat-line warning.*
- **Scope:** `skills/develop/SKILL.md` Phase B agent-teams consultation step; plugin detection via `admin/environment.json.plugins[]`; `config.json.teams[].scope` glob matcher; `team-spawn` invocation with matching preset; error handler for `team-spawn` failure (preset unmatched, plugin version-skew, internal error); single-specialist fallback path; chat-line warning surfacer.
- **Acceptance:** When `admin/environment.json.plugins` includes `agent-teams` AND a task's lane is Green or Yellow (NOT Red) AND the task's scope matches a `config.json.teams[].scope` glob, the per-task plan logs the literal line `team-spawn invoked: preset={preset-name}, members={member-list}`. When any of those conditions is false, the per-task plan logs the literal line `single-specialist invoked: agent={agent-name}, stack-match={glob}`. When `team-spawn` is invoked but returns an error, the per-task plan logs the literal line `team-spawn failed: {error}; fell back to single-specialist invoked: agent={agent-name}` AND the skill surfaces a chat-line warning of the form `[agent-teams: team-spawn failed for preset {name} — falling back to single-specialist {agent}]`. The fallback does not abort the task. (Verifies AC-6 in part with T9.)
- **Depends on:** T3.
- **Notes:** Loud-fallback shape matches Phase 1 `specflow:design` Codex-degraded-path precedent. Per Gate 2 dogfood (concern da-r1-f1).
- **gate2-revision:** da-r1-f1

### T9 — Red-lane never spawns a team

- **Anchor:** PRD R9 — *Red-lane tasks always pick a single specialised agent regardless of `config.json.teams` configuration.*
- **Scope:** `skills/develop/SKILL.md` Phase B agent-teams consultation step (Red-lane short-circuit); single-specialist picker for Red-lane tasks.
- **Acceptance:** For every Red-lane task in any run, the per-task plan logs the literal line `single-specialist invoked: agent={agent-name}, stack-match={glob}` and never logs `team-spawn invoked`, regardless of whether `admin/environment.json.plugins` includes `agent-teams` and regardless of whether the task scope matches a `config.json.teams[].scope` glob. A Red-lane plan that logs `team-spawn invoked` is a failed run. (Verifies AC-6 in part with T8.)
- **Depends on:** T4, T8.
- **Notes:** Multi-agent composition would defeat Red's "AI assists on bounded subtasks only" safety property. Red-lane safety is non-negotiable per `docs/PRD.md` Phase 2 scope item 1. T9 short-circuits T8's plugin-consultation step entirely for Red-lane tasks; T8's failure-fallback path is unreachable on Red-lane.

### T10 — Gate 4 reviewer set is the standard five (no Codex)

- **Anchor:** PRD R10 + R11 — *Gate 4 uses devils-advocate + simplicity + surgical + think-before-coding + goal-driven; Codex does not join Gate 4 in v1.*
- **Scope:** `skills/develop/SKILL.md` Phase C Gate 4 reviewer set; manifest header writer (`Reviewers:` line); Codex-Gate-4 refusal (no opt-in flag in v1).
- **Acceptance:** For every Gate 4 manifest written under `features/{NNN-slug}/debate-log/develop-gate4/manifest.md`, the `Reviewers:` line contains exactly the literal string `devils-advocate, simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer` (in that order). Codex does not appear in the Gate 4 `Reviewers:` line, regardless of `admin/environment.json.cli.codex.available`. A Gate 4 manifest whose `Reviewers:` line lists more or fewer than these five reviewers, or includes Codex, is a failed run. (Verifies AC-7.)
- **Depends on:** T1.
- **Notes:** Lens emphasis at Gate 4: Think-Before-Coding is load-bearing (plan-level unstated assumptions); Goal-Driven cross-checks the PRD anchor against the cited requirement ID; Surgical flags scope drift between task `Scope` and plan file paths. The lens-emphasis ordering is documented in the SKILL.md body, not the manifest.

### T11 — Codex invoked at Gate 5 only (not Gate 4)

- **Anchor:** PRD R11 — *Codex hard-coded to Gate 5 only in v1; no `config.json.develop.codexAtGate4` knob.*
- **Scope:** `skills/develop/SKILL.md` Codex invocation step (Phase E Gate 5 only); refusal of any opt-in for Codex at Gate 4.
- **Acceptance:** In any run with `admin/environment.json.cli.codex.available: true`, Codex is invoked exactly once per task per Gate 5 manifest and never during Gate 4. The Gate 4 manifest's `Reviewers:` line never contains the string `codex` (verified by T10). The skill exposes no configuration knob `config.json.develop.codexAtGate4`. Reading the field returns undefined; setting the field via config edit produces no observable change in the Gate 4 manifest `Reviewers:` line (verified by re-running T10 acceptance with the field set). (Verifies AC-7 in part with T10.)
- **Depends on:** T10.
- **Notes:** Per Gate 2 dogfood (concern simplicity-r1-f1) — the speculative `codexAtGate4` knob was dropped because no documented consumer asked for it. Consumers wanting Gate-4 Codex review request it as a v2 enhancement. AC-7 verification requires T10 AND T11 together: T10 verifies the literal `Reviewers:` line, T11 verifies that no Codex opt-in flag exists in v1 (setting the flag produces no observable change).

### T12 — Gate 5 always fires; Codex-absent degraded path with explicit manifest note

- **Anchor:** PRD R12 — *Gate 5 always fires (never skipped); when Codex absent, standard five-reviewer set runs with explicit manifest note declaring degraded coverage.*
- **Scope:** `skills/develop/SKILL.md` Phase E Gate 5 reviewer set; Codex-presence detector (`admin/environment.json.cli.codex.available`); manifest writer (Codex section vs Codex-absent sentinel); Goal-Driven Reviewer load-bearing-at-Gate-5 emphasis.
- **Acceptance:** Every task whose code change set landed has a Gate 5 manifest at `features/{NNN-slug}/debate-log/develop-gate5/manifest.md`. When `admin/environment.json.cli.codex.available: false` (or the field is absent), the manifest's `Codex` section contains the literal sentinel *"Codex not detected — same-provider review only. Cross-provider findings may be missed; install Codex CLI for full Gate 5 coverage."* and the `Reviewers:` line lists the standard five reviewers. When `admin/environment.json.cli.codex.available: true`, the manifest's `Codex` section captures Codex's findings under a sub-heading and the `Reviewers:` line lists the standard five plus `codex`. A run that ships code without a Gate 5 manifest, or whose Codex-absent manifest omits the literal sentinel, is a failed run. (Verifies AC-9.)
- **Depends on:** T10, T11.
- **Notes:** Devil's Advocate runs as part of the standard set; the manifest note (not DA) is the authoritative framing of degraded coverage — same-provider DA cannot cover cross-provider concerns by definition. Per Gate 2 dogfood (concern tbc-r1-f2).

### T13 — Codex-only findings auto-promote via `specflow:misc --auto`

- **Anchor:** PRD R13 — *findings Codex catches at Gate 5 that the five other reviewers missed get auto-promoted via `misc-task` proposing the rule; skill never writes `guidelines.md` directly.*
- **Scope:** `skills/develop/SKILL.md` Phase E Gate 5 Codex-only-finding extractor (Codex round-1 finding ID present, no overlapping `claim` in any other reviewer's round-1 finding); `specflow:misc --auto` invocation with the named-field payload (`manifest_path`, `gate_finding_id`, proposed rule text); refusal of direct `guidelines.md` writes.
- **Acceptance:** When Codex's Gate 5 round-1 file contains a finding ID with no overlapping `claim` field in any other Gate 5 reviewer's round-1 file, the skill invokes `specflow:misc --auto` with a payload containing exactly three named fields: `manifest_path: features/{NNN-slug}/debate-log/develop-gate5/manifest.md`, `gate_finding_id: codex-r1-f{N}`, and `proposed_rule: {rule text derived from the finding's evidence + claim}`. The misc-task entry references the Gate 5 manifest path and the Codex finding ID; verification: `admin/misc-tasks.json` (or whichever path `specflow:misc` writes to per its v1 schema) contains a new entry with these three fields populated. The skill must not write to `admin/rules/guidelines.md` directly during the Gate 5 close — a run that edits `guidelines.md` from `specflow:develop` is a failed run. **Schema-gap branch:** when `specflow:misc --auto`'s schema does not accept `manifest_path` and `gate_finding_id` as named fields (detected by reading `skills/misc/SKILL.md` for the documented payload schema), the skill skips the auto-promotion AND surfaces a chat-line warning of the form `[develop: codex-only finding {id} not auto-promoted — specflow:misc schema gap; see AC-10 schema-dependency clause]`. The skip-with-warning path is recorded in the Gate 5 manifest as `auto_promotion_skipped: schema_gap`. A run that silently swallows the codex-only finding without the chat-line warning is a failed run. (Verifies AC-10.)
- **Depends on:** T12.
- **Notes:** Per AC-10 schema-dependency clause: this task assumes `specflow:misc --auto` accepts `manifest_path` and `gate_finding_id` as named fields. If the current Phase 1 schema does not include these, ship them via a separate `specflow:misc` enhancement PRD before `specflow:develop` v1 lands. See `skills/misc/SKILL.md` for the current schema.

### T14 — Linear status transitions (three transitions; never auto-Done)

- **Anchor:** PRD R14 — *(a) Backlog/Todo → In Progress on invocation; (b) In Progress → In Review on PR open; (c) NEVER auto-Done; absent MCP → chat-only line.*
- **Scope:** `skills/develop/SKILL.md` Phase A Linear-MCP detector (`admin/environment.json.mcp.linear.available`); pre-Step-1 transition firer (Backlog/Todo → In Progress); post-Verifier+PR-open transition firer (In Progress → In Review); auto-Done refusal; absent-MCP chat-only fallback.
- **Acceptance:** When `admin/environment.json.mcp.linear.available: true` AND the task's Linear status is Backlog or Todo, the skill fires a Linear status transition to "In Progress" before Phase B context primer. When Verifier passes AND the per-task PR opens, the skill fires a Linear status transition to "In Review". The skill never fires a transition to "Done" — a run that fires a Done transition from `specflow:develop` is a failed run. When `admin/environment.json.mcp.linear.available: false` (or absent), the skill prints the chat-only line `[linear status: T{N} → In Progress (skipped — MCP not available)]` (and the equivalent line for In Review) and continues. (Verifies AC-11.)
- **Depends on:** T1.
- **Notes:** Auto-invocation when the user moves a task to In Progress in Linear (per the SKILL.md trigger list) lands the skill at Phase B of the same flow. See `skills/linear/SKILL.md` for the underlying MCP wrapper; this task wires the transition firers, not the MCP itself.

### T15 — Verifier rejection: structured failure payload + four user-elected options

- **Anchor:** PRD R15 — *write structured failure payload; user picks from four options; skill never auto-loops.*
- **Scope:** `skills/develop/SKILL.md` Phase F Verifier-rejection handler; structured failure payload writer (`admin/scratch/{NNN-slug}-develop/verifier-failure-{task-id}.json`); four-option prompt; option-1 (re-implement) loop-back to Phase D with payload as additional context; option-2 (re-plan) loop-back to Phase C (re-fires Gate 4); option-3 (abort) handoff to `specflow:scope-change`; option-4 (accept-and-ship) `decision-log.md` writer.
- **Acceptance:** When Verifier rejects a task, the skill writes `admin/scratch/{NNN-slug}-develop/verifier-failure-{task-id}.json` containing exactly four named fields: `failed_ac: AC-{N}`, `verification_check_finding: {summary}`, `diff_section: {file:line-range}`, `agent_name: {agent-id}`. The skill then prompts the user with exactly four options labelled `1`, `2`, `3`, `4` matching the PRD R15 list. The skill must not proceed without an explicit user choice; an empty input or any input not matching `1|2|3|4` re-prompts. Option-4 (accept-and-ship) writes a `decision-log.md` entry capturing the AC, the failure summary, and the user's stated reason for shipping. The skill must not silently re-implement — a run that loops back to Phase D after a Verifier rejection without user election of option-1 is a failed run. (Verifies AC-12.)
- **Depends on:** T2.
- **Notes:** Auto-loop-on-rejection is a known failure mode this skill exists to remove. See `skills/scope-change/SKILL.md` for the option-3 handoff target.

### T16 — Orchestrator pattern compliance (forks, scratch, command substitution)

- **Anchor:** PRD R16 — *every sub-step runs in a forked sub-agent; verbatim file content via command substitution; intermediate artefacts in `admin/scratch/{NNN-slug}-develop/`; parent context ≤2K tokens per gate manifest, ≤30K tokens for a 10-task feature end-to-end.*
- **Scope:** `skills/develop/SKILL.md` all phases (A through F) — every reviewer dispatch, every plan generation, every agent execution, every Verifier run; scratch-directory convention `admin/scratch/{NNN-slug}-develop/`; command-substitution pattern `!{cat ...}` for file injection.
- **Acceptance:** End-to-end on a 10-task feature with the agent-teams plugin and Codex CLI both available, running `specflow:budget --skill specflow:develop {NNN-slug}` after the run produces a JSON report with two numeric fields `parent_context_tokens: N` (where N < 30000) and `max_per_gate_growth_tokens: M` (where M < 2000); a report whose `N >= 30000` or `M >= 2000` is a failed run. The scratch directory `admin/scratch/{NNN-slug}-develop/` exists during the run and is cleaned up on successful close (or retained on failure for debugging). Every reviewer dispatch in Gate 4 and Gate 5 forks a sub-agent context (no Read tool calls in the parent for reviewer inputs); a run whose parent context contains a Read-tool response with a reviewer's full output is a failed run. (Verifies AC-14.)
- **Depends on:** T10, T12.
- **Notes:** The pattern is inherited from `templates/orchestrator-pattern.md`. Calibration anchor: ~5K tokens parent context, matching `skills/prd/SKILL.md` and `skills/task/SKILL.md`.

### T17 — PRD-anchor format on every per-task plan's first paragraph

- **Anchor:** PRD R17 — *every per-task plan's first paragraph follows `"We're doing X because of PRD requirement Y. This aligns with Z."`; X paraphrases task scope, Y is a literal `R{N}`, Z paraphrases `Serves goal:`; non-conforming plans are a failed plan emission.*
- **Scope:** `skills/develop/SKILL.md` Phase D plan emitter; first-paragraph format validator (regex match on the literal preamble + `R{N}` reference + `Serves goal:` paraphrase); pre-Gate-4 refusal on non-conforming plans.
- **Acceptance:** For every per-task plan emitted under any feature, the first paragraph matches the regex `We're doing .+ because of PRD requirement R\d+(\.\d+)?\. This aligns with .+\.` (case-sensitive, single line). The literal `R{N}` reference resolves to a real requirement in the feature's PRD; a plan whose `R{N}` reference does not resolve to a PRD requirement is a failed plan emission. A plan whose first paragraph does not match the regex is a failed plan emission and Gate 4 must not be invoked. The PR description for every per-task PR opens with the same first paragraph verbatim (so PR readers see the PRD anchor without invoking the skill). (Verifies AC-13.)
- **Depends on:** T2, T3.
- **Notes:** Closes Gate 2 dogfood block (goal-r1-f1) — the architectural Phase 2 scope item 7 rule now has R-level expression in this PRD body and a covering task. The PR-description-mirrors-plan rule serves PRD goal Audience surface (tertiary stakeholders).
- **gate2-revision:** goal-r1-f1

### T18 — Append development-time fields to `task-history.json` on completion

- **Anchor:** PRD AC-15 / Goal Outcome surface (h) — *on task completion (Verifier passes OR option-4 accept-and-ship), append/update `admin/task-history.json` entry with `lane_assigned`, `ai_assistance_level`, `elapsed_minutes`, `what_worked`, `what_didnt_work`, `blast_radius_outcome`.*
- **Scope:** `skills/develop/SKILL.md` Phase F task-history writer; `task-history.json` appender (creates entry by id if absent, updates if present); six-field schema enforcer.
- **Acceptance:** On Verifier pass OR option-4 accept-and-ship (per T15), the skill reads `admin/task-history.json`, locates the entry with id `{NNN-slug}-T{N}` (or creates one if absent), and writes exactly six development-time fields: `lane_assigned: green|yellow|red`, `ai_assistance_level: {full|partial|bounded-subtasks}`, `elapsed_minutes: {integer}`, `what_worked: {string}`, `what_didnt_work: {string}`, `blast_radius_outcome: {actual files touched count}`. A task whose `task-history.json` entry is missing any of the six fields after Verifier pass is a failed run. The schema follows `docs/PRD.md` Phase 3 scope item 1 + Appendix I3. (Verifies AC-15.)
- **Depends on:** T15.
- **Notes:** Pre-existing entries (created at task creation by `specflow:task` Phase D for user overrides) are updated in place; the entry's pre-task-creation fields (e.g. `ai_proposal`, `user_override`) are preserved.

## Open questions inherited from PRD

- **Q:** Re-lane handling when an upstream task in a Green batch is re-laned mid-batch — pause at the boundary or re-evaluate the batch composition? — affects: T6, T7. PRD recommends pause-and-ask; defer to implementation.
- **Q:** Linear status when the user picks option-4 accept-and-ship on Verifier rejection — does the In Progress → In Review transition still fire? — affects: T14, T15. PRD recommends fire-and-record; confirm at implementation.
- **Q:** `config.json.develop.greenBatchCap = 0` semantics — does 0 mean "no Green-lane batching, every task signs off individually"? — affects: T6, T7. PRD recommends yes; document in `config.json` schema at implementation.

## See also

- PRD: [`./002-develop-skill-prd.md`](./002-develop-skill-prd.md)
- Interview: [`./002-develop-skill-interview.md`](./002-develop-skill-interview.md)
- Gate 2 manifest: [`./debate-log/prd-gate2/manifest.md`](./debate-log/prd-gate2/manifest.md)
- Gate 3 manifest: [`./debate-log/tasks-gate3/manifest.md`](./debate-log/tasks-gate3/manifest.md)
- Tests: `./002-develop-skill-test.md` (generated by `specflow:test`)
