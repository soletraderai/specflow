# Skill Chaining — Three Layers of Context Efficiency

**Author:** Saraev
**Source:** YouTube tutorial
**Duration:** 19:06

## Executive Summary
A practical deep-dive on the *implementation* side of skill systems: how to chain skills without bloating your context window. Saraev rebuilds his lead-research skill (V1 → V2) using three techniques layered together — context fork, file handoff, and the `!`backtick` command substitution — and reports an **85% reduction in context burn** (51K tokens added to the main conversation in V1 vs 5–8K in V2). The piece is essentially a token-economics argument for treating skills as small components that communicate through disk, not through the chat transcript.

## The bloat problem

Saraev's lead-research V1 was a single skill running step-by-step in pros: scrape LinkedIn → research → score → write report → write DMs → push to Google Sheets → HeyReach. Fine for one lead. By lead 25, every tool response, scrape result, and intermediate reasoning has compounded into the main context. By lead 50 it's eating both context and usage limits — and most of it is raw data the next step doesn't need.

> **Direct from video:** "By the time we've hit lead 25, we don't just have stuff from the run itself, we have it from every single run that took place with the 24 leads before this thing... most of this bloat over here is nothing but bloat. It is raw data that we do not need for any of this to actually take place."

## The three-layer solution

### 1. Context fork (Anthropic-native)

A YAML front-matter setting (`context: fork`) that runs a sub-agent in its own isolated fork. Whatever runs inside doesn't bleed back to the main window — only the valuable summary returns. All tool responses are discarded on exiting the fork.

> **Step from video:** "All of the lead scraping, the research, the scoring, and the writing, and all of the tasks that I've given this lives inside this fork. And all that we get back is the valuable information that we need to our main conversation because all of the tool responses have been discarded on exiting this fork."

### 2. File handoff (community pattern)

Forking alone doesn't solve compounding inside the fork itself. Solution: every step writes its distilled output to a tiny temp file (e.g. `profile.json`, `signals.json`) containing only what the next step needs — not the full Apify/Firecrawl scrape blob. The next step reads only that file.

> **Direct from video:** "Instead of storing the whole LinkedIn profile with all of the data that we scraped back from Firecrawl or Apify... we only stash the relevant information that is needed for the next step."

### 3. `!`...`` command substitution

The killer optimisation. Instead of having Claude `Read` a file (which costs tokens and reasoning), use shell substitution syntax inside the skill: `` !`cat signals.json` ``. This runs **before Claude does anything**, dumps the file output into the skill prompt programmatically, at zero token cost.

> **Direct from video:** "What happens when we use this placeholder in our skill is that a shell command runs and captures the output from this file and dumps it into where the placeholder was. So Claude doesn't need to use any effort or any tokens in order to go and read that file."

He notes this is dynamic: profile/signals change every run, so static prompts won't do, but `!`cat`` handles dynamic injection at parse time.

## Skill vs sub-skill vs agent

A useful distinction Saraev draws while walking through V2:

- **Skill** = the task / operating procedure for a specific thing.
- **Agent** = a behaviour you want repeated across multiple skill systems (e.g. "writes in my voice").

He chose *not* to make a writing-agent for lead research because LinkedIn DMs and cold emails are different voices — better to stash voice examples as references than to over-abstract.

## The V1 vs V2 numbers

| | V1 (monolith) | V2 (forked + files + commands) |
|---|---|---|
| Tokens added to main conversation | ~51K | ~5–8K |
| Architecture | One skill, prose-chained steps in main context | Orchestrator skill spawns forked sub-skills via the skill tool; each writes a tiny file and returns one-line summary |
| Steps visible in main window | scrape, context, research, reasoning, sheets push | only one-line summaries from each sub-skill |

The V2 orchestrator decides — based on the score returned — whether to gate the lead through to Google Sheets. None of the bloat reaches the main window.

## Picking which skills to optimise

Don't apply this to every skill. He recommends targeting skills that are **both big and frequently run**. Even his weekly lead-research skill was worth optimising because of the many moving parts and compounding tokens.

Operationally: schedule cron cleanup of the temp directory, and set up observability (he points to his "Claude Command Center" video and OTEL).

## A side-bar worth keeping

A note on Claude's training-data staleness when designing skills:

> **Direct from video:** "Claude is only as smart as its training data. So, if you have asked Claude for some kind of architecture decision before, it's probably going to tell you a bunch of trash because it's not making sure that it's checking the latest information out there."

His mitigation: always add the clause "find out as of today" or "in 2026 as of today" so Claude does fresh research instead of relying on stale architectural beliefs (his concrete example: Claude doesn't know MCP has lazy loading now).

## Key Exact Extracts

> **[01:51]** "By the time we've hit lead 25, we don't just have stuff from the run itself, we have it from every single run that took place with the 24 leads before this thing."

> **[02:57]** "Three layers of skill chaining. The first one being fork, the second one is to use files, and the third one is to use commands."

> **[05:30]** "Claude doesn't need to use any effort or any tokens in order to go and read that file... because that can all just be done programmatically."

> **[07:55]** "An agent is the behavior that you want."

> **[14:22]** "There was an 85% difference in context burn, which is absolutely insane. Just by being a little bit more efficient."

> **[14:29]** "V1 over here used 51K tokens added to the main conversation after the run. For V2, we had five to 8K."

> **[13:30]** "Claude is only as smart as its training data... I always add the clause of find out as of today or in 2026 as of today."

> **[17:20]** "I would look at my skills that were probably the ones not just the biggest ones, but ones that I was running frequently because those two things combined are going to be very problematic for you."
