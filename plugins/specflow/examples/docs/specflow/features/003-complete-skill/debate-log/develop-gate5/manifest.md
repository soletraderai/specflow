# Debate manifest — Gate 5: code vs plan review

**Feature:** 003-complete-skill
**Gate:** develop-gate5
**Artefact under review:** `plugins/specflow/skills/complete/SKILL.md` (472 lines, 8 phases A-H)
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate, codex-reviewer
**Codex:** present (v0.18.0 detected per `admin/environment.json` cli.codex.detected:true)
**Date:** 2026-05-06
**Closed:** 2026-05-06

This is the Phase 2 dogfood Gate 5 run for the `specflow:complete` SKILL body. Phase D (lane execution) produced the 472-line SKILL body covering all 14 PRD requirements (R1-R14) plus AC-15 (per T15's documented orphan-AC exception). Gate 5 reviewers fired against the produced SKILL body, the per-task plans, and the lane-assignments.json artefact. Codex joined as the sixth reviewer per `skills/develop/SKILL.md` Phase E spec. Eight `concern` findings landed across six reviewers; one defended push-back accepted in Round 3; one Round-3 sharpening on Codex's schema-validation finding; the rest accepted with revisions applied to the SKILL body between rounds.

---

## Context

### Artefact under review

`plugins/specflow/skills/complete/SKILL.md` — 472 lines. 8 phases A-H. Header `requires` covers the PRD, tasks file, and existing `task-history.json`. Header `produces` covers the appended `task-history.json` entry, the optional `decision-log.md` elevation, and the per-task lock at `admin/scratch/complete-{task-id}.lock`. Eval block names the binary schema-validation check, the idempotent re-invocation contract, and the elevation-write trace.

### Codex availability (from `admin/environment.json`)

- **Codex CLI:** detected (v0.18.0 at `/usr/local/bin/codex`). Gate 5 invoked Codex as the sixth reviewer per `skills/develop/SKILL.md` Phase E.2 spec. Codex Round-1 findings cover semantic-correctness (E.4 schema validation under-check) and path-orthogonality (lock path vs `specflow:develop` scratch cleanup boundary). The cross-provider lens is load-bearing here: codex-r1-f1 caught a correctness defect — E.4 validates one direction of the allow-list (rejects extraneous, accepts missing) — that the same-provider reviewers did not flag.
- **agent-teams plugin:** absent. Phase D ran sequential single-specialist invocation (per Gate 4 manifest's environment block).
- **Linear MCP:** detected. Phase F.4 fired Linear status transitions; Phase G of `specflow:complete` is documented at line 358 with implementer-inference annotation per goal-r1-f1 revision.

### Lane execution outcome (input to Gate 5)

13 Green tasks shipped via 5-batch ladder per the Gate 4 lane plan (cap=3): batch-1 (T2 + T3 + T4), batch-2 (T5 + T6 + T11), batch-3 (T7 + T8 + T9), batch-4 (T10 + T12 + T14), batch-5 (T15). T1 + T13 ran via Yellow-lane HITL pairing. Zero machine-check failures during execution (per `feature_lane_outcome.machine_check_failures: 0` in lane-assignments.json). Zero accept-and-ship tasks. The SKILL body assembled from all 15 per-task patches with no merge conflicts.

---

## Round 1 — Findings

| Reviewer | Findings (severity) |
|---|---|
| simplicity-reviewer | 1 (concern) |
| surgical-reviewer | 1 (concern) |
| think-before-coding-reviewer | 1 (concern) |
| goal-driven-reviewer | 2 (concern, concern) |
| devils-advocate | 1 (concern) |
| codex-reviewer | 2 (concern, concern) |
| **Total** | **8 findings (0 block, 8 concern)** |

Detail:
- **simplicity-r1-f1** — *concern* — Triple-flag pattern (`elevation_offered`, `elevation_fired_by`, `elevation_outcome`) materialises across four phases of the SKILL body; ~30 lines (6.4% of body) spent on flag bookkeeping where the consumer schema is undocumented. Defensible bloat or premature abstraction? (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — Phase F.5 binds the `decision-log.md` write format to a sister Phase 3 skill (`specflow:decision`) that does not yet exist; cross-skill scope creep / forward-binding to an unwritten artefact. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *concern* — Phase A.3's 30-minute stale-lock heuristic ships hard-coded with no calibration evidence; PRD Open Questions surfaced the issue but the SKILL body did not propagate as a documented assumption. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **goal-r1-f1** — *concern* — Phase G (Linear status sync) has no R-anchor in the PRD; orphan phase inferred from `specflow:develop` Phase F.4 calibration. Reverse-traceability lens (per E4 prompt-edit calibration). (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — H.4.4 verify step refers to 'one of the five sentinel failure lines' but H.3 enumeration count was not numbered; AC-15 verify-step-to-sentinel-list binding gap. (Same file.)
- **da-r1-f1** — *concern* — Cross-cutting concurrency window between Phase E.5 disk write and Phase H.1 lock release where `/insights` (concurrent reader) could observe entries mid-mutation; SKILL body did not document the window. (See `findings/round-1/devils-advocate.json`.)
- **codex-r1-f1** — *concern* — E.4 schema validation enforces only one direction of the allow-list (rejects extraneous, accepts missing required fields). Correctness defect — malformed entries missing required fields would pass validation. (See `findings/round-1/codex-reviewer.json`.)
- **codex-r1-f2** — *concern* — Lock path orthogonality with `specflow:develop` Phase F.6 cleanup is correct but undocumented; future skill authors might move the lock path inside the develop-scratch pattern and break the F.5 → F.6 boundary survival contract. (Same file.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **push_back** (defended the triple-flag pattern as Gate 2 push-back outcome with documented `/insights`-consumer rationale at SKILL.md:190-196; option (a)'s forward-binding to unwritten consumer schema would create the same shape Gate 5 surgical-r1-f1 flags; the SKILL body already documents the flag earnings)
- surgical-r1-f1 → **accept** (applied option (a) — reframed F.5 prescriptive format as tentative pending `specflow:decision` SKILL.md ship; cross-skill integration section extended with format-binding-deferred clause)
- tbc-r1-f1 → **accept** (applied option (b) — added rationale clause at A.3 case 3 documenting the 30-minute calibration choice; v2-deferral pointer to PRD Open Question for the configurable-knob path)
- goal-r1-f1 → **accept** (applied option (b) — added documented-inference note at Phase G header naming the calibration anchor and the PRD-scope-change ratification path)
- goal-r1-f2 → **accept** (numbered enumeration confirmed at H.3; per-H.3-numbered-list cross-reference added at H.4.4)
- da-r1-f1 → **accept** (applied option (a) — documented the concurrency window at Cross-skill integration `/insights` bullet with consumer-side snapshot recommendation; v2 deferral to single-write-transaction conditioned on consumer reports)
- codex-r1-f1 → **accept** (added third E.4 check — required-field-presence with structured-failure sentinel; closed asymmetric-validation gap)
- codex-r1-f2 → **accept** (added one-line note at A.3 lock-path declaration documenting the path-orthogonality with `specflow:develop` Phase F.6)

## Round 3 — Reviewers sharpen or accept

Seven of eight findings converged accepting the Round 2 disposition. One Codex finding sharpened:

- simplicity-r1-f1 → **accept** (push-back defended; SKILL.md:190-196 documents the triple-flag earnings against the documented `/insights` consumer rationale; mirrors 002-develop-skill Gate 3 simplicity-r1-f1 precedent)
- surgical-r1-f1 → **accept** (tentative-format clause + format-binding-deferred clause are the surgical fix; future Phase 3 skills can reuse the deferred-binding shape)
- tbc-r1-f1 → **accept** (calibration documented; v2-deferral path correctly preserves Simplicity-First in v1)
- goal-r1-f1 → **accept** (documented-inference shape mirrors 002-develop-skill Gate 2 R17-codification pattern at lower cost than minting a new R mid-Gate-5)
- goal-r1-f2 → **accept** (five-tuple sentinel binding mechanically traceable from AC-15 to source enumeration)
- da-r1-f1 → **accept** (cross-cutting concern surface explicitly named; future Phase 3 producer skills should adopt the same documented-window pattern)
- codex-r1-f1 → **sharpen** (the revision text says 'required fields present' but does not pin the semantic of 'required' versus 'has a non-default value'; sharpen: clarify that 'required' means 'present in the entry, possibly with default value (empty array, false boolean, zero number) per the Schema Appendix's documented defaults' — prevents future maintainers from over-tightening the check)
- codex-r1-f2 → **accept** (path-orthogonality documented; F.5 → F.6 boundary survival now observable from the SKILL body itself)

## Code revisions applied

The SKILL body (`plugins/specflow/skills/complete/SKILL.md`) was edited between Round 1 and Round 3 to incorporate every accepted finding:

1. **F.5 + Cross-skill integration tentative-format clause (concern surgical-r1-f1).** Phase F.5 prescriptive format reframed as 'tentative — to be confirmed when `specflow:decision` SKILL.md ships'. Cross-skill integration `specflow:decision` bullet extended with 'format binding deferred per Gate 5 surgical-r1-f1'.

2. **A.3 case 3 calibration clause (concern tbc-r1-f1).** Stale-lock chat-line extended with parenthetical rationale: '30-minute threshold chosen as 2-6× typical successful-run duration to bound false-positive overwrites; revisit if consumers report walking-away-mid-retro lock loss; v2 enhancement may surface as `config.json.complete.staleLockMinutes` per PRD Open Questions'.

3. **Phase G documented-inference note (concern goal-r1-f1).** New paragraph at Phase G header naming the calibration anchor (`specflow:develop` Phase F.4) and the PRD-scope-change ratification path; preserves cross-skill ecosystem consistency without dropping the phase.

4. **H.3 / H.4.4 sentinel binding (concern goal-r1-f2).** Numbered enumeration at H.3 (1-5) confirmed; H.4.4 cross-reference 'per H.3 numbered list' added to make AC-15 binding mechanical.

5. **Cross-skill integration concurrency note (concern da-r1-f1).** New clause at `/insights` bullet documenting the eventually-consistent read window and consumer-side snapshot recommendation.

6. **E.4 third check — required-field-presence (concern codex-r1-f1, sharpened in Round 3).** Required-field-presence check added with structured-failure sentinel. Round-3 sharpen: clarified 'required' means 'present, possibly with default value per Schema Appendix' to prevent over-tight validation in future maintenance.

7. **A.3 lock-path orthogonality note (concern codex-r1-f2).** One-line clause added documenting the lock path is intentionally outside `admin/scratch/{NNN-slug}-develop/` and survives the F.5 → F.6 boundary that auto-fires this skill.

## Findings rejected after Round 3

One finding had a defended push-back accepted in Round 3:

- **simplicity-r1-f1 (push-back on the triple-flag pattern).** The Gate 2 push-back outcome (defended at Gate 2 Round 3 with `/insights`-consumer-rationale) holds at Gate 5. The SKILL body at lines 190-196 explicitly documents 'each carries distinct semantic content (offered-vs-fired-vs-outcome), needed for /insights-cadence pattern detection. Collapsing to a single flag would lose signal.' That sentence IS the documentation the Gate 5 finding asked for — the consumer rationale is in the SKILL body, not forward-bound to an unwritten schema (which would be the same shape Gate 5 surgical-r1-f1 flags). The simplicity-vs-traceability calibration mirrors 002-develop-skill Gate 3 simplicity-r1-f1 precedent (T15 cross-cutting contract): simplicity at the artefact layer cannot override traceability anchored in a load-bearing trace surface.

## Findings escalated to human

None. All eight findings converged within three rounds.

## Closing decision

**Gate 5 status: passed-with-revisions**

Seven of eight findings were accepted by the AI and revisions applied to `plugins/specflow/skills/complete/SKILL.md` (F.5 tentative-format clause, A.3 calibration clause, Phase G documented-inference note, H.3 numbered enumeration, Cross-skill concurrency note, E.4 required-field-presence check, A.3 lock-path orthogonality note). One finding (codex-r1-f1) was sharpened in Round 3 (clarified 'required' semantic to prevent over-tight future validation). One finding (simplicity-r1-f1) had a defended push-back accepted in Round 3, citing the Gate 2 simplicity-r1-f1 push-back precedent and the SKILL body's documented `/insights`-consumer rationale. No findings escalated to human decision. Codex contributed two of the eight findings (codex-r1-f1 caught a correctness defect — schema validation under-check — that same-provider reviewers did not flag; codex-r1-f2 caught a path-orthogonality assumption the SKILL body left undocumented), justifying the cross-provider review surface per `skills/develop/SKILL.md` Phase E spec. The SKILL body is fit to proceed to Phase F (Verifier confirmation + PR open + Linear sync + task-history append). The post-Gate-5 SKILL body now satisfies all 14 PRD R-IDs plus AC-15 with reverse-traceability holding (Phase G's documented-inference clause closes the only orphan-phase risk).

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 5 reviewers and humans reading this as a **dogfood example** (the first full Phase 2 dogfood Gate 5 against a fully-revised SKILL body with Gate 2 + Gate 3 + Gate 4 findings already applied):

- **Eight concerns / zero blocks / one push-back / one sharpen is the right shape for a Phase 3 retro skill's SKILL body** when the upstream PRD (Gate 2), tasks file (Gate 3), and lane plan (Gate 4) have already been pre-tightened. Gate 5 findings fire on code-layer ambiguities (triple-flag bookkeeping volume, cross-skill format binding to unwritten sibling, magic-number documentation, orphan-phase reverse-traceability, sentinel-list binding granularity, concurrency window between producer and consumer, schema-validation under-check, path-orthogonality assumption) rather than missing R-implementation or wholly-broken phases. No finding earned a `block` because the upstream gates' work held.

- **Codex contributed 2 of 8 findings (25%) — both load-bearing.** codex-r1-f1 caught a correctness defect (E.4 schema validation under-check) that all five same-provider reviewers missed; the cross-provider lens is justified in the empirical record. codex-r1-f2 caught a path-orthogonality assumption that documents an architectural property of the producer-consumer pair (`specflow:complete` lock path vs `specflow:develop` scratch cleanup). The Codex sharpening in Round 3 (clarifying 'required' semantic) is the cross-provider review pattern at its sharpest — Codex saw both the under-check AND the next-level over-tightening risk that would surface if a future maintainer interpreted 'required' as 'has non-default value'. The orchestrator-pattern doc on cross-provider review predicts this shape; this gate confirms it.

- **The push-back precedent generalises across all four gates.** 002-develop-skill Gate 2 push-back on simplicity-r1-f1 (R6 interview-grounded cap), 002-develop-skill Gate 3 push-back on simplicity-r1-f1 (T15 cross-cutting contract), 003-complete-skill Gate 4 push-back on simplicity-r1-f1 (`b1_recheck.lane_changes: []` audit-trail field), and now Gate 5 push-back on simplicity-r1-f1 (triple-flag pattern with documented `/insights`-consumer rationale) all share the same shape: simplicity instinct to drop a field/section at the artefact layer, defended by citing a load-bearing trace anchor in the SAME artefact (interview line for cap; cross-cutting contract for T15; audit-trail-signal for `lane_changes`; consumer-rationale paragraph for triple-flag). The pattern is durable across Gates 2-5.

- **Goal-Driven was load-bearing here when Codex absent at Gate 4 and load-bearing-shared with Codex at Gate 5.** The role file's 'load-bearing at Gate 5 when Codex absent' framing extends naturally to 'load-bearing-shared with Codex when Codex present' — Goal-Driven owns the AC-to-code reverse-traceability lens (goal-r1-f1 + goal-r1-f2) while Codex owns the semantic-correctness lens (codex-r1-f1 + codex-r1-f2). The two lenses overlap at the AC-to-code boundary; future role-file revisions should document the overlap so reviewers do not double-fire.

- **Phase 2 dogfood Gate 5 succeeded.** This Gate 5 manifest is the dogfood of `specflow:develop` Phase E against the just-built `specflow:complete` Phase 3 retro skill. The setup phase (E.1) created the debate folder; the reviewer dispatch (E.2-E.3) fired six reviewers including Codex in parallel forked sub-agents; the AI response phase (E.4) produced one push-back and seven accepts; Round 3 (E.5) produced one sharpen + seven accepts; the closer (E.7) produced this manifest with passed-with-revisions status. The orchestrator pattern (forked sub-agents, JSON finding files, manifest closer) held end-to-end. Calibration anchor for future Phase 2 dogfood Gate 5 runs on subsequent Phase 3 skills (`specflow:decision`, `specflow:scope-change`, `/insights`, `/prune`, `/optimize`).
