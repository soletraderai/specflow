# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.8.0] - 2026-02-26

### Changed

- Task sizing model reduced from 8-hour max to 1-hour max per task
  - New 4-tier model in 15-minute increments: Trivial (0.25h), Small (0.5h), Medium (0.75h), Full (1h)
  - Expected task count per PRD increases from 5-10 to 15-40
  - Multi-task dependency chains are the expected pattern for user-facing behaviors
  - Review framework updated: all "over 8 hours" thresholds now "over 1 hour"
  - Summary and template examples use decimal hour notation (0.5h, 0.75h, 1h)

## [1.7.0] - 2026-02-25

### Changed

- PRD and Task skill agent analysis now delegates to orchestrator (`roles.orchestrator` / `agent-teams:team-lead`) which decides which specialist agents to spawn, instead of the skill template prescribing specific agents
- PRD Phase 3 error/loading probing sharpened — enumerates per network call and external resource instead of generic "what are the error states?"
- PRD Phase 3 concurrency probing includes concrete examples (double-submit, rapid toggle)
- PRD Phase 4 module review scales to feature complexity — inline confirmation for 3 or fewer modules, full review round for larger designs

### Added

- PRD Phase 1 now explicitly supports solution-focused input — extracts the implied problem and confirms with user instead of asking them to reframe
- PRD Phase 7 completeness check for error/loading states — flags every async interaction missing error or loading UX as a Warning
- Task skill updates parent PRD status from `Draft` to `Accepted` when user approves the task breakdown

## [1.6.0] - 2026-02-25

### Added

- Sequential PRD numbering — PRDs named `{NNN}-{name}.md` (e.g., `001-user-auth.md`), task reviews named `{NNN}-tasks-{name}.md`, `prd_id: PRD-NNN` in frontmatter, prime displays IDs
- Linear re-export with update-instead-of-duplicate — export map drives update vs create, orphan warnings, incremental saves for partial failure recovery

### Removed

- Empty placeholder skill directories (`orchestrator/`, `reviewer/`)

## [1.5.0] - 2026-02-25

### Added

- Project Constitution support — define non-negotiable coding principles, architectural boundaries, and quality standards during setup
  - Setup skill gains Phase 3b: optional constitution interview (max 5 principles per category)
  - Constitution stored in `config.json` under `constitution` key; omitted entirely if skipped
  - PRD skill threads constitution through interview (Phase 3), module validation (Phase 4), PRD template (Phase 5), and self-review (Phase 7)
  - Task skill checks constitution compliance during codebase exploration (Step 3), vertical slice rules (Step 4), and critical review (Step 6)
  - Prime skill surfaces constitution principle counts in session summary
- Brownfield/migration PRD support — better handling of refactors, migrations, and modifications to existing systems
  - PRD Phase 2 auto-classifies greenfield vs. brownfield with user confirmation
  - PRD Phase 3 adds brownfield-specific interview areas: current state analysis, migration/transition, backward compatibility, rollback/risk
  - PRD template gains `type: greenfield | brownfield` frontmatter and conditional Migration Strategy section (Current State, Transition Plan, Backward Compatibility, Rollback Plan)
  - PRD Phase 7 self-review extends completeness and feasibility lenses for brownfield
  - Task skill adds brownfield first-slice rule: migration scaffolding before behavioral changes

## [1.4.0] - 2026-02-25

### Added

- Human QA verification pipeline threaded through the entire PRD → Task → Linear workflow
  - PRD user stories now require 2-4 Given/When/Then verification scenarios each
  - PRD Testing Decisions section includes human QA strategy guidance
  - Task vertical slices include QA Verification (GWT checkboxes) and Definition of Done sections
  - Task critical review checks for QA Verification coverage on every task
  - Linear issue template exports QA Verification and Definition of Done verbatim from the task review
  - Linear export summary recommends adding a "QA Review" status to the team workflow

## [1.3.0] - 2026-02-24

### Added

- New `/specflow:prime` skill for session bootstrapping
  - Loads project context (config, specflow state, codebase configs, git status) at conversation start
  - Uses tech stack detection from config.json to dynamically determine which files to read
  - Supports optional `prime.files` in config.json for repo-specific files
  - Read-only — no side effects, stays within ~2,000-5,000 token context budget
  - Graceful degradation: works without config.json via auto-detection
- Setup skill now offers optional prime configuration (Phase 5b) for custom session files

## [1.2.0] - 2026-02-24

### Added

- PRD skill now leverages installed specialist agents for codebase exploration and enhanced devil's advocate review
- Task skill now leverages installed specialist agents for codebase exploration and critical task review
- Setup skill installs 3 base agent plugins (agent-teams, comprehensive-review, agent-orchestration) before tech-stack plugins
- Config.json now includes `agents.base`, `agents.techStack`, and `agents.roles` mappings for decoupled agent invocation
- Built-in review frameworks in PRD (completeness, consistency, feasibility, assumptions) and task (slice quality, estimate accuracy, coverage, dependency graph) skills

## [1.1.0] - 2026-02-24

### Added

- New `/specflow:setup` skill for first-run configuration
  - Creates `docs/specflow/prd/` and `docs/specflow/task/` directories
  - Verifies Linear MCP configuration and offers to set it up
  - Scans repository to detect tech stack (languages, frameworks, databases, infrastructure)
  - Installs relevant specialist agents from `wshobson/agents` marketplace
  - Saves setup state to `docs/specflow/config.json` for use by other skills

## [1.0.1] - 2026-02-24

### Changed

- Updated marketplace.json with owner metadata
- Bumped plugin version to 1.0.1

## [1.0.0] - 2026-02-24

### Added

- Initial release with 3 skills: `specflow:prd`, `specflow:task`, `specflow:linear`
- PRD creator with structured discovery interview and codebase exploration
- Task breakdown into vertical slices with dependency tracking and time estimates
- Linear export with project creation, issue sequencing, and `blockedBy` relationships
- Namespaced skill names with `specflow:` prefix for clearer autocomplete
- Marketplace structure with plugin subdirectory

[1.8.0]: https://github.com/soletraderai/specflow/compare/v1.7.0...v1.8.0
[1.7.0]: https://github.com/soletraderai/specflow/compare/v1.6.0...v1.7.0
[1.6.0]: https://github.com/soletraderai/specflow/compare/v1.5.0...v1.6.0
[1.5.0]: https://github.com/soletraderai/specflow/compare/v1.4.0...v1.5.0
[1.4.0]: https://github.com/soletraderai/specflow/compare/v1.3.0...v1.4.0
[1.3.0]: https://github.com/soletraderai/specflow/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/soletraderai/specflow/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/soletraderai/specflow/compare/v1.0.1...v1.1.0
[1.0.1]: https://github.com/soletraderai/specflow/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/soletraderai/specflow/releases/tag/v1.0.0
