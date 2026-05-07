# `agent-teams:team-review` ↔ Gate-5 manifest bridge

When `specflow:develop` Phase E (Gate 5) reviewers reach a ≥2-severity-level disagreement, the orchestrator can invoke `agent-teams:team-review` as an external tiebreaker per the v2.4+ master plan. Both surfaces produce "multi-reviewer findings with severities", but their schemas, severity scales, and dedup rules differ. This bridge defines the mapping so a `team-review` finding can be merged into the Gate-5 manifest without losing fidelity or double-counting evidence.

Introduced in v2.4.0 (`014-team-bridge-spec`). Mitigates Risk B from the v2.4+ master plan ("agent-teams semantics drift from gate manifests").

## Why this exists

`agent-teams:team-review` and `specflow:develop` Gate 5 each define their own structured review schemas. The plan's Sprint 4 (high-stakes planner skill) is the first sprint that actually invokes both — and without an explicit bridge, the outputs become incompatible: severity scales don't line up, finding fields differ, and dedup rules across the two schemas have no shared key. Reviewers reading both manifests can't tell whether `team-review`'s "Medium" is the same severity as Gate 5's "concern", or whether two findings citing the same file:line from the two surfaces are dupes or independent confirmations.

## Severity mapping

| `agent-teams:team-review` | Gate-5 manifest | Notes |
|---|---|---|
| Critical | block | Always blocks Gate 5 closure. Same-severity outcome on both surfaces. |
| High | block (when load-bearing) OR concern | Load-bearing = the finding cites a PRD requirement, AC, or non-negotiable rule. Otherwise downgrades to concern. |
| Medium | concern | Surfaces in the manifest's "Concerns" section; reviewers respond in Round 2; does not block. |
| Low | info | Surfaces in the manifest's "Info" section; no response required; recorded for traceability. |

The asymmetric mapping for `High` reflects Gate 5's stricter blocking semantics: Gate 5 reserves `block` for findings that, if accepted, force a plan revision. A `team-review` `High` finding that cites only code-quality concerns (no R/AC/rule trace) doesn't meet that bar, so it downgrades. A `High` finding that cites an R or AC is load-bearing → `block`.

## Field mapping

| `agent-teams:team-review` markdown | Gate-5 manifest JSON | Transform |
|---|---|---|
| `### [SEVERITY] Finding Title` | `findings[].id` (`tr-r1-fN`), `severity` per table above | Generate id from sequence; map severity per table |
| `**Location**: file:line` | `findings[].evidence` (prefix) | Prepend `Location: file:line — ` to the evidence string |
| `**Dimension**: …` | `findings[].evidence` (suffix in brackets) | Append ` [dimension: {Security/Performance/etc.}]` |
| `**Evidence**: …` | `findings[].evidence` (body) | Quote markdown-paragraph verbatim |
| `**Impact**: …` | `findings[].claim` | Reframe Impact as the claim — "If unaddressed, {impact}." |
| `**Recommended Fix**: …` | `findings[].proposed_change` | Verbatim |

The `reviewer` field on the Gate-5 manifest's wrapper JSON becomes `team-review/{dimension}` (e.g. `team-review/security`, `team-review/performance`). Round/gate/feature fields follow the standard Gate-5 wrapper schema.

## Dedup rules

When a `team-review` finding lands alongside Gate-5's same-round findings, dedup applies before the Round-2 response phase:

1. **Exact-location match.** Two findings citing the same file:line + the same dimension are duplicates. Preserve the higher-severity finding; merge evidence/claim text by appending `(also raised by {other reviewer})` to the survivor.
2. **Cross-reviewer overlap.** A `team-review/{dimension}` finding and a same-Gate-5-reviewer finding citing the same R or AC trace are overlap, not dupes. Preserve both; cite each from the other in the Round-3 sharpen pass. Mirrors the Codex-vs-Goal-Driven lens-overlap pattern (`develop/SKILL.md:87`).
3. **Independent fires.** A `team-review` finding citing nothing the Gate-5 reviewers cited is a fresh finding. Add it to the manifest with severity per the mapping table; treat in Round 2 as any other Round-1 finding.

Auto-promotion of `team-review`-only findings to `specflow:misc` follows the same Codex-only-finding contract per `develop/SKILL.md:475-490` (E.6) — same severity, same schema-gap fallback, same chat-line warning when `specflow:misc` schema lacks the named fields.

## When to invoke `team-review`

Per the v2.4+ master plan: invoke `agent-teams:team-review` ONLY when Gate 5's manifest reviewers disagree by ≥2 severity levels (e.g. one reviewer says `block`, another says `info`). Invocation is tiebreaker, not default. The bridge mapping above governs how the tiebreaker findings land in the Gate-5 manifest.

For all other cases, Gate 5 runs its standard 5-or-6-reviewer manifest without `team-review`.

## Worked example

See `examples/docs/specflow/features/014-team-bridge-spec/`.

## Resolution citation

`v2/docs/PRD.md` § Resolved decisions — 014-team-bridge-spec v2.4.0.
