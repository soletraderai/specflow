# Debate manifest — Gate 2: PRD vs interview review

**Feature:** 008-optimize-skill
**Artefact under review:** `008-optimize-skill-prd.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06 18:42

This is the dogfood Gate 2 run for the Phase 3 `/optimize` PRD. The skill is the most architecturally novel of Phase 3 — it is the only Phase 3 skill that modifies other skills' prompts and the only one that generalises an existing Phase 1 discipline-installer (`simplify`) across the verifiable-skill catalog. The PRD was synthesised first-pass from the interview (no hand-iteration before Gate 2 ran), so this manifest is the authentic adversarial pass through a real first-pass PRD on an architecturally-novel skill — including two `block` findings, one push-back partially defended, and the convergence path back to a passing PRD.

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
- **simplicity-r1-f1** — *concern* — R1 + AC-2 introduce `config.json.optimize.judgementWords` configurable extension without a documented project-level second consumer (speculative configurability per the AI's framing). (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — R12(c) + AC-11(c) require a workflow file at `.github/workflows/optimize-merge-gate.yml` whose ownership and install mechanism are unspecified; cross-skill creep risk. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *block* — R13 + AC-12's decline-feedback mechanism has unstated semantics for "consecutive decline" (does a refused run reset the streak?) and "7 days have passed" (wall-clock vs cron-schedule); two implementations would produce different counter values. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **tbc-r1-f2** — *concern* — R10's `eval_reads ⊂ produces:` mapping is an unstated assumption; the `produces:` field's documented semantics ("files the skill emits") and the eval's read surface are usually the same set but can diverge. (Same file.)
- **goal-r1-f1** — *block* — R11's score-direction handling references a `direction: maximize|minimize` declaration mechanism that no R-level requirement establishes; AC-10 verifies a contract the PRD didn't make (orphan AC); cross-skill schema dependency on every initial-six target's `eval:` frontmatter shape is buried. (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — Goal Outcome names Phase 3 corpus-mining as a success surface; AC-15 verifies presence-of-fields but not the corpus integrity invariants (id uniqueness, operator preservation on declined records, target preservation on refused records, JSONL line-parseability) Phase 3 mining relies on. (Same file.)
- **da-r1-f1** — *concern* — R7 + R8 + R13 + R14 interact at runtime in undocumented ways: 30-day-skip + manual-invocation path is silent; stale-branch accumulation across long-running `/optimize` adoption is undocumented. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **push_back** (defended R1's configurable extension; interview Round 1 line 38 records the user-driven edit naming a specific use case (`publishable` in a publishing-stack project) — that is the documented consumer ask the Simplicity sub-clause requires)
- surgical-r1-f1 → **accept** (applied option (a): `/optimize` named as workflow file owner; `.github/workflows/optimize-merge-gate.yml` added to `produces:`; install mechanism specified; AC-11 extended with install sub-clause)
- tbc-r1-f1 → **accept** (applied option (a): R13 extended with explicit semantics for "consecutive decline" (counter walks `merge_decision IN (closed-without-merge, merged)` only) and "7 days have passed" (wall-clock comparison against `ended_at`); AC-12 extended with two sub-clauses)
- tbc-r1-f2 → **accept** (R10 extended with `eval_reads ⊂ produces:` strict contract; AC-9 extended with sub-clause (d) verifying the check fires)
- goal-r1-f1 → **accept** (added new R17 codifying structured score-block format AND naming cross-skill schema dependency on per-target enhancement PRDs; R11 simplified to delegate to R17; AC-10's Verifies updated to cite R17)
- goal-r1-f2 → **accept** (AC-15 extended with four corpus-integrity sub-clauses: id uniqueness, operator preservation on declined records, target+refuse_reason preservation on refused records, JSONL line-parseability)
- da-r1-f1 → **accept** (R13 extended with manual-override clause; new AC-16 added; new Non-goals entry deferring stale-branch housekeeping to separate skill)

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- simplicity-r1-f1 → **accept** (defended push-back held: interview Round 1 line 38 records user-driven edit naming a specific use case — same calibration as 002-develop-skill Gate 2 simplicity-r1-f1's split outcome; the speculative-knob count stays at 1, defensible because user explicitly demanded the knob during grilling)
- surgical-r1-f1 → **accept** (single-skill ownership of the workflow file is the right calibration; `produces:` surface is now complete)
- tbc-r1-f1 → **accept** (explicit semantics make R13 implementable without invented comparison logic; the Goodharting-detection signal now fires reliably; block resolved)
- tbc-r1-f2 → **accept** (the unstated assumption is now stated AND enforced via structured pre-flight check; targets unsuitable for redirection refuse cleanly)
- goal-r1-f1 → **accept** (R17 closes the orphan-AC gap; cross-skill schema dependency named at PRD level rather than smuggled; matches 002-develop-skill AC-10 precedent for cross-skill schema dependency call-outs; block resolved)
- goal-r1-f2 → **accept** (AC-15 sub-clauses bind the skill to the integrity invariants Phase 3 mining requires)
- da-r1-f1 → **accept** (manual-override clause traces the cron-vs-manual interaction; AC-16 verifies all three branches; stale-branch housekeeping correctly deferred to separate skill matching discipline-installer precedent)

No sharpening occurred — every reviewer accepted the AI's Round 2 disposition (revisions applied or push-back-defended). No `ai-revision.md` needed in Round 3.

---

## Closing decision

**Gate 2 status: passed-with-revisions**

Two `block` findings landed in Round 1 — both accepted, both resolved with PRD revisions (R17 added; R13 semantics + manual-override clause added). Five `concern` findings landed — four accepted with revisions, one defended (the push-back on the configurable judgement-word extension, accepted by Simplicity in Round 3 as the right calibration given interview-grounded consumer ask).

### PRD revisions applied

The PRD was edited between Round 1 and Round 3 to incorporate every accepted finding:

1. **R12 + AC-11 (concern surgical-r1-f1).** R12 extended with explicit ownership clause naming `/optimize` as the workflow file's owner; install mechanism specified (write on first run if missing, idempotent); workflow file path added to the `produces:` surface (declared in `skills/optimize/SKILL.md` frontmatter). AC-11 extended with install sub-clause verifying the file is created on first invocation and unchanged on subsequent ones.
2. **R13 + AC-12 (block tbc-r1-f1).** R13 rewritten with explicit semantics for "consecutive decline" (counter walks `merge_decision IN (closed-without-merge, merged)` only — refused runs are skipped, neither resetting the streak nor counting toward it) and "7 days have passed" (wall-clock minus `ended_at` of prior decline). AC-12 extended with two streak-state sub-clauses verifying both directions (refused runs don't reset; merged runs do reset).
3. **R10 + AC-9 (concern tbc-r1-f2).** R10 extended with the strict `eval_reads ⊂ produces:` contract: every file the target's `eval:` field clauses read must be enumerated in the target's `produces:` field; targets with eval reads outside `produces:` refuse at pre-flight. AC-9 extended with sub-clause (d) verifying the check fires.
4. **New R17 + R11 simplification + AC-10 update (block goal-r1-f1).** Added R17 codifying the structured score-block format (`score: { signal: "...", direction: "maximize|minimize" }`) AND naming the cross-skill schema dependency on per-target enhancement PRDs (one per initial-six target). R11 simplified to delegate the direction-declaration mechanism to R17. AC-10's `Verifies:` line updated from "R11" to "R11, R17". The orphan-AC gap is closed and the cross-skill schema dependency is named at the PRD level.
5. **AC-15 corpus-integrity sub-clauses (concern goal-r1-f2).** AC-15 extended with four sub-clauses verifying corpus integrity: (a) id uniqueness across records, (b) winner_operator preservation on `closed-without-merge` records, (c) target+refuse_reason preservation on `null-failed-run` records, (d) JSONL line-parseability under concurrency (with R14's lockfile named as the load-bearing mechanism). The goal Outcome surface for Phase 3 corpus-mining is now AC-enforced.
6. **R13 manual-override clause + new AC-16 + Non-goals entry (concern da-r1-f1).** R13 extended with manual-override clause: manual invocation on a 30-day-skipped target proceeds with the chat-line warning surfaced; a subsequent `merged` decision clears the skip; a subsequent `closed-without-merge` does NOT extend the skip. New AC-16 verifies all three branches of the manual-override path. New Non-goals entry deferring stale-branch housekeeping to a separate skill (matches discipline-installer precedent of keeping the skill bounded).

### Findings rejected after Round 3

One finding had a defended outcome where the AI pushed back and the reviewer accepted:
- **simplicity-r1-f1 (R1's configurable judgement-word extension).** Interview Round 1 (line 38) records the user explicitly editing the AI's recommended answer to add the configurability AND naming a specific use case (publishing-stack project's `publishable` word). That is a user-driven edit during grilling, not an AI-recommended-then-user-confirmed extension — meets the Simplicity sub-clause's "concrete second consumer demanding" threshold. The Round-3 reviewer accepted the defence under the same calibration as 002-develop-skill Gate 2 simplicity-r1-f1's split outcome: where the principle's evidence threshold is met by an interview-grounded user-driven edit, defend; where it isn't, accept. The speculative-knob count stays at 1 (the configurable extension), which is acceptable given the user's explicit demand during grilling.

### Findings escalated to human

None. All seven findings converged within three rounds.

The PRD is fit to proceed to `specflow:task`. No revisions to the interview's Goal section were required (no scope-change triggered). The most architecturally-load-bearing addition is R17 (score-direction declaration as cross-skill schema dependency) — without it, AC-10 verified a contract no R-level requirement established, and the dependency on every initial-six target's `eval:` frontmatter shape would have been silently smuggled into implementation.

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 2 reviewers and humans reading this as a **dogfood example** for Phase 3 architecturally-novel skills (vs. the hand-iterated calibration anchor at `001-design-skill/debate-log/prd-gate2/manifest.md` and the Phase 2 first-pass anchor at `002-develop-skill/debate-log/prd-gate2/manifest.md`):

- **Two `block` findings is the realistic shape for an architecturally-novel first-pass PRD.** This run surfaced an orphan AC with cross-skill schema creep (goal-r1-f1 — score-direction declaration mechanism) and an unstated load-bearing assumption (tbc-r1-f1 — decline-feedback streak-state semantics). Both are exactly the failure modes Phase 3's novelty surface invites: the skill modifies other skills' contract surfaces, and the discipline-installer pattern's state machine has nuanced semantics that don't write themselves into a PRD.
- **The relationship to `simplify` is the most load-bearing context for reviewers.** `/optimize` is the *generalisation* of `simplify`'s discipline-installer pattern across the verifiable-skill set — branch-per-run (R8), sequential variants (R5), no LLM-as-judge (implicit throughout, structurally enforced by R6's binary eval surface), human merge owns taste (R12's structural lockout). Reviewers who don't have `simplify`'s SKILL.md as context risk re-litigating already-locked-in discipline tenets.
- **Each reviewer fired its lens distinctly.** Simplicity flagged speculative configurability (one config knob); Surgical flagged unowned cross-skill resource (workflow file); Think-Before-Coding flagged unstated state semantics (decline-counter) plus an unstated mapping assumption (`eval_reads ⊂ produces:`); Goal-Driven flagged an orphan AC plus a goal-coverage gap (corpus integrity invariants); Devil's Advocate flagged an interaction-mode coverage gap (cron-vs-manual on 30-day-skipped target). No two reviewers flagged the same finding; lens distinctness held.
- **The push-back was defensible and accepted.** The defence on simplicity-r1-f1 is the right shape — the interview-grounding (user-driven edit naming a specific use case) is exactly what the Simplicity sub-clause's evidence threshold names. Accepting both findings would have been rubber-stamping at the cost of dropping a knob the user explicitly demanded.
- **No findings escalated.** All seven converged within three rounds. R17 is the most load-bearing addition — the cross-skill schema dependency it makes explicit (every initial-six target needs the structured score block before `/optimize` v1 can run on them) is the kind of dependency that, if buried in implementation, would have surfaced as a Phase 4 surprise: `/optimize` shipping but unable to actually run on `feedback-loop-audit` because the target's eval frontmatter wasn't structured.
- **The recursive-bootstrap dimension surfaces nowhere in the findings.** `/optimize` can target `simplify` (and itself) as a verifiable skill — that recursive case is mentioned in Topics not discussed and left to surface naturally in the corpus rather than be specified up front. Reviewers correctly did NOT bite on it: there's no concrete failure mode yet, and adding a configurability knob to disable self-targeting before any failure mode has surfaced would be exactly the speculative configurability Simplicity correctly flags. The discipline holds.
