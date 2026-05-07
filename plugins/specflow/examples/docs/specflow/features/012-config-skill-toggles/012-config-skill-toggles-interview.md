# PRD interview — features/012-config-skill-toggles

## Original request
> Config file should be able to enable/disable certain parts of the plugin. Things like disabling the development function. (User chat feedback, 2026-05-06.)

## Codebase context (pre-grilling)
- `plugins/specflow/skills/setup/SKILL.md` Phase 8.2 currently writes `config.json` with four top-level keys (`specflowVersion`, `linear`, `confidentialPaths`, `teams`); no `skills` block.
- Audit confirmed no current pattern for per-skill enable/disable; the chain assumption is hard-coded across reviewers and `SKILLS.md`.
- `plugins/specflow/CORE_PRINCIPLES.md` and `templates/orchestrator-pattern.md` describe how skills should structure their pre-flight; neither mentions a config-driven toggle.
- v2.3 projects lack the `skills` field; backward-compat is required.

## Round 1 — toggle granularity

**Question.** Should the toggle be project-level (current proposal) or also per-feature (a feature can opt out of `develop` even when the project default enables it)?

**Answer.** Project-level only. Per-feature toggles multiply the surface for marginal benefit; the use cases the user described (org-wide tooling preference, regulated environments) are project-uniform. If a one-off feature needs implementation outside the chain, the user invokes another tool directly — that's a workflow choice, not a toggle.

## Round 2 — chain-breakage behaviour

**Question.** When a chain skill (e.g. `develop`) is disabled, should the chain auto-reroute (skip directly from `task` to `test`), or should downstream skills fail their `requires:` check?

**Answer.** Fail the `requires:` check. Auto-rerouting hides the consequence of the user's choice; a project that disables `develop` should see the chain stop at `task` and pick up at `test` only if the user manually invokes it (with the produced-by-some-other-tool implementation already on disk). The chain breaking visibly is the right user signal.

## Round 3 — refusal message uniformity

**Question.** Should each skill author choose its refusal phrasing, or should the message be canonical?

**Answer.** Canonical, verbatim. Cross-skill consistency reduces the cognitive cost of seeing a refusal — users learn the format once, then know how to handle it everywhere. Skill-specific phrasing risks drift over time (each new skill's author makes a slightly different copy choice); locking the format in the doctrine doc avoids that.

## Round 4 — rollout strategy

**Question.** Should v2.4 modify every skill's Phase A to enforce the toggle, or roll out incrementally?

**Answer.** Incrementally — demonstrate in `specflow:develop` for v2.4, document the contract for all, and let other skills adopt as they're next touched. Touching every skill in one sprint to add a single mechanical sub-step would (a) trigger needless gate manifests for unrelated changes and (b) surface no real value beyond what the doctrine already provides. The contract is the load-bearing artefact; per-skill enforcement is a low-friction follow-up.

## Topics not discussed

- Whether the toggle should affect `/grill`, `/simplify`, `/panic`, `/format`, `/release-version-check`, `/feedback-loop-audit`, `/confidence-check` (the bare-name skills). Treat them the same as prefixed skills — bare name is the toggle key. Documented in the resolver doctrine but not in the seeded `skills` block (those bare-name skills are covered when needed).
- Whether disabling a skill should propagate to Linear or external systems. Not applicable — toggle is local; external systems are unaware.
- Whether the toggle should support a "warn-only" mode (allow but warn). Two-state design is simpler; if the user wants to be warned, they enable.
