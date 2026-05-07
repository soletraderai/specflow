# PRD interview — features/014-team-bridge-spec

## Original request
> `team-review` and Gate 5 both produce "multi-reviewer findings with severities" but their schemas, severity scales, and dedup rules are unaligned. When Sprint 4 invokes both (planner-skill is high-stakes), the outputs become unmergeable. Add a one-pager defining how a `team-review` finding maps to a Gate-5 manifest entry. (Risk B from the v2.4+ master plan.)

## Codebase context (pre-grilling)
- `plugins/specflow/skills/develop/SKILL.md` Phase E (lines 426-514) defines the Gate 5 manifest schema; severity scale `block | concern | info`; reviewer roster fixed.
- `~/.claude/plugins/cache/claude-code-workflows/agent-teams/1.0.2/agents/team-reviewer.md` § Output Format defines the `team-review` finding shape; severity scale `Critical | High | Medium | Low`; dimension labels.
- `develop/SKILL.md:87` and `:475-490` document the Codex-vs-Goal-Driven lens-overlap pattern and the Codex-only-finding auto-promotion contract — both relevant patterns for this bridge.
- The v2.4+ master plan's Risk B was added by the Plan agent during master-plan validation; the plan calls for a one-pager spec in Sprint 1.

## Round 1 — when does the bridge fire

**Question.** Does `team-review` always run at Gate 5, or only when invoked by escalation?

**Answer.** Only by escalation. Per the master plan, `team-review` fires when Gate 5 manifest reviewers disagree by ≥2 severity levels — tiebreaker, not default. Always-on would (a) duplicate review work and (b) bloat manifest output for low-disagreement gates where the standard 5-or-6-reviewer roster is sufficient.

## Round 2 — bidirectional mapping?

**Question.** Should the bridge be bidirectional (Gate-5 manifest → `team-review` markdown) too?

**Answer.** No. The use case is one-way: when `team-review` is invoked as tiebreaker, its findings need to land in the Gate-5 manifest. The reverse direction (Gate-5 findings expressed in `team-review` markdown) has no use case — `team-review`'s output is consumed by humans reading markdown, not by Gate 5. Bidirectional mapping would multiply the spec for no benefit.

## Round 3 — severity asymmetry on `High`

**Question.** Should `team-review`'s `High` always map to Gate-5 `block`, or asymmetrically (block when load-bearing, concern otherwise)?

**Answer.** Asymmetric. Gate 5's `block` semantics reserve the level for findings that, if accepted, force a plan revision — a stricter bar than `team-review`'s `High` which just means "important". A `team-review` `High` finding that cites only code-quality concerns (no R/AC/non-negotiable trace) doesn't meet the Gate-5 `block` bar; downgrading it to `concern` preserves the urgency signal without falsely-blocking the gate. A `High` finding that DOES cite an R or AC is load-bearing → `block`. The asymmetry preserves Gate 5's blocking discipline while honouring `team-review`'s severity.

## Topics not discussed

- Whether the bridge should support the four other `agent-teams:*` review skills (architect-review, security-auditor, performance-engineer, etc.). Scope creep — those skills have their own output formats; if/when they integrate with Gate 5, they get their own bridge spec.
- Whether the auto-promotion of `team-review`-only findings to `specflow:misc` should differ from Codex-only auto-promotion. Considered, rejected: same contract = less surface area to track.
- Whether the bridge should specify a JSON schema validator for the merge step. Out of scope for v2.4 — manual application of the mapping is sufficient at the volume Sprint 4 expects.
