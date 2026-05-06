# Debate manifest — Gate 2: PRD vs interview review

**Feature:** 006-insights-skill
**Artefact under review:** `006-insights-skill-prd.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06

This is the dogfood Gate 2 run for the Phase 3 `/insights` PRD. The PRD was synthesised first-pass from the interview (no hand-iteration before Gate 2 ran), so this manifest is the authentic adversarial pass through a real first-pass PRD on a Phase 3 self-evolution skill — including six concern findings, two block findings, two push-backs (one full + one partial), and full convergence within three rounds. Calibration sibling: `003-complete-skill/debate-log/prd-gate2/manifest.md` (the closest Phase 3 anchor — same gate-shape, 7 findings + 2 push-backs).

---

## Round 1 — Findings

| Reviewer | Findings (severity) |
|---|---|
| simplicity-reviewer | 1 (concern) |
| surgical-reviewer | 1 (concern) |
| think-before-coding-reviewer | 2 (block, concern) |
| goal-driven-reviewer | 2 (block, concern) |
| devils-advocate | 1 (concern) |
| **Total** | **7 findings (2 block, 5 concern)** |

Detail:
- **simplicity-r1-f1** — *concern* — R3 + Schema Appendix reserve a `semantic` cluster-source label and a `semantic_clusters` runs.jsonl counter for a v2 capability that no documented consumer has asked for; speculative configurability under Simplicity First sub-clause. (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — R1 + R2 + R4 implicitly require `specflow:complete`'s v1 Schema Appendix to be stable, but no R-level requirement names the cross-skill schema dependency. Unstated cross-skill contract, mirrors 003-complete-skill's surgical-r1-f1 precedent. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *block* — R6 + R7's cadence knob design has zero behavioural effect inside the skill (changes only R7's chat-line, never enforces cadence) but the load-bearing assumption that the cadence knob is purely-informational is buried in R7's parenthetical, not named as a contract; downstream readers may infer enforcement that doesn't exist. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **tbc-r1-f2** — *concern* — R9's substring-match-on-rule-id approach for detecting guideline-flagged violations eliminates the alternative of a structured `rule_violations` field on `task-history.json` entries, without articulating why the elimination happened or naming the false-positive/false-negative costs. (Same file.)
- **goal-r1-f1** — *block* — AC-16 + the runs.jsonl Schema Appendix track a `promotions_deferred` count for a state ('user neither accepted nor rejected at this run') that no R-level requirement establishes; R8's three-option binary prompt forbids deferral. Orphan AC clause, reverse traceability failure under E4. (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — AC-1's 'listing every cluster' phrasing is ambiguous on the empty-cluster case (corpus passes R10's minCorpusSize but no clusters reach the 3-observation threshold). A fresh agent verifying AC-1 against an empty-cluster report could read it two ways. (Same file.)
- **da-r1-f1** — *concern* — R6's dual-trigger-path contract (manual + cron) creates a TOCTOU race when both paths fire concurrently on the same calendar month; R11's replace-in-place semantic makes the race more dangerous than `specflow:complete`'s because both paths could produce divergent reports interleavingly. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **push_back** (defended the `semantic` cluster-source reservation as interview-grounded — interview Round 1's resolution explicitly names the reservation as part of v1 design, the user signed it off; the runs.jsonl `semantic_clusters` counter is the additivity surface that makes v2 embedding clustering an additive-not-breaking schema revision)
- surgical-r1-f1 → **accept** (added new R14 + AC-14 with explicit cross-skill schema dependency clause; mirrors 003-complete-skill AC-4 precedent)
- tbc-r1-f1 → **accept** (option (b): documentation-only Note in R6 + R7 articulating the cadence knob's purely-informational status as a v1 contract; partial push-back on option (a)'s four-way determinism check — verification cost not justified for a no-behaviour-change contract)
- tbc-r1-f2 → **accept** (R9 rationale extended to articulate the substring-match-vs-structured-field tradeoff; v2 structured-field path named as the higher-fidelity follow-up; v1 false-positive risk mitigated by R8's three-option human-confirmed prompt)
- goal-r1-f1 → **accept** (option (b): dropped the `promotions_deferred` counter from AC-16 chat-line tokens AND from the runs.jsonl Schema Appendix; reverse traceability now clean — every counter traces to an R8-defined user choice)
- goal-r1-f2 → **accept** (AC-1 extended with explicit empty-cluster sentinel-body convention; binary check now exhaustively covers the cluster-list and empty-cluster cases)
- da-r1-f1 → **accept** (added new R15 + AC-15 specifying the per-month lock-file pattern at `admin/scratch/insights-{YYYY-MM}.lock` with 60-minute stale-lock heuristic and structured refusal chat line)

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- simplicity-r1-f1 → **accept** (defence held: the reservation is interview-grounded per Round 1's resolution; the v2 embedding-clustering downstream consumer is documented in the goal's Out-of-scope item 3; the runs.jsonl counter is the additivity-not-breaking surface; principle bites where speculation isn't anchored, releases where second-consumer evidence is documented in interview + goal)
- surgical-r1-f1 → **accept** (cross-skill dependency now named at both R-level and AC-level; lockstep-revision clause in R14 closes the maintenance-drift loop; mirrors 003-complete-skill AC-4 precedent)
- tbc-r1-f1 → **accept** (load-bearing assumption no longer hidden; documentation-only Note is the right calibration without four-way determinism check; partial push-back held; block resolved)
- tbc-r1-f2 → **accept** (tradeoff articulation makes the substring approach defensible; documentation-only path; matches 003-complete-skill tbc-r1-f2 outcome pattern)
- goal-r1-f1 → **accept** (orphan-AC clause removed; reverse traceability clean; option (b) was the right surgical edit; block resolved)
- goal-r1-f2 → **accept** (empty-cluster handling now binary; sentinel-body convention is the canonical handling; concern resolved)
- da-r1-f1 → **accept** (R15 + AC-15 close the TOCTOU window; lock-file pattern matches Phase 1 + Phase 3 scratch convention; 60-minute stale-lock threshold calibrated for /insights's longer interactive surface; concern resolved)

No sharpening occurred — every reviewer accepted the AI's Round 2 disposition (revisions applied or push-back-defended). No `ai-revision.md` needed in Round 3.

---

## Closing decision

**Gate 2 status: passed-with-revisions**

Two `block` findings landed in Round 1 — both accepted, both resolved with PRD revisions. Five `concern` findings landed — three accepted with revisions, one defended via full push-back, one accepted with partial push-back (option (a) verification-cost-not-justified). The principle reviewers accepted the defences in Round 3 as the right calibration. Six PRD revisions were applied between Round 2 and Round 3 (one new R + AC pair for surgical-r1-f1, one rationale extension on R6 + R7 for tbc-r1-f1, one rationale extension on R9 for tbc-r1-f2, one AC scope reduction + Schema Appendix line removal for goal-r1-f1, one AC empty-cluster extension for goal-r1-f2, one new R + AC pair for da-r1-f1). The PRD is fit to proceed to `specflow:task`.

### PRD revisions applied

The PRD was edited between Round 1 and Round 3 to incorporate every accepted finding:

1. **R14 + AC-14 cross-skill schema dependency clause (concern surgical-r1-f1).** New R14 specifying schema-stability check on read with explicit cross-skill dependency clause naming `003-complete-skill-prd.md`'s Schema Appendix as the authoritative contract; AC-14 verifies the schema-drift refusal sentinel chat line. The cross-skill dependency is now named at both R-level and AC-level rather than buried in implementation; mirrors the 003-complete-skill AC-4 precedent.

2. **R6 + R7 cadence-knob informational-status Note (block tbc-r1-f1).** R7 extended with the explicit Note: 'the cadence knob is purely informational inside this skill. It changes only this chat-line; cluster detection and refusal logic are cadence-independent — running with `cadence=weekly` produces the same cluster output as `cadence=manual-only` on the same corpus. v2 enhancements adding cadence enforcement (refuse-too-soon, etc.) would be breaking changes against this v1 contract.' R6's trace updated. Partial push-back: option (a)'s four-way determinism check was rejected as verification-cost-not-justified; documentation-only Note is the right calibration.

3. **R9 substring-match tradeoff articulation (concern tbc-r1-f2).** R9 rationale extended to articulate the substring-match-vs-structured-field tradeoff: implementation-cheap; works against existing schema; known false-positive + false-negative risks; v1 false-positive mitigated by R8's three-option human-confirmed prompt; v2 may add structured `rule_violations` field as `specflow:complete` schema enhancement; v1 false-negative is the cost of substring matching's lexical-not-semantic limit (embedding-based matching deferred per R3).

4. **AC-16 + Schema Appendix deferred-counter removal (block goal-r1-f1).** AC-16's chat-line tokens reduced from six to five (dropped `(`{D}` deferred)` segment and the deferred-state definition); runs.jsonl Schema Appendix dropped the `promotions_deferred: 0` line. Reverse traceability now clean: every counter in the runs.jsonl traces to an R8-defined user choice (accept-and-write, edit-and-write, reject). Option (b) was the right surgical edit; option (a) (add a fourth user-prompt option for deferral) would have expanded R8's binary-prompt contract without a documented consumer ask.

5. **AC-1 empty-cluster sentinel handling (concern goal-r1-f2).** AC-1 extended with the explicit empty-cluster sentinel-body convention: 'when zero clusters meet the threshold for a section, the section's body contains the literal sentinel: "No clusters above the 3-observation threshold this run." Sections are always present (even when empty); a report missing either section header is a failed render.' Binary-check surface now exhaustively covers the cluster-list and empty-cluster cases.

6. **New R15 + AC-15 concurrent-trigger guard (concern da-r1-f1).** Added R15 specifying the per-month lock-file pattern at `admin/scratch/insights-{YYYY-MM}.lock` with the 60-minute stale-lock heuristic and structured refusal chat line. AC-15 verifies the create/remove discipline and the stale-lock case. The 60-minute threshold (longer than 003-complete-skill's 30-minute) is calibrated for /insights's longer interactive surface (potentially many promotion proposals to walk through). Open Questions section retains the stale-lock-knob question as the v2 follow-up shape.

### Findings rejected after Round 3

One finding had a full push-back the AI defended successfully:

- **simplicity-r1-f1 (full push-back on dropping the `semantic` cluster-source reservation).** The reservation is interview-grounded per Round 1's resolution (line 39 of `006-insights-skill-interview.md`): 'embedding-based clustering deferred to v2 with a `semantic` cluster-source label reserved in the report schema.' The user explicitly confirmed the reservation as part of the v1 design. The Simplicity sub-clause's anti-pattern targets speculative configurability (knobs introduced without a documented consumer ask); the reservation isn't a config knob, it's a schema-shape decision. The downstream consumer (v2 embedding-based clustering) is documented in the goal's Out-of-scope item 3. The runs.jsonl `semantic_clusters: 0` counter is the additivity surface — without it, the v2 PR adding embedding clustering becomes a breaking schema revision (every prior month's runs.jsonl needs migration). Round 3 simplicity-reviewer accepted: the principle bites where speculation isn't anchored, releases where second-consumer evidence is documented in interview + goal. Pattern matches the 003-complete-skill simplicity-r1-f1 push-back outcome (elevation triple-flag defended on identical interview-grounded second-consumer-evidence basis).

One finding had a partial push-back the AI defended successfully:

- **tbc-r1-f1 (partial push-back on option (a)'s four-way determinism check).** The proposed AC extension to verify 'cluster-detection behaviour is identical regardless of `config.json.insights.cadence` value' would require running the cluster pipeline four times (once per cadence value) on the same corpus and byte-comparing outputs as a test surface — verification-cost-not-justified for a no-behaviour-change contract. Option (b) (documentation-only Note in R6 + R7) was the right calibration: matches the principle's 'state assumptions explicitly' framing without forcing a four-way determinism check into the test plan. The block was resolved by option (b)'s documentation alone; the partial push-back on option (a) prevented test-plan bloat.

### Findings escalated to human

None. All seven findings converged within three rounds.

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 2 reviewers and humans reading this as a **dogfood example** on a Phase 3 self-evolution skill (smaller surface area than Phase 2 calibration anchors but larger than `specflow:complete` — `/insights` has read-mostly I/O against multiple admin files and writes to the rules registry on user accept):

- **Two `block` findings on a Phase 3 self-evolution skill is a realistic shape.** Bigger I/O surface than `specflow:complete` (reads `task-history.json` + `decision-log.md` + `admin/rules/*`; writes `admin/insights/*` + `admin/rules/*` on user accept), so the lenses had more surface to bite on. Think-Before-Coding flagged a hidden load-bearing assumption (cadence knob's purely-informational status); Goal-Driven flagged an orphan AC counter (deferred state with no R-level expression). Both are exactly the failure modes typical of first-pass PRDs that synthesise from interview without hand-iteration.
- **Two push-backs (one full + one partial) on five concerns is the right friction shape.** Accepting all seven would have been rubber-stamping; pushing back on more than two would have suggested the principles weren't biting. The full push-back (simplicity's interview-grounded schema reservation) and the partial push-back (Think-Before-Coding's option-(a) verification-cost defence) are the calibration shape that says 'principles bite where evidence is weak, release where evidence is strong'. Same pattern as 003-complete-skill's split push-back surface.
- **The cross-skill schema dependency finding (surgical-r1-f1) is a Phase 3 calibration signal.** This is the second consecutive Phase 3 PRD where the surgical reviewer caught an implicit cross-skill schema dependency (003-complete-skill's surgical-r1-f1 caught the `superseded_by_retro` flag dependency on `specflow:develop`; this PRD's surgical-r1-f1 caught the v1 Schema Appendix dependency on `specflow:complete`). The pattern is: Phase 3 skills inevitably depend on Phase 2 / earlier-Phase-3 skills' output schemas; the dependency must be named explicitly at both R-level and AC-level, with a lockstep-revision clause covering schema evolution. Worth a `/insights`-cadence pattern note (recursive dogfooding!) — three consecutive cross-skill schema findings would surface as a candidate guideline promotion under R9's substring-match logic, with the rule id `NAME_CROSS_SKILL_SCHEMA_DEPS_EXPLICITLY` as a plausible auto-suggested id.
- **The race-condition finding (da-r1-f1) is the second instance of the Phase 3 dual-trigger-path pattern.** R6's manual + cron contract creates the same TOCTOU race the 003-complete-skill PRD's R1 + R2 (auto-fire + manual) created. The lock-file pattern is the canonical fix at this layer; the calibration of the stale-lock threshold (30 minutes for `specflow:complete`, 60 minutes for `/insights`) tracks the skill's interactive-surface depth. Future Phase 3 skills with multiple legitimate trigger paths should ship the lock-file pattern from PRD-write rather than waiting for Devil's Advocate to catch it at Gate 2 — a `/insights`-cadence pattern note in its own right.
