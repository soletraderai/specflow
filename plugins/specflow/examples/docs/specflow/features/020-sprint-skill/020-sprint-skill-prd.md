---
feature: 020-sprint-skill
status: shipped
created: 2026-05-08
templateVersion: v2.7
shipped: v2.7.0
interview: ./020-sprint-skill-interview.md
---

# Sprint skill (with sprint-worktree absorbed)

## Vision

A new lightweight `specflow:sprint` skill (≤500-line target) is invoked by `specflow:develop` Phase A.5.5 as a sub-step. Sprint pulls the feature's mapped Linear project, reconciles with local `tasks.md`, filters to the in-scope batch via `sprint-bucket: N` (per 025) and `config.develop.maxIssuesPerSprint` (default 5), synthesises a sprint plan with per-stage team assignments per 026, presents the sprint-plan gate to the developer, and on approval creates a git work-tree at `admin/scratch/{NNN-slug}-sprint/worktree/`. Returns the approved plan to develop, which iterates Phase B-F across the approved batch.

## Problem

Pre-020, `specflow:develop` operated one task at a time per invocation. Multi-task sessions required manual orchestration: read tasks.md, decide which to do, manually invoke develop for each. The Linear MCP integration was per-task (Backlog→In Progress at A.4; In Progress→In Review at F.4) but had no project-level pull or batch-shaping. Per-task `sprint-bucket: N` (025) and `context-budget-estimate` (029) existed but had no consumer at the sprint level.

## Goals

- New skill at `plugins/specflow/skills/sprint/SKILL.md` — single-purpose, ≤500-line target, invoked by develop's Phase A.5.5.
- Sprint pulls Linear (when MCP available) + reads local tasks.md + reconciles drift (Linear-only, local-only, status drift).
- Sprint filters to the in-scope batch using `sprint-bucket: N` + `config.develop.maxIssuesPerSprint` (default 5).
- Sprint synthesises a plan with per-stage team assignments per 026's `templates/admin/stage-teams.md` doctrine.
- Sprint-plan gate fires for developer approve / adjust / defer.
- On approval, sprint creates a git work-tree (one per sprint by default; opt-in per-issue parallelism).
- Sprint returns the approved plan as a structured result; develop iterates Phase B-F over the in-scope batch.

## Non-goals

- Building 026 itself. 020 consumes 026's doctrine; 026 ships in this same sprint.
- Cross-feature sprint planning (multi-Linear-project batches). Parked.
- Auto-promoting bucket numbers into Linear cycle assignments. Out of scope.
- Linear-side issue creation. Linear is the source of truth; specflow reads, doesn't create issues there.

## Requirements

- **R1.** New skill `plugins/specflow/skills/sprint/SKILL.md` with frontmatter (`name: specflow:sprint`, `requires:`, `produces:`, `eval:`).
- **R2.** Sprint Phase A reads `admin/config.json` (incl. `develop.maxIssuesPerSprint`, `teams.{stage}`) + `admin/environment.json` + `tasks.md` + `task-history.json` + Gate 3 manifest.
- **R3.** Sprint Phase B pulls Linear (when available) and detects drift; absent MCP degrades gracefully.
- **R4.** Sprint Phase B.3 filters to the in-scope batch by `sprint-bucket: N` and `develop.maxIssuesPerSprint`.
- **R5.** Sprint Phase C synthesises the plan with per-task `team_assignments` block per 026.
- **R6.** Sprint Phase D fires the sprint-plan gate; on approval creates a git work-tree.
- **R7.** Sprint returns the approved plan (manifest path, task IDs, work-tree path, team assignments) to the parent develop invocation.
- **R8.** `specflow:develop` Phase A.5.5 invokes sprint as a sub-skill; Phase B operates over sprint's `tasks_in_scope`.
- **R9.** New config knob `config.develop.maxIssuesPerSprint` (default 5) seeded by setup.
- **R10.** Sprint refuses standalone invocation with the canonical message: *"`specflow:sprint` is invoked by `specflow:develop` Phase A.x as a sub-step. Run `specflow:develop {NNN-slug}` to begin a sprint."*

## Acceptance criteria

- **AC-1.** `plugins/specflow/skills/sprint/SKILL.md` exists with `name: specflow:sprint` in the frontmatter.
- **AC-2.** Sprint's eval field references the sprint-plan manifest closing decision.
- **AC-3.** `specflow:develop` Phase A.5.5 cites `skills/sprint/SKILL.md` and forks the sub-skill.
- **AC-4.** Setup seeds `config.develop.maxIssuesPerSprint: 5`.
- **AC-5.** Sprint refuses standalone invocation with the canonical message.

## See also

- Skill: `plugins/specflow/skills/sprint/SKILL.md`
- Stage-teams doctrine: `plugins/specflow/templates/admin/stage-teams.md` (per 026)
- 022 — cross-task-review (consumed before sprint plans)
- 025 — sprint-task-flagging (`sprint-bucket: N` is the filter input)
- 029 — single-context-task (`context-budget-estimate` is surfaced in the plan)
