# Task manifest — schema + read-first contract

A standardised per-task manifest at `debate-log/tasks/T-NN-manifest.md` accumulates entries from the moment the task is born (`specflow:task` synthesis) through to validation (`specflow:complete` retro). Replaces 017's interim `manifest-stub-{task-id}.md` scratch path.

Introduced in v2.6.0 (`019-task-manifest`).

## Read-first contract

Every agent invoked against a task **must read the manifest** before contributing. This is how agents see what others have already proposed, found, or decided. Per 027-reviewer-context-isolation, the manifest is part of the artefact-context handed to fresh-context reviewers; the writer's chat is not.

## Standardised entry format

One block per entry, append-only, in YAML-frontmatter style:

```
---
timestamp: 2026-05-07T12:34:56Z
agent_id: {role-name or principle name, per 027 format — e.g. "task-synthesiser", "goal-driven-reviewer", "implementer-yellow", "edge-case-reviewer", "verifier"}
phase: {task-creation | task-review | develop-red | develop-green | develop-refactor | test | gate-4 | gate-5 | complete}
event_type: {proposal | finding | decision | iteration-result | escalation | sign-off}
input_ref: {paths to artefacts the agent read — PRD, prior manifest entries, design folder, lessons.json query result}
output_ref: {paths/links to what the agent produced — code diff, test result, finding json}
body: |
  {the actual content — proposal text, finding details, decision rationale, iteration outcome}
outcome: {open | accepted | rejected | deferred-to-misc | superseded-by-T-NN-entry-M}
---
```

## Lifecycle phases captured

1. **task-creation** — synthesiser appends the initial task spec + the lessons.json query result that shaped it (per 018-lessons-registry).
2. **task-review** (Gate 3) — Gate 3 reviewers (per-task lens AND cross-task lens from 022) append findings; orchestrator appends the closing decision.
3. **develop-red / develop-green / develop-refactor** — per Pocock's framing from 017-tdd-discipline. Each sub-phase logs the test written (Red), the minimal code (Green), the structural improvement (Refactor). Replaces 017's interim `manifest-stub-{task-id}.md`.
4. **gate-4** (plan-vs-PRD) + **gate-5** (code-vs-plan) — principle reviewers + edge-case-reviewer (028) append findings with `recommendation` + `reasoning`; orchestrator appends accept/reject/defer per finding.
5. **test** — `specflow:test` Phase D appends AC pass/fail + brand-consistency lens findings (per 023).
6. **complete** — `specflow:complete` retro appends the final sign-off + the lessons.json entries this task generated.

## File location

`docs/specflow/features/{NNN-slug}/debate-log/tasks/T-NN-manifest.md` — one file per task, opened at synthesis (`specflow:task` Phase E.6 closer creates the file with the `task-creation` entry), closed at completion (`specflow:complete` retro appends the `sign-off` entry).

## Migration from 017's stub

When 019 ships, the existing `admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md` interim format is migrated as follows:

- The three-line marker block (`red:` / `green:` / `refactor:`) is converted to three entries with `phase: develop-red | develop-green | develop-refactor` and `event_type: iteration-result`. The outcome value (`passed | failed | skipped (config) | skipped (trivial) | skipped (human-led Red lane) | failed (new-file-attempted)`) lands in the entry body's first line.
- The `agent_id` field on each migrated entry is `implementer-{lane}` (e.g. `implementer-yellow`, `implementer-green`).
- Migration is one-way: 019's manifest format is canonical going forward; 017's stub becomes legacy noise on existing features. New features land directly in 019's format.

## Cross-feature integration

- **017 (TDD)** — Red/Green/Refactor markers land as `phase` values on develop entries.
- **018 (lessons-registry)** — task-creation entries cite the lessons that shaped the task; complete entries cite the lessons being appended.
- **022 (cross-task review)** — cross-task findings are entries in EVERY affected task's manifest (not a separate doc).
- **027 (reviewer isolation)** — each entry's `agent_id` is the writer/reviewer ID; isolation is auditable from the manifest alone. The manifest's input_ref + output_ref fields are the artefact-context-only declared inputs reviewers see.
- **028 (edge-case reviewer)** — its `recommendation` + `reasoning` payload is the entry body; orchestrator's accept/reject decision is the next entry's `body` referencing it.

## Read-first enforcement

Every agent's role-def file under `admin/agents/standard/{lifecycle,principles}/` is updated to include the line:

> *"Before contributing, read the per-task manifest at `debate-log/tasks/T-NN-manifest.md` for this task. Your contribution becomes the next entry; your `agent_id` and `input_ref` fields document what you read."*

The orchestrator's pre-dispatch step verifies the manifest exists; on missing manifest, refuses with: *"Per-task manifest at `debate-log/tasks/T-NN-manifest.md` does not exist. Run `specflow:task {NNN-slug}` to open the manifest at synthesis."*

## Cross-references

- **017 — tdd-discipline** — interim stub format that 019 replaces.
- **018 — lessons-registry** — task-creation + complete entries cite lessons.
- **022 — cross-task-review** — cross-task findings land as entries in affected manifests.
- **027 — reviewer-context-isolation** — manifest is the declared-input context for fresh-context reviewers.
- **028 — edge-case-reviewer** — recommendation + reasoning land as entry body.
