# Session handoff — specflow v2.1.0 shipped + Phase 3 partial

**Date:** 2026-05-06
**Prior sessions:** earlier handoffs archived as `SESSION-HANDOFF-2026-04-30-codex-review.md`. This file is the current snapshot.
**Status:** Phase 1 shipped as **v2.0.0**; Phase 2 shipped as **v2.1.0** with the chain dogfooded end-to-end through all six gates on `specflow:complete`. **Phase 3 build is now ~50% complete**: `specflow:complete` (472 lines), `specflow:decision` (280 lines), and `specflow:scope-change` (436 lines) are operational. E6-E10 prompt edits applied. `/insights`, `/prune`, `/optimize` remain stubs (no PRDs yet). All work post-2.1.0 sits in `[Unreleased]`; cut to v2.2.0 when remaining Phase 3 skills land.
**Next session goal:** draft PRDs for the remaining three Phase 3 skills (`/insights`, `/prune`, `/optimize`) via the recursive-bootstrap chain, then build their bodies.

---

## TL;DR — How to resume in 60 seconds

1. Glance at `plugins/specflow/CHANGELOG.md` — 2.0.0 + 2.1.0 release entries plus an `[Unreleased]` block with the post-2.1.0 Phase 3 build progress.
2. Read this handoff (you're here).
3. Three Phase 3 skills are now operational at `plugins/specflow/skills/{complete,decision,scope-change}/SKILL.md`. The 003-complete-skill DOGFOOD-DEBRIEF (E6-E10) has been applied across `skills/develop/SKILL.md`, `skills/complete/SKILL.md`, and `templates/agents/standard/principles/goal-driven-reviewer.md`.
4. Three Phase 3 skills remain stubbed: `/insights`, `/prune`, `/optimize`. No PRD anchors exist for them yet — the next session drafts those via the recursive-bootstrap chain.

If you only have 5 minutes, jump to §"Next up" — it's the priority-ordered punch list.

---

## What shipped in 2.1.0

### Phase 2 skills (operational)

- **`specflow:develop`** — 649-line orchestrator at `plugins/specflow/skills/develop/SKILL.md`. 6 phases: pre-flight + plugin/CLI/MCP detection; lane triage with rule-based confidentiality classification; mechanical pre-Gate-4 lane recheck (R5.1); Gate 4 plan-vs-PRD debate manifest; lane execution (green batched / yellow HITL / red human-led); Gate 5 code-vs-plan with Codex degradation; Verifier + PR + task-history. Implements the dogfooded 002-develop-skill PRD.
- **`specflow:agent`** — 232-line per-repo registry at `plugins/specflow/skills/agent/SKILL.md`. 4 verbs: `list | add | remove | refresh`. Snapshots specialised agents; surfaces drift on refresh; refuses removal of standard agents.

### Phase 3 skill (operational by dogfood byproduct)

- **`specflow:complete`** — 472-line orchestrator at `plugins/specflow/skills/complete/SKILL.md`. 8 phases A-H produced by running `specflow:develop` on its own PRD. Idempotency, race-protection via lock file, schema-validated `task-history.json` writes, optional elevation to `decision-log.md` with triple-flag tracking, Linear sync.

### Worked examples added

- **`features/003-complete-skill/`** — full lifecycle: interview + PRD (14 R / 15 AC) + Gate 2 (passed-with-revisions, 7 findings, 2 push-backs defended) + 15 tasks + Gate 3 (passed-with-revisions, 6 findings) + Gate 4 (passed-with-revisions, 6 findings) + Gate 5 with Codex (passed-with-revisions, 8 findings; Codex contributed 2/8 including a real correctness defect) + Verifier outcome (`verified-with-conditions`: 14 pass + 1 conditional) + DOGFOOD-DEBRIEF with E6-E10.
- **`features/004-decision-skill/`** — Phase 3 PRD spec for `specflow:decision`: interview + PRD + Gate 2 manifest passed-with-revisions. Ready for next-session implementation.
- **`features/005-scope-change-skill/`** — Phase 3 PRD spec for `specflow:scope-change`: interview + PRD + Gate 2 manifest passed-with-revisions. Ready for next-session implementation.

### Prompt edits applied

- **E1** `skills/prd/SKILL.md` — Vision verbatim-vs-paraphrase contract codified.
- **E2** `skills/prd/SKILL.md` — Phase C.3 self-check now cross-references ACs against Phase 1 skill schemas.
- **E3** `skills/prd/SKILL.md` — Gate 2 status taxonomy extended with `passed-with-revisions`.
- **E4** `templates/agents/standard/principles/goal-driven-reviewer.md` — orphan-AC reverse-traceability lens.
- **E5** `skills/task/SKILL.md` — Phase A.2 surfaces Gate 2 block-finding resolutions.

### Migrations

- `MIGRATIONS.md` v2.0 → v2.1 entry: introduces `config.json.develop.{greenBatchCap default 3, codexAtGate5 env-derived}`, optional `stack_match_reason` on `agents/index.json`, on-demand `develop-gate4/` and `develop-gate5/` debate-log subdirectories. Additive; backups retained.

### Phase 3 partial (post-2.1.0, in `[Unreleased]`)

- **`specflow:decision`** — operational at `plugins/specflow/skills/decision/SKILL.md` (280 lines, 6 phases A-F). Implements `004-decision-skill-prd.md` end-to-end via the abbreviated chain (PRD → SKILL body without per-skill Gate 3/4/5 ceremony, since 003 dogfood proved the full chain).
- **`specflow:scope-change`** — operational at `plugins/specflow/skills/scope-change/SKILL.md` (436 lines, 8 phases A-H). Implements `005-scope-change-skill-prd.md`.
- **E6-E10 prompt edits applied** — `skills/develop/SKILL.md` (E6 lens-overlap note, E7 `b1_recheck` aggregate-outcome schema with `batch_shape_at_default_cap`, E10 conditional-pass escalation contract); `skills/complete/SKILL.md` (E8 v2-candidate note for `staleLockMinutes`); `templates/agents/standard/principles/goal-driven-reviewer.md` (E9 orphan-phase reverse-traceability lens).

### Phase 3 still-stubbed

- `/insights`, `/prune`, `/optimize` ship as frontmatter-only stubs. No PRD anchors yet.

---

## Where everything lives

```
plugins/specflow/                                # THE PLUGIN — released as v2.1.0
├── .claude-plugin/plugin.json                   # version: 2.1.0
├── README.md                                    # plugin operational entry point
├── CHANGELOG.md                                 # 2.0.0 + 2.1.0 release entries
├── CORE_PRINCIPLES.md
├── SKILLS.md                                    # skill glossary
├── MIGRATIONS.md                                # v1.x → v2.0 + v2.0 → v2.1
├── skills/                                      # 22 skills:
│   │                                              ✦ 18 operational (Phase 1's 15 + develop, agent, complete)
│   │                                              ✦ 4 stubbed (decision, scope-change, /insights, /prune)
│   │                                              (/optimize is also a stub — 22 total)
│   └── …
├── templates/                                    # orchestrator-pattern.md + admin/ + agents/standard/
└── examples/
    └── docs/specflow/
        ├── admin/                               # populated reference tree (Northwind Orders)
        └── features/
            ├── 001-design-skill/                # full lifecycle (Gates 2+3 passed)
            ├── 002-develop-skill/               # Phase 2 dogfood (Gates 2+3 passed) → drove 2.1.0
            ├── 003-complete-skill/              # Phase 3 dogfood (Gates 2/3/4/5 passed-with-revisions, Verifier verified-with-conditions) → drove 2.1.0
            ├── 004-decision-skill/              # Phase 3 PRD spec (Gate 2 passed-with-revisions); chain-ready
            └── 005-scope-change-skill/          # Phase 3 PRD spec (Gate 2 passed-with-revisions); chain-ready

v2/docs/                                         # planning + architectural truth
├── PRD.md                                       # the architectural spec
├── SESSION-HANDOFF.md                           # this file
└── …

.claude-plugin/marketplace.json                  # version: 2.1.0; description synced
```

---

## Architectural decisions — locked in (DO NOT re-litigate)

These are settled:

1. **Goal-confirmation step** before grilling.
2. **Interview file is the audit trail** (markdown only, no HTML render).
3. **`/grill` is a sub-skill of `specflow:prd`**.
4. **Multi-agent debate manifest** at every adversarial-review gate.
5. **Principle reviewer category** alongside lifecycle agents.
6. **Orchestrator pattern** is mandatory (`templates/orchestrator-pattern.md`).
7. **Skill contracts** via `requires:`/`produces:` frontmatter.
8. **Per-skill token tracking** via `skill-invocations.jsonl`.
9. **`NNN-{slug}-` filename prefix** preserved on top-level feature files.
10. **Directory split** `v2/docs/` vs `plugins/specflow/`.
11. **Design iteration log** captures decisions, not just diffs.
12. **Gate status taxonomy** (E3): `passed | passed-with-revisions | passed-with-escalations | failed`. Used uniformly across Gates 2-5.
13. **Codex is the sixth Gate 5 reviewer** when detected; degradation is graceful when absent (manifest header records `codex: unavailable`).
14. **Lane triage tuple** is verifiability + blast radius + dependency state + confidentiality. Confidentiality is rule-based via `confidentialPaths`, not AI-rated.
15. **B.1 mechanical lane recheck** runs after triage and before Gate 4 — file-count + module + path-glob — and records outcome even when no upgrades trigger.
16. **PRD-anchor format on every plan** (R17 of `specflow:develop`): `"We're doing X because of PRD requirement Y. This aligns with goal field Z."`

### What NOT to do

- ❌ Don't merge interview into PRD body.
- ❌ Don't add HTML render for interview, tasks, or test plan.
- ❌ Don't expand lifecycle agents beyond Orchestrator/Devil's Advocate/Verifier.
- ❌ Don't elevate "Explicit beats clever" or "Local reasoning" to top-level principles.
- ❌ Don't drop `/grill` as a skill.
- ❌ Don't relocate per-feature debate logs to `admin/`.
- ❌ Don't drop the `NNN-{slug}-` filename prefix.
- ❌ Don't add `--no-verify` shortcuts, mock-the-database tests, or feature flags for "later".
- ❌ Don't mention Claude, Anthropic, or any AI tooling in user-facing output, commits, PRs, code, or docs (CLAUDE.md). Technical filesystem path references (e.g. `~/.claude/plugins/`) are exempt where the path is the literal install location the skill must operate on.

---

## Next up — priority-ordered

### Priority 1 — Phase 3 PRD specs for the remaining three skills

**Why first:** `/insights`, `/prune`, `/optimize` are the autoresearch-loop and memory-cadence skills. They have no PRD anchors yet — same recursive-bootstrap shape as 003/004/005 produced. Each gets its own feature folder (006/007/008), interview, PRD, and Gate 2 manifest.

**Sequence:**

1. Recursive bootstrap PRDs via the `specflow:prd` chain on each:
   - `006-insights-skill` — surface recurring patterns from `task-history.json` (monthly cadence). Suggested rule registry promotions (observation → guideline → non-negotiable).
   - `007-prune-skill` — prune stale rules, decisions, and snapshots (quarterly cadence). Reversible via archive.
   - `008-optimize-skill` — autoresearch loop across the verifiable-skill set (~6 initial targets). Single-skill PR per run; human merge owns taste.
2. Each PRD passes Gate 2 (allow `passed-with-revisions`; authentic dogfood discipline holds — push back on at least one finding per skill).

**Estimated effort:** one focused session per skill, possibly parallel agents (one per skill).

---

### Priority 2 — Phase 3 skill bodies for the remaining three

**Why next:** with PRDs anchored, the abbreviated-chain pattern proven in this session can apply directly — read PRD, write SKILL body to spec. The 003 chain demonstrated the full ceremony; subsequent Phase 3 builds can use the abbreviated pattern unless they're architecturally novel.

**Note on `008-optimize-skill`:** this skill's body is non-trivial — it orchestrates a bounded autoresearch loop across multiple verifiable skills. It will likely be the largest of the remaining three. Consider running the full chain (Gate 4 + Gate 5 + Verifier) on `008-optimize` rather than the abbreviated path.

---

### Priority 3 — Cut v2.2.0

**Why:** all six Phase 3 skills will be operational. Same release discipline as v2.1.0:

1. Bump `plugins/specflow/.claude-plugin/plugin.json` to 2.2.0.
2. Bump `.claude-plugin/marketplace.json` (metadata.version + plugins[0].version) to 2.2.0.
3. Move CHANGELOG `[Unreleased]` to `[2.2.0] — {date}` with the full Phase 3 inventory.
4. Cut a GitHub release tagged `2.2.0`.
5. Update `MIGRATIONS.md` v2.1 → v2.2 if any new schema additions land (e.g. autoresearch-eval config for `/optimize`).

**Don't start until:** Priorities 1 + 2 are both shipped.

---

### Priority 4 — Real consumer-project dogfood

**Why eventually:** the recursive-bootstrap dogfoods exercise the chain end-to-end on synthetic targets. A real consumer-project run (small implementation feature in ClaimXPro or another live repo) is the next discipline test. Watch for friction the synthetic worked examples didn't surface.

**Don't start until:** Phase 3 skill set is complete (Priorities 1 + 2 + 3).

---

## Open questions to revisit

- **Render parity for tasks/tests/interview.** Current decision: PRD only.
- **Rendered PRD commit policy.** Recommend committed (small, diffable).
- **Reviewer permission to amend the interview during debate.** Likely yes; implementation TBD.
- **`/insights` and `/prune` cadence.** Decide when those skills land.
- **Mobile viewport defaults for `specflow:design`.** Worked example specifies iPhone 15 Pro / Pixel 8.
- **Codex integration scope for upgrade migrations.** Likely surfaces in v2.x → v2.y migration entries.
- **Per-region drift thresholds for `specflow:design`.** Recorded in 001's "Topics not discussed".
- **`config.json.complete.staleLockMinutes`** (E8) — surface as a config knob? Currently 30-min hard-coded in `skills/complete/SKILL.md`.

---

## How to validate the work survived this session

```bash
# Versions all on 2.1.0
grep version plugins/specflow/.claude-plugin/plugin.json .claude-plugin/marketplace.json

# Plugin layout intact
ls plugins/specflow/skills/*/SKILL.md                                              # 22 skills
wc -l plugins/specflow/skills/{develop,agent,complete}/SKILL.md                    # 649, 232, 472

# Worked examples
ls plugins/specflow/examples/docs/specflow/features/                               # 001..005 directories
cat plugins/specflow/examples/docs/specflow/features/003-complete-skill/DOGFOOD-DEBRIEF.md | head -20

# JSON parses cleanly across all gates
find plugins/specflow/examples/docs/specflow/features -name "*.json" | xargs -L1 python3 -c "import json,sys; json.load(open(sys.argv[1]))"
```

---

## One last thing

The 2.1.0 release proves the chain handles a complete Phase 3 spec end-to-end through every gate. Of the 35 findings across 4 gates of the 003 dogfood (Gate 2 + Gate 3 + Gate 4 + Gate 5):

- 0 blocks landed at any gate (the chain caught architectural ambiguity earlier; the design discipline scales).
- 4 push-backs from the AI; 1 sharpened by Codex (the lone non-trivial sharpen across all gates) — proving Codex earned its sixth-reviewer slot.
- 1 conditional-pass at Verifier — the only non-clean Verifier outcome — and it cites a concrete cross-skill schema dependency rather than ambiguous failure.

The architecture is healthy. Phase 3 build picks up cleanly from here.
