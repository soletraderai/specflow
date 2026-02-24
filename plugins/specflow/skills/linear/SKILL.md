---
name: "specflow:linear"
description: >
  Export an approved task review document to Linear as a project with actionable
  issues. Use this skill whenever the user says "export to Linear", "push to
  Linear", "create Linear issues", "send tasks to Linear", "prd to linear",
  or runs /specflow:linear. Also trigger when a user has just approved a task
  review from /specflow:task and wants to export it, or when they mention
  exporting tasks from a review document. This skill reads a task review file
  from docs/specflow/task/, creates a Linear project, then creates issues
  in dependency order with blockedBy relationships and updates the review
  document with an export map.
---

# PRD to Linear

Export an approved task review document to Linear as a project with properly sequenced issues.

This skill is the export half of the task planning pipeline. The `specflow:task` skill breaks a PRD into a task review document; this skill takes that review document and pushes it to Linear. The review document is the contract -- this skill reads it faithfully and creates exactly what it describes.

## Process

### 1. Locate the Task Review

Look for task review files in `docs/specflow/task/` matching the `*-tasks-*.md` pattern.

- If the user provides a filename or path directly, use that
- If multiple review files exist, list them with their PRD IDs and ask which one to export
- If only one exists, confirm it with the user

Read the full file content. Check the `status` field in the YAML frontmatter:
- `Pending Review` or `Approved` -- proceed normally
- `Exported` -- check whether an Export Map section exists in the document:
  - **Export map exists:** "Found N existing issues. Re-exporting will UPDATE existing and CREATE new tasks. Continue?"
  - **No export map but status is Exported:** "No export map found. Re-exporting will create NEW issues (duplicates may result). Continue?"
  - Only proceed if the user confirms.

### 2. Parse and Validate

Extract everything needed for the export:

**From YAML frontmatter:**
- `prd` -- PRD title (used in issue descriptions)
- `prd_id` -- PRD identifier (e.g., `PRD-001`), included in issue descriptions
- `prd_file` -- path to the parent PRD (linked in issues)
- `project` -- Linear project name
- `task_prefix` -- the 3-letter prefix (e.g., PIU)
- `total_hours` -- sum of all estimates
- `duration_weeks` -- calculated duration
- `team` -- Linear team name (from frontmatter; prompt user if missing)
- `priority` -- project-level priority
- `target_date` -- project target date

**From Export Map (if re-exporting):**
- If the document contains an `## Export Map` section, parse the table into a lookup: `{ reviewId → { linearId, linearUrl } }`
- This map determines which tasks get updated vs created during re-export

**From Quick Reference table:**
- Ordered list of task IDs, titles, hours, blocked-by references, priorities, labels

**From Task Details sections:**
For each task, extract:
- Estimate, Priority, Label, Blocked by, Layers, User stories
- Current State, Expected State, Technical Implementation
- Files to Modify, Files to Create
- Acceptance criteria
- QA Verification criteria
- Definition of Done

**Cross-validate:** Every ID in the Quick Reference table must have a matching Task Details section, and vice versa. If there's a mismatch, report it and halt.

Show the user what was parsed:

```
Found [N] tasks in [filename]:

[Quick Reference table from the document]

Total: [X]h | ~[Y] weeks at 50h/week | Target: [date]
```

### 3. Confirm Export Parameters

Before creating anything in Linear, confirm with the user:

```
Ready to export to Linear:
- Project: "[project name]"
- Team: [team]
- Priority: [priority]
- Target date: [date]
- Issues: [N] tasks, [X]h total

Create new project, or link to an existing one?
```

Allow the user to override any parameter. If they want to use an existing project, ask for the project name and use `mcp__plugin_linear_linear__get_project` to resolve it.

### 4. Create the Linear Project

Use `mcp__plugin_linear_linear__save_project` with:
- **name**: From frontmatter `project` field (or user override)
- **description**: "Implementation of [prd]. See `[prd_file]` for full requirements."
- **team**: From frontmatter `team` field
- **priority**: Map from string to number (see priority mapping below)
- **targetDate**: From frontmatter `target_date` in ISO format
- **state**: "planned"

Skip this step if the user chose an existing project.

After creation, confirm the project with the user.

**Priority mapping (project level):**

| Frontmatter | Linear value |
|-------------|-------------|
| Urgent      | 1           |
| High        | 2           |
| Medium      | 3           |
| Low         | 4           |

### 5. Create or Update Issues in Dependency Order

This is the core of the export. Issues must be processed in dependency order so that blocker issues exist before the issues they block -- otherwise the `blockedBy` field can't reference real Linear identifiers.

**Dependency ordering algorithm:**

1. Parse each task's `Blocked by` field. `--` means no blockers. Otherwise split on `, ` to get a list of review IDs.
2. Build a map of each task's unresolved blocker count (in-degree).
3. Start with all tasks that have zero blockers (in-degree 0).
4. Process one task at a time:
   a. Create or update it in Linear (see below).
   b. Store its Linear identifier in the export map.
   c. For every task that lists it as a blocker, decrement that task's in-degree.
   d. If any task's in-degree reaches 0, it becomes ready to process.
5. If tasks remain unprocessed after the queue empties, there's a circular dependency -- report it and halt.

**First export (no existing export map):**

For each task, use `mcp__plugin_linear_linear__create_issue` with:

- **title**: Task title from the review document
- **team**: From frontmatter `team` field
- **project**: Project name from step 4
- **description**: Populated from the issue template below
- **priority**: Per-task priority mapped to number (High=2, Normal=3, Low=4)
- **labels**: `[label]` from the task (typically "Feature")
- **estimate**: Hour estimate as a number
- **state**: "Todo" if the task has no blockers; "Backlog" if it has any blockers
- **blockedBy**: Array of Linear issue identifiers, mapped from review IDs using the export map built so far

**Re-export (export map exists):**

For each task in the review, check the export map:
- **Found in map** → use `mcp__plugin_linear_linear__update_issue` with: `id` (Linear ID from map), `title`, `description`, `priority`, `estimate`, `state`, `blockedBy`, `labels`, `project`
- **Not in map** → use `mcp__plugin_linear_linear__create_issue` (same params as first export)

After processing all tasks, check for **orphaned tasks** -- entries in the export map whose review IDs no longer appear in the review document. Warn the user about these but do NOT auto-delete them from Linear:

```
Warning: [N] tasks were removed from the review but still exist in Linear:
- [LinearID]: [Title]
Consider manually closing or cancelling these issues.
```

**Partial failure recovery:** After EACH successful create or update, immediately save the current export map to the review document. This makes the export idempotent -- if the process fails partway through, re-running it will update already-created issues and create only the remaining ones.

**Progress reporting:**

After each issue, report:

```
[Created|Updated] [LinearID]: [Title] ([Xh], [status])
```

Keep a running export map: `{ reviewId -> { linearId, linearUrl } }`

<issue-template>
## Parent PRD

[PRD title] (PRD-NNN) -- see `[prd_file]`

## Current State

[Current State section from the task review, verbatim]

## Expected State

[Expected State section from the task review, verbatim]

## Technical Implementation

[Technical Implementation section from the task review, verbatim]

## Files to Modify

[Files to Modify section from the task review, verbatim]

## Files to Create

[Files to Create section from the task review, verbatim]

## Layers Touched

[Layers field from the task, formatted as a bullet list:]
- API
- UI
- Tests

## Acceptance Criteria

[Acceptance criteria from the task review, verbatim]

## QA Verification

[QA Verification from the task review, verbatim]

## Definition of Done

[Definition of Done from the task review, verbatim]

## Estimate

**[X] hours** of development effort

## User Stories Addressed

[User stories field from the task, formatted as a bullet list:]
- User story 1
- User story 4
- User story 5
</issue-template>

### 6. Update the Review Document

**First export:**

After all issues are created successfully:

1. Change `status: Pending Review` to `status: Exported` in the YAML frontmatter
2. Add `exported_at: YYYY-MM-DD` (today's date) to the frontmatter
3. Insert an `## Export Map` section immediately after the frontmatter, before the first heading:

```markdown
## Export Map

| Review ID | Linear ID | Linear URL |
|-----------|-----------|------------|
| PIU-001   | CXP-42    | [link]     |
| PIU-002   | CXP-43    | [link]     |
```

This preserves full traceability between the review document and the Linear issues.

**Re-export:**

The export map was already saved incrementally during Step 5 (partial failure recovery). Do a final verification pass:

1. Update `exported_at` to today's date
2. Ensure the export map is complete and matches all tasks in the review
3. Keep removed tasks in the export map annotated as `(removed from review)` for traceability:

```markdown
| PIU-003   | CXP-44    | [link] (removed from review) |
```

### 7. Print Summary

After everything is complete, print a summary.

**First export:**

```
Project: [name]
Total: [sum] hours | Duration: ~[X] weeks at 50h/week | Target: [date]

| Linear ID | Title | Estimate | Blocked By | Priority | Status |
|-----------|-------|----------|------------|----------|--------|
| CXP-42 | [title] | 12h | None | High | Todo |
| CXP-43 | [title] | 6h | CXP-42 | Normal | Backlog |

Critical path: CXP-42 -> CXP-44 -> CXP-45 ([X] hours)
Parallel work available after CXP-42: CXP-43 and CXP-44 can run simultaneously

Review document updated: docs/specflow/task/[filename].md

Note: Each issue includes QA Verification scenarios and a Definition of Done checklist.
Consider adding a "QA Review" status to your Linear workflow (between "In Review" and "Done")
to enforce QA sign-off before tasks are marked complete.
```

**Re-export:**

```
Re-export complete: [N] updated | [M] created | [K] orphaned (removed from review)

| Linear ID | Title | Action | Estimate | Status |
|-----------|-------|--------|----------|--------|
| CXP-42 | [title] | Updated | 12h | Todo |
| CXP-45 | [title] | Created | 4h | Backlog |

Orphaned issues (removed from review — clean up manually):
- CXP-44: [title]

Note: Re-export overwrites any manual changes made to issues in Linear since the last export.

Review document updated: docs/specflow/task/[filename].md
```

Note which issues can be worked on in parallel and identify the critical path (longest dependency chain in hours).

## Important Notes

- All issues go to the team specified in the frontmatter (prompt the user if missing)
- Team capacity is **50 hours per week** -- used for all timeline references
- Dependencies use Linear's native `blockedBy`/`blocks` relationships so they appear in the issue dependency graph
- Every issue is linked to the project so they appear in the project view
- Issues with unresolved blockers get status **"Backlog"**; unblocked issues get **"Todo"**
- Hour estimates go in the `estimate` field on each issue
- Both "Medium" and "Normal" map to priority 3 -- the frontmatter may use "Medium" while task details use "Normal"
- The issue template content is taken verbatim from the review document -- do not rewrite or summarize it during export
- Do **NOT** modify the parent PRD document
- If any issue creation fails, report which issues succeeded and which failed, and offer to retry the failures
- Consider adding a **"QA Review"** status to your Linear team's workflow (between "In Review" and "Done") so that issues move through a dedicated QA gate before being marked complete. This is optional but recommended for teams that want to enforce QA sign-off in Linear itself.
