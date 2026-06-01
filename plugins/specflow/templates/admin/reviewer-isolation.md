# Reviewer context isolation

Across every multi-agent debate gate (Gate 2, Gate 3, Gate 4, Gate 5), every reviewer runs in a **fresh context** with **zero exposure** to the writer's chat history. Reviewer input is the artefact under review (PRD, task list, plan, code) plus its declared context inputs (interview, prior gate manifest) — never the writer's reasoning trace. Same agent identity may not write AND validate the same artefact within a feature.

Introduced in v2.6.0 (`027-reviewer-context-isolation`). Codifies the Medin reviewer-isolation discipline as cross-cutting doctrine; absorbs and extends 022's interim `agent_id` mechanism.

## Why this exists

Two converging signals from the research dataset:

- **Reviewer should never see the writer's chat** (`knowledge/medin-parallel-agentic-playbook.md` § Reviewer isolation). Medin's exact framing: an agent that reviews its own work cannot see what's wrong with it; the writer's reasoning trace primes the reviewer to reach the same conclusions. Fresh-context spawn is the only effective remedy.
- **Smart-zone cliff sits at ~100K tokens** (`knowledge/pocock-ai-coding-real-engineers.md` § LLM Constraint 1). Reviewers sharing the writer's context start in the dumb zone — they review against degraded reasoning. Fresh context starts the reviewer in the smart zone with only the artefact under review loaded.

Together: review in the writer's context is the dual failure mode of "no review" — the bookkeeping artefact says "reviewed" but the reasoning is shared. The contract this doc carries is: **same agent identity may not write AND validate the same artefact within a feature**.

## The contract

For every gate (2, 3, 4, 5) in a feature:

1. **Each reviewer invocation is a fresh subagent spawn.** The orchestrator dispatches via the `Agent` tool (or equivalent fork mechanism) with no carryover of conversation state.
2. **Input is explicit, file-listed, never inferred.** The orchestrator passes ONLY the artefact under review + its declared dependencies (interview, prior gate manifest, role-definition file) — never the writer's chat or scratchpad.
3. **`writer_id` and `reviewer_ids` differ pairwise.** The gate manifest carries:
   ```
   **writer_id:** {opaque}
   **reviewer_ids:**
     - devils-advocate: {opaque}
     - simplicity-reviewer: {opaque}
     - surgical-reviewer: {opaque}
     - think-before-coding-reviewer: {opaque}
     - goal-driven-reviewer: {opaque}
     - cross-task-reviewer: {opaque}  # Gate 3 only, when 3+ tasks
     - codex-reviewer: {opaque}        # Gate 5 only, when codex available
     - edge-case-reviewer: {opaque}    # Gates 4 + 5
   ```
   Pairwise non-equality is verified at gate close. Any collision is a `FRESH-CONTEXT-VIOLATION` and aborts the gate.
4. **Reviewer role-defs forbid consulting the writer's chat.** Each principle reviewer's role file under `admin/agents/standard/principles/` carries the line *"You read only: [the artefact under review] and [the declared inputs]. You do NOT consult the writer's chat or the orchestrator's deliberation transcripts."*

## Format contract for `agent_id` (v2.6.0)

The agent ID format is the harness-emitted run ID concatenated with the reviewer slot name:

```
{harness-run-id}-{slot}
```

Where `{harness-run-id}` is the unique identifier the agent harness emits when spawning a fresh subagent (UUIDv4, ULID, or harness-specific opaque value), and `{slot}` is one of: `writer | devils-advocate | simplicity-reviewer | surgical-reviewer | think-before-coding-reviewer | goal-driven-reviewer | cross-task-reviewer | applier | codex-reviewer | edge-case-reviewer`.

When the harness does not emit a run ID (legacy harness; standalone invocation), fall back to ISO-8601-with-suffix: `2026-05-07T12:34:56Z-{slot}-{4-hex}`.

## Runtime verification

The closer's `agent_id` collision check runs at every gate's E.6 (or equivalent) close step:

```python
ids = collect_agent_ids_from_manifest(manifest_path)
seen = set()
for slot, value in ids.items():
    if not value:
        continue  # absent fields short-circuit; not all slots populated on every gate
    if value in seen:
        raise FreshContextViolation(f"agent_id collision: slot={slot} id={value}")
    seen.add(value)
```

A `FreshContextViolation` aborts the gate close with status `failed` and surfaces the violation to the developer with the colliding slot pair.

## Cross-cutting impact across gates

| Gate | Where it fires | Reviewer slots that must differ |
|---|---|---|
| Gate 2 | `specflow:prd` Phase D | writer + 5 principle reviewers (+ codex when avail) |
| Gate 3 | `specflow:task` Phase E | writer + 5 principle reviewers + cross-task-reviewer + applier (when 3+ tasks) |
| Gate 4 | `specflow:develop` Phase C | writer + 5 principle reviewers + edge-case-reviewer (per 028) |
| Gate 5 | `specflow:develop` Phase E | writer + 5 principle reviewers + codex-reviewer (when avail) + edge-case-reviewer (per 028) |

## What this absorbs from 022

022-cross-task-review shipped `writer_id`, `cross_task_reviewer_id`, `applier_id` as a best-effort manifest-field-only convention with no format contract. 027 absorbs:

- The field NAMES + the populate-at-dispatch convention (kept verbatim).
- The audit signal of pairwise non-equality (kept; promoted to runtime verification).

027 adds (new):

- The format contract for `agent_id` (above).
- The runtime collision check at every gate close.
- The `FRESH-CONTEXT-VIOLATION` escalation surface.
- The reviewer-role-def line forbidding writer's-chat access.

## What this enables for 028

028-edge-case-reviewer adds a new principle reviewer at Gate 4 + Gate 5. By landing 027 first, 028's new reviewer immediately conforms to the fresh-context contract — there's no retrofit pass.

## Cross-references

- **022 — cross-task-review** — interim `agent_id` mechanism that 027 absorbs and extends.
- **028 — edge-case-reviewer** — new reviewer at Gate 4 + 5 that conforms to this contract on landing.
- **029 — single-context-task** — orthogonal: 029 governs the writer's context window; 027 governs the reviewers' contexts. Both contracts hold simultaneously.
- **`templates/orchestrator-pattern.md`** — extended to document the fork convention as the substrate for fresh-context dispatch.
