# Debate manifest — Gate 3: tasks vs PRD review

**Feature:** 001-design-skill
**Artefact under review:** `001-design-skill-tasks.md`
**Reviewers:** simplicity-reviewer, surgical-reviewer, think-before-coding-reviewer, goal-driven-reviewer, devils-advocate
**Codex:** not configured for this run (skipped)
**Closed:** 2026-05-06 11:20

This is the worked example of Gate 3 in operation. Five reviewers fired in parallel (forked sub-agents), the AI responded in Round 2, reviewers sharpened-or-accepted in Round 3, and this manifest is the Orchestrator's closing entry. Use this as the calibration anchor for what a healthy Gate 3 looks like — Goal-Driven Reviewer is the heaviest hitter (coverage matrix is its primary lens), and the AI defends a coverage call without losing the rest of the round.

---

## Round 1 — Findings

| Reviewer | Findings (severity) |
|---|---|
| simplicity-reviewer | 1 (concern) |
| surgical-reviewer | 1 (concern) |
| think-before-coding-reviewer | 1 (concern) |
| goal-driven-reviewer | 2 (concern, concern) |
| devils-advocate | 1 (concern) |
| **Total** | **6 findings** |

Detail:
- **goal-r1-f1** — *concern* — T7's threshold-override clause uses 'honoured' rather than a binary check on the iteration-log diff value. (See `findings/round-1/goal-driven-reviewer.json`.)
- **goal-r1-f2** — *concern* — T8 only verifies the three-option prompt half of AC-7; the diff-visualisation half has no binary check. (Same file.)
- **simplicity-r1-f1** — *concern* — T6 and T10 split a single shippable surface (the comment-block writer) across two tasks both verified by AC-5. (See `findings/round-1/simplicity-reviewer.json`.)
- **surgical-r1-f1** — *concern* — T7 reads `admin/config.json` (project-wide surface) without declaring whether the schema addition is in scope or assumed. (See `findings/round-1/surgical-reviewer.json`.)
- **tbc-r1-f1** — *concern* — T11's Depends-on `T7, T8` is ambiguous between unconditional and conditional ordering. (See `findings/round-1/think-before-coding-reviewer.json`.)
- **da-r1-f1** — *concern* — T11 acceptance abort clause is silent on Codex-availability; the (available + aborted) cross-product case is implementer-dependent. (See `findings/round-1/devils-advocate.json`.)

## Round 2 — AI responses

Detail in `findings/round-2/responses.json`. Summary by finding ID:

- goal-r1-f1 → **push_back** (cited Phase 1 scope item 15 — verb 'honoured' summarises a contract whose binary check is the diff-value-as-number rule already in T7 acceptance line 2)
- goal-r1-f2 → **accept** (added the `Cap-reached diff: {path}` log line + non-empty-PNG check to T8)
- simplicity-r1-f1 → **push_back** (cited Gate 2 simplicity-r1-f1 — R6 and R10 are separate single-concern requirements; the 1:1 R→T trace argument applies at the task level)
- surgical-r1-f1 → **accept** (applied option (b): added Notes line to T7 documenting default-to-literal-0.05 with no write to `admin/config.json`)
- tbc-r1-f1 → **accept** (sharpened T11 Depends-on to 'T7 always; T8 only when T7 ends in `cap-reached`' + added Notes line documenting conditional ordering)
- da-r1-f1 → **accept** (sharpened T11 abort clause to 'regardless of whether `admin/environment.json.cli.codex.available` is `true` or `false`')

## Round 3 — Reviewers sharpen or accept

All findings converged in Round 3:

- goal-r1-f1 → **accept** (AI's defence held; contract-vs-mechanism separation cited from Phase 1 scope item 15)
- goal-r1-f2 → **accept** (revision applied; AC-7 coverage gap closed)
- simplicity-r1-f1 → **accept** (AI's defence held; Gate 2 simplicity-r1-f1 outcome cited; trace-integrity argument holds at task level)
- surgical-r1-f1 → **accept** (revision applied; cross-feature scope drift closed)
- tbc-r1-f1 → **accept** (revision applied; conditional ordering documented)
- da-r1-f1 → **accept** (revision applied; (Codex available + aborted) cross-product case pinned to binary check)

No sharpening occurred. No `ai-revision.md` needed.

---

## Closing decision

**Gate 3 status: passed**

4 of 6 findings were accepted by the AI and revisions applied to `001-design-skill-tasks.md`:
- T8 gained a `Cap-reached diff: {path}` clause closing the second binary half of AC-7 (visual diff written to a non-empty PNG before the prompt fires).
- T7 gained a Notes line declaring that an absent `admin/config.json.design.diffThreshold` defaults to literal `0.05` without writing back; schema addition is consumer-project setup, not this skill.
- T11 Depends-on was sharpened to 'T7 always; T8 only when T7 ends in `cap-reached`' and a Notes line documents the conditional-ordering pattern (converged → reads iteration-log terminal state; cap-reached → reads T8's `Cap-reached resolution`).
- T11 acceptance abort clause was sharpened to pin the (Codex available + user aborted) cross-product case: regardless of `cli.codex.available`, abort emits neither the `## Codex review` section nor the `Codex not detected — semantic review skipped.` note.

2 findings were rejected after Round 3 with the reviewer accepting the AI's defence:
- **goal-r1-f1** (T7 threshold-override 'honoured' wording) — the verb 'honoured' summarises a contract whose binary check (iteration-log diff value as a number) is already in T7's acceptance. Verbosity is not strength; binary is. Sharpened wording would have restated the same check with more words.
- **simplicity-r1-f1** (T6/T10 merge) — Gate 2's simplicity-r1-f1 already established R6 and R10 as separate single-concern requirements. The same 1:1 R→T trace argument carries to the task level: merging T10 into T6 would create a multi-anchor task and lose the coverage matrix's symmetric mapping. AC-5 verifies both rules either way; the split serves reviewability, not test surface.

No findings escalated to human decision.

The tasks file is fit to proceed to `specflow:test` for verification cadence, or to `specflow:develop` (Phase 2) to begin implementation. No revisions to the PRD or interview were required (no scope-change triggered).

— Orchestrator, 2026-05-06

---

## Calibration notes

For future Gate 3 reviewers and humans reading this as an example:

- **Healthy Gate 3 looks like this** — 4-7 findings spread across reviewers, most with `concern` severity, Goal-Driven Reviewer carrying the heaviest load (coverage matrix is its primary lens per `principles/goal-driven-reviewer.md`'s 'Particularly load-bearing at Gate 3' section), a mix of accept and push-back in Round 2, convergence in Round 3. Zero `block` findings on a well-grilled task list is a good outcome; a sea of `block` would mean Phase B's coverage matrix self-check (per `skills/task/SKILL.md` Phase B.4) wasn't doing its job.
- **Each reviewer fired its lens** — Goal-Driven flagged a non-binary acceptance phrase and a coverage gap on AC-7's second half; Simplicity flagged over-decomposition; Surgical flagged cross-feature scope drift; Think-Before-Coding flagged unstated conditional ordering; Devil's Advocate flagged a cross-product branch the writer missed. None overlapped much.
- **The AI's two pushbacks were defensible and accepted** — both cited prior-round evidence (Gate 2 simplicity-r1-f1's outcome; Phase 1 scope item 15's contract-vs-mechanism separation). Push-back is healthy when grounded in artefacts the reviewers can verify; accepting every finding wholesale would be rubber-stamping in reverse.
- **Goal-Driven was load-bearing here** — 2 of 6 findings, both `concern`. One was a soft-verification flag on T7's override clause (defended); the other was a coverage gap on AC-7 (accepted, closed by adding the diff-visualisation path check to T8). Real first-pass Gate 3 runs typically surface 1-2 Goal-Driven findings per task batch — watch for missing binary checks on multi-half ACs and for unverified `eval:` mechanisms.
- **No `block` findings on this task list** is plausible because the worked example was iterated by hand against the Phase B.4 self-check before this Gate 3 ran. Real first-pass Gate 3 runs typically surface 1-2 `block` findings — a task acceptance with no binary check at all (Goal-Driven auto-blocks), a PRD requirement with no covering task (coverage-matrix gap), an orphan task that doesn't trace to any requirement (Surgical auto-blocks). Watch for those.
- **Cross-gate consistency check** — at least one Gate 3 finding (here `da-r1-f1`) tested whether the Gate 2 sharpening of R11 was carried forward into the task acceptances. This is the Devil's Advocate cross-cutting lens at its best: previous gates fixed a PRD ambiguity, and Gate 3 verifies the tasks didn't reintroduce it. Look for this pattern in real runs.
