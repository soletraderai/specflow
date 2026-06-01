# Skills Glossary

Every skill specflow ships, organised by phase. Each entry: one-line purpose · when it triggers · what it produces · what it requires.

New skills cannot ship without an entry in this file. Same rule for standard agents (see `templates/agents/standard/`).

Every skill's SKILL.md frontmatter declares **`requires:`** and **`produces:`** as file-path lists — the lightweight contract orchestrators use to validate handoff between skills. See [`skills/README.md`](./skills/README.md#field-semantics) for the field semantics.

Status legend: ✅ shipped (v1) · 🔧 v2 enhancement · 🆕 v2 addition (operational across Phases 1-3)

---

## Phase 1 — Foundation

### `specflow:setup` 🔧
- **Purpose:** initialise specflow in a project — folders, profiles, environment inventory, rules seeding, self-learning corpus seeding.
- **Triggers:** "set up specflow", "/specflow:setup", first-run detection (no `admin/` folder).
- **Produces:** `docs/specflow/admin/` with config/profiles/environment/rules/agents; empty `features/`, `misc-task/`, `docs/`; CONTEXT.md template; standard agents copied in; `admin/lessons.json` seeded as `[]` (the project's self-learning corpus).
- **Requires:** Playwright CLI (hard); detects Codex, MCPs, plugins, agents.
- **Eval:** `admin/environment.json` exists, has detected stack, hard requirements satisfied; `admin/lessons.json` exists.

### `specflow:prime` ✅
- **Purpose:** prime the codebase context for an upcoming piece of work.
- **Triggers:** "prime for X", "/specflow:prime".
- **Produces:** in-conversation context loaded with the relevant files for the target feature.
- **Requires:** none.
- **Eval:** working set established before downstream skill runs.

### `/grill` 🆕
- **Purpose:** AI interrogates the user one question at a time, recommending an answer per question with reasoning **grounded in the confirmed Goal section**, **re-evaluating what to ask next after each answer**. Sub-skill of `specflow:prd` (auto-invoked Phase B). **Gate 1** of the adversarial review chain.
- **Triggers:** auto-invoked by `specflow:prd` Phase B; manual `/grill {feature-id}` to extend an existing interview file with new rounds.
- **Produces:** appends rounds to `features/NNN-{slug}/NNN-{slug}-interview.md`.
- **Requires:** interview file with **confirmed Goal section** (from `specflow:prd` Phase A); `admin/CONTEXT.md`, `admin/decision-log.md`, `admin/profiles.json`.
- **Eval:** Goal section is confirmed before any rounds fire; every round has Q + AI's recommended answer (with reasoning citing the goal field where applicable) + user's answer + non-empty Resolved line; sign-off line dated.
- **Blocks:** refuses to start if Goal section is unconfirmed; blocks `specflow:prd` Phase C (synthesis) until interview is signed off.

### `specflow:prd` 🔧 (2.14.1: brief auto-invocation removed)
- **Purpose:** user-facing entry point for PRD creation. Four-phase orchestrator: writes interview preamble, **articulates and confirms the goal with the user** (the precedent everything else anchors to), invokes `/grill` for the grilling phase, synthesises the PRD body from the goal + resolved assumptions, fires Gate 2 multi-agent debate manifest. The browser-readable brief is no longer auto-generated — the closing chat-line points the user at `specflow:brief {NNN-slug}` as an explicit next step.
- **Triggers:** "create a PRD for X", "/specflow:prd", overview of what you want to achieve.
- **Produces:** `features/NNN-{slug}/NNN-{slug}-interview.md` (with confirmed Goal section) + `NNN-{slug}-prd.md` + `debate-log/prd-gate2/manifest.md`. No brief — that's manual.
- **Requires:** `admin/profiles.json`, `admin/CONTEXT.md`, `admin/decision-log.md`, `admin/rules/`; optional research files in `features/NNN-{slug}/docs/` and `docs/specflow/docs/`.
- **Eval:** Goal confirmed before grilling; interview signed off; PRD body has no orphan requirements and every requirement serves the goal; Gate 2 manifest closes with Orchestrator sign-off; closing chat-line recommends `specflow:brief` as an opt-in next step.

### `specflow:brief` 🔧 (2.14.1: manual invocation only — no auto-fire upstream)
- **Purpose:** compose a self-contained, browser-readable feature brief by combining the PRD body, the interview transcript, and (when present) the Gate 2 / Gate 3 manifests into one HTML file. Includes a Visual abstract section at the top compiled from `:::flow|comparison|scope|tree` blocks in the PRD markdown.
- **Triggers:** `specflow:brief {feature-id}` — manual user invocation only. As of v2.14.1 no upstream skill auto-fires this one (`specflow:prd` no longer invokes at Phase E; `specflow:scope-change` no longer refreshes at D.3). Bulk re-compose via `specflow:brief --all` still supported (used by upgrade migration).
- **Produces:** `features/NNN-{slug}/NNN-{slug}-brief.html` — inline CSS, no JS, deterministic output. PRD prose, interview Q&A, and manifest content appear verbatim; only the structured visual blocks are interpreted.
- **Requires:** `NNN-{slug}-prd.md` and `NNN-{slug}-interview.md` both exist. Gate 2 / Gate 3 manifests are optional inputs.
- **Eval:** `NNN-{slug}-brief.html` opens cleanly in a browser; deterministic re-compose produces byte-identical output for unchanged inputs; sidebar TOC links resolve.

### `specflow:task` 🔧
- **Purpose:** user-facing entry point for task generation. 5-phase orchestrator: read PRD/interview/Gate-2 manifest → query `admin/lessons.json` for matched prior gaps and surface them → synthesise tasks with forward + reverse coverage → surface 3-5 intent summaries in chat → capture user overrides to `task-history.json` → fire Gate 3 multi-agent debate manifest. Resumes intelligently if invoked on an in-flight feature.
- **Triggers:** `specflow:task {NNN-slug}`, "create tasks for X", "/specflow:task".
- **Produces:** `features/NNN-{slug}/{NNN-slug}-tasks.md` (frontmatter + coverage matrix + tasks; tasks tagged with `lesson-anchor: L-NNN` where applicable); 2-sentence intent summaries for 3-5 highlighted tasks (chat-only, not in file); override records appended to `admin/task-history.json`; `features/NNN-{slug}/debate-log/tasks-gate3/manifest.md`.
- **Requires:** PRD + interview + Gate 2 manifest with passed status; `admin/rules/`; `admin/task-history.json`; `admin/lessons.json` (self-learning corpus, queried on entry); optionally `admin/decision-log.md`.
- **Eval:** tasks file exists with one task per PRD requirement; coverage matrix shows 100% PRD-requirement coverage and zero orphan tasks; every task acceptance is binary; Gate 3 manifest closes with Orchestrator sign-off; any user-driven recut wrote a record to `task-history.json`; every matched active lesson is either anchored to a task or recorded as user-accepted-uncovered.

### `specflow:design` 🆕
- **Purpose:** generate HTML/CSS mockups for a feature, grounded in the live codebase. 5-phase orchestrator: pre-flight + target detection (web/mobile, frame) + mini-interview → value extraction with codebase-truth audit trail (every colour/spacing/etc. cited file:line in the comment block) → generate current.html + proposed.html → Playwright iteration loop (per-iteration decision-capture log entry, empty *Why* is a verify-step failure) → optional Codex semantic review.
- **Triggers:** `specflow:design {NNN-slug}`, `specflow:design {NNN-slug} --iterate`, "create a design mockup".
- **Produces:** `features/NNN-{slug}/design/{NNN-slug}-current.html` + `{NNN-slug}-proposed.html` (each with auditable comment block); `{NNN-slug}-iteration-log.md` (append-only decision capture per PRD Appendix C3.1); screenshots in `features/NNN-{slug}/assets/`.
- **Requires:** PRD; Playwright CLI (hard); component source files; `admin/config.json` (defaults: target, mobileFrame, diffThreshold); Codex CLI (optional).
- **Eval:** both HTMLs exist with comment block listing every value's source file:line; Playwright diff below threshold OR user accepted drift; iteration log captures every change with non-empty *Why*; if Codex available, semantic review recorded in log.

### `specflow:linear` ✅
- **Purpose:** export tasks (and misc-tasks) to Linear with bidirectional sync.
- **Triggers:** "export to Linear", "/specflow:linear".
- **Produces:** Linear issues + the file's Export Map updated with Linear IDs/URLs.
- **Requires:** Linear MCP installed.
- **Eval:** every task has a Linear ID in the Export Map; round-trip status updates work.

### `specflow:test` 🔧
- **Purpose:** verification cadence skill AND project self-learning entry point. Three documented modes: default / `--task` (filter) / `--plan-only` (forced or auto-inferred when no shipped code exists). The 3-phase flow reads PRD/tasks/pages → queries `admin/lessons.json` for matched prior gaps → synthesises a test plan with one binary case per AC + a covering case per matched lesson → executes capturing artefacts. Feedback capture (Phase D) is no longer flag-gated — the skill auto-prompts after a green run, and Phase A.0 detects conversational gap signals to route there directly. Phase D writes a `Retroactive: true` test case + a lesson + an attribution row. Designed to be invoked many times over a feature's life. Owns the lessons-registry schema (defined in this skill's body).
- **Triggers:** `specflow:test {NNN-slug}` (default), `specflow:test {NNN-slug} --task T1,AC-2`, `specflow:test {NNN-slug} --plan-only`. The legacy `--targeted` alias normalises to `--task`; the legacy `--feedback` alias routes to Phase D directly (both retained for one release).
- **Produces:** `features/NNN-{slug}/test/{NNN-slug}-test.md` (frontmatter + coverage matrix + test cases with explicit `Retroactive:` field + append-only execution log); artefacts in `features/NNN-{slug}/test/screenshots/` (Playwright captures) and `features/NNN-{slug}/assets/` (runner output, manual smoke evidence); on Phase D entry also: appended/superseded entry in `admin/lessons.json` (with `.bak`), appended `escaped-issue` row in `admin/task-history.json`; pass/fail summary in chat.
- **Requires:** PRD + tasks closed Gate 3; `admin/pages.json` (UI scenarios); `admin/environment.json` (Playwright + detected runners); `admin/lessons.json` (self-learning corpus). Refuses if Gate 3 not closed.
- **Eval:** every PRD acceptance criterion has a test case; coverage matrix shows 100% AC-to-test traceability; on execution, every non-retroactive test produces a pass/fail signal with a concrete artefact referenced from the test plan; retroactive cases (`Retroactive: true`) are recorded with `⏭ retroactive` status and skipped; lesson-query in B.0 surfaces matched active lessons; Phase D entry produces a schema-valid lessons.json entry plus a covering test case (marked `Retroactive: true`) tagged with the lesson id.

### `specflow:log` 🆕 (2.14)
- **Purpose:** unified entry point for "log something out of band" — the agent classifies the user's free-form intent and routes to one of three internal handlers (`decision` / `misc` / `scope-change`). Replaces the three direct user-facing entry points; auto-invocation contracts from other skills (e.g. `specflow:develop` E.6 → misc) continue to call the handlers directly.
- **Triggers:** `specflow:log {free-form intent}`, `/specflow:log`.
- **Produces:** `admin/scratch/log-{timestamp}/intent.txt` (user prose), `admin/scratch/log-{timestamp}/routing.json` (the classification trace); the dispatched handler produces its own artefacts (`decision-log.md`, `misc-task/`, or PRD/tasks regeneration via scope-change).
- **Requires:** the three internal handlers (`specflow:decision`, `specflow:misc`, `specflow:scope-change`) — installed by default.
- **Eval:** every invocation lands in exactly one handler or returns cleanly after a user abort; routing.json records the chosen handler + confidence + prompt outcome; the handler's own eval block governs the resulting artefacts.

### `specflow:misc` 🔧 (2.14: internal handler — invoke via `specflow:log`)
- **Purpose:** single-task workflow for bugs, small fixes, and out-of-scope-but-shouldn't-be-lost observations. Two invocation modes — interactive (driven by `:log` dispatch with prose pre-filled) and auto (structured payload from another skill, typically the Surgical Reviewer flagging a rule violation that should not be fixed inline). Initialises the rolling file if missing, allocates the next MISC-NNN id, appends the entry, optionally saves assets, updates the Quick reference + Export map tables.
- **Triggers:** `specflow:log "{description}"` (preferred user surface); `specflow:misc --auto {payload-path}` (internal auto-invocation from other skills); direct `specflow:misc` invocation supported for the deprecation runway.
- **Produces:** new entry in `docs/specflow/misc-task/000-tasks-misc-tasks.md` (Pending tasks + Quick reference + Export map); optional asset under `misc-task/assets/`; for auto mode, a result file at `admin/scratch/misc-result-{timestamp}.json` for the calling skill to consume.
- **Requires:** `admin/config.json`, `admin/rules/`. Linear export (separate skill) targets `000-misc-tasks` project.
- **Eval:** new entry has unique MISC-NNN id; required fields populated; for auto-created entries the rule reference + why are present; any referenced assets exist on disk; rolling file passes structural lint.

### `specflow:upgrade` 🆕
- **Purpose:** refresh an aged specflow installation without losing customisation. Drives version-to-version migrations.
- **Triggers:** "upgrade specflow", "/specflow:upgrade", version-skew detection.
- **Produces:** applied migrations from `MIGRATIONS.md`; refreshed `environment.json`; backup `.bak` files for every modified file.
- **Requires:** none.
- **Eval:** `config.json.specflowVersion` matches plugin version; doctor passes.

### `specflow:doctor` 🆕
- **Purpose:** read-only installation validator. Five-category check pass — install layout, config schema, standard-agent set, feature integrity (PRD/HTML drift, gate closure), environment + hard requirements. Produces a report with PASS / FAIL / WARN per check and concrete remediation for every failure.
- **Triggers:** `/specflow:doctor`, `specflow:doctor --category {install|config|agents|features|environment}`, `specflow:doctor --feature {NNN-slug}`. Auto-invoked by `specflow:upgrade` post-migration and (Phase 2) by `specflow:develop` on entry.
- **Produces:** in-chat report; for auto-invocations, also `admin/scratch/doctor-report-{timestamp}.md` whose path is returned to the calling skill.
- **Requires:** `admin/config.json`, `admin/environment.json`, `plugin.json`. Read-only — never modifies files.
- **Eval:** every FAIL line cites concrete file/line/symptom; every FAIL has a Fix line; overall status PASS / PASS-WITH-WARNINGS / FAIL matches the check tallies.

### `specflow:budget` 🆕
- **Purpose:** visibility on subscription consumption AND per-skill context-window cost. Skills self-report at completion via append-only `skill-invocations.jsonl`; budget reads, aggregates, and flags trending-up Δ tokens as the early-warning signal for orchestrator-pattern violations. Honest about what's measured (self-reported estimates + provider billing where exposed) vs. unavailable.
- **Triggers:** `/specflow:budget` (full), `--tokens` (per-skill only), `--spend` (subscription only), `--window {7d|30d|90d|all}`, `--skill {name}`.
- **Produces:** `admin/budget/usage-summary.md` (human-readable rolling report), `admin/budget/per-skill-tokens.json` (structured aggregation with trend + leak_signals).
- **Requires:** `admin/budget/skill-invocations.jsonl` (skills append on completion); `admin/environment.json` for provider billing-surface paths.
- **Eval:** report covers every skill invoked in the window (no silent omissions); subscription rows mark `unavailable` rather than fabricating; `rising` trend flags reference `templates/orchestrator-pattern.md` audit checklist; jsonl is append-only.

### `panic` 🆕
- **Purpose:** trust-ladder primitive — the big red button. Snapshot-then-rewind with mandatory user confirmation (literal phrase `yes, rewind` required). Snapshot captures diff, status, untracked index, scratch archive, in-flight gate paths. Never destructive without confirmation; never touches `features/` or `admin/` (other than `scratch/`).
- **Triggers:** `/panic` (default rewind to HEAD), `/panic --to {ref}` (named target, requires second confirmation), `/panic --snapshot-only` (no rewind).
- **Produces:** `docs/specflow/admin/panic-snapshots/{timestamp}/` with diff.patch, status.txt, untracked.txt, recent-commits.txt, branch.txt, head-sha.txt, scratch-archive/, inflight-gates.txt.
- **Requires:** git repo.
- **Eval:** snapshot exists with required files; working tree clean of changes from the panicked session OR user explicitly declined the rewind step; user confirmation captured before destructive actions.

### `confidence-check` 🆕
- **Purpose:** trust-ladder primitive — AI declares uncertainty in plain language before acting, names ≥1 specific source of uncertainty (assumption / missing data / conflicting signal), blocks until explicit user confirmation. Inline-invoked by other skills before risky actions; manually invokable. Logs every declaration + response to `confidence-log.json` for Phase 3 self-learning.
- **Triggers:** inline payload from another skill (`{calling_skill, action_summary, category, uncertainty_sources, reversibility, blast_radius}`); manual `/confidence-check`.
- **Produces:** structured declaration in chat; new entry in `admin/confidence-log.json`; for inline mode, a result file at `admin/scratch/confidence-result-{timestamp}.json` returned to the caller.
- **Requires:** none.
- **Eval:** declaration is plain-language (no jargon, no internal-reasoning dump); names ≥1 specific source of uncertainty; blocks until explicit confirmation; declaration + response logged.

### `feedback-loop-audit` 🆕
- **Purpose:** 5-phase audit of the rate of feedback the codebase provides — test coverage, type strictness, e2e health, lint/format gating, build pipeline. Generates the slim `admin/CONTEXT.md` (≤500 line target, 700 hard cap) as the seed for the project's live context document. Preserves user-maintained blocks across regenerations.
- **Triggers:** `/specflow:feedback-loop-audit`, `--report-only` for preview; auto-invoked by `specflow:setup` and `specflow:upgrade`.
- **Produces:** `admin/CONTEXT.md` (live document), `admin/feedback-loop-audit-{timestamp}.md` (full audit report).
- **Requires:** `admin/environment.json`; codebase access (Glob + Read for stack detection).
- **Eval:** every reference in CONTEXT.md resolves on disk; length within target/cap; audit report covers all five dimensions with concrete signals; rerun is idempotent (preserves user-maintained blocks).

### `simplify` 🆕
- **Purpose:** Karpathy-style discipline-installer — bounded autoresearch loop. Branch-per-run, sequential variant generation (default 5, max 10), read-only machine eval (lines deleted + tests + lints + types), no LLM-as-judge inside the loop, human merge owns taste. Pre-flight gates refuse oversized scopes (>50 files), broken baselines, dirty working trees, and budget exhaustion. The deliverable that matters most is the *pattern*, not the immediate output.
- **Triggers:** `/simplify {scope}`, `--variants {N}`, `--dry-run`, `--override-budget {reason}`; scheduled via GH Actions.
- **Produces:** PR on `simplify/{scope-slug}-{ts}` branch with eval results table + variants table; new entry in `admin/simplify-runs.jsonl` (pending merge_decision).
- **Requires:** clean working tree, green baseline eval, project-detected test/typecheck/lint commands, budget under $20/week.
- **Eval:** PR exists with passing CI; weekly spend under cap; human merge decision recorded in simplify-runs.jsonl. **No auto-merge.**

---

## Phase 2 — Development

### `specflow:develop` 🆕
- **Purpose:** orchestrate implementation of tasks generated from a PRD. Lane-based execution (green/yellow/red); integrates agent teams; runs Gates 4 and 5.
- **Triggers:** "develop {task-id}", "/specflow:develop", Linear status change to "In Progress".
- **Produces:** code changes, PR, plan-vs-PRD anchor commentary, debate transcripts for Gates 4 and 5.
- **Requires:** Phase 1 substrate; agent indexing; standard agents available.
- **Eval:** Verifier confirms task acceptance criteria pass; Gate 5 (Codex code review) signs off.

### `specflow:agent` 🆕
- **Purpose:** manage the per-repo agent registry — `add`, `remove`, `list`, `refresh`.
- **Triggers:** "/specflow:agent {verb} {agent}".
- **Produces:** snapshot files in `admin/agents/specialised/`; updated `index.json`.
- **Requires:** scanned global agent index across installed plugins.
- **Eval:** every snapshotted agent resolves to an installed source plugin; `doctor` passes.

---

## Phase 3 — Memory

### `specflow:complete` 🆕
- **Purpose:** retro skill — captures task outcome at completion; feeds the self-learning loop. Final chat-line includes a soft prompt to run `specflow:test {slug}` and mention any gap discovered on review (Phase A.0 intent detection routes the gap to Phase D).
- **Triggers:** "/specflow:complete {task-id}", invoked manually or via Linear webhook.
- **Produces:** entry in `task-history.json`; significant patterns appended to `decision-log.md`; soft chat-line reminder about the test-skill feedback flow after every successful retro.
- **Requires:** completed task with PRD anchor.
- **Eval:** entry has all required fields (id, scope, AI assistance level, what worked, what didn't, blast-radius outcome); feedback-prompt chat-line emitted on every successful retro write.

### `specflow:decision` 🔧 (2.14: internal handler — invoke via `specflow:log`)
- **Purpose:** lightweight skill for recording an out-of-band decision (library pin, architectural reversal, convention ratified). Six interactive prompts, single append-only write to `decision-log.md`.
- **Triggers:** `specflow:log "{description}"` (preferred user surface); `specflow:decision "{title}"` for direct invocation during the deprecation runway and from `specflow:scope-change` G7.
- **Produces:** entry in `decision-log.md`.
- **Requires:** none.
- **Eval:** entry has title, context, decision, rationale, date, related files/tasks.

### `specflow:scope-change` 🔧 (2.14: internal handler — invoke via `specflow:log`)
- **Purpose:** capture mid-development scope changes — *why* the intent changed, what the PRD now needs to say, which tasks regenerate, what in-flight work is impacted.
- **Triggers:** `specflow:log "{description}"` (preferred user surface; the agent routes here when intent is scope-change-shaped); auto-suggested by `specflow:develop` on detected drift (auto-invocation contract unchanged — routes here directly).
- **Produces:** updated PRD; regenerated affected tasks; impact list for in-flight work; `decision-log.md` entry.
- **Requires:** active feature with existing PRD/tasks.
- **Eval:** PRD diff is reviewable; affected tasks have updated coverage; impact list cites every in-flight artefact.

### `/insights` 🔧 (2.14: background loop — scheduled cadence, not part of feature workflow)
- **Purpose:** surface recurring patterns from `task-history.json` (monthly cadence).
- **Triggers:** scheduled cron (primary); manual `/insights` invocation for ad-hoc runs.
- **Produces:** report; suggested rule registry promotions (observation → guideline → non-negotiable).
- **Requires:** populated `task-history.json`.
- **Eval:** suggestions cite at least three observations per promotion.

### `/prune` 🆕
- **Purpose:** prune stale rules, decisions, and snapshots (quarterly cadence).
- **Triggers:** "/prune", scheduled cron.
- **Produces:** archive of pruned items; updated registry.
- **Requires:** registry with at-least-one-quarter age.
- **Eval:** pruning is reversible (archive retained); user signs off before delete.

### `/optimize` 🆕
- **Purpose:** autoresearch loop across the verifiable-skill set (~6 initial targets: `release-version-check`, `simplify`, `format`, `tdd-cadence`, `init`, `feedback-loop-audit`).
- **Triggers:** "/optimize", overnight GH Actions.
- **Produces:** PR with optimised prompt for one skill; eval results; review summary.
- **Requires:** read-only eval surface; the target skill must have a binary eval.
- **Eval:** machine eval improves; human merge decision.

---

## Standard agents

Shipped with the plugin in `templates/agents/standard/`. Copied into every new project's `admin/agents/standard/`. Two categories.

### Lifecycle agents — `templates/agents/standard/lifecycle/`

Cover the plan / challenge / confirm moments of any non-trivial work item. **Set kept deliberately small**; additions only when a clearly distinct lifecycle moment emerges that none of the three covers.

#### Orchestrator
- **Role:** coordinates multi-agent workflows; holds the plan; delegates to specialists; keeps multiple points of view in play. **Also writes the closing decision entry in every multi-agent debate manifest** (Appendix N).
- **Lifecycle moment:** plan + final-call.

#### Devil's Advocate
- **Role:** challenges decisions in-flight — surfaces blind spots in PRDs, scope ambiguity in tasks, architectural gotchas during development. One of the parallel reviewers fired into the debate manifest at every gate.
- **Lifecycle moment:** challenge (during the work).

#### Verifier
- **Role:** confirms work meets the bar at the end of any task. Reads the original requirement and the produced output; verifies they match.
- **Lifecycle moment:** confirm (at completion).

### Principle reviewers — `templates/agents/standard/principles/`

Fire as parallel reviewers in the multi-agent debate manifest (Appendix N). One reviewer per core principle from `CORE_PRINCIPLES.md`. **Set scales 1:1 with the principle list** — adding a fifth principle requires adding a fifth reviewer.

#### Simplicity Reviewer
- **Principle enforced:** Simplicity First (incl. AI-specific sub-tests: explicit beats clever, local reasoning beats cross-file elegance).
- **First question at every gate:** *"Is there a simpler way to achieve the acceptance criteria?"*
- **Common findings:** premature abstraction, speculative configurability, defensive error handling, clever-over-explicit code.

#### Surgical Reviewer
- **Principle enforced:** Surgical Changes.
- **Common findings:** "while I was here" edits, missed `misc-task` creation when out-of-scope rule violations were spotted, style normalisation creep, refactor smuggling.

#### Think-Before-Coding Reviewer
- **Principle enforced:** Think Before Coding.
- **Common findings:** silent assumptions, missing tradeoff articulation, confident-wrong-shaped answers, hidden confusion.
- **Particularly load-bearing at:** Gates 1 (grill), 2 (PRD), 4 (plan).

#### Goal-Driven Reviewer
- **Principle enforced:** Goal-Driven Execution.
- **Common findings:** soft verification ("looks correct"), end-of-pipeline test phase, missing `eval:` field, acceptance criteria without test cases.
- **Particularly load-bearing at:** Gates 3 (tasks), 5 (code), 6 (tests).

### How they're used together

Every adversarial-review gate (Appendix N) fires the principle reviewers + Devil's Advocate in parallel into a shared **debate manifest** file. AI responds in round 2; reviewers sharpen or accept in round 3; **Orchestrator writes the closing decision entry**. The manifest is queryable — Phase 3 self-learning reads it for patterns (which reviewers' findings are accepted most often → well-calibrated; always rejected → over-aggressive prompt to retune).
