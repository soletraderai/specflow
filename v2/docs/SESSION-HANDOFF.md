# Session handoff

**Date:** 2026-05-07
**Status:** v2.4.0 shipped. Sprint 2 is the next ship cycle (target v2.5.0). All v2.x original PRD open questions are resolved.
**Companion docs:** `FEATURES.md` (full feature inventory), `ROADMAP.md` (what to do, in execution order).

---

## Resume in 60 seconds

1. Read `ROADMAP.md` § Now — Sprint 2 startup. Five features, one-shot template-churn window.
2. Read `FEATURES.md` § Sprint 2 — full scope per feature (015, 016, 017, 022, 025).
3. Glance at `plugins/specflow/CHANGELOG.md` — full v2.4.0 entry covers what just shipped.
4. Sprint 1 mitigated both unnamed risks pre-emptively (`templateVersion` field + `team-review-bridge.md`) so Sprint 2's template churn won't cascade into example debt.

---

## What just shipped — v2.4.0 (Sprint 1)

- Closed 3 PRD open questions (009-pages-policy as decide-not-build; 010-design-readback as build; 011-brief-commit-policy as build).
- 3 new doctrine docs in `plugins/specflow/templates/admin/`: `skill-toggles.md`, `example-versioning.md`, `team-review-bridge.md`.
- 5 new worked examples (010 / 011 / 012 / 013 / 014).
- 8 existing worked examples gained `templateVersion: v2.3` informational tag.
- 0 new skills shipped (additive across existing skill bodies).

Detailed inventory: `plugins/specflow/CHANGELOG.md` § [2.4.0].

---

## Architectural decisions — locked in (DO NOT re-litigate)

1. **Goal-confirmation step** before grilling.
2. **Interview file is the audit trail** (markdown only, no HTML render).
3. **`/grill` is a sub-skill of `specflow:prd`**.
4. **Multi-agent debate manifest** at every adversarial-review gate.
5. **Principle reviewer category** alongside lifecycle agents.
6. **Orchestrator pattern** is mandatory (`templates/orchestrator-pattern.md`).
7. **Skill contracts** via `requires:` / `produces:` frontmatter.
8. **Per-skill token tracking** via `skill-invocations.jsonl`.
9. **`NNN-{slug}-` filename prefix** preserved on top-level feature files.
10. **Directory split** `v2/docs/` (planning) vs `plugins/specflow/` (the plugin).
11. **Design iteration log** captures decisions, not just diffs.
12. **Gate status taxonomy** — `passed | passed-with-revisions | passed-with-escalations | failed`.
13. **Codex is the sixth Gate 5 reviewer** when detected; degradation is graceful when absent.
14. **Lane triage tuple** is verifiability + blast radius + dependency state + confidentiality.
15. **B.1 mechanical lane recheck** runs after triage and before Gate 4.
16. **PRD-anchor format on every plan** (R17 of `specflow:develop`): "We're doing X because of PRD requirement Y. This aligns with goal field Z."
17. **`templateVersion` frontmatter** on every worked-example PRD (v2.4 — 013).
18. **`config.json.skills.{name}.enabled` toggles** — project-level, default-true (v2.4 — 012).
19. **`agent-teams:team-review` is tiebreaker, not default** at Gate 5 — fires only on ≥2-severity-level disagreement (v2.4 — 014).
20. **specflow's own gates ARE the in-feature agent team** — `agent-teams:*` orchestration layers only at sprint boundaries.
21. **Single context window per task implementation** — every task's Red/Green/Refactor runs in one agent context window (no mid-task compaction; no cross-session resumption mid-implementation). Tasks too big to fit are split at synthesis time, not patched mid-execution. Compaction during develop is a defect signal, not a recovery move (Sprint 2 — 029).

### What NOT to do

- Don't merge interview into PRD body.
- Don't add HTML render for interview, tasks, or test plan.
- Don't expand lifecycle agents beyond Orchestrator / Devil's Advocate / Verifier.
- Don't elevate "Explicit beats clever" or "Local reasoning" to top-level principles.
- Don't drop `/grill` as a skill.
- Don't relocate per-feature debate logs to `admin/`.
- Don't drop the `NNN-{slug}-` filename prefix.
- Don't add `--no-verify` shortcuts, mock-the-database tests, or feature flags for "later".
- Don't wrap each feature in `agent-teams:team-spawn` — specflow's gates already define the in-feature team.
- Don't auto-invoke `agent-teams:team-review` on every Gate 5 — tiebreaker only.
- Don't backfill `templateVersion` retroactively to mark v2.0–v2.3 examples as v2.4 — they were authored against the v2.3 template; the tag documents authorship intent, not currency.
- Don't mention Claude, Anthropic, or any AI tooling in user-facing output, commits, PRs, code, or docs (per `CLAUDE.md`). Technical filesystem path references (e.g. `~/.claude/plugins/`) are exempt where the path is the literal install location.

---

## Where everything lives

```
plugins/specflow/                                # THE PLUGIN — released as v2.4.0
├── .claude-plugin/plugin.json                   # version: 2.4.0
├── CHANGELOG.md                                 # 2.0 / 2.1 / 2.2 / 2.3 / 2.4 release entries
├── CORE_PRINCIPLES.md
├── SKILLS.md                                    # skill glossary
├── MIGRATIONS.md                                # v1.x → v2.0 → 2.1 → 2.2 → 2.3 → 2.4
├── skills/                                      # all operational skills
├── templates/admin/                             # CONTEXT.md, lessons.json, skill-toggles.md (v2.4),
│                                                # example-versioning.md (v2.4), team-review-bridge.md (v2.4)
└── examples/docs/specflow/features/             # one folder per shipped feature (001–014)

v2/docs/                                         # planning + handoff (this folder)
├── FEATURES.md                                  # full feature inventory — shipped, sprints 2-4, parked
├── ROADMAP.md                                      # what to do, in execution order
├── SESSION-HANDOFF.md                           # this file
└── knowledge/                                   # research dataset (kept separate)

.claude-plugin/marketplace.json                  # version: 2.4.0; description synced
```

---

## How to validate the work survived this session

```bash
# Versions all on 2.4.0
grep version plugins/specflow/.claude-plugin/plugin.json .claude-plugin/marketplace.json

# Sprint 1 worked examples present
ls plugins/specflow/examples/docs/specflow/features/01[0-4]-*

# All 001-008 PRDs have templateVersion: v2.3
for f in plugins/specflow/examples/docs/specflow/features/00[1-8]-*-prd.md; do
  awk '/^---$/{flag=!flag; next} flag && /^templateVersion:/{print FILENAME": "$0}' "$f"
done

# New doctrine docs present
ls plugins/specflow/templates/admin/{skill-toggles,example-versioning,team-review-bridge}.md

# JSON parses cleanly across all gates and config files
find plugins/specflow/examples/docs/specflow -name "*.json" \
  | xargs -L1 python3 -c "import json,sys; json.load(open(sys.argv[1]))"

# Gate 2 manifests for Sprint 1 features all closed
for f in plugins/specflow/examples/docs/specflow/features/01[0-4]-*/debate-log/prd-gate2/manifest.md; do
  echo "$f"
  grep "^\*\*Status:" "$f"
done

# v2.3 → v2.4 MIGRATIONS entry exists
grep -A2 "^## v2.3 → v2.4" plugins/specflow/MIGRATIONS.md | head -5
```

---

## One last thing

v2.4.0 was the cleanest sprint cycle yet — every Sprint 1 feature closed Gate 2 `passed` (no `passed-with-revisions` or escalations). Sprint 2 will be a harder test — it actually changes the templates the rest of the chain reads. The `templateVersion` audit and the `team-review-bridge` doctrine are the load-bearing scaffolding that should keep Sprint 2 from cascading into example debt.

The architecture is healthy. Sprint 2 picks up cleanly from here.
