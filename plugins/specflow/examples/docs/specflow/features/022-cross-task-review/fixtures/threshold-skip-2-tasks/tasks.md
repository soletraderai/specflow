---
feature: 999-trivial-feature
status: draft
created: 2026-05-07
prd: ./999-trivial-feature-prd.md
interview: ./999-trivial-feature-interview.md
gate3: ./debate-log/tasks-gate3/manifest.md
---

# Tasks — Trivial Feature

## Coverage matrix

| PRD requirement | Tasks satisfying it |
|---|---|
| R1 | T1 |
| R2 | T2 |

## Tasks

### T1 — Add config knob `feature.x.enabled`

- **Anchor:** PRD R1 — feature x can be toggled
- **Scope:** docs/specflow/admin/config.json
- **Acceptance:** AC-1 — config.feature.x.enabled defaults to true
- **Depends on:** none
- **context-budget-estimate:** 18000
- **Notes:** none

### T2 — Wire toggle into specflow:design

- **Anchor:** PRD R2 — design respects the toggle
- **Scope:** plugins/specflow/skills/design/SKILL.md
- **Acceptance:** AC-2 — design refuses with canonical sentinel when disabled
- **Depends on:** T1
- **context-budget-estimate:** 22000
- **Notes:** none

## Open questions inherited from PRD

(none)
