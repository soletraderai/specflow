# Lessons registry — schema + read-write loop

`admin/lessons.json` is the project's self-learning corpus. A single mutable JSON array of lesson entries, written by `specflow:test --feedback` (Phase D) and `specflow:complete` (retro), read by `specflow:prd` (Phase A.4) and `specflow:task` (Phase A.4) at the start of every new feature, and clustered by `specflow:insights` for promotion to `admin/rules/guidelines.md`.

Introduced in v2.6.0 (`018-lessons-registry`). Formalises the in-flight write log as a closed read-write loop.

## Schema

Each lesson entry is one JSON object in the top-level array:

```json
{
  "id": "L-001",
  "created": "2026-05-08T12:34:56Z",
  "tags": ["auth", "migration"],
  "surface": ["api", "data-model"],
  "outcome": "worked | failed | mixed",
  "context": "1-2 sentence description of what was being attempted",
  "lesson": "the load-bearing claim — what to do or avoid",
  "source": {
    "feature_id": "032-notif-badge",
    "retro_link": "features/032-notif-badge/032-notif-badge-retro.md#L7",
    "phase": "complete | test-feedback"
  },
  "confidence": "single-occurrence | repeated | validated",
  "superseded_by": null,
  "status": "active | superseded"
}
```

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | `L-NNN` (zero-padded, monotonically incrementing per-project) |
| `created` | ISO-8601 datetime | yes | When the lesson was written |
| `tags` | string[] | yes | Free tag vocabulary — domain + audience + tools (e.g. `auth`, `migration`, `mobile`, `playwright`) |
| `surface` | string[] | yes | Canonical surface vocabulary — `ui | data-model | api | auth | migration | infra | cli | docs | prd-shape | task-decomposition` |
| `outcome` | enum | yes | `worked` (what to repeat) / `failed` (what to avoid) / `mixed` (judgement-dependent) |
| `context` | string | yes | 1-2 sentences — what was being attempted when the lesson surfaced |
| `lesson` | string | yes | The load-bearing claim. One sentence ideally; never more than three. |
| `source` | object | yes | Where the lesson came from — feature ID, retro link, originating phase |
| `confidence` | enum | yes | `single-occurrence` (one event), `repeated` (≥3 occurrences clustered by insights), `validated` (promoted to `guidelines.md`) |
| `superseded_by` | string \| null | optional | If a later lesson supersedes this one, the later lesson's `id` lands here |
| `status` | enum | yes | `active` (read by skills) / `superseded` (filtered out by query) |

## Write path

### Path 1 — `specflow:test --feedback` (Phase D)

When a Verifier-passed task turns out wrong on human review, `specflow:test --feedback` captures the gap as a lesson with `outcome: "failed"` (or `"mixed"` if the gap is judgement-dependent) and `confidence: "single-occurrence"`. The captured `what_was_missed` becomes `context`; the user-stated remediation becomes `lesson`; the misattribution map becomes `source`. Operational details: `skills/test/SKILL.md` § Phase D — Feedback mode.

### Path 2 — `specflow:complete` (retro)

At feature close, `specflow:complete` retro prompts the user for what worked + what failed in this feature. Each surfaced item becomes a lesson with the appropriate `outcome` and `confidence: "single-occurrence"`. The retro file becomes the `source.retro_link`.

### Path 3 — `specflow:insights` (clustering)

`specflow:insights` reads `lessons.json`, clusters by tag + outcome, and promotes ≥3-occurrence clusters to `admin/rules/guidelines.md`. The promotion flow updates each contributing lesson's `confidence` to `repeated` (and to `validated` if the resulting guideline is accepted by the user).

## Read path

### Path 1 — `specflow:prd` Phase A.4

After codebase context (A.3) and before requirements drafting (B), compute the feature's tag profile from the interview, design folder, and linked Linear issues. Query `lessons.json` by tag-match + recency + confidence weighting. Surface the top N (default 5; configurable via `config.json.prd.maxLessonsSurfaced`) inline in the interview as a **"What we've learned before that applies here"** section so the user sees what's being pulled and can correct/dismiss any non-applicable ones. Lessons that survive sign-off propagate into Phase B as constraints.

### Path 2 — `specflow:task` Phase A.4

When synthesising tasks, re-query `lessons.json` with the now-finalised PRD's tag profile. Each task that touches a tagged surface gets a `prior-lessons: [id1, id2]` field linking the lessons that shaped it. Influences task shape (e.g. *"for migration tasks, prior runs flagged data-shape mismatch — task synthesis must include a pre-migration validation step"*) and ordering (e.g. *"for refactors of legacy modules, seam-cut task must precede behavioural change task"*).

### Path 3 — Cross-task review (022) 

The cross-task reviewer's better-arrangement lens consults lessons too — its tag-based query suggests re-orderings prior runs proved valuable.

### Path 4 — `specflow:develop` does NOT query lessons directly

By design. Lessons must influence the *plan* upstream (PRD + tasks); develop executes the plan. This keeps the develop stage Claude-Code-native and avoids cross-context contamination during code execution.

## Query algorithm

Inputs: `{tags: string[], surfaces: string[], outcome_filter?, recency_weeks?: int, confidence_min?: 'single-occurrence' | 'repeated' | 'validated'}`.

Defaults: `recency_weeks: 26` (≈2 quarters), `confidence_min: 'single-occurrence'`, no outcome filter (both worked + failed are useful at PRD time; only `failed` at task time when checking for blockers).

Scoring: `tag_overlap_count × confidence_weight × recency_decay`, where:

- `tag_overlap_count` is the count of tags shared between the query and the lesson.
- `confidence_weight` is `1.0 | 1.5 | 2.0` for `single-occurrence | repeated | validated`.
- `recency_decay` is `exp(-weeks_since_created / recency_weeks)`.

Top N by score are returned with full body for prompt inclusion. Superseded lessons are filtered out unless explicitly requested (`include_superseded: true`).

## Config knobs

`config.json.prd.maxLessonsSurfaced` — integer, default 5. Caps the lessons surfaced inline at PRD time.

`config.json.task.maxLessonsSurfaced` — integer, default 5. Caps the lessons surfaced at task time.

Both knobs default to the same value; lower for token-sensitive providers; raise cautiously.

## Cross-references

- **022 — cross-task-review** — the better-arrangement lens consults lessons.
- **019 — task-manifest** — `task-creation` entries cite the lessons that shaped the task; `complete` entries cite the lessons being appended.
- **`specflow:insights`** — the clustering + promotion path.
- **`admin/rules/guidelines.md`** — promotion target for repeated lessons.
