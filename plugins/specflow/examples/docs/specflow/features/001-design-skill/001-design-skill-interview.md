# PRD interview — features/001-design-skill

## Original request

> I want specflow to ship a `specflow:design` skill that produces HTML/CSS mockups for a feature, grounded in the live codebase (codebase-truth principle). Past attempts at AI-generated mockups drifted because the AI made up colours/spacing/components instead of extracting them from the real code. The skill should iterate visually against the live app via Playwright until the mockup matches.

## Codebase context (pre-grilling)

- **PRD spec already exists.** `docs/PRD.md` Appendix C documents `specflow:design` in detail (sections C1-C6: skill behaviour, codebase-truth principle, Playwright iteration loop, optional Codex review, scope/tone, open questions). This PRD synthesis must align with the spec.
- **Stub SKILL.md exists.** `skills/design/SKILL.md` is a description-level stub (frontmatter + brief description). The stub already has correct `requires:` / `produces:` for the v2 conventions.
- **Hard requirement: Playwright.** Already enforced by `specflow:setup` (Phase 1.1). The `design` skill can assume Playwright is available — fail loud if it isn't, never degrade.
- **Soft requirement: Codex.** Used as adversarial reviewer at the end of the iteration loop. Detected via `admin/environment.json.cli.codex.available`. Skill skips that step gracefully when absent.
- **Per-feature folder convention.** Mockups live at `features/NNN-{slug}/design/NNN-{slug}-current.html` and `NNN-{slug}-proposed.html`. Iteration log at `NNN-{slug}-iteration-log.md`. Not at the project root — this is a v2 layout decision (PRD §"Reference: target structure").
- **Adjacent skills.** `specflow:render` already produces self-contained HTML (PRD Appendix P) — same convention applies here: inline CSS, no JS dependencies, opens directly in a browser.
- **Decision-log precedent.** No prior decision specifically about design fidelity tooling. The closest precedent is the rules registry's `NO_HARDCODED_VALUES` rule — it codifies "extracted not invented" for code; this skill applies the same principle to mockups.
- **CONTEXT.md note.** This project (specflow itself) does not have a frontend codebase to extract design tokens from. The skill is being built for *consumer projects* of specflow that DO have a frontend. The worked example for this feature should illustrate consumption from a hypothetical React+Tailwind project.
- **No prior PRDs touching adjacent surfaces.** This is the first design-tooling PRD in specflow.

## Goal (confirmed before grilling)

- **Outcome:** consumer projects of specflow can produce HTML/CSS mockups of *current* and *proposed* feature states that visually match their real app, without the AI inventing colours, spacing, components, or typography. The mockup is an alignment artefact for cross-functional discussion, not a final design.
- **Audience:** product designers and frontend engineers in consumer projects (Power User and Developer profiles per `admin/profiles.json`). Secondary: stakeholders reviewing the mockup as a discussion artefact (any role).
- **Success looks like:** running `specflow:design {feature-id}` on a feature with an existing PRD produces two self-contained HTML files (`current.html`, `proposed.html`) whose visual diff against the live app falls below an agreed threshold; every value used in the mockup is auditable in a comment block at the top of the file with its source `file:line` citation.
- **Driving value:** removes the "AI hallucinated the design" failure mode that has historically poisoned mockup-based alignment conversations. Designers can trust the mockup matches the real codebase; engineers can verify the mockup's claims against source.
- **Out of scope at the goal level:** (1) producing the *final* visual design — proposed mockups are a discussion starting point, not the deliverable; (2) generating designs without a codebase to extract from (e.g. greenfield ideation — that's a different skill); (3) replacing Figma or other dedicated design tools — specflow is a workflow plugin, not a design tool; (4) maintaining design-system parity automatically when the codebase changes — the mockup is a snapshot; staleness is a `specflow:scope-change` concern.

User confirmed: 2026-05-05.

## Rounds

### Round 1 — Mode of consumption: invoked per-feature, or per-project library?

- **Q:** When a user invokes `specflow:design`, do they specify a feature (e.g. `specflow:design 003-claim-submission` produces mockups for that feature only), or does the skill maintain a project-wide design library that gets refreshed?
- **AI's recommended answer:** Per-feature only. Each invocation produces `current.html` + `proposed.html` for one feature. — *Reasoning: matches the per-feature folder convention from the v2 layout decision (PRD §"Reference: target structure"); avoids the maintenance burden of a project-wide design library; keeps blast radius small per the Surgical Changes principle. A project-wide library is a different feature with different tradeoffs — defer it.*
- **User's answer:** confirmed.
- **Resolved:** per-feature invocation only; outputs land in `features/NNN-{slug}/design/`. — flows into §Requirements R1, R2.

### Round 2 — Triggers and inputs

- **Q:** What triggers should the skill respond to, and what inputs does it require?
- **AI's recommended answer:** Triggers: `/specflow:design {feature-id}`, *"create a design mockup for X"*. Required inputs: a valid feature folder (`features/NNN-{slug}/`) containing a synthesised `prd.md` (so the skill knows what's being designed). Optional inputs: the user's tone/intent for the conversation the mockup will support (one prompt at the start). — *Reasoning: the PRD anchors what's being designed — without it, the skill is generating mockups for a target it doesn't understand. Asking the tone/intent up front avoids producing a mockup with the wrong framing (formal vs exploratory, polished vs sketch).*
- **User's answer:** confirmed.
- **Resolved:** triggers `/specflow:design {feature-id}` + natural-language equivalent; requires `features/NNN-{slug}/prd.md` to exist; asks one tone/intent question at the start. — flows into §Requirements R1, R3.

### Round 3 — Codebase-truth enforcement: how strict?

- **Q:** The codebase-truth principle says "extract values, don't invent". How strict is this? Does the skill abort if it can't find the source for a referenced component, or does it ask the user, or does it fall back to a sensible default?
- **AI's recommended answer:** Abort and ask. If the skill can't find the source for a referenced component, theme file, or design token, it stops and asks the user where to look — never falls back to a sensible default. Defaults are how mockups drifted in the past. — *Reasoning: the goal's Driving value field explicitly names "removes the AI-hallucinated-the-design failure mode". Fall-back defaults reintroduce that failure mode. Asking is cheap; guessing is expensive.*
- **User's answer:** edit — confirm the abort-and-ask, but allow a `--default-when-missing` flag for explicit opt-in (e.g. when the user is rapid-prototyping and accepts that some values will be invented, with those values flagged distinctly in the comment block).
- **Resolved:** strict abort-and-ask by default; `--default-when-missing` flag opts into invented-value mode with each invented value explicitly flagged in the comment block (e.g. `/* INVENTED: no source found for primary brand colour */`). — flows into §Requirements R4, R5; AC AC-3.

### Round 4 — Playwright iteration loop: convergence criterion

- **Q:** The Playwright loop iterates the mockup against a screenshot of the live app until the diff falls below threshold. What's the threshold, and what happens if convergence isn't reached after N iterations?
- **AI's recommended answer:** Configurable threshold defaulting to 5% per-pixel difference (industry-standard for visual regression tools). Cap at 10 iterations — if the mockup still hasn't converged, surface the remaining differences to the user with the diff visualisation and let them either accept the drift, adjust the threshold, or abort. — *Reasoning: 5% per-pixel difference is the convention in tools like Percy, Chromatic, and Playwright's own `toHaveScreenshot` matcher — using a known anchor avoids debate about what's "close enough". 10-iteration cap prevents runaway loops (Goal-Driven Execution: bounded loops, not unbounded). User-in-the-loop on non-convergence preserves human judgement.*
- **User's answer:** confirmed.
- **Resolved:** 5% per-pixel diff threshold (configurable via `admin/config.json.design.diffThreshold`); 10-iteration cap; on non-convergence, surface diff + remaining drift to user for accept/adjust/abort. — flows into §Requirements R6, R7; AC AC-4, AC-5.

### Round 5 — Asset comment block: which fields are mandatory?

- **Q:** The mockup's comment block lists every extracted value with `file:line` citation. Which fields are mandatory vs nice-to-have?
- **AI's recommended answer:** Mandatory: every colour, every typography token (font-family, size, weight, line-height), every spacing value used, every component shape (border-radius, shadow). Each with `source: {file}:{line}`. Nice-to-have: viewport/breakpoint definitions, asset references (images, icons). — *Reasoning: the goal's Success-looks-like field requires "every value used in the mockup is auditable" — that has to mean *every* value, not a curated subset. Reviewers can spot-check any value and trace it back. Nice-to-have items don't earn a slot in the load-bearing audit trail.*
- **User's answer:** confirmed, but add: when a value is `INVENTED` (per Round 3's flag), the comment block calls it out distinctly with a separate `INVENTED VALUES` sub-block at the top, before the extracted values. So reviewers see invented values first.
- **Resolved:** mandatory fields = colours, typography (family/size/weight/line-height), spacing, component shape (radius/shadow), each with `source: {file}:{line}` citation. Nice-to-have = viewports, asset refs. INVENTED values get a distinct sub-block at the top of the comment block before the extracted values. — flows into §Requirements R8; AC AC-6.

### Round 6 — Codex adversarial review: when does it fire?

- **Q:** PRD Appendix C4 says Codex is invoked at the end of the iteration loop as an optional adversarial reviewer. When exactly — after the Playwright loop converges, after the user accepts non-convergence, both?
- **AI's recommended answer:** After convergence OR after explicit user-accepted drift. Not after abort. Codex reviews the generated HTML against the live source and the user's stated tone/intent — flags semantic gaps the visual diff misses ("this proposed flow loses the cancel affordance" rather than "this button is 2px off"). If Codex isn't installed, skip silently with a one-line note in the iteration log. — *Reasoning: Codex's value is semantic review — it's complementary to the visual diff, not a replacement. Running it after abort wastes tokens (the mockup is incomplete). Running it after accepted drift is valuable because the user may have accepted visual drift but Codex can spot semantic regressions in the proposed direction.*
- **User's answer:** confirmed.
- **Resolved:** Codex review fires after convergence OR after user-accepted drift (NOT after abort). Skipped silently with iteration-log note when Codex absent. — flows into §Requirements R9; AC AC-7.

## Topics not discussed

- *Figma export / round-tripping* — out of scope per goal's "Out of scope at the goal level" line about not replacing dedicated design tools. If the user wants the mockup in Figma, they paste-edit; the skill doesn't integrate.
- *Design system generation from existing codebase* — different feature with different shape. A `specflow:design-system` skill could surface this in a future PRD.
- *Mobile-specific viewport handling beyond the configured frame* — PRD Appendix C1 mentions the phone-frame wrapper for mobile output; we deferred deeper mobile-specific handling (gesture overlays, safe-area insets, etc.) to keep this PRD shippable. Captured here so reviewers know it was deliberate.
- *Animation / interaction prototyping* — out of scope. Static HTML/CSS mockups only. Anyone wanting interaction prototyping should use Figma or a dedicated tool.
- *Accessibility audit of the generated mockup* — interesting future addition (e.g. axe-core check post-render) but not in v1; doesn't serve the goal's primary outcome of codebase-truth visual fidelity.
- *Per-region drift thresholds* — deferred to v2 if consumers report needs. v1 ships a single configurable global per `admin/config.json.design.diffThreshold`. Surfaced during PRD Gate 2 review (think-before-coding finding tbc-r1-f1).

## Sign-off

- 2026-05-05 — user confirmed alignment; specflow:prd Phase C (synthesis) may proceed.
