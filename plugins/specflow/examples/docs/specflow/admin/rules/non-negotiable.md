# Non-negotiable rules

Hard rules. Always-on. Loaded into every skill's system prompt. Project-tuned at setup; additive over time.

When a rule is violated, the AI:
1. Does not fix inline (Surgical Changes).
2. Auto-creates a `misc-task` with the rule reference, the location, and the *why*.
3. Continues with the original work item.

If the violation is *inside* the current work item's scope, it must be addressed before the change set ships — surfaced as a blocker, not a misc-task.

---

## Starter set

The starter set proposed at setup. Users review, accept/edit, commit. Each rule has a frontmatter block plus a body that names the *why* and *exceptions*.

---

```yaml
---
id: NO_HARDCODED_VALUES
tier: non-negotiable
paths: ["src/**/*.ts", "src/**/*.tsx", "src/**/*.js", "src/**/*.jsx"]
---
```

**Rule:** Hardcoded values (strings, numbers, URLs, magic constants) should be moved to config or environment unless absolutely necessary.

**Why:** Hardcoded values resist change, leak across environments, and silently couple components. Dynamic by default.

**On violation:** Auto-create a misc-task with the file:line reference, the value, and why it should be dynamic.

**Exceptions:** Test fixtures; one-off scripts; literal protocol values (HTTP status codes, well-known port numbers).

---

```yaml
---
id: NO_COMMENTS_WITHOUT_WHY
tier: non-negotiable
paths: ["**/*.ts", "**/*.tsx", "**/*.js", "**/*.jsx", "**/*.py", "**/*.go", "**/*.rb"]
---
```

**Rule:** Comments are only acceptable when they explain a non-obvious *why*: a hidden constraint, a subtle invariant, a workaround for a specific bug, behavior that would surprise a reader.

**Why:** Comments that describe *what* the code does duplicate well-named identifiers and rot as the code evolves. Comments that explain *why* survive refactors and earn their cost.

**On violation:** Auto-create a misc-task referencing the comment block and explaining why it doesn't earn its place.

**Exceptions:** JSDoc/TSDoc on public API surfaces where the *why* is also documented (parameter purpose, edge cases, contract).

---

```yaml
---
id: NEVER_BYPASS_AUTH
tier: non-negotiable
paths: ["**/auth/**", "**/middleware/**", "**/security/**"]
---
```

**Rule:** Authentication, authorization, and session checks are never bypassed, weakened, or replaced with TODO scaffolding "for now."

**Why:** "We'll add auth back later" is the most common path to security incidents. Auth gaps are not a sprint backlog item — they're a Red-lane (human-led) concern even at prototype stage.

**On violation:** Surface as a Red-lane blocker. Do NOT proceed with the change set; do NOT log as a misc-task. Bring the human in.

**Exceptions:** None. Test environments use real auth flows with test credentials, not bypassed checks.

---

```yaml
---
id: PROTECTED_PATHS_REQUIRE_RED_LANE
tier: non-negotiable
paths: []  # populated at setup from config.json.confidentialPaths
---
```

**Rule:** Files matching the project's `confidentialPaths` glob are Red-lane: human-led with AI assisting on bounded subtasks only. The lane is rule-based (path globs in `admin/config.json`), never AI-rated.

**Why:** No work item should escape Red because the AI feels confident. Confidentiality is a property of the surface, not of the change.

**On violation:** Reassign the work item to the Red lane. Do NOT proceed in green/yellow lane workflow.

**Exceptions:** None. The path glob is the contract.

---

```yaml
---
id: TESTS_REQUIRED_FOR_VERIFIABLE_SKILLS
tier: non-negotiable
paths: ["plugins/**/skills/**", "v2/specflow/skills/**"]
---
```

**Rule:** Skills that produce verifiable output MUST declare a binary `eval:` field in their SKILL.md frontmatter and ship a verification step that exercises it.

**Why:** Strong success criteria let the LLM iterate independently; weak ones force constant clarification. The eval field is the contract that makes Goal-Driven Execution possible.

**On violation:** Surface as a blocker before the skill is merged.

**Exceptions:** Skills that are purely interactive (e.g. `/grill` — its eval is the interview file having signed-off rounds, which is itself a binary check).
