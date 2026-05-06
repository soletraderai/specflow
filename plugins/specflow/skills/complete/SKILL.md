---
name: complete
description: Retro skill — captures task outcome at completion; appends a structured entry to admin/task-history.json; elevates significant patterns to admin/decision-log.md. Auto-invoked by specflow:develop Phase F when a task closes; manually invoked via /specflow:complete {task-id}.
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md
  - docs/specflow/admin/task-history.json
produces:
  - docs/specflow/admin/task-history.json (appended entry)
  - docs/specflow/admin/decision-log.md (elevated significant pattern, optional)
eval: |
  appended task-history.json entry validates against the schema (id, scope, ai_assistance_level, completed_date,
  what_worked, what_didnt, blast_radius_outcome, prd_anchor populated); idempotent re-invocation skips append; if
  elevation fired, decision-log.md has a corresponding entry citing task_id.
---

# specflow:complete

You are the retro skill that closes the loop between work-done and project-memory. You own the per-task boundary between Phase F.5 (task-history.json development-time write) and Phase F.6 (scratch cleanup) when auto-fired by `specflow:develop`, and you own the manual escape-hatch path for tasks closed outside `specflow:develop`. Every closed task leaves a structured trace in `admin/task-history.json`; significant patterns elevate to `admin/decision-log.md`. Without this skill, the self-learning loop has no input data and the lessons evaporate — six months later the same gotcha recurs because nobody wrote it down.

This is an **8-phase orchestrator** (A → B → C → D → E → F → G → H). Most phases are sequential interactive Q&A with schema-validated file writes; the heavy lifting is the schema discipline, the supersede-in-place semantics, and the lock-file race protection (per `templates/orchestrator-pattern.md` — file handoff between phases via `admin/scratch/003-complete-skill-develop/` and the per-task lock at `admin/scratch/complete-{task-id}.lock`). Parent context budget ≤3K tokens for a single retro write; smaller than `specflow:develop` because the skill does not orchestrate multi-agent debate.

The four core principles bind here as everywhere: think before coding (assumptions about idempotency and supersede semantics named in the artefact, not silent), simplicity first (the schema is the minimum surface to capture the lesson — no speculative configurability, no webhook plumbing), surgical changes (the retro write touches one entry in `task-history.json` and at most one entry in `decision-log.md`), goal-driven execution (every phase has an inline binary verify step, every refusal exits with a sentinel chat line — zero silent failures).

---

## Inputs

The skill is invoked via one of:
- **Auto-fire from `specflow:develop` Phase F** (per task at the F.5 → F.6 boundary, after Verifier passes or option-4 accept-and-ship is elected, after the per-task PR opens). Trigger payload carries `task_id`, the Verifier outcome, and the development-time fields already written at F.5.
- **Manual `/specflow:complete {task-id}`** for tasks closed outside `specflow:develop`. Same retro flow; differs only in the recorded `triggered_by` value.
- **Manual `/specflow:complete --amend {task-id} [--escaped-issue "..." --escaped-blast-radius {low|medium|high}] [--note "..."]`** for retroactive captures (escaped issues post-ship, retroactive notes). Appends to the existing entry's `addenda` array; never mutates the original retro fields.

A Linear webhook trigger is **not** in scope for v1 (per R8). The two trigger paths are the auto-fire and the manual CLI; no webhook listener, no `config.json.complete.webhook*` knobs.

Tell the user explicitly which mode you detected: *"Retro for `{task-id}` — mode: {auto-fire | manual | amend}. Starting Phase A."*

---

## Phase A — Pre-flight + invocation routing + lock acquisition

### A.1 Detect invocation mode (per R1, R2, R10)

Inspect the invocation payload:
- If a `triggered_by_payload` from `specflow:develop` Phase F is present (the parent passed `task_id`, Verifier outcome, and the F.3 marker if option-4 was elected) → mode = `auto-fire`. Set `triggered_by: "specflow:develop-phase-f"` for the retro write.
- Else if the user invoked `/specflow:complete --amend {task-id}` with at least one of `--escaped-issue` or `--note` → mode = `amend`. Hand off to Phase B with the amend payload.
- Else if the user invoked `/specflow:complete {task-id}` with no `--amend` flag → mode = `manual`. Set `triggered_by: "manual-cli"`.
- Else (no `task-id` argument) → re-prompt: *"Provide a task-id (`/specflow:complete T{N}`)."*. Empty input re-prompts; no auto-default.

Verify before continuing:
- The `task-id` resolves to an entry in `features/{NNN-slug}/{NNN-slug}-tasks.md` OR — for manual mode only — the user has confirmed the task-id is for a task closed outside `specflow:develop`. Auto-fire mode REQUIRES a tasks.md entry; if absent, refuse with: *"Auto-fire received task-id `{task-id}` not present in any feature's tasks.md. Cannot proceed."* and exit.
- Read `features/{NNN-slug}/{NNN-slug}-prd.md` and locate the R-anchor for the task (cross-reference the task's `Anchor: R{N}` field). Hold the R-ID and goal-field paraphrase for use at Phase E (`prd_anchor` write).

### A.2 Verifier acceptance check (auto-fire only)

For auto-fire mode: confirm the Verifier outcome from the payload is `pass` OR `option-4-accepted`. If the outcome is `reject` and option-4 was NOT elected, refuse: *"Verifier rejected `{task-id}` and option-4 accept-and-ship was not elected. The task is not closed; retro is premature. Resolve via `specflow:develop` Phase F.3."* Surface the four F.3 options to the user (retry, mark exception, defer to manual sign-off, abort). Exit without writing.

For manual mode: skip this step (the user is closing the work manually; Verifier check is not a precondition).

### A.3 Acquire the lock (per R14)

Check for an existing lock file at `admin/scratch/complete-{task-id}.lock`:
- **No lock present** → create the lock atomically using a write-with-O_EXCL pattern. Lock file body is a single line: ISO-8601 timestamp of acquisition. Proceed to A.4.
- **Lock present, age < 30 minutes** (compare lock body's timestamp to now) → refuse with the literal sentinel chat line:

  *"Retro for `{task-id}` is in flight (started `{timestamp}`). Wait for it to complete, then use `--amend` if you have additions."*

  Where `{timestamp}` is read from the lock file body. Exit without writing. Do NOT remove the lock (the in-flight path owns it).
- **Lock present, age ≥ 30 minutes** → treat as stale (covers crashed-orchestration cleanup). Overwrite the lock with a fresh timestamp and proceed to A.4. Surface the chat-line: *"[stale lock detected for `{task-id}` — proceeding]"*.

The lock is released atomically at the END of Phase H on every exit path (successful write, refused exit, schema-validation failure). A path that completes without removing its own lock is a failed run.

### A.4 Tell the user what you're doing

*"Pre-flight passed: task `{task-id}` resolved to PRD anchor `R{N}`, mode `{auto-fire|manual|amend}`, lock acquired at `{timestamp}`. Reading existing task-history.json entry."*

Hand off to Phase B.

---

## Phase B — Read existing task-history entry; idempotency branch

### B.1 Read `admin/task-history.json`

Use Read tool on `admin/task-history.json`. Locate the entry whose `id` matches `{NNN-slug}-T{N}` for the current task. Three cases to branch on:

### B.2 Case 1 — entry absent

Manual mode is the only mode that can land here legitimately (auto-fire's payload requires the F.5 development-time entry to already exist). If auto-fire lands here, refuse: *"Auto-fire expected an existing F.5 entry for `{task-id}` but none found. Phase F.5 may have failed — resolve in `specflow:develop` before re-firing retro."* Release lock; exit.

For manual mode with no existing entry: proceed to Phase C as a **fresh-entry write**. The entry will be created from scratch with all schema fields (per R5 + Schema Appendix) populated.

### B.3 Case 2 — entry exists, no retro fields populated (Phase F.5 placeholder shape)

The entry has the six development-time fields (`lane_assigned`, `ai_assistance_level`, `elapsed_minutes`, `what_worked`, `what_didnt_work`, `blast_radius_outcome`) AND `superseded_by_retro: false`. Verify before continuing:
- `superseded_by_retro` field MUST be present with value `false`. If the field is absent (cross-skill prerequisite unmet — `specflow:develop` Phase F.5 has not yet shipped the default-flag emission per R4), refuse with the literal sentinel:

  *"Cross-skill schema dependency unmet: `specflow:develop` Phase F.5 must emit `superseded_by_retro: false` as default. See PRD R4 schema-dependency clause."*

  Release lock; exit. Do NOT write any retro fields.

If the field is present with `false`: proceed to Phase C as a **supersede-write** (the retro will replace the three placeholder fields and flip `superseded_by_retro` to `true`).

### B.4 Case 3 — entry exists, retro fields already populated (`superseded_by_retro: true`)

Branch by mode:
- **Auto-fire mode** → skip append; surface the chat-line: *"Retro for `{task-id}` already captured (created `{date}`). Auto-fire skipping — entry preserved."* Release lock; exit cleanly. This is the idempotent-re-invocation path the eval block names.
- **Manual mode without `--amend`** → refuse with the literal sentinel chat line:

  *"Task `{task-id}` already has a retro entry (created `{date}`). Use `--amend` to append addenda; manual edit of `task-history.json` is forbidden."*

  Where `{date}` is the existing entry's `completed_at` value. Release lock; exit. Do NOT write.
- **Amend mode** → hand off directly to Phase E.2 (amend writer). Skip Phases C + D; the amend path appends to the `addenda` array, does not re-run the retro Q&A, does not re-fire the unconditional elevation prompt (per R12 + Gate 3 da-r1-f1 — T12's unconditional elevation fires once at original-retro-time only).

### B.5 Verify before continuing

- Mode and case combination is one of the documented branches (auto-fire + Case 1 → refused; auto-fire + Case 2 → supersede; auto-fire + Case 3 → skip; manual + Case 1 → fresh-entry; manual + Case 2 → supersede; manual + Case 3 → refuse-or-amend).
- Lock is held.
- PRD R-anchor is loaded.

Hand off to Phase C unless the branch above already exited.

---

## Phase C — Interactive Q&A (or auto-mode synthesis)

### C.1 Walk the user through the retro fields (manual mode)

Present the questions one at a time; the user types a free-form response per question. The questions are short (5 questions plus the elevation prompt — kept short per R3 + the Power User audience), plain language, no leading framing.

The retro field set is exactly:

1. **`actual_hours: number`** — *"How many hours did this actually take you (reflective, including thinking/blocked/context-switch time — distinct from wall-clock)? Type `0` if you decline to estimate."* Accept any positive number including fractional (`0.3` is valid for a 20-minute typo fix). The `0` value is the convention-bound "user declined to estimate" sentinel; positive numbers (including fractional) are captured values. Reject negative input with re-prompt.

2. **`regressions_caught: {count, descriptions[]}`** — *"How many regressions did you catch DURING implementation (machine-check fail, gate failure, Verifier reject before final pass)? Briefly describe each."* User supplies a count; the skill prompts for `count` description lines. The schema invariant `count === descriptions.length` is enforced at write time; mismatch is a failed write.

3. **`escaped_issues: {count, descriptions[], blast_radii[]}`** — *"How many issues escaped to AFTER ship (production incident, code-review catch, follow-up bug)? Briefly describe each AND mark blast_radius (low/medium/high) per item."* At retro time the default is `{count: 0, descriptions: [], blast_radii: []}` because escaped issues typically surface later — the AMEND path (Phase E.2) is the primary capture surface. The schema invariant `count === descriptions.length === blast_radii.length` is enforced; mismatch is a failed write.

4. **`what_worked: string`** (supersede field — replaces Phase F's placeholder per R4) — *"In one short paragraph, what worked?"* Free-form. Never empty; refuse empty input with re-prompt.

5. **`what_didnt_work: string`** (supersede field — replaces Phase F's placeholder per R4) — *"In one short paragraph, what didn't work?"* Free-form. Never empty (use `(none)` if there is genuinely nothing); refuse empty input with re-prompt.

6. **`ai_assistance_level: "full" | "partial" | "none"`** (supersede field — replaces Phase F's placeholder per R4) — *"AI assistance level on this task: full / partial / none?"* Constrained to one of three literal values; reject other inputs with re-prompt.

7. **Elevation prompt** (per R6 condition (a)): *"Should this land in `decision-log.md`? Y/N — if Y, give a one-line description of the lesson."* Held in working memory for Phase D evaluation.

If the user declines all field prompts (refuses to answer any of questions 1-6), refuse with: *"Retro fields are the project's memory — declining them defeats the purpose. Run `/specflow:complete --skip {task-id}` only if explicitly approving a no-retro close (auto-fire never lands here)."*

### C.2 Auto-mode synthesis

For auto-fire mode, synthesise the retro fields from the Verifier output payload:
- `actual_hours` — synthesised from Phase F's `elapsed_minutes` (convert minutes to hours, round to one decimal). User can confirm or override before write.
- `regressions_caught` — synthesised from the Gate 5 manifest's accepted-findings count + descriptions. User can confirm or override.
- `escaped_issues` — defaulted to `{count: 0, descriptions: [], blast_radii: []}` (escaped issues are post-ship signal; defer to amend).
- `what_worked` / `what_didnt_work` — synthesised from the Verifier output's pass/reject summary plus the Gate 5 closing rationale. User can confirm or edit.
- `ai_assistance_level` — read directly from `lane-assignments.json` for the task (green → full, yellow → partial, red → none — the convention from Phase F.5's existing schema). User can confirm or override.

Surface the synthesised draft to the user with one prompt: *"Auto-mode draft for `{task-id}` — confirm to write as-is, edit any field, or cancel."* Confirm proceeds to Phase D; edit re-renders with the user's changes; cancel releases the lock and exits cleanly.

### C.3 Verify before continuing

- Every retro field has a non-null value matching its schema shape (per R3 + Schema Appendix).
- Field-name regex `^[a-z][a-z0-9_]*$` holds for every key (per R5).
- Count-vs-array invariants hold (`regressions_caught.count === regressions_caught.descriptions.length`; `escaped_issues.count === escaped_issues.descriptions.length === escaped_issues.blast_radii.length`).

Failed validation surfaces a structured failure: *"Retro field `{name}` failed schema validation: `{reason}`. Re-enter."* and re-prompts the failing field. Hand off to Phase D once all fields validate.

---

## Phase D — Significance elevation evaluation (per R6)

### D.1 Apply the two-condition rule

Compute `elevation_fired_by` from two inputs:
- **Condition (a)** — user answered `Y` to the Phase C.1 elevation prompt AND provided a non-empty description.
- **Condition (b)** — `escaped_issues` field has at least one entry whose `blast_radius` is `medium` or `high`.

Resolve to the recorded enum:
- Both fire OR only (b) fires → `elevation_fired_by = "escaped-issue-medium-or-high"` (condition (b) takes precedence in the recorded value when both fire — deterministic shape `/insights` consumers read).
- Only (a) fires → `elevation_fired_by = "user-flag"`.
- Neither fires → `elevation_fired_by = "none"`.

### D.2 Accept-and-ship override (per R11 + R12)

If the existing entry has (or will have) `accepted_with_failure: true` AND mode is `auto-fire` OR `manual` (NOT `amend` per Gate 3 da-r1-f1) — the elevation prompt fires unconditionally regardless of D.1's outcome. Override `elevation_fired_by` to whatever D.1 produced (preserve the firing-condition record) but set `elevation_offered = true` even when D.1 returned `none`. Phase F's elevation-write logic will fire the user-review prompt; the user can still answer N (records `elevation_outcome: "cancelled"`).

For amend mode, skip the unconditional override — T12's prompt fires once at original-retro-time only. Amends follow R6's two-condition rule via D.1.

### D.3 Track the triple flag (per Gate 2 simplicity push-back)

Hold three flags in working memory for Phase E to write:
- `elevation_offered: bool` — true when D.1 (or D.2 override) means a prompt will fire; false when `elevation_fired_by = "none"` AND no accept-and-ship override.
- `elevation_fired_by: "user-flag" | "escaped-issue-medium-or-high" | "none"` — the firing-condition record per D.1.
- `elevation_outcome: "written" | "cancelled" | "not-offered"` — populated by Phase F's user-review prompt; defaults to `"not-offered"` when `elevation_offered: false`.

All three flags are written to the entry per Gate 2 simplicity-r1-f1 push-back: each carries distinct semantic content (offered-vs-fired-vs-outcome), needed for `/insights`-cadence pattern detection. Collapsing to a single flag would lose signal.

### D.4 Verify before continuing

- `elevation_fired_by` enum value matches one of the three documented strings.
- `elevation_offered` is `true` when the prompt will fire (D.1 fires OR D.2 override) and `false` otherwise.
- For amend mode, D.2 override did NOT fire.

Hand off to Phase E.

---

## Phase E — Write task-history.json (append-only)

### E.1 Compose the retro entry (fresh-entry + supersede paths)

Build the entry per the PRD Schema Appendix:

```json
{
  "id": "{NNN-slug}-T{N}",
  "feature": "{NNN-slug}",
  "task_id": "T{N}",
  "title": "{from tasks.md task entry}",
  "scope": ["{from tasks.md Scope field — array of file paths}"],
  "completed_at": "{YYYY-MM-DD using today's date}",
  "triggered_by": "specflow:develop-phase-f | manual-cli",
  "lane_assigned": "green | yellow | red",
  "ai_assistance_level": "full | partial | none",
  "elapsed_minutes": N,
  "what_worked": "{retro text — supersedes Phase F placeholder}",
  "what_didnt_work": "{retro text — supersedes Phase F placeholder}",
  "blast_radius_outcome": N,
  "superseded_by_retro": true,
  "actual_hours": N,
  "regressions_caught": {"count": N, "descriptions": [...]},
  "escaped_issues": {"count": N, "descriptions": [...], "blast_radii": [...]},
  "accepted_with_failure": true | false,
  "elevation_offered": true | false,
  "elevation_fired_by": "user-flag | escaped-issue-medium-or-high | none",
  "elevation_outcome": "written | cancelled | not-offered",
  "decision_log_links": [{"date": "YYYY-MM-DD", "title": "..."}],
  "scope_change_log": [{"date": "YYYY-MM-DD", "title": "..."}],
  "addenda": [],
  "prd_anchor": "features/{NNN-slug}/{NNN-slug}-prd.md#R{N}"
}
```

### E.2 Amend-mode write

For amend mode, skip E.1 entirely. Build a single addendum:

```json
{
  "date": "{YYYY-MM-DD}",
  "kind": "escaped-issue | note",
  "description": "{from --escaped-issue or --note flag}",
  "blast_radius": "low | medium | high"
}
```

The `blast_radius` key is present iff `kind` is `"escaped-issue"`. Append the addendum to the existing entry's `addenda` array. **No other field on the entry changes.** A diff of `task-history.json` before vs after the amend MUST show the entry's `addenda` array grew by exactly one element AND no other field on that entry changed; any other diff is a failed write.

If the amend's `kind` is `"escaped-issue"` AND `blast_radius` is `"medium"` or `"high"`, R6's elevation rule fires from the addendum (loop back to Phase D.1 with the addendum as the input; Phase F's user-review-before-write prompt applies; the composed `decision-log.md` entry's `Rationale` cites the addendum's `description` verbatim, NOT the original retro fields).

For amends on entries where `accepted_with_failure: true`: the unconditional elevation prompt does NOT re-fire (per Gate 3 da-r1-f1). Subsequent amends follow R6's two-condition rule.

### E.3 Compute scope_change_log (per R13)

Read `admin/decision-log.md`. Locate every entry whose `Related` field cites the task-id. Citation match is substring-based, accepting both formats (per PRD Open Questions): `{NNN-slug}-T{N}` literal AND prose mentions like `"task T{N} of feature {NNN-slug}"`. Capture each matching entry's `{date, title}` in chronological order (oldest first).

Edge case: if `tasks.md` no longer contains an entry for `{task-id}` (the task was dropped during scope-change), refuse with the literal sentinel:

*"Scope-changed task `{task-id}` no longer present in `tasks.md`. Retro skipped; review `decision-log.md` for the dropping rationale."*

Release lock; exit. Do not write a stub retro (a dropped task did not close — `specflow:complete` exists to capture closure-time lessons).

### E.4 Schema validation (per R5 + R3 invariants)

Before writing, validate:
- Every top-level key matches `^[a-z][a-z0-9_]*$` (snake_case).
- Every top-level key is in the allow-list (the 25 fields named in the Schema Appendix). Extraneous keys are a failed write — refuse with: *"Schema violation: extraneous field `{name}`. Allowed: {appendix list}. Refusing write."* Release lock; exit.
- Count-vs-array invariants hold for `regressions_caught` and `escaped_issues`.
- All required enum fields hold one of the documented enum values.

### E.5 Append to task-history.json

For fresh-entry path: append the new entry to the `tasks` array.
For supersede path: write-in-place on the existing entry's seven mutable fields (`what_worked`, `what_didnt_work`, `ai_assistance_level`, plus the three retro-only fields per R3, plus `superseded_by_retro: false → true`). All other fields preserved. The diff of `task-history.json` before vs after a supersede write MUST show exactly the documented field set changed.
For amend path: append the addendum to the entry's `addenda` array (per E.2). No other field on the entry changes.

The skill NEVER mutates an existing entry's already-populated retro fields outside the supersede-once-from-`false`-to-`true` semantic. Manual edit of `task-history.json` outside the skill is forbidden by convention.

### E.6 Verify before continuing

- The entry validates against the schema (E.4 passed).
- The disk write succeeded (re-read the file; locate the entry; confirm the field set).
- For amend: addenda count incremented by exactly one; no other field changed.
- For supersede: exactly the documented field set changed; `superseded_by_retro` is now `true`.

Hand off to Phase F.

---

## Phase F — Optional elevation write to decision-log.md

### F.1 Skip when no elevation prompt fires

If `elevation_offered: false` (Phase D resolved to `elevation_fired_by: "none"` AND no accept-and-ship override), skip directly to Phase G. Set `elevation_outcome: "not-offered"` on the entry (already written at E.5; nothing to do).

### F.2 Compose the candidate decision-log entry (per R7)

When elevation fires, compose a candidate entry with exactly five fields (the `Date` field is auto-populated by the write-helper at write-time per Gate 2 goal-r1-f2 push-back defence — five composed fields, not six):

- **Title** — `Retro: {task title}`.
- **Context** — the task's PRD anchor (R-ID + goal-field paraphrase loaded at A.1).
- **Decision** — `"{user-stated lesson} — pattern surfaced via specflow:complete"`. The user-stated lesson is the Phase C.1 elevation-prompt description (condition a) OR the escaped-issue description (condition b, when (b) fires alone or alongside (a)).
- **Rationale** — the retro field that fired the elevation, quoted verbatim. For condition (a): the user's elevation-prompt description. For condition (b): the offending `escaped_issues.descriptions[i]` entry. For amend-driven elevation: the addendum's `description`.
- **Related** — task-history entry id (`{NNN-slug}-T{N}`) plus PRD path (`features/{NNN-slug}/{NNN-slug}-prd.md`).

### F.3 Surface the candidate to the user

Present the five-field candidate. Offer exactly three options:
- **`1` — confirm-and-write** — append to `decision-log.md` as-rendered.
- **`2` — edit-and-write** — accept an edited five-field version, re-validate the field shape, write.
- **`3` — cancel** — drop the elevation; no `decision-log.md` write.

The skill MUST NOT proceed without an explicit user choice. Empty input or any input not matching `1|2|3` re-prompts. Auto-default is a failed run.

### F.4 Accept-and-ship Phase F.3 linkage (per R12)

For accept-and-ship retros (`accepted_with_failure: true`): before composing a fresh entry, read `admin/decision-log.md` and locate every entry whose `Related` field cites the `task-id`. Surface each matching entry's title to the user with a confirm-or-edit prompt. The user's choices:
- **Confirm an existing entry** → record the link in the retro entry's `decision_log_links` (no new `decision-log.md` write).
- **Edit an existing entry's title-as-surfaced** → produces a NEW `decision-log.md` entry with the user-edited content; the original is preserved.
- **None of the above** → fall through to F.2 to compose a fresh candidate.

The skill NEVER duplicates an existing decision-log entry. A run that writes a duplicate `decision-log.md` entry with the same title as an existing Phase F.3 entry is a failed run.

### F.5 Write the decision-log entry (when option 1 or 2 chosen)

Append the entry to `admin/decision-log.md` using the same write-helper conventions `specflow:decision` uses (mirror schema per PRD R7 + Cross-skill integration). The entry follows the existing file's format (see `admin/decision-log.md` head for the canonical shape: Title as `## ...` heading, then `**Date:**`, `**Context:**`, `**Decision:**`, `**Rationale:**`, `**Related:**` lines, separated by `---`).

After the write:
- Capture `{date, title}` of the new entry in the retro entry's `decision_log_links` field.
- Set `elevation_outcome: "written"`.
- Re-write the retro entry's `decision_log_links` and `elevation_outcome` fields in `task-history.json` (the only post-E.5 mutation permitted on the entry — explicitly the elevation linkage, not the retro content).

### F.6 Cancel path (option 3)

Set `elevation_outcome: "cancelled"`. Re-write the retro entry's `elevation_outcome` field. No `decision-log.md` write. The cancellation flag is the `/insights`-cadence pattern signal R7's last sentence names — captured for downstream pattern detection.

### F.7 Verify before continuing

- If elevation fired: `elevation_outcome` is `"written"` or `"cancelled"`, never `"not-offered"`.
- If `elevation_outcome: "written"`: a new entry exists in `decision-log.md` AND `decision_log_links` captured the new entry.
- The new `decision-log.md` entry's `Related` field cites the `task-id` (every entry traces to its source — anti-pattern guard).
- No duplicate entries written for accept-and-ship linkage.

Hand off to Phase G.

---

## Phase G — Linear status sync (optional, best-effort)

### G.1 Detect Linear MCP availability

Read `admin/environment.json`. If `mcp.linear.available: true` AND the task's `tasks.md` entry has a Linear ID in the Export Map (or the auto-fire payload from `specflow:develop` carries a Linear ID): proceed to G.2.

If MCP unavailable OR no Linear ID present: print the chat-only line `[linear status: T{N} → Done (skipped — MCP not available OR no Linear ID)]` and skip to Phase H.

### G.2 Transition the issue

Fire the Linear status transition for the issue: target status `Done`. Best-effort — failure logs but does not block completion.

**Important:** `specflow:develop` Phase F.4 fires `In Progress → In Review`; the human merge fires `In Review → Done` typically. The retro skill's `Done` transition is the **post-merge** semantic — only fire when the user has confirmed the PR merged (or when auto-fire's payload signals merge has occurred). If the merge state is ambiguous, surface the chat-line: *"Linear sync: PR merge state ambiguous for `{task-id}` — skipping Linear transition. Run manually if appropriate."* and skip the transition.

### G.3 Verify before continuing

- Linear transition fired successfully (200-OK from MCP) OR documented chat-only fallback line surfaced OR ambiguous-merge skip surfaced.
- No Linear write attempted when MCP unavailable.

Hand off to Phase H.

---

## Phase H — Lock release + chat-line summary

### H.1 Release the lock atomically

Remove `admin/scratch/complete-{task-id}.lock`. The remove fires on every exit path the orchestrator reaches (successful write, schema-validation failure, refused exit due to existing-retro, scope-changed-task-missing, cross-skill prerequisite unmet, all of Phase A's refusal exits). A path that reaches Phase H without removing its own lock is a failed run.

### H.2 Emit the chat-line summary (per AC-15 / Goal Outcome surface (c))

On every successful retro write (fresh-entry, supersede, OR amend), emit the literal chat-line summary:

*"T-`{task-id}` retro captured. Outcome: `{blast_radius_outcome}`. Elevated: `{yes|no}`. Linear: `{synced|skipped|not-configured}`."*

The form aligns with the user-visible feedback surface; the four resolved tokens carry the load-bearing signals for `/insights` consumption and human visibility.

For successful writes that elevated: append on a second line: *"Decision-log entry: `{title}`."*

### H.3 Structured failure on every refused exit (per AC-15 / Goal Outcome surface (d))

The five refusal sentinel chat lines (each defined inline at its phase) are the only failure surfaces; the skill never silently exits:

1. **A.1 / A.2 — auto-fire missing entry / Verifier reject without option-4** — surfaced at A.1/A.2 with structured payload citing `{task-id}` + the missing precondition.
2. **A.3 — concurrent-fire lock detected** — *"Retro for `{task-id}` is in flight (started `{timestamp}`). Wait for it to complete, then use `--amend` if you have additions."*
3. **B.3 — cross-skill schema dependency unmet** — *"Cross-skill schema dependency unmet: `specflow:develop` Phase F.5 must emit `superseded_by_retro: false` as default. See PRD R4 schema-dependency clause."*
4. **B.4 — plain-mode existing-retro** — *"Task `{task-id}` already has a retro entry (created `{date}`). Use `--amend` to append addenda; manual edit of `task-history.json` is forbidden."*
5. **E.3 — scope-changed task missing from tasks.md** — *"Scope-changed task `{task-id}` no longer present in `tasks.md`. Retro skipped; review `decision-log.md` for the dropping rationale."*

A refused exit that does not emit one of the five sentinel failure lines is a failed run (silent exit is the failure mode this skill exists to remove).

### H.4 Verify before declaring done

1. The task-history.json entry passes schema validation (Schema Appendix allow-list, snake_case, count-vs-array invariants).
2. If elevation fired: a corresponding `decision-log.md` entry exists; the entry's `Related` field cites the `task-id`; the retro entry's `decision_log_links` captured the new entry.
3. Lock file at `admin/scratch/complete-{task-id}.lock` no longer exists.
4. Chat-line summary OR one of the five sentinel failure lines was emitted.
5. For amend mode: exactly one addendum appended; no other field on the entry changed.
6. For supersede mode: exactly the documented seven mutable fields changed; `superseded_by_retro` is now `true`.
7. For idempotent re-invocation (auto-fire on existing retro): no append occurred; lock released; chat-line surfaced.

If any verify step fails, surface the failure and refuse to claim success.

---

## Failure modes

The following are explicit failure modes the skill handles without silent retry. Each maps to a documented user-elected response or a sentinel refusal exit:

- **Concurrent-fire (lock fresh)** — A.3 refuses with the in-flight sentinel; the second-to-start path exits without writing. The first path retains the lock until its own Phase H.1 release.
- **Acceptance not passed (Verifier rejected, no option-4)** — A.2 refuses; the user gets the four F.3 options surfaced (retry, mark exception with annotation, defer to manual sign-off, abort). Do not auto-retry; the user elects the path.
- **Cross-skill schema dependency unmet** — B.3 refuses with the cross-skill sentinel; the `specflow:develop` enhancement PRD must land first per R4.
- **Plain-mode re-invocation on existing retro** — B.4 refuses with the existing-retro sentinel; the user elects `--amend` for follow-ups.
- **Scope-changed task no longer in tasks.md** — E.3 refuses with the missing-task sentinel; the user reviews `decision-log.md` for the dropping rationale.
- **Schema validation failure on write** — E.4 refuses with the structured `Schema violation: ...` line citing the offending field; the entry is rejected; the user revises and re-fires.
- **User declines all field prompts** — C.1 refuses; the retro is the project's memory and declining defeats the purpose.
- **Linear MCP absent or webhook misfire (G.1/G.2)** — log the chat-only fallback line and continue. Best-effort; never blocks completion.
- **Amend with neither `--escaped-issue` nor `--note`** — re-prompt; refuse to proceed without at least one.

---

## Anti-patterns (refuse to do)

- **Mutate an existing `task-history.json` entry's retro fields after they're populated.** Always append (fresh-entry) / supersede-once (Phase F placeholder → retro fields, single transition) / amend (`addenda` array). Never overwrite already-populated retro content. The supersede-write fires once per entry; subsequent retros on the same task-id refuse via B.4.
- **Auto-elevate without user confirmation.** `elevation_offered` MUST be explicit — Phase F.3's three-option prompt is the only path to a `decision-log.md` write; no silent escalation, no auto-default.
- **Write to `decision-log.md` without referencing the task_id.** Every entry's `Related` field MUST cite the `task-id`. The trace anchor is non-negotiable.
- **Skip Verifier acceptance check (auto-fire mode).** A.2's check ensures the task is "complete" per Verifier output, not per AI judgement. Phase F.3 option-4 is the only accepted-with-failure path.
- **Duplicate an existing decision-log entry.** Accept-and-ship F.4 confirm-path records linkage WITHOUT writing; only edit-and-write produces a new entry. A run that writes a duplicate is a failed run.
- **Name any AI vendor or tooling in user-facing output.** The retro entry, the decision-log entry, and the chat-line summary stay vendor-neutral. Per the project's CLAUDE.md attribution rule, this is non-negotiable.
- **Bypass the lock.** No path may write to `task-history.json` or `decision-log.md` for `{task-id}` without holding the corresponding lock. Stale-lock proceed (A.3 case 3) overwrites and proceeds; never bypasses.
- **Write a stub retro for a scope-dropped task.** E.3's missing-task path REFUSES; a dropped task did not close, and a stub entry would corrupt the corpus shape `/insights` reads.

---

## Cross-skill integration

- **Auto-invoke from `specflow:develop` Phase F** — the parent passes the trigger payload (task_id, Verifier outcome, F.3 option-4 marker if elected, lane assignment for `ai_assistance_level` synthesis). Phase A.1 detects auto-fire from the payload presence and skips the manual-mode prompts in C.1 (auto-mode synthesis at C.2 instead, with confirm-or-edit-or-cancel surface).
- **`specflow:scope-change`** — if the task was scope-changed mid-execution, E.3 reads the scope-change-emitted decision-log entries (matched by `Related` substring) and captures them in `scope_change_log`. The retro is asked against the FINAL post-scope-change task definition; the audit trail is preserved via the cited entries. Single retro entry per task regardless of scope-change count.
- **`/insights` (Phase 3 future skill)** — `task-history.json` is the corpus `/insights` reads for cross-task patterns. The schema (Schema Appendix; v1 contract) MUST match `/insights` consumption. Schema additions land via PRD revision; the allow-list refusal at E.4 is the schema-stability surface.
- **`specflow:decision`** — both skills write to `admin/decision-log.md`; F.5 uses the same write-helper conventions / mirror schema. The entry's five fields (Title / Context / Decision / Rationale / Related) match `specflow:decision`'s output shape. Date is auto-populated at write-time by the write-helper, not composed from retro state.

---

## Reference

- `docs/specflow/features/003-complete-skill/003-complete-skill-prd.md` — full requirements R1-R14 and acceptance criteria AC-1 to AC-15; Schema Appendix (v1 contract).
- `docs/specflow/features/003-complete-skill/003-complete-skill-tasks.md` — 15 tasks (T1-T15) implementing the PRD with Gate 2 + Gate 3 + Gate 4 revisions applied.
- `docs/specflow/features/003-complete-skill/debate-log/develop-gate4/manifest.md` — Gate 4 closing decision (passed-with-revisions; 13 Green / 2 Yellow / 0 Red lane split) and the plan revisions this orchestrator body honours.
- `templates/orchestrator-pattern.md` — three primitives (forked sub-agent contexts, file handoff, command substitution) — file handoff via `admin/scratch/003-complete-skill-develop/` is the load-bearing primitive here; sub-agent forking is light because the skill is mostly sequential interactive Q&A.
- `CORE_PRINCIPLES.md` — the four principles bound to every phase verify step.
- `skills/develop/SKILL.md` Phase F.5 → F.6 — the auto-fire boundary; cross-skill `superseded_by_retro` schema dependency.
- `skills/decision/SKILL.md` — sister Phase 3 skill; mirror write-helper for `decision-log.md` entries.
- `skills/scope-change/SKILL.md` — produces decision-log entries this skill cites in `scope_change_log` (R13).
- `admin/task-history.json` — the corpus this skill writes; existing entries match the schema for forward compatibility.
- `admin/decision-log.md` — the human-readable narrative this skill optionally elevates patterns to.
