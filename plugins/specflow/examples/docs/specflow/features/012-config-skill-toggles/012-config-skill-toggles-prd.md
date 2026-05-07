---
feature: 012-config-skill-toggles
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
interview: ./012-config-skill-toggles-interview.md
---

# 012 — `config.json.skills.{name}.enabled` toggles

## Vision

Add a project-level on/off knob for every shipped specflow skill. Default `true` for all; users disable selectively when their workflow doesn't need a particular skill. The resolver contract is documented once, in `templates/admin/skill-toggles.md`; each skill's Phase A pre-flight checks its own toggle as the first sub-step (A.0) and refuses with a canonical one-line message when disabled. The change unlocks three real configurations the plugin currently can't express: orgs that prefer GitHub Copilot or Cursor for implementation (disable `specflow:develop`); research-heavy teams that use only the planning surface (disable everything past `specflow:brief`); regulated environments where every change must run through the PRD chain (disable `specflow:misc`). Setup seeds all skills enabled; users opt-out, never opt-in. Backward-compat is structural — missing field == enabled.

## Problem

Every project that adopts specflow currently inherits the full skill set whether they want it or not. A team that does not use `specflow:develop` (because they implement features in another tool) still has the skill available, still has implementations recommend it, and still has documentation referencing it. Worse, the chain assumption (`prd` → `task` → `develop` → `test` → `complete`) is hard-coded across reviewers and the README — there's no canonical answer to *"how do I run specflow without develop?"*. The same applies to `specflow:misc` (regulated repos shouldn't allow out-of-PRD changes) and the Phase 3 trio (`/insights`, `/prune`, `/optimize` — projects that don't run them shouldn't see them suggested in `complete`'s closing chat). Without a toggle the workaround is forking the plugin or hand-editing `SKILLS.md` per project — both worse than a one-field config knob. The fix is a `skills.{name}.enabled` block in `config.json`, default `true` for every shipped skill, with a documented resolver contract that each skill's Phase A.0 enforces.

## Goals

- New config schema field `config.json.skills` — an object whose keys are shipped skill names (sans `specflow:` prefix) and whose values are `{ "enabled": boolean }`.
- Default value at setup is `enabled: true` for every shipped skill (current set: `prd`, `task`, `develop`, `test`, `complete`, `decision`, `scope-change`, `design`, `brief`, `misc`, `agent`, `insights`, `prune`, `optimize`).
- Resolver contract documented in `plugins/specflow/templates/admin/skill-toggles.md`. Each skill's Phase A.0 pre-flight reads `config.json` and refuses if `config.skills.{this-skill}.enabled === false`.
- Refusal message is canonical and verbatim across skills: *"`specflow:{skill} is disabled in this project (`config.skills.{skill}.enabled = false`). Re-enable in `docs/specflow/admin/config.json` or invoke a different skill."*
- Missing `config.skills.{name}` field treated as `enabled: true` (backward-compat for v2.3 projects).
- Disabling a chain skill (`prd`, `task`, `develop`, `test`, `complete`) breaks the chain by design — downstream skills' `requires:` checks fail naturally; no auto-rerouting. The toggle surfaces consequences, not silences them.
- v2.4.0 demonstrates the pattern in `specflow:develop`'s SKILL.md (the most-likely-to-be-disabled skill per user feedback). Other skills adopt the same Phase A.0 step as they're touched in subsequent sprints — the contract is documented once; per-skill enforcement is mechanical and migration-safe.
- `specflow:upgrade` (when run on a v2.3 → v2.4 project) seeds the `skills` block with all-enabled and surfaces the new toggle field as a one-line note, never silently rewrites existing user toggles.

## Non-goals

- **Per-feature toggles.** The knob is project-level. A project does not say "`specflow:develop` is disabled for feature 042 but enabled for everything else"; that's a workflow lifestyle choice the toggle is too coarse for. If a feature needs implementation outside the chain, the user invokes another tool directly.
- **Conditional toggles based on file paths or task properties.** No "disable develop for confidentialPaths only"; that's the Red lane's responsibility. Toggles are on/off, full stop.
- **Auto-detection of which skills a project should enable.** No heuristic on detected stack, repo size, or workflow signals. Setup seeds default-on; the user picks.
- **Runtime auto-rerouting when a chain skill is disabled.** No fallback "if `specflow:develop` is disabled, route to `specflow:misc` instead". Disabled means disabled; the chain breaks visibly so consequences are clear.
- **External-system propagation of the toggle state.** The toggle only affects local skill invocation. Linear / GitHub / external CI systems are unaware; that's their concern, not the plugin's.

## Users

- **Setup operators on a fresh repo** who want every skill enabled (the default-good answer). They never touch the `skills` block; setup seeds it; downstream skills check it transparently.
- **Setup operators on an opinionated repo** who know upfront they don't want a particular skill (e.g. Copilot-using orgs disabling `develop`). They edit `config.json` post-setup, set `enabled: false`, and the disabled skill refuses with the canonical message on next invocation.
- **`specflow:upgrade` users** crossing v2.3 → v2.4. They see the new field seeded with all-enabled; their existing workflow is unchanged. If they later want to disable a skill, the field is already there to flip.
- **Skill authors writing new skills** in v2.4+. They follow the resolver contract: Phase A.0 reads the toggle; refusal message is canonical. The doctrine doc tells them what to do.

## Requirements

- **R1.** `config.json` schema gains a top-level `skills` object. Keys are shipped skill names (sans prefix); values are `{ "enabled": boolean }`. Initial seeded keys are the v2.4 shipped set: `prd`, `task`, `develop`, `test`, `complete`, `decision`, `scope-change`, `design`, `brief`, `misc`, `agent`, `insights`, `prune`, `optimize`.
  - Trace: skills/setup/SKILL.md Phase 8.2 (v2.4.0).
  - Serves goal: schema field exists, default is set.

- **R2.** Resolver contract documented in `plugins/specflow/templates/admin/skill-toggles.md`. Doc covers: the canonical refusal message, backward-compat (missing field = enabled), chain-breakage by design, how `specflow:upgrade` handles the field across version transitions.
  - Trace: templates/admin/skill-toggles.md (new in v2.4.0).
  - Serves goal: contract is documented once; skill authors follow it.

- **R3.** `specflow:develop` Phase A.0 enforces the toggle. New sub-step before A.1; reads `admin/config.json`; refuses with the canonical message and returns when `config.skills.develop.enabled === false`. Treats missing field as enabled.
  - Trace: skills/develop/SKILL.md Phase A.0 (v2.4.0).
  - Serves goal: pattern is demonstrated in the highest-stakes skill.

- **R4.** Refusal message is canonical, verbatim, and skill-name-parameterised:
  > *"`specflow:{skill}` is disabled in this project (`config.skills.{skill}.enabled = false`). Re-enable in `docs/specflow/admin/config.json` or invoke a different skill."*
  - Trace: templates/admin/skill-toggles.md § Refusal message format.
  - Serves goal: cross-skill consistency for refusal copy.

- **R5.** Disabled skills do no work. No partial reads, no scratch directory creation, no Linear writes, no chat output beyond the refusal message itself. Refusal-and-return is atomic.
  - Trace: templates/admin/skill-toggles.md § The contract.
  - Serves goal: disabled = absent at the skill boundary.

- **R6.** `specflow:upgrade` (or any tool that mutates `config.json`) seeds missing skill toggles with `enabled: true` on cross-version transitions; preserves existing user-set toggles; warns but does not delete orphan toggles when a skill is removed in a future version.
  - Trace: templates/admin/skill-toggles.md § How `specflow:upgrade` handles the toggle field.
  - Serves goal: upgrade is non-destructive.

- **R7.** Per-skill rollout is mechanical: future skills adopt the Phase A.0 pattern as they're touched. v2.4 demonstrates the pattern in `specflow:develop`; other skills inherit the contract via doctrine but only enforce when they're next edited (avoids touching every SKILL.md in one sprint and triggering needless gate manifests).
  - Trace: this PRD's Vision section + project plan Sprint 1.
  - Serves goal: change is incremental, not all-at-once.

## Acceptance criteria

- **AC-1.** Running `specflow:setup` on a fresh repo writes `config.json` with a `skills` block listing every shipped v2.4 skill, each with `{ "enabled": true }`. Verifies R1.
- **AC-2.** `plugins/specflow/templates/admin/skill-toggles.md` exists and documents: refusal message, backward-compat, chain-breakage, upgrade handling. Verifies R2.
- **AC-3.** Reading `plugins/specflow/skills/develop/SKILL.md` shows a Phase A.0 sub-step that reads `config.json`, refuses with the canonical message when `develop.enabled === false`, returns. Verifies R3.
- **AC-4.** The refusal message in develop's Phase A.0 matches the canonical format verbatim. Verifies R4.
- **AC-5.** A v2.3 project (no `skills` field in `config.json`) running `specflow:develop` proceeds normally — backward-compat default-true. Verifies R6 backward-compat path.
- **AC-6.** A v2.4 project setting `config.skills.develop.enabled: false` and running `specflow:develop` produces only the refusal message — no scratch directory, no Linear status update, no other side effects. Verifies R5.
- **AC-7.** v2.3 → v2.4 MIGRATIONS entry covers the new `skills` field, the seeded all-true default, and the upgrade-time seeding behaviour. Verifies R6 upgrade path.
- **AC-8.** Worked example folder `examples/docs/specflow/features/012-config-skill-toggles/` exists with PRD, interview, tasks file, Gate 2 manifest closed `passed`. Verifies the dogfood discipline.

## Open questions

None — the toggle surface is small and bounded.

## See also

- `plugins/specflow/skills/setup/SKILL.md` Phase 8.2 — the schema producer
- `plugins/specflow/skills/develop/SKILL.md` Phase A.0 — the canonical pattern demonstration
- `plugins/specflow/templates/admin/skill-toggles.md` — the resolver contract
- `plugins/specflow/MIGRATIONS.md` v2.3 → v2.4 — the migration entry (Sprint 1 close)
- `v2/docs/PRD.md` § Resolved decisions — the resolution citation
