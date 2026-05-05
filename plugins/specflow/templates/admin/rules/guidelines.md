# Guidelines

Soft rules. Project taste. Adaptive — the system can suggest additions based on observed patterns. Loaded into skills on-demand when relevant (e.g. during `specflow:develop` for files matching a guideline's `paths:` glob).

Guidelines are weaker than non-negotiable rules:
- A non-negotiable violation triggers an auto-`misc-task` (or a Red-lane block).
- A guideline violation is surfaced for human judgement — fix inline if obvious, log a misc-task if non-trivial, ignore if the guideline doesn't fit this case.

The Phase 3 self-evolution loop scans `task-history.json` and `misc-task` entries for repeated violations of the same shape. Three observations of a pattern → suggest a new guideline. Three guideline-flagged violations → suggest promotion to non-negotiable. Human signs off at each promotion.

---

## How to add a guideline

Each entry follows the same frontmatter shape as non-negotiable rules:

```yaml
---
id: PREFER_COMPOSITION_OVER_INHERITANCE
tier: guideline
paths: ["src/**/*.ts", "src/**/*.tsx"]
---
```

**Rule:** Prefer composition over inheritance for component reuse in this codebase.

**Why:** [project-specific reason — usually cites a past incident or architectural decision]

**On violation:** [what the AI should do — surface, log, suggest]

**Exceptions:** [where the rule doesn't apply]

---

## Initial state

This file ships empty for new projects. Guidelines accumulate as the project matures and the team's taste settles. The setup skill does not seed guidelines — they emerge from real work, not from a starter pack.
