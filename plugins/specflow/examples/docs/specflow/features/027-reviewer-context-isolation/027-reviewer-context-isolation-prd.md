---
feature: 027-reviewer-context-isolation
status: shipped
created: 2026-05-08
templateVersion: v2.6
shipped: v2.6.0
interview: ./027-reviewer-context-isolation-interview.md
---

# Reviewer context isolation

## Vision

Across every multi-agent debate gate (Gate 2, Gate 3, Gate 4, Gate 5), every reviewer runs in a fresh context with zero exposure to the writer's chat history. The contract — fresh-context spawn, declared-input-only, pairwise-non-equal `agent_id` — lives in `plugins/specflow/templates/admin/reviewer-isolation.md`. 027 absorbs 022's interim manifest-field convention, adds the format contract, and adds the runtime collision check.

## Problem

Pre-027, reviewer isolation was best-effort: forked sub-agent dispatch was the norm but not contractual; writer's-chat exposure was not explicitly forbidden in role-defs; manifest `agent_id` fields existed at Gate 3 (per 022) but had no format contract and no runtime verification. When the same agent identity wrote AND validated the same artefact within a feature (e.g. same fork lineage, no runtime check), the bookkeeping artefact said "reviewed" but the reasoning was shared. Medin's reviewer-isolation discipline (`knowledge/medin-parallel-agentic-playbook.md` § Reviewer isolation) is the load-bearing antidote; Pocock's smart-zone-cliff observation (`knowledge/pocock-ai-coding-real-engineers.md` § LLM Constraint 1) is the second pressure point — reviewers sharing the writer's context start in the dumb zone.

## Goals

- Doctrine doc at `plugins/specflow/templates/admin/reviewer-isolation.md` defining: rationale, contract clauses, agent_id format, runtime verification, cross-cutting impact, what 027 absorbs from 022, what 027 enables for 028.
- Extend `templates/orchestrator-pattern.md` with a "Reviewer fresh-context dispatch" section.
- Each principle reviewer's role-def (`templates/agents/standard/principles/*.md`) carries the line *"Does NOT consult the writer's chat or the orchestrator's deliberation transcripts."* + the `agent_id` non-equality clause.
- Skill bodies (`prd`, `task`, `develop`) emit `writer_id` + `reviewer_ids` block at gate dispatch and verify pairwise non-equality at gate close.
- 028's edge-case-reviewer conforms to the contract on landing (no retrofit pass).

## Non-goals

- Building 028 itself (Sprint 3 sibling). 027 lays the substrate; 028 conforms.
- Changing the per-task review behaviour. The contract applies to existing reviewers without changing their lenses.
- Removing `agent-teams:team-review` interactions at Gate 5. The bridge mapping (per `templates/admin/team-review-bridge.md`) stands; team-review-bridge participants are subject to the same isolation contract.
- Cross-feature `agent_id` reuse policy. Within-feature is the contract surface; cross-feature is unconstrained (the orchestrator's run-id space is per-feature).

## Requirements

- **R1.** Doctrine doc `plugins/specflow/templates/admin/reviewer-isolation.md` exists with: rationale, contract clauses (fresh subagent spawn, explicit input listing, pairwise-non-equal `agent_id`, role-def writer's-chat prohibition), `agent_id` format contract (harness-emitted run ID + slot suffix; ISO-8601-with-suffix fallback), runtime verification pseudocode, cross-cutting impact table.
  - Trace: doctrine-doc home pattern (mirrors `single-context-task.md` and `tdd-discipline.md`).
  - Serves goal: single citation point.

- **R2.** Each principle reviewer template at `templates/agents/standard/principles/*.md` carries:
  - The line *"Does NOT consult the writer's chat or the orchestrator's deliberation transcripts."*
  - The `writer_id ≠ {reviewer-name} agent_id` non-equality clause.
  - Citation to `templates/admin/reviewer-isolation.md`.
  - Trace: contract clause #4 in the doctrine doc.
  - Serves goal: every reviewer's role-def documents the contract.

- **R3.** `templates/orchestrator-pattern.md` gains a "Reviewer fresh-context dispatch" section between the three primitives and the skill-author checklist. The checklist itself adds one bullet: *"Every reviewer dispatch in a multi-agent gate honours the fresh-context contract."*
  - Trace: orchestrator-pattern is the operational substrate for fresh-context dispatch.
  - Serves goal: orchestrators consume the contract.

- **R4.** Gate manifests (Gate 2 / 3 / 4 / 5) carry the `writer_id` + `reviewer_ids` block. The format is documented in the doctrine doc; skill bodies cite the doctrine for the format and emit the block at gate close.
  - Trace: Gate 3's existing `writer_id` / `cross_task_reviewer_id` / `applier_id` block (per 022) is the substrate; 027 generalises and standardises.
  - Serves goal: pairwise-non-equal verification has a manifest surface.

- **R5.** The closer's `agent_id` collision check runs at every gate's E.6 (or equivalent) close step. A `FRESH-CONTEXT-VIOLATION` aborts the gate close with status `failed`.
  - Trace: contract clause #3 in the doctrine doc.
  - Serves goal: runtime verification (not advisory).

- **R6.** 022's manifest-field convention is preserved verbatim — field names (`writer_id`, `cross_task_reviewer_id`, `applier_id`) and populate-at-dispatch convention. 027 adds the format contract and runtime verification on top; the field shape is unchanged so 022's worked examples remain valid.
  - Trace: chain-don't-absorb continuity with 022.
  - Serves goal: backward compatibility.

## Acceptance criteria

- **AC-1.** `plugins/specflow/templates/admin/reviewer-isolation.md` exists with all sections from R1.
  ```sh
  test -f plugins/specflow/templates/admin/reviewer-isolation.md
  grep -q '## Why this exists' plugins/specflow/templates/admin/reviewer-isolation.md
  grep -q '## The contract' plugins/specflow/templates/admin/reviewer-isolation.md
  grep -q '## Format contract for `agent_id`' plugins/specflow/templates/admin/reviewer-isolation.md
  grep -q '## Runtime verification' plugins/specflow/templates/admin/reviewer-isolation.md
  ```
  - Verifies: R1.

- **AC-2.** Each principle reviewer template carries the writer's-chat prohibition + `writer_id ≠ {reviewer} agent_id` clause + citation:
  ```sh
  for f in plugins/specflow/templates/agents/standard/principles/*.md; do
    grep -q "writer's chat" "$f" || exit 1
    grep -q "writer_id" "$f" || exit 1
    grep -q "templates/admin/reviewer-isolation.md" "$f" || exit 1
  done
  ```
  - Verifies: R2.

- **AC-3.** `templates/orchestrator-pattern.md` has the "Reviewer fresh-context dispatch" section + the additional checklist bullet:
  ```sh
  grep -q '## Reviewer fresh-context dispatch' plugins/specflow/templates/orchestrator-pattern.md
  grep -qE 'fresh-context contract' plugins/specflow/templates/orchestrator-pattern.md
  ```
  - Verifies: R3.

- **AC-4.** Worked example demonstrates the contract — `agent_id` triplet at Gate 2, all three populated and pairwise-distinct:
  ```sh
  FIX=plugins/specflow/examples/docs/specflow/features/027-reviewer-context-isolation/debate-log/prd-gate2/manifest.md
  test -f "$FIX"
  grep -qE '\*\*writer_id:\*\*' "$FIX"
  grep -qE 'reviewer_ids:' "$FIX"
  ```
  - Verifies: R4.

## Open questions

None — the contract was already substantially in place via 022's interim mechanism; 027 formalises.

## See also

- Doctrine: `plugins/specflow/templates/admin/reviewer-isolation.md`
- Orchestrator pattern: `plugins/specflow/templates/orchestrator-pattern.md` § Reviewer fresh-context dispatch
- Reviewer templates: `plugins/specflow/templates/agents/standard/principles/*.md`
- 022 — cross-task-review (interim manifest-field convention 027 absorbs)
- 028 — edge-case-reviewer (sibling Sprint 3 feature; conforms on landing)
