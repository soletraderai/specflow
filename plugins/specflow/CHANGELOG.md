# Changelog

All notable changes to specflow v2 are documented here. Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versioning: [SemVer](https://semver.org/).

## [Unreleased]

_The dogfood pass for 003-complete-skill produced E6-E10 prompt-edit recommendations (DOGFOOD-DEBRIEF). Apply in the next session, then build out the remaining Phase 3 skills (`specflow:decision`, `specflow:scope-change`, `/optimize`, `/insights`, `/prune`) using the chain. Phase 3 PRD specs for `decision` and `scope-change` are pre-drafted at `examples/.../features/{004-decision-skill,005-scope-change-skill}/`._

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
