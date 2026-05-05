# Session handoff — specflow v2.0.0 shipped

**Date:** 2026-05-06
**Prior sessions:** earlier handoffs archived as `SESSION-HANDOFF-2026-04-30-codex-review.md`. This file is the current snapshot.
**Status:** Phase 1 shipped as **v2.0.0**. The plugin lives at `plugins/specflow/`; v1 is replaced. Marketplace + plugin manifests are in sync. The recursive-bootstrap dogfood pass produced a real Phase 2 PRD anchor (`002-develop-skill`) that the next session builds against.
**Next session goal:** apply the dogfood debrief's prompt edits (E1-E5), then start the Phase 2 build (`specflow:develop`) from the anchored PRD.

---

## TL;DR — How to resume in 60 seconds

1. Glance at `plugins/specflow/CHANGELOG.md` — the 2.0.0 release entry lists what shipped.
2. Read this handoff (you're here).
3. Read `plugins/specflow/examples/docs/specflow/features/002-develop-skill/DOGFOOD-DEBRIEF.md` — the dogfood pass surfaced 2 `block` findings, 5 `concern` findings, and 5 concrete prompt-edit recommendations (E1-E5) with file:line citations.
4. The Phase 2 PRD anchor is `plugins/specflow/examples/docs/specflow/features/002-develop-skill/002-develop-skill-prd.md` (18 requirements, 15 ACs, post-Gate-2 revisions applied). Phase 2 build runs `specflow:task` against this PRD next.

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

### Stubbed by design (not yet operational)

- **Phase 2:** `specflow:develop`, `specflow:agent`. Phase 2 build starts from the `002-develop-skill` PRD anchor.
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

### Priority 1 — Apply dogfood debrief prompt edits (E1-E5)

**Why first:** the dogfood pass surfaced five concrete prompt-edit opportunities, all with file:line citations and replacement text. Applying them tightens Phase 1 prompts before Phase 2 builds new orchestrator surfaces on top.

**Edits to apply** (full text in `plugins/specflow/examples/docs/specflow/features/002-develop-skill/DOGFOOD-DEBRIEF.md`):

- **E1** — `skills/prd/SKILL.md:240` — codify the Vision verbatim-vs-paraphrase contract.
- **E2** — `skills/prd/SKILL.md:298` — add Phase C.3 sub-step cross-checking ACs against Phase 1 skill schemas.
- **E3** — `skills/prd/SKILL.md:454` — extend Gate 2 status taxonomy with `passed-with-revisions`.
- **E4** — `templates/agents/standard/principles/goal-driven-reviewer.md` — add reverse-traceability lens (every AC must verify ≥1 R).
- **E5** — `skills/task/SKILL.md:62` — surface Gate 2 block-finding resolutions as task synthesis input.

**Estimated effort:** one focused half-session.

**Don't:** auto-apply without re-reading the recommended replacement text. Each edit's *Why* references friction the dogfood surfaced; verify the friction still exists in the current prompt before patching.

---

### Priority 2 — Phase 2 build: `specflow:develop`

**Why next:** the dogfood produced the PRD anchor (`002-develop-skill-prd.md`, 18 requirements + 15 ACs, post-Gate-2 revisions applied). The Phase 2 build is now a `specflow:task` run + lane-by-lane implementation against that PRD.

**Sequence:**

1. Run `specflow:task` against `002-develop-skill-prd.md`. Produces `002-develop-skill-tasks.md` + Gate 3 multi-agent debate manifest.
2. Build the skill body (`plugins/specflow/skills/develop/SKILL.md`) following the orchestrator pattern. Watch the parent-context budget — `specflow:develop` is the largest orchestrator yet (lane triage + agent-team integration + Gates 4 + 5).
3. Build `specflow:agent` (per-repo agent registry; `add`/`remove`/`list`/`refresh`). Smaller scope; do this in parallel or after `develop`.
4. Update `MIGRATIONS.md` with v2.0 → v2.1 entry covering any schema additions (e.g. `config.json.develop.greenBatchCap` introduced by R6).

**Don't start until:** Priority 1 (E1-E5) is applied. The dogfood-validated prompts are what the Phase 2 build relies on.

---

### Priority 3 — Phase 3 build

**Scope:** `specflow:complete`, `specflow:decision`, `specflow:scope-change`, `/insights`, `/prune`, `/optimize`. Self-learning memory loop closes here. Gate 6 of the adversarial chain.

**Don't start until:** Phase 2 is shipped and dogfooded.

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
