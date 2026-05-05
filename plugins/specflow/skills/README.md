# Skills

One folder per skill. Each folder has a `SKILL.md` with frontmatter + body that describes the skill's purpose, triggers, inputs, outputs, and verification steps.

The authoritative glossary across all skills lives in [`../SKILLS.md`](../SKILLS.md). Every skill in this folder MUST have an entry there — it's the discoverability contract.

## Layout

```
skills/
├── README.md                  # this file
├── setup/SKILL.md             # specflow:setup
├── prime/SKILL.md             # specflow:prime
├── prd/SKILL.md               # specflow:prd
├── render/SKILL.md            # specflow:render
├── task/SKILL.md              # specflow:task
├── test/SKILL.md              # specflow:test
├── design/SKILL.md            # specflow:design
├── linear/SKILL.md            # specflow:linear
├── misc/SKILL.md              # specflow:misc
├── upgrade/SKILL.md           # specflow:upgrade
├── doctor/SKILL.md            # specflow:doctor
├── budget/SKILL.md            # specflow:budget
├── grill/SKILL.md             # /grill
├── panic/SKILL.md             # panic
├── confidence-check/SKILL.md  # confidence-check
├── feedback-loop-audit/SKILL.md
├── simplify/SKILL.md          # simplify (autoresearch loop)
├── develop/SKILL.md           # specflow:develop (Phase 2)
├── agent/SKILL.md             # specflow:agent (Phase 2)
├── complete/SKILL.md          # specflow:complete (Phase 3)
├── decision/SKILL.md          # specflow:decision (Phase 3)
└── scope-change/SKILL.md      # specflow:scope-change (Phase 3)
```

## Required frontmatter

```yaml
---
name: <command-name>
description: <one sentence — used to decide when to invoke the skill>
status: shipped | v2-enhancement | v2-new
phase: 1 | 2 | 3
requires: [<list of file paths the skill expects to read; empty list if none>]
produces: [<list of file paths the skill writes; empty list if none — e.g. in-context-only skills>]
eval: <binary success check — what the verifier reads to confirm the skill produced what it should>
---
```

### Field semantics

- **`requires:`** — file paths (with `{slug}` / `{NNN}` placeholders allowed) the skill reads as input. Used by orchestrators to validate handoff before invoking the skill. Empty list (`[]`) is valid for skills that take no file input.
- **`produces:`** — file paths the skill writes as output. Used by orchestrators to know what's available for downstream skills. Empty list (`[]`) is valid for skills whose output is in-context only (e.g. `prime`, `confidence-check`). Side effects on derived artefacts (like `prd.html` re-rendered after `prd.md` changes) belong here too.
- **`eval:`** — non-negotiable for any skill that produces output. Strong success criteria let the AI iterate independently; weak ones force constant clarification.

### Why `requires:` and `produces:` are file-level, not schema-level

Phase 1 keeps the contracts lightweight — file paths only, not full JSON schemas. Orchestrators can validate "did the file appear?" without inspecting body structure. This is enough for Phase 1's mostly-linear chains. Phase 2 (when `specflow:develop` orchestrates dynamic compositions) may extend to declared output schemas — but only when there's a concrete second consumer demanding the validation. (Simplicity First.)

## How to add a new skill

1. Create the folder: `skills/<skill-name>/`.
2. Write `SKILL.md` with the required frontmatter and body sections (Triggers, Inputs, Outputs, Verify steps, Reference).
3. Add the entry to `../SKILLS.md` glossary — one-line purpose, when it triggers, what it produces, what it requires.
4. If the skill changes the project on disk, add a migration entry to `../MIGRATIONS.md`.
5. If the skill exposes a new requirement (CLI, MCP, plugin), update `../README.md`'s requirements section.
