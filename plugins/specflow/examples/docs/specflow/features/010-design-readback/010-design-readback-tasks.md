---
feature: 010-design-readback
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
prd: ./010-design-readback-prd.md
---

# 010-design-readback — task plan

## Coverage matrix

| Requirement | Tasks |
|---|---|
| R1 — prd Phase A.3 reads proposed.html | T-1 |
| R2 — prd Phase A.3 reads iteration-log.md | T-1 |
| R3 — task Phase A.2.5 partitions by timestamp | T-2 |
| R4 — silent no-op when design absent | T-1, T-2 |
| R5 — Design-intent bullet prefix | T-1 |
| R6 — design-decision: iteration-N field | T-2 |
| R7 — uncovered-iteration prompt | T-2 |

| AC | Tasks |
|---|---|
| AC-1 — design-intent bullet present | T-1 |
| AC-2 — no warning when absent | T-1 |
| AC-3 — design-decision field present | T-2 |
| AC-4 — uncovered-iteration prompt verbatim | T-2 |
| AC-5 — frontmatter unchanged | T-1, T-2 |
| AC-6 — worked example present | T-3 |

---

## T-1 — Edit `prd/SKILL.md` Phase A.3 to ingest design folder

- **Anchors:** R1, R2, R4, R5.
- **Acceptance:** Phase A.3 reads `features/NNN-{slug}/design/` when present; produces *Codebase context* bullets prefixed `Design intent (from design/):`; no warning when absent.
- **Lane:** Green (verifiable, low blast radius, isolated to one SKILL file).
- **Files:** `plugins/specflow/skills/prd/SKILL.md` (Phase A.3 only).
- **Estimate:** 15 minutes.

---

## T-2 — Add `task/SKILL.md` Phase A.2.5 (post-PRD iteration readback)

- **Anchors:** R3, R6, R7.
- **Acceptance:** Phase A.2.5 partitions iteration entries by timestamp vs PRD frontmatter date; tasks gain `design-decision: iteration-N` field when matched; uncovered entries surface the prompt verbatim.
- **Lane:** Green.
- **Files:** `plugins/specflow/skills/task/SKILL.md` (Phase A.2.5 only).
- **Estimate:** 20 minutes.
- **design-decision: iteration-3** — task synthesis surfaces the post-PRD `grouped-by-day` flip captured at `design/010-design-readback-iteration-log.md` iteration 3. The task carries this field per R6; reviewers at Gate 3 can grep for it to audit traceability.

---

## T-3 — Build worked example folder `010-design-readback/`

- **Anchors:** AC-6.
- **Acceptance:** `examples/docs/specflow/features/010-design-readback/` contains: PRD, interview, tasks file, design folder with proposed.html + iteration-log.md, Gate 2 manifest closed `passed`.
- **Lane:** Green.
- **Files:** New folder; no edits to existing files.
- **Estimate:** 25 minutes.
