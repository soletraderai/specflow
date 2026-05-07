# Gate 2 manifest — 012-config-skill-toggles PRD

**Feature:** 012-config-skill-toggles
**Gate:** 2 (PRD plan debate)
**Reviewers:** devils-advocate, simplicity-reviewer, surgical-reviewer, goal-driven-reviewer
**Codex:** unavailable (Gate 2 does not invoke Codex; Codex is reserved for Gate 5).
**Status:** **passed**
**Closed:** 2026-05-06

---

## Round 1 — parallel findings

**devils-advocate.** One finding, info-level: R7's "incremental rollout" relies on future skills adopting the pattern when next touched. Risk — a skill that doesn't get touched for 6+ months remains toggle-unaware; a user who disables it and runs it gets no refusal, just normal execution. *Counter (PRD author):* the v2.4 setup seeds the toggle at default-true; an unaware skill is harmless when the toggle is true. The risk surfaces only if a user explicitly sets `enabled: false` and the skill hasn't been updated — in which case they'll see normal execution and reasonably re-check their config; the visible-anomaly is acceptable for a transitional period. Status: kept as-is; track in PRD's Vision as "incremental rollout."

**simplicity-reviewer.** No findings — schema gains one nested block, contract is one doc, demonstration is one Phase A.0 sub-step. Smallest possible surface for the resolved policy.

**surgical-reviewer.** One finding, info-level: the canonical refusal message includes the literal string `config.skills.{skill}.enabled = false`, which is a copy-paste-able config path. Reviewers reading the message can grep for the exact field. Good. *No counter required.*

**goal-driven-reviewer.** Surfaces R5 (disabled = atomic refuse-and-return) — verifying R5 in AC-6 requires evidence that no scratch directory was created. AC-6 references "no scratch directory" but the test plan would need to grep `admin/scratch/` after invocation. Note for `specflow:test` plan: AC-6 needs a binary check on `test -d admin/scratch/{NNN-slug}-develop` after invocation. *Author response:* logged as a test-plan note; AC-6 is unchanged but the evaluation surface is captured for tasks/test phase.

---

## Round 2 — author response

(Per Round 1 counters.) No revisions to the PRD body required. One test-plan note logged from goal-driven-reviewer for AC-6 evaluation.

---

## Round 3 — reviewer sharpen

(skipped — Round 2 closed all open threads.)

---

## Closing decision

The PRD describes a single config field, a contract doc, and one demonstration in `specflow:develop`. The incremental rollout is justified (avoids touching every skill in one sprint), the backward-compat path is structural (missing field == enabled), and the canonical refusal message provides cross-skill consistency. All R/AC pairs are reachable.

**Status:** **passed**.

Tasks may proceed (`specflow:task 012-config-skill-toggles`).

---

*Orchestrator signature:* manifest closed at 2026-05-06.
