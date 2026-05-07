# Skill Systems — Composing Modular Skills into End-to-End Workflows

**Author:** Saraev
**Source:** YouTube tutorial
**Duration:** 12:57

## Executive Summary
Saraev argues most users are misusing Claude Code skills in two opposite ways: running them in isolation as one-shot prompts, or building 1,000-line "mega skills" that try to do everything. The right pattern sits in the middle — small, focused, reusable skills wired together by an orchestrator skill into what he calls a **skill system**. He demonstrates with a 5-skill YouTube-to-shorts pipeline and shows how the same component skills (e.g. transcript extraction) plug into multiple skill systems (shorts, newsletter, SEO blog) once you have a refined library of 20–30 skills.

## The two failure modes

**Skills in isolation.** You download something like a copywriting skill, ask it for a LinkedIn post, get an output. You used Claude Code "the same way you would actually use something like ChatGPT" — single isolated output, and *you* are still the connector between research, writing, visuals, and scheduling.

**Mega skills.** Overcorrecting in the opposite direction: one giant `skill.md` for the whole content workflow. You lose modularity (the copywriting logic is locked in and can't be reused for newsletter intros), you lose maintainability (hunt through 1,000 lines for the right block), and crucially you lose **progressive disclosure** — Anthropic designed skills to load only the context that's needed, and a mega skill blows that up by loading everything at once.

> **Direct from video:** "Anthropic specifically designed skills to load only the context that's needed. And that's how they keep responses fast and maintain their high quality. So a mega skill in this case is going to blow that all up."

He cites Anthropic's own growth marketing team breaking ad-copy automations into specialised sub-agents (one for headlines, one for descriptions) — not because it was easier, but because it improves debugging and output quality on complex requirements.

## What a skill system actually is

A **skill system** is "a prompt and an instruction set wired around multiple skills." The prompt kicks it off; the instruction set (an orchestrator `skill.md`) is the brain that runs the chain. The orchestrator must understand five things:

1. **Skill architecture** — which skills are involved and in what order.
2. **Inputs** — what each step needs to do its job.
3. **Handoffs** — how output of skill N becomes clean input for skill N+1.
4. **Human-in-the-loop checkpoints** — where you step in to approve.
5. **Visual results** — how progress/results surface back to the user (markdown links, HTML dashboard, PNGs, etc.).

He notes this aligns with what Anthropic calls **sequential workflow orchestration**: explicit step ordering, clear dependencies, validation at each stage.

> **Definition:** "Skills are effectively your components and skill systems are the automations that you build with them. It's a wrapper around skills."

## Worked example — YouTube long-form to 5 short clips

A weekly skill system that takes one long-form YouTube URL and produces five faceted, captioned, illustrated portrait-mode clips ready to post. Five chained skills:

1. **Transcript extraction** — input: video URL; output: word-level timestamped transcript. Precision matters because illustrations get synced to exact frames where keywords are spoken.
2. **Clip selection** — generates five clip-worthy moments, each scored across five categories.
3. **Reframe / clip extraction** — runs face detection on every sampled frame, renders to 9:16 portrait, face-tracks across the screen.
4. **Editing** — input is the reframed clip plus the transcript; output adds pop-out illustrations timed to keywords. All illustrations created in Remotion so they're unique per video.
5. **Packaging + scheduling** — wraps the rendered clip with thumbnail, description, title, file; pushes through `zioo.com` to schedule across platforms.

The whole thing kicks from a single prompt — "take this video URL and produce five short-form clips ready to post" — and the user comes back when it pings on completion.

## Context management is everything

When chaining skills, each one gets exactly what it needs — nothing more, nothing less. Saraev spins off **sub-agents** at relevant parts to keep the context window narrow and quality high before passing data between skills.

## The skill library payoff

Skills built as proper modular components plug into multiple skill systems. The same transcript skill feeds the shorts pipeline AND a newsletter creation pipeline (transcript → copywriting skill → newsletter) AND an SEO blog pipeline.

> **Direct from video:** "By the time you've got 10 skill systems running, you might only need actually 20 or 30 unique skills powering all of them."

Updates compound: any improvement to the transcript skill ports automatically into every skill system using it.

## What's coming next

He flags upcoming systems being shipped through the **Agentic Academy**: ad generation + outlier detection, SEO/GEO content production (blog generation, page optimization), social media carousels, long-form content generation. Frames the goal as "high-quality outputs, not AI slop."

The closing teaser explicitly previews **Agentic OS**: "I'm going to walk you through the full Agentic OS setup to show you how we inject the right context into these skill systems without high effort."

## Key Exact Extracts

> **[00:00]** "Downloaded claw code skills are generic. They're bloated and they're also at the same time lacking context. But that's not even their biggest problem. They're still built for one task at a time."

> **[00:24]** "Real work isn't one process document with five different steps, it's a sequence of processes that need to connect to each other."

> **[03:33]** "You build small focused skills and then you wire them together into something bigger using one orchestrator skill."

> **[05:35]** "At its crux, a skill system is a prompt and an instruction set wired around multiple skills."

> **[06:42]** "Anthropic talks about in their own skills guide. They call it sequential workflow orchestration which is explicit step ordering, clear dependencies between steps and validation at each stage."

> **[10:07]** "When you're chaining skills like this, context management becomes everything. Each skill in the chain gets exactly what it needs to do its job. Nothing more and nothing less."

> **[11:31]** "By the time you've got 10 skill systems running, you might only need actually 20 or 30 unique skills powering all of them."

> **[12:36]** "Skills aren't single isolated endpoints, and they're not meant to be megapiles that do everything either. They're modular components designed to plug into skill systems."
