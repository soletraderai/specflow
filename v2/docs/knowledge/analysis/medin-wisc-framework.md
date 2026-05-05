# Cole Medin's WISC Framework: Context Engineering for Claude Code

## Identity
- Title: The WISC Framework — Battle-Tested Claude Code Strategies (Write / Isolate / Select / Compress)
- Speaker/Channel: Cole Medin (coleam00) — YouTube channel "Cole Medin"
- Likely URL: Linked from his X post https://x.com/cole_medin/status/1953258783976616423 and reflected in repo `coleam00/context-engineering-intro/tree/main/use-cases/ai-coding-wisc-framework`
- Suggested slug: `cole-medin-wisc-framework-claude-code`
- Confidence: High. The speaker self-identifies as "Cole," demos Archon (Medin's well-known project, 13k+ stars), and the WISC/"Whisk" framework matches his public repo.

## Thesis
About 80% of AI-coding agent failures come from poor context management, not model weakness. Even with a 1M-token window, "context rot" (distractor noise in long contexts) degrades reliability — especially on large, repetitive codebases. The WISC framework treats context as a precious resource to be engineered: **W**rite (externalize memory), **I**solate (sub-agents for research), **S**elect (just-in-time, layered context), **C**ompress (last resort). Goal: keep the window as lean as possible while still giving the agent everything it needs.

## Key points

### KP-1 Context rot is the dominant failure mode
- **Point**: ~80% of agent mistakes trace to overstuffed context, not the model. Larger windows make recall worse, not better, because of "distractors" — similar-but-wrong patterns the LLM confidently latches onto. Cites Chroma's technical report on input-token scaling.
- **Why it matters to our goals**: For a dev team new to AI coding, this reframes the #1 cause of errors. Fewer errors (goal 3) starts with context discipline, not better prompts.
- **Evidence**: 2,000+ hours of Claude Code use; Chroma needle-in-haystack research on context degradation.
- **Sources**: Transcript lines 65–127.

### KP-2 Use the git log as long-term memory
- **Point**: Standardized, detailed commit messages double as durable agent memory. A `/commit` slash command enforces a two-part format: "what we built" + "how we improved the AI layer (rules/commands)." Prime commands then read git log to brief the agent on recent work.
- **Why it matters to our goals**: Zero new tooling — leverages git the team already uses. Helps the docs creator and devs share continuity across sessions, reducing re-explanation.
- **Evidence**: Cole's `/commit` command in `.claude/commands` of the WISC repo; live demo in Archon.
- **Sources**: Transcript lines 161–249.

### KP-3 Always plan and implement in separate context windows
- **Point**: One session for planning that produces a single structured markdown spec; a second fresh session that takes only that spec as input for implementation. Planning research must not pollute the implementation window.
- **Why it matters to our goals**: Better product faster (goal 2) — implementation stays focused; spec becomes a reusable artifact the docs creator can review/edit. Aligns naturally with specflow's spec-driven flow.
- **Evidence**: Demonstrates `/plan` and `/execute` commands; explicit "no other context" rule for execute.
- **Sources**: Transcript lines 250–289.

### KP-4 Progress files / handoff.md for unavoidable long sessions
- **Point**: When a session does balloon (e.g., end-to-end browser testing pushed past 200k tokens), run a `/handoff` command that writes a structured summary, then start a fresh session from it. Avoids degraded performance from token bloat.
- **Why it matters to our goals**: Productivity (goal 1) — you don't lose state when you must reset. A simple template the team can adopt.
- **Evidence**: Live demo at ~200k/1M tokens in Archon e2e testing.
- **Sources**: Transcript lines 290–342.

### KP-5 Sub-agents for research (Isolate)
- **Point**: Spawn sub-agents in parallel for codebase + web research. They burn hundreds of thousands of tokens; the main agent receives only a ~500-token summary. Cites Anthropic's claim of ~90.2% improvement.
- **Why it matters to our goals**: Fewer errors + faster planning. Parallelism shortens research wall-clock time.
- **Evidence**: Demo: 2 sub-agents (codebase + web) for "workflow builder in Archon" finished using only 4% of main window (44k tokens).
- **Sources**: Transcript lines 343–405.

### KP-6 The "Scout" pattern — pre-decide what context to load
- **Point**: Before committing main context, send a sub-agent to scan a docs folder (e.g., `.claude/docs`, Confluence, Drive) and recommend which deep-dive docs are relevant to the current task. Then load only those.
- **Why it matters to our goals**: Directly applicable to specflow knowledge folders. Prevents accidentally loading 10 irrelevant deep-dives "just in case."
- **Evidence**: Live `explore` sub-agent demo recommending one doc out of many.
- **Sources**: Transcript lines 406–449.

### KP-7 Four-layer context selection model
- **Point**: Load context just-in-time, in layers:
  1. **Global rules** (always-on): 500–700 lines max, architecture, commands, testing/logging conventions.
  2. **On-demand context**: rules scoped to a slice (frontend rules, API rules, workflow YAML reference) loaded only when relevant.
  3. **Skills**: progressively-disclosed capability bundles (description loaded up front; full skill.md only when triggered).
  4. **Prime commands**: dynamic codebase exploration at session start, multiple specialized variants (`/prime`, `/prime-workflows`).
- **Why it matters to our goals**: Concrete blueprint specflow could mirror or audit against. Especially the 500–700 line ceiling for CLAUDE.md / global rules.
- **Evidence**: Examples from Archon's `.claude` directory.
- **Sources**: Transcript lines 450–566.

### KP-8 Skills are the right home for "sometimes" capabilities
- **Point**: Skills (the description-first, lazy-loaded format) are how he packages tools used regularly but not constantly — e.g., his agent-browser skill for e2e testing.
- **Why it matters to our goals**: Validates specflow's plugin-as-skills design. Reinforces "small description, deeper content loaded on demand."
- **Evidence**: His agent-browser skill used daily for browser automation/testing.
- **Sources**: Transcript lines 494–517.

### KP-9 Don't use sub-agents for implementation
- **Point**: Sub-agents excel at research (lossy summary is fine). For implementation you need full context of decisions and code written, so keep it in the main agent.
- **Why it matters to our goals**: Counter-balances over-eager parallelization. Useful boundary for the dev team learning multi-agent patterns.
- **Evidence**: Direct statement: "I don't recommend them for implementation because usually you want all the context of what you did."
- **Sources**: Transcript lines 400–404.

### KP-10 Compression is a last resort, not a strategy
- **Point**: "The best compression strategy is not needing compression." If you must, prefer `/handoff` (custom workflow) or `/compact` with explicit summarization instructions ("focus on the edge cases we just tested"). Re-prompt the agent to recall after compaction to verify retention.
- **Why it matters to our goals**: Discourages the common cargo-cult of routinely running `/compact`. Better hygiene = fewer errors mid-implementation.
- **Evidence**: Demo of `/compact` with focus instruction; rule: more than 2 compactions = start fresh with handoff.
- **Sources**: Transcript lines 567–623.

### KP-11 Standardized commit messages enable agent self-improvement loops
- **Point**: His commit format includes "how we improved the AI layer" — i.e., changes to rules/commands. The agent's own scaffolding evolves with the project, and that evolution is auditable via git.
- **Why it matters to our goals**: A pattern specflow's pipeline could adopt — capture rule/skill edits in commit history so the team can see how the AI layer evolves alongside the product.
- **Evidence**: Sample commit: "test improvements to the CLI" + AI-layer notes.
- **Sources**: Transcript lines 230–249.

### KP-12 The 1M-token window is a trap, not a feature
- **Point**: Just because you can fit it doesn't mean you should. Perceived performance starts degrading after a few hundred thousand tokens regardless of advertised window size.
- **Why it matters to our goals**: Sets realistic expectations for the team. Don't treat large context as license for sloppy loading.
- **Evidence**: Personal observation at ~200k tokens in Archon sessions; Chroma research.
- **Sources**: Transcript lines 87–127, 314–321.

## Tools / repos / frameworks mentioned
- **Claude Code** (Anthropic) — primary tool, GA May 22.
- **Archon** (`coleam00/archon`) — Medin's AI command center; demo codebase.
- **context-engineering-intro** repo (`coleam00/context-engineering-intro/use-cases/ai-coding-wisc-framework`) — example commands, rules, docs referenced throughout the video.
- **Chroma technical report** on input-token scaling and context rot.
- **Vercel agent-browser CLI** — used for end-to-end browser testing.
- **n8n / Dify** — referenced as inspiration for Archon's workflow builder.
- **Slash commands shown**: `/commit`, `/plan`, `/execute`, `/handoff`, `/compact`, `/prime`, `/prime-workflows`, `/context`.
- **Skills**: agent-browser skill (his own).
- **CTOX / Leor Weinstein** — co-host of his April 2 AI transformation workshop (outro plug, not technical content).

## Verification log
- WebSearch query: `Cole Medin "WISC" framework Claude Code context engineering write isolate select compress Archon` returned the canonical repo `coleam00/context-engineering-intro/tree/main/use-cases/ai-coding-wisc-framework`, his X post, GitNation talk listing, and self.md profile.
- Speaker self-identification: addressed as "Cole" in the transcript (line 59); demos Archon, which is Medin's flagship project.
- Framework name: WISC, pronounced "Whisk" — confirmed by repo path and self.md article on his context-engineering method.
- Confidence: High.
