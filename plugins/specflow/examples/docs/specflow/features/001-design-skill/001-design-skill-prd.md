---
templateVersion: v2.3
feature: 001-design-skill
status: draft
created: 2026-05-05
interview: ./001-design-skill-interview.md
---

# specflow:design — codebase-truth HTML/CSS mockups

## Vision

Consumer projects of specflow can produce HTML/CSS mockups of *current* and *proposed* feature states that visually match their real app, without the AI inventing colours, spacing, components, or typography. The mockup is an alignment artefact for cross-functional discussion — a starting point for the conversation, not the final design — and every value used in it is auditable back to the source code that produced it. Designers and frontend engineers in consumer projects can trust that what the AI rendered reflects what the codebase actually does, and stakeholders reviewing the mockup as a discussion artefact see a faithful representation of today's app alongside the proposed direction.

## Problem

Past attempts at AI-generated design mockups have drifted because the AI invented values — colours close to but not matching the brand palette, spacing approximated to round numbers instead of the design tokens in use, components reshaped to a generic Material/Tailwind aesthetic instead of the project's actual look. This drift poisoned the conversations the mockups were meant to support: designers spotted the inaccuracies and lost trust in the artefact; engineers caught the invented values and rejected the mockup as a basis for discussion; stakeholders reviewing the proposed direction couldn't tell what was a real change versus what was just AI hallucination dressed up as a redesign.

The failure mode is structural: when the AI doesn't have a strict source-of-truth contract, it falls back to its training-data priors, which produce generic-looking output that approximates rather than matches. A skill that produces mockups for alignment conversations needs to enforce extraction over invention as a non-negotiable rule, not a best-effort guideline.

## Goals

- Mockups visually match the live app for the *current* state, with every visual value (colour, typography, spacing, component shape) extractable to a source file:line citation in the mockup itself.
- Mockups for the *proposed* state draw from the same design system unless the user explicitly asks for a departure, and any departure is flagged distinctly so reviewers can spot what's a real visual change versus what's an artefact of AI choice.
- The skill iterates visually against the live app via a Playwright loop until the diff is below an agreed threshold, with a hard cap on iterations to prevent runaway loops.
- When extraction fails (component source not findable, design token absent), the skill aborts and asks rather than guessing — invented values are an explicit opt-in, never a silent fallback.
- The skill shares the same *self-contained HTML* convention as `specflow:render` — inline CSS, no JS dependencies, opens directly in a browser, shares as a `file://` link.

## Non-goals

- **Producing the *final* visual design.** Proposed mockups are a discussion starting point. The deliverable that ships is whatever the design + engineering teams iterate to *after* the alignment conversation.
- **Generating designs without a codebase to extract from.** Greenfield ideation is a different problem with different tradeoffs; a future `specflow:greenfield-design` skill could surface this if needed.
- **Replacing Figma or other dedicated design tools.** specflow is a workflow plugin, not a design tool. Mockups produced here are alignment artefacts; if the team needs a design system, dedicated component playground, or interactive prototyping, those live in their existing tools.
- **Maintaining design-system parity automatically when the codebase changes.** The mockup is a snapshot at the time of generation. Staleness is a `specflow:scope-change` concern (re-generate the mockup when the underlying design system shifts).
- **Interactive prototyping (animations, gesture overlays, state transitions).** Static HTML/CSS only. Use a dedicated tool for interaction.
- **Accessibility audit of the generated mockup.** Interesting future addition (e.g. axe-core check post-render) but not in v1 — doesn't serve the goal's primary outcome of codebase-truth visual fidelity.

## Users

- **Power User (per `admin/profiles.json`)** — frontend engineers using specflow on a daily basis. They invoke `specflow:design` after PRD synthesis to produce a visual artefact for the upcoming feature conversation. They need the mockup to match real source so they can verify any of its values without leaving the codebase.
- **Developer / API Consumer (per `admin/profiles.json`)** — designers reviewing the mockup. They need to trust that the *current* state is faithful (so they can compare it to the *proposed* state without doubting the baseline). They do NOT need to read the source themselves; they need confidence that the comment block's citations are real and any reviewer could verify them.
- **Secondary: any role reviewing the mockup as a discussion artefact** — product managers, stakeholders, ops. They consume the mockup in a browser. They benefit from the codebase-truth principle indirectly: the conversation about the proposed direction stays grounded in what the app actually does.

## Requirements

- **R1.** Per-feature invocation. `specflow:design {NNN-slug}` produces mockups for one feature only; output lives in `features/NNN-{slug}/design/`.
  - Trace: interview Round 1 — *per-feature invocation only; outputs land in `features/NNN-{slug}/design/`*.
  - Serves goal: Outcome (mockups for "a feature" — singular), Out-of-scope (no project-wide library).

- **R2.** Output files. Skill produces three files per invocation: `NNN-{slug}-current.html`, `NNN-{slug}-proposed.html`, and `NNN-{slug}-iteration-log.md`.
  - Trace: interview Round 1.
  - Serves goal: Success-looks-like (two HTML files for current + proposed).

- **R3.** Required input: synthesised PRD. Skill aborts with a clear message if `features/NNN-{slug}/NNN-{slug}-prd.md` doesn't exist.
  - Trace: interview Round 2 — *requires `features/NNN-{slug}/prd.md` to exist*.
  - Serves goal: Outcome (mockups grounded in what's being designed; no PRD = no anchor).

- **R4.** Tone/intent prompt at the start. Skill asks one question up front: *"What conversation will this mockup support? (formal review, exploratory discussion, sketch for a meeting, etc.)"* — captured in the iteration log and used by Codex review (R9) to evaluate semantic alignment.
  - Trace: interview Round 2 — *asks one tone/intent question at the start*.
  - Serves goal: Driving value (avoids producing mockup with wrong framing for the discussion it'll support).

- **R5.** Codebase-truth: strict abort-and-ask by default. When the skill cannot find the source for a referenced component, theme file, or design token, it stops and asks the user where to look. Never falls back to invented defaults.
  - Trace: interview Round 3 — *strict abort-and-ask by default; never falls back to a sensible default*.
  - Serves goal: Driving value (the entire reason this skill exists is to remove the AI-hallucinated-the-design failure mode).

- **R6.** `--default-when-missing` opt-in flag. When the user passes this flag, the skill is allowed to invent values where source extraction fails, but each invented value is flagged distinctly in the comment block (`/* INVENTED: {description} */`) and aggregated under a separate `INVENTED VALUES` sub-block at the top of the comment block, before the extracted values.
  - Trace: interview Round 3 (edited) — *`--default-when-missing` flag opts into invented-value mode with each invented value explicitly flagged*.
  - Serves goal: Outcome (still serves the rapid-prototyping case while preserving auditability — invented values are loud, not quiet).

- **R7.** Playwright iteration loop, 5% diff threshold, 10-iteration cap. Skill iterates the mockup HTML against a screenshot of the live app at the agreed viewport. Threshold is 5% per-pixel difference, configurable via `admin/config.json.design.diffThreshold`. Loop caps at 10 iterations. A single project-wide threshold is the v1 choice; per-region or per-component overrides were considered as alternatives but deferred — the configurable global is the simplest consumer interface that preserves auditability. If consumers report drift-blindness on specific UI surfaces, per-region thresholds become a v2 candidate.
  - Trace: interview Round 4 — *5% per-pixel diff threshold (configurable); 10-iteration cap*.
  - Serves goal: Success-looks-like (visual diff below threshold).

- **R8.** Non-convergence handling. If the loop hits the 10-iteration cap without converging, the skill surfaces the remaining diff visualisation to the user with three options: accept the drift, adjust the threshold, or abort. The user's choice is logged in the iteration log.
  - Trace: interview Round 4 — *on non-convergence, surface diff + remaining drift to user for accept/adjust/abort*.
  - Serves goal: Driving value (preserves human judgement on edge cases; the loop doesn't fail silently).

- **R9.** Mandatory comment block. Every generated HTML file has a comment block at the top listing every value used: colours, typography (family/size/weight/line-height), spacing, component shape (border-radius, shadow). Each with `source: {file}:{line}` citation. Nice-to-have entries (viewports, asset references) may also appear but are not required.
  - Trace: interview Round 5 — *mandatory fields = colours, typography, spacing, component shape, each with source citation*.
  - Serves goal: Success-looks-like ("every value used in the mockup is auditable in a comment block at the top").

- **R10.** INVENTED VALUES sub-block. When invented values exist (per R6), they appear in a distinct `/* === INVENTED VALUES === */` sub-block at the top of the comment block, *before* the extracted values, so reviewers see them first.
  - Trace: interview Round 5 (edited) — *INVENTED values get a distinct sub-block at the top of the comment block before the extracted values*.
  - Serves goal: Driving value (invented values are loud, not buried).

- **R11.** Codex adversarial review (optional, conditional). After the Playwright loop converges OR after the user explicitly accepts non-convergence drift, the skill invokes Codex (when available per `admin/environment.json.cli.codex.available`) as a semantic reviewer. Codex reviews `proposed.html` against `current.html` (as the live-state baseline) and the user's tone/intent (R4); flags semantic gaps the visual diff missed (lost affordances, contrast regressions, intent drift). Codex does NOT review `current.html` standalone — it's the baseline, not the candidate. When Codex is absent, the skill skips silently with a one-line note in the iteration log. The silent-skip-with-log choice was preferred over a louder warning to avoid noise on low-stakes mockups; users who need semantic-review enforcement set a project-level requirement on Codex via `admin/environment.json.cli.codex.required` (a future config flag if demand surfaces). Codex review is NOT invoked after abort.
  - Trace: interview Round 6 — *Codex review fires after convergence OR after user-accepted drift (NOT after abort); skipped silently with iteration-log note when Codex absent*.
  - Serves goal: Driving value (semantic review catches what visual diff misses — e.g. lost affordances).

- **R12.** Self-contained HTML output. Each generated HTML file uses inline `<style>`, no JS, no external assets. Opens directly in a browser. Shareable as a `file://` link or static-hosted artefact. Same convention as `specflow:render`.
  - Trace: codebase context (pre-grilling) bullet — *adjacent skills: specflow:render already produces self-contained HTML; same convention applies here*. No grilling round was needed because the convention was inherited from `specflow:render` and pre-confirmed.
  - Serves goal: Success-looks-like (HTML files reviewers can open and inspect without infrastructure).

## Acceptance criteria

- **AC-1.** Running `specflow:design {NNN-slug}` on a feature with an existing `NNN-{slug}-prd.md` produces three files in `features/NNN-{slug}/design/`: `NNN-{slug}-current.html`, `NNN-{slug}-proposed.html`, `NNN-{slug}-iteration-log.md`. (Verifies R1, R2.)

- **AC-2.** Running `specflow:design {NNN-slug}` when `NNN-{slug}-prd.md` doesn't exist aborts with a clear message instructing the user to run `specflow:prd` first. No partial output is written. (Verifies R3.)

- **AC-3.** The iteration log records the user's response to the tone/intent prompt as the first line of the log. (Verifies R4.)

- **AC-4.** When the skill encounters an unresolvable component reference (e.g. a component name that returns no source matches), the skill stops, asks the user, and does NOT proceed until the user provides a source path or passes `--default-when-missing`. (Verifies R5.)

- **AC-5.** When invoked with `--default-when-missing`, every invented value appears in the generated HTML's comment block under a `/* === INVENTED VALUES === */` sub-block, listed before the extracted values. Each invented value has a one-line description naming what was missing. (Verifies R6, R10.)

- **AC-6.** Running on a feature with a working dev server: the Playwright loop iterates until the per-pixel diff is below 5% (or the configured threshold) OR hits the 10-iteration cap. Either outcome is recorded in the iteration log. (Verifies R7.)

- **AC-7.** When the loop hits the 10-iteration cap, the skill presents the remaining diff to the user (visual + numerical) and offers three options: accept, adjust threshold, abort. The user's choice is logged. (Verifies R8.)

- **AC-8.** Every generated HTML file has a comment block at the top listing every colour, typography token (family/size/weight/line-height), spacing value, and component shape used in the mockup, each with `source: {file}:{line}` citation. (Verifies R9.)

- **AC-9.** With Codex available (per `admin/environment.json`), after convergence or accepted drift, the iteration log includes a Codex review section with at least one finding (or "no semantic concerns found"). With Codex absent, the iteration log includes a one-line note: *"Codex not detected — semantic review skipped."* (Verifies R11.)

- **AC-10.** Generated HTML files contain only inline `<style>` (no `<link rel="stylesheet">`), no `<script>` tags, no external resource references except images (which may be inlined as base64 or relative-linked). The file opens cleanly in a browser without console errors. (Verifies R12.)

- **AC-11.** Every iteration entry in `NNN-{slug}-iteration-log.md` has a non-empty *Why* field (per PRD Appendix C3.1). An entry with an empty *Why* field is a failed run; the skill must not declare done. (Verifies R2's iteration-log requirement, against the C3.1 quality rule.)

## Open questions

- **Should the iteration log be HTML-rendered too?** Currently it's markdown only (matches the interview-file precedent). For visual review in a browser, the HTML mockups themselves are sufficient. Defer until the design skill has shipped and we see whether teams want a "review packet" format.
- **What happens when the user wants a mockup for a feature that doesn't have a working dev server?** PRD Appendix C3 mentions "use a recorded screenshot when the dev server isn't available". This needs concrete UX — does the skill ask for a screenshot path? Auto-detect screenshot files in `features/NNN-{slug}/assets/`? Defer to implementation; the spec is loose enough that the implementer can pick the cleanest UX.
- **Mobile viewport defaults.** PRD Appendix C1 mentions phone-frame wrapping for mobile output (iOS / Android / generic) defaulting from `admin/config.json`. The default values for each are not specified. Reasonable defaults: iOS = iPhone 15 Pro (393×852 logical), Android = Pixel 8 (412×915 logical), generic = no frame, just the viewport. Implementer can confirm or override.

## See also

- Interview: [`./001-design-skill-interview.md`](./001-design-skill-interview.md)
- PRD spec: [`../../../../docs/PRD.md`](../../../../docs/PRD.md) Appendix C
- Adjacent skill: [`../002-render-skill/`](../002-render-skill/) (hypothetical example) — same self-contained HTML convention.
- Tasks: `001-design-skill-tasks.md` (generated by `specflow:task`)
- Tests: `001-design-skill-test.md` (generated by `specflow:test`)
