# Debate manifest — Gate 3: tasks vs PRD review

**Feature:** 003-complete-skill
**Artefact under review:** `003-complete-skill-tasks.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06

This is the dogfood Gate 3 run for the Phase 3 `specflow:complete` task list. The tasks file was synthesised against a 14-requirement PRD (R1-R14, no sub-requirements) where two of the requirements (R4's cross-skill schema dependency and R14's lock-file guard) carry Gate 2 finding revisions in their trace lines. The task list has 15 tasks: one per R, plus T15 covering AC-15 / Goal Outcome surfaces (c)+(d) — the chat-line summary cross-cutting contract. The Gate 3 reviewers fired against the coverage matrix, the per-task acceptance binaries, the Gate-2-revision separation, the cross-task AC dependencies, and the goal-anchored AC trace. Six `concern` findings landed; one `push_back` defended; the rest accepted.

---

## Round 1 — Findings

| Reviewer | Findings (severity) |
|---|---|
| simplicity-reviewer | 1 (concern) |
| surgical-reviewer | 1 (concern) |
| think-before-coding-reviewer | 1 (concern) |
| goal-driven-reviewer | 2 (concern, concern) |
| devils-advocate | 1 (concern) |
| **Total** | **6 findings (0 block, 6 concern)** |

Detail:
- **goal-r1-f1** — *concern* — T15 anchors to AC-15 + goal Outcome surfaces (c)/(d) but the coverage-matrix row reads as if AC-15 traces to no R; reverse-traceability ambiguity at the matrix layer between goal-anchored AC and unstated-contract AC. (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — T9's existence-check uses the three-retro-only-fields-populated heuristic, which is observationally indistinguishable from the retro-time defaults (`actual_hours: 0`, `regressions_caught: {count:0, descriptions:[]}`, `escaped_issues: {count:0, descriptions:[], blast_radii:[]}`) on a clean-pass-zero-everything task; binary boundary failure. (Same file.)
- **simplicity-r1-f1** — *concern* — T15 tracks a cross-cutting emit-on-exit contract spanning seven dependent tasks; possible over-decomposition. (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — T4's Scope line names `skills/develop/SKILL.md` Phase F.5 as a code surface, which PRD R4 explicitly defers to a separate `specflow:develop` enhancement PRD; cross-skill scope drift. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *concern* — T13's branch (c) (task dropped by scope-change → retro skipped) embeds an implementation assumption not explicit in PRD R13/AC-13. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **da-r1-f1** — *concern* — T10's amend path and T12's unconditional-elevation-prompt collide on the (accept-and-ship + amend with kind:note) cross-product; firing logic implementer-dependent. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- goal-r1-f1 → **accept** (applied option (b): reshape T15's coverage-matrix row to explicitly document the goal-anchored exception; T15's Anchor line preserved — it already cites the goal Outcome surfaces by name)
- goal-r1-f2 → **accept** (sharpened T9 to bind 'existing retro' to the single boolean `superseded_by_retro: true`; values-that-look-like-defaults boundary closed)
- simplicity-r1-f1 → **push_back** (cited PRD line 135 — AC-15 verifies goal Outcome (c)+(d) directly as one cross-cutting contract; folding into seven per-task acceptances would split the verification surface and unwind goal-r1-f1's just-landed resolution; 002-develop-skill Gate 3 simplicity-r1-f1 precedent applies — simplicity at the task layer cannot override traceability anchored in a load-bearing cross-cutting contract)
- surgical-r1-f1 → **accept** (removed `skills/develop/SKILL.md` Phase F.5 surface from T4 Scope; added Notes-line clause naming the cross-skill prerequisite as out of scope; mirrors 002-develop-skill Gate 2 surgical-r1-f1 precedent)
- tbc-r1-f1 → **accept** (extended T13 Notes with the dropped-task implementation-choice articulation, naming the choice and rationale explicitly, leaving the PRD-revision pathway open)
- da-r1-f1 → **accept** (sharpened T10 acceptance with explicit accept-and-ship-interaction clause + T12 Notes with matching clarification; pinned T12's unconditional logic to once-per-original-retro-only firing)

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- goal-r1-f1 → **accept** (revision applied; coverage-matrix row makes goal-anchoring explicit; reverse traceability holds with the documented exception)
- goal-r1-f2 → **accept** (revision applied; T9 acceptance binary on a single boolean the retro write controls)
- simplicity-r1-f1 → **accept** (AI's defence held; cross-cutting contract with single-AC verification is not over-decomposition; precedent matches 002-develop-skill)
- surgical-r1-f1 → **accept** (revision applied; cross-skill enhancement surface removed from `specflow:complete`'s task layer; runtime-branch refusal is surgical-changes-compliant)
- tbc-r1-f1 → **accept** (revision applied; dropped-task assumption now explicit; PRD-revision pathway open)
- da-r1-f1 → **accept** (revision applied; cross-product (accept-and-ship + amend) pinned to deterministic firing logic)

No sharpening occurred. No `ai-revision.md` needed.

---

## PRD/tasks revisions applied

The tasks file was edited between Round 1 and Round 3 to incorporate every accepted finding:

1. **T15 coverage-matrix row reshape (concern goal-r1-f1).** T15's coverage-matrix row now reads `AC-15 / Goal Outcome surfaces (c)+(d) — anchored to goal Outcome (c) chat-line summary citing the new entry's id + (d) zero silent failures, not to an R; documented exception per Gate 3 finding goal-r1-f1`. The matrix reader can now distinguish a goal-anchored AC from an unstated-contract AC; reverse traceability under E4 holds with the documented exception.

2. **T9 acceptance sharpened to single-boolean existence signal (concern goal-r1-f2).** T9's acceptance now binds 'existing retro' to `superseded_by_retro: true` (the supersede flag set only by T4's supersede-write), avoiding the values-that-look-like-defaults boundary on clean-pass-zero-everything tasks. T9 Depends-on extended to T4; T9 Notes line documents the binary signal. Carries `gate3-revision: goal-r1-f2`.

3. **T4 Scope line tightened + cross-skill prerequisite Notes clause (concern surgical-r1-f1).** T4's Scope line no longer names `skills/develop/SKILL.md` Phase F.5 as a code surface; the cross-skill enhancement-PRD prerequisite is now confined to a runtime detection check inside `specflow:complete`. T4 Notes line carries the cross-skill prerequisite clause directing the change to a separate `specflow:develop` enhancement PRD per PRD R4. Carries both `gate2-revision: surgical-r1-f1` (from PRD-level finding) and `gate3-revision: surgical-r1-f1` (from this gate's task-level scope tightening).

4. **T13 Notes line extension articulating the dropped-task assumption (concern tbc-r1-f1).** T13 Notes line now carries the explicit articulation of branch (c) as `retro skipped + structured failure line` rather than `stub retro recording the drop`, with rationale (a dropped task did not close; `specflow:complete` captures closure-time lessons). PRD-revision pathway named in case the assumption breaks. Carries `gate3-revision: tbc-r1-f1`.

5. **T10 acceptance accept-and-ship-interaction clause + T12 Notes matching clarification (concern da-r1-f1).** T10 acceptance now carries the explicit cross-product clause: on amend invocations where the existing entry has `accepted_with_failure: true`, T12's unconditional-elevation-prompt does NOT re-fire; subsequent amends follow R6's two-condition rule. T12 Notes carries the matching clarification: T12's unconditional logic fires exactly once per retro entry — at original retro-write time. Both tasks carry `gate3-revision: da-r1-f1`.

## Findings rejected after Round 3

One finding had a push-back outcome the AI defended successfully:

- **simplicity-r1-f1 (push-back on dropping T15).** The chat-line summary contract IS one cross-cutting emit-on-exit rule that AC-15 verifies as a single load-bearing surface (PRD line 135 names AC-15 as verifying goal Outcome (c) chat-line summary citing the new entry's id + goal Outcome (d) zero silent failures). Folding the contract into seven per-task acceptances would split the verification across seven tasks AND lose the cross-cutting surface — a future change to the summary line format would have to be applied seven times AND a Goal-Driven re-review would have to re-verify each task individually. The 002-develop-skill Gate 3 simplicity-r1-f1 precedent (T5/T5b kept distinct because Gate 2 deliberately split R5/R5.1) bites in the same direction here: simplicity at the task layer cannot override traceability anchored in a load-bearing cross-cutting contract. Goal-Driven's goal-r1-f1 acceptance just sharpened T15's coverage-matrix row to make the goal-anchoring explicit; dropping T15 would unwind that resolution. Round-3 simplicity-reviewer accepted: principle bites on bookkeeping that adds no shippable surface, releases on cross-cutting contracts with a single-AC verification.

## Findings escalated to human

None. All six findings converged within three rounds.

## Closing decision

**Gate 3 status: passed-with-revisions**

Five of six findings were accepted by the AI and revisions applied to `003-complete-skill-tasks.md` (T15 coverage-matrix row reshape, T9 single-boolean existence signal, T4 cross-skill scope tightening + Notes clause, T13 dropped-task assumption articulation, T10 + T12 cross-product clauses). One finding (simplicity-r1-f1) had a defended push-back accepted in Round 3, citing PRD line 135's AC-15 verification of goal Outcome (c)+(d) as a single load-bearing cross-cutting contract and the 002-develop-skill Gate 3 precedent for keeping cross-cutting tasks distinct. No findings escalated to human decision. The tasks file is fit to proceed to `specflow:develop` Phase 2 (the Phase 2 implementation orchestrator) or to `specflow:test` for verification cadence. No revisions to the PRD or interview were required (no scope-change triggered).

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 3 reviewers and humans reading this as a **dogfood example** alongside `002-develop-skill/debate-log/tasks-gate3/manifest.md`:

- **Six concerns / zero blocks / one push-back is the right shape for a Phase 3 retro skill** whose PRD already passed Gate 2 with revisions. The PRD's pre-tightening from Gate 2 (two block findings resolved + five concerns) meant the requirements were sharp enough that no Gate 3 finding earned a `block` severity — every finding fired on a task-layer ambiguity (coverage-matrix row shape, single-boolean vs heuristic existence-check, cross-skill scope drift, unstated dropped-task assumption, cross-product collision) rather than a missing R or orphan task. The push-back lands where it should — on a simplicity-vs-traceability trade-off the cross-cutting contract anchors definitively.
- **Goal-Driven was load-bearing here (2 of 6 findings, both `concern`).** Both were on traceability boundaries: goal-r1-f1 on the goal-anchored AC (E4 reverse traceability with documented exception); goal-r1-f2 on the binary boundary between populated-zero-values and never-written-defaults. The pattern matches the role file's "particularly load-bearing at Gate 3" framing — coverage-matrix integrity AND binary verification surfaces are the two Goal-Driven primary lenses, and both fired here.
- **The push-back precedent generalises across Gate 3 runs.** The 002-develop-skill push-back on simplicity-r1-f1 (T5/T5b kept distinct) and this Gate's push-back on simplicity-r1-f1 (T15 kept distinct) share the same shape: simplicity instinct to merge tasks at the task layer, defended by citing a load-bearing trace anchor (Gate 2 R5/R5.1 split there; goal Outcome (c)+(d) cross-cutting contract here). The pattern is durable: cross-cutting AC contracts and Gate-2-deliberately-split R-sets both warrant distinct tasks, and simplicity-driven merging at Gate 3 unwinds upstream resolutions. Two anchor patterns, same simplicity-vs-traceability calibration.
