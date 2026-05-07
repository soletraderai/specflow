# Fixture — Yellow lane happy path

Task: T3 — Add notification badge unread-count rendering.
Lane: Yellow (paired-HITL; verifiability=high, blast=medium, deps=satisfied, conf=non-confidential).

## Per-task manifest stub

Located at `admin/scratch/032-notif-badge-develop/manifest-stub-T3.md`:

```
red: passed (2026-05-07T12:34:56Z) — failing test authored at __tests__/Badge.test.tsx; targeted vitest exit 1.
green: passed (2026-05-07T12:38:11Z) — minimal implementation at src/header/Badge.tsx returning the unread-count span; targeted vitest exit 0.
refactor: passed (2026-05-07T12:41:02Z) — extracted unread-count formatter into the existing util; no new files; targeted vitest still exit 0.
```

## Cycle narrative

**Red.** Wrote `__tests__/Badge.test.tsx` asserting `expect(screen.getByTestId('unread-count')).toHaveTextContent('3')`. Ran `vitest run __tests__/Badge.test.tsx` against the pre-implementation state; vitest reported `Test Files  1 failed (1)` with exit 1. Trace appended to `admin/scratch/032-notif-badge-develop/red-test-trace-T3.log`.

**Green.** Added `<span data-testid="unread-count">{count}</span>` to `src/header/Badge.tsx` reading `count` from the existing notification context. Re-ran the same vitest command; exit 0. Trace appended.

**Refactor.** Extracted the count-formatting `> 99 ? '99+' : String(count)` ternary into `src/header/lib/format-count.ts`'s already-existing `formatBadgeCount` helper (no new file; helper already existed). Re-ran the same vitest command; exit 0 still. No new files created; no scope creep.

## Audit signal

All three markers `passed`, in cycle order, with strictly increasing ISO-8601 timestamps inside the same context window. The cycle held cleanly; retro reads it as a successful Yellow-lane TDD cadence.
