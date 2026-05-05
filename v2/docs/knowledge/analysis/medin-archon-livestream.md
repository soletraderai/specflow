# Archon Live: Harness Engineering as the Next Evolution of AI Coding

## Identity
- Title: Full Archon Guide — Build AI Coding Harnesses That Actually Ship (LIVE)
- Speaker/Channel: Cole Medin (@ColeMedin / coleam00) — livestream
- Likely URL: https://www.youtube.com/watch?v=srx9iwnjK2M
- Suggested slug: cole-medin-archon-harness-builder-livestream
- Confidence: high

## Thesis
AI coding has evolved from prompt engineering (2022) to context engineering (2025) to **harness engineering** (2026). A "harness" is the deterministic layer that wraps a coding agent — orchestrating sessions, enforcing process, injecting human-in-the-loop, and running deterministic gates. Same model + better harness can take PR acceptance from ~6.7% to ~70%. Archon is positioned as the first open-source *harness builder* (not a single opinionated harness like BMAD/Spec Kit/GSD) so teams can encode their own SDLC into reusable, parallel-runnable workflows that combine LLM nodes, deterministic script nodes, loops, and human gates.

## Key points

### KP-1: Harness > model/tool improvements
- **Point**: Most reliability gains in 2026 come from the layer wrapping the agent, not from swapping models. Cole cites studies showing PR acceptance climbing from 6.7% to ~70% from harness alone, and notes Sonnet-in-Archon often outperforms raw Opus-in-Claude-Code.
- **Why it matters to our goals**: Directly addresses goal 3 (fewer errors) and goal 2 (better product faster) — invest harness energy into specflow rather than chasing model upgrades.
- **Evidence**: "you can go from a 6.7% pull request acceptance rate to almost 70% and the only thing is the harness"; "I've had better results using Archon with Sonnet than I have using Opus by itself."
- **Sources**: lines 226–235, 681–702.

### KP-2: Three-stage evolution: prompt → context → harness engineering
- **Point**: Each stage builds on the previous. Prompt engineering is part of context engineering, which is part of harness engineering. Harnesses combine sessions, not just curate context within one.
- **Why it matters**: Frames specflow's roadmap — we should treat skills/commands/rules (context engineering) as ingredients, and the workflow that orchestrates them as the harness layer.
- **Evidence**: "context engineering does not replace prompt engineering... harness is a layer on top of the coding agent. It's the tooling and the process you build on top of the coding agent to make it more reliable."
- **Sources**: lines 134–224.

### KP-3: Stop "shepherding" between sessions
- **Point**: Without a harness, humans manually carry artifacts between fresh planning, implementation, and review sessions because keeping them in one session builds bias. Harness automates the hand-off while preserving fresh context per stage.
- **Why it matters**: Our docs creator + dev team is exactly the audience that loses time to manual session shepherding. A harness encodes the right cadence (plan → implement → validate) so juniors don't have to memorize it.
- **Evidence**: "you have to walk the different coding agent sessions through each one... shepherding."
- **Sources**: lines 318–352.

### KP-4: Hybrid secret — deterministic nodes + human gates
- **Point**: Archon's "hybrid secret" is mixing deterministic script nodes (bash/Python/TypeScript), LLM nodes, and explicit human-approval pause states. Deterministic nodes guarantee testing/linting actually happens; human gates prevent error compounding (the Ralph-Loop failure mode).
- **Why it matters**: Goal 3 (fewer errors). Maps directly to specflow's existing "readiness check" and QA-label patterns — we should keep adding deterministic gates rather than trusting prompts.
- **Evidence**: "sometimes you'll tell the coding agent run the tests... and it won't listen... we can have certain steps where we're just running code... we're guaranteeing that happens after the implementation."
- **Sources**: lines 463–541.

### KP-5: Ralph-Loop failure mode (compounding errors)
- **Point**: Ralph-Loop-style fully-autonomous loops compound first-iteration mistakes — later iterations build on already-wrong code. Human checkpoints break the compounding.
- **Why it matters**: Justifies specflow's anti-vibe-coding stance and the existing "no premature pipeline CTAs" feedback memory.
- **Evidence**: "if the coding agent makes a mistake in the first iteration of the Ralph Loop, that issue can propagate and blow up from the rest of the loop because it builds on top of a code base that's already wrong."
- **Sources**: lines 510–541.

### KP-6: Build YOUR harness, don't adopt someone else's
- **Point**: BMAD, Spec Kit, GSD, Stripe Minions, Shopify Roast are valuable as *inspiration*, not as adoption targets — they force teams to change how they work. Archon lets you encode your existing process.
- **Why it matters**: For a small team new to AI coding, adopting BMAD wholesale is overkill ("enterprise theater"). specflow should pitch itself as the team's own harness, not a competitor framework.
- **Evidence**: "they require you to pretty much change how you work fundamentally. That's just not going to fly, especially if you're working on a team."
- **Sources**: lines 269–302, 3115–3182, 3284–3340.

### KP-7: Per-node model selection for token efficiency
- **Point**: Don't run Opus everywhere. Classification/triage nodes can use Haiku, implementation Opus, validation Sonnet. Cole hit only 37% of his 5-hour Claude limit while running 4 parallel issue-fix workflows + GSD workflow + interactive PRD.
- **Why it matters**: Goal 1 (more productive) — small team needs to stretch limited Anthropic subscription budgets across many parallel tasks.
- **Evidence**: "when you're classifying or investigating, maybe you only need to use Haiku... only the case that a single node like the fix issue is the only one where you'd want to use Opus."
- **Sources**: lines 660–702, 2925–2990, 4416–4445.

### KP-8: Parallelism via worktree isolation
- **Point**: Archon spawns each workflow in its own git worktree so 4–30+ workflows can run simultaneously without merge conflicts. Cole demoed 4 parallel GitHub-issue-fix → validate-PR chains in one prompt.
- **Why it matters**: Goal 2 (better product in shorter time). Pattern specflow could borrow for parallel doc-creator and dev-team workflows.
- **Evidence**: "I can have it use the CLI to fix eight GitHub issues at the exact same time. They all run in different work trees so they don't step on each other's toes."
- **Sources**: lines 415–429, 2080–2144, 3208–3214.

### KP-9: GitHub issues as task management + Git log as long-term memory
- **Point**: Cole uses GitHub issues as his task DB and the git commit log as the agent's long-term memory of past work. Avoids needing Linear/Jira MCP for solo/small-team work.
- **Why it matters**: Cheap, durable memory pattern — no extra infra. Fits a small docs+dev team without dedicated PM tools.
- **Evidence**: "I actually love using the Git log as long-term memory for my coding agents... I might as well track things in issues as well."
- **Sources**: lines 2148–2188.

### KP-10: Skills as the entry point — agents teach themselves the harness
- **Point**: An "Archon skill" lives in `.claude/skills` and teaches whatever coding agent is running how to invoke the CLI. Drop the skill into any repo and the agent can use the harness — no docs to read for the human.
- **Why it matters**: Lowers onboarding for the dev team new to AI coding. specflow already uses this pattern; reinforces that skills are the right primitive.
- **Evidence**: "as long as the Archon skill is there, the Archon CLI is a global CLI... our coding agent only knows how to use it... if we have the skill."
- **Sources**: lines 1722–1842, 2007–2014.

### KP-11: Use the agent to build the workflow ("workflow builder workflow")
- **Point**: Archon ships a workflow that builds workflows. Best practice: point the agent at an existing workflow + an external repo for inspiration (e.g., GSD), have it ask clarifying questions, then generate YAML. Live demo built a GSD-style 1,200-line workflow in one shot.
- **Why it matters**: specflow could ship a "skill that builds skills" or "command that builds commands" — accelerates docs creator authoring new flows without learning YAML.
- **Evidence**: "in fact, one of the workflows we have is a workflow builder. It's very meta... a harness around building harnesses."
- **Sources**: lines 1142–1183, 3279–3411, 3820–3939.

### KP-12: Two valid context-passing patterns: continue session vs. fresh + artifact
- **Point**: Each node can either continue the prior session's context or start fresh and read from an `artifact_dir`. Planning writes a plan file; implementation reads it in a new session. Avoids bias buildup while preserving state.
- **Why it matters**: Concrete pattern for specflow's PRD → tasks → implementation handoff. Filesystem artifacts beat trying to cram everything into a single agent session.
- **Evidence**: "the planning step is going to output a plan to our artifact directory... when we go in a brand new session in the implement stage, we are going to prompt it to read the plan."
- **Sources**: lines 2456–2524.

### KP-13: Ask-me-questions prompting pattern
- **Point**: Standard preamble: "ask me questions to make sure you understand X before you start." Reduces upfront assumptions; surfaces gaps before time is spent. Cole uses this for both workflow generation and PRD interactive flows.
- **Why it matters**: Cheap reliability boost for the docs creator drafting specs and the dev team kicking off features.
- **Evidence**: "I like asking it to ask me questions... that way you're reducing the assumptions it makes up front."
- **Sources**: lines 2425–2447, 3363–3370.

### KP-14: Interactive PRD with PM best-practice questions
- **Point**: Archon ships an interactive PRD workflow that pauses to ask classic product-manager questions (who has the problem, why can't they solve it today, what does success look like). Pause/resume is built into the harness, web UI, CLI, and adapters.
- **Why it matters**: Directly relevant to specflow's docs creator persona. We could mirror these question sets in the spec/PRD skill.
- **Evidence**: "these are like really standard questions for product managers to ask when they're first creating a PRD."
- **Sources**: lines 2196–2342.

### KP-15: Subscription-safe usage of Claude Agent SDK
- **Point**: Anthropic (per Boris Cherny) explicitly allows using your Claude Pro/Max subscription with the Claude Agent SDK for *personal use*. Banned scenarios: third-party harnesses (Open Claude/Open Code) or production-deployed agents serving other users. Archon-on-your-machine = fine.
- **Why it matters**: Risk management — specflow team can run automation against their existing subs without account bans, but shouldn't deploy shared agents on those subs.
- **Evidence**: "you are allowed to use your Anthropic subscription with the Claude agent SDK as long as it is for personal use."
- **Sources**: lines 764–818, 3183–3186.

### KP-16: Default workflows as reference scaffolds
- **Point**: Archon ships defaults (fix-GitHub-issue, validate-PR, interactive-PRD, Ralph-loop, plan-implement-validate, refactor). Even if you build custom, defaults serve as reference patterns the agent reads when generating new workflows.
- **Why it matters**: specflow should ship a small library of "exemplar" skills/commands so the agent has patterns to mimic when users ask for variants.
- **Evidence**: "two uses for them. One: use them directly out of the box... the other maybe even more important: it's a reference point for your coding agent to build something custom."
- **Sources**: lines 980–1024, 2606–2638.

### KP-17: Fresh session for each phase — bias accumulates
- **Point**: A planning session that also implements becomes biased toward its own plan; a single review session that also wrote the code can't see its own bugs. Force-rotate sessions at phase boundaries.
- **Why it matters**: Underpins specflow's separation of test/QA from implementation; reinforces multi-skill choreography over mega-prompts.
- **Evidence**: "your planning session can get pretty bogged down and you can build up a lot of bias over time."
- **Sources**: lines 322–328, 446–462.

### KP-18: Harness-built code bases evolve faster (1-shot extension)
- **Point**: Cole architected Archon's adapter/agent interfaces generically up-front. Result: adding Slack support, Codex support, etc. was one-shotted by the agent because patterns were copyable. Code that's "agent-friendly" (clear patterns, TypeScript types) gets extended faster than human-friendly code.
- **Why it matters**: Architecture decisions for an AI-coded codebase are different — design for agent legibility, not just human DX. specflow plugin code should follow consistent skeletons.
- **Evidence**: "I asked Archon to build support for Slack as well as Telegram... it just one-shotted it. Like I didn't even have to iterate."
- **Sources**: lines 1140–1183.

### KP-19: TypeScript > Python for agent-edited codebases
- **Point**: Cole chose TypeScript for Archon partly because static types help coding agents avoid mistakes. Even when not required by SDK availability, type safety acts as a deterministic guardrail at compile time.
- **Why it matters**: Language/tooling choices are a harness decision. For specflow plugin authors, prefer typed codebases.
- **Evidence**: "TypeScript is even a bit better for coding agents than Python, because it has the type safety built right in."
- **Sources**: lines 1046–1060.

### KP-20: Token economics of comprehensive workflows
- **Point**: A 10-node fix-GitHub-issue workflow (classify → research → plan → implement → validate → review → PR) running on Sonnet is *cheaper* than asking Claude Code to do the same thing once with Opus, because of right-sized model picks per node. Cole ran 4 in parallel + a GSD build inside ~50% of a 5-hour limit.
- **Why it matters**: Counter-intuitive: more nodes ≠ more tokens if you choose models well. Reframes ROI of harnesses.
- **Evidence**: "using this whole thing with Sonnet is still cheaper than asking Claude Code to fix an issue by itself with Opus cuz Opus is just so much more expensive."
- **Sources**: lines 695–702, 2942–2990.

### KP-21: Visual workflow editor (n8n-for-coding mental model)
- **Point**: Archon's web UI is moving toward visual node connection. Pitch: "n8n but for software development" — explicit because n8n itself can't handle worktree isolation, agent SDK calls, or coding-agent-specific concerns.
- **Why it matters**: Lowers barrier for non-coder docs creator to author/inspect workflows. specflow could similarly invest in visual or markdown-rendered representations of skill chains.
- **Evidence**: "think n8n, but for software development."
- **Sources**: lines 894–937.

### KP-22: Dark Factory experiment — code base where AI is the only writer
- **Point**: A dark factory is a codebase where AI handles every commit, review, and release; humans only file issues. Cole is running a public experiment using Archon workflows for triage/implementation/review/release on a YouTube-RAG app. Inspired by FANUC's lights-out robotics plants and Dan Shapiro's framing.
- **Why it matters**: Aspirational ceiling for goal 1 (productivity). Probably not appropriate for production specflow features today, but useful to track. Surprising: he is intentionally removing the human-approval gate for the experiment, contradicting the rest of the talk's hybrid-secret thesis — framed as an experiment, not a recommendation.
- **Evidence**: "all code just goes straight to the main branch once coding agents finish it... no human approval allowed... I'm actually really excited for this. For a public experiment."
- **Sources**: lines 1320–1369, 4531–4596.

### KP-23: StrongDM real-world precedent
- **Point**: StrongDM's internal AI lab is pushing thousands of lines to production with no human review. Cole flags it as not necessarily the most reliable path but a real datapoint that companies are doing it.
- **Why it matters**: Useful evidence for "what's the bleeding edge" — but read alongside the Ralph Loop warning.
- **Evidence**: "they built a system where they are pushing thousands and thousands of lines to production... humans never review."
- **Sources**: lines 4574–4595.

### KP-24: Anthropic rate limits are tightening
- **Point**: Claude Code subscription rate limits have gotten significantly worse in 2026. $200 Max plan is ~4x the $100 plan and ~20x the $20 plan. Even Cole on the Max plan expects to hit weekly limits by Tuesday/Wednesday.
- **Why it matters**: Practical operational risk for specflow team. Plan for multi-provider (Codex, MiniMax, GLM, Ollama) fallback. Cole was running MiniMax M2.7 through Claude Code routing as a workaround.
- **Evidence**: "Anthropic has made their rate limits a lot worse for Claude Code recently... I'm probably going to hit my weekly limit around Tuesday or Wednesday."
- **Sources**: lines 686–702, 2956–2990, 3097–3112.

### KP-25: Advisor mode pattern
- **Point**: Anthropic's Advisor Mode = small model does grunt-work, occasionally calls a larger model (Opus) for guidance, then continues. Same idea as per-node model selection but dynamic.
- **Why it matters**: Future direction for specflow — could let cheap models do bulk work and "phone a friend" only when stuck.
- **Evidence**: "advisor mode is basically you can use less powerful Anthropic models to do the grunt work, the implementation, and then call into a larger model like Opus for guidance."
- **Sources**: lines 4477–4490.

## Tools / repos / frameworks mentioned
- **Archon** (coleam00/Archon) — the harness builder being demoed
- **BMAD** — opinionated agentic dev harness (referenced as inspiration, "over-engineered")
- **GitHub Spec Kit** — another spec-driven harness
- **GSD** — "Get Shit Done", lighter-weight spec-driven harness; rebuilt live as Archon workflow
- **Claude Flow** — referenced in passing
- **Stripe Minions** — Stripe's internal harness, ~1,300 AI-only PRs/week
- **Shopify Roast** — Shopify's internal harness
- **Ralph Loop** — minimal autonomous loop pattern (criticized for compounding errors)
- **Claude Agent SDK** (TS + Python)
- **Codex SDK** (TypeScript only)
- **Pi agent SDK** — Cole's next integration target (general-purpose, supports many models)
- **Open Code, Amp, GitHub Copilot CLI, Gemini CLI** — discussed for SDK availability
- **n8n** — analogy for Archon's visual workflow style
- **Ollama, MiniMax M2.7, GLM, Gemma 4** — local/alternative models
- **Telegram, Slack, GitHub Issues** — Archon adapters
- **Dynamis community** — Cole's paid community with agentic-coding course, AI agent mastery course, second-brain bootcamp
- **StrongDM** — referenced as real-world dark factory precedent
- **collab mem** — chat-mention, agent memory context engine (~100 stars)
- **Claude Mem, beads** — alt memory frameworks mentioned
- **Antigravity** — Gemini 3 frontend builder
- **dark-factory-experiment** (coleam00/dark-factory-experiment) — public RAG-over-YouTube app, will be the first Archon-managed dark factory codebase

## Verification log
- **Speaker**: Confirmed Cole Medin (@ColeMedin, GitHub coleam00) via WebSearch — only Archon livestream creator matching the transcript's tone, projects (Archon, second brain, Dynamis), and product references.
- **Likely URL**: https://www.youtube.com/watch?v=srx9iwnjK2M ("Full Archon Guide — Build AI Coding Harnesses That Actually Ship (LIVE)") — top WebSearch hit on coleam00 Archon livestream.
- **Dark Factory term origin**: WebSearch confirmed Cole's framing traces to Dan Shapiro (Glowforge) → FANUC's 1980s lights-out robotics plants. Aligns with transcript's "term coined in the 1990s" remark.
- **Archon repo**: github.com/coleam00/Archon — "first open-source harness builder for AI coding". 16.2k stars at livestream time matches Cole's on-air number.
- **Companion experiment**: github.com/coleam00/dark-factory-experiment — "AI chat app for conversational RAG over YouTube video transcripts" matches Cole's stated use case at lines 4609–4663.
- **Boris Cherny SDK quote**: Not independently verified in this session; transcript-internal claim only. Treated as Cole's report, not first-hand confirmation.
