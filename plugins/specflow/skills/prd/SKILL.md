---
name: specflow:prd
description: User-facing entry point for PRD creation. Five-phase orchestrator — A preamble + goal confirmation, B grilling (invokes /grill), C PRD body synthesis, D HTML render (invokes specflow:render), E Gate 2 multi-agent debate manifest. Resumes intelligently if invoked on an in-flight feature.
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
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.html
  - docs/specflow/features/{NNN-slug}/debate-log/prd-gate2/manifest.md
  - docs/specflow/features/{NNN-slug}/debate-log/prd-gate2/findings/
eval: Goal confirmed before grilling; interview signed off; PRD body has Vision (traces to Goal), every requirement traces to a Resolved line AND serves the goal, every AC is binary; HTML renders with working interview link; Gate 2 manifest closes with Orchestrator sign-off entry.
---

# specflow:prd

You are the user-facing entry point for PRD creation. You own the full flow from "I want to build X" to "PRD reviewed and signed off."

This is a **5-phase orchestrator**. You delegate the grilling to `/grill` and the rendering to `specflow:render`; both run as forked sub-skills per the orchestrator pattern (see `templates/orchestrator-pattern.md`). Your parent context never accumulates the sub-skills' raw work.

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
   - **PRD body exists, no HTML or HTML stale** → skip to Phase D.
   - **PRD + HTML exist, no Gate 2 manifest** → skip to Phase E.
   - **Everything complete** → ask the user: *"This feature appears complete (interview signed off, PRD synthesised, HTML rendered, Gate 2 closed). What do you want to do? (1) re-render only, (2) re-run Gate 2, (3) `specflow:scope-change` to revise."*

Tell the user explicitly which phase you're starting at.

---

## Phase A — Preamble + Goal confirmation

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

Distill what you saw into bullet points for the *Codebase context* section. Be concrete: cite file paths, line counts, conventions detected, prior PRD references.

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

Before invoking the render in Phase D, verify:

1. **Vision traces to Goal.** Re-read the interview's Goal section and the PRD's Vision. The Vision should be the prose form of the Goal — same Outcome, same Audience, same Driving value. If they diverge, fix the Vision.
2. **Every requirement has a Trace + Serves-goal pair.** No requirements that don't trace to a Resolved line. No requirements that don't serve at least one goal field.
3. **Every AC verifies a requirement.** No orphan ACs. No requirements without coverage.
4. **No requirement contradicts the goal's Out-of-scope-at-goal-level.** Cross-check.
5. **ACs cross-checked against Phase 1 skill schemas they depend on.** For every AC that names another specflow skill (e.g. `specflow:misc --auto`, `specflow:linear`, `specflow:render`), open that skill's SKILL.md and verify the AC's named fields exist in the documented payload schema. If the AC references fields the schema doesn't include, EITHER edit the AC to use only existing fields OR add an explicit "Schema dependency" note naming the schema gap and the enhancement PRD that must land first. Don't smuggle a Phase 1 schema change into a downstream PRD's AC.

If any check fails, fix the PRD before proceeding.

---

## Phase D — Render

### D.1 Invoke specflow:render

Use the Skill tool:

```
Skill: specflow:render {NNN-slug}
```

It produces `features/NNN-{slug}/NNN-{slug}-prd.html` with a header link to the sibling interview file.

### D.2 Verify render

Read the HTML file's first 50 lines. Verify:
- File exists.
- Header strip includes the feature ID + slug.
- Header includes a link to `./NNN-{slug}-interview.md`.
- No drift banner (if there is one, the markdown is newer than the HTML — re-invoke render).

---

## Phase E — Gate 2 multi-agent debate manifest

### E.1 Set up the debate folder

Use Bash:

```bash
mkdir -p docs/specflow/features/NNN-{slug}/debate-log/prd-gate2/findings/{round-1,round-2,round-3}
mkdir -p docs/specflow/features/NNN-{slug}/debate-log/prd-gate2/raw
touch docs/specflow/features/NNN-{slug}/debate-log/prd-gate2/manifest.md
```

### E.2 Identify reviewers

From `docs/specflow/admin/agents/standard/`, the standing reviewer set is:
- `lifecycle/devils-advocate.md` — always present.
- `principles/simplicity-reviewer.md`
- `principles/surgical-reviewer.md`
- `principles/think-before-coding-reviewer.md`
- `principles/goal-driven-reviewer.md`

Plus, if `admin/environment.json` has `cli.codex.available: true`, include Codex as a sixth reviewer.

### E.3 Round 1 — parallel finding fire

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

### E.4 Round 2 — AI responds

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

### E.5 Round 3 — Reviewers sharpen or accept

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

### E.6 Closer — Orchestrator collates

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

### E.7 Final disposition

If Gate 2 status is **passed**, **passed-with-revisions**, or **passed-with-escalations**: tell the user *"PRD synthesised and Gate 2 review complete. Status: {status}. Manifest at `debate-log/prd-gate2/manifest.md`. Next step: `specflow:task {NNN-slug}` when ready."*

If escalations exist, list them in your response so the user sees them without opening the manifest.

If Gate 2 status is **failed**: tell the user *"PRD failed Gate 2 review. Blocking findings:\n{list}\n\nReview the manifest at `debate-log/prd-gate2/manifest.md` and either revise the PRD or adjust the interview's Resolved lines, then re-run `specflow:prd {NNN-slug}` to resume from Phase C."*

---

## What you MUST NOT do

- **Do not skip Phase A's goal confirmation.** Even if the user provides a detailed overview, articulate the goal and get explicit confirmation. The goal is the precedent everything anchors to.
- **Do not let `/grill` start without a confirmed Goal section.** `/grill` will refuse, but you should not even invoke it if the goal isn't confirmed.
- **Do not synthesise the PRD body until the interview is signed off.** No exceptions.
- **Do not duplicate the interview's content into the PRD body.** PRD references the interview by relative path; the interview is the audit trail.
- **Do not bypass Gate 2.** A PRD that hasn't been through Gate 2 is not a finished PRD; downstream skills (`specflow:task`) should reject it.
- **Do not edit the Goal section.** Only `specflow:scope-change` does that.
- **Do not mention Claude, Anthropic, or any AI tooling** in any user-facing output, the interview file, the PRD body, or the debate manifest. Per the project's CLAUDE.md, this is non-negotiable.

---

## Verify before declaring done

Before returning to the user with "PRD complete":

1. `features/NNN-{slug}/NNN-{slug}-interview.md` exists with confirmed Goal + at least one round + sign-off line.
2. `features/NNN-{slug}/NNN-{slug}-prd.md` exists with all required sections (Vision, Problem, Goals, Non-goals, Users, Requirements, AC, Open questions, See also).
3. Vision traces to Goal; every requirement traces to a Resolved line AND serves a goal field; every AC is binary and verifies a requirement; no requirements contradict Out-of-scope-at-goal-level.
4. `features/NNN-{slug}/NNN-{slug}-prd.html` exists; header link to interview works.
5. `features/NNN-{slug}/debate-log/prd-gate2/manifest.md` exists with closing decision entry signed by the Orchestrator.
6. Gate 2 status is recorded (passed / passed-with-escalations / failed) and surfaced to the user.

If any verify step fails, fix it before returning.

---

## Reference

- `docs/PRD.md` Appendix Q — interview file structure spec.
- `docs/PRD.md` Appendix N — multi-agent debate manifest spec.
- `docs/PRD.md` Appendix P — PRD HTML rendering spec.
- `templates/orchestrator-pattern.md` — fork / file handoff / command substitution conventions.
- `skills/grill/SKILL.md` — the sub-skill invoked in Phase B.
- `skills/render/SKILL.md` — the sub-skill invoked in Phase D.
- `templates/agents/standard/lifecycle/{orchestrator,devils-advocate,verifier}.md` — reviewer prompts for Gate 2.
- `templates/agents/standard/principles/*.md` — principle-reviewer prompts for Gate 2.
