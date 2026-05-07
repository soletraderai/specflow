---
feature: 026-agent-teams-per-stage
status: shipped
created: 2026-05-08
templateVersion: v2.7
shipped: v2.7.0
interview: ./026-agent-teams-per-stage-interview.md
---

# Agent teams per stage

## Vision

Promote the canonical pipeline shape — Plan → Build → Test → Iterate → Validate — to first-class doctrine. Each stage has a designated agent team configurable per project via `config.json.teams.{stage}`. Default rosters and override path are documented in `plugins/specflow/templates/admin/stage-teams.md`. Sprint planner (020) emits per-task team assignments; develop / test skills consume them at their respective gates. Cross-cuts with 027-reviewer-context-isolation (every team member runs in fresh context).

## Problem

Pre-026, agent dispatch was ad-hoc per skill: develop had its own reviewer set at Gates 4 + 5; test had its own; sprint did not exist. The Plan / Build / Test / Iterate / Validate framing the user articulated had no codified home in the plugin. Without per-stage teams, multi-task sessions composed agents inconsistently across develop and test.

## Goals

- Doctrine doc at `plugins/specflow/templates/admin/stage-teams.md` defining the five stages, default rosters, config schema, override path, and consumption by 020 / develop / test.
- `config.json.teams.{stage}` schema with default rosters per stage. Empty `{}` seeds the doctrine defaults at first invocation.
- 020-sprint-skill emits per-task `team_assignments` block resolved against `config.json.teams.{stage}` (or the doctrine defaults).
- Develop / test consume the assignments at their respective gates without falling back to defaults silently.

## Non-goals

- Building 020 itself. Co-designed in this sprint; 020 consumes 026's doctrine.
- Auto-detecting the optimal team composition. The defaults are the anchor; project-taste adjustments are explicit.
- Per-task team overrides at the develop layer (not at sprint). The sprint-plan gate is the only override surface.

## Requirements

- **R1.** Doctrine doc at `plugins/specflow/templates/admin/stage-teams.md` with the five stages, default rosters, config schema, override path, and consumption notes.
- **R2.** `config.json.teams.{stage}` schema documented in the doctrine doc and seeded as `teams: {}` (empty) by setup.
- **R3.** 020's `specflow:sprint` Phase C.1 step 3 reads `config.json.teams.{stage}` and resolves per-task team assignments.
- **R4.** Develop / test SKILL.md citations to the doctrine doc at the consumption points.

## Acceptance criteria

- **AC-1.** Doctrine doc exists at `plugins/specflow/templates/admin/stage-teams.md` with the five stages enumerated.
- **AC-2.** Setup seeds `config.json.teams: {}` (empty object) for default-roster materialisation by sprint.
- **AC-3.** `specflow:sprint` Phase C.1 cites `templates/admin/stage-teams.md`.
- **AC-4.** Default rosters cover all five stages: Plan / Build / Test / Iterate / Validate.

## See also

- Doctrine: `plugins/specflow/templates/admin/stage-teams.md`
- 020 — sprint-skill (consumer)
- 022 — cross-task-review (Iterate team's edge-case-reviewer informs better-arrangement lens)
- 023 — test-brand-consistency (Test team's brand-consistency lens)
- 027 — reviewer-context-isolation (every team member runs in fresh context)
- 028 — edge-case-reviewer (Iterate team's principle reviewer)
