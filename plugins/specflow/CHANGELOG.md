# Changelog

All notable changes to specflow v2 are documented here. Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versioning: [SemVer](https://semver.org/).

## [Unreleased]

_v2.4.0 closed Sprint 1 of the v2.4+ master plan. Sprint 2 (template + doctrine churn — `015-key-features-section`, `016-brief-enhancements`, `017-tdd-discipline`) is the next milestone. The first `templateVersion` audit fires at Sprint 2 close per `templates/admin/example-versioning.md`._

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
