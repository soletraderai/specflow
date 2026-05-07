---
feature: 029-single-context-task
status: drafted
created: 2026-05-07
templateVersion: v2.5
prd: ./029-single-context-task-prd.md
gate3: ./debate-log/tasks-gate3/manifest.md
---

# 029-single-context-task — task plan

This tasks file is itself a meta-demonstration: every task carries the `context-budget-estimate` field that this feature introduces. ONE task (T-3) was deliberately synthesised over-budget to demonstrate the auto-flag-for-split path; that task is shown after its split into T-3.A and T-3.B, with the original block preserved so the rule firing is visible.

## Coverage matrix

| Requirement | Tasks |
|---|---|
| R1 — doctrine doc exists | T-1 |
| R2 — verbatim rule quote | T-1 |
| R3 — `context-budget-estimate` on every task | T-2 |
| R4 — synthesis auto-flags over-budget tasks | T-2 |
| R5 — develop Phase A pre-flight | T-3.A |
| R6 — D/E/F single-line reminder | T-3.B |
| R7 — compaction events log as escalation | T-4 |
| R8 — worked example | T-5 |

| AC | Tasks |
|---|---|
| AC-1, AC-2 | T-1 |
| AC-3, AC-4 | T-2 |
| AC-5 | T-3.A |
| AC-6 | T-3.B |
| AC-7 | T-4 |
| AC-8 | T-5 |

## Tasks

### T-1 — Author `templates/admin/single-context-task.md` (full doctrine)

- **Anchors:** PRD R1, R2.
- **Scope:** `plugins/specflow/templates/admin/single-context-task.md` (new).
- **Acceptance:** Doctrine doc exists with all seven required sections (rationale, the rule verbatim, schema, no-mid-task-compaction contract, how task-synthesis applies the rule, how develop Phase A pre-flights, cross-references). Length ≤120 lines. The rule section quotes decision #21 from `SESSION-HANDOFF.md` verbatim.
- **Depends on:** none.
- **Lane:** Green.
- **context-budget-estimate:** 18000 tokens — PRD slice ~6K (R1+R2+ACs), task spec ~1K, prior-lessons (none — fresh feature) 0K, manifest scaffold 2K, codebase-context (`team-review-bridge.md` reference only) ~3K, test plan 1.5K, headroom 4.5K.
- **sprint-bucket:** 1.
- **prior-lessons:** [] *(forward-looking; 018 will populate at task synthesis once the lessons registry ships)*.

---

### T-2 — Add `context-budget-estimate` to `specflow:task` synthesis

- **Anchors:** PRD R3, R4.
- **Scope:** `plugins/specflow/skills/task/SKILL.md` (additive ≤25 lines: per-task field schema + budget self-check + auto-flag note + chat-line prompt). Cite the doctrine doc for the estimation algorithm; do not inline.
- **Acceptance:** A fresh `specflow:task` run on any feature emits `context-budget-estimate: <int_tokens>` on every task entry. A test feature with one synthesised task whose estimate exceeds `config.json.task.contextBudget` produces an inline auto-flag note in `tasks.md` AND a chat-line prompt naming `specflow:scope-change` as the recut path.
- **Depends on:** T-1.
- **Lane:** Green.
- **context-budget-estimate:** 32000 tokens — PRD slice 6K, task spec 1K, prior-lessons 0K, manifest scaffold 2K, codebase-context (full `task/SKILL.md` 484 lines + doctrine doc) ~18K, test plan 1.5K, headroom 3.5K.
- **sprint-bucket:** 1.
- **prior-lessons:** [].

---

### T-3 *(ORIGINAL — over-budget; auto-flagged for split)*

> **Budget overrun: estimate 96000 tokens vs budget 80000 — split required before develop.**
>
> Synthesis ran the budget self-check and the estimate exceeded `config.json.task.contextBudget` (80K). The auto-flag fired; the chat-line prompt directed the user to `specflow:scope-change` to recut. The original block is preserved here so the rule firing is auditable; T-3.A and T-3.B below are the post-split successors.

- **Anchors (original, since reassigned):** PRD R5, R6.
- **Scope (original):** `plugins/specflow/skills/develop/SKILL.md` — add the Phase A pre-flight sub-step (R5) AND embed the no-mid-task-compaction reminder + supporting prose across Phases D/E/F (R6) in a single change set.
- **Acceptance (original):** Phase A pre-flight fires on ≥20% divergence; D/E/F carry the no-compaction reminder; develop stays ≤500 lines after the change set.
- **context-budget-estimate (original):** 96000 tokens — PRD slice 8K, task spec 1.5K, prior-lessons 0K, manifest scaffold 2K, codebase-context (full `develop/SKILL.md` 697 lines + doctrine doc + `task/SKILL.md` for symmetry checks + `team-review-bridge.md` for shape reference + R5 + R6 PRD anchors + cross-skill chain checks) ~78K, test plan 1.5K, headroom -5K (overflow).
- **Why over-budget:** the codebase-context payload bundles the entire 697-line `develop/SKILL.md` plus three doctrine references plus cross-skill checks for the Phase A reminder placement. Loading the full skill body for a surgical addition violates "just-in-time, not just-in-case" (per `medin-wisc-framework.md`). Splitting the work along the natural seam — Phase A pre-flight (R5) vs D/E/F reminder placement (R6) — lets each successor load only the slice of `develop/SKILL.md` it actually edits.

---

### T-3.A — Add Phase A pre-flight sub-step to `specflow:develop`

- **Anchors:** PRD R5.
- **Scope:** `plugins/specflow/skills/develop/SKILL.md` Phase A only (additive ≤15 lines: new sub-step at A.6 reading per-task `context-budget-estimate` against actual loaded context, three-option developer prompt on ≥20% divergence, non-optional route to `specflow:scope-change` when actual exceeds budget outright).
- **Acceptance:** Phase A.6 exists; the prompt fires on ≥20% divergence; the route-to-scope-change path is non-optional when actual exceeds the configured budget. Develop stays ≤710 lines after the change.
- **Depends on:** T-1, T-2.
- **Lane:** Green.
- **context-budget-estimate:** 42000 tokens — PRD slice 4K (R5 only), task spec 1K, prior-lessons 0K, manifest scaffold 2K, codebase-context (`develop/SKILL.md` Phase A region lines 67-113 ~3K + doctrine doc 4K + R5 anchor verification ~2K + Phase A.5/B boundary check ~1K) — but the load is dominated by the develop-skill-relevant slices loaded just-in-time → estimate ~32K with headroom 10K. Total under budget.
- **sprint-bucket:** 1.
- **prior-lessons:** [].

---

### T-3.B — Add no-mid-task-compaction reminder line to `specflow:develop` Phases D/E/F

- **Anchors:** PRD R6.
- **Scope:** `plugins/specflow/skills/develop/SKILL.md` Phase D entry ONLY (additive ≤5 lines: a single one-line reminder at Phase D entry citing the doctrine doc as the canonical "no mid-task compaction" reference; D/E/F all read it because Phase D entry is the loop entrypoint that gates the per-task execution).
- **Acceptance:** Exactly ONE reminder line exists at Phase D entry; the line cites `templates/admin/single-context-task.md` as the canonical reference; the compaction-is-a-defect-signal stance is *not* duplicated in the skill body. Develop stays ≤710 lines after the change.
- **Depends on:** T-3.A *(must land first so the line numbers are stable when this edit runs)*.
- **Lane:** Green.
- **context-budget-estimate:** 18000 tokens — PRD slice 3K (R6 only), task spec 1K, prior-lessons 0K, manifest scaffold 2K, codebase-context (Phase D region of `develop/SKILL.md` lines 387-432 ~4K + doctrine doc 4K) ~8K, test plan 1.5K, headroom 2.5K.
- **sprint-bucket:** 1.
- **prior-lessons:** [].

> **Split rationale.** T-3 attempted to bundle two surgical additions across two distant regions of `develop/SKILL.md` (Phase A near line 67-113, Phase D near 387-432). Each addition only needs its own region loaded; bundling forced the full 697-line skill body into context. T-3.A and T-3.B each load only their target region. Both successors fit budget; the original 96K estimate is now 42K + 18K = 60K total across two windows. Since each successor runs in its own context window (per the rule this feature codifies), the relevant comparison is per-window, not aggregate.

---

### T-4 — Document compaction-event manifest entry shape

- **Anchors:** PRD R7.
- **Scope:** `plugins/specflow/templates/admin/single-context-task.md` § No-mid-task-compaction contract — a one-line note naming the 019-task-manifest field shape (`event_type: escalation`, outcome `compacted-mid-task`) so retro consumers know the field name. Already covered in T-1's authored doctrine doc — this task is the verification slice.
- **Acceptance:** The doctrine doc names the field shape; a fresh agent reading the doc can locate the manifest field without invoking 019 docs.
- **Depends on:** T-1.
- **Lane:** Green.
- **context-budget-estimate:** 8000 tokens — PRD slice 2K (R7 only), task spec 1K, prior-lessons 0K, manifest scaffold 2K, codebase-context (doctrine doc only — verification, not authoring) ~2K, test plan 1K.
- **sprint-bucket:** 1.
- **prior-lessons:** [].

---

### T-5 — Build worked example folder + meta-demonstration

- **Anchors:** PRD R8, AC-8.
- **Scope:** `plugins/specflow/examples/docs/specflow/features/029-single-context-task/` — PRD, interview, this tasks file (with the meta `context-budget-estimate` field on every task and ONE deliberately-oversized task split into two successors), Gate 2 manifest closed `passed`.
- **Acceptance:** Folder exists with all four artefacts; tasks file demonstrates the auto-flag-for-split path with T-3 → T-3.A + T-3.B; Gate 2 manifest closes `passed`.
- **Depends on:** T-1, T-2, T-3.A, T-3.B, T-4.
- **Lane:** Green.
- **context-budget-estimate:** 28000 tokens — PRD slice 6K (full PRD R/AC list), task spec 1K, prior-lessons 0K, manifest scaffold 2K, codebase-context (`014-team-bridge-spec/` reference shape + doctrine doc) ~16K, test plan 1.5K, headroom 1.5K.
- **sprint-bucket:** 1.
- **prior-lessons:** [].

---

## Open questions inherited from PRD

None — the PRD's grilling rounds closed all four open questions (D/E/F coverage, default budget, divergence handling, escape-hatch absence).

## See also

- PRD: [`./029-single-context-task-prd.md`](./029-single-context-task-prd.md)
- Interview: [`./029-single-context-task-interview.md`](./029-single-context-task-interview.md)
- Doctrine: [`../../../../templates/admin/single-context-task.md`](../../../../templates/admin/single-context-task.md)
- Gate 2 manifest: [`./debate-log/prd-gate2/manifest.md`](./debate-log/prd-gate2/manifest.md)
