---
name: decision
description: Lightweight skill to log a decision out-of-band. Interactive prompt walking the user through title / context / decision / rationale / related; appends a structured entry to admin/decision-log.md. Complement to specflow:complete which captures task-completion retros — decision is for noteworthy moments not tied to a closing task.
requires:
  - docs/specflow/admin/decision-log.md (created if missing)
produces:
  - docs/specflow/admin/decision-log.md (appended entry)
eval: |
  appended decision-log.md entry has all six required fields (title, date, context, decision, rationale, related)
  in canonical order; date matches today's YYYY-MM-DD; entry follows the file's existing format byte-for-byte
  (H2 title, blank line, five bold-prefixed paragraphs, blank line, `---`, blank line); idempotent re-invocation
  on a duplicate title surfaces the existing entry and prompts supersede / abort rather than silently appending.
---

# specflow:decision

You are the lightweight out-of-band decision-capture skill. The user hits a decision-worthy moment mid-flow — picking between two libraries, reversing a previous architectural choice, sunsetting a tool, ratifying a convention — and invokes you to capture it without leaving the conversation. You walk the user through the six required fields with sensible inferences (date auto-populated, related references soft-validated), write a single new entry to `admin/decision-log.md` matching the file's existing format byte-for-byte, and return one chat line confirming the entry. Without this skill, out-of-band decisions stay un-captured because no other skill fires at user-decision moments — `specflow:complete` only fires at task close, `specflow:scope-change` only fires when intent changes mid-development.

This is a **6-phase orchestrator** (A → B → C → D → E → F). Each phase is sequential interactive Q&A with a single append-only write at the end. There is no sub-agent forking — the skill is small, parent-context budget ≤2K tokens, and forking would add round-trip overhead with no context savings (per `templates/orchestrator-pattern.md` "When NOT to fork").

The four core principles bind here as everywhere: *Think Before Coding* (assumptions about duplicate-title semantics, schema-drift handling, and `specflow:complete` boundary surfaced explicitly in the artefact); *Simplicity First* (six fields are the minimum to capture the lesson — no `--from-clipboard`, no `--scratch-file`, no auto-detection); *Surgical Changes* (the write touches one entry at end-of-file; prior entries are byte-identical before and after the run); *Goal-Driven Execution* (every phase has an inline binary verify step, every refusal exits with a sentinel chat line — zero silent failures).

---

## Inputs

Two invocation modes (per R1); date is always auto-populated from today (per R2):

- **Full interactive:** `/specflow:decision` — five prompts run in sequence (title, context, decision, rationale, related).
- **Title-prefilled:** `specflow:decision "{title-string}"` — title pre-filled from the argument; remaining four prompts run interactively.

Unrecognised flags (`--from-clipboard`, `--all-fields-as-args`, `--scratch-file`) produce an error message naming the two supported invocation shapes and exit. Tell the user explicitly: *"Decision capture — mode: {full-interactive | title-prefilled}. Date will be auto-populated to today (`{YYYY-MM-DD}`). Starting Phase A."*

---

## Phase A — Pre-flight: file discovery, parse, and `specflow:complete` boundary check

### A.1 Detect `admin/decision-log.md` (per R8)

Use Read on `docs/specflow/admin/decision-log.md`. If the file exists, proceed to A.2. If missing, hold the canonical preamble in working memory for E.2 — H1 `# Decision log`; one-line file-purpose statement (*"The *why* behind non-obvious choices in this project. One entry per decision; entries do not get edited after the fact — superseding decisions get a new entry that links back."*); a `Format per entry:` bullet list naming the six fields (Title, Date, Context, Decision, Rationale, Related) with one-line descriptors per the worked-example file's preamble; trailing blank line + `---`. Tell the user: *"`admin/decision-log.md` not found — will create with the canonical preamble before appending the new entry."* Skip A.2 (no tail to parse).

### A.2 Tail-only parse for malformed-file refusal (per R9)

Verify the file ends with one of two well-formed shapes: **(a)** the canonical preamble alone followed by `---` (no entries yet), or **(b)** a complete entry — H2 title, all five canonical bold-prefixed fields (`**Date:**`, `**Context:**`, `**Decision:**`, `**Rationale:**`, `**Related:**`) in canonical order, trailing `---` separator on its own line.

If neither shape matches (truncated entry, missing trailing separator, dangling preamble without `---`), refuse with the literal sentinel:

*"`admin/decision-log.md` appears truncated or malformed at end-of-file — last shape detected: {description}. Aborting to avoid compounding the corruption. Run `specflow:doctor --feature admin` to validate, or fix the file by hand and re-run."*

Exit without writing; do NOT auto-repair. The R9 parse looks only at the file's tail; mid-file drift is handled by R5's warn-and-proceed path at D.1, not here.

### A.3 `specflow:complete` boundary soft-prompt (per R7)

Use Read on `docs/specflow/admin/task-history.json`. If the file is missing OR its `mtime` is older than 3600 seconds, the soft prompt does NOT fire — proceed to A.4. If `mtime` is within the last 3600 seconds, fire:

*"You closed a task recently (`task-history.json` written {N} minutes ago). If this decision is task-related, consider running `specflow:complete` instead — it will link the decision to the task. Continue with out-of-band entry, or abort?"*

The user must explicitly pick continue or abort. Informational only — never blocks beyond the explicit user choice.

### A.4 Verify before continuing

- File state is one of: missing (E.2 will create), exists-and-well-formed (E.2 will append), or refused-and-exited.
- For exists path: every existing entry's H2 title is held in working memory (Grep `^## `) for the C.1 duplicate-title check.
- The `specflow:complete` soft prompt either did not fire, or the user picked continue.

Hand off to Phase B.

---

## Phase B — Interactive Q&A: title

### B.1 Prompt for title (full-interactive mode only)

For `/specflow:decision` (no argument): prompt — *"Title (short, declarative — e.g. 'Pin Playwright to 1.43.x' or 'Composition over inheritance in src/services/'):"*. For `specflow:decision "{title-string}"`: skip the prompt; use the argument verbatim. Surface the captured title back: *"Title (from argument): `{title}`."*

### B.2 Empty-title refusal

If the user's input (after whitespace trim) is empty, refuse with re-prompt: *"Title is required — without it, the entry has no anchor for `Related` references and no surface for the duplicate-title check. Re-enter:"*. Required-field empty is never silently passed.

### B.3 Verify before continuing

- Title is non-empty after whitespace trim. For title-prefilled mode: title equals the argument value verbatim.

Hand off to Phase C.

---

## Phase C — Duplicate-title check (per R6)

### C.1 Normalise and compare

Compute the comparison form: lowercase, whitespace-trimmed (leading + trailing). Compare against each existing entry's H2 title (same normalisation). On no match, proceed directly to Phase D. On exact match, fire C.2. Title-match is exact-match-after-normalisation; fuzzy matching is explicitly out of scope.

### C.2 Surface the existing entry

Read the matching entry. Surface to the user:

*"A decision titled `{matched title}` already exists (Date: {YYYY-MM-DD}). First 80 chars of its Decision field: `{first 80 chars of existing entry's **Decision:** field}`. How should we proceed?"*

Offer three options:

- **`s` — supersede** — write a new entry whose `**Related:**` field automatically prepends the literal prefix `Supersedes: "{original title}" ({original date})` followed by any user-provided related references separated by `; `. The original entry stays untouched.
- **`a` — abort** — exit without writing. No scratch file (no draft state worth preserving — the user has not yet entered the body fields).
- **`e` — edit-in-place** — REJECTED. Respond with the literal: *"Existing entries are append-only by file preamble. Supersede instead? The new entry can link back to the original."* and re-prompt for `s` / `a`.

Never auto-supersede; always prompt. Empty input or any input not matching `s|a|e` re-prompts.

### C.3 Verify before continuing

- A duplicate-title decision was made: no match found, OR user picked supersede (skill holds the supersede flag for E.1), OR user picked abort and skill has exited.
- For supersede: the original entry's title and date are held in working memory for E.1's `**Related:**` field prefix.

Hand off to Phase D unless aborted.

---

## Phase D — Interactive Q&A: context, decision, rationale, related

### D.1 Schema-drift detection on the last entry (per R5)

Identify the last entry in **file order** — the entry immediately preceding the file's final `---` separator, or the only entry. File order is the canonical reading order; the skill does NOT use date-based ordering. Verify the last entry contains exactly the five canonical bold-prefixed fields in canonical order. If the entry has additional fields (e.g. `**Tags:**`), missing fields, or fields out of order, surface a one-line warning naming both the entry's H2 title AND start line number:

*"Last entry in file order ('{title}', line {N}) contains {description of drift} — continuing with canonical schema."*

Warning fires once and does not block. The new entry will match the canonical schema regardless. Skip this step on the file-missing path (no entries to check).

### D.2 Prompt the four body fields

Each prompt is required; empty input refuses with re-prompt naming why the field cannot be blank:

1. **Context** — *"Context — what was the situation that prompted the decision? One short paragraph:"* Refusal text on empty: *"Context is required — without it, the future reader cannot understand why the decision was made."*
2. **Decision** — *"Decision — what was chosen? One short paragraph:"* Refusal text on empty: *"Decision is required — the entry's namesake field cannot be blank."*
3. **Rationale** — *"Rationale — why this over the alternatives? One short paragraph:"* Refusal text on empty: *"Rationale is required — `decision` without `why` is a stub, not a memory."* Rationale is the most load-bearing field per the file's preamble.
4. **Related** — *"Related files / tasks / PRDs (comma-separated or newline-separated; type `none` if no references):"* Soft-validated per D.3.

### D.3 Soft validation on related references (per R3)

Classify each entry from D.2#4:

- **File path** — contains `/` OR ends with a recognised file extension (`.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.go`, `.rs`, `.md`, `.json`, `.yml`, `.yaml`, `.html`, `.css`, etc.). Check against disk.
- **Task ID** — matches `T-?\d+` (e.g. `T-0042`, `T42`). Resolve against `admin/task-history.json`'s `tasks[].id` field.
- **Free-text** — everything else (e.g. `"Linear ENG-1417"`, prose mentions). Pass through; never validated.

For unresolvable file paths and task IDs, surface a single warning naming each unresolvable reference and prompt keep / edit / drop:

*"Soft validation: path `src/foo.ts` not found on disk; task `T-0042` not in `task-history.json`. Keep both anyway, edit the list, or drop the unresolvable refs?"*

Proceed regardless of choice. Free-text never warns. Never blocks — the user may legitimately reference a path they're about to create. If the user typed `none`, the `**Related:**` field will be the literal `none`.

### D.4 Verify before continuing

- All four prompted fields have non-empty values after whitespace trim, OR related is the literal `none`.
- Related references have been classified and any soft-validation warnings surfaced + resolved.

Hand off to Phase E.

---

## Phase E — Compose and append (per R4 + R10 + R11 Stage 1)

### E.1 Compose the entry byte-for-byte against the canonical schema

Build the entry exactly:

```markdown

## {title}

**Date:** {YYYY-MM-DD using today's date}

**Context:** {context paragraph}

**Decision:** {decision paragraph}

**Rationale:** {rationale paragraph}

**Related:** {comma-separated list, or the literal text "none"}

---

```

For supersede mode: prepend `Supersedes: "{original title}" ({original date})` to the `**Related:**` field, separated from user-provided refs by `; `. If no user-provided refs in supersede mode, the field is just the supersede prefix. If supersede is off and user typed `none`, the field is `none`.

### E.2 Write to disk (append-only)

For the file-missing path: write the preamble (held from A.1) followed by the entry from E.1. For the existing-file path: append the entry to the end of the file. Use append-mode write; never rewrite the file. Prior entries' bytes must not change.

### E.3 Write-failure preservation path (per R10)

If the write raises an error: catch it; compose `{YYYYMMDD-HHMMSS}` in UTC; create `docs/specflow/admin/scratch/` if missing; write the entry content (just the entry, not the preamble — the preamble is the skill's responsibility on retry) to `docs/specflow/admin/scratch/decision-{timestamp}.md`. Surface:

*"Could not write `docs/specflow/admin/decision-log.md`: {underlying error message}. Your draft is preserved at `docs/specflow/admin/scratch/decision-{timestamp}.md` — copy it into the file by hand once the underlying issue is resolved."*

Exit non-zero (or return a structured failure marker so callers can detect the failure programmatically).

### E.4 Stage-1 verification: post-write read-back (per R11)

Re-read the file's last entry. Confirm: (i) H2 title matches the user's input verbatim (or supersede-mode title); (ii) all five bold-prefixed fields are present in canonical order; (iii) `**Date:**` equals today's `YYYY-MM-DD`; (iv) the entry is preceded by `---` (existing-file path) or by the file's preamble + `---` (file-missing path). Any Stage-1 failure aborts the run with a structured error pointing at the scratch path (E.3's preservation applies).

### E.5 Verify before continuing

- E.4 Stage-1 verification passed.
- For supersede mode: `**Related:**` carries the supersede prefix.
- For file-missing path: a subsequent `specflow:decision` invocation would NOT trigger R9's malformed-file refusal on the file just created.
- Prior entries (existing-file path) are byte-identical to their pre-write state.

Hand off to Phase F.

---

## Phase F — Stage-2 chat confirmation (per R11)

### F.1 Emit the binary user-surface contract

After a successful Stage-1 verification, emit exactly one chat line:

*"Logged decision `{title}` to `docs/specflow/admin/decision-log.md`. Date: {YYYY-MM-DD}. Related: {comma-separated list, or the literal text "none"}."*

The chat line is the user's verification surface — they see what the file now says without opening it. It is the only output line returned to the parent context on success. Per R11, Stage 2 is the binary user-surface contract; AC-10 verifies it by literal-text match.

### F.2 Verify before declaring done

1. New entry matches the canonical schema (E.4 Stage-1 passed); today's date in `**Date:**`.
2. Prior entries (existing-file path) byte-identical to pre-write — diff confirms only the new entry was added.
3. For supersede: `**Related:**` carries the supersede prefix. For file-missing path: the canonical preamble is at the file's head; the new entry follows the preamble's `---`.
4. Stage-2 chat line emitted exactly once.

If any verify step fails, refuse to claim success; re-emit with the scratch-file path if E.3 preservation has fired.

---

## Failure modes and anti-patterns

Refusal exits — each emits a documented sentinel line; a refused exit without a sentinel line is a failed run:

- **Malformed file at tail (R9)** — A.2 sentinel routes to `specflow:doctor --feature admin`. No auto-repair.
- **Required field empty (title / context / decision / rationale)** — re-prompt; never write a stub entry. Rationale-empty refusal is non-negotiable — `decision` without `why` is a stub, not a memory.
- **Write failure (R10)** — E.3 preserves the draft to `admin/scratch/decision-{timestamp}.md`; exits non-zero with the scratch path.
- **Stage-1 read-back mismatch (R11)** — E.4 aborts; falls through to E.3's scratch preservation.

User-elected branches — surface the prompt and let the user choose:

- **Duplicate title (R6)** — C.2 prompts supersede / abort; edit-in-place is rejected with the canonical re-prompt. Never auto-supersedes.
- **`specflow:complete` recently fired (R7)** — A.3 soft prompt; user picks continue or abort. Informational only.
- **Soft-validation mismatch on Related refs (R3)** — D.3 prompts keep / edit / drop; never blocks the write.

Warn-and-proceed:

- **Schema drift in last entry (R5)** — D.1 warns once with title + line number; new entry written canonical regardless.

Refuse to do:

- **Mutate an existing entry.** Always append; supersede = new entry that links back via the `**Related:**` prefix; never edit-in-place. Prior entries are byte-identical before and after the run (F.2 verify).
- **Auto-populate fields from chat history.** The user types each field; the skill does not infer body content from prior conversation turns.
- **Reorder, remove, or modify prior entries.** R4's append-only contract is the surgical-changes anchor.
- **Auto-repair a malformed file or auto-supersede on duplicate-title.** Unconditional refusals; the user (or `specflow:doctor`) owns the corrective path.
- **Name any AI vendor or tooling in the entry, the chat line, or any user-facing output.** Per the project's CLAUDE.md attribution rule.

---

## Cross-skill integration

- **`specflow:complete`** — both skills write to `admin/decision-log.md` via the same canonical schema. `specflow:complete` writes task-derived entries when a retro surfaces a significant pattern; `specflow:decision` writes user-driven out-of-band entries. Boundary is intent-source — both can coexist on the same file. The A.3 soft prompt makes the `specflow:complete` lane visible at the user's mid-flow moment when `task-history.json` was written within the last 60 minutes; informational only.
- **`specflow:scope-change`** — also appends decision-log entries when scope changes mid-development. If `decision` and `scope-change` both fire on overlapping surfaces, the most recent write wins by file order; the C.1 duplicate-title check catches accidental same-title re-entry across skills.
- **`specflow:doctor`** — when R9's malformed-file refusal fires, the skill routes to `specflow:doctor --feature admin` for schema validation. `decision` does not auto-repair.
- **`/insights` (Phase 3 future skill)** — reads `decision-log.md` for cross-decision patterns. The schema-canonical writes this skill produces preserve `/insights` consumption even when prior entries have drifted (R5's warn-and-proceed path).

---

## Reference

- `docs/specflow/features/004-decision-skill/004-decision-skill-prd.md` — full requirements R1-R11 and acceptance criteria AC-1 to AC-10.
- `docs/specflow/features/004-decision-skill/004-decision-skill-interview.md` — six interview rounds.
- `docs/specflow/features/004-decision-skill/debate-log/prd-gate2/manifest.md` — Gate 2 (passed-with-revisions); load-bearing constraints: R5 file-order definition (tbc-r1-f1 block resolved), R11 Stage-2 user-surface contract (goal-r1-f1 block resolved), R7 file-missing path (da-r1-f1 partial), R9 tail-only parse scope (tbc-r1-f2 alternate form).
- `templates/orchestrator-pattern.md` — six sequential phases, no sub-agent forking.
- `CORE_PRINCIPLES.md` — the four principles bound to every phase verify step.
- `skills/complete/SKILL.md` — sister Phase 3 skill; mirror schema for `decision-log.md` entries; boundary-by-intent-source partner.
- `skills/scope-change/SKILL.md` — also appends decision-log entries; cross-skill coexistence by file order + duplicate-title check.
- `examples/docs/specflow/admin/decision-log.md` — worked-example file the canonical schema and preamble are taken from.
