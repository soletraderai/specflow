---
name: "specflow:task"
description: >
  Break a PRD into actionable Linear issues organized as vertical slices with
  dependency tracking, time estimates, and project linking. Use this skill
  whenever the user says "prd to tasks", "break this PRD into tasks",
  "create issues from PRD", "turn this into Linear issues", or runs
  /specflow:task. Also trigger when a user has just finished creating a PRD
  and wants to move to implementation planning. This skill creates a Linear
  project, estimates effort, and generates properly sequenced issues with
  blockedBy relationships so the team knows exactly what to build and in
  what order.
---

# PRD to Linear Tasks

Break a PRD into independently-grabbable Linear issues using vertical slices (tracer bullets), with time estimates and dependency tracking.

The philosophy here is tracer bullets -- thin vertical slices that cut through every integration layer end-to-end. Each issue should be demoable on its own. This is the opposite of horizontal slicing (one issue for DB, one for API, one for UI). Vertical slices reduce integration risk and give the team working software at every step.

## Process

### 1. Locate the PRD

Look for PRDs in `docs/specflow/prd/`. If multiple PRDs exist, ask the user which one to use. If only one exists, confirm it. Read the full PRD content and internalize every section -- the user stories, implementation decisions, and scope boundaries all inform how to slice the work.

If the user provides a PRD filename or path directly, use that.

### 2. Gather Project Parameters

Before exploring the codebase, confirm minimal project settings with the user:

**Priority level:**
- Ask: "What priority should the project have?" (Urgent / High / Medium / Low)
- Default suggestion: Medium (priority 3) unless the PRD suggests otherwise

**Team assignment:**
- Ask: "Which Linear team should these issues go to?"
- If the user doesn't specify, prompt them to provide a team name

**Timeline is NOT asked upfront.** The timeline is derived from the task estimates. After all tasks are estimated, calculate:
- `total_hours = sum of all task estimates`
- `duration_weeks = ceil(total_hours / 50)` (team capacity is 50 hours per week)
- `target_date = today + duration_weeks`

This means the estimates drive the timeline, not the other way around. The 50 hours/week is a conversion factor for translating hours into calendar time.

### 3. Explore the Codebase

Read the key modules and integration layers referenced in the PRD. Identify:

- The distinct integration layers the feature touches (e.g. DB/schema, API/backend, UI, tests, config)
- Existing patterns for similar features in the codebase
- Natural seams where work can be parallelized
- Technical risks or unknowns that might need spike issues
- **Specific file paths** that will need modification for each piece of work
- **New files** that will need to be created, following existing naming and directory conventions

This grounds the breakdown in reality rather than abstract planning. Having concrete file paths lets the developer jump straight into the relevant code.

#### Agent-Assisted Analysis (if available)

After the initial exploration above, check for specialist agents:

1. Read `docs/specflow/config.json` (if it exists)
2. If agents are configured (`agents.roles` exists), spawn relevant agents via the Task tool in parallel (max 3):
   - Use the `roles.architectureReview` agent to identify integration points and architectural patterns that inform slice boundaries
   - Use the appropriate tech-stack agent (from `agents.techStack`) for framework-specific implementation insights
3. Agent findings may reveal: additional integration points not obvious from manual exploration, technical risks warranting spike issues, better approaches based on framework-specific knowledge
4. If `docs/specflow/config.json` doesn't exist or has no agents configured, continue normally — this is purely additive

### 4. Draft Vertical Slices

Break the PRD into tracer bullet issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- The first slice should be the simplest possible end-to-end path (the "hello world" tracer bullet)
- Later slices add breadth: edge cases, additional user stories, polish
- Only include tasks that directly implement the PRD -- no generic infrastructure, no "nice to haves" that aren't in the PRD, no tasks for things that already exist
- Each slice must document the **current state** (what exists now) and **expected state** (what should exist after) in plain, behavior-focused language -- describe what the user sees and experiences, not code internals. A senior developer will oversee implementation, so these sections orient them on the "what" and "why", not the "how". Save file paths, interface names, and code-level details for the **Technical Implementation** and **Files to Modify/Create** sections.
- The **technical implementation** section is advisory guidance only -- a senior developer may choose a different approach
- Each slice must list **files to modify** and **files to create** so the developer can immediately orient themselves in the codebase
- Each slice must include **QA Verification** criteria -- Given/When/Then scenarios derived from the PRD's verification scenarios that a human QA tester can execute to verify the slice is complete
</vertical-slice-rules>

**Estimating each slice:**

For each vertical slice, estimate effort in hours:
- Trivial (config change, copy update): 1-2 hours
- Small (single component, straightforward logic): 3-5 hours
- Medium (multiple components, some integration): 6-8 hours
- **Maximum 8 hours per task.** Any slice over 8h must be split. 8h = one day of work.

The sum of all estimates determines the project duration. After estimating all slices:
- `total_hours = sum of all estimates`
- `duration_weeks = ceil(total_hours / 50)`
- Present this as: "[total_hours]h total = ~[duration_weeks] weeks at 50h/week"

### 5. Generate Review Document

Before discussing anything with the user, save a structured review document to `docs/specflow/task/`. This document is the single source of truth for the proposed breakdown. The user will review it, call out task IDs to remove or adjust, and only approved tasks get exported to Linear.

**Filename:** `task-review-[prd-kebab-name].md` (e.g., `task-review-contact-form-profile-collection.md`)

**Create the `docs/specflow/task/` directory** if it doesn't exist.

The document must follow this exact format:

```markdown
---
prd: "[PRD Title]"
prd_file: "docs/specflow/prd/[filename]"
project: "[Project Name]"
task_prefix: "[ABC]"
total_hours: [sum]
duration_weeks: [ceil(sum / 50)]
team: [team name from user]
priority: [level]
target_date: YYYY-MM-DD
status: Pending Review
---

# Task Review: [PRD Title]

## Quick Reference

| ID | Title | Hours | Blocked By | Priority | Label |
|----|-------|-------|------------|----------|-------|
| [PRE]-001 | [title] | [X]h | -- | High | Feature |
| [PRE]-002 | [title] | [X]h | [PRE]-001 | Normal | Feature |
| [PRE]-003 | [title] | [X]h | [PRE]-001 | Normal | Feature |
| [PRE]-004 | [title] | [X]h | [PRE]-002, [PRE]-003 | Normal | Feature |

**Total: [N] tasks | [sum]h estimated | ~[duration] weeks at 50h/week**
**Target date: [calculated date]**
**Critical path: [PRE]-001 -> [PRE]-003 -> [PRE]-004 ([X]h)**

---

## Task Details

### [PRE]-001: [Title]

- **Estimate:** [X] hours
- **Priority:** High
- **Label:** Feature
- **Blocked by:** --
- **Layers:** [API, UI, Tests]
- **User stories:** 1, 3, 5

**Current State:**
[Describe what the user or system currently experiences in plain language. Focus on behavior, not code. What does someone see or encounter today? If nothing exists yet, state "No equivalent functionality exists." Do NOT reference file paths, interface names, or line numbers here -- those belong in Technical Implementation and Files to Modify.]

**Expected State:**
[Describe what the user or system will experience after this task is complete. Focus on the observable outcome -- what changes, what's new, what's different from a user/behavior perspective. Be specific but non-technical. Save implementation details for the sections below.]

**Technical Implementation:**
[2-3 sentence guidance on how this slice could be delivered end-to-end. This is advisory -- the developer may choose a different approach.]

**Files to Modify:**
- `path/to/existing-file.ts` — [brief reason for modification]
- `path/to/another-file.tsx` — [brief reason for modification]

**Files to Create:**
- `path/to/new-file.ts` — [what this file will contain]
- _(None if no new files needed)_

**Acceptance criteria:**
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

**QA Verification:**
- [ ] Given [precondition], When [action], Then [expected outcome]
- [ ] Given [precondition], When [action], Then [expected outcome]

[Behavioral Given/When/Then scenarios derived from the PRD's verification scenarios. These are what a human QA tester will execute to verify this slice is complete. Each scenario must be independently testable.]

**Definition of Done:**
- [ ] Code implements all acceptance criteria
- [ ] Code reviewed
- [ ] QA verification scenarios manually tested and passed
- [ ] Issues found during QA resolved
- [ ] Task is demo-ready

---

### [PRE]-002: [Title]

[Same format for each task...]

---

## Phase 2 (Deferred)

[If any slices were cut to fit the budget, list them here with brief descriptions so they aren't lost. If nothing was deferred, omit this section.]
```

**ID format rules:**
- Each PRD gets a unique 3-letter prefix derived from its title. Choose letters that are memorable and clearly reference the feature (e.g., "Profile Image Upload" -> `PIU`, "User Authentication" -> `UAT`, "Dashboard Analytics" -> `DAN`). Record this prefix in the YAML frontmatter as `task_prefix`.
- Format: `{PREFIX}-` followed by 3-digit zero-padded numbers: `PIU-001`, `PIU-002`, etc.
- This prevents confusion when multiple PRDs are in flight -- every task ID is immediately traceable to its parent PRD.
- Number tasks in dependency order -- blockers get lower numbers
- These IDs are temporary for the review process only. They get replaced by real Linear identifiers during export.

**The Quick Reference table is the most important part.** It gives the user a scannable overview where they can quickly spot tasks that don't belong. Every task in the document must appear in this table, and every row in the table must have a corresponding detail section below.

### 6. Critical Review (Before Presenting)

Before presenting to the user, re-read the saved review document and apply a structured review. This catches issues before the user sees the document.

#### Built-in Task Review Framework

Apply each lens systematically:

**Slice quality:**
- Is each slice truly vertical (all layers end-to-end)? Any horizontal slices hiding (e.g., "set up database schema" without corresponding API/UI)?
- Is each slice independently demo-able or verifiable?

**Estimate accuracy:**
- Do estimates align with the complexity described?
- Are any tasks over 8 hours? (must split)
- Are dependencies captured correctly in the `Blocked by` column?

**Coverage:**
- Do all tasks trace back to specific PRD user stories?
- Are there PRD requirements with no corresponding tasks?
- Are edge cases and error handling represented?
- Does every task have QA Verification criteria derived from the PRD's verification scenarios?

**Dependency graph:**
- Any circular dependencies?
- Is the critical path realistic?
- Any unnecessary blocking relationships that reduce parallelism?

Produce structured findings:
- **Critical Issues** — must fix (horizontal slices, >8h tasks, missing PRD coverage, circular deps)
- **Warnings** — should fix (loose estimates, unnecessary blocking)
- **Observations** — worth noting

#### Agent-Assisted Review (if available)

1. Read `docs/specflow/config.json` (if it exists)
2. If review agents are configured, spawn for supplementary perspectives via the Task tool:
   - Use `roles.architectureReview` agent to review the task breakdown from an architecture perspective
3. Integrate findings — these supplement the built-in framework but do not replace it

#### Apply Corrections

Address all critical issues found:
- Fix horizontal slices by restructuring into vertical ones
- Split any tasks over 8 hours
- Add missing tasks for uncovered PRD requirements
- Correct dependency issues
- Update the saved review document with all corrections (rewrite the file)

Then proceed to present the corrected document.

### 7. Present the Review Document

After saving the file, tell the user:

```
Task review saved to: docs/specflow/task/[filename].md

[N] tasks | [sum]h estimated | ~[X] weeks at 50h/week

Quick reference:
[paste the Quick Reference table here]

Review the document and let me know:
- Any task IDs to REMOVE (e.g., "remove PIU-003, PIU-007")
- Any tasks to ADJUST (e.g., "PIU-002 should be 4h not 8h")
- Any tasks to SPLIT or MERGE
- Any missing tasks to ADD
- Or "approved" to export to Linear
```

**How to handle user feedback:**

The user may respond in several ways:

- **"Remove PIU-003, PIU-007"** -- Delete those tasks from the document, renumber remaining tasks sequentially ([PRE]-001, [PRE]-002, [PRE]-003...), update all `Blocked by` references to use the new IDs, recalculate totals. Save the updated document and present the new Quick Reference table.

- **"PIU-002 should be 4h"** -- Update the estimate in both the table and the detail section. Recalculate totals. Save and show updated table.

- **"Split PIU-004 into two tasks"** -- Ask what the split should be, create two new tasks, remove the original, renumber, save and show.

- **"Add a task for X"** -- Add it with the next available ID, place it in the correct dependency position, save and show.

- **"Approved"** or **"Looks good"** -- Proceed to step 8.

**Keep iterating** until the user explicitly approves. Each revision updates the saved document so it always reflects the current state.

### 8. Create the Linear Project

Only proceed here after the user has approved the review document.

Create a new Linear project using the Linear MCP:

Use `mcp__plugin_linear_linear__save_project` with:
- **name**: The PRD title (this becomes the project name)
- **description**: A summary of the PRD (2-3 sentences covering problem and solution)
- **team**: The team specified during setup
- **priority**: The priority level agreed with the user (0=None, 1=Urgent, 2=High, 3=Medium, 4=Low)
- **targetDate**: The calculated target date (ISO format)
- **state**: "planned"

If the project already exists (user set it up manually), ask for the project name and use that instead.

After creation, confirm the project URL with the user.

### 9. Create Milestones (Optional)

If the project naturally breaks into phases (e.g., "Foundation", "Core Features", "Polish"), create milestones:

Use `mcp__plugin_linear_linear__create_milestone` for each phase with:
- **project**: The project name/ID from step 8
- **name**: Phase name
- **targetDate**: Calculated based on the phase's proportion of total hours

Only create milestones if there are clear phases. Don't force phases where they don't exist naturally.

### 10. Create Linear Issues

Create issues in dependency order (blockers first) so you can reference real issue identifiers in the `blockedBy` field.

For each approved task from the review document, use `mcp__plugin_linear_linear__create_issue` with:

- **title**: The task title from the review document
- **team**: The team specified during setup
- **project**: The project name from step 8
- **description**: Use the issue template below, populated from the review document's detail section
- **priority**: The priority level for this specific issue (1=Urgent, 2=High, 3=Normal, 4=Low)
- **labels**: The label from the review document -- typically ["Feature"] for new work
- **estimate**: The hour estimate from the review document
- **state**: "Backlog" for blocked issues, "Todo" for ready-to-start issues
- **blockedBy**: Array of issue identifiers for any blocking issues (use the identifiers returned from previous create calls, mapped from the [PRE]-XXX references in the review document)
- **blocks**: Array of issue identifiers this issue blocks (if creating out of order)
- **milestone**: The milestone name if applicable

**Wait for each issue to be created before creating the next one** so you have real issue identifiers for the `blockedBy` field. Keep a mapping of `[PRE]-XXX -> Linear issue ID` as you go.

<issue-template>
## Parent PRD

[PRD title and link to the PRD file in the repo: `docs/specflow/prd/[filename]`]

## Current State

[What the user or system currently experiences, in plain behavior-focused language. What does someone see or encounter today? Do not reference file paths or code here -- keep it focused on the experience and behavior. If nothing exists yet, state "No equivalent functionality exists."]

## Expected State

[What the user or system will experience after this task is complete. Describe the observable outcome from a user/behavior perspective -- what changes, what's new, what's different. This is what "done" looks like. Keep it non-technical; implementation details belong in Technical Implementation below.]

## Technical Implementation

[Advisory guidance on how this could be implemented. The developer may choose a different approach based on their assessment.]

## Files to Modify

[List each existing file that will likely need changes:]
- `path/to/file.ts` — [brief reason]
- `path/to/file.tsx` — [brief reason]

## Files to Create

[List any new files that need to be created:]
- `path/to/new-file.ts` — [what this file will contain]
- _(None if no new files needed)_

## Layers Touched

[From the review document:]
- Database/Schema
- API endpoint
- UI component
- Tests
- Configuration

## Acceptance Criteria

[From the review document:]
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## QA Verification

[From the review document:]
- [ ] Given [precondition], When [action], Then [expected outcome]
- [ ] Given [precondition], When [action], Then [expected outcome]

## Definition of Done

- [ ] Code implements all acceptance criteria
- [ ] Code reviewed
- [ ] QA verification scenarios manually tested and passed
- [ ] Issues found during QA resolved
- [ ] Task is demo-ready

## Estimate

**[X] hours** of development effort

## User Stories Addressed

[From the review document:]
- User story 3
- User story 7
</issue-template>

### 11. Update Review Document

After all issues are created, update the review document:

- Change `status: Pending Review` to `status: Exported`
- Add a new section at the top (after frontmatter) mapping task IDs to real Linear identifiers:

```markdown
## Export Map

| Review ID | Linear ID | Linear URL |
|-----------|-----------|------------|
| [PRE]-001 | CXP-42 | [link] |
| [PRE]-002 | CXP-43 | [link] |
```

This preserves traceability between the review and what actually got created.

### 12. Summary

After creating all issues, print a summary table:

```
Project: [name] ([Linear project URL])
Total: [sum] hours | Duration: ~[X] weeks at 50h/week | Target: [date]

| Linear ID | Title | Estimate | Blocked By | Priority | Status |
|-----------|-------|----------|------------|----------|--------|
| CXP-42 | Basic widget creation | 8h | None | High | Todo |
| CXP-43 | Widget listing view | 6h | CXP-42 | Normal | Backlog |
| CXP-44 | Widget editing | 12h | CXP-42 | Normal | Backlog |
| CXP-45 | Error handling | 5h | CXP-43, CXP-44 | Normal | Backlog |

Critical path: CXP-42 -> CXP-44 -> CXP-45 (25 hours)
Parallel work available after CXP-42: CXP-43 and CXP-44 can run simultaneously
```

Also note:
- Which issues can be worked on in parallel
- The critical path (longest dependency chain)

Do NOT modify the parent PRD document.

End with: "Run `/specflow:linear` to export this review to Linear." (if the user chose not to export during this session)

## Important Notes

- All issues go to the team specified during setup
- The team capacity is **50 hours per week** -- use this for all timeline calculations
- Dependencies use Linear's native `blockedBy`/`blocks` relationships so they show in the issue graph
- Every issue must be linked to the project so they appear in the project view
- Issues that have unresolved blockers get status "Backlog"; unblocked issues get "Todo"
- Hour estimates go in the `estimate` field on each issue
- If the user has already created the project manually, just ask for the project name and link issues to it
- **Nothing goes to Linear until the user explicitly approves the review document.** The review step is mandatory, not optional.
- Keep tasks focused on what the PRD actually requires. Don't generate tasks for generic infrastructure, boilerplate setup, or things that already exist in the codebase. Every task should trace back to a specific user story or implementation decision in the PRD.
