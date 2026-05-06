# Session handoff — specflow v2.1.0 shipped

**Date:** 2026-05-06
**Prior sessions:** earlier handoffs archived as `SESSION-HANDOFF-2026-04-30-codex-review.md`. This file is the current snapshot.
**Status:** Phase 1 shipped as **v2.0.0**; Phase 2 shipped as **v2.1.0** with the chain dogfooded end-to-end through all six gates on the Phase 3 retro skill (`specflow:complete`). `specflow:complete` itself ships as the lane-execution byproduct (472 lines, 8 phases). Phase 3 PRD specs are pre-drafted for `specflow:decision` (004) and `specflow:scope-change` (005). E6-E10 prompt-edit recommendations from the 003 dogfood debrief await next-session application.
**Next session goal:** apply E6-E10 prompt edits, then build out the remaining Phase 3 skills (`specflow:scope-change`, `specflow:decision`, `/insights`, `/prune`, `/optimize`).

---

## TL;DR — How to resume in 60 seconds

1. Glance at `plugins/specflow/CHANGELOG.md` — 2.0.0 + 2.1.0 release entries list what shipped at each cut.
2. Read this handoff (you're here).
3. Read `plugins/specflow/examples/docs/specflow/features/003-complete-skill/DOGFOOD-DEBRIEF.md` — the 003 dogfood surfaced 5 prompt-edit recommendations (E6-E10) with file:line citations. None blocked the 2.1.0 release; all are incremental tuning.
4. Look at `plugins/specflow/skills/complete/SKILL.md` (472 lines, 8 phases) — the first Phase 3 skill, produced by running `specflow:develop` on its own PRD.
5. The Phase 3 PRD anchors at `examples/.../features/{004-decision-skill,005-scope-change-skill}/` are ready for the next chain run.

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

### Phase 3 still-stubbed

- `specflow:scope-change`, `specflow:decision`, `/optimize`, `/insights`, `/prune` ship as frontmatter-only stubs. PRD anchors exist for 004 and 005 (chain ready); 006-009 not yet drafted.

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

### Priority 1 — Apply E6-E10 prompt edits

**Why first:** the 003 dogfood surfaced 5 incremental edits with file:line citations and replacement text. Apply before any Phase 3 chain runs so downstream artefacts inherit the tighter prompts.

**Edits** (full text in `plugins/specflow/examples/docs/specflow/features/003-complete-skill/DOGFOOD-DEBRIEF.md`):

- **E6** `skills/develop/SKILL.md` Phase A.2 — surface Codex availability check at A.1 with lens-overlap note.
- **E7** `skills/develop/SKILL.md` Phase B.1.5 (new) — formalise `b1_recheck` aggregate-outcome schema with `batch_shape_at_default_cap` field.
- **E8** `skills/complete/SKILL.md` Phase A.3 — surface 30-min stale-lock heuristic as `config.json.complete.staleLockMinutes` (deferred for v2 per Simplicity-First; capture as v2-candidate note for now).
- **E9** `templates/agents/standard/principles/goal-driven-reviewer.md` — codify orphan-phase reverse-traceability lens for code-review gates (currently documented for PRD/tasks gates only).
- **E10** `skills/develop/SKILL.md` Phase F.1.5 (new) — define conditional-pass escalation contract with two-option user prompt.

**Estimated effort:** one focused half-session.

---

### Priority 2 — Phase 3 build (skill bodies for `decision`, `scope-change`)

**Why next:** PRD anchors already exist at `004-decision-skill/` and `005-scope-change-skill/` (both Gate 2 closed). The chain is `specflow:task` → `specflow:develop` → lane execution = SKILL body.

**Sequence:**

1. Run `specflow:task` against `004-decision-skill-prd.md`. Produces tasks + Gate 3 manifest.
2. Run `specflow:develop` against the 004 tasks. Lane plan → Gate 4 → lane execution = `skills/decision/SKILL.md` → Gate 5 → Verifier. Same shape as the 003 dogfood; should run cleanly with E6-E10 applied.
3. Repeat for 005-scope-change-skill.
4. Update MIGRATIONS.md if any new schema additions land (e.g. config keys for these skills).

**Don't start until:** Priority 1 (E6-E10) applied.

---

### Priority 3 — Phase 3 PRD specs for the remaining three skills

**Scope:** `/insights`, `/prune`, `/optimize`. The autoresearch-loop and memory-cadence skills. Same shape as 003/004/005 — recursive-bootstrap PRDs via `specflow:prd`.

**Don't start until:** Priority 2 ships the previous three Phase 3 skill bodies (proves the chain handles the full Phase 3 surface area).

---

### Priority 4 — Real consumer-project dogfood

**Why eventually:** the recursive-bootstrap dogfoods exercise the chain end-to-end on synthetic targets. A real consumer-project run (small implementation feature in ClaimXPro or another live repo) is the next discipline test. Watch for friction the synthetic worked examples didn't surface.

**Don't start until:** Phase 3 skill set is complete (Priorities 2 + 3).

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
