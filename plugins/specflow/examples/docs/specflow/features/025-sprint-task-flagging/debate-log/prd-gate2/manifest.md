# Gate 2 — PRD vs interview review

**Feature:** 025-sprint-task-flagging
**Date:** 2026-05-07
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate, codex

## Accepted findings

- **simplicity-r1-f1** (simplicity-reviewer, block) — R10's recompute-and-compare hand-edit detector was defensive plumbing for an impossible-by-construction scenario, AND the recompute would false-positive on legitimate scope-change delta-regeneration paths.
  - Revision applied: R10 enforcement DROPPED; AC-12 dropped. R10 retained as documentation-only (field is read-only at the doc level).

- **simplicity-r1-f2** (simplicity-reviewer, concern) — Two-rule formulation (topological floor + scope-overlap split) was clever where one fixpoint definition was more explicit.
  - Revision applied: R2 rewritten as a single-rule fixpoint definition: `bucket(T) = 1 if T has no predecessors and no same-bucket scope conflicts with an earlier-ID task; otherwise bucket(T) = 1 + max(bucket(P) for P in Depends-on(T) ∪ EarlierIDSameBucketScopeConflicts(T))`.

- **simplicity-r1-f3** (simplicity-reviewer, concern) — R9's topological-floor invariant was guaranteed by R2's rule construction.
  - Revision applied: R9 dropped. Topological floor noted as a one-line strict-less-than corollary inside R2 (`bucket(T_j) < bucket(T_i)` for every T_j in T_i.Depends-on, by construction).

- **simplicity-r1-f4** (simplicity-reviewer, note) — R7's verbatim-listing requirement was speculative phase-ordering specification.
  - Revision applied: R7 simplified to a behavioural property; AC-13 split into doc check + execution-order fixture (per codex-r1-f4 / goal-driven-r1-f3).

- **simplicity-r1-f5** (simplicity-reviewer, note) — Lexicographic-vs-natural-number disambiguation was a footgun.
  - Revision applied: R2 + AC-9 reworded — typed comparator over `(int, optional-letter)` per the canonical task ID grammar `T-<int>(\.<letter>)?`. AC-3 worked example demonstrates `T-2 < T-10`.

- **surgical-r1-f1** (surgical-reviewer, block) — R10's hand-edit-detection enforcement was never requested by the user; cross-skill blast radius into specflow:scope-change.
  - Revision applied: R10 + AC-11 + AC-12 dropped. Field read-only documented-only.

- **surgical-r1-f2, f3** (surgical-reviewer, concerns) — R7's verbatim-listing + R9's standalone topological-floor verify were beyond the interview's signed-off ask.
  - Revisions applied: R7 simplified to property; R9 dropped (per simplicity-r1-f3).

- **surgical-r1-f4** (surgical-reviewer, note) — AC-3's "or example fixture under examples/" alternative quietly expanded the change footprint.
  - Revision applied: AC-3 constrained to `templates/task/sprint-bucket-heuristic.md` (the doctrine doc). No examples/ alternative.

- **surgical-r1-f5** (surgical-reviewer, note) — R8's licence for prose tightening risked drive-by edits.
  - Revision applied: R8 reworded with chain-don't-absorb framing — additions limited to one field-emission line + two one-line citations; the heuristic itself extracted to `templates/task/sprint-bucket-heuristic.md` (mirrors 029's `templates/admin/single-context-task.md`).

- **tbc-r1-f1** (think-before-coding-reviewer, block) — DAG assumption was load-bearing but unstated.
  - Revision applied: Added R11 — pre-bucketing graph-validity step rejects cycles, self-loops, dangling refs, duplicate IDs (and **duplicate edges** per Round 3 codex-r3-f1) with verbatim `GRAPH-INVALID:` diagnostic. AC-14 verifies via fixtures.

- **tbc-r1-f2** (think-before-coding-reviewer, block) — Task ID format unstated; lex tie-break would break on `T-3.A` / `T-10`.
  - Revision applied: R2 + AC-9 specify canonical grammar `T-<int>(\.<letter>)?` and a typed comparator over `(int, optional-letter)`.

- **tbc-r1-f3** (think-before-coding-reviewer, concern) — Within-feature ID uniqueness assumption implicit.
  - Revision applied: R2 preamble states "Task IDs are unique within a feature; bucket assignment relies on within-feature ID uniqueness for the typed-comparator tie-break to be defined."

- **tbc-r1-f4** (think-before-coding-reviewer, concern) — 020 mapping is a hypothesis, not a contract.
  - Revision applied: R5 trace tagged as hypothesis: "Per the current 020 design hypothesis (FEATURES.md § 020 — open design)... If 020's mapping evolves, 025's feature-scoping should be re-examined."

- **tbc-r1-f5** (think-before-coding-reviewer, concern) — Bump iteration discipline unstated.
  - Revision applied: R2 extended with explicit iteration rule — bumps applied iteratively until fixed point; pass order is `(bucket-asc, T-id-asc, T-id-asc)`; termination guaranteed by strict bucket-number increase + topological-floor bound.

- **goal-driven-r1-f1** (goal-driven-reviewer, block) — task/SKILL.md eval field didn't exercise sprint-bucket.
  - Revision applied: Added R12 — eval extended with two clauses: (1) every task has `sprint-bucket: N` with `N>=1`; (2) strict-less-than topological floor holds for every dependency. AC-15 verifies.

- **goal-driven-r1-f2** (goal-driven-reviewer, block) — AC-9's grep `^sprint-bucket:` doesn't match the markdown bullet format.
  - Revision applied: AC-9 rewritten with markdown-aware extraction — Python regex over `### T-<id>` heading + `- **sprint-bucket:** N` bullet; pairs sorted by typed comparator; diff between two runs returns empty.

- **goal-driven-r1-f3** (goal-driven-reviewer, block) — AC-13 verified doc, not execution.
  - Revision applied: AC-13 split into doc check (heuristic doc lists ordering as property) + execution-order fixture (oversize task aborts before any sprint-bucket: N is written for that task).

- **goal-driven-r1-f4** (goal-driven-reviewer, block) — AC-12 hand-edit diagnostic under-specified. (Moot; AC-12 dropped.)

- **goal-driven-r1-f5** (goal-driven-reviewer, concern) — Edge cases (self-loop / cycle / duplicate / dangling) had no AC.
  - Revision applied: AC-14 covers four malformed-graph cases (now five with codex-r3 duplicate-edge addition) with verbatim `GRAPH-INVALID:` prefix.

- **goal-driven-r1-f6** (goal-driven-reviewer, concern) — AC-7 / AC-10 absence-of-text checks were soft.
  - Revision applied: AC-7 rewritten as positive structural assertion (no shared file under skills/ references both feature IDs alongside sprint-bucket); AC-10 rewritten with closed token list scoped to outside Non-goals call-out.

- **da-r1-f1** (devils-advocate, block) — R10 enforcement breaks scope-change delta-regeneration. (Moot; R10/AC-12 dropped.)

- **da-r1-f2** (devils-advocate, concern) — 022 doesn't actually own auto-tuning.
  - Revision applied: Non-goals reworded — "Auto-detecting the optimal number of buckets is out of scope; whether it lands in 022 or a future feature is a separate planning decision (FEATURES.md § 022 currently scopes 022 as task-merge / split / re-order, NOT bucket-count tuning)."

- **da-r1-f3** (devils-advocate, concern) — task/SKILL.md doesn't have a "Phase D verify-steps" section.
  - Revision applied: All "Phase D verify-steps" references replaced with "Phase B.4 self-check" (where 029's budget check already lives at line 208). Heuristic extracted to doctrine doc.

- **da-r1-f4** (devils-advocate, note) — 020 listed as a User leaks consumer concerns.
  - Revision applied: 020 moved to "Downstream consumers (informational)" note that explicitly disclaims contract authority.

- **da-r1-f5** (devils-advocate, note) — R8's 14-line headroom claim unrealistic.
  - Revision applied: Heuristic + worked example extracted to `templates/task/sprint-bucket-heuristic.md` per chain-don't-absorb. task/SKILL.md additions limited to one field-emission line + two one-line citations.

- **codex-r1-f1** (codex, block) — Graph-validity invariant unstated. (Same as tbc-r1-f1; resolved by R11/AC-14.)

- **codex-r1-f2** (codex, concern) — AC-9 needed canonical (task_id, bucket) pairs sorted by typed comparator. (Resolved by AC-9 markdown-aware extraction.)

- **codex-r1-f3** (codex, concern) — Lex vs natural-number disambiguation. (Same as simplicity-r1-f5.)

- **codex-r1-f4** (codex, concern) — AC-13 doc-only. (Same as goal-driven-r1-f3.)

- **codex-r1-f5** (codex, block) — Topological floor allowed `≤ N` (same-bucket dependencies); should be strict `< N`.
  - Revision applied: R2 corollary states strict-less-than (`bucket(T_j) < bucket(T_i)` by construction). R12 eval clause 2 mirrors with `M < N`. AC-15 verify uses strict less-than.

- **codex-r3-f1** (codex Round 3, sharpen) — R11 graph-validity missed duplicate dependency edges.
  - Revision applied: R11 extended with fifth subtype `GRAPH-INVALID: duplicate edge T-X -> T-Y`. AC-14 extended with duplicate-edge fixture invocation.

## Rejected findings

None — every Round-1 finding either accepted or made moot by the substantive Round-2 dropping of R10 enforcement (which cascaded across simplicity-r1-f1 / surgical-r1-f1 / da-r1-f1 / goal-driven-r1-f4).

## Escalated to human

None.

## Closing decision

Gate 2 status: **passed-with-revisions**

PRD revisions applied across all 31 Round-1 findings + 1 Round-3 sharpen. Strong convergence — every reviewer accepted the writer's defence by Round 3. Major substantive revisions: R10 enforcement dropped (avoids circular diagnostic with scope-change); R9 dropped (R2 guarantees by construction); single-rule fixpoint definition for R2; canonical task ID grammar `T-<int>(\.<letter>)?` with typed comparator; graph-validity step (R11/AC-14) covering five malformed-graph cases; eval field extended (R12/AC-15); markdown-aware AC-9 extraction; Phase B.4 (not Phase D) is the home for verify-steps; heuristic extracted to `templates/task/sprint-bucket-heuristic.md` (chain-don't-absorb).

PRD is fit to proceed to `specflow:task`. No human intervention required to clear Gate 2.

— Orchestrator, 2026-05-07
