# Migrations

Source of truth for what every specflow version added, changed, moved, or removed. Consumed by `specflow:upgrade` to compute the chain of migrations between a project's installed version and the plugin's current version.

Every PR that ships a breaking change MUST add a migration entry here.

Format per entry:
- **Version range** the migration covers.
- **Scope** of changes.
- **Steps** the upgrade skill will execute (with reversibility notes).
- **Backups** taken before applying.
- **Verify** how to confirm the migration succeeded.

The upgrade skill is **purely additive** — never deletes user data without explicit confirmation.

---

## v2.11.0 → v2.12.0

Reshapes the task block from an 8-field bullet list to a 13-section structured-narrative format. Adds a Linear export prompt at the end of `specflow:task` Phase E. Dual-format readers in `develop` / `sprint` / `linear` keep existing task files working — no force-migration of in-flight features.

### Scope

- `specflow:task` Phase B.3 — new task block template (Parent PRD, Dependencies, Current State, Expected State, Technical Implementation, Technical References, Files to Modify, Files to Create, Layers Touched, Acceptance Criteria, QA Verification, Definition of Done, User Stories Addressed, Stats line, ai-metadata HTML comment).
- `specflow:task` Phase B.4 — new self-checks (step 5: Conversational State check; step 6: Technical References completeness).
- `specflow:task` Phase E.9 — Linear export prompt after Gate 3 close.
- `specflow:task` Phase E.10 — feature.md status bump to `development`.
- `specflow:develop` Phase A.6 — format detection per task block; reads `context-budget-estimate` from HTML comment in new format, from visible field in legacy.
- `specflow:sprint` Phase B — format detection per task block; reads `sprint-bucket` from HTML comment in new format, from visible field in legacy. Dependencies parsing handles both `**Dependencies**` section (new) and `**Depends on:**` field (legacy).
- `specflow:linear` — format detection per task block; Linear issue body composed from new section set or legacy field set as appropriate.

### Steps

1. Pull v2.12.0.
2. No `admin/config.json` updates required.
3. Existing task files (v2.11.x and earlier) continue to work — dual-format readers detect via `**Parent PRD:**` presence.
4. Newly-synthesised tasks land in the new format. To migrate an existing task file to the new format manually, re-invoke `specflow:task --regenerate {NNN-slug}` (note: `--regenerate` flag is NOT a v2.12.0 feature; manual migration requires hand-editing or a future migration tool).

### Backups

None required — no user data is moved or rewritten by the upgrade itself.

### Verify

- `grep -n 'Parent PRD' plugins/specflow/skills/task/SKILL.md` returns a match in the Phase B.3 template.
- `grep -n 'Technical References' plugins/specflow/skills/task/SKILL.md` returns a match.
- `grep -n 'Linear export prompt' plugins/specflow/skills/task/SKILL.md` returns a match for Phase E.9.
- `grep -n 'Task block format detection' plugins/specflow/skills/develop/SKILL.md` returns a match.
- `grep -n 'Sprint-bucket parsing' plugins/specflow/skills/sprint/SKILL.md` returns a match.
- `grep -n 'Task block format' plugins/specflow/skills/linear/SKILL.md` returns a match.
- `plugin.json` and `marketplace.json` both report `2.12.0`.

---

## v2.10.1 → v2.11.0

Adds light-mode coverage across `feature` / `prd` / `task` / `grill`. Additive; backward-compatible with pre-2.11.0 features (mode defaults to `full` when absent).

### Scope

- New frontmatter field on `{NNN-slug}-feature.md`: `mode: light | full`. Auto-detected by `specflow:feature` Phase C.1; user confirms at reflection.
- `specflow:feature` Phase C.1 adds complexity detection from goal answers + assets/design content.
- `specflow:prd` Phase A.0 extended to read `mode:` and skip Phase B.5 (Codex) + Phase D (Gate 2) when `light`; Phase B (grilling) caps at 0-2 rounds.
- `specflow:task` gains new Phase A.0 step (mode read); skips Phase B.5 (Codex) + Phase D (per-task reviewers) + Phase E.4.5 (cross-task review) + Phase E (Gate 3) when `light`.
- `grill` sub-skill pre-flight adds step 6 (mode read); applies 0-2 round cap when `light`.

### Steps

1. Pull v2.11.0.
2. No `admin/config.json` updates required.
3. Existing feature.md files without `mode:` continue to work — every reading skill defaults to `full` when the field is missing.
4. New features kicked off via `specflow:feature` after upgrade will carry the `mode:` field automatically.

### Backups

None required — no user data is moved or rewritten.

### Verify

- `grep -n 'Mode read (per 032-lightweight-mode' plugins/specflow/skills/prd/SKILL.md` returns a match.
- `grep -n 'Mode read (per 032-lightweight-mode' plugins/specflow/skills/task/SKILL.md` returns a match.
- `grep -n 'mode: {light' plugins/specflow/skills/feature/SKILL.md` returns a match (frontmatter template).
- `plugin.json` and `marketplace.json` both report `2.11.0`.

---

## v2.10.0 → v2.10.1

Wires `assets/` folder readback into `specflow:prd` Phase A.3. Additive; no schema changes; no migration script required.

### Scope

- `specflow:prd` Phase A.3 gains a parallel readback block for `features/{NNN-slug}/assets/` mirroring the existing design-folder readback. Globs `*.yaml | *.yml | *.json | *.html | *.md | *.txt | *.csv`; caps per-file at 200 lines; distills into `Reference assets (from assets/):` bullets.

### Steps

1. Pull v2.10.1.
2. No `admin/config.json` updates required.
3. Existing features without `assets/` content continue to work — the new readback skips silently when `assets/` is absent or only contains `.gitkeep`.

### Backups

None required — no user data is moved or rewritten.

### Verify

- `grep -n 'Assets folder readback' plugins/specflow/skills/prd/SKILL.md` returns a match.
- `plugin.json` and `marketplace.json` both report `2.10.1`.

---

## v2.9.1 → v2.10.0

Adds `specflow:feature` as a new pipeline step (kickoff, before PRD). Additive; existing features unaffected; new features get the meta file scaffold.

### Scope

- New skill: `plugins/specflow/skills/feature/SKILL.md`.
- New per-feature artefact: `features/{NNN-slug}/{NNN-slug}-feature.md`.
- `specflow:prd` Phase A gains an A.0 step that reads feature.md when present and skips the slug allocation + goal articulate/confirm/write cycle.
- `specflow:setup` config.json seed gains `skills.feature.enabled: true` (and `skills.learn.enabled: true` — back-fills a v2.8.0 oversight).
- README pipeline doc reordered: `feature` first, `test` last (per pipeline correction).

### Steps

1. Pull v2.10.0.
2. `specflow:upgrade` patches `admin/config.json`: adds `skills.feature.enabled: true` and `skills.learn.enabled: true` if absent. Never overwrites user values.
3. Existing features without `{NNN-slug}-feature.md` continue to work — `specflow:prd` falls back to the pre-v2.10 A.1-A.7 flow when feature.md is absent.
4. New features should be kicked off via `/specflow:feature {overview}` to get the goal-locked + scaffolded structure.

### Backups

- `admin/config.json.bak` written before the toggle back-fill.

### Verify

- `test -f plugins/specflow/skills/feature/SKILL.md` returns 0.
- `grep -q '"feature":' docs/specflow/admin/config.json` returns 0.
- `grep -q '"learn":' docs/specflow/admin/config.json` returns 0.
- `grep -n 'A.0 Feature-skill handoff' plugins/specflow/skills/prd/SKILL.md` returns a match.

---

## v2.9.0 → v2.9.1

Simplifies the v2.9.0 duration knob per user feedback. Unit changes from minutes to hours; option set shrinks to `1 | 4 | 8 | auto`; the per-task field and B.4 self-check step are removed.

### Scope

- Config key renamed: `task.maxDurationMinutes` → `task.maxDurationHours`. Default `60` (mins) → `1` (hour). Options `30 / 60 / 90 / 120` → `1 / 4 / 8`.
- Task-block field `estimated-duration-minutes` removed from B.3 template. Existing task blocks that carry the legacy field are left untouched; new tasks created post-upgrade omit it.
- `specflow:task` Phase B.4 step 8 (Duration self-check) removed. Sizing is enforced by the existing step 5 (token-budget self-check) alone; duration is a synthesis-time target.

### Steps

1. Pull v2.9.1.
2. `specflow:upgrade` rewrites `admin/config.json`: deletes `task.maxDurationMinutes` if present; writes `task.maxDurationHours: 1` if absent. User-edited values are not preserved across the rename — the conversion is intentionally lossy because the option sets don't overlap.
3. Legacy task blocks with `estimated-duration-minutes` are left in place; the field is now a no-op.

### Backups

- `admin/config.json.bak` written before the key rename.

### Verify

- `grep -q '"maxDurationHours"' docs/specflow/admin/config.json` returns 0.
- `grep -q '"maxDurationMinutes"' docs/specflow/admin/config.json` returns 1 (key removed).
- `grep -n 'estimated-duration-minutes' plugins/specflow/skills/task/SKILL.md` returns no matches.
- `grep -n 'Duration self-check' plugins/specflow/skills/task/SKILL.md` returns no matches.

---

## v2.8.0 → v2.9.0

Adds a new user-facing config knob for per-task duration. Additive; existing projects get a sensible default at upgrade time.

### Scope

**New config key** (`admin/config.json`):

- `task.maxDurationMinutes` — integer minutes (`30 | 60 | 90 | 120`) or the string `"auto"`. Default `60`. Captured at setup time via a new 8.2 prompt; back-filled silently on upgrade. Read by `specflow:task` Phase B.4 step 8 to flag oversize tasks for `specflow:scope-change` splitting, parallel to the existing token-budget check at step 5. When `"auto"`, the per-task `estimated-duration-minutes` field becomes informational only — the token-budget rule remains the sizing authority.

**New task-block field** (`{NNN-slug}-tasks.md`):

- `estimated-duration-minutes` — positive integer; the AI's synthesis-time estimate of human-time to implement. Surfaces between `context-budget-estimate` and `sprint-bucket`. Required for every task block.

**Skill body changes:**

- `specflow:setup` Phase 8.2 — new prompt between brief commit policy and config.json write. Maps user choice 1-5 to `30 / 60 / 90 / 120 / "auto"`. Default `60` on skip.
- `specflow:task` Phase B.3 — task-block template adds `estimated-duration-minutes` field.
- `specflow:task` Phase B.4 — adds step 8 (Duration self-check), parallel to step 5 (budget self-check). Skipped when the config value is `"auto"`.

### Steps

1. Pull v2.9.0.
2. `specflow:upgrade` back-fills `task.maxDurationMinutes: 60` into existing `admin/config.json` files (creating the key only when absent; never overwrites a user value).
3. Existing tasks files lack `estimated-duration-minutes` on prior task blocks. `specflow:task` consumers fall back to "estimate not present, skip the duration check for this task" semantics; new tasks created post-upgrade will include the field. No retroactive rewrite of historical tasks files.
4. No data loss; no schema breakage.

### Backups

- `admin/config.json.bak` written before the back-fill (mirrors the v2.7.0 backup discipline).

### Verify

- `grep -q '"maxDurationMinutes"' docs/specflow/admin/config.json` returns 0.
- `grep -n 'estimated-duration-minutes' plugins/specflow/skills/task/SKILL.md` shows the new field in B.3.
- `grep -n 'Duration self-check' plugins/specflow/skills/task/SKILL.md` shows step 8 in B.4.

---

## v2.7.1 → v2.8.0

Adds a new Phase 3 skill (`specflow:learn`). Additive; no schema changes; no migration script required for existing projects.

### Scope

**New skill** (`plugins/specflow/skills/learn/`):

- `specflow:learn` — repo-local self-learning consumer. Reads `docs/specflow/admin/plugin-findings.jsonl` (lazy-created on first append by any producer); clusters deterministically by `signal_pattern` at a 3-observation threshold; auto-applies Tier-A additive rules under a per-run cap of 3. Tier-B (plugin-level) and Tier-C (conflict) clusters log to `admin/scratch/` for manual review.

### Steps

1. Pull v2.8.0.
2. No `admin/config.json` updates required — `specflow:learn` reads existing registry files (`rules/guidelines.md`, `CONTEXT.md`, `config.json`) and writes to them additively. The corpus file is lazy-created.
3. No existing skill bodies modified; `specflow:test` producer integration ships in a follow-up release once real-world test-run signal grounds the sidecar schema.

### Backups

None required — no user data is moved or rewritten.

### Verify

- `test -f plugins/specflow/skills/learn/SKILL.md` returns 0.
- `plugin.json` and `marketplace.json` both report `2.8.0`.

---

## v2.7.0 → v2.7.1

Hardening release. No new features, no schema changes, no new files. Pure prompt-asset edits to existing skill bodies and templates.

### Scope

**Branding compliance:** vendor-name guardrails neutralised across 14 skill bodies + 3 templates. All `Claude, Anthropic, or any AI tooling` phrases replaced with `the underlying AI tooling or vendor`. Two incidental leaks in lifecycle/orchestrator agent docs aligned.

**Sprint Phase D.3 — idempotent worktree creation:**
- Six-state resolution (reuse / mismatch-HALT / elsewhere-HALT / leftover-HALT / attach / fresh-create) replaces single unconditional `git worktree add`.
- Dirty-state probe with explicit `DIRTY_PROBE_STATUS` (skipped/ok/failed); never silently treats failed probes as clean.
- Portable `run_with_timeout` helper (GNU `timeout` → `gtimeout` → POSIX `/bin/sh`-hosted watchdog with TERM→KILL escalation).
- Path parsing handles whitespace, regex metacharacters, and literal backslash sequences (uses awk `substr` + `ENVIRON`, not `-v`).

**Develop — `T_run` scope binding:**
- New section A.6.5 binds `T_run` (sprint-mode = tasks_in_scope; feature-mode = full tasks file; per-task = [T{N}]) and persists to `admin/scratch/{NNN-slug}-develop/t-run.json`.
- Resume logic loads `t-run.json` BEFORE artefact checks; halts on missing `t-run.json` rather than auto-widening to feature-mode.
- Per-task loops in Phase B.1, B.1.5, D, F + final verify checklist all scope to `T_run` with symmetric out-of-scope absence assertions.

### Steps

1. Pull v2.7.1 (no migration script required — pure prompt-asset changes).
2. No `admin/config.json` updates needed.
3. No `docs/specflow/` schema changes.
4. Existing in-flight feature folders are unaffected.
5. The next `specflow:develop` invocation will write `t-run.json` to scratch on first sprint or feature-mode run.

### Backups

None required — no user data is moved or rewritten.

### Verify

- `plugin.json` and `marketplace.json` both report `2.7.1`.
- `grep -r 'Claude, Anthropic' plugins/specflow/skills plugins/specflow/templates` returns zero matches.
- `awk '/^### D\.3 /,/^### D\.4 /' plugins/specflow/skills/sprint/SKILL.md` documents probes 1–5 and states 1–6 with HALT predicates.
- `grep -c T_run plugins/specflow/skills/develop/SKILL.md` ≥ 17.

---

## v1.x → v2.0

The foundation migration. Establishes the substrate every later phase depends on: feature-grouped layout, `admin/` folder, rules registry, environment inventory, standard agents, and per-feature brief composition.

### Scope

**File relocations:**
- `docs/specflow/config.json` → `docs/specflow/admin/config.json`
- `docs/specflow/pages.json` → `docs/specflow/admin/pages.json`
- `docs/specflow/prd/NNN-{slug}.md` → `docs/specflow/features/NNN-{slug}/NNN-{slug}-prd.md`
- `docs/specflow/task/NNN-tasks-{slug}.md` → `docs/specflow/features/NNN-{slug}/NNN-{slug}-tasks.md`
- `docs/specflow/test/NNN-test-{slug}.md` → `docs/specflow/features/NNN-{slug}/NNN-{slug}-test.md`
- `docs/specflow/test/assets/{NNN-...}` → `docs/specflow/features/NNN-{slug}/assets/{...}` (rehomed by feature ID)

**Naming convention preserved:** Top-level feature files keep the `NNN-{slug}-` prefix on every filename (`NNN-{slug}-prd.md`, `NNN-{slug}-tasks.md`, etc.) so multiple PRDs are distinguishable when files are open in editor tabs or surface in search. Folder-level uniqueness alone isn't enough.

**New folders created:**
- `docs/specflow/admin/`
- `docs/specflow/admin/agents/standard/lifecycle/` (Orchestrator, Devil's Advocate, Verifier copied from templates)
- `docs/specflow/admin/agents/standard/principles/` (Simplicity, Surgical, Think-Before-Coding, Goal-Driven reviewers copied from templates)
- `docs/specflow/admin/agents/specialised/`
- `docs/specflow/admin/debate-log/` (the multi-agent debate manifest archive)
- `docs/specflow/admin/rules/`
- `docs/specflow/features/` (parent — per-feature subdirectories already exist after the relocation above)
- `docs/specflow/misc-task/` (with `assets/`)
- `docs/specflow/docs/`

**New files seeded:**
- `admin/environment.json` — generated by re-running detection
- `admin/profiles.json` — generated by running the profile interview
- `admin/CONTEXT.md` — copied from the template
- `admin/rules/non-negotiable.md` — seeded from the starter set; user accepts/edits
- `admin/rules/guidelines.md` — empty, ready for project-taste additions
- `admin/rules/glossary.md` — populated from the starter set
- `admin/decision-log.md` — empty template, populated in Phase 3
- `admin/task-history.json` — empty template, populated in Phase 3
- `admin/agents/index.json` — built from the agent index scan

**Per-feature derived artefact:**
- `features/NNN-{slug}/NNN-{slug}-brief.html` — composed for every relocated PRD via `specflow:brief --all`.

**Per-feature retroactive interview file:**
- For every relocated PRD, the v1→v2 migration creates a stub `features/NNN-{slug}/NNN-{slug}-interview.md` containing:
  - Original request — populated as "(unknown — PRD pre-dates interview discipline)" if no source available.
  - Codebase context — populated from a fresh codebase scan at migration time, not retroactively.
  - Rounds — empty section with a note "(retroactive migration — no grilling transcript available; PRD body is the source of truth for this feature)".
  - Topics not discussed — empty list with a note "(retroactive — no record of intentional silences)".
  - Sign-off — dated as the migration date with note "(retroactive)".
- The stub is honest about what's a real audit trail vs. a backfill. Future PRDs (post-v2) get full interview files from `specflow:prd`.

**Debate-log relocation:**
- v1 had no debate-log directory (no adversarial review chain). v2 introduces it under `features/NNN-{slug}/debate-log/` (per-feature) plus `admin/debate-log/` (cross-feature gates only). The migration creates the empty directory under each existing feature; gates only populate as `specflow:prd` / `specflow:task` / `specflow:develop` run going forward.

**Config updates:**
- `admin/config.json.specflowVersion` set to `2.0.0`.
- `admin/config.json.confidentialPaths` seeded if the project is detected to have auth/secrets/billing surfaces (path globs; user confirms).

### Steps

1. **Backup.** Copy every file to be moved or modified to a sibling `.bak` (e.g. `prd/001-foo.md.bak`). Backups retained until the user confirms the migration succeeded.
2. **Move config.** Relocate `config.json` and `pages.json` into `admin/`.
3. **Consolidate features.** For each existing PRD `NNN-{slug}.md`, create `features/NNN-{slug}/`. Move the PRD, the matching task file, the matching test file, and rehome the test's assets into the new feature folder. Rename to `prd.md` / `tasks.md` / `test.md`.
4. **Seed admin folder.** Create `admin/` subfolders. Copy standard agent templates. Seed rules registry. Run profile interview. Run environment inventory.
5. **Compose briefs.** Invoke `specflow:brief --all` so every relocated PRD gets a sibling `{NNN-slug}-brief.html` (composed from PRD + retroactive interview stub).
6. **Stamp version.** Write `2.0.0` to `admin/config.json.specflowVersion`.
7. **Verify.** Run `specflow:doctor`. Surface any failures and pause for user resolution before declaring success.

### Reversibility

Backups (`.bak`) are written for every modified file before the migration starts. If the user rejects the result, the upgrade skill can restore from `.bak` files. Backups are retained until the next successful upgrade or explicit cleanup via `/specflow:upgrade --clean-backups`.

### Verify

- `docs/specflow/admin/config.json` exists; `specflowVersion === "2.0.0"`.
- `docs/specflow/features/` contains one subdirectory per existing PRD.
- Every feature folder contains `prd.md`, `tasks.md` (if the feature had tasks), `test.md` (if the feature had tests), and `brief.html`.
- `docs/specflow/admin/agents/standard/` contains Orchestrator, Devil's Advocate, Verifier.
- `docs/specflow/admin/rules/non-negotiable.md` contains the starter set.
- `docs/specflow/admin/environment.json` is populated and Playwright is detected.
- `specflow:doctor` reports no failures.

### Failure modes

- **Conflicting feature slug.** If two features share an `NNN-{slug}` shape (shouldn't happen — IDs are unique), abort and ask the user to resolve.
- **Missing Playwright.** Hard requirement at v2 — abort migration, prompt user to install before retrying.
- **Brief composition failure.** Migration continues; failed compositions logged. User can re-run `specflow:brief --all` after fixing.
- **Codex absent.** Soft requirement — migration completes; adversarial-review degraded mode flagged in the summary.

---

## v2.0 → v2.1

The Phase 2 development-layer migration. Adds the `develop` config block, surfaces specialised-agent drift via `specflow:agent refresh`, and introduces the per-feature `develop-gate4/` and `develop-gate5/` debate-log subdirectories.

### Scope

**New config keys:**
- `admin/config.json.develop.greenBatchCap` — default `3`. Maximum number of green-lane tasks the orchestrator batches into a single AFK-eligible run before requiring a human sign-off. User-tunable per Phase 2 PRD R6 (interview Round 2 — user lowered the default from 5 to 3 to keep PR-review fatigue in check; projects with strong CI signal can raise it). The migration adds the key with the default; user re-confirms or edits at next `specflow:develop` invocation.
- `admin/config.json.develop.codexAtGate5` — boolean default derived from `environment.json.codex.detected`. If Codex is detected, defaults `true`; if absent, `false`. User can override.

**New folders created (per existing feature, on demand):**
- `features/NNN-{slug}/debate-log/develop-gate4/` — created by `specflow:develop` Phase C.
- `features/NNN-{slug}/debate-log/develop-gate5/` — created by `specflow:develop` Phase E.
- `admin/scratch/{NNN-slug}-develop/` — orchestrator scratch path for lane assignments and gate manifests during execution.

**Index schema additions (per agent registry):**
- `admin/agents/index.json` gains an optional `stack_match_reason` field on specialised agent entries; existing entries without it remain valid (default empty string). Migration is non-destructive — does NOT add the field to existing entries; new `specflow:agent add` invocations populate it going forward.

**Convention additions (no schema change but enforced by skills):**
- Plans produced by `specflow:develop` MUST start with the PRD-anchor format `"We're doing X because of PRD requirement Y. This aligns with goal field Z."` (Phase 2 PRD R17). Enforced by Gate 4 reviewers (Goal-Driven Reviewer's reverse-traceability lens — added by E4 prompt edit).

### Steps

1. **Backup.** Copy `admin/config.json` to `admin/config.json.bak` before modifying.
2. **Add config keys.** Insert the `develop` block under the top level of `config.json`. Set `greenBatchCap: 3` and derive `codexAtGate5` from `environment.json.codex.detected`. Pause and ask the user to confirm both values before writing — the green-batch cap is project-taste, not auto-imposed.
3. **Stamp version.** Update `admin/config.json.specflowVersion` to `2.1.0`.
4. **Refresh agents.** Invoke `specflow:agent refresh` to surface any specialised-agent drift introduced by Phase 2 (no specialised agent additions ship with the plugin in v2.1, but the refresh is a no-op-when-clean discipline). Surface drift report path to the user.
5. **Verify.** Run `specflow:doctor`. Confirm `config.json.develop.greenBatchCap` is set, `codexAtGate5` matches the environment, and the index validates.

### Reversibility

- `config.json.bak` retained until the next successful upgrade or explicit cleanup. User can manually restore.
- Specialised-agent snapshots are NOT touched by this migration — refresh is additive only (drift surfaced, not auto-applied).

### Verify

- `admin/config.json.develop.greenBatchCap` exists, integer.
- `admin/config.json.develop.codexAtGate5` exists, boolean.
- `admin/config.json.specflowVersion === "2.1.0"`.
- `specflow:agent refresh` ran successfully and any drift report is at `admin/scratch/agent-refresh-{timestamp}.md`.
- `specflow:doctor` passes.

### Failure modes

- **User declines greenBatchCap default.** Pause the migration; let user enter a value. Refuse default below 1.
- **environment.json missing codex section.** Default `codexAtGate5: false`; warn user that Gate 5 will run without Codex; no migration failure.
- **`specflow:agent refresh` reports orphaned specialised agents.** Surface to user; offer `specflow:agent remove {name} --keep-snapshot` if the open question from agent skill is implemented, else `specflow:agent remove {name}`. Migration succeeds either way.

---

## v2.1 → v2.2

The brief release. Replaces the per-PRD HTML render (`specflow:render` → `{NNN-slug}-prd.html`) with a richer feature brief (`specflow:brief` → `{NNN-slug}-brief.html`) that composes the PRD body, interview transcript, and Gate 2 / Gate 3 manifests into a single self-contained HTML document with a Visual abstract section at the top. Lands alongside the Phase 3 skill additions (`specflow:decision`, `specflow:scope-change`) — see the CHANGELOG 2.2.0 entry for the full release scope.

### Scope

**Skill changes:**
- `specflow:render` removed. Its responsibility is fully absorbed by `specflow:brief`.
- `specflow:brief` added. Composes `{NNN-slug}-brief.html` from `{NNN-slug}-prd.md` + `{NNN-slug}-interview.md` + (optional) gate manifests. Supports a structured-block vocabulary (`:::flow`, `:::comparison`, `:::scope`, `:::tree`) for visualising flows, mode comparisons, scope, and decision trees deterministically.
- `specflow:prd` Phase D and Phase E swapped: Gate 2 is now Phase D; Brief is now Phase E. Phase E asks the user whether to open the brief in their browser, then opens it on confirmation.
- `specflow:doctor` `features.{NNN-slug}.html_drift` check renamed to `features.{NNN-slug}.brief_drift`. The check now compares the brief mtime against the latest of PRD / interview / gate-manifest mtimes, not just the PRD.
- `specflow:upgrade` step 10 (`specflow:render --all`) replaced with `specflow:brief --all`. The migration also deletes `{NNN-slug}-prd.html` files after a successful brief is written for the same feature.

**Per-feature artefact:**
- New: `features/NNN-{slug}/NNN-{slug}-brief.html`. Composed for every feature with both a PRD and an interview file present.
- Removed: `features/NNN-{slug}/NNN-{slug}-prd.html`. Deleted once the sibling brief is written. The brief supersedes it.

**No schema changes.** No config keys added or removed. No agent-set changes. The migration is purely the artefact rename + composition source widening.

### Steps

1. **Backup.** Copy every existing `{NNN-slug}-prd.html` to `{NNN-slug}-prd.html.bak` before deletion.
2. **Compose briefs.** Run `specflow:brief --all` across every feature folder containing both `{NNN-slug}-prd.md` and `{NNN-slug}-interview.md`. Brief composition failures abort the migration; surface failures and pause for resolution.
3. **Remove old prd.html.** For each feature where the brief was successfully written, delete `{NNN-slug}-prd.html`. Leave the `.bak` in place per backup discipline.
4. **Stamp version.** Update `admin/config.json.specflowVersion` to `2.2.0`.
5. **Verify.** Run `specflow:doctor`. Every feature should report `brief_drift` PASS (or be absent if no brief is needed).

### Reversibility

- `.bak` files retained until the next successful upgrade or explicit `/specflow:upgrade --clean-backups`. To roll back, restore each `{NNN-slug}-prd.html.bak` to `{NNN-slug}-prd.html`, delete the new `{NNN-slug}-brief.html`, and downgrade the plugin version.
- No source markdown is modified by this migration. The PRD body, interview, and manifests are untouched.

### Verify

- For every feature with a PRD + interview: `features/NNN-{slug}/NNN-{slug}-brief.html` exists and opens in a browser.
- No `features/NNN-{slug}/NNN-{slug}-prd.html` files remain.
- `admin/config.json.specflowVersion === "2.2.0"`.
- `specflow:doctor` passes; `brief_drift` is PASS for every feature.

### Failure modes

- **Brief composition fails on a feature.** The migration aborts before deleting any `prd.html`. Resolve the failure (typically: an unclosed `:::` visual block, or an unsupported block kind), then re-run upgrade — the migration resumes from the failing feature.
- **Pre-2.2.0 PRD has no interview file.** This indicates a partial install. The migration can't compose a brief without the interview. Surface to the user; offer to either run `specflow:upgrade` (which seeds a retroactive interview stub) or skip the feature with a warning. Default: skip with warning, continue with other features.

---

## v2.2 → v2.3

The Phase 3 memory-and-discipline release. Adds the `insights`, `prune`, and `optimize` config blocks; introduces three append-only data surfaces (`admin/insights/`, `admin/archive/`, `admin/optimize-runs.jsonl`); seeds default values for the per-skill thresholds with user confirmation at first invocation. Migration is purely additive — no file relocations, no schema rewrites, no destructive operations.

### Scope

**New config keys:**
- `admin/config.json.insights.minCorpusSize` — integer, default `10`. Refusal threshold for `/insights` clustering — corpus below this size produces no proposals (registry-stable signal, not failure). Per `006-insights-skill` PRD R10.
- `admin/config.json.insights.cadence` — string, default `"monthly"`, accepts `"weekly" | "monthly" | "quarterly" | "manual-only"`. Informational knob: the skill emits a chat-line on every successful run naming the next-suggested-run date computed from `last_successful_run_at + cadence_interval`; users wire their own scheduler. Per `006-insights-skill` PRD R6.
- `admin/config.json.prune.thresholds.decisionLog.ageDays` — integer, default `365` (1 year / 4Q). Decision-log entries older than this AND with no reference in the dormancy window are prune candidates. Per `007-prune-skill` PRD R2.
- `admin/config.json.prune.thresholds.decisionLog.dormancyDays` — integer, default `182` (~2Q). Reference-window for the decision-log staleness check.
- `admin/config.json.prune.thresholds.guidelines.dormancyDays` — integer, default `365` (4Q). Guidelines with zero references in this window become prune candidates (or guidelines whose cited rules are superseded — whichever fires first). Per `007-prune-skill` PRD R3.
- `admin/config.json.prune.thresholds.taskHistory.ageDays` — integer, default `365`. Task-history entries older than this AND with `superseded_by_retro: true` AND no addenda within the dormancy window are prune candidates. Per `007-prune-skill` PRD R5.
- `admin/config.json.prune.thresholds.taskHistory.dormancyDays` — integer, default `182` (~2Q). Addenda window for the task-history staleness check.
- `admin/config.json.optimize.targetCapUsd` — number (USD), default `10`. Per-target weekly spend cap for `/optimize` runs. Aggregate envelope across all targets is implicit at $60/week (six initial targets × $10); no aggregate cap is enforced. Override via `--override-budget {reason}` extends only that target's cap for the current run; the override reason is recorded to `admin/optimize-runs.jsonl`. Per `008-optimize-skill` PRD R7.
- `admin/config.json.optimize.judgementWords` — array of strings, default `["appropriately", "adequately", "cleanly", "concrete signals", "coverage", "idiomatic", "well", "properly", "correctly"]`. Eligibility pre-flight rejects targets whose `eval:` field contains any of these words (the eval must be machine-checkable, not opinion). The default list is hard-coded; project-level entries extend it (do not replace). Per `008-optimize-skill` PRD R1.

The decline-streak windows (7-day operator-avoid, 30-day target-skip) and unique-id promotion threshold (3 contributing ids per cluster) are intentionally hard-coded in v1 — not configurable surfaces. Per the PRDs, those values are the discipline-installer contract; surfacing them as knobs would invite Goodharting on the values themselves.

**New folders + files (created on demand, not at migration time):**
- `admin/insights/` — created on first `/insights` run. Contains `{YYYY-MM}-report.md` (replaced-in-place on within-month re-runs — represents current state of the month's pattern detection) and `{YYYY-MM}-runs.jsonl` (append-only execution log; one JSON object per line per run). Cross-month rollover writes fresh files; prior-month files are never touched.
- `admin/scratch/insights-{YYYY-MM}.lock` — created during an `/insights` run; deleted on every exit path. Per-month concurrency lock between manual and cron paths.
- `admin/archive/` — created on first `/prune` run. Contains `{YYYY-Q}-prune.md` files (one per quarterly run, append-only). The `prune` skill never modifies its own archive.
- `admin/scratch/prune-{YYYY-Q}.lock` — created during a prune run; deleted on completion. Concurrency lock.
- `admin/scratch/prune-history.json` — append-only run record + drift-continuity tracking across consecutive prune runs.
- `admin/scratch/optimize-{target}-{run-ts}/` — per-run scratch directory with variants, eval logs, scoring outputs.
- `admin/scratch/optimize.lock` — created during an optimize run.
- `admin/optimize-runs.jsonl` — append-only run history with `merge_decision` field updated through human review.

**Lessons registry (the project self-learning corpus):**
- `admin/lessons.json` — new file, seeded as `[]`. Mutable JSON array of lesson entries; schema and lifecycle defined in `skills/test/SKILL.md` § Lessons registry. Status values: `active | superseded | resolved | promoted-to-rule`. Kinds: `escape | success | rule-candidate`. Tags drawn from a controlled vocabulary derived from `environment.json` (stack), `profiles.json` + `glossary.md` (domain), and a canonical surface set (`ui`, `data-model`, `api`, `auth`, `migration`, `infra`, `cli`, `docs`).
- `specflow:test` gains `--feedback` mode (Phase D) for capturing gaps that escaped the gates, and Phase B.0 lesson query before plan synthesis. Feedback writes one lesson + one covering test case + one `task-history.json` attribution row, atomically with `lessons.json.bak`.
- `specflow:task` gains A.4 lesson query before task derivation; matched lessons become `lesson-anchor: L-NNN` fields on derived tasks.
- `specflow:complete` gains H.2.1 soft chat-line reminder pointing the user at `specflow:test {slug} --feedback` after every successful retro write.
- `specflow:setup` seeds `admin/lessons.json` as `[]` during first-run scaffold; verify step 9.2 includes the new file.
- Promotion path: a lesson reaching 3 occurrences across distinct features auto-prompts for promotion to `admin/rules/guidelines.md`. Promoted rules are then read by `specflow:prd` and `specflow:task` on every future feature, so the lesson graduates from "heads-up" to "non-negotiable on relevant features." This path is the data input that `/insights` later consumes for cross-project pattern detection (the lessons corpus is the substrate; `/insights` does the higher-order clustering).

**No schema changes to existing files.** Migration does not modify `task-history.json`, `decision-log.md`, `rules/*`, `agents/index.json`, or any other pre-existing surface. The existing schemas are forward-compatible with the new consumer skills. The new `admin/lessons.json` is a new file alongside, not a modification of an existing one.

**Convention additions (no schema change but enforced by skills):**
- `/insights` mints promotion proposals by appending to the rules registry through `specflow:decision`'s mirror schema (audit-trail entry per accepted promotion).
- `/prune` archive entries follow the decision-log entry shape (Title / Context / Decision / Rationale / Date / Related) so a restoration round-trip is byte-identical.
- `/optimize` PR descriptions MUST include the literal "Human merge owns taste" footer linking to `CORE_PRINCIPLES.md`. Auto-merge is structurally impossible (no `--auto` call in CI; HTML comment guard; GH Action human-actor check).

### Steps

1. **Backup.** Copy `admin/config.json` to `admin/config.json.bak` before modifying.
2. **Add config keys.** Insert the `insights`, `prune`, and `optimize` blocks under the top level of `config.json` with the defaults listed above. Pause and ask the user to confirm or edit the per-target budget cap and the prune thresholds before writing — those are project-taste, not auto-imposed. The `insights.cadence` and `optimize.judgementWords` defaults can be accepted or edited inline; the hardcoded thresholds (3-observation promotion, 7/30-day decline windows) are not surfaced as knobs.
3. **Seed `lessons.json`.** Write `[]` to `docs/specflow/admin/lessons.json` if missing. If a project somehow has a pre-existing `lessons.json`, do NOT overwrite — surface a one-line note in the migration summary and continue.
4. **Stamp version.** Update `admin/config.json.specflowVersion` to `2.3.0`.
5. **Verify.** Run `specflow:doctor`. Confirm the new config keys exist with the expected types; confirm `lessons.json` exists and parses as an array; confirm `specflowVersion` is stamped; confirm the existing surfaces (`task-history.json`, `decision-log.md`, `rules/*`, `agents/index.json`) are untouched.

### Reversibility

- `config.json.bak` retained until the next successful upgrade or explicit `/specflow:upgrade --clean-backups`. To roll back: restore the `.bak`, downgrade the plugin version. The new on-demand directories (`admin/insights/`, `admin/archive/`) can be deleted manually if no run has populated them.
- No source markdown is modified by this migration. The PRD bodies, interviews, manifests, and registry files are untouched.

### Verify

- `admin/config.json.insights.minCorpusSize` (integer) and `insights.cadence` (string ∈ four documented values) exist.
- `admin/config.json.prune.thresholds.{decisionLog,guidelines,taskHistory}` blocks exist with the expected sub-keys.
- `admin/config.json.optimize.{targetCapUsd, judgementWords}` exist with the expected types.
- `admin/lessons.json` exists; parses as a JSON array; ready for `specflow:test --feedback` writes and `specflow:test` / `specflow:task` reads.
- `admin/config.json.specflowVersion === "2.3.0"`.
- `specflow:doctor` passes; new config blocks validate; existing surfaces report no drift.

### Failure modes

- **User declines `optimize.targetCapUsd` default.** Pause the migration; let the user enter a value. Refuse zero (an `/optimize` run with budget 0 is structurally a no-op and would surface as misleading).
- **User declines all `prune.thresholds` defaults.** Pause; let the user edit each. The defaults are calibrated for projects with quarterly cadences; projects on monthly cadences may want shorter windows.
- **Pre-2.3.0 project has no `/insights`/`/prune`/`/optimize` runs.** Expected — the on-demand directories don't exist yet. Migration succeeds; first run of each skill creates its directories.
- **`specflow:doctor` reports drift.** Migration succeeds the file-write step but surfaces the doctor report; user resolves before declaring the migration complete.

---

## v2.3 → v2.4

The Sprint 1 sweep release. Closes the three remaining v2.x open PRD questions and pre-emptively mitigates two structural risks surfaced by the v2.4+ master plan. No skill ships from scratch; all changes are additive across `setup`, `prd`, `task`, `test`, `develop`, `brief` SKILL.md bodies plus two new doctrine docs and a worked-example versioning policy.

### Scope

**Config schema additions** (`admin/config.json`):

- `brief.commitPolicy` — string ∈ `committed | derived`. Default `committed`. Drives `specflow:brief` policy banner + setup-time `.gitignore` write. Closes 011-brief-commit-policy.
- `skills.{name}.enabled` — block of per-skill toggle objects. Default seeds every shipped skill with `{ "enabled": true }`. Drives Phase A.0 toggle-check across skills (demonstrated in `specflow:develop` for v2.4; other skills adopt the pattern incrementally as touched). Closes 012-config-skill-toggles.

**Skill body changes** (no new skills):

- `specflow:setup` Phase 8.2 — prompts user for brief commit policy; writes the new config block; appends `*-brief.html` to `.gitignore` idempotently when user picks `derived`. Phase 8.3 documents `pages.json` lazy-population (no future `specflow:pages` skill ships — closes 009-pages-policy).
- `specflow:test` Phase C.2.1 — new sub-step lazy-populates `admin/pages.json` from Playwright-visited routes on UI test runs. Frontmatter `produces:` gains `admin/pages.json (lazy-appended on first UI run per route; never duplicates)`.
- `specflow:prd` Phase A.3 — ingests feature-local `design/` folder (`proposed.html`, `iteration-log.md`, `current.html`) when present; surfaces `Design intent (from design/):` bullets in *Codebase context*. Silent no-op when absent. Closes 010-design-readback.
- `specflow:task` Phase A.2.5 — partitions iteration-log entries by timestamp vs PRD frontmatter date; tasks gain `design-decision: iteration-N` field when matched; uncovered post-PRD entries surface scope-change candidate prompt.
- `specflow:brief` step 9 — reads `config.json.brief.commitPolicy` (default `committed` when absent for v2.3 backward-compat); renders one-line policy banner near top of HTML. Frontmatter `requires:` gains `admin/config.json (optional)`; `eval` extends to verify the banner.
- `specflow:develop` Phase A.0 — new sub-step before A.1 reading `admin/config.json.skills.develop.enabled`; refuses with the canonical message and returns when disabled. Treats missing field as enabled. Demonstrates the toggle pattern; other skills adopt it incrementally.

**New doctrine docs** (under `plugins/specflow/templates/admin/`):

- `skill-toggles.md` — resolver contract for `config.json.skills.{name}.enabled` (canonical refusal message format, backward-compat semantics, chain-breakage by design, upgrade handling).
- `example-versioning.md` — worked-example versioning policy (`templateVersion: vX.Y` frontmatter field, backfill / grandfather / retire decisions, audit script). Mitigates Risk A from master plan.
- `team-review-bridge.md` — `agent-teams:team-review` ↔ Gate-5 manifest mapping (severity table, field table, dedup rules, when-to-invoke trigger). Mitigates Risk B from master plan.

**Worked-example backfill:**

- All v2.0-v2.3 worked-example PRDs (`001-design-skill` through `008-optimize-skill`) gain `templateVersion: v2.3` frontmatter. Informational tag — content unchanged. Sprint 2 (the first real template-changing release) triggers the first audit run.

**New worked examples** (Sprint 1 features):

- `010-design-readback/` — full PRD + interview + tasks + Gate 2 manifest + design folder demonstrating the readback.
- `011-brief-commit-policy/` — full PRD + interview + tasks + Gate 2 manifest demonstrating the config knob.
- `012-config-skill-toggles/` — full PRD + interview + tasks + Gate 2 manifest demonstrating the toggle pattern.
- `013-example-migration-policy/` — full PRD + interview + tasks + Gate 2 manifest demonstrating the versioning policy.
- `014-team-bridge-spec/` — full PRD + interview + tasks (with sample `team-review` → Gate-5 translation) + Gate 2 manifest.

(009-pages-policy ships as a decide-not-build PRD-level closure; no worked-example folder needed — the resolution lives in `v2/docs/PRD.md` § Resolved decisions.)

**PRD § Open questions** — eight of nine resolved through the v2.x ship cycle; the remaining one (009-pages-policy original) closes here. v2.4.0 leaves `Open questions` empty. Refinements surface as new PRDs going forward.

### Steps the upgrade skill will execute

1. **Backup** `admin/config.json.bak`.
2. **Read** existing `admin/config.json`. If `brief` block is absent, prompt the user with the brief commit-policy question per `setup/SKILL.md` Phase 8.2; default `committed`. Write the new `brief.commitPolicy` field. If user picks `derived`, append `*-brief.html` to project `.gitignore` (create if absent; idempotent — never duplicate).
3. **Seed** the `skills` block: for every shipped v2.4 skill not already present, add `{ "enabled": true }`. Preserve any user-set toggles. Warn (one-line chat) but never delete orphan toggles for skills removed in this version (none in v2.3 → v2.4).
4. **Tag** existing worked examples — one-shot awk pass adding `templateVersion: v2.3` to every PRD frontmatter under `examples/docs/specflow/features/00[1-8]-*/` that doesn't already carry the field. (No-op for projects that haven't installed worked examples directly; primarily relevant for plugin contributors.)
5. **Stamp** `admin/config.json.specflowVersion` to `2.4.0`.
6. **Verify.** Run `specflow:doctor`. Confirm new config keys exist with expected types; confirm existing surfaces (`task-history.json`, `decision-log.md`, `lessons.json`, `rules/*`, `agents/index.json`, `pages.json`) are untouched.

### Reversibility

- `config.json.bak` retained until next successful upgrade or explicit `/specflow:upgrade --clean-backups`. To roll back: restore the `.bak`, downgrade the plugin version. The `*-brief.html` `.gitignore` line can be hand-removed if the user reverts to `committed` post-upgrade.
- No source markdown is modified by this migration. PRD bodies, interviews, manifests, and registry files are untouched.
- The `templateVersion` frontmatter field is informational; removing it has no functional effect.

### Verify

- `admin/config.json.brief.commitPolicy ∈ { "committed", "derived" }`.
- `admin/config.json.skills` exists with at least the v2.4 shipped skill names, each as `{ "enabled": true | false }`.
- `admin/config.json.specflowVersion === "2.4.0"`.
- Project `.gitignore` contains `*-brief.html` exactly once (only when `commitPolicy === "derived"`).
- `specflow:doctor` passes; new config blocks validate; existing surfaces report no drift.

### Failure modes

- **User declines the brief commit-policy prompt.** Use the default (`committed`) and proceed; the choice can be edited in `config.json` later. Surface a one-line note: *"Default brief.commitPolicy: committed. Edit `admin/config.json.brief.commitPolicy` to change."*
- **Project `.gitignore` is read-only or absent and uncreatable.** Pause; surface the failure path; let the user create the file manually with the snippet, then resume.
- **Existing v2.3 project has a `skills` block already (impossible per current schema, but guard against future schema drift).** Preserve it byte-for-byte; do not seed duplicates.
- **Worked-example backfill on a project that has cloned the plugin examples into its own repo.** The backfill targets the plugin's `examples/`, not the user's `docs/specflow/features/`. User-authored worked examples are never touched.

---

## v2.4 → v2.5

The Sprint 2 sweep release. Five features rewrite primary contracts (PRD/task/develop/brief). Adds four new doctrine docs, a new principle reviewer, two new config knobs, two new per-task fields, and a new core principle (TDD).

### Scope

**Config schema additions** (`admin/config.json`):

- `task.contextBudget` — integer, default 80000. Per-task token ceiling for the single-context-window rule (per 029-single-context-task). Tasks whose `context-budget-estimate` exceeds this value auto-flag at synthesis and route to `specflow:scope-change` for splitting.
- `develop.tddRequired` — boolean, default `true`. When `true`, Green lane behaves identically to Yellow on the Red artefact contract (per 017-tdd-discipline). When `false`, Green may skip Red. Knob applies to Green only — Yellow always enforces; Red is human-led.

**New doctrine docs** (chain-don't-absorb pattern):

- `templates/admin/single-context-task.md` — single-context-window-per-task contract (per 029).
- `templates/admin/tdd-discipline.md` — Pocock's Red → Green → Refactor cycle (per 017).
- `templates/task/cross-task-review.md` — Phase E.4.5 + Phase F doctrine for whole-set review (per 022).
- `templates/task/sprint-bucket-heuristic.md` — single-rule fixpoint heuristic + graph-validity diagnostics (per 025).

**New principle reviewer** (under `examples/docs/specflow/admin/agents/standard/principles/`):

- `cross-task-reviewer.md` — coherence + better-arrangement lenses applied to the entire task list at Gate 3 Round 2.5.

**New per-task fields** (`{NNN-slug}-tasks.md`):

- `sprint-bucket: N` — derived deterministically per `templates/task/sprint-bucket-heuristic.md`.
- `context-budget-estimate: <int_tokens>` — formalised in `templates/admin/single-context-task.md`.

**Skill body changes** (no new skills):

- `specflow:task` Phase B.3 / B.4 — adds the two new per-task fields, budget self-check, graph-validity check, sprint-bucket assignment.
- `specflow:task` Phase E — Phase E.4.5 (Cross-task review three-round mini-debate) inserted between E.4 and E.5; Phase E.5 hybrid R3 surface.
- `specflow:task` Phase E.6 closer — manifest gains `writer_id` / `cross_task_reviewer_id` / `applier_id` triplet, "Cross-task findings" H2 section, `passed-with-revisions` status added.
- `specflow:task` Phase F (NEW) — `--apply-cross-task-feedback {NNN-slug}` applier flow.
- `specflow:develop` Phase A.6 — context-budget pre-flight per in-scope task.
- `specflow:develop` Phase D — Red / Green / Refactor sub-step structure with cycle-marker contract; `tddRequired` knob.
- `specflow:develop` Phases D / E / F — single-context-window reminder.
- `specflow:test` `--plan-only --task T{N}` mode — per-task variant with `Status: red (failing)` default; B.5 skipped.
- `specflow:brief` Visual Block Grammar — four new blocks (`:::key-features`, `:::resources`, `:::key-decisions`, `:::phase-split`); inline SVG-base64 icon set for source-type pills; eight-kind grammar.
- `specflow:setup` Phase 8.2 — seeds `task.contextBudget: 80000` in the config.json template.

**New core principle** (`CORE_PRINCIPLES.md`):

- `## TDD` section adopting Pocock's Red → Green → Refactor framing.

### Steps the upgrade skill will execute

1. **Backup** `admin/config.json.bak`.
2. **Seed** `task.contextBudget: 80000` if absent. Preserve any user-set value.
3. **Seed** `develop.tddRequired: true` if absent. Preserve any user-set value.
4. **Stamp** `admin/config.json.specflowVersion` to `2.5.0`.
5. **Synthesis re-run** is NOT triggered automatically — existing tasks files retain their pre-2.5 shape (no `sprint-bucket: N` or `context-budget-estimate` field). New `specflow:task` runs synthesise the new shape; user can manually re-run on existing features if desired.
6. **Verify.** Run `specflow:doctor`. Confirm new config keys exist; confirm new doctrine docs are reachable from skill bodies.

### Reversibility

- `config.json.bak` retained until next successful upgrade or explicit `/specflow:upgrade --clean-backups`.
- No source markdown is modified by this migration. PRD bodies, interviews, manifests, and registry files are untouched.
- Existing tasks files retain their pre-2.5 shape; the new fields are forward-compatible (their absence on legacy task files is treated as "not yet bucketed; not yet budget-estimated").

### Verify

- `admin/config.json.task.contextBudget` is a positive integer.
- `admin/config.json.develop.tddRequired` is a boolean.
- `admin/config.json.specflowVersion === "2.5.0"`.
- New doctrine docs exist: `templates/admin/single-context-task.md`, `templates/admin/tdd-discipline.md`, `templates/task/cross-task-review.md`, `templates/task/sprint-bucket-heuristic.md`.
- New principle reviewer exists: `templates/agents/standard/principles/cross-task-reviewer.md` (or under `admin/agents/standard/principles/` once seeded by setup).
- `CORE_PRINCIPLES.md` has a `## TDD` section citing `templates/admin/tdd-discipline.md`.
- `specflow:doctor` passes.

### Failure modes

- **Existing tasks files have hand-edited fields colliding with new field names.** New fields are appended only when synthesis re-runs; legacy tasks files retain their shape. No silent overwrites.
- **`brief/SKILL.md` and `task/SKILL.md` exceed the ≤500-line skill-size ceiling after Sprint 2 additions.** Acknowledged tradeoff (see CHANGELOG `### Known acknowledged tradeoff`). Chain-don't-absorb extraction (e.g. sibling `brief-blocks` skill) can land in v2.6 or later.

---

## v2.5 → v2.6

The Sprint 3 sweep release. Five features extend operational runtime instrumentation: lessons-registry formalisation, per-task manifest, brand-consistency lens, reviewer-context-isolation contract, edge-case-reviewer.

### Scope

**Config schema additions** (`admin/config.json`):

- `prd.maxLessonsSurfaced` — integer, default 5. Caps lessons surfaced inline at PRD time (per 018).
- `task.maxLessonsSurfaced` — integer, default 5. Caps lessons surfaced at task time (per 018).

**New doctrine docs**:

- `templates/admin/lessons-registry.md` — formalises the read-write loop for `lessons.json` (per 018).
- `templates/admin/task-manifest-schema.md` — per-task lifecycle workspace at `debate-log/tasks/T-NN-manifest.md` (per 019).
- `templates/admin/reviewer-isolation.md` — fresh-context contract across all gates (per 027).

**New principle reviewer agent**:

- `examples/docs/specflow/admin/agents/standard/principles/edge-case-reviewer.md` — Gate 4 + Gate 5 reviewer with deliberately-not-goal-aware lens (per 028).

**Skill body changes**:

- `specflow:prd` Phase A.3.5 (NEW) — queries lessons.json; surfaces inline.
- `specflow:task` Phase B.3 — `prior-lessons: [L-NNN, ...]` per-task field.
- `specflow:test` Phase B.3 — Brand-consistency lens with 8 standard questions.
- `specflow:develop` Phase C.2 + Phase E.2 — `edge-case-reviewer` joins the reviewer set at Gate 4 + Gate 5.
- `specflow:setup` Phase 8.2 — seeds the new config knobs.

**Reviewer template updates** — each principle reviewer's role-def gains the "Does NOT consult the writer's chat" + `writer_id` non-equality clause + isolation citation.

**Orchestrator pattern extension** — new "Reviewer fresh-context dispatch" section + checklist bullet.

### Steps the upgrade skill will execute

1. **Backup** `admin/config.json.bak`.
2. **Seed** `prd.maxLessonsSurfaced: 5` and `task.maxLessonsSurfaced: 5` if absent. Preserve user-set values.
3. **Stamp** `admin/config.json.specflowVersion` to `2.6.0`.
4. **Manifest migration** — for active in-flight features whose tasks have shipped 017's interim stub, the new manifest format is forward-only; existing stubs remain valid until those features close. New features land directly in 019's format.
5. **Verify.** Run `specflow:doctor`.

### Reversibility

- `config.json.bak` retained until next successful upgrade.
- No source markdown is modified by this migration.
- Existing per-task scratch stubs (017 format) retained until feature closes; new features land in 019's format.

### Verify

- `admin/config.json.prd.maxLessonsSurfaced` and `admin/config.json.task.maxLessonsSurfaced` are positive integers.
- `admin/config.json.specflowVersion === "2.6.0"`.
- New doctrine docs exist: `templates/admin/lessons-registry.md`, `templates/admin/task-manifest-schema.md`, `templates/admin/reviewer-isolation.md`.
- New principle reviewer exists at `templates/agents/standard/principles/edge-case-reviewer.md` (or under `admin/agents/standard/principles/` once seeded).
- `specflow:doctor` passes.

### Failure modes

- **Existing per-task scratch stubs (017 format) on active features.** Retained as-is; no in-place migration. New features land in 019.
- **Reviewer-template updates land on the plugin's `templates/agents/`, not on user-installed `admin/agents/`.** Users get the updates via `specflow:setup --upgrade-agents` (idempotent re-copy from templates). Without this, hand-edited reviewer files retain their pre-v2.6 shape; the contract still holds because the orchestrator's pre-dispatch check uses the doctrine doc, not the reviewer's role-def text.

---

## v2.6 → v2.7

The Sprint 4 sweep release — and the closing release of the v2.x cycle. Two features: the new `specflow:sprint` skill (the first new skill since v2.3) and the `agent-teams-per-stage` doctrine.

### Scope

**New skill** (`plugins/specflow/skills/sprint/`):

- `specflow:sprint` — lightweight sprint planner invoked by `specflow:develop` Phase A.5.5 as a sub-step. Skill toggle `config.skills.sprint.enabled` defaults to `true` (per 012-config-skill-toggles).

**New doctrine doc**:

- `templates/admin/stage-teams.md` — Plan → Build → Test → Iterate → Validate first-class doctrine.

**Config schema additions** (`admin/config.json`):

- `develop.maxIssuesPerSprint` — integer, default 5.
- `teams` — object, default `{}` (empty). Per-stage roster overrides; doctrine defaults materialise on first sprint invocation.
- `skills.sprint` — `{ "enabled": true }` toggle.

**Skill body changes**:

- `specflow:develop` Phase A.5.5 (NEW) — invokes `specflow:sprint` as a sub-skill.
- `specflow:setup` Phase 8.2 — seeds the new config knobs and the new skill toggle.

### Steps the upgrade skill will execute

1. **Backup** `admin/config.json.bak`.
2. **Seed** `develop.maxIssuesPerSprint: 5` if absent. Preserve user-set values.
3. **Seed** `teams: {}` if absent. Preserve user-set values.
4. **Seed** `skills.sprint: { "enabled": true }` if absent. Preserve user-set values.
5. **Stamp** `admin/config.json.specflowVersion` to `2.7.0`.
6. **Verify.** Run `specflow:doctor`. Confirm new config keys; confirm the new skill is registered in `skills.{name}.enabled` toggles.

### Reversibility

- `config.json.bak` retained until next successful upgrade.
- No source markdown is modified by this migration.
- The new sprint skill and stage-teams doctrine are forward-only; v2.6 projects continue to work without invoking them (develop's Phase A.5.5 short-circuits when `skills.sprint.enabled === false`).

### Verify

- `admin/config.json.develop.maxIssuesPerSprint` is a positive integer.
- `admin/config.json.teams` exists (default `{}`).
- `admin/config.json.skills.sprint.enabled` exists (default `true`).
- `admin/config.json.specflowVersion === "2.7.0"`.
- New doctrine doc exists at `templates/admin/stage-teams.md`.
- New skill exists at `skills/sprint/SKILL.md`.
- `specflow:doctor` passes.

### Failure modes

- **Existing in-flight features have no `team_assignments` block in their tasks.md.** Sprint reads `config.json.teams.{stage}` (or doctrine defaults) at sprint-plan time and resolves assignments for the in-scope batch only. Pre-v2.7 features whose tasks have already shipped don't need backfill.

---

## Future entries

Future versions append below. Format above. Newest at top.

<!-- v2.x → v2.y entries land here as Phase 4+ ships breaking changes. -->
