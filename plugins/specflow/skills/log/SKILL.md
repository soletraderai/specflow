---
name: specflow:log
description: Unified entry point for "log something out of band" — decisions, miscellaneous follow-ups, and mid-development scope changes. The agent classifies the user's free-form intent into one of three internal handlers (decision / misc / scope-change) and dispatches; explicit ambiguity surfaces a three-option prompt. Replaces the three direct entry points (specflow:decision, specflow:misc, specflow:scope-change) for user-facing invocation. Auto-invocation contracts (e.g. specflow:develop calling misc on a rule violation) continue to call the handlers directly — those are internal calls, not user surface.
status: v2-new
phase: 3
requires:
  - docs/specflow/admin/decision-log.md (resolved lazily by handlers)
produces:
  - admin/scratch/log-{timestamp}/routing.json (one entry per dispatch — the classification trace)
eval: |
  every invocation lands in exactly one handler (decision / misc / scope-change) or returns cleanly after a
  user-elected abort; routing.json records the chosen handler, the agent's confidence, and the user prompt
  outcome (auto | confirmed | overridden | aborted); the chosen handler's own eval block governs the
  resulting artefacts — log adds nothing beyond the routing trace.
---

# specflow:log

Single user-facing entry point for the three "log something out of band" workflows. The user states intent in free-form prose; the agent classifies and routes. No flags. No "which command should I use" decision pushed onto the user.

This is a **thin dispatcher** — Phase A reads the intent, Phase B classifies, Phase C confirms (when low-confidence) and dispatches. The chosen handler does the real work; this skill adds only the routing trace.

---

## Inputs

```
specflow:log {free-form description}
specflow:log                            # prompt the user for the description
```

The free-form description is the user's intent in their own words. Examples that route to each handler:

- *"We pinned date-fns at 3.0 because of the timezone bug we kept hitting"* → **decision**
- *"eas.json still has the stale prod domain we should rip out"* → **misc**
- *"We need to add a third payment provider mid-flight — the PRD is wrong about provider-count"* → **scope-change**

The skill never asks "which kind?" up front. Classification is the agent's job.

---

## Phase A — Capture intent

If the description was passed inline, use it verbatim. If not, prompt:

> What do you want to log? (one to three sentences — what happened, why it matters, where it lives)

Write the captured intent to `admin/scratch/log-{timestamp}/intent.txt`. This is the canonical artefact the dispatcher routes against.

---

## Phase B — Classify

Read the intent and decide a handler. The three handlers and their signals:

### decision

The user is recording a choice between alternatives — a library pinned, an architectural reversal, a tool sunsetted, a convention ratified. No task closure. No PRD impact. No file:line trace required.

Signals: verbs like *"decided / picked / going with / reversed / sunsetting / ratified / pinned"*. Mentions of two or more options weighed. Mentions of the *why* behind the choice. Often forward-looking ("from now on we ..."). No feature ID in the intent (the decision applies project-wide or to multiple features).

### misc

The user is flagging a bug, a small fix, or an observation that shouldn't be lost. Often has a file path or rough location. Doesn't warrant a PRD. Sometimes payload-shaped (the calling skill found a rule violation — see auto-invocation contract in `misc/SKILL.md`).

Signals: verbs like *"spotted / found / noticed / TODO / follow up / clean up / rip out"*. A file:line reference (literal `path/to/file:42` or close). Mentions of a rule by name (e.g. *"DOMPurify missing here"*, *"hardcoded value"*). Action-shaped ("we should X").

### scope-change

The user is signalling mid-development PRD drift. A feature is in flight (post-Gate-2 PRD), and the spec itself is now wrong — the PRD needs surgical update + the tasks file needs delta regeneration. This is the heaviest of the three and requires a feature ID.

Signals: verbs like *"the PRD is wrong about ..."*, *"we should also support ..."*, *"actually we need to ..."*. A feature ID mentioned (`{NNN-slug}`). References to existing R-IDs, AC-IDs, or in-flight tasks. Implied requirement for regenerating downstream artefacts.

### Classification confidence

After reading the intent, the agent settles on a handler with one of three confidence levels:

- **High**: signals overwhelmingly point to one handler. Proceed to Phase C dispatch with no prompt.
- **Medium**: signals favour one handler but a second has weight. Phase C surfaces a one-line confirmation: *"Routing this as a `{handler}` — say `change` to pick a different one."*
- **Low (ambiguous)**: signals split. Phase C surfaces the three-option prompt with one-line summaries.

Write the classification + confidence + reasoning to `admin/scratch/log-{timestamp}/routing.json`:

```json
{
  "intent_path": "admin/scratch/log-{timestamp}/intent.txt",
  "chosen_handler": "decision | misc | scope-change",
  "confidence": "high | medium | low",
  "reasoning": "{one or two sentences naming the signals that drove the choice}",
  "prompt_outcome": null
}
```

`prompt_outcome` is populated in Phase C: `"auto"` (high confidence, no prompt), `"confirmed"` (medium confidence prompt accepted), `"overridden"` (medium/low prompt picked a different handler), or `"aborted"`.

---

## Phase C — Confirm or prompt; dispatch

### High confidence

Tell the user one line and dispatch immediately:

> Logging this as a `{handler}` — invoking the handler now.

Set `routing.json.prompt_outcome: "auto"`. Hand off.

### Medium confidence

Tell the user one line and allow override:

> Routing this as a `{handler}` because `{reasoning}`. Reply `ok` to proceed, `change` to pick a different handler, or `abort` to exit.

On `ok`: set `prompt_outcome: "confirmed"` and hand off.
On `change`: fall through to the low-confidence three-option prompt.
On `abort`: set `prompt_outcome: "aborted"` and exit. Leave `intent.txt` for re-invocation; remove `routing.json` (no false trace).

### Low confidence

Surface the three-option prompt:

> This intent could route three ways. Pick one:
> (1) `decision` — record a choice in `admin/decision-log.md` (no task closure, no PRD impact).
> (2) `misc` — log a follow-up bug / small fix / observation in `docs/specflow/misc-task/000-tasks-misc-tasks.md`.
> (3) `scope-change` — mid-development PRD drift; requires a feature ID and regenerates PRD/tasks.
> (4) `abort` — exit without logging.

On a numbered choice: set `prompt_outcome: "overridden"` (or `"confirmed"` if the user's choice matches the agent's classification), update `chosen_handler` if the user overrode, and hand off.
On `abort`: as above.

### Dispatch contract

For the chosen handler, invoke the underlying skill with the intent.txt content. The handler is responsible for its own validation (e.g. `scope-change` will refuse without a feature ID; `misc --auto` requires the payload schema; `decision` requires the six fields).

- `decision` → invoke `specflow:decision "{title-derived-from-intent}"` if a title is extractable in one line; otherwise `specflow:decision` (full interactive).
- `misc` → invoke `specflow:misc "{intent-as-prefill}"` (interactive-with-prefill mode).
- `scope-change` → invoke `specflow:scope-change {NNN-slug}` once the feature ID is resolved. If the intent doesn't name a feature ID and one cannot be inferred from the conversation context, prompt: *"Which feature does this scope change apply to?"* before dispatching.

The agent never bypasses the handler's own phases — log is a router, not a re-implementation.

---

## Verify before declaring done

1. `admin/scratch/log-{timestamp}/intent.txt` exists with the user-captured prose.
2. `admin/scratch/log-{timestamp}/routing.json` exists with all four fields populated (including the final `prompt_outcome`).
3. The chosen handler returned cleanly (its own eval block governs its artefacts; log inherits the handler's success/failure).
4. On `abort`: `routing.json` was removed; `intent.txt` retained for re-invocation.
5. No artefacts under the three handlers' canonical write paths beyond what the handler itself wrote — log adds nothing direct.

---

## What you MUST NOT do

- **Do not write to `decision-log.md`, `000-tasks-misc-tasks.md`, or any feature's PRD/tasks directly.** Always dispatch via the handler. Log is a router; the handlers own their schemas.
- **Do not auto-default on low confidence.** Always surface the three-option prompt. Silent routing is the failure mode this skill exists to remove.
- **Do not invoke `:learn` or `:insights` from here.** Those skills observe corpus state, not individual log events.
- **Do not retain `routing.json` on user abort.** Stale routing traces clutter the scratch directory.
- **Do not mention the underlying AI tooling or vendor** in any user-facing output. Per CLAUDE.md, non-negotiable.

---

## Cross-skill integration

- **`specflow:decision`, `specflow:misc`, `specflow:scope-change`** — the three internal handlers. From 2.14 onward they are flagged in their own SKILL.md as internal: users invoke `specflow:log` and the agent routes. Direct invocation by name still works (and is the path taken by skills like `specflow:develop` Phase E.6's `--auto` payload to misc) — only the user-facing recommendation moves to `:log`.
- **`specflow:develop`, `specflow:task`, `specflow:test`** — auto-invocation contracts unchanged. These skills call `misc --auto` / `decision` / `scope-change` directly with structured payloads; they never route through `:log`.

---

## Reference

- `skills/decision/SKILL.md` — handler for decisions; full schema + phase contract.
- `skills/misc/SKILL.md` — handler for misc-tasks; includes the `--auto` payload schema used by other skills.
- `skills/scope-change/SKILL.md` — handler for PRD-drift; 8-phase orchestrator with PRD/tasks regeneration.
- `templates/orchestrator-pattern.md` — fork conventions inherited by the handlers.
