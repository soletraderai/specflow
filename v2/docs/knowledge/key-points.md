# specflow — Key Points from External Research

A consolidated synthesis of 14 video transcripts and 6 repo analyses, organized by theme. Each point cites all sources that support it. Tensions and disagreements are flagged for later debate. `[heuristic]` marks claims that are one practitioner's preference without measurement.

## Source index

**Transcripts (14)**
- Matt Pocock — *AI Coding For Real Engineers (Full Workshop)* — https://www.youtube.com/watch?v=-QFHIoCo-Ko
- Matt Pocock — *It Ain't Broke: Why Software Fundamentals Matter More Than Ever* — https://www.youtube.com/watch?v=v4F1gFy-hqg
- Matt Pocock — *How To De-Slop A Codebase Ruined By AI (with one skill)* — https://www.youtube.com/watch?v=3MP8D-mdheA
- Matt Pocock — *Building a REAL feature with Claude Code: every step explained* — https://www.youtube.com/watch?v=hX7yG1KVYhI
- Cole Medin — *Full Archon Guide — Build AI Coding Harnesses That Actually Ship (LIVE)* — https://www.youtube.com/watch?v=srx9iwnjK2M
- Cole Medin — *A Playbook to 10x Your AI Coding with Parallel Agentic Development* — https://www.youtube.com/@ColeMedin
- Cole Medin — *Unveiling the new Archon — open-source harness builder for AI coding* — https://www.youtube.com/@ColeMedin
- Cole Medin — *The WISC Framework — Battle-Tested Claude Code Strategies* — https://x.com/cole_medin/status/1953258783976616423
- Cole Medin — *AI Transformation Workshop — PIV loop* (with Leor Weinstein) — https://www.youtube.com/@ColeMedin
- Nick Saraev — *The Definitive Claude Code Course for Advanced Users* — https://www.youtube.com/watch?v=UPtmKh1vMN8
- Nick Saraev — *Stop Fixing Your Claude Skills. Autoresearch Does It For You* — https://www.youtube.com/watch?v=qKU-e0x2EmE
- Nick Saraev — *Claude Code + Karpathy Autoresearch = The New Meta* — https://www.youtube.com/watch?v=4Cb_l2LJAW8
- Andrej Karpathy — *Skill Issue: Code Agents, AutoResearch, and the Loopy Era of AI* (No Priors) — https://www.youtube.com/watch?v=kwSVtQ7dziU
- Andrej Karpathy — *From Vibe Coding to Agentic Engineering* (Sequoia AI Ascent) — https://www.youtube.com/watch?v=96jN2OCOfLs

**Repo analyses (6)**
- karpathy/autoresearch — https://github.com/karpathy/autoresearch
- coleam00/Archon — https://github.com/coleam00/Archon
- coleam00/GitHubIssueTriager — https://github.com/coleam00/GitHubIssueTriager
- coleam00/context-engineering-intro (WISC) — https://github.com/coleam00/context-engineering-intro/tree/main/use-cases/ai-coding-wisc-framework
- openai/codex-plugin-cc — https://github.com/openai/codex-plugin-cc
- mattpocock/skills + ecosystem — https://github.com/mattpocock/skills

---

## Theme 1: Context engineering (write / isolate / select / compress)

### KP-1: Context rot is the dominant agent failure mode; ~100k tokens is the practical ceiling
- **Point**: LLM quality degrades as context grows ("context rot" / "distractor noise"). Practical "smart zone" ceiling is ~100k tokens regardless of advertised window (200k, 1M). Roughly 80% of agent mistakes trace to overstuffed context, not model weakness.
- **Why it matters**: Single biggest "fewer errors" lever. Sets a hard sizing constraint for every specflow skill, command, and pipeline phase. Reframes the #1 cause of errors for an AI-novice team — discipline > prompt tweaks.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko) — "smart zone / dumb zone", credits Dex Horthy
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423) — cites Chroma "Context Rot" report; 2,000+ hours of Claude Code use
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8) — "zone of good"
  - coleam00/context-engineering-intro WISC repo — https://github.com/coleam00/context-engineering-intro

### KP-2: 1M-token windows are a trap — fitting it doesn't mean using it
- **Point**: Even with Opus/Codex 1M-token windows, perceived performance degrades after a few hundred thousand tokens. Larger windows ship "more dumb zone."
- **Why it matters**: Sets realistic expectations for an AI-novice team. Don't treat large context as license for sloppy loading.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423)
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)

### KP-3: The WISC priority — Write > Isolate > Select > Compress
- **Point**: Four context-engineering strategies in priority order: **W**rite (externalize memory to files), **I**solate (sub-agents for research), **S**elect (load just-in-time, layered), **C**ompress (last resort). "Write and Isolate have the most impact, Select is the force multiplier, Compress is the safety net."
- **Why it matters**: Operating doctrine for managing context. Maps to skills, primes, handoff docs, and per-node context boundaries.
- **Sources**:
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423)
  - coleam00/context-engineering-intro WISC repo

### KP-4: Three-tier context architecture (CLAUDE.md, path-scoped rules, on-demand docs)
- **Point**: Layered progressive disclosure. (Tier 1) Global rules (`CLAUDE.md`), capped ~500–700 lines, always loaded. (Tier 2) Path-scoped rules (`.claude/rules/*.md`) with YAML `paths:` frontmatter, auto-loaded by the harness when matching files are touched. (Tier 3) Reference docs in `.claude/docs/*.md`, never auto-loaded — each opens with a `> Purpose / When to use / Size` header so a scout sub-agent can decide whether to load.
- **Why it matters**: Concrete blueprint specflow can mirror. Lets the docs creator and dev team load only what they need at session start.
- **Sources**:
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423)
  - coleam00/context-engineering-intro WISC repo
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8) — local vs global CLAUDE.md split

### KP-5: CLAUDE.md is four things at once — and has a tight calibration test
- **Point**: A well-built CLAUDE.md is (1) knowledge compression of the workspace (Saraev demos 45x compression), (2) personal/programming preferences, (3) declarations of agent capabilities the model otherwise forgets, (4) a running log of failures and successes. Calibration heuristic: "If removing a line wouldn't cause the AI to make mistakes, cut it."
- **Why it matters**: Single highest-leverage habit for fewer errors and shorter time. Saves tokens and re-derivation.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423)
  - coleam00/GitHubIssueTriager repo — "Non-obvious" CLAUDE.md section pattern

### KP-6: Mirror CLAUDE.md → AGENTS.md → GEMINI.md to diversify away from monoculture
- **Point**: ~70% Claude / ~30% Codex+open-source [heuristic]. Mirror the same context file across providers as cheap insurance against vendor outages or rate-limit collapse.
- **Why it matters**: Anthropic outages have happened (e.g. Dec 17 2025 Opus 4.5 degradation). A 100% Claude team has zero output during incidents.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU) — program.md / AGENTS.md framing

### KP-7: Ubiquitous language file (CONTEXT.md / domain glossary)
- **Point**: A markdown file of domain terms (verbs, nouns, named flows, aliases-to-avoid) shared by humans and the agent. Reduces verbosity, collapses prompts (e.g. "materialization cascade" replaces a paragraph), keeps naming consistent across PRD/code/tests/commits. Borrowed from Domain-Driven Design.
- **Why it matters**: Triple-duty artifact for a small team — docs creator's glossary, dev team's naming guide, AI's planning context. Reduces docs↔code drift.
- **Sources**:
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA) — controlled glossary: Module/Interface/Implementation/Depth/Seam/Adapter/Leverage/Locality
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)
  - mattpocock/skills repo — `CONTEXT.md` + `CONTEXT-MAP.md` patterns

### KP-8: Track token usage live as a discipline
- **Point**: Keep a status line showing exact token count visible during every session. Knowing how close you are to ~100k drives decisions to clear/handoff. [heuristic]
- **Why it matters**: Small habit, big leverage on goal 1 + goal 3. Could be a `/status` or `/tokens` skill.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)

### KP-9: Inject context per-node, not globally (skills, MCPs, rules)
- **Point**: Don't preload every skill/MCP. Bind tools to specific phases — a validation-only skill, a planning-only MCP. Per-node context selection keeps each agent lean and reduces cross-phase confusion.
- **Why it matters**: Direct error-reduction lever. Agent receives only the tools it needs at the moment it needs them.
- **Sources**:
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - coleam00/Archon repo

### KP-10: Push standards to reviewers, let implementers pull them
- **Point**: Implementer agents *pull* coding standards via skills they choose to invoke (keeps context lean). Reviewer agents have standards *pushed* into context (they need to compare against everything).
- **Why it matters**: Design principle for how to structure CLAUDE.md vs skill files vs reviewer prompts.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)

---

## Theme 2: Specs, PRDs, and the discovery phase

### KP-11: Reach a shared "design concept" via aggressive interviewing ("grilling")
- **Point**: Before writing any spec, run a skill that interviews the human one question at a time, with a *recommended answer* for each, until shared understanding emerges. Often 22–80+ questions, ~20–30 minutes. Output is the conversation history (not just a doc), because Q&A pairs are high-attention LLM food.
- **Why it matters**: Misalignment is the #1 failure mode. A grilling step catches assumptions humans miss ("should points be retroactive?"). For an AI-novice team, this is the single highest-leverage move for fewer errors.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin) — "ask me clarifying questions one at a time"
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - mattpocock/skills repo — `grill-me`, `grill-with-docs`

### KP-12: Always tell the LLM the WHY, not just the WHAT
- **Point**: When stating requirements, explaining the *why* lets the LLM suggest alternatives. Only the *what* means it can only build what you said.
- **Why it matters**: Small repeatable rule. Prevents "built exactly what I asked, not what I needed."
- **Sources**:
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)

### KP-13: Two layers of planning — project-level PRD + task-level plan.md
- **Point**: PMs/discovery produce a PRD (high-level: features, scope, target users — *no* code-level detail, "modules and interface changes" only). Then a separate task-level `plan.md` per ticket containing files to touch, patterns to follow, task order, and validation strategy.
- **Why it matters**: Prevents agents diving into code before scope is locked; PRD is reusable across tickets, plan.md is the implementer's contract.
- **Sources**:
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg) — PRDs must name modules and interface changes
  - mattpocock/skills repo — `to-prd` template
  - coleam00/context-engineering-intro WISC `/plan-feature` (5-phase) + `/execute`

### KP-14: PRDs should name modules and interface changes — not file paths or code
- **Point**: PRDs include numbered user stories ("As a / I want / so that"), implementation decisions (modules, interfaces, schema, contracts), testing decisions, out-of-scope, further notes. They explicitly reference modules from the ubiquitous language. They do *not* prescribe file paths or code.
- **Why it matters**: Survives codebase reshuffles. Makes plans reviewable and AI-actionable. Investing in design vs divesting from it (Kent Beck).
- **Sources**:
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - mattpocock/skills repo — `to-prd`

### KP-15: Vertical slices ("tracer bullets") beat horizontal layers
- **Point**: AI defaults to coding all DB → all API → all frontend. That blocks feedback until phase 3. Force thin vertical slices so each issue produces a runnable, reviewable end-to-end result. Same applies to TDD: do `RED→GREEN: test1→impl1, test2→impl2` per slice, not "all tests then all code."
- **Why it matters**: Concrete rule for `/prd-to-issues` skill. Prevents bulk-written tests that test imagined behavior instead of actual behavior.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko) — cites *Pragmatic Programmer*
  - mattpocock/skills repo — `tdd` skill

### KP-16: Don't over-optimize the PRD — the value is in grilling and QA
- **Point**: Polishing a PRD into perfection has low ROI. The real value is upstream (grilling/alignment) and downstream (QA). PRD is a hint of direction.
- **Why it matters**: Avoids wasted iterations refining specs.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko) — "I don't think there's a lot of value in that... the place that you need to be putting the work is in QA"
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI) — explicitly skips reviewing auto-written PRD
- **Tensions / disagreements**: Direct tension with Cole Medin's PIV loop, which treats both PRD and plan.md as **explicit human-reviewable gates** ("it's not good enough to just immediately create stories from that") and with WISC's `/plan-feature` review step. See Headline disagreement #1.

### KP-17: Acceptance criteria must be verifiable; verifiability decides what automates
- **Point**: Every spec should ship verifiable acceptance criteria — not vibes. Verifiability is the single best predictor of automation speed. Models become superhuman where labs can build RL environments with verification rewards (math, code); soft tasks need humans.
- **Why it matters**: Tells the team where to lean hard on agents (tests/types/lints/formal AC) vs keep humans tight in the loop (UX, design tradeoffs).
- **Sources**:
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs)
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU)

### KP-18: Anything prompted >3 times becomes a command or skill
- **Point**: "Anytime you find yourself prompting something more than three times, you should turn it into a command or skill." [heuristic]
- **Why it matters**: Direct rule for specflow content strategy and team adoption.
- **Sources**:
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)

### KP-19: Voice/speech-to-text for brain dumps
- **Point**: Lower friction at the front of the loop with dictation (Whisper Flow, native dictation). Especially helpful for non-developers (PMs, docs creator).
- **Why it matters**: Increases the chance the team actually does the upfront context-loading step.
- **Sources**:
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)
  - Nick Saraev — *Autoresearch Self-Improving Skills* (https://www.youtube.com/watch?v=qKU-e0x2EmE)

### KP-20: Don't review the PRD/issues — trust LLM summarization (controversial)
- **Point**: Pocock explicitly skips reviewing the auto-written PRD and auto-generated issues, on the basis that (a) LLMs are good summarizers and (b) review effort was already spent in grilling Q&A. The trade-off: front-load review into the alignment phase, not the artefacts.
- **Why it matters**: Saves time but trades cycles. This is one of the most contested points in the corpus.
- **Sources**:
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)
- **Tensions / disagreements**: Directly opposed to Cole Medin's "PRD.md and plan.md as explicit gates" (KP-13, KP-16). See Headline disagreement #1.

---

## Theme 3: Self-learning, autoresearch, and the AI layer

### KP-21: Karpathy's autoresearch loop — three ingredients, domain-agnostic
- **Point**: Wrap any artifact in a hypothesis → execute → assess → log loop. Three minimum requirements: (1) an objective metric, (2) an automated measurement tool, (3) something mutable. Without all three the pattern breaks. Karpathy's autoresearch repo: agent edits one file (`train.py`), runs a fixed 5-minute experiment, compares `val_bpb`, keeps or discards via git, ~12 experiments/hour, ~100 overnight.
- **Why it matters**: Generic recipe for hardening every specflow skill. Once wrapped, future model upgrades inherit accumulated improvements automatically.
- **Sources**:
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU)
  - karpathy/autoresearch repo — https://github.com/karpathy/autoresearch
  - Nick Saraev — *Autoresearch Self-Improving Skills* (https://www.youtube.com/watch?v=qKU-e0x2EmE)
  - Nick Saraev — *Autoresearch Cold Email* (https://www.youtube.com/watch?v=4Cb_l2LJAW8)
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8) — references Karpathy + Toby Lütke's Shopify Liquid 53% gains
- **Tensions / disagreements**: Karpathy's NEVER-STOP autonomy norm (no human checkpoints inside the loop) sits in tension with KP-32 (human gates prevent compounding errors). Resolution: autoresearch loops are bounded by a fixed metric and read-only files; pipeline loops are not. See Headline disagreement #4.

### KP-22: Use binary yes/no evals; beware overspecified rubrics (reward hacking)
- **Point**: Binary evals (does X contain Y? is text legible?) are stable. Likert/0–7 scales compound variability. But: too-many narrow rules cause the model to reward-hack and parrot the criteria back. "Like a student who doesn't really understand the material but still gets 100%."
- **Why it matters**: Tells us how to write rubrics for specflow skills. Binary keeps the judge stable, cheap, and resistant to gaming — but rubrics must be broad enough to demand real quality.
- **Sources**:
  - Nick Saraev — *Autoresearch Self-Improving Skills* (https://www.youtube.com/watch?v=qKU-e0x2EmE)
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs) — councils of LLM judges for fuzzy domains

### KP-23: Cost math is tractable — ~$10 to autoresearch one skill across ~50 cycles
- **Point**: ~2 cents per generation × 10 per cycle × ~50 cycles ≈ $10 to optimize a skill from acceptable (32/40, 80%) to near-perfect (39/40, 97.5%). Saraev's website-speed run: 1100ms → 67ms across 67 tests (81.3% gain).
- **Why it matters**: Removes the "we can't afford agentic experiments" objection for a small team.
- **Sources**:
  - Nick Saraev — *Autoresearch Self-Improving Skills* (https://www.youtube.com/watch?v=qKU-e0x2EmE)

### KP-24: Self-healing AI layer — every escaped bug becomes a system improvement
- **Point**: When a bug escapes review or QA, ask: "what in our rules / skills / workflows / CLAUDE.md should change so this never recurs?" Treat context engineering as a living artifact. Four artifacts to evolve: commands, on-demand context, global rules, plan/PRD templates.
- **Why it matters**: Compounding-returns mechanism. The plugin gets better with use rather than rotting.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin) — "we don't just fix the bug and move on, but we fix the underlying system that allowed for the bug"
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin) — "every bug is also a system bug"
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8) — `/insights` slash command across full session history
  - Nick Saraev — *Autoresearch Self-Improving Skills* (https://www.youtube.com/watch?v=qKU-e0x2EmE)
  - coleam00/GitHubIssueTriager `cross-review.md` — promote Codex blind-spot wins to CLAUDE.md rules

### KP-25: Lab notes — make Claude log its own mistakes
- **Point**: Add a single line in CLAUDE.md: "When you have made a mistake, update the CLAUDE.md with a running log of things not to do next time." Pair with: "How could you have arrived at these conclusions and done everything I asked faster?"
- **Why it matters**: Trivial to add, compounds value. Direct hit on fewer errors.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)

### KP-26: The optimization log itself is a durable, model-portable asset
- **Point**: A persistent `resources.md` (or similar) accumulates learnings across runs. Future smarter models pick up where predecessors left off. Consolidate around 500–1000 entries to avoid context bloat.
- **Why it matters**: Mirror specflow's spec memory. Knowledge files are append-and-consolidate logs.
- **Sources**:
  - Nick Saraev — *Autoresearch Cold Email* (https://www.youtube.com/watch?v=4Cb_l2LJAW8)
  - Nick Saraev — *Autoresearch Self-Improving Skills* (https://www.youtube.com/watch?v=qKU-e0x2EmE)
  - karpathy/autoresearch repo — `results.tsv` ledger + git-as-experiment-log

### KP-27: Autoresearch only works for verifiable domains
- **Point**: Auto-research excels where there are objective metrics (kernels, training loss, unit tests, page speed). Soft tasks (clarifying intent, judgment calls, taste) still need humans. "If you can't evaluate, you can't auto research it."
- **Why it matters**: Defines which parts of the dev pipeline auto-loop vs human-gate. Code/perf optimization → autonomous. Spec interpretation, design tradeoffs, UX → humans.
- **Sources**:
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU)
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs)
  - Nick Saraev — *Autoresearch Cold Email* (https://www.youtube.com/watch?v=4Cb_l2LJAW8) — sober gating checklist

### KP-28: A meta-skill that autoresearches every other skill
- **Point**: One optimizer walks the catalog and tunes each skill in turn. Effectively a `/optimize-skills` command for the whole plugin.
- **Why it matters**: One artifact compounds quality across the whole plugin without per-skill engineering.
- **Sources**:
  - Nick Saraev — *Autoresearch Self-Improving Skills* (https://www.youtube.com/watch?v=qKU-e0x2EmE)

---

## Theme 4: Workflow harness / orchestration

### KP-29: Three-stage evolution — prompt → context → harness engineering
- **Point**: Each stage builds on the previous. Harnesses are the deterministic layer wrapping a coding agent — orchestrating sessions, enforcing process, injecting human-in-the-loop, running deterministic gates. Same model + better harness can take PR acceptance from ~6.7% to ~70%. ~40% of leaked Claude Code source is harness code.
- **Why it matters**: Frames specflow's roadmap and value prop. Invest harness energy rather than chasing model upgrades.
- **Sources**:
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8) — Anthropic's Nov 26 2025 "Effective Harnesses" blog as inflection point
  - coleam00/Archon repo
  - openai/codex-plugin-cc repo — bridge plugin pattern

### KP-30: Workflows are DAGs of nodes — deterministic + LLM + human gates
- **Point**: A workflow is a directed acyclic graph. Each node is either deterministic (bash/Python/TS, git ops, test runners, validation gates) or LLM-driven (prompt or command), optionally inside a `loop:` with a stop condition. Some nodes fan out in parallel; join nodes use `trigger_rule: one_success`. The "hybrid secret" is enforcing deterministic nodes the agent might otherwise skip.
- **Why it matters**: Concrete pipeline shape. Steps like "run tests", "load PRD", "QA label" should be deterministic nodes, not things we hope the agent remembers.
- **Sources**:
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - coleam00/Archon repo — `depends_on:`, `trigger_rule: one_success`, `loop … until`, `interactive: true`

### KP-31: Plan / Implement / Validate (PIV) is the foundational loop
- **Point**: Per-ticket loop: Plan (writes plan.md, no code edits) → Implement (fresh session reads plan, no chat history) → Validate (fresh session reads diff, optionally plan). The whole system is just three phases overall: ideate → PIV → system evolution.
- **Why it matters**: Foundational pipeline shape that maps to specflow's existing Plan/Build/Test/Release. The "system evolution" outer loop is continuous improvement.
- **Sources**:
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - coleam00/Archon repo — `archon-piv-loop.yaml`
  - coleam00/GitHubIssueTriager repo — Plan/Implement/Validate as separate Claude sessions
  - coleam00/context-engineering-intro WISC `/plan-feature` → `/execute`

### KP-32: Ralph-Loop failure mode — fully-autonomous loops compound errors
- **Point**: Without checkpoints, mistakes from iteration 1 propagate through the rest of the loop because later iterations build on already-wrong code. Human gates break the compounding.
- **Why it matters**: Core argument for hybrid harnesses with explicit human-approval pauses, not pure AFK autonomy.
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
- **Tensions / disagreements**: Sits against Pocock's Ralph loop (KP-43) and Karpathy's autoresearch NEVER-STOP norm (KP-21). Resolution: depends on whether the loop has a fixed metric and read-only eval surface. See Headline disagreement #4.

### KP-33: Two valid context-passing patterns — continue session vs fresh + artifact
- **Point**: Each node either continues the prior session's context or starts fresh and reads from an `artifact_dir`. Planning writes a plan file; implementation reads it in a new session. Filesystem artifacts beat cramming everything into one session.
- **Why it matters**: Concrete pattern for the PRD → plan → implementation handoff.
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - coleam00/context-engineering-intro WISC `/plan-feature` → `/execute` (plan file is the only context the execute session needs)
  - coleam00/GitHubIssueTriager repo

### KP-34: Per-node model selection (Haiku for routing, Opus for hard reasoning)
- **Point**: Don't run Opus everywhere. Classification/triage/extraction → Haiku. Web research/codebase scan → Sonnet. Implementation/architecture → Opus. Ten-node Sonnet workflow can be cheaper than one Opus invocation. Cole Medin hit only 37% of his 5-hour limit running 4 parallel issue-fix workflows + GSD + interactive PRD.
- **Why it matters**: Direct cost lever. Lets a small team scale parallel agents without runaway spend.
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU) — open-source ~6–8 months behind frontier; reserve frontier for hard problems
  - coleam00/Archon repo — classify-then-route pattern (Haiku classifier → JSON enum)

### KP-35: Build YOUR harness — don't adopt a heavyweight framework wholesale
- **Point**: BMAD, Spec Kit, GSD, Cloudflow, Stripe Minions, Shopify Roast are useful inspiration, not adoption targets. They force teams to change how they work. Build your own thin harness over the patterns you choose; own the stack.
- **Why it matters**: Validates specflow's positioning as a small, opinionated, inspectable plugin rather than a competing framework. Also a self-check: are we becoming the framework people over-rely on?
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko) — "you need to own as much of your planning stack as you possibly can"

### KP-36: Workflows ship with a description for routing (skill-style)
- **Point**: Workflows expose a description (frontmatter) so the coding agent can route to the right one without loading the full YAML. Mirrors Claude Code's skill description pattern. Descriptions are routing metadata, not tutorials.
- **Why it matters**: Confirms the skill-frontmatter approach is the right shape. Also: a "workflow router" lets users say "use specflow to fix issue #42" without memorizing names.
- **Sources**:
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - coleam00/Archon repo

### KP-37: Pair every CLI with a skill or it's invisible to the agent
- **Point**: A CLI without a paired skill won't actually be invoked through Claude. Drop the skill into any repo and the agent can use the harness — no docs to read for the human.
- **Why it matters**: Direct rule: any tooling specflow ships includes a skill alongside it.
- **Sources**:
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)

### KP-38: GitHub issues = task DB; git log = long-term memory
- **Point**: GitHub issues as input contract, PRs as validation input, git commit log as durable agent memory. Avoids needing Linear/Jira for solo/small-team work. Matches Cole Medin's full pipeline.
- **Why it matters**: Cheap, durable memory pattern with no extra infra. Issues enforce write-down of scope; commits write down changes.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin) — `/prime` reads recent git
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423) — `/commit` enforces "what we built + how we improved the AI layer"
  - coleam00/context-engineering-intro WISC repo — enriched commits with `Context:` section

### KP-39: Build human-approval gates *into* the workflow — don't rely on memory
- **Point**: Approval gates are explicit nodes (e.g. `interactive: true` in YAML), not norms humans must remember. Codifies the "no premature pipeline CTAs" principle.
- **Why it matters**: Goal 3 fewer errors. Concrete safety primitive for autonomous loops.
- **Sources**:
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA) — `human-in-the-loop` issue label tells Ralph to skip
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)
  - coleam00/Archon repo — `interactive: true`, `loop … until: APPROVED`

### KP-40: Use the agent to build the workflow ("workflow builder workflow")
- **Point**: Ship a meta-workflow (or skill) that builds workflows. Point it at an existing exemplar, have it ask clarifying questions, generate YAML/markdown.
- **Why it matters**: Accelerates docs creator authoring new flows without learning DSLs.
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - mattpocock/skills repo — `write-a-skill` skill

### KP-41: Issue-vs-PR retroactive diffing detects scope drift
- **Point**: After merge, compare PR's actual changes against the original issue. Plan-vs-implementation drift check. Discrepancies feed back into rule updates.
- **Why it matters**: Cheap audit ritual. Hardens both agent and team's spec discipline.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin) — agent compares output to plan.md
  - mattpocock/skills repo — `triage` state machine

---

## Theme 5: Skill / agent / sub-agent design

### KP-42: Skills are markdown files; description is the routing key
- **Point**: SKILL.md under 100 lines, with `name` + `description` (max 1024 chars, **third person**, "Use when [specific triggers]"). Description is the *only* thing the orchestrator sees when picking a skill — must include trigger keywords. Add scripts when operations are deterministic.
- **Why it matters**: Schema for every specflow skill. Wrong description = skill is never picked.
- **Sources**:
  - mattpocock/skills repo — `write-a-skill`
  - coleam00/context-engineering-intro WISC — `/plan-feature` frontmatter pattern
  - coleam00/Archon repo — workflow descriptions for routing

### KP-43: Skills as the entry point — agents teach themselves the harness
- **Point**: Drop a skill into `.claude/skills` and the agent learns to invoke a CLI/workflow. No docs to read for the human. Lowers onboarding for an AI-novice team.
- **Why it matters**: Reinforces specflow's plugin-as-skills design. Skills are the right primitive.
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423)
  - mattpocock/skills repo

### KP-44: Sub-agents are for research only — not implementation
- **Point**: Sub-agents excel at research (lossy summary is fine, parent receives ~500-token summary). For implementation you need full context of decisions and code written, so keep it in the main agent. Sub-agents may burn 100k+ tokens; main stays at ~40k.
- **Why it matters**: Counter-balances over-eager parallelization. Useful boundary for AI-novice teams learning multi-agent patterns.
- **Sources**:
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423)
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI) — `explore` sub-agent

### KP-45: The "Scout" pattern — pre-decide what context to load
- **Point**: Send a sub-agent to scan a docs folder (`.claude/docs`, Confluence, Drive) and recommend which deep-dive docs are relevant. Then load only those. Each doc starts with a `> Purpose / When to use / Size` header.
- **Why it matters**: Directly applicable to specflow knowledge folders. Prevents loading 10 irrelevant deep-dives "just in case."
- **Sources**:
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423)
  - coleam00/context-engineering-intro WISC repo

### KP-46: Skip elaborate "agent org charts" — sub-agent ≈ skill with fresh context
- **Point**: Both are markdown files with name/description/allowed-tools. Only material difference: sub-agent gets a fresh context window. Skills are role-by-function; sub-agents are role-by-role. CEO/CMO/CTO "agent org chart" patterns (Paperclip, CompanyHelm, OpenGoat, GASTown, SwarmClaude) are mostly anthropomorphism that doesn't help — and they multiply probability of divergence.
- **Why it matters**: Don't waste time building elaborate orgs. Stick to two patterns: (a) parent + Sonnet researchers + Opus QA, or (b) developer + QA loop.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)

### KP-47: Subagent-as-forwarder pattern (don't double-think)
- **Point**: For "delegate to external reviewer" subagents, the body is strict forwarding rules. Allowed exactly one Bash call to an external tool/CLI; must return that stdout verbatim. Not allowed to read files, summarize, or do its own analysis. Avoids "double-thinking" between Claude and Codex.
- **Why it matters**: Direct template for any specflow subagent that wraps an external reviewer (Codex, Gemini, etc.).
- **Sources**:
  - openai/codex-plugin-cc repo — `agents/codex-rescue.md`

### KP-48: Caveman mode — token-saving response style
- **Point**: A skill whose entire purpose is to drop articles, filler, and pleasantries while keeping technical accuracy. Auto-clarity exception: revert to normal prose for security warnings, irreversible-action confirms, multi-step sequences. Reduces hallucination surface area by reducing token output.
- **Why it matters**: Fewer words → fewer chances to drift. Tiny adoption tip.
- **Sources**:
  - mattpocock/skills repo — `caveman`

### KP-49: Don't use the built-in AskUserQuestion tool — token cost
- **Point**: Pocock's grill skill deliberately doesn't call AskUserQuestion because tool calls cost JSON-wrapping overhead. Plain conversation is cheaper.
- **Why it matters**: Cost-tuning insight worth knowing when designing skills. [heuristic]
- **Sources**:
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)
- **Tensions / disagreements**: Cole Medin's PIV loop *recommends* `ask_user_question` because it supports multiple-choice answers. Trade-off: structured choice vs token cost.

### KP-50: CAN/CANNOT framing beats policy paragraphs
- **Point**: Instead of narrative rules, two bullet lists: what the agent CAN do, what it CANNOT do. Agents follow this much more reliably.
- **Why it matters**: Better skill/rule structure for AI-novice teams writing their first guardrails.
- **Sources**:
  - karpathy/autoresearch repo — `program.md` structure (Setup contract → Constraints → Output format → Logging schema → LOOP FOREVER → Autonomy norms)

---

## Theme 6: Quality, verification, errors

### KP-51: Never let an agent grade its own homework — fresh-context review
- **Point**: The reviewer must run in a fresh context window. Same model, same agent — but a new session, fed only the issue + PR diff. Removes in-context bias and sycophancy. Possibly the single biggest quality lever.
- **Why it matters**: Cheap to adopt: `/clear` then `/review-pr`. Direct error reduction.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin) — "like asking a kid to grade their own homework"
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M) — fresh session per phase
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko) — review in smart zone, not implementer's burned context
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8) — spawn unbiased QA agent
  - coleam00/GitHubIssueTriager repo — Plan/Implement/Validate as separate sessions

### KP-52: Cross-provider adversarial review (Claude reviews Codex, Codex reviews Claude)
- **Point**: For high-stakes PRs, run a second review with a *different* coding agent. Cole Medin uses Codex plugin's `/codex:adversarial-review`. Different models have different blind spots; pairing catches more bugs. Findings Codex catches that Claude missed become *new CLAUDE.md rules*.
- **Why it matters**: Direct alignment with specflow's existing Codex plugin. Two cheap reviewers from different distributions beat one expensive reviewer from one distribution.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - openai/codex-plugin-cc repo — `/codex:adversarial-review` with attack-surface taxonomy
  - coleam00/GitHubIssueTriager repo — `cross-review.md`
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8) — Claude/Codex/Gemini diversification

### KP-53: Multi-reviewer fan-out by dimension
- **Point**: Review fans out into specialized agents (code, error-handling, test-coverage, comment-quality, docs-impact, silent-failure-hunter, code-simplifier) running in parallel, then a synthesizer reconciles into severity-bucketed findings. Self-fixes findings rather than just reporting.
- **Why it matters**: Stronger quality bar than single review pass at roughly the same wall-clock time.
- **Sources**:
  - coleam00/Archon repo — 5-parallel-reviewers + synthesize node pattern
  - coleam00/GitHubIssueTriager repo — `review-pr.md` fan-out (4 reviewers in one Task batch)
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8) — fan-out/fan-in pattern

### KP-54: Deep modules with simple interfaces (Ousterhout) — the codebase shape AI thrives in
- **Point**: A *deep module* hides lots of implementation behind a small interface (e.g. TanStack Query). Shallow modules — complex interface, thin implementation — are the antipattern. AI is *very good* at producing shallow-module sprawl. Deep modules are easier to test (test at the interface), give callers high leverage per unit of API learned.
- **Why it matters**: Architectural target for any code specflow generates or refactors. Compounds AI productivity. Two health metrics: high **locality** (changes concentrate in one module) and high **leverage** (more capability per unit of interface).
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA)
  - mattpocock/skills repo — `improve-codebase-architecture`

### KP-55: Seams + adapters — where you test, mock, replace
- **Point**: A *seam* is the location where a module's interface lives — the boundary between modules. Seams are where unit/integration tests and mocks attach. Adapters satisfy seams (hexagonal architecture). A first detection pattern: duplicated parallel implementations across a missing seam (front-end + back-end "insertion point" with no shared contract).
- **Why it matters**: Test harnesses around seams convert "scary changes" into safe ones. Specific code smell to teach review skills.
- **Sources**:
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA)
  - mattpocock/skills repo — `improve-codebase-architecture`

### KP-56: Design interfaces, delegate implementations (gray-box trust)
- **Point**: Humans own the *shape* (module map, interfaces) and delegate the *insides* to AI. Modules become "gray boxes" verified via interface tests. Saves cognitive load. Don't apply for high-stakes domains (finance).
- **Why it matters**: Direct answer to "I know my codebase less well than I used to." Tells AI-novice devs *where* to spend review attention.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI) — review interfaces, not implementations

### KP-57: TDD with vertical slices — AI cheats at tests if you let it write them last
- **Point**: Red-green-refactor with the test written *before* implementation. Otherwise the AI writes implementation, then tests that match its (possibly wrong) code. Bulk-written tests test *imagined* behavior, not actual.
- **Why it matters**: Test skill should enforce TDD ordering, not just "write tests."
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - mattpocock/skills repo — `tdd`

### KP-58: Quality of feedback loops is the ceiling on AI code quality
- **Point**: If `npm test` and `npm typecheck` are slow, flaky, or absent, AI is "coding blind." Investing in fast deterministic feedback loops raises the ceiling more than prompt tweaks. "The rate of feedback is your speed limit."
- **Why it matters**: A "feedback loop audit" should run before anything else does. For AI-novice teams: the bug isn't fixed by reading code, it's fixed by *making the bug observable on demand*.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA) — tests are entry ticket for legacy codebases
  - mattpocock/skills repo — `diagnose` six-phase loop ("the loop is the skill")

### KP-59: Static analysis is not enough — agents must run the app end-to-end
- **Point**: For real validation, agents start the app and use it like a user. Forces solving port conflicts, dep installs, database state.
- **Why it matters**: Sets the bar for "done." Where most "looks good, ships broken" errors come from. Agent self-validation: unit tests + integration + linting + type checks + e2e browser via agent-browser skill.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)
  - mattpocock/skills repo — agent-browser

### KP-60: Tests + types must run on every Ralph-loop commit
- **Point**: Every iteration of an autonomous loop runs tests and types before commit. Without this feedback, agents drift.
- **Why it matters**: Single biggest "fewer errors" lever for AFK agent runs. Non-negotiable in any autonomous mode.
- **Sources**:
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)

### KP-61: Bad codebases produce bad AI output (mutual reinforcement)
- **Point**: Garbage in, garbage out applies to codebases. AI in a bad codebase produces bad code. Investing in codebase quality compounds AI productivity. "AI has simply accelerated software entropy."
- **Why it matters**: Onboarding-to-AI flow for any existing repo should start with: identify deep modules → add seam tests → only then let the agent loose. Periodic architectural-health passes (every couple of days for fast-moving codebases).
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA)

### KP-62: Agent code is bloaty by default — humans must enforce taste/simplicity
- **Point**: RL doesn't optimize for aesthetics or simplicity. Default output has copy-paste, awkward abstractions, brittle patterns. Models "hate" simplifying. Explicit simplification step is required. Borrow Karpathy's autoresearch criterion: "0.001 improvement that adds 20 lines of hacky code? Probably not worth it. 0.001 from deleting code? Definitely keep."
- **Why it matters**: Bloaty code = future bugs. Specflow already has a `simplify` skill — this validates it.
- **Sources**:
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs)
  - karpathy/autoresearch repo — simplicity criterion verbatim
  - mattpocock/skills repo — `zoom-out`, `improve-codebase-architecture`

### KP-63: Models are jagged — brilliant + 10-year-old simultaneously
- **Point**: A model that refactors 100k-line codebases will simultaneously tell you to walk to a 50m car wash. They will burn compute on obviously wrong paths until you intervene. Treat as tools, not colleagues; stay in the loop.
- **Why it matters**: Codifies why review gates and readiness checks shouldn't be optimized away even as capability grows. Concrete example: agent matched Stripe and Google accounts by *email* instead of stable user ID — design-level error a human wouldn't make.
- **Sources**:
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs)
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU)

### KP-64: "Look harder" — push back on the LLM's first wrong answer
- **Point**: When Claude wrongly claims something doesn't exist (e.g. "no test harnesses"), reply "look harder" — and it often finds them. Don't accept the first wrong answer.
- **Why it matters**: Simple, teachable behavior for AI-novice teams. Reduces errors from premature LLM agreement.
- **Sources**:
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)

### KP-65: Don't auto-apply review fixes — present, then ask
- **Point**: After presenting review findings, STOP. Never auto-apply fixes. "You MUST explicitly ask the user which issues, if any, they want fixed."
- **Why it matters**: Prevents silent fix-then-ship loops. Direct alignment with the "no premature pipeline CTAs" rule.
- **Sources**:
  - openai/codex-plugin-cc repo — `codex-result-handling` skill
  - coleam00/GitHubIssueTriager repo — `review-pr.md` "Do not apply fixes automatically"

### KP-66: Structured review output schema (severity, file, line_start, line_end, confidence)
- **Point**: Constrain review output JSON: `{verdict: approve|needs-attention, summary, findings[], next_steps[]}` where each finding has severity (critical/high/medium/low), file, line_start, line_end, confidence (0–1), recommendation. Validate against the schema before rendering.
- **Why it matters**: Eliminates regex parsing. Renders directly as a fix-list with checkboxes.
- **Sources**:
  - openai/codex-plugin-cc repo — `schemas/review-output.schema.json`

### KP-67: Adversarial-review attack-surface taxonomy
- **Point**: Steerable challenge review enumerates: auth, data loss, rollback, races, empty-state, version skew, schema drift, observability gaps. Use as the standard taxonomy for any specflow design/develop critique.
- **Why it matters**: Maps cleanly to "fewer errors" north star. Operator-style XML prompt with `<role>`, `<task>`, `<operating_stance>`, `<attack_surface>`, etc.
- **Sources**:
  - openai/codex-plugin-cc repo — `prompts/adversarial-review.md`

### KP-68: AI accelerates software entropy unless you actively counter it
- **Point**: AI hasn't broken fundamentals — it has accelerated entropy. Every change made without full-codebase context introduces friction that snowballs. Every regeneration drifts the system further. "Bad code is now the most expensive it has ever been."
- **Why it matters**: Argues for periodic architectural-health passes inside specflow's pipeline, not just feature-shipping.
- **Sources**:
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA)

### KP-69: PRD doc-rot — old PRDs poison future agents
- **Point**: Pocock closes PRDs as GitHub issues after merge (visible record, not in active context) because old PRDs misalign future Claude sessions when code drifts from the original spec. He frames it as "a question that doesn't have a clear answer."
- **Why it matters**: Worth explicit decision in specflow: keep, archive, or delete?
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
- **Tensions / disagreements**: Pocock acknowledges this is contested. Cole Medin treats artifacts (plans, handoffs, commits with `Context:`) as durable memory worth keeping. See Headline disagreement #2.

---

## Theme 7: Parallel execution & productivity

### KP-70: One git worktree per agent (filesystem-level isolation)
- **Point**: Spin up a separate git worktree per parallel task. Claude Code supports it natively (`claude -w issue-10`); for other agents, a `w.sh`/`w.ps1` script wraps the same flow. Foundational unblocker for >1 agent.
- **Why it matters**: Without it the team trips over each other; with it they ship multiple features per day without merge chaos.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - coleam00/GitHubIssueTriager repo — `scripts/w.sh` provisioning
  - coleam00/Archon repo — worktree-per-run isolation

### KP-71: Database branching = "worktree for the database"
- **Point**: Codebase isolation alone is insufficient if agents mutate shared DB state. Use Neon database branches (or per-worktree SQLite) so each parallel agent gets an isolated DB copy with production-like data. Sub-second copy-on-write.
- **Why it matters**: Closes the most common "parallel agents broke each other" failure mode.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - coleam00/GitHubIssueTriager repo — Neon branches in `worktree-setup.sh`

### KP-72: Deterministic unique ports per worktree (hash, no registry)
- **Point**: Hash the worktree name into a unique port (base 4000 → 4107, 4161). Stateless, no registry, no race conditions on teardown. ~15 lines.
- **Why it matters**: Tiny detail, big payoff. Removes a flaky failure mode.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - coleam00/GitHubIssueTriager repo — `scripts/assign-port.ts`

### KP-73: Sandboxed AFK runs — Sandcastle / Docker per issue
- **Point**: Run agents inside Docker / Podman / Vercel Firecracker sandboxes with branch-strategy management and merge-back. Pocock's Sand Castle: Planner / Implementer / Reviewer / Merger pattern. Sonnet for implementation, Opus for review.
- **Why it matters**: Safe AFK execution for AI-novice devs — Claude can't damage host repo or leak secrets.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)
  - mattpocock/sandcastle repo — TypeScript SDK
- **Note**: Heavy infra — small team may not need yet.

### KP-74: Fan-out pattern — one planner splits work, N implementers in parallel
- **Point**: One planning agent decomposes a sprint into many GitHub issues. Then fan out N parallel coding sessions, each pointed at one issue. Cole Medin demoed 4 parallel issue-fix workflows + GSD build inside one prompt.
- **Why it matters**: Compresses calendar time without adding headcount. Matches small-team reality: docs creator drives planning fan-out, dev team executes in parallel.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M) — 6 GitHub issues fixed concurrently as background processes
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko) — Kanban DAG enables parallel agents

### KP-75: Issue size — not too big, not too small (4–6 per PRD)
- **Point**: Issues should be neither tiny ("we pay the cost of kicking up an entire agent") nor huge. ~4–6 per PRD, each spanning UI/schema/API as a coherent slice.
- **Why it matters**: Calibration heuristic for slicing. Bad slicing wastes either tokens or context.
- **Sources**:
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)

### KP-76: Token throughput is the new GPU utilization
- **Point**: Anxiety about leftover subscription quota is the new anxiety about idle GPUs. "I feel nervous when I have subscription left over."
- **Why it matters**: Suggests a team metric: are we maximizing concurrent agent token spend? Reframes "AI is expensive" worry into a productivity north-star.
- **Sources**:
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU) — Peter Steinberger's tiled monitor of ~10 Codex agents
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M) — 4 parallel workflows at 37% of 5-hour limit

### KP-77: Day-shift / night-shift — humans grill, agents implement AFK
- **Point**: Human's slow work (grilling/extraction) happens while AFK agents in the background implement the previous session's PRD. "I do the day shift, Claude does the night shift" (credit: Jamon).
- **Why it matters**: Reframes "AI coding is slow." Productivity gain comes from parallelism, not raw speed. For a small team, the docs creator can grill the next feature while devs and agents work the current one.
- **Sources**:
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)

### KP-78: Stochastic consensus + debate patterns
- **Point**: Beyond fan-out/fan-in: (a) **stochastic consensus** — N agents on near-identical prompts, count modes to find robust answers + outliers. (b) **debate** — agents see each other's answers across rounds and refine.
- **Why it matters**: Divergent stochastic runs find solutions one agent misses. Different blind spots = better coverage.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)

### KP-79: 10x is easier than 2x because it forces system-thinking
- **Point**: 2x output is "use AI to type faster"; 10x output requires self-sustaining agents, which forces the systems work above (worktrees, DB branches, fan-out, self-healing). Karpathy: "the new 10x is much bigger than 10x."
- **Why it matters**: Strategic framing. specflow shouldn't sell "type faster" — it should sell the system that makes 10x credible.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin) — cites Sullivan & Hardy
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs)

### KP-80: When *you* become the bottleneck, invest in the AI layer
- **Point**: PR pileup means humans are reviewing too much. Instead of throwing more time at it, lift validation into the agent's automated review/skills layer.
- **Why it matters**: Frames hand-off discipline. The team will hit this wall fast — this is the prescribed escape hatch.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)

---

## Theme 8: Human-AI roles

### KP-81: Engineer's role shifts from typing code to planning and validating
- **Point**: "Our job as an engineer is no longer to write the code, but to do the higher-leverage tasks like planning and validating." The human becomes orchestrator/validator. Karpathy: 0% hand-typed code since December 2025. Move in macro-actions, not lines of code.
- **Why it matters**: Single biggest mindset shift for AI-novice devs. Sets expectation that quality comes from spec/plan/review, not from the LLM "just being smart."
- **Sources**:
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU)
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs) — 80/20 → 20/80 ratio inflection

### KP-82: AI is a tactical sergeant; you are the strategist
- **Point**: Frame AI as an excellent on-the-ground programmer making local code changes. Human stays at strategic level — system design, module boundaries, interfaces, domain language. That role requires the same fundamentals devs have used for 20+ years.
- **Why it matters**: Operating principle for the whole plugin. Docs creator + senior dev are strategists; AI executes. Sets correct expectations for AI-novice devs (don't trust blindly, don't fear obsolescence).
- **Sources**:
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA)

### KP-83: Two task types — Human-in-the-Loop vs AFK
- **Point**: Planning/alignment cannot be looped — humans must sit there. Implementation can be AFK. Designing skills around this distinction is fundamental. Don't pretend planning can be automated.
- **Why it matters**: specflow should explicitly tag steps HITL vs AFK. Architecture-deepening (like `improve-codebase-architecture`) is *explicitly not* an AFK skill — it demands judgment calls.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA)
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI) — `human-in-the-loop` issue label

### KP-84: PMs / docs creators are first-class users of the coding agent
- **Point**: PMs/docs creators are "the first ones that have a touch point with the coding agent" — they own the brain dump → PRD → tickets stage. Ticket descriptions written via this pipeline are often *better* than human-only descriptions because the agent enriches with codebase context.
- **Why it matters**: Direct fit for our docs-creator + dev-team shape. Suggests specflow should explicitly market a docs-creator entry point, not just a developer one.
- **Sources**:
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)

### KP-85: Manual QA is where human taste re-enters
- **Point**: Teams that automate idea→PRD→implement→QA end up with "slop." Humans must QA manually to inject taste, opinion, product judgment. QA *generates new issues* that flow back into the Kanban — QA is not the end of the loop, it feeds the loop.
- **Why it matters**: specflow should not pitch full automation. QA is the human re-insertion point where product taste lives.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko) — "if you try to automate the QA, automate the research... you end up with apps that lack taste and are bad"
- **Tensions / disagreements**: Sits in tension with Cole Medin's "Dark Factory" experiment (KP-95) where humans only file issues. See Headline disagreement #5.

### KP-86: Treat the LLM as a junior teammate you delegate to
- **Point**: Get the most out of Claude Code by treating it like someone you delegate to: focus on architecture, feedback loops, and clear interfaces — not on planning every line up front.
- **Why it matters**: Stops AI-novice devs from either over-prompting or treating Claude as a magic spec-to-code box.
- **Sources**:
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)

### KP-87: "You can outsource your thinking, but you can't outsource your understanding"
- **Point**: The human becomes the bottleneck on knowing what is worth building and why. The unit of contribution from an expert is "a few bits" — the irreducible insight. Everything else (explanation, formatting, variants) is now agent work.
- **Why it matters**: Foundational rationale for why specs matter. Tells team where to spend energy — not on docs polish, not on boilerplate, on the hard-won insight.
- **Sources**:
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs)
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU)

### KP-88: Demand for software is going UP, not down (Jevons paradox)
- **Point**: Cheaper code → more code wanted. Software engineering remains a good bet near term. Counter-narrative to "AI replacing devs."
- **Why it matters**: Reassures the dev team — jobs aren't disappearing, they're being amplified. Counters anxiety that may be slowing adoption.
- **Sources**:
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU)

### KP-89: Personality matters — the agent feels like a teammate
- **Point**: Claude's well-calibrated sycophancy ("praise feels earned") and "soul/identity" document make it feel like a teammate; Codex feels dry. This affects user adoption and how much real work gets done.
- **Why it matters**: For a small team interfacing daily with an agent, configuring persona/voice in skills is not cosmetic. specflow should encode persona/voice. [heuristic]
- **Sources**:
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU)

### KP-90: Hiring/evaluation must be refactored for agentic engineers
- **Point**: Toy puzzles are obsolete. Hand someone a large project and observe how they orchestrate agents end-to-end (build + adversarially test).
- **Why it matters**: Useful framing for how the dev team evaluates progress and onboards future hires.
- **Sources**:
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs)

---

## Theme 9: Tooling & integration

### KP-91: Workspace organization template (`business/active/`, `.claude/`, `.env`, CLAUDE.md)
- **Point**: Top-level workspace per business or client. Inside: `.claude/` (skills, agents), `.env` (secrets), `active/` (all generated artifacts — never pollute root), local `CLAUDE.md`. Skills declare WHERE they write outputs (e.g. `active/model-chat/...`). Periodically run a "clean up active/" prompt.
- **Why it matters**: Prevents workspace rot. For docs-creator + dev-team setup, replicate per-client subfolders.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)
  - coleam00/GitHubIssueTriager repo — `.archon/` + `.claude/` separation

### KP-92: Browser automation tier: HTTP → browser → computer use
- **Point**: Three tiers with setup-time/cost/reliability trade-off. (a) HTTP requests: most setup, fastest, cheapest, fragile. (b) Browser automation (Chrome DevTools MCP, browser-use.com, agent-browser): middle, JS-aware. (c) Computer use (Claude desktop): always works but slow + token-heavy. Start with browser automation to prototype, derive HTTP API contract for production.
- **Why it matters**: A small team will burn weeks if they jump straight to HTTP scraping. Saraev's playbook saves time.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)
  - Nick Saraev — *Autoresearch Cold Email* (https://www.youtube.com/watch?v=4Cb_l2LJAW8) — Chrome DevTools MCP fallback when no API
  - mattpocock/skills repo — agent-browser (snapshot-based accessibility tree)

### KP-93: MCP servers eliminate "backstage" admin work
- **Point**: Atlassian/Linear/GitHub MCP servers create issues, post comments, assign devs, update status, attach PRs. specflow should ship/recommend MCP integrations rather than reinventing ticketing.
- **Why it matters**: Removes meaningful chunks of human grunt work in the team's existing tools.
- **Sources**:
  - Cole Medin — *PIV Loop* (https://www.youtube.com/@ColeMedin)
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)

### KP-94: Subscription-safe usage rules + Codex plugin composition
- **Point**: Anthropic (per Boris Cherny) explicitly allows using Pro/Max subscription with Claude Agent SDK for *personal use*. Banned: third-party harnesses or production-deployed agents. specflow does not need to reimplement Codex — compose with `openai/codex-plugin-cc` (slash commands `/codex:adversarial-review`, `/codex:rescue`).
- **Why it matters**: Risk management. Direct integration path for cross-provider review without reinventing the bridge.
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
  - openai/codex-plugin-cc repo

### KP-95: Anthropic rate limits are tightening — plan for fallback
- **Point**: Claude Code subscription rate limits got significantly worse in 2026. $200 Max plan is ~4x $100 plan, ~20x $20. Even Cole on Max expects to hit weekly limits Tuesday/Wednesday. Plan for multi-provider fallback (Codex, MiniMax M2.7, GLM, Ollama, Pi SDK).
- **Why it matters**: Practical operational risk for a small team relying on subscriptions.
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)

### KP-96: Inline shell directive (`!`) folds dynamic state into prompts
- **Point**: Slash commands can use `` !`...` `` to run shell at invocation and fold state into the prompt without an extra tool call (e.g., `!`ls packages/``, `!`git log -5``).
- **Why it matters**: Cheap dynamic priming. specflow commands could fold current branch / marketplace version / last release inline.
- **Sources**:
  - coleam00/context-engineering-intro WISC repo — `/prime` and `/prime-*` family

### KP-97: Pre-install dependencies & DB branches *before* the agent starts
- **Point**: Worktree setup script pre-installs `node_modules` and provisions a Neon DB branch before Claude opens. Front-loads slow/deterministic setup so LLM-budgeted time goes to actual reasoning.
- **Why it matters**: Productivity multiplier. Shorter agent runs.
- **Sources**:
  - Cole Medin — *Parallel Agentic Playbook* (https://www.youtube.com/@ColeMedin)
  - coleam00/GitHubIssueTriager repo — `worktree-setup.sh`

### KP-98: Setup wizard for credentials runs in a separate terminal
- **Point**: Don't pass API keys through the coding-agent context. Setup runs in a different window so secrets don't enter agent transcripts.
- **Why it matters**: Security pattern. Specflow setup that touches secrets must avoid passing through Claude.
- **Sources**:
  - Cole Medin — *Archon Harness Builder* (https://www.youtube.com/@ColeMedin)
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8) — API keys leak into ~/.claude JSONL plaintext

### KP-99: Markdown for agents, HTML for humans (agent-native docs)
- **Point**: Education and docs are being redirected through agents. Don't write HTML docs for humans — write markdown for agents who explain to humans in their preferred style. "What is the piece of text I copy-paste to my agent?" not "go to this URL."
- **Yields**: Docs-creator's output should target agents as primary audience. specflow plugin docs, internal team docs, even client deliverables → agent-readable first. Structural rework of the docs-creator's job description.
- **Sources**:
  - Andrej Karpathy — *No Priors loopy era* (https://www.youtube.com/watch?v=kwSVtQ7dziU)
  - Andrej Karpathy — *Vibe to Agentic* (https://www.youtube.com/watch?v=96jN2OCOfLs)

### KP-100: TypeScript > Python for agent-edited codebases
- **Point**: Static types help coding agents avoid mistakes — type safety acts as a deterministic guardrail at compile time. Cole Medin chose TS for Archon partly for this reason.
- **Why it matters**: Language/tooling choice is a harness decision. [heuristic]
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)

---

## Theme 10: Pitfalls & anti-patterns

### KP-101: Specs-to-code (spec-only edits, regenerate code) is vibe coding by another name
- **Point**: Pocock explicitly rejects the workflow where you only edit specs and re-generate code. "I tried this. I really tried it and it sucks." Code drifts further with every regeneration. You must continuously shape the codebase; specs alone rot.
- **Why it matters**: Most provocative claim in the corpus. Directly challenges any "spec is source of truth" framing.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)
- **Tensions / disagreements**: Sits against Karpathy's "spec is the new code" framing (KP-17) and Cole Medin's PRD/plan.md gates (KP-13). See Headline disagreement #3.

### KP-102: Prefer `/clear` over `/compact` — Memento, not sediment
- **Point**: Compacting carries "sediment" that pollutes the next phase. Clearing returns to a clean system prompt; pair with persisted artifacts so context rebuilds deterministically. Treat each session like the protagonist of *Memento*.
- **Why it matters**: Skills should write durable artifacts, not rely on long-lived chat memory. WISC: "best compression strategy is not needing compression"; more than 2 compactions = start fresh with handoff.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko) — "I much prefer my AI to behave like the guy from Memento"
  - Cole Medin — *WISC Framework* (https://x.com/cole_medin/status/1953258783976616423)

### KP-103: Auto/yolo mode breaks human-in-the-loop skills
- **Point**: Auto mode "does some funny things with these human-in-the-loop style flows." Skills with approval gates must explicitly disable or warn against auto/yolo modes, otherwise the gate is bypassed.
- **Why it matters**: Aligns with "no premature pipeline CTAs" memory. Goal 3.
- **Sources**:
  - Matt Pocock — *De-Slop Codebase* (https://www.youtube.com/watch?v=3MP8D-mdheA)

### KP-104: Plan mode rushes to ship an asset before alignment exists
- **Point**: Direct critique of Claude Code's default plan mode — it's "extremely eager to create an asset" before alignment. Better: grill first (no asset), then write the asset.
- **Why it matters**: specflow should position its grill/discovery phase *before* plan mode, or replace plan mode for non-trivial work. Concrete differentiator.
- **Sources**:
  - Matt Pocock — *Software Fundamentals Matter More* (https://www.youtube.com/watch?v=v4F1gFy-hqg)

### KP-105: Hallucinated package names are a real attack surface
- **Point**: AI hallucinates npm package names → typosquat malware exfiltrates `~/.claude/`. Audit dependencies for unfamiliar packages.
- **Why it matters**: Concrete security failure for a team new to AI coding.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)

### KP-106: Five low-hanging-fruit security fixes
- **Point**: (1) API keys leak into `~/.claude/` JSONL conversation logs as plaintext — always reference via `.env`, never paste. (2) Hallucinated packages → audit npm deps. (3) Enable Supabase Row Level Security (off by default — Mol Bot RLS-disabled breach). (4) Don't expose agents on naked public URLs (constant bot scans). (5) Never let agents touch credit card numbers — use Stripe.
- **Why it matters**: A small team must internalize these *before* shipping anything client-facing.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)

### KP-107: Don't keep PRDs in the repo after merge — doc-rot
- **Point**: Old PRDs misalign future Claude sessions because code drifts. Pocock closes them as GitHub issues (visible record, not in active context).
- **Why it matters**: See KP-69. Contested decision worth making explicitly in specflow.
- **Sources**:
  - Matt Pocock — *AI Coding For Real Engineers* (https://www.youtube.com/watch?v=-QFHIoCo-Ko)
- **Tensions / disagreements**: See Headline disagreement #2.

### KP-108: Agents have no agency by default — declare capabilities
- **Point**: Agentic Claude forgets what it can do. Without an explicit capabilities section ("you can call this API, you can use Chrome DevTools MCP, you can run for 15 minutes autonomously, you don't have to give me CLI commands to paste — just run them") it underestimates itself and creates needless human-in-the-loop friction.
- **Why it matters**: AI-novice teams will hit "Claude says it can't" walls. A capabilities declaration eliminates the loop.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)

### KP-109: Don't let an alternative harness execute `rm -rf` from a Twitter thread
- **Point**: Some competing harnesses will execute destructive commands from random web sources (real Codex incident). The harness IS where reliability lives.
- **Why it matters**: Don't pick alternative harnesses for daily driver unless you have a specific reason. Diversification ≠ trusting all harnesses equally.
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)

### KP-110: Most challengers lose to baseline — don't kill the loop early
- **Point**: When introducing autonomous iteration, most challengers initially lose to the human-written baseline. Value comes from the long tail. Don't kill the loop after a few losing rounds.
- **Why it matters**: Sets realistic expectations for AI-novice team. Early "AI made it worse" reactions are normal; success is statistical.
- **Sources**:
  - Nick Saraev — *Autoresearch Cold Email* (https://www.youtube.com/watch?v=4Cb_l2LJAW8)

### KP-111: Software quality stops being a moat (3-prediction strategic frame)
- **Point**: Three near-certain predictions from Saraev: (1) Decreasing human involvement is monotonic — auto mode, automated planning, automated QA. Role becomes CEO of a fleet. (2) Software quality stops being a moat — distribution, reputation, legal/compliance, brand replace it. SaaS subs erode because users generate apps themselves. (3) Pace of change accelerates because intelligence improves intelligence.
- **Why it matters**: Strategic. For a small team, invest in distribution and customer relationships, not technical moats. Expect to delete and regenerate rather than maintain. [heuristic]
- **Sources**:
  - Nick Saraev — *Claude Code Advanced Course* (https://www.youtube.com/watch?v=UPtmKh1vMN8)

### KP-112: Specs-to-code misses edge cases — only QA surfaces them
- **Point**: Pure pre-planning will miss edge cases (e.g., directory creation succeeds but git init fails, leaving DB and FS out of sync). You only find them once you're iterating in QA.
- **Why it matters**: Important counterweight to over-rigid spec-driven workflows. Plan for an iterate-in-QA loop, not assume PRD is final.
- **Sources**:
  - Matt Pocock — *Real Feature Build* (https://www.youtube.com/watch?v=hX7yG1KVYhI)

### KP-113: Dark Factory experiment — humans only file issues (controversial)
- **Point**: Cole Medin's "Dark Factory" experiment: codebase where AI handles every commit, review, and release; humans only file issues. He's intentionally removing the human-approval gate for the experiment, *contradicting the rest of his hybrid-secret thesis*. StrongDM's internal lab is pushing thousands of lines to production with no human review.
- **Why it matters**: Aspirational ceiling for productivity. Cole frames it as an experiment, not a recommendation. Read alongside the Ralph-Loop warning (KP-32).
- **Sources**:
  - Cole Medin — *Archon Livestream* (https://www.youtube.com/watch?v=srx9iwnjK2M)
- **Tensions / disagreements**: See Headline disagreement #5.

---

## Headline disagreements to resolve

1. **Should the docs creator review PRDs and issues, or skip review and trust LLM summarization?**
   - Pocock (KP-16, KP-20): Skip reviewing the PRD/issues — front-load review effort into grilling Q&A, trust LLM summarization. "Don't over-optimize the PRD."
   - Cole Medin (KP-13, KP-16): Both PRD.md and plan.md are explicit human-reviewable gates. "It's not good enough to immediately create stories from that — important for us to review the artifact."
   - Tension: where does human review sit — upstream of artifacts (Pocock) or on the artifacts themselves (Medin)?

2. **Keep PRDs and old plans in the repo, or delete them after merge?**
   - Pocock (KP-69, KP-107): Close PRDs as GitHub issues after merge — old PRDs poison future agents because code drifts.
   - Medin (KP-26, KP-38): Treat artifacts (plans, handoffs, enriched commits with `Context:`) as durable memory. Git log is long-term agent memory.
   - Tension: doc-rot vs durable memory. Possibly resolved by archiving vs deleting, but no consensus.

3. **Are specs the source of truth, or is the code the only ground truth?**
   - Pocock (KP-101): "Specs-to-code is vibe coding by another name." Code is the battleground; specs alone rot.
   - Karpathy (KP-17, KP-99): "Markdown for agents, HTML for humans." Detailed specs are the human's primary leverage; agents implement against them.
   - Cole Medin (KP-13, KP-31): PRD + plan.md are explicit gates that drive implementation in fresh sessions.
   - Tension: how much of the workflow lives in specs vs in code? Likely a spectrum where Pocock anchors one end.

4. **Should autonomous loops have human gates, or run NEVER-STOP?**
   - Karpathy autoresearch (KP-21, KP-50): NEVER STOP, don't ask "should I continue?" — bounded by fixed metric and read-only files.
   - Cole Medin (KP-32, KP-39): Ralph-Loop failure mode — fully-autonomous loops compound errors; build human-approval gates into the workflow.
   - Pocock Ralph loop (KP-77): Runs ~100 iterations AFK, but with `human-in-the-loop` issue label as opt-out and tests/types per commit (KP-60).
   - Tension: autonomy is safe iff there's a fixed metric and read-only eval surface; without those, gates are required. No clean line between "research loop" and "pipeline loop."

5. **Should QA be fully automated, or is QA where human taste re-enters?**
   - Pocock (KP-85): "If you try to automate the QA, automate the research, you end up with apps that lack taste and are bad." Manual QA is the human re-insertion point.
   - Cole Medin Dark Factory (KP-113): Humans only file issues; AI does every commit, review, release. StrongDM precedent for production-without-review.
   - Tension: where does human judgment re-enter the loop? Is "taste" an automatable function (KP-22 council of judges) or irreducibly human?

6. **Use built-in `ask_user_question` tool, or plain conversation for grilling?**
   - Cole Medin (KP-11): Use `ask_user_question` for multiple-choice answers — structured, reduces ambiguity.
   - Pocock (KP-49): Avoid `ask_user_question` due to JSON-wrapping token cost; plain conversation is cheaper.
   - Tension: structured choice vs token economics. May be resolvable with measurement.

7. **Build elaborate agent org charts, or stick to two simple multi-agent patterns?**
   - Saraev (KP-46): Skip agent org charts (CEO/CMO/CTO patterns from Paperclip, CompanyHelm, OpenGoat, GASTown, SwarmClaude). Anthropomorphism that multiplies divergence. Stick to: (a) parent + Sonnet researchers + Opus QA, or (b) developer + QA loop.
   - Multi-reviewer fan-out (KP-53): Specialized reviewer agents by dimension (code/errors/tests/comments/docs) work well.
   - Tension: when does role specialization help vs add noise? Possibly resolved by "by function = good, by role = bad."

8. **Adopt a heavyweight framework (BMAD, Spec Kit, GSD), build your own thin harness, or extend an existing harness builder (Archon)?**
   - Pocock (KP-35), Medin (KP-35): Own your stack. Don't adopt heavyweight frameworks — they force teams to change how they work.
   - Medin Archon: Use Archon as the harness builder so teams encode their own SDLC.
   - Tension: where's the line between "owning your stack" and "reinventing the wheel"? Specflow's positioning depends on this answer.

9. **Should specflow's discovery phase replace Claude Code's plan mode, or coexist?**
   - Pocock (KP-104): Plan mode rushes to ship an asset; grill first, no asset.
   - Cole Medin (KP-31): Plan mode is fine, but the artifact (plan.md) lives on disk and is reviewed, not just held in chat.
   - Tension: positioning. Replace, supplement, or wrap Claude Code's plan mode?

10. **Should specflow ship full agent-team / fan-out infra (worktrees + DB branches + ports), or stay lean and let teams adopt later?**
    - Pocock Sandcastle, Cole Medin Archon, GitHubIssueTriager: full parallel infra is the natural endpoint.
    - Pocock himself flags Sandcastle as "heavy infra; small team may not need this yet" (KP-73 note).
    - Tension: when is this complexity earned vs premature? Likely depends on team size and PR throughput trigger (KP-80).
