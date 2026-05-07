---
feature: 028-edge-case-reviewer
status: shipped
created: 2026-05-08
templateVersion: v2.6
shipped: v2.6.0
interview: ./028-edge-case-reviewer-interview.md
---

# Edge-case reviewer

## Vision

A new principle reviewer (`edge-case-reviewer`) joins the Gate 4 + Gate 5 reviewer set. Its primary pass is **deliberately not goal-aware** — it asks five questions that surface what the work *inherits* from the codebase, the runtime, and the user environment, that the goal-focused reviewers won't catch. Findings carry `recommendation` + `reasoning` and are advisory; the orchestrator decides accept / reject / defer-to-misc per finding.

The reviewer's role-def lives at `plugins/specflow/examples/docs/specflow/admin/agents/standard/principles/edge-case-reviewer.md`. It conforms to 027-reviewer-context-isolation on landing (no retrofit pass).

## Problem

Goal-Driven Reviewer's reverse-traceability lens is necessary but creates a blindspot: it certifies "every R/AC has a task / every plan step traces to an R" but cannot see what's *missing* from the R/AC list. Edge cases that grilling cannot catch (per `knowledge/pocock-real-feature-build.md`) — non-git-repo rollback, empty input, partial input, malformed input, missing dependency, inherited error-handling that swallows failures — surface only in QA. Without an explicit lens for the gap, they ship.

## Goals

- New role-def at `examples/docs/specflow/admin/agents/standard/principles/edge-case-reviewer.md` documenting the five-question lens, the deliberately-not-goal-aware contract, the advisory output shape (recommendation + reasoning), and the 027 conformance.
- Add the reviewer to Gate 4 and Gate 5 reviewer sets in `develop/SKILL.md`.
- Findings are advisory — orchestrator decides accept / reject / defer-to-misc per finding. Decisions land as the next entry in the per-task manifest (per 019).

## Non-goals

- Auto-applying edge-case findings. The advisory-not-auto-applied surface is the load-bearing reason the reviewer exists.
- Folding the reviewer's role into Goal-Driven. The blindspot is the load-bearing reason for two roles.
- Surfacing edge cases at Gate 2 or Gate 3. The plan and code surfaces are where edge-case inheritance shows; the PRD and task surfaces are the wrong abstraction layer.

## Requirements

- **R1.** Role-def file at `examples/docs/specflow/admin/agents/standard/principles/edge-case-reviewer.md` documents the five-question lens, the not-goal-aware contract, the advisory finding shape with `recommendation` + `reasoning` fields.
- **R2.** `develop/SKILL.md` Phase C.2 (Gate 4) reviewer set includes `edge-case-reviewer`.
- **R3.** `develop/SKILL.md` Phase E.2 (Gate 5) reviewer set includes `edge-case-reviewer`.
- **R4.** Findings are advisory; the orchestrator's decision (accept / reject / defer-to-misc) lands as the next entry in the per-task manifest (per 019).
- **R5.** The reviewer conforms to 027-reviewer-context-isolation on landing — fresh-context spawn, declared-input-only, `writer_id ≠ edge-case-reviewer agent_id`.

## Acceptance criteria

- **AC-1.** Role-def file exists with the five-question lens enumerated and the not-goal-aware contract stated explicitly.
- **AC-2.** `develop/SKILL.md` Phase C.2 includes `edge-case-reviewer` in the bullet list.
- **AC-3.** `develop/SKILL.md` Phase E.2 includes `edge-case-reviewer` in the bullet list.
- **AC-4.** The role-def carries the literal "Does NOT consult the writer's chat" line + the `writer_id` non-equality clause + citation to `templates/admin/reviewer-isolation.md`.

## See also

- Role-def: `examples/docs/specflow/admin/agents/standard/principles/edge-case-reviewer.md`
- 027 — reviewer-context-isolation (reviewer conforms on landing)
- 019 — task-manifest (orchestrator's decisions land as entries)
