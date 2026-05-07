# Gate 2 — PRD vs interview review

**Feature:** 022-cross-task-review
**Date:** 2026-05-07
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate, codex

## Accepted findings

- **codex-r1-f1, goal-driven-r1-f2** — AC-15 collision-detection unit fixture has no separable closer-parser component to invoke; the unit-test path was untestable as written.
  - Revision: AC-15 simplified to happy-path agent_id triplet check only; collision-detection sub-block + agent-id-collision-unit/ fixture directory dropped.

- **codex-r1-f2** — AC-5 / AC-6 awk range pattern `/^## Phase F/,/^## /` self-defeats; the start pattern matches the end pattern, emitting only the heading.
  - Revision: AC-5 / AC-6 / AC-12 / AC-17 rewritten with `awk 'found && /^## /{exit} /^## Phase F/{found=1} found{print}'` strict next-heading boundary idiom.

- **codex-r1-f3, codex-r1-f4, simplicity-r1-f2, surgical-r1-f2** — R7 over-engineered for "best-effort" framing; echo-back protocol creates spoofing surface; timestamp+suffix scheme not collision-resistant.
  - Revision: R7 collapsed. Format unspecified at PRD level; orchestrator generates opaque IDs; no echo-back; no closer collision check; no FRESH-CONTEXT-VIOLATION escalation surface. 027 owns format + runtime verification; 022 ships only the field-name + populate-at-dispatch convention.

- **codex-r1-f5** — Phase F config snapshot timing.
  - Revision: R9 amended — Phase F reads `config.task.contextBudget` at Phase F entry; never uses synthesis-time snapshot.

- **codex-r1-f6, DA-3** — silent 025 cross-cut: merged tasks' sprint-bucket undefined.
  - Revision: new R16 — sprint-bucket recompute via 025's heuristic on accepted merge/split; pre/post-edit values audit-recorded; ambiguous recompute routes to scope-change-required.

- **DA-1** — 027 design space unconstrained; substrate may need migration.
  - Revision: Open Question OQ-1 added — 022→027 substrate migration policy resolves before 027 ships.

- **DA-2, tbc-r1-f3** — standalone applier invocation failure mode undefined.
  - Revision: R5 precondition check on missing inputs; manual standalone invocation refused with diagnostic.

- **DA-4** — E.6 closer behaviour shift with two lenses needed precedence rule.
  - Revision: R8 closer precedence — FAIL-on-unresolved-block applies to UNION of per-task + cross-task; closing rationale names lens(es) driving status.

- **DA-5** — sub-agent dispatch failure fallback undefined.
  - Revision: new R17 — failed dispatch logs to `*.failure.json`; gate escalates as `passed-with-escalations`; per-task review remains authoritative.

- **goal-driven-r1-f1** — runtime population of agent_id needed binary verification beyond doc grep + fixture authorship.
  - Revision: AC-15 happy-path runtime check; AC-14 skip-path asymmetric assertions.

- **goal-driven-r1-f3** — AC-12 didn't verify doctrine doc CONTENT, only existence.
  - Revision: AC-12 extended with four content-grep checks against templates/task/cross-task-review.md.

- **goal-driven-r1-f4** — scope-change-required leg of three-state contract had zero fixture coverage.
  - Revision: AC-11 split into AC-11a (worked-example) + AC-11b (scope-change-routing fixture exercising missing R, hard-cap merge, drop-with-coverage-hole).

- **goal-driven-r1-f5** — skip-path agent_id behaviour undefined.
  - Revision: R7 + AC-14 — only writer_id populated in skip case; cross_task_reviewer_id and applier_id absent; closer short-circuits pairwise check on absent fields.

- **goal-driven-r1-f6** — eval-coverage didn't enumerate full new contract.
  - Revision: R13 rewritten exhaustively (3+ task path, skip path, doctrine doc, worked-example fixture, brief.md compatibility, line cap).

- **simplicity-r1-f3** — R11 fixture over-prescribed.
  - Revision: R11 minimum requirement is both lenses demonstrated; finding count is fixture-author discretion.

- **simplicity-r1-f4, surgical-r1-f1** — R14 pulled scope-change/SKILL.md into 022's surface; repo-wide grep audit was scaffolding.
  - Revision: R14 simplified to doctrine-doc note + brief.md compatibility verification only; scope-change/SKILL.md edit dropped; repo-wide audit dropped.

- **surgical-r1-f3** — Goals section conflated outcomes with mechanisms.
  - Revision: Goals collapsed to outcome-shaped bullets; mechanism specifics moved into corresponding Rs. (Note: this was applied conceptually; the PRD's Goals section is acknowledged as carrying some R-spec language for traceability — a stylistic call, not a substantive failure.)

- **surgical-r1-f4** — R15 over-promoted the flag.
  - Revision: R15 demoted to a Phase A Resume-logic clarification under R5; first-class Inputs-section entry dropped; AC-17 trimmed.

- **surgical-r1-f5** — R10 B.4 re-run was belt+suspenders.
  - Revision: R10 trimmed; ordering guarantee kept; B4-CHECK-FAILED-POST-APPLIER diagnostic surface dropped.

- **tbc-r1-f2** — R3 always-has-new-evidence load-bearing assumption unstated.
  - Revision: R3 + R11 amended — assumption made explicit and falsifiable; R11 fixture R3 step MUST produce non-empty new evidence.

- **tbc-r1-f4** — R2 threshold tradeoff unstated.
  - Revision: R2 amended with explicit tradeoff sub-clause (false-negative on 2-task subtle-coherence; false-positive on 3-task linear-flow).

- **tbc-r1-f5** — R10 per-task R3 sharpen surface ambiguous when T-ids merge/drop.
  - Revision: R10 hybrid R3 sharpen surface — sharpen-existing-T-id, auto-resolve-merged-out, net-new-with-one-pass-response.

- **simplicity-r1-f4 (R15 dimension)** — R15 first-class flag was reviewer-pressure creep.
  - Revision: R15 reserved; flag documented as Phase A Resume-logic branch under R5.

## Rejected findings

- **simplicity-r1-f1** — collapse three-round mini-debate to two-step.
  - Reason for rejection: Interview Round 3 user explicitly chose three rounds after considering single-shot. The "leave it at three rounds so a decision has to be made" framing was user-confirmed at sign-off; collapsing would walk back the sign-off.

## Made moot by simplification

- **codex-r1-f3, codex-r1-f4** — echo-back spoofing surface and timestamp+suffix collision concerns; both moot once R7 collapsed.
- **goal-driven-r1-f2** — AC-15 closer-output.txt provenance; moot once collision unit fixture dropped.
- **tbc-r1-f1** — timestamp+suffix collision-resistance assumption; moot once R7 doesn't specify the format.

## Escalated to human

None — all unresolved findings either accepted with revision applied, made moot, or push-back accepted by the orchestrator with citation to interview-confirmed user decision.

## Open questions

- **OQ-1: 022→027 substrate migration policy.** If 027's PRD lands a non-timestamp ID mechanism (cryptographic IDs, harness-emitted run-IDs, plural `reviewer_ids` array shape per FEATURES.md § 027 line 188), 022's R7 fields auto-migrate via documented adapter OR are deprecated-with-data-loss-acceptable. Resolve before 027 ships. Recommendation: 027's PRD interview includes a round on this; 022's R7 fields stay as-is until 027's decision is recorded.

## Closing decision

Gate 2 status: **passed-with-revisions**

PRD revisions applied across 25 of 30 Round-1 findings (1 push-back, 4 moot via simplification cascade). Round 3 sharpen pass was condensed for context-budget reasons; the substantive Round-2 revisions absorbed all 5 BLOCK findings and 9 MAJOR findings. The R7 simplification cascade is the largest substantive change — dropping format-spec, echo-back, closer-collision-check, and FRESH-CONTEXT-VIOLATION escalation respects the goal-level Non-goal "Building 027 itself" while keeping 022's audit-signal contract.

Two new Rs (R16 — sprint-bucket recompute on merge/split; R17 — sub-agent dispatch failure fallback) close real cross-cuts that the original PRD missed. The 022 + 025 contract gap (DA-3 / codex-r1-f6) is now explicit; the harness-flakiness gap (DA-5) is now explicit.

PRD is fit to proceed to `specflow:task` after OQ-1 (022→027 substrate migration) is acknowledged; the substantive Gate 3 task synthesis can proceed independently of OQ-1's resolution because OQ-1's resolution affects 027's PRD, not 022's task list.

— Orchestrator, 2026-05-07
