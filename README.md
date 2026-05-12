# Specflow

> Production-grade, self-adaptive product workflow: install once globally, every project accumulates its own constitution, profiles, agents, decisions, and task history — and uses that memory to make every subsequent piece of work smarter than the last.

**Current version:** `2.11.0` ([CHANGELOG](plugins/specflow/CHANGELOG.md))

---

## The pipeline

Specflow is a seven-skill assembly line from "I want to build X" to "PR opened, Verifier passed, history updated":

```
specflow:setup     → first-run substrate (admin/, agents/, rules/, config)
specflow:feature   → kickoff: NNN-slug allocation, folder scaffold, goal interview
specflow:prd       → 5-phase orchestrator: interview → PRD → Gate 2 → brief
specflow:task      → 5-phase orchestrator: tasks → cross-task review → Gate 3
specflow:sprint    → batch the next N tasks, assign teams, isolate worktree
specflow:develop   → 6-phase orchestrator: triage → Gate 4 → execute → Gate 5 → PR
specflow:test      → test plan grounded in PRD acceptance criteria
```

Every gate is a multi-agent debate manifest with a writer, principle reviewers, and an Orchestrator that owns the closing decision. Every artefact is a markdown file in your repo — reviewable before anything is pushed.

---

## What's new in v2.7

| Capability | What it does |
|---|---|
| `specflow:sprint` | New skill. Filters the tasks file to the in-scope batch (`maxIssuesPerSprint`, default 5), resolves per-stage team assignments, creates an isolated git worktree. Invoked by `develop` Phase A.5.5; refuses standalone invocation. |
| Idempotent worktree state machine | `sprint` Phase D.3 resolves six explicit states (reuse / mismatch-HALT / elsewhere-HALT / leftover-HALT / attach / fresh-create) before touching `git worktree add`. Re-running on a partial failure resumes cleanly or halts with explicit recovery. |
| `T_run` scope binding | `develop` only touches the tasks the sprint asked for, and nothing else. Persisted to `admin/scratch/{NNN-slug}-develop/t-run.json`. Resume logic loads `T_run` before evaluating any artefact. |
| Lessons registry | `prd` Phase A.3.5 queries `admin/lessons.json` for tags matching the new feature; surfaces relevant prior incidents into the interview. `develop` stays out of it (lessons influence the plan, not execution). |
| Cross-task reviewer | `task` Phase E.4.5 fires a two-lens review (coherence + better-arrangement) over the full task set before Gate 3 closes. |
| Edge-case reviewer | New principle reviewer added to Gate 4 + Gate 5 reviewer sets in `develop`. Five-question lens: collateral surface, failure modes, inheritance, interaction, state/environment. Deliberately not goal-aware. |
| TDD discipline | `develop` Phase D enforces Red → Green → Refactor for Yellow-lane tasks (and Green-lane when `tddRequired: true`, default). Refactor halts if the agent tries to add files. |
| Stage teams | Plan → Build → Test → Iterate → Validate as first-class doctrine. Default rosters in `templates/admin/stage-teams.md`; `config.json.teams.{stage}` overrides per-project. |
| Single-context-task budget | Per-task context budget self-checked at task time, pre-flighted at develop time. ≥20% divergence surfaces a 3-option developer prompt. No mid-task compaction. |

For the full v2.x history see [CHANGELOG.md](plugins/specflow/CHANGELOG.md) and [MIGRATIONS.md](plugins/specflow/MIGRATIONS.md).

---

## Install

```
/plugin marketplace add soletraderai/specflow
/plugin install specflow
```

Then in the project root:

```
/specflow:setup
```

Setup is first-run-only. Subsequent version-to-version migrations are handled by `/specflow:upgrade`.

---

## Skills

### Core pipeline

| Skill | Purpose |
|---|---|
| `/specflow:setup` | Initialise specflow: folder layout, environment inventory, profile interview, rules registry, standard agents. |
| `/specflow:prime` | Prime the codebase context for an upcoming piece of work. |
| `/specflow:prd` | 5-phase orchestrator: interview, PRD body, Gate 2 manifest, brief composition. |
| `/specflow:task` | 5-phase orchestrator: task synthesis, cross-task review (Phase E.4.5), Gate 3 manifest. |
| `/specflow:test` | Test plan grounded in PRD acceptance criteria; full / targeted / `--plan-only --task` / `--feedback` modes. |
| `/specflow:sprint` | Sub-skill of `develop`. Sprint planner with idempotent worktree creation. |
| `/specflow:develop` | 6-phase orchestrator: lane triage, Gate 4, execute (R→G→R), Gate 5, PR + Linear + history. |
| `/specflow:complete` | Retro at task completion: appends to `task-history.json`, elevates patterns to `decision-log.md`. |

### Utilities

| Skill | Purpose |
|---|---|
| `/specflow:linear` | Export tasks (and misc-tasks) to Linear with bidirectional sync. Soft requirement: Linear MCP installed. |
| `/specflow:scope-change` | Capture mid-development scope changes with audit trail; regenerate affected tasks. |
| `/specflow:misc` | Single-task workflow for bugs and small fixes that don't warrant a PRD. |
| `/specflow:decision` | Log a decision out-of-band into `admin/decision-log.md`. |
| `/specflow:design` | Generate browser-ready HTML mockups grounded in the live codebase; Playwright iteration loop. |
| `/specflow:brief` | Compose a self-contained, browser-readable feature brief. Auto-fires at PRD Phase E. |
| `/specflow:upgrade` | Refresh an aged specflow installation; drives migrations from `MIGRATIONS.md`. |

### Introspection + housekeeping

| Skill | Purpose |
|---|---|
| `/specflow:budget` | Subscription + per-skill context-window cost visibility. |
| `/specflow:doctor` | Health check on the project's specflow state. |
| `/specflow:insights` | Monthly cadence: surfaces recurring patterns from `task-history.json`; proposes rule promotions. |
| `/specflow:prune` | Quarterly pruning of stale rules, decisions, agent snapshots, history entries. |
| `/specflow:simplify` | Bounded autoresearch loop on the simplification task. |
| `/specflow:optimize` | Autoresearch loop generalising simplify's discipline across the verifiable-skill set. |
| `/specflow:feedback-loop-audit` | Audit the rate of feedback the codebase already provides; seeds `admin/CONTEXT.md`. |

### Trust-ladder primitives

| Skill | Purpose |
|---|---|
| `/specflow:grill` | Interrogate the user one question at a time. Sub-skill of PRD Phase B; manually invokable. |
| `/specflow:confidence-check` | Declare uncertainty in plain language before acting on a non-trivial decision. |
| `/specflow:panic` | The big red button. Captures a postmortem snapshot, then rewinds to the last clean checkpoint. |

---

## Project layout

After `specflow:setup`, every project carries:

```
docs/specflow/
├── admin/
│   ├── config.json              ← skill toggles, knobs, seeded defaults
│   ├── CONTEXT.md               ← project context summary
│   ├── environment.json         ← detected stack
│   ├── profiles.json            ← user/team profiles
│   ├── decision-log.md
│   ├── task-history.json
│   ├── lessons.json
│   ├── rules/
│   │   ├── non-negotiable.md
│   │   ├── guidelines.md
│   │   └── glossary.md
│   ├── agents/
│   │   ├── standard/lifecycle/      ← orchestrator, devils-advocate, verifier
│   │   ├── standard/principles/     ← 6 reviewers (4 classic + cross-task + edge-case)
│   │   └── specialised/             ← stack-relevant agents
│   ├── debate-log/
│   └── scratch/                     ← per-skill working state
├── features/
│   └── {NNN-slug}/
│       ├── {NNN-slug}-prd.md
│       ├── {NNN-slug}-interview.md
│       ├── {NNN-slug}-tasks.md
│       ├── {NNN-slug}-test.md
│       ├── {NNN-slug}-brief.html
│       └── debate-log/
│           ├── prd-gate2/
│           ├── tasks-gate3/
│           ├── sprint-plan/
│           ├── develop-gate4/
│           └── develop-gate5/
├── misc-task/
│   └── 000-tasks-misc-tasks.md
└── docs/
```

Each step reads from the previous step's output. You always have a reviewable artefact before anything ships.

---

## Reference

- [`plugins/specflow/CHANGELOG.md`](plugins/specflow/CHANGELOG.md) — release notes
- [`plugins/specflow/MIGRATIONS.md`](plugins/specflow/MIGRATIONS.md) — version-to-version migrations
- [`plugins/specflow/templates/admin/`](plugins/specflow/templates/admin/) — doctrine docs (chain-don't-absorb pattern)
- [`plugins/specflow/templates/orchestrator-pattern.md`](plugins/specflow/templates/orchestrator-pattern.md) — sub-skill fork conventions

## License

[MIT](LICENSE)
