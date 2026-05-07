# PRD interview — features/010-design-readback

## Original request
> Wire the design folder into prd Phase A.3 and task Phase A.2 so the proposed.html and iteration-log decisions stop getting ignored downstream. Should be a small extension — read the folder when present, surface the constraints, no-op when absent.

## Codebase context (pre-grilling)
- `plugins/specflow/skills/design/SKILL.md` § C3.1 documents the iteration-log structure and declares every entry's *Why* field load-bearing — empty *Why* is a verify-step failure (`design/SKILL.md:404`).
- `plugins/specflow/skills/prd/SKILL.md` Phase A.3 (lines 72-87 pre-edit) gathers codebase context but does not check for the feature-local `design/` folder.
- `plugins/specflow/skills/task/SKILL.md` Phase A.2 surfaces Gate 2 block-finding resolutions but does not read the iteration log.
- Prior worked examples (001-design-skill, 002-develop-skill) demonstrate the design folder shape; no prior worked example demonstrates downstream readback.
- `v2/docs/PRD.md` § Open questions #2 lists this gap as the second of three live PRD questions; SESSION-HANDOFF.md flags it as Sprint 1 work.

## Round 1 — narrow scope vs. extension family

**Question.** Three readback surfaces are candidates: (a) `specflow:prd` Phase A.3, (b) `specflow:task` Phase A.2, (c) `specflow:develop` Gate 4 (plan-vs-design). Should v2.4 cover all three, or just (a) + (b)?

**Answer.** (a) and (b) only for v2.4. Gate 4 plan-vs-design readback is a v2.5 candidate — `specflow:develop` already has a heavy gate manifest; adding design-readback there risks reviewer overload without first letting the upstream readbacks bake. If Gate 4 lacks design awareness in practice, dogfood will surface the need.

## Round 2 — readback semantics

**Question.** When `iteration-log.md` has post-PRD entries, do tasks need explicit traceability fields, or is it enough to surface them in the user-facing summary?

**Answer.** Explicit `design-decision: iteration-N` frontmatter field per task. Two reasons: (1) reviewers at Gate 3 can grep tasks for the field to audit design-to-task coverage; (2) downstream `/insights` clustering benefits from structured trace fields rather than narrative summaries.

## Round 3 — uncovered iteration handling

**Question.** When a post-PRD iteration entry doesn't map to any R or AC, should `specflow:task` block, prompt, or proceed silently?

**Answer.** Prompt with default-proceed. Block is too aggressive (blocks the user mid-flow on a soft surface); silent-proceed loses the constraint. Prompt format from R7: surface the iteration summary, offer "proceed with note" or "run `specflow:scope-change`". Logged decisions land in `admin/scratch/{NNN-slug}-tasks/post-prd-design-decisions.json` for the user-facing summary at end of Phase B.

## Topics not discussed

- Auto-resolution of PRD/iteration conflicts (deferred to v2.5+).
- Design-folder change detection between successive `specflow:prd` runs (deferred — out of scope for first ship).
- Design-folder readback in `specflow:brief` (deferred — brief composes existing artefacts; design intent already lands via the embedded PRD).
