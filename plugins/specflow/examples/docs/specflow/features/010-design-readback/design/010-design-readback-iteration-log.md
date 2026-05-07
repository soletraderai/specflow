# Design iteration log — 010-design-readback

Append-only. Every entry MUST populate the *Why* field — empty *Why* is a verify-step failure (`design/SKILL.md:404`).

## Iteration 1 — initial extraction

- **Files changed:** `010-design-readback-proposed.html` (new).
- **What changed:** Generated initial proposed mockup from live `app/styles/tokens.ts` + `app/components/NotificationsPopover.tsx`.
- **Why (the decision):** First-pass alignment artefact. Reviewers needed a discussion surface before requirements crystallise.
- **Triggered by:** `manual-observation`.
- **Playwright drift:** Not applicable (initial generation).
- **Outstanding:** Component boundary for the design-bullet styling — initially used a custom badge; revisit in iteration 2.

---

## Iteration 2 — token consolidation

- **Files changed:** `010-design-readback-proposed.html`.
- **What changed:** Replaced custom badge styling for the `Design intent` prefix with the existing mono-uppercase heading grammar already used in the codebase context heading.
- **Why (the decision):** Keeping the prefix grammar consistent with the surrounding section header reduces visual noise and avoids introducing a new badge component for a one-off use case. The codebase-truth principle (`design/SKILL.md` C2) says use existing tokens unless the user explicitly asks for a departure — they did not.
- **Triggered by:** `codex-review`.
- **Playwright drift:** Diff against live PRD render: 0 region drift (the prefix now renders identically to the existing section heading).
- **Outstanding:** None.

---

## Iteration 3 — post-PRD: grouped-by-day surface

- **Files changed:** `010-design-readback-proposed.html`.
- **What changed:** Iteration-log readback bullet references "grouped-by-day list" rather than the original "single notification list" framing.
- **Why (the decision):** Post-PRD review surfaced that the notifications-popover example used in the proposed mockup needs to demonstrate a *post-PRD design decision* — that's the entire point of `specflow:task` Phase A.2.5. Flipping the list grouping after the PRD landed gives task synthesis a real iteration entry to surface as a constraint. The `specflow:task` worked example tasks file relies on this entry to demonstrate the `design-decision: iteration-3` field.
- **Triggered by:** `user-feedback`.
- **Playwright drift:** Not applicable (text-only change).
- **Outstanding:** None — this entry is the canonical post-PRD example for the worked-example feature.
