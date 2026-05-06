# Debate manifest — Gate 3: tasks vs PRD review

**Feature:** 002-develop-skill
**Artefact under review:** `002-develop-skill-tasks.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06 18:45

This is the dogfood Gate 3 run for the Phase 2 `specflow:develop` task list. The tasks file was synthesised against an 18-requirement PRD (R1-R17 plus R5.1) where two of the requirements (R5.1 and R17) and one of the AC-extensions (the team-spawn present-but-failing clause traced via T8) trace to Gate 2 block- and concern-resolutions. The Gate 3 reviewers fired against the coverage matrix, the per-task acceptance binaries, the Gate-2-revision separation, and the cross-task AC dependencies. Five `concern` findings landed; one `push_back` defended; the rest accepted.

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
- **goal-r1-f1** — *concern* — T16's `specflow:budget` confirmation clause uses 'confirms' rather than a binary check on the report's numeric fields. (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — AC-7 has two halves spread across T10 (Reviewers line) and T11 (Codex-Gate-4 refusal); the cross-task AC dependency was not surfaced in the coverage matrix or in T11's acceptance. (Same file.)
- **simplicity-r1-f1** — *concern* — T5 and T5b track one shippable surface (the lane-recheck pipeline) split across two task entries. (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — T13's schema-dependency Notes line surfaces the AC-10 cross-skill dependency at design time but the runtime branch (what the skill does when the schema is absent) was unspecified. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *concern* — T7's Depends-on `T6, T10, T12` was ambiguous between unconditional and per-task-conditional invocation of Gate 4 + Gate 5. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **da-r1-f1** — *concern* — T8's failure-fallback clause + T9's Red-lane refusal had an unhandled cross-product (Red-lane + plugin-present + team-spawn-failure-detection) that was implementer-dependent. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- goal-r1-f1 → **accept** (sharpened T16 verifier clause to specify the JSON report schema with two named numeric fields and the binary failure rule)
- goal-r1-f2 → **accept** (sharpened T11 acceptance to bind to observable behaviour; added AC-7-requires-T10-AND-T11 cross-reference in T11 Notes)
- simplicity-r1-f1 → **push_back** (cited Gate 2 block tbc-r1-f1 — the R5/R5.1 split was the load-bearing addition that made AC-8 fire; merging T5b into T5 would re-collapse what Gate 2 deliberately separated, breaking the 1:1 R→T trace and losing the load-bearing block-resolution from Gate 2)
- surgical-r1-f1 → **accept** (added runtime-branch clause to T13 acceptance: schema-gap → skip-with-warning + manifest record `auto_promotion_skipped: schema_gap`)
- tbc-r1-f1 → **accept** (sharpened T7 Depends-on to 'T6 always; T10 and T12 invoked per-task inside the batch loop'; added Notes line documenting per-task signal flow)
- da-r1-f1 → **accept** (added explicit `(NOT Red)` exclusion to T8's condition prefix; added T9 Notes clarification on T8 short-circuit for Red-lane)

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- goal-r1-f1 → **accept** (revision applied; T16 verifier clause now binary on the two named numeric fields)
- goal-r1-f2 → **accept** (revision applied; cross-task AC dependency surfaced at task layer)
- simplicity-r1-f1 → **accept** (AI's defence held; Gate 2 block tbc-r1-f1 outcome cited; trace integrity at the task level holds — accepting the merge would have been simplicity-overcorrection at the task layer for a property Gate 2 had already weighed and decided against)
- surgical-r1-f1 → **accept** (revision applied; loud-fallback shape matches Gate 2 da-r1-f1 precedent)
- tbc-r1-f1 → **accept** (revision applied; conditional ordering and per-task signal flow now stated)
- da-r1-f1 → **accept** (revision applied; cross-product (Red-lane + plugin-present + team-spawn-failure) case pinned to T9's single-specialist-only branch)

No sharpening occurred. No `ai-revision.md` needed.

---

## Closing decision

**Gate 3 status: passed-with-revisions**

Five of six findings were accepted by the AI and revisions applied to `002-develop-skill-tasks.md`:
- T16 acceptance verifier clause sharpened: `specflow:budget --skill specflow:develop {NNN-slug}` now produces a JSON report with two named numeric fields (`parent_context_tokens: N`, `max_per_gate_growth_tokens: M`) and the binary failure rule is `N >= 30000 OR M >= 2000`.
- T11 acceptance sharpened: setting `config.json.develop.codexAtGate4` produces no observable change in the Gate 4 manifest `Reviewers:` line (verified by re-running T10 acceptance with the field set). T11 Notes adds the AC-7-requires-T10-AND-T11 cross-reference.
- T13 acceptance gains a runtime-branch clause: when `specflow:misc --auto`'s schema does not accept `manifest_path` and `gate_finding_id`, the skill skips the auto-promotion + surfaces a chat-line warning + records `auto_promotion_skipped: schema_gap` in the Gate 5 manifest.
- T7 Depends-on sharpened to 'T6 always; T10 and T12 invoked per-task inside the batch loop'; T7 Notes documents the per-task signal flow (read manifest closing-decision after each gate, pause on first failure).
- T8 condition prefix gains explicit `(NOT Red)` exclusion; T9 Notes adds the short-circuit clarification pinning the cross-product (Red-lane + plugin-present) case to single-specialist-only.

One finding had a defended push-back accepted in Round 3:
- **simplicity-r1-f1** (T5 / T5b merge) — Gate 2's tbc-r1-f1 deliberately split R5 (reviewer-driven catch-all) and R5.1 (mechanical pre-Gate-4 recheck) into two requirements; the Gate 2 manifest's PRD-revisions-applied section names this split as the load-bearing addition that made AC-8 fire in practice. The 1:1 R→T trace carries to the task layer: R5 → T5, R5.1 → T5b. Merging the tasks would create a multi-anchor task spanning two requirements that Gate 2 said must remain distinct, and would lose the trace from R5.1 → T5b. Simplicity at the task level cannot override traceability anchored in a load-bearing block-resolution at Gate 2. The Round-3 reviewer accepted: keep the split.

No findings escalated to human decision.

The tasks file is fit to proceed to `specflow:develop` Phase 2 (this very skill, in production) or to `specflow:test` for verification cadence. No revisions to the PRD or interview were required (no scope-change triggered).

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 3 reviewers and humans reading this as a **dogfood example** alongside `001-design-skill/debate-log/tasks-gate3/manifest.md`:

- **Zero `block` findings on this run** is plausible because (a) the PRD had already been hand-iterated through Gate 2 with two `block` findings resolved, so the requirements were pre-tightened; (b) the Phase B.4 self-check was applied before Gate 3 fired; (c) the deliberate R5/R5.1 split forced an obvious 1:1 R→T mapping that Goal-Driven could verify mechanically. Real first-pass Gate 3 runs typically surface 1-2 `block` findings — orphan ACs (Goal-Driven auto-blocks), tasks with no binary acceptance check (Goal-Driven auto-blocks), or tasks not tracing to any requirement (Surgical auto-blocks). None of those tripped here.
- **Each reviewer fired its lens distinctly.** Goal-Driven flagged a soft verifier and a multi-half-AC coverage gap; Simplicity flagged the R5/R5.1 split (the dogfood tell); Surgical flagged a runtime-branch gap on a cross-skill schema dependency; Think-Before-Coding flagged unstated conditional ordering; Devil's Advocate flagged a cross-product branch (Red-lane + plugin-present + failure). No two reviewers flagged the same surface.
- **The defended push-back is the most architecturally interesting outcome.** Simplicity's instinct to merge T5b into T5 was correct *at the task layer* — the two tasks do touch one shippable surface. But Gate 2 already weighed this trade-off at the requirement layer and chose separation; accepting Simplicity's merge would have re-collapsed Gate 2's resolution. The push-back cites Gate 2's manifest as the source of authority — a downstream gate cannot unwind an upstream gate's load-bearing block-resolution. This is the cross-gate consistency property at work: Gate 3 verifies the tasks didn't reintroduce ambiguities Gate 2 sharpened, AND Gate 3 cannot unwind Gate 2's deliberate splits via simplicity arguments alone.
- **Goal-Driven was load-bearing here** (2 of 6 findings, both `concern`). Both were soft-verifier flags — one on T16's `specflow:budget` confirmation verb, one on the multi-half-AC coverage at AC-7. The pattern: any time a task acceptance reads 'X confirms Y' or 'verifies Z' without naming the observable signal, Goal-Driven bites.
- **Cross-gate consistency check** — at least one Gate 3 finding (here `da-r1-f1`) tested whether Gate 2's sharpening of R8 (present-but-failing path for agent-teams) was carried forward into the task acceptances + the Red-lane interaction. The Gate 2 da-r1-f1 finding had added a present-but-failing clause to R8; Gate 3's da-r1-f1 spotted that the task-level cross-product with R9 (Red-lane never spawns) was implementer-dependent. This is the cross-gate consistency lens at its best: previous gates fixed an artefact and this gate verifies the next artefact didn't reintroduce ambiguity at the boundary.
- **Healthy Gate 3 has 5-7 `concern` findings, 0-1 `block`s, 1-2 push-backs.** This run lands at 6 concerns / 0 blocks / 1 push-back — within the calibration band. A run with 8+ findings would suggest the Phase B.4 self-check missed; a run with 0 findings would suggest the reviewers rubber-stamped.
