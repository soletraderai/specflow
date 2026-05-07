# The WISC Framework — Context Engineering for Cloud Code Power Users

**Author:** Medin
**Source:** YouTube tutorial
**Duration:** 22:49

## Executive Summary
After 2,000+ hours in Claude Code, Medin argues that ~80% of agent failures trace back to poor context management — what he calls "context rot." The WISC framework (Write, Isolate, Select, Compress) is his bundled set of strategies for keeping the context window lean while still giving the agent everything it needs. The framing matters because raising the context ceiling to 1M tokens does not fix needle-in-the-haystack degradation; engineering the window does.

## Why Context Rot Is the Real Problem
The thesis up front: just because a model can fit a million tokens doesn't mean you should fill it. Medin cites the Chroma technical report on how increasing input tokens degrades LLM performance, and ties it to a specific failure mode in coding — "distractors," code patterns close to but not quite right, that the model confidently pulls from a bloated window. Larger and messier codebases hit this hardest because they have lots of similar-looking code.

> **Direct from video:** "About 80% of the time when your coding agent messes up in your codebase, it's because you aren't managing your context well enough."

The question every WISC strategy answers:

> **Direct from video:** "How do we keep our context window as lean as possible while still giving the coding agent all of the context it needs?"

## W — Write (Externalize the Agent's Memory)
Three concrete strategies for persisting decisions outside the live window:

1. **Git log as long-term memory.** Standardize commit messages with a `/commit` slash command so the log itself is a structured record the agent can read on future runs. Medin's `/commit` documents both the work shipped and any improvements made to rules/commands during the session.
2. **Brand-new context window for every implementation.** Plan in one conversation, produce a structured markdown plan, then start a fresh session and pass only the plan via an `/execute` command. Implementation never reuses planning context.
3. **Progress files and decision logs (handoff.md / todo.md).** When a session has to span a compaction, run a `/handoff` command that produces a summary the next session can resume from without inheriting hundreds of thousands of tokens of prior tool calls.

## I — Isolate (Sub-agents for Research)
Sub-agents are the lever for keeping the main window clean. Medin uses them on virtually every session, almost always during planning. He cites a 90.2% improvement (per Anthropic research) from off-loading research to sub-agents instead of letting the main agent absorb it all.

Two specific patterns:

- **Parallel research sub-agents.** Spin up two or more in parallel (e.g. one for codebase research, one for web research on best practices), then let the main agent synthesize the summaries. In the demo, sub-agents collectively used hundreds of thousands of tokens; only ~44K (4%) hit the main window.
- **Scout pattern.** Send a sub-agent ahead to inspect documentation that *might* be relevant. The scout decides what to bring into the main session, instead of the main agent loading everything just in case.

> **Direct from video:** "Send scouts ahead before you commit your main context."

Important caveat: Medin does **not** recommend sub-agents for implementation, because you usually want to keep the full context of what was just written.

## S — Select (Just-in-time Context Loading)
The S strategy answers when to load what. Medin uses a four-layer mental model:

1. **Global rules** — always loaded. ~500–700 lines for Medin (some advocate even less). Architecture, commands, testing/logging strategy.
2. **On-demand context** — loaded only when working on a specific area (e.g. front-end conventions, API endpoint conventions, the workflow YAML reference for the part of Archon that handles workflows).
3. **Skills** — progressive disclosure. Description loads up front; full SKILL.md loads only when the agent decides it needs the capability; deeper scripts/references load only when the skill needs them.
4. **Prime commands** — dynamic exploration. A prime command runs at the start of a planning session to read the current state of the codebase (including recent git log), so the agent's understanding is fresh, not stale documentation.

> **Direct from video:** "Load your context just in time, not just in case. If you're not 100% confident that a piece of information is important to your coding agent right now, then you shouldn't bother loading it."

## C — Compress (Avoid This If You Can)
Compression is the shortest section because if W/I/S are working, you rarely need it. When you must:

- **Handoff** — write a structured summary, start a fresh session. Preferred when you've already compacted once or twice.
- **`/compact` with summarization instructions** — the built-in compact, but always invoked with a focus directive (e.g. "focus on the edge cases we just tested") so the summary keeps the parts that matter. After a compact, Medin still asks the agent to recap what it remembers, to verify the summary is accurate.

> **Direct from video:** "The best compression strategy is not needing compression."

## Key Exact Extracts
> **[02:24]** "About 80% of the time when your coding agent messes up in your codebase, it's because you aren't managing your context well enough."

> **[04:43]** "How do we keep our context window as lean as possible while still giving the coding agent all of the context it needs? That is the context engineering that we are doing here."

> **[08:49]** "No matter what I'm working on, my workflow is always I have one conversation to plan with the coding agent. I'll create some kind of markdown that has my structured plan and then I will send that in as the only context to a new session going into the implementation."

> **[13:52]** "That is the power of sub agents. I don't recommend them for implementation because usually you want all the context of what you did, but for research it is very powerful."

> **[14:11]** "Send scouts ahead before you commit your main context."

> **[15:31]** "Load your context just in time, not just in case."

> **[20:22]** "The handoff and slashcompact are kind of either or. But I definitely find times where I want to use both. The handoff, especially when you run into a compaction more than twice. Usually that conversation is getting way too bloated."

> **[21:30]** "The best compression strategy is not needing compression."
