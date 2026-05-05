# Debate manifest — Gate 2: PRD vs interview review

**Feature:** 001-design-skill
**Artefact under review:** `001-design-skill-prd.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-05 14:50

This is the worked example of Gate 2 in operation. Five reviewers fired in parallel (forked sub-agents), the AI responded in Round 2, reviewers sharpened-or-accepted in Round 3, and this manifest is the Orchestrator's closing entry. Use this as the calibration anchor for what a healthy Gate 2 looks like — neither rubber-stamping nor bikeshedding.

---

## Round 1 — Findings

| Reviewer | Findings (severity) |
|---|---|
| simplicity-reviewer | 1 (concern) |
| surgical-reviewer | 1 (concern) |
| think-before-coding-reviewer | 2 (concern, note) |
| goal-driven-reviewer | 2 (concern, note) |
| devils-advocate | 1 (concern) |
| **Total** | **7 findings** |

Detail:
- **simplicity-r1-f1** — *concern* — R10 duplicates the spirit of R6's `--default-when-missing` flag specification. (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — R12 traces to "codebase context (pre-grilling)" rather than a Resolved line, breaking the trace pattern. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *concern* — R7's 5% threshold ships without articulating that per-region or per-component overrides were viable alternatives. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **tbc-r1-f2** — *note* — R11's "silent skip when Codex absent" frames an ambiguous case as a single decision. (Same file.)
- **goal-r1-f1** — *concern* — No AC verifies the iteration log's *Why* field, despite PRD Appendix C3.1 making it a hard rule. (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *note* — AC-9 verifies presence-of-section but not fidelity (Codex finding count). (Same file.)
- **da-r1-f1** — *concern* — R11 says "the mockup" (singular) but R2 produces two HTMLs; reviewer cannot tell which Codex reviews. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **push_back** (cited interview line 63 — Round 5's "but add" phrasing signals separate decision point)
- surgical-r1-f1 → **accept** (applied option (b): explicit trace + reason for pattern-break)
- tbc-r1-f1 → **accept** (added tradeoff sentence to R7; extended interview's "Topics not discussed")
- tbc-r1-f2 → **accept** (added alternative-naming note to R11)
- goal-r1-f1 → **accept** (added AC-11)
- goal-r1-f2 → **push_back** (cited Phase 1 scope item 15 — verify-mechanism belongs in SKILL.md, not the AC)
- da-r1-f1 → **accept** (specified Codex reviews proposed.html against current.html as baseline)

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- simplicity-r1-f1 → **accept** (AI's defence held; trace-integrity argument cited verbatim from Round 5)
- surgical-r1-f1 → **accept** (revision applied; pattern-break now self-documented)
- tbc-r1-f1 → **accept** (revision applied; tradeoff articulated, alternative deferred)
- tbc-r1-f2 → **accept** (revision applied; multi-interpretation framing now in PRD)
- goal-r1-f1 → **accept** (AC-11 added; coverage gap closed)
- goal-r1-f2 → **accept** (AI's defence held; over-specification concern conceded)
- da-r1-f1 → **accept** (revision applied; cross-artefact ambiguity resolved)

No sharpening occurred. No `ai-revision.md` needed.

---

## Closing decision

**Gate 2 status: passed**

5 of 7 findings were accepted by the AI and revisions applied to `001-design-skill-prd.md`:
- R7 gained a tradeoff articulation about per-region thresholds being a deferred v2 candidate.
- R11 was sharpened to specify Codex reviews `proposed.html` against `current.html` (as baseline), AND named the silent-skip alternative.
- R12's trace was strengthened with an explicit citation to the codebase-context bullet plus a reason for its different shape.
- AC-11 was added verifying the iteration log's *Why* field is non-empty (closing the gap to PRD Appendix C3.1).
- The interview's "Topics not discussed" section gained the per-region-threshold deferral.

2 findings were rejected after Round 3 with the reviewer accepting the AI's defence:
- **simplicity-r1-f1** (R10/R6 merge) — interview Round 5's "but add" phrasing established R10 as a separate decision point, not an elaboration on R6. Trace integrity beats requirement-count reduction.
- **goal-r1-f2** (AC-9 stdout-counting) — fidelity-counting is implementation-level (verify mechanism in SKILL.md), not contract-level (PRD AC). Locking AC-9 to one mechanism over-specifies the spec.

No findings escalated to human decision.

The PRD is fit to proceed to `specflow:task`. No revisions to the interview's Goal section were required (no scope-change triggered).

— Orchestrator, 2026-05-05

---

## Calibration notes

For future Gate 2 reviewers and humans reading this as an example:

- **Healthy Gate 2 looks like this** — 5-10 findings spread across reviewers, most with `concern` severity, a mix of accept and push-back in Round 2, convergence in Round 3. Zero `block` findings is a good outcome on a well-grilled PRD; a sea of `block` would mean the interview wasn't doing its job.
- **Each reviewer fired its lens** — Simplicity flagged duplication; Surgical flagged trace integrity; TBC flagged missing tradeoff articulation; Goal-Driven flagged a coverage gap; Devil's Advocate flagged cross-artefact ambiguity. None overlapped much.
- **The AI's two pushbacks were defensible and accepted** — push-back is healthy when grounded in the interview / scope items. Accepting every finding wholesale would be rubber-stamping in reverse.
- **No `block` findings on this PRD** is plausible because the worked example was already iterated by hand before this Gate 2 ran. Real first-pass PRDs typically surface 1-2 `block` findings — a missing AC on a load-bearing requirement, a non-binary acceptance check, a requirement that contradicts the goal's out-of-scope list. Watch for those.
