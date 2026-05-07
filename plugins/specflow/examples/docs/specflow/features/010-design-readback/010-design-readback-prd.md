---
feature: 010-design-readback
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
interview: ./010-design-readback-interview.md
---

# 010 — design folder readback in `specflow:prd` and `specflow:task`

## Vision

Wire the feature-local `design/` folder produced by `specflow:design` into the two upstream skills that previously ignored it: `specflow:prd` Phase A.3 (codebase context gathering) and `specflow:task` Phase A.2 (PRD-extract step). A feature whose mockups already exist when the PRD is written has *constrained design decisions* — proposed component boundaries, extracted design tokens, post-PRD iteration-log entries — and those constraints should bind the requirements and tasks the same way `non-negotiable.md` rules do. Currently no skill reads design output downstream; the `proposed.html` and `iteration-log.md` artefacts sit dormant and reviewers/implementers re-derive the same decisions from scratch. After this change, the design folder becomes a first-class input to both skills, and the iteration log's *Why* fields become traceable constraints rather than disposable footnotes.

## Problem

`specflow:design` produces three artefacts in `features/NNN-{slug}/design/`: a `current.html` mockup, a `proposed.html` mockup, and an `iteration-log.md` capturing the *Why* for every change. The iteration log is explicitly load-bearing per `design/SKILL.md` C3.1 — every revision must populate the *Why* field, an empty *Why* is a verify-step failure. The discipline exists because design decisions need to survive into downstream synthesis. But until v2.3, no downstream skill reads the folder. A user runs `specflow:design` to align on a mockup, then `specflow:prd` to write the requirements, and the PRD synthesis pass starts from zero — no awareness that the proposed mockup already established three component boundaries, two colour-token decisions, and one navigation pattern. Worse, when the user iterates the design *after* the PRD lands (a common pattern: PRD review surfaces a design constraint that flips a button into a menu), the iteration log entry captures the decision but the PRD body never reflects it. Tasks synthesised from that PRD inherit the gap.

## Goals

- `specflow:prd` Phase A.3 ingests `features/NNN-{slug}/design/` when present: `proposed.html` for component decisions and extracted-token comments; `iteration-log.md` for *Why*-field decisions; `current.html` only when the user's overview references a *change* (so the diff defines scope).
- `specflow:task` Phase A.2.5 surfaces post-PRD iteration-log entries as additional constraints: entries whose timestamp is later than the PRD's frontmatter date map to existing R/AC pairs (gain `design-decision: iteration-N` field) or surface as scope-change candidates if uncovered.
- Both readbacks are silent no-ops when `design/` is absent — design remains optional.
- The two skills' eval criteria extend to cover the readback: PRD's *Codebase context* section includes a `Design intent (from design/):` prefix bullet when the folder is present; tasks gain `design-decision` fields linking to iteration entries.

## Non-goals

- **No new skill.** The change extends two existing skills' Phase A; no `specflow:design-readback` skill ships.
- **No reverse-write.** `specflow:prd` and `specflow:task` never modify the design folder. The folder is a read-only input.
- **No diff against PRD body for inconsistencies.** If the PRD body says "button" and the iteration log records a flip to "menu", the task synthesis surfaces the conflict to the user but does not auto-resolve. Resolution is the user's via `specflow:scope-change` or by editing the PRD.
- **No design-folder-required gate.** PRD synthesis works without `design/`; tasks synthesis works without `design/`. The readback is opportunistic.

## Users

- **PRD authors** running `specflow:prd` on a feature whose design already landed. They benefit from automatic constraint surfacing — the proposed mockup's component boundaries appear in the *Codebase context* section, so the requirements they write are anchored to the design rather than free-floating.
- **Task synthesisers** running `specflow:task` on a feature whose design iterated post-PRD. They benefit from iteration-log surfacing — post-PRD decisions become `design-decision`-tagged tasks instead of getting lost.
- **Reviewers at Gate 2 / Gate 3** reading the manifest. They benefit because the *Codebase context* section now cites the design folder, making the design-to-requirements traceability explicit.

## Requirements

- **R1.** `specflow:prd` Phase A.3 reads `features/NNN-{slug}/design/{slug}-proposed.html` when the file exists. Extracts the comment-block listing of design-system values pulled from the live codebase and the proposed component boundaries. These appear as bullets in the *Codebase context* section prefixed `Design intent (from design/):`.
  - Trace: skills/prd/SKILL.md Phase A.3 edit (v2.4.0).
  - Serves goal: PRD synthesis gains design-folder awareness.

- **R2.** `specflow:prd` Phase A.3 reads `features/NNN-{slug}/design/{slug}-iteration-log.md` when present. Every entry's *Why* field is treated as a constraint citation; entries triggered by `prd-clarification` are forward-references to requirements; entries triggered by `user-feedback` or `codex-review` are constraints requirements must respect.
  - Trace: skills/prd/SKILL.md Phase A.3 edit (v2.4.0).
  - Serves goal: iteration-log decisions survive into PRD synthesis.

- **R3.** `specflow:task` Phase A.2.5 reads the iteration log and partitions entries by timestamp vs PRD frontmatter date. Pre-PRD entries are noise (already encoded in the PRD body via R1+R2). Post-PRD entries are constraints: each maps to an existing R/AC or surfaces as a scope-change candidate.
  - Trace: skills/task/SKILL.md Phase A.2.5 edit (v2.4.0).
  - Serves goal: post-PRD design decisions cannot get lost.

- **R4.** Both readbacks are silent no-ops when `design/` is absent or when `iteration-log.md` has no relevant entries. No spurious "design folder not found" warnings; no required-input refusals.
  - Trace: skills/prd/SKILL.md A.3 + skills/task/SKILL.md A.2.5 (both phrased as conditional).
  - Serves goal: design remains optional.

- **R5.** The PRD's *Codebase context* section, when design is present, includes 1-3 bullets prefixed `Design intent (from design/):`. The bullets cite the design folder explicitly so reviewers can audit the source.
  - Trace: skills/prd/SKILL.md A.3 (Distill paragraph).
  - Serves goal: traceability — design-to-requirements is auditable, not implicit.

- **R6.** Tasks derived from a feature with a post-PRD iteration log gain a `design-decision: iteration-N` field on their frontmatter pointing at the log entry. Tasks without a corresponding iteration entry omit the field. The field is grep-able for downstream consumers.
  - Trace: skills/task/SKILL.md A.2.5 + tasks-template (see worked-example tasks file).
  - Serves goal: design-to-task traceability is structured, not narrative.

- **R7.** Uncovered post-PRD iteration entries (no R or AC matches) surface to the user with a concrete prompt: *"Iteration {N}'s decision ({summary}) isn't covered by any PRD requirement. Proceed with the decision noted, or run `specflow:scope-change` to add a requirement?"* Default if user accepts: proceed; the decision is logged in `admin/scratch/{NNN-slug}-tasks/post-prd-design-decisions.json`.
  - Trace: skills/task/SKILL.md A.2.5.
  - Serves goal: scope-change candidates surface explicitly rather than getting silently absorbed.

## Acceptance criteria

- **AC-1.** Running `specflow:prd` on a feature where `features/NNN-{slug}/design/proposed.html` exists produces a PRD whose *Codebase context* section contains at least one bullet prefixed `Design intent (from design/):`. Verifies R1, R5.
- **AC-2.** Running `specflow:prd` on a feature where `design/` is absent produces a PRD whose *Codebase context* section contains zero bullets prefixed `Design intent (from design/):` and no warning about the missing folder. Verifies R4.
- **AC-3.** Running `specflow:task` on a feature whose `iteration-log.md` has at least one entry timestamped after the PRD frontmatter date produces at least one task with a `design-decision: iteration-N` frontmatter field. Verifies R3, R6.
- **AC-4.** Running `specflow:task` on a feature with a post-PRD iteration entry that doesn't map to any R surfaces the scope-change prompt described in R7 verbatim. Verifies R7.
- **AC-5.** Both skills' frontmatter `requires:` lists are unchanged (design folder is conditional, not declared); their `produces:` lists gain no new entries (the readback is read-only). Verifies the no-op posture in R4.
- **AC-6.** The 010-design-readback worked example in `examples/docs/specflow/features/010-design-readback/` has all four artefacts populated: PRD, interview, tasks file with at least one `design-decision` field, design folder with proposed.html and iteration-log.md, Gate 2 manifest closed `passed`. Verifies the dogfood discipline.

## Open questions

None. The change is mechanical and small. Future enhancements (auto-resolution of PRD/iteration conflicts, design-folder change detection between PRD runs) are deferred to v2.5+ if dogfood surfaces the need.

## See also

- `plugins/specflow/skills/prd/SKILL.md` Phase A.3 — the producer-side edit
- `plugins/specflow/skills/task/SKILL.md` Phase A.2.5 — the consumer-side edit
- `plugins/specflow/skills/design/SKILL.md` C3.1 — the iteration-log discipline this readback depends on
- `v2/docs/PRD.md` § Resolved decisions #2 — the resolution citation
