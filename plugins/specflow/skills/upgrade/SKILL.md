---
name: specflow:upgrade
description: Refresh an aged specflow installation without losing customisation. Drives version-to-version migrations from MIGRATIONS.md.
status: v2-new
phase: 1
requires: [docs/specflow/admin/config.json, plugin.json, MIGRATIONS.md]
produces: [docs/specflow/admin/config.json, docs/specflow/admin/environment.json]
eval: admin/config.json.specflowVersion matches plugin.json.version; specflow:doctor reports no failures; backups (.bak) exist for every modified file.
---

# specflow:upgrade

Refresh an existing `docs/specflow/` installation to the installed plugin version without losing project customisation. Migrations are versioned, explicit, resumable, and driven by `MIGRATIONS.md`.

Never skip backups. Never move, overwrite, or modify a file until a sibling `{path}.bak` exists for that exact file.

## Triggers

- `upgrade specflow`
- `refresh specflow config`
- `/specflow:upgrade`
- `/specflow:upgrade --clean-backups`

## Preflight

1. Locate roots.
   - Project state root: `docs/specflow/`.
   - Plugin root: the installed `specflow` plugin directory containing `MIGRATIONS.md`, `templates/`, and `plugin.json`.
   - If plugin metadata lives in a nested metadata folder, read the nearest `plugin.json` that contains a top-level `version`.
   - Verify: `test -d docs/specflow && test -f "$PLUGIN_ROOT/MIGRATIONS.md"`.

2. Check hard and soft environment requirements.
   - Playwright is mandatory for v2. If `command -v playwright` fails and `npx playwright --version` also fails, hard abort before presenting the migration plan.
   - Codex is optional. If absent, add a warning to the plan and continue.
   - Verify:
     ```sh
     command -v playwright >/dev/null 2>&1 || npx playwright --version >/dev/null 2>&1
     ```

3. Detect version skew.
   - Current version: read `docs/specflow/admin/config.json.specflowVersion` when present.
   - v1 fallback: if `admin/config.json` is missing but `docs/specflow/config.json` exists, read `docs/specflow/config.json.specflowVersion`; if absent, treat as `1.x`.
   - Target version: read `plugin.json.version`.
   - If current equals target, print no-op status. If `--clean-backups` is present, remove `.bak` files only after user confirmation.
   - Verify: both current and target are non-empty strings.

4. Compute migration chain.
   - Parse `MIGRATIONS.md` headings in order.
   - Select entries whose version range starts after the current version and ends at or before the target version.
   - For the first concrete migration, map any `1.x` current version to `v1.x -> v2.0`.
   - Abort if no complete chain reaches the target version.
   - Verify: selected chain begins at current and ends at target.

5. Present a checkbox plan and pause.
   - Print every migration entry and every step below as an unchecked item.
   - Include counts of PRDs, task files, test files, asset folders, and config files that will move.
   - Include warnings for optional missing tools.
   - Ask the user to confirm before applying any file operation.
   - Do not continue until the user explicitly confirms.

## v1.x -> v2.0 Migration

Run these steps in order. Each step is idempotent and has a verify clause so a partial migration can be resumed.

### 1. Inventory and collision check

Build a relocation manifest from the v1 tree:

- `docs/specflow/config.json` -> `docs/specflow/admin/config.json`
- `docs/specflow/pages.json` -> `docs/specflow/admin/pages.json`
- `docs/specflow/prd/NNN-{slug}.md` -> `docs/specflow/features/NNN-{slug}/NNN-{slug}-prd.md`
- `docs/specflow/task/NNN-tasks-{slug}.md` -> `docs/specflow/features/NNN-{slug}/NNN-{slug}-tasks.md`
- `docs/specflow/test/NNN-test-{slug}.md` -> `docs/specflow/features/NNN-{slug}/NNN-{slug}-test.md`
- `docs/specflow/test/assets/{NNN-...}` -> `docs/specflow/features/NNN-{slug}/assets/{...}`

Rules:

- A PRD without a matching task or test still migrates.
- A task or test without a matching PRD is not moved automatically; list it in the summary as orphaned.
- Abort on naming collisions before creating backups. A collision means two sources resolve to the same destination or a destination already exists with different content.
- Feature slug pattern is `NNN-{slug}` where `NNN` is three digits.

Verify:

```sh
find docs/specflow/prd -maxdepth 1 -type f -name '[0-9][0-9][0-9]-*.md' -print 2>/dev/null | sort
```

### 2. Backup every file to be modified or moved

For every source file in the manifest and every existing destination file that would be modified, create a sibling backup named `{path}.bak`.

- If `{path}.bak` already exists, keep it and verify it is non-empty.
- Never overwrite an existing `.bak`.
- Back up `config.json`, `pages.json`, every PRD, every matching task, every matching test, every matching asset file, and every admin file that will be modified.
- Backups are retained until the next successful upgrade or explicit `/specflow:upgrade --clean-backups`.

Verify:

```sh
test -f "docs/specflow/config.json.bak" || test -f "docs/specflow/admin/config.json.bak"
```

For each manifest source:

```sh
test -f "{source}.bak"
```

### 3. Create v2 folders

Create these folders if missing:

- `docs/specflow/admin/`
- `docs/specflow/admin/agents/standard/lifecycle/`
- `docs/specflow/admin/agents/standard/principles/`
- `docs/specflow/admin/agents/specialised/`
- `docs/specflow/admin/rules/`
- `docs/specflow/admin/debate-log/`
- `docs/specflow/features/`
- `docs/specflow/misc-task/`
- `docs/specflow/misc-task/assets/`
- `docs/specflow/docs/`

Verify:

```sh
test -d docs/specflow/admin/agents/standard/lifecycle
test -d docs/specflow/admin/agents/standard/principles
test -d docs/specflow/admin/agents/specialised
test -d docs/specflow/admin/rules
test -d docs/specflow/admin/debate-log
test -d docs/specflow/features
test -d docs/specflow/misc-task/assets
test -d docs/specflow/docs
```

### 4. Move config files into admin

Move only after backups exist:

- `docs/specflow/config.json` -> `docs/specflow/admin/config.json`
- `docs/specflow/pages.json` -> `docs/specflow/admin/pages.json`

If the destination already exists with identical content, keep the destination and leave the source in place until summary. If it differs, abort as a collision.

Verify:

```sh
test -f docs/specflow/admin/config.json
test ! -f docs/specflow/config.json || cmp -s docs/specflow/config.json docs/specflow/admin/config.json
test ! -f docs/specflow/pages.json || cmp -s docs/specflow/pages.json docs/specflow/admin/pages.json
```

### 5. Consolidate feature files

For each PRD `docs/specflow/prd/NNN-{slug}.md`:

1. Create `docs/specflow/features/NNN-{slug}/`.
2. Create `assets/`, `docs/`, `design/`, and `debate-log/` inside the feature folder.
3. Move PRD to `NNN-{slug}-prd.md`.
4. If `docs/specflow/task/NNN-tasks-{slug}.md` exists, move it to `NNN-{slug}-tasks.md`.
5. If `docs/specflow/test/NNN-test-{slug}.md` exists, move it to `NNN-{slug}-test.md`.
6. If matching test assets exist under `docs/specflow/test/assets/`, move them into `features/NNN-{slug}/assets/`.

Asset matching:

- Match asset names or directories beginning with `NNN-`.
- Preserve the remaining asset filename exactly.
- Abort if two assets would land at the same destination.

Verify for each PRD:

```sh
test -f "docs/specflow/features/NNN-{slug}/NNN-{slug}-prd.md"
test -d "docs/specflow/features/NNN-{slug}/assets"
test -d "docs/specflow/features/NNN-{slug}/docs"
test -d "docs/specflow/features/NNN-{slug}/design"
test -d "docs/specflow/features/NNN-{slug}/debate-log"
```

Optional file checks:

```sh
test ! -f "docs/specflow/task/NNN-tasks-{slug}.md"
test ! -f "docs/specflow/test/NNN-test-{slug}.md"
```

### 6. Copy standard agents

Copy templates into the project:

- `$PLUGIN_ROOT/templates/agents/standard/lifecycle/*` -> `docs/specflow/admin/agents/standard/lifecycle/`
- `$PLUGIN_ROOT/templates/agents/standard/principles/*` -> `docs/specflow/admin/agents/standard/principles/`

Do not overwrite edited project files. If a destination exists and differs from the template, keep it, list it as "manual merge required", and continue.

Verify:

```sh
test -f docs/specflow/admin/agents/standard/lifecycle/orchestrator.md
test -f docs/specflow/admin/agents/standard/lifecycle/devils-advocate.md
test -f docs/specflow/admin/agents/standard/lifecycle/verifier.md
test -f docs/specflow/admin/agents/standard/principles/simplicity-reviewer.md
test -f docs/specflow/admin/agents/standard/principles/surgical-reviewer.md
test -f docs/specflow/admin/agents/standard/principles/think-before-coding-reviewer.md
test -f docs/specflow/admin/agents/standard/principles/goal-driven-reviewer.md
```

### 7. Seed rules and pause for user review

Copy these templates when missing:

- `$PLUGIN_ROOT/templates/admin/rules/non-negotiable.md` -> `docs/specflow/admin/rules/non-negotiable.md`
- `$PLUGIN_ROOT/templates/admin/rules/guidelines.md` -> `docs/specflow/admin/rules/guidelines.md`
- `$PLUGIN_ROOT/templates/admin/rules/glossary.md` -> `docs/specflow/admin/rules/glossary.md`

After writing `non-negotiable.md`, pause and ask the user to accept or edit it. Continue only after the user confirms the project rules are acceptable.

Verify:

```sh
test -s docs/specflow/admin/rules/non-negotiable.md
test -s docs/specflow/admin/rules/guidelines.md
test -s docs/specflow/admin/rules/glossary.md
```

### 8. Seed admin support files

Create missing support files:

- Copy `$PLUGIN_ROOT/templates/admin/CONTEXT.md` -> `docs/specflow/admin/CONTEXT.md` if missing.
- Create `docs/specflow/admin/decision-log.md` with an empty heading if missing.
- Create `docs/specflow/admin/task-history.json` as `[]` if missing.
- Create `docs/specflow/admin/profiles.json` as `[]` if the profile interview is skipped.
- Create `docs/specflow/admin/agents/index.json` as an index of copied standard agents.
- Generate `docs/specflow/admin/environment.json` from current detection.

Environment inventory must include:

- `cli.playwright.present: true`
- `cli.codex.present: true|false`
- `plugins`
- `mcp`
- `agents`

Verify:

```sh
test -f docs/specflow/admin/CONTEXT.md
test -f docs/specflow/admin/decision-log.md
test -f docs/specflow/admin/task-history.json
test -f docs/specflow/admin/profiles.json
test -f docs/specflow/admin/agents/index.json
test -f docs/specflow/admin/environment.json
grep -q '"playwright"' docs/specflow/admin/environment.json
```

### 9. Generate retroactive interview stubs

For every migrated feature, create `features/NNN-{slug}/NNN-{slug}-interview.md` if missing.

Use this exact shape:

```md
# NNN-{slug} Interview

## Original request

(unknown - PRD pre-dates interview discipline)

## Codebase context

(captured during v1 to v2 migration)

## Rounds

(retroactive migration - no grilling transcript available; PRD body is the source of truth for this feature)

## Topics not discussed

- (retroactive - no record of intentional silences)

## Sign-off

Migration date: YYYY-MM-DD

(retroactive)
```

Use the migration date in `YYYY-MM-DD` format. Do not infer historical Q&A.

Verify:

```sh
test -f "docs/specflow/features/NNN-{slug}/NNN-{slug}-interview.md"
grep -q "retroactive migration" "docs/specflow/features/NNN-{slug}/NNN-{slug}-interview.md"
```

### 10. Render all relocated PRDs

Run `specflow:render --all` after PRDs are in feature folders. Rendering failures do not stamp the version; surface failures and pause for resolution.

Verify:

```sh
find docs/specflow/features -path '*/[0-9][0-9][0-9]-*-prd.md' -print | sort
find docs/specflow/features -path '*/[0-9][0-9][0-9]-*-prd.html' -print | sort
```

Each PRD must have a sibling HTML file before continuing.

### 11. Stamp version

Update `docs/specflow/admin/config.json.specflowVersion` to `2.0.0`.

- Preserve existing config fields.
- Keep JSON valid and deterministic.
- Do not stamp if any required verify clause above fails.

Verify:

```sh
grep -q '"specflowVersion"[[:space:]]*:[[:space:]]*"2.0.0"' docs/specflow/admin/config.json
```

### 12. Run doctor and pause on failures

Run `specflow:doctor`.

- If doctor reports failures, surface each failure and pause for resolution.
- Do not declare success until doctor passes.
- Warnings are allowed only for optional tooling, including absent Codex.

Verify:

```sh
specflow:doctor
```

### 13. Print summary

Print:

- Current version -> target version.
- Files moved.
- Files created.
- Files left for manual merge.
- Orphaned task/test files not moved.
- Backup count and cleanup instruction.
- Optional tooling warnings.
- Doctor result.

## Clean Backups

`/specflow:upgrade --clean-backups` only removes `.bak` files after:

1. `admin/config.json.specflowVersion` matches `plugin.json.version`.
2. `specflow:doctor` passes.
3. The user explicitly confirms cleanup.

Verify:

```sh
find docs/specflow -name '*.bak' -print
```

## Failure Handling

- Missing Playwright: hard abort before migration.
- Codex absent: warn and continue.
- PRD without matching task/test: migrate PRD only.
- Naming collision: abort before backups or moves; print source and destination paths.
- Existing edited destination: abort for relocated user content; for copied templates, keep destination and mark manual merge required.
- Partial failure: resume from the first step whose verify clause fails. Do not repeat successful moves except to verify them.
- Backups remain in place until a later successful upgrade or explicit cleanup.

## Final Verify Checklist

```sh
test -f docs/specflow/admin/config.json
test -f docs/specflow/admin/pages.json
test -d docs/specflow/features
test -d docs/specflow/admin/agents/standard/lifecycle
test -d docs/specflow/admin/agents/standard/principles
test -d docs/specflow/admin/agents/specialised
test -d docs/specflow/admin/rules
test -f docs/specflow/admin/environment.json
grep -q '"specflowVersion"[[:space:]]*:[[:space:]]*"2.0.0"' docs/specflow/admin/config.json
find docs/specflow/features -path '*/[0-9][0-9][0-9]-*-prd.md' -print | sort
find docs/specflow/features -path '*/[0-9][0-9][0-9]-*-prd.html' -print | sort
find docs/specflow -name '*.bak' -print
```
