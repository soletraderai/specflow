# Debate manifest — Gate 2: PRD vs interview review

**Feature:** 005-scope-change-skill
**Artefact under review:** `005-scope-change-skill-prd.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06

This is the dogfood Gate 2 run for the Phase 3 `specflow:scope-change` PRD. The PRD was synthesised first-pass from the interview with the surgical-r1-f1 schema-dependency revision pre-applied during synthesis (the AI Phase C surfaced the cross-skill `specflow:decision` schema gap during Round 5's writer pass and embedded the dependency clause directly into R10.1 + AC-10); the remaining six findings landed in this Gate 2 run as the authentic adversarial pass. One push-back was defended in Round 2 and accepted in Round 3 (the orphan-AC framing for token bounds — R12's binding-by-reference to `templates/orchestrator-pattern.md` IS the R-level expression).

---

## Round 1 — Findings

| Reviewer | Findings (severity) |
|---|---|
| simplicity-reviewer | 1 (concern) |
| surgical-reviewer | 1 (concern) |
| think-before-coding-reviewer | 1 (concern) |
| goal-driven-reviewer | 2 (block, concern) |
| devils-advocate | 1 (concern) |
| **Total** | **6 findings (1 block, 5 concern)** |

Detail:
- **simplicity-r1-f1** — *concern* — R8 + AC-8 introduce a `--include-backlog` flag without a documented consumer ask (speculative configurability). User confirmed the AI-recommended option in interview Round 4 but did not surface a use case where the flag would be flipped. (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — R10.1's scope-change-specific fields embedded inside the `references` block + the `SC-{NNN}` id-prefix imply a `specflow:decision` payload-schema extension; the cross-skill dependency must be loud, not buried. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *concern* — R4's `~~text~~` strikethrough markup is treated as a parseable contract by R5 and R10.1 without being named as such; line-number references in `superseded_resolved_lines` drift if the strikethrough adds characters mid-line. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **goal-r1-f1** — *block* — AC-8's "in-flight" boundary is non-binary at the across-source edges: draft PRs (`gh --state open` includes them) and Linear custom intermediate statuses (Blocked, On Hold) are silently undefined. A fresh agent cannot apply the rule unambiguously. (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — AC-13 verifies token-budget bounds (25K parent-context, 2K per-sub-skill) that no R-level requirement establishes; orphan-AC reverse-trace gap. (Same file.)
- **da-r1-f1** — *concern* — R2 trigger (a) (Verifier rejection option 3) fires from inside a paused `specflow:develop` run; the lifecycle-handoff contract was unspecified, and R8's impact list would silently surface the rejected task as its own trigger (circular reference). (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **accept** (dropped `--include-backlog` flag; v1 hard-codes Backlog exclusion; deferred to v2)
- surgical-r1-f1 → **accept** (applied option (b): explicit cross-skill schema dependency call-out in R10.1 and AC-10)
- tbc-r1-f1 → **accept** (named strikethrough markup as the canonical machine-readable signal in R4; changed `superseded_resolved_lines` shape to round+anchor-text; tightened AC-4)
- goal-r1-f1 → **accept** (added boundary clauses to AC-8: draft PRs INCLUDED with `(draft)` annotation, Linear custom statuses EXCLUDED with chat-line note)
- goal-r1-f2 → **push_back** (defended R12's binding-by-reference to `templates/orchestrator-pattern.md` as the R-level expression of the token-budget bounds; AC-13's `Verifies` line clarified to make the binding-by-reference explicit; no new R13 added)
- da-r1-f1 → **accept** (extended R2 with state-handoff clause introducing `aborted_for_scope_change` status; extended R8 with self-reference exclusion; added AC-2 and AC-8 sub-clauses; surfaced follow-on cross-skill schema dependency on `specflow:develop` accepting the new status value)

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- simplicity-r1-f1 → **accept** (flag drop is the right call; net speculative-knob count: 0)
- surgical-r1-f1 → **accept** (cross-skill dependency loud, not buried; matches 002-develop-skill precedent)
- tbc-r1-f1 → **accept** (parser-contract assumption now stated; tradeoff articulated; round+anchor-text shape is drift-resistant)
- goal-r1-f1 → **accept** (boundary clauses give fresh agents a binary check; lens question resolves)
- goal-r1-f2 → **accept** (defence holds — binding-by-reference to a cited template counts as R-level expression; orphan-AC heuristic mis-fires here because R12's referenced template is canonical)
- da-r1-f1 → **accept** (lifecycle-handoff named; circular-reference closed; new status value follows the explicit-dependency pattern)

No sharpening occurred — every reviewer accepted the AI's Round 2 disposition (revisions applied or push-back-defended). No `ai-revision.md` needed in Round 3.

---

## Closing decision

**Gate 2 status: passed-with-revisions**

One `block` finding landed in Round 1 — accepted, resolved with PRD revision. Five `concern` findings landed — four accepted with revisions, one defended in Round 2 and accepted in Round 3 (the goal-r1-f2 split where R12's binding-by-reference to `templates/orchestrator-pattern.md` was deemed sufficient R-level expression of the token-budget bounds without a new R13).

### PRD revisions applied

The PRD was edited between Round 1 and Round 3 to incorporate every accepted finding:

1. **R8 + AC-8 (concern simplicity-r1-f1).** Dropped the `--include-backlog` flag. R8 now reads: 'Linear (when MCP available) — every issue where `feature == {NNN-slug}` AND `status` ∈ {`In Progress`, `In Review`} AND the issue's Anchor R-ID is in the changed-R-ID set. Backlog tickets are excluded; v1 ships with this hard-coded.' AC-8's conditional second clause about the flag was removed. Net speculative-knob count: 0.
2. **R10.1 + AC-10 (concern surgical-r1-f1).** Schema-dependency clause embedded directly in R10.1 + AC-10 naming the `specflow:decision` payload-schema affordances (arbitrary keyed-blocks within `references`; `id_prefix: "SC"` parameter). The cross-skill dependency is loud; if `specflow:decision`'s current schema doesn't accept these, a `specflow:decision` enhancement PRD ships first.
3. **R4 + R10.1 + AC-4 (concern tbc-r1-f1).** R4 extended with the parser-contract clause naming `~~text~~` strikethrough paired with `(superseded by Scope change {YYYY-MM-DD})` as the canonical machine-readable signal. R10.1's `superseded_resolved_lines` reference shape changed from line-numbers to round+anchor-text (drift-resistant). AC-4 extended to verify markup-as-contract: orphan strikethrough runs without matching supersession notes are a failed step (i).
4. **AC-8 (block goal-r1-f1).** Two boundary clauses added: (a) Open PRs INCLUDE draft PRs with `(draft)` annotation; (b) Linear EXCLUDES custom intermediate statuses (Blocked, On Hold, etc.) with a chat-line note surfacing the skipped count. R8 updated to reference the explicit boundary. The AC now resolves the binary lens question for fresh agents.
5. **AC-13 clarification (concern goal-r1-f2, defended).** No new R13 added; AC-13's `Verifies` line was clarified to make the binding-by-reference to `templates/orchestrator-pattern.md` explicit ('the 25K/2K numbers are scope-change-specific calibrations of those targets, derived from the eight-step flow vs `specflow:develop`'s longer flow'). Push-back held on the basis that R12's binding-by-reference IS the R-level expression of the bounds.
6. **R2 + R8 + AC-2 + AC-8 (concern da-r1-f1).** R2 extended with the state-handoff clause introducing the new `aborted_for_scope_change` `task-history.json.status` value (distinct from `in_progress` so R7.1 doesn't misclassify the aborted task) and the no-auto-resume rule. R8 extended with the self-reference exclusion clause closing the circular-reference risk. AC-2 and AC-8 sub-clauses verify both contracts. Follow-on cross-skill schema dependency on `specflow:develop` accepting the new status value is named with the same explicit-dependency framing surgical-r1-f1 established.

### Findings rejected after Round 3

One finding had a defended push-back accepted in Round 3:
- **goal-r1-f2 (token-budget bounds at R-level).** The orphan-AC reverse-trace heuristic correctly fires when an AC's contract has no R-level expression anywhere in scope; here R12 binds the skill to `templates/orchestrator-pattern.md` by reference, and AC-13's 25K/2K numbers are explicitly framed as scope-change-specific calibrations of that template's general target. Binding-by-reference to a cited template counts as R-level expression. The Round-3 reviewer accepted the defence. AC-13's `Verifies` parenthetical was clarified to make the binding-chain explicit; no new R13 added.

### Findings escalated to human

None. All six findings converged within three rounds.

The PRD is fit to proceed to `specflow:task`. The interview's Goal section is unchanged (no scope-change-during-scope-change irony triggered). The most architecturally interesting outcome is tbc-r1-f1's parser-contract sharpen — the round+anchor-text shape for `superseded_resolved_lines` is the drift-resistant pattern that downstream skills (`/insights` Phase 3 cross-feature queries on supersession history) will read against, and naming the strikethrough markup as the canonical signal closes the human-readable-vs-machine-parseable tradeoff that interview Round 2 had left implicit.

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 2 reviewers and humans reading this as a **dogfood example** (vs. the hand-iterated calibration anchor at `001-design-skill/debate-log/prd-gate2/manifest.md` and the lower-iteration Phase 2 dogfood at `002-develop-skill/debate-log/prd-gate2/manifest.md`):

- **One `block` finding is the realistic shape for a Phase 3 first-pass PRD where one cross-skill schema gap was caught during Phase C synthesis.** The surgical-r1-f1 schema-dependency revision was pre-applied during synthesis (the AI Phase C surfaced the gap and embedded the clause directly into R10.1), so Gate 2 saw 6 findings instead of 7. The remaining `block` is goal-r1-f1's non-binary in-flight boundary at the across-source edges — exactly the Goal-Driven lens biting where the AC enumerates statuses for two sources cleanly and leaves the third (PRs) implicit. The orphan-AC reverse-trace lens (added to Goal-Driven Reviewer per E4) fired once and held on push-back the second time, which is the right calibration — the heuristic catches actual coverage gaps but releases when binding-by-reference covers the same ground.
- **Each reviewer fired its lens distinctly.** Simplicity flagged speculative configurability (a single new flag); Surgical flagged cross-skill creep (already pre-applied in synthesis but the dependency clause needed Gate 2 confirmation for the audit trail); Think-Before-Coding flagged an unstated parser-contract assumption (strikethrough markup as machine-readable signal); Goal-Driven flagged a non-binary AC boundary AND an orphan AC (two distinct sub-lenses); Devil's Advocate flagged a cross-skill state contamination (lifecycle-handoff between `specflow:develop` and `specflow:scope-change`). No two reviewers flagged the same finding.
- **The push-back was defensible and accepted.** The split response on goal-r1-f2 (defend R12's binding-by-reference, no new R13) is the right shape — the orphan-AC heuristic should release when the AC's contract HAS R-level expression via a cited canonical template. Adding R13 would have been redundant duplication of `templates/orchestrator-pattern.md`'s contract; declining to add it preserves the surgical-changes principle.
