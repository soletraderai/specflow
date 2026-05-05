---
name: specflow:agent
description: Manage the per-repo agent registry — add, remove, list, refresh. Snapshots agent definitions into admin/agents/specialised/ pinned and reviewable.
status: v2-new
phase: 2
requires: [docs/specflow/admin/environment.json]
produces: [docs/specflow/admin/agents/specialised/, docs/specflow/admin/agents/index.json]
eval: every snapshotted agent in admin/agents/index.json resolves to an installed source plugin; specflow:doctor passes.
---

# specflow:agent

Manage the per-repo agent registry. The registry is a browsable, indexed set of agents available for use in this project — standards always present, specialised matched to the project's stack.

**Triggers:**
- `/specflow:agent list` — print standards + specialised, with source plugin and last-snapshot date.
- `/specflow:agent add {name}` — search the global index by name/tag, snapshot into `specialised/`, update `index.json`.
- `/specflow:agent remove {name}` — remove a specialised agent from the registry. Standards cannot be removed.
- `/specflow:agent refresh {name|--all}` — re-snapshot from the current upstream source.

**How indexing works:**
- specflow scans all installed agent sources (plugins under `~/.claude/plugins/`).
- Builds a global index in memory: name, source plugin, tags.
- On `add`, copies the full agent definition into `admin/agents/specialised/`. Pinned and reviewable; shows as a diff in PRs when the upstream changes.
- `admin/agents/index.json` records: name, source plugin, tags, snapshot-date, source-version.

**Naming collisions:** namespace snapshot filenames (`{plugin}__{agent}.md`) when two marketplaces ship an agent with the same name.

**Wire-in to other skills:**
- `specflow:setup` — copies standards, runs the specialised proposal during tech detection, snapshots confirmed agents.
- `specflow:upgrade` — diffs snapshots against current upstream; flags new agents in marketplaces relevant to this project's stack.
- `specflow:doctor` — validates every indexed agent still resolves to an installed source.
- `specflow:prd`, `specflow:task` — read the agents section of `environment.json` to propose appropriate specialists.

**Verify steps:**
1. After `add`: snapshot file exists in `admin/agents/specialised/`; `index.json` updated.
2. After `remove`: snapshot file gone; `index.json` updated.
3. After `refresh`: snapshot-date updated; diff against the previous snapshot is reviewable in `git log`.
4. `specflow:doctor` agent-resolution check passes for every entry.

**Reference:** PRD Phase 2 scope items 2-5; Appendix K (full skill spec).
