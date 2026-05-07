# Autoresearch for Business Outcomes — Self-Improving Cold Email (and Beyond)

**Author:** Saraev
**Source:** YouTube tutorial
**Duration:** 24:43

## Executive Summary
Saraev's earlier autoresearch video, applied to a business problem rather than a skill. He wraps Karpathy's Autoresearch convention around an **Instantly cold-email pipeline**, with reply-rate as the objective metric and email copy as the thing to mutate. The agent runs every 4 hours via GitHub Actions cron — harvesting prior results, generating a new challenger, deploying campaigns, repeating. He generalises the pattern to landing pages (CRO), ad creatives, chatbot scripts, product descriptions, YouTube titles, newsletter subject lines — anywhere you have an objective metric and an API to change inputs.

## What auto research actually requires

Saraev distills the pattern down to three ingredients:

> **Definition:** "Anything that has an objective metric you can track and an API or application programming interface that you can send a request to to get."

Plus a thing to change (copy, code, hyperparameters). That's it.

He stresses **fast feedback loops**:

> **Direct from video:** "In 60 minutes you could run 12 experiments. And so obviously 12 experiments is a lot of data... your iteration loop will be much faster because you'll be able to kind of draw it like this as opposed to like this."

## The cold-email setup

Folder layout (mirrors Karpathy's repo):

- `orchestrator.py` — top-level prompt; tells the agent it's running auto-research, every 4 hours, mutating cold-email copy to optimise reply rate.
- `instantly_client.py` — API wrapper for the email platform (Instantly).
- `baseline.md` / `challenger.md` — the two competing versions of email copy.
- `resources.md` — the rolling knowledge base where the agent logs learnings across runs.
- Utility scripts (purge old leads, deploy in batch, parse responses).
- GitHub Actions workflow (cron-triggered hourly).

The loop has three phases each cycle:

1. **Harvest** — pull metrics for the previous experiment from Instantly's API.
2. **Generate** — write a new challenger informed by `resources.md` of accumulated learnings.
3. **Deploy** — create campaigns, draw leads from the existing pool, activate.

> **Step from video:** "Anything with C is what we call a challenger. Anything with B is what's called baseline. The model starts with a baseline type of copy... then it makes slight modifications based off of what it knows to perform really well in cold email copy before testing it out. It runs the two side by side and then automatically harvests."

## The compounding knowledge file (`resources.md`)

This is the part Saraev flags as more important than the immediate optimisation. Every run, the agent appends a hypothesis + result to `resources.md`:

> **Direct from video:** "As the models get better and better and better, they log all of their learnings to a resource.md MD that significantly improves future models abilities to make changes."

Over time you get a structured corpus of "what works for this audience" that future agents (and future smarter models) inherit.

## Concrete generalisations

Saraev rapidly enumerates use cases:

- **Cold email** — metric: reply rate. Change: email copy. API: Instantly.
- **Landing pages (CRO)** — metric: conversion rate. Change: page elements. API: Wix/WordPress/Webflow.
- **Ad creatives** — metric: CVR. Change: creative. API: Facebook/Google.
- **Chatbot scripts** — metric: customer satisfaction score. Change: master template.
- **Ecom product descriptions** — metric: revenue / sales. Change: description copy. API: Chrome DevTools MCP if no first-party API.
- **YouTube titles** — metric: CTR/views. Change: title. API: YouTube Data Analytics v3.
- **Newsletter subject lines, pricing pages, SEO pages** — same pattern.

## Setup walkthrough — the prompt template

> **Step from video:** "Hey, I want you to use the context in the auto research folder to help me build a very similar idea, except instead of testing for validation loss and iterating on a machine learning model, I want you to do all of this, but for cold email. The metric I'm interested in optimizing for is my reply rate. The platform I'm going to be doing all this stuff on is instantly... the thing that you're going to change between one experiment and the other is going to be the copy of the cold emails. Finally, I want you to take all this and then put this on the cloud using GitHub actions. So, it runs once every hour and it has everything it needs to work on autopilot."

That single voice-dictated prompt is what bootstraps the whole pipeline. Saraev also wires a Slack webhook so the system pings him each cycle with the new challenger vs baseline diff.

## Where auto-research fails

Use cases Saraev considers **bad fits**:

- **Slow feedback loops.** If you can only learn whether a change worked after a week, the loop crawls and few experiments stack up before drift dominates.
- **Fuzzy / subjective metrics.** "Warmth" or "happiness" can't be measured directly — you need a proxy (a scale, an analytic, a binary).
- **No API to change inputs.** If the agent can't write the change itself, you're back to manual deployment and the autonomy story collapses (though Chrome DevTools MCP can sometimes substitute).

> **Direct from video:** "If you don't have the API access, you could build some sort of Chrome dev tools or CLI based flow, but like you need to have that because if you don't, how the heck is the agent supposed to make any changes?"

## A subtle baseline observation

Saraev confesses most challengers initially lose to his hand-written baseline ("usually the baseline is better because I wrote it"). Eventually challengers do win — and at that point the new challenger becomes the new baseline, and the cycle repeats. Important framing: this isn't the AI replacing your craft on day one; it's persistent overnight A/B testing that eventually exceeds your craft because it never sleeps.

## Key Exact Extracts

> **[00:00]** "An open source project just dropped that when you combine it with claude code literally becomes self-improving AI."

> **[00:31]** "He says it right here. The idea is to give an AI agent a small but real LLM training setup and just let it experiment autonomously overnight. It'll modify the code, train for 5 minutes, check if the results improved, keep or discard, and then just repeat. You wake up in the morning to a log of experiments and hopefully a better model."

> **[03:56]** "As the models get better and better and better, they log all of their learnings to a resource.md MD that significantly improves future models abilities to make changes."

> **[07:13]** "The requirement that you need is anything that has an objective metric you can track and an API or application programming interface that you can send a request to."

> **[06:08]** "AI agents don't [eat, sleep]. You could very quickly and easily set this up on again an hourly loop and have this run 24 times a day whereas realistically if you were to try and do it all yourself you could only do it a couple times."

> **[22:10]** "Things that work really well are things that have fast feedback loops."

> **[22:34]** "How do you subjectively measure like warmth you know you can't... what you have to do is you have to find proxies for all these things which are usually like scales and metrics and analytics."
