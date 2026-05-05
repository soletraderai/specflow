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

## Project guidelines

The following guidelines have accumulated as the team's taste settled. Each one cites the decision-log entry or task-history pattern that earned it a slot.

---

```yaml
---
id: PREFER_LOCAL_TESTS
tier: guideline
paths: ["src/**/*.ts", "src/**/*.tsx", "packages/**/*.ts", "packages/**/*.tsx"]
---
```

**Rule:** Tests sit alongside source in `__tests__/` directories — never in a top-level `tests/` tree, except for cross-module integration suites that don't belong to any single module.

**Why:** Test files are co-edited with their source 90%+ of the time. Co-location halves the cognitive overhead of finding the test for a given module. See decision-log entry 2026-02-14.

**On violation:** Surface as a guideline note in the change set. If the test belongs to a single module but lives under top-level `tests/`, suggest moving it. Log a misc-task if the move is non-trivial (e.g. shared fixtures need extraction).

**Exceptions:** Cross-module integration suites that genuinely don't belong to any single module — these stay under top-level `tests/integration/`.

---

```yaml
---
id: PREFER_COMPOSITION_OVER_INHERITANCE
tier: guideline
paths: ["src/services/**/*.ts"]
---
```

**Rule:** Services in `src/services/` use composition (small functions composed with their dependencies injected at the call site) rather than class hierarchies.

**Why:** A `BaseService` class introduced in 2025 Q4 grew to 340 lines and four levels of inheritance before being removed. Two production bugs were traced to its behaviour being misunderstood at the call site. See decision-log entry 2026-01-08.

**On violation:** Surface as a guideline note. Reference the orders-service implementation as the canonical pattern. If the new service genuinely needs shared behaviour, surface as a misc-task to discuss whether a composable helper (not a base class) is warranted.

**Exceptions:** None currently. The base-class shape is explicitly out of fashion in this codebase.

---

```yaml
---
id: AUDIT_TRAIL_VIA_HELPER
tier: guideline
paths: ["src/**/*.ts", "app/**/*.ts", "app/**/*.tsx"]
---
```

**Rule:** State-changing actions write to the audit trail via `src/audit/log.ts`'s helper. Inline `console.log`, ad-hoc audit, or skipping the audit entirely is rejected.

**Why:** The audit trail is the single source of truth for compliance reviews and customer-support reproductions. Inline `console.log` doesn't survive log rotation; ad-hoc audit drifts from the helper's schema and breaks queryability. The helper records actor, timestamp, and diff — features the ad-hoc paths consistently forget.

**On violation:** Surface as a blocker if the change is in a code path that mutates customer or order state. Otherwise, surface as a guideline note and propose the helper call inline.

**Exceptions:** Read-only flows; test fixtures; one-off scripts that aren't part of the production code path.

---

```yaml
---
id: ORDER_DETAIL_TOP_DOWN_STATE
tier: guideline
paths: ["app/orders/[orderId]/**/*.tsx"]
---
```

**Rule:** State on the order-detail page flows top-down from the `useOrderDetail` hook in `app/orders/[orderId]/page.tsx`. Sibling components do NOT cross-import each other's state — if two components need shared state, lift it to the page hook.

**Why:** A bug where the status badge and the action bar held divergent copies of the order's status field shipped to production. Root cause: components were each fetching the order independently and mutating in-place. Hoisting state to the page hook eliminates the divergence by construction.

**On violation:** Surface as a blocker. Cross-component state cross-imports on this page are how the previous bug shipped; the rule exists to prevent recurrence. Propose lifting the shared state to `useOrderDetail`.

**Exceptions:** Local UI state (open/closed disclosure, hover state, transient input value) stays in the component.

---
