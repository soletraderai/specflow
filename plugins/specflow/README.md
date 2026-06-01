# Specflow

> **Global plugin, local memory, adaptive output.**

Specflow is installed once globally and behaves like a different plugin in every repo it touches. Each project accumulates its own constitution, profiles, agents, decisions, and task history — and uses that memory to make every subsequent piece of work smarter than the last.

Phase 1 of the v2 architecture has shipped (release 2.0.0). Fifteen operational skills, four principle reviewers, three lifecycle agents, and two worked-example features (`001-design-skill` for the full PRD-through-Gate-3 lifecycle; `002-develop-skill` for the recursive-bootstrap dogfood that anchored the release). Phase 2 (`specflow:develop`, `specflow:agent`) and Phase 3 (memory loop) ship in subsequent releases. See [`CHANGELOG.md`](./CHANGELOG.md) for the inventory.

This README is the **operational** entry point: what the plugin contains, how it's laid out, and how the pieces fit together. The architectural rationale lives at `v2/docs/PRD.md` in the repository root.

---

## Layout

```
plugins/specflow/
├── .claude-plugin/
│   └── plugin.json              # plugin metadata (version, description, keywords)
├── README.md                    # this file
├── CHANGELOG.md                 # release history (populated as v2 ships)
├── CORE_PRINCIPLES.md           # the four behavioral principles every skill loads
├── SKILLS.md                    # skill glossary — one entry per skill, source of truth for what the plugin can do
├── MIGRATIONS.md                # version-to-version migration plan consumed by specflow:upgrade
├── skills/                      # one folder per skill, each with SKILL.md + supporting files
├── templates/                   # files setup actively copies into a project
│   ├── agents/standard/
│   │   ├── lifecycle/           # Orchestrator, Devil's Advocate, Verifier — plan / challenge / confirm
│   │   └── principles/          # one reviewer per core principle — Simplicity, Surgical, Think-Before-Coding, Goal-Driven
│   ├── admin/                   # admin folder seeds (CONTEXT.md, rules registry, profile examples)
│   └── ...
└── examples/                    # read-only worked example users can browse
    └── docs/specflow/           # full ClaimXPro-shaped layout, sanitised
```

**Templates vs examples:** `templates/` are files setup actively copies into the user's repo. `examples/` is read-only reference material — users browse it to understand the full shape but never edit it directly.

---

## Per-repo isolation

Specflow is installed once globally at `~/.claude/plugins/specflow/`. Everything else lives inside each project at `docs/specflow/`, committed to that repo.

Each project gets its own:
- `config.json` and `pages.json` (in `admin/`)
- `profiles.json` — user personas
- `decision-log.md` and `task-history.json` — self-learning memory
- `environment.json` — inventory of CLIs, MCPs, plugins, agents available here
- `rules/` — project rules registry
- `agents/` — per-repo agent registry. Standards split into `lifecycle/` (Orchestrator, Devil's Advocate, Verifier — plan / challenge / confirm) and `principles/` (one reviewer per core principle — Simplicity, Surgical, Think-Before-Coding, Goal-Driven). Plus `specialised/` matched to the project's stack. All snapshotted.
- `features/NNN-{slug}/` — one self-contained directory per feature (PRD, tasks, tests, design, docs, assets)

Repo A's self-learning corpus never touches Repo B's. The plugin always resolves paths relative to the current working directory — never a global cache.

---

## The four behavioral principles

Loaded into the system prompt of every skill. Non-negotiable.

1. **Think before coding** — surface tradeoffs, state assumptions, push back when a simpler approach exists, stop when confused.
2. **Simplicity first** — minimum code that solves the problem; no speculative abstractions; no flexibility that wasn't requested.
3. **Surgical changes** — touch only what you must; if you spot something out-of-scope, log it as a `misc-task` rather than fixing inline.
4. **Goal-driven execution** — every skill that produces output declares verify steps inline; loop until verified.

Full text: [`CORE_PRINCIPLES.md`](./CORE_PRINCIPLES.md).

---

## How a project uses the plugin

The intended lifecycle of a feature, from blank slate to verified delivery:

1. `specflow:setup` — once per project. Creates `docs/specflow/admin/`, runs profile interview, seeds rules, inventories environment.
2. `specflow:prime` — primes the codebase context for a piece of work.
3. `specflow:prd` — user-facing entry point. Four-phase: writes interview preamble → invokes `/grill` (one question at a time, re-evaluating after each answer, appending to `NNN-{slug}-interview.md`) → synthesises `NNN-{slug}-prd.md` from resolved assumptions → fires Gate 2 multi-agent debate manifest into `features/NNN-{slug}/debate-log/prd-gate2/`. Closes with a chat-line pointing the user at `specflow:brief {NNN-slug}` as an opt-in next step.
4. `specflow:brief` (optional) — composes `NNN-{slug}-brief.html` (visual abstract + PRD body + interview + agent reviews) and opens it in the browser. Manual invocation; the PRD no longer auto-fires this.
5. `specflow:task` — generates `NNN-{slug}-tasks.md` with a coverage matrix and Gate 3 adversarial review.
6. `specflow:design` (optional) — produces `current.html` + `proposed.html` mockups under the feature folder, grounded in the live codebase.
7. `specflow:linear` — exports tasks to Linear with bidirectional sync.
8. `specflow:develop` (Phase 2) — orchestrates implementation through green/yellow/red lanes.
9. `specflow:test` — verifies the work against the PRD's acceptance criteria.
10. `specflow:complete` (Phase 3) — captures the retro that feeds memory.

Cross-feature workflows: `specflow:misc` for one-off bugs/fixes, `specflow:upgrade` to refresh aged installations, `specflow:doctor` to validate the install.

---

## Phasing

The full v2 scope is split across three phases. See [`SKILLS.md`](./SKILLS.md) for which skills land in which phase.

- **Phase 1 — Foundation.** The substrate. Folders, examples, `specflow:upgrade`, `specflow:design`, the rules registry, the four principles, the adversarial review chain (Gates 1-3), trust-ladder primitives, feature brief composition (`specflow:brief`), `simplify` autoresearch loop, `SKILLS.md` glossary discipline.
- **Phase 2 — Development.** `specflow:develop` with green/yellow/red lane execution. Agent indexing, agent teams, specialised agent matching, Gates 4-5 of the adversarial chain.
- **Phase 3 — Memory.** Self-learning loop closes. `task-history.json` / `decision-log.md` populated; `/optimize` runs across the verifiable-skill set; rules registry self-evolves; Gate 6 of the adversarial chain.

---

## Environment requirements

- **Playwright CLI** — hard requirement at setup. Used by `specflow:design` (visual diff loop) and `specflow:test` (UI verification).
- **Codex CLI** — soft requirement. Enables deeper adversarial review across the pipeline (design, plan, code, tests). Skills degrade gracefully when missing.
- **Linear MCP** — soft requirement for `specflow:linear` task export and bidirectional sync.

The full environment inventory lives in `docs/specflow/admin/environment.json` per project, refreshed by `specflow:setup`, `specflow:upgrade`, and `specflow:doctor`.

---

## Contributing

Every skill MUST have an entry in [`SKILLS.md`](./SKILLS.md). New skills cannot ship without one — it's the discoverability contract.

Every skill that produces output MUST declare verify steps inline (Goal-Driven Execution, principle 4) and a binary `eval:` field in its frontmatter describing the success check.

Every breaking change to file layout, schema, or config keys MUST add a migration entry to [`MIGRATIONS.md`](./MIGRATIONS.md) — that's how `specflow:upgrade` stays correct.
