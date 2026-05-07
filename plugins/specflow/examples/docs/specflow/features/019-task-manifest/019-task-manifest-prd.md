---
feature: 019-task-manifest
status: shipped
created: 2026-05-08
templateVersion: v2.6
shipped: v2.6.0
interview: ./019-task-manifest-interview.md
---

# Task manifest

## Vision

A standardised per-task manifest at `debate-log/tasks/T-NN-manifest.md` accumulates entries from the moment the task is born (`specflow:task` synthesis) through to validation (`specflow:complete` retro). Every agent invoked against a task reads the manifest before contributing; entries follow a YAML-frontmatter shape with `agent_id` (per 027) + `phase` + `event_type` + `input_ref` + `output_ref` + `body` + `outcome`. The schema lives in `plugins/specflow/templates/admin/task-manifest-schema.md`.

Replaces 017's interim `admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md` format.

## Problem

Pre-019, per-task lifecycle data lived in:
- The Gate 3 manifest (per-task findings interleaved with cross-task findings).
- The Gate 4 + Gate 5 manifests (one per task, but separate from Gate 3).
- The 017 manifest stub at `admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md` (Red/Green/Refactor markers only).
- The retro file (whole-feature, not per-task).
- `task-history.json` (development-time fields per task, but no narrative).

No single per-task surface accumulated everything; agents invoked at later phases couldn't see what earlier agents had decided about this specific task without reading three or four artefacts. 019 unifies.

## Goals

- Doctrine doc at `plugins/specflow/templates/admin/task-manifest-schema.md` defining: read-first contract, standardised entry format, lifecycle phases captured, cross-feature integration with 017/018/022/027/028, migration from 017's stub.
- Manifest path: `debate-log/tasks/T-NN-manifest.md` — opened at synthesis (`specflow:task` Phase E.6), closed at completion (`specflow:complete` retro).
- Read-first enforcement at the orchestrator's pre-dispatch step.

## Non-goals

- Migrating existing in-flight features to 019's format. New features land in 019's format; existing features keep their interim shape until they ship.
- Building a manifest viewer / renderer. The manifest is read by agents (machine-parseable YAML-frontmatter) and humans (Markdown-rendered).

## Requirements

- **R1.** Doctrine doc at `plugins/specflow/templates/admin/task-manifest-schema.md` with the schema, read-first contract, lifecycle phases, migration notes.
- **R2.** `specflow:task` Phase E.6 opens the manifest with the initial `task-creation` entry citing the lessons.json query result (per 018) and the synthesis-time `prior-lessons` field.
- **R3.** `specflow:develop` Phase D appends `develop-red` / `develop-green` / `develop-refactor` entries per the 017 cycle. The 017 interim stub format is migrated as documented.
- **R4.** `specflow:test` Phase D feedback-mode + `specflow:complete` retro append entries per the lifecycle table.
- **R5.** Every agent's role-def file under `admin/agents/standard/{lifecycle,principles}/` carries the read-first line.

## Acceptance criteria

- **AC-1.** Doctrine doc exists at the named path with all sections.
- **AC-2.** `task/SKILL.md` Phase E.6 emits a `task-creation` entry to `debate-log/tasks/T-NN-manifest.md` for every synthesised task.
- **AC-3.** `develop/SKILL.md` Phase D writes Red / Green / Refactor entries to the manifest (replacing 017's scratch stub).
- **AC-4.** `complete/SKILL.md` retro appends a `sign-off` entry per task.

## See also

- Doctrine: `plugins/specflow/templates/admin/task-manifest-schema.md`
- 017 — tdd-discipline (interim stub format that 019 replaces)
- 018 — lessons-registry (task-creation entries cite lessons)
- 022 — cross-task-review (cross-task findings land in affected manifests)
- 027 — reviewer-context-isolation (manifest is the declared-input context)
- 028 — edge-case-reviewer (recommendation + reasoning lands as entry body)
