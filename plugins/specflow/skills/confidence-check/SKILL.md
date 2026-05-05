---
name: confidence-check
description: Trust-ladder primitive — AI declares uncertainty in plain language before acting on a non-trivial decision, names at least one specific source of uncertainty, and asks for explicit confirmation. Inline-invoked by other skills before risky actions; manually invokable by the user. Logs the declaration and the user's response so Phase 3 self-learning can mine override frequency.
status: v2-new
phase: 1
requires: []
produces:
  - docs/specflow/admin/confidence-log.json
eval: declaration is plain-language (no jargon, no internal-reasoning dump); names at least one specific source of uncertainty (assumption, missing data, conflicting signal); blocks until explicit user confirmation; both declaration and response logged to confidence-log.json.

---

# confidence-check

Trust-ladder primitive. Before the AI takes a non-trivial action, it declares — in plain language — what it's about to do, what it's unsure about, and asks for explicit confirmation. Inline use by other skills; manual invocation when the user wants the AI to slow down.

This skill exists because **single-shot confident execution is what produces autonomous-AI horror stories.** Forcing a plain-language uncertainty declaration before risky actions makes the trust ladder explicit — users know what they're being asked to authorise.

---

## Inputs

The skill is invoked in two modes:

### Mode 1 — Inline (by another skill)

Another skill (`specflow:develop`, `specflow:upgrade`, `specflow:design`, etc.) calls confidence-check before a non-trivial action. The calling skill passes a structured payload:

```json
{
  "calling_skill": "specflow:develop",
  "action_summary": "About to apply a 12-file refactor to src/auth/. The plan moved session lifetime from 24h to 7 days.",
  "category": "red-lane | schema-migration | deletion | confidential-paths | external-api | other",
  "uncertainty_sources": [
    "Plan assumes session-extension is acceptable — interview round 4 mentioned 'longer sessions' but did not confirm 7d specifically.",
    "src/auth/refresh.ts has a TODO from 2025-09 that suggests the team was considering session changes; no decision-log entry."
  ],
  "reversibility": "reversible | hard-to-reverse | irreversible",
  "blast_radius": "single-file | feature | module | repo-wide"
}
```

### Mode 2 — Manual (by the user)

`/confidence-check` — the user wants the AI to articulate uncertainty about whatever the AI was about to do. The AI builds the payload from its own state instead of receiving it.

---

## Phase A — Compose the declaration

Whether inline or manual, produce a single message in this exact shape:

```
🟡 confidence-check before {action_summary in one short sentence}.

What I'm about to do:
{2-3 sentence plain-language description. No jargon. No code samples. No internal reasoning dump.}

What I'm unsure about:
- {uncertainty_source 1 — one sentence, named concretely}
- {uncertainty_source 2 — if multiple}

Reversibility: {reversible | hard-to-reverse | irreversible}.
Blast radius: {single-file | feature | module | repo-wide}.

Proceed?
- Type "yes" to proceed.
- Type "no" or "stop" to halt.
- Type "explain {topic}" to dig deeper into a specific uncertainty.
```

### Plain-language test

Re-read the declaration before sending. If any of these triggers fires, rewrite:

- Jargon the user didn't introduce ("idempotent", "monad", "side-effect", "race condition") → translate.
- Verb-only descriptions ("refactoring", "updating") → name what changes for the user/system.
- Reasoning chains ("I considered X then Y then Z") → state the conclusion, not the reasoning.
- Hedging without substance ("might", "perhaps", "possibly") with no concrete uncertainty → either name the uncertainty or remove the hedge.

The declaration must be readable by someone who didn't read the prior message in the conversation.

### At least one specific source

The "What I'm unsure about" section MUST name at least one concrete source of uncertainty:
- An assumption that might be wrong (cite the assumption).
- Missing data that would change the call (cite what's missing).
- A conflicting signal between sources (cite the two sources).

Generic uncertainty ("there might be edge cases") is NOT acceptable — that's the same as no uncertainty. If you genuinely have nothing concrete to flag, you don't need confidence-check; proceed without it.

---

## Phase B — Wait for explicit response

Block until the user responds. Match their input against:

- **"yes"** (or `proceed`, `go`, `confirm`, `do it`) — proceed to Phase C with `confirmed=true`.
- **"no"** (or `stop`, `cancel`, `halt`, `abort`) — proceed to Phase C with `confirmed=false`. The calling skill must NOT proceed with the original action.
- **"explain {topic}"** — drill into the specified uncertainty source. Re-render the declaration with sharper detail on that point. Then re-block.
- Anything else — re-prompt: *"I need a yes / no, or 'explain {topic}'. The action is paused until you respond."*

A timeout-style "default-yes" is NOT acceptable. The user's silence is not consent.

---

## Phase C — Log the declaration + response

Append to `docs/specflow/admin/confidence-log.json` (create with `{"entries": []}` if missing):

```json
{
  "id": "conf-{YYYY-MM-DD-HHMMSS}-{short-hash}",
  "timestamp": "{YYYY-MM-DD HH:MM:SS}",
  "mode": "inline | manual",
  "calling_skill": "{or 'manual' for mode 2}",
  "category": "{red-lane | schema-migration | …}",
  "action_summary": "{verbatim from declaration}",
  "uncertainty_sources": ["..."],
  "reversibility": "{reversible | hard-to-reverse | irreversible}",
  "blast_radius": "{single-file | feature | module | repo-wide}",
  "user_response": "yes | no | explain-{topic} | other",
  "explain_iterations": 0,
  "elapsed_seconds": "{time between declaration and final response}"
}
```

Use Edit tool to append the new entry to the `entries` array. Do not rewrite past entries.

`elapsed_seconds` is the wall-clock time between Phase A's send and Phase B's resolution — useful Phase 3 signal: very-fast yes responses correlate with rubber-stamping.

---

## Phase D — Report

### Inline mode (Mode 1)

Return a structured result to the calling skill — write to `admin/scratch/confidence-result-{timestamp}.json`:

```json
{
  "status": "confirmed | declined",
  "log_entry_id": "conf-{...}",
  "user_response": "yes | no | other",
  "elapsed_seconds": {n}
}
```

Return the file path (one line). The calling skill reads it via command substitution to decide whether to proceed.

### Manual mode (Mode 2)

Tell the user: *"Logged to `admin/confidence-log.json`. {Confirmed — proceeding | Declined — action halted}."* No further action.

---

## When this skill should be inline-invoked

Calling skills should invoke confidence-check when **any** of:

- The action category is `red-lane`, `schema-migration`, `deletion`, `confidential-paths`, or `external-api`.
- The reversibility is `hard-to-reverse` or `irreversible` (anything pushed remotely, anything that drops data, anything that touches `admin/rules/non-negotiable.md`).
- The blast radius is `repo-wide` (touching root config, build, CI files).
- The action contradicts a `decision-log.md` entry — even reversible actions deserve confidence-check when they're knowingly going against prior decisions.

Calling skills should NOT invoke confidence-check for:
- Routine reads (Read, Grep, Glob).
- Surgical edits inside a feature folder during normal development.
- Anything that's already been through a Gate review and signed off — confidence-check is for *uncaptured* uncertainty.

Over-invocation degrades the signal — when every action prompts confidence-check, users start rubber-stamping, and the discipline is lost.

---

## What you MUST NOT do

- **Do not skip Phase A's plain-language test.** A jargon-laden declaration defeats the point.
- **Do not invent uncertainty.** If you have nothing concrete to flag, don't invoke confidence-check.
- **Do not accept ambiguous user responses as confirmation.** "Sure?" / silence / a thumbs-up emoji are NOT yes.
- **Do not write the action's actual code or file edits during confidence-check.** This skill is a gate, not an executor. The calling skill (or the human, on resume) does the action after `confirmed=true`.
- **Do not log without writing.** Every confidence-check produces a log entry — even declined ones. That corpus is what Phase 3 self-learning mines.
- **Do not mention Claude, Anthropic, or any AI tooling** in the declaration or the log. Per the project's CLAUDE.md.

---

## Verify before declaring done

1. Declaration was sent and contains: action summary, "What I'm about to do" (plain language), "What I'm unsure about" (≥1 specific source), reversibility, blast radius, prompt for response.
2. User responded with one of the recognised inputs (or aborted; no default-yes).
3. Log entry was appended to `admin/confidence-log.json` with all fields populated.
4. (Inline mode) Result file written to scratch and path returned.

If any verify step fails, do not let the calling skill proceed.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 10 — trust-ladder primitives.
- `skills/panic/SKILL.md` — sister primitive for emergency rollback.
- `docs/PRD.md` Appendix I — admin folder + self-learning memory loop (where the confidence-log feeds Phase 3 `/optimize`).
