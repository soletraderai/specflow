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

## Open / deferred

- Plugin-wide reference links — skipped for now (YAGNI).
- Whether goal-sharpening rationale lives in `prd.md` (current decision) or a new `decisions.md` (rejected for now to avoid adding another file format).

## What this is not

- Not a full PRD for the feature-model feature itself. This is the working design captured for continuity. When implementation starts, this gets converted into a proper feature directory under `features/006-feature-model/` (or similar) using the very model it describes.
