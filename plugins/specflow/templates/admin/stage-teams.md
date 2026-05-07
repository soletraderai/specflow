# Stage teams — Plan / Build / Test / Iterate / Validate

Promotes the canonical pipeline shape — **Plan → Build → Test → Iterate → Validate** — to first-class doctrine. Each stage has a designated agent team configurable per project via `config.json.teams.{stage}`. Sprint planner (020) emits per-task team assignments; develop / test skills consume them at their respective gates.

Introduced in v2.7.0 (`026-agent-teams-per-stage`). Cross-cuts with 027-reviewer-context-isolation (every team member runs in fresh context).

## The five stages

| Stage | Default roster | Where it fires |
|---|---|---|
| **Plan** | `sprint-planner` (020) + `goal-driven-reviewer` + `devils-advocate` | `specflow:sprint` Phase C; `specflow:develop` Phase B.1.1 (per-task plan emission) |
| **Build** | `implementer-{lane}` + `simplicity-reviewer` + `surgical-reviewer` | `specflow:develop` Phase D (lane execution) |
| **Test** | `code-quality-reviewer` + `result-reviewer` + `simplicity-reviewer` + `security-reviewer` | `specflow:test` Phase B + Phase C |
| **Iterate** | `verifier` + `edge-case-reviewer` (per 028) | `specflow:develop` Phase F.1 (Verifier); cross-task review (per 022) |
| **Validate** | `orchestrator` + `codex-reviewer` (when avail) + `agent-teams:team-review` (when ≥2-severity-level disagreement, per `templates/admin/team-review-bridge.md`) | Gate 5 closing |

## Default roster details

### Plan team

The Plan team's role is to produce a plan artefact that traces to a PRD requirement. The sprint-planner contributes the multi-task plan; the goal-driven-reviewer verifies forward + reverse coverage; the devils-advocate flags cross-artefact ambiguity.

### Build team

The Build team's role is to write the code that implements the per-task plan. The implementer (single-specialist or team-spawn output, lane-dependent) writes; the simplicity-reviewer flags premature abstraction; the surgical-reviewer flags scope drift.

### Test team

The Test team's role is to verify the implementation against the AC list (binary) AND surface brand / consistency / coverage-gap concerns (advisory, per 023). The four lenses split: code-quality-reviewer (compiles cleanly; lints; types check); result-reviewer (the AC pass/fail signal); simplicity-reviewer (the test plan itself isn't over-decomposed); security-reviewer (no obvious vulnerabilities introduced).

### Iterate team

The Iterate team's role is to confirm the implementation against the per-task plan AND surface edge cases the writer missed. The verifier checks each acceptance clause against the diff's observable behaviour; the edge-case-reviewer (per 028) applies the five-question lens. Cross-task review (per 022) consults the iterate team's findings as one input among several.

### Validate team

The Validate team's role is to close the gate. The orchestrator collates findings and emits the closing decision; the codex-reviewer (when available) provides cross-provider correctness coverage; `agent-teams:team-review` fires only when there's ≥2-severity-level disagreement among same-provider reviewers (per `templates/admin/team-review-bridge.md`).

## Config schema

`config.json.teams.{stage}` — per-stage roster override. Schema:

```json
{
  "teams": {
    "plan": [
      { "agent": "sprint-planner", "lane": "any" },
      { "agent": "goal-driven-reviewer", "lane": "any" },
      { "agent": "devils-advocate", "lane": "any" }
    ],
    "build": [
      { "agent": "implementer", "lane": "any", "lane_dispatch": true },
      { "agent": "simplicity-reviewer", "lane": "any" },
      { "agent": "surgical-reviewer", "lane": "any" }
    ],
    "test": [
      { "agent": "code-quality-reviewer", "lane": "any" },
      { "agent": "result-reviewer", "lane": "any" },
      { "agent": "simplicity-reviewer", "lane": "any" },
      { "agent": "security-reviewer", "lane": "any" }
    ],
    "iterate": [
      { "agent": "verifier", "lane": "any" },
      { "agent": "edge-case-reviewer", "lane": "any" }
    ],
    "validate": [
      { "agent": "orchestrator", "lane": "any" },
      { "agent": "codex-reviewer", "lane": "any", "when_available": true },
      { "agent": "agent-teams:team-review", "lane": "any", "when": "2_severity_level_disagreement" }
    ]
  }
}
```

`lane`: `"any" | "green" | "yellow" | "red"` — the agent fires only on tasks of the matching lane. `"any"` is the default. The `lane_dispatch: true` flag on `implementer` means the agent name is resolved at dispatch time per the lane (`implementer-green | implementer-yellow | implementer-red`).

`when_available`: when true, the agent is included only if the relevant CLI / plugin is detected at runtime.

`when`: an event predicate. Currently supported: `"2_severity_level_disagreement"` (fires when same-provider reviewers disagree by ≥2 severity levels).

## Override path

Projects override the defaults by writing `config.json.teams.{stage}` with the desired roster. Setup seeds an empty `teams: {}` object; first invocation of `specflow:sprint` materialises the defaults from this doctrine doc.

A user-written override is preserved verbatim; setup never overwrites a present value. The doctrine defaults apply only when the field is absent.

## How sprint planner (020) consumes this

`specflow:sprint` Phase C.1 step 3 reads `config.json.teams.{stage}` and resolves a team-assignment block per task in the sprint plan. The resolved block lists the agents per stage, with `lane` constraints honoured against the task's predicted lane.

## How develop / test consume this

`specflow:develop` Phase B.4 (agent-teams plugin consultation) reads the resolved team-assignment block per task instead of computing the team from scratch. Phase D's lane execution dispatches the Build team. Phase E's Gate 5 dispatches the Validate team.

`specflow:test` Phase A reads the resolved Test team and dispatches accordingly.

## Cross-references

- **020 — sprint-skill** — emits the per-task team assignments.
- **022 — cross-task-review** — Iterate team's edge-case-reviewer findings inform the better-arrangement lens.
- **023 — test-brand-consistency** — Test team's brand-consistency lens is part of the Test stage.
- **027 — reviewer-context-isolation** — every team member runs in fresh context per the cross-cutting contract.
- **028 — edge-case-reviewer** — Iterate team's principle reviewer.
- **`templates/admin/team-review-bridge.md`** — when-to-invoke trigger for `agent-teams:team-review` at the Validate stage.
