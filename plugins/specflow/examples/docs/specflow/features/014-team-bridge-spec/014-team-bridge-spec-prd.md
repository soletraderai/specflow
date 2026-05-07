---
feature: 014-team-bridge-spec
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
interview: ./014-team-bridge-spec-interview.md
---

# 014 — `agent-teams:team-review` ↔ Gate-5 manifest bridge

## Vision

Define the schema bridge between the external `agent-teams:team-review` skill and `specflow:develop` Phase E (Gate 5). When a Gate 5 manifest's reviewers disagree by ≥2 severity levels, the orchestrator invokes `team-review` as an external tiebreaker; without an explicit field-and-severity mapping, the resulting outputs are unmergeable noise. The bridge specifies severity correspondence (Critical → block, Medium → concern, etc.), field mapping (markdown structure → Gate-5 JSON), and dedup rules (exact-location, cross-reviewer-overlap, independent-fire). It positions `team-review` as a tiebreaker, not a default — Gate 5's standard 5-or-6 reviewer manifest still runs first, and `team-review` only fires on the ≥2-severity-level disagreement signal. Sprint 4's planner skill is the first feature expected to actually invoke both surfaces; the bridge ships pre-emptively in Sprint 1 so Sprint 4 has a deterministic merge protocol from day one.

## Problem

`agent-teams:team-review` and `specflow:develop` Gate 5 each define their own structured review schemas. `team-review` uses a markdown finding format with severity scale `Critical | High | Medium | Low` and dimension labels (Security / Performance / Architecture / Testing / Accessibility). Gate 5 uses a JSON finding format with severity scale `block | concern | info` and reviewer roles (devils-advocate, simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, optionally Codex). Without an explicit mapping, two integration paths fail: (a) reviewers reading the merged output can't tell whether `team-review`'s `Medium` and Gate 5's `concern` are the same urgency level (they are, but only by inference); (b) dedup across the two surfaces has no shared key — a `team-review` Security finding citing `app/auth.ts:42` and a Gate-5 surgical-reviewer finding citing the same line could be true dupes (same defect spotted twice) or independent confirmations (two different lenses on the same line). Sprint 4's planner skill is high-stakes enough that the bridge MUST exist before that sprint runs; defining it pre-emptively in Sprint 1 means the bridge is doctrine, not retrofitted glue.

## Goals

- Doctrine doc at `plugins/specflow/templates/admin/team-review-bridge.md` defining: rationale, severity mapping table, field mapping table, dedup rules, when to invoke `team-review`, the worked-example reference.
- Severity mapping is asymmetric for `High`: `High → block` when load-bearing (cites R / AC / non-negotiable); `High → concern` otherwise. Reflects Gate 5's stricter blocking semantics.
- Field mapping covers every `team-review` field and every Gate-5 finding field; no field is dropped silently.
- Dedup rules cover three cases: exact-location, cross-reviewer-overlap, independent-fire.
- Auto-promotion of `team-review`-only findings to `specflow:misc` follows the Codex-only-finding contract (`develop/SKILL.md:475-490`) verbatim — same severity, same schema-gap fallback, same chat-line warning.
- The `team-review` tiebreaker fires only when Gate 5 reviewers disagree by ≥2 severity levels; default-off, opt-in by Round-3 escalation.

## Non-goals

- **Replacing Gate 5's standard manifest with `team-review`.** Gate 5's specflow-internal manifest is the canonical Gate 5; `team-review` is tiebreaker only.
- **Auto-invocation on every Gate 5.** No "always run `team-review`" config; the ≥2-severity-level disagreement is the trigger.
- **Bridging non-Gate-5 surfaces.** `team-review` against Gate 2, Gate 3, or Gate 4 is out of scope. Those gates have their own debate manifests; the bridge is Gate-5-specific.
- **Bidirectional mapping.** The bridge maps `team-review` → Gate-5 manifest. The reverse direction (Gate-5 manifest → `team-review` markdown) is not specified; it has no use case.
- **Tooling automation.** No script translates findings automatically. The mapping is human-applied during Round-3 sharpen if `team-review` was invoked.

## Users

- **`specflow:develop` orchestrators** at Round 3 of Gate 5 facing a ≥2-severity-level disagreement. They invoke `team-review`, receive a markdown finding set, apply the bridge mapping table to merge into the Gate-5 manifest, and proceed to closing decision.
- **Reviewers reading the merged manifest** post-Gate-5. They benefit from a manifest where every finding's severity is on the Gate-5 scale, every field has a Gate-5-compatible shape, and dedup decisions are explicit (the survivor's evidence cites "also raised by …").
- **Sprint-4 planner-skill author** designing the high-stakes feature that's expected to actually invoke both surfaces. They benefit from a doctrine that pre-dates the feature; the merge protocol is not invented under deadline.

## Requirements

- **R1.** Doctrine doc `plugins/specflow/templates/admin/team-review-bridge.md` exists and contains: rationale, severity mapping table, field mapping table, dedup rules, "when to invoke" guidance, worked-example reference.
  - Trace: templates/admin/team-review-bridge.md (new in v2.4.0).
  - Serves goal: doctrine is documented once.

- **R2.** Severity mapping table covers all four `team-review` levels (Critical / High / Medium / Low) with their Gate-5 equivalents and the `High` asymmetric rule.
  - Trace: templates/admin/team-review-bridge.md § Severity mapping.
  - Serves goal: severity correspondence is explicit and deterministic.

- **R3.** Field mapping table covers all six `team-review` markdown fields (`### [SEVERITY] Title`, Location, Dimension, Evidence, Impact, Recommended Fix) and how each lands in Gate-5 finding JSON (`id`, `severity`, `evidence`, `claim`, `proposed_change`).
  - Trace: templates/admin/team-review-bridge.md § Field mapping.
  - Serves goal: no field is dropped or invented.

- **R4.** Dedup rules cover exact-location duplicates, cross-reviewer overlap (preserved separately, mirrors Codex-vs-Goal-Driven lens-overlap), and independent fires.
  - Trace: templates/admin/team-review-bridge.md § Dedup rules.
  - Serves goal: reviewers know when to merge vs preserve.

- **R5.** Auto-promotion of `team-review`-only findings follows the Codex-only-finding contract from `develop/SKILL.md` § E.6 verbatim. Same severity criteria, same schema-gap fallback, same chat-line warning when `specflow:misc` schema lacks named fields.
  - Trace: templates/admin/team-review-bridge.md § Dedup rules + cross-reference to develop/SKILL.md:475-490.
  - Serves goal: auto-promotion path is consistent across external-reviewer surfaces.

- **R6.** "When to invoke" rule: `team-review` fires only when Gate 5 reviewers disagree by ≥2 severity levels at Round 1. Default-off, opt-in by escalation.
  - Trace: templates/admin/team-review-bridge.md § When to invoke.
  - Serves goal: tiebreaker, not default.

- **R7.** Worked example folder `examples/docs/specflow/features/014-team-bridge-spec/` demonstrates the bridge with a sample `team-review` finding translated to a Gate-5 manifest entry.
  - Trace: this PRD's See also.
  - Serves goal: dogfood discipline.

## Acceptance criteria

- **AC-1.** `plugins/specflow/templates/admin/team-review-bridge.md` exists with all sections from R1. Verifies R1.
- **AC-2.** Severity mapping table contains all four `team-review` levels with explicit Gate-5 mappings and the `High` asymmetric rule. Verifies R2.
- **AC-3.** Field mapping table maps every `team-review` markdown field to a Gate-5 JSON field. Verifies R3.
- **AC-4.** Dedup rules section explicitly covers exact-location, cross-reviewer-overlap, and independent-fire. Verifies R4.
- **AC-5.** Auto-promotion section cites `develop/SKILL.md:475-490` verbatim and follows the same contract. Verifies R5.
- **AC-6.** "When to invoke" section names the ≥2-severity-level disagreement trigger explicitly. Verifies R6.
- **AC-7.** Worked example folder exists with PRD, interview, tasks, Gate 2 manifest closed `passed`, and a sample `team-review`-to-Gate-5 translation in the tasks file. Verifies R7 + dogfood discipline.
- **AC-8.** v2.3 → v2.4 MIGRATIONS entry references the bridge doctrine doc. Verifies sprint-close consolidation.

## Open questions

None — the mapping is mechanical and bounded.

## See also

- `plugins/specflow/templates/admin/team-review-bridge.md` — the doctrine
- `plugins/specflow/skills/develop/SKILL.md:475-490` — the Codex-only-finding contract this bridge inherits
- `~/.claude/plugins/cache/claude-code-workflows/agent-teams/1.0.2/agents/team-reviewer.md` — `agent-teams:team-review` source schema
- `v2/docs/PRD.md` § Resolved decisions — the resolution citation (mitigates Risk B from master plan)
