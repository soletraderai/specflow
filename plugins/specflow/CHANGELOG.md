# Changelog

All notable changes to specflow v2 are documented here. Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versioning: [SemVer](https://semver.org/).

## [Unreleased]

**Phase 2 build staged.** `specflow:develop` (649-line orchestrator with Phases A-F: pre-flight, lane triage, mechanical pre-Gate-4 lane recheck, Gate 4 plan-vs-PRD debate, lane execution green/yellow/red, Gate 5 code-vs-plan with Codex degradation) and `specflow:agent` (232-line per-repo registry with `add | remove | list | refresh` verbs) now have operational bodies. The 002-develop-skill PRD is implemented end-to-end against 19 tasks (Gate 3 closed, status `passed-with-revisions`, 6 findings all `concern`, 1 push-back defended on the R5/R5.1 lane-recheck split).

The Phase 1 prompt-edit recommendations from the dogfood debrief (E1-E5) are now applied:
- E1: `skills/prd/SKILL.md` — Vision verbatim-vs-paraphrase contract codified for Gate 2 trace integrity.
- E2: `skills/prd/SKILL.md` — Phase C.3 self-check now cross-references ACs against Phase 1 skill schemas.
- E3: `skills/prd/SKILL.md` — Gate 2 status taxonomy extended with `passed-with-revisions` (distinguishing clean pass from revision-and-pass).
- E4: `templates/agents/standard/principles/goal-driven-reviewer.md` — reverse-traceability lens (orphan AC) added to anti-patterns.
- E5: `skills/task/SKILL.md` — Phase A.2 surfaces Gate 2 block-finding resolutions before extraction.

`MIGRATIONS.md` carries the v2.0 → v2.1 entry: introduces `config.json.develop.{greenBatchCap, codexAtGate5}` (defaults `3` and env-derived), the optional `stack_match_reason` field on `agents/index.json`, and the on-demand `develop-gate4/` / `develop-gate5/` debate-log subdirectories. Migration is additive; backups retained until next successful upgrade.

The 2.1.0 cut waits on real dogfood — running `specflow:develop` on an actual Phase 2 task in a consumer project to surface friction the synthetic worked example didn't. Phase 3 (`specflow:complete`, `specflow:decision`, `specflow:scope-change`, `/optimize`, `/insights`, `/prune`) remains stubbed.

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
