# Skill toggles — resolver contract

`config.json.skills.{name}.enabled` is the project-level on/off knob for every shipped specflow skill. Introduced in v2.4.0 (`012-config-skill-toggles`). Default `true` for every skill; users disable selectively when their workflow requires it.

## Why this exists

Some projects don't want every specflow skill firing. Examples:

- An org standardising on GitHub Copilot or Cursor for implementation turns off `specflow:develop` but keeps the planning-and-review chain (`specflow:prd`, `specflow:task`, `specflow:test`).
- A research-heavy team uses `specflow:design` and `specflow:brief` for stakeholder review but doesn't run the develop loop.
- A regulated environment disables `specflow:misc` because all changes must go through the PRD chain.

Without a toggle, those projects fork the plugin or hand-edit `SKILLS.md`. The toggle is the lighter-weight path.

## The contract

Every skill's Phase A pre-flight reads `docs/specflow/admin/config.json` (already required by most skills' `requires:`). The first sub-step of Phase A — before any other read — checks:

```
config.skills.{this-skill-name}.enabled === false  →  refuse with one-line message
```

The skill name is the shipped name without the `specflow:` prefix (e.g. `develop`, `task`, `prd`). Bare-name skills (`/grill`, `/optimize`, `/prune`, `/insights`, `/simplify`, `/panic`, `/feedback-loop-audit`, `/confidence-check`) use their bare names.

### Refusal message format

```
specflow:{skillName} is disabled in this project (config.skills.{skillName}.enabled = false).
Re-enable in docs/specflow/admin/config.json or invoke a different skill.
```

The skill returns immediately. No partial work, no warnings, no scratch directory creation, no Linear writes. Disabled means disabled.

### Backward compatibility

Skills MUST treat a missing `config.json.skills.{name}` field as `enabled: true`. Reasons:

1. v2.3 and prior projects don't have the field.
2. The default-true posture preserves existing project behaviour on upgrade.
3. `specflow:upgrade` (when run) seeds the field with `enabled: true` for every shipped skill — explicit no-op for projects that don't care to disable anything.

### Disabling chain skills

Disabling a chain skill (`prd`, `task`, `develop`, `test`, `complete`) breaks the chain. The toggle does not auto-re-route the chain; downstream skills that try to read the missing artefact fail their `requires:` check normally. This is by design — the chain is opinionated, and disabling part of it surfaces the consequences explicitly rather than silently re-routing.

For projects that want the chain in part, the supported pattern is:

- Disable `specflow:develop` only (keep `prd` → `task` → `test` for spec-and-review without auto-implementation).
- Or disable `specflow:test` only (keep `prd` → `task` → `develop` if the project's testing surface is governed elsewhere).

Disabling `specflow:prd` or `specflow:task` is supported but unusual — those skills produce the artefacts every other chain skill `requires:`. A project disabling them is opting out of specflow's planning discipline, which makes most other skills unhelpful.

### Disabling Phase 3 skills

`/insights`, `/prune`, `/optimize` are independent — disabling any one does not break the others. Each runs on its own cadence and reads independent corpora.

## How `specflow:upgrade` handles the toggle field

On upgrade from v2.3 → v2.4 (and forward), `specflow:upgrade` examines the existing `config.json`:

- If `skills` field is absent → seed with all-enabled.
- If `skills` field is present → preserve existing per-skill toggles; add any new skills (introduced since the project's last upgrade) with `enabled: true`.
- If a skill ships in v2.x but is removed in v(2.x+1) → leave the orphan toggle in place; warn but never delete (user data preservation).

## Worked example

See `examples/docs/specflow/features/012-config-skill-toggles/`.

## Resolution citation

`v2/docs/PRD.md` § Resolved decisions — 012-config-skill-toggles v2.4.0.
