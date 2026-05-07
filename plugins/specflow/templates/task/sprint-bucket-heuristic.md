# Sprint-bucket assignment heuristic

When `specflow:task` Phase B writes `{NNN-slug}-tasks.md`, every per-task entry carries a `sprint-bucket: N` integer field — `N` is a positive integer ≥ 1. The field is derived deterministically from the dependency graph + scope-overlap; given the same input, two synthesis runs produce identical bucket assignments. The future `specflow:sprint` (020) consumes the field to plan a parallel fan-out batch.

Introduced in v2.5.0 (`025-sprint-task-flagging`).

## The single rule

```
bucket(T) = 1 if T has no predecessors AND no same-bucket scope conflicts with an earlier-ID task;
          = 1 + max(bucket(P) for P in Depends-on(T) ∪ EarlierIDSameBucketScopeConflicts(T)) otherwise.
```

Bucket 1 always means "no predecessors" within the feature. There is no bucket 0; setup tasks (if present) land in bucket 1.

## Canonical task ID grammar

Task IDs follow `T-<int>(\.<letter>)?` (e.g. `T-1`, `T-2`, `T-3.A`, `T-3.B` per 029's split flow). Task IDs are unique within a feature.

## Typed comparator

Same-bucket scope-overlap conflicts are resolved by a typed comparator on `(int, optional-letter)` over the ID's grammar — NOT raw string lexicographic.

- `T-2 < T-10` (natural-number ordering on the int part).
- `T-3.A < T-3.B` (letter ordering after equal int).

## Bump iteration discipline

Bumps applied iteratively until no same-bucket scope overlap remains. In each pass, walk same-bucket task pairs in `(bucket-asc, T-id-asc, T-id-asc)` order under the typed comparator and bump the later-ID partner: `bucket(T_j) := bucket(T_j) + 1`. Repeat passes until a pass produces zero bumps (fixed point). Termination is guaranteed: each bump strictly increases bucket numbers; the topological-floor invariant bounds the maximum.

## Topological-floor corollary

By construction: for every `T_j ∈ Depends-on(T_i)`, `bucket(T_j) < bucket(T_i)` — strictly less-than. The heuristic does NOT need a separate runtime invariant check.

## Per-task budget respect

Bucket assignment runs *after* per-task budget enforcement (029 R4). Tasks whose `context-budget-estimate` exceeds the configured budget (`config.task.contextBudget`, default 80,000 tokens) are split per 029 R4 BEFORE bucketing operates. The heuristic never sees an oversize task — by the time bucket assignment runs, every task is fit-for-develop.

## Graph validity (pre-bucket)

Before bucket assignment, `task/SKILL.md` Phase B.4 invokes a graph-validity step that rejects malformed dependency graphs with deterministic `GRAPH-INVALID:` diagnostics. Synthesis aborts before any `sprint-bucket: N` is written; the user is pointed to `specflow:scope-change` for legitimate dependency-graph edits.

| Failure mode | Diagnostic |
|---|---|
| Cycle (any strongly-connected component of size >1) | `GRAPH-INVALID: cycle: T-X -> T-Y -> ... -> T-X` (lists members in traversal order) |
| Self-loop (`T.depends_on` contains `T`) | `GRAPH-INVALID: self-loop on T-X` |
| Duplicate task IDs | `GRAPH-INVALID: duplicate task ID T-X` |
| Duplicate dependency edge | `GRAPH-INVALID: duplicate edge T-X -> T-Y` |
| Dangling reference | `GRAPH-INVALID: T-X.depends_on references unknown task T-Y` |

## Worked example

Four tasks. T-1 has no predecessors. T-2 depends on T-1. T-3 has no `Depends on:` but its `Scope:` overlaps T-1's `Scope:`. T-10 has no `Depends on:` and `Scope:` disjoint from T-1.

Bucket assignment (apply the rule + bump iteration):

| Task | Depends on | Scope overlap | Bucket | Reasoning |
|---|---|---|---|---|
| T-1  | (none)   | (none)        | 1 | No predecessors; no same-bucket scope conflict with an earlier-ID task. |
| T-2  | T-1      | (disjoint)    | 2 | `bucket(T-1) = 1`, so `bucket(T-2) = 1 + 1 = 2`. |
| T-3  | (none)   | T-1's surface | 2 | Same-bucket scope conflict with earlier-ID T-1; typed comparator `T-1 < T-3` (`1 < 3`); T-3 bumps to bucket 2. |
| T-10 | (none)   | (disjoint)    | 1 | No predecessors; no scope overlap with any earlier-ID task. Parallel to T-1. |

Note `T-2 < T-10` under the typed comparator (`2 < 10` on the int part), even though string lex would say `T-10 < T-2`. The bucket assignment for T-2 (bucket 2) and T-10 (bucket 1) is determined by the rule, not by the comparator — but pair-walks during the bump pass use the typed comparator to pick which partner bumps.

## Bucket count is uncapped

A feature with a deep dependency graph may legitimately have many buckets. The heuristic does not tune the bucket count; it accepts whatever the topology produces.

## Bucket numbers are feature-scoped

Two features whose tasks could run in parallel do NOT share bucket numbers. Cross-feature sprint planning is a 020 concern (parked under the existing parked-memory entry); bucket numbers in this skill are scoped to the feature.

## The field is documented as read-only

Editing `sprint-bucket: N` by hand changes the topological grouping and is a `specflow:scope-change` concern. `specflow:task` does NOT recompute-and-compare on every synthesis run to police hand-edits — the field's read-only status is documentation-level only.

## Cross-references

- **022 — cross-task review** — sprint-bucket is recomputed when the applier accepts a merge or split (per `templates/task/cross-task-review.md` § Sprint-bucket recompute).
- **029 — single-context-task** — per-task budget enforcement runs before bucketing; the bucket heuristic never sees oversize tasks.
- **020 — sprint-skill** (Sprint 4) — consumes `sprint-bucket: N` to plan a fan-out batch.
