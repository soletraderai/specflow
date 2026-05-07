# Adversarial PRD Review — 016 Brief Enhancements

## Summary
The PRD is directionally aligned with the interview but is not gate-ready: it treats the existing brief skill as the implementation target even though the current skill is already 524 lines and the stated skill-size ceiling says new behaviour should become a new skill or agent, which directly undermines R8 and the implementation strategy.

## Findings

### [BLOCK] R8 contradicts the hard skill-size ceiling rule and the current baseline
**Axis:** Specific Push
**Evidence:** PRD R8 requires "`brief/SKILL.md` total line count after additions remains ≤500 lines" and says "Existing prose tightening ... is part of the change set" (PRD lines 74-76). The current skill is 524 lines (Brief skill lines 1-524). The interview also says adding four CSS rule-sets "must keep the file under 500 lines after additions" and "Existing prose tightening is required" (Interview lines 18-19).
**Impact:** The PRD asks implementers to add four grammars, examples, parser guidance, HTML templates, and inline CSS while deleting at least 25 existing lines before adding anything. That is not a requirement; it is an unproven compression budget. It also conflicts with the provided ceiling rule: new behaviour becomes a new skill or agent, not bolted onto an over-ceiling skill.
**Recommendation:** Either revise the scope to satisfy the ceiling rule by moving the new behaviour into a new skill/agent within the four-block scope, or add a concrete line-budget plan showing exactly which existing sections will be condensed and how many net lines each new block may consume. AC-3 should require a documented before/after line-count budget, not only `wc -l`.

### [BLOCK] Goal-level out-of-scope conflicts with the skill-size ceiling rule
**Axis:** Out-of-scope coherence
**Evidence:** PRD Non-goals say "Building a separate brief renderer. The four blocks ship inside the existing `brief/SKILL.md` template" (PRD line 28). The interview goal confirmation says "Out-of-scope-at-goal-level: building a separate brief renderer" (Interview line 7). The user-provided project rule says skills must stay ≤~500 lines and "new behaviour becomes a new skill or agent, not bolted onto existing skill."
**Impact:** The PRD traps implementation between two hard constraints: do not create a separate renderer, but also do not bolt new behaviour onto an over-size skill. A gate should not pass a plan whose permitted implementation path is internally invalid.
**Recommendation:** Reconcile the scope explicitly. If "separate brief renderer" is forbidden, define an allowed decomposition that is not a separate renderer but satisfies the skill-size ceiling, such as delegating only four-block compilation to a narrowly scoped companion skill/agent while preserving `specflow:brief` as the entrypoint.

### [CONCERN] Vision is aligned but not traced verbatim to the goal confirmation
**Axis:** Vision-to-Goal trace
**Evidence:** The interview goal says: "Extend the brief skill so the rendered HTML carries four new structured blocks — Key Features ... Resources ... Key Decisions ... This-phase / Next-phase..." and "The brief is a visual artefact, not a text dump..." (Interview line 5). The PRD Vision says: "The brief becomes the layered, interface-first feature artefact..." and adds "Pocock-shape" framing in the Problem (PRD lines 12 and 16).
**Impact:** The PRD preserves the broad intent, but it does not trace the Vision to the interview Goal verbatim where possible. The added "layered, interface-first" and "Pocock-shape" language may be true context, but it dilutes the original goal's sharper contract: minor-technical/non-technical scanning, deterministic HTML, inline CSS, and PRD stays technical.
**Recommendation:** Rewrite the Vision first sentence to reuse the goal confirmation language from Interview lines 5 and 7, then add the "layered/interface-first" framing as a secondary sentence if still needed.

### [BLOCK] Requirement traces do not meet the requested traceability contract
**Axis:** Requirement traceability
**Evidence:** Requirements R1-R10 include Trace and Serves goal lines (PRD lines 47-84), but the Trace lines cite broad sections such as "interview Round 1" and "codebase context bullet 2" rather than a "Resolved interview line." The interview has line-addressable resolved answers, for example Round 1 Answer at Interview line 25, Round 2 Answer at line 31, Round 3 Answer at line 37, and Round 4 Answer at line 43.
**Impact:** Reviewers cannot mechanically verify that each requirement maps to a resolved interview line. "Serves goal: Outcome + Audience" also cites fields that are not explicit fields in the interview Goal confirmation section (Interview lines 3-7).
**Recommendation:** For each R1-R10, replace broad Trace prose with exact source references such as `Trace: Interview line 25`. Replace "Serves goal: Outcome/Audience/Driving value" with explicit citations to goal fields or add those named fields to the interview/PRD and cite them consistently.

### [CONCERN] R7 only partially preserves the existing brief contract
**Axis:** Specific Push
**Evidence:** R7 says the new blocks are stripped from rendered PRD body prose "exactly as the four existing blocks are stripped at `brief/SKILL.md:71`" (PRD lines 70-72). The existing Step 3 scans only `{kind}` in `flow | comparison | scope | tree` (Brief skill lines 67-71), while unsupported block kinds render as `<pre>` with a warning (Brief skill lines 212-215).
**Impact:** Expanding the strip list is necessary for supported new blocks, but it changes the boundary between "unsupported block shown as warning" and "supported block stripped from PRD prose." If parsing accepts malformed new block openers or partially supported forms, content could disappear from the PRD body instead of being preserved as warning text, which forks the existing safety contract.
**Recommendation:** R7 should state that only syntactically valid supported new blocks are stripped after successful compilation. Malformed or unsupported variants must follow the existing unsupported/unclosed-block behaviour from Brief skill lines 214-215.

### [CONCERN] AC-7 under-tests R7 because it exercises only one new block
**Axis:** AC binary-observability
**Evidence:** R7 applies to each of `:::key-features | :::resources | :::key-decisions | :::phase-split` (PRD line 70). AC-7 tests only "a `:::key-features` block followed by `## Vision`" (PRD lines 106-107).
**Impact:** The acceptance suite could pass while `resources`, `key-decisions`, or `phase-split` still duplicate into the PRD body or get stripped incorrectly.
**Recommendation:** Expand AC-7 to require all four new block kinds to appear exactly once under `<section id="visual-abstract">` and zero times under `<section id="prd">`.

### [CONCERN] R10 fallback lacks malformed and nested marker coverage
**Axis:** Specific Push
**Evidence:** R10 says unsupported block kinds continue to render as `<pre>` with "Unsupported visual block" (PRD lines 82-84). The existing strict rules separately say unsupported kinds render as `<pre>` and an unclosed block is fatal (Brief skill lines 214-215). AC-8 tests only a simple `:::xxx` block (PRD lines 109-110).
**Impact:** The PRD does not force implementation to preserve behaviour for edge cases that are easy to break while extending the parser: nested `:::` markers, a malformed opener, an unsupported block inside a supported-looking body, or an unclosed new block.
**Recommendation:** Add an AC for fallback/parser edge cases: unsupported closed block renders as warning `<pre>`, unclosed supported or unsupported block aborts with file:line, and nested/malformed `:::` markers are either escaped inside the preserved body or rejected deterministically.

### [CONCERN] AC-10 contains a visual judgment call
**Axis:** AC binary-observability
**Evidence:** AC-10 requires `:::phase-split` and `:::scope` to "compile to visually distinct shapes" (PRD lines 115-116). R4 similarly says the block is "Visually distinct from `:::scope`'s three-column shape" (PRD lines 58-60).
**Impact:** "Visually distinct" is reviewer judgment unless backed by exact DOM/CSS assertions. A two-column versus three-column DOM assertion is binary; visual distinctness is not.
**Recommendation:** Reword AC-10 as binary DOM checks: `phase-split` emits exactly two column containers headed `this-phase` and `next-phase`; `scope` emits exactly three headed `v1`, `v2`, and `out`; their root CSS classes differ.

### [CONCERN] AC-4 relies on "documented HTML shape" without precise selectors
**Axis:** AC binary-observability
**Evidence:** AC-4 says each block produces "the documented HTML shape" and then describes a "CSS-grid row of feature cards," "row of link cards," a table, and a "two-column panel" (PRD lines 97-98).
**Impact:** "Documented HTML shape" and "card" are not strict enough for a binary review. Two implementations could both claim cards while producing incompatible DOM for CSS, tests, or future maintenance.
**Recommendation:** Define required root classes/selectors and minimum child structure for each block in R1-R4 or AC-4, for example `.key-features > .key-feature`, `.resources > .resource-card`, `.key-decisions table`, and `.phase-split > .phase-col`.

### [CONCERN] Resource icon set drops Slack from the interview without explanation
**Axis:** Requirement traceability
**Evidence:** Interview Round 3 says known source-type icons include "Linear, Doc, Design, GitHub, Slack" (Interview line 37). PRD Goal says source-type pills are "Linear / Doc / Design / GitHub" (PRD line 24), and R2 ships inline base64 icons for `linear | doc | design | github` only (PRD line 50).
**Impact:** The PRD silently narrows a resolved interview answer. If Slack resources appear, implementation may treat them as unsupported even though the interview explicitly named Slack.
**Recommendation:** Either add `slack` to the supported source-type icon set in R2/AC-6, or explicitly state that Slack was intentionally deferred and define its fallback rendering.

### [NOTE] R1 mentions optional thumbnails/icons but no acceptance criterion exercises them
**Axis:** Requirement traceability
**Evidence:** R1 says `:::key-features` supports "optional thumbnail or icon" (PRD line 46). The interview codebase context similarly says "optional thumbnail/icon" (Interview line 19). No AC mentions thumbnail or icon handling for `:::key-features` (PRD lines 88-116).
**Impact:** The optional media surface could be omitted, implemented with external assets, or handled inconsistently without failing acceptance.
**Recommendation:** Add a focused AC that verifies a `key-features` item with an inline icon or local thumbnail compiles deterministically and does not introduce external assets.

### [NOTE] Open questions overstate closure of "Topics not discussed"
**Axis:** Unstated Assumption
**Evidence:** The PRD says "None — all questions resolved during grilling" and claims the four Topics not discussed were "either resolved by assumption-with-callout ... or flagged out-of-scope" (PRD line 120). The interview lists title/subtitle, key-decisions auto-linking, phase-split versus scope attribute, and resource tags (Interview lines 51-54).
**Impact:** The PRD converts assumptions and deferrals into "None," which hides follow-up risk. R9 covers title/subtitle, and non-goals cover auto-linking and tags (PRD lines 33-34), but the phase attribute rejection is only indirectly covered by R4/AC-10 rather than listed as an explicit non-goal.
**Recommendation:** Keep "Open questions: None" only if a short "Resolved assumptions" subsection records each Topics-not-discussed item and its exact PRD disposition, including the phase attribute rejection.

### [NOTE] Unstated parser and escaping assumptions are embedded in the block additions
**Axis:** Unstated Assumption
**Evidence:** R5 says new blocks use `:::{kind} key="value"` with "optional space-separated attributes" and "Whitespace inside the body is preserved as authored" (PRD lines 62-64). The existing markdown conversion rules require escaped interpolated text and raw HTML escaping (Brief skill lines 90-92 and 227-230).
**Impact:** The PRD assumes an attribute parser that can safely handle quoted values, links, data URIs, pipes, and malformed input, but it does not state escaping or validation rules for the new block body fields.
**Recommendation:** Add a requirement that all interpolated block text and attributes are HTML-escaped according to the existing markdown conversion contract, and define failure behaviour for malformed attributes.

## Score Card
| Axis | Status | Notes |
|------|--------|-------|
| Vision-to-Goal trace | PARTIAL | The Vision aligns with the goal but does not preserve the interview goal wording verbatim where possible and adds extra Pocock/interface framing. |
| Requirement traceability (R1-R10) | PARTIAL | Every requirement has Trace and Serves goal lines, but they cite broad sections rather than resolved interview lines, and "Outcome/Audience/Driving value" are not explicit goal fields. |
| AC binary-observability (AC-1-10) | PARTIAL | Several ACs are binary, but AC-4 and AC-10 rely on "documented shape" or "visually distinct"; AC-7 and AC-8 under-cover the stated requirements. |
| Out-of-scope coherence | FAIL | The PRD forbids a separate renderer while R8 requires shrinking an already over-cap skill; this conflicts with the skill-size ceiling rule that new behaviour should not be bolted onto the existing skill. |


— User reviewed; revisions applied autonomously per session-start authorisation, 2026-05-07.
