# Changelog

All notable changes to specflow v2 are documented here. Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versioning: [SemVer](https://semver.org/).

## [Unreleased]

_v2.11.0 adds light-mode coverage — `specflow:feature` auto-detects complexity at kickoff, writes `mode: light | full` to feature.md frontmatter, and downstream `specflow:prd` / `specflow:task` / `grill` skip the heavy review chain when `light`. Proportional ceremony: full pipeline for big features, lightweight pass for trivial changes. Closes the dogfood-time friction where a "remove an image" change ran through six reviewers + multi-agent debates._

## [2.11.0] — 2026-05-12

### Added

**Complexity-aware pipeline mode** — `mode: light | full` field in `{NNN-slug}-feature.md` frontmatter. Auto-detected by `specflow:feature` at kickoff; user confirms or overrides at the Phase C reflection step. Downstream skills read the mode at Phase A entry and skip their heavy review chain when `light`.

**Detection heuristic** (`specflow:feature` Phase C.1):

- Light signals: verbs like *remove / rename / tweak / swap / hide / update copy*; acceptance shape *"X is hidden" / "X reads Y"*; no content in `assets/` or `design/`; one-paragraph goal.
- Full signals: verbs like *add / build / integrate / support / introduce*; acceptance shape *"X works with Y under conditions Z"*; reference content present; two-paragraph goal.
- Mixed: prefer `full`; flag the ambiguity in the reflection so the user can downgrade.

### Changed

**`specflow:prd` Phase A.0** — extended to read `mode:` from feature.md. When `mode: light`:

- Phase B (grilling) caps at 0-2 rounds (delegates to `grill` sub-skill's own mode-read).
- Phase B.5 (Codex adversarial pass) skipped; stub `pre-gate-codex.md` written.
- Phase D (Gate 2 multi-agent debate) skipped; stub manifest written with closing decision *"passed (light mode — no multi-agent review)"*.
- Phase C (PRD body synthesis) and Phase E (brief render) still run.

**`specflow:task`** — new A.0 step reads `mode:` from feature.md. When `mode: light`:

- Phase B.5 (Codex pass) skipped; stub `pre-gate-codex.md` written.
- Phase D (per-task reviewer multi-agent rounds) skipped.
- Phase E.4.5 (cross-task review) skipped.
- Phase E (Gate 3 multi-agent debate) skipped; stub manifest written.
- Phases B.1-B.4 (coverage matrix, self-checks for budget / duration / graph-validity), B.6 (sprint-bucket assignment), Phase C (intent summaries) all still run.

**`grill` sub-skill pre-flight** — adds step 6 (mode read). When `mode: light`, applies a 0-2 round cap; if no load-bearing question surfaces, writes a one-line round and proceeds to sign-off without asking anything. User can override at any round (*"switch to full"* / *"switch to light"*).

### Notes

- **Default behaviour preserved.** When feature.md is absent OR the `mode:` field is missing (pre-2.11.0 features), `MODE` defaults to `full` and every skill runs the legacy review chain unchanged.
- **Override semantics.** Mode is not locked. Users can switch at any phase by editing feature.md or by telling `grill` "switch to full/light". Downstream skills read the mode on every Phase A entry, so the next invocation picks up the change.
- **Scope.** v2.11.0 covers `feature` / `prd` / `task` / `grill` — the four skills the user hit friction on during the 2026-05-12 dogfood. `specflow:develop` / `specflow:test` retain their full reviewer flow; light-mode coverage there lands if/when friction is reported.
- **Not a replacement for `specflow:misc`.** Misc remains the no-PRD escape hatch for true one-offs (single bullet in `000-tasks-misc-tasks.md`). Light mode is the middle ground — full pipeline with proportional ceremony.

## [2.10.1] — 2026-05-12

### Added

**`specflow:prd` Phase A.3 — assets folder readback:**

- When `features/{NNN-slug}/assets/` contains files (other than `.gitkeep`), the PRD interview now ingests them. Globs for `*.yaml | *.yml | *.json | *.html | *.md | *.txt | *.csv`; reads up to 200 lines per file (truncation noted); distills 1-3 codebase-context bullets prefixed `Reference assets (from assets/):`.
- Binary files (images, PDFs, fonts) are noted by filename only — no content ingestion.
- Absent or `.gitkeep`-only `assets/` is skipped silently. Backward-compatible — features without reference assets are unaffected.

### Why

The v2.10.0 release scaffolded `assets/` at feature-kickoff time but `specflow:prd` only read from `design/`. Users dropping a YAML form spec or HTML reference into `assets/` expected the PRD generator to see it; today it doesn't. v2.10.1 wires it in so the AI flow benefits from reference materials, not just human readers.

## [2.10.0] — 2026-05-12

### Added

**New skill** (`plugins/specflow/skills/feature/`):

- `specflow:feature` — feature kickoff. Runs FIRST in the pipeline. Allocates the `NNN-slug`, scaffolds `design/`, `docs/`, `assets/`, `test/screenshots/`, `debate-log/` with `.gitkeep` in each, runs a four-question goal interview (headline goal / why now / who benefits / what does done look like), reflects what it heard, and writes `{NNN-slug}-feature.md` with the goal + open questions + folder index + status frontmatter. Four-phase orchestrator — A scaffold, B goal interview, C reflection + confirmation, D write meta file + handoff. No sub-agent forking, no multi-agent gate, no Codex pass; lightweight by design.

**New per-feature artefact** (`features/{NNN-slug}/{NNN-slug}-feature.md`):

- Frontmatter: `slug`, `status` (`kickoff | prd-pending | tasks-pending | development | test-pending | shipped`), `created`, `goal_locked`.
- Body: `## Goal` (≤2 paragraphs, locked at kickoff), `## Open questions raised at kickoff` (optional, ≤3 bullets), `## PRD fields implied at kickoff` (optional, ≤3 bullets), `## Folder index` (every standard artefact path), `## Status` (ladder doc).
- Slim by design — ~80 lines max. The folder index lives ONCE here; no per-folder READMEs (`.gitkeep` only).

### Changed

**Pipeline order** (corrected per user feedback):

```
specflow:setup → specflow:feature → specflow:prd → specflow:task → specflow:sprint → specflow:develop → specflow:test
```

`specflow:test` moves to last position (post-development) in the canonical doc. The testing-as-cadence design (`specflow:test` can also run mid-cycle for plan-only and per-task slices) is unchanged — only the canonical pipeline order in the README is updated.

**`specflow:prd` Phase A** — new A.0 step (Feature-skill handoff). Reads `{NNN-slug}-feature.md` when present. When found, skips A.1 (slug already allocated), A.2's mkdir is idempotent, and A.5 / A.6 / A.7 (goal articulate / confirm / write) are skipped — the goal is populated verbatim from feature.md into the interview file's Goal section. The PRD grilling questions cover decomposition only; strategic shape is already locked. After the normal hand-off point, bumps `{NNN-slug}-feature.md` status to `prd-pending`.

When feature.md is absent, A.1-A.7 run as before (the pre-v2.10 flow). `specflow:feature` is the recommended entry point but `specflow:prd` remains valid as a direct entry for quick PRDs.

**`specflow:setup` Phase 8.2** — `skills.feature.enabled: true` seeded in the default config.json. `skills.learn.enabled: true` also seeded (caught a v2.8.0 oversight where the learn-skill toggle was missing from the default seed).

### Notes

- **Goal changes after kickoff** are a direct edit to `{NNN-slug}-feature.md`'s `## Goal` section. Downstream skills pick up the new value on their next read. No new skill is required; `specflow:scope-change` remains reserved for PRD-shape changes, not goal changes.
- **Idempotent on existing features.** If `specflow:feature` is invoked on an existing feature folder without a meta file, it runs the goal interview and writes the meta file + scaffolds missing subfolders without disturbing existing artefacts. If the meta file already exists, it refuses with a documented sentinel (status `kickoff` → edit-directly hint; status beyond → goal-change-is-direct-edit hint).
- **Skill body size.** 306 lines — well under the 500-line ceiling. No bloat-room scope creep absorbed from adjacent concerns.

## [2.9.1] — 2026-05-12

### Changed

- `task.maxDurationMinutes` renamed to `task.maxDurationHours`. Options: `1 | 4 | 8` or `"auto"`; default `1`. Setup prompt simplified to a single line.
- `specflow:task` reads the value at synthesis time as a sizing target. No new task-block field; no new B.4 self-check step.

### Removed

- `estimated-duration-minutes` task-block field (added in v2.9.0; redundant — token-budget already drives enforcement).
- B.4 step 8 (Duration self-check) (added in v2.9.0; redundant).
- Setup's multi-paragraph duration prompt and the orthogonality explanation (added in v2.9.0; the simpler shape doesn't need them).

### Migration

- `specflow:upgrade` deletes any existing `task.maxDurationMinutes` key and writes `task.maxDurationHours: 1` if absent. `.bak` of `config.json` preserved.

## [2.9.0] — 2026-05-12

### Added

**New config key** (`admin/config.json`):

- `task.maxDurationMinutes` — integer minutes (`30 | 60 | 90 | 120`) or the string `"auto"`. Default `60`. Captured at setup time via a new 8.2 prompt; back-filled silently on upgrade. The cap is orthogonal to `task.contextBudget` — tokens cap the AI's context window, minutes cap human review burden. A task can pass one and fail the other.

**New task-block field** (`{NNN-slug}-tasks.md`):

- `estimated-duration-minutes` — positive integer; the AI's synthesis-time estimate of human-time to implement. Surfaces between `context-budget-estimate` and `sprint-bucket`. When `config.task.maxDurationMinutes == "auto"`, the field is informational only; otherwise it MUST be ≤ the configured cap or the task auto-flags for split via `specflow:scope-change`.

### Changed

**Skill body changes** (no new skills):

- `specflow:setup` Phase 8.2 — new prompt between the brief-commit-policy question and the `config.json` write. Five options (`1 → 30`, `2 → 60`, `3 → 90`, `4 → 120`, `5 → "auto"`); default `60` on skip. Advises smaller tasks are easier to manage. Sub-prompt rejects malformed input with a re-prompt rather than coercing.
- `specflow:task` Phase B.3 — task-block template adds `estimated-duration-minutes` field between `context-budget-estimate` and `sprint-bucket`.
- `specflow:task` Phase B.4 — adds step 8 (Duration self-check), parallel to step 5 (budget self-check). When `config.task.maxDurationMinutes` is an integer, every task's `estimated-duration-minutes` MUST be ≤ the cap or it auto-flags with an inline `> Duration overrun: estimate {N} mins vs cap {M} mins — consider splitting via specflow:scope-change.` warning and a chat-line prompt. When the config value is `"auto"`, the check is skipped entirely — the estimate is informational only.

### Notes

- **Orthogonality.** Duration and token budget are independent sizing dimensions. A task can pass tokens and fail minutes (long human review of a small AI payload) or vice versa (heavy AI context, quick human review). Both flag; both route to `specflow:scope-change` for splitting. The user picks both knobs at setup — the token budget defaults to `80000`, the duration defaults to `60`.
- **Auto semantics.** `"auto"` means "the system synthesises the estimate informationally but enforces nothing on human-time" — sizing authority falls back to the token-budget rule alone. It does NOT mean "the system picks a number for you to enforce against."
- **Upgrade behaviour.** Existing projects get `task.maxDurationMinutes: 60` back-filled silently into `admin/config.json` by `specflow:upgrade` (creating the key only when absent; never overwrites a user value). Historical task files are not retroactively rewritten — only new tasks created post-upgrade carry the `estimated-duration-minutes` field.

## [2.8.0] — 2026-05-11

### Added

**New skill** (`plugins/specflow/skills/learn/`):

- `specflow:learn` — repo-local self-learning loop. Consumes `docs/specflow/admin/plugin-findings.jsonl` (append-only structured corpus emitted by `specflow:test` or any producer); clusters deterministically by `signal_pattern` at a 3-observation threshold; auto-applies Tier-A additive rules under a per-run cap of 3. Five-phase orchestrator — A lock + corpus + registry read, B schema-validate + cluster, C tier-route (A repo-local / B plugin-level-logged / C conflict-logged), D Tier-A auto-apply with `.bak` before every write, E end-of-feature report + run log + lock release. No LLM-as-judge inside the loop. No sub-agent forking. Additive-only on every registry; existing rules and existing config keys are never mutated. The per-run cap of 3 is the safety valve for the burst case where a full-feature test run dumps 50+ findings in one shot.

**Tier-A destinations:**

- `admin/rules/guidelines.md` — new guideline blocks with `source_finding_ids` provenance frontmatter; conservative `paths: ["**/*"]` default the user narrows at edit-time.
- `admin/CONTEXT.md` — new "Known weak spots" entries; section header created on first write if missing.
- `admin/config.json` — new top-level keys only (the cluster's contributing findings must nominate `KEY=path; VALUE=<json>` for the write to land; otherwise the cluster demotes to Tier C).

**Tier-B / Tier-C logging:**

- `admin/scratch/plugin-candidates-{date}.md` — Tier-B clusters (categories `bug` or `architecture` — these touch the plugin itself, not the repo). Logged for manual review; the user routes plugin-repo issues from there.
- `admin/scratch/learn-conflicts-{date}.md` — Tier-C clusters (signal-pattern collides with an existing rule id; `proposed_fix` cardinality > 2; post-write verification failed; config key already set; config cluster missing key/value nomination).

**Reports:**

- `admin/learn/{feature_slug | "full-corpus"}-learn-{ts}.md` — six-section end-of-feature report (Auto-applied / Hit-threshold-but-blocked / Tier-B candidates / Tier-C conflicts / Below-threshold / System-learned). Closes the user-facing loop: read the diff in your next commit to review the auto-applied changes.
- `admin/learn/runs.jsonl` — append-only run log mirroring the `insights` and `optimize` corpus patterns.

### Notes

- **`specflow:test` is intentionally unchanged in this release.** The corpus (`plugin-findings.jsonl`) is producer-agnostic; the consumer is shippable on its own. The producer integration (emit one JSONL line per verification gap; invoke `specflow:learn` best-effort at end of Phase C) lands in a follow-up release once a real consumer-project test run grounds the sidecar schema against actual emitted signal — not against an imagined shape.
- **Independent of `insights`.** Both write to `admin/rules/guidelines.md` but on different cadences and from different corpora (`insights` mines `task-history.json` for project-domain lessons monthly; `specflow:learn` mines `plugin-findings.jsonl` for process-meta findings per-test). Conflict-detection at Phase C prevents id collisions.
- **Out of scope for v1.** Tier-B auto-PR machinery (plugin-level changes via `gh`) is intentionally deferred — the design conversation framed this loop as repo-specific, so plugin-touching changes stay manual.

## [2.7.1] — 2026-05-07

Hardening release driven by a thirteen-round adversarial review of the v2.7.0 ship surface. No new features. No public-contract changes. Every change tightens an existing invariant or closes a portability gap surfaced under adversarial pressure.

### Changed

**Branding compliance** (the project's non-negotiable vendor-neutrality rule):

- Neutralised vendor-name guardrail lines across 14 skill bodies (`brief`, `budget`, `confidence-check`, `design`, `develop`, `doctor`, `grill`, `misc`, `panic`, `prd`, `setup`, `simplify`, `task`, `test`) and 3 templates (`admin/lessons-registry.md`, `agents/standard/lifecycle/orchestrator.md`, `templates/orchestrator-pattern.md`). All guardrails now read "the underlying AI tooling or vendor" instead of naming a specific vendor. The two example-copy leaks (`examples/.../lifecycle/orchestrator.md` and `templates/.../orchestrator.md`) are aligned.

**Sprint Phase D.3 — idempotent worktree creation**:

- Replaced the unconditional `git worktree add` with an explicit six-state machine (reuse / branch-mismatch-or-dirty HALT / branch-elsewhere HALT / unregistered-leftover HALT / attach-existing-branch / fresh-create). State predicates use absolute paths from `git rev-parse --show-toplevel`.
- Added `DIRTY_PROBE_STATUS` (skipped/ok/failed) so a failed `git status` probe (timeout, missing dir, permission denied) NEVER collapses to "clean reuse". State 1 (reuse) requires registered + matching branch + path-on-disk + probe-ok + empty-dirty-state — five conjuncts.
- New `run_with_timeout` helper: GNU `timeout` / `gtimeout` (probed via `--kill-after=1 1 true` to detect non-GNU implementations and shell-function shadowing) → POSIX `/bin/sh`-hosted watchdog fallback that escalates TERM → KILL after 2s grace and normalises rc=143/137 → rc=124. Subshell function form (no `local`) so the body is POSIX-compliant. Hosting under `/bin/sh` prevents zsh's BG_NICE diagnostics from leaking into captured stderr. Verified across 4 shells × 2 PATH configs × 3 scenarios.
- Worktree-list parsing rewritten to handle paths with whitespace, regex metacharacters (`.`, `[`, `]`), and literal backslash sequences (`\t`, `\n`). Uses `substr($0, length("worktree ")+1)` for path capture and `ENVIRON["TARGET_PATH"]` (NOT `awk -v`) to preserve byte-literal paths across the awk boundary.

**Develop — `T_run` scope binding**:

- New section A.6.5 "Bind T_run" — defines `T_run = sprint-mode ? tasks_in_scope : full tasks file (or [T{N}] for --task mode)`, persists to `admin/scratch/{NNN-slug}-develop/t-run.json`, and adds an explicit out-of-scope guard. Tasks not in `T_run` get no lane assignment, no recheck, no manifest stub, no Gate 4/5 manifest, no new task-history entry from this run.
- Threaded `T_run` through every per-task loop and assertion: Phase B.1, B.1.5 (lane recheck), Phase D, Phase F closure, and the final verify checklist (5 bullets). 17 `T_run` references end-to-end.
- Resume logic now binds `T_run` BEFORE evaluating any artefact: loads `t-run.json` if scratch exists; HALTS with explicit user prompt if `t-run.json` is missing on retry (no auto-widening to feature-mode, which would invent artefact-existence requirements for tasks the user never asked to process). Every resume predicate (B/B.1/C/D+E/F/completion) scopes to `T_run`.
- Eval line at top of develop's frontmatter binds entries-per-task assertions to `T_run` and adds the symmetric out-of-scope absence requirement.

### Why this is a 2.7.1 not a 2.8.0

No new skills, no new commands, no new config keys, no breaking schema changes, no new doctrine docs. Every change either tightens an invariant the v2.7.0 release already promised (idempotency claim in sprint's eval; vendor-neutrality non-negotiable) or hardens an internal helper (timeout wrapper portability, worktree probe path-safety). Per SemVer this is a patch.

### Acknowledged tradeoff (NOT a failure)

- The POSIX timeout fallback cannot escape a target wedged in uninterruptible kernel state (D state — typically a hung NFS mount or buggy filesystem driver). KILL is uncatchable for normal processes but D-state is the kernel's wait, not the process's. Documented inline; mitigation: install GNU coreutils for `gtimeout` so the supervisor runs in a separate process group.

## [2.7.0] — 2026-05-08

The Sprint 4 sweep release of the v2.4+ master plan — and the closing release of the v2.x cycle. Two features ship, tightly co-designed: the new `specflow:sprint` skill (the first new skill since v2.3) and `agent-teams-per-stage` doctrine. After Sprint 4, the recommended next step is a real consumer-project dogfood.

### Added

**New skill** (`plugins/specflow/skills/sprint/`):

- `specflow:sprint` — lightweight sprint planner invoked by `specflow:develop` Phase A.5.5 as a sub-step (not a top-level user entry point). Pulls the feature's mapped Linear project (when MCP available); reconciles drift with local `tasks.md`; filters to the in-scope batch via `sprint-bucket: N` (per 025) and `config.develop.maxIssuesPerSprint` (default 5); synthesises a sprint plan with per-stage team assignments (per 026); presents the sprint-plan gate to the developer; on approval creates a git work-tree at `admin/scratch/{NNN-slug}-sprint/worktree/`. Returns the approved plan as a structured result. Refuses standalone invocation. Closes 020-sprint-skill (with 024-sprint-worktree absorbed).

**New doctrine doc**:

- `templates/admin/stage-teams.md` — Plan → Build → Test → Iterate → Validate as first-class doctrine; default rosters per stage; `config.json.teams.{stage}` schema; override path; consumption by 020 / develop / test. Closes 026-agent-teams-per-stage.

**New config knobs** (`admin/config.json`):

- `develop.maxIssuesPerSprint` — integer, default 5. Caps the in-scope batch per sprint plan (per 020).
- `teams` — object, default `{}` (empty). Per-stage roster overrides per 026's schema. First sprint-plan invocation materialises the doctrine defaults from `templates/admin/stage-teams.md` when keys are absent.

**Skill body changes** (no new skills beyond `specflow:sprint`):

- `specflow:develop` Phase A.5.5 (NEW between A.5 and A.6) — invokes `specflow:sprint {NNN-slug}` as a sub-skill; awaits the approved plan; iterates Phase B-F across the in-scope batch (per 020).
- `specflow:setup` Phase 8.2 — seeds `develop.maxIssuesPerSprint: 5`, `teams: {}`, and the new `sprint` skill toggle.

**New worked examples** (Sprint 4 features):

- `examples/docs/specflow/features/020-sprint-skill/` (PRD + Gate 2 manifest).
- `examples/docs/specflow/features/026-agent-teams-per-stage/` (PRD + Gate 2 manifest).

### v2.x cycle recap

The v2.x cycle (v2.0 → v2.7) shipped:

- 4 sprints, 21 features (001-008, 010-014, 016-020, 022-023, 025-029 — IDs 009 / 015 / 021 / 024 stayed allocated as decide-not-build / merged / removed / absorbed).
- 1 new skill in Sprint 4 (`specflow:sprint`); all other v2.x changes were additive across existing skills.
- 11 doctrine docs under `templates/admin/` and `templates/task/`: `CONTEXT.md`, `lessons.json` schema, `skill-toggles.md`, `example-versioning.md`, `team-review-bridge.md` (Sprint 1); `single-context-task.md`, `tdd-discipline.md`, `cross-task-review.md`, `sprint-bucket-heuristic.md` (Sprint 2); `lessons-registry.md`, `task-manifest-schema.md`, `reviewer-isolation.md` (Sprint 3); `stage-teams.md` (Sprint 4).
- 21 worked-example folders under `examples/docs/specflow/features/`.
- 2 new core principles (TDD added in Sprint 2; the original 4 unchanged).
- A locked-in architectural decision list of 21 items (in `v2/docs/SESSION-HANDOFF.md`).

The v2.x cycle establishes the chain-don't-absorb pattern as the dominant evolution mode: new features land their operational detail in doctrine docs; SKILL.md bodies carry citations. This keeps skills under their context budget and makes future feature work surgical.

### Acknowledged tradeoffs carried into v2.7

- `brief/SKILL.md` and `task/SKILL.md` exceed the ≤500-line skill-size ceiling (Sprint 2 additions). Chain-don't-absorb extraction (sibling `brief-blocks` / `cross-task-applier` skills) can land in v2.8+ if pressure remains.

## [2.6.0] — 2026-05-08

The Sprint 3 sweep release of the v2.4+ master plan. Five features extend the operational runtime instrumentation: lessons-registry formalisation, per-task manifest, brand-consistency lens, reviewer-context-isolation contract, edge-case-reviewer. All chain-don't-absorb-shaped — five new doctrine docs absorb operational detail; SKILL.md bodies carry citations.

### Added

**New doctrine docs**:

- `templates/admin/lessons-registry.md` — schema (id / created / tags / surface / outcome / context / lesson / source / confidence / superseded_by / status), write paths (test --feedback, complete retro, insights clustering), read paths (prd Phase A.3.5, task Phase A.4, cross-task review), query algorithm (tag-overlap × confidence-weight × recency-decay), config knobs. Closes 018-lessons-registry.
- `templates/admin/task-manifest-schema.md` — read-first contract, standardised entry format (timestamp / agent_id / phase / event_type / input_ref / output_ref / body / outcome), six lifecycle phases captured, migration from 017's interim stub. Closes 019-task-manifest.
- `templates/admin/reviewer-isolation.md` — fresh subagent spawn contract, declared-input-only, pairwise-non-equal `agent_id` (format contract: harness-emitted run ID + slot suffix; ISO-8601-with-suffix fallback), runtime collision check (`FRESH-CONTEXT-VIOLATION` aborts the gate), cross-cutting impact across Gates 2/3/4/5. Absorbs 022's interim convention; enables 028's reviewer to land conforming. Closes 027-reviewer-context-isolation.

**New principle reviewer agent**:

- `examples/docs/specflow/admin/agents/standard/principles/edge-case-reviewer.md` — five-question lens (collateral surface / failure modes / inheritance / interaction / state-environment); deliberately NOT goal-aware (the load-bearing complement to Goal-Driven's reverse-traceability blindspot); advisory output shape (`recommendation` + `reasoning`); fresh-context per 027. Closes 028-edge-case-reviewer.

**New config knobs** (`admin/config.json`):

- `prd.maxLessonsSurfaced` — integer, default 5. Caps lessons surfaced inline at PRD Phase A.3.5.
- `task.maxLessonsSurfaced` — integer, default 5. Caps lessons surfaced at task Phase A.4.

**Skill body changes** (no new skills):

- `specflow:prd` Phase A.3.5 (NEW between A.3 and A.4) — queries `lessons.json` with the feature's tag profile; surfaces inline as "What we've learned before that applies here" subsection in the interview's Codebase context.
- `specflow:task` Phase B.3 — per-task entry gains `prior-lessons: [L-NNN, ...]` field (per 018).
- `specflow:test` Phase B.3 write template — adds Brand-consistency lens section with eight standard questions in a table; advisory-not-AC-failing semantics documented inline (per 023).
- `specflow:develop` Phase C.2 (Gate 4) reviewer set — adds `edge-case-reviewer` to the standard reviewers list (per 028).
- `specflow:develop` Phase E.2 (Gate 5) reviewer set — adds `edge-case-reviewer` (per 028).
- `specflow:setup` Phase 8.2 — seeds `prd.maxLessonsSurfaced: 5` and `task.maxLessonsSurfaced: 5` in the config.json template.

**Reviewer template updates** (under `templates/agents/standard/principles/`):

- `goal-driven-reviewer.md`, `simplicity-reviewer.md`, `surgical-reviewer.md`, `think-before-coding-reviewer.md` — each gains the line *"Does NOT consult the writer's chat or the orchestrator's deliberation transcripts"* + the `writer_id ≠ {reviewer-name} agent_id` non-equality clause + citation to `templates/admin/reviewer-isolation.md`. Per 027.

**Orchestrator pattern extension**:

- `templates/orchestrator-pattern.md` — new "Reviewer fresh-context dispatch" section between the three primitives and the skill-author checklist; new checklist bullet *"Every reviewer dispatch in a multi-agent gate honours the fresh-context contract"*. Per 027.

**New worked examples** (Sprint 3 features):

- `examples/docs/specflow/features/018-lessons-registry/` (PRD + Gate 2 manifest).
- `examples/docs/specflow/features/019-task-manifest/` (PRD + Gate 2 manifest).
- `examples/docs/specflow/features/023-test-brand-consistency/` (PRD + Gate 2 manifest).
- `examples/docs/specflow/features/027-reviewer-context-isolation/` (PRD + Gate 2 manifest demonstrating the agent_id contract on its own gate).
- `examples/docs/specflow/features/028-edge-case-reviewer/` (PRD + Gate 2 manifest).

## [2.5.0] — 2026-05-07

The Sprint 2 sweep release of the v2.4+ master plan. Five features rewrite primary contracts (PRD/task/develop/brief). One-shot template-churn window — every Sprint 2 feature touches `task/SKILL.md` or `develop/SKILL.md` or `brief/SKILL.md`; one MIGRATIONS entry, one round of worked-example backfill (using the 013 versioning policy from Sprint 1).

### Added

**New doctrine docs** (chain-don't-absorb pattern):

- `templates/admin/single-context-task.md` — single-context-window-per-task contract: rationale, verbatim rule (locked-in decision #21), `context-budget-estimate` schema, no-mid-task-compaction contract, develop Phase A pre-flight, cross-references. Closes 029-single-context-task. Worked example: `examples/docs/specflow/features/029-single-context-task/`.
- `templates/admin/tdd-discipline.md` — Pocock's Red → Green → Refactor cycle: cycle steps with bounded Refactor (no new behaviour, no new files, no scope creep / route to `specflow:scope-change`), per-task manifest stub schema (`red:` / `green:` / `refactor:` markers with outcome enum + ISO timestamp), lane interactions table, `--plan-only --task` Red artefact contract, pre-implementation test execution. Closes 017-tdd-discipline. Worked fixtures: `examples/docs/specflow/features/017-tdd-discipline/fixtures/{yellow-happy-path,green-skip-config,refactor-new-file-block}.md`.
- `templates/task/cross-task-review.md` — Phase E.4.5 + Phase F doctrine: three-round mini-debate (Cross-task R1 → Applier R2 + apply → Cross-task R3 sharpen → Applier final pass), per-finding decision schema (`accepted | rejected | scope-change-required`), hard-cap enforcement (per 029-R4) at the applier, sprint-bucket recompute (per 025) on accepted merge/split, manifest schema extension (`writer_id` / `cross_task_reviewer_id` / `applier_id` triplet, "Cross-task findings" H2 section), sub-agent dispatch failure fallback. Closes 022-cross-task-review. Worked fixtures: `examples/docs/specflow/features/022-cross-task-review/fixtures/{cross-task-worked-example,threshold-skip-2-tasks/}`.
- `templates/task/sprint-bucket-heuristic.md` — single-rule fixpoint heuristic with typed `(int, optional-letter)` comparator, bump iteration discipline, topological-floor corollary, graph-validity step (`GRAPH-INVALID:` diagnostics for cycle / self-loop / duplicate-task-id / duplicate-edge / dangling-reference), per-task budget respect. Closes 025-sprint-task-flagging. Worked fixtures: `examples/docs/specflow/features/025-sprint-task-flagging/fixtures/{four-task-bucketing,graph-invalid-cases}.md`.

**New principle reviewer agent** (under `examples/docs/specflow/admin/agents/standard/principles/`):

- `cross-task-reviewer.md` — coherence + better-arrangement lenses applied to the entire task list as a single artefact at Gate 3 Round 2.5. Reads `context-budget-estimate` per task (per 029) as a soft signal for the better-arrangement lens. Never sees the writer's chat (per 027 substrate). Surfaces `coherence` / `better-arrangement` findings with `lens` field + severity + claim + evidence + proposed_change.

**New config knobs** (`admin/config.json`):

- `task.contextBudget` — integer, default 80000. Per-task token ceiling for the single-context-window rule (per 029). Tasks whose `context-budget-estimate` exceeds this value auto-flag at synthesis (`specflow:task` Phase B.4) and route to `specflow:scope-change` for splitting. Resolver contract: `templates/admin/single-context-task.md`.
- `develop.tddRequired` — boolean, default `true`. When `true`, Green lane behaves identically to Yellow on the Red artefact contract (per 017). When `false`, Green may skip Red and the per-task manifest stub records `red: skipped (config) (...)` alongside the operator's strong-CI-signal attestation. Knob applies to Green only — Yellow always enforces; Red is human-led.

**New per-task field** (`{NNN-slug}-tasks.md`):

- `sprint-bucket: N` — positive integer ≥ 1, derived deterministically from the dependency graph + scope-overlap per `templates/task/sprint-bucket-heuristic.md` (per 025). Read by future `specflow:sprint` (020, Sprint 4) for parallel fan-out batch planning.
- `context-budget-estimate: <int_tokens>` — pre-existing field formalised in `templates/admin/single-context-task.md`'s schema (per 029).

**New core principle** (`CORE_PRINCIPLES.md`):

- `## TDD` section adopting Pocock's Red → Green → Refactor framing with the canonical Pocock quote *"TDD forces the LLM to really take small steps"*. Cites `templates/admin/tdd-discipline.md` as the doctrine home.

**Skill body changes** (no new skills):

- `specflow:task` Phase B.3 write template — adds `sprint-bucket: N` per-task field; `context-budget-estimate` per-task field formalised.
- `specflow:task` Phase B.4 self-check — gains budget self-check (per 029), graph-validity check (per 025), and sprint-bucket assignment step (per 025).
- `specflow:task` Phase E adds Phase E.4.5 (Cross-task review three-round mini-debate) between E.4 (Round 2) and E.5 (Round 3). Per-task Round 3 (E.5) hybrid surface — sharpen surviving findings, auto-resolve merged-out / dropped, treat applier-introduced tasks as `round-3-net-new`.
- `specflow:task` Phase E.6 closer — manifest gains `writer_id` / `cross_task_reviewer_id` / `applier_id` triplet, "Cross-task findings" H2 section with three H3 sub-headings, `passed-with-revisions` status added to the taxonomy. FAIL rule applies to UNION of per-task and cross-task findings.
- `specflow:task` Phase F (NEW) — `--apply-cross-task-feedback {NNN-slug}` applier flow with precondition check, per-finding decision (`accepted | rejected | scope-change-required`), hard-cap enforcement (per 029-R4), sprint-bucket recompute (per 025).
- `specflow:develop` Phase A.6 — context-budget pre-flight per in-scope task (estimate-vs-actual ≥20% divergence triggers a three-option developer prompt; outright budget breach routes to `specflow:scope-change` non-optionally). Per 029.
- `specflow:develop` Phase D — Red sub-step (`specflow:test --plan-only --task T{N}` invocation; pre-implementation test execution; manifest marker `red:`); Green sub-step (gated on Red artefact for Yellow always and Green when `tddRequired: true`; manifest marker `green:`); Refactor sub-step (bounded structural improvement; manifest marker `refactor:`). Per 017.
- `specflow:develop` Phases D / E / F single-context-window reminder — no mid-task compaction; escalate to developer per A.6's three-option prompt instead. Per 029.
- `specflow:test` `--plan-only --task T{N}` mode — per-task variant writes only the per-task plan section into `{NNN-slug}-test.md` and marks the primary AC's case as `Status: red (failing)` by default. Phase B.5 (Codex pass + user prompt) is skipped in `--task` mode. Per 017.
- `specflow:brief` Visual Block Grammar — four new structured blocks added: `:::key-features` (non-technical card grid), `:::resources` (link cards with source-type pill icons; built-in inline SVG-base64 icons for `linear | doc | design | github`; `icon=` overrides; malformed overrides fall back), `:::key-decisions` (decision table — Decision | Why | Source columns), `:::phase-split` (two-column iteration boundary). Eight-kind grammar; strip rule, eval field, supported-set documentation all updated. CSS extends to mobile breakpoints (1100px / 860px collapse to single-column). Per 016.
- `specflow:setup` Phase 8.2 — seeds `task.contextBudget: 80000` in the example config.json template.

**New worked examples** (Sprint 2 features):

- `examples/docs/specflow/features/016-brief-enhancements/` (PRD, brief, interview, debate-log).
- `examples/docs/specflow/features/017-tdd-discipline/` (PRD, brief, interview, debate-log, three fixture files).
- `examples/docs/specflow/features/022-cross-task-review/` (PRD, brief, interview, debate-log, two fixtures).
- `examples/docs/specflow/features/025-sprint-task-flagging/` (PRD, brief, interview, debate-log, two fixtures).
- `examples/docs/specflow/features/029-single-context-task/` (PRD, interview, tasks, debate-log).

### Known acknowledged tradeoff

`brief/SKILL.md` and `task/SKILL.md` exceed the ≤500-line skill-size ceiling after the Sprint 2 additions (brief at ~630 lines; task at ~640 lines). The chain-don't-absorb path was applied (four new doctrine docs absorb the operational detail), but the four new visual blocks in `brief/SKILL.md` and the new Phase E.4.5 / Phase F sub-phases in `task/SKILL.md` are large enough to push past the cap even with the doctrine extraction. Acknowledged tradeoff for v2.5.0; chain-don't-absorb extraction (e.g. a sibling `brief-blocks` skill, or a sibling `cross-task-applier` skill) can land in v2.6 or later if the cap pressure remains operationally relevant.

## [2.4.0] — 2026-05-06

The Sprint 1 sweep release of the v2.4+ master plan. Closes the three remaining v2.x open PRD questions, pre-emptively mitigates two structural risks surfaced during master-plan validation, and adds two cross-cutting doctrine surfaces (skill toggles + worked-example versioning). No new skills ship; all changes are additive across `setup`, `prd`, `task`, `test`, `develop`, `brief` SKILL.md bodies plus three new doctrine docs.

### Added

**Config schema additions** (`admin/config.json`):

- `brief.commitPolicy` — string ∈ `committed | derived`. Default `committed`. Setup prompts the user; `derived` mode appends `*-brief.html` to project `.gitignore`. `specflow:brief` reads the knob and renders a one-line policy banner near the top of the HTML so reviewers self-document the policy. Closes 011-brief-commit-policy. Worked example: `examples/docs/specflow/features/011-brief-commit-policy/`.
- `skills.{name}.enabled` — block of per-skill toggle objects. Default seeds every shipped v2.4 skill with `{ "enabled": true }`. Each skill's Phase A.0 pre-flight reads the field; refuses with the canonical refusal message and returns when disabled. Backward-compat: missing field == enabled. Resolver contract documented in `templates/admin/skill-toggles.md`. Demonstrated in `specflow:develop`'s Phase A.0; other skills adopt the pattern incrementally as touched. Closes 012-config-skill-toggles. Worked example: `examples/docs/specflow/features/012-config-skill-toggles/`.

**Skill body changes** (no new skills):

- `specflow:test` Phase C.2.1 — lazy-populates `admin/pages.json` from Playwright-visited routes on UI test runs. The lazy-population path is the canonical inventory mechanism; no dedicated `specflow:pages` skill ships. Closes 009-pages-policy at the PRD level.
- `specflow:prd` Phase A.3 — ingests feature-local `design/` folder (`proposed.html`, `iteration-log.md`, `current.html`) when present; surfaces `Design intent (from design/):` bullets in the *Codebase context* section. Silent no-op when absent. Closes 010-design-readback. Worked example: `examples/docs/specflow/features/010-design-readback/`.
- `specflow:task` Phase A.2.5 — partitions iteration-log entries by timestamp vs PRD frontmatter date; tasks gain `design-decision: iteration-N` field when matched; uncovered post-PRD entries surface a scope-change candidate prompt.
- `specflow:brief` step 9 — reads `config.json.brief.commitPolicy`; renders one-line policy banner near top of HTML.
- `specflow:develop` Phase A.0 — checks `config.skills.develop.enabled`; refuses with the canonical message when false.
- `specflow:setup` Phase 8.2 — prompts for the brief commit policy, seeds the `skills` toggle block, manages `.gitignore`. Phase 8.3 documents `pages.json` lazy-population.

**New doctrine docs** (under `plugins/specflow/templates/admin/`):

- `skill-toggles.md` — resolver contract for `config.json.skills.{name}.enabled`: canonical refusal message format, backward-compat semantics, chain-breakage by design, upgrade handling.
- `example-versioning.md` — worked-example versioning policy: `templateVersion: vX.Y` frontmatter field on every PRD/tasks/test, backfill / grandfather / retire decisions on template-changing releases, audit script for sprint close. Mitigates Risk A from the v2.4+ master plan ("Worked-example debt cascade").
- `team-review-bridge.md` — `agent-teams:team-review` ↔ Gate-5 manifest mapping: severity correspondence (Critical → block, etc.), field correspondence (markdown → JSON), dedup rules (exact-location, cross-reviewer-overlap, independent-fire), when-to-invoke trigger (≥2 severity-level disagreement). Mitigates Risk B from the v2.4+ master plan ("agent-teams semantics drift from gate manifests"). Worked example: `examples/docs/specflow/features/014-team-bridge-spec/`.

**Worked-example backfill:**

- All v2.0-v2.3 worked-example PRDs (`001-design-skill` through `008-optimize-skill`) gained `templateVersion: v2.3` frontmatter. Informational tag; content unchanged. Sprint 2 (the first real template-changing release per the master plan) triggers the first audit run.

**New worked examples** (Sprint 1 features):

- `010-design-readback/` — full PRD + interview + tasks + Gate 2 manifest + design folder demonstrating the readback.
- `011-brief-commit-policy/` — full PRD + interview + tasks + Gate 2 manifest demonstrating the config knob.
- `012-config-skill-toggles/` — full PRD + interview + tasks + Gate 2 manifest demonstrating the toggle pattern.
- `013-example-migration-policy/` — full PRD + interview + tasks + Gate 2 manifest demonstrating the versioning policy.
- `014-team-bridge-spec/` — full PRD + interview + tasks (with sample `team-review` → Gate-5 translation) + Gate 2 manifest.

(009-pages-policy ships as a decide-not-build PRD-level closure; no worked-example folder needed — the resolution lives in `v2/docs/PRD.md` § Resolved decisions.)

### Changed

- `v2/docs/PRD.md` § Open questions — all nine original questions are now resolved through the v2.x ship cycle. v2.4.0 leaves Open questions empty. Refinements surface as new PRDs going forward.

### Migrated

See `MIGRATIONS.md` v2.3 → v2.4 for the full upgrade path. Highlights: backup `admin/config.json.bak`; prompt for brief commit policy (default `committed`); seed `skills` toggle block with all-enabled; tag existing worked-example PRDs with `templateVersion: v2.3`; stamp `specflowVersion: "2.4.0"`. All changes additive; no source markdown modified.

### Risk mitigations baked in

- **Risk A (worked-example debt cascade)** — `templateVersion` frontmatter + audit-script-driven backfill plan ready before Sprint 2's template-changing release.
- **Risk B (agent-teams semantics drift)** — `team-review-bridge.md` defines the merge protocol before Sprint 4's planner skill becomes the first feature to invoke both surfaces.

### Sprint 1 close — what's next

Sprint 2 (`015-key-features-section`, `016-brief-enhancements`, `017-tdd-discipline`, plus `022-cross-task-review` + `025-sprint-task-flagging` absorbed from the user's mid-session features.md) ships as v2.5.0. Sprint 3 (`018-lessons-registry`, `019-task-manifest`, `023-test-brand-consistency`) as v2.6.0. Sprint 4 (`020-sprint-skill` with `024-sprint-worktree` absorbed, `021-design-image-gen`) as v2.7.0. Master plan: `v2/docs/MASTER-PLAN.md`.

## [2.3.0] — 2026-05-06

The Phase 3 memory-and-discipline release. All six target skills (`complete`, `decision`, `scope-change`, `insights`, `prune`, `optimize`) now have operational bodies. Closes the v2 architectural arc.

### Added

**Project self-learning corpus (lessons registry):**

- `admin/lessons.json` — new file, seeded as `[]` by `specflow:setup`. Mutable JSON array of lesson entries; canonical schema and lifecycle defined in `skills/test/SKILL.md` § Lessons registry. Supports four kinds (`escape | success | rule-candidate`) and four statuses (`active | superseded | resolved | promoted-to-rule`) with first-class supersession (`superseded_by` pointer) and rule promotion (`promoted_to_rule` anchor). Tags drawn from a controlled vocabulary derived from `environment.json` (stack), `profiles.json` + `glossary.md` (domain), and a canonical surface set (`ui`, `data-model`, `api`, `auth`, `migration`, `infra`, `cli`, `docs`).
- `specflow:test` extended with `--feedback` mode (Phase D — 4-step lesson capture flow: read context → capture user's three plain-language answers verbatim → tag and similarity-check → atomic write of three artefacts with `lessons.json.bak` discipline) and Phase B.0 (lesson query before plan synthesis; matched lessons become covering test cases).
- `specflow:task` A.4 — query `lessons.json` by feature tags before task derivation; matched lessons become `lesson-anchor: L-NNN` fields on derived tasks or are recorded as user-accepted-uncovered in `admin/scratch/{slug}-tasks/uncovered-lessons.json`.
- `specflow:complete` H.2.1 — soft chat-line reminder pointing the user at `specflow:test {slug} --feedback` after every successful retro write. Non-blocking; raises the floor on remembering the loop is available.
- `specflow:setup` 8.4 + 9.2 — seeds `admin/lessons.json` as `[]` during first-run scaffold; verify checklist now includes the new file.
- Promotion path: a lesson reaching 3 occurrences across distinct features auto-prompts for promotion to `admin/rules/guidelines.md`. Promoted rules are then read by `specflow:prd` and `specflow:task` on every future feature, graduating from "heads-up" to "non-negotiable on relevant features." This corpus is also the substrate that `specflow:insights` clusters from.

**Phase 3 skills (operational):**

- `specflow:insights` (stub → 450 lines, 7 phases A-G) — read-only mining of `task-history.json` for cross-task patterns. Two-pass deterministic clustering (field-shape exact-match + token-frequency n-grams); ≥3-observation promotion threshold; produces `admin/insights/{YYYY-MM}-report.md` (replaced-in-place on within-month re-runs) + `{YYYY-MM}-runs.jsonl` (append-only execution log); uses `specflow:decision` for audit-trail entries on accepted promotions.
- `specflow:prune` (stub → 316 lines, 8 phases A-H + standalone `restore` verb) — quarterly pruning with per-surface staleness detection (decision-log, rules, agent snapshots, task-history). Two-stage archive-then-remove flow with byte-identical round-trip restoration as the binary eval property. Append-only archive at `admin/archive/{YYYY-Q}-prune.md`; skill never modifies its own archive.
- `specflow:optimize` (stub → 510 lines, 9 phases A-I) — generalises `simplify`'s discipline across the verifiable-skill set. Six structured mutation operators (`tighten` / `consolidate` / `clarify` / `deduplicate` / `reorder` / `split-by-phase`); per-variant evaluation via the target's machine eval only (no LLM-as-judge inside the loop); three independent auto-merge guardrails (HTML comment in PR body, no `--auto` call in CI, GH Action human-actor check). Initial six targets: `release-version-check`, `simplify`, `format`, `tdd-cadence`, `init`, `feedback-loop-audit`.

**Phase 3 PRD anchors (recursive-bootstrap chain via `specflow:prd`):**

- `examples/.../features/006-insights-skill/` — 15 R / 16 AC, Gate 2 closed `passed-with-revisions`, 7 findings (2 blocks resolved), 2 push-backs defended.
- `examples/.../features/007-prune-skill/` — 11 R / 11 AC, Gate 2 `passed-with-revisions`, 5 findings (2 blocks resolved), 1 push-back with concession.
- `examples/.../features/008-optimize-skill/` — 17 R / 16 AC, Gate 2 `passed-with-revisions`, 7 findings (2 blocks resolved → R17 score-direction + R13 decline-streak semantics).

### Changed

**Frontmatter shape standardised across all 25 SKILL.mds:** every SKILL.md carries `name`, `description`, `status`, `phase`, `requires`, `produces`, `eval`. `status:` ∈ `shipped | v2-enhancement | v2-new`; `phase:` ∈ `1 | 2 | 3`. Bare-name skills (`/X` style — `panic`, `simplify`, `confidence-check`, `feedback-loop-audit`, `grill`, `optimize`, `prune`, `insights`) use bare names; `specflow:X` skills use the prefixed form.

**`SKILLS.md` glossary refreshed:** every operational skill now carries the `🆕`/`🔧`/`✅` marker; legacy `⏳`/`⏳⏳` (Phase 2/3 stub) markers removed. Listed inventory matches disk 1:1.

**`task-history.json` schema field naming aligned:** the retro field is `what_didnt_work` (not bare `what_didnt`). Worked-example admin tree migrated; eval blocks updated.

**Cross-skill reference graph audited:** every `specflow:X` and bare-name reference inside SKILL.md bodies resolves to a real skill directory. H1 headings match `name:` frontmatter convention. One vendor-name guard rewritten to vendor-neutral phrasing per CLAUDE.md.

### Migrations

`MIGRATIONS.md` v2.2 → v2.3 entry. Adds:

- `config.json.insights.{minCorpusSize default 10, cadence default "monthly"}`.
- `config.json.prune.thresholds.{decisionLog.{ageDays 365, dormancyDays 182}, guidelines.dormancyDays 365, taskHistory.{ageDays 365, dormancyDays 182}}`.
- `config.json.optimize.{targetCapUsd default $10, judgementWords default ["appropriately","adequately","cleanly","concrete signals","coverage","idiomatic","well","properly","correctly"]}`.
- `admin/lessons.json` seeded as `[]` (project self-learning corpus; existing files not overwritten).

Migration is purely additive — no file relocations, no schema rewrites, no destructive operations. New on-demand directories: `admin/insights/`, `admin/archive/`, `admin/scratch/optimize-{target}-{run-ts}/`. New always-on files: `admin/lessons.json`. New append-only files: `admin/optimize-runs.jsonl`, `admin/scratch/prune-history.json`. Backups retained until next successful upgrade or `/specflow:upgrade --clean-backups`.

The decline-streak windows (7-day operator-avoid, 30-day target-skip per `008-optimize-skill` PRD R13) and the unique-id promotion threshold (3 per `006-insights-skill` PRD R2) are intentionally hardcoded in v1 — the discipline-installer contract; surfacing as knobs would invite Goodharting.

### PRD § Resolved decisions extended

`v2/docs/PRD.md` § "Resolved decisions" gains 16 new entries dated 2026-05-06 covering E1-E10 prompt edits, brief replacing render, Codex sixth-reviewer at Gate 5, B.1 mechanical recheck, conditional-pass escalation, two-pass deterministic clustering for `/insights`, six-operator variant generation for `/optimize`, per-surface staleness boundaries for `/prune`, and the frontmatter shape standardisation.

## [2.2.0] — 2026-05-06

The brief release. Replaces per-PRD HTML rendering with a richer feature brief that composes PRD + interview + gate manifests into a single self-contained HTML document; lands two more Phase 3 skills (`specflow:decision`, `specflow:scope-change`); applies the E6-E10 prompt-edit recommendations from the 003-complete-skill dogfood.

### Added

**Brief skill (replaces render):**

- `specflow:brief` (520-line skill body) — composes `{NNN-slug}-brief.html` from `{NNN-slug}-prd.md` + `{NNN-slug}-interview.md` + (optional) Gate 2 / Gate 3 manifests. Self-contained HTML with sidebar TOC and a Visual abstract section at the top. Supports a structured-block vocabulary (`:::flow`, `:::comparison`, `:::scope`, `:::tree`) for visualising flows, mode comparisons, scope, and decision trees deterministically.
- `specflow:prd` Phase E — Brief — added; invokes `specflow:brief` after Gate 2 closes and asks the user whether to open the resulting brief in their browser.

**Phase 3 skills (operational):**

- `specflow:decision` (38-line stub → 280 lines, 6 phases A-F) — interactive decision-log writer. Pre-flight + tail-parse, title prompt with duplicate-title resolution, schema-drift warning + body capture, append-only write with read-back, two-stage chat-line user-surface contract. Implements all 11 R-IDs from `004-decision-skill-prd.md`.
- `specflow:scope-change` (35-line stub → 436 lines, 8 phases A-H) — mid-development scope-change capture. Pre-flight + routing, drift articulation, `/grill` extend-mode with strikethrough markup for superseded resolved-lines, surgical PRD synthesis + Gate 2 re-fire, delta task regeneration + Gate 3 re-fire, four-source impact list, `specflow:decision` invocation with `id_prefix: "SC"`, final disposition with best-effort Linear sync. Implements all 12 R-IDs from `005-scope-change-skill-prd.md`.

### Changed

**Render → Brief:**

- `specflow:render` removed. Its responsibility is fully absorbed by `specflow:brief`.
- `specflow:doctor`'s `features.{NNN-slug}.html_drift` check renamed to `brief_drift`; now compares brief mtime against the latest of PRD / interview / gate-manifest mtimes.
- `specflow:upgrade` step 10 (`specflow:render --all`) replaced with `specflow:brief --all`; deletes superseded `{NNN-slug}-prd.html` after the brief is written.

**Per-feature artefact:**

- New: `features/NNN-{slug}/NNN-{slug}-brief.html` (composed for every feature with both a PRD and an interview file present).
- Removed: `features/NNN-{slug}/NNN-{slug}-prd.html` (deleted once the sibling brief is written; the brief supersedes it).

**E6-E10 prompt edits applied** (from `examples/.../features/003-complete-skill/DOGFOOD-DEBRIEF.md`):

- E6 `skills/develop/SKILL.md` Phase A.2 — Codex availability check carries a lens-overlap note distinguishing Codex's correctness lens from Goal-Driven's reverse-traceability lens at Gate 5; both firing on the same finding is independent confirmation, not duplication.
- E7 `skills/develop/SKILL.md` new Phase B.1.5 — formalises `b1_recheck` aggregate-outcome schema with `batch_shape_at_default_cap` field.
- E8 `skills/complete/SKILL.md` Phase A.3 — captures the 30-min stale-lock heuristic as a v2 candidate for promotion to `config.json.complete.staleLockMinutes`. Literal 30 retained for v1.
- E9 `templates/agents/standard/principles/goal-driven-reviewer.md` — orphan-phase reverse-traceability lens added; extends the orphan-AC pattern from PRD/tasks gates to code-review gates.
- E10 `skills/develop/SKILL.md` new Phase F.2.1 — conditional-pass escalation contract (two-option user prompt: accept-and-proceed with documented condition vs defer with `specflow:misc --auto` follow-up; no third "force-pass" path).

### Migrations

- `MIGRATIONS.md` v2.1 → v2.2 entry. Composition source widening (PRD + interview + gates) replacing the prior PRD-only render. Migration is artefact rename + deletion of superseded `prd.html` files; no schema changes; backups retained.

### Phase 3 still-stubbed in 2.2.0

`/insights`, `/prune`, `/optimize` ship as frontmatter-only stubs. PRDs land in v2.3.0.

## [2.1.0] — 2026-05-06

The development-layer release. Phase 2 ships; the chain has been dogfooded end-to-end through all six gates on a Phase 3 retro skill (`specflow:complete`); `specflow:complete` itself ships as the lane-execution byproduct.

### Added

**Phase 2 skills:**
- `specflow:develop` (649-line orchestrator, 6 phases: pre-flight + plugin/CLI/MCP detection; lane triage with rule-based confidentiality classification; mechanical pre-Gate-4 lane recheck per R5.1; Gate 4 plan-vs-PRD debate manifest; lane execution green/yellow/red; Gate 5 code-vs-plan with Codex degradation; Verifier + PR + task-history). Implements the dogfooded 002-develop-skill PRD (R1-R17 plus R5.1).
- `specflow:agent` (232-line per-repo registry, 4 verbs: `list | add | remove | refresh`; snapshots specialised agents; surfaces drift on refresh; refuses removal of standard agents).

**Phase 3 skills (operational by dogfood byproduct):**
- `specflow:complete` (472-line orchestrator, 8 phases A-H: pre-flight + invocation routing + lock acquisition; idempotency check; interactive Q&A or auto-mode synthesis; significance elevation evaluation with triple-flag tracking; append-only `task-history.json` write with schema validation; optional `decision-log.md` elevation; Linear status sync; lock release + chat-line summary). Produced by the lane-execution dogfood on 003-complete-skill.

**Worked examples:**
- `features/003-complete-skill/` — full Phase 3 retro-skill specification: interview + PRD (14 R, 15 AC) + Gate 2 (passed-with-revisions, 7 findings, 2 push-backs defended) + 15 tasks + Gate 3 (passed-with-revisions, 6 findings) + Gate 4 plan-vs-PRD (passed-with-revisions, 6 findings) + Gate 5 code-vs-plan including Codex (passed-with-revisions, 8 findings; Codex contributed 2/8 including a real correctness defect on schema validation under-check) + Verifier outcome (`verified-with-conditions`: 14 pass + 1 conditional) + DOGFOOD-DEBRIEF surfacing E6-E10 prompt edits.
- `features/004-decision-skill/` — Phase 3 PRD spec for `specflow:decision` (interview + PRD + Gate 2 manifest passed-with-revisions; ready for Phase 3 implementation).
- `features/005-scope-change-skill/` — Phase 3 PRD spec for `specflow:scope-change` (interview + PRD + Gate 2 manifest passed-with-revisions).

**Six-reviewer Gate 5 with Codex:** The `develop-gate5` debate manifest fires Codex as the sixth parallel reviewer when detected in `environment.json`; when absent, the manifest header records `codex: unavailable` and reviewers proceed without it. The 003 dogfood demonstrates Codex catching a load-bearing correctness defect that same-provider reviewers missed.

**Lane-plan artefact:** `admin/scratch/{NNN-slug}-develop/lane-assignments.json` records lane triage outcome including the four-axis triage tuple per task, B.1 mechanical recheck outcome, and lane summary. Read by Gate 4 reviewers; consumed by Phase D execution.

**Verifier-outcome artefact:** `admin/scratch/{NNN-slug}-develop/verifier-outcome.json` records pass / conditional-pass / fail per task with evidence cited at file:line. Read by `specflow:complete` Phase A.2 (acceptance check) and by Phase F final disposition.

### Changed

- `skills/prd/SKILL.md` — applied E1-E3 prompt edits (Vision verbatim-vs-paraphrase contract; Phase C.3 cross-checks ACs against Phase 1 skill schemas; Gate 2 status taxonomy extended with `passed-with-revisions`).
- `skills/task/SKILL.md` — applied E5 prompt edit (Phase A.2 surfaces Gate 2 block-finding resolutions before extraction; numbering shifted A.2→A.3, A.3→A.4).
- `templates/agents/standard/principles/goal-driven-reviewer.md` — applied E4 prompt edit (orphan-AC reverse-traceability lens).
- Gate-2/3/4/5 status taxonomy now uniformly: `passed | passed-with-revisions | passed-with-escalations | failed`. The 003-complete-skill dogfood ran every gate at `passed-with-revisions`.

### Migrations

- `MIGRATIONS.md` v2.0 → v2.1 entry. Adds `config.json.develop.{greenBatchCap default 3, codexAtGate5 env-derived}`, optional `stack_match_reason` field on `agents/index.json` (additive — existing entries valid without it), and on-demand `develop-gate4/` / `develop-gate5/` debate-log subdirectories. Migration backs up `config.json` before writing; user confirms `greenBatchCap` default at next `specflow:develop` invocation.

### Dogfood debrief

The 003-complete-skill dogfood validates the Phase 2 chain end-to-end:

- **Lane triage** classified 13/2/0 (Green/Yellow/Red) with rule-based confidentiality matching zero confidential paths.
- **B.1 mechanical recheck** ran across all 15 tasks (file-count, module, path-glob), no upgrades triggered, outcome recorded.
- **Gate 4** (plan-vs-PRD) closed `passed-with-revisions` with 6 concerns and 1 defended push-back; 5 plan revisions applied.
- **Lane execution** produced the 472-line `skills/complete/SKILL.md` honouring all Gate 4 revisions and prior Gate 2 push-backs.
- **Gate 5** (code-vs-plan) closed `passed-with-revisions` with 8 findings; Codex contributed 2/8 including a load-bearing correctness defect on schema validation (`required = present, possibly with default` rather than `required ⇒ rejects extraneous`).
- **Verifier** returned `verified-with-conditions` with 14/15 pass + 1 conditional-pass on T4 (cross-skill schema dependency on `specflow:develop` Phase F.5 default-flag emission).
- E6-E10 prompt-edit recommendations captured in `features/003-complete-skill/DOGFOOD-DEBRIEF.md` for next-session application.

## [2.0.0] — 2026-05-06

The foundation release. Major architectural rework of v1.

### Operational inventory at 2.0.0

**Skills (15 operational + 2 v1-shipped/v2-aligned):**

| Spine | Workflow | Trust ladder | Telemetry + discipline | v1-shipped |
|-------|----------|--------------|-----------------------|------------|
| `specflow:setup` | `specflow:task` | `panic` | `specflow:budget` | `specflow:prime` |
| `specflow:prd` | `specflow:test` | `confidence-check` | `specflow:feedback-loop-audit` | `specflow:linear` |
| `/grill` | `specflow:misc` | | `simplify` | |
| `specflow:render` | `specflow:design` | | | |
| `specflow:upgrade` | `specflow:doctor` | | | |

**Standard agents (7 operational):**

- Lifecycle: `orchestrator` (with closer collation logic + PASS/FAIL/HUMAN-DECISION-NEEDED rules), `devils-advocate` (parallel reviewer), `verifier`.
- Principle reviewers: `simplicity-reviewer`, `surgical-reviewer`, `think-before-coding-reviewer`, `goal-driven-reviewer` — each with explicit input contracts, output JSON schema, Round 1 + Round 3 behaviour, severity calibration, and forking discipline.

**Worked example calibration anchors:**

- `examples/docs/specflow/features/001-design-skill/` — full lifecycle: interview + PRD + Gate 2 manifest + tasks + Gate 3 manifest. Status: both gates passed.
- `examples/docs/specflow/features/002-develop-skill/` — dogfood artefact (recursive bootstrap on Phase 2 `specflow:develop`). Includes interview + PRD + Gate 2 manifest + dogfood debrief noting friction surfaced and recommended prompt edits.
- `examples/docs/specflow/admin/` — populated reference tree (config, profiles, environment, CONTEXT, decision-log, task-history, rules, agents) showing what a project looks like after a few months of use.

Phase 2 (`specflow:develop`, `specflow:agent`) and Phase 3 (`specflow:complete`, `specflow:decision`, `specflow:scope-change`, `/optimize`, `/insights`, `/prune`) ship as frontmatter stubs; their bodies arrive in the respective phase releases.

### Added
- `docs/specflow/admin/` folder for plugin configuration and per-repo memory.
- Feature-grouped directories under `docs/specflow/features/NNN-{slug}/` (PRD, tasks, tests, design, docs, assets all live together).
- `prd.html` static rendering of every PRD for browser-readable review (`specflow:render`).
- Project rules registry (`admin/rules/non-negotiable.md`, `guidelines.md`, `glossary.md`).
- The four behavioral principles (`CORE_PRINCIPLES.md`) loaded into every skill's system prompt.
- Standard agents split into two categories:
  - **Lifecycle** (`admin/agents/standard/lifecycle/`): Orchestrator, Devil's Advocate, Verifier — plan / challenge / confirm.
  - **Principle reviewers** (`admin/agents/standard/principles/`): Simplicity, Surgical, Think-Before-Coding, Goal-Driven — one per core principle. Fire as parallel reviewers in the debate manifest.
- **Multi-agent debate manifest** — every adversarial-review gate fires N reviewers in parallel into a shared manifest file. AI responds in round 2; reviewers sharpen or accept in round 3; Orchestrator writes the closing decision entry. Replaces the prior single-reviewer 3-iteration loop. **Manifests live inside the feature folder** (`features/NNN-{slug}/debate-log/{gate}/`) — co-located so all context for a feature is in one place. Cross-feature gates remain under `admin/debate-log/`.
- **PRD interview file** — every feature folder has `NNN-{slug}-interview.md` capturing original request, codebase context, grilling Q&A with reasoning per answer, and resolved assumptions. Written by `/grill` as a sub-skill of `specflow:prd`. PRD body references the interview by relative path but does not duplicate it. Markdown only — no HTML render (the PRD's HTML render links to it).
- **`specflow:prd` is now a multi-phase orchestrator** — owns the full flow from "I want to build X" to "PRD reviewed and signed off." Phases: A preamble, B grilling (invokes `/grill` as sub-skill), C synthesis, D render, E Gate 2 multi-agent debate manifest.
- **`NNN-{slug}-` filename prefix preserved** on every top-level feature file (`NNN-{slug}-prd.md`, `NNN-{slug}-tasks.md`, etc.) so multiple PRDs are distinguishable when open in editor tabs or surfacing in search.
- `specflow:upgrade` skill driving version-to-version migrations from `MIGRATIONS.md`.
- `specflow:design` skill — codebase-truth HTML/CSS mockups with Playwright visual diff loop, **decision-capture iteration log per Appendix C3.1** (every iteration records *Why* the change was made; empty *Why* is a verify-step failure).
- `specflow:misc` skill — single-task workflow for bugs and small fixes; interactive + auto invocation modes (calling skills like Surgical Reviewer pass structured payloads).
- `specflow:doctor` skill — read-only 5-category installation validator.
- `specflow:budget` skill — subscription-spend visibility AND per-skill context-window cost via append-only `skill-invocations.jsonl`. Trending-up Δ tokens are flagged as the early-warning signal for orchestrator-pattern violations.
- `panic` (snapshot-then-rewind with mandatory `yes, rewind` confirmation) and `confidence-check` (plain-language uncertainty declaration with ≥1 specific source) trust-ladder primitives.
- `feedback-loop-audit` skill + `admin/CONTEXT.md` template (≤700-line cap; preserves user-maintained blocks across regenerations).
- `simplify` autoresearch loop (Karpathy discipline-installer; branch-per-run, sequential variants, no LLM-as-judge inside the loop, $20/week cap, human merge owns taste).
- `/grill` skill — AI interrogates user one question at a time until alignment is reached. Refuses to start without a confirmed Goal section.
- Adversarial review chain Gates 1-3 (grill, PRD vs brief, tasks vs PRD); operational principle reviewers fire Round 1 findings in parallel, AI responds Round 2, reviewers sharpen-or-accept Round 3, Orchestrator writes the closing decision entry.
- `admin/environment.json` — first-class inventory of CLIs, MCPs, plugins, agents.
- `admin/profiles.json` — user personas, defined once and reused across PRDs.
- `admin/confidence-log.json` — every confidence-check declaration + response logged for Phase 3 self-learning.
- `admin/budget/skill-invocations.jsonl` — append-only per-skill self-reports for budget aggregation.
- `admin/simplify-runs.jsonl` — every simplify autoresearch run recorded with merge_decision tracked through human review.
- Path-scoped rules infrastructure; AGENTS.md auto-mirror commit hook.
- `SKILLS.md` glossary discipline — every skill must have an entry to ship.

### Changed
- `config.json` and `pages.json` move from `docs/specflow/` root into `admin/` (handled by upgrade migration).
- Flat `prd/` + `task/` + `test/` folders consolidated into per-feature directories (handled by upgrade migration).
- `specflow:setup` extended to seed the new layout, run profile interview, run environment inventory.
- `specflow:prd` extended with adversarial misalignment-scan (Gate 2).
- `specflow:task` extended with coverage matrix and adversarial pre-review (Gate 3).
- `specflow:test` extended with feature-folder-scoped assets.

### Removed
- The flat `prd/`, `task/`, `test/` top-level folders (consolidated into `features/`).
- Top-level `design/` folder (feature-specific mockups now live in the feature folder; cross-feature exploratory mockups remain optional).

---

## v1 history (pre-v2)

Pre-v2 changelog lives in the [v1 plugin folder](../../plugins/specflow/CHANGELOG.md) (or in `CHANGELOG.md` at repo root depending on where v1 maintained it). Not duplicated here.
