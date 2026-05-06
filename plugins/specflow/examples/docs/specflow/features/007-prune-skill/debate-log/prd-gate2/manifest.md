# Debate manifest — Gate 2: PRD vs interview review

**Feature:** 007-prune-skill
**Artefact under review:** `007-prune-skill-prd.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06

This is the Gate 2 run for the Phase 3 `/prune` PRD — the quarterly registry-pruning cadence skill. The PRD was synthesised from the six-round interview, then submitted unchanged for adversarial review. This manifest closes the round; all five Round-1 findings converged within three rounds (one push-back-with-concession on simplicity, four straight accepts).

---

## Round 1 — Findings

| Reviewer | Findings (severity) |
|---|---|
| simplicity-reviewer | 1 (concern) |
| surgical-reviewer | 1 (concern) |
| think-before-coding-reviewer | 1 (block) |
| goal-driven-reviewer | 1 (block) |
| devils-advocate | 1 (concern) |
| **Total** | **5 findings (2 block, 3 concern)** |

Detail:
- **simplicity-r1-f1** — *concern* — five `admin/config.json` knobs introduced for v1 (`prune.thresholds.decisionLog.{ageDays, dormancyDays}`, `prune.thresholds.guidelines.dormancyDays`, `prune.thresholds.taskHistory.{ageDays, dormancyDays}`, `prune.archiveRetention`, `prune.staleLockMinutes`); the retention knob's v1 surface is a refusal message — speculative configurability with extra steps. (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — R12 Stage 2's source-registry mutation has four distinct shapes per surface (decision-log untouched; rules removed; agents file-moved; task-history `archived_at` set); decision-log already demonstrates the cleanest pattern (read-only at Stage 2; archive file is single source of truth); the asymmetry creates four post-mutation invariants to verify. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *block* — R3 + AC-3 specify title-based citation detection but the existing rules in the worked example (`PREFER_LOCAL_TESTS`, `PREFER_COMPOSITION_OVER_INHERITANCE`) cite decision-log entries by date, not title; title-based detection produces zero candidates against the existing rules — false negative; the bridge between the existing date-based citation convention and the title-based supersede-link signal is unstated. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **goal-r1-f1** — *block* — round-trip restoration property (an archived item, restored from the archive file, reproduces source-registry pre-archive state byte-for-byte) is the goal-eval contract per SKILLS.md line 190 but exists nowhere in the R-set as an explicit requirement; AC-11 verifies the byte-identical-capture mechanism but no AC verifies the restoration property the mechanism serves (orphan AC). (See `findings/round-1/goal-driven-reviewer.json`.)
- **da-r1-f1** — *concern* — R8's lock-concurrency machinery has two unaddressed cases: (i) `--force-unlock` race window (cron path can capture the freed lock between unlock and re-invoke); (ii) hours-scale clock-skew at the cron host silently invalidates the lock semantics in two distinct ways (battery-failed past clock allows concurrent runs; misconfigured-timezone future clock makes the lock un-removable). The 60-second tolerance R8 names handles NTP-drift scale, not the actual failure mode. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **push_back_with_concession** (defended `prune.thresholds` knobs citing interview Round 1 line 35 user-edit naming `admin/config.json prune.thresholds` configurability with the user's named rationale 'so projects on different cadences can tune' — interview-grounded, not speculative; conceded `prune.archiveRetention` knob — dropped from R7 entirely, moved to Open Questions; confirmed `prune.staleLockMinutes` is already a v2-promotion-candidate per R8)
- surgical-r1-f1 → **accept** (stated revision: option (a) refined — uniform read-only Stage 2 across decision-log, rules, task-history; bespoke Stage 2 retained for agent snapshots only because the live agent registry is the runtime surface — pruning means moving-from-runtime; structural exception with named rationale)
- tbc-r1-f1 → **accept** (stated revision: option (a) — citation-resolution helper extracts both title (quoted) and date (`YYYY-MM-DD`) forms; for date citations, lookup the matching `**Date:**` field; once resolved to a title, supersession detection is title-based per `specflow:decision`'s R6 + AC-5)
- goal-r1-f1 → **accept** (stated revision: option (b) — keep R12 as the single requirement covering Stage 1 + Stage 2 but explicitly name round-trip restoration as the R-level contract Stage 1's byte-identical capture establishes)
- da-r1-f1 → **accept** (stated revision: extend R8 with implausible-timestamp refusal (24h-past / 1h-future bounds) AND unlock-marker priority window (5-minute manual-priority window via `admin/scratch/prune-{YYYY-Q}.unlock-marker`); update AC-8 to enumerate both handlers)

## Round 3 — Reviewers sharpen or accept

Five accepts:

- simplicity-r1-f1 → **accept** (push-back on `prune.thresholds` holds on interview-grounded specification — Round 1 line 35 user-edit AND Round 1 line 33 user-rationale-for-tuning are concrete and load-bearing; concession on `prune.archiveRetention` landed cleanly in PRD body — R7 dropped the knob; AC-7 dropped the refusal-on-non-forever-value clause; v2 enhancement path moved to Open Questions covering both `prune.archiveRetention` and `prune.staleLockMinutes`; the principle bites where speculation isn't anchored, releases where it is — well-calibrated outcome)
- surgical-r1-f1 → **accept** (asymmetric mutation collapsed correctly to uniform-read-only-on-three-surfaces + structural-exception-on-agents; R12 documents the runtime-vs-Phase-3-consumer-surface rationale once with the exception named; AC-11 inherits the read-only verification on three surfaces (re-read confirms byte-identical pre-Stage-2 state) plus mutation verification on agents (index entry no longer present + archive-agents file exists with non-empty body); round-trip restoration extends correctly to both treatments)
- tbc-r1-f1 → **accept** (block resolved — R3's citation-resolution helper is named explicitly with both extraction patterns; AC-3 enumerates the helper's two patterns AND the fall-through cases (guidelines without extractable citations fall through to dormancy clause; non-negotiable rules without resolvable citations are never candidates); the bridge between date-based citation convention and title-based supersede-link signal is the citation-resolution step the original PRD silently elided)
- goal-r1-f1 → **accept** (block resolved — R12 names round-trip restoration as the R-level contract explicitly; AC-11 verifies the property by literal restoration test; the goal-eval contract from SKILLS.md line 190 is now traced to an R-level requirement and an AC-level binary verification test; option (b) — the smaller surgical edit — was applied correctly with the restoration contract named inline rather than split into a new R)
- da-r1-f1 → **accept** (R8 extended with both handlers — implausible-timestamp refusal (24h-past / 1h-future bounds, far enough from the 60-second NTP-drift tolerance that the two don't collide) AND unlock-marker priority window (5-minute marker file at `admin/scratch/prune-{YYYY-Q}.unlock-marker` consumed by next manual invocation OR auto-expires); AC-8 enumerates both handlers in execution order — implausible-timestamp gate fires before the three-case nominal flow; unlock-marker priority applies to the next-invocation handshake after force-unlock; both edge cases the original finding flagged are now deterministic at R and AC level)

No Round-3 sharpenings.

---

## Closing decision

**Gate 2 status: passed-with-revisions**

Two `block` findings landed in Round 1 (tbc-r1-f1, goal-r1-f1) — both accepted, both resolved with PRD revisions verifiable in the PRD body. Three `concern` findings landed — two accepted with revisions applied (surgical-r1-f1 in refined-option-(a) form; da-r1-f1 with both extensions), one push-back-with-concession (simplicity-r1-f1) accepted by the reviewer in Round 3 on the load-bearing 80% (interview-grounded threshold knobs) AND accepted on the 20% concession (`prune.archiveRetention` dropped).

### PRD revisions applied

The PRD was edited between Round 1 and Round 3 to incorporate accepted findings. Revisions verifiable in the PRD body:

1. **R3 + AC-3 (block tbc-r1-f1).** R3 now names the citation-resolution helper extracting both title and date forms; the bridge to title-based supersession detection is explicit. AC-3 enumerates the helper's two patterns and the fall-through cases for rules without extractable citations.
2. **R12 + AC-11 (block goal-r1-f1).** R12 names round-trip restoration as the R-level contract Stage 1's byte-identical capture establishes. AC-11 verifies the property by literal restoration test (capture pre-archive bytes, run prune, run restoration, diff against pre-archive bytes — diff is empty for the restored entry on every surface). The orphan-AC gap is resolved.
3. **R12 + AC-11 + Goals bullet (concern surgical-r1-f1).** R12 Stage 2 is now uniform read-only on three surfaces (decision-log, rules, task-history) — the live source registry stays unmodified at Stage 2; the archive file is the single source of truth for archive state on these surfaces; Phase 3 consumers cross-reference the archive file. Stage 2 retains bespoke mutation for agent snapshots only because the live agent registry is the runtime surface; the structural rationale is documented once. AC-11 inherits the read-only-vs-bespoke split. The Goals bullet for two-stage verification is updated to match.
4. **R8 + AC-8 (concern da-r1-f1).** R8 extends the lock-concurrency machinery with implausible-timestamp refusal (24h-past / 1h-future bounds) AND the unlock-marker priority window (5-minute manual-priority window after force-unlock via `admin/scratch/prune-{YYYY-Q}.unlock-marker`). AC-8 enumerates both handlers in execution order.
5. **R7 + AC-7 + Goals bullet + Open Questions (concession from simplicity-r1-f1).** R7 dropped the `prune.archiveRetention` config knob entirely; v1 is forever-by-default with no configuration surface. AC-7 dropped the refusal-on-non-forever-value clause. The Goals bullet for archive retention is updated to match. Open Questions adds a single bullet covering `prune.archiveRetention` and `prune.staleLockMinutes` as v2 enhancement candidates gated on documented consumer ask.

**Renumbering note.** The pre-Gate-2 PRD had twelve requirements (R1-R12). Findings reference R-numbers as they were at Round 1 — surgical-r1-f1 cites R12 (verification); goal-r1-f1 cites R12 (verification + restoration). After Gate 2 closed, a final structural consolidation merged the original R11 (ambiguous user input refusal) into R6 (sub-phase orchestration) and renumbered R12 → R11; the input-contract clauses now live in R6 alongside the per-item prompt mechanics. The post-Gate-2 PRD has eleven requirements (R1-R11) and eleven acceptance criteria (AC-1 to AC-11). The renumbering is structural; the Round-1 findings' R-references retain their original numbers in the JSON artefacts as the historical record.

### Findings rejected after Round 3

One finding had a defended-with-concession outcome:
- **simplicity-r1-f1 (the `prune.thresholds` config knobs)** — interview Round 1 (line 35) records the user explicitly editing the AI-recommended threshold shape down to `admin/config.json prune.thresholds` configurability with the rationale 'so projects on different cadences can tune'. Round 1 (line 33) records the user's tuning rationale ('the dormancy threshold for rules is too aggressive for non-negotiable rules') as the surface that drove the configurability ask. The threshold knobs are interview-grounded, not AI-fabricated speculative configurability. The Round-3 reviewer accepted the defence on the load-bearing 80% — the principle bites where speculation isn't anchored, releases where it is. The concession on `prune.archiveRetention` was the right shape for the 20% the reviewer was right about — the knob's v1 surface was a refusal message, which is configurability without a consumer.

### Findings escalated to human

None. All five findings converged within three rounds (four straight accepts on the two blocks and two non-conceded concerns; one push-back-with-concession accepted by the reviewer in Round 3). The PRD is fit to proceed to `specflow:task`.

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 2 reviewers and humans reading this manifest as part of the Phase-3-skill review history:

- **Each reviewer fired its lens distinctly.** Simplicity flagged speculative configurability and was push-back-defended-with-concession (knobs were interview-grounded but one stretched the principle); Surgical flagged asymmetric Stage-2 mutation across four surfaces and proposed uniform-read-only with one structural exception; Think-Before-Coding flagged an unstated citation-extraction strategy that would have produced false negatives against the worked example's date-based citations (block); Goal-Driven flagged the round-trip restoration property as an unstated R-level contract (orphan AC, block); Devil's Advocate flagged the cron-host clock-skew and force-unlock-race edge cases the lock-concurrency machinery silently assumed away. No two reviewers flagged the same finding; lens distinctness held.
- **The push-back was defensible-with-concession.** Simplicity's defence on `prune.thresholds` cited concrete interview lines (Round 1 line 35 user-edit + Round 1 line 33 user-rationale) showing the user named the knobs and the reason for tuning. That is the consumer-ask the Simplicity sub-clause requires for v1 configurability. The concession on `prune.archiveRetention` recognised the reviewer's narrower point — a knob whose v1 surface is a refusal is itself speculative — and dropped that knob without dropping the user-grounded ones. The outcome is well-calibrated: defend the load-bearing 80%, concede the 20%. A reviewer who would have rejected the defence outright would have ignored documented user intent; a reviewer who would have accepted the defence outright would have missed the legitimate critique on the retention knob.
- **The block on tbc-r1-f1 was a worked-example-grounded block.** The reviewer caught that R3's title-based citation detection would produce zero candidates against the existing date-based citations in `examples/docs/specflow/admin/rules/guidelines.md`. This is the kind of finding that lens-fires only when the reviewer reads both the artefact under review AND the calibration anchor — a fresh agent reading just R3 might not have caught the convention mismatch. The bridge — citation-resolution helper extracting both forms — is the load-bearing piece the original PRD silently elided.
- **The block on goal-r1-f1 named the goal-eval contract directly.** SKILLS.md line 190 ('pruning is reversible (archive retained)') is the eval that was unstated as an R-level contract. R12 Stage 1's byte-identical capture is the mechanism; round-trip restoration is the contract that mechanism serves. The reviewer's option-(b) — name the contract inline within R12 — was the smaller surgical edit and matched the calibration anchor in features/004-decision-skill (where R11 was rewritten to name Stage 2 as an R-level contract via the same inline-naming pattern).
- **Watch-for: persistence ledger schema stability.** R4 and R10 use `admin/scratch/prune-history.json` as the persistence surface for the two-consecutive-run agent-drift threshold and the per-item status ledger. The schema is documented inline in the requirements but lives in `admin/scratch/`. Open Questions flags the v2 promotion path; a future implementer should confirm that schema drift across quarters does not silently invalidate the two-consecutive-run threshold computation.
- **Watch-for: cross-quarter rolling-archive partition.** R7 partitions archive files by quarter (`{YYYY-Q}-prune.md`), and R8's lock follows the same partition (`prune-{YYYY-Q}.lock`). The interaction across quarter boundaries (e.g. cron-fire on the first day of a new quarter while the prior quarter's run was still mid-deferral) was not surfaced in the interview; Open Questions recommends file-naming follows the run-fire's calendar date and per-quarter locks make boundary collisions a non-issue, but a future implementer should confirm at implementation.
