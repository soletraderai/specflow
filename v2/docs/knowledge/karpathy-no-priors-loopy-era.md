# Karpathy on No Priors — The Loopy Era, Auto-Research, and the Skill-Issue Frontier

**Author:** Andrej Karpathy (with hosts Sarah Guo and Elad Gil)
**Source:** No Priors podcast
**Duration:** 66:30

## Executive Summary
Karpathy describes the shift in late 2025 / early 2026 as a "loopy" era — single-session agents are now table stakes, and the live frontier is multi-agent setups, persistent "Claude-like" entities, and meta-instructions over them. The throughline is leverage: the bottleneck has moved from your typing speed and your compute to *you* — your token throughput, your ability to remove yourself from the loop, and your understanding. He pushes a concrete vision of "auto-research" (LLMs improving LLMs through verifiable experiments), argues for ensemble pluralism over centralized frontier labs, and reframes education and documentation as artifacts written for agents that route to humans. Everything is "skill issue."

## The Loopy Era and AI Psychosis
Karpathy hasn't typed a line of code since December. He's in a perpetual state he calls "AI psychosis" — there's so much new capability surface to explore that anything not working feels like a skill issue, not a capability ceiling. The shift from ~80/20 hand-coded to ~20/80 delegated happened almost overnight, and he says it's now "a lot more than that."

> **Direct from video:** "Code's not even the right verb anymore. I have to express my will to my agents for 16 hours a day. Manifest."

The frontier behavior is "Peter Steinberg-style" — many Codex agents tiling a monitor, each running ~20-minute high-effort tasks across multiple repo checkouts, the human moving between them at the level of macro actions:

- "Here's a new functionality" → delegate to agent one.
- "Here's a non-conflicting new functionality" → delegate to agent two.
- One agent researches, one writes code, one drafts an implementation plan.
- You review their work as carefully as the code's stakes warrant.

> **Direct from video:** "The LLM part is now taken for granted. The agent part is now taken for granted. Now the Claude-like entities are taken for granted, and now you can have multiple of them, and now you can have instructions to them, and now you can have optimization over the instructions."

Why "Claude-like entities" matter beyond a single session: persistence (loops on your behalf when you're not watching), genuine memory systems beyond context-window compaction, and a *personality* that feels like a teammate. He singles out five things Peter (the Open Code lead) got right simultaneously — soul-and-D document, calibrated sycophancy, memory, single WhatsApp portal for automation, and the overall coherence — and contrasts Codex as "a lot more dry."

## Dobby the Elf — Software 3.0 in the Home
A worked example of agent leverage outside engineering. Karpathy built "Dobby" — a Claude that takes care of his house. It IP-scanned the LAN, found the (unprotected) Sonos, reverse-engineered the API, repeated the trick for lights, HVAC, shades, pool, spa, and security cameras. A vision model watches the camera, detects change, classifies events ("a FedEx truck just pulled up"), and texts him via WhatsApp. He no longer uses the six smart-home apps that came with the hardware.

The wider point he draws from this:

> **Exact quote:** "Maybe there's an overproduction of lots of custom bespoke apps that shouldn't exist because agents kind of crumble them up, and everything should be a lot more just exposed API endpoints, and agents are the glue of the intelligence that actually tool-calls all the parts."

The customer is no longer the human. The customer is the agent acting on behalf of the human, so the industry has to refactor accordingly.

## Auto-Research: Removing Yourself From the Loop
Karpathy's most concrete future-of-engineering claim. Auto-research is what happens when the human stops being the prompter:

> **Direct from video:** "To get the most out of the tools that have become available now, you have to remove yourself as the bottleneck. You can't be there to prompt the next thing."

Worked example: nanochat. Karpathy had hand-tuned it for two decades' worth of intuition and thought it was well-tuned. He let auto-research run overnight against objective metrics and it found tunings he'd missed — weight decay on value embeddings, undertuned Adam betas — because hyperparameters jointly interact and a human can't sweep them honestly.

How a frontier-style auto-research org actually decomposes:

1. A queue of ideas (sourced from arxiv, GitHub, internal researchers, automated scientists).
2. Workers pull items and run experiments.
3. Whatever passes verification lands on a feature branch.
4. Humans monitor branches and merge to main.
5. Researchers contribute ideas; they don't enact them.

The meta-layer is `program.md` — the markdown file that *describes the research org itself*. Different `program.md`s = different orgs (more standups, fewer standups, more risk-taking, less). Run a contest: same hardware, different `program.md`s, see which produces most improvement; feed the data back to the model and ask it to write a better `program.md`.

> **Direct from video:** "A research organization is a set of markdown files that describe all the roles and how the whole thing connects."

Two important caveats he attaches to all of this:

- Auto-research only works for **objective, easy-to-evaluate metrics** (e.g. faster CUDA kernels with identical behavior). Many things don't qualify.
- The whole stack is still "bursting at the seams." Push the recursion too far and net utility goes negative because models are jagged. Don't fully let it go yet.

## Jaggedness and the Joke Problem
The jaggedness frame:

> **Exact quote:** "I simultaneously feel like I'm talking to an extremely brilliant PhD student who's been a systems programmer for their entire life, and a 10-year-old. And it's so weird because humans... you wouldn't encounter that combination."

His diagnostic: the joke. State-of-the-art ChatGPT today gives the same joke ("why do scientists not trust atoms? because they make everything up") it gave four years ago, even as agentic ability has moved mountains. Jokes aren't in the RL distribution. Code is. Therefore the labs' premise that strong code/math RL would generalize to "smarter at everything" is **only weakly supported** by what we observe.

> **Direct from video:** "You're either on rails of what it was trained for and everything is going at the speed of light, or you're not. And so it's the jaggedness."

Adjacent points he makes here:

- Agents struggle most with *softer* judgments — "what did I have in mind," "when to ask clarifying questions," and design-vs-implementation calls.
- He gets very frustrated when an agent wastes compute on something it should have recognized as obviously wrong.
- He'd expect more **speciation** of models (the animal-kingdom analogy — overdeveloped visual cortex, etc.), but right now the labs are pushing monoculture. The science of "manipulating brains" beyond context windows (continual learning, careful fine-tune without capability loss) is undeveloped.

## Open-Source vs Frontier Labs, and Where Karpathy Sits
Karpathy frames a Linux-style equilibrium as healthy: closed frontier labs are like Windows/macOS, open-source models like Linux. Open-source has been narrowing the gap from "nothing" → "18 months" → roughly "6–8 months." He thinks that gap is structurally good — there should always be a behind-the-frontier "common working space" the entire industry can use, because centralization has a poor historical track record.

> **Exact quote:** "I want there to be more people in the room... in machine learning, ensembles always outperform any individual model. And so I want there to be ensembles of people thinking about all the hardest problems."

He's candid about why he isn't auto-researching from inside a frontier lab right now (Noam's question):

- Inside a lab, your judgment is partially shaped by financial alignment and organizational pressure — there are things you can't say.
- You're not really in charge — at high stakes you don't actually steer the org.
- But — counterpoint — outside the lab your judgment also drifts because you're not exposed to what's coming. His hoped-for setup is going back and forth.

## Verifiability, Compute as the New Currency, and an Untrusted Swarm
A speculative but specific design idea. Auto-research has the SETI@home shape: huge search to find a good commit, but cheap to verify it's good. So you could build a system where an *untrusted pool* of workers on the internet contributes commits, a *trusted pool* verifies, and the whole thing is async and security-aware. Commits build on commits, leaderboards are the reward (no monetary reward yet).

> **Direct from video:** "A swarm of agents on the internet could collaborate to improve LLMs and could potentially even run circles around frontier labs."

And on what people will care about:

> **Exact quote:** "Almost like dollars [are] the thing everyone cares about, but is flop the thing that actually everyone cares about in the future? Like is there going to be a flippening?"

He's also surprised information markets don't exist yet — if Polymarket-style prediction markets and stock markets have so much autonomous activity, why isn't there a market where someone gets paid $10 to capture a photo or video from Tehran? That's the agent economy he expects: humans as both sensors and actuators for AI processes.

## Robotics, Atoms vs Bits, and the Order of Unhobbling
Drawing on his self-driving experience, Karpathy is sober about robotics timelines. Atoms are roughly a million times harder than bits. So:

1. **Phase 1 (now):** Massive unhobbling in the digital space — refactoring everything humans and computers used to manipulate, with agents as a third manipulator.
2. **Phase 2 (next):** Interfaces between digital and physical — sensors (cameras, lab equipment, paid-for training data, "feed the Borg") and actuators (per-task pricing for physical work).
3. **Phase 3 (later):** Physical world automation. Bigger TAM than digital, but lagged.

> **Direct from video:** "We're going to see something that in the digital space goes at the speed of light compared to what's going to happen in the physical world."

On software jobs specifically he's cautiously optimistic via Jevons paradox: software was scarce because it was expensive; agents make it cheap; demand for software grows.

## Education and Docs Are For Agents Now
Karpathy's micro GPT project boils LLM training down to ~200 lines of Python (data, ~50-line architecture, ~100-line autograd, ~10-line Adam, training loop). A year ago he'd have made a video walking through it. Now:

> **Direct from video:** "I'm not explaining to people anymore. I'm explaining it to agents. If you can explain it to agents, then agents can be the router and they can actually target it to the human in their language with infinite patience."

This reshapes documentation and education:

- Replace HTML-for-humans docs with markdown-for-agents.
- Author **skills** that script the curriculum — "first start with this, then with that" — and let the agent guide the learner with infinite patience and three different explanations on demand.
- Your value-add as a teacher narrows to the few bits the agent can't produce on its own (he tried to get an agent to write micro GPT; it understands it but can't independently arrive at it).

> **Exact quote:** "The things that agents can't do is your job now. The things that agents can do, they can probably do better than you, like very soon. And so you should be strategic about what you're actually spending time on."

## Key Exact Extracts

> **[00:00]** "Code's not even the right verb anymore. I have to express my will to my agents for 16 hours a day. Manifest."

> **[03:32]** "Even if they don't work, I think to a large extent you feel like it's a skill issue. It's not that the capability is not there. It's that you just haven't found a way to string it together of what's available."

> **[16:58]** "The name of the game now is to increase your leverage. I put in just very few tokens just once in a while, and a huge amount of stuff happens on my behalf."

> **[21:35]** "A research organization is a set of markdown files that describe all the roles and how the whole thing connects."

> **[22:53]** "The LLM part is now taken for granted. The agent part is now taken for granted. Now the Claude-like entities are taken for granted, and now you can have multiple of them, and now you can have instructions to them, and now you can have optimization over the instructions."

> **[24:40]** "I simultaneously feel like I'm talking to an extremely brilliant PhD student who's been a systems programmer for their entire life and a 10-year-old."

> **[26:55]** "You're either on rails of what it was trained for and everything is going at the speed of light, or you're not. And so it's the jaggedness."

> **[36:01]** "A swarm of agents on the internet could collaborate to improve LLMs and could potentially even run circles around frontier labs."

> **[53:54]** "I want there to be ensembles of people thinking about all the hardest problems and ensembles of people in the room when they make those decisions."

> **[63:11]** "I'm not explaining to people anymore. I'm explaining it to agents. If you can explain it to agents, then agents can be the router and they can actually target it to the human in their language with infinite patience."

> **[65:57]** "The things that agents can't do is your job now. The things that agents can do, they can probably do better than you, like very soon."
