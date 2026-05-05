---
name: specflow:doctor
description: Validate the local installation. Five-category check pass — install layout, config schema, standard-agent set, feature integrity (PRD/HTML drift), environment + hard requirements. Produces a human-readable report with PASS / FAIL / WARN per check and concrete remediation for every failure. Invoked manually for confidence; auto-invoked by specflow:upgrade post-migration and (Phase 2) by specflow:develop on entry.
status: v2-new
phase: 1
requires:
  - docs/specflow/admin/config.json
  - docs/specflow/admin/environment.json
  - docs/specflow/admin/agents/index.json
  - plugins/specflow/.claude-plugin/plugin.json
produces: []
eval: report has one line per check; every FAIL line cites concrete file/line/symptom; every FAIL has a suggested-fix line; doctor exits with overall PASS when zero FAIL entries; standard-agent check verifies every file in templates/agents/standard/{lifecycle,principles}/ is present in admin/agents/standard/.

---

# specflow:doctor

Validate the local installation. Read-only: doctor never modifies state. It reports problems and points at the skill that would fix them (`specflow:upgrade`, `specflow:setup --repair`, manual edit).

This is a **3-phase skill**: read state in parallel, run all checks, collate the report.

---

## Inputs

The user invokes you with one of:
- `/specflow:doctor` — full check pass.
- `specflow:doctor --category {install|config|agents|features|environment}` — run only one category.
- `specflow:doctor --feature {NNN-slug}` — run only the per-feature checks for one feature.

Auto-invocations:
- `specflow:upgrade` invokes you after migration to confirm the upgrade landed cleanly.
- (Phase 2) `specflow:develop` invokes you on entry to refuse to run on a broken install.

---

## Phase A — Read state

Use Read in parallel on:
- `docs/specflow/admin/config.json`
- `docs/specflow/admin/environment.json`
- `docs/specflow/admin/agents/index.json` (if exists; absent in pre-Phase-2 installs is fine)
- `plugins/specflow/.claude-plugin/plugin.json`
- `docs/specflow/admin/rules/non-negotiable.md`
- `docs/specflow/admin/rules/guidelines.md`

Use Glob in parallel:
- `docs/specflow/features/*/` — feature folders.
- `docs/specflow/admin/agents/standard/lifecycle/*.md` — installed lifecycle agents.
- `docs/specflow/admin/agents/standard/principles/*.md` — installed principle reviewers.

Use Bash to capture environment facts the JSON might be stale on:
- `which playwright || echo missing` — the hard-required Phase 1 dep.
- `git rev-parse --show-toplevel` — confirm we're inside a repo.

If any required input file is missing, mark the corresponding check FAIL and continue (don't abort — partial reports still help).

---

## Phase B — Run checks

Run every check in scope. Each check returns a structured result:

```
{
  category: install | config | agents | features | environment,
  name: "{short check name}",
  status: PASS | FAIL | WARN,
  detail: "{one-line observation}",
  fix: "{concrete remediation, only when status != PASS}"
}
```

### B.1 Install layout

- **`install.folders`** — all required folders exist:
  - `docs/specflow/admin/`
  - `docs/specflow/admin/agents/standard/lifecycle/`
  - `docs/specflow/admin/agents/standard/principles/`
  - `docs/specflow/admin/agents/specialised/`
  - `docs/specflow/admin/rules/`
  - `docs/specflow/features/`
  - `docs/specflow/misc-task/`
  - `docs/specflow/docs/`

  Each missing folder = one FAIL with fix `Run /specflow:upgrade or specflow:setup --repair to recreate scaffolding.`

- **`install.scratch_clean`** — `docs/specflow/admin/scratch/` is either absent or contains nothing older than 24h. Stale scratch directories indicate a previous orchestration crashed without cleanup. WARN, not FAIL. Fix: `rm -rf docs/specflow/admin/scratch/{name}` after confirming no orchestration is in flight.

### B.2 Config schema

- **`config.exists`** — `admin/config.json` parses as JSON.
- **`config.specflow_version`** — `specflowVersion` is present.
- **`config.version_match`** — `specflowVersion` matches `plugin.json.version`. Mismatch = WARN with fix `Run /specflow:upgrade to apply the {plugin-version} migration chain.`
- **`config.linear_misc_project`** — if `config.linear` is set, `linear.miscProject` is non-empty. WARN if missing (the `specflow:misc` skill has a default but the config field documents intent).

### B.3 Standard-agent set

- **`agents.lifecycle`** — every file in `templates/agents/standard/lifecycle/*.md` (the plugin's source-of-truth set) exists in `admin/agents/standard/lifecycle/`. The expected names:
  - `orchestrator.md`
  - `devils-advocate.md`
  - `verifier.md`

  Each missing file = one FAIL with fix `Run /specflow:upgrade — the standard agent set has new entries since this installation.`

- **`agents.principles`** — every file in `templates/agents/standard/principles/*.md` exists in `admin/agents/standard/principles/`. Expected names:
  - `simplicity-reviewer.md`
  - `surgical-reviewer.md`
  - `think-before-coding-reviewer.md`
  - `goal-driven-reviewer.md`

  Each missing file = one FAIL with fix `Run /specflow:upgrade.`

- **`agents.index_integrity`** — only if `admin/agents/index.json` exists (Phase 2+). For each agent referenced there, the source plugin still resolves. (Phase 1 stub: SKIP this check if the index file is absent; report status `SKIP` rather than `WARN`.)

### B.4 Feature integrity

For each `features/NNN-{slug}/` folder:

- **`features.{NNN-slug}.prd_exists`** — `{NNN-slug}-prd.md` exists. Missing = FAIL with fix `Feature folder exists without a PRD; run specflow:prd {NNN-slug} or remove the empty folder.`

- **`features.{NNN-slug}.interview_exists`** — `{NNN-slug}-interview.md` exists. Missing = FAIL with fix `PRD exists without an interview file; the install is partial. Run /specflow:upgrade or recreate via specflow:prd {NNN-slug}.`

- **`features.{NNN-slug}.html_drift`** — if `{NNN-slug}-prd.html` exists, its mtime is no older than `{NNN-slug}-prd.md`'s mtime. Older = WARN with fix `Run /specflow:render {NNN-slug} — the HTML render is stale.`

- **`features.{NNN-slug}.gate2_closed`** — if a `debate-log/prd-gate2/manifest.md` exists, it has a `Gate 2 status: **passed**` or `**passed-with-escalations**` line. Other status = WARN, missing manifest = WARN. Fix: `Re-run specflow:prd {NNN-slug} — Gate 2 is incomplete.`

- **`features.{NNN-slug}.gate3_closed`** — same shape, for `debate-log/tasks-gate3/manifest.md`. Tasks file exists but no gate3 manifest = FAIL.

(If `--feature {NNN-slug}` was provided, scope this category to just that one.)

### B.5 Environment + hard requirements

- **`environment.json_exists`** — `admin/environment.json` exists and parses.
- **`environment.playwright`** — `tools.playwright.available` is `true` AND `which playwright` returns a path. Both must hold; if `environment.json` says present but the binary is missing, FAIL with fix `Run /specflow:upgrade or reinstall Playwright; environment.json is out of date.`
- **`environment.codex`** — soft requirement. Absent = WARN with fix `Codex CLI enables higher-stakes adversarial review at Gates 5-6. Install via {wherever}.`
- **`environment.repo`** — `git rev-parse --show-toplevel` returns a non-empty path. If specflow is being used outside a git repo, WARN with fix `Initialize a git repo; specflow's audit trail (PRDs, interviews, manifests) assumes git history.`

---

## Phase C — Collate the report

Surface the report in chat. Format:

```
## specflow:doctor — {full | category: agents | feature: NNN-slug}
Run at {YYYY-MM-DD HH:MM}.

### Install
- ✅ install.folders → all 8 required folders present
- ⚠️ install.scratch_clean → admin/scratch/old-orchestration-2026-04-12/ is 23 days old
  Fix: rm -rf docs/specflow/admin/scratch/old-orchestration-2026-04-12/

### Config
- ✅ config.exists → admin/config.json parses
- ✅ config.specflow_version → 2.0.0-dev
- ❌ config.version_match → installed plugin is 2.0.0-dev, config says 1.9.2
  Fix: Run /specflow:upgrade to apply the 1.9.2 → 2.0.0-dev migration chain.

### Agents (standard set)
- ✅ agents.lifecycle → orchestrator, devils-advocate, verifier all present
- ✅ agents.principles → all 4 principle reviewers present
- ⏭ agents.index_integrity → skipped (index.json absent; Phase 2+ check)

### Features
- ✅ features.001-design-skill.prd_exists
- ✅ features.001-design-skill.interview_exists
- ⚠️ features.001-design-skill.html_drift → prd.html is 6 days older than prd.md
  Fix: Run /specflow:render 001-design-skill.
- ✅ features.001-design-skill.gate2_closed → passed
- ⏭ features.001-design-skill.gate3_closed → tasks file absent (skip)

### Environment
- ✅ environment.json_exists
- ✅ environment.playwright → 1.49.0 detected
- ⚠️ environment.codex → not installed
  Fix: Codex CLI enables higher-stakes adversarial review at Gates 5-6.
- ✅ environment.repo → /Users/.../specflow

---
**Summary:** 9 PASS · 3 WARN · 1 FAIL · 2 SKIP

❌ 1 failure must be addressed:
- config.version_match → Run /specflow:upgrade.

⚠️ 3 warnings (non-blocking):
- install.scratch_clean
- features.001-design-skill.html_drift
- environment.codex

Overall status: **FAIL** (run /specflow:upgrade and re-check).
```

### C.1 Overall status rules

- **PASS** — zero FAIL, zero WARN.
- **PASS-WITH-WARNINGS** — zero FAIL, ≥1 WARN.
- **FAIL** — ≥1 FAIL.

The summary line names the overall status and (for FAIL) the single most important remediation step. For PASS, end with: *"Install is healthy."*

### C.2 Auto-invocation result

When invoked by another skill (`specflow:upgrade` or `specflow:develop`):
- Write the same report content to `admin/scratch/doctor-report-{timestamp}.md`.
- Return the file path + the overall status (one line: `{path} — {status}`).
- The calling skill reads the path via command substitution to surface failures in its own output.

---

## What you MUST NOT do

- **Do not modify any files.** Doctor is read-only by contract. Failures must be remediated by the user via the suggested-fix command, not by doctor inline.
- **Do not invoke `specflow:upgrade` automatically.** Suggest it; never run it.
- **Do not skip a check because it would FAIL.** Every check that's in scope runs; partial reports are useless.
- **Do not invent fixes.** If a failure has no clear remediation path, write `Fix: {investigation needed — surface to the maintainer}`. Do not fabricate a step.
- **Do not mention Claude, Anthropic, or any AI tooling** in the report or any output. Per the project's CLAUDE.md, this is non-negotiable.

---

## Verify before declaring done

1. Report has one line per check that ran (PASS / FAIL / WARN / SKIP all visible).
2. Every FAIL and WARN has a Fix: line with a concrete command.
3. Summary line accurately counts each status.
4. Overall status matches the rules in C.1.
5. (Auto-invocation only) result file written and path returned.

If any verify step fails, fix the report before returning.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 9 — `specflow:doctor` (Phase 1 implicit, full skill in Phase 2).
- `docs/PRD.md` Appendix H3 — full doctor spec.
- `docs/PRD.md` Appendix M — environment inventory schema (for `environment.json` checks).
- `skills/upgrade/SKILL.md` — primary remediation target for FAIL findings.
- `skills/setup/SKILL.md` — `--repair` mode for scaffolding-level FAILs.
