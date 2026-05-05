---
feature: 001-design-skill
status: draft
created: 2026-05-06
requires:
  - ./001-design-skill-prd.md
  - ./001-design-skill-interview.md
  - ./debate-log/prd-gate2/manifest.md
produces:
  - ./001-design-skill-test.md
  - ./design/{NNN-slug}-current.html
  - ./design/{NNN-slug}-proposed.html
  - ./design/{NNN-slug}-iteration-log.md
prd: ./001-design-skill-prd.md
interview: ./001-design-skill-interview.md
gate3: ./debate-log/tasks-gate3/manifest.md
---

# Tasks — specflow:design (codebase-truth HTML/CSS mockups)

## Coverage matrix

| PRD requirement | Tasks satisfying it |
|---|---|
| R1 — Per-feature invocation; outputs in `features/NNN-{slug}/design/` | T1 |
| R2 — Three output files (current.html, proposed.html, iteration-log.md) | T2, T13 |
| R3 — Required input PRD; abort if missing | T3 |
| R4 — Tone/intent prompt up front; logged | T4 |
| R5 — Strict abort-and-ask on extraction failure | T5 |
| R6 — `--default-when-missing` opt-in flag with flagged invented values | T6 |
| R7 — Playwright loop; 5% threshold (configurable); 10-iteration cap | T7 |
| R8 — Non-convergence handling (accept/adjust/abort) | T8 |
| R9 — Mandatory comment block with file:line citations | T9 |
| R10 — INVENTED VALUES sub-block at top of comment block | T10 |
| R11 — Conditional Codex review after convergence or accepted drift | T11 |
| R12 — Self-contained HTML output (inline style, no JS) | T12 |

Forward coverage: 12/12 PRD requirements covered. Reverse traceability: all 13 tasks anchor to ≥1 requirement (T13 supports R2's iteration-log-quality rule, anchored to AC-11).

## Tasks

### T1 — Wire feature-id argument and output folder

- **Anchor:** PRD R1 — *per-feature invocation; outputs land in `features/NNN-{slug}/design/`.*
- **Scope:** `skills/design/SKILL.md` (Phase A entry); argument parser for `{NNN}` and `{NNN-slug}` forms; output folder creation at `features/NNN-{slug}/design/`.
- **Acceptance:** Invoking `specflow:design 001-design-skill` (with the PRD present) creates `features/001-design-skill/design/` if absent and resolves all three output paths under it. Invoking `specflow:design` with no argument prompts for a feature ID and refuses to proceed without one. (Verifies AC-1 in part.)
- **Depends on:** none.
- **Notes:** Use the resume logic from `skills/task/SKILL.md` Phase A as the pattern for argument resolution.

### T2 — Emit the three output files at the correct paths

- **Anchor:** PRD R2 — *three files per invocation: current.html, proposed.html, iteration-log.md.*
- **Scope:** `skills/design/SKILL.md` (output stage); file writers for `{NNN-slug}-current.html`, `{NNN-slug}-proposed.html`, `{NNN-slug}-iteration-log.md`.
- **Acceptance:** A successful run produces exactly three files at `features/{NNN-slug}/design/{NNN-slug}-{current.html|proposed.html|iteration-log.md}`. Missing any file is a failed run. (Verifies AC-1.)
- **Depends on:** T1.
- **Notes:** None.

### T3 — Abort with a clear message when the PRD is missing

- **Anchor:** PRD R3 — *requires `features/NNN-{slug}/{NNN-slug}-prd.md` to exist.*
- **Scope:** `skills/design/SKILL.md` Phase A pre-flight; PRD-existence check; abort message string.
- **Acceptance:** Invoking `specflow:design {NNN-slug}` when the PRD file does not exist exits non-zero with the literal sentinel *"PRD not found at `features/{NNN-slug}/{NNN-slug}-prd.md`. Run `specflow:prd {NNN-slug}` first."* and writes zero files to the design folder. (Verifies AC-2.)
- **Depends on:** T1.
- **Notes:** Match the abort-message style used by `skills/task/SKILL.md` Phase A.

### T4 — Capture tone/intent prompt as the first iteration-log line

- **Anchor:** PRD R4 — *one tone/intent question up front; captured in iteration log.*
- **Scope:** `skills/design/SKILL.md` (Phase B prompt); iteration-log writer first-line contract.
- **Acceptance:** After a successful run, line 1 of `{NNN-slug}-iteration-log.md` is `Tone/intent: {user response}` exactly, with no preceding blank lines or headers. The user's response is also passed forward to T11's Codex invocation. (Verifies AC-3.)
- **Depends on:** T2.
- **Notes:** The exact prompt wording is fixed per R4: *"What conversation will this mockup support? (formal review, exploratory discussion, sketch for a meeting, etc.)"*.

### T5 — Strict abort-and-ask on unresolved component/token references

- **Anchor:** PRD R5 — *strict abort-and-ask by default; never falls back to invented defaults.*
- **Scope:** `skills/design/SKILL.md` extraction stage; component/theme/token resolver; abort-and-ask flow.
- **Acceptance:** Given a feature whose PRD references a component name that returns zero source matches and `--default-when-missing` was not passed, the skill stops, prints a question naming the unresolved reference, and writes zero output files until the user supplies a source path or re-invokes with `--default-when-missing`. The iteration log records the unresolved reference if the user provides a path on prompt. (Verifies AC-4.)
- **Depends on:** T1.
- **Notes:** Surfaces the "abort is cheap, guess is expensive" principle (interview Round 3).

### T6 — Implement `--default-when-missing` flag with per-value invention flags

- **Anchor:** PRD R6 — *opt-in flag for invented values, each flagged in the comment block.*
- **Scope:** `skills/design/SKILL.md` argument parser; invention path; comment-block writer (the `/* INVENTED: ... */` per-value tag).
- **Acceptance:** Running `specflow:design {NNN-slug} --default-when-missing` on a feature with at least one unresolved reference produces an HTML file in which every invented value is preceded by a `/* INVENTED: {description} */` comment naming what was missing. Without the flag, the same feature run aborts per T5. (Verifies AC-5 in part with T10.)
- **Depends on:** T5.
- **Notes:** Per-value flag tagging is separate from the sub-block placement, which is T10's surface.

### T7 — Playwright loop with configurable threshold and 10-iteration cap

- **Anchor:** PRD R7 — *5% per-pixel diff (configurable); 10-iteration cap.*
- **Scope:** `skills/design/SKILL.md` Phase C iteration loop; Playwright screenshot capture at agreed viewport; per-pixel diff computation; threshold read from `admin/config.json.design.diffThreshold` with default `0.05`; iteration counter capped at 10.
- **Acceptance:** On a feature with a working dev server, the loop terminates either when the per-pixel diff is `< threshold` or when the iteration counter reaches 10. The iteration log records each iteration's diff value as a number and the reason for termination as one of the literals `converged` or `cap-reached`. Threshold override via `admin/config.json.design.diffThreshold = 0.03` is honoured. (Verifies AC-6.)
- **Depends on:** T2, T9.
- **Notes:** Playwright is a hard requirement enforced by `specflow:setup` per the interview's pre-grilling notes; fail loud if absent rather than degrading. If `admin/config.json.design.diffThreshold` is absent, the skill uses the literal default `0.05` without writing to `admin/config.json`. Schema addition is deferred to consumer-project setup, not this skill.

### T8 — Non-convergence handler with accept/adjust/abort prompt

- **Anchor:** PRD R8 — *on cap-reached, surface diff + offer accept/adjust/abort; log the choice.*
- **Scope:** `skills/design/SKILL.md` post-loop branch when `cap-reached`; diff visualisation surfacer; three-option prompt; iteration-log writer.
- **Acceptance:** When the loop terminates with `cap-reached`, the skill presents the remaining diff (visual + numerical) and waits on a prompt that accepts exactly one of `accept`, `adjust`, `abort`. The user's choice appears in the iteration log on the line `Cap-reached resolution: {choice}`. Any other input re-prompts; the skill does not auto-default. Before the prompt fires, the skill writes the diff visualisation to a path recorded on the iteration-log line `Cap-reached diff: {path}`. The path resolves to a non-empty PNG file; a run with an empty path or unwritten file is a failed run. (Verifies AC-7.)
- **Depends on:** T7.
- **Notes:** `accept` proceeds to T11 (Codex review fires after accepted drift); `abort` skips T11 (Codex does NOT review after abort, per R11).

### T9 — Comment block with file:line citations for every mandatory field

- **Anchor:** PRD R9 — *every colour, typography token, spacing value, and component shape cited with `source: {file}:{line}`.*
- **Scope:** `skills/design/SKILL.md` extraction stage; comment-block writer; mandatory-field list (colour, font-family, font-size, font-weight, line-height, spacing, border-radius, shadow); citation formatter.
- **Acceptance:** For both `current.html` and `proposed.html`, the comment block at the top contains an entry for every colour, typography token (family/size/weight/line-height), spacing value, and component shape (border-radius, shadow) used anywhere in the file's inline `<style>`. Each entry has the literal substring `source: ` followed by a `{file}:{line}` citation that resolves to a real line in the consumer codebase. A field that appears in the `<style>` but not in the comment block is a failed run. (Verifies AC-8.)
- **Depends on:** T2.
- **Notes:** Nice-to-have entries (viewports, asset references) are permitted but not enforced.

### T10 — INVENTED VALUES sub-block placed before extracted values

- **Anchor:** PRD R10 — *invented values appear in a distinct sub-block at the top of the comment block, before the extracted values.*
- **Scope:** `skills/design/SKILL.md` comment-block writer (ordering rule); sub-block markers `/* === INVENTED VALUES === */` and `/* === EXTRACTED VALUES === */`.
- **Acceptance:** When `--default-when-missing` produced ≥1 invented value, the comment block opens with `/* === INVENTED VALUES === */`, lists each invented value with a one-line description naming what was missing, then opens `/* === EXTRACTED VALUES === */` and lists the cited values. The byte offset of the INVENTED VALUES marker is strictly less than the byte offset of the EXTRACTED VALUES marker. When zero invented values exist, the INVENTED VALUES sub-block is omitted entirely. (Verifies AC-5 alongside T6.)
- **Depends on:** T6, T9.
- **Notes:** "Invented values are loud, not buried" — interview Round 5.

### T11 — Conditional Codex review after convergence or accepted drift

- **Anchor:** PRD R11 — *Codex reviews proposed.html against current.html (baseline) plus tone/intent; fires after convergence OR accepted drift; skipped silently with log note when absent; never after abort.*
- **Scope:** `skills/design/SKILL.md` post-loop Codex stage; environment check on `admin/environment.json.cli.codex.available`; Codex invocation with `proposed.html`, `current.html`, and the captured tone/intent (T4); silent-skip path with iteration-log note.
- **Acceptance:** With `cli.codex.available: true` and the loop ending in `converged` or in `cap-reached` followed by user choice `accept`, the iteration log contains a `## Codex review` section with at least one finding line OR the literal sentinel `no semantic concerns found`. With `cli.codex.available: false`, the iteration log contains the literal one-line note `Codex not detected — semantic review skipped.` and no Codex section. When the user chose `abort` in T8, the iteration log contains neither the `## Codex review` section nor the `Codex not detected — semantic review skipped.` note, regardless of whether `admin/environment.json.cli.codex.available` is `true` or `false`. Abort short-circuits both Codex branches. (Verifies AC-9.)
- **Depends on:** T7 always; T8 only when T7 ends in `cap-reached`.
- **Notes:** Codex reviews `proposed.html` against `current.html` as the live-state baseline — it does NOT review `current.html` standalone. Per Gate 2 finding `da-r1-f1`. On the `converged` path, T11 reads the loop's terminal state directly from the iteration log and skips T8 entirely. On `cap-reached`, T11 reads T8's `Cap-reached resolution` line and fires only when the value is `accept`.

### T12 — Self-contained HTML output (inline style, no JS, no external CSS)

- **Anchor:** PRD R12 — *self-contained HTML, inline style, no JS, opens directly in a browser.*
- **Scope:** `skills/design/SKILL.md` HTML writer; output-validation step; constraint check on `<link rel="stylesheet">`, `<script>`, and external resource references (images excepted as base64 or relative-linked).
- **Acceptance:** Both `current.html` and `proposed.html` contain zero `<link rel="stylesheet">` tags, zero `<script>` tags, and zero external resource references except images (base64 inline or relative-linked). Opening either file via `file://` in a headless browser produces zero console errors. (Verifies AC-10.)
- **Depends on:** T2.
- **Notes:** Same convention as `specflow:render` per the codebase context bullet.

### T13 — Iteration-log quality: every entry has a non-empty Why field

- **Anchor:** PRD R2 (iteration-log requirement) reinforced by AC-11 — *empty Why fields are a failed run; the skill must not declare done.*
- **Scope:** `skills/design/SKILL.md` iteration-log writer; per-iteration entry schema with mandatory `Why:` line; pre-completion validator that scans the log.
- **Acceptance:** Every iteration entry written into `{NNN-slug}-iteration-log.md` has a non-empty `Why:` field (≥1 non-whitespace character after the colon). The pre-completion validator returns non-zero if any entry has an empty Why field; the skill refuses to declare done in that case and surfaces the offending iteration index to the user. (Verifies AC-11.)
- **Depends on:** T7, T8.
- **Notes:** Per PRD Appendix C3.1 quality rule. Closed Gate 2 finding `goal-r1-f1`.

## Open questions inherited from PRD

- **Q:** Should the iteration log be HTML-rendered too? — affects: T2, T13. Currently markdown only; defer until skill ships.
- **Q:** What happens when the feature has no working dev server? PRD Appendix C3 mentions recorded-screenshot fallback but the UX is loose. — affects: T7. Implementer picks the cleanest UX.
- **Q:** Mobile viewport defaults (iOS / Android / generic) per PRD Appendix C1. — affects: T7. Reasonable defaults proposed in PRD Open questions; implementer to confirm.

## See also

- PRD: [`./001-design-skill-prd.md`](./001-design-skill-prd.md)
- Interview: [`./001-design-skill-interview.md`](./001-design-skill-interview.md)
- Gate 2 manifest: [`./debate-log/prd-gate2/manifest.md`](./debate-log/prd-gate2/manifest.md)
- Gate 3 manifest: [`./debate-log/tasks-gate3/manifest.md`](./debate-log/tasks-gate3/manifest.md)
- Tests: `./001-design-skill-test.md` (generated by `specflow:test`)
