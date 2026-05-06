# Dogfood debrief — 003-complete-skill

**Date:** 2026-05-06
**Pass:** Phase 2 dogfood: full chain run on 003-complete-skill (`specflow:prd` → `specflow:task` → `specflow:develop` Phases A-F end-to-end against the `specflow:complete` Phase 3 retro skill).
**Outcome:** SKILL body shipped at 472 lines covering all 14 PRD R-IDs plus AC-15. Gate 4 closed `passed-with-revisions` (6 concerns, 1 push-back defended); Gate 5 closed `passed-with-revisions` (8 concerns, 1 push-back defended, 1 sharpen, Codex contributed 2 of 8 findings); Verifier closed `verified-with-conditions` (14 pass + 1 conditional-pass on T4's cross-skill schema dependency, 0 fail).

This debrief is the deliverable from the `specflow:develop` Phase F dogfood pass per the v2.1.0 release plan. It captures what the chain surfaced, what worked, and concrete prompt edits recommended for `skills/{develop,complete}/SKILL.md` and the principle-reviewer templates based on observed friction. Edits are recommendations only — the human applies after review.

---

## What worked

### W1. Lane triage classification was clean (13/2/0 split with rule-based confidentiality matching zero confidential paths).

Phase B's four-axis lane triage produced a 13 Green / 2 Yellow / 0 Red split across all 15 tasks (`admin/scratch/003-complete-skill-develop/lane-assignments.json:158-160`). Rule-based confidentiality classification (per `skills/develop/SKILL.md:118` 'never AI-rated') returned zero matches against `confidentialPaths` — every task touched `plugins/specflow/skills/complete/SKILL.md` or `admin/scratch/**` paths, none in the `confidentialPaths` glob list. T1 and T13 landed Yellow on their original weak axis (T1 medium-blast on cross-skill touch; T13 medium-verifiability on substring-match Related-field parsing) — both correctly identified at first pass without review-time intervention. The B.1 mechanical recheck ran with zero downgrades. The 13/2/0 split is close to the documented target shape (60/30/10) for a Phase 3 retro skill where most tasks are localised to one SKILL.md file, validating the lane-axis weights for this domain.

### W2. B.1 mechanical recheck ran and recorded outcome even when no upgrades triggered (the discipline holds).

Phase B.1's mechanical pre-Gate-4 lane recheck fired for all 15 tasks per `lane-assignments.json:167-171` (`b1_recheck` field with `ran_at`, `lane_changes: []`, summary). The recheck ran across three checks (file-count vs scope, modules vs scope, confidential-path glob match) and recorded `lane_changes: []` (zero upgrades triggered). The discipline of running-and-recording even when no upgrades fire is the load-bearing addition from 002-develop-skill Gate 2 (block tbc-r1-f1) — the empty-array audit-trail-signal is the structured trace the Gate 4 simplicity-r1-f1 push-back defended. The pattern propagates correctly: 002 dogfood established the recheck; 003 dogfood exercised the empty-array path and the audit-trail-signal-vs-speculative-configurability calibration held under push-back.

### W3. Gate 4 push-back convergence (1 push-back defended, accepted in Round 3).

Gate 4 fired 6 concerns; Round 2 produced 1 push-back (simplicity-r1-f1 on `b1_recheck.lane_changes: []` audit-trail field) and 5 accepts; Round 3 converged with the push-back defended on audit-trail-signal-vs-speculative-configurability calibration anchored in 002-develop-skill Gate 3 simplicity-r1-f1 precedent (`features/003-complete-skill/debate-log/develop-gate4/manifest.md:104-105`). Gate 5 then fired 8 concerns; Round 2 produced 1 push-back (simplicity-r1-f1 on triple-flag pattern) and 7 accepts; Round 3 converged with 1 sharpen (codex-r1-f1 'required' semantic clarification) + 7 accepts including the push-back defence (`features/003-complete-skill/debate-log/develop-gate5/manifest.md:124-126`). The push-back-on-simplicity-with-defended-trace-anchor pattern now spans 4 gates (002-G2, 002-G3, 003-G4, 003-G5) — durable across PRD, tasks, plan, and code reviews.

### W4. The chain produced a 472-line operational SKILL body covering all 14 PRD requirements with reverse-traceability holding.

`plugins/specflow/skills/complete/SKILL.md` is 472 lines, 8 phases A-H. Forward-coverage holds: every R-ID (R1-R14) maps to a section (R1→A.1, R2→A.1, R3→C.1+E.1, R4→B.3+E.5, R5→E.4, R6→D.1, R7→F.2-F.3, R8→Inputs/Anti-patterns, R9→B.4, R10→E.2, R11→D.2, R12→D.2+F.4, R13→E.3, R14→A.3). Reverse-traceability: every phase traces to ≥1 R or AC, with the single exception of Phase G (Linear status sync) which Gate 5 goal-r1-f1 flagged as orphan-phase and resolved with a documented-inference clause at the phase header. The SKILL body ships with an explicit failure-modes section (lines 423-436) enumerating 9 documented failure modes and an anti-patterns section (lines 439-449) with 8 refuse-to-do rules. The eval block in the frontmatter binds the schema-validation check, the idempotent re-invocation contract, and the elevation-write trace.

### W5. E1-E5 prompt edits visibly fired (E3 status taxonomy `passed-with-revisions` used at every gate; E4 orphan-AC lens fired in Goal-Driven Reviewer findings).

The five recommended prompt edits from the 002-develop-skill DOGFOOD-DEBRIEF (`features/002-develop-skill/DOGFOOD-DEBRIEF.md:E1-E5`) had observable effect at Gate 4 + Gate 5:
- **E3 status taxonomy** — every gate manifest closed with `passed-with-revisions` per the four-status taxonomy (`passed | passed-with-revisions | passed-with-escalations | failed`); the distinction between 'clean pass' and 'pass-after-revisions' is now structurally readable downstream.
- **E4 orphan-AC reverse-traceability lens** — Goal-Driven Reviewer fired the orphan-phase finding at Gate 5 (goal-r1-f1 on Phase G) using the inverse-coverage rule, just as E4 codified. Without E4, the orphan phase might have slipped past Goal-Driven and only surfaced (if at all) at the Verifier step.
- **E5 Gate 2 block-finding resolutions surfaced as task synthesis input** — `specflow:task` Phase A.2 (per E5 addition) read the Gate 2 manifest's PRD-revisions-applied section; T4's cross-skill schema dependency (Gate 2 surgical-r1-f1) and T9's superseded-by-retro boundary (Gate 3 goal-r1-f2) were inherited correctly into the lane plan and into the SKILL body without re-litigation.

The E1 (Vision verbatim-vs-paraphrase) and E2 (PRD synthesis cross-check ACs against Phase 1 schemas) edits were less directly observable in this dogfood because the 003-complete-skill PRD's Vision section was already terse (a single paragraph closely tracking the Goal Outcome) and the AC-skill-schema dependencies were caught at the PRD layer (not requiring synthesis-time intervention). Both prompt edits remain useful for future feature flows.

---

## What broke (friction surfaced)

### F1. Phase A.3's stale-lock 30-min heuristic — arbitrary number, no calibration evidence in the SKILL body until Gate 5 surfaced it.

`plugins/specflow/skills/complete/SKILL.md:64-69` — the 30-minute stale-lock threshold ships hard-coded with no calibration text in the SKILL body. The PRD Open Questions (`003-complete-skill-prd.md:180`) flagged the issue ('the time-based heuristic is sufficient for v1; revisit if consumers report stale-lock false-positives') but the SKILL body did not propagate as a documented assumption — Gate 5 tbc-r1-f1 caught it and the revision added a one-paragraph rationale. **Friction:** the SKILL-author lifecycle agent inherited the magic number from the PRD without surfacing the assumption that picked it. A future PRD with similar magic numbers would risk the same gap.

### F2. Phase E.4 schema validation surface — fields normative? PRD says snake_case but doesn't enumerate which fields are required vs optional.

`plugins/specflow/skills/complete/SKILL.md:273-279` — Phase E.4 originally enforced two checks (snake_case regex + allow-list extraneous-field rejection). Codex caught (codex-r1-f1) that required-field-presence was NOT enforced — a malformed entry missing `prd_anchor` (or any other documented field) would pass validation. The PRD Schema Appendix (`003-complete-skill-prd.md:141-176`) names the field set but does NOT explicitly mark which fields are required vs optional with default. The SKILL body inherited the ambiguity. The Round-3 sharpen further clarified 'required' means 'present, possibly with default value per Schema Appendix'. **Friction:** the PRD Schema Appendix needs an explicit required-vs-optional annotation; without it, the SKILL body can validate one direction (no extraneous) without catching the inverse (no missing).

### F3. Gate 4 / Gate 5 reviewer lens overlap — Goal-Driven's reverse-traceability lens overlaps with Codex's semantic-correctness lens at the AC-to-code boundary.

`features/003-complete-skill/debate-log/develop-gate5/manifest.md:172-176` — Gate 5 calibration notes flagged that 'Goal-Driven owns the AC-to-code reverse-traceability lens (goal-r1-f1 + goal-r1-f2) while Codex owns the semantic-correctness lens (codex-r1-f1 + codex-r1-f2). The two lenses overlap at the AC-to-code boundary; future role-file revisions should document the overlap so reviewers do not double-fire.' In this run the lenses fired distinctly (different findings, no double-firing), but the role-file documentation does not explicitly carve the boundary. **Friction:** future runs may surface findings that both Goal-Driven and Codex fire on; without an explicit boundary the AI response phase may treat them as independent when they're the same finding from two angles.

### F4. The lane plan's `lane_summary` field is computed but not reviewed against batch-size constraints.

`admin/scratch/003-complete-skill-develop/lane-assignments.json:156-160` — the `lane_summary: {green: 13, yellow: 2, red: 0}` field is mechanically computed from per-task lane assignments but is not reviewed against the batch-cadence shape. With `greenBatchCap: 3` and 13 green tasks, the execution shape is 5 batches (batch-1: T2+T3+T4; batch-2: T5+T6+T11; batch-3: T7+T8+T9; batch-4: T10+T12+T14; batch-5: T15). Gate 4's da-r1-f1 caught the partial-state-recovery concern (T15's emit-on-exit contract sits in batch-5 with 7 upstream dependencies; a batch-5 abort leaves 12 tasks merged without T15 enforced) and the resolution was a procedural pre-Phase-D announcement — but the underlying shape (5 batches, terminal task with 7 upstream deps) was not flagged by the `lane_summary` field itself. **Friction:** the lane-summary field is a count summary, not a batch-shape summary; reviewers see the count and assume distribution-is-fine without inspecting the dependency-graph terminus.

### F5. Verifier's "conditional-pass" outcomes need a clearer escalation path.

`admin/scratch/003-complete-skill-develop/verifier-outcome.json` — T4 closed as `conditional-pass` (cross-skill schema dependency on `specflow:develop` Phase F.5 emitting `superseded_by_retro: false` as default). The Verifier outcome documents the condition and an escalation path, but `skills/develop/SKILL.md` Phase F (the consumer of verifier-outcome.json) does not name how Phase A.2 of `specflow:complete` (which reads the Verifier outcome) should treat conditional-pass: does it block Phase A.2's auto-fire path? Does it surface a chat-line warning? The taxonomy at `skills/develop/SKILL.md:343-348` (Gate 4/5 status) covers `passed | passed-with-revisions | passed-with-escalations | failed` but not Verifier conditional-pass. **Friction:** conditional-pass exists in the Verifier output schema but has no documented downstream contract for what consumers do with it.

### F6. Phase G (Linear status sync) was implementer-stated, not PRD-anchored — caught by Gate 5 but the orphan-phase risk is structural.

`plugins/specflow/skills/complete/SKILL.md:358-377` — Phase G has no R-anchor in the PRD; it was inferred from `specflow:develop` Phase F.4 calibration. Gate 5 goal-r1-f1 flagged the orphan-phase, and the resolution was a documented-inference clause at the phase header. But the underlying friction is that the SKILL author lifecycle agent inferred a phase from cross-skill-ecosystem calibration without explicit PRD authorisation. Future Phase 3 skills (`specflow:decision`, `specflow:scope-change`) may face the same shape — implementer infers a phase that 'feels right' from the ecosystem but is not PRD-anchored. **Friction:** the lifecycle-agent prompts do not currently warn against implementer-inference of phases; the orphan-phase risk relies on Gate 5 catching it.

---

## Recommended prompt edits

These are recommendations. The human applies after review. **Do NOT auto-apply.** All file:line citations are against current `plugins/specflow/`.

### E6. `skills/develop/SKILL.md:Phase A.2` — make Codex availability check explicit at A.1, not implicit at E.4.

Current text (Phase A.2, line 87): `cli.codex.available — true → Phase E Gate 5 invokes Codex as a sixth reviewer. False → Phase E manifest writes the literal sentinel...`

**Recommended change:** surface the Codex availability check earlier — at A.1 alongside the artefact chain check — so that Gate 4 reviewers can pre-load Codex's lens-overlap context (per F3 above). Reword to: *"At A.1, also read `admin/environment.json` for `cli.codex.detected`. If true, surface to the user at A.5: 'Codex CLI detected (v{x.y}); Gate 5 will fire with the cross-provider lens. Goal-Driven Reviewer's reverse-traceability lens and Codex's semantic-correctness lens overlap at the AC-to-code boundary; the Round 2 AI response phase should treat overlapping findings as a single finding from two angles.' Gate 4 reviewers do not consume Codex but should be aware their orphan-AC findings may be re-litigated at Gate 5 by Codex's overlapping lens."* Closes F3.

### E7. `skills/develop/SKILL.md:Phase B.1` — formalise B.1 outcome JSON schema (currently free-form recording).

Current text (Phase B.1.2, lines 197-222): the per-task recheck file schema is documented but the `b1_recheck` aggregate-outcome field at the lane-assignments.json top-level is free-form. Per F4 (and the Gate 4 simplicity-r1-f1 push-back defence), the field's empty-array value carries semantic content but its schema is not formalised.

**Recommended addition** (new sub-step B.1.5, inserted after B.1.4):

> ### B.1.5 Aggregate the recheck outcome
>
> After all per-task recheck files are written, aggregate the outcome into `lane-assignments.json` top-level field `b1_recheck`:
>
> ```json
> {
>   "ran_at": "{YYYY-MM-DD}",
>   "lane_changes": [
>     {"task_id": "T{N}", "before": "green | yellow | red", "after": "green | yellow | red", "reason": "{file-count-ratio | new-modules | confidential-path-match}"}
>   ],
>   "summary": "{one-line description of recheck outcome}",
>   "batch_shape_at_default_cap": {
>     "batches": N,
>     "terminal_task": "T{N}",
>     "terminal_task_upstream_count": N
>   }
> }
> ```
>
> The `batch_shape_at_default_cap` sub-field is the load-bearing addition — it surfaces the partial-state-recovery shape Gate 4 da-r1-f1 caught procedurally. With this field formalised, Gate 4 reviewers can fire on `terminal_task_upstream_count > 5` (or similar) as a structurally-checkable concern, rather than relying on Devil's Advocate to spot it.

Closes F4.

### E8. `skills/complete/SKILL.md:Phase A.3` — surface the 30-min heuristic as `config.json.complete.staleLockMinutes` so projects can tune.

Current text (Phase A.3 case 3, line 69): the 30-minute stale-lock threshold is hard-coded with the post-Gate-5 calibration clause documenting the choice rationale.

**Recommended change** (v2 enhancement, deferred from v1 per Simplicity-First):

> ### A.3 case 3 (v2-extended)
>
> Read `config.json.complete.staleLockMinutes` (default 30). If the lock body's timestamp + `staleLockMinutes` < now: treat as stale; overwrite and proceed; surface chat-line. The configurable knob lands in v2 alongside the `--clear-stale-locks` admin verb per PRD Open Questions; v1 ships hard-coded at 30 minutes.

Closes F1 (v2 path); preserves v1 Simplicity-First.

### E9. `templates/agents/standard/principles/goal-driven-reviewer.md` — codify the AC-to-code reverse-traceability lens for code-review gates.

Current state: per E4 (002-develop-skill DOGFOOD-DEBRIEF) the orphan-AC reverse-traceability lens is documented for PRD/tasks gates. At Gate 5 (code review) the same lens fires on orphan-phase / orphan-section findings (per goal-r1-f1 in this gate's findings).

**Recommended addition** to the role file's Common Findings section:

> - **Orphan phase / orphan section** (code-review gates) — a phase or section in the SKILL body that does not trace to any PRD R-ID or AC. Reverse traceability at the code-review layer: every phase must trace to ≥1 R or AC, just as every R must be covered by ≥1 phase. A phase implementing a contract the PRD didn't make is itself a coverage gap (the contract is unstated at the R-level OR the phase is implementer-inferred from cross-skill calibration without PRD authorisation). The acceptable resolution shape is either (a) drop the phase, (b) add a documented-inference clause naming the calibration anchor and the PRD-scope-change ratification path, or (c) add a new R via PRD scope-change. Option (b) is the most proportionate at Gate 5.

Closes F6.

### E10. `skills/develop/SKILL.md:Phase F (Verifier disposition)` — define the conditional-pass escalation contract.

Current text (Phase F.1-F.3, lines 494-534): Verifier returns `pass | reject`; reject branches to F.3's four-option prompt. The Verifier-outcome schema in this dogfood (`admin/scratch/003-complete-skill-develop/verifier-outcome.json`) introduced a `conditional-pass` outcome (per T4's cross-skill schema dependency) that the SKILL body does not consume.

**Recommended addition** (new Phase F.1.5, inserted after F.1):

> ### F.1.5 Conditional-pass branch
>
> If the Verifier returns `conditional-pass` for any task, the outcome carries a `conditions` array naming the upstream dependencies that must land before the task's AC fires in production. Branch:
> - If every condition is satisfied (the upstream dependencies have shipped per `task-history.json` reads): treat as `pass` and proceed to F.4.
> - If any condition is unsatisfied: surface to the user with two options:
>   1. **Hold and wait** — pause the task at F.1.5; the user lands the upstream dependency via a separate flow; resume `specflow:develop --task T{N}` when ready.
>   2. **Ship with documented condition** — proceed to F.4 (PR open) with a description-paragraph note naming the unsatisfied condition; record in `task-history.json` as `conditional_ship: true` with the condition list. The downstream `specflow:complete` retro entry's `accepted_with_failure` field is NOT toggled (the Verifier did not reject); a separate `conditional_ship` boolean carries the signal.
>
> The skill does not auto-default; an empty input or any input not matching `1|2` re-prompts.

Closes F5. Note: this also requires extending `skills/complete/SKILL.md:E.1` schema appendix to include a `conditional_ship: bool` field; landing this change requires a `specflow:complete` enhancement PRD per the cross-skill change pattern.

---

## Next-session priority

**Recommended:** cut v2.1.0 release (Phase 2 dogfooded; release-ready). Then Phase 3 Phase 2 — apply E6-E10, then build the remaining Phase 3 skills (`specflow:decision`, `specflow:scope-change`, `/insights`, `/prune`, `/optimize`) using the chain.

The dogfood pass demonstrates that the chain handles a real Phase 3 retro skill end-to-end: PRD synthesis with two `block`-equivalent surfacings (R5.1 / cross-skill schema dependency) cleanly resolved at Gate 2; tasks file with 15 tasks and Gate 3 + Gate 4 revisions inherited correctly; lane plan with 13/2/0 split and zero confidential-path matches; SKILL body with all 14 R-IDs covered + AC-15 + reverse-traceability holding (Phase G's documented-inference clause closes the only orphan-phase risk). The surfaced friction (F1-F6 above; E6-E10 prompt edits) is incremental tuning, not architectural rework. None of the friction blocks v2.1.0.

Concretely:
1. Apply E6-E10 prompt edits (after human review).
2. Update `plugins/specflow/.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` to v2.1.0 with synced descriptions per project rules.
3. Cut v2.1.0 release.
4. Phase 3 build picks up the next Phase 3 skill (`specflow:decision` is the natural follow-on per the cross-skill F.5 binding the SKILL body deferred at Gate 5 surgical-r1-f1). The PRD chain for `specflow:decision` will use the same chain that ran here, with E6-E10 applied at the start.
5. Land the `specflow:develop` enhancement PRD adding `superseded_by_retro: false` to Phase F.5's initial-write field set — the cross-skill prerequisite the Verifier flagged at T4 conditional-pass. Without this, every supersede-mode invocation of `specflow:complete` v1 refuses with the cross-skill-schema-unmet sentinel.

The recursive-bootstrap nature of this run means the Phase 3 build pipeline now has THREE tested PRD+SKILL artefacts (001-design-skill, 002-develop-skill, 003-complete-skill) plus four manifests (Gate 2 + Gate 3 + Gate 4 + Gate 5) demonstrating the chain handles Phase-3-domain features correctly. That's the strongest possible foundation for the remaining Phase 3 builds.

---

## Closing note

This dogfood proves three things the v2.1.0 release rests on. First: the `specflow:develop` Phase E + F orchestration (Gate 5 with Codex + Verifier with conditional-pass) works end-to-end — the chain produced a 472-line operational SKILL body covering all 14 PRD requirements without any gate failing or any reviewer escalation. Second: cross-provider review (Codex at Gate 5) is empirically load-bearing — Codex contributed 2 of 8 Gate 5 findings and one of those (codex-r1-f1, the schema-validation under-check) was a correctness defect the same-provider reviewers missed. Third: the push-back-on-simplicity-with-defended-trace-anchor pattern is durable across all four gates (002-G2, 002-G3, 003-G4, 003-G5) — the simplicity-vs-traceability calibration holds when the trace anchor is in the same artefact, regardless of whether that artefact is a PRD, a tasks file, a lane plan, or a SKILL body. v2.1.0 is release-ready.
