# Specflow PRD: Production-Grade, Self-Adaptive Plugin

A living PRD for evolving specflow from its current shape into a production-grade plugin that adapts independently to every project it's installed in, and learns from each task it helps deliver. Reference implementation: ClaimXPro (`/Users/marepomana/Web/ClaimXPro/docs/specflow/`).

This document drives the next two phases of work. Update as decisions are made or scope shifts.

---

## Vision

specflow is installed once globally and behaves like a different plugin in every repo it touches. Each project accumulates its own constitution, profiles, agents, decisions, and task history — and uses that memory to make every subsequent piece of work smarter than the last.

Shorthand: **global plugin, local memory, adaptive output.**

## Problem

Today, specflow is a templated workflow plugin: it scaffolds folders, drives interviews, and produces PRDs/tasks/tests. It doesn't *remember* anything. There's no per-project memory loop, no visibility into which agents a project has access to, no way to refresh an aged installation without nuking customisation, and no orchestration layer between "task created" and "test written." A project that ran specflow six months ago is no smarter today than the day it started.

## Goals

1. **Per-repo adaptivity.** Plugin is global; every project's state, memory, and behavior lives in that repo and stays in that repo.
2. **Self-learning memory.** Every completed task feeds a decision log + task history that future tasks read from.
3. **Agent visibility.** A browsable, indexed per-repo agent registry — standard agents always present, specialised agents matched to the stack.
4. **Adaptive workflow.** Existing skills (PRD, task, test) and new ones (develop) consult the project's memory and surface relevant past lessons.
5. **Refreshable.** A project can be brought up-to-date with the current plugin without losing customisation.
6. **Self-documenting.** A complete worked example ships with the plugin so users see the full shape before committing.
7. **Environment-aware.** specflow inventories what's available in the user's environment (CLIs, MCPs, plugins, agents) and feeds that knowledge to skills so outputs match what the system can actually do.

## Non-goals

- A SaaS or hosted product. specflow stays a local Claude Code plugin.
- Cross-project knowledge sharing. Each repo learns in isolation — no central memory.
- Replacing Linear, GitHub, or any external tool. specflow integrates; it doesn't compete.
- Pixel-perfect design tooling. Mockups are alignment artefacts, not deliverables.
- **Team-to-team / human-to-human handoffs.** Handled out-of-band (Loom recordings + messaging). specflow doesn't replace human communication.
- **Dashboards.** Metrics are captured for the system to consume; we don't ship UI for humans to browse them.

## Behavioral principles (non-negotiable, embedded in every skill)

These are loaded into the system prompt of every specflow skill. Adopted from `forrestchang/andrej-karpathy-skills` with light edits.

1. **Think before coding.** Don't assume. Don't hide confusion. Surface tradeoffs. State assumptions explicitly. Present multiple interpretations when ambiguity exists. Push back when a simpler approach exists. Stop when confused — name what's unclear.
2. **Simplicity first.** Minimum code that solves the problem. Nothing speculative. No features beyond what was asked. No abstractions for single-use code. No flexibility or configurability that wasn't requested. No error handling for impossible scenarios. If 200 lines could be 50, rewrite it. The test: would a senior engineer say this is overcomplicated?
3. **Surgical changes.** Touch only what you must. Don't improve adjacent code, comments, or formatting. Don't refactor what isn't broken. Match existing style. Every changed line must trace directly to the user's request. **Quality nuance:** if you spot something wrong outside the work item's scope, do not fix it inline — auto-create a `misc-task` capturing the observation *and the why*, so quality concerns don't fall through cracks while the change set stays clean.
4. **Goal-driven execution.** Every skill that produces output declares verify steps inline (`1. Step → verify: check`). Loop until verified. Strong success criteria let the LLM iterate independently; weak ones force constant clarification.

## Architectural principles (for the plugin itself)

- **Global plugin, local state.** Code in `~/.claude/plugins/specflow/`; everything else in `docs/specflow/admin/` per repo, committed.
- **Snapshots over references.** Agents and definitions are pinned in-repo so they're reviewable and don't drift when upstream changes.
- **Observable memory.** Decision log and task history are markdown/JSON in the repo — diffable, greppable, human-readable.
- **Convention over configuration.** Folder layout is the contract; users shouldn't have to wire things together by hand.
- **Context-conscious.** The plugin inventories what's available, but exposes only the relevant slice to each skill — better outputs without bloating the context window.
- **Testing is a cadence, not a terminal step.** Every skill that produces output runs verification inline, not at the end. Playwright loops on every UI change, even one-line ones. Iteration is the default, not the exception.
- **The metric is a signal, not a goal.** Team judgment is ground truth. If a metric improves but the work feels worse, the metric is wrong. (Goodharting guard for the autoresearch loops — keep benchmarks as low as possible so improvements only show when they're real.)

---

## Reference: target structure

```
docs/specflow/
├── admin/                         # 🆕 plugin configuration + memory (everything specflow needs to operate)
│   ├── config.json                # 🔧 moved from docs/specflow/ root
│   ├── pages.json                 # 🔧 moved from docs/specflow/ root
│   ├── profiles.json              # 🆕 user personas
│   ├── decision-log.md            # 🆕 key decisions, what worked, what didn't
│   ├── task-history.json          # 🆕 AI completion outcomes feeding the self-learning loop
│   ├── environment.json           # 🆕 inventory of CLIs, MCPs, plugins, agents available here
│   ├── rules/                     # 🆕 project rules registry (drives misc-task creation on violation)
│   │   ├── non-negotiable.md      # always-on hard rules (e.g. no hardcoded values, no comments unless WHY)
│   │   ├── guidelines.md          # soft rules / project taste — adaptive
│   │   └── glossary.md            # rule descriptions + why each exists
│   ├── CONTEXT.md                 # 🆕 slim live context doc — how this project works, updated by human or AI
│   └── agents/                    # 🆕 per-repo agent registry
│       ├── standard/              # always present, two categories:
│       │   ├── lifecycle/         #   plan / challenge / confirm — Orchestrator, Devil's Advocate, Verifier
│       │   └── principles/        #   one reviewer per core principle — Simplicity, Surgical, Think-Before-Coding, Goal-Driven
│       ├── specialised/           # matched to the project's stack
│       └── index.json             # snapshot index
├── features/                      # 🔧 self-contained per-feature workspaces (replaces flat prd/ + task/ + test/)
│   └── NNN-{slug}/                # one directory per feature — everything related to the feature lives here
│       ├── NNN-{slug}-interview.md  # 🆕 PRD interview transcript — original request + grilling Q&A + resolved assumptions
│       ├── NNN-{slug}-prd.md        # the PRD body, synthesised from the interview
│       ├── NNN-{slug}-prd.html      # 🆕 static HTML rendering of the PRD; header links to the interview
│       ├── NNN-{slug}-tasks.md      # the task list
│       ├── NNN-{slug}-test.md       # the test plan
│       ├── design/                  # 🆕 feature-specific HTML/CSS mockups (current + proposed)
│       │   ├── NNN-{slug}-current.html
│       │   ├── NNN-{slug}-proposed.html
│       │   └── NNN-{slug}-iteration-log.md
│       ├── docs/                    # 🆕 feature-specific research, meeting notes, source material
│       ├── assets/                  # screenshots, diagrams (test artefacts + design captures)
│       └── debate-log/              # 🆕 multi-agent debate manifests for every gate that fired on this feature
│           ├── prd-gate2/           #   ↳ PRD vs interview review
│           ├── tasks-gate3/         #   ↳ tasks vs PRD review
│           ├── develop-gate4/       #   ↳ plan vs tasks + PRD anchor (Phase 2)
│           └── develop-gate5/       #   ↳ code vs plan, cross-provider Codex (Phase 2)
├── misc-task/                     # 🆕 single-task bug/fix workflow — cross-feature, stays at root
│   ├── 000-tasks-misc-tasks.md
│   └── assets/
└── docs/                          # 🆕 cross-feature research that doesn't belong to a single feature
    └── {topic}.md
```

Status legend: ✅ shipped · 🆕 to add · 🔧 to extend or migrate

Migration note: `config.json` and `pages.json` move from `docs/specflow/` root into `admin/`. The flat `prd/`, `task/`, and `test/` folders consolidate into `features/NNN-{slug}/` directories so each feature is a self-contained workspace. **Debate logs (multi-agent gate manifests) live inside the feature folder** at `features/NNN-{slug}/debate-log/{gate}/` — co-located with the feature so reviewers and downstream agents can read prior gate context without chasing files across `admin/`. Cross-feature gates (misc-task review, `/optimize` runs) stay under `admin/debate-log/` only when there's no feature folder to home them in. All breaking changes handled by `specflow:upgrade`, which ships in Phase 1 alongside the migration so existing installs have a clean path forward.

---

## The rules registry

A new first-class concept. Every project has a small registry of rules the AI must respect. Detection of violations triggers automatic `misc-task` creation — the AI captures the observation and the *why* without fixing it inline (preserves Surgical Changes).

Two tiers:

- **Non-negotiable** (`admin/rules/non-negotiable.md`) — hard rules. Examples: no hardcoded values unless absolutely necessary; no comments unless a non-obvious WHY; never bypass auth checks; protected paths require Red-lane (human-led) treatment. Always-on, project-tuned at setup, additive over time.
- **Guidelines** (`admin/rules/guidelines.md`) — soft rules / project taste. Examples: prefer composition over inheritance in this codebase; tests sit alongside code in `__tests__`. Adaptive; the system can suggest additions based on observed patterns.
- **Glossary** (`admin/rules/glossary.md`) — every rule listed with a one-line description and the *why* (why we have it). Self-documenting; reviewable in PRs as the rule set evolves.

The four behavioral principles are rules-of-rules — they govern how the AI behaves. The registry is rules-of-the-project — they govern what the AI accepts in this codebase. Both ship to every skill's system prompt.

When a rule is violated, the AI:
1. Does not fix inline (surgical).
2. Auto-creates a `misc-task` with the rule reference, the location, and the *why*.
3. Continues with the original work item.

The registry self-evolves in Phase 3: repeated violations of the same shape get promoted from observation → guideline → non-negotiable, with human sign-off at each promotion.

---

## Approach: three phases

The work is split into three phases. Phase 1 is the foundation — folders, examples, the upgrade tool, the design tool, the rules registry, the four behavioral principles, the adversarial review chain, and the trust-ladder primitives the team needs before parallelism is safe. Phase 2 is the develop layer — `specflow:develop` orchestrates green/yellow/red lane execution against generated tasks. Phase 3 closes the loop — self-learning memory, autoresearch on verifiable skills only, parallel infrastructure, rule registry self-evolution.

The throughline: **survive Tuesday before optimising the quarter.** Recovery primitives, alignment gates, and the rules substrate ship in Phase 1 — before anything compounds on top of them.

### Phase 1 — Foundation

**Goal:** ship the substrate every later phase depends on.

**Scope (priority order):**

1. **Worked example tree + README.** Full ClaimXPro shape under `plugins/specflow/examples/docs/specflow/`, sanitised. README written as a user guide covering folder roles, per-repo isolation, environment requirements, and the four behavioral principles.
2. **`specflow:setup` extension.** Creates the new folders (`admin/`, `admin/agents/{standard,specialised}/`, `admin/rules/`, `features/`, `misc-task/`, `docs/`). Per-feature folders (`features/NNN-{slug}/{design,docs,assets}/`) are created on demand by `specflow:prd` when a new feature is initialised, not up-front. Hard-checks Playwright. Detects optional tools (Codex, MCPs). Writes `admin/environment.json`. Runs `profiles.json` interview. Seeds `admin/rules/non-negotiable.md` with a starter set; user accepts/edits. Copies `admin/CONTEXT.md` template.
3. **Migration + `specflow:upgrade`.** `config.json` + `pages.json` move into `admin/`; flat `prd/` + `task/` + `test/` consolidate into `features/NNN-{slug}/` directories (one feature per folder, files renamed to `prd.md` / `tasks.md` / `test.md`). Both ship in Phase 1 alongside `specflow:upgrade` (version-aware via `specflowVersion` + `MIGRATIONS.md`). Existing installs run `/specflow:upgrade` once after pulling.
4. **Behavioral principles + rules registry shipped to every skill.** `plugins/specflow/CORE_PRINCIPLES.md` (the four) + `admin/rules/non-negotiable.md` + `admin/rules/guidelines.md` are loaded into every skill's system prompt. Skill template includes the `MUST be loaded` reference.
5. **`/grill` skill (sub-skill of `specflow:prd`).** Standalone skill that AI uses to interrogate the user one question at a time, recommending an answer per question with reasoning grounded in the project's prior context (decision-log, CONTEXT.md, environment.json, prior PRDs) **and the confirmed Goal section of the interview file**. **Re-evaluates what to ask next after each answer.** Appends each round to `features/NNN-{slug}/NNN-{slug}-interview.md`. Iterates until the user signs off on alignment (no fixed cap). Cannot run until the interview's Goal section is confirmed. Can also be invoked directly to extend or re-grill an existing interview file.
6. **`specflow:prd` as the user-facing entry point.** Multi-phase skill that owns the full PRD-creation flow:
   - *Phase A — preamble + goal confirmation.* Creates the feature folder if it doesn't exist. Writes the interview file's preamble (original request + codebase context). **Then articulates a goal — Outcome / Audience / Success-looks-like / Driving value / Out-of-scope-at-goal-level — grounded in codebase context, decision-log, and CONTEXT.md, and asks the user to confirm, edit, or replace.** Once the user confirms, the Goal section is written to the interview. Grilling does NOT start until the goal is confirmed. The confirmed goal is the precedent every downstream artefact anchors to.
   - *Phase B — grilling.* Invokes `/grill` as a sub-skill, which appends Q&A rounds to the same interview file until the user signs off. Every recommended answer grounds its reasoning in the confirmed goal where applicable.
   - *Phase C — synthesis.* Generates `NNN-{slug}-prd.md` from the resolved assumptions in the interview. The PRD body's *Vision* section synthesises directly from the goal (it does not re-derive it from the Q&A). Other PRD sections (Problem, Goals, Non-goals, Users, Requirements, AC) draw from the resolved assumptions in the rounds. PRD references the interview by relative path; does NOT duplicate the Q&A.
   - *Phase D — render.* Auto-invokes `specflow:render` to produce `NNN-{slug}-prd.html`. The rendered HTML's header includes a link to `NNN-{slug}-interview.md` so reviewers can open both side-by-side. The interview itself is not HTML-rendered (markdown only).
   - *Phase E — Gate 2.* Multi-agent debate manifest into `features/NNN-{slug}/debate-log/prd-gate2/`. Reviewers read the goal first, then the PRD, then the rounds. Findings cite the goal as the primary lens — any PRD requirement that contradicts or fails to serve the goal is a Think-Before-Coding finding.
6a. **PRD static HTML rendering (`specflow:render`).** Every PRD is rendered to a self-contained `NNN-{slug}-prd.html` inside its feature folder so reviewers can scan it in a browser instead of fighting markdown soup. Auto-fires whenever `specflow:prd` writes or updates the PRD markdown; also exposed as `/specflow:render {feature-id}` for manual re-render. Inline CSS, no JS dependencies, opens directly. The rendered HTML's header links to the sibling interview file. Detail in Appendix P.
7. **`specflow:task` enhancement: coverage matrix + intent summary in chat.** When tasks are generated:
   - Adversarial pre-review fires automatically (gate 3 in the chain — see Appendix N).
   - Coverage matrix produced: PRD requirements → tasks satisfying them.
   - For 3-5 highlighted tasks, AI surfaces a 2-sentence non-technical intent summary in chat (not as a file artefact). Docs creator reviews matrix + samples; signs off once.
   - Override capture: every recut/correction logs to `task-history.json` for self-learning (consumed in Phase 3).
8. **Adversarial review chain — gates 1, 2, 3 land here.** Gate 1 = `/grill` (interrogating the human; runs as Phase B of `specflow:prd`). Gate 2 = PRD vs interview (auto-fires after `specflow:prd` synthesises the body). Gate 3 = tasks vs PRD (auto-fires on `specflow:task`). Each gate runs the **multi-agent debate manifest** (Appendix N): N principle-aligned reviewers fire findings in parallel into the gate's manifest; AI responds round 2; reviewers sharpen or accept round 3; Orchestrator writes the closing decision entry. **Manifests live inside the feature folder** at `features/NNN-{slug}/debate-log/{gate}/` so all context for a feature is co-located.
9. **Standard agents shipped — two categories.** **Lifecycle agents** (Orchestrator, Devil's Advocate, Verifier — copied to `admin/agents/standard/lifecycle/`) cover the plan / challenge / confirm moments. **Principle reviewers** (one per core principle, copied to `admin/agents/standard/principles/`) — Simplicity Reviewer, Surgical Reviewer, Think-Before-Coding Reviewer, Goal-Driven Reviewer — fire as parallel reviewers in the debate manifest. Lifecycle set deliberately small (3); principle set scales 1:1 with `CORE_PRINCIPLES.md`. No indexing logic yet.
10. **Trust-ladder primitives.** `confidence-check` (AI declares uncertainty in plain language before acting); `panic` (rewinds in-flight changes, kills background agents, snapshots state); `getting-started` profile (slim onboarding for AI-novice devs day 1).
11. **`specflow:design`.** Web/mobile detection, codebase-truth principle, Playwright iteration loop, optional Codex adversarial review. Detail in Appendix C.
12. **Path-scoped rules infrastructure.** `paths:`-frontmatter rules in `admin/rules/` so context auto-loads only when matching files are touched. CLAUDE.md cap at 500-700 lines. AGENTS.md auto-mirror commit hook (cheap multi-provider insurance).
13. **`/budget` skill + binary acceptance eval block in every SKILL.md.** Budget skill = visibility on subscription consumption. Binary eval block = mandatory `eval:` field per skill describing the success check. Cultural artefact (Saraev's discipline-installer — installs the metric/judge habit even before any optimiser exists).
14. **`SKILLS.md` glossary.** Plugin-side: every skill listed with one-sentence purpose, when it triggers, what it produces, what it requires. Same for standard agents. New skills can't ship without an entry.
15. **Testing as cadence.** Every skill that produces output declares verify steps inline. Playwright loops on every UI change, even one-line ones. No skill ships a "test" phase at the end — verification happens after every step.
16. **One bounded autoresearch loop on `simplify`.** Karpathy-style discipline-installer. Read-only eval surface (lines deleted + tests + lints + types), branch-per-run, $20/week budget, merge after human review. The deliverable is the *template* future skills inherit, not the optimised prompt.
17. **`feedback-loop-audit` + `CONTEXT.md` glossary skill.** Audits the rate of feedback (test coverage, type strictness, e2e health). Generates the slim `admin/CONTEXT.md` from the codebase as a starting point for human/AI to maintain as a live document.

**Phase 1 environment requirements:**
- Playwright CLI — hard requirement.
- Codex CLI — soft requirement; enables deeper adversarial review.

**Items NOT in Phase 1 (deferred deliberately):**
- `specflow:develop`, agent indexing, agent teams → Phase 2.
- Self-learning memory loop, `/optimize` across multiple skills, worktree parallelism → Phase 3.

### Phase 2 — Development

**Goal:** ship the implementation orchestration layer. Build `specflow:develop` and the agent layer it depends on. This is where specflow gains its hands.

**Scope:**

1. **`specflow:develop` with green/yellow/red lane execution.** For each task generated from a PRD:
   - Lane assigned by triage tuple: verifiability + blast radius + dependency state + confidentiality classification.
   - **Green** (verifiable + low blast + non-confidential): batched, AFK-eligible, single batched human sign-off per batch. Machine checks pass before any PR opens.
   - **Yellow** (one axis weak): HITL — agent + human paired in real time, one at a time.
   - **Red** (high blast OR low verifiability OR confidential): human-led; AI assists on bounded subtasks only.
   - **Confidentiality classification is rule-based** (path globs in `config.json`), not AI-rated. No issue escapes Red because the AI feels confident.
   - Human reviews the DAG once (one sign-off on the plan), not every issue individually.
   - Initial target lane ratio: 60/30/10 (G/Y/R). If 30/40/30, the PRD needs to be re-cut into smaller, more isolated issues (this is itself a learning signal, not a failure).
2. **Agent indexing + snapshot mechanism.** Scan installed plugins; build the global index referenced by `admin/environment.json`. Pin agent definitions into `admin/agents/specialised/` with snapshots. Refresh logic in `specflow:upgrade`.
3. **`specflow:agent` skill.** `add`, `remove`, `list`, `refresh`. Detail in Appendix K5.
4. **Agent teams composition.** `config.json` `teams` block; team-spawn via `specflow:develop`. Soft dependency on the agent-teams plugin; degrades to sequential agent invocation if absent.
5. **Specialised agent matching to stack.** Runs during setup tech detection AND on upgrade.
6. **Adversarial review chain — gates 4 and 5.** Gate 4 = plan vs tasks + PRD anchor (during develop). Gate 5 = code vs plan (cross-provider Codex review). Each runs the 3-iteration debate loop.
7. **Dev plan ↔ PRD anchor.** Every plan starts with: *"We're doing X because of PRD requirement Y. This aligns with Z."* Then the technical plan. Devs get PRD context for free at plan time, without reading the whole PRD.
8. **Rule violation auto-flagging.** When `specflow:develop` spots a rule registry violation outside scope, it auto-creates a `misc-task` with the rule reference, location, and *why*. Surgical Changes preserved; nothing falls through cracks.
9. **`specflow:doctor` (full).** Validates installation, environment, agent index integrity. On-demand re-detection.

### Phase 3 — Memory and self-learning

**Goal:** close the loop. Every completed task feeds memory; every new task reads from it. Compounding begins.

**Scope:**

1. **Memory schemas live.** `decision-log.md`, `task-history.json` populated throughout the lifecycle. Tracking captured at four sources:
   - Decisions (every `/grill` round, PRD edit, scope change) → `decision-log.md`
   - Tasks (intent summary, PRD anchor, G/Y/R class, files touched) → `task-history.json`
   - Development (AI assistance level, time, what worked, what didn't, blast-radius outcome) → same record
   - Testing (tests added, manual QA notes, regressions caught, escaped issues) → same record
2. **`specflow:complete` retro skill.** Captures outcome at task completion. Writes to `task-history.json` and (when significant) `decision-log.md`.
3. **`specflow:decision` skill.** Lightweight; user logs a decision out-of-band.
4. **`specflow:scope-change` skill.** When intent changes mid-development: capture the *why*, update PRD, regenerate affected tasks, flag impacts on in-flight work. Prevents scope drift from going undocumented.
5. **Similarity search.** Wired into `specflow:task` and `specflow:misc` — surfaces relevant past tasks during creation. Simple tag/keyword match initially.
6. **Profile consumption.** `specflow:prd` actor probe reads `profiles.json`. `docs/` folder integrated into PRD problem discovery.
7. **`/optimize` on the verifiable skills only.** Six initial targets: `release-version-check`, `simplify`, `format`, `tdd-cadence`, `init`, `feedback-loop-audit`. ~$60 total via overnight GH Actions. Skills with taste-driven output (grill, plan, design, QA-with-judgment) explicitly excluded — autoresearch would reward-hack the LLM judge.
8. **Rules registry self-evolution.** Repeated misc-tasks of the same shape get promoted: observation → guideline → non-negotiable, with human sign-off at each promotion.
9. **Adversarial review chain — gate 6.** Tests vs requirements coverage.
10. **`/insights` (monthly) + `/prune` (quarterly).** Surface recurring patterns from `task-history.json`; prune stale rules and decisions.
11. **Compounding signals captured (no dashboard).** Five numbers per PRD logged to a rolling `metrics.md`: override rate, green-lane percentage, escaped-bug rate, cycle time, adversarial flag rate. The system reads them; humans look only when something is wrong.

**Items still deferred (gated on triggers, not phases):**
- Worktree `w` script + DB-branching: gated on real reviewer pile-up AND `panic` having shipped.
- Cross-provider Codex review on every code change: gated on team size ≥ 3 AND a real outage costing > 4 hours.
- Auto-research loop across the whole catalog: revisit when there are months of task history to learn from.

---

## Resolved decisions

Snapshot of decisions locked in.

- **Phase model** — three phases: Foundation, Development, Memory.
- **Migration timing** — `config.json` + `pages.json` move into `admin/` in Phase 1 alongside `specflow:upgrade`. The flat `prd/` + `task/` + `test/` folders consolidate into `features/NNN-{slug}/` in the same migration.
- **Feature-grouped layout** — every feature is a self-contained directory: PRD, tasks, tests, design, docs, and assets all live under `features/NNN-{slug}/`. Replaces the flat `prd/` + `task/` + `test/` layout. Cross-feature material (`misc-task/`, root `docs/`) stays at `docs/specflow/` root.
- **PRD static HTML rendering** — every `prd.md` gets a sibling `prd.html` for browser-based review. Auto-rendered when the PRD is written or updated; exposed manually via `/specflow:render`. Inline CSS, no JS, self-contained.
- **Standard agents — two categories.** **Lifecycle:** Orchestrator + Devil's Advocate + Verifier (deliberately small, expands only when a clearly distinct lifecycle moment emerges). **Principles:** one reviewer per core principle (Simplicity, Surgical, Think-Before-Coding, Goal-Driven), scales 1:1 with `CORE_PRINCIPLES.md`. Both categories ship with the plugin and copy into every project at setup.
- **Multi-agent debate manifest.** Every adversarial-review gate runs N reviewers in parallel (lifecycle DA + principle reviewers + Codex when available) into a shared manifest file. AI responds in round 2; reviewers sharpen or accept in round 3; **Orchestrator writes the closing decision entry**. Replaces the prior single-reviewer 3-iteration loop. Manifests live inside the feature folder (`features/NNN-{slug}/debate-log/{gate}/`) — co-located so all context for a feature is in one place. Detail in Appendix N.
- **PRD interview file.** Every PRD has a sibling `NNN-{slug}-interview.md` capturing original request, codebase findings, grilling Q&A with AI's reasoning per answer, and resolved assumptions. Written by `/grill` as a sub-skill of `specflow:prd`. The PRD body references the interview but does not duplicate it. Markdown only — no HTML render (the PRD's HTML render links to the interview file). Detail in Appendix Q.
- **Naming convention preserved.** Top-level feature files use `NNN-{slug}-` prefix on every filename (`NNN-{slug}-prd.md`, `NNN-{slug}-tasks.md`, etc.) so multiple PRDs are distinguishable when files are open in tabs or surface in search. Folder-level uniqueness alone isn't enough — file-level uniqueness matters for editor UX. Nested files inside `debate-log/`, `assets/`, and `docs/` skip the prefix (their parent path provides scope).
- **Behavioral principles** — Forrest/Karpathy four (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution) adopted as non-negotiable, embedded in every skill's system prompt.
- **Rules registry** — separate first-class concept. Two tiers (non-negotiable + guidelines) plus glossary. Drives auto-`misc-task` creation on violation. Self-evolves in Phase 3.
- **Adversarial review chain** — six gates phased across Phase 1-3. Each gate runs a **3-iteration debate loop**: AI defends or revises; reviewer counters; up to 3 rounds; Claude makes the final call (it's the writer/developer; it owns the decision). Detail in Appendix N.
- **Testing as cadence, not terminal** — every skill that produces output runs verification inline. Playwright loops on every UI change. No separate end-of-pipeline test phase.
- **Goodharting principle encoded** — the metric is a signal, not a goal. Benchmarks kept as low as possible so improvements only show when real.
- **Phase 1 includes design, upgrade, `/grill`, `/budget`, `panic`, `confidence-check`** — full alignment + trust-ladder primitives ship before parallelism.
- **Existing skill changes in Phase 1** — mechanical path updates only. Deeper rework reserved for Phase 2/3.
- **PIV survives as convention enforced inside skills, not as YAML DSL.** Frontmatter Claude Code already parses; no new schema.
- **One bounded autoresearch loop on `simplify` lands in Phase 1** as discipline-installer for the loop pattern; `/optimize` across the verifiable-skill set is Phase 3.
- **Coverage matrix + intent summary delivered in chat at task creation** (not as a file artefact).
- **Skills glossary required.** `SKILLS.md` ships with the plugin; every skill has an entry. Same for standard agents.
- **Profiles** — schema + setup interview in Phase 1; consumption by `specflow:prd` in Phase 3.
- **Environment inventory** — first-class. `admin/environment.json` generated at setup, refreshed by upgrade and doctor.
- **Playwright CLI** — hard requirement at setup. **Codex CLI** — soft requirement; enables deeper adversarial review.
- **Codex usage scope** — adversarial review backend across the whole pipeline (design Phase 1, plan/code Phase 2, tests Phase 3).
- **Auto-research loop across the catalog** — explicitly deferred. Out of scope for these three phases.
- **Human-to-human handoffs** — not specflow's problem. Handled out-of-band (Loom + messaging).
- **Dashboards** — not shipped. Metrics captured for system consumption only.

### Decisions added during the v2.x ship cycle

These were locked in during the 2.0 → 2.2 ship cycle (2026-05-06). Each carries the date applied for traceability.

- **Brief skill replaces render skill** (2026-05-06, v2.2.0) — `specflow:render` removed; `specflow:brief` composes `{NNN-slug}-brief.html` from PRD + interview + (optional) Gate 2/3 manifests with sidebar TOC + Visual abstract section. Structured-block vocabulary (`:::flow`, `:::comparison`, `:::scope`, `:::tree`) renders deterministically. `specflow:prd` Phase E invokes brief; `specflow:doctor`'s `html_drift` check is now `brief_drift`. Migration path documented at MIGRATIONS v2.1 → v2.2.
- **Gate 2 status taxonomy extended** (2026-05-06, v2.1.0 — E3 from 002-develop-skill dogfood) — `passed | passed-with-revisions | passed-with-escalations | failed`. `passed-with-revisions` distinguishes a clean pass from one where `block` findings landed and were resolved with PRD edits. Applies uniformly across Gates 2-5.
- **Vision verbatim-vs-paraphrase contract** (2026-05-06, v2.1.0 — E1) — Vision incorporates the Goal Outcome's load-bearing phrases verbatim where possible; paraphrase only for prose flow. Gate 2 reviewers check Vision-to-Goal trace integrity.
- **Phase C.3 PRD self-check cross-references Phase 1 skill schemas** (2026-05-06, v2.1.0 — E2) — every AC that names another specflow skill is verified against that skill's documented payload schema during PRD synthesis (catches schema-dependency smuggling at PRD time, not at Gate 2).
- **Goal-Driven Reviewer's reverse-traceability lens** (2026-05-06, v2.1.0 — E4 + 2.2.0 — E9) — the orphan-AC pattern (an AC verifying a contract no R establishes) is named explicitly. E9 extends the lens to code-review gates: an orphan phase, function, or behavioural surface in the produced artefact that doesn't trace to any R or AC is itself a coverage gap.
- **Block-finding resolutions surface in task synthesis** (2026-05-06, v2.1.0 — E5) — `specflow:task` Phase A.2 reads the Gate 2 manifest's "PRD revisions applied" section and treats each revision as a load-bearing constraint tasks must respect (e.g. R5 + R5.1 must remain separate tasks if the Gate 2 process deliberately separated them).
- **Mechanical pre-Gate-4 lane recheck (B.1)** (2026-05-06, v2.1.0) — `specflow:develop` Phase B.1 fires unconditionally for every task before Gate 4. Three checks: file-count plan-vs-scope ratio (>1.5x downgrades blast radius); module count plan-vs-scope (new modules downgrade); confidential-path glob match (matches force lane to red). Mechanical, rule-based — never reviewer-judgement-driven. Aggregate outcome at `lane-assignments.json.b1_recheck` includes `batch_shape_at_default_cap` (E7).
- **Codex as the sixth Gate 5 reviewer** (2026-05-06, v2.2.0) — when `cli.codex.available: true`, Gate 5 fires Codex alongside the standard five reviewers. Codex's correctness lens is independent of (not duplicative with) Goal-Driven's reverse-traceability lens (E6 lens-overlap note). When Codex is absent, the manifest header records `codex: unavailable` and the gate proceeds with five reviewers.
- **Conditional-pass escalation contract** (2026-05-06, v2.2.0 — E10) — `specflow:develop` Phase F supports `pass | conditional-pass | reject` outcomes from the Verifier. Conditional-pass surfaces a two-option user prompt: accept-and-proceed with documented condition vs defer with `specflow:misc --auto` follow-up. No third "force-pass" option.
- **`specflow:complete` retro skill** (2026-05-06, v2.1.0 byproduct of 003 dogfood) — Phase 3 retro skill produced as the lane-execution output of running `specflow:develop` on its own PRD. Eight phases A-H: pre-flight + lock; idempotency check; interactive Q&A or auto-mode synthesis; significance elevation evaluation with triple-flag tracking; append-only `task-history.json` write; optional `decision-log.md` elevation via `specflow:decision`'s mirror schema; Linear status sync; lock release. Auto-fired by `specflow:develop` Phase F when a task closes; manually invoked via `/specflow:complete {task-id}`.
- **30-min stale-lock heuristic for `specflow:complete`** (2026-05-06, v2.2.0 — E8) — `specflow:complete` Phase A.3 treats a lock file older than 30 minutes as stale and reclaims it. The 30-minute threshold is hard-coded in v1; v2 candidate is to surface as `config.json.complete.staleLockMinutes`. Tracked for promotion to MIGRATIONS when consumed.
- **PRD-anchor format on every plan** (2026-05-06, v2.1.0) — plans produced by `specflow:develop` MUST start with `"We're doing X because of PRD requirement Y. This aligns with goal field Z."`. Enforced by Gate 4 reviewers via the reverse-traceability lens.
- **Six structured mutation operators for `/optimize`** (2026-05-06, v2.3.0-staged) — `tighten`, `consolidate`, `clarify`, `deduplicate`, `reorder`, `split-by-phase`. Frontmatter is immutable across variants. No LLM-as-judge inside the loop — variants ranked only by the target's binary machine eval. Three independent auto-merge guardrails: HTML comment in PR description, no `--auto` call in CI, GH Action human-actor check.
- **Two-pass deterministic clustering for `/insights`** (2026-05-06, v2.3.0-staged) — Pass 1: field-shape exact-match grouping; Pass 2: token-frequency n-grams with case-folding + Unicode NFC normalisation + fixed stop-word list, no stemming. `semantic` cluster-source label reserved for v2 embedding-clustering. Promotion threshold: `len(unique_contributing_ids) >= 3`.
- **Per-surface staleness boundaries for `/prune`** (2026-05-06, v2.3.0-staged) — decision-log: age > 4Q AND no reference in 2Q. Non-negotiable rules: superseded-citation only. Guidelines: superseded-citation OR zero references in 4Q. Agent snapshots: persistent orphan/drift across two consecutive runs. Task-history: age > 4Q AND `superseded_by_retro: true` AND no addenda in 2Q. Archive append-only; skill never modifies its own archive. Byte-identical round-trip restoration is the binary eval property.
- **Frontmatter shape across SKILL.md** (2026-05-06, v2.3.0-staged) — every SKILL.md carries `name`, `description`, `status`, `phase`, `requires`, `produces`, `eval`. `status:` ∈ `shipped | v2-enhancement | v2-new`; `phase:` ∈ `1 | 2 | 3`. Bare-name skills (`/X` style — `panic`, `simplify`, `confidence-check`, `feedback-loop-audit`, `grill`, `optimize`, `prune`, `insights`) use bare names in the frontmatter; `specflow:X` skills use the prefixed form.
- **Six-skill verifiable-skill set for `/optimize`** (2026-05-06, v2.3.0) — initial targets: `release-version-check`, `simplify`, `format`, `tdd-cadence`, `init`, `feedback-loop-audit`. Per-target weekly budget cap default $10 (configurable via `config.json.optimize.targetCapUsd`); aggregate envelope $60/week implicit. Decline-streak governance: 7-day operator-avoid window AND 30-day target-skip window after consecutive `merge_decision: closed-without-merge` outcomes (windows hardcoded; not config knobs). Budget override path: `--override-budget {reason}` extends the target's cap for one run; decline-streak has no override flag — manual invocation simply proceeds with the chat-line warning.

## Open questions

After the v2.3.0 ship, six of the original nine are resolved (see § Resolved decisions for the inline closures). The genuinely-open list is below.

### Still open

1. **`pages.json` ownership** — currently a setup-time stub (template-seeded with placeholder routes); the PRD's setup spec mentions a future `specflow:pages` skill that would inventory live routes from the project's router config (Next.js / Remix / Express / etc.). Decide if `specflow:pages` is worth shipping in v2.4 or if the manual-stub-plus-test-time-population approach (`specflow:test` updates pages.json on first UI run) is enough.

2. **Design mockup readback** — should `specflow:prd` Phase A (codebase context gathering) or `specflow:task` Phase B (synthesis) consume the design folder's `current.html` / `proposed.html` / `iteration-log.md` as context? Currently no skill reads design output downstream. Two surfaces where this could matter: (a) PRD synthesis on a feature with an existing design — the proposed.html's component decisions ought to constrain the requirements; (b) task synthesis on a feature whose design has post-PRD iteration log entries — the iteration log captures decisions the PRD body might not yet reflect.

3. **`brief.html` commit policy** — the brief composes PRD + interview + manifests into a single self-contained HTML. Default recommendation: committed (the diffable surface for review); but for projects sensitive to repo size, gitignored-as-derived is defensible since `specflow:brief --all` regenerates from sources. Surface the choice as a setup-time prompt or `config.json.brief.commitPolicy`?

### Resolved during the v2.x ship cycle

4. ~~Rule registry starter set~~ — RESOLVED (v2.0.0). Non-negotiable.md template ships with the four starter rules (no hardcoded values, no comments unless WHY, never bypass auth, protected paths get Red lane); user accepts/edits at setup.
5. ~~Misc task rotation~~ — RESOLVED (v2.0.0). Single-rolling-file at `docs/specflow/misc-task/000-tasks-misc-tasks.md`; append-only with auto-allocated MISC-NNN ids.
6. ~~Retro trigger~~ — RESOLVED (v2.1.0). `specflow:complete` supports three trigger paths: manual `/specflow:complete {task-id}`, auto-fire from `specflow:develop` Phase F when a task closes, optional Linear webhook integration.
7. ~~Task history privacy~~ — RESOLVED (v2.0.0). Committed by default (the corpus must be diffable + greppable for `/insights` consumption); projects with sensitive surfaces can manually gitignore specific entries via per-entry markers.
8. ~~Agent snapshot refresh + cross-marketplace name collisions~~ — RESOLVED (v2.1.0). `specflow:agent refresh` produces a drift report without auto-overwriting; user re-snapshots per agent on confirmation. Name collisions across plugins require `--source {plugin-name}` to disambiguate; the index records the source.
9. ~~Rendered PRD commit policy / render parity~~ — SUPERSEDED (v2.2.0). `specflow:render` removed; `specflow:brief` composes PRD + interview + gate manifests into one self-contained HTML. The composition source widening eliminates the per-artefact render-parity question. Brief commit policy is the residual open question (#3 above).

---

## Appendix: candidate work (preserved roadmap detail)

The detailed sketches below are the menu Phase 1 and Phase 2 are picking from. Kept here so design intent isn't lost — sort each item into the right phase as decisions land above.

### A. Foundation: examples, README, setup updates

**Goal:** Make the plugin self-documenting. Ship a worked example users can browse, and have setup explain itself.

**A1. Build the examples directory**
- Create `plugins/specflow/examples/docs/specflow/` mirroring the full ClaimXPro structure.
- Populate every folder with realistic sample content (sanitised — no client identifiers).
- Files: populated `config.json`, `pages.json`, one well-formed PRD + matching task + test + screenshots, `misc-task/` examples, one design mockup, one example research doc.

**A2. Write the in-repo README**
- File: `plugins/specflow/examples/docs/specflow/README.md`
- Documents what every folder is for, when each gets created, and the workflow that produces it.
- Write as a *user guide*, not a technical reference.

**A3. Extend `specflow:setup`**
- Create new top-level directories during Phase 1: `admin/`, `admin/agents/standard/`, `admin/agents/specialised/`, `admin/rules/`, `features/`, `misc-task/`, `misc-task/assets/`, `docs/`.
- Per-feature subdirectories (`features/NNN-{slug}/{design,docs,assets}/`) are created lazily by `specflow:prd` when a feature is initialised — setup leaves `features/` empty.
- Copy the README into `docs/specflow/README.md` on first setup only — never overwrite if the user has edited it.
- Surface the location of the full example tree (`${CLAUDE_PLUGIN_ROOT}/examples/`) in the Phase 6 summary.

**A4. Templates strategy**
- Recommendation: `examples/` = read-only reference, `templates/` = files setup actively copies. Avoids the trap where users edit example files thinking they're configuration.

**A5. Document per-repo isolation explicitly**
- Plugin (skills, code, templates) installed once globally. State lives in each repo at `docs/specflow/admin/`, committed.
- Each project gets its own `config.json`, `profiles.json`, `decision-log.md`, `task-history.json`, agent set, and pages map. Repo A's self-learning corpus never touches Repo B's.
- Plugin always resolves paths relative to the current working directory — never a global cache.
- README must call this out prominently: *"specflow is installed once globally and adapts to each project independently. Your project's memory, profiles, decisions, and agents stay in this repo."*
- Setup must detect "no `admin/` folder here" rather than "plugin not installed" — entering a fresh repo with the plugin already global is the common case.

### B. `specflow:misc` skill

**Goal:** Single-task workflow for bugs and small fixes that don't warrant a PRD.

**B1. Skill behaviour**
- Triggers: "add a misc task", "log a quick bug", "/specflow:misc".
- Reads `docs/specflow/misc-task/000-tasks-misc-tasks.md`.
- Auto-detects next MISC-### ID from existing entries.
- Captures: title, scope (WEB/MOBILE/BACKEND/SHARED), priority, label (Bug/Feature), estimate, description, verification steps.
- Accepts attached screenshots/recordings — saves to `misc-task/assets/MISC-{NNN}-{slug}.{ext}`.
- Appends a new entry to the rolling file in the same shape as ClaimXPro's `000-tasks-misc-tasks.md`.

**B2. Linear integration**
- On export, target the project named `000-misc-tasks` (configurable in `config.json` under `linear.miscProject`).
- Auto-create the Linear project if it doesn't exist — check first, create if missing, store the project ID.
- Update the file's Export Map table with the resulting Linear ID + URL.

**B3. File initialisation**
- If `000-tasks-misc-tasks.md` doesn't exist when the skill runs, create it from a template (frontmatter scaffold + empty Quick Reference table + empty Pending Tasks section).

**B4. Open question**
- Single rolling file forever, or rotate per quarter/year? Recommendation: keep as one file until volume becomes a problem.

### C. `specflow:design` skill

**Goal:** Generate HTML/CSS mockups that match the live codebase exactly — alignment artefacts for cross-functional discussion, grounded in the project's actual design system.

**C1. Skill behaviour**
- Triggers: "create a design mockup", "/specflow:design".
- Step A — target detection: web or mobile? Mobile output is wrapped in a phone-frame viewport (configurable: iOS / Android / generic; defaults read from `config.json`).
- Step B — prime the codebase for the target feature (re-uses logic from `specflow:prime`).
- Step C — interview: what page, what change, what's the goal of the conversation this mockup will support?
- Step D — value extraction: read the actual component files (React/Vue/etc.), theme files, design tokens. Pull colours, typography, spacing, component shapes directly. **No invented values.** Extracted values are listed in a comment block at the top of each generated HTML file so reviewers can audit the source.
- Step E — generate two self-contained HTML files in the feature's `design/` folder (`docs/specflow/features/NNN-{slug}/design/`):
  - `{slug}-current.html` — faithful HTML/CSS rendering of how the feature looks today.
  - `{slug}-proposed.html` — proposed direction.
- Inline CSS, no JS dependencies, opens directly in a browser or shares as a file.
- Cross-feature or exploratory mockups that aren't tied to a specific PRD can still go in a top-level `docs/specflow/design/` if the user wants — but feature-specific mockups always live with their feature.

**C2. Codebase-truth principle (non-negotiable)**
- The number-one rule: **the mockup matches the live app**. Past attempts have drifted because the AI made up values instead of extracting them. The skill enforces this by:
  - Hard requirement: read source files for the relevant components before generating any HTML.
  - Listing every extracted value (colours, fonts, spacing tokens, breakpoints) in a comment block at the top of the generated HTML.
  - Running the Playwright iteration loop (C3) to verify the rendered output against the live app visually.
- If the skill cannot find the source for a referenced component, it stops and asks rather than guessing.

**C3. Playwright screenshot iteration loop**
- Required dependency: Playwright CLI installed locally (Phase 1 setup hard-checks for this — see appendix M).
- Loop:
  1. Boot the local dev server (or use a recorded screenshot when the dev server isn't available).
  2. Playwright captures a screenshot of the **live** component at the agreed viewport.
  3. Skill renders the generated mockup HTML.
  4. Playwright captures a screenshot of the mockup at the same viewport.
  5. Diff (size, position, colour, typography). Skill identifies discrepancies.
  6. Iterate the HTML/CSS until the diff is below threshold or the user explicitly accepts the remaining drift.
- Iteration log saved alongside the design files (`{slug}-iteration-log.md`) — captures what changed each round and why. Structure defined in C3.1.

**C3.1. Iteration log structure (decision capture)**

Static design files are iterated frequently. Every iteration is a decision being made — colour choice, layout shift, component swap, copy edit. Without explicit decision capture, the *why* gets lost and reviewers months later cannot reconstruct how the design landed where it did.

The iteration log is append-only. Every change to `current.html` or `proposed.html` MUST land a corresponding log entry with the *Why* field populated (an empty *Why* is a verify-step failure, not an acceptable shorthand).

Each entry records:
- *Files changed* — which static file, which section / component.
- *What changed* — concrete diff summary; before / after values where relevant.
- *Why (the decision)* — the reasoning, the alternative that was considered, the trigger (PRD requirement, interview round, Codex finding, Playwright diff, user-feedback turn). This is the load-bearing field.
- *Triggered by* — one of: `user-feedback`, `playwright-diff`, `codex-review`, `manual-observation`, `prd-clarification`.
- *Playwright drift* (when applicable) — before / after pixel-diff numbers or region identifiers.
- *Outstanding* — anything this iteration did not resolve, deferred to next round.

Reversals are their own entry citing the original iteration number ("Reverts iteration 4 — see entry below for new direction"). Past entries are never rewritten — that's the audit trail.

The `specflow:design` skill enforces this format; downstream reviewers (multi-agent debate manifest at any later gate touching the design) read the iteration log alongside the static files to see the decision chain, not just the final state.

**C4. Codex adversarial review (optional, when available)**
- If Codex CLI is detected in `admin/environment.json`, the skill invokes Codex as an adversarial reviewer at the end of the iteration loop.
- Codex reviews the generated HTML/CSS against the live source and the user's stated goal — flags semantic gaps the Playwright loop misses (e.g. "the proposed flow loses the cancel affordance" rather than "this button is 2px off").
- If Codex isn't installed, the skill skips this step gracefully.

**C5. Tone and scope**
- Mockups are *discussion artefacts* — they communicate intent and layout. Codebase-truth applies to fidelity of the *current* state. The *proposed* state can vary, but should still draw values from the existing design system unless the user explicitly asks for a departure.
- Banner at the top of generated files: "This is a discussion mockup, not a final design."

**C6. Open questions**
- Should PRD/task skills read these mockups back as context? If yes, the design skill writes a sidecar `.md` summary alongside each HTML file.
- Should the iteration loop run on the *proposed* design (no live counterpart)? Or skip Playwright on proposed and rely solely on Codex review?

### D. `docs/` folder integration

**Goal:** Treat external research as a first-class input to PRD creation.

**D1. Convention only — no new skill**
- Two locations: `docs/specflow/docs/` for cross-feature material; `docs/specflow/features/NNN-{slug}/docs/` for feature-specific source material that lives with the feature.
- Document both in the README. Setup creates the top-level folder empty; the per-feature folder is created lazily by `specflow:prd`.

**D2. Wire into `specflow:prd` Phase 1**
- Phase 1 (Problem Discovery) detects files in both locations: the cross-feature `docs/specflow/docs/` and (when re-running on an existing feature) the feature's own `features/NNN-{slug}/docs/`.
- If present, asks: "I found {n} research files. Should I read them as context for this PRD?"
- If yes, reads them and incorporates into the problem framing.

### E. `specflow:upgrade` skill

**Goal:** Refresh an aged specflow installation without losing user customisation.

**E1. Why this exists**
- Projects run for months. Tech stack changes, new agents become available in marketplaces, the plugin itself evolves. Right now there's no way to "catch up" without deleting `config.json` and running setup from scratch.

**E2. Skill behaviour**
- Triggers: "upgrade specflow", "refresh specflow config", "/specflow:upgrade".
- Step 1 — version detection: read `specflowVersion` from `docs/specflow/admin/config.json` and compare to the installed plugin version (`plugins/specflow/.claude-plugin/plugin.json`).
- Step 2 — migration plan: read `plugins/specflow/MIGRATIONS.md` and compute the chain of migration entries that apply between the project's current version and the plugin's version. Each entry declares its scope: schema changes, file moves, new fields, deprecations.
- Step 3 — environment + tech-stack diff: re-detect tech stack and re-run environment inventory (appendix M). Surface drift — new CLIs, missing MCPs, new agents, deprecated agents.
- Step 4 — present plan: list each migration step + each environment/stack change as a checkbox. User confirms which to apply.
- Step 5 — apply with safety: backup affected files (`.bak` suffix); apply each step; record outcome.
- Step 6 — write new `specflowVersion` to `config.json` and emit a summary of what changed.

**E3. Versioning support**
- `config.json` includes `specflowVersion`.
- `plugins/specflow/MIGRATIONS.md` is the source of truth for what each version added/changed/moved. Every PR that ships a breaking change updates this file.
- The upgrade skill is purely additive — never deletes user data without explicit confirmation.

**E4. First migration entry (Phase 1 ship)**
- v1.x → v2.0 covers: relocate `config.json` and `pages.json` into `admin/`; consolidate `prd/`, `task/`, and `test/` into `features/NNN-{slug}/` (each existing PRD becomes a folder; `prd/NNN-{slug}.md` → `features/NNN-{slug}/prd.md`, the matching task/test files move and rename to `tasks.md` / `test.md`, `test/assets/` items rehome to the feature's `assets/`); create `admin/agents/` (standard + specialised + index), `admin/rules/`, `admin/environment.json`, `admin/profiles.json` from interview; seed empty `decision-log.md` and `task-history.json` for Phase 3.
- Migration also generates `prd.html` for every relocated PRD via the new render skill (Appendix P) so existing features pick up the browser-readable view in one pass.
- Existing projects upgrade by running `/specflow:upgrade` once after pulling the new plugin version. Backups (`.bak`) are written for every moved file before the relocation.

### F. User profiles / personas

**Goal:** Define the actors that show up in PRD user stories once, reuse everywhere. Profiles are first-class.

**F1. New mandatory phase in `specflow:setup`**
- Add **Phase 3c: Define User Profiles** to setup, immediately after the constitution phase (3b).
- Required step in both quick and custom modes — users can skip individual profiles, but the phase always runs.

**F2. Plugin-shipped profile examples**
- Ship a library of common examples: `plugins/specflow/templates/profile-examples.json`.
- Curated set: Admin/platform operator, End user/customer, Power user, Support/customer success, Developer/API consumer, Finance/billing, Auditor/compliance, Field worker/mobile-first user.
- Each example includes filled-out `name`, `role`, `goals`, `constraints`, `painPoints`.

**F3. Setup interview flow**
1. Setup explains the concept.
2. Reads `profile-examples.json` and shows a curated subset (3–5) most relevant to the detected stack.
3. Asks which apply or invites custom profiles.
4. For each: name (canonical, used in `As a <name>` stories), role, primary goals, key constraints, pain points.
5. Auto-generate option — same as constitution, user can say "you decide" and setup proposes a full profile set based on CLAUDE.md, auth code, and role definitions.
6. Confirm the full set back to the user before writing.

**F4. Quick mode behaviour**
- Default to auto-generation: read the codebase, propose 3–5 profiles, present all at once for one-shot confirmation.

**F5. Storage**
- Store as `docs/specflow/admin/profiles.json` (separate file in the admin folder).
- Schema:
  ```json
  {
    "profiles": [
      {
        "name": "Project Manager",
        "role": "Day-to-day user managing claim assessments",
        "goals": ["Complete site visits efficiently", "Generate accurate reports"],
        "constraints": ["Often on mobile", "May lose connectivity in field"],
        "painPoints": ["Repetitive data entry", "Inconsistent report formatting"]
      }
    ]
  }
  ```

**F6. Wire into `specflow:prd`**
- Phase 3 ("Users and actors" probe) reads profiles and offers them: "Which of these actors does this feature affect? [list]"
- User stories use the canonical names from profiles instead of free-text actors.

**F7. Wire into `specflow:upgrade`**
- Existing projects without profiles get an offer to define them during upgrade.

### G. Test asset support

**Goal:** Make `test/assets/` a first-class concept in the test skill.

**G1. Test skill changes**
- When a test plan is generated, the skill knows it can request screenshots and they go to the feature's own asset folder: `features/NNN-{slug}/assets/{description}.png`. Test artefacts and design captures share this folder so everything for one feature lives together.
- Verification scenarios reference asset filenames (relative to the feature folder, so test plans stay portable).
- Reads `pages.json` to navigate to the right pages during execution.

**G2. `pages.json` first-class support**
- Document `pages.json` in the README and example.
- Decide ownership: written by setup, by a separate skill, or by the user manually?
- Recommendation: separate optional `specflow:pages` skill (or sub-phase of setup) that scans the codebase for routes (Next.js `app/` or `pages/`) and proposes a starting `pages.json`.

### I. Admin folder + self-learning memory loop

**Goal:** Give specflow a persistent memory of decisions and outcomes so each new task benefits from what was learned on previous ones. Per-repo by design — the entire memory loop lives inside `docs/specflow/admin/` and is committed to the repo.

**I1. The `admin/` folder**
- New top-level folder: `docs/specflow/admin/`.
- Holds all files specflow itself authors and reads — config, page maps, personas, decision log, task history, agents.
- Clean separation from user-authored content (`prd/`, `task/`, `docs/`, `design/`).
- Migrating `config.json` and `pages.json` into `admin/` is a breaking change handled by the upgrade skill.

**I2. Decision log (`admin/decision-log.md`)**
- Markdown file capturing structured decisions over the project's lifetime.
- Sections: **Key decisions** (architectural/workflow choices), **What worked** (tagged), **What didn't work** (tagged), **Improvements identified** (meta-observations).
- Updated from two sources: explicit user input (`/specflow:decision`) and automated observations from the feedback loop.

**I3. Task history (`admin/task-history.json`)**
- Structured record of every completed task and its outcome.
- Schema:
  ```json
  {
    "tasks": [
      {
        "id": "PIU-003",
        "linearId": "CLA-127",
        "title": "Add notification popover to header",
        "scope": ["WEB"],
        "estimateHours": 4,
        "actualHours": 5.5,
        "completedAt": "2026-04-28",
        "aiAssistance": "partial",
        "aiNotes": "Component scaffolded by AI; manual fixes needed for animation timing and a11y",
        "whatWorked": ["Reading existing popover patterns first", "Generating Storybook story alongside component"],
        "whatDidntWork": ["AI assumed a global toast context that didn't exist"],
        "tags": ["ui", "notifications", "popover"]
      }
    ]
  }
  ```
- `aiAssistance` enum: `"full"` | `"partial"` | `"none"`.

**I4. The feedback loop**
- **Capture:** when a developer marks a task done (Linear or `/specflow:complete <task-id>`), specflow runs a short retro: AI assistance level, what worked, what didn't, hours estimated vs actual.
- **Persist:** answers append to `task-history.json`. Significant patterns also append to `decision-log.md`.
- **Reuse:** when a *new* task is being created, the skill searches `task-history.json` for similar past tasks (matching tags, scope, or title keywords) and surfaces: "3 similar tasks completed previously. Average estimate accuracy: -25%. Common gotcha: [from whatDidntWork]."

**I5. Decision-log skill (`/specflow:decision`)**
- Lightweight skill for users to manually log a decision without leaving the flow.
- Captures: title, context, decision, rationale, date, related files/tasks.
- Appends to `decision-log.md`.

**I6. Auto-research loop (longer-term, out of scope for current phases)**
- References to study: `forrestchang/andrej-karpathy-skills`, `karpathy/autoresearch`.
- Vision: at the end of a sprint, specflow runs a "review pass" — re-reads `task-history.json`, looks for repeated friction, proposes updates to constitution/agents/skills.

**I7. Wire into existing skills**
- `specflow:setup` — creates `admin/` and seeds `decision-log.md` and `task-history.json` with empty templates.
- `specflow:task` and `specflow:misc` — query `task-history.json` for similar tasks during creation; surface insights.
- `specflow:upgrade` — handles the file relocation from `docs/specflow/` root into `admin/`.
- `specflow:prd` — optionally consults `decision-log.md` during interview.

**I8. Open questions**
- Retro trigger: manual (`/specflow:complete`) or automated via Linear webhook polling? Manual first.
- Search strategy for "similar tasks": simple tag/keyword match initially, embeddings later if useful.
- Privacy: `task-history.json` always committed or sometimes gitignored? Recommend committed by default — it's project memory.

### J. Agent teams (Phase 2)

**Goal:** specflow supports invoking named agent teams per project — compositions defined in `config.json`, callable through skills (especially `specflow:develop`).

**J1. Team definitions in `config.json`**
- New section in `config.json`:
  ```json
  {
    "teams": {
      "feature-build": ["orchestrator", "frontend-developer", "backend-developer", "verifier"],
      "review": ["devils-advocate", "security-auditor", "verifier"],
      "debug": ["orchestrator", "team-debugger", "devils-advocate"]
    }
  }
  ```
- Each team is a named list of agents (referenced from the indexed agent set).
- Standard agents (Orchestrator, Devil's Advocate, Verifier) can appear in any team.

**J2. Invocation**
- Via `specflow:develop` — Orchestrator picks an appropriate team based on task scope, or the user explicitly invokes one.
- Via direct skill — `/specflow:team feature-build "build the notifications popover"`.

**J3. Soft dependency on agent-teams plugin**
- If the agent-teams plugin is installed (detected in `environment.json`), specflow uses its team-spawn mechanics.
- If not installed, specflow falls back to invoking each agent sequentially via the Agent tool. Degrades, doesn't break.

**J4. Open questions**
- Should team definitions be validated against the indexed agent set at write-time, or lazily when invoked?
- How does `specflow:upgrade` handle team definitions that reference a now-missing agent?

### K. Agents directory

**Goal:** A visible, browsable per-repo agent registry. Running `ls docs/specflow/admin/agents/` should answer "which agents are available on this project?" — modeled on the `~/.claude/agents/` pattern.

**K1. Folder structure**
- New folder: `docs/specflow/admin/agents/`.
- Subfolders: `standard/` (every project gets these — split into `lifecycle/` and `principles/` categories) and `specialised/` (matched to stack).
- Each agent is a markdown file (frontmatter + body), browsable, diffable.

**K2. Standard agents — Lifecycle (shipped with the plugin)**

Source-of-truth files: `plugins/specflow/templates/agents/standard/lifecycle/`. Copied to `admin/agents/standard/lifecycle/` at setup. Set kept deliberately small. Additions only when a clearly distinct lifecycle moment emerges that none of the three covers.

- **Orchestrator** — coordinates multi-agent workflows, holds the plan, delegates to specialists. Always-on; routes work through the right agents and keeps multiple points of view in play. **Also writes the closing decision entry in every multi-agent debate manifest.**
- **Devil's Advocate** — challenges decisions *in-flight* — surfaces blind spots in PRDs, scope ambiguity in tasks, architectural gotchas during development. One of the parallel reviewers fired into the debate manifest at every gate.
- **Verifier** — confirms work meets the bar *at the end* of any task. Reads the original requirement, the produced output, and verifies they match. Distinct from Devil's Advocate: DA challenges in-flight, Verifier confirms-on-completion.

Lifecycle coverage: Orchestrator owns *plan*, Devil's Advocate owns *challenge*, Verifier owns *confirm*. Each runs at a different phase of any non-trivial work.

**K2b. Standard agents — Principle reviewers (shipped with the plugin)**

Source-of-truth files: `plugins/specflow/templates/agents/standard/principles/`. Copied to `admin/agents/standard/principles/` at setup. One reviewer per core principle. Set scales 1:1 with `CORE_PRINCIPLES.md` — adding a fifth principle requires adding a fifth reviewer.

- **Simplicity Reviewer** — enforces Simplicity First including the AI-specific sub-tests (explicit beats clever, local reasoning beats cross-file elegance). Asks "is there a simpler way?" first at every gate.
- **Surgical Reviewer** — enforces Surgical Changes; flags adjacent-fix creep, missed misc-task creation when a rule violation is spotted out-of-scope, and changes that touch lines unrelated to the work item.
- **Think-Before-Coding Reviewer** — enforces Think Before Coding; flags hidden assumptions, missing-tradeoff articulation, and premature commitment to one approach when alternatives weren't articulated.
- **Goal-Driven Reviewer** — enforces Goal-Driven Execution; verifies inline verify-steps are present, that they're binary (pass/fail not "looks fine"), and that the skill's `eval:` field actually exercises the produced output.

Principle reviewers run in parallel as part of the multi-agent debate manifest (Appendix N). Their findings target a specific principle each, which keeps each finding sharp — a Simplicity Reviewer finding cites the Simplicity principle line it's challenging.

**K3. Specialised agents**
- Drawn from installed marketplaces (whobson and others) and matched to the project's stack.
- During setup tech detection, specflow proposes a starter set: React → `frontend-developer`; Postgres-heavy → `database-architect`; security-sensitive → `security-auditor`.
- User confirms; each confirmed agent is brought in via snapshot.

**K4. Indexing + snapshots**
- specflow scans all installed agent sources and builds a global index.
- When an agent is added, take a *snapshot*: copy the full definition into the repo. Pinned and reviewable.
- Snapshots show as diffs in PRs.
- Index file: `docs/specflow/admin/agents/index.json` — name, source plugin, tags, snapshot-date, source-version.
- `specflow:upgrade` compares snapshots against current upstream and prompts to refresh stale ones.

**K5. The `specflow:agent` skill**
- Verbs: `add <agent>`, `remove <agent>`, `list`, `refresh <agent>`.
- `list` prints standards + specialised, with source plugin and last-snapshot date.
- `add` searches the global index by name/tag, snapshots into `specialised/`, updates `index.json`.

**K6. Wire into existing skills**
- `specflow:setup` — creates `admin/agents/`, copies standards, runs the specialised proposal during tech detection, snapshots confirmed agents.
- `specflow:upgrade` — diffs snapshots against current upstream; flags new agents in marketplaces relevant to this project's stack.
- `specflow:doctor` — validates every indexed agent still resolves to an installed source.
- `specflow:prd`, `specflow:task` — can reference the project's available agent set when proposing how to execute work.

**K7. Open questions**
- Refresh strategy: prompt every upgrade, or auto-refresh standards and only prompt for specialised? Recommendation: latter.
- Naming collisions across marketplaces: namespace snapshot filenames (`whobson__frontend-developer.md`).

### L. `specflow:develop` skill (Phase 2 centrepiece)

**Goal:** Dedicated skill that orchestrates the *implementation* phase — the gap between task creation (Linear-ready) and test. The core deliverable of Phase 2.

**L1. Skill behaviour**
- Triggers: "develop this task", "/specflow:develop {task-id}", or invoked automatically when a Linear task is moved into "In Progress".
- Step 1 — context primer: read the task, the related PRD, relevant decision-log entries, the project's available agent set from `admin/environment.json`, and the rules registry slice that applies to the touched paths.
- Step 2 — lane assignment + team composition: lane assigned by triage tuple (verifiability + blast radius + dependency state + confidentiality classification). Orchestrator picks the right specialised agents based on lane and task scope; agent teams from `config.json` compose the working group.
- Step 3 — plan emission with PRD anchor: every plan starts with *"We're doing X because of PRD requirement Y. This aligns with Z."* Then the technical plan with verify-steps inline (Goal-Driven Execution).
- Step 4 — implementation loop: agents execute with Orchestrator coordinating; Devil's Advocate intervenes at decision points; testing-as-cadence — Playwright/test verification fires after every step, not at the end.
- Step 5 — adversarial review (Gate 5): cross-provider Codex review on the diff, running the 3-iteration debate loop (Appendix N).
- Step 6 — verification: Verifier runs at completion vs the original task acceptance criteria.
- Step 7 — handoff: emits a summary the test skill can pick up; updates Linear status; logs the run for Phase 3 retro consumption.

**L2. Green / Yellow / Red lane execution**
- **Green** (verifiable + low blast + non-confidential): batched, AFK-eligible, single batched human sign-off per batch. Machine checks pass before any PR opens.
- **Yellow** (one axis weak): HITL — agent + human paired in real time, one at a time.
- **Red** (high blast OR low verifiability OR confidential surface — auth, secrets, schemas, billing, public surface): human-led; AI assists on bounded subtasks only. Confidentiality classification is rule-based via path globs in `config.json.confidentialPaths`, never AI-rated.
- Initial target ratio: 60/30/10 (G/Y/R). 30/40/30 means the PRD needs to be re-cut into smaller, more isolated issues — itself a learning signal logged to `task-history.json`.

**L3. PR / commit conventions**
- Skill enforces project-defined conventions from `config.json` (commit style, PR title format, branch naming).
- Pauses for human input at defined trigger points (protected files, migrations, new dependencies).

**L4. Rule violation auto-flagging**
- When `specflow:develop` spots a rule registry violation outside the current task's scope, it auto-creates a `misc-task` with the rule reference, file:line, observation, and the *why* (citing the rule's why-line). Does not fix inline (Surgical Changes).
- If the violation is *inside* the current task's scope, it's surfaced as a blocker, not a misc-task — must be addressed before the change set ships.

**L5. Codex integration**
- If Codex is in the inventory, it's the standing adversarial reviewer for Gate 5 (code vs plan). Separate role from Devil's Advocate (DA challenges decisions; Codex challenges code).
- Findings Codex catches that Claude missed get promoted to new entries in `admin/rules/guidelines.md` (system evolution).

**L6. Dependencies**
- Phase 1: standard agents shipped, environment inventory live, rules registry seeded, behavioral principles loaded, `panic` + `confidence-check` available.
- Phase 2: agent indexing + specialised agent matching + agent teams composition all land before `specflow:develop` is functional.

### M. Environment inventory

**Goal:** specflow knows what tools (CLIs, MCPs, plugins, agents) are available in the user's environment, exposes only the relevant slice to each skill, and can degrade or escalate based on what's present. Better outputs without bloating the context window.

**M1. What gets inventoried**
- **CLIs** — Playwright (required), Codex (optional, enables adversarial review), git, gh, Linear CLI, others as detected.
- **MCPs** — every MCP server installed in the user's Claude Code config (Linear, Drive, Gmail, Calendar, custom).
- **Plugins** — every plugin under `~/.claude/plugins/`, with version.
- **Agents** — every agent exposed by installed plugins, indexed by name + source plugin.

**M2. Storage**
- File: `docs/specflow/admin/environment.json`. Committed to the repo.
- Schema (initial sketch):
  ```json
  {
    "lastDetected": "2026-04-30T10:00:00Z",
    "cli": [
      {"name": "playwright", "available": true, "version": "1.49.0", "required": true, "uses": ["specflow:design", "specflow:test"]},
      {"name": "codex", "available": true, "version": "0.3.0", "required": false, "uses": ["adversarial-review"]}
    ],
    "mcp": [
      {"name": "linear", "available": true, "scope": ["task-export", "misc-export"]},
      {"name": "google-drive", "available": false}
    ],
    "plugins": [
      {"name": "specflow", "version": "1.9.2"},
      {"name": "agent-teams", "version": "0.5.0"}
    ],
    "agents": [
      {"name": "orchestrator", "source": "specflow", "scope": "standard"},
      {"name": "frontend-developer", "source": "frontend-mobile-development", "scope": "specialised"}
    ]
  }
  ```

**M3. Hard vs soft requirements**
- **Hard** — missing breaks setup. Currently: Playwright CLI.
- **Soft** — missing changes behaviour, doesn't block. Currently: Codex CLI (adversarial review degrades to skipped); specific MCPs (Linear export disabled if missing).
- The hard/soft distinction is encoded in the inventory file so skills decide whether to fail loud or degrade quietly.

**M4. When the inventory updates**
- `specflow:setup` — initial detection; hard-requirement enforcement (fails setup if Playwright missing).
- `specflow:upgrade` — re-runs detection, surfaces additions and losses as part of the upgrade plan.
- `specflow:doctor` (Phase H3) — on-demand re-detection without applying any changes; useful sanity check.

**M5. Context-conscious consumption**
- Skills don't dump the full inventory into context — they query the slice they need.
- Examples:
  - `specflow:design` reads `{cli.playwright, cli.codex}` plus relevant plugin/agent entries.
  - `specflow:develop` reads the agent set heavily, ignores MCP details unless a specialist needs them.
  - `specflow:prd` reads agent names only — proposes specialists by name, doesn't load full definitions.
- This keeps each skill's working context scoped to what it actually needs to produce a good output.

**M6. Wire into existing + new skills**
- `specflow:setup` (Phase 1) — generates `environment.json` during tech detection; enforces hard requirements.
- `specflow:upgrade` (Phase 1) — diffs detected vs stored, prompts to apply changes, refreshes the file.
- `specflow:design` (Phase 1) — reads CLI section; gates Playwright loop and Codex adversarial review on availability.
- `specflow:develop` (Phase 2) — heavily relies on the inventory to compose teams and pick agents.
- `specflow:doctor` (Phase 1 stub or Phase 2) — re-runs detection on demand.
- `specflow:prd`, `specflow:task` — read agents section to propose appropriate specialists.

**M7. Open questions**
- Refresh strategy — explicit only (recommended), or also at the start of every skill invocation? Latter is too noisy.
- Privacy — `environment.json` committed by default (it's project-scoped, no secrets) or gitignored? Recommend committed.
- Stale detection — when does the inventory become "old enough" that a refresh prompt is warranted? Time-based (e.g. > 30 days) or only on plugin version change?

---

### H. Production polish

**Goal:** Production-grade discoverability, versioning, contributor experience.

**H1. Documentation**
- Update root `README.md` with a complete workflow diagram (setup → prime → prd → task → design → linear → develop → test → misc).
- Add `docs/specflow/MIGRATIONS.md` (plugin-side) for upgrade-skill consumption.
- Make sure CHANGELOG lists folder/schema changes for each release.

**H2. Versioning discipline**
- `config.json` includes `specflowVersion`.
- Plugin install/upgrade flow respects version skew.

**H3. Validation**
- `specflow:doctor` skill validates the local installation: required folders exist, `config.json` schema valid, referenced agents installed, etc.

---

### N. Adversarial review chain + 3-iteration debate loop

**Goal:** wherever specflow produces an artefact (brief, PRD, tasks, plan, code, tests), a reviewer runs adversarially against it. Each gate uses the same 3-iteration debate loop. Codex when available; Devil's Advocate as fallback.

**N1. The six gates (phased across Phase 1-3)**
- **Gate 1 — `/grill` (Phase 1).** AI interrogates the human (one question at a time, re-evaluating after each answer). Output: rounds appended to `features/NNN-{slug}/NNN-{slug}-interview.md`. This *is* the alignment review; no separate reviewer needed.
- **Gate 2 — PRD vs brief (Phase 1).** Auto-fires when `specflow:prd` produces a draft. Reviewer compares PRD content against grilling Q&A: did the PRD lose anything, add unstated scope, or paper over an unresolved question?
- **Gate 3 — tasks vs PRD (Phase 1).** Auto-fires on `specflow:task`. Reviewer asks: is every PRD requirement covered? Does any task introduce scope not in the PRD? Are assumptions and acceptance criteria explicit?
- **Gate 4 — plan vs tasks + PRD anchor (Phase 2).** Inside `specflow:develop`. Does the plan match the task scope? Does it anchor back to the PRD requirement?
- **Gate 5 — code vs plan (Phase 2).** Cross-provider Codex review on diffs. Catches what same-provider review misses (Codex finds Claude's blind spots; Devil's Advocate runs as backup). Findings that Codex catches and Claude missed get promoted to new rules in `admin/rules/`.
- **Gate 6 — tests vs requirements (Phase 3).** Adversarial coverage review on the test suite for the work item.

**N2. The multi-agent debate manifest (every gate uses this)**

Replaces the prior single-reviewer 3-iteration loop with a parallel-reviewer extension of the same shape. The artefacts live inside the feature folder under `features/NNN-{slug}/debate-log/{gate}/`:

- `manifest.md` — the collated, human-readable debate transcript (findings + responses + closing).
- `findings/round-{1,2,3}/{reviewer-name}.json` — one minimal finding file per reviewer per round. Just severity, evidence, proposed change. **Not the reviewer's internal reasoning trail** — that stays in the reviewer's forked sub-agent context and dies with it.
- `raw/{reviewer-name}.txt` (optional) — full reasoning trace, retained only when the orchestration was flagged for Phase 3 self-learning archival. Most gates discard it.

**Why feature-scoped paths:** When a downstream agent works on the same feature (e.g. `specflow:develop` after `specflow:task`), it can read every prior gate's manifest from the same feature folder. No cross-folder chase. The feature folder is a complete record of how the feature was built. Cross-feature gates (misc-task review, `/optimize` runs that don't belong to a single feature) keep the legacy `admin/debate-log/{date}-{slug}-{gate}/` location since there's no feature folder to home them in.

**Why this shape (orchestrator-pattern compliance):**
Reviewers run in forked sub-agent contexts. They read the artefact under review via command substitution (zero token cost), do their work, write a minimal finding JSON, return one line (the file path) to the Orchestrator. The Orchestrator never sees the reviewers' internal reasoning — it sees only structured findings. This keeps the parent context bounded regardless of how many reviewers fire or how chatty each one is. Detail in `templates/orchestrator-pattern.md`.

**Participants (per gate):**
- **Lifecycle:** Devil's Advocate (always present as a parallel reviewer).
- **Principles:** the four principle reviewers from `admin/agents/standard/principles/` (Simplicity, Surgical, Think-Before-Coding, Goal-Driven). Each fires findings scoped to its principle.
- **Cross-provider:** Codex when available — fires alongside the principle reviewers, especially load-bearing for Gates 5 and 6.
- **Closer:** Orchestrator. Does NOT fire findings; reads the full manifest and writes the closing decision entry.

**Round 1 — Parallel finding fire.**
Each reviewer runs in a forked sub-agent context. The Orchestrator dispatches all reviewers in parallel; each reviewer:
- Reads the artefact under review via command substitution (`!{cat features/NNN-{slug}/prd.md}`) — zero token cost.
- Does its work in the fork (raw reasoning, exploratory tool calls, internal drafts) — none of this returns to the parent.
- Writes a minimal finding JSON to `features/NNN-{slug}/debate-log/{gate}/findings/round-1/{reviewer-name}.json`:
  - *reviewer:* the agent name.
  - *principle / concern:* which principle or concern this finding maps to.
  - *severity:* `block | concern | note`.
  - *evidence:* file:line, requirement ID, rule registry entry, decision-log precedent.
  - *proposed_change:* a concrete revision suggestion (not "this is bad" without "do this instead").
- Returns one line (the finding file path) to the Orchestrator.
- The first question every reviewer must answer: *"Is there a simpler way to achieve the acceptance criteria?"* (Simplicity First check, applied even by reviewers whose principle isn't Simplicity.)

**Round 2 — AI responds.**
The writer (AI that produced the artefact) runs in its own forked context. Reads every Round-1 finding via command substitution. For each finding:
- *accept:* will revise. The revision is described and applied.
- *push_back:* the approach is correct because X, Y, Z. Must cite evidence (code structure, PRD requirement, rule registry entry, decision-log precedent). Push-backs without evidence are themselves a finding the next round can sharpen.

Writes the structured response to `features/NNN-{slug}/debate-log/{gate}/findings/round-2/responses.json` keyed by Round-1 finding ID.

**Round 3 — Reviewers sharpen or accept.**
Each reviewer (forked again — fresh context, same reviewer prompt) reads the AI's response to its own Round-1 finding via command substitution. Writes one of:
- *accept:* AI's defence holds.
- *sharpen:* new evidence, reframed concern, escalated severity.

To `features/NNN-{slug}/debate-log/{gate}/findings/round-3/{reviewer-name}.json`. If sharpened, the AI revises one more time and the revision is recorded in `round-3/ai-revision.md`.

**Closer — Orchestrator collates the manifest.**
The Orchestrator reads every round's structured findings (small JSON files), collates them into the human-readable `manifest.md`, and writes the closing decision entry. The structure:
- *Accepted findings:* what was revised and where.
- *Rejected findings:* what was pushed back and why (with the evidence chain).
- *Escalated:* findings where reviewers and AI didn't converge in 3 rounds — surfaced for human decision.
- *Sign-off:* gate passes or fails.

**Why the manifest beats the singular debate transcript:**
- More eyes catch more failure modes. Each principle reviewer is sharp on one concern instead of fuzzy on many.
- The manifest is queryable. Phase 3 self-learning reads it for patterns: which reviewers' findings are accepted most often (well-calibrated prompts) vs. always rejected (over-aggressive prompt to retune).
- Orchestrator's closing entry replaces "Claude makes the final call" with an explicit, traceable decision step — so debates aren't won by whoever spoke last.

**Cost discipline:**
- Round 1 is parallel — N reviewers fire at once, each in its own forked context (no carry-over bias from the others, no shared accumulated context with the parent Orchestrator).
- Reviewers use cheap models (Haiku for routing, Sonnet for the actual review) except where Codex is available for high-stakes gates.
- 3-round cap retained — more rounds = diminishing returns and drift from original intent.
- **Context budget per gate (target):** Orchestrator parent context grows by ≤2K tokens per gate regardless of how many reviewers fire. Round-1 findings are ≤200 tokens each; Round-2 responses are ≤400 tokens; Round-3 entries are ≤200 tokens; closing entry is ≤500 tokens. If the Orchestrator's context is growing faster than this, reviewers are leaking — see `templates/orchestrator-pattern.md` for the audit checklist.

**N3. Why the debate loop matters**
- Single-shot review is what produces rubber-stamping (Saraev's reliability decay; CEO Round 1's rubber-stamp risk).
- Forcing AI to defend its choices catches lazy reasoning that single-shot review accepts.
- Forcing the reviewer to produce a substantive Round 3 stops drive-by critiques.
- Capping at 3 rounds with a designated tiebreaker (Claude) prevents the loop from becoming infinite negotiation.

**N4. Cost / context cost**
- Three rounds per gate; six gates total when fully phased in. Most gates use cheap models (Haiku for routing, Sonnet for the actual review). Codex calls for high-stakes gates (5, 6).
- Each gate runs with a fresh context (no carry-over bias from earlier gates). Each round inside a gate carries only the artefact + the prior round's claim.
- Total cost per PRD (full pipeline): low single-digit dollars. Acceptable.

**N5. What flows out of the loop**
- The debate transcript itself (saved, queryable in Phase 3).
- Findings the reviewer made that AI conceded → may surface as new entries in `admin/rules/guidelines.md`.
- Findings AI defended successfully → may inform `admin/CONTEXT.md` (the slim live doc).
- Override frequency on a particular skill or rule → input to `/optimize` Phase 3.

---

### O. Rules registry

**Goal:** every project has a small, slim, live registry of rules the AI must respect. Detection of violations triggers automatic `misc-task` creation (the AI captures the observation and *why*, doesn't fix inline). The registry self-evolves in Phase 3.

**O1. Two tiers + glossary**
- `admin/rules/non-negotiable.md` — hard rules, always-on. Examples: no hardcoded values unless necessary; no comments unless non-obvious WHY; never bypass auth; protected paths require Red lane (human-led).
- `admin/rules/guidelines.md` — soft rules / project taste. Examples: prefer composition over inheritance; tests sit in `__tests__` next to source.
- `admin/rules/glossary.md` — every rule listed with a one-line description and the *why* it exists.

**O2. Rule format**
Each rule is a small frontmatter block + body:
```markdown
---
id: NO_HARDCODED_VALUES
tier: non-negotiable
paths: ["src/**/*.ts", "src/**/*.tsx"]   # auto-load when these files are touched
---
**Rule:** Hardcoded values (strings, numbers, URLs, magic constants) should be moved to config or environment unless absolutely necessary.
**Why:** Hardcoded values resist change, leak across environments, and silently couple components. Dynamic by default.
**On violation:** Auto-create a misc-task with the file:line reference, the value, and why it should be dynamic.
**Exceptions:** Test fixtures; one-off scripts; literal protocol values (HTTP status codes, well-known port numbers).
```

**O3. Loading**
- Always-on rules from `non-negotiable.md` load into every skill's system prompt.
- Path-scoped rules auto-load when the AI is touching files matching the `paths:` glob.
- Guidelines load on demand when relevant (e.g. during `specflow:develop`).
- The four behavioral principles are loaded universally — they govern *how the AI behaves*. The registry governs *what the AI accepts in this codebase*.

**O4. On violation — auto-misc-task**
When the AI detects a rule violation outside the current work item's scope:
1. Does NOT fix inline (Surgical Changes).
2. Auto-creates a `misc-task` entry with: rule ID, file:line, observation, *why* (citing the rule's why-line), suggested fix.
3. Continues with the original work item.

If the violation is *inside* the current work item, it must be addressed before the change set ships — the AI surfaces it as a blocker, not a misc-task.

**O5. Self-evolution (Phase 3)**
- `task-history.json` and `misc-task` entries are scanned for repeated violations of the same shape.
- Three observations of the same pattern → suggest a new entry in `guidelines.md`. Human signs off.
- Three guideline-flagged violations → suggest promotion to `non-negotiable.md`. Human signs off.
- This compounds: the registry grows toward the project's actual taste over time, without human-curated lists.

**O6. Setup seeding**
At setup, `specflow:setup` proposes a starter set of non-negotiables:
- `NO_HARDCODED_VALUES`
- `NO_COMMENTS_WITHOUT_WHY`
- `NEVER_BYPASS_AUTH`
- `PROTECTED_PATHS_REQUIRE_RED_LANE`
- `TESTS_REQUIRED_FOR_VERIFIABLE_SKILLS`

User reviews, accepts/edits, the set is committed to `admin/rules/non-negotiable.md`. Additive over time.

**O7. Open questions**
- Should rules versioning ship — i.e. `id: NO_HARDCODED_VALUES@2` so promotions are auditable in `git log`?
- How does the registry interact with cross-marketplace agent definitions that may carry their own opinionated rules?

---

### P. PRD static HTML rendering (`specflow:render`)

**Goal:** every PRD has a sibling `prd.html` that's pleasant to read in a browser. The markdown stays the source of truth; the HTML is a derived view for human review. Solves the "PRDs are becoming too hard to read" problem without forcing markdown back into a tool that fights the writing experience.

**P1. Skill behaviour**
- Triggers: auto-fires whenever `specflow:prd` writes or updates `features/NNN-{slug}/prd.md`. Also exposed as `/specflow:render {feature-id}` for manual re-render, and `/specflow:render --all` for a one-shot pass over every existing feature (useful right after the Phase 1 migration).
- Reads `features/NNN-{slug}/prd.md`. Parses the markdown into a single self-contained HTML file written next to it as `prd.html`.
- Idempotent: running the skill on an unchanged PRD produces a byte-identical output (deterministic timestamps stripped or pinned to the PRD's git mtime).

**P2. Output shape**
- One self-contained HTML file. Inline `<style>`, no JS, no external assets. Opens directly in a browser; works as a file:// share or as a static-hosted artefact.
- Designed for *reading*, not *editing*: clear section headings, sticky table of contents, readable line length (~70ch), meaningful typographic hierarchy (h1→h2→h3 differentiated), syntax highlighting for fenced code blocks, callouts for `> Note` / `> Warning` blocks, collapsible sections for long appendices.
- Header strip shows: feature ID + slug, PRD status (draft / approved / shipped), last-updated date, link back to the feature folder. Footer notes "Generated from prd.md — markdown is the source of truth."
- Banner if the rendered HTML is older than the underlying `prd.md` (mtime drift) so reviewers don't accidentally read a stale view.

**P3. Renderer**
- Use a CommonMark-compliant markdown library plus GFM extensions (tables, task lists, strikethrough, autolinks).
- Custom extensions: callout blockquotes (`> [!NOTE]`), Mermaid diagrams rendered to inline SVG at build-time (no client-side JS), checklist styling for acceptance criteria.
- Asset references in `prd.md` (e.g. `./assets/foo.png`) are resolved relative to the feature folder and either inlined as base64 (small images) or kept as relative links (large ones). Rule of thumb: inline below 50 KB, relative-link above.

**P4. When it runs**
- After `specflow:prd` writes a draft.
- After `specflow:prd` revises an existing PRD.
- After Gate 2 of the adversarial review chain (Appendix N) lands its final accepted version.
- On demand via the manual command.
- During `specflow:upgrade` migration, once for every relocated PRD.

**P5. What it does NOT do**
- Not a PRD editor. Read-only artefact.
- No print/PDF rendering in Phase 1 — if it ships, it ships later (browsers print HTML acceptably out of the box anyway).
- No diff view between PRD revisions in Phase 1 — `git diff` on the markdown is the diff. A side-by-side HTML diff is a Phase 3 candidate if the demand surfaces.
- Not feature-spanning — one HTML file per feature, never a combined index. (The feature folder *is* the index.)

**P6. Wire into existing skills**
- `specflow:prd` — calls the renderer after every write.
- `specflow:upgrade` — invokes `--all` after the feature-folder migration.
- `specflow:render` — the skill itself, also callable directly.
- `specflow:doctor` — flags features whose `prd.html` is missing or older than `prd.md`.

**P7. Open questions**
- Should the rendered HTML be committed to the repo, or gitignored as a derived artefact? Recommend committed: it's small, diffable, lets reviewers click through PRs without running the skill locally.
- Should `tasks.md` and `test.md` get the same treatment, or is the markdown view sufficient for those? Defer until Phase 1 ships and we see whether the readability complaint extends past PRDs.
- Theme/branding: ship one opinionated stylesheet, or expose a slot in `config.json` for projects to point at their own CSS? Recommend opinionated default in Phase 1, configurable hook in Phase 2 if anyone asks.

---

### Q. PRD interview file

**Goal:** every feature folder has a sibling interview file capturing the chronological audit trail of how the PRD was built — original request, codebase findings, the grilling Q&A with each answer's reasoning, and explicit "topics not discussed". Becomes primary context for downstream reviewers (the multi-agent debate manifest, Appendix N) and for humans returning to a PRD months later asking "why did we decide that?"

**Q1. Why this exists**
- The PRD body says *what* was decided. The interview captures *how the decision was reached* and *why each answer became a requirement*.
- Reviewers don't need to reverse-engineer decisions from the PRD body. They read the interview, see the reasoning, and challenge the reasoning directly.
- Found in real use to be the single highest-leverage addition to the PRD format: review quality jumped because reviewers had transparency into the why, not just the what.
- Splitting it out (separate file, not a section in `NNN-{slug}-prd.md`) keeps the PRD body focused on "what we're building" and the interview focused on "how we got here." Two artefacts, two purposes, both committed.

**Q2. Where it lives**
- Inside the feature folder as `features/NNN-{slug}/NNN-{slug}-interview.md`.
- Markdown only — no HTML render. The PRD's HTML render (`NNN-{slug}-prd.html`) includes a header link to the interview file so reviewers can open both side-by-side.
- The PRD body references the interview by relative path but does not duplicate its contents.

**Q3. File structure**

```markdown
# PRD interview — features/NNN-{slug}

## Original request
> [verbatim user ask provided when invoking specflow:prd, in a blockquote]

## Codebase context (pre-grilling)
[What specflow saw before asking anything. Bullet list:]
- Files inspected and what they revealed
- Conventions detected (testing layout, error handling, state mgmt, etc.)
- Existing PRDs/tasks touching adjacent surfaces
- Constraints surfaced from `admin/CONTEXT.md`, `admin/rules/`, `admin/decision-log.md`

## Goal (confirmed before grilling)
[The AI's articulated understanding of what the user is trying to achieve,
 confirmed by the user before any grilling begins. This is the precedent
 every downstream artefact anchors to.]

- **Outcome:** [what changes for the user / system when this is done]
- **Audience:** [who benefits, by canonical profile name from admin/profiles.json]
- **Success looks like:** [the observable, ideally testable, definition of done at the goal level]
- **Driving value:** [why this is worth doing — the business / user pain it removes]
- **Out of scope at the goal level:** [what we are explicitly NOT trying to achieve, named here so grilling doesn't drift into it]

User confirmed: {YYYY-MM-DD}.

## Rounds

### Round 1 — {topic}
- **Q:** [the question, in full]
- **AI's recommended answer:** [what the AI proposed, with reasoning grounded in the codebase context above]
- **User's answer:** [the user's actual response — confirmed, edited, or rejected]
- **Resolved:** [the resolved-assumption that flows into the PRD body — *this is the load-bearing field*. Cites which PRD section it informs, e.g. "→ flows into §Users R1, R2"]

### Round 2 — {topic}
[same shape — `/grill` re-evaluates what to ask next based on prior answers]

[...continues until user signs off on alignment]

## Topics not discussed
[Explicit list of intentional silences. One bullet per topic with a one-line reason:]
- *{Topic}* — [why this was left out: out of scope, deferred to a later PRD, deliberately ambiguous because of {reason}]

This section exists so reviewers can distinguish silence-by-choice from oversight. If a reviewer raises a topic that's listed here, the AI's response cites the reason. If a reviewer raises a topic that's NOT listed here, that's a gap worth investigating.

## Sign-off
- {YYYY-MM-DD} — user confirmed alignment; specflow:prd proceeded to synthesis
```

**Q4. When it's written**
- Phase A of `specflow:prd` writes the preamble (original request + codebase context), then articulates the goal, gets the user's explicit confirmation, and writes the confirmed goal section. Grilling does NOT start until the goal is confirmed.
- Phase B (the grilling) — `/grill` appends each round to the rounds section as the conversation happens. Every recommended-answer reasoning chain grounds itself in the confirmed goal where applicable.
- Phase B closes when the user signs off on alignment; the sign-off line is appended.
- Re-running `/grill` directly on an existing interview file appends new rounds without overwriting prior ones. The sign-off line moves to the latest sign-off. The goal section is NOT re-edited by `/grill` — only `specflow:scope-change` updates it (since a goal change IS a scope change by definition).

**Q5. Wire into the multi-agent debate manifest (Appendix N)**
- Every reviewer reads the interview file first — including the confirmed Goal section. Findings can cite specific rounds AND the goal: *"PRD §R4 introduces a marketing-channel notification — directly contradicts the goal's 'Out of scope at the goal level' line about marketing."*
- The goal section is the primary lens for the Think-Before-Coding Reviewer: every PRD requirement should serve the confirmed goal, and any requirement that doesn't is a finding.
- The Orchestrator's closing decision entry references the goal when explaining why a finding was rejected: *"Reviewer's concern about push notifications is addressed in goal's 'Out of scope' — deferred per app-store review timeline."*

**Q6. Verify steps (for `specflow:prd` to enforce)**
1. `NNN-{slug}-interview.md` exists in the feature folder.
2. *Original request* present and quoted as a blockquote.
3. *Codebase context* non-empty.
4. **Goal section present and confirmed.** Has Outcome, Audience, Success-looks-like, Driving value, Out-of-scope-at-goal-level. User-confirmed date is set.
5. At least one round exists in the *Rounds* section.
6. Every round has a non-empty *Resolved* line (the load-bearing field).
7. *Topics not discussed* present (can be empty list with a note "no intentional silences" — but the heading is mandatory).
8. *Sign-off* line dated and present.

**Q7. Open questions**
- Should the interview be skippable for very small PRDs (e.g. one-paragraph features)? Recommend no — the discipline is what makes the audit trail load-bearing. Tiny PRDs get tiny interviews. The goal-confirmation step is mandatory regardless.
- Should reviewers be able to amend the interview (to add a "topics not discussed" entry) during the multi-agent debate? Likely yes — a reviewer flagging an intentional-silence-that-wasn't-flagged is itself a finding, and the manifest should let the reviewer propose the addition. The amendment becomes a new Round in the interview, attributed to the reviewer. **Reviewers cannot amend the goal** — only `specflow:scope-change` does that, since a goal change is a scope change.
- Diffability across PRD revisions: when a PRD changes substantially, the interview's *Resolved* entries must be re-checked against the new body. Should drift-detection land as part of `specflow:doctor`, or only when `specflow:scope-change` runs? Defer until Phase 1 ships and we see how often PRDs revise.
