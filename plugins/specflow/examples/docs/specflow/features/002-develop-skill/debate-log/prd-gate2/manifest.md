# Debate manifest — Gate 2: PRD vs interview review

**Feature:** 002-develop-skill
**Artefact under review:** `002-develop-skill-prd.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06 16:20

This is the dogfood Gate 2 run for the Phase 2 `specflow:develop` PRD. The PRD was synthesised first-pass from the interview (no hand-iteration before Gate 2 ran), so this manifest is the authentic adversarial pass through a real first-pass PRD — including two `block` findings, one push-back, and the convergence path back to a passing PRD.

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
- **simplicity-r1-f1** — *concern* — R6 + R11 introduce two new `config.json.develop.*` config knobs without a documented second consumer (speculative configurability). (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — AC-10 implicitly modifies `specflow:misc --auto` payload schema (Phase 1 cross-skill creep). (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *block* — R5 + AC-8 lane-recheck mechanism relies on reviewer judgement, but no reviewer's lens natively owns lane-checking → AC-8 is non-firing in practice. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **tbc-r1-f2** — *concern* — R12's rationale claims Devil's Advocate covers cross-provider concerns in same-provider review, which is contradictory by definition. (Same file.)
- **goal-r1-f1** — *block* — AC-13 verifies a PRD-anchor format that no R-level requirement establishes (orphan AC). (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — Goal Outcome surface (h) names task-history.json development-time-field extension; no AC verifies it (goal-coverage gap). (Same file.)
- **da-r1-f1** — *concern* — Soft-dependency contract for agent-teams plugin covers absent path but not present-but-failing path. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **push_back** (split — defended R6's configurable cap, citing interview Round 2 line 78's user-edit naming two specific use cases; accepted R11's flag drop as speculative)
- surgical-r1-f1 → **accept** (applied option (b): explicit cross-skill schema dependency call-out in AC-10)
- tbc-r1-f1 → **accept** (applied option (a): added new R5.1 mechanical pre-Gate-4 lane-recheck; clarified R5 as catch-all path)
- tbc-r1-f2 → **accept** (rewrote R12 rationale to remove DA cross-provider over-claim)
- goal-r1-f1 → **accept** (added new R17 codifying PRD-anchor format; updated AC-13 Verifies)
- goal-r1-f2 → **accept** (added AC-15 verifying task-history.json development-time-field extension)
- da-r1-f1 → **accept** (extended R8 with present-but-failing clause; extended AC-6 with sub-clause)

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- simplicity-r1-f1 → **accept** (split outcome held: R11 drop + R6 defence is the right calibration; speculative-knob count reduced from 2 to 1, which is the simplicity win the principle was protecting)
- surgical-r1-f1 → **accept** (cross-skill dependency is now named, not buried in implementation)
- tbc-r1-f1 → **accept** (mechanical recheck R5.1 makes the trigger reliable without depending on a lens none of the standard five own; block resolved)
- tbc-r1-f2 → **accept** (manifest-note text remains the authoritative framing of degraded coverage)
- goal-r1-f1 → **accept** (R17 closes the orphan-AC gap; downstream `specflow:test` plan can now bind to the architectural rule via R-level expression; block resolved)
- goal-r1-f2 → **accept** (AC-15 closes the goal-coverage gap; new R-level requirement deemed unnecessary since the architectural Phase 3 scope item already establishes the schema)
- da-r1-f1 → **accept** (loud-fallback shape matches Phase 1 `specflow:design` precedent)

No sharpening occurred — every reviewer accepted the AI's Round 2 disposition (revisions applied or push-back-defended). No `ai-revision.md` needed in Round 3.

---

## Closing decision

**Gate 2 status: passed**

Two `block` findings landed in Round 1 — both accepted, both resolved with PRD revisions. Five `concern` findings landed — four accepted with revisions, one defended in part (the split push-back on the configurable green-batch cap, accepted by Simplicity in Round 3 as the right calibration).

### PRD revisions applied

The PRD was edited between Round 1 and Round 3 to incorporate every accepted finding:

1. **R5 + new R5.1 (block tbc-r1-f1).** R5 reframed as the reviewer-driven catch-all path; new R5.1 codifies a mechanical pre-Gate-4 lane recheck (file-count comparison, module comparison, confidential-path re-glob). The lane-recheck mechanism is now mechanically enforced, not reviewer-judgement-dependent. AC-8 was rewritten to verify both paths fire correctly.
2. **R8 + AC-6 (concern da-r1-f1).** R8 extended with a present-but-failing clause for the agent-teams plugin (loud-fallback to single-specialist + chat-line warning); AC-6 extended with a verification sub-clause for the team-spawn-failed log line.
3. **R11 + AC-7 (concern simplicity-r1-f1, partial).** Dropped the `config.json.develop.codexAtGate4` knob — Codex is hard-coded to Gate 5 only in v1; consumers wanting Gate-4 Codex review can request it as a v2 enhancement. AC-7 simplified to drop the conditional second clause.
4. **R12 (concern tbc-r1-f2).** Rationale rewritten to remove the over-claiming "DA covers cross-provider concerns" phrasing; the manifest-note text remains the authoritative framing of degraded coverage.
5. **New R17 + AC-13 update (block goal-r1-f1).** Added R17 codifying the PRD-anchor format as an R-level requirement (closing the orphan-AC gap); AC-13's `Verifies:` line updated from "R10's PRD anchor cross-check" to "R17". The architectural Phase 2 scope item 7 rule now has R-level expression in this PRD body.
6. **New AC-10 dependency clause (concern surgical-r1-f1).** AC-10 sharpened to add an explicit cross-skill schema dependency: AC-10 depends on `specflow:misc --auto` accepting `manifest_path` + `gate_finding_id` as named fields. If the current Phase 1 schema doesn't include these, a separate `specflow:misc` enhancement PRD lands first.
7. **New AC-15 (concern goal-r1-f2).** Added AC-15 verifying that on task completion the skill appends/updates the `admin/task-history.json` entry with the Phase 3 scope item 1 development-time fields. The goal Outcome surface (h) is now verified.

### Findings rejected after Round 3

One finding had a split outcome where the AI defended part:
- **simplicity-r1-f1 (R6's configurable cap)** — interview Round 2 (line 78) records the user explicitly editing the AI's recommended default of 5 down to 3, naming two specific use cases (low-CI projects wanting smaller batches, high-CI projects wanting larger). That is the documented consumer-ask the Simplicity sub-clause requires; configurability is grounded in the interview, not speculation. The Round-3 reviewer accepted the split — drop the genuinely speculative R11 knob, retain the interview-grounded R6 knob. Net speculative-knob count: 1 (down from 2 in the first-pass PRD), which is the simplicity win the principle was protecting.

### Findings escalated to human

None. All seven findings converged within three rounds.

The PRD is fit to proceed to `specflow:task`. No revisions to the interview's Goal section were required (no scope-change triggered). The mechanical lane-recheck (R5.1) is the most load-bearing addition — without it, the lane-aggressive-flag mechanism R5 + AC-8 promised would not have fired in practice.

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 2 reviewers and humans reading this as a **dogfood example** (vs. the hand-iterated calibration anchor at `001-design-skill/debate-log/prd-gate2/manifest.md`):

- **Two `block` findings is the realistic shape for a first-pass PRD.** This run surfaced a missing R-level requirement (goal-r1-f1 — PRD-anchor format) and an unstated load-bearing assumption (tbc-r1-f1 — reviewer lens for lane-checking). Both are exactly the failure modes the SESSION-HANDOFF.md called out as typical first-pass surfaces (line 132). The hand-iterated 001-design-skill PRD had zero `block` findings because it was already iterated by hand — that's not the realistic baseline.
- **Each reviewer fired its lens distinctly.** Simplicity flagged speculative configurability (two new config knobs); Surgical flagged cross-skill creep (AC-10 implicitly editing `specflow:misc`'s schema); Think-Before-Coding flagged an unstated load-bearing assumption (the lane-recheck lens) plus a self-contradicting rationale (DA covering cross-provider in same-provider); Goal-Driven flagged an orphan AC and a goal-coverage gap; Devil's Advocate flagged a degraded-path coverage gap (plugin present-but-failing). No two reviewers flagged the same finding.
- **The push-back was defensible and accepted.** The split response on simplicity-r1-f1 (defend R6's interview-grounded cap, accept R11's speculative flag drop) is the right shape — push back where the principle's evidence threshold is met, accept where it isn't. Accepting both knobs would have been rubber-stamping; pushing back on both would have ignored the Simplicity sub-clause's literal language.
- **No findings escalated.** All seven converged within three rounds, which is the target convergence rate. If a real run hits the 3-round cap with reviewers and AI not converging, escalation surfaces the disagreement to the human — that didn't happen here, but the manifest format documents it as a possibility.
- **The mechanical-vs-reviewer-judgement split for the lane recheck (R5.1 vs R5)** is the most architecturally interesting outcome. Phase 2 will likely surface similar splits — wherever the PRD relies on reviewer judgement for a structurally-checkable property, Think-Before-Coding's "name the unstated assumption" lens should bite. R5.1 is a template for resolving that pattern.
