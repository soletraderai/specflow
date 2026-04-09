---
name: "specflow:test"
description: >
  Verify completed Linear tasks against their acceptance criteria, QA scenarios,
  and definition of done using code review and Playwright browser testing.
  Use this skill whenever the user says "verify tasks", "QA check",
  "test completed tasks", "check done tasks", "verify done issues",
  "run QA", or runs /specflow:test. Also trigger when a user has just
  marked tasks as Done in Linear and wants to verify they actually meet
  the acceptance criteria from the task review. This skill reads completed
  Linear issues, traces them back to their task review documents, runs
  multi-layer verification (code review, schema checks, Playwright screenshots),
  and posts a structured QA report to Linear.
---

# QA Verification

Verify completed Linear tasks against their acceptance criteria, QA scenarios, and definition of done. This skill closes the specflow pipeline loop — after `specflow:prd` creates requirements, `specflow:task` breaks them into issues, `specflow:linear` exports to Linear, and developers build the work, `specflow:test` checks that the completed work actually meets what was specified.

```
specflow:prd → specflow:task → specflow:linear → [dev builds] → [marks Done] → specflow:test
```

## Process

### Phase 0: System Readiness Check

Before doing anything else, check all prerequisites and present a clear status summary. This runs once at the start — not per-task.

#### 0a. Check All Prerequisites

Run these checks in parallel:

1. **Linear MCP** — call `mcp__plugin_linear_linear__list_teams` to verify the Linear MCP is accessible. If this fails, halt: "Linear MCP is not configured. Run `/specflow:setup` to set it up."
2. **Specflow config** — read `docs/specflow/config.json` for `linear.team`. Note whether it exists.
3. **Task review files** — glob for `docs/specflow/task/*-tasks-*.md`. If none exist, halt: "No task review files found. Run `/specflow:task` first."
4. **Export maps** — check if any task review files contain an `## Export Map` section. Note the count.
5. **Playwright** — run `npx playwright --version 2>/dev/null` to check if Playwright is installed. Note version or "not installed".
6. **Environment variables** — read the `.env` file directly and parse `KEY=VALUE` lines. Check for `SPECFLOW_TEST_BASE_URL`, `SPECFLOW_TEST_EMAIL`, `SPECFLOW_TEST_PASSWORD`. Note which are present and which are missing.
7. **Pages config** — check if `docs/specflow/pages.json` exists. Note present or missing.

#### 0b. Present Readiness Summary

Present the results as a status table before proceeding:

```
System Readiness Check:

| Requirement                  | Status                          |
|------------------------------|---------------------------------|
| Linear MCP                   | Connected                       |
| Linear team                  | Engineering (from config.json)  |
| Task review files            | 3 found (2 with export maps)    |
| Playwright                   | Installed (v1.59.1)             |
| SPECFLOW_TEST_BASE_URL       | Set (http://localhost:3000)     |
| SPECFLOW_TEST_EMAIL          | Set                             |
| SPECFLOW_TEST_PASSWORD       | Set                             |
| pages.json                   | Present (4 pages configured)    |

Verification modes available:
- Code review: YES
- Visual verification (Playwright): YES
```

If any Playwright prerequisites are missing, clearly state the impact:

```
| SPECFLOW_TEST_BASE_URL       | MISSING                         |
| SPECFLOW_TEST_EMAIL          | MISSING                         |
| SPECFLOW_TEST_PASSWORD       | MISSING                         |
| pages.json                   | MISSING                         |

Verification modes available:
- Code review: YES
- Visual verification (Playwright): NO — missing .env credentials and pages.json

To enable visual verification, add to .env:
  SPECFLOW_TEST_BASE_URL=http://localhost:3000
  SPECFLOW_TEST_EMAIL=your-test-email
  SPECFLOW_TEST_PASSWORD=your-test-password
And create docs/specflow/pages.json (see Phase 2d for schema).
```

#### 0c. Confirm or Abort

After presenting the readiness summary, ask:

- If everything is available: "All systems ready. Proceed with verification?"
- If Playwright prerequisites are missing: "Visual verification is unavailable — verification will use code review only. Proceed, or set up the missing prerequisites first?"
- If critical prerequisites are missing (no Linear, no task reviews): halt with a clear message about what needs to be done first.

Wait for the user to confirm before proceeding to Phase 1.

---

### Phase 1: Task Selection

Identify completed Linear tasks that need verification.

#### 1a. Load Configuration

Use the team name already resolved in Phase 0. If `config.json` didn't have `linear.team`, ask the user: "Which Linear team should I check for completed tasks?"

#### 1b. Fetch Done Tasks from Linear

1. Call `mcp__plugin_linear_linear__list_issue_statuses` with the team name to get all workflow states
2. Find the state whose name is "Done" (case-insensitive) and note its ID
3. Call `mcp__plugin_linear_linear__list_issues` filtered by the team name
4. Client-side filter the results to keep only issues whose state matches the Done state ID

#### 1c. Build Reverse Lookup from Export Maps

Build a mapping from Linear issue identifiers back to task review documents and review IDs:

1. Glob for `docs/specflow/task/*-tasks-*.md`
2. For each task review file found:
   a. Read the file content
   b. Look for an `## Export Map` section
   c. Parse the Export Map table — each row has columns: `Review ID`, `Linear ID`, `Linear URL`
   d. For each row, store in a reverse lookup: `{ Linear ID → { review_file: path, review_id: Review ID } }`
3. Also read the YAML frontmatter of each task review to capture `prd_file` for later use

This reverse lookup is the bridge between Linear and the structured QA criteria in the task review documents.

#### 1d. Check Existing Verification Logs

1. Glob for `docs/specflow/test/*-test-*.md`
2. For each verification log found:
   a. Parse the verification table rows
   b. Extract: Linear ID, Verdict, Verified At timestamp
3. Build a verification status map: `{ Linear ID → { verdict, verified_at } }`

#### 1e. Filter and Present Tasks

Cross-reference the Done tasks against the reverse lookup and verification logs:

- **Skip:** Tasks with verdict PASS and a `verified_at` timestamp — already verified
- **Re-verify:** Tasks with verdict FAIL in the log — show as "previously failed — re-verify?"
- **Unverified:** Tasks in the Done state with no verification log entry
- **Limited verification:** Tasks in the Done state that have NO entry in any Export Map — flag as "no structured QA criteria found" (these can still be verified using the Linear issue body, but with reduced confidence)

Present a selection table to the user:

```
Done tasks available for verification:

| # | Linear ID | Title | Project | Status |
|---|-----------|-------|---------|--------|
| 1 | CXP-42 | Widget creation API | Widget MVP | Unverified |
| 2 | CXP-43 | Widget listing UI | Widget MVP | Previously failed |
| 3 | CXP-45 | User settings page | Settings | Limited (no export map) |

Enter "all" to verify all, or specific numbers/IDs (e.g., "1, 3" or "CXP-42, CXP-45")
```

Wait for the user to select tasks before proceeding.

#### 1f. Fallback for Tasks Without Export Map

For tasks flagged as "Limited verification":

1. Call `mcp__plugin_linear_linear__get_issue` to fetch the full issue body
2. Extract any structured criteria from the issue description (acceptance criteria checkboxes, QA verification scenarios, definition of done)
3. If structured criteria are found, proceed with verification using those
4. If no structured criteria exist, flag: "Limited verification — no structured QA criteria found. Verification will be based on issue title and description only."

---

### Phase 2: Task Context Loading

For each selected task, load the full context needed for verification.

#### 2a. Load Linear Issue Details

Call `mcp__plugin_linear_linear__get_issue` with the Linear issue identifier to get:
- Title, description, state, labels, priority
- Any comments or attachments already on the issue

#### 2b. Load Task Review Details

Using the reverse lookup from Phase 1c, locate the task in its review document:

1. Read the task review file at the path from the reverse lookup
2. Find the task detail section matching the review ID (e.g., `### PIU-003: [Title]`)
3. Extract all structured fields:
   - **Acceptance criteria** — the checkbox list under `**Acceptance criteria:**`
   - **QA Verification** — the Given/When/Then scenarios under `**QA Verification:**`
   - **Files to Modify** — listed under `**Files to Modify:**`
   - **Files to Create** — listed under `**Files to Create:**`
   - **Layers** — from the `**Layers:**` field (e.g., `[API, UI, Tests]`)
   - **Current State / Expected State** — behavioral descriptions
   - **Definition of Done** — checkbox list under `**Definition of Done:**`
   - **Technical Implementation** — advisory guidance

#### 2c. Load Parent PRD Context

1. Read the task review file's YAML frontmatter for the `prd_file` path
2. Read the parent PRD to get broader context: user stories, implementation decisions, scope boundaries
3. This context helps interpret ambiguous acceptance criteria

#### 2d. Resolve Frontend Verification Mode

Use the results from Phase 0's readiness check — do NOT re-read `.env` or re-check Playwright.

1. If Phase 0 confirmed Playwright is available (all prerequisites met):
   - Read `docs/specflow/pages.json` and find the page entry matching this task's feature area (match by layer name, component path, or route keywords)
   - If a matching page entry is found: Playwright verification is available for this task
   - If no matching page entry: fall back to code review only for this task (note: "No pages.json entry matches this feature area")
2. If Phase 0 showed Playwright as unavailable: code review only — this was already communicated to the user

**`pages.json` expected structure** (referenced by Phase 0 and used here):

```json
{
  "auth": {
    "loginPath": "/login",
    "emailSelector": "#email",
    "passwordSelector": "#password",
    "submitSelector": "button[type=submit]"
  },
  "pages": {
    "feature-area-name": {
      "path": "/dashboard/widgets",
      "description": "Widget management page"
    }
  }
}
```

**`.env` variables** (checked once in Phase 0, not per-task):
- `SPECFLOW_TEST_BASE_URL` — the base URL for Playwright testing (e.g., `http://localhost:3000`)
- `SPECFLOW_TEST_EMAIL` — login email for authenticated pages
- `SPECFLOW_TEST_PASSWORD` — login password for authenticated pages

#### 2e. Present Verification Plan

Before executing any verification, present the plan to the user:

```
Verification plan for CXP-42: Widget creation API

Layers to verify: API, Database/Schema, Tests
Acceptance criteria: 5 items
QA scenarios: 3 items
Files to check: 7 (4 modified, 3 created)
Frontend visual check: [Yes — Playwright available | No — missing credentials/pages.json | N/A — no UI layer]

Proceed with verification? (yes/no)
```

The user must confirm before execution begins. In batch mode, show all plans first and get a single confirmation.

---

### Phase 3: Multi-Layer Verification

Execute verification across each layer the task touches. Each sub-phase produces evidence: code snippets, schema excerpts, or screenshots with pass/fail per criterion.

#### 3a. Database Layer

**Trigger:** Layers includes "Database/Schema" OR any file in Files to Modify/Create references `prisma/`

Steps:
1. Read `prisma/schema.prisma`
2. For each acceptance criterion related to database/schema:
   - Verify the model exists with the correct name
   - Verify fields exist with correct types, defaults, and constraints
   - Verify relations are correctly defined (foreign keys, relation types)
   - Verify indexes and unique constraints match requirements
3. If the task mentions migrations, check for migration files in `prisma/migrations/` — verify a migration exists that corresponds to the schema changes
4. Record evidence: schema snippets showing the relevant models/fields, with PASS/FAIL per criterion

#### 3b. Backend Layer

**Trigger:** Layers includes "API" OR files reference backend paths (e.g., `src/server/`, `src/api/`, `apps/api/`, `packages/api/`)

Steps:
1. Read each file listed in Files to Modify and Files to Create
2. For each acceptance criterion related to backend behavior:
   - Verify the endpoint/handler/service exists and is wired up
   - Verify the implementation matches the Expected State description
   - Check that input validation, error handling, and response shapes match criteria
   - Verify DTOs, API client types, or tRPC routers if mentioned in the task
3. Check architectural conventions from `config.json` constitution (if it exists):
   - Does the implementation follow stated coding principles?
   - Are architectural boundaries respected?
4. Record evidence: code snippets showing relevant implementations, with PASS/FAIL per criterion

#### 3c. Frontend Layer

**Trigger:** Layers includes "UI" OR files reference frontend paths (e.g., `src/components/`, `src/app/`, `apps/web/`, `src/pages/`)

##### Step 1: Code Review

1. Read each frontend file listed in Files to Modify and Files to Create
2. Verify:
   - Component logic, props, and state management match acceptance criteria
   - UI renders the expected elements described in Expected State
   - Event handlers and user interactions are wired correctly
   - Styling/layout matches requirements (if specified)
3. Record evidence: code snippets with PASS/FAIL per criterion

##### Step 2: Playwright Visual Verification

**Prerequisites — ALL must be true to proceed:**
- `.env` has `SPECFLOW_TEST_BASE_URL`, `SPECFLOW_TEST_EMAIL`, and `SPECFLOW_TEST_PASSWORD`
- `docs/specflow/pages.json` exists and has a matching page entry for the feature area
- Node.js is available on the system

If any prerequisite is missing, skip Playwright and note: "Visual verification skipped — [reason]. Proceeding with code review only."

**Playwright installation (global, one-time):**

1. Check if Playwright is available: run `npx playwright --version`
2. If not available:
   a. Install globally: `npm install -g playwright`
   b. Install the Chromium browser binary: `npx playwright install chromium`
3. If global install fails (permissions, no Node), degrade to code review only and note in the report: "Playwright installation failed — visual verification skipped."

**Verification script:**

1. Generate a temporary Node.js script at `/tmp/specflow-verify-{LINEAR_ID}.js`
2. The script resolves the global Playwright install and uses the library API:
   ```javascript
   // Resolve global playwright — handles varying global node_modules paths
   const globalRoot = require('child_process')
     .execSync('npm root -g').toString().trim();
   const { chromium } = require(globalRoot + '/playwright');

   (async () => {
     const browser = await chromium.launch();
     const context = await browser.newContext();
     const page = await context.newPage();
     // Login using selectors from pages.json auth config
     await page.goto(BASE_URL + LOGIN_PATH);
     await page.fill(EMAIL_SELECTOR, EMAIL);
     await page.fill(PASSWORD_SELECTOR, PASSWORD);
     await page.click(SUBMIT_SELECTOR);
     await page.waitForNavigation();
     // Navigate to the target page (resolved from pages.json by feature area)
     await page.goto(BASE_URL + TARGET_PATH);
     await page.waitForLoadState('networkidle');
     // Screenshot
     await page.screenshot({ path: SCREENSHOT_PATH, fullPage: true });
     // Optional: check for specific selectors from acceptance criteria
     // const element = await page.$(EXPECTED_SELECTOR);
     // console.log(JSON.stringify({ selectorFound: !!element }));
     await browser.close();
   })();
   ```
3. Execute with `node /tmp/specflow-verify-{LINEAR_ID}.js`
4. Save screenshot to `docs/specflow/test/assets/{LINEAR_ID}-{slug}.png`
5. Read the screenshot file to visually verify the page matches Expected State
6. Upload the screenshot to Linear via `mcp__plugin_linear_linear__create_attachment` on the issue
7. Clean up the temp script: delete `/tmp/specflow-verify-{LINEAR_ID}.js`

Record evidence: screenshot reference + any selector check results + PASS/FAIL

#### 3d. Mobile Layer

**Trigger:** Files reference `apps/expo/`, React Native paths, or `*.native.tsx` patterns

Steps:
1. Code review ONLY — Playwright does not apply to mobile
2. Read each mobile file listed in Files to Modify and Files to Create
3. Verify:
   - NativeWind/styling patterns follow project conventions
   - React Native component structure matches acceptance criteria
   - Navigation and screen setup are correct
   - Platform-specific handling (if any) is implemented
4. Flag in the report: "Code review only — manual device testing required for mobile layer"
5. Record evidence: code snippets with PASS/FAIL per criterion

#### 3e. Test Layer

**Trigger:** Layers includes "Tests" OR Files to Modify/Create reference test files (`*.test.*`, `*.spec.*`, `__tests__/`)

Steps:
1. Read each test file listed in the task
2. Verify:
   - Tests exist for the functionality described in acceptance criteria
   - Test coverage matches what the Definition of Done requires
   - Test assertions align with the Expected State
3. Record evidence: test snippets with PASS/FAIL

---

### Phase 4: QA Scenario Execution

For each Given/When/Then scenario from the task review's QA Verification section, map it to the appropriate verification method and execute.

#### 4a. Scenario Mapping

For each scenario, determine the verification method based on its content:

| Scenario mentions | Method |
|---|---|
| API, endpoint, route, request, response, Swagger | Backend code review (Phase 3b) |
| UI, page, navigation, "I am on", component, button, form | Playwright (if available) or frontend code review (Phase 3c) |
| Database, schema, model, records, migration | Prisma schema check (Phase 3a) |
| Pure logic, calculation, validation rule | Code path analysis of the relevant module |

#### 4b. Scenario Execution

For each scenario:

1. Parse the Given/When/Then structure
2. Execute using the mapped Phase 3 method:
   - **Given** — verify the precondition exists or can exist (e.g., "Given a user is logged in" → check auth is implemented)
   - **When** — verify the action is implemented (e.g., "When they submit the form" → check form handler exists and processes input)
   - **Then** — verify the expected outcome occurs (e.g., "Then they see a success message" → check the success state is rendered)
3. Record the result with one of four statuses:
   - **PASS** — the scenario is met with clear evidence (cite specific file:line)
   - **FAIL** — the scenario is definitively not met, with evidence showing what's missing or wrong
   - **SKIP** — cannot verify automatically (explain why, e.g., "requires running server" or "depends on third-party service")
   - **WARN** — partially met or ambiguous (explain what's uncertain and why human review is needed)

---

### Phase 5: Devil's Advocate Self-Review

This phase is **mandatory** — never skip it, even if all results are PASS.

Re-read ALL collected evidence from Phases 3 and 4, then challenge every finding.

#### 5a. Challenge Every PASS

For each PASS verdict, ask:

- "Am I sure this is actually implemented, or am I just seeing the file exists?"
- "Does the code actually handle the edge case described in the criterion?"
- "Could there be a runtime failure that static review wouldn't catch?"
- "Am I reading the correct version of the code, or could this be on an unmerged branch?"
- "Does the implementation achieve the specific behavior described, or just something superficially similar?"

If any challenge raises doubt, downgrade to WARN with an explanation.

#### 5b. Challenge Every FAIL

For each FAIL verdict, ask:

- "Am I reading the code correctly? Could the implementation be in a different file?"
- "Is the acceptance criterion perhaps outdated — was the task descoped during development?"
- "Am I being too strict? Does the implementation achieve the spirit of the criterion even if not the letter?"
- "Could the feature be implemented via a different pattern than what I expected?"

If any challenge reveals a misreading, upgrade to PASS or WARN with an explanation.

#### 5c. Check for Uncovered Concerns

Look for things the acceptance criteria might have missed:

- **Error handling** — are error states handled, or does the code only cover the happy path?
- **Security** — any obvious vulnerabilities (unsanitized input, missing auth checks, exposed secrets)?
- **Performance** — any N+1 queries, unbounded loops, or missing pagination?
- **Accessibility** — if UI work, are aria labels, keyboard navigation, and screen reader support present?

These observations go in the "Additional Observations" section of the report — they don't change verdicts on existing criteria but flag potential issues.

#### 5d. Document Adjustments

Record every verdict change made during devil's advocate review:

```
Devil's advocate adjustment: Criterion 3 changed PASS → WARN
Reason: Code exists but error handling for network timeout is not implemented.
The criterion says "handles all error states" but only 400/401/500 are caught.

Devil's advocate adjustment: Scenario 2 changed FAIL → PASS
Reason: Implementation uses a different component name than expected,
but the behavior matches the scenario exactly. Found at src/components/WidgetForm.tsx:47.
```

---

### Phase 6: Post Verdict to Linear

#### 6a. Determine Overall Verdict

Based on all criteria, scenarios, and devil's advocate adjustments:

- **PASS** — All acceptance criteria and QA scenarios pass (including after devil's advocate review). Minor observations from Phase 5c (uncovered concerns) do not block PASS — they go in the Additional Observations section as informational notes. WARN results from devil's advocate adjustments DO block PASS only if they relate to a specific acceptance criterion or QA scenario. A general observation like "could add more error handling" is an observation, not a WARN.
- **FAIL** — One or more acceptance criteria are definitively not met, with clear evidence showing what's missing or wrong.
- **NEEDS REVIEW** — Any of: a specific acceptance criterion or QA scenario was downgraded to WARN by devil's advocate (partially met but uncertain), criteria couldn't be verified automatically (SKIP with no alternative evidence), or devil's advocate raised a concern that directly contradicts a stated criterion.

#### 6b. Post Structured Comment

Use `mcp__plugin_linear_linear__save_comment` on the Linear issue with this template:

<comment-template>
## Specflow QA Verification Report

**Verdict: [PASS|FAIL|NEEDS REVIEW]**
**Verified at:** [ISO 8601 timestamp, e.g., 2026-04-09T14:30:00Z]
**Layers reviewed:** [comma-separated list, e.g., Database/Schema, API, UI, Tests]

---

### Acceptance Criteria

- [x] Criterion 1 — **PASS** Evidence: `src/server/api/widgets.ts:23` implements the creation endpoint with validation
- [ ] Criterion 2 — **FAIL** Missing: No input sanitization found. Expected in `src/server/api/widgets.ts` based on task spec.
- [ ] Criterion 3 — **WARN** Partial: Component renders but loading state not implemented. See `src/components/Widget.tsx:45`

### QA Scenarios

- [x] Given a user is logged in, When they create a widget, Then it appears in the list — **PASS** Verified via code review of `src/server/api/widgets.ts` and `src/components/WidgetList.tsx`
- [ ] Given invalid input, When they submit, Then they see validation errors — **FAIL** No client-side validation found in `src/components/WidgetForm.tsx`
- [ ] Given the API is down, When they submit, Then they see an error message — **SKIP** Requires running server to verify error handling behavior

### Screenshots

[If Playwright was used:]
- `CXP-42-widget-creation.png` — Widget creation page after login (attached)

[If Playwright was not used:]
- No screenshots — visual verification was not available ([reason])

### Devil's Advocate Notes

[List all adjustments, or "No adjustments needed — all findings confirmed on review."]

### Additional Observations

[Anything the acceptance criteria didn't explicitly cover but is worth noting:]
- Error handling: [observation]
- Security: [observation]
- Performance: [observation]
- [Or: "No additional concerns identified."]

---

> This report was generated by Specflow QA — an automated verification system.
</comment-template>

#### 6c. Upload Screenshots

If Playwright produced screenshots:

1. For each screenshot file in `docs/specflow/test/assets/`:
   a. Use `mcp__plugin_linear_linear__create_attachment` with:
      - **issueId**: The Linear issue identifier
      - **title**: `{LINEAR_ID}-{slug}.png`
      - **url**: The local file path (or upload URL if Linear requires remote URLs)
   b. Reference the attachment in the comment

#### 6d. Apply QA Label to Linear Issue

After posting the comment, apply the appropriate QA label to the Linear issue using `mcp__plugin_linear_linear__save_issue`. The label group is **Specflow QA** with three child labels:

| Verdict | Label to Apply |
|---------|---------------|
| PASS | `QA Pass` |
| FAIL | `QA Fail` |
| NEEDS REVIEW | `QA Need Review` |

Steps:
1. Read the issue's existing labels from the Linear issue data (fetched in Phase 2a)
2. Build the updated labels array: keep all existing labels, remove any previous QA labels (`QA Pass`, `QA Fail`, `QA Need Review`), and add the new verdict label
3. Call `mcp__plugin_linear_linear__save_issue` with:
   - **id**: The Linear issue identifier
   - **labels**: The updated labels array (existing labels + new QA label)

This ensures:
- Previous QA labels are replaced (e.g., a re-verified task moves from `QA Fail` to `QA Pass`)
- Existing non-QA labels (e.g., `Feature`, `Bug`) are preserved
- The team can filter Done issues by QA status at a glance

---

### Phase 7: Verification Tracking

Maintain a local verification log so future runs can skip already-verified tasks.

#### 7a. Determine Log File

The log filename mirrors the task review naming convention:
- Pattern: `{NNN}-test-{project-slug}.md`
- The `{NNN}` prefix inherits from the parent task review's number
- Example: If the task review is `001-tasks-widget-mvp.md`, the log is `001-test-widget-mvp.md`

#### 7b. Create Directories

Create the following directories if they don't exist (silently, without asking):
- `docs/specflow/test/`
- `docs/specflow/test/assets/`

#### 7c. Create or Update Log File

**If the log file does not exist**, create it with this format:

```markdown
---
project: "[Project Name]"
task_review: "docs/specflow/task/{NNN}-tasks-{slug}.md"
last_run: YYYY-MM-DD
---

# Verification Log: [Project Name]

| Linear ID | Title | Verdict | Verified At | Report |
|-----------|-------|---------|-------------|--------|
| CXP-42 | Widget creation API | PASS | 2026-04-09T14:30:00Z | [Linear comment](url) |
```

**If the log file exists**, update it:

1. Read the existing file
2. Parse the verification table
3. For the current task:
   - If the Linear ID already has a row (previously failed), update the Verdict, Verified At, and Report columns. The old Linear comment is preserved for audit trail — this row now links to the new comment.
   - If the Linear ID has no row, append a new row
4. Update `last_run` in the YAML frontmatter to today's date
5. Save the updated file

#### 7d. Batch Mode Behavior

When verifying multiple tasks:
- Process tasks one at a time — complete all phases (2-6) for one task before starting the next
- Show the verdict for each task before moving to the next: "CXP-42: PASS. Moving to CXP-43..."
- If one task fails verification, continue with the remaining tasks — do not halt
- Update the verification log after each task (not all at once at the end)
- After all tasks are processed, show a summary:

```
Verification complete:

| Linear ID | Title | Verdict |
|-----------|-------|---------|
| CXP-42 | Widget creation API | PASS |
| CXP-43 | Widget listing UI | FAIL |
| CXP-45 | User settings page | NEEDS REVIEW |

Results: 1 PASS, 1 FAIL, 1 NEEDS REVIEW
All reports posted to Linear. Verification log updated: docs/specflow/test/001-test-widget-mvp.md

For FAIL/NEEDS REVIEW tasks: fix the issues and re-run /specflow:test to re-verify.
```

---

## Guidance

- **Read-only verification.** NEVER modify source code, schema files, test files, or any project files other than the verification log in `docs/specflow/test/`. This skill is strictly an observer.
- **Degrade gracefully.** If `config.json` is missing, prompt for the Linear team name manually. If Playwright isn't installed, skip visual verification. If `.env` credentials are missing, skip Playwright. If `pages.json` is missing, fall back to code review only for frontend. Never halt the entire verification because one capability is unavailable.
- **Always run devil's advocate.** Phase 5 is mandatory for every verification, even if all criteria pass. The goal is honest assessment, not rubber-stamping.
- **Create directories silently.** `docs/specflow/test/` and `docs/specflow/test/assets/` should be created without asking the user.
- **Screenshots go two places.** When Playwright runs, screenshots are saved locally to `docs/specflow/test/assets/` AND uploaded to Linear as attachments. Both are mandatory.
- **Process one task at a time in batch mode.** Complete all phases for one task before starting the next. Show results between tasks so the user can see progress.
- **Respect the Export Map as source of truth.** The reverse lookup from Linear ID to review document is built from Export Maps. If a task has no Export Map entry, it gets limited verification — flag this clearly.
- **Evidence over opinion.** Every PASS and FAIL must cite specific `file:line` references. "Looks correct" is not acceptable evidence. "Implemented at `src/api/widgets.ts:47` with input validation matching criterion 2" is.
- **Do NOT modify the parent PRD or task review documents.** Only the verification log in `docs/specflow/test/` is written by this skill.

## Error Handling

- **Site unreachable during Playwright:** Skip Playwright for that task, continue with code review, note in the report: "Visual verification skipped — site unreachable at [URL]."
- **Login fails during Playwright:** Report credentials issue, skip Playwright, continue with code review, note: "Visual verification skipped — login failed. Check SPECFLOW_TEST_EMAIL and SPECFLOW_TEST_PASSWORD in .env."
- **Files listed in task no longer exist:** Don't auto-fail. Search the codebase for the content/patterns that should be in those files (files may have been renamed or moved). Note the discrepancy in the report. If the content is found elsewhere, verify against it. If truly missing, flag as FAIL with explanation.
- **Task was descoped during development:** If the Linear issue or review document shows signs of descoping (comments mentioning scope reduction, criteria that don't match the implementation), flag as NEEDS REVIEW rather than FAIL. Note: "Implementation may reflect descoped requirements — human review recommended."
- **Code not merged to main:** Check if the implementation exists on the current branch. If not, check if a feature branch exists (look for branch names matching the Linear ID or task title). If code is on an unmerged branch, flag as NEEDS REVIEW with: "Code may not be merged yet — found on branch [name]" or "Code not found on current branch."
- **Linear API failures:** Save all results to the local verification log. Offer to retry the Linear comment posting: "Verification complete but Linear comment failed to post. Results saved locally. Retry posting to Linear?"
- **Playwright install fails:** Skip visual verification entirely. Note in the report: "Playwright not available — visual verification skipped. Install with `npm install -g playwright && npx playwright install chromium`."
- **Playwright not installed and global install not possible:** Degrade to code-review-only for all frontend verification. Document in report: "Visual verification unavailable — Playwright could not be installed."
- **General principle:** The skill must always run to completion for every selected task. Degrade gracefully — verify what you can, skip what you can't, and document all limitations in the report rather than halting.
