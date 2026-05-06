# Feature Model — Design Notes

Working design for how features are captured, tracked, and built in specflow. Captured during planning conversation on 2026-05-06. Not yet implemented.

## Underlying intent

- Every feature has a **goal**. Everything downstream (PRD, tasks, tests) works toward that goal.
- Maximum context preservation across sessions: an agent or human starting cold should find everything they need.
- Machine-readable where it matters; narrative where humans read.
- Cheap to populate, low friction. No over-engineered templates.

## Per-feature directory layout

Each feature lives in `plugins/specflow/examples/docs/specflow/features/{nnn}-{name}/` and contains:

| File | Purpose | Format |
| --- | --- | --- |
| `manifest.json` | Structured metadata. Single source of truth. | JSON |
| `transcript.json` | Verbatim record of all conversation across feature lifecycle. Agent-populated. | JSON, sessioned |
| `interview.md` | Curated interview only (existing convention). | Markdown |
| `prd.md` | Problem statement + PRD body. Opens by restating the goal so it reads complete on its own. | Markdown |
| `tasks.md` | Task breakdown. | Markdown |
| `debate-log/` | Existing decision/debate record. | Folder |

### `manifest.json` shape

```json
{
  "schema_version": "1",
  "id": "006",
  "name": "feature-model",
  "goal": "One paragraph max. What we are trying to achieve.",
  "references": [
    { "url": "https://...", "note": "context", "tags": ["api"] },
    { "url": "https://...", "note": "inspiration site", "tags": ["inspiration"] }
  ],
  "status": "active"
}
```

- **`goal`**: paragraph max. Narrative context — e.g. "convert more paying users", "speed up the framework", "better UI". Provides the lens for all downstream work.
- **`references`**: flat array. Optional freeform `tags` (e.g. `api`, `inspiration`, `docs`, `slack`, `screenshot`). Avoids hardcoded buckets that churn when a new reference type doesn't fit.
- **`status`**: `active | blocked | shipped | parked`. Sub-states (in discovery vs. PRD vs. building) inferable from which files exist; don't re-encode.

### `transcript.json` shape

Sessioned to keep size bounded. Agents load just the latest session unless they need history.

```json
{
  "schema_version": "1",
  "sessions": [
    {
      "session_id": "...",
      "started_at": "2026-05-06T...",
      "ended_at": "2026-05-06T...",
      "turns": [
        { "role": "user", "text": "..." },
        { "role": "assistant", "text": "..." }
      ]
    }
  ]
}
```

### Goal mutability

Goal can sharpen as discovery deepens. Manifest holds the *current* goal as a plain string. When the goal changes, append a one-liner to `prd.md` explaining the sharpening. Manifest stays clean; rationale stays findable without parsing the verbatim transcript.

## System-level manifest

Location: `plugins/specflow/examples/docs/specflow/manifest.json` (sits next to `features/`).

```json
{
  "schema_version": "1",
  "active_feature": "006",
  "features": [
    { "id": "001", "name": "design-skill", "status": "shipped", "path": "features/001-design-skill" },
    { "id": "002", "name": "develop-skill", "status": "shipped", "path": "features/002-develop-skill" }
  ]
}
```

- No `next_id` counter. Derive next id from `max(features.id) + 1` to avoid write-contention between concurrent agents.
- Index gives agents O(1) feature lookup without scanning directories.

## Backfill plan for 001–005

- **`manifest.json`** — write full manifest for each. Extract `goal` from existing interview/PRD docs. Set `status` based on current state (most are `shipped`).
- **`transcript.json`** — stub with `{ "schema_version": "1", "legacy": true, "note": "predates verbatim transcript capture", "see_also": ["interview.md", "debate-log/"] }`. Verbatim conversations from past sessions are not recoverable; stub keeps directory shape consistent.

## Adversarial review (Codex)

Specflow already has a multi-reviewer adversarial framework: gates (`prd-gate2`, `tasks-gate3`, `develop-gate5`) run rounds with parallel reviewers writing JSON findings. `codex-reviewer` already exists but is currently active only at `develop-gate5` in some features. Goal: make Codex review standard at every meaningful gate.

### Where Codex runs

- **PRD gate** — Codex added to the standard reviewer lineup. Runs in parallel with existing same-model reviewers (devils-advocate, goal-driven-reviewer, etc.). PRD findings are the highest-leverage in the pipeline (every downstream gate inherits the frame), so the orchestrator's synthesis explicitly calls out Codex as the cross-provider lens — not fungible with same-model reviewers.
- **Task gate** — Codex reviews the **whole task list per round**, not per-task. The failure modes Codex catches are cross-task by construction (missing tasks, wrong ordering, duplicated coverage, AC-without-task). A per-task review cannot produce "AC-7 has no task."
- **Develop gate** — already integrated (existing).

### What Codex sees as input

- At PRD gate: PRD draft + interview log. Without the interview, Codex can't catch "you said X in interview but PRD says Y."
- At task gate: PRD + task list + checkpoint structure (see Checkpoints below).

### Stopping rule

Up to 3 rounds per gate. Stop when:

1. Codex returns no new substantive findings, OR
2. Round 3 reached.

**Critical:** if Claude rejects all of Codex's feedback in a round AND Codex re-fires the same findings in the next round → escalate to human review. Don't let Claude self-certify against cross-provider dissent. "Claude rejected everything" is *not* a convergence signal on its own.

### Failure mode

If Codex unavailable: gate proceeds with remaining reviewers; reuse the existing degraded-coverage sentinel pattern from `skills/develop/SKILL.md` (consistency with current convention). Codex is a quality multiplier, not a blocking gate.

### Auto-promotion behavior

The develop gate has Codex-only finding auto-promotion to `specflow:misc` for rule-layer changes. **Skip this at PRD/task gates** — those findings are feature-specific, not code-layer rules.

### Disagreement weighting (deferred)

When Codex disagrees with same-model reviewers, no tie-breaker rule yet. Log disagreements across the next several features and calibrate empirically before baking in a weighting.

### No new artifact formats

Codex findings land in the existing `debate-log/{gate}/findings/round-N/codex-reviewer.json` path. No new directories, no new file formats.

## Checkpoints (HITL pacing)

**Principle:** the unit of work between human verification points. Not a throughput optimization; a pacing primitive. Every checkpoint is a human verification gate. The human is always the slowest reviewer — design around that, not against it.

Terminology note: "checkpoint" deliberately, not "sprint." Avoids Scrum baggage (cadence, retros, planning ceremony) and disambiguates from Linear cycles or PRD-level milestones.

### When checkpoints are created

At **task gate**, alongside the task list — not at develop. Checkpoint boundaries reveal hidden cross-task dependencies that change the task list itself (split T7, merge T3+T4). Discovering that at develop time corrupts a sealed artefact and forces a round-trip back through task gate. Do it once, with Codex reviewing tasks + sequencing as a single artefact.

No new gate. Add a checkpoint lens to the existing goal-driven-reviewer at task gate: *"checkpoint boundaries deliver coherent goal-progress slices; no checkpoint depends on a later one."*

### Checkpoint shape

Each checkpoint = a named group of tasks with:

- A one-line **deliverable** — what does merging this checkpoint *prove* about goal progress?
- An **`Anchor:`** line tracing to a PRD goal (not a requirement). The deliverable is what proves goal progress.
- Member tasks (subset of the task list).
- Implicit ordering: tasks in checkpoint N must not depend on tasks in checkpoint N+1.
- **Sized for human review tractability** — small enough that one human can verify the work in a single sitting. Not by task count; by reviewability.

### Checkpoint location

Headers in `tasks.md` above the grouped tasks (single source for tasks + grouping). Mirror in `manifest.json` for machine-readable iteration:

```json
"checkpoints": [
  { "name": "...", "deliverable": "...", "anchor": "goal: ...", "task_ids": ["T1", "T2"] }
]
```

### Develop runs per checkpoint

`develop-gate5` fires **once per checkpoint completion**, not once for the whole feature. After each checkpoint:

1. Agent develops the checkpoint's tasks.
2. Codex + same-model reviewers run on the checkpoint as a unit.
3. **Human verifies and signs off.** No auto-flow to the next checkpoint.

### Rollback unit

The whole checkpoint, by construction. Partial rollback breaks the demoable contract. Failed gate → revert the checkpoint's branch, revise the failing task(s), re-run develop on the checkpoint.

### Mid-feature replanning

When checkpoint N reveals checkpoint N+2 is wrong, route through an explicit `specflow:scope-change --checkpoint` path. **Silent edits to `tasks.md` mid-run are forbidden** (existing skill convention).

### Escape hatch

Small features (≤3 tasks) may use a single checkpoint or skip the primitive entirely. Checkpoints are a tool, not mandatory ceremony.

## Open / deferred

- Plugin-wide reference links — skipped for now (YAGNI).
- Whether goal-sharpening rationale lives in `prd.md` (current decision) or a new `decisions.md` (rejected for now to avoid adding another file format).
- Codex-vs-same-model disagreement weighting — log empirically across several features before baking in.
- Lane interaction with checkpoints (Red/Green HITL gating) — moot under HITL-pacing principle: every checkpoint is a human gate regardless of lane mix.

## What this is not

- Not a full PRD for the feature-model feature itself. This is the working design captured for continuity. When implementation starts, this gets converted into a proper feature directory under `features/006-feature-model/` (or similar) using the very model it describes.
