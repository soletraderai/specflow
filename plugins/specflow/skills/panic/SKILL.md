---
name: panic
description: Trust-ladder primitive — the big red button. Captures a postmortem snapshot of the current session state, then rewinds the working tree to the last clean checkpoint (default = HEAD; configurable to a named tag). Confirms with the user before any destructive action; never operates silently. Stops in-flight orchestrations by removing scratch directories.
status: v2-new
phase: 1
requires: []
produces:
  - docs/specflow/admin/panic-snapshots/{timestamp}/
eval: snapshot bundle exists at the reported path with diff.patch + status.txt + scratch-archive/; working tree is clean of changes from the panicked session OR user explicitly declined the rewind step; user confirmation captured before destructive actions.

---

# panic

Trust-ladder primitive. The big red button users press when work is heading somewhere it shouldn't.

This skill is **safety-critical**. Every destructive step requires explicit user confirmation; nothing is silent. The primary failure mode to avoid is destroying user work the user did not intend to discard.

---

## Inputs

The user invokes you with one of:
- `/panic` — full panic flow with default rewind target (`HEAD`).
- `/panic --to {ref}` — rewind to a named tag, branch, or SHA instead of `HEAD`.
- `/panic --snapshot-only` — capture the postmortem bundle but do NOT rewind. Useful when the user wants to inspect state without losing it.

If invoked with no argument, default to `--to HEAD` (rewind discards uncommitted changes; committed work is preserved).

---

## Phase A — Snapshot (always; never destructive)

### A.1 Create the snapshot directory

```bash
ts=$(date +%Y%m%d-%H%M%S)
mkdir -p "docs/specflow/admin/panic-snapshots/${ts}"
```

Tell the user: *"Capturing snapshot to `docs/specflow/admin/panic-snapshots/{ts}/`. Nothing destructive yet."*

### A.2 Capture the working tree

Use Bash to write the following files into the snapshot directory:

```bash
# The full diff — every uncommitted change, staged and unstaged
git diff HEAD > "docs/specflow/admin/panic-snapshots/${ts}/diff.patch"

# Untracked files index (paths only; not the contents — keeps bundle small)
git ls-files --others --exclude-standard > "docs/specflow/admin/panic-snapshots/${ts}/untracked.txt"

# Status snapshot for human review
git status > "docs/specflow/admin/panic-snapshots/${ts}/status.txt"

# Recent log so the postmortem has anchor points
git log --oneline -20 > "docs/specflow/admin/panic-snapshots/${ts}/recent-commits.txt"

# Current branch + HEAD SHA for reproducibility
git branch --show-current > "docs/specflow/admin/panic-snapshots/${ts}/branch.txt"
git rev-parse HEAD > "docs/specflow/admin/panic-snapshots/${ts}/head-sha.txt"
```

### A.3 Archive any in-flight scratch

Specflow orchestrations write to `docs/specflow/admin/scratch/{orchestration-id}/`. If any scratch directory exists, archive it (don't delete yet — that's Phase C):

```bash
if [ -d docs/specflow/admin/scratch ] && [ "$(ls -A docs/specflow/admin/scratch)" ]; then
  cp -r docs/specflow/admin/scratch "docs/specflow/admin/panic-snapshots/${ts}/scratch-archive"
fi
```

### A.4 Capture in-flight debate state

Glob for `features/*/debate-log/*/findings/round-1/*.json` files modified in the last hour — these indicate an in-flight Gate review the user might be panicking out of. Copy the list of paths (not the contents) into the snapshot:

```bash
find docs/specflow/features -path "*/debate-log/*/findings/*" -mmin -60 -type f 2>/dev/null \
  > "docs/specflow/admin/panic-snapshots/${ts}/inflight-gates.txt"
```

### A.5 Confirm snapshot

Tell the user: *"Snapshot captured. Contents: diff.patch ({N} lines), {M} untracked files indexed, scratch-archive ({K} dirs), inflight-gates ({J} files in flight). Path: `docs/specflow/admin/panic-snapshots/{ts}/`."*

---

## Phase B — Confirm rewind (mandatory pause)

If `--snapshot-only` was provided, skip to Phase D.

Otherwise, present the destructive plan and require explicit confirmation:

```
Ready to rewind:
  • Discard {N} uncommitted lines (preserved in diff.patch above).
  • Remove {M} untracked files matching {summary of untracked.txt}.
  • Delete docs/specflow/admin/scratch/* ({K} in-flight orchestration scratch dirs, archived above).
  • Restore working tree to {target ref, e.g. HEAD = abc1234 "commit message"}.

In-flight Gate reviews ({J} files): {list paths from inflight-gates.txt}.
These will NOT be deleted automatically — they live inside features/ which we don't touch.
Re-run the gate later if needed.

Type "yes, rewind" to proceed. Anything else aborts (snapshot is preserved).
```

Wait for the user's response. Match their input against the literal string `yes, rewind`. Anything else (including capital-letter variations, "y", "yes", "proceed") = abort. The full phrase is required to make the destructive action deliberate.

If the user aborts: tell them *"Rewind aborted. Snapshot preserved at `docs/specflow/admin/panic-snapshots/{ts}/`. Working tree unchanged."* Skip to Phase D.

---

## Phase C — Rewind (only after confirmation)

### C.1 Discard uncommitted changes

```bash
git restore .              # discards unstaged changes
git restore --staged .     # unstages anything that was staged
git clean -fd              # removes untracked files and directories
```

### C.2 Reset to target ref

If the target is `HEAD` (default):
- The above steps already left the tree at HEAD. No further git command needed.

If the target is a different ref (e.g. `--to v2.0.0` or `--to feature-branch`):
- This is a non-default destructive action. Re-confirm: *"Target is `{ref}`, not HEAD. This will move the branch pointer. Confirm with `yes, reset to {ref}` exactly."*
- On confirmation: `git reset --hard {ref}` (this is the only `--hard` use; never use it without explicit confirmation).

### C.3 Remove scratch directories

```bash
rm -rf docs/specflow/admin/scratch/*
```

The scratch contents are already in the snapshot archive (Phase A.3); this step just clears the live state so future orchestrations start clean.

### C.4 Stop background processes

This skill cannot reliably reach across process boundaries; the harness owns that. Tell the user: *"Background-process cleanup is your responsibility — check for in-flight Playwright sessions, dev servers, or background agents you started. The snapshot lists what was captured at panic time."*

---

## Phase D — Report

Final report to the user:

```
panic complete.
- Snapshot: docs/specflow/admin/panic-snapshots/{ts}/
- Action taken: {snapshot-only | rewound to {ref}}
- Working tree: {clean | unchanged}
- Scratch: {cleared | unchanged}

Postmortem starting points:
- diff.patch — exactly what was discarded
- inflight-gates.txt — gate reviews that were in flight
- recent-commits.txt — last 20 commits for orientation

Next steps you might want:
- Inspect the diff: cat docs/specflow/admin/panic-snapshots/{ts}/diff.patch
- Recover a specific change: git apply docs/specflow/admin/panic-snapshots/{ts}/diff.patch
- Investigate the in-flight gates: open the paths in inflight-gates.txt
```

---

## What you MUST NOT do

- **Never destructive without explicit confirmation.** Phase B is mandatory; the literal-phrase match (`yes, rewind`) is the discriminator. A user typing "yes" or "proceed" is NOT consent.
- **Never `git reset --hard` against a default target.** `--hard` is reserved for `--to {non-HEAD-ref}` invocations and only after the second confirmation.
- **Never skip the snapshot.** Even on `--snapshot-only` the snapshot is the whole point. Even on rewind, snapshot must complete before any destructive step.
- **Never delete from `features/`, `admin/` (other than `scratch/`), or any path outside `admin/scratch/` and the working-tree state.** Feature folders, debate logs, rule registries, etc. are user data — preserved across panic.
- **Never push, force-push, or otherwise touch remote refs.** Local-only.
- **Never invoke other specflow skills automatically.** Panic is read-only on the broader workflow; suggest skills, never run them.
- **Never mention Claude, Anthropic, or any AI tooling** in the snapshot bundle, the report, or any output. Per the project's CLAUDE.md.

---

## Verify before declaring done

1. Snapshot directory `docs/specflow/admin/panic-snapshots/{ts}/` exists with at least: `diff.patch`, `status.txt`, `untracked.txt`, `recent-commits.txt`, `branch.txt`, `head-sha.txt`.
2. If rewind was performed: `git status` shows no working-tree changes (post-rewind), AND `docs/specflow/admin/scratch/` is empty.
3. If rewind was declined: snapshot exists, working tree is unchanged.
4. Report to the user matches the action actually taken (no claims of "rewound" if user declined).

If any verify step fails, surface the discrepancy concretely — never claim panic completed when it didn't.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 10 — trust-ladder primitives (panic, confidence-check, getting-started profile).
- `skills/confidence-check/SKILL.md` — sister primitive for non-emergency uncertainty.
