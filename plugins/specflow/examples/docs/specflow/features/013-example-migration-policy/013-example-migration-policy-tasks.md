---
feature: 013-example-migration-policy
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
prd: ./013-example-migration-policy-prd.md
---

# 013-example-migration-policy — task plan

## Coverage matrix

| Requirement | Tasks |
|---|---|
| R1 — frontmatter field on PRD/tasks/test | T-1 (doctrine spec), T-3 (backfill applies it) |
| R2 — doctrine doc | T-1 |
| R3 — audit script | T-1 (script lives in the doctrine doc) |
| R4 — Sprint 1 backfill | T-3 |
| R5 — Sprint 2 audit run record | (Sprint 2 close action) |
| R6 — no automation beyond bash | T-1 |

| AC | Tasks |
|---|---|
| AC-1 | T-1 |
| AC-2 | T-3 |
| AC-3 | T-2 (Sprint 1 worked examples self-tag) |
| AC-4 | T-1 (script in doctrine) |
| AC-5 | T-4 (MIGRATIONS slice) |
| AC-6 | T-2 |

---

## T-1 — Author `templates/admin/example-versioning.md` (doctrine + audit script)

- **Anchors:** R1, R2, R3, R6.
- **Acceptance:** Doctrine doc exists; covers rationale, policy, backfill rules, audit script, v2.3 backfill plan.
- **Lane:** Green.
- **Files:** `plugins/specflow/templates/admin/example-versioning.md` (new).
- **Estimate:** 25 minutes.

---

## T-2 — Author Sprint 1 worked examples with the field

- **Anchors:** R4 baseline, AC-3, AC-6.
- **Acceptance:** Every Sprint 1 worked example PRD (009/010/011/012/013/014) carries `templateVersion: v2.3` in frontmatter.
- **Lane:** Green.
- **Files:** `examples/docs/specflow/features/{009-014}-*/` PRDs as authored during their respective tasks.
- **Estimate:** 0 minutes incremental — covered as part of each Sprint 1 feature's worked-example authoring.

---

## T-3 — Backfill 001-008 worked examples

- **Anchors:** R4 backfill, AC-2.
- **Acceptance:** Every PRD under `examples/docs/specflow/features/00[1-8]-*-/` carries `templateVersion: v2.3` immediately after the opening `---`.
- **Lane:** Green (mechanical frontmatter edit; no semantic content change).
- **Files:** All eight 001-008 PRDs.
- **Estimate:** 5 minutes (one-shot awk command).

---

## T-4 — Add v2.3 → v2.4 MIGRATIONS entry slice

- **Anchors:** R5 (deferred to Sprint 2 close), AC-5 (Sprint 1 close slice).
- **Acceptance:** `MIGRATIONS.md` v2.3 → v2.4 entry references the new field, the doctrine doc, and the v2.3 backfill action.
- **Lane:** Green.
- **Files:** `plugins/specflow/MIGRATIONS.md` (entry slice; consolidated at sprint close).
- **Estimate:** 5 minutes.
