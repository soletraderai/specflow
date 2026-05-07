# Gate 2 manifest — 011-brief-commit-policy PRD

**Feature:** 011-brief-commit-policy
**Gate:** 2 (PRD plan debate)
**Reviewers:** devils-advocate, simplicity-reviewer, goal-driven-reviewer
**Codex:** unavailable (Gate 2 does not invoke Codex; Codex is reserved for Gate 5).
**Status:** **passed**
**Closed:** 2026-05-06

---

## Round 1 — parallel findings

**devils-advocate.** One observation, info-level: the no-goal "no per-feature override" might be too rigid for projects with one-off compliance briefs that need committing while project default is `derived`. *Counter (PRD author):* gitignore exclusions (`!features/special/*-brief.html`) handle the one-off case via standard git tooling without expanding the skill's surface. The trade-off is documented in the no-goals section. Status: kept as no-goal; the gitignore-exclusion path is the documented workaround.

**simplicity-reviewer.** No findings — the schema gains one field, the setup gains one prompt, the brief gains one banner. Smallest possible surface for the resolved policy.

**goal-driven-reviewer.** No orphan ACs — every AC traces. Light note: AC-7 (backward-compat for v2.2/v2.3 projects without the field) is the load-bearing AC for projects upgrading; surface explicitly in the migration entry. *Author response:* R7 already captures the migration scope; AC-7 is the binary verification. Kept as-is.

---

## Round 2 — author response

(Per Round 1 counters.) No revisions to the PRD body required.

---

## Round 3 — reviewer sharpen

(skipped — Round 2 closed all open threads.)

---

## Closing decision

The PRD describes a small, bounded knob with explicit defaults, an idempotent setup-time prompt, and a deterministic banner. All R/AC pairs are reachable. The no-goals section explicitly excludes the marginal-benefit cases (per-feature override, retroactive cleanup, banner customisation).

**Status:** **passed**.

Tasks may proceed (`specflow:task 011-brief-commit-policy`).

---

*Orchestrator signature:* manifest closed at 2026-05-06.
