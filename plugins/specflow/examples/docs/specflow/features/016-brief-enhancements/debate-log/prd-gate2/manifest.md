# Gate 2 — PRD vs interview review

**Feature:** 016-brief-enhancements
**Date:** 2026-05-07
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate, codex

## Accepted findings

- **simplicity-r1-f3** (simplicity-reviewer, concern) — AC-3 over-prescribed prose-tightening tactics inside the AC body.
  - Evidence: PRD AC-3 prescribed `(a) collapse the four block-grammar examples into a shared shape … (b) extract CSS comment headers`.
  - Revision applied: AC-3 demoted to a single binary `wc -l ≤ 500` check; prescriptive tactics moved to non-binding implementation notes; baseline 524-line reality + chain-don't-absorb fallback surfaced in Open Questions.

- **SURG-1** (surgical-reviewer, block) — AC-3 was smuggling a refactor of the four existing blocks under the cover of a line cap.
  - Evidence: AC-3 mandated tightening passes on `:::flow / :::comparison / :::scope / :::tree` content the user did not request.
  - Revision applied: Non-goals now declares the existing four blocks untouched; AC-3 is a binary check; chain-don't-absorb path lives in OQ-1.

- **SURG-2** (surgical-reviewer, block) — AC-3 carried a contingent in-PRD escape hatch.
  - Revision applied: Conditional clause stripped; chain-don't-absorb risk surfaced as OQ-1.

- **SURG-3** (surgical-reviewer, concern) — Slack silently dropped from interview's five-icon set without callout.
  - Revision applied: R2 trace cites decision-log 2026-03-22 (*Slack removed from the stack*) + CONTEXT.md Stack-section Slack-free confirmation.

- **TBC-1** (think-before-coding-reviewer, block) — 524-line baseline was the load-bearing unstated assumption behind R8/AC-3.
  - Revision applied: R8 explicit baseline + chain-don't-absorb fallback + measured budget. OQ-1 (Round-3-extended) names in-scope tightening targets so the line arithmetic is auditable.

- **TBC-2** (think-before-coding-reviewer, block) — Slack drop tradeoff unstated. (Same revision as SURG-3.)

- **TBC-4** (think-before-coding-reviewer, concern) — R9's "consistent across all eight blocks" was factually wrong about the baseline.
  - Revision applied: R9 restated — new four follow `:::flow`'s most-complete pattern; existing four unchanged.

- **TBC-5** (think-before-coding-reviewer, concern) — Icon format/source/byte cost were silent assumptions.
  - Revision applied: R2 extended with three sub-clauses — SVG-base64 format; author-drawn monochrome glyphs (avoids brand-mark licensing); each icon ≤1 line in SKILL.md.

- **GDR-1 through GDR-6** (goal-driven-reviewer, blocks/concerns) — Multiple ACs were soft / verb-level / not actually binary.
  - Revisions applied: AC-1 / AC-2 rewritten as for-loops with grep; AC-3 binary line-count; AC-4 four selector-grep assertions with frozen class names; AC-5 cmp-s; AC-6 pinned format regex + base64-decode; AC-7 DOM-scoped Python assertion; AC-8 hardened regex; AC-10 column-count + class-grep; AC-11 three executable sub-assertions; AC-12 source-order Python; AC-13 strip rule for all four kinds via DOM-scope; AC-14 subtitle= rendering; AC-15 eval enumeration check; AC-19 icon-override fallback recording; new R11 (eval enumeration), R12 (four-callsite lockstep), R13 (icon-override validation).

- **DA-1, DA-2, DA-3, DA-4** (devils-advocate, blocks/concerns) — 524-line baseline reality (DA-1); four hardcoded list call-sites must update in lockstep (DA-2); Slack drop silence (DA-3); TOC inflation from new headings (DA-4).
  - Revisions applied: OQ-1 carries the 524 → ≤500 arithmetic + chain-don't-absorb path with named candidate ranges (Round-3-extended per DA-r3-f1); R12 + AC-16 enumerate the four call-sites with lockstep checks; R2 trace cites the Slack decision; R9 pins block titles to `<div class="block-title">` (not `<h2>`/`<h3>`) + AC-17 verifies titles do not enter the TOC.

- **codex-r1-f1** (codex, concern) — Mobile breakpoint participation missing.
  - Revision applied: AC-18 added; **Round 3 sharpened** to cover BOTH 1100px AND 860px breakpoints AND a Playwright headless 360px render (no horizontal overflow). Mobile-warehouse-operator profile audience.

- **codex-r1-f2** (codex, concern) — AC-6 icon format under-specified.
  - Revision applied: AC-6 pinned to `data:image/(svg+xml|png);base64,`; **Round 3 sharpened** to base64-decode + SVG-or-PNG byte validation.

- **codex-r1-f3** (codex, concern) — `icon=` override validation/fallback missing.
  - Revision applied: R13 + AC-19 specify validation (data:image/(svg+xml|png);base64 only), HTML-escape, fallback-to-pill on malformed, source-comment audit trail.

- **codex-r1-f4** (codex, concern) — Field emission order unpinned.
  - Revision applied: R6 amended — emission order fixed by grammar, not map iteration.

- **codex-r1-f5** (codex, concern) — AC-7 DOM-scope ambiguous.
  - Revision applied: AC-7 rewritten as Python DOM-scoped assertion ignoring matches inside `<style>` blocks.

- **codex-r1-f6 + DA-5** (codex + devils-advocate, notes) — AC numbering out of order (AC-11 before AC-9, AC-10).
  - Revision applied: AC numbering renumbered sequentially (AC-1 through AC-19).

## Rejected findings

- **simplicity-r1-f1** (simplicity-reviewer, concern) — R9 title=/subtitle= speculative configurability.
  - Reason for rejection: Interview "Topics not discussed" explicitly resolved this — *"Assumed yes — consistency with the existing four blocks."* User-confirmed at sign-off. Round 3 reviewer pushed back a third time; the orchestrator stands on the user's interview decision.

- **simplicity-r1-f2** (simplicity-reviewer, concern) — R2 icon override speculative.
  - Reason for rejection: Interview Round 3 verbatim — *"explicit `icon=` attribute on a resource line overrides."* User-confirmed.

- **simplicity-r1-f4** (simplicity-reviewer, note) — R5 + AC-11 redundant with existing block contract.
  - Reason for rejection: R5 + AC-11 are the load-bearing consistency line for the new blocks. Per Goal-Driven Reviewer's mandatory eval-coverage matrix, every requirement must trace to a binary AC; collapsing them would create a coverage gap.

- **SURG-4** (surgical-reviewer, concern) — R10 defensive restatement.
  - Reason for rejection: R10 binds the new blocks to the existing unsupported-kind fallback; without it, the strip-rule's surface widening is undocumented.

- **SURG-5** (surgical-reviewer, note) — R5 + AC-11 selective re-statement.
  - Reason for rejection: Selective re-statement of high-leverage edge cases is the right cut; the symmetric reading would explode the AC list.

- **TBC-3** (think-before-coding-reviewer, concern) — R7 strip rule for `:::phase-split` differently from `:::key-features`.
  - Reason for rejection: Interview Round 1 sign-off covers all four new blocks rendering in Visual abstract; treating phase-split differently breaks per-kind symmetry.

## Escalated to human

- **simplicity-r3-f1** (simplicity-reviewer, concern) — Walking back R9's user-confirmed `title=`/`subtitle=` decision.
  - Reason: reviewer and writer did not converge in 3 rounds; reviewer wants a feature-level decision the user already made in interview "Topics not discussed".
  - Recommendation: stand on the user's interview sign-off. If the user wants to revisit the assumption, route through `specflow:scope-change`, not Gate 2.

- **surgical-r3-f1** (surgical-reviewer, concern) — Frame chain-don't-absorb as expected outcome rather than contingency.
  - Reason: reviewer and writer agree on the substantive content; framing-level disagreement.
  - Recommendation: OQ-1's named candidate ranges + estimated savings already make the likely outcome auditable. Accept Gate 2 as-is; framing can be revised in 022's cross-task-review pass.

## Closing decision

Gate 2 status: **passed-with-escalations**

PRD revisions applied across 28 of 32 Round-1 findings; four push-backs cited interview-confirmed decisions. Round 3 added two more sharpens (codex on mobile + icon validation) which were accepted and applied; one Round-3 sharpen (simplicity-r3-f1, third pass) did not converge on R9's user-confirmed title=/subtitle= consistency assumption; one Round-3 sharpen (surgical-r3-f1) is a framing-level non-convergence that does not block ship.

PRD is fit to proceed to `specflow:task` after the escalation is resolved by the human. Revisions applied are listed under "Accepted findings" above; escalations are listed under "Escalated to human."

— Orchestrator, 2026-05-07
