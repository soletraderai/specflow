---
name: specflow:linear
description: Export tasks (and misc-tasks) to Linear with bidirectional sync. Updates the file's Export Map with Linear IDs and URLs. Two source modes — feature tasks (one feature at a time) and misc tasks (rolling file). Soft requirement: Linear MCP installed.
status: shipped
phase: 1
requires:
  - docs/specflow/admin/config.json
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md
  - docs/specflow/misc-task/000-tasks-misc-tasks.md
produces: []
eval: every source-file task has a Linear ID in the Export Map after export; round-trip status updates surface in the source file's status column on next sync; if the misc Linear project doesn't exist, it's created and the project ID stored in admin/config.json.linear.miscProject.

---

# specflow:linear

Export tasks and misc-tasks to Linear with bidirectional sync. Soft requirement: Linear MCP installed.

**Triggers:** `specflow:linear {NNN-slug}` (feature tasks), `specflow:linear --misc` (misc rolling file), `specflow:linear --sync` (round-trip status update from Linear back to source file), `/specflow:linear` (asks).

**Inputs:**
- For feature export: `features/NNN-{slug}/{NNN-slug}-tasks.md` (note the `NNN-{slug}-` prefix per the v2 file-naming convention — handoff decision #9).
- For misc export: `docs/specflow/misc-task/000-tasks-misc-tasks.md`.
- Always: `admin/config.json.linear` for team / project mapping. For misc tasks, targets `linear.miscProject` (auto-created if missing — check first, create if absent, store ID).

**v2 changes from v1:**
- Source path moved: `task/NNN-tasks-{slug}.md` → `features/NNN-{slug}/{NNN-slug}-tasks.md`.
- Misc rolling file path unchanged: `docs/specflow/misc-task/000-tasks-misc-tasks.md`.
- Reads `admin/config.json.linear` for project mapping.
- Auto-creates the misc Linear project if it doesn't exist.

**Verify steps:**
1. Every task in the source file has a Linear ID in the Export Map after export.
2. Linear issues match source-file content (title, description, scope labels, MISC-NNN id when applicable).
3. Round-trip: status changes in Linear surface in the source file's status column on next sync.
4. The misc Linear project exists (auto-created if absent) and its ID is stored in `admin/config.json.linear.miscProject`.

**Reference:** v1 SKILL.md at `plugins/specflow/skills/linear/SKILL.md` for the existing implementation that v2 builds on; PRD Appendix B2 (Linear integration for misc).
