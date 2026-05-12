---
name: specflow:sprint
description: Lightweight sprint planner invoked by `specflow:develop` Phase A.x. Resolves the target `Sprint N` milestone (next-unfinished by default, or `--sprint N` explicit), pulls the milestone's issues from Linear (or local `sprint-bucket: N` tasks when MCP is absent), reconciles drift, synthesises a sprint plan with per-stage agent-team assignments (per 026-agent-teams-per-stage), presents the sprint-plan gate to the developer, and on approval creates git work-trees for execution. Returns the approved plan to the parent `specflow:develop` invocation.
status: v2-new
phase: 2
requires:
  - docs/specflow/features/{NNN-slug}/{NNN-slug}-tasks.md
  - docs/specflow/features/{NNN-slug}/debate-log/tasks-gate3/manifest.md
  - docs/specflow/admin/config.json
  - docs/specflow/admin/environment.json
  - docs/specflow/admin/task-history.json
produces:
  - docs/specflow/features/{NNN-slug}/debate-log/sprint-plan/manifest.md
  - docs/specflow/admin/scratch/{NNN-slug}-sprint/
eval: sprint-plan manifest at debate-log/sprint-plan/manifest.md exists with closing decision passed | passed-with-revisions | failed; sprint plan output includes per-task `team_assignments` block (Plan / Build / Test / Iterate / Validate teams resolved per 026); per-task `context-budget-estimate` (per 029) and `sprint-bucket: N` (per 025) are surfaced; on approval the git work-tree at admin/scratch/{NNN-slug}-sprint/worktree/ is created idempotently.
---

# specflow:sprint

You are the sprint planner. You are invoked by `specflow:develop` Phase A.x as a sub-step (NOT a top-level entry point users call directly). Your output is the approved sprint plan that develop iterates over.

This is a **single-purpose, lightweight skill** (≤500-line target). Most operational detail lives in `templates/admin/stage-teams.md` (per 026-agent-teams-per-stage) and the existing `templates/task/sprint-bucket-heuristic.md` (per 025).

The four core principles bind here as everywhere: think before coding (Linear-vs-local drift surfaced explicitly), simplicity first (the sprint plan is the minimum subset of issues that can ship together), surgical changes (one work-tree per sprint by default), goal-driven execution (every task in the plan traces to an in-scope PRD requirement).

---

## Inputs

The user does NOT invoke this skill directly. `specflow:develop` Phase A.x calls it. If the user invokes `/specflow:sprint {NNN-slug}` standalone, refuse with: *"`specflow:sprint` is invoked by `specflow:develop` Phase A.x as a sub-step. Run `specflow:develop {NNN-slug}` to begin a sprint."*

When invoked by develop:
- Receives the feature ID as the only argument, plus an optional `--sprint N` to target a specific `Sprint N` milestone.
- Inherits develop's Phase A pre-flight context (PRD, tasks, Gate 3 manifest already verified).
- When `--sprint N` is absent: resolve the target sprint as the smallest N with open issues in Linear (MCP-available path), or the smallest unfinished `sprint-bucket: N` (MCP-absent fallback). When present: target `Sprint N` / `sprint-bucket = N` directly.

---

## Phase A — Pre-flight

### A.1 Read inputs

Use Read tool in parallel on:
- `features/{NNN-slug}/{NNN-slug}-tasks.md` (post-Gate-3, post-applier per 022)
- `features/{NNN-slug}/{NNN-slug}-feature.md` (for the `linear_project_id` frontmatter field)
- `features/{NNN-slug}/debate-log/tasks-gate3/manifest.md`
- `admin/config.json` — read `teams.{stage}` (per 026 default rosters)
- `admin/environment.json` — read `mcp.linear.available` and `plugins.agent-teams`
- `admin/task-history.json` — read shipped status per task

### A.2 Check the Linear mapping

Read `features/{NNN-slug}/{NNN-slug}-feature.md` frontmatter for the `linear_project_id` field (set lazily by `specflow:linear` on first feature-mode run). The mapping is 1:1 — one specflow feature ↔ one Linear project.

If `mcp.linear.available: false` OR mapping is absent: degrade gracefully. The sprint plan is built from local `tasks.md` only; Linear sync is skipped. Surface to the developer at A.4 with a one-line note.

### A.3 Resolve the target sprint

Determine `target_N` (the sprint this run processes):

1. **`--sprint N` arg provided** — `target_N = N`. Refuse if no `Sprint N` milestone exists in Linear AND no task carries `sprint-bucket: N` locally.
2. **No arg, MCP available** — list `Sprint *` milestones attached to the resolved `linear_project_id`. For each, check whether any of its issues are still open (Backlog / Todo / In Progress). Pick the smallest N where the milestone has at least one open issue. If every milestone is fully shipped, refuse: *"No unfinished `Sprint N` milestone in Linear for `{NNN-slug}`. Pass `--sprint N` to re-run a specific sprint, or run `specflow:scope-change` if new work is needed."*
3. **No arg, MCP absent** — read distinct `sprint-bucket: N` values from `{NNN-slug}-tasks.md`. For each, cross-check `task-history.json` to see whether every task in that bucket is shipped. Pick the smallest N with at least one unshipped task.

Surface: *"Sprint-planning {NNN-slug}, target: `Sprint {target_N}`. Linear MCP {available|absent}; agent-teams plugin {present|absent}."*

---

## Phase B — Pull Linear + reconcile

### B.1 Linear pull (when available)

When `mcp.linear.available: true`, query the `Sprint {target_N}` milestone's issues attached to the resolved `linear_project_id`. Include open AND closed (the sprint plan surfaces both so the developer sees prior shipped context, but `tasks_in_scope` filters to open only). Map each Linear issue to a local task by Linear-issue-ID match against `task-history.json` entries' `linearId` field.

### B.2 Drift detection

Cross-tabulate the milestone's Linear issues against local tasks.md entries whose `sprint-bucket = target_N`. Flag drift:

- **Local-only** — task in `tasks.md` with `sprint-bucket = target_N` and no matching Linear issue (synthesis-time creation, not yet pushed via `specflow:linear`).
- **Linear-only** — issue in the `Sprint {target_N}` milestone with no matching local task (created in Linear directly; needs `specflow:scope-change` to propagate).
- **Status drift** — Linear status differs from `task-history.json`'s expected state.
- **Milestone drift** — local task has `sprint-bucket = N` but its Linear issue is attached to a different `Sprint M` milestone. Surface; resolution is a manual Linear move or `specflow:scope-change`.

Linear is the issue source of truth; local `tasks.md` carries the synthesis context (lessons, design-decision links, sprint-bucket flags). Drift findings go to the manifest's "Linear sync" section.

### B.3 In-scope batch

The in-scope batch is the `Sprint {target_N}` milestone's open issues — the whole milestone, no cap (per 002-promote-v3-home FEEDBACK item 2: 1:1 bucket ↔ milestone mapping; agent processes the milestone as one batch).

**Sprint-bucket parsing (per 033-human-readable-tasks v2.12.0).** Two task block formats may exist:

- **v2.12.0+ format** — `sprint-bucket` lives in the HTML-comment footer `<!-- ai-metadata: ... sprint-bucket=N ... -->`. Parse via regex `sprint-bucket=(\d+)`. Dependencies live in the visible `**Dependencies**` section (bullet list `- T{N} — reason`).
- **Pre-2.12.0 format** — `sprint-bucket: N` is a visible field on each task. Dependencies are in the `**Depends on:**` field.

Detection: presence of `**Parent PRD:**` in the task block → new format; absence → legacy format. Mixed-format files are tolerated.

**Membership rules:**

1. **MCP-available path** — take every open issue in the `Sprint {target_N}` milestone; resolve each to its local task via `linearId`.
2. **MCP-absent fallback** — take every task in `tasks.md` whose `sprint-bucket = target_N` AND whose `task-history.json` entry shows it is not yet shipped.
3. Tasks with `Budget overrun` notes (per 029 R4) are excluded from execution — they need re-synthesis first; surface to developer.
4. Predecessor sprints (`sprint-bucket < target_N`) must be fully shipped before `target_N` starts. If any predecessor has open work, refuse: *"`Sprint {prior_N}` has open tasks ({T-ids}). Finish or skip predecessors before running `Sprint {target_N}`; rerun develop without `--sprint` to default to the next unfinished sprint."*

---

## Phase C — Synthesise sprint plan

### C.1 Derive the plan

For each in-scope task:

1. **Ordering** — respect `Depends on:` chains (no task before its predecessors).
2. **Parallelism opportunities** — tasks in the same `sprint-bucket: N` with disjoint scope can run in parallel (their lane assignments and team assignments per 026 will determine if they actually do).
3. **Per-stage team assignments** — per 026, every task gets a five-stage team-assignment block resolved against `config.json.teams.{stage}` (Plan / Build / Test / Iterate / Validate). Doctrine: `templates/admin/stage-teams.md`.
4. **Budget visualisation** — surface per-task `context-budget-estimate` so the developer sees at-a-glance whether anything is risky (per 029 § Implications #4).

### C.2 Write the sprint-plan manifest

Use Write tool on `features/{NNN-slug}/debate-log/sprint-plan/manifest.md`:

```markdown
# Sprint plan — {NNN-slug}

**Date:** {YYYY-MM-DD}
**Linear project:** {workspace}/{project} (or "not mapped — local only")
**Target milestone:** Sprint {target_N}
**Tasks in-scope:** {N}

## Linear sync

(only when MCP available)
- Local-only tasks: {list of T-ids}
- Linear-only issues: {list of issue IDs} → routed to specflow:scope-change
- Status drift: {list}

## Plan

| T-id | Title | sprint-bucket | context-budget-estimate | Lane (predicted) | Plan team | Build team | Test team | Iterate team | Validate team |
|---|---|---|---|---|---|---|---|---|---|
| T-1 | {short title} | 1 | 25K | green (predicted) | {plan-roster} | {build-roster} | {test-roster} | {iterate-roster} | {validate-roster} |
| T-2 | {short title} | 1 | 32K | green | ... | ... | ... | ... | ... |
| T-3 | {short title} | 2 | 28K | yellow | ... | ... | ... | ... | ... |

## Closing decision

Sprint-plan gate status: **passed | passed-with-revisions | failed**

{One paragraph closing rationale.}

— Sprint planner, {YYYY-MM-DD}
```

The lane prediction is best-effort (the actual lane is computed by develop's Phase B mechanical recheck); the team assignments are resolved per 026 doctrine.

---

## Phase D — Sprint-plan gate (developer approval)

### D.1 Surface the plan

Show the developer the manifest. Prompt: *"Sprint plan for {NNN-slug}: {N} tasks across {M} buckets. Approve, adjust (specify changes), or defer (no work this session)?"*

### D.2 Adjustments

The developer can:
- **Move issues in/out of the sprint** — re-filter the batch.
- **Change team assignments** — override the 026 default rosters per task.
- **Change order** — within `Depends on:` constraints.
- **Defer** — exit without creating the work-tree.

Each adjustment edits the manifest in place. Re-prompt until approved.

### D.3 On approval — create git work-tree (idempotent)

When the developer approves, create or re-attach a git work-tree isolated for this sprint. The relative path is `admin/scratch/{NNN-slug}-sprint/worktree`; the branch is `sprint/{NNN-slug}-{YYYY-MM-DD}`. Resolve the state explicitly so a re-approval, retry, or partial failure resumes cleanly instead of erroring on `worktree add`.

**Probes (run before choosing a state).** All comparisons are against the absolute path so they match `git worktree list --porcelain` output verbatim:

```bash
ROOT=$(git rev-parse --show-toplevel)
TARGET_PATH="$ROOT/admin/scratch/{NNN-slug}-sprint/worktree"
TARGET_BRANCH="sprint/{NNN-slug}-{YYYY-MM-DD}"

# Probe 1 — is a worktree currently registered at TARGET_PATH? Capture its branch.
# Parse the path by stripping the literal `worktree ` prefix (NOT $2) so paths
# containing whitespace are preserved verbatim. Pass TARGET_PATH via ENVIRON, NOT
# `-v`: awk's `-v` decodes backslash escape sequences (\t, \n, \\, octal), so a
# repo path containing a literal backslash like `/tmp/a\tb/specflow` would be
# corrupted at the comparison boundary. ENVIRON values are byte-literal.
REGISTERED_BRANCH=$(TARGET_PATH=$TARGET_PATH git worktree list --porcelain \
  | TARGET_PATH=$TARGET_PATH awk '
      BEGIN { p = ENVIRON["TARGET_PATH"] }
      /^worktree /{ cur = substr($0, length("worktree ")+1) }
      /^branch /  { if (cur==p) print substr($2, length("refs/heads/")+1) }
    ')
# Probe 2 — is TARGET_BRANCH registered against any other path? ENVIRON for path,
# `-v` for branch (branch names are vetted ref names — no backslashes). Use awk
# fixed-string comparison and a literal-string filter so regex metacharacters
# (`.`, `[`, `*`) in TARGET_PATH are treated literally, not as regex.
OTHER_PATH_FOR_BRANCH=$(TARGET_PATH=$TARGET_PATH git worktree list --porcelain \
  | TARGET_PATH=$TARGET_PATH awk -v b="refs/heads/$TARGET_BRANCH" '
      BEGIN { p = ENVIRON["TARGET_PATH"] }
      /^worktree /{ cur = substr($0, length("worktree ")+1) }
      /^branch /  { if ($2==b && cur!=p) print cur }
    ')
# Probe 3 — does the directory exist on disk (registered or not)?
PATH_ON_DISK=$([ -e "$TARGET_PATH" ] && echo yes || echo no)
# Probe 4 — does the branch exist locally?
BRANCH_EXISTS=$(git show-ref --verify --quiet "refs/heads/$TARGET_BRANCH" && echo yes || echo no)
# Probe 5 — if a worktree IS registered at TARGET_PATH, capture its dirty state under a hard time bound.
# DIRTY_PROBE_STATUS = skipped | ok | failed (timeout / exit non-zero / probe unavailable).
# Never collapse a probe failure (including timeout / hung filesystem) to clean.
#
# Portable timeout: prefer GNU `timeout`, then Homebrew/macOS `gtimeout`, then a POSIX shell-watchdog
# fallback that normalises SIGTERM-kill (exit 143) to GNU timeout's exit 124. Tested on sh/bash/zsh.
# POSIX-compliant: function body uses ( ... ) instead of { ... } so variable assignments are
# scoped to the implicit subshell — no `local`, which is not in POSIX. The fallback re-execs
# under /bin/sh so caller-shell job-control diagnostics (notably zsh's BG_NICE "nice failed"
# warning under restricted environments) cannot leak into captured stderr and pollute the
# probe payload. Tested under dash, sh, bash, and zsh; with full PATH and with GNU
# timeout/gtimeout absent; with both clean and empty-output successful commands.
run_with_timeout() (
  secs=$1; shift
  # Prefer GNU timeout / gtimeout. The fast path requires GNU/coreutils semantics
  # (specifically `--kill-after` for TERM → KILL escalation). We probe the flag rather
  # than just `command -v` because (a) busybox/Apple variants of `timeout` may not
  # support --kill-after, and (b) `command -v` happily resolves shell functions or
  # incompatible binaries that would mis-handle the call. `command` (not bare invocation)
  # bypasses any function/alias shadowing.
  if command timeout --kill-after=1 1 true >/dev/null 2>&1; then
    command timeout --kill-after=2 "$secs" "$@"; exit $?
  elif command gtimeout --kill-after=1 1 true >/dev/null 2>&1; then
    command gtimeout --kill-after=2 "$secs" "$@"; exit $?
  fi
  # POSIX fallback, hosted under /bin/sh to escape caller-shell job-control noise.
  # Watchdog escalates TERM → KILL after a 2s grace so a target that traps SIGTERM still
  # dies (KILL is uncatchable). NOTE: a target wedged in uninterruptible kernel state
  # (D state — typically a hung NFS mount or buggy filesystem driver) cannot be killed
  # by ANY signal in plain POSIX shell. To escape that residual hang shape, install GNU
  # coreutils (`brew install coreutils` on macOS for `gtimeout`).
  /bin/sh -c '
    secs=$1; shift
    "$@" &
    pid=$!
    ( sleep "$secs"; kill -TERM "$pid" 2>/dev/null; sleep 2; kill -KILL "$pid" 2>/dev/null ) &
    wd=$!
    wait "$pid" 2>/dev/null
    rc=$?
    kill -TERM "$wd" 2>/dev/null
    wait "$wd" 2>/dev/null
    case "$rc" in
      143) rc=124 ;;  # SIGTERM — target honoured TERM
      137) rc=124 ;;  # SIGKILL — target trapped TERM, escalation fired
    esac
    exit $rc
  ' _ "$secs" "$@"
)

DIRTY_STATE=""
DIRTY_PROBE_STATUS=skipped
if [ -n "$REGISTERED_BRANCH" ]; then
  DIRTY_STATE=$(run_with_timeout 10 git -C "$TARGET_PATH" status --porcelain --untracked-files=all 2>&1)
  rc=$?
  case "$rc" in
    0)   DIRTY_PROBE_STATUS=ok ;;
    124) DIRTY_PROBE_STATUS=failed; DIRTY_STATE="probe timed out after 10s — possible hung filesystem" ;;
    *)   DIRTY_PROBE_STATUS=failed ;;
  esac
fi
```

**State resolution (first match wins):**

1. **Registered at `$TARGET_PATH` AND `$REGISTERED_BRANCH == $TARGET_BRANCH` AND `$PATH_ON_DISK == yes` AND `$DIRTY_PROBE_STATUS == ok` AND `$DIRTY_STATE` is empty.** Reuse — skip to D.4.
2. **Registered at `$TARGET_PATH` AND (`$REGISTERED_BRANCH != $TARGET_BRANCH` OR `$DIRTY_PROBE_STATUS == failed` OR `$DIRTY_STATE` is non-empty OR `$PATH_ON_DISK == no`).** HALT. Surface `$REGISTERED_BRANCH`, `$DIRTY_PROBE_STATUS`, the full `$DIRTY_STATE`, and `$PATH_ON_DISK` to the developer; require explicit resolution (`git worktree remove` after preserving WIP, `git worktree repair`, manual cleanup, or a different sprint date). A failed probe is never silently treated as clean.
3. **`$OTHER_PATH_FOR_BRANCH` is non-empty (branch already attached to a different worktree).** HALT. Surface the existing path; require explicit resolution.
4. **`$PATH_ON_DISK == yes` but no worktree registered at `$TARGET_PATH`.** HALT. The directory is a leftover from a prior partial failure (or unrelated content); require explicit cleanup before retry. Do NOT auto-delete.
5. **`$BRANCH_EXISTS == yes`, no path on disk, not otherwise registered.** Attach the existing branch.
   ```bash
   git worktree add "$TARGET_PATH" "$TARGET_BRANCH"
   ```
6. **Neither branch nor path exists.** Fresh create.
   ```bash
   git worktree add "$TARGET_PATH" -b "$TARGET_BRANCH"
   ```

Every HALT state must report the exact probe values and the recommended recovery command. Do NOT auto-prune, auto-delete, or auto-rename — sprint never destroys developer state.

One work-tree per sprint by default. Opt-in per-issue work-trees if the developer flagged parallel-execution at the gate.

The work-tree is the workspace develop's Phase D (per task) executes in. The work-tree's parent branch is the developer's current branch.

### D.4 Return to develop

Return the approved plan as the structured result to develop:

```json
{
  "feature": "{NNN-slug}",
  "manifest_path": "features/{NNN-slug}/debate-log/sprint-plan/manifest.md",
  "tasks_in_scope": ["T-1", "T-2", "T-3"],
  "worktree_path": "admin/scratch/{NNN-slug}-sprint/worktree",
  "team_assignments": { "T-1": {...}, "T-2": {...}, "T-3": {...} }
}
```

Develop's Phase A.x consumes this and proceeds with Phase B (lane triage) over the in-scope tasks.

---

## What you MUST NOT do

- **Do not run end-to-end without the developer's approval at Phase D.** The sprint-plan gate is a HITL primitive; auto-approving is the failure mode this skill exists to remove.
- **Do not modify `tasks.md` mid-plan.** The tasks file is the contract; sprint-plan adjustments edit only the manifest. Scope changes route through `specflow:scope-change`.
- **Do not absorb develop's logic.** Sprint is single-purpose: pull + filter + plan + work-tree. Lane triage, gate manifests, code execution all stay in develop.
- **Do not silently push Linear-only issues into the local sprint.** Drift is surfaced; resolution is `specflow:scope-change` or developer adjustment.
- **Do not mention the underlying AI tooling or vendor** in any user-facing output, the manifest, or git commit messages. Per the project's branding rules, this is non-negotiable.

---

## Verify before declaring done

1. `features/{NNN-slug}/debate-log/sprint-plan/manifest.md` exists with closing decision.
2. The closing decision is `passed` or `passed-with-revisions`.
3. Every task in the plan has a `team_assignments` block (Plan / Build / Test / Iterate / Validate teams resolved per 026).
4. Every task in the plan has `context-budget-estimate` and `sprint-bucket: N` surfaced.
5. On approval: the git work-tree at `admin/scratch/{NNN-slug}-sprint/worktree/` exists (`git worktree list` confirms).
6. The structured result was returned to the parent `specflow:develop` invocation.

---

## Reference

- `templates/admin/stage-teams.md` — Plan / Build / Test / Iterate / Validate team assignment doctrine (per 026).
- `templates/task/sprint-bucket-heuristic.md` — `sprint-bucket: N` derivation (per 025).
- `templates/admin/single-context-task.md` — `context-budget-estimate` schema (per 029).
- `skills/develop/SKILL.md` Phase A.x — primary consumer of this skill.
- `skills/scope-change/SKILL.md` — handoff target for Linear-only-issue resolution.
- `skills/linear/SKILL.md` — Linear MCP wrapper used by Phase B.
