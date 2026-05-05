---
name: specflow:render
description: Render features/NNN-{slug}/NNN-{slug}-prd.md to a self-contained, browser-readable NNN-{slug}-prd.html for human review. Auto-fires after specflow:prd writes; manual via /specflow:render; bulk via --all.
status: v2-new
phase: 1
requires: [docs/specflow/features/{NNN-slug}/NNN-{slug}-prd.md]
produces: [docs/specflow/features/{NNN-slug}/NNN-{slug}-prd.html]
eval: features/NNN-{slug}/NNN-{slug}-prd.html exists; a second render from unchanged markdown is byte-identical; header link to NNN-{slug}-interview.md resolves.
---

# specflow:render

Render one PRD markdown file to one sibling HTML file:

`features/NNN-{slug}/NNN-{slug}-prd.md` -> `features/NNN-{slug}/NNN-{slug}-prd.html`

Markdown is the source of truth. Do not render `NNN-{slug}-interview.md`, `NNN-{slug}-tasks.md`, `NNN-{slug}-test.md`, or any other markdown file.

No build step, package install, network call, script tag, or external asset is allowed. Perform the markdown-to-HTML conversion directly in the current runtime and write the final HTML file to disk.

## Inputs

- `/specflow:render NNN-{slug}` renders exactly `docs/specflow/features/NNN-{slug}/NNN-{slug}-prd.md`.
- `/specflow:render --all` finds every `docs/specflow/features/[0-9][0-9][0-9]-*/[0-9][0-9][0-9]-*-prd.md` and renders each one independently.
- If the current working directory is already `docs/specflow`, use paths relative to that folder. Otherwise use `docs/specflow/...` from the repo root.

Abort with a clear message if the PRD markdown is missing, the feature folder name and PRD filename prefix differ, or the target path would leave the feature folder.

## Determinism

The output must be byte-identical for unchanged input.

- Set `{generated_date}` to the PRD markdown's source timestamp, not the render time.
- Source timestamp rule: use `git log -1 --format=%cI -- <prd.md>` when the file is tracked and has a commit; otherwise use filesystem mtime as ISO-8601 seconds with timezone.
- Sort generated IDs, attributes, table rows created by the renderer, and bulk `--all` file order lexicographically.
- Do not include random IDs, current clock time, machine-specific absolute paths, or tool version strings.
- Preserve intentional markdown content, but normalize generated HTML indentation exactly as in the template.

## Step-By-Step Execution

1. Resolve scope.
   - Single feature: validate `NNN-{slug}` matches `^[0-9]{3}-[a-z0-9]+(?:-[a-z0-9]+)*$`.
   - Bulk: list matching PRDs under `features/`, sorted lexicographically.
   - Verify: `test -f "features/NNN-{slug}/NNN-{slug}-prd.md"`.

2. Read markdown.
   - Read the full PRD markdown as UTF-8.
   - Derive `{feature_id_slug}` from the parent folder name.
   - Derive `{title}` from the first H1. If absent, use `{feature_id_slug} PRD`.
   - Set `{interview_link}` to `NNN-{slug}-interview.md`.
   - Verify: the derived feature prefix equals the parent folder and the sibling interview path is inside the same folder.

3. Parse structure.
   - Convert headings to stable slug IDs. Slug rule: lowercase, strip HTML, replace non-alphanumeric runs with `-`, trim `-`, append `-2`, `-3` for duplicates.
   - Build `{toc_html}` from H2 and H3 headings only. If there are no H2/H3 headings, emit `<p class="toc-empty">No sections found.</p>`.
   - Convert the full markdown body to `{body_html}` using the rules below.
   - Verify: every TOC `href="#..."` has exactly one matching heading `id`.

4. Resolve assets.
   - Resolve markdown image destinations relative to the feature folder.
   - For local files under 50 KB, inline as `data:{mime};base64,{payload}`.
   - For local files 50 KB or larger, keep a relative URL from the HTML file to the asset.
   - Leave `http:`, `https:`, `mailto:`, and anchor links unchanged.
   - Abort if a local image reference resolves outside the feature folder.
   - Verify: every non-remote local image destination exists or the render aborts before writing.

5. Emit HTML.
   - Fill the template exactly once.
   - HTML-escape all interpolated text except `{toc_html}` and `{body_html}`, which are already sanitized renderer output.
   - Show the drift banner only when an existing HTML file is present and its mtime is older than the PRD markdown mtime at the start of the render. A fresh successful render should omit the banner on the next unchanged run.
   - Write to `features/NNN-{slug}/NNN-{slug}-prd.html`.
   - Verify: `test -f "features/NNN-{slug}/NNN-{slug}-prd.html"`.

6. Verify idempotence.
   - Render the same input to a temporary file using the same algorithm.
   - Compare bytes with the just-written HTML.
   - Verify: `cmp -s "features/NNN-{slug}/NNN-{slug}-prd.html" "$tmpfile"`.

## Markdown Conversion Rules

Use CommonMark plus these GFM extensions:

- Tables.
- Task lists: `- [ ]` and `- [x]` become disabled checkboxes.
- Strikethrough: `~~text~~`.
- Autolinks.

Required custom handling:

- Escape raw HTML from markdown by default. Do not pass through `<script>`, inline event handlers, or external stylesheet links.
- Fenced code blocks become `<pre><code class="language-{lang}">...</code></pre>`. Escape code text. Add language class when a fence language is present.
- Inline code becomes `<code>...</code>`.
- Blockquotes beginning with `[!NOTE]`, `[!WARNING]`, or `[!TIP]` become `<div class="callout note|warning|tip">` with a `.callout-title`. Other blockquotes remain `<blockquote>`.
- Markdown tables become `<table>` with `<thead>` when a header row exists.
- Horizontal rules become `<hr>`.
- Links are escaped and normalized. Local markdown links remain relative to the feature folder.
- Images get `loading="lazy"` and escaped alt text.
- Mermaid fences are not rendered with client-side script. In Phase 1, render them as syntax-highlighted code blocks with `language-mermaid`.
- Long appendices may use native `<details>` only when the markdown heading text starts with `Appendix `; keep the heading visible as the summary.

## HTML Template

Use this template verbatim. Replace exactly these placeholders: `{title}`, `{feature_id_slug}`, `{generated_date}`, `{toc_html}`, `{body_html}`, `{interview_link}`. When a drift banner is required, insert it as the first element inside `<main>` before `<div class="source-strip">`; otherwise insert nothing.

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
  .layout {
    display: grid;
    grid-template-columns: 280px 1fr;
    gap: 0;
    min-height: 100vh;
  }
  nav.toc {
    position: sticky;
    top: 0;
    align-self: start;
    height: 100vh;
    overflow-y: auto;
    padding: 32px 24px;
    background: #1a2632;
    color: #d8d8d4;
    border-right: 1px solid #283442;
  }
  nav.toc h2 {
    margin: 0 0 4px;
    font-size: 18px;
    color: #ffffff;
    letter-spacing: -0.01em;
  }
  nav.toc .subtitle {
    margin: 0 0 24px;
    font-size: 12px;
    color: #88a;
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }
  nav.toc ul {
    list-style: none;
    padding: 0;
    margin: 0 0 24px;
  }
  nav.toc ul ul {
    margin: 4px 0 12px 12px;
    padding-left: 12px;
    border-left: 1px solid #2a3a4a;
  }
  nav.toc li { margin: 6px 0; }
  nav.toc a {
    color: #c8d0d8;
    text-decoration: none;
    font-size: 14px;
    line-height: 1.4;
    display: block;
    padding: 2px 0;
    transition: color 0.15s;
  }
  nav.toc a:hover { color: #6cb6ff; }
  nav.toc ul ul a { font-size: 13px; color: #98a4b0; }
  nav.toc .toc-section {
    font-size: 11px;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: #5a6878;
    margin: 16px 0 6px;
    font-weight: 600;
  }
  .toc-empty { color: #98a4b0; font-size: 13px; margin: 0; }
  main {
    max-width: 920px;
    padding: 56px 56px 96px;
  }
  .source-strip {
    margin: -24px 0 32px;
    padding: 10px 14px;
    border: 1px solid #d8d8d0;
    border-radius: 6px;
    background: #ffffff;
    color: #4a5868;
    font-size: 13px;
    display: flex;
    flex-wrap: wrap;
    gap: 10px 18px;
  }
  .source-strip strong { color: #1a2632; }
  .source-strip a { color: #245f8f; text-decoration: none; }
  .source-strip a:hover { color: #6cb6ff; }
  .drift-banner {
    margin: -24px 0 24px;
    padding: 14px 18px;
    border-left: 4px solid #c8965a;
    border-radius: 0 6px 6px 0;
    background: #faf4ec;
    color: #5a3a10;
    font-size: 14px;
  }
  header.intro {
    border-bottom: 1px solid #d8d8d0;
    padding-bottom: 32px;
    margin-bottom: 48px;
  }
  header.intro h1 {
    font-size: 44px;
    font-weight: 700;
    margin: 0 0 8px;
    letter-spacing: -0.02em;
    color: #1a2632;
  }
  header.intro .tagline {
    font-size: 20px;
    color: #4a5868;
    margin: 0;
    font-style: italic;
  }
  header.intro .meta {
    margin-top: 16px;
    font-size: 13px;
    color: #888;
  }
  section { margin-bottom: 56px; }
  h2 {
    font-size: 30px;
    font-weight: 700;
    margin: 48px 0 16px;
    padding-bottom: 8px;
    border-bottom: 2px solid #1a2632;
    letter-spacing: -0.015em;
    color: #1a2632;
    scroll-margin-top: 32px;
  }
  h3 {
    font-size: 22px;
    font-weight: 600;
    margin: 32px 0 12px;
    color: #243446;
    scroll-margin-top: 32px;
  }
  h4 {
    font-size: 17px;
    font-weight: 600;
    margin: 20px 0 8px;
    color: #2c3e50;
  }
  p { margin: 0 0 16px; }
  strong { color: #1a2632; font-weight: 600; }
  em { color: #4a5868; }
  a { color: #245f8f; }
  a:hover { color: #6cb6ff; }
  code {
    font-family: ui-monospace, "SF Mono", "Cascadia Code", Consolas, "Roboto Mono", monospace;
    background: #f0eee8;
    color: #2a3540;
    padding: 1px 6px;
    border-radius: 3px;
    font-size: 0.92em;
  }
  pre {
    background: #1e2a36;
    color: #d8e0e8;
    padding: 16px 20px;
    border-radius: 6px;
    overflow-x: auto;
    font-size: 13px;
    line-height: 1.5;
    margin: 16px 0;
  }
  pre code {
    background: transparent;
    color: inherit;
    padding: 0;
    font-size: 13px;
  }
  .language-json .token-string, .language-yaml .token-string { color: #9bd17c; }
  .language-js .token-keyword, .language-ts .token-keyword, .language-sh .token-keyword { color: #8bb9ff; }
  .language-js .token-string, .language-ts .token-string, .language-sh .token-string { color: #d7ba7d; }
  .language-js .token-comment, .language-ts .token-comment, .language-sh .token-comment { color: #7f8a96; }
  ul, ol { margin: 0 0 16px; padding-left: 24px; }
  li { margin-bottom: 6px; }
  li > strong:first-child { color: #1a2632; }
  input[type="checkbox"] { margin-right: 8px; }
  blockquote {
    border-left: 3px solid #d8d4c4;
    padding: 4px 16px;
    margin: 16px 0;
    color: #4a5868;
    font-style: italic;
  }
  .callout {
    border-left: 4px solid;
    padding: 14px 18px;
    margin: 20px 0;
    border-radius: 0 4px 4px 0;
    font-size: 15px;
  }
  .callout.note { border-color: #5a8db8; background: #eef4fa; }
  .callout.warning { border-color: #c8965a; background: #faf4ec; }
  .callout.tip { border-color: #6a9a5a; background: #f0f5ec; }
  .callout-title {
    font-weight: 700;
    text-transform: uppercase;
    font-size: 11px;
    letter-spacing: 0.08em;
    margin-bottom: 4px;
  }
  .callout.note .callout-title { color: #2a5a8a; }
  .callout.warning .callout-title { color: #8a5a1a; }
  .callout.tip .callout-title { color: #3a6a2a; }
  table {
    border-collapse: collapse;
    width: 100%;
    margin: 16px 0;
    font-size: 14px;
  }
  th, td {
    padding: 8px 12px;
    border-bottom: 1px solid #e0dccc;
    text-align: left;
    vertical-align: top;
  }
  th {
    background: #f0ece0;
    font-weight: 600;
    color: #1a2632;
  }
  img {
    max-width: 100%;
    height: auto;
    border: 1px solid #e0dccc;
    border-radius: 6px;
    background: #ffffff;
  }
  details {
    border-top: 1px solid #d8d4c4;
    border-bottom: 1px solid #d8d4c4;
    padding: 12px 0;
    margin: 24px 0;
  }
  summary {
    cursor: pointer;
    color: #1a2632;
    font-weight: 700;
  }
  hr {
    border: none;
    border-top: 1px solid #d8d4c4;
    margin: 32px 0;
  }
  footer {
    border-top: 1px solid #d8d4c4;
    padding-top: 24px;
    margin-top: 64px;
    font-size: 13px;
    color: #888;
  }
  footer code { background: #f0eee8; }
  @media (max-width: 860px) {
    .layout { display: block; }
    nav.toc {
      position: relative;
      height: auto;
      max-height: none;
      padding: 24px;
    }
    main {
      max-width: none;
      padding: 36px 24px 72px;
    }
    header.intro h1 { font-size: 34px; }
    header.intro .tagline { font-size: 18px; }
  }
</style>
</head>
<body>
<div class="layout">
<nav class="toc">
  <h2>{feature_id_slug}</h2>
  <p class="subtitle">PRD render</p>
  <div class="toc-section">Contents</div>
  {toc_html}
</nav>
<main>
  <div class="source-strip">
    <span><strong>Feature:</strong> {feature_id_slug}</span>
    <span><strong>Source date:</strong> {generated_date}</span>
    <span><a href="{interview_link}">Open interview</a></span>
  </div>
  <header class="intro">
    <h1>{title}</h1>
    <p class="tagline">Markdown is the source of truth.</p>
    <p class="meta">Generated from {feature_id_slug}-prd.md.</p>
  </header>
  {body_html}
  <footer>
    Generated from <code>{feature_id_slug}-prd.md</code>. Markdown is the source of truth.
  </footer>
</main>
</div>
</body>
</html>
```

Drift banner HTML, inserted only when required:

```html
  <div class="drift-banner">This rendered PRD was stale when rendering started. The file has now been refreshed from the markdown source.</div>
```

## Binary Verify Steps

Run these after rendering each feature:

```sh
test -f "features/NNN-{slug}/NNN-{slug}-prd.html"
test -f "features/NNN-{slug}/NNN-{slug}-interview.md"
grep -q 'href="NNN-{slug}-interview.md"' "features/NNN-{slug}/NNN-{slug}-prd.html"
grep -q '<nav class="toc">' "features/NNN-{slug}/NNN-{slug}-prd.html"
grep -q '<main>' "features/NNN-{slug}/NNN-{slug}-prd.html"
```

Idempotence check:

```sh
tmp="$(mktemp)"
cp "features/NNN-{slug}/NNN-{slug}-prd.html" "$tmp"
```

Re-run `/specflow:render NNN-{slug}`, then run:

```sh
cmp -s "features/NNN-{slug}/NNN-{slug}-prd.html" "$tmp"
rm -f "$tmp"
```

Bulk check:

```sh
find features -path 'features/[0-9][0-9][0-9]-*/[0-9][0-9][0-9]-*-prd.md' -print | sort
find features -path 'features/[0-9][0-9][0-9]-*/[0-9][0-9][0-9]-*-prd.html' -print | sort
```
