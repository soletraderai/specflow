# Fixture — four-task bucketing (worked example)

Demonstrates the heuristic on a four-task feature where one task gets bumped due to same-bucket scope overlap with an earlier-ID task.

## Pre-bucketing tasks

```
### T-1 — Add Badge component
- Scope: src/header/Badge.tsx
- Depends on: none
- context-budget-estimate: 25000

### T-2 — Wire Badge into Header
- Scope: src/header/Header.tsx
- Depends on: T-1
- context-budget-estimate: 22000

### T-3 — Add unread-count formatter
- Scope: src/header/Badge.tsx; src/header/lib/format-count.ts
- Depends on: none
- context-budget-estimate: 30000

### T-10 — Add notification API endpoint
- Scope: src/api/notifications.ts; src/api/__tests__/notifications.test.ts
- Depends on: none
- context-budget-estimate: 32000
```

## Heuristic application

| Task | Predecessors | Earlier-ID same-bucket scope conflict | Bucket | Reasoning |
|---|---|---|---|---|
| T-1  | none | none (T-1 is the earliest ID) | 1 | No predecessors; no earlier-ID conflict. |
| T-2  | T-1  | n/a | 2 | `bucket(T-1) = 1`, so `bucket(T-2) = 1 + 1 = 2`. |
| T-3  | none | T-1 (`src/header/Badge.tsx` overlap) | 2 | Bumped from 1 → 2 because T-3.Scope overlaps T-1.Scope; typed comparator says `T-1 < T-3` (`1 < 3`); T-3 (the later-ID partner) bumps. |
| T-10 | none | none (no overlap with T-1, T-2, or T-3) | 1 | No predecessors; disjoint Scope. Parallel to T-1. |

## Post-bucketing tasks

```
### T-1 — Add Badge component
- sprint-bucket: 1

### T-2 — Wire Badge into Header
- sprint-bucket: 2

### T-3 — Add unread-count formatter
- sprint-bucket: 2

### T-10 — Add notification API endpoint
- sprint-bucket: 1
```

## Typed comparator demonstration

The comparator orders `T-2 < T-10` (natural-number ordering on the int part), even though string lexicographic ordering would say `T-10 < T-2`. The comparator only matters for picking which partner bumps in same-bucket scope-overlap pair-walks; the bucket numbers themselves are determined by the rule.

## Audit signal

- All four tasks fit budget (max estimate 32K, well under 80K cap).
- Topological floor holds: T-2's `sprint-bucket: 2` > T-1's `sprint-bucket: 1`.
- Bucket 1 contains tasks that can run in parallel (T-1 + T-10); bucket 2 contains tasks that must wait for bucket 1 to finish (T-2 + T-3).
- 020-sprint-skill (when it ships in Sprint 4) reads these buckets to plan a fan-out batch without recomputing the topology.
