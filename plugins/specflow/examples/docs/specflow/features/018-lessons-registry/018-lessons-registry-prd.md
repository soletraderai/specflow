---
feature: 018-lessons-registry
status: shipped
created: 2026-05-08
templateVersion: v2.6
shipped: v2.6.0
interview: ./018-lessons-registry-interview.md
---

# Lessons registry

## Vision

Anchor the in-flight `lessons.json` self-learning registry as a formal feature with a closed read-write loop across the pipeline. Schema, query algorithm, and write paths live in `plugins/specflow/templates/admin/lessons-registry.md`. PRD Phase A.3.5 (NEW) and task Phase A.4 (already implemented) query the registry; test Phase D feedback-mode and complete retro append. Insights clusters lessons and promotes ≥3-occurrence patterns to `admin/rules/guidelines.md`. Develop never queries directly — by design.

## Problem

Pre-018, lessons.json existed as a write log (test --feedback wrote; complete retro wrote; insights clustered) but the read path was implicit. PRD synthesis, task synthesis, and cross-task review consulted environment.json + glossary + tag heuristics piecewise without a unified contract. The user's emphasis: continuous auto-research / self-improving / self-learning is only half the story — the other half is **passing prior learnings forward into early stages** so the system knows what's been successful previously, especially at PRD time and task synthesis.

## Goals

- Doctrine doc at `plugins/specflow/templates/admin/lessons-registry.md` spelling out: schema (id / created / tags / surface / outcome / context / lesson / source / confidence / superseded_by / status), write paths, read paths, query algorithm, config knobs.
- `specflow:prd` Phase A.3.5 (NEW between A.3 and A.4) — query lessons by feature tag profile; surface inline in interview's Codebase context as "What we've learned before that applies here".
- `specflow:task` Phase B.3 — every task entry gains `prior-lessons: [L-NNN, ...]` field; existing A.4 lessons-query stays.
- `config.json.prd.maxLessonsSurfaced` (default 5) and `config.json.task.maxLessonsSurfaced` (default 5).

## Non-goals

- Migrating the existing lessons.json shape. The schema documented here is what's already written; the doctrine doc formalises rather than redefines.
- Auto-superseding lessons. Supersession remains a human-driven retro decision, never an algorithmic one.
- Surfacing lessons during develop. The plan-time surfacing is sufficient; develop executes the plan.

## Requirements

- **R1.** Doctrine doc exists at `plugins/specflow/templates/admin/lessons-registry.md` with the schema, query algorithm, write paths, read paths, config knobs.
- **R2.** `specflow:prd` Phase A.3.5 queries lessons.json after A.3 codebase context and before A.4 interview file write. Surfaces inline as "What we've learned before that applies here" subsection.
- **R3.** `specflow:task` per-task entry carries `prior-lessons: [L-NNN, ...]` field (empty array when no matches).
- **R4.** `config.json.prd.maxLessonsSurfaced` (default 5) and `config.json.task.maxLessonsSurfaced` (default 5) are documented in `setup/SKILL.md` Phase 8.2.

## Acceptance criteria

- **AC-1.** Doctrine doc at the named path with schema + read/write paths + query algorithm.
- **AC-2.** `prd/SKILL.md` Phase A.3.5 cites `templates/admin/lessons-registry.md` and surfaces inline.
- **AC-3.** `task/SKILL.md` Phase B.3 write template includes `prior-lessons:` field.
- **AC-4.** `setup/SKILL.md` Phase 8.2 mentions `prd.maxLessonsSurfaced` and `task.maxLessonsSurfaced`.

## See also

- Doctrine: `plugins/specflow/templates/admin/lessons-registry.md`
- 022 — cross-task-review (consults lessons in better-arrangement lens)
- 019 — task-manifest (cites lessons that shaped each task at task-creation entry)
