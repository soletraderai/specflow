# Project context

> Slim live context doc. How this project actually works. Updated by humans, by `feedback-loop-audit`, and incidentally by other skills as they learn things worth recording.

This file is loaded into context for skills that need a high-level understanding of the project before they touch the codebase. Keep it slim — the cap is roughly the same as CLAUDE.md (target 500 lines, hard limit 700). When it grows past that, distill rather than append.

The `feedback-loop-audit` skill generates a starter draft from the codebase. Replace, edit, or augment as the project evolves.

---

## What this project is

[One paragraph. The product, the audience, the problem it solves. Plain language — non-technical readers should be able to follow.]

## Core domain concepts

[List the 3-7 nouns that recur everywhere in this codebase. For each: one-line definition + a representative file or two where the concept lives.]

## How the major pieces fit together

[Short architecture sketch. Frontend ↔ backend ↔ data ↔ external integrations. ASCII diagram welcome but not required.]

## Conventions worth knowing

[Project-specific patterns that surprise newcomers: testing layout, error handling style, state management choice, deployment pipeline, build tooling. Each one a sentence or two.]

## Where the friction lives

[What's known to be brittle, slow, or under active rework. The honest "watch out for X" notes that save people from stepping on the same rakes.]

## Where to look next

- `docs/specflow/admin/decision-log.md` — *why* this project made its non-obvious choices.
- `docs/specflow/admin/rules/glossary.md` — what the AI must respect when it touches this codebase.
- `docs/specflow/features/` — the work in flight, organised one folder per feature.

---

*This file is meant to be read end-to-end in under 5 minutes. If it can't be, distill it.*
