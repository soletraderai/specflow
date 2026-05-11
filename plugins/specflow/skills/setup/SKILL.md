---
name: specflow:setup
description: Initialise specflow in a project — folder layout, environment inventory, profile interview, rules registry seeding, standard agents copy. First-run only; specflow:upgrade handles version-to-version migrations after.
status: v2-enhancement
phase: 1
requires: []
produces:
  - docs/specflow/admin/config.json
  - docs/specflow/admin/pages.json
  - docs/specflow/admin/profiles.json
  - docs/specflow/admin/environment.json
  - docs/specflow/admin/CONTEXT.md
  - docs/specflow/admin/decision-log.md
  - docs/specflow/admin/task-history.json
  - docs/specflow/admin/lessons.json
  - docs/specflow/admin/rules/non-negotiable.md
  - docs/specflow/admin/rules/guidelines.md
  - docs/specflow/admin/rules/glossary.md
  - docs/specflow/admin/agents/standard/lifecycle/
  - docs/specflow/admin/agents/standard/principles/
  - docs/specflow/admin/agents/specialised/
  - docs/specflow/admin/agents/index.json
  - docs/specflow/admin/debate-log/
  - docs/specflow/features/
  - docs/specflow/misc-task/
  - docs/specflow/misc-task/000-tasks-misc-tasks.md
  - docs/specflow/docs/
eval: admin/environment.json populated with detected stack and hard requirements satisfied (Playwright); every required folder exists; standard agents copied (3 lifecycle + 4 principle); rules registry seeded with starter set; profiles.json has at least one profile; config.json.specflowVersion stamped to plugin.json.version.
---

# specflow:setup

You initialise specflow in a project — first-run only. Once setup completes, `specflow:upgrade` handles every subsequent version-to-version change.

The shape you create is the substrate every other specflow skill depends on. Get it wrong and downstream skills (`specflow:prd`, `specflow:task`, `specflow:develop`) will fail loudly or silently. The verify steps at the end are non-negotiable.

---

## Triggers

- *"set up specflow"*
- *"/specflow:setup"*
- First-run detection: a specflow skill was invoked but `docs/specflow/admin/` doesn't exist.

If `docs/specflow/admin/` already exists when you're invoked, STOP and tell the user: *"This project already has specflow set up at `docs/specflow/admin/`. Use `specflow:upgrade` to refresh, or `specflow:doctor` to diagnose. If you want to re-run setup from scratch, delete `docs/specflow/` first (DESTRUCTIVE — back up first)."*

---

## Phase 1 — Hard requirements check

Hard requirements MUST be satisfied or setup aborts (loudly, with a clear remediation message).

### 1.1 Playwright CLI

Check via Bash:

```sh
command -v playwright >/dev/null 2>&1 || npx playwright --version >/dev/null 2>&1
```

If both fail: STOP and tell the user *"Playwright CLI is required and not detected. Install with `npm install -g playwright` (or `npx playwright install` for project-local). Then re-run `/specflow:setup`."* Do not proceed.

### 1.2 Git repository

Check via Bash:

```sh
git rev-parse --git-dir >/dev/null 2>&1
```

If fails: STOP and tell the user *"specflow requires a git repository. Run `git init` then re-run `/specflow:setup`."*

---

## Phase 2 — Soft requirements detection

Detect everything specflow can use but doesn't strictly need. Each detection feeds `admin/environment.json`. Soft requirements never abort setup; they degrade behaviour gracefully.

### 2.1 Codex CLI

```sh
command -v codex >/dev/null 2>&1 && codex --version
```

Capture: `available: true|false`, `version: "{version}"` if present. Codex enables cross-provider adversarial review at Gates 5 and 6.

### 2.2 MCPs

Read `~/.claude.json` (or platform equivalent — check `~/Library/Application Support/Claude/` on macOS) and enumerate installed MCP servers. For each, capture name, scope hint (e.g. Linear → `task-export, misc-export`).

If the file is unreadable: log `mcp: []` and continue. Do not abort.

### 2.3 Other CLIs

Check for `gh`, `git`, `linear-cli` (the official Linear CLI, not the MCP). Capture available + version where present.

### 2.4 Installed plugins

Glob `~/.claude/plugins/*/.claude-plugin/plugin.json` and capture each as `{name, version}`. Specflow itself (the plugin running this skill) should appear in the list.

### 2.5 Available agents

For each installed plugin, glob `agents/**/*.md` and `templates/agents/**/*.md`. Capture each as `{name, source: plugin-name, scope: standard|specialised}`. The standard set is what each plugin ships in `templates/agents/standard/`; everything else is specialised.

---

## Phase 3 — Tech stack detection

Inspect the codebase to detect the project's stack. This drives the specialised agent proposal in Phase 7.

Use Glob + Read on the repo root:
- `package.json` → Node, with framework detection from dependencies (React, Vue, Next, Nuxt, Express, etc.).
- `requirements.txt` / `pyproject.toml` → Python, with framework detection (FastAPI, Django, Flask, etc.).
- `Cargo.toml` → Rust.
- `go.mod` → Go.
- `Gemfile` → Ruby.
- `pom.xml` / `build.gradle` → JVM.
- `*.csproj` → .NET.
- Database hints from dependency lists (postgres, mysql, mongodb, sqlite, prisma, drizzle, sequelize, etc.).
- Auth surface hints (auth0, clerk, supabase, next-auth, passport, etc.).
- Test framework (jest, vitest, pytest, mocha, playwright, cypress, etc.).

Capture as `stack: { language, framework, database, auth, test }` — null where not detected.

---

## Phase 4 — Folder scaffold

Create the v2 layout. Use Bash:

```sh
mkdir -p docs/specflow/admin/agents/standard/lifecycle
mkdir -p docs/specflow/admin/agents/standard/principles
mkdir -p docs/specflow/admin/agents/specialised
mkdir -p docs/specflow/admin/rules
mkdir -p docs/specflow/admin/debate-log
mkdir -p docs/specflow/admin/scratch
mkdir -p docs/specflow/features
mkdir -p docs/specflow/misc-task/assets
mkdir -p docs/specflow/docs
```

Verify each folder exists before continuing:

```sh
test -d docs/specflow/admin/agents/standard/lifecycle && \
test -d docs/specflow/admin/agents/standard/principles && \
test -d docs/specflow/admin/agents/specialised && \
test -d docs/specflow/admin/rules && \
test -d docs/specflow/admin/debate-log && \
test -d docs/specflow/features && \
test -d docs/specflow/misc-task/assets && \
test -d docs/specflow/docs
```

If any fails: STOP, report which folder couldn't be created, and surface the underlying filesystem error.

---

## Phase 5 — Copy templates from the plugin

The plugin's `templates/` directory contains files setup actively copies into the project. Find the plugin root (`${CLAUDE_PLUGIN_ROOT}` environment variable; or locate via `~/.claude/plugins/specflow/`) and copy:

### 5.1 Standard agents — lifecycle

Copy each file from `${PLUGIN_ROOT}/templates/agents/standard/lifecycle/` to `docs/specflow/admin/agents/standard/lifecycle/`:
- `orchestrator.md`
- `devils-advocate.md`
- `verifier.md`

### 5.2 Standard agents — principles

Copy each file from `${PLUGIN_ROOT}/templates/agents/standard/principles/` to `docs/specflow/admin/agents/standard/principles/`:
- `simplicity-reviewer.md`
- `surgical-reviewer.md`
- `think-before-coding-reviewer.md`
- `goal-driven-reviewer.md`

### 5.3 CONTEXT.md template

Copy `${PLUGIN_ROOT}/templates/admin/CONTEXT.md` to `docs/specflow/admin/CONTEXT.md`. Tell the user: *"`docs/specflow/admin/CONTEXT.md` is the slim live context document. The `feedback-loop-audit` skill will populate it later, or you can edit it now."*

### 5.4 Rules registry seeds

Copy each:
- `${PLUGIN_ROOT}/templates/admin/rules/non-negotiable.md` → `docs/specflow/admin/rules/non-negotiable.md`
- `${PLUGIN_ROOT}/templates/admin/rules/guidelines.md` → `docs/specflow/admin/rules/guidelines.md`
- `${PLUGIN_ROOT}/templates/admin/rules/glossary.md` → `docs/specflow/admin/rules/glossary.md`

After copying, present the starter non-negotiable rules to the user (read the file you just wrote and list them by ID + one-line description). Ask: *"These five non-negotiable rules ship as a starter set. Accept all, edit specific ones, or skip any?"*

For each rule the user edits or skips, update `non-negotiable.md` accordingly. Removed rules are deleted; edited rules have their body updated; the glossary entry stays in `glossary.md` even if the rule is removed (with a note "removed at setup").

### 5.5 Misc-task seed file

Create `docs/specflow/misc-task/000-tasks-misc-tasks.md` with this preamble:

```markdown
---
type: misc-tasks
linear_project: 000-misc-tasks
---

# Misc tasks

Single-task workflow for bugs and small fixes that don't warrant a PRD. Tasks accumulate here; the `specflow:misc` skill manages entries.

## Quick reference

| ID | Title | Scope | Status | Linear |
|----|-------|-------|--------|--------|

## Pending tasks

(no pending tasks — `/specflow:misc` will append entries here)
```

---

## Phase 6 — Profile interview

Profiles define the actors that show up in PRD user stories. Define them once, reuse everywhere.

### 6.1 Read profile examples

Read `${PLUGIN_ROOT}/templates/profile-examples.json` (curated set: Admin/operator, End user, Power user, Support, Developer/API consumer, Finance, Auditor, Field worker).

### 6.2 Propose a starter set

Based on the detected tech stack from Phase 3, propose 3-5 relevant profiles. Examples:
- Stack has admin surface (auth, multi-tenant, RBAC) → propose Admin/Operator + End User + Support.
- Stack has external API → propose Developer/API Consumer.
- Stack has billing surface (Stripe, billing schema) → propose Finance.
- Stack has compliance surface (audit logs, retention, RBAC strict) → propose Auditor.
- Stack has mobile or field-work hint (React Native, Flutter, geolocation deps) → propose Field Worker.

Present the proposed set with each profile's name + role + 2-3 bullet goals. Ask: *"These {N} profiles look relevant to your stack. Accept all, edit, add, or replace?"*

### 6.3 Iterate

For each profile the user accepts, edits, or adds: capture name, role, goals (list), constraints (list), painPoints (list).

If the user wants to add a custom profile, ask the four fields one at a time.

### 6.4 Auto-generate option

If the user asks "you decide" or wants a quick start: read the codebase (CLAUDE.md if present, auth config files, role definitions, any RBAC code) and propose a full set. Present back for one-shot confirmation.

### 6.5 Write profiles.json

Write `docs/specflow/admin/profiles.json`:

```json
{
  "profiles": [
    {
      "name": "{canonical name used in As a {name} stories}",
      "role": "{one sentence}",
      "goals": ["...", "..."],
      "constraints": ["...", "..."],
      "painPoints": ["...", "..."]
    },
    ...
  ]
}
```

---

## Phase 7 — Specialised agent proposal

Based on the tech stack from Phase 3, propose specialised agents from the available agent set (Phase 2.5). Examples:
- React/Vue/Next detected → propose `frontend-developer` (from `frontend-mobile-development` plugin if available).
- Postgres-heavy → propose `database-architect`.
- Auth surface or compliance hints → propose `security-auditor`.
- Backend-heavy Node → propose `backend-architect`.

For each proposed specialist, present: agent name + source plugin + why this stack matches. Ask: *"Add these specialised agents to your project? (yes / pick subset / skip)"*

For each accepted agent, copy its definition file from the source plugin into `docs/specflow/admin/agents/specialised/`. Use the namespaced filename `{plugin-name}__{agent-name}.md` to avoid cross-marketplace collisions.

---

## Phase 8 — Write environment.json and admin metadata

### 8.1 environment.json

Write `docs/specflow/admin/environment.json` with everything captured in Phase 2 + Phase 3:

```json
{
  "lastDetected": "{ISO-8601 timestamp}",
  "specflowVersion": "{from plugin.json.version}",
  "stack": {
    "language": "...",
    "framework": "...",
    "database": "...",
    "auth": "...",
    "test": "..."
  },
  "cli": [
    {"name": "playwright", "available": true, "version": "...", "required": true, "uses": ["specflow:design", "specflow:test"]},
    {"name": "codex", "available": true|false, "version": "...", "required": false, "uses": ["adversarial-review"]},
    ...
  ],
  "mcp": [
    {"name": "linear", "available": true|false, "scope": ["task-export", "misc-export"]},
    ...
  ],
  "plugins": [
    {"name": "specflow", "version": "..."},
    ...
  ],
  "agents": [
    {"name": "orchestrator", "source": "specflow", "scope": "standard", "category": "lifecycle"},
    {"name": "simplicity-reviewer", "source": "specflow", "scope": "standard", "category": "principles"},
    ...
  ]
}
```

### 8.2 config.json

Before writing, prompt the user for the brief commit policy:

> *"`specflow:brief` produces `features/NNN-{slug}/NNN-{slug}-brief.html` — a self-contained HTML composition of PRD + interview + manifests, suited to PR review. Commit policy?*
> *(1) `committed` — recommended; the brief is the diffable surface reviewers reach for.*
> *(2) `derived` — gitignored; `specflow:brief --all` regenerates from sources. Pick this if your repo is sensitive to size."*

Default is `committed`. If the user picks `derived`, append the line `*-brief.html` to `.gitignore` (create `.gitignore` if absent; idempotent — never duplicate).

Then prompt for the per-task duration cap:

> *"How long should a single specflow task take? Smaller is easier to review and undo. Pick `1` / `4` / `8` hours, or `auto` to let the token budget alone size tasks."*

Map the user's choice to `task.maxDurationHours`: `1 | 4 | 8 | "auto"`. Default on skip or malformed input: `1`. Re-prompt on any other value.

Then write `docs/specflow/admin/config.json`:

```json
{
  "specflowVersion": "{from plugin.json.version}",
  "linear": {
    "miscProject": "000-misc-tasks"
  },
  "confidentialPaths": [],
  "brief": {
    "commitPolicy": "committed"
  },
  "task": {
    "contextBudget": 80000,
    "maxLessonsSurfaced": 5,
    "maxDurationHours": 1
  },
  "prd": {
    "maxLessonsSurfaced": 5
  },
  "develop": {
    "maxIssuesPerSprint": 5,
    "tddRequired": true
  },
  "skills": {
    "prd":          { "enabled": true },
    "task":         { "enabled": true },
    "develop":      { "enabled": true },
    "test":         { "enabled": true },
    "complete":     { "enabled": true },
    "decision":     { "enabled": true },
    "scope-change": { "enabled": true },
    "design":       { "enabled": true },
    "brief":        { "enabled": true },
    "misc":         { "enabled": true },
    "agent":        { "enabled": true },
    "insights":     { "enabled": true },
    "prune":        { "enabled": true },
    "optimize":     { "enabled": true },
    "sprint":       { "enabled": true }
  },
  "teams": {}
}
```

`confidentialPaths` is the rule-based confidentiality globs that drive Red-lane assignment in `specflow:develop`. Seed empty; populate when the user knows what their auth/secrets/billing/schema surfaces are.

`brief.commitPolicy` ∈ `committed | derived`. Default `committed`. The choice is read by `specflow:brief` to emit a one-line policy banner in the rendered HTML and to drive the `.gitignore` write at setup time. Resolution citation: `v2/docs/PRD.md` § Resolved decisions — 011-brief-commit-policy v2.4.0.

`task.contextBudget` is the per-task token ceiling for the single-context-window rule (per 029-single-context-task v2.5.0). Default `80000` — sits inside the smart zone with headroom for tool-call output. Tasks whose `context-budget-estimate` exceeds this value auto-flag at synthesis (`specflow:task` Phase B.4) and route to `specflow:scope-change` for splitting. Lower the value on token-sensitive providers; raise it cautiously and only after empirical evidence the smart zone is wider than 80K on your stack. Estimation algorithm and no-mid-task-compaction contract live in `templates/admin/single-context-task.md`.

`task.maxDurationHours` is the soft sizing target per task in hours — `1 | 4 | 8 | "auto"`; default `1`. Captured at setup-time via the 8.2 prompt. `specflow:task` aims for tasks within the cap during synthesis; when `"auto"`, sizing falls back to `contextBudget` alone.

`skills.{name}.enabled` toggles individual skills on/off at the project level. Default `true` for every shipped skill. Each skill checks its toggle in its Phase A pre-flight and refuses with a one-line message when disabled (e.g. *"`specflow:develop` is disabled in this project (config.skills.develop.enabled = false). Re-enable in admin/config.json or invoke a different skill."*). The resolver contract is documented in `templates/admin/skill-toggles.md`. Setup leaves all skills enabled by default; users disable selectively (for example, an org that uses GitHub Copilot for implementation might disable `specflow:develop` but keep `specflow:prd` + `specflow:task` + `specflow:test`). Resolution citation: `v2/docs/PRD.md` § Resolved decisions — 012-config-skill-toggles v2.4.0.

### 8.3 pages.json

Write `docs/specflow/admin/pages.json` with a placeholder structure. The placeholder is lazy-populated by `specflow:test` on the first UI run for a feature — `specflow:test` Phase C captures rendered routes and appends new entries to `pages.json`. No dedicated `specflow:pages` skill ships; the lazy-population path is the canonical inventory mechanism.

```json
{
  "routes": []
}
```

### 8.4 decision-log.md, task-history.json, and lessons.json

Write `docs/specflow/admin/decision-log.md`:

```markdown
# Decision log

Per-project decision-of-record. Captures key architectural and workflow decisions, what worked, what didn't, improvements identified. Updated by `specflow:decision`, `specflow:complete`, and `specflow:scope-change`.

(no entries yet — the log accumulates as decisions are made)
```

Write `docs/specflow/admin/task-history.json`:

```json
{
  "tasks": []
}
```

Write `docs/specflow/admin/lessons.json`:

```json
[]
```

`lessons.json` is the project's self-learning corpus. It accumulates as `specflow:test --feedback` captures gaps that escaped the gates, supersessions when a previously-failed approach later works, and rule-promotion candidates when a lesson recurs 3+ times. `specflow:test` Phase B.0 and `specflow:task` A.4 query it on every relevant invocation. Schema and lifecycle defined in `skills/test/SKILL.md` § Lessons registry.

### 8.5 Agent index

Write `docs/specflow/admin/agents/index.json` enumerating every agent now in `admin/agents/`:

```json
{
  "agents": [
    {"name": "orchestrator", "scope": "standard", "category": "lifecycle", "source": "specflow", "snapshotDate": "{today}", "sourceVersion": "{plugin version}"},
    {"name": "simplicity-reviewer", "scope": "standard", "category": "principles", "source": "specflow", "snapshotDate": "{today}", "sourceVersion": "{plugin version}"},
    ...
  ]
}
```

---

## Phase 9 — Verify and report

Run every verify step. If any fails, do NOT report success — fix and re-verify, or surface the failure to the user.

### 9.1 Folder layout

```sh
test -d docs/specflow/admin/agents/standard/lifecycle && \
test -d docs/specflow/admin/agents/standard/principles && \
test -d docs/specflow/admin/agents/specialised && \
test -d docs/specflow/admin/rules && \
test -d docs/specflow/admin/debate-log && \
test -d docs/specflow/features && \
test -d docs/specflow/misc-task/assets && \
test -d docs/specflow/docs
```

### 9.2 Required files

```sh
test -f docs/specflow/admin/config.json && \
test -f docs/specflow/admin/pages.json && \
test -f docs/specflow/admin/profiles.json && \
test -f docs/specflow/admin/environment.json && \
test -f docs/specflow/admin/CONTEXT.md && \
test -f docs/specflow/admin/decision-log.md && \
test -f docs/specflow/admin/task-history.json && \
test -f docs/specflow/admin/lessons.json && \
test -f docs/specflow/admin/rules/non-negotiable.md && \
test -f docs/specflow/admin/rules/guidelines.md && \
test -f docs/specflow/admin/rules/glossary.md && \
test -f docs/specflow/admin/agents/index.json && \
test -f docs/specflow/misc-task/000-tasks-misc-tasks.md
```

### 9.3 Standard agents copied

```sh
test -f docs/specflow/admin/agents/standard/lifecycle/orchestrator.md && \
test -f docs/specflow/admin/agents/standard/lifecycle/devils-advocate.md && \
test -f docs/specflow/admin/agents/standard/lifecycle/verifier.md && \
test -f docs/specflow/admin/agents/standard/principles/simplicity-reviewer.md && \
test -f docs/specflow/admin/agents/standard/principles/surgical-reviewer.md && \
test -f docs/specflow/admin/agents/standard/principles/think-before-coding-reviewer.md && \
test -f docs/specflow/admin/agents/standard/principles/goal-driven-reviewer.md
```

### 9.4 JSON validity

Read each `.json` file and confirm it parses. If any file is malformed, fix it (you wrote them — read back and validate the JSON syntax).

### 9.5 environment.json contents

Read `docs/specflow/admin/environment.json` and verify:
- `cli` array contains `playwright` with `available: true`.
- `stack` is populated (at least `language` is non-null).
- `agents` array has at least 7 entries (3 lifecycle + 4 principles).

### 9.6 profiles.json contents

Read and verify:
- `profiles` array has at least 1 entry.
- Each profile has `name`, `role`, `goals` (non-empty), `constraints` (non-empty), `painPoints` (non-empty).

### 9.7 Stamp version

Verify `docs/specflow/admin/config.json.specflowVersion` matches `${PLUGIN_ROOT}/.claude-plugin/plugin.json.version`. If they differ, fix `config.json` and re-verify.

### 9.8 Final report

Tell the user, in this exact order:

```
specflow setup complete.

Layout: docs/specflow/{admin, features, misc-task, docs}
Stack detected: {language} / {framework} / {database} / {auth} / {test}
Hard requirements: Playwright {version} ✓
Soft requirements: {Codex available|absent}, {N MCPs}, {N plugins}
Standard agents: 3 lifecycle + 4 principle reviewers copied
Specialised agents: {list}
Profiles: {N defined: comma-separated names}
Rules: {N non-negotiable rules seeded}

Worked example to browse: ${PLUGIN_ROOT}/examples/

Next steps:
  specflow:prd {your first feature overview}    — generate your first PRD
  specflow:doctor                                — re-validate at any time
```

If any soft requirement was missing (Codex, Linear MCP, etc.), include a one-line note: *"Codex not detected — adversarial review will degrade gracefully. Install with `npm install -g @openai/codex` to enable cross-provider review."*

---

## What you MUST NOT do

- **Do not skip the Playwright check.** It is a hard requirement; setup aborts without it. Do not silently continue.
- **Do not overwrite an existing `docs/specflow/admin/`.** That's destructive. Redirect to `specflow:upgrade` or `specflow:doctor`.
- **Do not auto-accept the rules registry without showing the user the starter set.** They need to know what's being applied to their codebase.
- **Do not invent profiles the user didn't ask for.** Propose, get confirmation. If they say "you decide", read the codebase and propose — but still get final confirmation before writing.
- **Do not write empty arrays where a populated one is expected.** `profiles: []`, `agents: []`, `cli: []` all fail verification. If detection genuinely returns nothing, surface that to the user as a warning before completing setup.
- **Do not mention the underlying AI tooling or vendor** in any user-facing output, in the seeded files, or in the report. Per the project's CLAUDE.md, this is non-negotiable.

---

## Reference

- `docs/PRD.md` Appendix A3 — setup folder creation spec.
- `docs/PRD.md` Appendix F — profile interview spec.
- `docs/PRD.md` Appendix M — environment inventory spec.
- `docs/PRD.md` Appendix O6 — rules registry starter set.
- `docs/PRD.md` Appendix K — agent registry spec.
- `templates/profile-examples.json` — the profile starter library.
- `templates/admin/rules/{non-negotiable,guidelines,glossary}.md` — rule registry seeds.
- `templates/admin/CONTEXT.md` — slim context doc template.
- `templates/agents/standard/{lifecycle,principles}/` — standard agents copied at setup.
