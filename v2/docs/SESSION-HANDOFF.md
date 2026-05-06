# Session handoff — specflow v2.0.0 shipped + Phase 2 staged

**Date:** 2026-05-06
**Prior sessions:** earlier handoffs archived as `SESSION-HANDOFF-2026-04-30-codex-review.md`. This file is the current snapshot.
**Status:** Phase 1 shipped as **v2.0.0**; Phase 2 build is staged at `plugins/specflow/skills/{develop,agent}/SKILL.md` with the dogfood-produced 002-develop-skill PRD now implemented end-to-end (19 tasks, Gate 3 closed `passed-with-revisions`). Prompt edits E1-E5 applied. v2.0 → v2.1 migration entry written. Awaiting real dogfood of Phase 2 in a consumer project before cutting v2.1.0.
**Next session goal:** dogfood `specflow:develop` on a real Phase 2 task in a consumer project (or recursive bootstrap on a Phase 3 skill spec); then bump to v2.1.0.

---

## TL;DR — How to resume in 60 seconds

1. Glance at `plugins/specflow/CHANGELOG.md` — the 2.0.0 release entry lists what shipped; the Unreleased section documents the Phase 2 staging.
2. Read this handoff (you're here).
3. Read `plugins/specflow/examples/docs/specflow/features/002-develop-skill/DOGFOOD-DEBRIEF.md` — the dogfood pass surfaced 2 `block` findings and 5 prompt-edit recommendations (E1-E5). All five are now applied.
4. Look at `plugins/specflow/skills/develop/SKILL.md` (649 lines, 6 phases) and `plugins/specflow/skills/agent/SKILL.md` (232 lines, 4 verbs) — these are the Phase 2 skill bodies, ready for dogfood. The 002-develop-skill tasks file + Gate 3 manifest in `examples/docs/specflow/features/002-develop-skill/` shows the chain handled the develop spec end-to-end.

If you only have 5 minutes, jump to §"Next up" — it's the priority-ordered punch list.

---

## What shipped in 2.0.0

### Skills (15 operational + 2 v1-shipped/v2-aligned)

| Spine (5) | Workflow (5) | Trust ladder (2) | Telemetry + discipline (3) | v1-shipped (2) |
|-----------|--------------|------------------|---------------------------|---------------|
| `specflow:setup` (9 phases) | `specflow:task` (5 phases, Gate 3) | `panic` | `specflow:budget` | `specflow:prime` |
| `specflow:prd` (5 phases, Gate 2) | `specflow:test` (3 phases) | `confidence-check` | `specflow:feedback-loop-audit` | `specflow:linear` |
| `/grill` (Goal-pre-flight + adaptive loop) | `specflow:misc` (interactive + auto) | | `simplify` | |
| `specflow:render` (markdown→HTML) | `specflow:design` (5 phases, decision-capture iteration log) | | | |
| `specflow:upgrade` (resumable v1→v2) | `specflow:doctor` (5-category check) | | | |

### Standard agents (7 operational)

- **Lifecycle:** `orchestrator` (closer collation logic + PASS/FAIL/HUMAN-DECISION-NEEDED rules), `devils-advocate` (parallel reviewer covering cross-cutting concerns), `verifier` (operational verification.md schema).
- **Principle reviewers:** `simplicity-reviewer`, `surgical-reviewer`, `think-before-coding-reviewer`, `goal-driven-reviewer` — each with explicit input contracts (command-substitution paths), output JSON schema, Round-1 + Round-3 behaviour, severity calibration, forking discipline.

### Worked examples (two, both fully populated)

- **`001-design-skill/`** — full PRD-through-Gate-3 lifecycle. Interview + PRD + Gate 2 manifest (passed) + 13 tasks + Gate 3 manifest (passed). Calibration anchor for healthy multi-gate flow.
- **`002-develop-skill/`** — recursive-bootstrap dogfood artefact. Interview + PRD + Gate 2 manifest (passed after PRD revisions for 2 `block` findings) + `DOGFOOD-DEBRIEF.md` capturing friction surfaced and 5 prompt-edit recommendations. **This is the Phase 2 PRD anchor.**

### Worked-example admin tree

`plugins/specflow/examples/docs/specflow/admin/` is now populated for a hypothetical SaaS product (Northwind Orders, internal order-management dashboard). Demonstrates `config.json`, `pages.json`, `profiles.json` (5 personas), `environment.json`, `CONTEXT.md`, `decision-log.md` (5 entries), `task-history.json` (5 entries), the rules registry (with 4 illustrative project guidelines), and the agents index. Cross-references between files are internally consistent.

### Phase 2 build (staged, awaiting dogfood)

- **`specflow:develop`** — operational at `plugins/specflow/skills/develop/SKILL.md` (649 lines, 6 phases: pre-flight, lane triage, mechanical pre-Gate-4 lane recheck, Gate 4 plan-vs-PRD debate, lane execution, Gate 5 code-vs-plan with Codex degradation). Implements the full 002-develop-skill PRD (R1-R17 plus R5.1).
- **`specflow:agent`** — operational at `plugins/specflow/skills/agent/SKILL.md` (232 lines, 4 verbs: `add | remove | list | refresh`).
- **MIGRATIONS v2.0 → v2.1** — written; introduces `config.json.develop.{greenBatchCap, codexAtGate5}` (defaults 3 and env-derived), the optional `stack_match_reason` field on `agents/index.json`, and on-demand `develop-gate4/` / `develop-gate5/` debate-log subdirectories.
- **E1-E5 prompt edits** — all five applied to `skills/prd/SKILL.md` (3), `skills/task/SKILL.md` (1), and `templates/agents/standard/principles/goal-driven-reviewer.md` (1).

### Stubbed by design (not yet operational)

- **Phase 3:** `specflow:complete`, `specflow:decision`, `specflow:scope-change`, `/optimize`, `/insights`, `/prune`. Specs not yet drafted.

---

## Where everything lives

```
plugins/specflow/                                # THE PLUGIN — released as v2.0.0
├── .claude-plugin/plugin.json                   # version: 2.0.0
├── README.md                                    # plugin operational entry point
├── CHANGELOG.md                                 # release history
├── CORE_PRINCIPLES.md                           # the four behavioral principles
├── SKILLS.md                                    # skill glossary — every skill has an entry
├── MIGRATIONS.md                                # v1→v2 migration plan (consumed by specflow:upgrade)
├── skills/                                      # 22 skills, 15 operational, 7 stubbed by design
├── templates/
│   ├── orchestrator-pattern.md                  # MUST-READ before building any orchestrator skill
│   ├── profile-examples.json                    # 8 starter profiles
│   ├── admin/                                   # CONTEXT.md template, rules registry seeds
│   └── agents/standard/
│       ├── lifecycle/                           # Orchestrator, Devil's Advocate, Verifier — all operational
│       └── principles/                          # 4 principle reviewers — all operational
└── examples/
    └── docs/specflow/
        ├── admin/                               # ✅ NEW: populated reference tree (Northwind Orders)
        └── features/
            ├── 001-design-skill/                # ✅ full lifecycle (Gates 2 + 3 passed)
            └── 002-develop-skill/               # ✅ NEW: dogfood artefact + Phase 2 PRD anchor

v2/docs/                                         # planning + architectural truth (kept post-ship)
├── PRD.md                                       # the architectural spec (source of truth)
├── SESSION-HANDOFF.md                           # this file
├── SESSION-HANDOFF-2026-04-30-codex-review.md   # archived
├── specflow-overview.html                       # visual overview — open in browser
├── codex-prd-feedback.md                        # earlier review feedback
└── knowledge/                                   # research inputs (historical)

.claude-plugin/marketplace.json                  # version: 2.0.0; description synced with plugin.json
```

`v2/specflow/` no longer exists — its contents were moved to `plugins/specflow/` during the 2.0.0 cut. `v2/docs/` is retained as the architectural reference.

---

## Architectural decisions — locked in (DO NOT re-litigate)

These are settled. Re-opening them costs hours and the original reasoning is durable.

1. **Goal-confirmation step.** `specflow:prd` Phase A articulates a Goal (Outcome / Audience / Success-looks-like / Driving value / Out-of-scope-at-goal-level) and gets explicit user confirmation **before** grilling starts. The goal is the precedent every downstream artefact anchors to. `/grill` refuses to run if the Goal section is unconfirmed.

2. **Interview file replaces `discoveries.md`.** Per-feature `NNN-{slug}-interview.md` is the audit trail. Markdown only (no HTML render). The PRD body references the interview by relative path; the PRD's HTML header links to it.

3. **`/grill` is a sub-skill of `specflow:prd`.** Auto-invoked in Phase B. Re-evaluates the next question after every answer. Can also be invoked manually to extend an existing interview.

4. **Multi-agent debate manifest.** N principle-aligned reviewers + Devil's Advocate (+ Codex when available) fire findings in parallel into a manifest. AI responds Round 2; reviewers sharpen-or-accept Round 3; Orchestrator writes the closing decision entry. Manifests live inside the feature folder (`features/NNN-{slug}/debate-log/{gate}/`).

5. **Principle reviewer category.** New agent category alongside lifecycle. One reviewer per core principle. Set scales 1:1 with `CORE_PRINCIPLES.md`.

6. **Orchestrator pattern.** First-class operational doc at `templates/orchestrator-pattern.md`. Three primitives: forked sub-agent contexts, file handoff between steps, command substitution. Every orchestrator skill MUST follow this pattern. Calibration anchor: 51K tokens (V1 leak) → ~5K tokens (V2 with pattern).

7. **Skill contracts.** Every SKILL.md has `requires:` and `produces:` frontmatter. File-level only (no JSON schemas — Simplicity First).

8. **Per-skill token consumption tracking.** `specflow:budget` tracks parent-context delta per skill invocation via append-only `skill-invocations.jsonl`. Trending-up Δ tokens are the early-warning signal for orchestrator-pattern violations.

9. **File naming convention.** Top-level feature files keep the `NNN-{slug}-` prefix on every filename so multiple PRDs are distinguishable in editor tabs and search. Folder-level uniqueness alone isn't enough.

10. **Directory split.** `v2/docs/` (planning + architectural truth) is separated from the released plugin at `plugins/specflow/`.

11. **Design iteration log captures decisions, not just diffs.** Every change to `design/{slug}-{current|proposed}.html` must land an entry in `design/{slug}-iteration-log.md` with the *Why* field populated. Append-only; reversals are new entries citing the original iteration. Defined in PRD Appendix C3.1. Empty *Why* is a verify-step failure.

### What NOT to do

- ❌ Don't merge interview into PRD body.
- ❌ Don't add HTML render for interview (PRD-only).
- ❌ Don't expand lifecycle agents beyond Orchestrator/Devil's Advocate/Verifier.
- ❌ Don't elevate "Explicit beats clever" or "Local reasoning beats cross-file elegance" to top-level principles (sub-clauses under Simplicity First).
- ❌ Don't drop `/grill` as a skill.
- ❌ Don't relocate per-feature debate logs to `admin/`.
- ❌ Don't drop the `NNN-{slug}-` filename prefix on top-level feature files.
- ❌ Don't add `--no-verify` shortcuts, mock-the-database tests, or feature flags for "later" (CORE_PRINCIPLES).
- ❌ Don't mention Claude, Anthropic, or any AI tooling in user-facing output, commits, PRs, code, or docs (CLAUDE.md).

---

## Next up — priority-ordered

### Priority 1 — Real dogfood pass for Phase 2 (`specflow:develop`)

**Why first:** Phase 2 skill bodies are written but only exercised against the synthetic 002-develop-skill PRD (which the chain itself produced). The skills haven't been run on actual implementation work. Real dogfood means: pick a small implementable feature in a consumer project (or recursively bootstrap a Phase 3 skill spec); run `specflow:develop` on its tasks; observe lane triage, B.1 mechanical recheck, Gate 4 + 5 debate manifests, and Verifier behaviour against real changes.

**Options:**
- **Option A (recommended):** recursive bootstrap on a Phase 3 skill (e.g. `specflow:complete`). Low blast radius — failures land in a new feature folder, not a real codebase. Generates a useful Phase 3 PRD as a byproduct, same shape as the 001/002 examples.
- **Option B:** a small actual implementation feature in a consumer project — bigger blast radius but closer to real conditions.

**Watch for:**
- Lane triage misclassifying a clearly-confidential path (rule-based check failing) → R5 in PRD: confidentiality classification is rule-based; failures here are bug-shaped.
- B.1 mechanical recheck not catching a task that picks up new modules at execution time → R5.1 was added by Gate 2 because reviewer-driven re-lane (R5) couldn't catch this; if B.1 misses it too, the recheck logic needs sharpening.
- Gate 4 reviewers rubber-stamping the lane plan → prompt teeth need sharpening; or all-finding-rejecting → too aggressive.
- Codex absent + Gate 5 manifest still recording Codex findings → the degradation path is broken.

**Estimated effort:** one focused session.

---

### Priority 2 — Cut v2.1.0

**Why:** once dogfood passes, the Phase 2 work is shippable. Same release discipline as v2.0.0:

1. Bump `plugins/specflow/.claude-plugin/plugin.json` from 2.0.0 to 2.1.0.
2. Bump `.claude-plugin/marketplace.json` (metadata.version + plugins[0].version) to 2.1.0; sync description if changed.
3. Move the Phase 2 entry in `CHANGELOG.md` from `[Unreleased]` to `[2.1.0] — {date}`.
4. Cut a GitHub release tagged `2.1.0`.

**Don't start until:** Priority 1 dogfood passes cleanly OR surfaces only fixable issues fixed before promotion.

---

### Priority 3 — Phase 3 build

**Scope:** `specflow:complete`, `specflow:decision`, `specflow:scope-change`, `/insights`, `/prune`, `/optimize`. Self-learning memory loop closes here. Gate 6 of the adversarial chain.

**Don't start until:** Phase 2 is shipped (Priorities 1 + 2) and dogfooded.

---

## Open questions to revisit

These are deferred — not decided, but not blocking. Address when the relevant context arrives.

- **Render parity for tasks/tests/interview.** Should `NNN-{slug}-tasks.md`, `NNN-{slug}-test.md`, or `NNN-{slug}-interview.md` also render to HTML? Current decision: PRD only. Defer until consumers report a readability complaint that extends past PRDs.
- **Rendered PRD commit policy.** Commit `NNN-{slug}-prd.html` alongside markdown, or gitignore as derived? Recommend committed (small, diffable, lets reviewers click through PRs without running render locally). User hasn't explicitly decided.
- **Reviewer permission to amend the interview during debate.** Likely yes — a reviewer flagging an unflagged intentional-silence is itself a finding. Implementation TBD.
- **`/insights` and `/prune` cadence.** Phase 3 — monthly + quarterly. Decide when Phase 3 lands.
- **Mobile viewport defaults for `specflow:design`.** Specified in worked example as iPhone 15 Pro / Pixel 8. Implementer can confirm.
- **Codex integration scope for upgrade migrations.** Not yet specified. Likely surface in v2.x → v2.y migration entries when those exist.
- **Per-region drift thresholds for `specflow:design`.** Recorded in worked-example interview's "Topics not discussed" as a v2 candidate. Surface if consumers report drift-blindness on specific UI surfaces.

---

## How to validate the work survived this session

After clearing context and re-loading:

```bash
# Plugin layout intact at the released location
ls -lh plugins/specflow/.claude-plugin/plugin.json                                 # version: 2.0.0
ls -lh plugins/specflow/CHANGELOG.md                                               # 2.0.0 entry dated 2026-05-06
ls plugins/specflow/skills/*/SKILL.md                                              # 22 skills, all should exist
ls plugins/specflow/templates/agents/standard/lifecycle/                           # orchestrator, devils-advocate, verifier
ls plugins/specflow/templates/agents/standard/principles/                          # 4 principle reviewers

# Marketplace + plugin manifests in sync
grep version plugins/specflow/.claude-plugin/plugin.json .claude-plugin/marketplace.json     # all 2.0.0

# Worked examples
ls plugins/specflow/examples/docs/specflow/admin/                                  # populated reference tree
ls plugins/specflow/examples/docs/specflow/features/001-design-skill/              # full lifecycle through Gate 3
ls plugins/specflow/examples/docs/specflow/features/002-develop-skill/             # dogfood artefact + Phase 2 anchor
cat plugins/specflow/examples/docs/specflow/features/002-develop-skill/DOGFOOD-DEBRIEF.md   # E1-E5 prompt edits

# Architectural reference still intact
ls -lh v2/docs/PRD.md v2/docs/SESSION-HANDOFF.md                                   # source of truth + this file
```

---

## One last thing

The Phase 1 ship is the discipline test, not the destination. The 2.0.0 release proves the chain end-to-end on two features: a synthetic worked example (`001`) hand-iterated for calibration cleanliness, and a dogfood artefact (`002`) that surfaced authentic friction (2 `block` findings, 1 split push-back, 5 prompt-edit recommendations). Both gates passed cleanly in both features after the chain handled them.

Phase 2 builds on this substrate. The `002-develop-skill` PRD is not just an artefact — it's the spec the Phase 2 build implements against. Every Phase 2 task should trace back to a numbered requirement in that PRD. If a build decision can't cite an R, either the PRD is missing a requirement (run `specflow:scope-change` when that ships, or amend manually for now) or the build is exceeding scope.

The architecture survived the dogfood. Apply E1-E5, then ship Phase 2.
