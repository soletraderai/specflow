# Session Handoff — specflow PRD for Codex Adversarial Review

**Date:** 2026-04-30
**Audience:** Codex (or another reviewer running an adversarial pass)
**Goal of the handoff:** give a fresh reviewer everything needed to challenge the work produced this session and surface improvements.

---

## What you're being asked to do

Run an adversarial review of the specflow plugin roadmap (`PRD.md`) and the supporting research/debate that informed it. Specifically:

1. Read the PRD end-to-end. Find weak claims, internal contradictions, and missing pieces.
2. Cross-check the PRD against the source research (`knowledge/key-points.md` plus the per-source analyses) — has anything important been dropped or distorted?
3. Pressure-test the major design decisions against the team's three north-star goals: more productive, better product in shorter time, fewer errors.
4. Surface what's missing for the team's specific situation — a documentation creator who works alone, plus a development team new to AI-assisted coding, on a confidential codebase.
5. Write your findings to `knowledge/codex-review.md` (or similar) with structured sections: weak claims, contradictions, missing pieces, suggested improvements, and explicit pushbacks on individual decisions.

Be unsparing. The north-star goals are the only judge. Don't flatter the existing analysis.

---

## Quick context

**What specflow is:** a Claude Code plugin that helps a small team go from idea → PRD → tasks → development → test, with self-learning memory and adaptive per-project behaviour. Reference implementation: ClaimXPro (`/Users/marepomana/Web/ClaimXPro/docs/specflow/`).

**Audience:**
- A documentation creator who works alone (writes specs, owns intent integrity, currently overwhelmed when reviewing 50+ tasks per PRD)
- A development team new/very new to AI-assisted coding, located overseas
- Repo is confidential — every issue/change requires human sign-off; protected paths must always be human-led

**North-star goals (the only judge):**
1. More productive
2. Better product in shorter time
3. Fewer errors

---

## What this session produced

In rough order:

1. **Initial roadmap discussion** — clarified per-repo isolation (global plugin, local state per project), confirmed agent registry pattern with Verifier as third standard agent, expanded design skill requirements (web/mobile, codebase-truth principle, Playwright iteration, Codex adversarial review), added environment inventory as first-class concept.

2. **Renamed `todo.md` → `PRD.md`** and restructured as a proper PRD (vision, problem, goals, principles, three-phase approach).

3. **Research synthesis** — read 14 YouTube transcripts (Pocock ×4, Medin ×5, Saraev ×3, Karpathy ×2) and 6 repos (autoresearch, Archon, GitHubIssueTriager, codex-plugin-cc, context-engineering-wisc, mattpocock profile). Produced `knowledge/key-points.md` with 113 consolidated key points across 10 themes plus 10 headline disagreements.

4. **Multi-perspective debate** — 4 position papers (one per author's worldview), then a CEO Round 1 challenges document picking apart each position, then 4 rebuttals (each conceded their weakest claim and pushed back where they had ground), then CEO Round 2 final recommendations.

5. **Strategic planning conversation** — covered task-execution model (green/yellow/red lanes by triage tuple, not AI confidence), the docs-creator-as-bottleneck problem and how to make their job easier, tracking architecture, adversarial review chain with 3-iteration debate, the four behavioral principles from `forrestchang/andrej-karpathy-skills`, the rules registry concept.

6. **Major PRD update** applying all decisions and the CEO synthesis pseudo-diff.

---

## Major decisions made (the load-bearing ones)

### Architectural / Conceptual

- **Three-phase model.** Phase 1 = Foundation (lightweight + critical scaffolding). Phase 2 = Development (`specflow:develop` + agent layer). Phase 3 = Memory and self-learning. Was originally two phases; user chose three to separate the develop layer from the self-learning loop.
- **Per-repo isolation contract.** Plugin code lives globally in `~/.claude/plugins/`. Per-project state lives in `docs/specflow/admin/` and is committed to that repo. Each project adapts independently — no cross-project memory.
- **Behavioral principles (4) embedded in every skill.** Adopted from `forrestchang/andrej-karpathy-skills`: Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution. Loaded into every skill's system prompt as non-negotiable.
- **Rules registry as separate concept.** `admin/rules/non-negotiable.md` + `admin/rules/guidelines.md` + `admin/rules/glossary.md`. Drives auto-`misc-task` creation on violation. Self-evolves in Phase 3.
- **Standard agents kept small.** Orchestrator + Devil's Advocate + Verifier (each owns a different lifecycle moment: plan / challenge / confirm).
- **Environment inventory.** `admin/environment.json` captures CLIs (Playwright required, Codex optional), MCPs, plugins, agents available locally. Generated at setup, refreshed by upgrade and doctor.
- **Skills glossary.** `plugins/specflow/SKILLS.md` lists every skill with one-sentence purpose, when it triggers, what it produces. New skills can't ship without an entry.

### Workflow / Execution

- **Adversarial review chain — six gates phased across Phase 1-3.** Gate 1 = `/grill`. Gate 2 = PRD vs brief. Gate 3 = tasks vs PRD. Gate 4 = plan vs tasks + PRD anchor. Gate 5 = code vs plan (cross-provider Codex). Gate 6 = tests vs requirements.
- **3-iteration debate loop at every gate.** Reviewer fires (first question always: *is there a simpler way?*); AI defends or revises with reasoning; reviewer counters; up to 3 rounds; **Claude makes the final call** because it owns the writing/development. Full transcript saved per gate.
- **Green / Yellow / Red lane execution.** Each task gets a triage tuple (verifiability + blast radius + dependency state + confidentiality classification). Confidentiality is rule-based (path globs in `config.json`), never AI-rated. Lanes:
  - Green: batched, AFK-eligible, single batched human sign-off.
  - Yellow: HITL paired, one at a time.
  - Red: human-led, AI assists on bounded subtasks only.
  - Initial target ratio 60/30/10 (G/Y/R). Skewed ratios are themselves a learning signal (PRD needs re-cutting).
- **Testing as cadence, not terminal step.** Every skill that produces output runs verification *inline* (Playwright loops on every UI change). No separate test phase at the end.
- **Surgical changes + auto-misc-task on out-of-scope quality issues.** AI doesn't fix unrelated problems inline; auto-creates a `misc-task` with the rule reference, file:line, observation, and *why*. Quality concerns don't fall through cracks while change set stays clean.
- **Coverage matrix + intent summary at task creation, in chat (not as file artefact).** When `specflow:task` runs, AI shows the docs creator a 2-sentence non-technical intent summary for 3-5 highlighted tasks, plus a coverage matrix mapping PRD requirements to tasks. Single sign-off line replaces 50-task review.
- **Override capture.** Every recut, scope change, or correction by the docs creator logs to `task-history.json` for self-learning (consumed in Phase 3).
- **`/grill` skill blocks plan mode** until alignment file exists. Iterates rounds (no fixed cap) — runs until docs creator says ready.
- **Dev plan ↔ PRD anchor.** Every plan starts with *"We're doing X because of PRD requirement Y. This aligns with Z."* — devs get PRD context for free.

### What's NOT in scope

- **Team-to-team / human-to-human handoffs.** Handled out-of-band (Loom recordings + messaging). User explicitly said this isn't specflow's job.
- **Dashboards.** Metrics captured (rolling `metrics.md` per PRD) but no UI for humans.
- **Auto-research loop across the catalog.** Deferred. Only one bounded autoresearch loop ships in Phase 1 (on `simplify`) as a discipline-installer; `/optimize` extends to six verifiable skills in Phase 3.
- **Worktree parallelism, DB branching.** Gated on real reviewer pile-up trigger AND `panic` skill having shipped. Not phase-gated, trigger-gated.
- **Cross-project knowledge sharing.** Each repo learns alone.
- **Pixel-perfect design tooling.** Mockups are alignment artefacts; codebase-truth principle ensures fidelity to live app, not aesthetic perfection.

### Other notable decisions

- **Migration timing:** `config.json` + `pages.json` move into `admin/` happens in Phase 1 alongside `specflow:upgrade` (so existing installs have a clean path).
- **Goodharting guard:** "the metric is a signal, not a goal." Benchmarks kept as low as possible so improvements only show when real. The team's actual judgment is ground truth.
- **Per-skill binary eval block.** Mandatory `eval:` field per skill describing the success check. Cultural artefact (Saraev's discipline-installer).
- **Codex usage scope:** standing adversarial-review backend across the whole pipeline (design Phase 1; plan/code Phase 2; tests Phase 3).
- **Trust-ladder primitives in Phase 1, not Phase 3.** `panic` (rewinds, kills agents, snapshots), `confidence-check` (AI declares uncertainty before acting), `getting-started` profile (slim onboarding) — ship before parallelism, not after.

---

## File roadmap (what to read, in what order)

### Tier 1 — primary deliverables

| Path | Purpose |
|---|---|
| `PRD.md` | The main artefact. ~1100 lines. Read end-to-end first. |
| `knowledge/key-points.md` | 113 consolidated key points from research, organized by 10 themes, plus 10 headline disagreements. The factual basis for the PRD's decisions. |

### Tier 2 — debate artefacts (the reasoning trail)

| Path | Purpose |
|---|---|
| `knowledge/debate/position-pocock.md` | Pocock-perspective position paper (engineer's pragmatism, grill-me, deep modules) |
| `knowledge/debate/position-medin.md` | Medin-perspective (harness engineering, PIV loop, Archon DAGs) |
| `knowledge/debate/position-saraev.md` | Saraev-perspective (autoresearch, measure-everything, no agent org charts) |
| `knowledge/debate/position-karpathy.md` | Karpathy-perspective (spec-as-leverage, autoresearch-as-loop, jagged intelligence) |
| `knowledge/debate/ceo-round-1-challenges.md` | CEO Round 1: picked apart each position, adjudicated 10 tensions, surfaced blind spots |
| `knowledge/debate/rebuttal-pocock.md` | Pocock's rebuttal (conceded the "skip PRD review" claim) |
| `knowledge/debate/rebuttal-medin.md` | Medin's rebuttal (conceded YAML DSL approach; PIV survives as convention not schema) |
| `knowledge/debate/rebuttal-saraev.md` | Saraev's rebuttal (conceded `/optimize` as catalog flagship; survives scoped to verifiable skills) |
| `knowledge/debate/rebuttal-karpathy.md` | Karpathy's rebuttal (conceded "code disposable" overgeneralized; spec-as-intent survives) |
| `knowledge/debate/ceo-final-recommendations.md` | Closing document. Phase-mapped roadmap, four watchwords, risks, PRD pseudo-diff. |

### Tier 3 — research analyses (knowledge/analysis/)

**Transcript analyses (14):**
- `pocock-ai-coding-real-engineers.md` — workshop arguing classical SE fundamentals + grill-me beat spec-driven pipelines
- `pocock-software-fundamentals-matter-more.md` — code is not cheap; design concept before any plan asset
- `pocock-deslop-codebase-deep-modules.md` — deep modules + seams; `improve-codebase-architecture` skill
- `pocock-real-feature-build.md` — full feature build via grill → ubiquitous-language → auto-PRD → Ralph loop → manual QA
- `medin-archon-livestream.md` — Archon harness builder (livestream)
- `medin-archon-harness-builder.md` — Archon concept overview
- `medin-parallel-agentic-playbook.md` — five pillars for parallel agentic dev
- `medin-piv-loop.md` — Plan-Implement-Validate; system evolution; >3-prompts-rule
- `medin-wisc-framework.md` — Write/Isolate/Select/Compress context engineering
- `saraev-claude-code-advanced-course.md` — long advanced course on CLAUDE.md, harnesses, parallelization
- `saraev-autoresearch-self-improving-skills.md` — wrap skills in Karpathy autoresearch
- `saraev-autoresearch-cold-email.md` — generic autoresearch loop applied beyond code
- `karpathy-no-priors-loopy-era.md` — code agents, AutoResearch, the "loopy era" of AI
- `karpathy-vibe-to-agentic.md` — vibe coding → agentic engineering shift (Sequoia AI Ascent)

**Repo analyses (6):**
- `repo-karpathy-autoresearch.md` — autoresearch loop primitives (program.md, git-as-experiment-log)
- `repo-coleam00-archon.md` — DAG-of-nodes harness; `context: fresh`, `loop until` patterns
- `repo-coleam00-github-issue-triager.md` — parallel-agent toolkit (worktree+DB-branch+port-hash)
- `repo-coleam00-context-engineering-wisc.md` — WISC framework with `paths:` frontmatter
- `repo-openai-codex-plugin-cc.md` — exact mechanics of Codex bridge plugin for Claude Code (HOW to invoke from specflow)
- `repo-mattpocock-profile.md` — Matt Pocock's repos (`mattpocock/skills` cousin to specflow; `sandcastle`; `graph-docs-cli`; `git-guardrails`)

### Tier 4 — supporting

| Path | Purpose |
|---|---|
| `CLAUDE.md` | Project rules (no Claude/Anthropic mentions in user-facing output; release version sync rules) |
| `knowledge/repo.txt` | Original repo links (six URLs that drove the repo research phase) |
| `knowledge/{slug}.json` | The 14 source transcript JSON files (renamed by content) — only useful if you need to verify a quote against original source |

---

## Open questions still in the PRD

1. **Rule registry starter set** — which non-negotiables ship by default? Candidates: no hardcoded values, no comments without WHY, never bypass auth, protected paths require Red lane. Confirmed at setup with user.
2. **`pages.json` ownership** — manual, setup-generated, or own skill?
3. **Misc task rotation** — single rolling file vs periodic.
4. **Retro trigger (Phase 3)** — manual `/specflow:complete` vs Linear webhook polling.
5. **Task history privacy** — committed (default) vs gitignored.
6. **Agent snapshot refresh strategy** + cross-marketplace name collisions (e.g. two plugins ship a `frontend-developer`).
7. **Design mockup readback** — should later skills (PRD, task) consume design HTML as context?

---

## Specific things I want you to challenge

These are areas where I'm least confident the current PRD is right. Push hard.

1. **Phase 1 has 17 priority items.** Is it actually still "lightweight"? Or has it become a Phase 1.5 in disguise? If it should be cut, what's the smallest defensible cut?

2. **The 3-iteration debate loop with Claude as final tiebreaker.** Is "Claude makes the final call because it's the writer" too generous to the AI? Should the human always be the tiebreaker on confidential-repo work? Or is human-as-tiebreaker exactly the bottleneck we're trying to remove?

3. **Rules registry vs behavioral principles** — is there overlap that creates confusion? The behavioral principles govern *how the AI behaves*; the rules registry governs *what the AI accepts in this codebase*. But "no comments without WHY" feels like a behavioral principle of Surgical Changes. Where's the line, and is it defensible?

4. **Confidentiality classification by path globs** — is path-glob enough, or does the system need semantic classification too? A file in `src/utils/` could touch auth indirectly. What's the failure mode?

5. **`/grill` runs until docs creator says ready, no cap.** Saraev would call this unbounded. Is there a soft cap (e.g., warn at 30 questions) that protects against grill-fatigue without imposing a hard cutoff?

6. **Coverage matrix + intent summary in chat (not file).** Is "in chat" the right channel? It's ephemeral. Should there be a paper trail somewhere for the docs creator's sign-off? Or is the chat-only delivery exactly the point (don't accumulate dead artefacts)?

7. **Green/yellow/red ratio target of 60/30/10.** Where did this come from — is it grounded, or a heuristic the docs creator made up that will distort early phase planning?

8. **Pocock's "delete PRDs after merge" idea was rejected** by keeping them as durable memory. But the PRD now has both: PRDs persist + decision-log captures decisions. Is there double-bookkeeping risk?

9. **The four behavioral principles plus the rules registry plus `CONTEXT.md` plus `CLAUDE.md` plus `SKILLS.md`** — that's five places where AI guidance lives. Is this still slim enough, or has the system accumulated rule files?

10. **Phase 1 ships `/grill`, `/budget`, `panic`, `confidence-check`, `getting-started`, `simplify` autoresearch, `feedback-loop-audit`, design, upgrade**, plus enhanced `setup`/`prd`/`task`. That's ~10 net new skills in Phase 1. Counter-claim: Phase 1 is now *Phase 2* and the real lightweight foundation has been postponed.

---

## Adversarial review brief (what to write)

Output to `/Users/marepomana/Web/specflow/knowledge/codex-review.md` with sections:

1. **TL;DR** — 5 sentences. Net assessment.
2. **Weak claims** — list each, with evidence.
3. **Internal contradictions** — places the PRD says two incompatible things.
4. **Missing pieces** — what should be in the PRD that isn't, especially for the team's specific situation.
5. **Pushbacks on specific decisions** — pick the 5-10 you'd most question; for each: the decision, your objection, your alternative.
6. **What you'd cut** — the smallest defensible Phase 1 if you had to ship in 4 weeks.
7. **What you'd add** — anything the four advisors and the CEO missed.
8. **Cross-checks against research** — places where the PRD's claims don't match `key-points.md` or the source analyses.

Be unsparing. Don't flatter. Match the energy of the CEO Round 1 document — that's the bar.

---

## Notes on tone

The PRD has been through:
- 4 advisor position papers
- 1 CEO Round 1 challenge document
- 4 rebuttals (each conceded their weakest claim)
- 1 CEO Round 2 synthesis
- A long planning conversation with the user adding constraints

It's been polished, but four of those rounds were arguing with itself — which means the PRD may have accumulated everyone's compromises. A fresh adversarial pass is exactly what catches the things internal debate normalised.
