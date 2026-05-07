# PRD interview — features/016-brief-enhancements

## Goal confirmation

The user invoked `specflow:prd 016-brief-enhancements` with the brief: "Extend the brief skill so the rendered HTML carries four new structured blocks — Key Features (non-technical card grid at the top), Resources (link cards), Key Decisions (decision table), and This-phase / Next-phase (two-column split). The brief is a visual artefact, not a text dump; every new block compiles to deterministic HTML with inline CSS, no JS. The Key Features section absorbs the original 015 scope and lives only in the brief — the PRD stays purely technical."

Confirmed: the goal is to add four new `:::key-features | :::resources | :::key-decisions | :::phase-split` block grammars to `plugins/specflow/skills/brief/SKILL.md`, with HTML templates and inline CSS for each, plus a placement note that Key Features renders at the top of the brief (after Vision / scope-at-a-glance, before the PRD body). Out-of-scope-at-goal-level: building a separate brief renderer; relaxing the "no JS, no external assets" contract; promoting Key Features into the PRD body; touching any other skill.

## Original request

> "The brief should be presented to a minor technical level… we need to list key features within a feature in non-technical format" — chat feedback (the originating drive for both 015 and 016). 015 was folded into 016 as a fourth block, per FEATURES.md § 016: "PRD stays purely technical; the brief carries the non-technical Key Features overview for the user's own scanning."

## Codebase context (pre-grilling)

- `plugins/specflow/skills/brief/SKILL.md` carries the brief contract — frontmatter at lines 1-13, the four existing block kinds (`:::flow`, `:::comparison`, `:::scope`, `:::tree`) under § Visual Block Grammar (lines 122-216), the inline-CSS HTML template (lines 240-444), and the no-JS / no-external-assets guarantee. The 3-column scope-at-a-glance pattern lives at `brief/SKILL.md:166`; that visual shape is the precedent for the new `:::phase-split` block.
- The four existing blocks compile deterministically — same input bytes produce same HTML output bytes. The new blocks must hold the same contract (per the file's § Determinism, lines 44-53).
- `knowledge/pocock-real-feature-build.md` is the inspiration. Pocock sketches modules and interfaces *first* and updates ubiquitous-language inline as new terms emerge ("materialize", "materialization cascade") — the brief is the layered, interface-first artefact analogue at the feature level. The four new blocks formalise the layering: Key Features at the top (interface), Resources mid-brief (referenced docs), Key Decisions (the synthesis trace), Phase split (the iteration boundary).
- The existing brief template loads ~200 lines of inline CSS (lines 253-403). Adding four new block CSS rule-sets must keep the file under 500 lines after additions, per the project's skill-size ceiling (`feedback_skill_size_ceiling`). Existing prose tightening is required.
- The `:::comparison` block already does card-grid rendering; `:::key-features` reuses the same CSS-grid scaffolding but with simpler card content (title + one-line description + optional thumbnail/icon — no meta tail, no accent variants). Reuse beats invent.

## Round 1 — where does Key Features sit in the brief layout

**Question.** Is Key Features rendered inside the Visual abstract section (alongside compiled `:::flow | :::comparison | :::scope | :::tree` blocks) or as its own top-level section above Visual abstract?

**Answer.** Inside the Visual abstract section, but the block-compile order in the source is what determines layout — the brief author places the `:::key-features` block first in the PRD body so it renders first in the Visual abstract (the existing "Each block keeps its position in the source" rule from § Step 3 carries this for free). No new top-level `<section>` wrapper. This keeps the brief's structural shape unchanged and means the new block joins the existing four under the same composition rules.

## Round 2 — does the PRD carry Key Features content too

**Question.** The Key Features content is non-technical. Where does it physically live — only in the PRD as a `:::key-features` block (which the brief skill compiles), or split across PRD-as-source vs brief-as-rendering?

**Answer.** Only in the PRD as a `:::key-features` block. The PRD body is the source of truth for the brief; placing the block in the PRD means the brief renders it without inventing content (the existing brief contract: "Markdown is the source of truth. The brief is a derived artefact… it never rewords or paraphrases the source content"). The block stays in the PRD's body, but the brief skill strips it from the rendered PRD prose (per the existing § Step 3 "Strip the same blocks from the PRD body content before rendering it as prose" rule) so it appears only inside the Visual abstract. **Important nuance:** the *content* of `:::key-features` is non-technical (one-line user-facing descriptions, optional thumbnails). The PRD's R/AC list around it remains technical. The non-technical / technical separation is by *block* in the PRD, not by *file*.

## Round 3 — should resources auto-fetch favicons

**Question.** The `:::resources` block renders link cards with favicon + title + source-type pill. Should the brief skill fetch favicons at compose time (network call) or are they declared inline in the block source?

**Answer.** Declared inline. The brief contract is no-network-calls / no-build-step (§ "What you MUST NOT do" line 514). Favicons are inline data URIs declared in the block source, OR a small set of known source-type icons (Linear, Doc, Design, GitHub, Slack) is shipped as inline base64 in the brief skill itself and selected by the source-type pill value. Default route: the source-type pill drives the icon; explicit `icon=` attribute on a resource line overrides. This preserves byte-identical determinism (same source → same output) and the self-contained-file guarantee.

## Round 4 — phase-split layout vs scope-at-a-glance

**Question.** The `:::phase-split` block is "this phase / next phase". The existing `:::scope` block is "v1 / v2 / out of scope" (three columns). Why a new block — couldn't `:::scope` be extended?

**Answer.** Different semantics, different visual weight. `:::scope` is the *whole-feature* scope-at-a-glance — what the feature ships now, what's deferred to a later version, what's permanently out. It is referenced once near the feature's top. `:::phase-split` is *iteration-boundary*: when a feature is being built across multiple phases inside the same version (e.g. "Sprint 2 ships R1-R5, Sprint 3 ships R6-R8"), this block makes the iteration boundary visible. Two columns (this-phase, next-phase), no third "out" column — out-of-scope already lives in `:::scope`. Layered on top of `:::scope`, not a replacement. A feature uses `:::scope` always; `:::phase-split` only when there is an iteration boundary inside the version. The user explicitly does not want to fold the two — different decisions, different audience, different visual weight.

## Sign-off

User confirmed: four new blocks (`:::key-features`, `:::resources`, `:::key-decisions`, `:::phase-split`), each with deterministic HTML compilation + inline CSS + no JS; Key Features lives in the PRD as a block but renders only in the brief's Visual abstract; favicons inline / source-type-pill-driven; phase-split distinct from scope. Proceed to PRD synthesis.

## Topics not discussed

- Whether the four new blocks should support the same `title="..."` / `subtitle="..."` attribute pattern as `:::flow` and `:::comparison`. Assumed yes — consistency with the existing four blocks; spelled out at PRD-synthesis time per block.
- Whether `:::key-decisions` should auto-link the `Source` column to a `decision-log.md` file when present. Considered, deferred — auto-resolution adds a path-traversal step and the existing brief contract is byte-deterministic; explicit links in the block source are sufficient. May surface as a follow-up if `decision-log.md` becomes a primary artefact in a later sprint.
- Whether `:::phase-split` should be replaceable by a `phase=` attribute on `:::scope` itself. Rejected per Round 4 — distinct semantics, distinct visual weight.
- Whether `:::resources` should support a per-resource `tags=[...]` filter for dimensioned scanning (e.g. "show only Linear resources"). Out of scope — the brief is a static visual artefact; filtering implies JS / interactivity, which the no-JS contract forbids.
