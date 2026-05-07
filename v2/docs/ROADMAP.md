# Specflow roadmap

**Status:** v2.4.0 shipped. Sprint 2 next.
**Companion docs:** `FEATURES.md` (full feature inventory), `SESSION-HANDOFF.md` (current state + locked decisions).

---

## Now — Sprint 2 startup (target v2.5.0)

Five features rewrite primary contracts. One-shot disruption window per the master plan; piecemeal would force three template-version bumps and three example-backfill audits. Sprint 1's `templateVersion` field + audit script + `team-review-bridge` doctrine make this sprint structurally safer than it would have been pre-Sprint-1.

**Features:** 016-brief-enhancements (absorbs 015 key-features-in-brief) · 017-tdd-discipline · 022-cross-task-review · 025-sprint-task-flagging · 029-single-context-window-per-task
(Full scope per feature in `FEATURES.md` § Sprint 2.)

### Per-feature flow (every feature)

1. `/specflow:prd {NNN-slug}` — interview → Phase A codebase context → Phase B requirements + ACs → Phase C draft → Gate 2 manifest debate.
2. `/grill` (sub-skill of `prd`) — adversarial questioning loop until Gate 2 closes `passed` or `passed-with-revisions`.
3. `/specflow:task {NNN-slug}` — Phase A read-back → Phase B coverage matrix → Gate 3 manifest debate.
4. `/specflow:develop {NNN-slug}` — Phase A pre-flight → Phase B lane triage → B.1 mechanical recheck → Phase C Gate 4 → Phase D execution → Phase E Gate 5 (Codex 6th, mandatory for Sprint 2-4) → Phase F Verifier + PR + task-history append.
5. `/specflow:test {NNN-slug}` — full mode by default; targeted re-runs as needed during develop.
6. `/specflow:complete {NNN-slug}` — fires automatically from develop Phase F. Captures retro entry + appends to `task-history.json`.

**Estimated effort:** ~one full session per feature; Sprint 2 likely spans two-to-three sessions.

### Sprint 2 close actions (per master plan iteration discipline)

- [ ] Run the `templateVersion` audit script per `templates/admin/example-versioning.md` § Audit tooling. Backfill / grandfather / retire decisions per stale example.
- [ ] `/specflow:insights` clusters Sprint 2 retros into rule candidates. Promote ≥3-occurrence patterns to `admin/rules/guidelines.md`.
- [ ] `/specflow:prune` staleness-checks decision-log + agent snapshots + task-history.
- [ ] Schedule the weekly `/specflow:optimize` cron (per master plan, the trigger fires at end of Sprint 2).
- [ ] Bump versions: `plugin.json`, `marketplace.json` (both occurrences), `CHANGELOG.md`, MIGRATIONS entry (v2.4 → v2.5).
- [ ] Refresh `SESSION-HANDOFF.md` for Sprint 3 startup.

---

## Next — Sprint 3 startup (target v2.6.0)

**Features:** 018-lessons-registry · 019-task-manifest · 023-test-brand-consistency · 027-reviewer-context-isolation · 028-edge-case-reviewer

Same per-feature flow as Sprint 2.

**Special notes:**
- Co-design schemas so `task-manifest.json` + `lessons.json` + brand-check question set share tag vocabularies and lifecycle conventions. Don't ship the three independently and reconcile later.
- 027 + 028 are the "review-hygiene" pair: 027 enforces fresh-context for every reviewer (cross-cutting doctrine + manifest schema change); 028 adds the edge-case-reviewer principle agent with advisory-finding semantics. Land 027 first so 028's new reviewer immediately conforms.
- Sprint 3 has 5 features (vs Sprint 2's 5) — likely 3+ sessions. Re-bucket if it overruns.

### Sprint 3 close actions

- [ ] `/specflow:insights`, `/specflow:prune`, weekly `/specflow:optimize` cron should be running by now — check it's still healthy.
- [ ] Bump versions; MIGRATIONS entry v2.5 → v2.6.
- [ ] Refresh `SESSION-HANDOFF.md` for Sprint 4 startup.

---

## Later — Sprint 4 startup (target v2.7.0)

**Features:** 020-sprint-skill (with 024-sprint-worktree absorbed) · 026-agent-teams-per-stage

(021-design-image-gen removed during 2026-05-07 review — no clear use case.)

Same per-feature flow. Highest novelty, lowest coupling. Sprint-skill needs mature `lessons.json` + `task-history.json` + the `sprint-bucket` field on tasks (from Sprint 2's 025) + `context-budget-estimate` per task (from Sprint 2's 029). **026 pairs tightly with 020** — the sprint plan output is where per-stage team assignments land; co-design 020 + 026 (don't ship 020 first then retrofit team assignments).

### Sprint 4 close actions

- [ ] Bump versions; MIGRATIONS entry v2.6 → v2.7.
- [ ] After Sprint 4: run a real consumer-project dogfood (not recursive specflow-on-itself). Watch for friction the recursive dogfoods didn't surface.
- [ ] Refresh `SESSION-HANDOFF.md`.

---

## Iteration discipline (standing cadence)

- **Per feature:** `/specflow:complete` runs automatically from develop Phase F.
- **Per sprint close:** `/specflow:insights` clusters that sprint's retros into rule candidates. Promote ≥3-occurrence patterns to `admin/rules/guidelines.md` via the `/insights --promote` flow.
- **Weekly from end of Sprint 2:** `/specflow:optimize` cron on the 6 verifiable-skill targets (`release-version-check`, `simplify`, `format`, `tdd-cadence`, `init`, `feedback-loop-audit`). Per-target weekly budget cap default $10. Schedule via `agent-teams:schedule` or repo CI cron.
- **Per sprint close (after `/insights`):** `/specflow:prune` to staleness-check decision-log + agent snapshots + task-history.
- **Constant improvement signal:** after Sprint 2's insights run, audit whether any cluster surfaces a feature missing from `FEATURES.md` — if yes, slot into Sprint 4 or a new Sprint 5.

---

## Operational refinements (small open items, no sprint assigned)

These are not features — they're small operational knobs that surface as we go. Resolve when next-touched.

- **`config.json.complete.staleLockMinutes`** — surface as a config knob? Currently 30-min hard-coded.
- **Per-region drift thresholds for `specflow:design`** — recorded in 001's "Topics not discussed".
- **Codex integration scope for upgrade migrations** — likely surfaces in v2.x → v2.y migration entries.
- **Mobile viewport defaults for `specflow:design`** — worked example specifies iPhone 15 Pro / Pixel 8.
- **Reviewer permission to amend the interview during debate** — likely yes; implementation TBD.
- **Skill-toggle seed JSON gap:** `/grill`, `/simplify`, `/panic`, `/format`, `/release-version-check`, `/feedback-loop-audit`, `/confidence-check` are not in the `config.json.skills` seed — doctrine documents bare-name skills but they're not in the seeded JSON. Add when next-touched.

---

## Sprint-5 placeholder

Reserve `v2/docs/SPRINT-5-CANDIDATES.md` if Sprint 2's `/specflow:insights` run surfaces clusters not already in `FEATURES.md`.

---

## Validation (runnable, after each sprint)

```bash
# Versions sync across plugin + marketplace
grep version plugins/specflow/.claude-plugin/plugin.json .claude-plugin/marketplace.json

# Every new feature has a worked example
ls plugins/specflow/examples/docs/specflow/features/

# Every example PRD parses + has a closing Gate 2 manifest
find plugins/specflow/examples/docs/specflow/features -name "*-prd.md" -newer plugins/specflow/CHANGELOG.md
find plugins/specflow/examples/docs/specflow/features -path "*debate-log/prd-gate2/manifest.md"

# JSON parses cleanly
find plugins/specflow/examples/docs/specflow -name "*.json" \
  | xargs -L1 python3 -c "import json,sys; json.load(open(sys.argv[1]))"

# MIGRATIONS entry present for the new version
grep -A2 "v2.${SPRINT}" plugins/specflow/MIGRATIONS.md
```

After Sprint 2 onward, weekly:
```bash
# /specflow:optimize logs land in admin/optimize-runs/
ls docs/specflow/admin/optimize-runs/$(date +%Y-W%V)-*.json
```

After every sprint close: confirm `/specflow:insights` produced ≥1 rule candidate; confirm `/specflow:prune` ran without removing live state; confirm `SESSION-HANDOFF.md` refreshed.
