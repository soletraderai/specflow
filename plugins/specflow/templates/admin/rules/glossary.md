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

(Empty for new projects. Populated as project taste settles.)

---

## How rules promote

The Phase 3 self-evolution loop tracks rule-shaped patterns in `task-history.json` and `misc-task/`:

1. **Observation** — three independent occurrences of the same pattern surface a suggestion in the monthly `/insights` report.
2. **Promotion to guideline** — human reviews, accepts, the entry lands in `guidelines.md` with frontmatter + body + glossary line.
3. **Promotion to non-negotiable** — three guideline-flagged violations of the same shape surface a further suggestion. Human reviews, accepts, the entry promotes.

Versioning the rule itself is an open question (see PRD Appendix O7 — `id: NO_HARDCODED_VALUES@2` is a candidate). Until that's decided, promotions are tracked through `git log` on this glossary.
