---
name: grill
description: Interrogate the user one question at a time, recommending an answer per question with reasoning, re-evaluating what to ask next after each answer. Sub-skill of specflow:prd Phase B; can also be invoked directly to extend an existing interview file. Gate 1 of the adversarial review chain.
status: v2-new
phase: 1
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-interview.md
  - docs/specflow/admin/CONTEXT.md
  - docs/specflow/admin/decision-log.md
  - docs/specflow/admin/profiles.json
  - docs/specflow/admin/rules/non-negotiable.md
  - docs/specflow/admin/rules/guidelines.md
produces:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-interview.md
eval: every round has Q + AI's recommended answer with cited reasoning + user's answer + non-empty Resolved line; sign-off line dated and present; pre-flight passed (Goal section confirmed before any rounds fire).
---

# grill

You interrogate the user one question at a time about a feature they want to build. For every question you propose your *recommended answer* with citation-backed reasoning grounded in the project's prior context, then re-evaluate what to ask next based on the user's answer.

This is **Gate 1** of the adversarial review chain. Your output is rounds appended to `features/{NNN-slug}/{NNN-slug}-interview.md`.

---

## Pre-flight check (mandatory — do this first)

You will be invoked in one of two ways:
- **As a sub-skill of `specflow:prd` Phase B.** The parent passes `{NNN-slug}` and the interview file already has a confirmed Goal section.
- **Manually:** `/grill {feature-id}` — locate the interview file directly.

Before asking any question:

1. Resolve the feature folder. If the user invoked you with `{feature-id}` as `NNN` or `NNN-slug` or just the slug, find the matching `docs/specflow/features/NNN-{slug}/` (use Glob if needed).
2. Read the interview file at `features/{NNN-slug}/{NNN-slug}-interview.md`. Verify:
   - File exists.
   - `## Goal (confirmed before grilling)` heading exists.
   - A `User confirmed: {date}` line appears in that section.
3. **If the Goal section is missing or unconfirmed: STOP.** Tell the user, in plain language: *"The interview file at `{path}` does not have a confirmed Goal section. Grilling cannot start until the goal is confirmed. Run `specflow:prd` to establish the goal, or `specflow:scope-change` if the goal needs revision."* Return without writing anything.
4. If confirmed: load these into your working context (Read tool):
   - The interview file (full).
   - `docs/specflow/admin/CONTEXT.md`.
   - `docs/specflow/admin/decision-log.md` — recent entries especially.
   - `docs/specflow/admin/profiles.json`.
   - `docs/specflow/admin/rules/non-negotiable.md` and `guidelines.md`.
   - Any prior PRDs in `features/*/` that touch adjacent surfaces (Glob the feature directory; if a prior feature's slug overlaps with this one's domain, read its `prd.md`).
5. Note the resolved-assumptions from any rounds already in the interview's `## Rounds` section. If rounds exist, you're extending an existing interview — do not repeat resolved questions.
6. **Mode read (per 032-lightweight-mode v2.11.0).** Read `features/{NNN-slug}/{NNN-slug}-feature.md` if present. Extract `mode:` from frontmatter. Bind `MODE` for this run. Default to `full` when feature.md is absent or the field is missing.

   When `MODE == "light"`, apply a **round cap of 0-2**: ask only the genuinely load-bearing questions (per step 1 criteria below). If no load-bearing question exists, skip grilling entirely and write a one-line round to the interview: *"Round 1 — Light mode: no load-bearing questions surfaced. Skipping to sign-off."* then proceed to sign-off. Surface a one-line chat note before the first question (or skip): *"Mode: `light` — capping grilling at 0-2 rounds."*

   When `MODE == "standard"` (the new default per 034-conditional-rounds v2.15.0), apply a **round cap of 2-4**: ask the load-bearing questions but stop once they're resolved rather than continuing past sign-off triggers. Surface: *"Mode: `standard` — capping grilling at 2-4 rounds."*

   When `MODE == "full"`, the legacy uncapped flow runs.

   The user can override at any round by typing *"switch to full"* or *"switch to light"* — update `mode:` in feature.md, surface the change in the round's Resolved line, continue.

---

## The grilling loop

Repeat steps 1–7 until the user signs off (or, in light mode, until the round cap is reached).

### Step 1 — Decide what to ask next

Based on the confirmed goal, all previously resolved assumptions in this interview, the codebase context, decision-log precedents, profiles, and rules:

Pick the **single most load-bearing unresolved question**. Criteria for "load-bearing":
- The answer would change a top-level requirement, AC, or constraint in the eventual PRD.
- The answer affects which audience profile this serves.
- The answer reveals a hidden assumption.
- The answer surfaces a tradeoff between two viable approaches.

Skip questions that are:
- Already resolved by a prior round in this interview.
- Out of scope per the goal's *Out-of-scope-at-goal-level* field.
- Pre-canned without grounding in the goal or codebase context.

### Step 2 — Form the recommended answer

For your chosen question, draft the answer you'd propose **with reasoning that cites at least one source**. Acceptable sources:
- Goal field — *"serves the goal's Audience: end users, not admins"*
- `CONTEXT.md` — *"per CONTEXT.md, this project uses the job-queue pattern in `src/jobs/`"*
- `decision-log.md` — *"decision-log 2026-02 rejected marketing channels"*
- A prior feature's PRD — *"features/003-claim-submission established this pattern"*
- A rule from `admin/rules/` — *"`NEVER_BYPASS_AUTH` applies here"*
- A profile — *"matches the Power User profile's constraint about resisting UI churn"*

Keep the reasoning tight — one or two cited sources, not a paragraph.

### Step 3 — Ask the user

Present in this exact format (no markdown formatting beyond what's shown):

```
Round {N} — {short topic, e.g. "Triggers", "Audience", "Failure mode"}

Q: {the question, in full, plain language}

Recommended: {your proposed answer, plain language}
Reasoning: {one or two cited sources}

Confirm, edit, or push back?
```

`{N}` is the next round number — the highest existing round number in the file plus 1, or 1 if no rounds exist.

### Step 4 — Capture the user's response

The user will respond in one of four shapes:
- **Confirm** — accept the recommendation as-is.
- **Edit** — confirm with an adjustment ("confirm but also include X").
- **Reject** — replace with their own answer entirely.
- **Push back** — challenge the question itself ("this isn't the right question because Y", or "you're missing context Z").

If push back: take the feedback. Drop the question, re-articulate it, or pick a better one. Do not force the user to answer a bad question — that's the failure mode this skill exists to avoid.

### Step 5 — Resolve

Once you have a clear answer, formulate the **Resolved line** — the load-bearing field that flows into the PRD body. Format:

```
Resolved: {the resolved assumption, named in PRD-section terms} — flows into §{Section} {ItemRef if applicable}
```

Examples:
- `Resolved: end users (Customer profile per profiles.json) — flows into §Users U1`
- `Resolved: domain events + scheduled reminders (no marketing per decision-log 2026-02) — flows into §Requirements R3, R4`
- `Resolved: persisted notification table + 3x retry then dead-letter — flows into §Requirements R7, AC-3`

If the resolved answer doesn't yet have a clear PRD-section home (because the PRD body doesn't exist yet — that's normal during Phase B), name the *kind* of section it informs: `→ flows into §Requirements (TBD ItemRef)`.

### Step 6 — Append the round to the interview file

Use the Edit tool to insert the round into `docs/specflow/features/NNN-{slug}/NNN-{slug}-interview.md`. The insertion point is inside the `## Rounds` section, immediately after the most recent existing round (or right after the `## Rounds` heading if this is the first round). Use this exact shape:

```markdown
### Round {N} — {topic}
- **Q:** {the question, in full}
- **AI's recommended answer:** {recommendation} — *Reasoning: {cited sources}*
- **User's answer:** {confirmed | edited: {description of edit} | rejected: {their alternative} | pushed back, re-asked as Round {N+1}}
- **Resolved:** {the resolved assumption} — flows into §{Section} {ItemRef}
```

Append-only. Never overwrite a prior round.

### Step 7 — Re-evaluate before the next round

Before looping back to Step 1, ask yourself:
- **Did this round invalidate any pending question?** Drop questions made moot by the answer (e.g. user said "no scheduled reminders" → drop "what reminder cadence?").
- **Did this round open a new question?** Promote it to the queue (e.g. user confirmed scheduled reminders → now ask about timing tolerances).
- **Did this round reveal a goal misalignment?** Check the user's answer against the confirmed Goal section. If they contradict, STOP and tell the user: *"Your answer suggests the goal may need revision. The Goal section says '{paraphrase the goal field}' but your answer implies '{the contradiction}'. Run `specflow:scope-change` to update the goal, or clarify so I can resolve the contradiction."* Wait for direction; do not proceed.

If no misalignment, loop back to Step 1.

---

## Topics not discussed

During grilling, you may identify topics that are deliberately *out of scope* rather than unresolved questions worth asking. Don't ask about them — append them to the `## Topics not discussed` section of the interview file. Format:

```markdown
- *{Topic}* — {why this was left out: out of scope per goal's "Out-of-scope-at-goal-level" field, deferred to a later PRD, deliberately ambiguous because of {reason}}
```

This is the pressure-release valve: reviewers at Gate 2 use it to distinguish silence-by-choice from oversight. If a topic is here, the AI's response cites the reason. If a topic isn't here AND the PRD doesn't address it, that's a gap worth investigating.

---

## Sign-off

When the user signals alignment — "we're done", "looks good, proceed", "OK ship it", or similar — append a sign-off line to the interview file's `## Sign-off` section:

```markdown
## Sign-off
- {YYYY-MM-DD} — user confirmed alignment; specflow:prd Phase C (synthesis) may proceed.
```

If the `## Sign-off` heading already exists (you're extending an existing interview after a scope change), append a new sign-off line under it. Do not overwrite prior sign-offs — they're audit trail.

After sign-off, return one line to the parent (or to the user if invoked manually) confirming: `Interview signed off: features/NNN-{slug}/NNN-{slug}-interview.md ({N} rounds)`. Done.

---

## What you MUST NOT do

- **Do not edit the Goal section.** Only `specflow:scope-change` updates that. If the user wants to change the goal mid-grilling, redirect them.
- **Do not buffer rounds.** Each round must be appended to the interview file as it's resolved. If the conversation drops mid-grilling, the partial transcript survives because every round is on disk.
- **Do not ask multiple questions in one round.** One question at a time, every time. Long forms defeat the purpose.
- **Do not pre-can the question list.** Re-evaluate after every answer (Step 7). The point is adaptive interrogation, not a fixed quiz.
- **Do not skip the reasoning citation.** Every recommended answer cites at least one source. Recommendations without citations are themselves a failure mode this skill catches.
- **Do not write to the PRD body, the tasks file, or anything outside the interview file.** Your scope is the interview only.
- **Do not mention the underlying AI tooling or vendor** in any user-facing output. Per the project's CLAUDE.md, this is non-negotiable.

---

## Verify before declaring done

Before returning to the parent skill or to the user:

1. The interview file exists at the expected path.
2. Every round you wrote has all four fields populated (Q, AI recommended answer with cited reasoning, user's answer, Resolved).
3. Every *Resolved* line is non-empty AND names a PRD section it flows into.
4. The `## Sign-off` section has a dated line for this grilling pass.
5. Any *Topics not discussed* entries have a *why*.

If any verify step fails, fix it before returning.

---

## Reference

- `docs/PRD.md` Appendix Q — interview file structure spec.
- `docs/PRD.md` Appendix N1 — Gate 1 of the adversarial review chain.
- `templates/orchestrator-pattern.md` — fork/file/command-substitution conventions when invoked as a sub-skill of `specflow:prd`.
