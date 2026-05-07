# Fixture — full Round 2.5 flow with both lenses

A 4-task feature where the cross-task reviewer surfaces one coherence finding and one better-arrangement finding; the applier accepts the first, rejects the second; the reviewer sharpens the second; the applier accepts the sharpened version.

## Setup — pre-Round-2.5 state

`{NNN-slug}-tasks.md` (post-orchestrator-Round-2):

```
### T1 — Add unread badge to header
- Anchor: PRD R3 — header surfaces unread count
- Scope: src/header/Badge.tsx; src/header/Header.tsx
- Acceptance: AC-1 — badge renders count when notifications exist
- context-budget-estimate: 35000

### T2 — Persist read-state via API
- Anchor: PRD R4 — clicking a notification marks it read
- Scope: src/header/NotificationItem.tsx; src/api/notifications.ts
- Acceptance: AC-2 — server marks read on click
- context-budget-estimate: 28000

### T3 — Optimistic update on click
- Anchor: PRD R4 — clicking a notification marks it read
- Scope: src/header/NotificationItem.tsx; src/state/notifications.ts
- Acceptance: AC-2 — UI reflects read state before server response
- context-budget-estimate: 25000

### T4 — Add formatter and tests for badge unread count
- Anchor: PRD R3 — header surfaces unread count
- Scope: src/header/lib/format-count.ts; __tests__/format-count.test.ts
- Acceptance: AC-1 — count > 99 shows "99+"
- context-budget-estimate: 22000
```

## E.4.5.1 — Cross-task R1 fire

`debate-log/tasks-gate3/findings/round-1/cross-task-reviewer.json`:

```json
{
  "writer_id": "writer-2026-05-07T11:42:00Z-a3",
  "findings": [
    {
      "id": "cross-task-r1-f1",
      "lens": "coherence",
      "severity": "concern",
      "claim": "T2 and T3 both anchor R4 (clicking marks read) with overlapping scope and the same AC-2 — they are operationally one concept (mark-read with optimistic UI), split unnecessarily.",
      "evidence": "T2.Scope and T3.Scope both touch src/header/NotificationItem.tsx; both anchor R4; both cite AC-2. T2 (28K) + T3 (25K) = 53K combined estimate, well within the 80K cap.",
      "proposed_change": "Merge T2 and T3 into a new T2 covering API-call + optimistic-state under R4/AC-2."
    },
    {
      "id": "cross-task-r1-f2",
      "lens": "better-arrangement",
      "severity": "concern",
      "claim": "T1 has weak coherence — it bundles 'add Badge component' and 'wire to Header'. Splitting would let T1a ship as a pure component (testable in isolation) and T1b as the integration step.",
      "evidence": "T1.Scope spans src/header/Badge.tsx (new component) AND src/header/Header.tsx (integration site). The two surfaces have different reviewable boundaries.",
      "proposed_change": "Split T1 into T1a (Badge component, isolated) and T1b (Header integration)."
    }
  ]
}
```

## E.4.5.2 — Applier R2 response + apply

`debate-log/tasks-gate3/findings/round-2/cross-task-responses.json`:

```json
{
  "round": 2,
  "applier_id": "applier-2026-05-07T11:46:18Z-b1",
  "responses": {
    "cross-task-r1-f1": {
      "decision": "accepted",
      "revision_applied": "Merged T2 + T3 into a new T2 (api-call + optimistic-state under R4/AC-2). Combined estimate 53K within cap. Sprint-bucket recomputed: both source tasks bucket=1; merged task bucket=1 (no graph drift).",
      "bucket_audit": {"T2": {"before": 1, "after": 1}, "T3": {"before": 1, "after": null}}
    },
    "cross-task-r1-f2": {
      "decision": "rejected",
      "rationale": "Splitting T1 inflates the dependency graph (T1b depends on T1a) without a clear blast-radius win. The Badge component is small (single file, ~40 lines); shipping it without integration is itself an artefact ambiguity. Keeping T1 monolithic preserves the single-context window per 029."
    }
  }
}
```

Applier writes the merged T2 to tasks.md; T3 is removed; T4 renumbers stay or remain stable depending on bucket convention. Pre-applier coverage matrix re-verified post-edit.

## E.4.5.3 — Cross-task R3 sharpen

The reviewer sharpens its rejected finding with new evidence:

`debate-log/tasks-gate3/findings/round-3/cross-task-reviewer.json`:

```json
{
  "findings": [
    {
      "id": "cross-task-r3-f2",
      "supersedes": "cross-task-r1-f2",
      "lens": "better-arrangement",
      "severity": "concern",
      "claim": "Re-sharpening: the issue is not that T1 is large but that it bundles a NEW pattern (the Badge component) with an EXISTING pattern (Header integration). New-pattern tasks benefit from isolated review per Pocock's issue-sizing-heuristic — the reviewer can attend to the pattern's edges without the integration noise. T1a (~25K, isolated, reviewer attends to component contract) + T1b (~12K, trivial integration) is a strictly better arrangement than T1 (~35K, mixed concerns).",
      "evidence": "Pattern-introduction is the higher-cost-per-line surface. Mixing it with trivial integration buries the review attention.",
      "proposed_change": "Re-propose the split. The split successors each fit the budget; bucket recompute is trivial (both stay in bucket 1)."
    }
  ]
}
```

## E.4.5.4 — Applier final pass

`debate-log/tasks-gate3/findings/round-2/cross-task-responses.json` (appended):

```json
{
  "cross-task-r3-f2": {
    "decision": "accepted",
    "revision_applied": "Split T1 into T1a (Badge component, src/header/Badge.tsx, est 25K) and T1b (Header integration, src/header/Header.tsx, est 12K). T1b depends on T1a. Bucket recompute: T1a=1, T1b=1; no graph drift.",
    "bucket_audit": {"T1a": {"before": null, "after": 1}, "T1b": {"before": null, "after": 1}}
  }
}
```

## Closing manifest excerpt

`debate-log/tasks-gate3/manifest.md`:

```
**writer_id:** writer-2026-05-07T11:42:00Z-a3
**cross_task_reviewer_id:** ctr-2026-05-07T11:43:55Z-x7
**applier_id:** applier-2026-05-07T11:46:18Z-b1

## Cross-task findings

### Accepted findings
- **cross-task-r1-f1** (cross-task-reviewer, lens: coherence, concern) — T2 and T3 both anchor R4 with overlapping scope.
  - Evidence: Both touch src/header/NotificationItem.tsx; both anchor R4; both cite AC-2.
  - Revision applied: Merged T2 + T3 into new T2; bucket recompute audited.
- **cross-task-r3-f2** (cross-task-reviewer, lens: better-arrangement, concern) — T1 bundles new-pattern + existing-integration; isolated review preferred.
  - Evidence: Pattern-introduction is the higher-cost-per-line surface.
  - Revision applied: Split T1 into T1a + T1b; both fit budget; bucket recompute trivial.

### Rejected findings
(none — both findings ultimately accepted)

### Escalated to human
(none)

## Closing decision

Gate 3 status: **passed-with-revisions**

Per-task PASSED (all five reviewers converged in Round 3 against the post-applier tasks.md). Cross-task PASSED-WITH-REVISIONS (two findings accepted; tasks.md edited; coverage and budget preserved). All three agent_id fields populated and pairwise distinct (writer / reviewer / applier).

— Orchestrator, 2026-05-07
```

## Audit signal

- Both lenses represented (coherence + better-arrangement).
- Both findings accepted (one immediately, one after sharpen).
- Three `agent_id` values populated and distinct (writer / cross-task-reviewer / applier).
- Sprint-bucket audit recorded for every accepted edit.
- No `scope-change-required` findings — coverage matrix preserved.
