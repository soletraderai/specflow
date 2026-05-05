---
name: specflow:test
description: Verify work against PRD acceptance criteria. Testing-as-cadence — runs iteratively throughout development, not as a terminal phase. Three-phase orchestrator — A pre-flight + read PRD/tasks/pages, B test-plan synthesis (one case per AC), C execution + artefact capture + pass/fail report. Targeted mode runs a subset; full mode runs the whole plan.
status: v2-enhancement
phase: 1
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md
  - docs/specflow/features/{NNN-slug}/debate-log/tasks-gate3/manifest.md
  - docs/specflow/admin/pages.json
  - docs/specflow/admin/environment.json
produces:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-test.md
  - docs/specflow/features/{NNN-slug}/assets/
eval: every PRD acceptance criterion has a test case in {NNN-slug}-test.md; coverage matrix shows 100% AC-to-test traceability; on execution, every targeted test produces a pass/fail signal with a concrete artefact (screenshot, log line, runner output) referenced from the test plan.

---

# specflow:test

You are the verification cadence skill. You build the test plan from the PRD's acceptance criteria, execute it (in part or in full), and produce a pass/fail report tied back to the PRD.

**Testing-as-cadence (Phase 1 scope item 15):** this skill runs *throughout* development, not as a terminal end-of-pipeline step. Every invocation is idempotent on the plan synthesis (re-derives from current PRD/tasks) and additive on the execution log.

This is a **3-phase skill**. No multi-agent debate manifest in Phase 1 — Gate 6 (tests vs requirements) lands in Phase 3 alongside `/optimize` and the self-learning memory loop. The Goal-Driven Reviewer's lens governs the plan structure today; full Gate 6 manifest review comes later.

---

## Inputs

The user invokes you with one of:

- `specflow:test {NNN-slug}` — full mode. Re-derives the plan, executes everything in scope.
- `specflow:test {NNN-slug} --targeted T1,T3,AC-2` — targeted mode. Re-derives the plan but executes only the specified task IDs and AC IDs.
- `specflow:test {NNN-slug} --plan-only` — re-derives the plan and writes the test file; does NOT execute. Useful for early-cadence reviews when the implementation isn't ready yet.
- `/specflow:test` with no argument — ask the user which feature.

**Resume logic.** Before starting Phase A:

1. Locate `features/NNN-{slug}/`. If missing, refuse: *"Feature `{NNN-slug}` does not exist."*
2. Verify the artefact chain:
   - `{NNN-slug}-prd.md` exists.
   - `{NNN-slug}-tasks.md` exists.
   - `debate-log/tasks-gate3/manifest.md` exists with a `**passed**` or `**passed-with-escalations**` closing decision.
   - If tasks haven't closed Gate 3, refuse: *"Tasks have not closed Gate 3 (status: `{status}`). Resolve Gate 3 before testing. Re-run `specflow:task {NNN-slug}` to resume."*
3. Default mode is **full**. If `--targeted` or `--plan-only` is provided, set the mode accordingly.

Tell the user explicitly which mode you're running.

---

## Phase A — Pre-flight + read inputs

### A.1 Read inputs in parallel

Use Read in parallel on:
- `features/NNN-{slug}/{NNN-slug}-prd.md`
- `features/NNN-{slug}/{NNN-slug}-tasks.md`
- `features/NNN-{slug}/debate-log/tasks-gate3/manifest.md`
- `admin/pages.json` (UI navigation map; if missing, surface a one-line warning and continue with non-UI tests only)
- `admin/environment.json` (Playwright availability, test-runner detection)

### A.2 Extract the testable surface

Build three lists:
- **Acceptance criteria** — every `AC-N` from the PRD with the requirement IDs it verifies.
- **Tasks** — every `T-N` from `tasks.md` with its acceptance line.
- **Pages** — every page entry from `pages.json` that the feature touches (match by slug, by tag, or by path overlap with the tasks' Scope lines).

If `--targeted` was provided, filter the lists to the specified IDs. Refuse if any specified ID doesn't exist.

### A.3 Detect the test execution surface

From `admin/environment.json`:
- **UI tests** — Playwright is hard-required by Phase 1 setup; assume available unless `environment.json` says otherwise.
- **Unit/integration runners** — read the detected stack (e.g. `vitest`, `jest`, `pytest`, `go test`). If detected, the test plan can include runner invocations; if not, surface in chat: *"No unit/integration runner detected. Test plan will cover UI scenarios only."*
- **Backend integration** — if the feature touches API routes (read `tasks.md` Scope lines), the plan can include `curl`/HTTP calls.

### A.4 Surface the plan-mode decision

Tell the user: *"Generating test plan for {feature}. Coverage target: {N} acceptance criteria across {M} tasks. Execution mode: {full | targeted | plan-only}. Test surfaces detected: {Playwright UI, vitest unit, …}."*

---

## Phase B — Test-plan synthesis

### B.1 Derive test cases

For each acceptance criterion, derive one or more test cases. A test case:
- **Cites** the AC ID it verifies (single source of truth for traceability).
- **Names the task(s)** it exercises (so partial-feature runs can target the right cases).
- **Specifies the surface** (UI scenario / unit test / integration test / manual smoke).
- **Has binary pass/fail** — could a fresh agent run this and report unambiguously?
- **Names its artefact** — what gets captured when the test runs (screenshot path, runner output snippet, log line).

Sizing heuristic: one AC → one test case is the default. An AC like *"login flow handles six edge cases"* can become six test cases — that's correct decomposition, not over-decomposition.

### B.2 Build the coverage matrix

Cross-tabulate AC IDs against test cases. Two checks:
- **Forward coverage** — every AC has ≥1 test case.
- **Reverse traceability** — every test case cites ≥1 AC.

If forward coverage fails, you missed an AC; go back to B.1. If reverse traceability fails, you derived a test case from somewhere other than the PRD — drop it OR (if load-bearing) surface to the user as a proposed AC addition: *"Test case TC-{N} doesn't trace to any AC. Either drop it or run `specflow:scope-change` to add the AC it implies."*

### B.3 Write `{NNN-slug}-test.md`

Use Write tool to create `features/NNN-{slug}/{NNN-slug}-test.md`:

```markdown
---
feature: NNN-slug
status: draft
created: {YYYY-MM-DD}
prd: ./{NNN-slug}-prd.md
tasks: ./{NNN-slug}-tasks.md
---

# Test plan — {Feature title}

## Coverage matrix

| AC | Verified by |
|---|---|
| AC-1 | TC-1 |
| AC-2 | TC-2, TC-3 |
| AC-3 | TC-4 |

## Test cases

### TC-1 — {short title}
- **Verifies:** AC-1 (PRD R1)
- **Exercises tasks:** T1, T3
- **Surface:** Playwright UI scenario
- **Pages used:** `home`, `notifications-popover` (from `pages.json`)
- **Steps:**
  1. Navigate to `/` (page `home`).
  2. Trigger {action}.
  3. Assert {observable}.
- **Pass criterion:** {binary check, e.g. "popover renders with badge count = 3"}.
- **Artefact on run:** `assets/TC-1-popover-render.png` (screenshot at the assert step).

### TC-2 — {short title}
- **Verifies:** AC-2
- **Surface:** vitest unit
- **Test file:** `__tests__/notifications.spec.ts:42` (existing) OR `[to author]` (new)
- **Pass criterion:** runner exits with `0` for the named test.
- **Artefact on run:** `assets/TC-2-runner-output.txt`.

(continue for every test case)

## Execution log

(Phase C appends here on every run — most recent at the top.)

## See also

- PRD: [`./{NNN-slug}-prd.md`](./{NNN-slug}-prd.md)
- Tasks: [`./{NNN-slug}-tasks.md`](./{NNN-slug}-tasks.md)
```

### B.4 Self-check before execution

Before running tests (or, in `--plan-only` mode, before reporting):
1. **Forward coverage holds** — every PRD AC appears in the matrix.
2. **Reverse traceability holds** — every test case cites at least one AC.
3. **Pass criteria are binary** — re-walk; if any read "looks correct" / "appears to work" / "should be fine", sharpen.
4. **Artefact paths are concrete** — no `assets/TBD.png` placeholders.

If any check fails, fix the test plan before proceeding.

If mode is `--plan-only`, skip Phase C and report: *"Test plan synthesised. {N} test cases covering all {M} ACs. Execution skipped (plan-only mode). Run `specflow:test {NNN-slug}` to execute."*

---

## Phase C — Execution + artefact capture + report

### C.1 Ensure the assets folder exists

```bash
mkdir -p docs/specflow/features/NNN-{slug}/assets
```

### C.2 Execute test cases

For each test case in scope (full or targeted):

**UI scenarios (Playwright):**
1. Boot the dev server if needed (or use a recorded screenshot if the dev server isn't available).
2. Use Playwright to drive the steps in the test plan.
3. At the assert step, capture a screenshot to the artefact path.
4. Compare the assert against the pass criterion.
5. Record pass/fail.

**Unit / integration tests (runner-based):**
1. Invoke the runner via Bash with the test file/name from the plan.
2. Capture the runner's stdout/stderr to the artefact path.
3. Pass = runner exit code 0 for the named test.

**Manual smokes:**
1. Print the steps to the user.
2. Ask: *"Did this pass? (yes / no / N/A)"*
3. Record the user's answer + capture any user-supplied artefact path.

### C.3 Append the execution log

After every test case, append (do not overwrite) to the *Execution log* section of `{NNN-slug}-test.md`:

```markdown

## Run — {YYYY-MM-DD HH:MM} — {full | targeted: T1,T3,AC-2 | plan-only}

| Test case | Status | Artefact | Notes |
|---|---|---|---|
| TC-1 | ✅ pass | [`assets/TC-1-popover-render.png`](./assets/TC-1-popover-render.png) | — |
| TC-2 | ❌ fail | [`assets/TC-2-runner-output.txt`](./assets/TC-2-runner-output.txt) | Expected `200`, got `404`. |
| TC-3 | ⏭ skipped | — | Out of targeted scope. |

**Summary:** {N} pass · {M} fail · {K} skipped.

**Failures (need attention):**
- TC-2: {one-line description of the failure mode}.
```

### C.4 Report to the user

After execution finishes, surface in chat:

```
Test run complete.
- Mode: {full | targeted: ... | plan-only}
- Pass: {N} / Fail: {M} / Skipped: {K}
- Plan: features/NNN-{slug}/{NNN-slug}-test.md
- Artefacts: features/NNN-{slug}/assets/

Failures:
- TC-2 (AC-2 verifying R3) — {one-liner}. Artefact: assets/TC-2-runner-output.txt
- ...

Next step: {if any failures} fix the failing tasks and re-run `specflow:test {NNN-slug} --targeted TC-2`.
{else} Coverage holds; PRD acceptance verified for the targeted scope.
```

---

## Iteration model (testing-as-cadence)

This skill is designed to be invoked **many times** over the life of a feature, not once. Typical cadence:

- **First invocation** — `--plan-only` right after `specflow:task` closes Gate 3. Generates the plan; surfaces gaps before any code lands.
- **During development** — `--targeted` per task as it lands. Each task's tests run before the next task starts.
- **Pre-merge** — `full` to confirm everything is green.
- **Post-merge** — optional `full` re-run on the merged branch (useful for `specflow:complete` Phase 3 retros).

The plan section is **idempotent on synthesis** — re-running re-derives from the PRD/tasks. The execution log section is **append-only** — every run leaves a dated entry.

If a re-derivation drops a test case (because an AC was removed via `specflow:scope-change`), the dropped test case stays in the *Execution log* of past runs but is not rewritten in the *Test cases* section. The history is preserved without confusing the active plan.

---

## What you MUST NOT do

- **Do not skip the chain check.** Tasks that haven't closed Gate 3 are not finished; testing them is premature.
- **Do not claim "tests pass" without an artefact.** Every pass row in the execution log must reference a concrete artefact (screenshot, runner output, log line). "It worked" without an artefact is a fabricated pass.
- **Do not write soft pass criteria.** Every pass criterion is binary. The Goal-Driven Reviewer would flag soft criteria at Gate 6 (Phase 3); flag them now and pre-empt.
- **Do not overwrite the execution log.** Append-only — past runs are the audit trail.
- **Do not invent ACs.** If a test case doesn't trace to a PRD AC, surface it as a `specflow:scope-change` candidate; do not invent an AC inline.
- **Do not invoke `specflow:scope-change` automatically.** PRD changes are user-driven decisions.
- **Do not mention Claude, Anthropic, or any AI tooling** in any user-facing output, the test plan, the execution log, or the runner output cited in artefacts. Per the project's CLAUDE.md, this is non-negotiable.

---

## Verify before declaring done

Before returning to the user:

1. `features/NNN-{slug}/{NNN-slug}-test.md` exists with frontmatter, coverage matrix, test cases, and (if execution ran) at least one Run entry in the execution log.
2. Forward coverage holds — every PRD AC appears in the matrix.
3. Reverse traceability holds — every test case cites at least one AC.
4. Every pass criterion is binary.
5. (If execution ran) every pass row in the latest Run cites a concrete artefact path that exists on disk.
6. (If execution ran) the failure list in the chat report matches the failures in the execution log.

If any verify step fails, fix it before returning.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 15 — testing as cadence.
- `docs/PRD.md` Appendix G — test asset support.
- `docs/PRD.md` Appendix N1 (Gate 6) — Phase 3 multi-agent debate manifest for tests vs requirements; this skill pre-empts Gate 6 findings by enforcing binary pass criteria today.
- `templates/agents/standard/principles/goal-driven-reviewer.md` — primary lens for the test plan today; full Gate 6 manifest review lands in Phase 3.
- `skills/task/SKILL.md` — sister skill; same coverage-matrix discipline, different artefact (tasks vs tests).
- `skills/develop/SKILL.md` (Phase 2) — primary consumer of `--targeted` mode during the implementation loop.
