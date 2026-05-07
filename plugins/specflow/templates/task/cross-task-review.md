# Cross-task review — Phase E.4.5 + Phase F doctrine

When `specflow:task` Phase E (Gate 3) closes the per-task Round 2 (orchestrator response), a dedicated **whole-set review** fires before per-task Round 3 sharpen. The whole-set review reads the entire task list as a single artefact through two lenses — coherence + better-arrangement — and produces feedback that a separate fresh-context applier reads in isolation, decides on, and writes back into `tasks.md`. The original writer never sees or responds to cross-task feedback.

Introduced in v2.5.0 (`022-cross-task-review`). Codifies the Pocock issue-sizing-heuristic and Medin reviewer-isolation-discipline at Gate 3.

## When it fires

Phase E.4.5 fires only when the synthesised `{NNN-slug}-tasks.md` carries **3 or more tasks** (counted by `### T-` headings). Below the threshold, the manifest's "Cross-task findings" section is populated with a single line: `Cross-task review skipped: task count {N} below threshold (3)`.

## The two lenses

### Coherence

Do tasks work together; flow correctly; not overlap; all trace back to PRD requirements + goal? Coherence findings flag:

- Overlap (two tasks both anchoring the same R with overlapping scope).
- Phasing mismatch (task ordering does not honour `Depends on:` chains).
- Goal drift (a task that traces to an R but whose scope drifts past the requirement's `Serves goal:` field).
- Anchor-coverage gap (tasks that group oddly relative to PRD requirements — e.g. three tasks for one R when one would do).

### Better arrangement

Could two tasks merge; could one split; is the dependency ordering optimal; is there a missing task the per-task lens couldn't see? Findings include:

- Merge candidates — two tasks operationally similar / overlapping concerns.
- Split candidates — one task with weak coherence in its own scope (large but operationally fragmented).
- Reorder candidates — a different ordering reduces dependency tangles.
- Missing-task candidates — a gap the per-task lens couldn't see (typically a setup / migration task implied by the PRD but not derived).

## Phase E.4.5 — three-round mini-debate

### E.4.5.1 — Cross-task R1 fire

Dispatch a forked sub-agent reading `admin/agents/standard/principles/cross-task-reviewer.md`, the post-Round-2 `{NNN-slug}-tasks.md`, the PRD, the interview, and the Gate 2 manifest. The reviewer writes findings to `debate-log/tasks-gate3/findings/round-1/cross-task-reviewer.json` with per-finding shape:

```json
{
  "id": "cross-task-r1-f{n}",
  "lens": "coherence | better-arrangement",
  "severity": "info | concern | block",
  "claim": "...",
  "evidence": "...",
  "proposed_change": "..."
}
```

### E.4.5.2 — Applier R2 response + apply

Re-invoke `specflow:task --apply-cross-task-feedback {NNN-slug}` in a fresh context window. The applier reads (a) the cross-task R1 findings, (b) the post-Round-2 tasks.md, (c) the PRD/interview — never the writer's chat or the cross-task reviewer's deliberation transcripts beyond the written findings.

For each finding, the applier writes a decision to `debate-log/tasks-gate3/findings/round-2/cross-task-responses.json`:

```json
{
  "round": 2,
  "responses": {
    "cross-task-r1-f1": {
      "decision": "accepted | rejected | scope-change-required",
      "revision_applied": "if accepted: brief description of the tasks.md edit",
      "rationale": "if rejected: text",
      "gap": "if scope-change-required: description of the PRD-coverage delta"
    }
  }
}
```

`accepted` decisions produce in-place edits to `{NNN-slug}-tasks.md` (merge / split / reorder / add). `rejected` decisions stand. `scope-change-required` decisions halt application for that finding and route to `specflow:scope-change`.

### E.4.5.3 — Cross-task R3 sharpen

Re-dispatch the cross-task reviewer (fresh forked sub-agent) reading its R1 findings + the applier's R2 responses + the post-applier tasks.md. The reviewer may sharpen any rejection with new evidence or escalated severity. Writes to `debate-log/tasks-gate3/findings/round-3/cross-task-reviewer.json`.

### E.4.5.4 — Applier final pass

If any sharpens, re-invoke the applier. Decisions append to `cross-task-responses.json`. Unresolved findings escalate to the manifest's Cross-task Escalations sub-section.

## Phase F — Cross-task feedback application (applier-side)

When `specflow:task` is invoked with `--apply-cross-task-feedback {NNN-slug}`, it skips Phases A / B / C / D / E entirely and enters Phase F. Phase F's three inputs:

1. The post-Round-2 `{NNN-slug}-tasks.md`.
2. The cross-task findings JSON from `round-1/cross-task-reviewer.json` (or `round-3/...` for the final pass).
3. The PRD + interview for coverage-matrix validation.

**Precondition check** runs first. On missing `cross-task-reviewer.json`: refuse with *"Cross-task review has not fired for this feature. Run `specflow:task {NNN-slug}` from Phase E to produce the findings, then re-invoke `--apply-cross-task-feedback`."* On missing tasks.md or PRD: refuse with the analogous diagnostic.

**Decision rules:**

- **`accepted`** — the finding's `proposed_change` is applied verbatim or with minor adaptation (recorded in `revision_applied`). Coverage matrix is re-verified after edit (Phase B.4 logic re-applied — forward + reverse coverage; binary acceptance; budget self-check per 029-single-context-task).
- **`rejected`** — the finding's claim is unconvincing under the applier's reading; rationale recorded.
- **`scope-change-required`** — the finding implies a PRD coverage delta the applier cannot resolve in tasks.md alone. PRD-coverage-changing findings (add missing task with new R/AC anchor; merge-with-coverage-ambiguity; drop-with-coverage-hole) MUST be flagged this way.

**Hard-cap enforcement (per 029-R4).** The applier MUST decline any merge whose combined `context-budget-estimate` would exceed the configured per-task budget cap. Phase F reads `config.task.contextBudget` from the active config at Phase F entry — never from a value embedded in the Round-1 finding, tasks.md, or synthesis-time snapshot. Such mergers are flagged `scope-change-required` with rationale `merge would violate 029 R4 hard-cap (combined budget {N}K > cap {C}K) — re-synthesis required`.

**Sprint-bucket recompute (per 025).** When the applier accepts a merge or split:

- **Merge** — the merged task's `sprint-bucket` is recomputed via 025's heuristic at `templates/task/sprint-bucket-heuristic.md` against the merged scope (NOT inherited). If recompute would alter buckets outside the merged component, flag `scope-change-required` with rationale `merge bucket-recompute creates graph-wide bucket drift — re-synthesis required`.
- **Split** — each child task re-runs the heuristic.
- **Audit** — pre-edit and post-edit bucket values for each affected task ID are recorded in `cross-task-responses.json` per finding under a `bucket_audit` field.

## Manifest schema extension

The Gate 3 manifest at `debate-log/tasks-gate3/manifest.md` gains:

```markdown
**writer_id:** {opaque}
**cross_task_reviewer_id:** {opaque}
**applier_id:** {opaque}

## Cross-task findings

### Accepted findings
- **{finding-id}** (cross-task-reviewer, lens: coherence|better-arrangement, {severity}) — {claim}
  - Evidence: {evidence}
  - Revision applied: {description}

### Rejected findings
- **{finding-id}** (cross-task-reviewer, lens: ..., {severity}) — {claim}
  - Evidence: {evidence}
  - Reason for rejection: {rationale}

### Escalated to human
- **{finding-id}** (cross-task-reviewer, lens: ..., {severity}) — {claim}
  - Reason: {escalation reason}
```

The `agent_id` triplet is best-effort opaque (timestamp+suffix; UUIDv4; harness-emitted run ID — implementation chooses) and recorded verbatim. No echo-back protocol; no closer-side collision check; no FRESH-CONTEXT-VIOLATION escalation surface — those are 027 concerns. The audit signal is: in the green path all three fields are populated and pairwise non-equal; in the skip path only `writer_id` is populated.

## Closer precedence (Phase E.6)

The existing FAIL-on-unresolved-`block` rule applies to the UNION of per-task and cross-task findings. A `block`-severity finding from EITHER lens triggers the corresponding status. The closing rationale paragraph names which lens(es) drove the status (e.g. *"passed-with-escalations: per-task PASSED; cross-task escalated 1 block-severity coherence finding"*).

## Sub-agent dispatch failure fallback

If the cross-task reviewer's forked sub-agent OR the applier's Phase F invocation fails to return a valid finding/response JSON (network drop, harness crash, malformed JSON, missing finding file), Phase E.4.5 logs the failure to `debate-log/tasks-gate3/findings/round-{N}/cross-task-{role}.failure.json` with the failure mode + timestamp. The gate escalates as `passed-with-escalations` (NOT failed — cross-task is the additive lens; per-task review remains authoritative for the run). The manifest closer notes: *"Cross-task review unavailable: {reason}; per-task review remains authoritative for this run."*

## Per-task Round 3 (E.5) hybrid surface

Per-task reviewers re-fire AFTER Phase E.4.5 closes, against the post-applier `{NNN-slug}-tasks.md`. They operate as a hybrid R3:

- Sharpen R1 findings whose anchored T-id still exists in the post-applier tasks.md (standard R3 sharpen).
- Treat R1 findings whose T-id was merged-out / dropped as auto-resolved — recorded in manifest as `resolved-by-cross-task-merge: T{N}->T{M}` or `resolved-by-cross-task-drop: T{N}` per finding.
- Net-new findings on tasks introduced by the applier (e.g. a merged T3+T7 → new T3.5) are recorded as `round-3-net-new` per-task findings AND require a one-pass orchestrator response (no R4 sharpen — net-new findings are themselves an exit lever).

## Scope-change re-fire interaction

When `specflow:scope-change` Phase E.6 delta-regeneration re-fires `specflow:task` Phase E, the re-fire now traverses cross-task review on the regenerated task list. Acceptable by construction; not a scope-change-side edit.

## Cross-references

- **025 — sprint-task-flagging** — sprint-bucket recompute on accepted merge/split.
- **027 — reviewer-context-isolation** (Sprint 3) — absorbs the format contract for `agent_id` and adds runtime verification.
- **029 — single-context-task** — hard-cap budget enforcement at the applier; `context-budget-estimate` is the soft signal for the better-arrangement lens.

## Worked example

See `examples/docs/specflow/features/022-cross-task-review/fixtures/cross-task-worked-example/`.
