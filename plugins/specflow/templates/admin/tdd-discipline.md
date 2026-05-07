# TDD discipline — Red → Green → Refactor

When `specflow:develop` enters a task's lane execution (Phase D), the agent's work inside the task's single context window (per 029-single-context-task) follows Pocock's **Red → Green → Refactor** cycle. The cycle is *internal* to the task — it does NOT replace or rename the lane sub-phases (D.1 Green-batch / D.2 Yellow-HITL / D.3 Red-human-led). Cycle steps are documented here, not phase-numbered in the skill body.

Introduced in v2.5.0 (`017-tdd-discipline`). Codifies the failing-test-first discipline.

## Why this exists

Two converging signals from the research dataset:

- **Pocock's TDD framing** (`knowledge/pocock-software-fundamentals-matter-more.md` § Failure Mode 3) — the canonical Red/Green/Refactor definition with the load-bearing observation: *"TDD forces the LLM to really take small steps."* Without TDD, AI agents routinely cheat the test-write step by writing the test *after* the implementation, against the implementation's actual behaviour, producing tests that are tautological with the code (`knowledge/pocock-ai-coding-real-engineers.md` § Phase 5 names this verbatim).
- **The smart-zone cliff** (`knowledge/pocock-ai-coding-real-engineers.md` § LLM Constraint 1) — taking small steps keeps the agent inside the smart zone for the duration of the cycle. Big-bang implementations push the agent through the cliff before the test exists to catch the drift.

Without the cycle, develop's lane execution drifts into the failure mode this skill exists to remove: implementation-first, test-after, with the test encoding what the code does rather than what it should do.

## The cycle

### Red

1. Author the failing test against the task's primary acceptance criterion. The test name and assertion must reference the AC's binary check by ID (e.g. `// AC-3: badge shows unread count`).
2. Run the targeted test command (e.g. `vitest run path/to/test`) against the **pre-implementation state**. Capture the failing exit + stderr to `admin/scratch/{NNN-slug}-develop/red-test-trace-{task-id}.log`.
3. **A pre-implementation pass blocks as an invalid Red artefact.** The test was written after the implementation (the Pocock failure mode) — refuse to enter Green; halt and surface to the developer.
4. Append the cycle marker to the per-task manifest stub:
   ```
   red: passed (2026-05-07T12:34:56Z) — failing test authored at __tests__/badge.test.ts; targeted vitest exit 1.
   ```
5. **Yellow lane** refuses to enter Green without the Red artefact. **Green lane** with `config.develop.tddRequired: true` (the default) behaves identically to Yellow. **Green lane** with `tddRequired: false` may skip — manifest records `red: skipped (config) (...)`. **Red lane** (human-led) — the human supplies the Red artefact OR manifest records `red: skipped (human-led Red lane) (...)` and **no `green: passed` marker is emitted by the skill** (audit signal stays honest).

### Green

1. Write the simplest change that makes the failing test pass. No premature abstraction; no incidental refactors; no new files unrelated to the test.
2. Re-run the same targeted test command. The exit-0 result must come from the SAME command captured at Red. Append to the trace log.
3. Append the cycle marker:
   ```
   green: passed (2026-05-07T12:38:11Z) — minimal implementation at src/header/Badge.tsx; targeted vitest exit 0.
   ```
4. Failed Green (test still red after the implementation attempt) — agent escalates to the developer per the develop Phase D failure-mode contract; never silent re-implement.

### Refactor

Refactor is bounded structural improvement under the green test as guard. Three explicit bounds:

- no new behaviour — every test that passed at Green still passes at Refactor end (re-run the trace command; exit 0 still)
- no new files — Refactor only modifies files Green touched
- no scope creep / route to `specflow:scope-change` — if Refactor wants to add a file or change scope, the agent halts with the literal message *"Refactor cannot add files; route to specflow:scope-change"* and routes to the scope-change skill

Refactor is optional for trivial tasks (one-line changes that do not benefit from a structural pass). When omitted, manifest records `refactor: skipped (trivial) (...)`. Violations are recorded post-hoc as `refactor: failed (new-file-attempted) (...)` (or analogous) — no runtime file-set diff detection.

## Marker schema

Per-task manifest stub at `admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md` (one file per task) carries three markers in this exact shape:

```
red: {outcome} ({YYYY-MM-DDTHH:MM:SSZ}) — {one-line note}
green: {outcome} ({YYYY-MM-DDTHH:MM:SSZ}) — {one-line note}
refactor: {outcome} ({YYYY-MM-DDTHH:MM:SSZ}) — {one-line note}
```

Outcome enum: `passed | failed | skipped (config) | skipped (trivial) | skipped (human-led Red lane) | failed (new-file-attempted)`. Markers append in cycle order.

**Schema ownership:** 017 owns *only* this interim stub contract — semantic fields (`red` / `green` / `refactor`), outcome enum, ISO timestamp, one-line note. **019-task-manifest owns the canonical per-task manifest schema** and provides a migration adapter from this stub when 019 lands; 019 may transform the stub into its own format. The stub path lives in scratch (NOT in `{NNN-slug}-tasks.md`) so it does not broadcast into every downstream skill that reads tasks.md.

## Lane interactions

| Lane | TDD enforcement | Red marker | Green marker | Refactor marker |
|---|---|---|---|---|
| Yellow (paired-HITL) | mandatory | `passed` (always) | `passed` after Red `passed` | `passed` or `skipped (trivial)` |
| Green with `tddRequired: true` (default) | mandatory | `passed` | `passed` | `passed` or `skipped (trivial)` |
| Green with `tddRequired: false` | optional | `skipped (config)` | `passed` | `passed` or `skipped (trivial)` |
| Red (human-led) | not enforced by skill | `skipped (human-led Red lane)` (when AI assists) | not emitted by skill | not emitted by skill |

## How `specflow:test --plan-only --task T{N}` slots in

The Red artefact is the per-task plan section in `{NNN-slug}-test.md`. Develop invokes `specflow:test {NNN-slug} --plan-only --task T{N}` at task entry; the test skill writes the per-task plan section with the primary AC's case marked `Status: red (failing)` by default. The `--task T{N}` co-flag SKIPS Phase B.5 (Codex pass + user prompt) — the per-task slice inherits whatever B.5 outcome the feature-level test plan recorded; Gate 5's Codex reviewer covers cross-provider concerns at code-vs-plan time.

## Pre-implementation test execution

After the Red sub-step authors the test, the targeted command runs against the pre-implementation state. The failing exit + stderr is captured to `admin/scratch/{NNN-slug}-develop/red-test-trace-{task-id}.log`. Green entry is gated on this trace existing. Green appends its exit-0 line to the same log so the cycle's whole arc is auditable in one file.

## Phase E (Gate 5) interaction

Phase E.3 reviewers see only the code diff, the per-task plan, the tasks-file entry, and their own role definition — they do NOT see the manifest stub or the cycle markers. The cycle is retro-audit, not in-gate review (per 027 fresh-context: cycle structure does not leak into dumb-zone review).

## Cross-references

- **029 — single-context-task** — Red → Green → Refactor runs inside the task's single context window.
- **019 — task-manifest** — owns the canonical per-task manifest; 017 owns the interim stub.
- **022 — cross-task review** — the "better arrangement" lens may re-cut tasks when Refactor's new-file-attempted block fires repeatedly.
- **027 — reviewer-context-isolation** — Phase E reviewers see only the artefact-context input list, never the cycle markers.

## Worked example

See `examples/docs/specflow/features/017-tdd-discipline/`.
