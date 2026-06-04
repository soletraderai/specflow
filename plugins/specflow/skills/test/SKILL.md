---
name: specflow:test
description: Verify work against PRD acceptance criteria. Testing-as-cadence — runs iteratively throughout development, not as a terminal phase. Default flow is A pre-flight → B test-plan synthesis with lesson-query → C execution; `--task T1,T3,AC-2` filters to a subset; `--plan-only` skips execution (auto-inferred when no shipped code exists yet). Feedback capture (Phase D) is no longer flag-gated — it auto-prompts after a green run and at Phase A.0 when the agent detects a shipped-behaviour gap in the conversation context. Phase D writes a Retroactive: true test case + a lesson + an attribution row so the system gets smarter on the next feature. Phase C is interactive and tiered: each case is checked in sequence — a pass passes silently, a hard-tier contract failure stops the run for the user, and a soft-tier divergence is surfaced in plain English (spec said X, build does Y) for a keep/reject call that is remembered in a reconcile manifest so later runs test against the confirmed truth, not the stale spec.
status: v2-enhancement
phase: 1
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md
  - docs/specflow/features/{NNN-slug}/debate-log/tasks-gate3/manifest.md
  - docs/specflow/admin/pages.json
  - docs/specflow/admin/environment.json
  - docs/specflow/admin/lessons.json
produces:
  - docs/specflow/features/{NNN-slug}/test/{NNN-slug}-test.md
  - docs/specflow/features/{NNN-slug}/test/{NNN-slug}-reconcile.json (interactive reconcile manifest — living record of confirmed deviations + confirmed fails)
  - docs/specflow/features/{NNN-slug}/assets/
  - docs/specflow/admin/lessons.json (mutated; .bak preserved on every write)
  - docs/specflow/admin/task-history.json (appended on Phase D feedback capture)
  - docs/specflow/admin/pages.json (lazy-appended on first UI run per route; never duplicates)
eval: every PRD acceptance criterion has a test case in test/{NNN-slug}-test.md; coverage matrix shows 100% AC-to-test traceability; on execution, every non-retroactive test produces a pass/fail signal with a concrete artefact (screenshot, log line, runner output) referenced from the test plan; retroactive test cases (Retroactive: true) are recorded with ⏭ retroactive status and skipped; lesson-query in B.0 surfaces matched active lessons; the post-green feedback prompt or invocation-time intent detection routes to Phase D to write one new lesson to lessons.json plus one new (Retroactive: true) test case to test/{NNN-slug}-test.md tagged with the lesson id; UI scenarios append unseen routes to admin/pages.json idempotently (009-pages-policy v2.4.0); when invoked with `--plan-only --task T{N}`, the per-task plan section written into test/{NNN-slug}-test.md contains at least one test case marked `Status: red (failing)` (017-tdd-discipline v2.5.0).

---

# specflow:test

You are the verification cadence skill. You build the test plan from the PRD's acceptance criteria, execute it (in part or in full), and produce a pass/fail report tied back to the PRD.

**Testing-as-cadence (Phase 1 scope item 15):** this skill runs *throughout* development, not as a terminal end-of-pipeline step. Every invocation is idempotent on the plan synthesis (re-derives from current PRD/tasks) and additive on the execution log.

This is a **3-phase skill**. No multi-agent debate manifest in Phase 1 — Gate 6 (tests vs requirements) lands in Phase 3 alongside `/optimize` and the self-learning memory loop. The Goal-Driven Reviewer's lens governs the plan structure today; full Gate 6 manifest review comes later.

---

## Inputs

The user invokes you with one of:

- `specflow:test {NNN-slug}` — default. Re-derives the plan, executes every in-scope, non-retroactive test case.
- `specflow:test {NNN-slug} --task T1,T3,AC-2` — filter to the listed task IDs / AC IDs (single or comma-list; supersedes the old `--targeted` flag from 2.13 and earlier — the alias is deprecated but accepted for one release).
- `specflow:test {NNN-slug} --plan-only` — explicit override that skips Phase C even when shippable code exists (TDD red-artefact use case). Plan-only is otherwise **auto-inferred** when no executable code exists for the targeted scope — the flag is only needed to force plan-only against shipped code.
- `specflow:test {NNN-slug} --plan-only --task T{N}` — per-task variant of plan-only. Writes only the per-task plan section into `test/{NNN-slug}-test.md` (not the whole feature plan) and marks the primary AC's case as `Status: red (failing)` by default — the Red artefact for `specflow:develop`'s 017-tdd-discipline cycle. **Phase B.5 (Codex pass + user prompt) is skipped** in `--task` mode — the per-task slice inherits whatever B.5 outcome the feature-level test plan recorded; Gate 5's Codex reviewer covers cross-provider concerns at code-vs-plan time. Doctrine: `templates/admin/tdd-discipline.md`.
- `/specflow:test` with no argument — ask the user which feature.

**Feedback capture has no flag.** Phase D (lesson capture) fires through two implicit paths: (a) automatically prompted at the end of a green run via Phase C.4, (b) routed at Phase A.0 when the agent detects a shipped-behaviour gap in the immediately-prior conversation context. The old `--feedback` flag is removed; the alias is accepted for one release and routes to Phase D directly.

**Resume logic.** Before starting Phase A:

1. Locate `features/NNN-{slug}/`. If missing, refuse: *"Feature `{NNN-slug}` does not exist."*
2. Verify the artefact chain:
   - `{NNN-slug}-prd.md` exists.
   - `{NNN-slug}-tasks.md` exists.
   - `debate-log/tasks-gate3/manifest.md` exists with a `**passed**` or `**passed-with-escalations**` closing decision.
   - If tasks haven't closed Gate 3, refuse: *"Tasks have not closed Gate 3 (status: `{status}`). Resolve Gate 3 before testing. Re-run `specflow:task {NNN-slug}` to resume."*
3. Parse flags: `--task` and `--plan-only` are explicit; the deprecated `--targeted` alias is normalised to `--task` with a one-line deprecation notice in chat.
4. Run **Phase A.0 — intent detection** below before continuing to A.1.

### Phase A.0 — Intent detection

Skim the immediately-prior conversation context (the user's last 2-3 turns) for signals that this invocation is really a feedback capture, not a verification run. Signals: phrases like *"X was wrong"*, *"all green but Y looked off in production"*, *"the splash screen had the wrong font"*, *"the lesson is…"* — any sentence pinning a *gap in shipped behaviour* the test plan missed.

If signal present:

> Sounds like you spotted a gap rather than wanting a re-run. Route to Phase D (capture a lesson) instead? [yes/no]

On `yes` → skip Phases A.1-C and jump to Phase D. On `no` → continue normal A.1.

If no signal present, continue silently to A.1.

The deprecated `--feedback` flag, if passed, short-circuits this prompt and jumps straight to Phase D.

Tell the user explicitly which mode you're running (default / filtered / plan-only / feedback-via-prompt).

---

## Phase A — Pre-flight + read inputs

### A.1 Read inputs in parallel

Use Read in parallel on:
- `features/NNN-{slug}/{NNN-slug}-prd.md`
- `features/NNN-{slug}/{NNN-slug}-tasks.md`
- `features/NNN-{slug}/debate-log/tasks-gate3/manifest.md`
- `admin/pages.json` (UI navigation map; if missing, surface a one-line warning and continue with non-UI tests only)
- `admin/environment.json` (Playwright availability, test-runner detection)
- `features/NNN-{slug}/test/{NNN-slug}-reconcile.json` (the reconcile manifest — confirmed deviations + confirmed fails from prior runs; if missing, this is the feature's first reconcile and the manifest is created lazily in Phase C)

### A.2 Extract the testable surface

Build three lists:
- **Acceptance criteria** — every `AC-N` from the PRD with the requirement IDs it verifies.
- **Tasks** — every `T-N` from `tasks.md` with its acceptance line.
- **Pages** — every page entry from `pages.json` that the feature touches (match by slug, by tag, or by path overlap with the tasks' Scope lines).

If `--task` was provided (or the deprecated `--targeted` alias), filter the lists to the specified IDs (single or comma-list). Refuse if any specified ID doesn't exist.

### A.3 Detect the test execution surface

From `admin/environment.json`:
- **UI tests** — Playwright is hard-required by Phase 1 setup; assume available unless `environment.json` says otherwise.
- **Unit/integration runners** — read the detected stack (e.g. `vitest`, `jest`, `pytest`, `go test`). If detected, the test plan can include runner invocations; if not, surface in chat: *"No unit/integration runner detected. Test plan will cover UI scenarios only."*
- **Backend integration** — if the feature touches API routes (read `tasks.md` Scope lines), the plan can include `curl`/HTTP calls.

### A.4 Surface the plan-mode decision

Tell the user: *"Generating test plan for {feature}. Coverage target: {N} acceptance criteria across {M} tasks. Execution mode: {default | filtered by --task | plan-only (forced) | plan-only (auto-inferred — no shipped code yet)}. Test surfaces detected: {Playwright UI, vitest unit, …}."*

**Auto plan-only inference.** Before announcing the mode, decide whether Phase C can execute against the targeted scope. Read the per-AC implementation evidence: for every AC in scope, is there code on disk under the task's `Scope` paths? If zero scoped ACs have shippable code (the TDD red case — `specflow:develop` hasn't run yet, or the targeted task is pre-implementation), force the mode to `plan-only (auto-inferred)` without requiring the `--plan-only` flag. The user can still pass `--plan-only` explicitly to force plan-only against shipped code.

---

## Phase B — Test-plan synthesis

### B.0 Query the lessons registry

Before deriving test cases, query `admin/lessons.json` per the **Lessons registry** section's query algorithm (later in this file). Filter active lessons by overlap with the feature's tags (stack from `environment.json`, surface from PRD Requirements keywords, domain from PRD frontmatter + glossary).

Surface matched lessons in chat:

```
Heads up — relevant lessons from prior work on this project:
- L-001 (Splash screen wrong font + wrong loading state) — UI tasks need a Playwright screenshot diff before Verifier passes.
- L-007 (Token refresh silent failure) — auth tasks must include a token-expiry test case.

The plan below biases toward covering these.
```

Cap at 5 surfaced lessons; if more match, summarise the count and write the full matched-lesson list to `admin/scratch/test-{slug}-{ts}/matched-lessons.json` for the Phase 3 audit trail.

For each matched `escape` or `success` lesson, derive at least one test case in B.1 that explicitly verifies the lesson's remediation. Tag the case with `Source: lesson L-NNN` (in addition to the AC it cites) so the connection is auditable. Lesson-derived test cases still satisfy reverse traceability — they cite the AC the lesson maps to. If a lesson doesn't map to any existing AC, surface to the user as a `specflow:scope-change` candidate; do not invent an AC inline.

If no lessons match, surface: *"No prior lessons match this feature's tags."* Continue.

### B.0.5 Partition matched lessons — REQUIRED vs advisory (per 035-self-learning-loop v2.16.0)

For each matched active lesson, partition:

- **REQUIRED** — when the lesson's `test_fragment.scope` glob overlaps an in-scope task's `Scope` path AND tags overlap `>=1` surface tag with the feature. A REQUIRED lesson's `test_fragment` MUST become a concrete test case in B.1:
  - `ci-check` / `grep` → a runner/grep case with the `assertion` verbatim as the command + `expect` as the pass criterion.
  - `testcase` → the skeleton imported as a `Status: red` case.
  - Tag the case `Source: lesson L-NNN (REQUIRED)` (in addition to its AC reference).
- **advisory** — matched but `test_fragment.scope` does NOT overlap any task's `Scope`, OR no surface-tag overlap, OR `test_fragment` absent / `runnable: false`. Surface the lesson; derive a case if it cleanly maps to an AC; otherwise mention and continue.

Surface each REQUIRED / advisory lesson with its reason. Write the REQUIRED set to `admin/scratch/test-{slug}-{ts}/required-lessons.json` — an array of `{id, test_fragment, derived_tc_id}` objects. B.4 reads this file and refuses if any REQUIRED lesson lacks a covering case.

### B.1 Derive test cases

For each acceptance criterion, derive one or more test cases. A test case:
- **Cites** the AC ID it verifies (single source of truth for traceability).
- **Names the task(s)** it exercises (so partial-feature runs can target the right cases).
- **Specifies the surface** (UI scenario / unit test / integration test / manual smoke).
- **Has binary pass/fail** — could a fresh agent run this and report unambiguously?
- **Names its artefact** — what gets captured when the test runs (screenshot path, runner output snippet, log line).

Sizing heuristic: one AC → one test case is the default. An AC like *"login flow handles six edge cases"* can become six test cases — that's correct decomposition, not over-decomposition.

### B.1.6 Tier each test case — hard vs soft (reconcile tiering)

Every test case carries a `Tier: hard | soft`. The tier decides what happens when the case FAILS in Phase C (see C.2.5): a hard failure stops the run for the user; a soft failure is offered for reconcile.

Assign the tier by ruleset, defaulting to **hard** — an untiered check is treated as a contract until a human says otherwise:

- **hard** — the AC verifies a contract that must not silently drift: data integrity, schema/migrations, auth or tenant isolation, money/payments/billing, idempotency, deletion or retirement of endpoints/data, public API response shape, security. Match on the AC's requirement keywords and the task Scope paths (e.g. Scope touches `**/prisma/**`, `**/migrations/**`, `**/auth/**`, a payments module, or the AC text names a status code / uniqueness / NOT NULL / 410 / 422 contract).
- **soft** — the AC verifies experience or product detail that legitimately moves with the brief during develop: copy, labels, which fields show, layout, ordering, optional-vs-required, product-judgement thresholds (e.g. "exactly 2 vs at least 1"), which media is mandatory, visual styling.

Record the chosen tier and a one-line reason on each test case. A `Source: lesson L-NNN (REQUIRED)` case is always **hard** — a lesson-enforced check is a contract. If the ruleset is ambiguous for a case, default hard and note `Tier: hard (default — unclassified)` so the misclassification is visible and a human can correct it to soft.

### B.2 Build the coverage matrix

Cross-tabulate AC IDs against test cases. Two checks:
- **Forward coverage** — every AC has ≥1 test case.
- **Reverse traceability** — every test case cites ≥1 AC.

If forward coverage fails, you missed an AC; go back to B.1. If reverse traceability fails, you derived a test case from somewhere other than the PRD — drop it OR (if load-bearing) surface to the user as a proposed AC addition: *"Test case TC-{N} doesn't trace to any AC. Either drop it or run `specflow:scope-change` to add the AC it implies."*

### B.3 Write `test/{NNN-slug}-test.md`

Use Write tool to create `features/NNN-{slug}/test/{NNN-slug}-test.md`. The test plan now lives alongside its screenshots inside the feature's `test/` subfolder (per the 2.14 folder reorg) — relative links from the test plan resolve as `./screenshots/...` and `../assets/...`.

```markdown
---
feature: NNN-slug
status: draft
created: {YYYY-MM-DD}
prd: ../{NNN-slug}-prd.md
tasks: ../{NNN-slug}-tasks.md
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
- **Retroactive:** false
- **Tier:** hard | soft (one-line reason — see B.1.6)
- **Surface:** Playwright UI scenario
- **Pages used:** `home`, `notifications-popover` (from `pages.json`)
- **Steps:**
  1. Navigate to `/` (page `home`).
  2. Trigger {action}.
  3. Assert {observable}.
- **Pass criterion:** {binary check, e.g. "popover renders with badge count = 3"}.
- **Artefact on run:** `screenshots/TC-1-popover-render.png` (screenshot at the assert step).

### TC-2 — {short title}
- **Verifies:** AC-2
- **Retroactive:** false
- **Surface:** vitest unit
- **Test file:** `__tests__/notifications.spec.ts:42` (existing) OR `[to author]` (new)
- **Pass criterion:** runner exits with `0` for the named test.
- **Artefact on run:** `../assets/TC-2-runner-output.txt`.

(continue for every test case)

**Retroactive marker.** `Retroactive: true` flags a test case that documents shipped behaviour the original plan didn't cover — typically written by the post-green feedback prompt or Phase A.0 intent-detection routing into Phase D. Phase C **skips** retroactive cases (the feature has already shipped; there is no live implementation to verify in this run) and logs them with `⏭ retroactive` in the run table. Retroactive cases remain in the plan as the auditable lesson-to-AC mapping; future feature plans can reference them via the lessons registry. Defaulting `Retroactive: false` for every newly synthesised case keeps Phase C inclusive by default.

## Brand-consistency lens (per 023-test-brand-consistency v2.6.0)

Beyond binary AC pass/fail, ask the following questions and record findings *separately* from AC findings (so they don't pollute the binary signal). One row per question per page/component touched.

| Question | Answer (yes/no/n-a) | Evidence | Severity (info/concern/block) |
|---|---|---|---|
| Are fonts correct on every page? | | | |
| Are colors / palette tokens correct? | | | |
| Are spacing / layout tokens correct? | | | |
| Are CTAs styled consistently with the rest of the product? | | | |
| Are loading / empty / error states designed (not default)? | | | |
| Are accessibility primitives present (alt text, aria-labels, focus rings, skip links)? | | | |
| Does the change feel on-brand on first read (subjective but recorded)? | | | |
| What might have been missed that the AC list didn't cover? | | | |

Brand findings are **advisory** — they don't fail the binary AC pass/fail signal; they surface in the Execution log for human review. `severity: block` flags an item the human MUST resolve before ship; `concern` is documented technical debt; `info` is observational.

## Execution log

(Phase C appends here on every run — most recent at the top. Each run includes both the AC table outcomes and the brand-consistency lens table.)

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
5. **REQUIRED lessons covered (per 035-self-learning-loop v2.16.0, BLOCKING).** Read `admin/scratch/test-{slug}-{ts}/required-lessons.json`. For every entry, confirm a plan test case carries `Source: lesson L-NNN (REQUIRED)` AND its pass criterion contains the assertion verbatim (for `grep` / `ci-check` kinds) or the named skeleton (for `testcase`). If ANY required lesson is uncovered, the plan does NOT pass B.4 — refuse to proceed to Phase C (or in `--plan-only` refuse `complete`) with:

   *"Plan blocked: required lesson L-NNN ({title}) has no covering test case. Add the case (assertion: `{assertion}`, scope `{scope}`) or run `specflow:scope-change` to retire the lesson before this plan can close."*

If any check fails, fix the test plan before proceeding.

### B.5 Pre-execution Codex adversarial pass

Before execution (or, in `--plan-only` mode, before reporting), run a programmatic Codex adversarial pass against the test plan and capture verbatim output as a file artefact at `features/{NNN-slug}/{NNN-slug}-pre-execution-codex.md`. The user can revise the test plan inline before execution begins. Pre-empts what the full Gate 6 manifest review will cover in Phase 3.

If `admin/environment.json` has `cli.codex.available: false`, write the file with one line — *"Codex CLI not detected — pre-execution pass skipped. Install via `/codex:setup` for full coverage."* — and continue.

Otherwise:

1. Bash-invoke `codex adversarial-review` against the test plan per the orchestrator-pattern fork convention (mirrors develop Phase E.2's in-gate `codex review` invocation). Frame the prompt to challenge whether the pass criteria are truly binary, whether edge cases are covered, and whether any AC is under-tested. Capture stdout to the file path above.
2. On invocation failure (auth, network, exec error), write the error verbatim to the same path with prefix *"Codex pass failed at runtime:"* and continue to step 3.
3. Tell the user: *"Pre-execution Codex pass written to `{path}`. Reply `continue` to proceed, `revise: <description>` to address a specific gap inline, or `skip` to proceed without revisions."*

On `continue`: append `— User reviewed; no revisions, {YYYY-MM-DD}.` to the file. Proceed to C.1 (or to the plan-only report).
On `revise: <description>`: edit `test/{NNN-slug}-test.md` to address the gap, re-run B.2 (coverage matrix) and B.4 (self-check), then re-prompt at B.5.
On `skip`: append `— User skipped without revisions, {YYYY-MM-DD}.` to the file. Proceed; if executing, record *"Pre-execution Codex pass skipped by user"* as a note in the next Run row of the Execution log.

If mode is plan-only (forced via `--plan-only` or auto-inferred per A.4), skip Phase C and report: *"Test plan synthesised. {N} test cases covering all {M} ACs ({R} retroactive). Execution skipped ({plan-only forced | plan-only auto-inferred — no shipped code for the targeted scope}). Run `specflow:test {NNN-slug}` after the implementation lands to execute."*

---

## Phase C — Execution + artefact capture + report

### C.1 Ensure the assets folder exists

```bash
mkdir -p docs/specflow/features/NNN-{slug}/assets
```

### C.2 Execute test cases

**Filter retroactive cases first.** Every test case with `Retroactive: true` is excluded from execution — the feature has already shipped and there is no live implementation to verify in this run. Retroactive cases are recorded in the run table with `⏭ retroactive` status (see C.3). They remain in the plan as the auditable lesson-to-AC mapping; future feature plans cross-reference them via the lessons registry.

**Consult the reconcile manifest first.** Before judging any case, read `{NNN-slug}-reconcile.json`. For each test case with a `confirmed-deviation` entry, evaluate against the manifest's recorded `confirmed_expectation`, NOT the stale AC text — the user already confirmed "look for orange, not green," and that confirmation is the truth now; a match records a silent ✅ pass. For each case with a `confirmed-fail` entry, it stays a fail but is auto-flagged from the manifest without re-asking permission (the user already said "yes, flag it") — record it `❌ fail (known — reconcile {date})`. Only cases with NO manifest entry run the interactive flow in C.2.5 when they diverge.

For each non-retroactive test case in scope:

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

### C.2.5 Interactive reconcile on divergence (sequential, one case at a time)

Phase C is interactive. Walk the in-scope cases in order. A pass passes silently — no user input. When a case **fails** and has no prior manifest entry (C.2's manifest consult already handled the ones that do), branch on its `Tier`:

**Hard-tier failure → stop and ask, before running the next case.** The run pauses here. Surface, in plain English, what broke and why it is a hard contract:

> Hard check failed — {AC-N}: {plain-English what the spec requires vs what the build does}. This is a {data integrity | auth | payments | migration | API-contract} contract, so it cannot be waved through. {file:line evidence}.
> How do you want to handle it? [flag as a real fail / this is an intended change]

- `flag as a real fail` (default) → record `❌ fail (hard)` in the run table, write a `confirmed-fail` entry to the reconcile manifest, append an `escaped-issue` row to `task-history.json`, and route it as a bug. Then continue to the next case.
- `this is an intended change` → a hard contract is changing; this is heavier than a soft reconcile and must go through the proper path. Do NOT silently rewrite the AC. Tell the user: *"That changes a hard contract — run `specflow:scope-change {NNN-slug}` to amend AC-N deliberately, then re-test."* Leave the case as `❌ fail (hard — pending scope-change)` and continue.

If `--task` mode or a non-interactive context means no user is present to answer, do not guess: record the hard failure as `❌ fail (hard — unreviewed)` and list it at the top of the report for the user to handle. Never auto-pass a hard failure.

**Soft-tier failure → classify cosmetic vs judgement, then act.** Not every soft divergence needs the user. First sub-classify, defaulting to *judgement* whenever unsure — ambiguity always escalates:

- **cosmetic** — pure presentation with no behavioural, validation, or data consequence: copy/label wording, spacing/layout, colour or design-token swaps, icon choice, presentational ordering, animation. Auto-reconcile: write a `confirmed-deviation` entry with `approved_by: "auto (cosmetic)"`, flip the case to `✅ pass (reconciled — auto)`, and add it to the run's auto-reconciled digest (surfaced in the C.4 report so the user can audit and override later). No interrupt.
- **judgement** — the divergence changes behaviour, completeness, validation, a threshold, a count, which fields are required or shown, gating, or anything data-affecting (e.g. "exactly 2" vs "at least 1", photo-required vs notes-only). These STOP for the user. Surface the keep/reject question:

  > Looks different here — {AC-N}: spec said "{X}", the build does "{Y}". {file:line}. Is the build correct? [yes, keep it / no, that's wrong]

- `yes, keep it` → the brief moved and the build is the truth. Write a `confirmed-deviation` entry to the reconcile manifest capturing the new expected behaviour in the user's own words (e.g. "Section 9 completeness = at least 1 material, not exactly 2"), the approver, and the date; flip the case to `✅ pass (reconciled {date})`; append a `scope-change` row to `task-history.json`; and surface a one-liner that the underlying PRD AC text is now stale and can be synced via `specflow:scope-change` at the user's convenience (do NOT auto-edit the PRD here — see MUST NOT). From the next run on, C.2's manifest consult tests against this confirmed behaviour, silently.
- `no, that's wrong` → it is a real fail. Confirm before logging, per the user's "happy for me to flag this?" model:
  > Then this is a fail — happy for me to flag it? [yes / not sure]
  On `yes` → record `❌ fail (soft — confirmed)`, write a `confirmed-fail` manifest entry, append an `escaped-issue` row, route as a bug. On `not sure` → record `⏳ fail (soft — unresolved)` and leave it open in the report; it is NOT written to the manifest, so it re-surfaces next run.

Process cases one at a time so the user can steer the run as it goes. Each answer is remembered in the manifest so settled questions are never re-asked — that is what makes later runs converge instead of re-litigating every divergence.

### C.2.1 Lazy-populate `pages.json`

For every UI scenario executed via Playwright, capture the routes visited (the navigated URLs reduced to path-only). After the scenario completes:

1. Read `admin/pages.json` (`{ "routes": [...] }` shape).
2. For each visited path not already present in `routes[].path`, append a new entry:
   ```json
   {
     "path": "/the/visited/path",
     "slug": "{kebab-cased path tail or route name}",
     "first_seen": "{NNN-slug feature id that triggered population}",
     "first_seen_at": "{YYYY-MM-DD}"
   }
   ```
3. Write `admin/pages.json` back. Idempotent — never duplicate by path.
4. The append is the canonical inventory mechanism (`v2/docs/PRD.md` § Resolved decisions: 009-pages-policy v2.4.0). No dedicated `specflow:pages` skill ships.

If a captured path looks dynamic (contains a numeric or UUID segment), normalise to a route pattern (`/users/123` → `/users/:id`) before appending. Use the project's router config when available (Next.js `app/`, Remix `app/routes/`, Express route table) to choose pattern boundaries; fall back to UUID/integer detection.

### C.3 Append the execution log

After every test case, append (do not overwrite) to the *Execution log* section of `test/{NNN-slug}-test.md`:

```markdown

## Run — {YYYY-MM-DD HH:MM} — {default | filtered: T1,T3,AC-2 | plan-only (forced) | plan-only (auto-inferred)}

| Test case | Status | Artefact | Notes |
|---|---|---|---|
| TC-1 | ✅ pass | [`screenshots/TC-1-popover-render.png`](./screenshots/TC-1-popover-render.png) | — |
| TC-2 | ❌ fail | [`../assets/TC-2-runner-output.txt`](../assets/TC-2-runner-output.txt) | Expected `200`, got `404`. |
| TC-3 | ⏭ skipped | — | Out of filtered scope. |
| TC-4 | ⏭ retroactive | — | Documents shipped behaviour (lesson L-007); not executed. |

**Summary:** {N} pass · {M} fail · {K} skipped · {R} retroactive.

**Reconcile:** {D} reconciled-pass · {H} hard-fail · {C} confirmed-fail · {U} unresolved. (Reconciled and confirmed-fail outcomes are recorded in `{NNN-slug}-reconcile.json`.)

**Failures (need attention):**
- TC-2: {one-line description of the failure mode}.
```

### C.4 Report + post-green feedback prompt

After execution finishes, surface in chat:

```
Test run complete.
- Mode: {default | filtered: ... | plan-only (forced|auto-inferred)}
- Pass: {N} / Fail: {M} / Skipped: {K} / Retroactive: {R}
- Plan: features/NNN-{slug}/test/{NNN-slug}-test.md
- Artefacts: features/NNN-{slug}/test/screenshots/ + features/NNN-{slug}/assets/
- Auto-reconciled (cosmetic — no input needed): {A}. Listed below for audit; reply to override any.

Failures:
- TC-2 (AC-2 verifying R3) — {one-liner}. Artefact: assets/TC-2-runner-output.txt
- ...
```

**Post-green prompt** (only when {M} == 0 AND mode was a real execution — skip on plan-only, skip when any failures landed). Before emitting the "Next step" line, ask:

> All tests passed. Any gaps in shipped behaviour the plan missed?

If the user answers `yes` or describes a gap, drop into **Phase D** inline using the user's answer as the seed for D.2's Q1. If the user answers `no` or a clear declination, emit the next-step line:

```
Next step: {if any failures} fix the failing tasks and re-run `specflow:test {NNN-slug} --task TC-2`.
{else} Coverage holds; PRD acceptance verified for the targeted scope.
```

The implicit prompt replaces the old `--feedback` flag. The flag still routes here for one release; new users go through the prompt.

---

## Iteration model (testing-as-cadence)

This skill is designed to be invoked **many times** over the life of a feature, not once. Typical cadence:

- **First invocation** — `--plan-only` right after `specflow:task` closes Gate 3. Generates the plan; surfaces gaps before any code lands.
- **During development** — `--task T{N}` per task as it lands. Each task's tests run before the next task starts.
- **Pre-merge** — `full` to confirm everything is green.
- **Post-merge** — optional `full` re-run on the merged branch (useful for `specflow:complete` Phase 3 retros).

The plan section is **idempotent on synthesis** — re-running re-derives from the PRD/tasks. The execution log section is **append-only** — every run leaves a dated entry.

If a re-derivation drops a test case (because an AC was removed via `specflow:scope-change`), the dropped test case stays in the *Execution log* of past runs but is not rewritten in the *Test cases* section. The history is preserved without confusing the active plan.

---

## Phase D — Feedback capture (auto-routed from C.4 prompt or Phase A.0 intent detection)

Feedback capture is the entry point for the project's self-learning loop. Use it when something that was marked "done" (passed Gate 3, code merged, Verifier signed off) turned out on human review to be wrong. Phase D captures the gap in plain language, attributes it to whichever skill / gate / agent should have caught it, writes the lesson to `admin/lessons.json`, and back-fills the test plan with a covering case (marked `Retroactive: true`) so the same gap is documented and queryable by future feature plans.

**How Phase D is entered:**
- **Post-green prompt (Phase C.4)** — after a real green run, the skill asks if the user spotted a gap. A `yes` lands here with the gap text seeding Q1.
- **Phase A.0 intent detection** — when conversation context shows the user has spotted a shipped-behaviour gap, the agent prompts to route here directly without running A/B/C.
- **Deprecated `--feedback` flag** — accepted for one release as a direct route into Phase D; new users go through the prompts.

Whatever the entry path, **skip Phases A.1-C.4** when running Phase D. Phase D is its own four-step orchestration.

### D.1 Read context

Use Read in parallel on:
- `features/NNN-{slug}/{NNN-slug}-prd.md` — for tag derivation and AC mapping.
- `features/NNN-{slug}/{NNN-slug}-tasks.md` — to map the miss to specific tasks and surface candidate attributions.
- `features/NNN-{slug}/test/{NNN-slug}-test.md` — to surface the existing test cases (the user might say which one should have caught it).
- `admin/lessons.json` — for similarity check against existing entries.
- `admin/environment.json` — for stack-tag derivation.
- `admin/profiles.json` and `admin/rules/glossary.md` — for domain-tag derivation.

If Phase D is entered against a feature whose tasks file is missing or whose Gate 3 hasn't closed, refuse: *"Feedback capture runs against a feature that completed Gate 3. Tasks/Gate 3 status: `{status}`. Resolve before logging feedback."* Feedback only makes sense for features that were "done" — that's what the registry is learning from.

### D.2 Capture the feedback

Prompt the user with three plain-language questions, one at a time. Wait for the answer before showing the next question. Capture each answer **verbatim** — no rephrasing, no summarisation.

```
Q1. What was missed?
    Plain language. Example: "splash screen used the wrong font and showed
    the logo image as the initial loading state instead of a blank background."
```

```
Q2. What should have caught it?
    Examples:
    - "specflow:test plan should have included a visual-diff test case"
    - "Verifier signed off on the code without reviewing screenshots"
    - "Task spec for T-3 didn't require font citation from the design mockup"
    - "The PRD AC was too soft — 'splash screen renders' instead of binary visual checks"
```

```
Q3. What should the system do next time?
    Concrete remediation. Examples:
    - "UI tasks must include a Playwright screenshot diff against the
      proposed.html mockup before Verifier passes"
    - "Add 'visual sign-off required' to the task acceptance line for any
      task that touches a component file"
    - "Test plan synthesis must require a binary pass criterion that names
      a specific font / colour / layout value cited from the design mockup"
```

The user's answers populate `what_was_missed`, `what_should_have_caught_it`, and `remediation` respectively. Do NOT auto-summarise to make them shorter. The user's words are the audit trail.

### D.3 Tag and similarity check

Auto-suggest tags from three sources (controlled vocabulary per the **Lessons registry** section):

1. PRD frontmatter — domain tags (audience, feature category).
2. `environment.json` — stack tags.
3. Keyword-scan of the user's three answers against the surface vocabulary (`ui`, `data-model`, `api`, `auth`, `migration`, `infra`, `cli`, `docs`).

Present:

```
Suggested tags: ["mobile", "react-native", "expo", "ui", "splash-screen"]
Edit the tag set, or press enter to accept.
```

If the user adds a tag that isn't in the controlled vocabulary, refuse: *"Tag '{tag}' isn't in the project vocabulary. Add it to admin/rules/glossary.md (domain), admin/environment.json (stack), or accept one of the canonical surface tags ({list})."*

Then run the similarity check against existing entries in `lessons.json`. For each entry with `status: "active"` and ≥1 overlapping tag, compute string similarity between the captured `what_was_missed` and the existing entry's same field (case-insensitive, ignore stop words). If any score above 0.6, surface the candidate:

```
L-{NNN} ({title}) looks similar:
  what_was_missed: "..."
  remediation:    "..."

Is this:
  (1) a recurrence of L-{NNN} (append an occurrence to the existing entry; status stays active)
  (2) a new attempt that worked, superseding L-{NNN}'s prior failure (this captures becomes a "success" lesson; L-{NNN} flips to "superseded")
  (3) a new, distinct lesson (different root cause despite surface similarity)
```

The user picks. Default if no candidate above threshold: write a new `escape` lesson.

### D.4 Write three artefacts atomically

After D.3 is settled, perform the writes in this order. Make `lessons.json.bak` first; do not start the writes until the backup exists.

1. **`admin/lessons.json`** — depending on the D.3 outcome:
   - **New lesson** → allocate the next `L-{NNN}` id (scan the file for the highest existing id and increment); append the entry with `kind: "escape"`, `status: "active"`, `first_seen: today`, `occurrences: [{feature, date}]`.
   - **Recurrence** → find the matched entry; append the new occurrence to its `occurrences` array. If the array now has ≥3 entries across distinct features, set a flag for the auto-promotion prompt at the end of D.4.
   - **Supersession** → set the matched entry's `status: "superseded"`, `superseded_by: L-{new-id}`. Append a new entry with `kind: "success"`, `status: "active"`, the user's three answers, the same tags, and `first_seen: today`.

2. **`features/NNN-{slug}/test/{NNN-slug}-test.md`** — append a new test case under "## Test cases":
   - Title: `TC-{N+1} — {short title derived verbatim from Q1 — no rephrasing}`.
   - `Verifies:` the AC the lesson maps to (ask the user if not obvious from the user's answers — e.g. "Which AC does this gap relate to?").
   - `Retroactive: true` — feedback-captured test cases describe shipped behaviour, so Phase C will skip execution. The case lives in the plan as the lesson-to-AC audit anchor.
   - `Source: lesson L-{NNN}` (in addition to the AC).
   - Pass criterion: derived **directly from Q3** — make it binary; if the user's Q3 answer isn't binary on its face, prompt: *"Express Q3 as a binary pass criterion — what specific check decides pass/fail?"*
   - Artefact path: `../assets/TC-{N+1}-feedback-L-{NNN}.{png|txt}`.
   - Add the new TC row to the coverage matrix table.

3. **`admin/task-history.json`** — append:
   ```json
   {
     "kind": "escaped-issue",
     "feature": "NNN-slug",
     "lesson_id": "L-{NNN}",
     "attribution": "{user's Q2 answer verbatim}",
     "captured_at": "YYYY-MM-DD"
   }
   ```

If `lessons.json` writes had occurrences ≥ 3 from D.4 step 1, prompt at the end:

- **Runnable promotion (per 035-self-learning-loop v2.16.0):** if `test_fragment.runnable == true`, prompt *"L-{NNN} occurred {n} times AND has a runnable check (`{assertion}`). Promote to CI gate and/or guideline? [ci/rule/both/no]"*. On `ci` or `both`: write the standard `skills/misc/SKILL.md` auto payload (trigger=`rule-violation`, calling_skill=`specflow:test`, title=`Wire L-NNN check into CI: {assertion}`, scope inferred from `fragment.scope`, priority=`P1`, description + verification) to `admin/scratch/misc-payload-{ts}.json` and invoke `specflow:misc --auto admin/scratch/misc-payload-{ts}.json`. Set `promoted_to_ci: "misc-task MISC-NNN"` on the lesson; `status` stays `active`. On `rule` or `both` also draft the guideline (existing path).
- **Non-runnable promotion:** if `test_fragment.runnable == false` (or absent on pre-2.16.0 lessons), prompt the legacy *"L-{NNN} has now occurred 3 times across distinct features. Promote to admin/rules/guidelines.md? [yes/no]"*. On yes: draft a one-paragraph rule from the lesson's title + remediation, present for user edit, append to `guidelines.md` with `(promoted from L-{NNN})` back-reference, flip the lesson's `status` to `promoted-to-rule` with `promoted_to_rule` set to the rule anchor.

The runnable path is the L-001 / L-004 fix: prose guidelines are read-not-enforced; a CI gate is enforced.

### D.4.5 Auto-fire `specflow:learn --feature {NNN-slug}`

After D.4's three writes land cleanly (lessons.json + test plan + task-history.json), invoke `specflow:learn --feature {NNN-slug}` as a forked sub-skill so the new lesson clusters into the corpus and any Tier-A rule promotions / CI promotions auto-apply within the same flow. The sub-skill reads `admin/lessons.json` (the single corpus — per 035-self-learning-loop v2.16.0 the `plugin-findings.jsonl` reference is gone) and operates over the whole append-only history.

The user never invokes `:learn` directly under normal workflow — it's a background loop wired here. The forked invocation is fire-and-forget for the parent skill: if `:learn` reports nothing actionable, no chat message lands; if it auto-applies a Tier-A rule or promotes to CI via `specflow:misc --auto`, `:learn` surfaces its own one-line summary. On `:learn` invocation failure (lock contention, schema gap), log to chat `[test: :learn auto-fire failed — {reason}; corpus retains the lesson]` and continue D.5 — the lesson capture itself is not blocked.

### D.5 Surface to the user

```
Feedback captured as L-{NNN}.
- New test case TC-{N+1} added to features/NNN-{slug}/test/{NNN-slug}-test.md (marked Retroactive: true).
- Attribution recorded in admin/task-history.json.
- Tags: [...]

The new TC is `Retroactive: true`, so Phase C won't execute it — it documents shipped behaviour. Future feature plans will reference this lesson via the lessons registry's query algorithm.
{If promotion prompt fired and was accepted:}
This lesson has been promoted to admin/rules/guidelines.md — future PRD/task generation will reference it as a non-negotiable on relevant features.
```

### D.6 Verify

1. `lessons.json` parses; new entry has all required fields populated; status/kind values are from the controlled set.
2. New test case in `{slug}-test.md` references the lesson id in its `Source:` line and the AC in its `Verifies:` line; pass criterion is binary.
3. `task-history.json` entry references the lesson id and includes the user's Q2 verbatim.
4. `lessons.json.bak` exists.
5. Tags on the new entry are all in the controlled vocabulary (no free-form tags).

---

## Lessons registry — schema, query, supersession

Specflow's self-learning corpus lives at `docs/specflow/admin/lessons.json` — a single mutable JSON array of lesson entries. Lessons are written by `specflow:test` Phase D (entered via the post-green prompt or Phase A.0 intent detection) and read by every skill that synthesises tasks or test plans for a new feature (`specflow:task`, `specflow:test` Phase B.0, and Phase 2 `specflow:develop`). The corpus answers two questions on every relevant invocation: *"What have we tried that didn't work?"* and *"What have we figured out since?"*

### Entry shape

```json
{
  "id": "L-001",
  "kind": "escape",
  "title": "Splash screen wrong font + wrong loading state",
  "tags": ["mobile", "react-native", "expo", "ui", "splash-screen"],
  "what_was_missed": "Splash screen used the wrong font and showed the logo image as the initial loading state instead of a blank background.",
  "what_should_have_caught_it": "specflow:test plan should have included a visual-diff test case against the design mockup.",
  "remediation": "UI tasks must include a Playwright screenshot diff against the proposed.html mockup before Verifier passes.",
  "test_fragment": {
    "kind": "ci-check",
    "assertion": "rg -n --pcre2 '#[0-9a-fA-F]{6}' apps/expo -g '*.tsx'",
    "scope": "apps/expo/**",
    "expect": "zero matches",
    "runnable": true
  },
  "status": "active",
  "superseded_by": null,
  "promoted_to_rule": null,
  "first_seen": "2026-05-06",
  "occurrences": [
    {"feature": "012-splash-screen", "date": "2026-05-06", "task": "T-3"}
  ]
}
```

### Field semantics

- `id` — `L-{NNN}`, monotonically incrementing across the project. Allocated when the lesson is written.
- `kind`:
  - `escape` — something passed Gate 3 / was marked done, but turned out wrong on human review.
  - `success` — a non-obvious approach that worked; recorded so the system doesn't re-derive it from scratch. Often paired with a superseded `escape` (the entry that previously said "this didn't work").
  - `rule-candidate` — a recurring pattern flagged for promotion to `admin/rules/`. Set by the auto-promotion prompt; the lesson stays in this kind until the user accepts/declines promotion.
- `tags` — controlled vocabulary (see "Tag vocabulary" below). Free-form tags are rejected at write time.
- `what_was_missed` — required when `kind: "escape"`. Plain language, captured verbatim from the user. The skill never paraphrases.
- `what_should_have_caught_it` — required when `kind: "escape"`. Names the gate / skill / agent that should have caught it, ideally with file:line if known. This field is the lever for targeted system improvement.
- `remediation` — required for all kinds. Concrete change description. Used as the seed for the test case the skill writes back into the test plan.
- `test_fragment` — required for every NEW escape lesson written from v2.16.0 onward. Sub-fields:
  - `kind` — one of `grep | testcase | ci-check`. Determines how the assertion is materialised in a future plan.
  - `assertion` — verbatim copy-pasteable line. For `grep`: the `grep` / `rg` command. For `ci-check`: the full CI command (shell-runnable). For `testcase`: the test-case skeleton (a function name + body the runner can execute).
  - `scope` — glob (e.g. `apps/expo/**`) the assertion applies to. Used by the REQUIRED-check to decide whether a future plan's task Scope overlaps.
  - `expect` — binary pass phrase (e.g. `zero matches`, `exit 0`, `assertion passes`).
  - `runnable` — `true` if the assertion can be executed verbatim by CI / a runner; `false` if it's a manual gate or human-judged check.
  Pre-2.16.0 lessons with no `test_fragment` read as `runnable: false` and Phase B falls back to prose remediation (backward-compatible).
- `status`:
  - `active` — the lesson is live and queried on matching features.
  - `superseded` — a later lesson replaces this one. `superseded_by: L-NNN` points to the replacement.
  - `resolved` — the remediation was validated by a later feature without recurrence (set by the resolution prompt at the end of a successful `full`-mode run on a tag-overlapping feature).
  - `promoted-to-rule` — the lesson has been promoted to `admin/rules/guidelines.md`. `promoted_to_rule: "{anchor-or-path}"`.

  When a lesson is promoted to CI (per 035-self-learning-loop v2.16.0), `status` stays `active` and the `promoted_to_ci` field carries the misc-task id. CI promotion is a wiring annotation, not a status transition.
- `superseded_by` — populated when D.3's similarity check identified a working approach for a prior failure. The user confirms supersession (audit-integrity preserved — system suggests, human signs off).
- `promoted_to_rule` — populated when the lesson is promoted to a rule. Format: `"admin/rules/guidelines.md#{anchor}"`.
- `first_seen` — `YYYY-MM-DD` of the first capture.
- `occurrences` — array; new entries appended when the same lesson is captured again on a new feature. The 3+ rule for promotion-to-rule reads `occurrences.length` across distinct `feature` values.

### Tag vocabulary

Tags are NOT free-form. They derive from three project sources to keep the registry queryable across a 12-month project lifecycle:

1. **Stack tags** — from `admin/environment.json`. Examples: `react-native`, `expo`, `nestjs`, `prisma`, `playwright`, `vitest`. Whatever the environment inventory detected.
2. **Surface tags** — controlled set, defined here as the canonical Phase 1 vocabulary: `ui`, `data-model`, `api`, `auth`, `migration`, `infra`, `cli`, `docs`. Adding a new surface tag requires editing this section of the test SKILL.md (treated as a vocabulary change).
3. **Domain tags** — from `admin/profiles.json` audiences and `admin/rules/glossary.md`. Examples (project-specific): `field-tech`, `inspector`, `splash-screen`, `chamber`. New domain tags must first be added to `glossary.md`; then they become available everywhere.

When Phase D writes a new lesson, the skill auto-suggests tags from these sources based on PRD frontmatter, the feature's slug, and a keyword scan of the user's three answers. The user accepts or edits. Free-form tags are rejected at write time with a remediation prompt (which source to add them to).

### Tag rot prevention

On every read, the skill validates each entry's `tags` against the live vocabulary. Unknown tags are surfaced as drift warnings:

```
Lessons registry drift: L-007 references unknown tag "react-native-mobile" (stack vocabulary has "react-native"; "expo"). Reads continue but consider editing L-007 to canonical tags.
```

Drift warnings don't block reads — just flag for cleanup at the user's convenience.

### Query algorithm

When `specflow:test` Phase B.0, `specflow:task` (entry phase), or any future skill needs to surface relevant lessons:

1. **Build the query tag set.** Read the feature's PRD frontmatter for stack/domain tags; read the parent folder name for the slug; detect the surface(s) the feature touches by scanning the PRD's Requirements for the canonical surface keywords (`UI`, `endpoint`, `migration`, `auth`, etc.).
2. **Filter.** Select entries where `status == "active"` (excluding `resolved`, `superseded`, `promoted-to-rule`) AND the entry's `tags` overlap the query tag set by ≥1 tag.
3. **Rank.** By `occurrences.length` desc (recurrence is a strong signal of relevance), then by `first_seen` desc (recency tiebreaker).
4. **Cap.** Surface up to 5 lessons in chat. If more match, summarise the count and write the full matched list to `admin/scratch/{slug}-{ts}/matched-lessons.json` for audit.
5. **Inject.** For `specflow:test` Phase B, each matched lesson becomes a constraint on the derived plan (B.1 must include a test case verifying the lesson's remediation). For `specflow:task`, the matched remediation is included in the task-generation prompt as a "must consider" line.

### Supersession workflow

Two paths into `superseded`:

**A. User confirms during D.3 (similarity check).** When capturing a new lesson, if the similarity score against an existing entry is above threshold, the user is asked whether the new capture is a recurrence, a supersession, or a distinct lesson. On supersession: the existing entry's `status` flips to `superseded` with `superseded_by: L-{new-id}`; the new lesson is written with `kind: "success"`.

**B. Auto-suggested on validated remediation.** When `specflow:test` runs `full` mode on a feature whose tags overlap an `active` `escape` lesson, AND every test case sourced from that lesson passes, the skill prompts at the end of Phase C: *"L-{NNN}'s remediation appears to have worked here (no recurrence). Mark resolved? [yes/no/skip]"*. On `yes`: the entry's `status` flips to `resolved`. The `kind` stays `escape`; only `status` and `occurrences` (with the validating feature added) update.

### Promotion to rule

When an `active` lesson's `occurrences.length` reaches 3 across distinct features, D.4 (or the resolution prompt) flags it for promotion. The user confirms; the skill drafts a rule from the lesson's `title` + `remediation`, presents for edit, appends to `admin/rules/guidelines.md` with a `(promoted from L-{NNN})` back-reference, and flips the lesson's `status` to `promoted-to-rule`. Promoted rules are read by `specflow:prd` (during synthesis) and `specflow:task` (during task derivation) on every future feature — the lesson graduates from "heads-up" to "non-negotiable on relevant features."

### Update discipline

`lessons.json` is mutable, but every write follows discipline:

- A `.bak` of `lessons.json` is written before each modification (mirrors the upgrade/scope-change backup pattern).
- Status changes (`active → superseded`, `active → resolved`, `active → promoted-to-rule`) preserve the entry — no entries are ever deleted. The corpus is the audit trail.
- On read, validate each entry's shape against the schema. Malformed entries are surfaced as drift warnings; reads continue with valid entries.
- Never edit an entry's historical fields (`what_was_missed`, `first_seen`, the original answers from D.2). Only `status`, `superseded_by`, `promoted_to_rule`, and `occurrences` mutate after first write.

---

## Reconcile manifest — schema, lifecycle

The reconcile manifest is the **living memory** of the interactive Phase C. It lives at `features/NNN-{slug}/test/{NNN-slug}-reconcile.json` and records every keep/reject decision the user has made, so the system tests against the confirmed truth and never re-asks a settled question. It is the difference between a suite that re-litigates every divergence on every run and one the user can steer once and trust thereafter.

### Entry shape

```json
{
  "feature": "021-initial-assessment-v3-...",
  "decisions": [
    {
      "id": "RC-001",
      "ac": "AC-12",
      "test_case": "TC-10",
      "kind": "confirmed-deviation",
      "spec_said": "Section 9 completeness requires exactly 2 materials",
      "build_does": "accepts 1 or more materials",
      "confirmed_expectation": "at least 1 material is correct — exactly-2 was dropped during develop",
      "tier": "soft",
      "approved_by": "{user}",
      "approved_at": "2026-06-04"
    },
    {
      "id": "RC-002",
      "ac": "AC-8",
      "test_case": "TC-14",
      "kind": "confirmed-fail",
      "spec_said": "all 12 legacy submit endpoints return 410",
      "build_does": "endpoints still return 2xx",
      "tier": "hard",
      "approved_to_flag_by": "{user}",
      "approved_at": "2026-06-04"
    }
  ]
}
```

### Field semantics

- `kind`:
  - `confirmed-deviation` — the user confirmed the build is correct and the spec was stale. `confirmed_expectation` (verbatim user words) becomes the behaviour future runs test against. Always soft tier — a hard contract change never lands here; it routes through `specflow:scope-change`.
  - `confirmed-fail` — the user confirmed the case is a genuine failure and approved flagging it. Future runs auto-flag it (status `❌ fail (known)`) without re-asking permission, until the bug is fixed and the case passes (the entry is then pruned on the next green run — see lifecycle).
- `confirmed_expectation` — required for `confirmed-deviation`; the user's own words for the new truth. This is what C.2's manifest consult evaluates against.
- `approved_by` / `approved_to_flag_by` — who signed off. The manifest is an audit trail; decisions are never anonymous. A value of `approved_by: "auto (cosmetic)"` marks a divergence the system auto-reconciled as purely cosmetic (no user prompt); these are collected in the run's auto-reconciled digest (C.4 report) so the user can audit and override any of them.
- `tier` — carried from the test case for traceability.

### Lifecycle + discipline

- The manifest is created lazily on the first reconcile decision for a feature; absent file = no decisions yet.
- A `.bak` is written before each modification (mirrors the lessons.json discipline).
- **Consulted at the top of Phase C (C.2)** before any case is judged: a `confirmed-deviation` re-points the pass criterion to `confirmed_expectation`; a `confirmed-fail` auto-flags without re-asking.
- **Self-healing:** when a case that has a `confirmed-fail` entry PASSES on a later run (the bug was fixed), prune that entry on the green run and note `reconcile: RC-NNN resolved (now passing)` in the run log. `confirmed-deviation` entries persist — they are the new spec truth, not a temporary state.
- A `confirmed-deviation` is the lightweight in-band record; the authoritative PRD AC is only updated when the user runs `specflow:scope-change`. The manifest keeps tests honest in the meantime so the user is never forced into doc admin mid-develop to get a clean run. Surface the stale-AC reminder once per reconciled AC per run (not nagging) so the eventual scope-change stays visible without blocking.

## What you MUST NOT do

- **Do not skip the chain check.** Tasks that haven't closed Gate 3 are not finished; testing them is premature.
- **Do not claim "tests pass" without an artefact.** Every pass row in the execution log must reference a concrete artefact (screenshot, runner output, log line). "It worked" without an artefact is a fabricated pass.
- **Do not write soft pass criteria.** Every pass criterion is binary. The Goal-Driven Reviewer would flag soft criteria at Gate 6 (Phase 3); flag them now and pre-empt.
- **Do not overwrite the execution log.** Append-only — past runs are the audit trail.
- **Do not invent ACs.** If a test case doesn't trace to a PRD AC, surface it as a `specflow:scope-change` candidate; do not invent an AC inline.
- **Do not invoke `specflow:scope-change` automatically.** PRD changes are user-driven decisions.
- **Do not auto-pass or reconcile a hard-tier failure.** Hard contracts (data, auth, payments, migrations, idempotency, API shape, endpoint retirement) stop the run for the user and can only be flagged as a fail or routed to `specflow:scope-change`. A happy-with-the-screen approval never clears a data-integrity failure.
- **Do not auto-edit the PRD when a soft deviation is reconciled.** Record it in the reconcile manifest (in-band, no admin) and surface the optional `specflow:scope-change` to sync the PRD AC. The PRD is user-owned; the manifest keeps the run honest without forcing a mid-develop doc edit.
- **Do not auto-reconcile a soft divergence that changes behaviour, validation, a threshold, a count, required/shown fields, gating, or data.** Only purely-cosmetic divergences (copy, spacing, colour tokens, icons, presentational ordering) auto-reconcile; anything that could change what the product *does* escalates to the user. When unsure which side a divergence falls on, escalate — ambiguity is never cosmetic.
- **Do not mention the underlying AI tooling or vendor** in any user-facing output, the test plan, the execution log, or the runner output cited in artefacts. Per the project's CLAUDE.md, this is non-negotiable.

---

## Verify before declaring done

Before returning to the user:

1. `features/NNN-{slug}/test/{NNN-slug}-test.md` exists with frontmatter, coverage matrix, test cases (every TC has an explicit `Retroactive:` field, defaulting to `false`), and (if execution ran) at least one Run entry in the execution log.
2. Forward coverage holds — every PRD AC appears in the matrix.
3. Reverse traceability holds — every test case cites at least one AC.
4. Every pass criterion is binary.
5. (If execution ran) every pass row in the latest Run cites a concrete artefact path that exists on disk.
6. (If execution ran) the failure list in the chat report matches the failures in the execution log.
7. (If Phase B.0 ran) every REQUIRED matched lesson has a covering case in the plan that embeds its `test_fragment.assertion` verbatim (and is tagged `Source: lesson L-NNN (REQUIRED)`); advisory ones are surfaced in chat or explicitly declined. A `done` with an uncovered REQUIRED lesson is a FAILED run.
8. (If Phase D ran) `lessons.json` validates against the schema; the new test case tagged with the lesson id is in the plan; `task-history.json` references the lesson id; `lessons.json.bak` exists.
9. (If Phase C ran interactively) every hard-tier failure was either flagged as a fail (with a `confirmed-fail` manifest entry + `escaped-issue` row) or routed to `specflow:scope-change` — none were silently passed; every soft reconcile decision is written to `{NNN-slug}-reconcile.json` with an approver and date; `{NNN-slug}-reconcile.json.bak` exists if the manifest was modified.

If any verify step fails, fix it before returning.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 15 — testing as cadence.
- `docs/PRD.md` Appendix G — test asset support.
- `docs/PRD.md` Appendix N1 (Gate 6) — Phase 3 multi-agent debate manifest for tests vs requirements; this skill pre-empts Gate 6 findings by enforcing binary pass criteria today.
- `templates/agents/standard/principles/goal-driven-reviewer.md` — primary lens for the test plan today; full Gate 6 manifest review lands in Phase 3.
- `skills/task/SKILL.md` — sister skill; same coverage-matrix discipline, different artefact (tasks vs tests). Reads `admin/lessons.json` on entry per the query algorithm above.
- `skills/develop/SKILL.md` (Phase 2) — primary consumer of `--task T{N}` filtering during the implementation loop.
- `skills/complete/SKILL.md` — prompts the user at retro time; any captured gap routes through `specflow:test {slug}` and the Phase A.0 intent detection prompt.
- `admin/lessons.json` — the project's self-learning corpus; schema and lifecycle defined in the **Lessons registry** section above.
