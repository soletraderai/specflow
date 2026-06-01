---
name: specflow:misc
description: Internal handler — for user-facing invocation, use `specflow:log` (the agent routes to this handler when your intent is misc-task-shaped). Single-task workflow for bugs and small fixes that don't warrant a PRD. Two invocation modes — interactive (used by `:log` dispatch with the user's prose pre-filled) and auto (structured payload from another skill that spotted an out-of-scope rule violation — this contract is unchanged and routes here directly). Initialises the rolling misc-task file if missing, allocates the next MISC-NNN id, appends the entry, optionally saves assets.
status: v2-new
phase: 1
requires:
  - docs/specflow/admin/config.json
  - docs/specflow/admin/rules/non-negotiable.md
  - docs/specflow/admin/rules/guidelines.md
produces:
  - docs/specflow/misc-task/000-tasks-misc-tasks.md
  - docs/specflow/misc-task/assets/
eval: new entry has unique MISC-NNN id; required fields populated (title, scope, priority, label, estimate, description, verification steps); for auto-created entries the rule reference + why are present; any referenced assets exist on disk; rolling file passes a structural lint (frontmatter + Quick Reference table + Pending Tasks section all present).
---

# specflow:misc

Single-task workflow for bugs, small fixes, and out-of-scope-but-shouldn't-be-lost observations. Two invocation modes — interactive (user-driven) and auto (a calling skill passes a structured payload).

This skill is **load-bearing for the Surgical Changes principle**. When the Surgical Reviewer flags an out-of-scope rule violation that should not be fixed inline, the corrective action is "log it as a misc-task." If this skill isn't operational, that corrective action has no destination and the workflow has a dead-end.

---

## Inputs

The user (or a calling skill) invokes you with one of:

- **Interactive:** `specflow:misc`, "log a quick bug", "add a misc task". You'll ask for the fields.
- **Interactive with prefill:** `specflow:misc {free-form description}`. Parse what you can; ask for what's missing.
- **Auto (from another skill):** `specflow:misc --auto {payload-path}` where the payload is a JSON file at e.g. `admin/scratch/misc-payload-{timestamp}.json` with the fields below.

### Auto-invocation payload schema

```json
{
  "trigger": "surgical-reviewer | rule-violation | other",
  "calling_skill": "{e.g. specflow:task}",
  "title": "{short title — required}",
  "scope": "WEB | MOBILE | BACKEND | SHARED",
  "priority": "P0 | P1 | P2 | P3",
  "label": "Bug | Feature | Tech-debt",
  "estimate": "{e.g. '2h', '1d', or 'unknown'}",
  "description": "{plain-language description — required}",
  "verification_steps": [
    "{step 1}",
    "{step 2}"
  ],
  "rule_reference": "{rule ID from admin/rules/, e.g. NO_HARDCODED_VALUES}",
  "file_line": "{path:line where the violation was spotted}",
  "observation": "{what you saw}",
  "why": "{citing the rule's why-line, this matters because…}",
  "suggested_fix": "{concrete proposal}",
  "assets": [
    {"path": "{source path}", "alt": "{alt text}"}
  ]
}
```

For auto-invocation, `rule_reference`, `file_line`, `observation`, and `why` are **required**. Refuse the invocation if any are missing — the calling skill needs to provide the trace.

---

## Phase A — Detect mode

1. If invoked with `--auto {path}`, set mode to **auto** and read the payload via `Read`.
2. Otherwise set mode to **interactive**.

Tell the user (or log silently for auto): *"Logging a misc-task ({mode} mode)."*

---

## Phase B — Verify or initialise the rolling file

### B.1 Check existence

Use `Glob` for `docs/specflow/misc-task/000-tasks-misc-tasks.md`.

### B.2 Initialise if missing

If the file doesn't exist, use `Bash` + `Write` to create both the directory and the file from this template:

```markdown
---
title: Misc tasks
created: {YYYY-MM-DD}
---

# Misc tasks

Rolling file for bugs, small fixes, and out-of-scope observations. New entries append at the end. Linear export updates the Export Map table.

## Quick reference

| ID | Title | Scope | Priority | Label | Status |
|---|---|---|---|---|---|

## Export map

| ID | Linear ID | Linear URL |
|---|---|---|

## Pending tasks

(entries appended below)
```

Also ensure `docs/specflow/misc-task/assets/` exists:

```bash
mkdir -p docs/specflow/misc-task/assets
```

---

## Phase C — Allocate the next MISC-NNN id

1. Read `000-tasks-misc-tasks.md`.
2. `Grep` for `^### MISC-` to find every existing entry header.
3. Parse the highest existing `NNN`. If none, start at `001`.
4. Increment by 1; zero-pad to 3 digits.

The new id is `MISC-{NNN}`.

---

## Phase D — Gather entry data

### D.1 Interactive mode

Ask the user for each missing field, in this order. Don't ask for fields the user already provided in the prefill.

1. **Title** — one line, ≤8 words.
2. **Scope** — `WEB | MOBILE | BACKEND | SHARED`. Suggest based on the title; confirm.
3. **Priority** — `P0 | P1 | P2 | P3`. Default `P2` if user wants to defer the decision.
4. **Label** — `Bug | Feature | Tech-debt`.
5. **Estimate** — short string like `30m`, `2h`, `1d`, or `unknown`.
6. **Description** — 2-4 sentences in plain language. What's broken / missing, the symptom, where it surfaces.
7. **Verification steps** — bulleted list. Each step binary (could a fresh agent run it and report pass/fail unambiguously).
8. **Assets** — *"Any screenshots / recordings to attach? Paste the path or say 'none'."*

If the user types `done` or `enough` after providing the required fields, proceed even if optional fields are blank.

### D.2 Auto mode

The payload already has every field. Validate:
- `title`, `scope`, `priority`, `label`, `description`, `verification_steps` all present and non-empty.
- `rule_reference`, `file_line`, `observation`, `why` all present (auto-mode requires the trace).

If any required field is missing, refuse the invocation: write `{"status": "refused", "reason": "missing field: {field}"}` to `admin/scratch/misc-result-{timestamp}.json` and return that path. The calling skill must fix the payload and retry.

---

## Phase E — Save assets (if any)

For each asset:

1. Determine the destination filename: `misc-task/assets/MISC-{NNN}-{slug-from-title}.{ext}` (kebab-case the title for the slug; preserve the original extension).
2. Copy the source file to the destination using `Bash`:
   ```bash
   cp "{source-path}" "docs/specflow/misc-task/assets/MISC-{NNN}-{slug}.{ext}"
   ```
3. Verify the copy succeeded (the destination file exists and has the same size as the source).

If any asset copy fails, surface the failure and ask the user how to proceed (skip the asset, retry, or abort the entry).

---

## Phase F — Append the entry

### F.1 Compose the entry markdown

```markdown

### MISC-{NNN} — {title}

- **Scope:** {scope}
- **Priority:** {priority}
- **Label:** {label}
- **Estimate:** {estimate}
- **Status:** Pending
- **Created:** {YYYY-MM-DD}

**Description**

{description}

**Verification steps**

- {step 1}
- {step 2}
- ...

{# For auto-created entries, also include:}

**Trace (auto-created)**

- **Rule:** {rule_reference}
- **Spotted at:** {file_line}
- **Observation:** {observation}
- **Why this matters:** {why} *(citing the rule's why-line)*
- **Suggested fix:** {suggested_fix}
- **Originated from:** {calling_skill}

{# For entries with assets:}

**Assets**

- ![{alt}](./assets/MISC-{NNN}-{slug}.{ext})
```

### F.2 Append to the Pending tasks section

Use `Edit` tool to insert the entry markdown at the end of the file (just above EOF, after the last existing entry or after the `## Pending tasks` header if the file is empty).

### F.3 Update the Quick reference table

Use `Edit` tool to add a row to the Quick reference table:

```markdown
| MISC-{NNN} | {title} | {scope} | {priority} | {label} | Pending |
```

### F.4 Add a placeholder row to the Export map

```markdown
| MISC-{NNN} | — | — |
```

(`specflow:linear` will fill the Linear ID + URL on export.)

---

## Phase G — Verify and report

### G.1 Self-check

Before returning, verify:

1. Entry has a unique `MISC-{NNN}` id (no duplicates).
2. Required fields are non-empty.
3. For auto-created entries, the Trace section is present and complete.
4. Any referenced asset exists on disk at the path the entry cites.
5. Quick reference table has a new row matching the entry.
6. Export map has a placeholder row.

If any check fails, fix it before reporting success.

### G.2 Report

**Interactive mode:** *"Logged `MISC-{NNN} — {title}`. File: `docs/specflow/misc-task/000-tasks-misc-tasks.md`. Run `specflow:linear` to export to Linear when you're ready."*

**Auto mode:** Write a result file to `admin/scratch/misc-result-{timestamp}.json`:

```json
{
  "status": "created",
  "id": "MISC-{NNN}",
  "title": "{title}",
  "file": "docs/specflow/misc-task/000-tasks-misc-tasks.md",
  "trace": {
    "calling_skill": "{calling_skill}",
    "rule_reference": "{rule_reference}"
  },
  "created_at": "{YYYY-MM-DD HH:MM}"
}
```

Return the path of the result file (one line). The calling skill reads it via command substitution to confirm success and surface the new MISC id in its own output.

---

## What you MUST NOT do

- **Do not append entries without unique ids.** Re-run Phase C if there's any doubt — never re-use a MISC-NNN.
- **Do not skip the Trace section for auto-created entries.** Surgical Reviewer's findings depend on the trace being present so the rule violation can be audited later.
- **Do not write entries to the Pending tasks section without also updating the Quick reference table.** They must stay in sync.
- **Do not modify other entries when appending a new one.** Append-only — past entries stay frozen unless the user explicitly invokes a separate edit.
- **Do not invoke `specflow:linear` automatically.** Linear export is a separate user decision.
- **Do not mention the underlying AI tooling or vendor** in any user-facing output, the rolling file, or the result payload. Per the project's CLAUDE.md, this is non-negotiable.

---

## Verify before declaring done

1. `docs/specflow/misc-task/000-tasks-misc-tasks.md` exists and parses (frontmatter + Quick reference table + Export map table + Pending tasks section all present).
2. The new entry's MISC-NNN id is unique.
3. Quick reference table and Export map both have a row for the new id.
4. Any referenced assets exist on disk.
5. For auto mode: result file written to `admin/scratch/misc-result-{timestamp}.json` with `status: "created"`.

If any verify step fails, fix it before returning.

---

## Reference

- `docs/PRD.md` Appendix B — `specflow:misc` skill spec.
- `docs/PRD.md` Appendix O4 — auto-misc-task on rule violation.
- `templates/agents/standard/principles/surgical-reviewer.md` — primary auto-invocation source; the Surgical Reviewer's findings cite "should be a misc-task" as the corrective action.
- `skills/linear/SKILL.md` — handles export to Linear, reads the Export map table written by this skill.
