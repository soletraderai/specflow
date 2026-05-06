# Debate manifest — Gate 2: PRD vs interview review

**Feature:** 004-decision-skill
**Artefact under review:** `004-decision-skill-prd.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06

This is the Gate 2 run for the Phase 3 `specflow:decision` PRD — the lightweight out-of-band decision-capture skill. The PRD was synthesised from the six-round interview, then submitted unchanged for adversarial review. This manifest closes the round; one Round-3 sharpening was raised against an accepted-but-unapplied revision and is documented below for the next iteration to pick up.

---

## Round 1 — Findings

| Reviewer | Findings (severity) |
|---|---|
| simplicity-reviewer | 1 (concern) |
| surgical-reviewer | 1 (concern) |
| think-before-coding-reviewer | 2 (block, concern) |
| goal-driven-reviewer | 1 (block) |
| devils-advocate | 1 (concern) |
| **Total** | **6 findings (2 block, 4 concern)** |

Detail:
- **simplicity-r1-f1** — *concern* — R7's mtime-based soft prompt is speculative state-aware machinery built around an informational-only nudge; R6's duplicate-title check could be the sole safety net for v1. (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — R8's file-missing path has `specflow:decision` carry the canonical preamble text in its prompt body, duplicating `specflow:setup`'s seed template (cross-skill creep). (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *block* — R5's "most-recently-written existing entry" trigger has three plausible interpretations (file-order-last, date-field-most-recent, git-blame-most-recent); the PRD does not state which, so AC-4 cannot fire deterministically. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **tbc-r1-f2** — *concern* — R9's parse strategy is unstated as a tradeoff; the preamble-only-no-entries edge case requires head-reading while the existing-entries case requires tail-reading, and the PRD does not name the read strategy. (Same file.)
- **goal-r1-f1** — *block* — AC-10's literal chat-line shape is a contract no R-level requirement establishes; R11 frames the chat line as a verify-step mechanism, not as the user-facing output contract being verified (orphan AC). (See `findings/round-1/goal-driven-reviewer.json`.)
- **da-r1-f1** — *concern* — R7's mtime check has two unaddressed edge cases: (i) `task-history.json` does not exist (fresh project), and (ii) file exists but contains zero task entries (e.g. setup just touched it) — the soft prompt would fire incorrectly. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- simplicity-r1-f1 → **push_back** (defended R7 citing interview Round 4 line 50's user-edit naming the mtime-on-`task-history.json` shape with the 60-minute window as the user's explicit specification — interview-grounded configurability, not speculative)
- surgical-r1-f1 → **accept** (stated revision: rewrite R8 to refuse on file-missing, point at `specflow:setup`, eliminating the cross-skill duplication; option (a))
- tbc-r1-f1 → **accept** (stated revision: define "most-recently-written" as "the last entry in file order" with explicit note that file order is the canonical reading order)
- tbc-r1-f2 → **accept** (stated revision: name the read strategy as full-file-read with practical-size justification)
- goal-r1-f1 → **accept** (stated revision: option (b) — keep R11 as a single requirement but reframe Stage 2 as an R-level user-surface contract, not a verify-step mechanism)
- da-r1-f1 → **accept** (stated revision: extend R7 with explicit file-missing path AND empty-file-with-recent-mtime path; update AC-6's branch list to match)

## Round 3 — Reviewers sharpen or accept

Five accepts, one sharpen:

- simplicity-r1-f1 → **accept** (interview Round 4 citation is concrete and load-bearing; the 60-minute window is the user's named specification, not a fabricated constant; the principle bites where speculation isn't anchored, here it's anchored)
- surgical-r1-f1 → **sharpen** (Round-2 disposition was correct in shape but the agreed revision did not land in the PRD body — R8 line 76 still carries the original duplicate-preamble shape, AC-7 line 106 still verifies the create-on-missing path; sharpened claim spells out the exact R8 / AC-7 / AC-10 / Goals-bullet edits required to land the cross-skill duplication fix)
- tbc-r1-f1 → **accept** (block resolved — R5's file-order definition is now explicit, AC-4 inherits the same definition, the three-way ambiguity is gone)
- tbc-r1-f2 → **accept** (PRD chose tail-only read instead of full-file read, but named the choice explicitly with scope-of-the-parse clause; the corner-case coverage gap (preamble-only and existing-entries) is closed in AC-8 either way)
- goal-r1-f1 → **accept** (block resolved — R11 now names Stage 2 as the binary user-surface contract, AC-10 verifies the contract by literal-text match, the orphan-AC gap is gone)
- da-r1-f1 → **accept** (file-missing path is named explicitly in R7 and AC-6; empty-file-with-recent-mtime case is implicitly covered by the goal-anchored "never blocks" principle and the file-missing rationale "no `task-history.json` means no boundary to check" — flagged as watch-for so a future implementer confirms explicit empty-file handling)

One Round-3 sharpening was raised; see `findings/round-3/surgical-reviewer.json` for the concrete unapplied edits.

---

## Closing decision

**Gate 2 status: passed-with-revisions**

Two `block` findings landed in Round 1 (tbc-r1-f1, goal-r1-f1) — both accepted, both resolved with PRD revisions verifiable in the PRD body. Four `concern` findings landed — three accepted with revisions applied (surgical-r1-f1 partial, tbc-r1-f2 in alternate form, da-r1-f1 partial), one defended via push-back accepted by the reviewer in Round 3 (simplicity-r1-f1).

### PRD revisions applied

The PRD was edited between Round 1 and Round 3 to incorporate accepted findings as documented below. Revisions verifiable in the PRD body at the cited line ranges:

1. **R5 + AC-4 (block tbc-r1-f1).** R5 (line 64) now defines "most-recently-written existing entry" as "the last entry in file order" with the explicit note that file order is the canonical reading order, not date order. AC-4 (line 100) inherits the same definition; the warning text now names file-order with title + line number per the user's Round 2 edit. The trace line (line 65) cites Gate 2 tbc-r1-f1 grounding.
2. **R11 + AC-10 (block goal-r1-f1).** R11 (line 88) now names Stage 2 as an R-level contract explicitly: *"Stage 1 is the binary verify step; Stage 2 is the binary user-surface contract."* The trace line (line 89) cites Gate 2 goal-r1-f1 grounding. AC-10 (line 112) verifies the contract by literal-text match; the orphan-AC gap is resolved.
3. **R7 + AC-6 (concern da-r1-f1, partial).** R7 (line 72) now names the file-missing path explicitly: *"when `admin/task-history.json` does not exist (fresh project, or `specflow:complete` has never written), the soft prompt does not fire — the skill proceeds directly to the write step."* AC-6 (line 104) inherits the file-missing branch verbatim. The empty-file-with-recent-mtime sub-case is implicitly covered by the goal-anchored "never blocks" principle but not named explicitly — flagged as watch-for.
4. **R9 + AC-8 (concern tbc-r1-f2, alternate form).** R9 (line 80) now includes an explicit scope-of-the-parse clause naming tail-only as the chosen strategy with a clear demarcation against R5's mid-file drift path. AC-8 (line 108) enumerates both branches (preamble-alone-followed-by-separator OR complete-entry-with-trailing-separator). The corner-case coverage gap is closed even though the PRD picked tail-only over the Round-2-stated full-file-read framing.

### Revisions accepted in Round 2 but unapplied in the PRD

1. **R8 + AC-7 (concern surgical-r1-f1) — UNAPPLIED.** Round-2 stated that R8 would be rewritten to refuse on file-missing and point at `specflow:setup`, eliminating the cross-skill duplication of preamble text. The PRD body still carries the original create-on-missing shape (line 76) and AC-7 still verifies the create path (line 106). Round-3 sharpening from surgical-reviewer spells out the concrete edits required (R8 + AC-7 rewrites, plus a downstream Goals-bullet edit and an AC-10 Stage-1-clause edit dropping the file-missing branch). This is the load-bearing iteration item to pick up before `specflow:task` runs.

### Findings rejected after Round 3

One finding had a defended outcome:
- **simplicity-r1-f1 (R7's mtime soft prompt)** — interview Round 4 (line 50 of the interview) records the user explicitly editing the AI-recommended check shape down to mtime-on-`task-history.json` AND naming the 60-minute window as the user's specification with the rationale "doesn't depend on harness behaviour we don't control". That is the documented consumer-ask the Simplicity sub-clause requires; the 60-minute number is interview-grounded, not a fabricated constant. The Round-3 reviewer accepted the defence — the principle bites where speculation isn't anchored, releases where it is.

### Findings escalated to human

None. All six findings either converged within three rounds (five) or surfaced a concrete sharpened claim for the next iteration to apply (one). The sharpened claim is procedural — the agreed Round-2 fix did not land in the PRD body — and the manifest documents the exact edits required, so no human disambiguation is needed; the next pass through `specflow:plan` (or a manual PRD edit) lands the surgical fix.

The PRD is fit to proceed to `specflow:task` once the surgical-r1-f1 sharpened edits are applied. The two `block`-grade gaps (tbc-r1-f1 reading-order ambiguity, goal-r1-f1 orphan AC) are closed. The remaining iteration is the cross-skill duplication fix on R8 + AC-7.

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 2 reviewers and humans reading this manifest as part of the Phase-3-skill review history:

- **One sharpen on an unapplied accepted revision is the most interesting outcome here.** The AI's Round-2 disposition for surgical-r1-f1 was correct in principle (option (a) eliminates the duplication cleanly) but the corresponding PRD edit did not land. Reviewers reading round-2 disposition text alone would have accepted; reviewers reading the PRD body caught the discrepancy. This is a healthy guardrail — Round 3 verifies that revisions claimed in Round 2 are actually present in the artefact, not just in the response JSON.
- **Each reviewer fired its lens distinctly.** Simplicity flagged speculative configurability and was push-back-defended via interview citation; Surgical flagged cross-skill duplication; Think-Before-Coding flagged an unstated reading-order assumption (block) plus an unstated parse-strategy tradeoff (concern); Goal-Driven flagged an orphan AC for the user-surface chat-line contract (block); Devil's Advocate flagged R7's edge cases. No two reviewers flagged the same finding.
- **Watch-for: empty-file-with-recent-mtime sub-case for `task-history.json`.** The Round-3 acceptance on da-r1-f1 noted that the file-missing path is named explicitly but the empty-file-with-recent-mtime sub-case (file exists, zero entries, recent mtime — e.g. setup just touched it) is only implicitly covered by the goal-anchored "never blocks" principle. A future implementer should confirm explicit handling so the soft prompt does not fire on every fresh project's first decision-log entry.
- **Watch-for: surgical-r1-f1 unapplied revision.** Before tasks generate against this PRD, the R8 + AC-7 cross-skill duplication fix must land. The Round-3 sharpened-claim text is the canonical edit specification.
- **The push-back was defensible and accepted.** Simplicity's defence on R7 cited a concrete interview line (Round 4 line 50) showing the user named the mtime-shape and 60-minute window during grilling; that is the consumer-ask the Simplicity sub-clause requires for v1 configurability. Accepting the principle's bite would have meant ignoring documented user intent.
