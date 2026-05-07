# Fixture — Refactor new-file-attempted (halt + post-hoc audit)

Task: T7 — Persist notification read-state to API on click.
Lane: Yellow.

## Cycle narrative

**Red.** Wrote `__tests__/notification-read.test.tsx` asserting `await api.markRead(notificationId)` was called when the user clicks. Targeted vitest exit 1.

**Green.** Added `await markRead(id)` invocation inside the click handler in `src/header/NotificationItem.tsx`, importing `markRead` from the existing `src/api/notifications.ts`. Targeted vitest exit 0.

**Refactor (attempted, halted).** The agent attempted to extract the click-handler logic into a new file `src/header/lib/use-mark-read.ts`. Halted with the literal message:

> Refactor cannot add files; route to specflow:scope-change

The agent did NOT create the new file. The agent re-ran the targeted vitest command; exit 0 still. Manifest records the post-hoc audit outcome.

## Per-task manifest stub

Located at `admin/scratch/032-notif-badge-develop/manifest-stub-T7.md`:

```
red: passed (2026-05-07T13:08:11Z) — failing test authored at __tests__/notification-read.test.tsx; targeted vitest exit 1.
green: passed (2026-05-07T13:11:42Z) — minimal implementation at src/header/NotificationItem.tsx; targeted vitest exit 0.
refactor: failed (new-file-attempted) (2026-05-07T13:13:05Z) — attempted to extract handler to src/header/lib/use-mark-read.ts; halted; routed to specflow:scope-change.
```

## Audit signal

The cycle's Refactor bound held: no new file landed in this PR. The post-hoc marker `failed (new-file-attempted)` is the audit hook for retros to identify "this task wanted to grow"; the developer can decide whether the proposed extraction is worth a follow-up `misc-task` or a future scope change.

## Halt message location

The doctrine doc carries the literal halt message at `templates/admin/tdd-discipline.md` § Refactor: *"Refactor cannot add files; route to specflow:scope-change"*. The agent quotes it verbatim when refusing the new-file extension.
