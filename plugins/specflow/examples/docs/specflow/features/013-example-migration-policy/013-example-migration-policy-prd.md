---
feature: 013-example-migration-policy
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
interview: ./013-example-migration-policy-interview.md
---

# 013 — worked-example versioning policy

## Vision

Schema-tag every worked example in `plugins/specflow/examples/docs/specflow/features/NNN-{slug}/` with a `templateVersion: vX.Y` field on its PRD/tasks/test frontmatter. The tag names the plugin minor version the example was authored against. When a future sprint changes a primary template (PRD shape, brief composition, develop phase ordering), the audit script lists examples with stale `templateVersion` and the release author chooses backfill / grandfather / retire per stale entry. The change pre-emptively mitigates the highest unnamed risk in the v2.4+ master plan: Sprint 2's template churn would otherwise silently invalidate every prior worked example, with no in-repo mechanism to detect or document the divergence. Versioning makes drift visible; visible drift is fixable drift.

## Problem

The v2.4+ plan's Sprint 2 is a template-disruption window: `015-key-features-section` adds a non-technical "Key Features" block to the PRD shape; `016-brief-enhancements` adds resources/decisions/phase-split framing to the brief shape. Without a versioning policy, every worked example authored before Sprint 2 silently violates the new PRD/brief contract. Setup users reading the examples for guidance get the *old* shape; reviewers checking new examples against the *new* shape see drift; and the dogfood discipline ("worked example must demonstrate new behaviour") becomes ambiguous — does the example demonstrate the *current* contract or the *contract at authorship time*? Without an answer, the examples fork into a maintenance graveyard. The fix is mechanical: tag every example with the version it was authored against, audit on sprint close, choose remediation per stale entry.

## Goals

- New frontmatter field `templateVersion: vX.Y` on every worked-example PRD, tasks file, and test file. Interview files are exempt (their shape is open-ended).
- Doctrine doc `plugins/specflow/templates/admin/example-versioning.md` defining the policy: when to tag, what `vX.Y` means, backfill rules (backfill / grandfather / retire), audit script.
- Bash audit script (in the doctrine doc) that lists every PRD's `templateVersion`, surfaces missing fields as `MISSING`, and groups by version. Run at sprint close.
- Backfill on Sprint 1: existing 001-008 worked examples get tagged `v2.3` (the version they were authored against). Sprint 1 features (009 + 010-014) get tagged `v2.3` as well — they were authored against the v2.3 template; v2.4 didn't change the template surface.
- Sprint 2 (the first real template-change release) triggers the first audit run; examples flagged stale go through the backfill / grandfather / retire decision per the doctrine.

## Non-goals

- **Auto-backfill on template changes.** The decision per stale example is human (backfill vs grandfather vs retire); auto-backfill risks rewriting examples whose intent is to demonstrate the *prior* shape on purpose.
- **Per-section versioning.** A worked example's `templateVersion` covers the whole example, not "the PRD section is v2.3 but the tasks section is v2.4". Per-section versioning multiplies the audit surface for marginal benefit.
- **Backporting policy to non-example artefacts.** Real PRDs in user repos are not subject to this policy — they're the user's content, not plugin shipped examples. The audit script targets `plugins/specflow/examples/` only.
- **Tooling automation beyond the bash script.** No GitHub Action, no CI hook, no pre-commit check. The audit is a manual sprint-close action; turning it into automation is a v2.5+ enhancement if dogfood surfaces the need.

## Users

- **Sprint-close release authors** running the audit script. They benefit from a single command that lists every example's version state; they make per-stale-example decisions based on the listing.
- **Setup users reading worked examples** as documentation. They benefit from the version field implicitly: a v2.4 user reading a `templateVersion: v2.3` example knows the example was authored against the prior template and either (a) was backfilled or (b) grandfathered with intent. The `examples/README.md` § "Template-version index" surfaces the choices.
- **Plugin contributors** writing new worked examples. They benefit from a documented field they include reflexively; the doctrine doc tells them what version to use (the plugin minor version they're authoring against).

## Requirements

- **R1.** Every worked-example PRD frontmatter MUST include `templateVersion: vX.Y`. The tasks file and test file MUST include the same field when they exist. Interview files MUST NOT include the field (informational fail-fast on mistyped frontmatter).
  - Trace: templates/admin/example-versioning.md § The policy.
  - Serves goal: schema is uniform across examples.

- **R2.** Doctrine doc at `plugins/specflow/templates/admin/example-versioning.md` defines: rationale, the policy, backfill rules (backfill / grandfather / retire), the audit script, the v2.3 backfill plan for existing examples.
  - Trace: templates/admin/example-versioning.md (new in v2.4.0).
  - Serves goal: the policy is documented once; release authors follow it.

- **R3.** Audit script (bash) in the doctrine doc lists every PRD's `templateVersion`, surfaces missing fields as `MISSING`, groups by version. Run at sprint close.
  - Trace: templates/admin/example-versioning.md § Audit tooling.
  - Serves goal: the audit is one-shot and inspectable.

- **R4.** Sprint 1 backfill: existing 001-008 examples gain `templateVersion: v2.3` on their PRD frontmatter. Sprint 1 features (009 through 014) author with `templateVersion: v2.3`.
  - Trace: this PRD's Vision + the inserted-frontmatter on existing 001-008 PRDs.
  - Serves goal: baseline is set before Sprint 2 template churn.

- **R5.** Sprint 2's release process triggers the first audit run. Backfill / grandfather / retire decisions are recorded in the v2.5 release's CHANGELOG entry (one line per stale example explaining the choice).
  - Trace: this PRD's Vision + Sprint 2 close action item in the master plan.
  - Serves goal: backfill decisions are auditable.

- **R6.** No automation beyond the bash script. CI / pre-commit / GH Action enforcement is a v2.5+ candidate; v2.4 ships the manual policy.
  - Trace: this PRD's Non-goals + master plan iteration discipline.
  - Serves goal: small, bounded change.

## Acceptance criteria

- **AC-1.** `plugins/specflow/templates/admin/example-versioning.md` exists and contains the policy, backfill rules, and audit script. Verifies R2.
- **AC-2.** Every PRD under `examples/docs/specflow/features/00[1-8]-*/` carries `templateVersion: v2.3` in its frontmatter. Verifies R4 backfill.
- **AC-3.** Every Sprint 1 worked example (009 / 010 / 011 / 012 / 013 / 014) carries `templateVersion: v2.3` in its PRD frontmatter. Verifies R4 baseline.
- **AC-4.** The audit script in the doctrine doc, when run from the repo root, lists every example PRD's `templateVersion` and groups by version. Verifies R3.
- **AC-5.** v2.3 → v2.4 MIGRATIONS entry references the new field, the doctrine doc, and the backfill action. Verifies R5.
- **AC-6.** Worked example folder `examples/docs/specflow/features/013-example-migration-policy/` exists with PRD, interview, tasks, Gate 2 manifest closed `passed`, and the `templateVersion: v2.3` frontmatter. Verifies R1 + the dogfood discipline.

## Open questions

None — the policy is small, manual, and bounded.

## See also

- `plugins/specflow/templates/admin/example-versioning.md` — the doctrine
- `plugins/specflow/MIGRATIONS.md` v2.3 → v2.4 — the migration entry (Sprint 1 close)
- `v2/docs/PRD.md` § Resolved decisions — the resolution citation (mitigates Risk A from master plan)
