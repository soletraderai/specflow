# Gate 2 — PRD vs interview review

**Feature:** 017-tdd-discipline
**Date:** 2026-05-07
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate, codex

## Accepted findings

- **simplicity-r1-f5** (simplicity-reviewer, note) — Refactor's "no new files" hard-block had no detection host inside D.3.
  - Revision applied: R11 demoted to documented-discipline + manifest outcome value `refactor: failed (new-file-attempted)`. AC-11 split into AC-11a (doctrine-doc grep) + AC-11b (worked fixture).

- **simplicity-r1-f2** (simplicity-reviewer, concern) — TDD section in CORE_PRINCIPLES.md duplicated develop/SKILL.md content.
  - Revision applied: Created `templates/admin/tdd-discipline.md` as the cycle's canonical home (mirrors 029's `templates/admin/single-context-task.md`). CORE_PRINCIPLES.md gets a one-paragraph (≤8 lines) entry citing the doctrine doc. develop/SKILL.md gets one-line citations per cycle reference.

- **surgical-r1-f1** (surgical-reviewer, block) — R9 stub-section in `{NNN-slug}-tasks.md` broadcasts markers into a published artefact every downstream skill reads.
  - Revision applied: Stub moved to `admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md` (one file per task). Tasks.md untouched.

- **surgical-r1-f4** (surgical-reviewer, concern) — AC-5's "or admin/config.json schema" alternative invited schema creep.
  - Revision applied: AC-5 tightened to develop/SKILL.md only, matching the existing greenBatchCap precedent.

- **surgical-r1-f5** (surgical-reviewer, note) — R8 needed an explicit edit-bound clause.
  - Revision applied: R8 amended — edits scoped to test/SKILL.md lines 39, 51, 214; Phase B's case-derivation logic and B.3's write template unchanged in shape.

- **TBC-r1-f1** (think-before-coding-reviewer, block) — 017→019 absorption contract was unstated.
  - Revision applied: Stub moved to scratch (per surgical-r1-f1); R9 reworded to declare 017 owns interim stub schema only, 019 owns canonical with migration adapter; Open Question OQ-1 surfaces the 017→019 contract resolution requirement before 019's PRD synthesis.

- **TBC-r1-f2** (think-before-coding-reviewer, block) — "Strong CI signal" was a phrase, not a definition.
  - Revision applied: R6 extended with operational definition — (a) every PR runs vitest + typecheck + lint as required checks; (b) merge blocked on red checks; (c) project enforces binary-AC convention. Operator self-attests; manifest records attestation alongside `red: skipped (config)`.

- **TBC-r1-f3** (think-before-coding-reviewer, block) — AC-8's grep for "manifest" false-positives on Gate 5's existing manifest.md references.
  - Revision applied: AC-8 rewritten as positive-input-list assertion — Phase E.3 reviewer-dispatch input list does NOT include the per-task manifest scratch or the cycle markers. Phase E reviewers see only code diff, per-task plan, tasks-file entry, role definition.

- **TBC-r1-f4** (think-before-coding-reviewer, concern) — AC-2's verification mechanism was ambiguous.
  - Revision applied: AC-2 amended — grep against develop/SKILL.md prose with 10-line proximity to a verb anchor (invoke / fire / run / execute).

- **TBC-r1-f5** (think-before-coding-reviewer, concern) — `--task T{N}` mode would fire test/SKILL.md Phase B.5's user prompt inside develop's Red sub-step.
  - Revision applied: R8 amended — `--task T{N}` mode skips Phase B.5; per-task slice inherits the feature-level B.5 outcome; Gate 5's Codex reviewer covers cross-provider concerns at code-vs-plan time.

- **gd-r1-f1, gd-r1-f2, gd-r1-f3, gd-r1-f4, gd-r1-f5** (goal-driven-reviewer, blocks/concerns) — AC binarity gaps across AC-1 (no order assertion), AC-3 (narrative refusal), AC-8 (false-positive grep), AC-11 (no fixture path), AC-2 (verb-anchor missing).
  - Revisions applied: AC-1 sort-by-line-number assertion; AC-3 two-part grep (Status: red (failing) + refuse + tddRequired conditional within 30 lines after Green sub-step); AC-8 input-list test (above); AC-11 split into 11a/11b with fixture file path; AC-2 verb-anchor proximity.

- **gd-r1-f6** (goal-driven-reviewer, block) — Mandatory eval-coverage. develop and test SKILL.md eval fields didn't exercise the TDD discipline.
  - Revision applied: Added R13 — both eval fields extended with manifest-stub regex + `Status: red (failing)` write check. AC-12 verifies the eval extensions.

- **devils-advocate-r1-f1** (devils-advocate, block) — D.1/D.2/D.3 numbering collision: existing develop/SKILL.md uses those headings for lane execution (Green/Yellow/Red).
  - Revision applied: Cycle steps RENAMED to "Red sub-step / Green sub-step / Refactor sub-step" (no D.X numbering). The cycle is internal to a task's execution within whatever lane it landed on. R1/R2/R3 + AC-1 wording uses cycle-step terminology, NOT phase-numbering.

- **devils-advocate-r1-f2** (devils-advocate, concern) — 029's chain-don't-absorb pattern not followed; cycle absorbed into develop/SKILL.md instead of doctrine doc.
  - Revision applied: Created `plugins/specflow/templates/admin/tdd-discipline.md` (cycle's canonical home). develop/SKILL.md adds three cycle-step citations + one-line invocation contracts; CORE_PRINCIPLES.md gets one-paragraph entry citing doctrine.

- **devils-advocate-r1-f3** (devils-advocate, block) — Stub in tasks.md broadcasts to every downstream skill.
  - Revision applied: Stub moved to admin/scratch/ (same as surgical-r1-f1).

- **devils-advocate-r1-f4** (devils-advocate, concern) — Blast radius (CORE_PRINCIPLES.md cache invalidation, schema coupling) under-acknowledged.
  - Revision applied: Vision now carries an explicit "Blast radius" note enumerating five files + scratch + doctrine + CORE_PRINCIPLES. Lane stays Yellow because: CORE_PRINCIPLES gets one paragraph (not 30 lines); doctrine doc absorbs cycle contract; Red lane bypasses TDD enforcement so PROTECTED_PATHS_REQUIRE_RED_LANE unaffected.

- **devils-advocate-r1-f5** (devils-advocate, concern) — R11's hard-block detection had no host.
  - Revision applied: R11 demoted to documented-discipline + manifest outcome value (same as simplicity-r1-f5).

- **codex-r1-f1** (codex, concern) — Red lane TDD opt-out (R7) creates protected-path tasks bypassing Red artefact entirely.
  - Revision applied: R7 extended — when AI assists with testable code in Red lane, EITHER human supplies Red artefact OR manifest records `red: skipped (human-led Red lane)` with no `green: passed` marker emitted.

- **codex-r1-f2** (codex, concern) — 017 pre-claimed canonical schema ownership for 019.
  - Revision applied: R9 reworded — 017 owns interim stub contract only; 019 owns canonical schema with migration adapter from stub.

- **codex-r1-f3** (codex, note) — AC-1 grep order assertion missing. (Same as gd-r1-f1.)

- **codex-r1-f4** (codex, block) — `Status: red (failing)` is a doc marker, not runner output. Pocock failure mode (test-after-implementation) is not actually prevented.
  - Revision applied: Added R14 + AC-13 — pre-implementation test command runs against pre-implementation state; failing exit + stderr captured in `admin/scratch/{NNN-slug}-develop/red-test-trace-{task-id}.log`. Green may flip Status: green (passing) only after same command exits 0. A pre-implementation pass blocks as invalid Red artefact.

## Rejected findings

- **simplicity-r1-f1** (simplicity-reviewer, concern) — `tddRequired` knob speculative configurability.
  - Reason for rejection: Interview Round 1 explicitly resolved as a configurable knob with default `true`; user-confirmed at sign-off. TBC-2's separate concern about "strong CI signal" being undefined was addressed by adding the operational definition (R6 extension). The reviewer's Round 3 sharpen ("defer to OQ") asks the orchestrator to walk back a user-confirmed interview decision; declined.

- **simplicity-r1-f3** (simplicity-reviewer, concern) — Three-line manifest schema over-spec.
  - Reason for rejection: Three lines are user-readable in retro at a glance; per-step ISO timestamp is the audit-grade signal R14 provides. Single-line collapse would push retro readers to a render layer the doctrine doc doesn't currently specify. Reviewer Round 3 maintained the sharpen; surfaced as escalation.

- **simplicity-r1-f4** (simplicity-reviewer, note) — Three worked-example fixtures duplicate.
  - Reason for rejection: Per Goal-Driven Reviewer's coverage-matrix mandate, both `skipped (config)` and `skipped (trivial)` outcomes need binary ACs (AC-7a, AC-7b). Reviewer Round 3 conceded conditionally on f1 resolving; if f1 escalates, f4 escalates with it.

- **surgical-r1-f2** (surgical-reviewer, concern) — TDD section in CORE_PRINCIPLES.md is adjacent-fix creep.
  - Reason for rejection: Original user request invoked Pocock framing — *"TDD should inherit Matt Pocock's Red, Green approach"*. Principle-level adoption is in scope. The verbatim cycle contract was moved to the doctrine doc per the chain-don't-absorb pattern (resolves the cross-file abstraction concern).

- **surgical-r1-f3** (surgical-reviewer, concern) — AC-7a/7b worked examples decorative.
  - Reason for rejection: Per Goal-Driven Reviewer's gd-r1-f6 finding, both skip outcomes must be exercised — examples ARE the binary check.

## Escalated to human

- **simplicity-r3-f1** (simplicity-reviewer, concern) — Defer the `tddRequired` knob until a concrete second consumer demands it.
  - Reason: reviewer and writer did not converge in 3 rounds; reviewer wants to walk back the user's interview Round 1 sign-off ("Green lane: configurable via `config.json.develop.tddRequired` (default `true`)"). Orchestrator declines to override interview-confirmed decisions in Gate 2.
  - Recommendation: stand on the user's interview sign-off. If the user wants to defer the knob, route through `specflow:scope-change`, not Gate 2.

- **simplicity-r3-f3** (simplicity-reviewer, concern) — Collapse the three-line manifest schema to a single line.
  - Reason: reviewer argues R14's pre-implementation test trace duplicates the per-step ISO timestamps; orchestrator argues three lines are user-readable in retro at a glance. Schema-readability tradeoff; reviewer and writer did not converge.
  - Recommendation: the user weighs the readability/storage tradeoff. If single-line is preferred, the doctrine doc's R9 section is the natural home for the change.

- **simplicity-r3-f4** (simplicity-reviewer, note) — Drop AC-7b. (Conditional on simplicity-r3-f1.)
  - Reason: convergent only if the knob escalation resolves toward defer. Surfaced as conditional escalation.
  - Recommendation: defer until simplicity-r3-f1 resolves.

## Closing decision

Gate 2 status: **passed-with-escalations**

PRD revisions applied across 22 of 28 Round-1 findings + Round-3 absorbed all convergent sharpens via the substantive Round-2 restructure (cycle steps renamed; doctrine doc created; stub moved to scratch; pre-implementation test execution added; eval extensions; protected-path Red artefact requirement; canonical schema ownership clarified). Three rejections (simplicity-r1-f1, simplicity-r1-f2, simplicity-r1-f3) are user-confirmed interview decisions; three Round-3 sharpens (simplicity-r3-f1, f3, f4) did not converge and challenge interview-confirmed decisions or substantive readability tradeoffs.

PRD is fit to proceed to `specflow:task` after the three simplicity-reviewer escalations are resolved by the human (decisions on tddRequired knob retention, three-line schema retention, AC-7b retention). All three are user-product-decisions, not technical defects.

— Orchestrator, 2026-05-07
