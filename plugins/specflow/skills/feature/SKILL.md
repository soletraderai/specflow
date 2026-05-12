---
name: specflow:feature
description: Feature kickoff — runs FIRST in the specflow pipeline. Allocates the NNN-slug, scaffolds the feature folder + subfolders (design, docs, assets, test/screenshots, debate-log), runs a four-question goal interview, reflects what it heard, and writes a slim per-feature meta file ({NNN-slug}-feature.md) that downstream skills read. The goal is locked at kickoff; subsequent skills inherit it without re-asking.
status: v2-new
phase: 1
requires:
  - docs/specflow/admin/CONTEXT.md
  - docs/specflow/admin/profiles.json
  - docs/specflow/admin/environment.json
produces:
  - docs/specflow/features/{NNN-slug}/
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-feature.md
  - docs/specflow/features/{NNN-slug}/{design,docs,assets,test/screenshots,debate-log}/.gitkeep
eval: feature folder exists with all five subfolders; .gitkeep present in each subfolder; {NNN-slug}-feature.md exists with valid frontmatter (slug, status=kickoff, created, goal_locked); Goal section present and ≤2 paragraphs; Folder index lists every standard artefact path; user confirmed the goal before write.
---

# specflow:feature

Feature kickoff. Runs FIRST — before `specflow:prd`, before any artefact synthesis. The job is to identify what we're working towards, lock the goal in one slim file, and scaffold the folder structure so subsequent skills have a place to put their output.

The goal is two paragraphs max. The meta file is slim by design — frontmatter for structured admin (status, dates), prose for the goal, a folder index so the AI and humans both know what belongs where. Downstream skills read this file and inherit the goal without re-asking the strategic questions.

Goal changes after kickoff are a direct edit to the meta file by the user. No new skill needed; downstream skills pick up the new value on their next read.

This is a **4-phase skill** (A → B → C → D). No sub-agent forking; no multi-agent gate; no Codex pass. Lightweight by design — the heavy lifting happens downstream.

---

## Inputs

The user invokes you with one of:

- `/specflow:feature {one-line overview}` — start a new feature from a one-line description.
- `/specflow:feature` with no overview — prompt the user for the overview, then proceed.

If the user passes `--slug {custom-slug}`, use it verbatim (validate kebab-case). Otherwise propose a slug from the overview.

Tell the user explicitly: *"`/specflow:feature` for `{overview}`. Allocating slug and scaffolding folder."*

---

## Phase A — Pre-flight + scaffold

### A.1 Allocate the NNN-slug

1. Glob `docs/specflow/features/*/` to list existing feature folders.
2. Parse the highest existing `NNN` prefix.
3. Increment by 1; zero-pad to 3 digits. This is the new `NNN`.
4. Propose a slug from the user's overview: kebab-case, 2-4 words, descriptive. Example overview *"add notifications popover to the header"* → `notifications-popover`.
5. Present: *"I'll create `NNN-{slug}`. Confirm or edit the slug."*
6. On confirm: proceed. On edit: validate kebab-case (no spaces, no path separators, no leading digits beyond the NNN); re-prompt on invalid.

### A.2 Refuse if the folder already exists

If `docs/specflow/features/{NNN-slug}/` already exists:

- **Folder exists, no `{NNN-slug}-feature.md`** → idempotent mode. Skip to Phase B; the goal interview still runs. At D.1, write the meta file + scaffold missing subfolders without disturbing existing files.
- **Folder exists, meta file exists, status `kickoff`** → refuse: *"`{NNN-slug}-feature.md` already exists with status `kickoff`. Edit it directly to refine the goal, or proceed to `/specflow:prd {NNN-slug}` to lock the goal and move to PRD."*
- **Folder exists, meta file exists, status beyond `kickoff`** → refuse: *"`{NNN-slug}` is past kickoff (status: `{status}`). Goal changes are a direct edit to `{NNN-slug}-feature.md`; downstream skills pick up the new value on their next read."*

Exit cleanly on either refusal.

### A.3 Scaffold the folder + subfolders

```bash
mkdir -p docs/specflow/features/{NNN-slug}/{design,docs,assets,test/screenshots,debate-log}
```

The five subfolders:

- `design/` — static designs, versioned filenames (`v1-{description}.{ext}`, `v2-{description}.{ext}`).
- `docs/` — feature-specific docs that aren't the PRD / tasks / test plan (e.g. domain reference, supporting notes).
- `assets/` — reference materials (YAMLs, HTMLs, source data) the AI ingests during PRD / task synthesis.
- `test/screenshots/` — Playwright CLI captures.
- `debate-log/` — gate manifests (auto-populated by downstream skills).

### A.4 Write `.gitkeep` in each subfolder

```bash
touch docs/specflow/features/{NNN-slug}/{design,docs,assets,test/screenshots,debate-log}/.gitkeep
```

Empty folders aren't tracked by git; `.gitkeep` ensures the structure survives commits. The folder purposes live ONCE in the meta file's "Folder index" section — no duplicated READMEs.

### A.5 Verify before continuing

- Folder `docs/specflow/features/{NNN-slug}/` exists.
- All five subfolders exist.
- All five `.gitkeep` files exist.

Hand off to Phase B.

---

## Phase B — Goal interview

### B.1 Read context

Read in parallel:
- `docs/specflow/admin/CONTEXT.md` — project-level context.
- `docs/specflow/admin/profiles.json` — user profile registry (for audience question).
- `docs/specflow/admin/environment.json` — stack detection (informs reflection).

The context informs the AI's reflection at Phase C; it does NOT inform the user's answers (that would lead the user). Surface the context only after the user has answered all four questions.

### B.2 Ask the four questions

Ask each question one at a time. Wait for the user's answer before moving to the next. Each answer is one to three sentences max; redirect if the user goes long.

```
1. **Headline goal** — In one sentence, what's the goal of this feature?

2. **Why now** — What's the motivation? What pain or opportunity is driving it?

3. **Who benefits** — Who's the audience? Cite a profile from profiles.json if one fits; describe a new audience if none does.

4. **What does done look like** — What's the observable signal that this feature has shipped successfully?
```

If the user answers all four in one shot ("here's everything"), parse them out and confirm: *"I parsed four answers; let me reflect."* If parsing is ambiguous, ask the specific missing question.

### B.3 Verify before continuing

- All four questions have answers (non-empty, user-provided text).
- Answers are succinct (none exceeds 3 sentences).

Hand off to Phase C.

---

## Phase C — Reflection + confirmation

### C.1 Reflect what was heard

Before composing the reflection, gauge complexity. Read the four answers and decide `mode: light | full`:

- **Light signals.** Verbs like *remove / rename / tweak / swap / hide / update copy*. Acceptance shape *"X is hidden" / "X reads Y" / "the image no longer appears"*. No content yet in `assets/` or `design/`. Goal fits in one paragraph. → `mode: light`.
- **Full signals.** Verbs like *add / build / integrate / support / introduce*. Acceptance shape *"X works with Y under conditions Z"*. Reference content present in `assets/` or `design/`. Goal needs two paragraphs to express. → `mode: full`.
- **Mixed.** When signals split, prefer `full` and flag the ambiguity in the reflection so the user can downgrade.

The mode determines whether downstream skills (`specflow:prd`, `specflow:task`, `grill`) run their full review chain (`full`) or skip the heavy passes (`light`). It is not a hard threshold — the user confirms at C.2 and can override at any phase.

Compose a reflection block in chat:

```
Here's what I heard:

**Goal:** {one-paragraph synthesis of question 1 + 4 — what we're trying to achieve and how we'll know it's done}

**Motivation:** {one-line synthesis of question 2}

**Audience:** {audience from question 3, cited against profiles.json when matched}

**Mode:** {light | full} — {one-line rationale citing the signals above}

**Open questions I'd raise before PRD:**
- {1-3 questions the AI surfaces from gaps in the answers, not yet load-bearing — to be resolved during PRD interview}

**PRD fields this implies:**
- {1-3 fields the PRD interview will need to cover, e.g. "scope of supported file types", "fallback behaviour when offline"}

Confirm, edit, or replace.
```

The reflection is the AI's interpretation. The user owns the goal — they can confirm verbatim, edit any field (including `mode:`), or replace wholesale.

### C.2 Iterate to confirmation

User responds:

- **Confirm** → proceed to Phase D.
- **Edit `{field}`: {new content}** → apply the edit, re-present, loop until confirmed.
- **Replace** → take their version verbatim. If they leave a field blank, ask one targeted question to fill it.

Do not proceed until every field has user-confirmed content. Empty input or unrecognised input re-prompts.

### C.3 Verify before continuing

- Goal is ≤ 2 paragraphs (~150 words max). If longer, surface: *"The goal is meant to be slim — two paragraphs max. Want me to tighten or keep as-is?"* The user decides.
- Open questions list is 0-3 entries.
- PRD fields list is 0-3 entries.

Hand off to Phase D.

---

## Phase D — Write meta file + hand off

### D.1 Compose `{NNN-slug}-feature.md`

Write to `docs/specflow/features/{NNN-slug}/{NNN-slug}-feature.md`:

```markdown
---
slug: {NNN-slug}
status: kickoff
mode: {light | full}
created: {YYYY-MM-DD}
goal_locked: {YYYY-MM-DD}
---

# {Feature title — derived from the slug, title-case}

## Goal

{The confirmed goal — ≤2 paragraphs. What we're trying to achieve and what done looks like.}

**Motivation:** {confirmed one-line motivation from question 2}

**Audience:** {confirmed audience from question 3, cited against profiles.json when matched}

## Open questions raised at kickoff

{0-3 bullets — questions to resolve during the PRD interview. Skip the section header entirely if empty.}

## PRD fields implied at kickoff

{0-3 bullets — fields the PRD interview will need to cover. Skip the section header entirely if empty.}

## Folder index

- `{NNN-slug}-feature.md` — this file (goal + folder index; locked at kickoff)
- `{NNN-slug}-prd.md` — PRD (pending)
- `{NNN-slug}-tasks.md` — tasks (pending)
- `{NNN-slug}-test.md` — test plan (pending)
- `{NNN-slug}-brief.html` — rendered brief (pending)
- `design/` — static designs, versioned (`v1-*`, `v2-*`)
- `docs/` — feature-specific docs (domain reference, supporting notes)
- `assets/` — reference materials (YAMLs, HTMLs, source data)
- `test/screenshots/` — Playwright CLI captures
- `debate-log/` — gate manifests (auto-populated)

## Status

- `kickoff` — meta file written; goal locked. ← *current*
- `prd-pending` — awaiting `specflow:prd`.
- `tasks-pending` — PRD shipped; awaiting `specflow:task`.
- `development` — tasks shipped; PRs landing via `specflow:develop`.
- `test-pending` — development done; awaiting full test pass via `specflow:test`.
- `shipped` — test passed; feature complete.

Each downstream skill bumps `status:` in the frontmatter on its Phase A entry. Goal changes after kickoff are a direct edit to this file's `## Goal` section; downstream skills pick up the new value on their next read.
```

The title (line `# {Feature title}`) derives from the slug — replace `-` with spaces and title-case each word. Example: `notifications-popover` → `Notifications Popover`. The user can edit this freely after the write.

### D.2 Hand off

Surface in chat:

```
specflow:feature complete.

Feature: NNN-{slug}
Meta file: docs/specflow/features/NNN-{slug}/NNN-{slug}-feature.md
Status: kickoff

Next step: /specflow:prd NNN-{slug}
  (the PRD interview will read this file and skip the strategic questions)

Drop reference materials in:
  - design/  for mockups
  - assets/  for YAMLs / HTMLs / source data
  - docs/    for domain reference or supporting notes
```

### D.3 Verify before declaring done

1. `docs/specflow/features/{NNN-slug}/{NNN-slug}-feature.md` exists with valid frontmatter (slug, status=kickoff, created, goal_locked all present).
2. Goal section is ≤2 paragraphs.
3. Folder index lists every standard artefact path.
4. All five subfolders + `.gitkeep` files present.
5. User confirmed the goal before write (Phase C.2 closed on confirmation).

If any verify step fails, surface the failure and refuse to claim success.

---

## Failure modes

- **Folder already exists + meta file present (A.2)** — refuse with one of the two documented sentinel lines (status `kickoff` → edit-directly hint; status beyond → goal-change-is-direct-edit hint).
- **Slug validation fails (A.1)** — re-prompt until kebab-case + no path separators.
- **User answers exceed 3 sentences (B.2)** — redirect: *"Keep it to a sentence or two — the goal is meant to be slim."*
- **User refuses to confirm (C.2)** — loop indefinitely on edit/replace. The skill never auto-defaults.
- **Goal exceeds 2 paragraphs after confirmation (C.3)** — surface the length warning; user decides whether to tighten or proceed.

---

## Anti-patterns (refuse to do)

- **Skip the goal interview.** The goal IS the point of this skill. A meta file without a user-confirmed goal is a failed run.
- **Write a long meta file.** Two paragraphs max for the goal; the folder index is just a list of paths; open-questions / PRD-fields sections cap at 3 bullets each. If the meta file exceeds ~80 lines, something has gone wrong.
- **Auto-populate the open-questions or PRD-fields sections beyond 3 entries.** The lists are signal, not exhaustive coverage; downstream skills do the deep work.
- **Recommend `specflow:prd` will overwrite the goal.** It doesn't. The goal is locked at kickoff. Goal changes are direct edits to the meta file; downstream skills pick up the new value on their next read.
- **Create folder READMEs.** The folder index lives once in the meta file. Per-folder READMEs duplicate without adding signal.
- **Modify any other admin file.** This skill writes to `features/{NNN-slug}/` only — no `admin/` touches, no rules-registry writes, no decision-log entries.
- **Name any AI vendor or tooling in user-facing output.** Per the project's CLAUDE.md attribution rule.

---

## Cross-skill integration

- **`specflow:prd`** — primary downstream consumer. Reads `{NNN-slug}-feature.md` at its Phase A entry; when present, skips slug allocation (A.1), folder creation (A.2 is idempotent), and the goal articulate/confirm/write cycle (A.5/A.6/A.7). Writes the goal verbatim from feature.md into the interview file's Goal section; bumps `status:` to `prd-pending`.
- **`specflow:task`** — consumer of `status:`. Bumps to `tasks-pending` on Phase A entry; bumps to `development` when handing off to `specflow:develop`.
- **`specflow:develop`** — bumps `status:` to `test-pending` on feature-mode completion (every task shipped).
- **`specflow:test`** — bumps `status:` to `shipped` on full-mode pass.
- **`specflow:setup`** — seeds `skills.feature.enabled: true` in `admin/config.json` at first-run.
- **`specflow:scope-change`** — NOT involved in goal changes. Goal changes are direct user edits to the meta file; `scope-change` is reserved for PRD-shape changes.

---

## Reference

- `skills/prd/SKILL.md` — primary downstream consumer; Phase A reads this skill's meta file when present.
- `skills/setup/SKILL.md` — seeds the `feature` skill toggle.
- `templates/orchestrator-pattern.md` — four sequential phases; no sub-agent forking.
- `CORE_PRINCIPLES.md` — the four principles bound to every phase verify step.
