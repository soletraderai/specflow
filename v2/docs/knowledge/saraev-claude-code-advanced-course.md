# The Definitive Claude Code Course for Advanced Users

**Author:** Saraev
**Source:** YouTube course chapter (~3h 18m, full course)
**Duration:** 3:18:25

## Executive Summary
Saraev's full-length advanced Claude Code course, mostly demoed inside Anti-Gravity with the Claude Code extension. Eight functional chapters: (1) what `CLAUDE.md` actually *is* and how to evolve it, (2) agent harnesses, (3) parallelization patterns (fan-out/fan-in, debate, sub-agents), (4) skills and sub-agents as two flavours of the same thing, (5) Karpathy-style autoresearch applied to business outcomes, (6) browser/computer automation tradeoffs, (7) handling Claude performance fluctuations through diversification, (8) workspace organisation, security, and a closing essay on the future. The thread running through it: skill at this stack is not configuration tweaking; it's understanding the harness, the model, and the organisational hierarchy of agents well enough to design context flow deliberately.

## Chapter 1 — What a `CLAUDE.md` actually is

Saraev opens by reframing system prompts. A `CLAUDE.md` is four things simultaneously:

1. **Knowledge compression** — a bird's-eye summary so Claude doesn't have to walk the file tree on every query.
2. **Personal preferences and conventions** — the things Claude doesn't yet bake in natively (file path returns, programming style, library defaults).
3. **A declaration of capabilities** — "Hey Claude, you *can* do X, Y, and Z autonomously." Without this, Claude often underestimates its own agency. He gives the example of asking how long a feature would take, getting a "3 months" estimate, and having to remind it: "no, *you're* building it, you can do it in 5 seconds."
4. **A log of failures and successes** — accumulated learnings that prune the solution space. Visualised as carving 80% of dead branches out of the theoretical solution-space "planet" so Claude focuses tokens on the 20% that matters.

> **Direct from video:** "What a cloudmd is is it's a declaration of capabilities... half the time, okay, if it's not in your cloud and cloud will just look at you metaphorically... and it will say like, oh, like I don't have a built-in way to do this."

**Global vs local scope.** Global (`~/.claude/CLAUDE.md`) carries high-level reasoning, who you are, conventions, and capability declarations. Local (`.claude/CLAUDE.md`) carries the project description, where things live, project-specific API docs, project capabilities. Critically he notes you should not just inherit a system prompt from someone else — Claude entities are personal.

**Workflow for evolving them.** Plan a feature → instantiate it → the run produces failures and successes → compile those into compressed high-information-density additions → update `CLAUDE.md`. Repeat. Same loop globally and locally.

## Chapter 2 — Agent harnesses

Brief but pointed. He cites Anthropic's November 26, 2025 blog post *Effective Harnesses for Long-running Agents* as the inflection that "kicked off Claude Code superiority over most other harnesses." The takeaway: knowing Claude Code at the harness level — not just the prompt level — is the moat. He treats this chapter as a prerequisite primer rather than the deep dive (the rest of the course is built on the harness mental model).

## Chapter 3 — Parallelization

Why parallelise: (a) autonomous agents now run for 15+ minutes, so sitting and waiting is expensive idle time; (b) many tasks have independent steps; (c) agents are **stochastic** — same query, slightly different answer each time, and running 5 in parallel widens the coverage of the answer distribution; (d) **performance degrades with context length**, so keeping each sub-agent's context narrow keeps every one of them in the "zone of good."

Three concrete patterns:

1. **Fan-out / fan-in** — orchestrator spawns N research sub-agents on slightly different angles; a synthesizer fans them back in. He demos this with a `use a fan out fan in and researcher synthesizer approach... minimum five sub agents use sonnet to do the research... opus to synthesize` prompt.
2. **Model mix by step** — Sonnet for research (cheap, token-heavy, low reasoning), Opus for synthesis (smart, low token volume). He notes ~60% direct cost saving at the input-token level, plus throughput wins.
3. **Stochastic consensus / debate** — run the same query N times in parallel and union the unique answers. Each run only gets ~3-5 of the possible findings; 5 parallel runs surface 10-15+.

> **Direct from video:** "If I ran, you know, Claude five times on basically the exact same thing, every single time I have a slightly different response... you can actually run multiple times with the same or similar queries and then you can actually have different answers given to you that just sort of live outside of the distribution."

## Chapter 4 — Skills, sub-agents, and the org-chart fallacy

The most unifying argument in the course: **skills and sub-agents are two flavours of the same thing**. Both are markdown files with a name, description, allowed tools, and an SOP. The real difference is sub-agents get a fresh context window; skills don't (but skills are written compressed enough that they push toward a short context anyway).

> **Direct from video:** "Sub agents are honestly basically skills and skills are basically sub agents. They're just slightly different ways of storing information."

He critiques the "AI org chart" trend — Paperclip, Company Helm, OpenGoat, GasTown, SwarmClaude, the system, etc. — as misguided anthropomorphism: "Why would we try and fit agents, which think very differently than human beings, into the exact same organizational hierarchies we've been using for the last 150 years?"

His **two recommended delegation patterns** (everything else is bloat):

1. **Parent + researchers + QA.** Opus orchestrator → multiple Sonnet researchers (fan out) → Opus QA agent reviews → loops back. Lean, role-aware, takes advantage of model-cost asymmetries.
2. **Developer + QA.** Just two: a smart parent develops, a fresh-context QA agent reviews after every cycle. The QA's lack of context is the *feature* — it isn't biased by the parent's accumulated story about the project.

> **Direct from video:** "The QA has like literally no prompt other than, you know, you're a QA agent with no context. Read this code and apply the following whatever like design principles to it."

He warns about the multiplicative probability of divergence: every additional agent hop without a human in the loop multiplies the chance of drifting from the original goal.

## Chapter 5 — Autoresearch (Karpathy method)

A condensed version of the dedicated autoresearch videos, here grounded in a concrete demo: improving the load speed of his website `leftclick.ai`. He uses the Google Lighthouse score (LCP, FCP, TBT, performance score) as the objective metric and lets the loop iterate. By the time he revisits it later in the course, the auto-researcher has driven the site to ~8000ms from a baseline of 1802 (he frames the takeaway as "imagine running this 3,000 days in a row").

The reusable recipe: **objective metric + measurement tool + thing to change + tight feedback loop**. He emphasises this is what every major frontier lab is doing internally to train their models — democratised through Karpathy's released repo.

## Chapter 6 — Browser and computer automation

A practical decision tree across three tiers, ordered by cost/setup/robustness:

1. **HTTP requests** — fastest, cheapest once set up, but websites actively try to block you and it requires understanding internal APIs.
2. **Browser automation** (Chrome DevTools MCP, Browser Use) — middle ground. Works on rendered DOMs, can click and fill forms. Browser Use specifically markets undetectability via fingerprinting; Saraev says it's worth it for stuff that breaks under Chrome DevTools MCP — Facebook scraping, social media DMs, Instagram, anything with strong anti-bot.
3. **Computer use** (Claude desktop app) — controls mouse and keyboard, screenshot loop. Always works, but very expensive in tokens and slow. Demos it renaming a downloaded file in Finder.

His default flow: try Chrome DevTools MCP first → if blocked try Browser Use → use Chrome DevTools MCP's network tab to extract the underlying API, then switch to HTTP requests for production volume. Notes the ToS implications and explicitly does not endorse anything.

## Chapter 7 — Performance fluctuations and diversification

Frames Claude Code as a **monoculture risk** using the *Interstellar* / corn-blight analogy. Claude Code is the best harness; therefore most users put 10/10 productivity eggs in it; therefore when Anthropic has an outage (he cites the Opus 4.6 outage "yesterday" and the December 17, 2025 Opus 4.5 garbage-collection regression), 95% of developer productivity craters globally.

Recommendation: 7/10 eggs in Claude, 1-3 eggs spread across **Codex, Anti-Gravity (Gemini), pi/local-model harnesses**. Concretely: write `agents.md` and `gemini.md` alongside `CLAUDE.md` and keep them synchronised so any of those harnesses can pick up your project mid-stride during an outage.

> **Direct from video:** "If a model is like 1% better than another model that 1% once you get smart enough is like the difference of like a gulf right Einstein is like 1% smarter than a normal human being or something like that. So if you have the ability to use the best model, just use the best model. But don't put all your eggs in that basket."

## Chapter 8 — Workspace organisation

His personal scheme:

- Top-level `business/` and `personal/` workspaces (with VS Code header colour distinguishing them).
- Inside `business/`: `.claude/skills/`, `.claude/CLAUDE.md` (local), `.env`, an `active/` folder for everything generated.
- Sub-folders for each client: `business/clientA/.claude/skills/`, `business/clientA/.env`, etc. Client-specific skills stay scoped to clients.
- **Never pollute the root.** Always store generated files under `active/<skill-name>/`. Each skill's `skill.md` declares exactly where its outputs go ("dump to `active/model-chat/`, named like X").
- Periodically run a "clean my active folder" prompt to reorganise loose files into sub-folders or delete temp ones.
- Personal workspace mirrors the structure but groups by life-domain (`citizenship/`, `health/`) instead of by client.

He flags client-skill access from a business workspace via a `CLAUDE.md` line that tells Claude: "Some skills aren't in `.claude/skills/`; if you need a client-specific one, look inside the client folder I'm referencing."

## Security (mini-chapter inside Workspace)

80/20 framing — these are the cheap wins:

- **Enable Row-Level Security on Supabase.** It's off by default. Cites the "Moltbook" leak: a security researcher had read+write to every AI-agent profile and created 100,000 fakes in seconds because RLS wasn't on.
- **Don't run a public-facing OpenClaw.** Bot farms scan all open IPs constantly for vulnerabilities.
- **Never let an agent see a credit card number.** It will end up in a transcript, a git commit, a log file. Use Stripe; let them be the PCI-compliant party.
- **Run a security audit prompt** in a fresh conversation (no context bias) after every project.

## Auto Mode and the future of human involvement

He uses Anthropic's recent **auto mode** (a fourth permission-handling option beyond ask-before-edits, edit-automatically, and bypass-permissions) as a microcosm of the trend: things that used to require a human in the loop are getting absorbed into the model. Planning, implementation, QA — historically each had a human gate. Each gate is collapsing.

> **Direct from video:** "Rather than have a 100 people do a task in some specific company like we used to have, we might have one person do 100 tasks. Leverage will go up."

His broader thesis (the closing 10-minute essay):

- **Decreasing human involvement** in the develop → plan → implement → QA loop is the dominant arrow.
- **Software products are no longer the moat.** "I can code Netflix in 5 minutes with three or four agents on fast mode." The moat is now distribution, brand, and direction — not the artifact.
- **Productivity divide.** Quoting William Gibson — "the future is here, it's just unevenly distributed." The 1% who actually understand harnesses, models, autoresearch, and skill systems will reap asymmetric returns over a small window.

> **Direct from video:** "You right now even if you don't have a lot of money have access to insane technology and leverage simply because you're in it."

## Key Exact Extracts

> **[00:05]** "I use Claude Code and AI agents in my own business every day to generate over $4 million a year in profit."

> **[07:13]** "Sub agents are honestly basically skills and skills are basically sub agents. They're just slightly different ways of storing information."

> **[44:50]** "Why would we try and fit agents, which think very differently than human beings, into the exact same organizational hierarchies we've been using for the last 150 years?"

> **[44:30]** "If I ran, you know, Claude five times on basically the exact same thing, every single time I have a slightly different response."

> **[55:22]** "If you parallelize your agents, you can actually run multiple times with the same or similar queries and then you can actually have different answers given to you that just sort of live outside of the distribution."

> **[2:09:37]** "The performance of cloud goes up so too is your entire productivity. If the performance of cloud goes down so too is your entire productivity."

> **[2:21:39]** "Don't pollute the root. Always store an active or subdirect root."

> **[3:04:00]** "My second one is more of like an economic uh consideration which is that software products and tools, the quality of the things that you build will no longer be the moat."

> **[3:16:48]** "The future is here. It's just unevenly distributed."
