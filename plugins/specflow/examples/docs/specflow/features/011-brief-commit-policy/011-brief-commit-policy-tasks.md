---
feature: 011-brief-commit-policy
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
prd: ./011-brief-commit-policy-prd.md
---

# 011-brief-commit-policy — task plan

## Coverage matrix

| Requirement | Tasks |
|---|---|
| R1 — config schema gains brief.commitPolicy | T-1 |
| R2 — setup prompt | T-1 |
| R3 — .gitignore append on derived | T-1 |
| R4 — brief reads policy and renders banner | T-2 |
| R5 — fixed banner copy | T-2 |
| R6 — banner is deterministic | T-2 |
| R7 — MIGRATIONS entry | T-3 |

| AC | Tasks |
|---|---|
| AC-1, AC-2, AC-3 | T-1 |
| AC-4, AC-5, AC-6, AC-7 | T-2 |
| AC-8 | T-3 |

---

## T-1 — Edit `setup/SKILL.md` Phase 8.2 (prompt + schema + .gitignore)

- **Anchors:** R1, R2, R3.
- **Acceptance:** Phase 8.2 prompts user with the two-option choice; writes `brief.commitPolicy` to config.json with default `committed`; appends `*-brief.html` to `.gitignore` idempotently when user picks `derived`.
- **Lane:** Green (verifiable, low blast radius, isolated to setup/SKILL.md).
- **Files:** `plugins/specflow/skills/setup/SKILL.md` (Phase 8.2 only).
- **Estimate:** 15 minutes.

---

## T-2 — Edit `brief/SKILL.md` step 9 (banner render)

- **Anchors:** R4, R5, R6.
- **Acceptance:** Brief skill reads `config.json.brief.commitPolicy`, defaults to `committed` when absent, renders the verbatim banner copy at the top of the HTML, banner is part of the deterministic compose.
- **Lane:** Green.
- **Files:** `plugins/specflow/skills/brief/SKILL.md` (frontmatter `requires` + `eval` + step 9).
- **Estimate:** 15 minutes.

---

## T-3 — Add v2.3 → v2.4 MIGRATIONS entry

- **Anchors:** R7.
- **Acceptance:** `plugins/specflow/MIGRATIONS.md` has a v2.3 → v2.4 entry covering `brief.commitPolicy` field addition, default value, and `.gitignore` snippet for derived mode. Existing v2.3 projects upgrade idempotently.
- **Lane:** Green.
- **Files:** `plugins/specflow/MIGRATIONS.md` (single new entry).
- **Estimate:** 10 minutes.
- **Note:** Sprint 1 close batches this task with all other v2.3 → v2.4 entries (012 config-skill-toggles, 013 example-migration-policy frontmatter field, 014 team-bridge-spec one-pager). T-3 here is the brief.commitPolicy slice of the consolidated entry.
