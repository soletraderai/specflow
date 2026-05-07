---
feature: 012-config-skill-toggles
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
prd: ./012-config-skill-toggles-prd.md
---

# 012-config-skill-toggles — task plan

## Coverage matrix

| Requirement | Tasks |
|---|---|
| R1 — config schema gains skills block | T-1 |
| R2 — resolver contract doc | T-2 |
| R3 — develop Phase A.0 demonstrates pattern | T-3 |
| R4 — canonical refusal message | T-2, T-3 |
| R5 — disabled = atomic refuse-and-return | T-3 |
| R6 — upgrade behaviour | T-2 (doc), T-4 (MIGRATIONS) |
| R7 — incremental rollout | (this PRD's design intent — no per-task work) |

| AC | Tasks |
|---|---|
| AC-1 | T-1 |
| AC-2 | T-2 |
| AC-3, AC-4, AC-5, AC-6 | T-3 |
| AC-7 | T-4 |
| AC-8 | T-5 |

---

## T-1 — Edit `setup/SKILL.md` Phase 8.2 (schema seed)

- **Anchors:** R1.
- **Acceptance:** Phase 8.2 writes `config.json` with a `skills` block listing every shipped v2.4 skill, each with `{ "enabled": true }`.
- **Lane:** Green.
- **Files:** `plugins/specflow/skills/setup/SKILL.md` (Phase 8.2 only).
- **Estimate:** 10 minutes.

---

## T-2 — Author `templates/admin/skill-toggles.md` (resolver contract)

- **Anchors:** R2, R4 (canonical message), R6 (doc only).
- **Acceptance:** New doc exists, covers refusal message format, backward-compat (missing field == enabled), chain-breakage by design, upgrade handling.
- **Lane:** Green.
- **Files:** `plugins/specflow/templates/admin/skill-toggles.md` (new).
- **Estimate:** 20 minutes.

---

## T-3 — Edit `develop/SKILL.md` Phase A (add A.0)

- **Anchors:** R3, R4, R5.
- **Acceptance:** Develop Phase A gains a new A.0 sub-step before A.1. Reads `admin/config.json`, refuses with the canonical verbatim message when `develop.enabled === false`, returns. Treats missing field as enabled.
- **Lane:** Green.
- **Files:** `plugins/specflow/skills/develop/SKILL.md` (Phase A only).
- **Estimate:** 10 minutes.

---

## T-4 — Add v2.3 → v2.4 MIGRATIONS entry slice

- **Anchors:** R6.
- **Acceptance:** `MIGRATIONS.md` v2.3 → v2.4 entry covers the new `skills` field, default-true seeding, and `specflow:upgrade` behaviour (seed missing, preserve existing, warn on orphans).
- **Lane:** Green.
- **Files:** `plugins/specflow/MIGRATIONS.md` (entry slice; consolidated with 011 and 013-014 slices at sprint close).
- **Estimate:** 10 minutes.

---

## T-5 — Build worked example folder `012-config-skill-toggles/`

- **Anchors:** AC-8.
- **Acceptance:** Folder exists with PRD, interview, tasks file, Gate 2 manifest closed `passed`.
- **Lane:** Green.
- **Files:** New folder.
- **Estimate:** 25 minutes.
