---
name: specflow:design
description: Generate HTML/CSS mockups for a feature, grounded in the live codebase. Five-phase orchestrator — A pre-flight + target detection + interview, B value extraction (codebase-truth principle, no invented values), C generate current.html + proposed.html, D Playwright iteration loop until visual diff is below threshold or user accepts drift, E optional Codex semantic review. Every iteration appends a decision-capture entry to the iteration log.
status: v2-new
phase: 1
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/admin/config.json
  - docs/specflow/admin/environment.json
produces:
  - docs/specflow/features/{NNN-slug}/design/{NNN-slug}-current.html
  - docs/specflow/features/{NNN-slug}/design/{NNN-slug}-proposed.html
  - docs/specflow/features/{NNN-slug}/design/{NNN-slug}-iteration-log.md
  - docs/specflow/features/{NNN-slug}/assets/
eval: both current.html and proposed.html exist; comment block lists every extracted value with source file:line; Playwright visual diff is below threshold OR user explicitly accepted remaining drift; iteration log captures every change with a non-empty *Why* field (empty *Why* fields are a verify-step failure).

---

# specflow:design

Generate HTML/CSS mockups for a feature. **Codebase-truth principle is non-negotiable**: every value used in a mockup is extracted from the live source, listed in the comment block, and auditable. No invented values.

This is a **5-phase skill**: pre-flight → value extraction → generate → Playwright iteration → optional Codex semantic review. Every iteration is recorded in the iteration log with the *decision* behind it (PRD Appendix C3.1).

---

## Inputs

The user invokes you with:
- `specflow:design {NNN-slug}` — feature must already have a PRD.
- `/specflow:design` with no argument — ask the user which feature.
- `specflow:design {NNN-slug} --iterate` — resume on an existing design; loops Phase D + iteration log without regenerating.

**Resume logic.** Before starting Phase A:

1. Locate `features/NNN-{slug}/`. Refuse if missing: *"Feature `{NNN-slug}` does not exist. Run `specflow:prd {free-form overview}` first."*
2. Verify the PRD exists.
3. Determine the resume point:
   - **No `design/` folder, or no current.html / proposed.html** → start Phase A (full pass).
   - **Both HTMLs + iteration log exist** → ask: *"Design exists for `{NNN-slug}`. (1) iterate (Phase D loop), (2) regenerate from scratch — old design archived, (3) Codex semantic review only (Phase E)."*
   - `--iterate` flag forces option 1.

Tell the user explicitly which phase you're starting at.

---

## Phase A — Pre-flight + target detection + interview

### A.1 Read inputs in parallel

- `features/NNN-{slug}/{NNN-slug}-prd.md`
- `features/NNN-{slug}/{NNN-slug}-interview.md` (for goal context)
- `admin/config.json` — defaults for mobile/web target, viewport, frame
- `admin/environment.json` — Playwright + Codex availability

### A.2 Target detection

Determine: web or mobile?

- **From PRD signals** — Vision and Users sections often name the surface. *"Login flow on the iOS app"* → mobile. *"Header notifications popover"* → web.
- **From config defaults** — `config.json.design.defaultTarget` if set.
- **Ask if ambiguous** — *"Web or mobile? PRD didn't make this explicit. Default for this project is `{config-default}`."*

For mobile, also pick the frame: iOS / Android / generic. Use `config.json.design.mobileFrame` default; ask if unset.

### A.3 Mini-interview

Ask the user (in chat, brief):
1. *"Which page or component is this mockup about?"* — used to scope value extraction in Phase B.
2. *"What's the change? Visual refresh, new component, layout shift?"* — shapes the proposed.html direction.
3. *"What conversation will this mockup support?"* — the goal of the design artefact, written into the banner at the top of each HTML.

Write the answers into a working note for Phase B; surface them in the iteration log's first entry.

---

## Phase B — Value extraction (codebase-truth)

### B.1 Locate the source files

Use Glob and Grep to find:
- The component(s) named in A.3.
- The theme / design tokens file (`tailwind.config.*`, `theme.ts`, `styles/tokens.*`, etc.).
- The shared layout primitives the component uses.
- Any `.css` / `.scss` / `.module.css` adjacent to the component.

If the source for a referenced component cannot be found, **stop and ask** rather than guess. Tell the user: *"Couldn't locate source for `{component}`. Search returned: {what was found}. Help me find it before I generate anything."*

### B.2 Extract values

Read every located file and pull:
- **Colours** — every hex / rgb / hsl / oklch / theme-token reference. Cite file:line.
- **Typography** — font family, size, weight, line-height. File:line.
- **Spacing** — margin, padding, gap values. File:line.
- **Borders + radii** — file:line.
- **Breakpoints + viewport sizes** — file:line.
- **Shadows + effects** — file:line.

For component shape (DOM structure), pull the JSX/template structure verbatim — paraphrase only when the framework's syntax doesn't translate to HTML directly.

### B.3 Produce the comment block

Every generated HTML opens with this block (proposed.html includes the same block):

```html
<!--
specflow:design — codebase-truth comment block
Feature: NNN-slug
Generated: {YYYY-MM-DD HH:MM}
Source files: {comma-separated list}

Extracted values:
  Colours
    --bg-primary: #1a1d29 (src/theme/tokens.ts:14)
    --text-primary: #e6e8ee (src/theme/tokens.ts:15)
    ...
  Typography
    font-family-body: "Inter", system-ui (src/theme/tokens.ts:42)
    font-size-md: 16px (src/theme/tokens.ts:51)
    ...
  Spacing
    space-3: 12px (src/theme/tokens.ts:67)
    space-4: 16px (src/theme/tokens.ts:68)
    ...
  Component shape
    Notification badge: src/components/NotificationBadge.tsx:1-48
    Popover container: src/components/Popover.tsx:12-94
-->
```

If a value used in the HTML is NOT in the comment block, that's a violation of the codebase-truth principle. Re-walk the HTML and either remove the value or add it to the block (with the file:line).

---

## Phase C — Generate current.html + proposed.html

### C.1 Create the design folder

```bash
mkdir -p docs/specflow/features/NNN-{slug}/design
mkdir -p docs/specflow/features/NNN-{slug}/assets
```

### C.2 Write `{NNN-slug}-current.html`

Faithful HTML/CSS rendering of how the feature looks **today**. Inline `<style>`, no JS, no external assets. Include the comment block (B.3) verbatim at the top.

Banner at the top of the body (visible to readers):

```html
<div class="banner">This is a discussion mockup, not a final design — current state of {feature page/component} as of {YYYY-MM-DD}.</div>
```

For mobile target: wrap the body in the phone frame (iOS / Android / generic) per `config.json.design.mobileFrame`.

### C.3 Write `{NNN-slug}-proposed.html`

Same skeleton; the proposed direction. **Same value-extraction rules** — the proposed mockup still draws values from the existing design system unless the user explicitly asked for a departure (e.g. "introduce a new accent colour"). New values are listed in the comment block as `proposed (no source)` with a one-line rationale.

Banner:

```html
<div class="banner">This is a discussion mockup, not a final design — proposed direction for {what the change covers}.</div>
```

### C.4 Initialise the iteration log

Use Write tool to create `{NNN-slug}-iteration-log.md` with the first entry:

```markdown
# Design iteration log — features/NNN-{slug}

## Iteration 1 — {YYYY-MM-DD HH:MM} — Initial generation

**Files changed:**
- `design/{NNN-slug}-current.html` — created.
- `design/{NNN-slug}-proposed.html` — created.

**What changed:**
- Initial generation. current.html captures live state; proposed.html drafts the direction.

**Why (the decision):**
- {Cite the user's mini-interview answers from Phase A.3.} The proposed direction takes {approach} because {reason grounded in the PRD's goal section / interview round / extracted values}. Alternative considered: {what was considered and why it lost}.

**Triggered by:** `prd-clarification`.

**Outstanding:**
- Playwright iteration loop pending.
```

---

## Phase D — Playwright iteration loop

### D.1 Capture the live baseline

If the dev server is reachable (check via curl or `playwright connect`):
1. Boot the local dev server if needed.
2. Use Playwright to navigate to the live page.
3. Capture a screenshot at the agreed viewport. Save to `assets/{NNN-slug}-live-baseline.png`.

If the dev server isn't reachable: ask the user for a manually captured screenshot, save it to the same path. The skill works either way; the iteration loop just becomes user-driven on the live side.

### D.2 Capture the mockup

Use Playwright (or a local browser via Bash) to render `{NNN-slug}-current.html` and capture the same viewport. Save to `assets/{NNN-slug}-mockup-current-iter-{N}.png`.

### D.3 Diff

Use Playwright's image diff (or an equivalent tool detected in `environment.json`):
- Pixel diff at the configured threshold (`config.json.design.diffThreshold`, default `0.02` = 2% of pixels).
- Per-region annotation: where do the differences cluster?

Surface the diff to the user:
- *"Iteration {N}: pixel diff = {x}%. Largest cluster in {region}. Suggested fixes: {list}."*

### D.4 Iterate or accept

Wait for the user's response:
- **Apply** — make the suggested fixes (or the user's specific instructions). Edit the relevant HTML. Re-capture and diff (D.2 + D.3). Increment iteration number.
- **Accept** — the user signs off on remaining drift. Record the acceptance in the iteration log.
- **Stop** — abort the loop without acceptance. Record the stop in the iteration log.

### D.5 Append to iteration log (every iteration)

For each iteration, append (per PRD Appendix C3.1):

```markdown
## Iteration {N} — {YYYY-MM-DD HH:MM} — {short title from the change made}

**Files changed:**
- `design/{NNN-slug}-current.html` — {section / property modified}.

**What changed:**
- {Concrete diff: e.g. "header padding 8px → 12px; primary button radius 4px → 6px"}.

**Why (the decision):**
- {The reasoning. Cite the Playwright diff finding / user feedback / Codex review / manual observation that drove this. What alternative was considered and why this won.}

**Triggered by:** `playwright-diff` | `user-feedback` | `codex-review` | `manual-observation` | `prd-clarification`.

**Playwright drift:**
- Before: {pixel-diff %} (largest cluster: {region})
- After: {pixel-diff %} (largest cluster: {region or 'none'})

**Outstanding:**
- {Anything this iteration didn't resolve, or 'none — diff below threshold'}
```

**Empty *Why* fields are a verify-step failure.** If you don't have a reason for the change, you shouldn't be making it.

---

## Phase E — Optional Codex semantic review

If `admin/environment.json.tools.codex.available == true`:

### E.1 Invoke Codex

Pass Codex:
- Both HTMLs.
- The PRD's Goal section + Vision.
- The user's stated goal from A.3.
- The iteration log.

Codex's job: flag **semantic** gaps the Playwright loop misses. Examples:
- "The proposed flow loses the cancel affordance present in current.html — was that intentional?"
- "Both mockups assume notifications are unread by default; PRD R2 says they should land as read for users on the privacy track."
- "The proposed colour scheme reduces contrast between primary text and primary surface from WCAG AA to below — was that intentional?"

### E.2 Surface findings

For each Codex finding, append a line to a new iteration entry triggered by `codex-review`:

```markdown
## Iteration {N+1} — {YYYY-MM-DD HH:MM} — Codex review

**Files changed:** none yet — surfacing findings for user decision.

**What changed:** none yet.

**Why (the decision):** Codex review identified semantic gaps to consider before sign-off.

**Triggered by:** `codex-review`.

**Codex findings:**
- *Finding 1:* {verbatim from Codex output}.
- *Finding 2:* {...}

**Outstanding:**
- User decides which findings to address. Each decision becomes its own iteration entry (Triggered by: `codex-review` for the addressed ones, `manual-observation` for explicit dismissals with rationale).
```

If Codex isn't installed, skip Phase E gracefully. Note in the iteration log: *"Codex semantic review — not available; skipped."*

---

## Final disposition

### Success

Tell the user:
```
Design generated for {NNN-slug}.
- Live baseline: assets/{NNN-slug}-live-baseline.png
- Mockups: design/{NNN-slug}-current.html, design/{NNN-slug}-proposed.html
- Iteration log: design/{NNN-slug}-iteration-log.md ({N} iterations)
- Final Playwright drift: {x}% ({status: below threshold | accepted as-is})
- Codex review: {findings count, or 'not available'}

Open the proposed mockup in a browser:
  open docs/specflow/features/{NNN-slug}/design/{NNN-slug}-proposed.html
```

### Failure

If the loop bailed (user said "stop", or the dev server fundamentally couldn't be reached): leave both HTMLs and the log in place; report the unresolved state explicitly. *"Design partially generated. Final state recorded in iteration log. Re-run `specflow:design {NNN-slug} --iterate` to resume."*

---

## What you MUST NOT do

- **Do not invent values.** Every colour, font, spacing, breakpoint comes from the source. The comment block is the audit trail; if it doesn't list a value, the value is not in the HTML.
- **Do not skip Phase B's "stop and ask" rule.** A missing source file means the AI is about to guess — refuse and ask the user.
- **Do not write iteration log entries with empty *Why* fields.** Empty *Why* is a verify-step failure (PRD Appendix C3.1). The change shouldn't have happened without a recorded decision.
- **Do not rewrite past iteration log entries.** Append-only. Reversals are new entries citing the original iteration number.
- **Do not commit to a final design without user sign-off** on the iteration loop's exit (either "diff below threshold" or explicit "accept remaining drift").
- **Do not invoke `specflow:scope-change` automatically** if Codex flags an apparent contradiction with the PRD. Surface the finding; the user decides whether the PRD or the design needs adjustment.
- **Do not mention the underlying AI tooling or vendor** in the HTMLs (including comment blocks), the iteration log, or any output. Per the project's CLAUDE.md.

---

## Verify before declaring done

1. `design/{NNN-slug}-current.html` and `design/{NNN-slug}-proposed.html` both exist and parse as HTML.
2. Comment block at the top of each HTML lists every extracted value with source file:line; no value appears in the HTML body that's missing from the comment block.
3. `design/{NNN-slug}-iteration-log.md` exists with at least the Phase C initial entry.
4. **Every iteration log entry has a non-empty *Why* field.** Walk the log and check. Empty *Why* = verify-step failure.
5. Playwright diff is below threshold OR the iteration log records an explicit "user accepted drift" entry.
6. (If Codex available) iteration log records the Codex review (findings or "no findings").
7. `assets/` contains the screenshots referenced from the iteration log.

If any verify step fails, fix it before returning. Do NOT claim the design is complete with empty *Why* fields or unrecorded iterations.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 11 — `specflow:design`.
- `docs/PRD.md` Appendix C — full skill spec.
- `docs/PRD.md` Appendix C3.1 — iteration log structure (decision capture).
- `docs/PRD.md` Appendix G — test asset support (shared `features/NNN-{slug}/assets/` folder with `specflow:test`).
- `templates/orchestrator-pattern.md` — Codex invocation as a forked sub-agent.
- `skills/test/SKILL.md` — sister skill; shared assets folder.
