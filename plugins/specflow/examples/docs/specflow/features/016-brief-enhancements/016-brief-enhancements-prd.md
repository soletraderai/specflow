---
feature: 016-brief-enhancements
status: draft
created: 2026-05-07
interview: ./016-brief-enhancements-interview.md
---

# Brief Enhancements

## Vision

The brief becomes the layered, interface-first feature artefact: a non-technical Key Features overview at the top, a Resources card row mid-brief, a Key Decisions table that exposes the synthesis trace, and a This-phase / Next-phase split that surfaces iteration boundaries inside a single version. Four new structured blocks extend `plugins/specflow/skills/brief/SKILL.md` with deterministic HTML compilation and inline CSS — no JS, no network calls, no external assets. The PRD stays purely technical; the brief carries the user-facing scan surface so a feature owner can hand the rendered HTML to a non-technical reader and have it land.

## Problem

The brief today compiles four block kinds (`:::flow`, `:::comparison`, `:::scope`, `:::tree`) that all serve technical or scope-shaping audiences. There is no block whose semantics are *user-facing feature description* (what the feature does in plain language), no block for *external references the brief should expose without forcing the reader into the source tree*, no block that captures the *decisions made during synthesis* in a glanceable table, and no block that distinguishes *iteration boundaries inside a single version* from whole-feature scope. Feature owners patch around the gap in prose, which the brief is contractually forbidden from rewording (`brief/SKILL.md` § What you MUST NOT do). The result: briefs stop short of the layered, interface-first Pocock-shape the team is targeting (interview Round 1; cited from `knowledge/pocock-real-feature-build.md`).

## Goals

- Extend the brief block grammar with four new kinds — `:::key-features`, `:::resources`, `:::key-decisions`, `:::phase-split` — each with deterministic HTML compilation and inline CSS.
- Preserve the no-JS / no-external-assets / no-network-calls contract (`brief/SKILL.md:514`).
- Target ≤500 lines for `brief/SKILL.md` after additions, per the skill-size ceiling. The file is currently **524 lines** (verified at PRD time); the change set must net-reduce by at least 24 + new-block-cost lines. If the line arithmetic does not close after a measured prose-tightening pass, the change set blocks pre-merge and a chain-don't-absorb feature creates a sibling `brief-blocks` skill — see Open Questions.
- Place Key Features at the top of the Visual abstract by source-order convention; do not introduce a new top-level `<section>` wrapper.
- Drive `:::resources` icons by source-type pill (Linear / Doc / Design / GitHub — Slack dropped per decision-log 2026-03-22 *Slack removed from the stack*; CONTEXT.md confirms Slack-free tooling) shipped as inline SVG-base64; explicit `icon="data:..."` override on a resource line; no runtime favicon fetch.

## Non-goals

- Building a separate brief renderer. The four blocks ship inside the existing `brief/SKILL.md` template (interview goal-confirmation block — Out of scope at the goal level).
- Relaxing the no-JS / no-external-assets / no-network-calls contract. Filtering, interactive controls, and runtime favicon fetches are forbidden (interview Round 3; goal-level out-of-scope).
- Promoting Key Features into the PRD body as prose. The block lives in the PRD as source for the brief, and the brief skill strips it from rendered PRD prose (interview Round 2).
- Touching any other skill — `prd`, `task`, `develop`, `test`, `complete` are out of scope (goal-level out-of-scope).
- Folding `:::phase-split` into `:::scope` via an attribute. Different semantics, different visual weight, deliberately separate (interview Round 4).
- Auto-resolving `:::key-decisions` `Source` column to a `decision-log.md` file. Considered, deferred — would add a path-traversal step that breaks byte-determinism (interview "Topics not discussed").
- Per-resource `tags=[...]` filter on `:::resources`. Filtering implies JS / interactivity (interview "Topics not discussed").
- Reformatting the four existing blocks' grammar examples or stripping their CSS section comments. The existing four blocks (`:::flow`, `:::comparison`, `:::scope`, `:::tree`) are not touched in this change set; if the line-cap arithmetic forces a refactor, chain-don't-absorb applies (see Open Questions). Surgical-changes constraint per the skill-size-ceiling memory feedback.

## Users

- **Power user — operations coordinator.** Reads the brief to scan a feature's user-facing surface in seconds before diving into the PRD; values Key Features for the non-technical at-a-glance, Phase split for sprint-boundary visibility.
- **First-time visitor — new internal hire.** Lands on a brief without prior context; needs Key Features and Resources up front to build a mental model without reading the technical PRD body.
- **Admin — platform operator.** Audits feature decisions; values Key Decisions as a synthesis trace pointing back to the interview rounds.
- **Mobile warehouse operator.** Reads briefs on a mobile device with intermittent connectivity; values that the new blocks degrade gracefully at the existing 1100px / 860px / 360px viewport widths (mobile breakpoints already present in `brief/SKILL.md:391-403`).

(Profiles per `admin/profiles.json`. Each user reads the brief; none of them author the blocks — block authoring is a feature-owner concern handled by the existing PRD skill flow.)

## Requirements

- **R1.** Add a `:::key-features` block grammar to `brief/SKILL.md` § Visual Block Grammar. The block compiles to a CSS-grid row of cards under selector `.key-features` with one `<article class="feature-card">` per feature line: title + one-line description + optional thumbnail or icon — no meta tail, no accent variants. The grid scaffolding reuses the `:::comparison` CSS pattern at `brief/SKILL.md:346` (`.modes-row` grid scaffolding, simpler card payload).
  - Trace: interview Round 1 — *Key Features renders inside the Visual abstract; source order determines layout.* Codebase context bullet 5 — *`:::comparison` already does card-grid rendering; reuse beats invent.*
  - Serves goal: Outcome (four new blocks compile deterministically) + Audience (first-time visitor scans non-technical surface).

- **R2.** Add a `:::resources` block grammar to `brief/SKILL.md` § Visual Block Grammar. The block compiles to a row of `<a class="resource-card">` link cards (icon + title + source-type pill) under selector `.resources`. Icons:
  1. Source-type pill drives the icon: built-in inline SVG-base64 glyphs for `linear | doc | design | github` ship in the brief skill itself.
  2. Format pinned: SVG-base64 (not PNG) for crisp rendering at any pill size and minimal byte cost.
  3. Source: author-drawn monochrome glyphs in the brand's silhouette, not the official brand mark — avoids trademark/licensing entanglement.
  4. Each icon adds at most one line to `brief/SKILL.md` (counted against R8's ceiling).
  5. Slack dropped from the icon set per decision-log 2026-03-22 *Slack removed from the stack*; CONTEXT.md Stack section confirms Slack-free tooling.
  6. Explicit `icon="data:image/(svg+xml|png);base64,..."` attribute on a resource line overrides the source-type default (validation rules in R13).
  - Trace: interview Round 3 — *Declared inline. Favicons are inline data URIs declared in the block source, OR a small set of known source-type icons ... is shipped as inline base64 in the brief skill itself.*
  - Serves goal: Outcome + Driving value (the brief is byte-deterministic and self-contained — adding network-fetched favicons would break both contracts).

- **R3.** Add a `:::key-decisions` block grammar to `brief/SKILL.md` § Visual Block Grammar. The block compiles to `<table class="key-decisions">` with three `<th>` columns in fixed grammar order — `Decision | Why | Source`. The Source column accepts plain text or an explicit markdown link the author wrote; the brief skill does not auto-resolve to `decision-log.md`.
  - Trace: interview Goal confirmation — *Key Decisions (decision table)*. Topics not discussed — *Whether `:::key-decisions` should auto-link the `Source` column to a `decision-log.md` file when present. Considered, deferred — auto-resolution adds a path-traversal step and the existing brief contract is byte-deterministic.*
  - Serves goal: Outcome + Audience (admin auditing the synthesis trace).

- **R4.** Add a `:::phase-split` block grammar to `brief/SKILL.md` § Visual Block Grammar. The block compiles to a two-column panel under selector `.phase-split` with two `<div class="phase-col this-phase">` and `<div class="phase-col next-phase">` panels; each column is a flat list, one item per line. Visually distinct from `:::scope`'s three-column shape — `:::phase-split` shows the iteration boundary inside a single version, `:::scope` shows the whole-feature scope.
  - Trace: interview Round 4 — *Different semantics, different visual weight … `:::scope` is the whole-feature scope-at-a-glance; `:::phase-split` is iteration-boundary … Two columns (this-phase, next-phase), no third "out" column.*
  - Serves goal: Outcome + Driving value (the iteration boundary inside a version is currently invisible in the brief; `:::phase-split` exposes it).

- **R5.** Each new block honours the existing block contract at `brief/SKILL.md:212-216`: `:::{kind} key="value"` opener with optional space-separated attributes, structured body, `:::` closer. Whitespace inside the body is preserved as authored (verbatim, leading-space-included). Each block keeps its position in the source — the brief author places blocks in the order they should render in the Visual abstract; the compiler does not re-order.
  - Trace: interview Round 1 — *the block-compile order in the source is what determines layout … the existing "Each block keeps its position in the source" rule from § Step 3 carries this for free.* Codebase context bullet 2 — *The four existing blocks compile deterministically.*
  - Serves goal: Outcome (deterministic compilation).

- **R6.** Each new block compiles deterministically — same input bytes produce same HTML output bytes, per the existing § Determinism contract at `brief/SKILL.md:44-53`. No clock time, no random IDs, no machine-specific paths in the compiled output. Field emission order is fixed by the documented grammar — Decision/Why/Source columns; resource card sub-elements; emitted HTML attributes — never by object/map iteration order.
  - Trace: codebase context bullet 2 — *The four existing blocks compile deterministically — same input bytes produce same HTML output bytes. The new blocks must hold the same contract.*
  - Serves goal: Outcome + Driving value (Visual abstract idempotence is the brief's review-surface guarantee).

- **R7.** The brief skill extends the existing block-strip rule (`brief/SKILL.md:71`) so the four new block kinds — `:::key-features | :::resources | :::key-decisions | :::phase-split` — are stripped from the rendered PRD body prose before emitting `{prd_body_html}`, exactly as the four existing blocks (`flow | comparison | scope | tree`) are. Blocks appear only inside the Visual abstract section, never duplicated in the PRD body. This is an extension of the existing contract — the strip rule's surface widens from four kinds to eight; no consumer behaviour beyond "blocks render in the Visual abstract, not in PRD prose" is changed.
  - Trace: interview Round 2 — *the brief skill strips it from the rendered PRD prose (per the existing § Step 3 "Strip the same blocks from the PRD body content before rendering it as prose" rule) so it appears only inside the Visual abstract.*
  - Serves goal: Outcome (the PRD body stays purely technical; non-technical Key Features content lives only in the brief's Visual abstract).

- **R8.** `brief/SKILL.md` total line count after the change set targets ≤500 lines. The file is currently **524 lines** (pre-change baseline). The change set must net-reduce by at least 24 + new-block-cost (estimated 60-100 lines for four block grammars + four CSS rule-sets + R11 eval enumeration). Existing prose tightening (e.g. condensing redundant block-grammar examples, distilling § HTML Template comments) is part of the change set — but the four existing blocks' grammar examples and CSS comments MUST NOT be touched (Non-goals constraint). If the arithmetic does not close, the change set blocks pre-merge per Open Questions.
  - Trace: codebase context bullet 4 — *Adding four new block CSS rule-sets must keep the file under 500 lines after additions, per the project's skill-size ceiling. Existing prose tightening is required.*
  - Serves goal: Driving value (skills must remain operable inside their context budget).

- **R9.** The four new blocks support the same `title="..."` / `subtitle="..."` attribute pattern as `:::flow` (the most-complete existing pattern at `brief/SKILL.md:131`). Block titles render as `<div class="block-title">` styled to look like a heading — NOT `<h2>` or `<h3>` — so they do not enter the sidebar TOC. The existing four blocks are unchanged; "consistent" means the new four follow the most-complete pattern, not that all eight have identical attribute surfaces.
  - Trace: interview "Topics not discussed" — *Whether the four new blocks should support the same `title="..."` / `subtitle="..."` attribute pattern … Assumed yes — consistency with the existing four blocks.* DA-4 cross-artefact concern resolved by the non-heading rendering decision.
  - Serves goal: Outcome (block grammar consistency without TOC inflation).

- **R10.** Unsupported block kinds (anything outside the eight listed: `flow | comparison | scope | tree | key-features | resources | key-decisions | phase-split`) continue to render as `<pre>` with the existing "Unsupported visual block" warning, never silently dropped. The new blocks expand the supported list; the unknown-block fallback's surface widens to eight but its behaviour is unchanged.
  - Trace: interview goal-level out-of-scope — *building a separate brief renderer; relaxing the no-JS / no-external-assets contract*. Codebase context bullet 1 — the unknown-block warning lives in `brief/SKILL.md:214`.
  - Serves goal: Outcome (extending the supported set without breaking the existing contract).

- **R11.** Update the `brief/SKILL.md` frontmatter `eval:` field to enumerate all eight block kinds. The current eval field reads "every structured visual block in the PRD renders deterministically into the Visual abstract section" — too vague for a fresh agent to verify. Replace with: enumerate `{flow, comparison, scope, tree, key-features, resources, key-decisions, phase-split}` and assert per-kind selector presence under `<section id="visual-abstract">` AND per-kind absence under `<section id="prd">`.
  - Trace: codebase context bullet 1 — the brief skill `eval:` field at `brief/SKILL.md:12` is the binary contract reviewers and agents read. Goal-Driven Reviewer GDR-1.
  - Serves goal: Outcome (the eval is the binary contract; enumerating the eight kinds makes the contract checkable).

- **R12.** Update all hardcoded references to the four-kind supported list in `brief/SKILL.md`. The current file lists the supported kinds at four call-sites: line 28 (intro paragraph), line 68 (Step 3 grammar), line 237 (markdown rules visualisation note), and line 513 ("Only flow, comparison, scope, tree are supported" in § What you MUST NOT do). All four call-sites must update to the eight-kind list in lockstep; the post-change file has zero references to the four-kind-only set.
  - Trace: DA-2 architectural gotcha — duplicated list copies across the skill must remain consistent or the contract is internally contradictory.
  - Serves goal: Outcome (the documented supported set is a single source of truth, not four drifting copies).

- **R13.** `:::resources` `icon="..."` override accepts only `data:image/(svg+xml|png);base64,...` URIs. Malformed overrides (non-base64, http(s), other MIME types) fall back to the source-type pill's default icon — no compose abort, since interview Round 3's tone was forgiving on attribute malformation. The fallback is recorded in the compiled HTML's source comment so a reviewer can audit which icons used the override path.
  - Trace: codex-r1-f3 — override validation was unspecified; without a rule, implementations diverge.
  - Serves goal: Outcome (deterministic icon resolution; no silent divergence).

## Acceptance criteria

- **AC-1.** Each new block kind has a fenced example under `brief/SKILL.md` § Visual Block Grammar.
  ```sh
  for kind in key-features resources key-decisions phase-split; do
    grep -qE "^### \`:::${kind}\`" plugins/specflow/skills/brief/SKILL.md || exit 1
  done
  ```
  - Verifies: R1, R2, R3, R4.

- **AC-2.** Each new selector class is defined inside the inline `<style>` block of `brief/SKILL.md`. The rule-sets are inline; no external stylesheet reference is added.
  ```sh
  for sel in key-features resources key-decisions phase-split; do
    grep -qE "^[[:space:]]+\\.${sel}[[:space:]\\{,]" plugins/specflow/skills/brief/SKILL.md || exit 1
  done
  ! grep -qE '<link[^>]+rel="stylesheet"' plugins/specflow/skills/brief/SKILL.md
  ```
  - Verifies: R1, R2, R3, R4, R6.

- **AC-3.** `wc -l plugins/specflow/skills/brief/SKILL.md` returns ≤500 after the change set. Binary check; the prescriptive prose-tightening tactics that lived in the previous draft of this AC have been demoted to non-binding implementation notes (see R8). If the arithmetic does not close, see Open Questions.
  - Verifies: R8.

- **AC-4.** Given a fixture PRD body containing one of each new block, the compiled brief HTML satisfies all four selector-grep assertions:
  ```sh
  grep -qE '<section id="visual-abstract">[\s\S]*<div class="key-features">[\s\S]*<article class="feature-card">' brief.html
  grep -qE '<section id="visual-abstract">[\s\S]*<div class="resources">[\s\S]*<a class="resource-card"[^>]*>[\s\S]*<img[^>]+src="data:' brief.html
  grep -qE '<section id="visual-abstract">[\s\S]*<table class="key-decisions">[\s\S]*<th[^>]*>Decision</th>[\s\S]*<th[^>]*>Why</th>[\s\S]*<th[^>]*>Source</th>' brief.html
  grep -qE '<section id="visual-abstract">[\s\S]*<div class="phase-split">[\s\S]*<div class="phase-col this-phase">[\s\S]*<div class="phase-col next-phase">' brief.html
  ```
  Selector names are frozen by R1-R4.
  - Verifies: R1, R2, R3, R4, R7.

- **AC-5.** The same fixture PRD body, run through `specflow:brief` twice with no source change, produces byte-identical HTML on both runs.
  ```sh
  cmp -s brief-run-1.html brief-run-2.html
  ```
  Matches the existing § Binary Verify Steps idempotence check at `brief/SKILL.md:489`.
  - Verifies: R6.

- **AC-6.** Compiling a `:::resources` block whose entries declare only a source-type pill (no `icon=` attribute) produces link cards whose `<img>` `src` matches a pinned format regex AND whose payload base64-decodes to a valid SVG or PNG:
  ```sh
  grep -qE '<img[^>]+src="data:image/(svg\+xml|png);base64,' brief.html
  ! grep -qE '<img[^>]+src="https?://' brief.html
  # base64 decode + format validation per built-in icon
  python3 -c "
  import re, base64, sys
  h = open('brief.html').read()
  for m in re.finditer(r'<img[^>]+src=\"data:image/(svg\+xml|png);base64,([A-Za-z0-9+/=]+)\"', h):
      mime, payload = m.group(1), m.group(2)
      decoded = base64.b64decode(payload, validate=True)
      if mime == 'svg+xml':
          assert decoded.lstrip().startswith(b'<svg'), 'svg payload does not start with <svg'
      elif mime == 'png':
          assert decoded.startswith(b'\\x89PNG\\r\\n\\x1a\\n'), 'png magic bytes missing'
  "
  ```
  No HTTP(S) URLs; no other image MIME types; payloads decode to valid image bytes (per codex-r3 sharpen).
  - Verifies: R2.

- **AC-7.** Given a fixture PRD body containing a `:::key-features` block followed by `## Vision`, the compiled brief HTML satisfies a DOM-scoped assertion:
  ```sh
  python3 -c "import sys; h=open('brief.html').read(); s=h.find('<section id=\"visual-abstract\">'); e=h.find('</section>',s); va=h[s:e]; assert va.count('<div class=\"key-features\">')==1, 'expected exactly one .key-features in visual-abstract'; ps=h.find('<section id=\"prd\">'); pe=h.find('</section>',ps); prd=h[ps:pe]; assert ':::key-features' not in prd, ':::key-features marker leaked into prd section'"
  ```
  Matches inside `<style>` blocks are ignored (the regex is scoped to section ranges, not the whole document).
  - Verifies: R7.

- **AC-8.** A PRD body containing a `:::xyzzy` block (where `xyzzy` is none of the eight supported kinds) compiles to a `<pre>` element with the literal text "Unsupported visual block" preceding it.
  ```sh
  grep -qE 'Unsupported visual block[\s\S]{0,200}<pre' brief.html
  ```
  - Verifies: R10.

- **AC-9.** A `:::key-features` block declared with `title="Key features"` produces compiled HTML in which the title text appears verbatim under a non-heading element, NOT under `<h2>` or `<h3>`.
  ```sh
  grep -qE '<div class="block-title"[^>]*>Key features</div>' brief.html
  ! grep -qE '<h[23][^>]*>Key features</h[23]>' brief.html
  ```
  - Verifies: R9.

- **AC-10.** `:::phase-split` and `:::scope` blocks in the same fixture PRD body compile to distinct shapes, verified by column counts and class-grep:
  ```sh
  grep -qE '<div class="scope">[\s\S]*<div class="scope-col v1">[\s\S]*<div class="scope-col v2">[\s\S]*<div class="scope-col out">' brief.html
  grep -qE '<div class="phase-split">[\s\S]*<div class="phase-col this-phase">[\s\S]*<div class="phase-col next-phase">' brief.html
  ! grep -qE '<div class="phase-split">[\s\S]*<div class="phase-col out">' brief.html
  [ "$(grep -oE '<div class=\"scope-col [a-z0-9]+\">' brief.html | wc -l)" -eq 3 ]
  [ "$(grep -oE '<div class=\"phase-col [a-z0-9-]+\">' brief.html | wc -l)" -eq 2 ]
  ```
  - Verifies: R4 + interview Round 4 stance against folding the two.

- **AC-11.** Edge-case rendering for the new blocks is binary-defined via three executable sub-assertions:
  - **Unclosed block** — fixture PRD with `:::key-features` and no closer:
    ```sh
    specflow:brief NNN-slug; rc=$?; [ $rc -ne 0 ] && grep -qE 'unclosed block.*key-features.*:[0-9]+' stderr.log
    ```
  - **Nested `:::`** — fixture PRD with `:::key-features` body containing a literal indented `:::` line as content:
    ```sh
    specflow:brief NNN-slug && [ $(grep -c '^:::$' brief.html) -eq 0 ] && [ $(grep -c '<div class="key-features">' brief.html) -eq 1 ]
    ```
  - **Malformed attribute** — fixture PRD with `:::resources title=` (no value):
    ```sh
    specflow:brief NNN-slug && grep -qE '<div class="resources">' brief.html && ! grep -qE '<div class="block-title"[^>]*></div>' brief.html
    ```
  - Verifies: R5, R6, R10.

- **AC-12.** Source-order preservation and whitespace verbatim render: given a fixture PRD with `:::key-features` followed by `:::resources` followed by `:::phase-split`, in that source order, with one feature line containing `  two-leading-spaces`:
  ```sh
  python3 -c "import sys; h=open('brief.html').read(); s=h.find('<section id=\"visual-abstract\">'); e=h.find('</section>',s); body=h[s:e]; idx=[body.find(c) for c in ['key-features','resources','phase-split']]; sys.exit(0 if idx==sorted(idx) and -1 not in idx else 1)"
  grep -qE '  two-leading-spaces' brief.html
  ```
  - Verifies: R5.

- **AC-13.** Strip rule extended to all four new kinds: given a fixture PRD body containing `:::key-features`, `:::resources`, `:::key-decisions`, `:::phase-split`:
  ```sh
  for kind in key-features resources key-decisions phase-split; do
    python3 -c "import sys; h=open('brief.html').read(); s=h.find('<section id=\"prd\">'); e=h.find('</section>',s); sys.exit(0 if ':::${kind}' not in h[s:e] else 1)" || exit 1
  done
  ```
  - Verifies: R7.

- **AC-14.** A `:::key-features title="Key features" subtitle="What the feature does"` block produces compiled HTML in which the subtitle renders inside the block's container:
  ```sh
  grep -qE '<div class="key-features"[\s\S]*<div class="block-subtitle"[^>]*>What the feature does</div>' brief.html
  ```
  - Verifies: R9 (subtitle= path, complementing AC-9's title= path).

- **AC-15.** `brief/SKILL.md` frontmatter `eval:` field enumerates all eight block kinds verbatim, asserting both per-kind visual-abstract presence AND per-kind PRD-section absence:
  ```sh
  grep -qE 'eval:.*\{flow.*comparison.*scope.*tree.*key-features.*resources.*key-decisions.*phase-split\}' plugins/specflow/skills/brief/SKILL.md
  ```
  - Verifies: R11.

- **AC-16.** All four hardcoded supported-kind references in `brief/SKILL.md` updated to the eight-kind list. After the change set, no line in the file lists exactly the four-kind-only set:
  ```sh
  ! grep -nE '^[^#]*:::flow.*:::comparison.*:::scope.*:::tree[^|]' plugins/specflow/skills/brief/SKILL.md
  ! grep -nE 'Only `flow`, `comparison`, `scope`, `tree`' plugins/specflow/skills/brief/SKILL.md
  ```
  - Verifies: R12.

- **AC-17.** TOC inflation guard: a brief whose PRD body contains all four new blocks (each with a `title="..."` attribute) renders a sidebar TOC that does not contain those titles:
  ```sh
  python3 -c "import sys; h=open('brief.html').read(); s=h.find('<nav class=\"toc\">'); e=h.find('</nav>',s); toc=h[s:e]; titles=['Key features','Resources','Key decisions','This phase']; assert not any(t in toc for t in titles), 'block titles leaked into TOC'"
  ```
  - Verifies: R9 (non-heading rendering decision; resolves DA-4 cross-artefact concern).

- **AC-18.** Mobile breakpoint behaviour: the four new selectors collapse to single-column at the existing `@media (max-width: 1100px)` AND `@media (max-width: 860px)` breakpoints documented at `brief/SKILL.md:391-403`, with no horizontal overflow at 360px viewport width:
  ```sh
  # Both breakpoints carry the new selectors
  for bp in '1100px' '860px'; do
    for sel in key-features resources phase-split; do
      grep -qE "@media \\(max-width: ${bp}\\)[\\s\\S]*\\.${sel}[\\s\\S]*grid-template-columns: 1fr" plugins/specflow/skills/brief/SKILL.md || exit 1
    done
  done
  # 360px headless render — no horizontal overflow
  npx playwright open --viewport-size=360,640 brief.html --eval "document.documentElement.scrollWidth <= 360 || (() => { throw new Error('horizontal overflow at 360px') })()"
  ```
  (The `:::key-decisions` `<table>` already has `width: 100%` per existing rule at `brief/SKILL.md:325`, so no per-block CSS override is needed; the 360px render check still applies to verify no overflow.)
  - Verifies: R1, R2, R4 (mobile-warehouse-operator profile audience). Per codex-r3 sharpen: 860px breakpoint AND 360px render now both verified, not just rule-presence at 1100px.

- **AC-19.** `:::resources` `icon=` override validation:
  - **Valid override** — `icon="data:image/svg+xml;base64,..."` produces an `<img src="..."/>` carrying the override:
    ```sh
    grep -qE '<a class="resource-card"[\s\S]*<img[^>]+src="data:image/svg\+xml;base64,' brief.html
    ```
  - **Malformed override** — `icon="https://example.com/icon.png"` falls back to the source-type pill's default; the compiled HTML carries a source comment recording the fallback:
    ```sh
    grep -qE '<!-- icon override fallback: .* -->' brief.html
    grep -qE '<a class="resource-card"[\s\S]*<img[^>]+src="data:image/(svg\+xml|png);base64,' brief.html
    ! grep -qE '<img[^>]+src="https?://' brief.html
    ```
  - Verifies: R13.

## Open questions

- **OQ-1: Chain-don't-absorb risk on the line-cap.** `brief/SKILL.md` is currently 524 lines (24 over the 500-line target). The change set's measured prose-tightening pass (R8) must absorb the 24-line debt PLUS the four-block additions (estimated 60-100 lines for grammar examples + CSS rule-sets + R11 eval enumeration). If the arithmetic does not close after the prose-tightening pass, the change set blocks pre-merge and creates a follow-up feature `017a-brief-blocks-extraction` (or successor number) that extracts § Visual Block Grammar into a sibling skill `brief-blocks/SKILL.md`. The decision-point is whether to do the extraction *prerequisite* (split into two PRs: extraction first, then 016 lands in the smaller `brief-blocks` skill) or *contingent* (016 lands monolithic-but-over-cap and 017a follows). Resolution needed before `specflow:task`.
  - Resolution path: at task-synthesis time, measure the actual prose-tightening savings against four benchmark fixtures; if net is < 0, route to prerequisite extraction.
  - **In-scope tightening targets** (non-existing-block; per Round 3 DA + TBC sharpen): § HTML Template comment density (lines 240-280, ~40 lines candidate); § Markdown Conversion rule-list redundancy (lines 220-238, ~10 lines candidate); § Reference / § What you MUST NOT do prose distillation (lines 510-524, ~10 lines candidate). Estimated ceiling savings ≈ 60 lines, which against +60-100 new-block cost suggests **the chain-don't-absorb path is the likely outcome**, not the contingency. Naming the candidate ranges keeps OQ-1 honest and lets the task skill decide on data, not vibes.

## See also

- Interview: [`./016-brief-enhancements-interview.md`](./016-brief-enhancements-interview.md)
- Tasks: [`./016-brief-enhancements-tasks.md`](./016-brief-enhancements-tasks.md) (generated by `specflow:task`)
- Tests: [`./016-brief-enhancements-test.md`](./016-brief-enhancements-test.md) (generated by `specflow:test`)
