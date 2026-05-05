# Decision log

The *why* behind non-obvious choices in this project. One entry per decision; entries do not get edited after the fact — superseding decisions get a new entry that links back.

Format per entry:
- **Title** — short, declarative.
- **Date** — YYYY-MM-DD.
- **Context** — what triggered the decision.
- **Decision** — what we chose.
- **Rationale** — why this over the alternatives.
- **Related** — files, tasks, or PRDs touched.

---

## Composition over inheritance in `src/services/`

**Date:** 2026-01-08
**Context:** A `BaseService` class introduced in Q4 2025 to share retry, logging, and audit-trail behaviour across services had grown to 340 lines, four levels of inheritance, and three of the five engineers had stopped reading it before extending it. Two production bugs in December were traced to the base class's behaviour being misunderstood at the call site.
**Decision:** Remove the `BaseService` hierarchy. Services become small functions composed with their dependencies (logger, audit helper, retry wrapper) injected at the call site.
**Rationale:** The base class's flexibility was speculative — three of its five extension points had no second consumer. The composition pattern keeps each service legible in one file (Simplicity First, line "local reasoning beats cross-file elegance"), preserves the audit-trail discipline through explicit injection, and the migration cost was 2 engineer-days against an estimated 4-6 days of recurring confusion-cost over the next quarter.
**Related:** `src/services/orders/orders-service.ts` (reference implementation), task-history entry T-2026-01-T1, `rules/guidelines.md` rule `PREFER_COMPOSITION_OVER_INHERITANCE`.

---

## Tests live alongside source in `__tests__/` directories

**Date:** 2026-02-14
**Context:** The repo had drifted into two test conventions: top-level `tests/` mirroring the `src/` tree (legacy), and sibling `__tests__/` directories (newer modules). Discoverability was the recurring complaint — engineers added a test in one location while the related code lived under the other convention.
**Decision:** Standardise on sibling `__tests__/` directories. The top-level `tests/` tree is reserved for cross-module integration suites that don't belong to any single module.
**Rationale:** Test files are co-edited with their source 90%+ of the time (per a one-week sample of git diffs). Co-location halves the cognitive overhead of finding the test for a given module. Top-level `tests/` is preserved for the rare cross-module case where the convention would be misleading. Migration was scripted and ran in a single PR.
**Related:** `rules/guidelines.md` rule `PREFER_LOCAL_TESTS`, `rules/glossary.md` matching entry, `scripts/migrate-tests.ts` (one-shot migration, since deleted).

---

## Slack removed from the stack

**Date:** 2026-03-22
**Context:** Slack was being used for release announcements, on-call escalation, and decision-making conversations that should have lived in Linear or the decision log. Two recent decisions were "remembered" inconsistently across team members because the source was a Slack thread that had aged out of the free tier's history retention.
**Decision:** Remove Slack from the team's working set. Release announcements move to Linear comments tagged on the relevant project. On-call escalation moves to PagerDuty. Decision-making conversations are required to land in this decision log within 24 hours of the call being made.
**Rationale:** Slack's value was real-time chat; its cost was disposable history of decisions. The discipline of writing decisions to a versioned file forces the rationale to be clear at the time of the decision (Think Before Coding, line "state assumptions explicitly"). PagerDuty already covers the on-call escalation case more reliably. Cost saved: ~$280/month in seat fees + the recurring confusion-cost of decisions decaying.
**Related:** `CONTEXT.md` Recent context entry of the same date.

---

## Adopt specflow for feature lifecycle

**Date:** 2026-04-12
**Context:** Feature work was being tracked across Linear (issues), GitHub PRs (code review), and a per-feature Notion page (PRD-shaped doc). The PRD was the weak link — half the time it was missing entirely; the rest of the time it drifted from what shipped, because nobody enforced the round-trip from PRD to acceptance criteria to tests.
**Decision:** Adopt the specflow plugin for the feature lifecycle. PRDs land in `docs/specflow/features/{NNN-slug}/{NNN-slug}-prd.md`. Each PRD goes through the multi-agent debate manifest at Gate 2 before tasks are written.
**Rationale:** The principle reviewers (Simplicity, Surgical, Think Before Coding, Goal-Driven) catch the drift the team was missing manually — speculative configurability, soft acceptance criteria, scope expansion between interview and PRD. The cost is a longer Phase A on each feature; the value is fewer mid-build scope renegotiations and tighter tests at the end. We're tracking the rate of post-Gate-2 amendments as the calibration metric — see task-history.
**Related:** `features/001-design-skill/` (first worked example), `CONTEXT.md` Recent context entry 2026-05-02, this project's adoption of `specflow:design`.

---

## Pin Playwright to 1.43.x

**Date:** 2026-04-29
**Context:** Playwright 1.44 was released with changes to the visual-diff matcher's default tolerance. Our existing baselines failed on the upgrade despite no real visual change in the app. Re-baselining would have hidden any genuine regressions during the upgrade window.
**Decision:** Pin Playwright to `~1.43.0` in `package.json` until we have time to re-baseline against 1.44 cleanly. Open a follow-up task tracked in Linear (ENG-1417).
**Rationale:** Re-baselining hides the signal we actually need — the diff between today's app and yesterday's app. Doing it under upgrade pressure would let real regressions slip in. The cost of staying on 1.43.x for a quarter is small; the risk of silent regressions is not. Goal-Driven Execution: keep the verify step binary, even at the cost of one minor version of currency.
**Related:** Linear ENG-1417, `package.json:devDependencies`, `CONTEXT.md` Recent context entry 2026-03-10 (incident note).

---
