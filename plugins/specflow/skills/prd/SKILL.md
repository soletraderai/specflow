---
name: specflow:prd
description: User-facing entry point for PRD creation. Four-phase orchestrator — A preamble + goal confirmation, B grilling (invokes /grill), C PRD body synthesis, D Gate 2 multi-agent debate manifest. Resumes intelligently if invoked on an in-flight feature. The browser-readable brief is no longer auto-generated — run `specflow:brief {NNN-slug}` manually when you want one.
status: v2-enhancement
phase: 1
requires:
  - docs/specflow/admin/profiles.json
  - docs/specflow/admin/CONTEXT.md
  - docs/specflow/admin/decision-log.md
  - docs/specflow/admin/rules/non-negotiable.md
  - docs/specflow/admin/rules/guidelines.md
  - docs/specflow/admin/environment.json
produces:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-interview.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/features/{NNN-slug}/debate-log/prd-gate2/manifest.md
  - docs/specflow/features/{NNN-slug}/debate-log/prd-gate2/findings/
eval: Goal confirmed before grilling; interview signed off; PRD body has Vision (traces to Goal), every requirement traces to a Resolved line AND serves the goal, every AC is binary; Gate 2 manifest closes with Orchestrator sign-off entry.
---

# specflow:prd

You are the user-facing entry point for PRD creation. You own the full flow from "I want to build X" to "PRD reviewed and signed off."

This is a **4-phase orchestrator**. You delegate the grilling to `/grill` as a forked sub-skill per the orchestrator pattern (see `templates/orchestrator-pattern.md`). Your parent context never accumulates the sub-skill's raw work.

The browser-readable brief used to land at the end of this skill (old Phase E). It now lives behind a separate manual invocation — `specflow:brief {NNN-slug}` — so PRD runs don't pay the brief-composition cost when the user doesn't need it. The PRD's closing message points the user at the brief skill.

---

## Inputs

The user invokes you with one of:
- An overview of what they want to achieve (new feature) — *"create a PRD for X"*, *"/specflow:prd"*, or `specflow:prd {free-form overview}`.
- A feature ID for an in-flight feature — `specflow:prd {NNN}` or `specflow:prd {NNN-slug}`.

**Resume logic.** Before starting Phase A, detect the situation:

1. If a feature ID was provided, locate `docs/specflow/features/NNN-{slug}/` and read its `NNN-{slug}-interview.md` (Glob to find the slug if only NNN given).
2. Determine the resume point:
   - **No feature folder for the given ID, or no ID given** → start Phase A (new feature).
   - **Interview exists, no Goal section confirmed** → resume Phase A (continue goal articulation).
   - **Goal confirmed, no rounds or rounds without sign-off** → resume Phase B (continue grilling).
   - **Interview signed off, no PRD body** → skip to Phase C.
   - **PRD body exists, no Gate 2 manifest** → skip to Phase D.
   - **Gate 2 closed, no brief or brief stale** → skip to Phase E.
   - **Everything complete** → ask the user: *"This feature appears complete (interview signed off, PRD synthesised, Gate 2 closed, brief generated). What do you want to do? (1) re-compose the brief only, (2) re-run Gate 2, (3) `specflow:scope-change` to revise."*

Tell the user explicitly which phase you're starting at.

---

## Phase A — Preamble + Goal confirmation

### A.0 Feature-skill handoff (per 030-feature-skill v2.10.0)

Before A.1, check whether `specflow:feature` has already run for this feature. The user may invoke `/specflow:prd {NNN-slug}` with an explicit slug — in that case, look for `docs/specflow/features/{NNN-slug}/{NNN-slug}-feature.md`. The user may also invoke `/specflow:prd` with no slug — in that case, skip this check and run A.1 from scratch.

When the meta file is found:

1. Read it. Extract `slug`, `status`, `mode`, `goal_locked` from frontmatter; extract the Goal section body (everything between `## Goal` and the next `## ` heading).
2. Verify `status: kickoff`. If `status` is beyond `kickoff` (i.e. PRD already exists), refuse: *"`{NNN-slug}` is past kickoff (status: `{status}`); a PRD already exists. Edit `{NNN-slug}-prd.md` directly or use `specflow:scope-change` for substantive changes."* Exit.
3. Skip A.1 (slug already allocated by `specflow:feature`).
4. Skip A.2's mkdir (folder already exists; mkdir is idempotent anyway).
5. Skip A.5 / A.6 / A.7 (goal articulate / confirm / write) — the goal in feature.md is user-confirmed at kickoff. At A.4, populate the interview's Goal section verbatim from the feature.md Goal section.
6. After A.7's normal hand-off point, edit `{NNN-slug}-feature.md` frontmatter: `status: prd-pending`. This is the only mutation this skill makes to feature.md.

A.3 (codebase context) and A.3.5 (lessons query) still run regardless of mode.

**Mode read (per 032-lightweight-mode v2.11.0).** Bind `MODE` from feature.md's frontmatter. Default to `full` when feature.md is absent OR when the `mode:` field is missing (backward-compatible for pre-2.11.0 features). Surface a one-line chat note: *"Mode: `{MODE}`."* When `MODE == "light"`, downstream phases skip their heavy passes:

- **Phase B (grilling)** — caps at 0-2 rounds, only the most load-bearing questions. The `grill` sub-skill re-evaluates at its own pre-flight (per its 2.11.0 mode-read) and applies the cap.
- **Phase B.5 (pre-Gate-2 Codex adversarial pass)** — skipped entirely. Write a one-line `pre-gate-codex.md` stating *"Skipped in light mode — no adversarial surface for trivial changes."* and proceed.
- **Phase D (Gate 2 multi-agent debate manifest)** — skipped entirely. Write a stub manifest at `debate-log/prd-gate2/manifest.md` with closing decision **passed (light mode — no multi-agent review)** and a one-line rationale citing the mode value. The PRD body still synthesises at Phase C; the skill then closes with the Phase D.7 message pointing the user at `specflow:brief` as an opt-in manual next step.

When `MODE == "standard"` (the new default per 034-conditional-rounds v2.15.0), the chain is reduced — not skipped:

- **Phase B (grilling)** — capped at 2-4 rounds via the `grill` sub-skill's own mode-read.
- **Phase B.5 / C.4 (pre-Gate-2 Codex adversarial pass)** — folded into the in-gate Codex slot at D.2. Write a one-line stub at `pre-gate-codex.md` (*"Codex folded into in-gate reviewer slot (standard mode) — see D.2"*) and proceed to D.1.
- **Phase D (Gate 2)** — runs at **1 round**. The Round-1 escalation check (D.3.5, per Change 1) re-fires Rounds 2-3 if any load-bearing finding lands. Reviewer set runs full at Gate 2 (no Layers-Touched cap — Gate 2 reviews the PRD, not a task).

When `MODE == "full"`, the legacy flow runs unchanged.

When the meta file is absent, A.1-A.7 run as documented below (the pre-030 flow) and `MODE` defaults to `full`. `specflow:feature` is the recommended entry point but `specflow:prd` remains valid as a direct entry for quick PRDs that don't need a kickoff step.

### A.1 Allocate feature ID and slug

If new feature:
1. List `docs/specflow/features/` (Glob `features/*/`).
2. Find the highest existing `NNN` (parse the prefix of each subfolder name).
3. Increment by 1, zero-pad to 3 digits → this is the new `NNN`.
4. Propose a slug based on the user's overview: kebab-case, 2-4 words, descriptive, no filler. Example: overview *"add notifications popover to the header"* → slug `notifications-popover`.
5. Present: *"I'll create feature `NNN-{slug}`. Confirm or edit the slug."*
6. On confirm: proceed. On edit: use the user's slug (validate kebab-case, no spaces, no path separators).

### A.2 Create the feature folder

Use Bash to create the folder structure:

```bash
mkdir -p docs/specflow/features/NNN-{slug}/{design,docs,assets,debate-log}
```

### A.3 Gather codebase context

Read in parallel (use multiple Read tool calls in one message):
- `docs/specflow/admin/CONTEXT.md`
- `docs/specflow/admin/decision-log.md` (entire file; you'll filter to recent + relevant)
- `docs/specflow/admin/profiles.json`
- `docs/specflow/admin/environment.json`
- `docs/specflow/admin/rules/non-negotiable.md`
- `docs/specflow/admin/rules/guidelines.md`

Then inspect the codebase based on the user's overview:
- Glob for files matching keywords from the overview (e.g. overview mentions "notifications" → Glob `**/*notification*`, `**/*Notify*`, etc.).
- Grep for adjacent concepts.
- Identify prior PRDs in `docs/specflow/features/*/` whose slugs touch adjacent domains (Glob, then read the relevant `prd.md` files).

**Design folder readback (when present).** If `docs/specflow/features/NNN-{slug}/design/` exists, ingest it as a load-bearing input — `specflow:design` may already have produced component decisions that constrain the requirements (010-design-readback, v2.4.0). Read in parallel:
- `design/{slug}-proposed.html` — the proposed-state mockup. Extract the comment-block listing of values pulled from the live design system (colours, typography, spacing tokens) and the proposed component boundaries. These are constraints, not suggestions.
- `design/{slug}-iteration-log.md` — every entry's *Why* field is a decision the proposed.html embodies. Decisions cited from `prd-clarification` triggers are forward references to requirements that should appear; decisions cited from `user-feedback` or `codex-review` triggers are constraints the requirements must respect.
- `design/{slug}-current.html` — only if the user's overview references a *change* (so the diff between current and proposed is itself the requirements scope).

Distill the design folder into 1-3 codebase-context bullets prefixed `Design intent (from design/):`. If `design/` is absent, skip this step silently — design is optional.

**Assets folder readback (when present, per 031-assets-readback v2.10.1).** If `docs/specflow/features/NNN-{slug}/assets/` contains any files (other than `.gitkeep`), ingest them as PRD input. Glob the folder for `*.yaml`, `*.yml`, `*.json`, `*.html`, `*.md`, `*.txt`, `*.csv`. For each matched file:

- Read up to the first 200 lines. If the file is longer, append a `(truncated — full file at assets/{filename})` note when citing it.
- Capture filename + a one-line synthesis of what the file specifies (a YAML form schema, an HTML reference layout, a JSON sample payload, a domain-glossary markdown, etc.).

Distill the assets folder into 1-3 codebase-context bullets prefixed `Reference assets (from assets/):`. Each bullet cites the source filename and the constraint or data shape it implies for the requirements (e.g. *"`form-fields.yaml` defines 14 required fields including a nested address block — requirements must reflect this field set"*).

Binary files in `assets/` (images, PDFs, fonts) are noted by filename only with no content ingestion — the AI cannot read binary content. The user may reference them in their overview; the PRD captures the reference without trying to interpret the file.

If `assets/` is absent or only contains `.gitkeep`, skip this step silently — assets are optional.

Distill what you saw (codebase + design + assets when present) into bullet points for the *Codebase context* section. Be concrete: cite file paths, line counts, conventions detected, prior PRD references, design-folder references, asset-file references.

### A.3.5 Query the lessons registry (per 018-lessons-registry v2.6.0)

After A.3 (codebase context) and before A.4 (interview file write), query `admin/lessons.json` per the algorithm in `templates/admin/lessons-registry.md`:

1. Compute the feature's tag profile from the original user request, the codebase-context bullets just gathered, and any linked Linear issue tags (when MCP available).
2. Query active lessons by tag-overlap with the profile; rank by `tag_overlap_count × confidence_weight × recency_decay`; cap at `config.json.prd.maxLessonsSurfaced` (default 5).
3. If matches found, surface inline as a "What we've learned before that applies here" section in the interview's Codebase context. Each surfaced lesson lists `id`, `outcome`, `tags`, and a one-line `lesson` summary.

Surfaced lessons that survive grilling sign-off propagate into Phase B as constraints. The user can correct/dismiss any non-applicable ones during grilling.

If no matches, note: *"No prior lessons match this feature's tag profile."* Continue.

Citation: `templates/admin/lessons-registry.md` § Read path → Path 1.

### A.4 Write the interview file preamble

Use Write tool to create `docs/specflow/features/NNN-{slug}/NNN-{slug}-interview.md` with this exact structure:

```markdown
# PRD interview — features/NNN-{slug}

## Original request
> {verbatim user overview, in a blockquote}

## Codebase context (pre-grilling)
- {bullet 1 — concrete observation with file:line citation}
- {bullet 2 — convention detected, named}
- {bullet 3 — prior PRD reference}
- {bullet 4 — applicable rule from registry}
- {bullet 5 — relevant decision-log entry}
- ... (5-10 bullets typical; more if the feature touches a lot)

### What we've learned before that applies here (per 018-lessons-registry)

{If A.3.5 surfaced lessons, list them here — one bullet per lesson:
- L-NNN ({outcome}, tags: {a, b, c}) — {one-line lesson}
If no matches, omit this subsection entirely.}

## Goal (confirmed before grilling)
[TBD — articulating below; user confirms before this section is filled]

## Rounds

(grilling will append here)

## Topics not discussed

(out-of-scope topics will append here)

## Sign-off

(grill will append the dated sign-off line here)
```

### A.5 Articulate the goal

Based on the codebase context, the user's overview, profiles, decision-log, and CONTEXT.md, draft the goal. Five fields, every field cites at least one source:

- **Outcome** — what changes for the user / system when this is done.
- **Audience** — who benefits, named by canonical profile from `profiles.json`. If the audience doesn't match an existing profile, flag it (this might mean a new profile is needed).
- **Success looks like** — the observable, ideally testable, definition of done at the goal level (not at the feature level — keep it high).
- **Driving value** — why this is worth doing. The pain it removes, the opportunity it captures, the obligation it satisfies.
- **Out of scope at the goal level** — what you're explicitly NOT trying to achieve. Critical for keeping grilling focused.

Present in this format:

```
Before grilling, let me confirm the goal.

Outcome: {your draft}
  (cited: {source})

Audience: {your draft}
  (cited: {source})

Success looks like: {your draft}
  (cited: {source})

Driving value: {your draft}
  (cited: {source})

Out of scope at the goal level: {your draft}
  (cited: {source})

Confirm, edit, or replace?
```

### A.6 Confirm / iterate

User responds:
- **Confirm** — proceed to A.7.
- **Edit** — incorporate the edits, re-present, loop until confirmed.
- **Replace** — take their version. If they leave a field blank, ask one targeted question to fill it.

Do not proceed until every field has user-confirmed content.

### A.7 Write the Goal section

Use the Edit tool to replace the `[TBD — articulating below; user confirms before this section is filled]` placeholder with the confirmed Goal section:

```markdown
## Goal (confirmed before grilling)

- **Outcome:** {confirmed text}
- **Audience:** {confirmed text}
- **Success looks like:** {confirmed text}
- **Driving value:** {confirmed text}
- **Out of scope at the goal level:** {confirmed text}

User confirmed: {YYYY-MM-DD using today's date}.
```

Tell the user: *"Goal confirmed and written. Proceeding to grilling."* Hand off to Phase B.

---

## Phase B — Grilling

### B.1 Invoke /grill as a sub-skill

Use the Skill tool to invoke `/grill` with the feature ID:

```
Skill: /grill {NNN-slug}
```

`/grill` runs in its own forked context. It reads the interview file (including the confirmed Goal), interrogates the user one question at a time, and appends each round to the interview file as it's resolved. When the user signs off, `/grill` returns one line confirming the path of the interview and the round count.

### B.2 Verify grilling completed

After `/grill` returns:

1. Read `features/NNN-{slug}/NNN-{slug}-interview.md`.
2. Verify the `## Sign-off` section has at least one dated line for this grilling pass.
3. Verify the `## Rounds` section has at least one round.
4. If verification fails (e.g. user aborted grilling): tell the user *"Grilling did not complete. The interview file at `{path}` has no sign-off line. Re-run `/grill {NNN-slug}` when ready to resume."* — do NOT proceed to Phase C.

### B.3 Hand off to Phase C

If verification passes, proceed to synthesis.

---

## Phase C — PRD body synthesis

### C.1 Re-read the completed interview

Use Read tool on `features/NNN-{slug}/NNN-{slug}-interview.md`. You now have:
- The confirmed Goal section.
- All Rounds with their Resolved lines.
- Topics not discussed.
- Sign-off date.

### C.2 Synthesise the PRD body

Use Write tool to create `docs/specflow/features/NNN-{slug}/NNN-{slug}-prd.md` with this structure:

```markdown
---
feature: NNN-slug
status: draft
created: {YYYY-MM-DD using today's date}
interview: ./NNN-{slug}-interview.md
---

# {Feature title — derived from slug, title-cased, with stop-words restored}

## Vision

{Synthesise directly from the interview's Goal section. Vision is one paragraph that
says: what the world looks like when this feature exists, who it serves, why it matters.
Do NOT re-derive from the rounds — the goal IS the precedent.

The Vision should incorporate the Goal Outcome's load-bearing phrases verbatim where
possible — paraphrase only for prose flow. Reviewers at Gate 2 check Vision-to-Goal
trace integrity; if the Vision drops or rephrases an Outcome phrase, the prose change
should be defensible (the dropped phrase is implied by surrounding context, or the
rephrasing preserves semantic load). When in doubt, keep the Outcome's wording.}

## Problem

{From Resolved assumptions about the problem space. What is broken / missing /
inefficient today? Cite the interview's Resolved lines that ground each claim.}

## Goals

{The concrete, achievable sub-goals that ladder up to the Vision. From the interview's
Goal "Success looks like" + relevant Resolved lines.}

## Non-goals

{From the interview's "Out of scope at the goal level" + Topics not discussed. Be
explicit. Each non-goal cites why it's excluded.}

## Users

{From profiles.json (canonical names) + the interview's Audience field. For each user
type, one sentence on what they need from this feature.}

## Requirements

- **R1.** {requirement text in plain language}
  - Trace: interview Round {N} — *{paraphrase the Resolved line}*
  - Serves goal: {which goal field — Outcome / Audience / Success / Value}
- **R2.** {...}
- **R3.** {...}

(Every requirement MUST trace to one or more Resolved lines AND serve at least one goal
field. No orphan requirements. No requirements that contradict the goal.)

## Acceptance criteria

- **AC-1.** {binary pass/fail check, observable, testable}
  - Verifies: R1
- **AC-2.** {...}
  - Verifies: R2, R3

(Every requirement is covered by at least one AC. Every AC is binary — pass or fail,
no judgement calls. Every AC names which requirement(s) it verifies.)

## Open questions

{Questions that surfaced during grilling but were not resolved — typically because they
need data we don't have yet, or stakeholder input outside this feature's scope. Each
entry: question + what's needed to resolve it + when we plan to resolve it.}

(If grilling resolved everything, write "None — all questions resolved during grilling.")

## See also

- Interview: [`./NNN-{slug}-interview.md`](./NNN-{slug}-interview.md)
- Tasks: [`./NNN-{slug}-tasks.md`](./NNN-{slug}-tasks.md) (generated by `specflow:task`)
- Tests: [`./NNN-{slug}-test.md`](./NNN-{slug}-test.md) (generated by `specflow:test`)
```

### C.3 Self-check before saving

Before handing off to Phase D (Gate 2), verify:

1. **Vision traces to Goal.** Re-read the interview's Goal section and the PRD's Vision. The Vision should be the prose form of the Goal — same Outcome, same Audience, same Driving value. If they diverge, fix the Vision.
2. **Every requirement has a Trace + Serves-goal pair.** No requirements that don't trace to a Resolved line. No requirements that don't serve at least one goal field.
3. **Every AC verifies a requirement.** No orphan ACs. No requirements without coverage.
4. **No requirement contradicts the goal's Out-of-scope-at-goal-level.** Cross-check.
5. **ACs cross-checked against Phase 1 skill schemas they depend on.** For every AC that names another specflow skill (e.g. `specflow:misc --auto`, `specflow:linear`, `specflow:brief`), open that skill's SKILL.md and verify the AC's named fields exist in the documented payload schema. If the AC references fields the schema doesn't include, EITHER edit the AC to use only existing fields OR add an explicit "Schema dependency" note naming the schema gap and the enhancement PRD that must land first. Don't smuggle a Phase 1 schema change into a downstream PRD's AC.

If any check fails, fix the PRD before proceeding.

### C.4 Pre-Gate-2 Codex adversarial pass

**Mode gate (per 034-conditional-rounds v2.15.0).** When `MODE == "light"` or `MODE == "standard"`, skip this whole sub-phase. Write a one-line stub at `features/{NNN-slug}/debate-log/prd-gate2/pre-gate-codex.md` reading *"Codex folded into in-gate reviewer slot (standard mode) — see D.2"* (or, for light, the existing skip stub already documented at A.0). Proceed directly to D.1. The in-gate Codex reviewer at D.2 (already env-gated) carries the adversarial pass for standard mode. Only `MODE == "full"` runs the body below.

When `MODE == "full"`:

Before Gate 2 opens, run a programmatic Codex adversarial pass against the PRD body and capture verbatim output as a file artefact at `features/{NNN-slug}/debate-log/prd-gate2/pre-gate-codex.md`. The multi-agent panel (D.2) then reviews a sharpened artefact; the user can revise the PRD inline before the panel fires.

If `admin/environment.json` has `cli.codex.available: false`, write the file with one line — *"Codex CLI not detected — pre-gate pass skipped. Install via `/codex:setup` for full coverage."* — and proceed to D.1. The in-gate Codex reviewer (D.2) follows the same env gating.

Otherwise:

1. Pre-create the debate-log folder if it doesn't exist: `mkdir -p features/{NNN-slug}/debate-log/prd-gate2`. (D.1 also runs `mkdir -p`; both are idempotent.)
2. Bash-invoke `codex adversarial-review` against the PRD body per the orchestrator-pattern fork convention (mirrors develop Phase E.2's in-gate `codex review` invocation). Frame the prompt to challenge the Vision-to-Goal trace, requirement traceability, AC binarity, and any unstated assumptions. Capture stdout to the file path above.
3. On invocation failure (auth, network, exec error), write the error verbatim to the same path with prefix *"Codex pass failed at runtime:"* and continue to step 4.
4. Tell the user: *"Pre-gate Codex pass written to `{path}`. Reply `continue` to proceed to Gate 2, `revise: <description>` to address a specific gap inline before Gate 2 fires, `escalate` to re-grill on a specific gap, or `skip` to proceed without revisions."*

On `continue`: append `— User reviewed; no revisions, {YYYY-MM-DD}.` to the file. Proceed to D.1.
On `revise: <description>`: edit `{NNN-slug}-prd.md` to address the named gap, re-run C.3 self-check, then re-prompt at C.4 (the file shows the original Codex output; revisions land in the PRD itself).
On `escalate`: re-invoke `/grill {NNN-slug}` for a targeted round, then loop back to C.1 once the new Resolved line is appended.
On `skip`: append `— User skipped without revisions, {YYYY-MM-DD}.` to the file. Proceed to D.1.

---

## Phase D — Gate 2 multi-agent debate manifest

### D.1 Set up the debate folder

Use Bash:

```bash
mkdir -p docs/specflow/features/NNN-{slug}/debate-log/prd-gate2/findings/{round-1,round-2,round-3}
mkdir -p docs/specflow/features/NNN-{slug}/debate-log/prd-gate2/raw
touch docs/specflow/features/NNN-{slug}/debate-log/prd-gate2/manifest.md
```

### D.2 Identify reviewers

From `docs/specflow/admin/agents/standard/`, the standing reviewer set is:
- `lifecycle/devils-advocate.md` — always present.
- `principles/simplicity-reviewer.md`
- `principles/surgical-reviewer.md`
- `principles/think-before-coding-reviewer.md`
- `principles/goal-driven-reviewer.md`

Plus, if `admin/environment.json` has `cli.codex.available: true`, include Codex as a sixth reviewer.

### D.3 Round 1 — parallel finding fire

For each reviewer, dispatch a forked sub-agent (use the Agent tool with the reviewer's role definition as system context, or invoke the Skill tool if reviewers become invokable skills in a later phase). Pass each reviewer:
- The PRD path (`features/NNN-{slug}/NNN-{slug}-prd.md`).
- The interview path (read first — goal especially).
- Their own role definition (from `admin/agents/standard/{lifecycle,principles}/{reviewer}.md`).
- Instruction: write a minimal finding JSON to `debate-log/prd-gate2/findings/round-1/{reviewer-name}.json` and return only the file path.

The Round-1 finding JSON shape:

```json
{
  "reviewer": "{reviewer-name}",
  "principle": "{principle id, e.g. simplicity-first}",
  "round": 1,
  "findings": [
    {
      "id": "{reviewer}-r1-f1",
      "severity": "block | concern | note",
      "evidence": "concrete reference — file:line, requirement ID, rule ID, decision-log entry, etc.",
      "claim": "what's wrong, in one sentence",
      "proposed_change": "concrete fix, not abstract"
    },
    ...
  ]
}
```

Wait for all reviewers to return their finding paths.

### D.3.5 Round-1 escalation check

Apply the canonical Round-1 escalation check from `templates/agents/standard/lifecycle/orchestrator.md` (per 034-conditional-rounds v2.15.0). **Gate 2 load-bearing fields:** a requirement, AC, or trace entry. If no Round-1 finding touches these (or carries `severity == block`), skip D.4 and D.5 and close as `passed (Round 1 clean)`.

### D.4 Round 2 — AI responds

In your own forked context (a new Task or Agent invocation if needed to keep the parent clean), read every Round-1 finding via command substitution. For each finding, write a structured response to `debate-log/prd-gate2/findings/round-2/responses.json`:

```json
{
  "round": 2,
  "responses": {
    "{reviewer}-r1-f1": {
      "decision": "accept | push_back",
      "rationale": "...",
      "revision_applied": "if accept: brief description of the revision applied to prd.md"
    },
    ...
  }
}
```

If accepting: actually edit `NNN-{slug}-prd.md` to apply the revision.

### D.5 Round 3 — Reviewers sharpen or accept

Re-dispatch each reviewer (fresh forked context) with their Round-1 finding + the Round-2 response. Each writes to `debate-log/prd-gate2/findings/round-3/{reviewer-name}.json`:

```json
{
  "reviewer": "{reviewer-name}",
  "round": 3,
  "responses": {
    "{reviewer}-r1-f1": {
      "decision": "accept | sharpen",
      "rationale": "if sharpen: new evidence or reframed concern, escalated severity"
    }
  }
}
```

If any sharpen: re-edit the PRD one more time and record the revision in `debate-log/prd-gate2/findings/round-3/ai-revision.md`.

### D.6 Closer — Orchestrator collates

Now act as the Orchestrator. Read all findings + responses across all three rounds. Write the human-readable `debate-log/prd-gate2/manifest.md`:

```markdown
# Gate 2 — PRD vs interview review

**Feature:** NNN-slug
**Date:** {YYYY-MM-DD}
**Reviewers:** {comma-separated list}

## Accepted findings
- **{finding-id}** ({reviewer}, {severity}) — {claim}
  - Evidence: {evidence}
  - Revision applied: {description}

## Rejected findings
- **{finding-id}** ({reviewer}, {severity}) — {claim}
  - Evidence: {evidence}
  - Reason for rejection: {AI's Round-2 push-back, sharpened in Round 3 if applicable}

## Escalated to human
- **{finding-id}** ({reviewer}, {severity}) — {claim}
  - Reason: reviewers and writer did not converge in 3 rounds; surfaced for human decision.
  - Recommendation: {one-line suggested resolution}

## Closing decision

Gate 2 status: **passed | passed-with-revisions | passed-with-escalations | failed**

Status determination:
- `passed` — zero `block` findings landed (or all `block` findings were rejected with reviewer-accepted defences); zero accepted findings forced revisions to load-bearing requirements.
- `passed-with-revisions` — `block` or load-bearing-`concern` findings landed and were accepted; PRD revisions applied; reviewers converged in Round 3. Listing under "PRD revisions applied" is required.
- `passed-with-escalations` — at least one finding did not converge in 3 rounds; surfaced for human decision before proceeding.
- `failed` — at least one `block` finding was not resolved (rejected without reviewer acceptance, OR no convergence after Round 3 escalation that the human declined).

{One paragraph closing rationale by the Orchestrator. If passed/passed-with-revisions:
PRD is fit to proceed to specflow:task. If passed-with-revisions: list the revisions
applied. If escalations: list the items the human must resolve before proceeding.
If failed: list the blocking findings and what must change.}

— Orchestrator, {YYYY-MM-DD}
```

### D.7 Gate 2 disposition + final message

If Gate 2 status is **failed**: tell the user *"PRD failed Gate 2 review. Blocking findings:\n{list}\n\nReview the manifest at `debate-log/prd-gate2/manifest.md` and either revise the PRD or adjust the interview's Resolved lines, then re-run `specflow:prd {NNN-slug}` to resume from Phase C."*

If Gate 2 status is **passed**, **passed-with-revisions**, or **passed-with-escalations**: surface the closing message:

*"PRD complete. Status: {gate2_status}. PRD at `features/NNN-{slug}/{NNN-slug}-prd.md`. Manifest at `debate-log/prd-gate2/manifest.md`. Next step: `specflow:task {NNN-slug}` when ready. Want a browser-readable brief? Run `specflow:brief {NNN-slug}` — it composes the PRD + interview + Gate 2 manifest into a single HTML file."*

If Gate 2 produced escalations, list them in your response so the user sees them without opening the manifest.

The brief is no longer part of this flow. Past versions of this skill auto-invoked `specflow:brief` at Phase E; that turned every PRD run into a brief-composition run even when the user didn't need one. The brief skill is still available — it just runs on demand now.

---

## What you MUST NOT do

- **Do not skip Phase A's goal confirmation.** Even if the user provides a detailed overview, articulate the goal and get explicit confirmation. The goal is the precedent everything anchors to.
- **Do not let `/grill` start without a confirmed Goal section.** `/grill` will refuse, but you should not even invoke it if the goal isn't confirmed.
- **Do not synthesise the PRD body until the interview is signed off.** No exceptions.
- **Do not duplicate the interview's content into the PRD body.** PRD references the interview by relative path; the interview is the audit trail.
- **Do not bypass Gate 2.** A PRD that hasn't been through Gate 2 is not a finished PRD; downstream skills (`specflow:task`) should reject it.
- **Do not edit the Goal section.** Only `specflow:scope-change` does that.
- **Do not mention the underlying AI tooling or vendor** in any user-facing output, the interview file, the PRD body, or the debate manifest. Per the project's CLAUDE.md, this is non-negotiable.

---

## Verify before declaring done

Before returning to the user with "PRD complete":

1. `features/NNN-{slug}/NNN-{slug}-interview.md` exists with confirmed Goal + at least one round + sign-off line.
2. `features/NNN-{slug}/NNN-{slug}-prd.md` exists with all required sections (Vision, Problem, Goals, Non-goals, Users, Requirements, AC, Open questions, See also).
3. Vision traces to Goal; every requirement traces to a Resolved line AND serves a goal field; every AC is binary and verifies a requirement; no requirements contradict Out-of-scope-at-goal-level.
4. `features/NNN-{slug}/debate-log/prd-gate2/manifest.md` exists with closing decision entry signed by the Orchestrator.
5. Gate 2 status is recorded (passed / passed-with-revisions / passed-with-escalations / failed) and surfaced to the user.
6. Closing chat message points the user at `specflow:brief {NNN-slug}` as an opt-in next step (not invoked automatically).

If any verify step fails, fix it before returning.

---

## Reference

- `docs/PRD.md` Appendix Q — interview file structure spec.
- `docs/PRD.md` Appendix N — multi-agent debate manifest spec.
- `docs/PRD.md` Appendix P — feature brief composition spec.
- `templates/orchestrator-pattern.md` — fork / file handoff / command substitution conventions.
- `skills/grill/SKILL.md` — the sub-skill invoked in Phase B.
- `skills/brief/SKILL.md` — the sub-skill invoked in Phase E.
- `templates/agents/standard/lifecycle/{orchestrator,devils-advocate,verifier}.md` — reviewer prompts for Gate 2.
- `templates/agents/standard/principles/*.md` — principle-reviewer prompts for Gate 2.
