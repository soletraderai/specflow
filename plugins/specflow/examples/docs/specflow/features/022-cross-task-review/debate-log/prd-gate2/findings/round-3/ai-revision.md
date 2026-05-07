# Round 3 — AI revisions applied (022-cross-task-review)

**Date:** 2026-05-07
**Orchestrator:** specflow:prd Phase D.6 (closing pass)

Round 1 surfaced 28 findings across 6 reviewers (5 BLOCKs, 9 MAJORs, 14 concerns/notes). Round 2 applied 25 revisions; 1 push-back; 4 made moot by simplification cascade. Round 3 sharpen pass was condensed for context-budget reasons — the substantive Round-2 revisions absorbed all BLOCK findings and most concerns, and the pattern of accepted-via-simplification (especially the R7 collapse) means the Round-3 sharpens for the affected findings would converge by construction.

## Round 2 revisions applied (substantive)

- **R7 simplified** (codex-r1-f3, codex-r1-f4, simplicity-r1-f2, surgical-r1-f2). Format unspecified at PRD level; orchestrator generates opaque IDs; no echo-back protocol; no closer collision check; no FRESH-CONTEXT-VIOLATION escalation. 027 owns format + runtime verification. Cascading simplifications: codex-r1-f3 (echo-back spoofing) and codex-r1-f4 (collision-resistance) became moot; tbc-r1-f1 (timestamp+suffix assumption) became moot; AC-15 collision-fixture path dropped.
- **R5 + R15 amended** (DA-2, tbc-r1-f3). Phase F precondition check on missing inputs; standalone manual invocation refused with diagnostic. R15 demoted to a Phase A Resume-logic clarification under R5; first-class Inputs-section promotion dropped.
- **R8 closer precedence** (DA-4). E.6 closer applies FAIL-on-unresolved-block to UNION of per-task + cross-task findings; closing rationale names which lens(es) drove status.
- **R9 config-at-runtime** (codex-r1-f5). Phase F reads `config.task.contextBudget` at execution time, never from synthesis snapshot.
- **R10 hybrid R3 sharpen surface** (tbc-r1-f5). Per-task reviewers sharpen R1 findings whose T-id still exists; auto-resolve findings whose T-id was merged-out / dropped; net-new findings on applier-introduced tasks recorded with one-pass orchestrator response.
- **New R16** (DA-3, codex-r1-f6). Sprint-bucket recompute via 025's heuristic on accepted merge/split; pre-edit and post-edit bucket values audit-recorded; ambiguous recompute routes to scope-change-required.
- **New R17** (DA-5). Sub-agent dispatch failure fallback — failed cross-task reviewer or applier dispatch logs to `*.failure.json`; gate escalates as `passed-with-escalations`; per-task review remains authoritative.
- **R14 simplified** (surgical-r1-f1, simplicity-r1-f4). scope-change/SKILL.md edit dropped; repo-wide grep audit dropped; doctrine-doc-only note + brief.md compatibility verification.
- **AC-5 / AC-6 awk pattern fixed** (codex-r1-f2). Strict next-heading boundary idiom replaces self-defeating `/^## Phase F/,/^## /` range pattern.
- **AC-11 split** (goal-driven-r1-f4). AC-11a worked-example fixture (accept + reject + sharpen); AC-11b scope-change-routing fixture exercising all three scope-change-required paths (missing R, hard-cap merge, drop-with-coverage-hole).
- **AC-12 doctrine-doc content** (goal-driven-r1-f3). Four content-grep checks asserting Phase E.4.5 section + Phase F section + manifest schema + agent_id generation scheme exist in the doctrine doc, not just the file.
- **AC-13 eval-coverage exhaustive** (goal-driven-r1-f6). Eval extension enumerates 3+ task path, <3 skip path, doctrine doc, worked-example fixture, brief.md compatibility, line cap.
- **AC-14 skip-path agent_id assertions** (goal-driven-r1-f5). Asymmetric state: writer_id present; cross_task_reviewer_id absent; applier_id absent; closer must NOT pseudo-error on absent fields.
- **AC-15 simplified** (above). Collision-fixture sub-block dropped.
- **AC-16 simplified** (surgical-r1-f1, simplicity-r1-f4). Doctrine-doc + brief.md compat checks only.
- **AC-17 simplified** (surgical-r1-f4). Phase A jump-to-F branch only.
- **R2 tradeoff sub-clause** (tbc-r1-f4). Threshold false-negative / false-positive tradeoff explicit.
- **R3 + R11 R3-sharpen-evidence assumption** (tbc-r1-f2). Worked-example fixture R3 step MUST produce non-empty new evidence; assumption made falsifiable.
- **R11 fixture content min requirement** (simplicity-r1-f3). Both lenses demonstrated; finding count is fixture-author discretion.

## Push-back (1)

- **simplicity-r1-f1** — collapse three-round mini-debate to two-step. **Push-back rationale:** Interview Round 3 user explicitly chose three rounds after considering single-shot. The 'leave it at three rounds so a decision has to be made' framing was user-confirmed; collapsing would walk back the sign-off. Recommendation: stand on the user's interview decision.

## Net status

PRD revisions applied land cleanly. The R7 simplification cascade (drop format spec, drop closer check, drop FRESH-CONTEXT-VIOLATION) addresses all five BLOCK-severity findings on R7 / agent_id mechanics. The DA-3 / codex-r1-f6 critical 025 cross-cut closed via new R16. The DA-5 dispatch-failure fallback closed via new R17. Round 3 sharpens omitted for context-budget; the orchestrator's read of the Round-2 revisions is that reviewer convergence is high.

Disposition: **passed-with-revisions**.
