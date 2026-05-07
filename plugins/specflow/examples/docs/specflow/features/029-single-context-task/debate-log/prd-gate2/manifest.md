# Gate 2 manifest — 029-single-context-task PRD

**Feature:** 029-single-context-task
**Gate:** 2 (PRD plan debate)
**Date:** 2026-05-07
**writer_id:** prd-author
**reviewer_ids:** [orchestrator, devils-advocate, goal-driven-reviewer]
**Reviewer-isolation contract (per 027):** every reviewer ran in a fresh context with zero exposure to the writer's chat. Writer ↔ reviewer intersection is empty.
**Codex:** unavailable (Gate 2 does not invoke Codex; Codex is reserved for Gate 5).
**Status:** **passed**
**Closed:** 2026-05-07

---

## Round 1 — parallel findings

**devils-advocate.** One finding, info-level: R5's three-option developer prompt at ≥20% divergence introduces a calibration knob (the 20% threshold). What protects this number from drifting? Risk — projects with optimistic synthesis estimates may set the threshold lower (10%) to catch under-estimation, while projects with conservative estimates may relax to 30%. The doctrine doc does not currently surface the threshold as configurable. *Counter (PRD author):* the 20% is empirical (matches the ±20% headroom band the 80K default budget carries within the ~100K cliff). Promoting it to `config.json.task.contextBudgetTolerance` would multiply the surface for marginal flexibility — projects that need a tighter threshold can tighten the budget itself instead. Status: not promoted to config; the doctrine doc adds a one-line note that 20% is the empirical band, not a knob. Resolved during synthesis (T-1 doctrine doc § How develop Phase A pre-flights gained the empirical-band note).

**goal-driven-reviewer.** One finding, info-level: AC-7 says "a compaction event mid-task produces a manifest entry" — but the manifest is owned by 019-task-manifest, which has not yet shipped (Sprint 3). This feature's worked example cannot demonstrate AC-7 without 019's manifest schema. *Counter (PRD author):* AC-7 is a forward-coverage anchor — when 019 lands, its manifest schema will need to accept the `event_type: escalation` outcome `compacted-mid-task` shape. The doctrine doc names the field; 019 honours the name. The worked example demonstrates the *doctrine* contract (the doc names the field), not the runtime emission (which requires 019). T-4 in tasks.md is the verification slice for the doctrine-side contract. Status: AC-7 reframed in the PRD body to "the doctrine doc names the manifest field shape" rather than "a compaction event produces an entry"; runtime emission deferred to 019. Resolved during synthesis.

**orchestrator.** No standalone findings — orchestrator role here is to collate and close. Read both Round-1 findings, applied the closer logic from `lifecycle/orchestrator.md`, confirmed no `block` severities, no unconverged threads.

---

## Round 2 — author response

Two info-level findings, both resolved during synthesis without PRD-body revisions to the requirements list. The doctrine doc gained a one-line empirical-band note (per devils-advocate); AC-7 was reframed (per goal-driven-reviewer) to bind the doctrine-side contract, not the runtime emission. Neither revision crosses a load-bearing PRD boundary; the PRD body's R/AC list is unchanged.

---

## Round 3 — reviewer sharpen

(skipped — Round 2 closed all open threads; no reviewer requested sharpening.)

---

## Closing decision

The PRD describes a single doctrine doc, a `context-budget-estimate` field on task synthesis (cited rather than inlined per chain-don't-absorb), a Phase A pre-flight on develop, a one-line reminder at Phase D entry, and a worked example that meta-demonstrates the auto-flag-for-split path. The 80K default sits inside the empirical smart-zone cliff with headroom; the ≥20% divergence threshold matches the empirical band. AC-7's forward-coverage framing aligns with 019's planned shape (Sprint 3) without creating a hard cross-feature dependency. All R / AC pairs are reachable with the slice of work that lands in v2.5.0.

The reviewer-isolation contract held: writer_id `prd-author` and reviewer_ids `[orchestrator, devils-advocate, goal-driven-reviewer]` have empty intersection. Per 027, the reviewers ran in fresh contexts with zero exposure to the writer's grilling chat — the artefact under review (PRD draft + interview log) was their only input.

**Status:** **passed**.

Tasks may proceed (`specflow:task 029-single-context-task`).

---

*Orchestrator signature:* manifest closed at 2026-05-07.
