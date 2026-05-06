# Debate manifest — Gate 2: PRD vs interview review

**Feature:** 003-complete-skill
**Artefact under review:** `003-complete-skill-prd.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06

This is the dogfood Gate 2 run for the Phase 3 `specflow:complete` PRD. The PRD was synthesised first-pass from the interview (no hand-iteration before Gate 2 ran), so this manifest is the authentic adversarial pass through a real first-pass PRD on a Phase 3 retro skill — including five concern findings, two push-backs (one defended fully, one defended in part), and full convergence within three rounds.

---

## Round 1 — Findings

| Reviewer | Findings (severity) |
|---|---|
| simplicity-reviewer | 1 (concern) |
| surgical-reviewer | 1 (concern) |
| think-before-coding-reviewer | 2 (block, concern) |
| goal-driven-reviewer | 2 (block, concern) |
| devils-advocate | 1 (concern) |
| **Total** | **7 findings (2 block, 5 concern)** |

Detail:
- **simplicity-r1-f1** — *concern* — R10's `addenda.kind: 'note'` taxonomy and R7's elevation triple-flag (`elevation_offered`, `elevation_fired_by`, `elevation_outcome`) introduced without a documented second consumer (speculative configurability under Simplicity First sub-clause). (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — R4 + AC-4 implicitly modify `specflow:develop` Phase F.5's task-history.json schema by adding the `superseded_by_retro: false` default flag — Phase 2 cross-skill creep smuggled into a Phase 3 PRD. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *block* — R3 + AC-3's `escaped_issues` retro-time default of `count: 0` hides the load-bearing assumption that escaped issues surface post-retro and the AMEND path is the primary capture surface; downstream consumers cannot distinguish 'no-issues-known-yet' from 'no-issues-confirmed'. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **tbc-r1-f2** — *concern* — R4's 'replaces in place' framing eliminates the alternative of preserving Phase F values for `/insights`-cadence divergence detection without articulating the cost. (Same file.)
- **goal-r1-f1** — *block* — AC-6's lane-vs-outcome-divergence sentence verifies a corpus-signal contract that no R-level requirement establishes (orphan AC, reverse traceability failure under E4). (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — AC-7 lists five composed decision-log fields but the canonical shape is six (Date missing); a malformed entry would pass the AC. (Same file.)
- **da-r1-f1** — *concern* — R9's existence-check is a TOCTOU race when R1 auto-fire and R2 manual CLI fire concurrently on the same task-id; the binary 'has retro fields populated yet?' framing cannot detect an in-flight retro because retro fields are not written until interactive Q&A completes. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **push_back** (defended both edits — `addenda.kind: 'note'` cited interview Round 5 line 65's 'retroactive notes' resolution as the second-consumer evidence; elevation triple-flag defended on the basis that the three flags carry distinct semantic content the proposed enum collapse loses, with R7's last sentence naming cancellation flags as a `/insights`-cadence pattern signal)
- surgical-r1-f1 → **accept** (applied option (b): explicit cross-skill schema dependency call-out in R4 + AC-4)
- tbc-r1-f1 → **accept** (applied option (b): documentation-only Note in R3 articulating retro-time `escaped_issues = 0` as canonical default + AMEND as primary capture surface)
- tbc-r1-f2 → **accept** (R4 rationale extended to articulate the Phase-F-vs-retro divergence-signal tradeoff; v2 may preserve via `phase_f_*` mirror fields)
- goal-r1-f1 → **accept** (applied option (b): scoped AC-6's verification surface to the firing-condition contract; corpus-signal capture lives in schema appendix's `lane_assigned` + `blast_radius_outcome` fields)
- goal-r1-f2 → **push_back** (defended AC-7's five-field listing on the basis that Date is auto-populated by `specflow:decision`'s write-helper at write-time, not composed by `specflow:complete` from retro state; the canonical six-field shape is verified at the sister-skill boundary per the mirror pattern in PRD line 188)
- da-r1-f1 → **accept** (added new R14 + AC-14 specifying the lock-file pattern at `admin/scratch/complete-{task-id}.lock` with 30-minute stale-lock heuristic and structured refusal chat line)

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- simplicity-r1-f1 → **accept** (two-part defence held: retroactive-note use case is interview-grounded per Round 5 line 65; triple-flag's three flags carry distinct semantic content the enum collapse loses; principle bites on speculation, releases on interview-grounded second-consumer evidence)
- surgical-r1-f1 → **accept** (cross-skill dependency now named at both R-level and AC-level; mirrors 002-develop-skill AC-10 precedent)
- tbc-r1-f1 → **accept** (load-bearing assumption no longer hidden in PRD prose; documentation-only Note is the right calibration without v1 schema expansion; block resolved)
- tbc-r1-f2 → **accept** (R4 rationale rewrite makes the elimination defensible; pattern matches Phase 2 `specflow:develop`'s tbc-r1-f2 outcome)
- goal-r1-f1 → **accept** (AC-6's verification surface now scoped cleanly; reverse traceability clean; corpus-signal contract lives in schema appendix as the bindable surface; block resolved)
- goal-r1-f2 → **accept** (sister-skill mirror pattern is the authoritative framing of Date composition; adding Date to AC-7 would create a duplicate verification surface; concern resolved)
- da-r1-f1 → **accept** (R14 + AC-14 close the TOCTOU window; lock-file pattern matches Phase 1 scratch convention; stale-lock heuristic covers crashed-orchestration cleanup; concern resolved)

No sharpening occurred — every reviewer accepted the AI's Round 2 disposition (revisions applied or push-back-defended). No `ai-revision.md` needed in Round 3.

---

## Closing decision

**Gate 2 status: passed-with-revisions**

Two `block` findings landed in Round 1 — both accepted, both resolved with PRD revisions. Five `concern` findings landed — three accepted with revisions, two defended via push-back (one fully, one in part), and the principle reviewers accepted the defences in Round 3 as the right calibration. Five PRD revisions were applied between Round 2 and Round 3 (one R-level + AC-level pair, two prose-extension revisions, one verification-scope tightening, one new R + AC pair). The PRD is fit to proceed to `specflow:task`.

### PRD revisions applied

The PRD was edited between Round 1 and Round 3 to incorporate every accepted finding:

1. **R4 + AC-4 cross-skill schema dependency clause (concern surgical-r1-f1).** R4's Trace line extended with 'Extended at Gate 2 per surgical-r1-f1: explicit cross-skill schema dependency call-out for the `superseded_by_retro` default flag.' AC-4 carries the matching Schema dependency clause naming the `specflow:develop` Phase F.5 prerequisite. The cross-skill dependency is now named at both R-level and AC-level rather than buried in implementation; mirrors the 002-develop-skill AC-10 precedent.

2. **R3 prose extension articulating the load-bearing assumption (block tbc-r1-f1).** R3 extended with the canonical-default articulation: retro-time `escaped_issues.count: 0` is the canonical default; AMEND path is the primary capture surface; downstream consumers should treat retro-time `0` as 'unknown' until the entry has an `addenda` array entry confirming zero. The assumption is now observable to downstream readers without expanding the v1 schema. Open Questions section retains the `actual_hours` declined-to-estimate sentinel question as the adjacent capture-state nuance.

3. **R4 rationale extension articulating the divergence-signal tradeoff (concern tbc-r1-f2).** R4 extended with the tradeoff articulation: replace-in-place chosen for entry-shape simplicity; Phase-F-vs-retro divergence signal explicitly named as the cost; v2 may preserve Phase F values as `phase_f_*` mirror fields if `/insights`-cadence pattern detection surfaces a real divergence-signal consumer. No schema change in v1; just the rationale.

4. **AC-6 verification-surface scoping (block goal-r1-f1).** AC-6's verification surface scoped to the firing-condition contract; the lane-vs-outcome corpus-signal capture lives in the schema appendix's `lane_assigned` + `blast_radius_outcome` fields rather than being verified by AC-6. Reverse traceability is now clean — `specflow:test` building a plan against this PRD binds the lane-vs-outcome non-firing case to the schema appendix (a bindable surface), not to a sentence floating inside AC-6.

5. **New R14 + AC-14 concurrent-trigger guard (concern da-r1-f1).** Added R14 specifying the per-task lock-file pattern at `admin/scratch/complete-{task-id}.lock` with the 30-minute stale-lock heuristic and structured refusal chat line. AC-14 verifies the create/remove discipline and the stale-lock case. Open Questions section adds the `--clear-stale-locks` admin verb question as the v2 follow-up shape.

### Findings rejected after Round 3

Two findings had push-back outcomes the AI defended successfully:

- **simplicity-r1-f1 (full push-back on both edits).** The `addenda.kind: 'note'` taxonomy is interview-grounded per Round 5 line 65's resolution naming 'retroactive notes' alongside escaped-issue captures as legitimate amend payloads; collapsing the kind would push retroactive notes into kind-misuse or manual-edit territory (the latter explicitly forbidden by the same resolution). The elevation triple-flag's three fields carry distinct semantic content (did-the-logic-run? / which-condition-fired? / what-did-the-user-choose?) that R7's `/insights`-cadence cancellation-flag pattern detection requires; collapsing to a single enum loses the dimensional independence. Round-3 simplicity-reviewer accepted: principle bites on speculation, releases on interview-grounded second-consumer evidence.
- **goal-r1-f2 (push-back on adding Date to AC-7).** Date is auto-populated by `specflow:decision`'s write-helper at write-time, not composed by `specflow:complete` from retro state; PRD `See also` line 188 confirms the sister-skill mirror pattern. Adding Date to AC-7's listed fields would create a duplicate verification surface (this AC + `specflow:decision`'s own AC both checking Date) and contradict the mirror pattern. Round-3 goal-driven-reviewer accepted: sister-skill mirror pattern is the authoritative framing; a malformed entry missing Date is correctly the responsibility of `specflow:decision`'s ACs to catch.

### Findings escalated to human

None. All seven findings converged within three rounds.

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 2 reviewers and humans reading this as a **dogfood example** on a Phase 3 retro skill (smaller surface area than the Phase 2 `specflow:develop` calibration anchor at `002-develop-skill/debate-log/prd-gate2/manifest.md`):

- **Two `block` findings on a Phase 3 retro skill is a realistic shape.** Smaller skill surface, smaller PRD, but the lenses still bit — Think-Before-Coding flagged a hidden load-bearing assumption (escaped-issues post-retro capture state); Goal-Driven flagged an orphan AC (lane-vs-outcome corpus-signal contract not at R-level). Both are exactly the failure modes typical of first-pass PRDs that synthesise from interview without hand-iteration.
- **Two push-backs on five concerns is the right friction shape.** Accepting all seven would have been rubber-stamping; pushing back on more than two would have suggested the principles weren't biting. The two defended push-backs (simplicity's interview-grounded surface defence, goal-driven's sister-skill-boundary defence) are the calibration shape that says 'principles bite where evidence is weak, release where evidence is strong'. Same pattern as 002-develop-skill's split push-back on simplicity-r1-f1 (defend interview-grounded knob, drop speculative knob), generalised here to two distinct findings.
- **The race-condition finding (da-r1-f1) is the most architecturally interesting outcome.** R9's existence-check looked sufficient under sequential reasoning but failed under concurrent reasoning; Devil's Advocate's lens caught it because the dual-trigger-path contract (R1 auto-fire + R2 manual CLI) creates exactly the cross-artefact ambiguity DA fires on. The lock-file fix (R14 + AC-14) is a template for any future Phase 3 skill with multiple legitimate trigger paths — wherever a skill can be invoked from two surfaces concurrently, the existence-check protecting against double-invocation needs to be paired with an in-flight marker, not just a post-write field check.
