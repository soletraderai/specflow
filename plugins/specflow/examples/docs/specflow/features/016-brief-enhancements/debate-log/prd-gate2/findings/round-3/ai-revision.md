# Round 3 — AI revisions applied (016-brief-enhancements)

**Date:** 2026-05-07
**Orchestrator:** specflow:prd Phase D.6 (closing pass)

In Round 3, six reviewers re-evaluated their Round-1 findings against the revised PRD. Most accepted. The following sharpens were accepted and inline revisions applied to the PRD:

## Accepted sharpens (revisions applied to `016-brief-enhancements-prd.md`)

- **codex-r3-f1 (mobile breakpoints — sharpen).** AC-18 only verified the 1100px breakpoint via grep; codex-r3 required (a) verification at 860px AND (b) an actual 360px render check.
  - Revision: AC-18 rewritten as a `for bp in '1100px' '860px'` loop covering both breakpoints, plus a Playwright headless render at viewport 360x640 asserting `document.documentElement.scrollWidth <= 360`. The grep-only check is no longer sufficient.

- **codex-r3-f2 (icon validation depth — sharpen).** AC-6's regex constrained the URI prefix but did not validate that the base64 payload decoded to valid SVG/PNG bytes.
  - Revision: AC-6 extended with a `python3` block that base64-decodes each `<img src="data:image/...;base64,...">` payload and asserts SVG payloads start with `<svg` and PNG payloads carry the magic byte signature `\x89PNG\r\n\x1a\n`. Catches malformed-payload-but-valid-prefix failure mode.

- **DA-r3-f1 (chain-don't-absorb candidate ranges — accept-with-sharpen).** OQ-1 named the chain-don't-absorb path but did not name which non-existing-block prose ranges were in scope for the prose-tightening pass; reader could not independently judge whether the math closes.
  - Revision: OQ-1 extended with three named line ranges and ceiling-savings estimates (§ HTML Template comments lines 240-280, ~40 lines; § Markdown Conversion rule-list lines 220-238, ~10 lines; § Reference / § What you MUST NOT do prose distillation lines 510-524, ~10 lines). Estimated ceiling savings ≈ 60 lines, which against +60-100 new-block cost suggests **the chain-don't-absorb path is the likely outcome**. OQ-1 honest about that; reader does not need to do the audit themselves.

- **DA-r3-f4 (existing-blocks TOC scope clarification — optional polish).** R9's non-heading rendering decision applies only to the four new blocks. The existing four blocks' `title=` rendering is not audited; if existing blocks render `title=` as `<h2>`/`<h3>`, the TOC may already be inflating today.
  - Revision: R9 trace block extended with one line — "The existing four blocks' title rendering is not audited in this feature; AC-17's TOC-inflation guard applies only to the new four. Existing-blocks behaviour is pre-existing, not a 016 regression." Clarifies scope intent for downstream consumers without changing scope.

## Push-backs (sharpens not applied — disposition: escalate)

- **simplicity-r3-f1 (R9 title=/subtitle= consistency — third sharpen).** Simplicity Reviewer maintains R9 is speculative configurability. The interview's "Topics not discussed" entry was user-confirmed at sign-off ("Assumed yes — consistency with the existing four blocks"). Round 3 sharpened to "drop R9 / AC-9 / AC-14 entirely". Orchestrator declines: walking back a user-confirmed interview decision in Round 2/3 of Gate 2 inverts the interview's role as the load-bearing precedent. Escalating as a non-converged finding for the human's resolution.

- **surgical-r3-f1 (AC-3 prose-tightening over-prescription — sharpen).** Surgical Reviewer wants the chain-don't-absorb fallback documented as the EXPECTED outcome rather than the contingency. Orchestrator: OQ-1 already names this honestly per DA-r3 sharpen. Marginal disagreement on framing; not load-bearing. Accepting the spirit by tightening OQ-1 prose (above) without changing R8's framing.

- **TBC-r3-f1 (524-line baseline math — sharpen).** Same content as DA-r3-f1; addressed by OQ-1 candidate-ranges revision above. Marking as accepted-via-DA.

## Net status

PRD revisions applied land cleanly. Two Round-3 sharpens did not converge with the writer's defence (simplicity-r3-f1, surgical-r3-f1 framing). Both are surfaced in the manifest as escalations for human resolution before downstream skills proceed.
