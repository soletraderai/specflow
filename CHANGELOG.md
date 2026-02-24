# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com),
and this project adheres to [Semantic Versioning](https://semver.org/).

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

[1.2.0]: https://github.com/soletraderai/specflow/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/soletraderai/specflow/compare/v1.0.1...v1.1.0
[1.0.1]: https://github.com/soletraderai/specflow/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/soletraderai/specflow/releases/tag/v1.0.0
