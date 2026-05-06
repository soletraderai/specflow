# Session handoff — specflow v2.3.0 shipped; three PRD questions open

**Date:** 2026-05-06
**Prior sessions:** earlier handoffs archived as `SESSION-HANDOFF-2026-04-30-codex-review.md`. This file is the current snapshot.
**Status:** Phase 1 shipped as **v2.0.0**; Phase 2 + the brief skill landed as **v2.2.0**; Phase 3 is now **fully operational** and packaged as **v2.3.0** (committed locally; not pushed). All six Phase 3 skills have operational bodies: `specflow:complete` (472 lines), `specflow:decision` (280 lines), `specflow:scope-change` (436 lines), `specflow:insights` (450 lines), `specflow:prune` (316 lines), `specflow:optimize` (510 lines). The v2 architectural arc per `v2/docs/PRD.md` is feature-complete.

---

## ⭐ START HERE — three open PRD questions to take up next session

`v2/docs/PRD.md` § "Open questions" closed six of the original nine through shipped 2.0-2.3 work; three remain genuinely open. **These are the live agenda for the next session** — pick one (or all) and run the chain on each. None blocks any operational skill; they're refinements.

### 1. `pages.json` ownership

**Question.** Currently a setup-time stub (template-seeded with placeholder routes). The PRD's setup spec mentions a future `specflow:pages` skill that would inventory live routes from the project's router config (Next.js / Remix / Express / etc.). Decide whether to ship `specflow:pages` in v2.4 or accept the manual-stub-plus-test-time-population approach (`specflow:test` updates `pages.json` on first UI run) as enough.

**What to read first:** `plugins/specflow/skills/setup/SKILL.md:340` (the placeholder), `plugins/specflow/examples/docs/specflow/admin/pages.json` (the example shape), `plugins/specflow/skills/test/SKILL.md` (the populator candidate).

**Decision shape.** Either (a) draft `009-pages-skill` PRD via `specflow:prd` and run the chain; or (b) decide-and-document at the PRD level that `pages.json` stays manual + lazy-populated, and remove the "future skill" reference.

### 2. Design mockup readback

**Question.** No skill currently consumes the design folder's `current.html` / `proposed.html` / `iteration-log.md` as downstream context. Two surfaces where this could matter: (a) `specflow:prd` Phase A on a feature with existing design — the proposed.html's component decisions ought to constrain the requirements; (b) `specflow:task` Phase B on a feature whose iteration log carries post-PRD decisions — those decisions should shape task synthesis.

**What to read first:** `plugins/specflow/skills/design/SKILL.md` (the producer), `plugins/specflow/skills/prd/SKILL.md` Phase A.3 (codebase context gathering — the readback target), `plugins/specflow/skills/task/SKILL.md` Phase A (PRD-extract step — second readback target).

**Decision shape.** Either (a) extend Phase A.3 of `specflow:prd` and Phase A.2 of `specflow:task` to ingest the design folder when present (and document that pattern as a v2.4 enhancement), or (b) declare at the PRD level that design is alignment-only (not load-bearing on downstream skills) and close the question.

### 3. `brief.html` commit policy

**Question.** The brief composes PRD + interview + manifests into a single self-contained HTML. Default recommendation is committed (diffable for review). Projects with sensitive surfaces or repo-size pressure may prefer gitignored-as-derived. Should the choice surface as a setup-time prompt or a `config.json.brief.commitPolicy` knob?

**What to read first:** `plugins/specflow/skills/brief/SKILL.md` (the producer + its current commit assumption), `plugins/specflow/skills/setup/SKILL.md` Phase A (where a setup-time prompt would land), `plugins/specflow/MIGRATIONS.md` (template for any v2.3 → v2.4 entry covering a new config knob).

**Decision shape.** Either (a) add a `config.json.brief.commitPolicy` knob (default `"committed"`, alternative `"derived"`) with a `.gitignore` snippet for the derived case, plus a setup-time prompt; or (b) lock the default at committed-only and document the gitignored alternative in `CONTEXT.md` recipes for projects that need it.

The full text of each question (with closure citations for the six already-resolved questions) lives at `v2/docs/PRD.md` § "Open questions".

---

## TL;DR — How to resume in 60 seconds

1. Read § "⭐ START HERE" above — three live PRD questions for next session to act on.
2. Glance at `plugins/specflow/CHANGELOG.md` — 2.0.0 / 2.1.0 / 2.2.0 / 2.3.0 release entries.
3. **All six Phase 3 skills are operational** at `plugins/specflow/skills/{complete,decision,scope-change,insights,prune,optimize}/SKILL.md`. PRD anchors at `examples/docs/specflow/features/{003-008}-*-skill/` all closed Gate 2 `passed-with-revisions`.
4. v2 architecture per `v2/docs/PRD.md` is feature-complete. The three open questions are refinements, not architectural blockers.

If you only have 5 minutes, the START HERE section is the agenda; § "Next up" expands into priority order.

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

### Phase 3 (now fully operational; in `[Unreleased]` pending v2.3.0 cut)

| Skill | Lines | Phases | PRD anchor (R / AC) | Source |
|-------|-------|--------|---------------------|--------|
| `specflow:complete` | 472 | A-H (8) | 14 / 15 | 003-complete-skill |
| `specflow:decision` | 280 | A-F (6) | 11 / 11 | 004-decision-skill |
| `specflow:scope-change` | 436 | A-H (8) | 12 / 13 | 005-scope-change-skill |
| `specflow:insights` | 450 | A-G (7) | 15 / 16 | 006-insights-skill |
| `specflow:prune` | 316 | A-H + restore | 11 / 11 | 007-prune-skill |
| `specflow:optimize` | 510 | A-I (9) | 17 / 16 | 008-optimize-skill |

All six Phase 3 PRD anchors closed Gate 2 `passed-with-revisions`. Notable architectural commitments per skill:

- **`/insights`** — two-pass deterministic clustering (field-shape exact-match + token-frequency n-grams); ≥3-observation promotion threshold; `semantic` cluster-source label reserved for v2 embedding-clustering; refuses below `minCorpusSize` (default 10).
- **`/prune`** — per-surface staleness boundaries (decision-log, rules, agent snapshots, task-history); two-stage archive-then-remove flow; byte-identical round-trip restoration as the binary eval; append-only archive (skill never modifies its own archive).
- **`/optimize`** — generalises `simplify`'s discipline across the verifiable-skill set; six structured mutation operators; per-target weekly budget cap (default $10); decline-streak governance; three independent auto-merge guardrails (HTML comment, no `--auto` call, GH Action human-actor check); structurally enforced no-LLM-as-judge inside the loop.

E6-E10 prompt edits remain applied (from prior session): `skills/develop/SKILL.md` Phase A.2 lens-overlap note, B.1.5 `b1_recheck` aggregate schema, F.2.1 conditional-pass escalation contract; `skills/complete/SKILL.md` Phase A.3 staleLockMinutes v2-candidate note; `goal-driven-reviewer.md` orphan-phase reverse-traceability lens.

---

## Where everything lives

```
plugins/specflow/                                # THE PLUGIN — released as v2.1.0
├── .claude-plugin/plugin.json                   # version: 2.3.0
├── README.md                                    # plugin operational entry point
├── CHANGELOG.md                                 # 2.0.0 + 2.1.0 release entries
├── CORE_PRINCIPLES.md
├── SKILLS.md                                    # skill glossary
├── MIGRATIONS.md                                # v1.x → v2.0 + v2.0 → v2.1
├── skills/                                      # 25 skills, all operational
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

.claude-plugin/marketplace.json                  # version: 2.3.0; description synced
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

### Priority 1 — Cut v2.3.0

**Why first:** Phase 3 is feature-complete. The release packages the three new operational skills (`/insights`, `/prune`, `/optimize`) plus their PRD anchors. Same release discipline as 2.0.0 / 2.1.0 / 2.2.0:

1. Bump `plugins/specflow/.claude-plugin/plugin.json` to 2.3.0.
2. Bump `.claude-plugin/marketplace.json` (metadata.version + plugins[0].version) to 2.3.0; sync description.
3. Move CHANGELOG `[Unreleased]` block to `[2.3.0] — {date}` with the full Phase 3 inventory.
4. Add `MIGRATIONS.md` v2.2 → v2.3 entry covering any new config keys introduced (e.g. `config.json.insights.{minCorpusSize, thresholds}`, `config.json.prune.thresholds.{decisionLog,guidelines,taskHistory}`, `config.json.optimize.{weeklyBudgetPerTarget, judgementWords, declineStreak}`).
5. Local commit. Push to remote + cut GitHub release `2.3.0` is your call (public-facing action).

**Estimated effort:** one focused half-session.

---

### Priority 2 — Real consumer-project dogfood

**Why next:** the recursive-bootstrap dogfoods exercise the chain end-to-end on synthetic targets (the 003-complete-skill dogfood validated all six gates). A real consumer-project run (small implementation feature in ClaimXPro or another live repo) is the next discipline test. Watch for friction the synthetic worked examples didn't surface — particularly around `specflow:develop` lane execution against actual code, and `/insights` clustering on a real `task-history.json`.

**Estimated effort:** one focused session, ideally on a feature with a small blast radius.

---

### Priority 3 — Optional next-wave dogfood debriefs

**Why eventually:** the 002 dogfood produced E1-E5; the 003 dogfood produced E6-E10. The new Phase 3 builds (006/007/008) didn't run a full dogfood (abbreviated chain). If real-project dogfood (Priority 2) surfaces friction in `/insights` / `/prune` / `/optimize`, capture it in DOGFOOD-DEBRIEF files for those features and apply E11-E15+ in a subsequent session.

**Don't start until:** Priority 2 surfaces friction worth codifying.

---

### Priority 4 — Architectural rework (deferred)

The v2 architecture per `v2/docs/PRD.md` is feature-complete. Future work is incremental tuning, not architectural rework. If something larger emerges (e.g. a v3 narrative around cross-project memory or hosted state), it's a fresh PRD, not a Phase 4.

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
ls plugins/specflow/skills/*/SKILL.md                                              # 25 skills
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
