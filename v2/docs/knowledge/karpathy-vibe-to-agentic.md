# From Vibe Coding to Agentic Engineering — Karpathy on the Software 3.0 Shift

**Author:** Andrej Karpathy
**Source:** YouTube talk (AI native conference fireside / interview, 2026)
**Duration:** 29:45

## Executive Summary
Karpathy argues that something fundamentally shifted in late 2025 — agentic coding workflows crossed a threshold where chunks of generated code stopped needing correction, and he stopped typing code himself. This pushes us out of the "vibe coding" phase (raise the floor for everyone) into "agentic engineering" (preserve the professional quality bar while going much faster). The deeper claim is that we're entering Software 3.0, where the unit of programming is the prompt or context window, agents are the substrate, and a huge amount of existing app surface area shouldn't exist anymore — it's just neural networks with tools.

## December Was the Inflection Point
Karpathy frames late 2025 as a sharp transition, not a smooth ramp. He'd used agentic tools for months — useful but error-prone — and then in December the chunks just came out fine, kept coming out fine, and he stopped correcting them. He explicitly stresses that anyone whose mental model of AI was set during the ChatGPT era from 2024-2025 needs to look again, because the "agentic coherent workflow" only really started working in this window.

> **Direct from video:** "I can't remember the last time I corrected it and then I just trusted the system more and more and then I was vibe coding."

He cautions normal users still don't realize how dramatic this was — a typical software engineer's default workflow at their desk is fundamentally different as of December.

## Software 3.0: Programming Becomes Prompting
Karpathy's three-paradigm framing:

- **Software 1.0** — explicit code, hand-written rules.
- **Software 2.0** — programming-by-data: you arrange datasets, objectives, neural net architectures, and gradient descent fills in the weights.
- **Software 3.0** — the trained LLM is itself a programmable computer. Your "program" is what's in the context window; the LLM is the interpreter.

The two examples he uses to drive this home are worth keeping:

1. **Open Code installation.** Traditionally a shell script ballooning to handle every platform. The modern install is a block of text you paste into your agent — the agent's intelligence handles the per-machine variation, debugs in the loop, and "performs intelligent actions to make things work." The piece of text you copy-paste to the agent *is* the program.
2. **MenuGen.** He vibe-coded a Vercel app that takes a menu photo, OCRs items, calls an image generator, and returns a rendered menu with pictures. Then he saw the Software 3.0 version: pass the photo to Gemini and tell it to use Nano Banana to overlay the items directly onto the menu pixels. The whole app is spurious in the new paradigm.

> **Direct from video:** "All of my menu gen is spurious. It's working in the old paradigm. That app shouldn't exist."

The lesson: don't only think about the new tools as speed-ups for what already exists. Whole categories of code shouldn't exist; new categories (e.g. LLM-built personal wikis) couldn't exist before.

## Verifiability Is the Capability Map
Why is the model brilliant at refactoring 100k-line codebases but tells you to *walk* to a car wash 50 meters away to wash your car? Karpathy's working theory: capabilities are jagged because they track two things — (1) whether a domain is verifiable enough to put into RL, and (2) whether the labs decided to put it in the data distribution.

> **Exact quote:** "Traditional computers can easily automate what you can specify in code... this latest round of LLMs can easily automate what you can verify."

His instructive anecdote: GPT-3.5 → GPT-4 chess improvement wasn't general capability lift, it was someone at OpenAI deciding to shove a huge chess corpus into pre-training. The implication for builders:

- Verifiable settings stay tractable even if the labs aren't focused there — you can build the RL environment yourself and fine-tune.
- "Almost everything can be made verifiable to some extent" — including writing, via a council of LLM judges.
- You're at the mercy of "what circuits you're in." If your application sits in a circuit the labs RL'd, you fly. If not, you struggle, and you need fine-tuning.

## Vibe Coding vs Agentic Engineering
The cleanest distinction Karpathy has offered to date:

- **Vibe coding** raises the floor — anyone can build software now.
- **Agentic engineering** preserves the professional ceiling — the quality bar of pre-AI software has to hold even as you go 10x+ faster.

> **Direct from video:** "You're not allowed to introduce vulnerabilities due to vibe coding... you're still responsible for your software just as before, but can you go faster?"

He thinks the speedup for people who are good at this is well above 10x. Practical implications he calls out:

- Hiring should change. Old-school puzzle interviews miss agentic-engineering capability. He proposes a working interview: build a real project (e.g. a Twitter clone for agents, fully secured), then spin up 10 Codex 5.4 high-effort agents to attack it. They shouldn't be able to break it.
- Treat agents as **intern-grade entities**: extreme recall, weak judgment. You stay in charge of taste, aesthetics, design, spec, and whether the right things are being asked for. They handle "fill in the blanks."

A representative bug he hit on MenuGen: the agent tied Stripe purchases to Google logins by matching email addresses, instead of designing a persistent user ID. That's exactly the class of design judgment a human still owns.

## Specs and Docs Are the New Programming Surface
Karpathy is mildly down on plan-mode-as-a-feature and pushes a more general principle: work *with* the agent to produce a detailed spec (the docs), then have the agents implement it.

> **Exact quote:** "You have to work with your agent to design a spec that is very detailed... and then get the agents to write them, and you're in charge of the oversight and the top-level categories."

He notes the generated code itself is often "bloaty, copy-paste, awkward abstractions, brittle — works but really gross." He hopes future RL adds aesthetics/quality reward, but for now you stay in charge of taste. His micro GPT project illustrates the floor: he keeps prompting models to simplify LLM training code further and they can't — that's outside the RL circuit.

## Agent-Native Infrastructure (and the Animals vs Ghosts Frame)
Two complementary points about how the world has to be rebuilt:

- **Sensors and actuators.** Decompose every workload into things the agent reads and things the agent acts on. Make data structures legible to LLMs. Docs should be markdown for agents, not HTML for humans. His pet peeve: docs that say "go to this URL" — "I don't want to do anything. What is the thing I should copy-paste to my agent?"
- **Animals vs ghosts.** LLMs are not animal intelligences shaped by evolution and intrinsic motivation; they're "statistical simulation circuits" with RL bolted on top. Yelling at them does nothing. The framing matters because it sets your expectations and your debugging mindset — be suspicious, don't expect aesthetics for free, and remember they're jagged because their training was jagged.

He also forecasts an agent-rep economy: your agent talking to my agent to schedule a meeting; the test for whether infra is becoming agent-native is whether you can prompt "build MenuGen" and the deployment, DNS, and service config also happen via agent without you logging into dashboards.

## What Still Matters for Humans (and Education)
Karpathy ends on the line he keeps returning to:

> **Direct from video:** "You can outsource your thinking, but you can't outsource your understanding."

The argument: even with high token throughput, *something* still has to direct the work and decide what's worth building. Understanding is the bottleneck because you can't be a good director without it, and LLMs don't excel at understanding — you do. Tools that increase your understanding (his LLM-knowledge-base wiki habit, where he asks his own corpus questions) are therefore especially valuable.

## Key Exact Extracts

> **[01:36]** "I just started to notice that with the latest models, the chunks just came out fine, and then I kept asking for more, and it just came out fine, and then I can't remember the last time I corrected it, and then I just trusted the system more and more, and then I was vibe coding."

> **[03:21]** "Software 3.0 is kind of about your programming now turns to prompting, and what's in the context window is your lever over the interpreter that is the LLM."

> **[06:02]** "All of my menu gen is spurious. It's working in the old paradigm. That app shouldn't exist."

> **[10:02]** "Traditional computers can easily automate what you can specify in code, and this latest round of LLMs can easily automate what you can verify."

> **[11:54]** "How is it possible that state-of-the-art Opus 4.7 will simultaneously refactor a 100,000-line codebase or find zero-day vulnerabilities and yet tells me to walk to this car wash? This is insane."

> **[13:08]** "You have to actually explore this thing that they give you that has no manual. And it works in certain settings, but maybe not in some settings... if you're in the circuits that were part of the RL, you fly."

> **[15:57]** "Vibe coding is about raising the floor for everyone in terms of what they can do in software... agentic engineering is about preserving the quality bar of what existed before in professional software."

> **[20:49]** "You have to work with your agent to design a spec that is very detailed, and maybe basically the docs, and then get the agents to write them, and you're in charge of the oversight and the top-level categories."

> **[25:53]** "Everything is still fundamentally written for humans... I don't want to do anything. What is the thing I should copy-paste to my agent?"

> **[28:10]** "You can outsource your thinking, but you can't outsource your understanding."
