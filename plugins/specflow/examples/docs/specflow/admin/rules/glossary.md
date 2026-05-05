# Rules glossary

Every rule in this project's registry, listed with a one-line description and the *why* it exists. Self-documenting — reviewable in PRs as the rule set evolves.

Format: `RULE_ID` (tier) — one-line description. **Why:** [reason]

---

## Non-negotiable

- `NO_HARDCODED_VALUES` (non-negotiable) — Hardcoded values move to config or environment unless absolutely necessary. **Why:** Dynamic by default; hardcoded values resist change and silently couple components.
- `NO_COMMENTS_WITHOUT_WHY` (non-negotiable) — Comments only when they explain a non-obvious *why*. **Why:** What-comments duplicate well-named identifiers; why-comments earn their cost.
- `NEVER_BYPASS_AUTH` (non-negotiable) — Auth, authz, and session checks are never bypassed for convenience. **Why:** "We'll add auth back later" is the most common path to security incidents.
- `PROTECTED_PATHS_REQUIRE_RED_LANE` (non-negotiable) — Files in `confidentialPaths` are human-led work. **Why:** No work item should escape Red because the AI feels confident.
- `TESTS_REQUIRED_FOR_VERIFIABLE_SKILLS` (non-negotiable) — Skills with verifiable output declare a binary `eval:` field. **Why:** Strong success criteria let the LLM iterate independently.

## Guidelines

- `PREFER_LOCAL_TESTS` (guideline) — Tests sit alongside source in `__tests__/` directories rather than under a top-level `tests/` tree. **Why:** Test files are co-edited with their source 90%+ of the time; co-location halves the cost of finding the test for a given module. See decision-log entry 2026-02-14.
- `PREFER_COMPOSITION_OVER_INHERITANCE` (guideline) — Services in `src/services/` compose small functions with injected dependencies; no class hierarchies. **Why:** A `BaseService` hierarchy grew to 340 lines and produced two production bugs before removal. See decision-log entry 2026-01-08.
- `AUDIT_TRAIL_VIA_HELPER` (guideline) — State-changing actions write to the audit trail via `src/audit/log.ts`'s helper, never inline. **Why:** The helper records actor, timestamp, and diff in a queryable schema; ad-hoc audit drifts from the schema and breaks compliance reviews.
- `ORDER_DETAIL_TOP_DOWN_STATE` (guideline) — State on the order-detail page flows top-down from `useOrderDetail`; sibling components do not cross-import each other's state. **Why:** Cross-component state divergence shipped a status-badge bug to production; hoisting state to the page hook eliminates the failure mode by construction.

---

## How rules promote

The Phase 3 self-evolution loop tracks rule-shaped patterns in `task-history.json` and `misc-task/`:

1. **Observation** — three independent occurrences of the same pattern surface a suggestion in the monthly `/insights` report.
2. **Promotion to guideline** — human reviews, accepts, the entry lands in `guidelines.md` with frontmatter + body + glossary line.
3. **Promotion to non-negotiable** — three guideline-flagged violations of the same shape surface a further suggestion. Human reviews, accepts, the entry promotes.

Versioning the rule itself is an open question (see PRD Appendix O7 — `id: NO_HARDCODED_VALUES@2` is a candidate). Until that's decided, promotions are tracked through `git log` on this glossary.
