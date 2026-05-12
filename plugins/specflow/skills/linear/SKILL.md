---
name: specflow:linear
description: Ensure a Linear project exists for the feature, then export tasks (and misc-tasks) to Linear with bidirectional sync. Updates the file's Export Map with Linear IDs and URLs. Two source modes — feature tasks (one feature at a time, with project-ensure) and misc tasks (rolling file). Soft requirement: Linear MCP installed.
status: shipped
phase: 1
requires:
  - docs/specflow/admin/config.json
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-feature.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md
  - docs/specflow/misc-task/000-tasks-misc-tasks.md
produces:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-feature.md (frontmatter `linear_project_id` set on first feature-mode run)
eval: for feature-mode runs, a Linear project named `{NNN-slug}` exists (reused if already present in Linear by name, created if absent), and its ID is persisted to `{NNN-slug}-feature.md` frontmatter under `linear_project_id`; every distinct `sprint-bucket: N` value in the tasks file has a corresponding `Sprint N` project milestone (reused if already present, created if absent — additive only); every source-file task has a Linear ID in the Export Map after export, every issue is attached to the resolved project, and every issue is attached to the `Sprint N` milestone where N is its source task's sprint-bucket; round-trip status updates surface in the source file's status column on next sync; if the misc Linear project doesn't exist, it's created and the project ID stored in admin/config.json.linear.miscProject.

---

# specflow:linear

Ensure a per-feature Linear project, then export tasks (and misc-tasks) to Linear with bidirectional sync. Soft requirement: Linear MCP installed.

**Triggers:** `specflow:linear {NNN-slug}` (feature tasks; project-ensure runs first), `specflow:linear --misc` (misc rolling file), `specflow:linear --sync` (round-trip status update from Linear back to source file), `/specflow:linear` (asks).

**Inputs:**
- For feature export: `features/NNN-{slug}/{NNN-slug}-feature.md` (read first for `linear_project_id`), `features/NNN-{slug}/{NNN-slug}-prd.md` (read for the description body used when creating a new project), `features/NNN-{slug}/{NNN-slug}-tasks.md` (note the `NNN-{slug}-` prefix per the v2 file-naming convention — handoff decision #9).
- For misc export: `docs/specflow/misc-task/000-tasks-misc-tasks.md`.
- Always: `admin/config.json.linear` for team mapping. For misc tasks, targets `linear.miscProject` (auto-created if missing — check first, create if absent, store ID). For feature tasks, the project mapping lives in `{NNN-slug}-feature.md` frontmatter under `linear_project_id` (set lazily on first run; see Phase A below).

**v2 changes from v1:**
- Source path moved: `task/NNN-tasks-{slug}.md` → `features/NNN-{slug}/{NNN-slug}-tasks.md`.
- Misc rolling file path unchanged: `docs/specflow/misc-task/000-tasks-misc-tasks.md`.
- Reads `admin/config.json.linear` for project mapping.
- Auto-creates the misc Linear project if it doesn't exist.

**Task block format (per 033-human-readable-tasks v2.12.0).** Two formats may appear in the source file:

- **v2.12.0+ format** — task block starts with `**Parent PRD:**`. The Linear issue title is `T{N} — {short title}` (the H3 heading). The issue description is composed from the block in this order, with each section as a bolded subheading in Linear:
  - **Current State** (one paragraph)
  - **Expected State** (one paragraph)
  - **Technical Implementation** (one paragraph)
  - **Technical References** (bullet list)
  - **Files to Modify** / **Files to Create** (bullet lists)
  - **Layers Touched** (one line)
  - **Acceptance Criteria** (bullet list)
  - **QA Verification** (Given/When/Then bullet list)
  - **Definition of Done** (checklist)
  - **User Stories Addressed** (bullet list)
  - **Dependencies** (bullet list; render as a Linear blocking-issue relation when the dep T-ID maps to a known Linear issue, otherwise as text)
  - **Stats** line at the bottom
  - The HTML-comment `ai-metadata` footer is NOT exported (vendor-neutral; humans on Linear don't need machine metadata)
- **Pre-2.12.0 format** — task block starts with `**Anchor:**`. Issue title same. Issue description = `Anchor:` + `Scope:` + `Acceptance:` + `Notes:` as bolded paragraphs. `Depends on:` mapped to Linear blocking relations where possible. `context-budget-estimate / sprint-bucket / prior-lessons` NOT exported.

Detection: presence of `**Parent PRD:**` in the task block → new format; absence → legacy format. Mixed-format files (some tasks new, some old) are tolerated — detect per-task block.

## Phase A — Project ensure (feature mode only)

Runs before export for `specflow:linear {NNN-slug}`. Skipped entirely for `--misc` (which targets the workspace-level `linear.miscProject`) and `--sync` (which only reads).

1. **Read `{NNN-slug}-feature.md` frontmatter.** If `linear_project_id` is present and non-empty, treat it as the resolved project ID, skip to Phase B. The mapping is sticky — once set, subsequent runs append issues to that exact project.
2. **No `linear_project_id` yet** — search Linear for an existing project whose name matches `{NNN-slug}` verbatim (e.g. `002-promote-v3-home`). Match is exact, case-sensitive. If multiple projects share the name, refuse: *"Multiple Linear projects named `{NNN-slug}` exist. Resolve manually in Linear (rename or merge), then re-run."*
3. **Existing project found.** Reuse it. Do NOT touch the project description, even if it has drifted from the current PRD. Persist `linear_project_id: {id}` to `{NNN-slug}-feature.md` frontmatter and proceed to Phase B.
4. **No existing project.** Create a new Linear project with:
   - **Name:** `{NNN-slug}` (e.g. `002-promote-v3-home`).
   - **Description:** the PRD body content starting from the first line *after* the closing `---` of the PRD's YAML frontmatter through end-of-file. The first H2 in this slice is `## Vision` per the standard PRD body structure. Send as markdown with real newlines (not literal `\n`).
   - **Team:** from `admin/config.json.linear.team`.
   Persist the returned project ID to `{NNN-slug}-feature.md` frontmatter as `linear_project_id: {id}`.
5. **Failure modes.** If Linear MCP is unavailable, refuse with the canonical *"Linear MCP not detected — install MCP and re-run `/specflow:linear {NNN-slug}`."* sentinel. If the create call fails (auth / network / quota), surface the error verbatim and halt; do NOT proceed to Phase B with no project ID.

The project description is set ONLY on first create. Subsequent `specflow:linear {NNN-slug}` runs are append-only — they add issues to the resolved project but never re-author the description.

## Phase B — Sprint milestones (feature mode only)

Runs after Phase A, before issue export. Each distinct `sprint-bucket: N` in the tasks file maps 1:1 to a Linear project milestone named `Sprint N`. Skipped for `--misc` and `--sync`.

1. **Collect distinct sprint-bucket values.** Parse `{NNN-slug}-tasks.md` for every task block's `sprint-bucket` value (HTML-comment `<!-- ai-metadata: ... sprint-bucket=N ... -->` for v2.12.0+ format; visible `sprint-bucket: N` field for pre-2.12.0 format). Produce the sorted set `{1, 2, 3, …}`.
2. **List existing project milestones.** Query Linear for milestones already attached to the resolved project (Phase A).
3. **Ensure one milestone per bucket value (additive, never destructive).** For each `N` in the set: if a milestone named `Sprint N` already exists in the project, reuse it. If not, create it with name = `Sprint N`, sortOrder = `N` (so they list in dependency order), and an empty description. Existing milestones are left untouched — descriptions are never re-authored, milestones from prior runs are never deleted.
4. **Build the bucket → milestone-id map.** Record `{1: milestoneId_for_Sprint_1, 2: ..., …}` for use in Phase C.

The mapping is intentionally 1:1 and uncapped — a single bucket with N tasks becomes a single `Sprint N` milestone with N issues, regardless of size. Agent context budgeting is the consumer's problem; this skill produces the canonical sprint grouping.

## Phase C — Issue export

The existing export contract above (task block formats, Export Map, sync). Every issue created in this phase MUST be:
- Attached to the project resolved in Phase A.
- Attached to the `Sprint N` milestone resolved in Phase B, where `N` is the task's own `sprint-bucket` value.

On re-runs (new tasks added via `specflow:scope-change`), existing issues' milestone assignments are left untouched; only newly-created issues get attached to their bucket's milestone. If a new task's `sprint-bucket` value is higher than any existing milestone, Phase B's create-if-absent step will have produced the new milestone before this phase runs.

**Verify steps:**
1. For feature mode: `{NNN-slug}-feature.md` frontmatter has a non-empty `linear_project_id` after the run.
2. Every distinct `sprint-bucket: N` value in the tasks file has a corresponding `Sprint N` milestone in the Linear project.
3. Every task in the source file has a Linear ID in the Export Map after export.
4. Every Linear issue created/updated by this run is attached to (a) the resolved project, AND (b) the `Sprint N` milestone where `N = the issue's source task's sprint-bucket`.
5. Linear issues match source-file content (title, description, scope labels, MISC-NNN id when applicable).
6. Round-trip: status changes in Linear surface in the source file's status column on next sync.
7. The misc Linear project exists (auto-created if absent) and its ID is stored in `admin/config.json.linear.miscProject`.
8. For v2.12.0+ format: no HTML comments leak into the Linear issue description; every section header is rendered as bold.

**Reference:** v1 SKILL.md at `plugins/specflow/skills/linear/SKILL.md` for the existing implementation that v2 builds on; PRD Appendix B2 (Linear integration for misc).
