---
name: specflow:brief
description: Compose a self-contained, browser-readable feature brief from features/NNN-{slug}/{NNN-slug}-prd.md, {NNN-slug}-interview.md, and (when present) the Gate 2 manifest. Produces a single NNN-{slug}-brief.html with a visual abstract, the PRD body, the interview transcript, and agent-review summary. Auto-fires after specflow:prd Phase E (Gate 2 closes); manual via /specflow:brief; bulk via --all.
status: v2-new
phase: 1
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-interview.md
  - docs/specflow/admin/config.json (optional; reads brief.commitPolicy)
produces:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-brief.html
eval: features/NNN-{slug}/{NNN-slug}-brief.html exists; a second compose from unchanged sources is byte-identical; sidebar TOC links resolve; every structured visual block in the PRD renders deterministically into the Visual abstract section; rendered HTML carries a one-line policy banner reflecting config.brief.commitPolicy (011-brief-commit-policy v2.4.0).
---

# specflow:brief

Compose one feature brief from the markdown sources in a feature folder:

```
features/NNN-{slug}/NNN-{slug}-prd.md          (required, source of prose + visual blocks)
features/NNN-{slug}/NNN-{slug}-interview.md    (required, rendered as Interview section)
features/NNN-{slug}/debate-log/prd-gate2/manifest.md   (optional, summarised in Agent reviews section)
features/NNN-{slug}/debate-log/tasks-gate3/manifest.md (optional, summarised in Agent reviews section)
        ↓
features/NNN-{slug}/NNN-{slug}-brief.html
```

Markdown is the source of truth. The brief is a derived artefact — it composes the existing files into one browser-readable view; it never rewords or paraphrases the source content. PRD prose appears verbatim. Interview Q&A appears verbatim. The only interpretive step is compiling structured visual blocks (`:::flow`, `:::comparison`, `:::scope`, `:::tree`) into HTML/CSS visuals — and that compilation is deterministic per the rules below.

No build step, package install, network call, script tag, or external asset is allowed. Compose directly in the current runtime and write the final HTML file to disk.

## Inputs

- `/specflow:brief NNN-{slug}` composes exactly the feature at `docs/specflow/features/NNN-{slug}/`.
- `/specflow:brief --all` finds every `docs/specflow/features/[0-9][0-9][0-9]-*-prd.md` and composes a brief for each one independently.
- If the current working directory is already `docs/specflow`, use paths relative to that folder. Otherwise use `docs/specflow/...` from the repo root.

Abort with a clear message if:
- The PRD markdown is missing.
- The interview markdown is missing.
- The feature folder name and PRD filename prefix differ.
- The target path would leave the feature folder.

## Determinism

The output must be byte-identical for unchanged inputs.

- Set `{generated_date}` to the latest source-file timestamp across PRD, interview, and (if present) the Gate 2 / Gate 3 manifests. Compute per file via `git log -1 --format=%cI -- <path>` when the file is tracked and has a commit; otherwise use filesystem mtime as ISO-8601 seconds with timezone. Take the latest of those values.
- Sort generated IDs, attributes, and bulk `--all` file order lexicographically.
- Do not include random IDs, current clock time, machine-specific absolute paths, or tool version strings.
- Preserve intentional markdown content; normalise generated HTML indentation exactly as in the template.
- Visual blocks compile deterministically — same block input produces same HTML output.

## Step-By-Step Execution

1. Resolve scope.
   - Single feature: validate `NNN-{slug}` matches `^[0-9]{3}-[a-z0-9]+(?:-[a-z0-9]+)*$`.
   - Bulk: list matching PRDs under `features/`, sorted lexicographically.
   - Verify per feature: `test -f "features/NNN-{slug}/NNN-{slug}-prd.md"` and `test -f "features/NNN-{slug}/NNN-{slug}-interview.md"`.

2. Read the PRD markdown.
   - Read full file as UTF-8.
   - Derive `{feature_id_slug}` from the parent folder name.
   - Derive `{title}` from the first H1; if absent, use `{feature_id_slug}`.
   - Derive `{status}` and `{type}` from PRD frontmatter if present (`status:`, `type:`); else omit those fields from the source strip.

3. Extract structured visual blocks.
   - Scan the PRD body for fenced blocks delimited by `:::{kind}` … `:::` where `{kind}` is one of `flow | comparison | scope | tree`.
   - Each block keeps its position in the source. Compile each block to its HTML form per "Visual block grammar" below.
   - Record the compiled visual blocks in source order. They populate the **Visual abstract** section at the top of the brief.
   - Strip the same blocks from the PRD body content before rendering it as prose. (They appear in the Visual abstract; not duplicated in the PRD body.)
   - If no structured blocks exist, omit the Visual abstract section entirely (no empty heading).

4. Read the interview markdown.
   - Read full file as UTF-8.
   - The interview content is rendered as a top-level `<section>` titled "Interview", with all of its body markdown converted to HTML.
   - Do not paraphrase or summarise the interview. Faithful render only.

5. Read agent-review sources (optional).
   - If `debate-log/prd-gate2/manifest.md` exists, read it.
   - If `debate-log/tasks-gate3/manifest.md` exists, read it.
   - Compose the **Agent reviews** section by faithfully rendering each manifest's body. Use H3 headings: "Gate 2 — PRD review" and "Gate 3 — Tasks review".
   - If neither manifest exists, omit the Agent reviews section entirely.

6. Parse PRD body structure.
   - Convert headings to stable slug IDs. Slug rule: lowercase, strip HTML, replace non-alphanumeric runs with `-`, trim `-`, append `-2`, `-3` for duplicates.
   - Build `{toc_html}` from H2 and H3 headings across the whole brief (Visual abstract, PRD body, Interview, Agent reviews). If there are no H2/H3 headings in any section, emit `<p class="toc-empty">No sections found.</p>`.
   - Verify: every TOC `href="#..."` has exactly one matching heading `id`.

7. Convert markdown bodies to HTML.
   - Use the conversion rules below for the PRD body, interview body, and each manifest body.
   - HTML-escape interpolated text.

8. Resolve assets.
   - Resolve markdown image destinations relative to the feature folder.
   - For local files under 50 KB, inline as `data:{mime};base64,{payload}`.
   - For local files 50 KB or larger, keep a relative URL from the HTML file to the asset.
   - Leave `http:`, `https:`, `mailto:`, and anchor links unchanged.
   - Abort if a local image reference resolves outside the feature folder.

9. Emit HTML.
   - Fill the template exactly once.
   - Show the drift banner only when an existing brief HTML file is present and its mtime is older than the latest source mtime at the start of the compose. A fresh successful compose should omit the banner on the next unchanged run.
   - Read `docs/specflow/admin/config.json` if present and resolve `brief.commitPolicy` (default `committed` when absent). Render a one-line policy banner at the top of the brief, just below the source strip:
     - `committed` → `<p class="commit-policy">This brief is committed to the repo as the diffable review surface (config.brief.commitPolicy = "committed").</p>`
     - `derived` → `<p class="commit-policy commit-policy-derived">This brief is gitignored as a derived artefact — regenerate via <code>specflow:brief NNN-{slug}</code> (config.brief.commitPolicy = "derived").</p>`
   - Write to `features/NNN-{slug}/NNN-{slug}-brief.html`.
   - Verify: `test -f "features/NNN-{slug}/NNN-{slug}-brief.html"`.

10. Open in the user's browser.
    - On macOS, run `open "features/NNN-{slug}/NNN-{slug}-brief.html"`.
    - On Linux, run `xdg-open` if available; otherwise print the file path and skip silently.
    - On Windows or unknown platforms, print the file path and skip silently.
    - Browser-open is best-effort and never blocks success. Failures are logged but the skill still returns success.
    - Skip browser-open in `--all` mode and when invoked as a sub-skill from `specflow:prd` (the parent skill decides whether to open).

11. Verify idempotence.
    - Compose the same input to a temporary file using the same algorithm.
    - Compare bytes with the just-written brief.
    - Verify: `cmp -s "features/NNN-{slug}/NNN-{slug}-brief.html" "$tmpfile"`.

## Visual Block Grammar

Four block kinds. Each opens with `:::{kind}` (optionally followed by space-separated `key="value"` attributes), holds a structured body, and closes with `:::`. Lines outside these blocks are normal markdown.

### `:::flow`

A numbered horizontal/grid flow diagram. One step per line in the block body. Each line: `verb | description` and optional trailing `| @duration` and `| !gate` markers.

```
:::flow title="Field tech happy path" subtitle="From project open to saved Structure"
Open project | Tech opens an active project on a LiDAR-capable iPhone.
Tap "Scan Structure" | One-tap entry into the new mode.
Instruction screen | Brief overview of how to walk; tap Start.
Live capture | Continuous walk-through with live 2D preview. | @~ 15–25 min
Building floor plan | StructureBuilder processes captured rooms. | !gate
Segmentation review | Top-down 2D editor with auto-segmented rooms.
Edit & label | Rename, set floors, split, merge, append.
Save | One transaction: 1 Structure + N Rooms + N+1 ScanVersions. | !gate
:::
```

Compiles to a horizontal sequence of numbered cards (CSS class `journey`). Steps marked `!gate` get a subtle background tint (CSS class `stage gate`). `@duration` becomes an annotation pill below the description.

### `:::comparison`

A row of comparison cards. One card per `- {name}` block separated by blank lines.

```
:::comparison title="Three scan modes coexist"
- Structure scan | tag="New · v1" | accent
  Continuous walk-through across the whole property. Auto-segmented into rooms.
  meta: SDK = Apple RoomPlan + StructureBuilder
  meta: Best for = Initial site visits, multi-room jobs
- Single room scan | tag="Existing"
  One room at a time. Unchanged. Fast path for one-off rooms.
  meta: SDK = RoomScanModule
  meta: Best for = Single rooms outside a structure
- Chamber scan | tag="Existing"
  Damage-zone grouping for drying equipment and environmental readings.
  meta: Concept = RoomChamber join
  meta: Best for = Drying allocations, readings
:::
```

Compiles to a CSS-grid row of `mode` cards (1–4 cards typical). The card with `accent` keyword on its first line gets the highlighted top-border (`mode.new`); others get the muted variant. `tag="..."` populates the small label pill. `meta:` lines become the small key/value tail.

### `:::scope`

Three-column scope-at-a-glance: shipping in v1, deferred to v2, out of scope. Each column is a flat list, one item per line.

```
:::scope
v1:
- Live 2D floor-plan preview
- Per-room USDZ download
- Split / merge in segmentation review
- Append-missed-room (re-walk)
v2:
- Mid-scan crash & resume
- Phone-call interruption recovery
- Thermal throttling on long scans
out:
- 3D editing of segmented rooms
- Outdoor space scanning
- Android
- ML beyond Apple's StructureBuilder
:::
```

Compiles to three side-by-side `scope-col` panels (green / amber / grey accent stripe).

### `:::tree`

Capability or decision tree across N stages. Each stage is one `## {question}` block; branches are `- {label} → {outcome}` lines under it. Use `→ next` to chain into the next stage.

```
:::tree title="Capability gate"
## Does the iPhone have LiDAR?
- No → No scan modes available
- Yes → next
## iOS 17+?
- No → Room + Chamber available; Structure disabled with tooltip
- Yes → next
## All checks pass
- Result → Structure + Room + Chamber unlocked
:::
```

Compiles to a horizontal three-column decision tree (`cap-tree` / `cap-q`). `→ next` advances to the next column; terminal branches show their outcome inline.

### Strict rules

- Block kinds outside the four above are rendered as plain `<pre>` with a `Unsupported visual block` warning above. They are never silently dropped.
- An unclosed block (missing trailing `:::`) is a fatal error — abort compose with a clear file:line message.
- Whitespace inside blocks is preserved as authored; the compiled HTML normalises only the surrounding indentation.

## Markdown Conversion Rules

Use CommonMark plus these GFM extensions:

- Tables.
- Task lists: `- [ ]` and `- [x]` become disabled checkboxes.
- Strikethrough: `~~text~~`.
- Autolinks.

Required custom handling:

- Escape raw HTML from markdown by default. Do not pass through `<script>`, inline event handlers, or external stylesheet links.
- Fenced code blocks become `<pre><code class="language-{lang}">...</code></pre>`. Escape code text.
- Inline code becomes `<code>...</code>`.
- Blockquotes beginning with `[!NOTE]`, `[!WARNING]`, or `[!TIP]` become `<div class="callout note|warning|tip">` with a `.callout-title`. Other blockquotes remain `<blockquote>`.
- Markdown tables become `<table>` with `<thead>` when a header row exists.
- Horizontal rules become `<hr>`.
- Links are escaped and normalised. Local markdown links remain relative to the feature folder.
- Images get `loading="lazy"` and escaped alt text.
- Mermaid fences are not rendered as diagrams. Render them as syntax-highlighted code blocks with `language-mermaid`. (Reason: Mermaid requires a runtime script and breaks the no-script rule. Visualisations belong in `:::flow|comparison|scope|tree` blocks.)
- Long appendices may use native `<details>` only when the markdown heading text starts with `Appendix `; keep the heading visible as the summary.

## HTML Template

Use this template verbatim. Replace exactly these placeholders: `{title}`, `{feature_id_slug}`, `{generated_date}`, `{status_pill}`, `{type_pill}`, `{toc_html}`, `{visual_abstract_html}`, `{prd_body_html}`, `{interview_body_html}`, `{reviews_html}`, `{interview_link}`. When a section is omitted (e.g. no visual blocks), insert an empty string for its placeholder; the surrounding `<section>` wrapper is part of the placeholder so removing the section also removes its container.

When a drift banner is required, insert the drift banner block (defined below) as the first element inside `<main>` before `<div class="source-strip">`; otherwise insert nothing.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{title}</title>
<style>
  *, *::before, *::after { box-sizing: border-box; }
  html { scroll-behavior: smooth; }
  body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    font-size: 16px;
    line-height: 1.65;
    color: #1a1a1a;
    background: #fafaf7;
  }
  .layout { display: grid; grid-template-columns: 280px 1fr; min-height: 100vh; }
  nav.toc {
    position: sticky; top: 0; align-self: start;
    height: 100vh; overflow-y: auto;
    padding: 32px 24px;
    background: #1a2632; color: #d8d8d4;
    border-right: 1px solid #283442;
  }
  nav.toc h2 { margin: 0 0 4px; font-size: 18px; color: #fff; letter-spacing: -0.01em; }
  nav.toc .subtitle { margin: 0 0 24px; font-size: 12px; color: #88a; text-transform: uppercase; letter-spacing: 0.08em; }
  nav.toc ul { list-style: none; padding: 0; margin: 0 0 24px; }
  nav.toc ul ul { margin: 4px 0 12px 12px; padding-left: 12px; border-left: 1px solid #2a3a4a; }
  nav.toc li { margin: 6px 0; }
  nav.toc a { color: #c8d0d8; text-decoration: none; font-size: 14px; line-height: 1.4; display: block; padding: 2px 0; transition: color 0.15s; }
  nav.toc a:hover { color: #6cb6ff; }
  nav.toc ul ul a { font-size: 13px; color: #98a4b0; }
  nav.toc .toc-section { font-size: 11px; text-transform: uppercase; letter-spacing: 0.08em; color: #5a6878; margin: 16px 0 6px; font-weight: 600; }
  .toc-empty { color: #98a4b0; font-size: 13px; margin: 0; }
  main { max-width: 980px; padding: 56px 56px 96px; }
  .source-strip {
    margin: -24px 0 32px; padding: 10px 14px;
    border: 1px solid #d8d8d0; border-radius: 6px; background: #fff;
    color: #4a5868; font-size: 13px;
    display: flex; flex-wrap: wrap; gap: 10px 18px;
  }
  .source-strip strong { color: #1a2632; }
  .source-strip a { color: #245f8f; text-decoration: none; }
  .source-strip a:hover { color: #6cb6ff; }
  .drift-banner {
    margin: -24px 0 24px; padding: 14px 18px;
    border-left: 4px solid #c8965a; border-radius: 0 6px 6px 0;
    background: #faf4ec; color: #5a3a10; font-size: 14px;
  }
  header.intro { border-bottom: 1px solid #d8d8d0; padding-bottom: 32px; margin-bottom: 48px; }
  header.intro h1 { font-size: 44px; font-weight: 700; margin: 0 0 8px; letter-spacing: -0.02em; color: #1a2632; }
  header.intro .tagline { font-size: 20px; color: #4a5868; margin: 0; font-style: italic; }
  header.intro .meta { margin-top: 16px; font-size: 13px; color: #888; }
  section { margin-bottom: 56px; }
  h2 { font-size: 30px; font-weight: 700; margin: 48px 0 16px; padding-bottom: 8px; border-bottom: 2px solid #1a2632; letter-spacing: -0.015em; color: #1a2632; scroll-margin-top: 32px; }
  h3 { font-size: 22px; font-weight: 600; margin: 32px 0 12px; color: #243446; scroll-margin-top: 32px; }
  h4 { font-size: 17px; font-weight: 600; margin: 20px 0 8px; color: #2c3e50; }
  p { margin: 0 0 16px; }
  strong { color: #1a2632; font-weight: 600; }
  em { color: #4a5868; }
  a { color: #245f8f; }
  a:hover { color: #6cb6ff; }
  code { font-family: ui-monospace, "SF Mono", "Cascadia Code", Consolas, monospace; background: #f0eee8; color: #2a3540; padding: 1px 6px; border-radius: 3px; font-size: 0.92em; }
  pre { background: #1e2a36; color: #d8e0e8; padding: 16px 20px; border-radius: 6px; overflow-x: auto; font-size: 13px; line-height: 1.5; margin: 16px 0; }
  pre code { background: transparent; color: inherit; padding: 0; font-size: 13px; }
  ul, ol { margin: 0 0 16px; padding-left: 24px; }
  li { margin-bottom: 6px; }
  input[type="checkbox"] { margin-right: 8px; }
  blockquote { border-left: 3px solid #d8d4c4; padding: 4px 16px; margin: 16px 0; color: #4a5868; font-style: italic; }
  .callout { border-left: 4px solid; padding: 14px 18px; margin: 20px 0; border-radius: 0 4px 4px 0; font-size: 15px; }
  .callout.note { border-color: #5a8db8; background: #eef4fa; }
  .callout.warning { border-color: #c8965a; background: #faf4ec; }
  .callout.tip { border-color: #6a9a5a; background: #f0f5ec; }
  .callout-title { font-weight: 700; text-transform: uppercase; font-size: 11px; letter-spacing: 0.08em; margin-bottom: 4px; }
  .callout.note .callout-title { color: #2a5a8a; }
  .callout.warning .callout-title { color: #8a5a1a; }
  .callout.tip .callout-title { color: #3a6a2a; }
  table { border-collapse: collapse; width: 100%; margin: 16px 0; font-size: 14px; }
  th, td { padding: 8px 12px; border-bottom: 1px solid #e0dccc; text-align: left; vertical-align: top; }
  th { background: #f0ece0; font-weight: 600; color: #1a2632; }
  img { max-width: 100%; height: auto; border: 1px solid #e0dccc; border-radius: 6px; background: #fff; }
  details { border-top: 1px solid #d8d4c4; border-bottom: 1px solid #d8d4c4; padding: 12px 0; margin: 24px 0; }
  summary { cursor: pointer; color: #1a2632; font-weight: 700; }
  hr { border: none; border-top: 1px solid #d8d4c4; margin: 32px 0; }
  footer { border-top: 1px solid #d8d4c4; padding-top: 24px; margin-top: 64px; font-size: 13px; color: #888; }

  /* Visual abstract — :::flow */
  .journey { display: grid; grid-template-columns: repeat(var(--n,8), 1fr); gap: 0; margin: 0 0 36px; background: #fff; border: 1px solid #d8d4c4; border-radius: 8px; overflow: hidden; }
  .stage { padding: 16px 14px; border-right: 1px solid #ece6d8; min-width: 0; }
  .stage:last-child { border-right: none; }
  .stage .num { font-family: ui-monospace, monospace; font-size: 11px; color: #88a; letter-spacing: 0.04em; font-weight: 700; }
  .stage .verb { font-size: 13px; font-weight: 700; color: #1a2632; margin: 6px 0 4px; line-height: 1.3; }
  .stage .desc { font-size: 11.5px; color: #5a6878; line-height: 1.45; }
  .stage .ann { font-size: 10px; color: #b25d1c; text-transform: uppercase; letter-spacing: 0.04em; font-weight: 700; margin-top: 6px; }
  .stage.gate { background: #f4f0e6; }
  .stage.gate .verb { color: #5a3a10; }

  /* Visual abstract — :::comparison */
  .modes-row { display: grid; grid-template-columns: repeat(var(--n,3), 1fr); gap: 14px; margin: 20px 0 36px; }
  .mode { background: #fff; border: 1px solid #d8d4c4; border-radius: 8px; padding: 18px 20px 16px; border-top: 3px solid #ccc4b0; }
  .mode.new { border-top-color: #245f8f; }
  .mode .mode-tag { display: inline-block; font-size: 10px; font-weight: 700; letter-spacing: 0.08em; text-transform: uppercase; background: #245f8f; color: #fff; padding: 2px 8px; border-radius: 3px; margin-bottom: 8px; }
  .mode .mode-tag.muted { background: #d0c8b4; color: #5a5040; }
  .mode .mode-name { font-size: 18px; font-weight: 700; color: #1a2632; margin: 0 0 6px; }
  .mode .mode-desc { font-size: 14px; color: #4a5868; margin: 0 0 12px; line-height: 1.5; }
  .mode .mode-meta { font-size: 12px; color: #6a7888; line-height: 1.6; }
  .mode .mode-meta .label { color: #98a0aa; text-transform: uppercase; letter-spacing: 0.06em; font-size: 10px; font-weight: 600; }

  /* Visual abstract — :::scope */
  .scope { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 14px; margin: 0 0 36px; }
  .scope-col { background: #fff; border: 1px solid #d8d4c4; border-radius: 8px; padding: 16px 18px; }
  .scope-col.v1 { border-top: 3px solid #6a9a5a; }
  .scope-col.v2 { border-top: 3px solid #c8965a; }
  .scope-col.out { border-top: 3px solid #888; }
  .scope-col h4 { margin: 0 0 10px; font-size: 13px; text-transform: uppercase; letter-spacing: 0.06em; }
  .scope-col.v1 h4 { color: #3a6a2a; }
  .scope-col.v2 h4 { color: #8a5a1a; }
  .scope-col.out h4 { color: #555; }
  .scope-col ul { padding: 0; margin: 0; list-style: none; }
  .scope-col li { font-size: 13px; color: #3a4858; padding: 5px 0 5px 18px; position: relative; line-height: 1.4; }
  .scope-col li::before { position: absolute; left: 0; top: 5px; font-family: ui-monospace, monospace; font-size: 12px; font-weight: 700; }
  .scope-col.v1 li::before { content: "+"; color: #6a9a5a; }
  .scope-col.v2 li::before { content: "›"; color: #c8965a; }
  .scope-col.out li::before { content: "−"; color: #888; }

  /* Visual abstract — :::tree */
  .cap-block { background: #fff; border: 1px solid #d8d4c4; border-radius: 8px; padding: 20px 24px; margin: 0 0 36px; }
  .cap-tree { display: grid; grid-template-columns: repeat(var(--n,3), 1fr); gap: 0; align-items: stretch; }
  .cap-q { padding: 10px 14px; border-right: 1px dashed #c8c2b4; font-size: 13px; }
  .cap-q:last-child { border-right: none; }
  .cap-q .q-text { font-weight: 700; color: #1a2632; margin-bottom: 8px; font-size: 13px; }
  .cap-branch { padding: 6px 0 6px 18px; border-left: 2px solid #d8d4c4; font-size: 12.5px; color: #4a5868; margin-bottom: 4px; }
  .cap-branch.no { border-left-color: #c8965a; }
  .cap-branch.yes { border-left-color: #6a9a5a; }
  .cap-branch .br-label { font-family: ui-monospace, monospace; font-size: 10px; font-weight: 700; letter-spacing: 0.06em; }
  .cap-branch.no .br-label { color: #8a5a1a; }
  .cap-branch.yes .br-label { color: #3a6a2a; }

  /* Status / type pills in the source strip */
  .pill { display: inline-block; padding: 1px 7px; border-radius: 3px; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.06em; }
  .pill.status { background: #e5efe2; color: #3a6a2a; }
  .pill.type { background: #f0ece0; color: #5a5040; }

  @media (max-width: 1100px) {
    .modes-row, .scope, .cap-tree, .journey { grid-template-columns: 1fr; }
    .cap-q { border-right: none; border-bottom: 1px dashed #c8c2b4; }
    .cap-q:last-child { border-bottom: none; }
    .stage { border-right: none; border-bottom: 1px solid #ece6d8; }
    .stage:last-child { border-bottom: none; }
  }
  @media (max-width: 860px) {
    .layout { display: block; }
    nav.toc { position: relative; height: auto; max-height: none; padding: 24px; }
    main { max-width: none; padding: 36px 24px 72px; }
    header.intro h1 { font-size: 34px; }
  }
</style>
</head>
<body>
<div class="layout">
<nav class="toc">
  <h2>{feature_id_slug}</h2>
  <p class="subtitle">Feature brief</p>
  <div class="toc-section">Contents</div>
  {toc_html}
</nav>
<main>
  <div class="source-strip">
    <span><strong>Feature:</strong> {feature_id_slug}</span>
    {status_pill}
    {type_pill}
    <span><strong>Source date:</strong> {generated_date}</span>
    <span><a href="{interview_link}">Open interview</a></span>
  </div>
  <header class="intro">
    <h1>{title}</h1>
    <p class="tagline">Feature brief — composed from the PRD, interview, and gate manifests.</p>
    <p class="meta">Generated from {feature_id_slug}-prd.md, {feature_id_slug}-interview.md, and (when present) debate-log/.</p>
  </header>
  {visual_abstract_html}
  {prd_body_html}
  {interview_body_html}
  {reviews_html}
  <footer>
    Feature brief · composed from <code>{feature_id_slug}-prd.md</code>, <code>{feature_id_slug}-interview.md</code>, and the gate debate logs. Markdown is the source of truth.
  </footer>
</main>
</div>
</body>
</html>
```

Drift banner HTML, inserted only when required:

```html
  <div class="drift-banner">This rendered brief was stale when composing started. The file has now been refreshed from the latest sources.</div>
```

Section wrappers, used inside the placeholders above:

```html
<!-- visual_abstract_html: -->
<section id="visual-abstract">
  <h2>Visual abstract</h2>
  {compiled_visual_blocks}
</section>

<!-- prd_body_html: -->
<section id="prd">
  {prd_body_converted_to_html}
</section>

<!-- interview_body_html: -->
<section id="interview">
  <h2>Interview</h2>
  {interview_body_converted_to_html}
</section>

<!-- reviews_html (only if at least one manifest exists): -->
<section id="agent-reviews">
  <h2>Agent reviews</h2>
  {gate2_manifest_html_if_present}
  {gate3_manifest_html_if_present}
</section>
```

If a section's source content is absent (no Visual abstract blocks, no manifests), set its placeholder to the empty string — the surrounding `<section>` is part of the substituted content, so removing the content also removes the container.

## Binary Verify Steps

Run these after composing each feature:

```sh
test -f "features/NNN-{slug}/NNN-{slug}-brief.html"
test -f "features/NNN-{slug}/NNN-{slug}-interview.md"
grep -q 'href="NNN-{slug}-interview.md"' "features/NNN-{slug}/NNN-{slug}-brief.html"
grep -q '<nav class="toc">' "features/NNN-{slug}/NNN-{slug}-brief.html"
grep -q '<main>' "features/NNN-{slug}/NNN-{slug}-brief.html"
grep -q 'Feature brief' "features/NNN-{slug}/NNN-{slug}-brief.html"
```

Idempotence check:

```sh
tmp="$(mktemp)"
cp "features/NNN-{slug}/NNN-{slug}-brief.html" "$tmp"
```

Re-run `/specflow:brief NNN-{slug}`, then run:

```sh
cmp -s "features/NNN-{slug}/NNN-{slug}-brief.html" "$tmp"
rm -f "$tmp"
```

Bulk check:

```sh
find features -path 'features/[0-9][0-9][0-9]-*/[0-9][0-9][0-9]-*-prd.md' -print | sort
find features -path 'features/[0-9][0-9][0-9]-*/[0-9][0-9][0-9]-*-brief.html' -print | sort
```

## What you MUST NOT do

- **Do not paraphrase or summarise the PRD body, interview, or manifest content.** Faithful render only. The only interpretive step is compiling the four structured visual block kinds.
- **Do not invent visual block kinds.** Only `flow`, `comparison`, `scope`, `tree` are supported. Other `:::xxx` blocks render as a `<pre>` with a "Unsupported visual block" warning.
- **Do not pull in external assets.** No CDN, no script tags, no external CSS. Self-contained file only.
- **Do not run a build step or install packages.** Compose markdown to HTML in the current runtime.
- **Do not modify the source markdown.** Brief is read-only on its inputs.
- **Do not mention Claude, Anthropic, or any AI tooling** in the brief or any output. Per the project's CLAUDE.md, this is non-negotiable.

## Reference

- `skills/prd/SKILL.md` Phase E — invokes this skill after Gate 2 closes.
- `skills/doctor/SKILL.md` `features.{NNN-slug}.brief_drift` — checks freshness of the brief file.
- `skills/upgrade/SKILL.md` step 10 — bulk regenerates briefs after PRD relocation.
- `MIGRATIONS.md` v2.1 → v2.2 — replaces the per-feature `-prd.html` with `-brief.html`.
