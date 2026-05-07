# Archon Livestream — Deep Dive on Harness Engineering

**Author:** Medin
**Source:** YouTube livestream (Archon launch livestream, with live Q&A)
**Duration:** 165:30

## Executive Summary
The companion livestream to Medin's Archon launch video. Same thesis — prompt engineering → context engineering → harness engineering, and Archon is the open-source builder for the harness layer — but with much more time on the *why*: how Archon differs from N8N, B-MAD, GSD, and the Ralph Loop; how Archon's TypeScript architecture lets new coding agents (Pi, Open Code, Amp, even GitHub Copilot if you want to write the adapter) drop in via SDK; how to think about token efficiency, model selection per node, and rate-limit pressure; and a live demo of building a custom GSD-inspired workflow from scratch via the workflow-builder workflow. The livestream ends before he can get to his "dark factory" experiment, which he previews instead.

## Why Harnesses, Why Now
The vision Medin keeps circling back to: a single coding agent has a ceiling, and the most leverage left to extract is in the orchestration layer above it.

> **Direct from video:** "It's all about like how do we create the system that wraps the coding agent."

Evidence cited:
- Studies showing PR acceptance rates moving from 6.7% to ~70% with a harness wrapping the same model.
- Stripe Minion shipping ~1,300 AI-generated PRs/week.
- The Claude Code source-code leak revealing ~40% of Anthropic's codebase is harness/agent-team code.
- All these harnesses are closed-source and not custom to you. Open-source harnesses (Ralph Loop, B-MAD, GSD, Spec Kit) are good but opinionated.

> **Direct from video:** "Up until this point, you've either had to build something internally like Stripe did, but obviously that's a massive amount of effort, or you just had to use a harness that's already out there like the Ralph Loop or B mad or whatever it is. And like yeah, those tools are very powerful, but they're not custom to you."

Archon's positioning is precise: it's a *harness builder*, not a harness. It packages your existing skills, commands, and rules into orchestrated workflows.

> **Direct from video:** "Archon is not competing with GitHub spec kit or B mad or Claude flow or GSD. It's more like those tools are great, but what if you want to build your own? That's why it's a harness builder."

## The Hybrid Secret — Deterministic Nodes + Coding-Agent Nodes
The architectural primitive Medin emphasises hardest. Archon workflows mix two kinds of nodes:
- **Coding-agent nodes** — prompts dispatched to Claude Code, Codex, etc. via the agent SDK.
- **Deterministic nodes** — bash, Python, or TypeScript scripts. No LLM, runs every time, in the right order.

This is what stops Archon from being a fancy vibe-coding loop. Tests, validation, context curation, and approval gates can all be made non-negotiable.

> **Direct from video:** "It's smart to take as many decisions from away from the coding agent as you possibly can because they're non-deterministic. They don't make the same decisions every time even if you give it the same prompt."

> **Direct from video:** "Sometimes there are steps that we want to run deterministically. We want to take the control away from the coding agent to make sure that our process is followed to a T."

Human-in-the-loop is the third axis: any node can pause for human approval. So while a workflow might feel like a Ralph Loop, the human can be inserted wherever bias would otherwise compound.

> **Direct from video:** "The Ralph Loop went viral a couple of months ago, but to me it felt like vibe coding... if the coding agent makes a mistake in the first iteration of the Ralph Loop, that issue can kind of propagate and like blow up from the rest of the loop."

## Architecture and Adapter Model
Archon is almost entirely TypeScript because that's where the SDKs live. The codebase is built around two pluggable interfaces:
- **Coding-agent adapters** — Claude (agent SDK), Codex (almost done), Pi planned, Amp/Open Code under consideration. Anything with an SDK can be added; Gemini CLI cannot yet because it doesn't have one.
- **Platform adapters** — CLI, web UI, GitHub, Slack, Telegram. Each runs in parallel; each routes through the same workflow engine.

> **Direct from video:** "I have like a sort of like generic interface implementation for every single adapter and every single coding agent."

When new adapters get added, the existing code structure is so consistent that Claude Code typically one-shots the implementation by following the existing patterns — Medin has done this for both Slack and Codex.

Subscription compatibility note: Archon uses the Claude agent SDK and the Codex SDK programmatically, which is allowed under the Anthropic terms for personal use (Boris Cherney has clarified this publicly). Open Claw / Open Code routes are different and risk subscription bans; Archon is not.

## Per-node Model Selection (and the Token Math)
A core token-efficiency lever: each node specifies which model to use. Classification with Haiku, research with Sonnet, implementation with Opus. The default model can be set at workflow scope and overridden per node.

> **Direct from video:** "I've had better results using Archon with Sonnet than I have using Opus by itself in Claude Code."

Live numbers from the stream: even with multiple parallel `fix-github-issue` workflows running, plus a validate-PR loop and a GSD workflow, Medin used ~37% of his five-hour Claude limit. The point: a comprehensive multi-step harness can still be cheaper than one Opus chat trying to do the same end-to-end work, because you use the right model at the right step.

Open feature request from the stream: dynamic model selection where the agent decides per-node based on complexity. Medin acknowledges it's a hard problem because the orchestrator agent doesn't have a clean way to reason about each model's capability, and would need a robust framework to do so.

## Context Persistence Between Nodes
Two mechanisms:
- **Continue vs. fresh** — every node can either continue the previous coding-agent session or start clean. This lets you switch models, inject different skills, or simply break bias mid-workflow without losing the conversation when you don't want to.
- **Artifact directory** — each workflow execution has an artifact dir. Earlier nodes write there (e.g. a `plan.md`); later nodes get prompted to read from it.

> **Direct from video:** "When we go in a brand new session in the implement stage, we are going to prompt it to read the plan from the artifact directory."

## Live: Building a GSD-Inspired Workflow
Medin asks Claude (loaded with the Archon skill) to research the GSD repo and Archon's existing workflows, then build a new `archon-gsd` workflow. The result is a 1,200-line YAML with four parallel research agents → research synthesis → requirements gathering → human approval gate → planning → execution → verification → code review → human acceptance.

The lessons he draws live:
- Even Archon-generated workflows usually need iteration. His first run had a node that didn't surface where requirements lived; he flags it as a prompt to improve.
- For long inline prompts, externalise them as commands (markdown files in the commands folder), not inline YAML.
- The workflow-builder workflow is itself meta — a harness around building harnesses — but it's the most reliable way to scaffold a custom workflow with the right node parameters.

## Linear, GitHub, Confluence, Anything
Medin uses GitHub issues as his primary task tracker because the GitHub CLI is so well-handled by coding agents that he doesn't even need an MCP server. For Linear, his recommended path is the Linear MCP or a Linear API skill, then point Archon at the existing `archon-fix-github-issue` workflow as inspiration to build `archon-fix-linear-issue`. Same pattern for any external system.

> **Direct from video:** "All of the default workflows that we have in Archon, there's two uses for them. One is you can just use them directly out of the box if there is one that matches how you already work, but the other maybe even more important part of these default workflows is it's a reference point for your coding agent to build something that's actually custom to you."

## The Dark Factory (Previewed, Not Demoed)
The livestream ran long, so Medin's "dark factory" demonstration moved to a separate stream. The concept:

> **Definition:** "The dark factory it's the concept of having a code base that is entirely managed by coding agents. The coding agents handle the coding, the reviewing, the the pull requests, and the releases. And so the only thing that a human gives is the issues for bugs or new features that we're requesting."

The planned public experiment: a Dynamis-community AI-coach app (RAG over Medin's YouTube + course content), with every PR / release managed end-to-end by Archon workflows. He references StrongDM's internal dark factory as prior art.

## Key Exact Extracts
> **[08:49]** "Even if you are using the exact same model, you're not improving underlying model or tool, you can go from a 6.7% pull request acceptance rate to almost 70% and the only thing is the harness."

> **[10:36]** "What if you want to take your software development life cycle, your process for working with AI coding assistance, and what if you want to package it up into your own harness? Well, that is the value proposition of Arkon."

> **[17:39]** "Sometimes there are steps that we want to run deterministically. We want to take the control away from the coding agent to make sure that our process is followed to a T."

> **[20:00]** "I actually think it's smart to take as many decisions from away from the coding agent as you possibly can because they're non-deterministic."

> **[24:54]** "I've had better results using Archon with Sonnet than I have using Opus by itself in Claude Code."

> **[88:04]** "We have a parameter for each node that specifies if we want to continue the session from the prior node or start fresh."

> **[110:59]** "B mad is a harness. Archon is a harness builder."

> **[112:54]** "If you're at an enterprise level and you already have a process for your software development life cycle, it's really really hard for you as an enterprise level, like as a team, to adopt something like B mad cuz you have to change how you work. But with Archon, you don't change how you work because you're building the layer on top of the coding agent that actually enforces that."

> **[159:02]** "The dark factory it's the concept of having a code base that is entirely managed by coding agents."
