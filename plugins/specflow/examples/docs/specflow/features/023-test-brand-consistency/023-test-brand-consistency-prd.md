---
feature: 023-test-brand-consistency
status: shipped
created: 2026-05-08
templateVersion: v2.6
shipped: v2.6.0
interview: ./023-test-brand-consistency-interview.md
---

# Test brand consistency

## Vision

`specflow:test`'s test plan gains a Brand-consistency lens section beyond the binary AC pass/fail. Eight questions covering fonts, colors, spacing, CTAs, empty/error states, accessibility primitives, on-brand subjective feel, and "what might have been missed". Brand findings are **advisory** — they don't fail the binary AC signal; they surface in the Execution log for human review.

## Problem

Pre-023, `specflow:test` evaluated the implementation against AC pass/fail only. ACs are binary by construction (R-IDs in PRD; AC-IDs verify Rs). But many quality concerns — font correctness, on-brand feel, accessibility primitives, edge cases that grilling cannot catch — surface only in QA (per `knowledge/pocock-real-feature-build.md`). Without a lens for these concerns, they ship undetected; the AC pass/fail signal becomes a false-confidence artefact.

## Goals

- Extend `test/SKILL.md` Phase B.3 write template with a "Brand-consistency lens" section between Test cases and Execution log.
- Eight standard questions documented inline; questions are not configurable (the project-taste configurability is the answers, not the question set).
- Brand findings are advisory (severity: info / concern / block); only `block` requires human resolution before ship.

## Non-goals

- Failing the AC binary signal on brand findings. They are advisory.
- Auto-detecting brand violations. The questions are human-answered (or human-assisted-by-AI) at test time.
- Per-project question customisation. The eight standard questions are fixed.

## Requirements

- **R1.** `test/SKILL.md` Phase B.3 write template includes a "Brand-consistency lens" section with the eight standard questions in a table.
- **R2.** Brand findings are recorded separately from AC findings; the AC pass/fail signal is unchanged in semantics.
- **R3.** Brand findings carry severity `info | concern | block`; `block` flags items the human must resolve before ship.

## Acceptance criteria

- **AC-1.** `test/SKILL.md` § Phase B.3 write template carries the literal "Brand-consistency lens" subheading.
- **AC-2.** The eight question rows are present in the table.
- **AC-3.** The advisory-not-AC-failing semantics are documented inline.

## See also

- 028 — edge-case-reviewer (sibling Sprint 3 feature; surfaces edge cases at Gate 4 + 5; brand findings cover post-ship lens)
