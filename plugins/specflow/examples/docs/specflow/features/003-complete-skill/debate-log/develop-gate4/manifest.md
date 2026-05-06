# Debate manifest — Gate 4: plan vs tasks/PRD review

**Feature:** 003-complete-skill
**Gate:** develop-gate4
**Artefact under review:** `admin/scratch/003-complete-skill-develop/lane-assignments.json` (the lane plan covering all 15 tasks T1-T15)
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not invoked at Gate 4 (v1 hard-coded — Codex joins Gate 5 only)
**Date:** 2026-05-06
**Closed:** 2026-05-06

This is the Phase 2 dogfood Gate 4 run for the `specflow:complete` lane plan. The plan was synthesised against a 15-task tasks file (T1-T14 + T15) where two of the 14 R-level requirements (R4's cross-skill schema dependency, R14's lock-file guard) carry Gate 2 finding revisions in their trace lines AND five tasks (T4, T9, T10, T12, T13) carry Gate 3 finding revisions. Gate 4 reviewers fired against the lane plan's per-task four-axis triage tuples, the B.1 mechanical recheck outcome, the lane-summary distribution (13 Green / 2 Yellow / 0 Red), and the cross-task dependency graph. Five `concern` findings landed; one `push_back` defended; the rest accepted.

---

## Context

### Lane summary table

| Lane | Count | Tasks |
|---|---|---|
| Green | 13 | T2, T3, T4, T5, T6, T7, T8, T9, T10, T11, T12, T14, T15 |
| Yellow | 2 | T1, T13 |
| Red | 0 | — |

T1 is Yellow on the medium-blast axis (cross-skill touch — `skills/complete/SKILL.md` AND `skills/develop/SKILL.md` Phase F.5 → F.6 hook). T13 is Yellow on the medium-verifiability axis (substring-match Related-field parsing per PRD Open Questions has one ambiguous edge). All other axes Green-qualifying for these two tasks.

### Environment availability (from `admin/environment.json`)

- **agent-teams plugin:** absent (not in detected plugins list — `plugins[]` shows specflow, frontend-mobile-development, comprehensive-review only). Phase D will run sequential single-specialist invocation. Throughput reduced; functionality preserved.
- **Codex CLI:** detected (v0.18.0 at `/usr/local/bin/codex`). Gate 5 will invoke Codex as the sixth reviewer. Gate 4 does not invoke Codex (v1 hard-coded).
- **Linear MCP:** detected (configured for `northwind-orders` workspace). Phase A.4 + Phase F.4 will fire Linear status transitions for each task as it enters/exits its lifecycle.

### B.1 mechanical recheck

Ran 2026-05-06 across all 15 tasks. File-count comparison (plan_files vs scope_claimed) showed no task exceeding the 1.5× ratio threshold. Module comparison (plan_modules vs scope_modules) showed zero new modules introduced beyond stated scope (T1's `skills/develop/SKILL.md` and T14's `admin/scratch/**` lock-file path were both already in their respective scope fields). Confidential-path glob match against `confidentialPaths` (`src/billing/**`, `src/auth/**`, `src/migrations/**`, `infrastructure/**`, `ops/runbooks/**`, `.env*`, `secrets/**`) returned zero matches across all 15 tasks. **Outcome: no upgrades triggered. lane_changes: [].** T1 + T13 retained Yellow on their original weak axes; T2-T12 + T14 + T15 retained Green.

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
- **simplicity-r1-f1** — *concern* — `b1_recheck.lane_changes: []` empty-array field shape may be speculative configurability without a documented downstream consumer. (The Green-batch 5-batch split for 13 tasks at cap=3 is interview-grounded and not at issue.) (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — T1's `files_touched` includes `skills/develop/SKILL.md` (cross-skill code-edit surface), but Gate 3 surgical-r1-f1 tightened T4's parallel cross-skill surface to runtime detection only; the asymmetric treatment of T1 vs T4 surfaces as scope drift on T1. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *concern* — T9's verifiability=high binds to `superseded_by_retro: true`, but the field's existence depends on a cross-skill prerequisite (`specflow:develop` Phase F.5 emitting the default flag) that is out-of-scope for this skill; the unstated sequencing dependency is hidden in the lane plan. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **goal-r1-f1** — *concern* — T15's `anchor_r: ['AC-15']` value structurally contradicts the field name `anchor_r` (R-array implied); a mechanical reader cannot disambiguate the documented orphan-AC exception from a malformed entry. (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — The lane plan as a scratch artefact has no closed-loop verification; the AI's pre-execution lane judgement is not exercised by any binary check after Phase D completes. (Same file.)
- **da-r1-f1** — *concern* — Cross-cutting partial-state recovery: with greenBatchCap=3 and 13 Green tasks (5 batches), T15's emit-on-exit contract sits in batch-5 with 7 upstream dependencies; a batch-5 abort leaves 12 tasks merged without T15 enforced. Lane plan does not surface this. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **push_back** (defended `b1_recheck.lane_changes: []` field shape: empty-array vs absent-key carries distinct semantic content (recheck-ran-zero-upgrades vs recheck-did-not-run); dropping the field per option (a) would silently lose audit-trail signal AND force a schema migration when the first downgrade fires; mirrors 002-develop-skill Gate 3 simplicity-r1-f1 defended push-back on T15-cross-cutting-contract; the 5-batch interview-grounded split is non-controversial)
- surgical-r1-f1 → **accept** (applied option (b) — defended T1's broader scope as in-scope for the auto-fire integration; the asymmetric T1-vs-T4 treatment is defensible per the data-default-vs-invocation-point distinction; T4's `superseded_by_retro: false` default is structurally separable, T1's hook is not; manifest-side documentation of the asymmetry as a known-and-accepted tradeoff)
- tbc-r1-f1 → **accept** (applied option (b) — extended T9's `lane_reason` text with the explicit cross-skill prerequisite clause; option (a)'s `depends_on_external` field rejected on Simplicity grounds (would introduce a per-task field no other task uses))
- goal-r1-f1 → **accept** (applied option (b) — added `anchor_exception` sibling field to T15 only; the 14 R-anchored tasks unchanged; mechanical reader can disambiguate via `anchor_exception.kind` field)
- goal-r1-f2 → **accept** (applied option (b) — added top-level `feature_lane_outcome` field initialised to zero counters; Phase F.5 will populate at feature close by aggregating from per-task `task-history.json` development-time fields)
- da-r1-f1 → **accept** (applied option (b) — procedural pre-Phase-D announcement surfacing the partial-state-recovery concern; option (a)'s `batch_priority` per-task field rejected on Simplicity grounds (redundant with `depends_on` mechanical batching))

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- simplicity-r1-f1 → **accept** (push-back defended; empty-array audit-trail-signal calibration mirrors 002-develop-skill Gate 3 precedent; the simplicity-vs-traceability rule releases on audit-trail fields whose empty value carries semantic content)
- surgical-r1-f1 → **accept** (data-default-vs-invocation-point distinction held; T1 retained Yellow with documented asymmetric tradeoff)
- tbc-r1-f1 → **accept** (revision applied; cross-skill prerequisite now visible in artefact prose; option-(b) choice defensible on Simplicity grounds)
- goal-r1-f1 → **accept** (revision applied; structurally-readable `anchor_exception` field disambiguates T15's documented orphan-AC exception)
- goal-r1-f2 → **accept** (revision applied; lane-plan loop now closed via `feature_lane_outcome` Phase F.5 populator)
- da-r1-f1 → **accept** (revision applied; partial-state-recovery concern surfaced procedurally; mechanical batching from `depends_on` ensures T15-last placement)

No sharpening occurred. No `ai-revision.md` needed.

---

## Plan revisions applied

The lane plan (`admin/scratch/003-complete-skill-develop/lane-assignments.json`) was edited between Round 1 and Round 3 to incorporate every accepted finding:

1. **T9 `lane_reason` extended with cross-skill prerequisite clause (concern tbc-r1-f1).** T9's lane reasoning now names the sequencing dependency on the `specflow:develop` enhancement PRD landing the `superseded_by_retro: false` default flag before `specflow:complete` v1 ships. The unstated assumption is now stated.

2. **T15 `anchor_exception` sibling field added (concern goal-r1-f1).** T15's entry carries `anchor_exception: { kind: 'goal-outcome', surfaces: ['c-chat-line-summary', 'd-zero-silent-failures'], gate3_finding: 'goal-r1-f1' }` for structurally-readable disambiguation of the documented orphan-AC exception. The 14 R-anchored tasks unchanged.

3. **Top-level `feature_lane_outcome` field added (concern goal-r1-f2).** Lane plan now carries `feature_lane_outcome: { upgrades_fired: 0, machine_check_failures: 0, accept_and_ship_tasks: 0, populated_at: null }`. Phase F.5 of `specflow:develop` will populate at feature close. Lane-plan loop closed.

## Plan revisions deferred to procedural surfaces

Two findings were resolved with manifest-side or skill-side documentation rather than lane-plan schema changes:

- **da-r1-f1 (partial-state-recovery announcement).** Phase B.5 lane-distribution announcement extended for this feature: the user is prompted before Phase D begins to confirm the batch-5 abort risk for T15. No lane-plan schema change.
- **surgical-r1-f1 (T1-vs-T4 asymmetric tradeoff).** Documented in this manifest's Closing decision section as a known-and-accepted tradeoff. No lane-plan schema change.

## Findings rejected after Round 3

One finding had a defended push-back accepted in Round 3:

- **simplicity-r1-f1 (push-back on the `b1_recheck.lane_changes: []` field).** The empty-array value carries distinct semantic content (recheck-ran-zero-upgrades vs recheck-did-not-run) — dropping the field per option (a) would silently lose audit-trail signal that the mechanical recheck ran AND found zero downgrades, AND would force a schema migration on the scratch artefact mid-rollout when the first downgrade fires. The 002-develop-skill Gate 2 dogfood (block tbc-r1-f1) made the mechanical recheck load-bearing precisely because reviewer judgement on lane-aggressive flags was unreliable; the recheck record is the structured trace that closes that block. The simplicity-vs-traceability calibration mirrors 002-develop-skill Gate 3 simplicity-r1-f1's defended push-back: an audit-trail field whose empty value carries semantic content is not speculative configurability. Round-3 simplicity-reviewer accepted: the principle releases on audit-trail surfaces whose empty value is the explicit no-event-fired record.

## Findings escalated to human

None. All six findings converged within three rounds.

## Closing decision

**Gate 4 status: passed-with-revisions**

Five of six findings were accepted by the AI and revisions applied to `admin/scratch/003-complete-skill-develop/lane-assignments.json` (T9 cross-skill prerequisite clause, T15 `anchor_exception` sibling field, top-level `feature_lane_outcome` populator field) or to procedural surfaces (Phase B.5 partial-state-recovery announcement; manifest-side documentation of T1-vs-T4 asymmetric tradeoff). One finding (simplicity-r1-f1) had a defended push-back accepted in Round 3, citing the audit-trail-signal-vs-speculative-configurability calibration anchored in the 002-develop-skill Gate 2 mechanical-recheck precedent. No findings escalated to human decision. The lane plan is fit to proceed to Phase D — Green-batch execution begins with batch-1 of 3 (T2 + T3 + T4 — all upstream-satisfied with T1 as Yellow predecessor); T1 + T13 ship via Yellow-lane HITL pairing; T15 lands in batch-5 with all upstream dependencies satisfied. The pre-Phase-D announcement will surface the batch-5-abort partial-state-recovery risk to the user before execution begins. The T1-vs-T4 asymmetric cross-skill tradeoff is documented as known-and-accepted: T1's auto-fire hook in `skills/develop/SKILL.md` Phase F.5 → F.6 is the producer-skill landing surface and cannot be split from `specflow:complete` v1 (R1 unimplementable otherwise); T4's `superseded_by_retro: false` default flag is structurally separable and was correctly routed to a separate enhancement PRD. The Codex CLI is detected (v0.18.0); Gate 5 will invoke Codex as the sixth reviewer per Phase E.

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 4 reviewers and humans reading this as a **dogfood example** (the first full Phase 2 dogfood Gate 4 against a fully-revised tasks file with Gate 2 + Gate 3 findings already applied):

- **Six concerns / zero blocks / one push-back is the right shape for a Phase 3 retro skill's lane plan** when the upstream PRD (Gate 2) and tasks file (Gate 3) have already been pre-tightened. The Gate 4 findings fire on lane-plan-layer ambiguities (audit-trail field shape, cross-skill scope drift propagated from Gate 3, unstated cross-skill prerequisite, orphan-AC anchor schema awkwardness, lane-plan-not-verified-after-execution, partial-state-recovery cross-cutting concern) rather than missing R-coverage or wholly-misclassified lanes — which is exactly the surface a Gate 4 reviewer should fire on when the upstream gates have done their work. No finding earned a `block` because the lane plan's mechanical recheck (B.1) had already caught the structurally-checkable lane-aggressive cases (zero-upgrades-triggered outcome).
- **The push-back precedent generalises across gates.** The 002-develop-skill Gate 2 push-back on simplicity-r1-f1 (R6's interview-grounded configurable cap), the 002-develop-skill Gate 3 push-back on simplicity-r1-f1 (T15 cross-cutting contract), and this Gate 4's push-back on simplicity-r1-f1 (`b1_recheck.lane_changes: []` audit-trail field) all share the same shape: simplicity instinct to drop a field/task at the artefact layer, defended by citing a load-bearing trace anchor (interview line for cap; cross-cutting contract for T15; audit-trail-signal-vs-absent-key for `lane_changes`). The pattern is durable: simplicity at the artefact layer cannot override traceability anchored in a load-bearing trace surface.
- **Goal-Driven was load-bearing here (2 of 6 findings, both `concern`).** Both fired on traceability-shape boundaries: goal-r1-f1 on the orphan-AC anchor schema awkwardness (E4 reverse traceability with a documented exception), goal-r1-f2 on the lane-plan-not-verified-after-execution gap. The pattern matches the role file's "load-bearing at Gate 4" framing — coverage-matrix integrity AND binary verification surfaces are the two Goal-Driven primary lenses, and both fired here.
- **Surgical at Gate 4 caught propagated drift from Gate 3.** Gate 3 surgical-r1-f1 tightened T4's cross-skill surface but did NOT pass over T1's parallel cross-skill surface; the asymmetric outcome surfaced at Gate 4 (because Gate 4's lens is plan-vs-tasks-vs-PRD and the lane plan's `files_touched` field made the asymmetry mechanically visible). The accept-with-defended-tradeoff outcome (data-default-vs-invocation-point distinction) is defensible. Future Gate-3-to-Gate-4 propagation: if Gate 3 tightens one cross-skill surface, audit parallel surfaces in the same tasks-file pass — the Gate 4 lane plan will surface anything missed but earlier intervention is cheaper.
- **Phase 2 dogfood succeeded.** This Gate 4 manifest is itself the dogfood of `specflow:develop` Phases A-C against the just-built `specflow:complete` Phase 3 retro skill. The pre-flight (Phase A) read PRD/tasks/Gate 3 successfully; the lane triage (Phase B) classified all 15 tasks with rule-based confidentiality (zero matches against `confidentialPaths`); the mechanical recheck (Phase B.1) ran with zero downgrades; the multi-agent debate (Phase C) surfaced 6 findings with 1 defended push-back. The orchestrator pattern (forked sub-agents per reviewer, JSON finding files, manifest closer) held end-to-end without context bloat. Calibration anchor for future Phase 2 dogfoods on subsequent Phase 3 skills (`specflow:decision`, `specflow:scope-change`, `specflow:misc`).
