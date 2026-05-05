# Karpathy on Code Agents, AutoResearch, and the Loopy Era of AI

## Identity
- Title: Skill Issue: Andrej Karpathy on Code Agents, AutoResearch, and the Loopy Era of AI
- Speaker/Channel: Andrej Karpathy (guest) interviewed by Sarah Guo and Elad Gil on No Priors podcast
- Likely URL: https://www.youtube.com/watch?v=kwSVtQ7dziU
- Suggested slug: karpathy-no-priors-code-agents-autoresearch-loopy-era
- Confidence: High — content matches verified No Priors episode, multiple press write-ups confirm direct quotes ("haven't typed a line of code since December", "state of psychosis", Dobby home automation, AutoResearch, NanoChat tuning).

## Thesis
Software engineering has flipped from human-typed code to humans expressing intent to multiple parallel coding agents. The bottleneck has shifted from compute to *skill at orchestrating agents*. Productivity scales with token throughput and ability to remove yourself from the loop, but agent "jaggedness" (brilliant + 10-year-old in one) and the fact that only verifiable tasks RL well means humans still need to design the rails, evaluators, and personality of the system.

## Key points

### KP-1
- **Point**: As of December 2025, Karpathy went from 80/20 hand-coded vs. agent-delegated to roughly 0% hand-typed code; he says "I don't think I've typed a line of code since December."
- **Why it matters to our goals**: Validates specflow's bet that the dev workflow has fundamentally changed. For a small dev team new to AI coding, the *default workflow* is now agent-delegation, not autocomplete. Productivity (goal 1) and shorter time (goal 2) come from internalizing this shift, not from incremental tooling.
- **Evidence**: "I don't think I've typed like a line of code probably since December basically… their default workflow of building software is completely different as of basically December."
- **Sources**: Transcript lines 56-79.

### KP-2
- **Point**: Skill issue framing — when the agent fails, assume the bottleneck is your instructions/memory/parallelization, not model capability.
- **Why it matters to our goals**: Reframes "AI didn't work" complaints from junior devs as solvable specflow problems (better instructions, AGENTS.md, memory tools). Directly maps to fewer errors (goal 3) — errors come from skill gaps, not unfixable model limits.
- **Evidence**: "It all kind of feels like skill issue when it doesn't work to some extent… I didn't give good enough instructions in the agents from the file or whatever it may be."
- **Sources**: Transcript lines 119-132.

### KP-3
- **Point**: Mastery looks like Peter Steinberger's setup — a tiled monitor of ~10 Codex agents across multiple repos, each running ~20 minutes on high-effort prompts. Operate in macro-actions, not lines of code.
- **Why it matters to our goals**: Concrete production model for parallelism. Even a 5-person team could get 50+ "engineer-equivalents" of throughput. specflow should treat *delegation patterns* and *non-interfering task decomposition* as a first-class skill.
- **Evidence**: "Peter… uses Codex. So lots of Codex agents tiling the monitor… they all take about 20 minutes if you prompt them correctly… you can move in much larger macro actions."
- **Sources**: Transcript lines 135-168.

### KP-4
- **Point**: Token throughput is the new GPU utilization. Anxiety about leftover subscription quota is the new anxiety about idle GPUs.
- **Why it matters to our goals**: Suggests a team metric: are we maximizing concurrent agent token spend? Underutilized subscriptions = under-leveraged team. Reframes "AI is expensive" worry into a productivity north-star.
- **Evidence**: "I feel nervous when I have subscription left over. That just means I haven't maximized my token throughput… now it's not about flops, it's about tokens."
- **Sources**: Transcript lines 186-203.

### KP-5
- **Point**: Personality of the agent matters — Claude's well-calibrated sycophancy ("praise feels earned") and "soul/identity" document make it feel like a teammate; Codex feels dry. This affects user adoption.
- **Why it matters to our goals**: For a small team where the docs creator and devs are interfacing daily with an agent, choosing/configuring the *persona* of the agent is not cosmetic — it changes how much real work gets done. specflow should encode persona/voice in its skills.
- **Evidence**: "When Claude gives me praise, I do feel like I slightly deserve it… I kind of feel like I'm trying to like earn its praise which is really weird. And so I do think the personality matters a lot."
- **Sources**: Transcript lines 277-310.

### KP-6
- **Point**: "Remove yourself as the bottleneck" — the central principle. AutoResearch is Karpathy's own demonstration: hand-tune NanoChat for 20 years of intuition, then let an autonomous loop run overnight and find tunings he missed (weight decay on value embeddings, Adam betas).
- **Why it matters to our goals**: A two-decade expert was beaten by an autonomous loop on his own well-tuned repo. Strong proof that even for experienced devs, codifying objectives and stepping back beats staying in the loop. Direct path to goal 1 (more productive) and goal 2 (better product).
- **Evidence**: "I let auto research go for like overnight and it came back with like tunings that I didn't see… I forgot the weight decay on the value embeddings and my Adam betas were not sufficiently tuned."
- **Sources**: Transcript lines 562-655.

### KP-7
- **Point**: AutoResearch only works where there are objective, verifiable metrics (kernels, training loss, unit tests). Soft tasks (clarifying intent, judgment calls) still need humans because RL hasn't optimized those.
- **Why it matters to our goals**: Defines *which* parts of the dev pipeline should be auto-loop vs human-gated. Code/perf optimization → autonomous. Spec interpretation, design tradeoffs, UX → keep humans tightly involved. Helps the team avoid errors (goal 3) by knowing where not to delegate yet.
- **Evidence**: "Extremely well suited to anything that has objective metrics that are easy to evaluate… many things will not be. If you can't evaluate then you can't auto research it."
- **Sources**: Transcript lines 809-826, 877-902.

### KP-8
- **Point**: Models are jagged — brilliant senior systems programmer and 10-year-old simultaneously. They will burn compute on an obviously wrong path until you intervene.
- **Why it matters to our goals**: Argues for review gates and readiness checks (which specflow already has). The team's biggest error source isn't model dumbness, it's *trusting the agent on out-of-rails tasks*. Bake review checkpoints into every skill.
- **Evidence**: "I get so frustrated with the agents… I feel like the agent wasted a lot of compute on something it should have recognized was an obvious problem."
- **Sources**: Transcript lines 843-875.

### KP-9
- **Point**: Bespoke apps are over-produced; agents are the glue layer. APIs should be exposed; UIs should be ephemeral. The customer of software is now agents acting on behalf of humans.
- **Why it matters to our goals**: For our docs creator + dev team: build for agent consumption. Internal tools should expose APIs first, UIs second. Documentation should be markdown for agents, not HTML for humans (KP-13).
- **Evidence**: "Maybe there's like an overproduction of lots of custom bespoke apps that shouldn't exist because agents kind of like crumble them up… everything should be a lot more just like exposed API endpoints."
- **Sources**: Transcript lines 442-482.

### KP-10
- **Point**: A "claw" (persistent layer beyond a single session) needs five things done well: personality/soul document, calibrated sycophancy, persistent memory beyond context-compaction, single chat portal (e.g., WhatsApp), and macro-action thinking.
- **Why it matters to our goals**: Direct checklist for designing specflow itself as a meta-orchestrator. specflow should think of itself as a "claw" for this team, not just slash-commands.
- **Evidence**: "There's at least five things that are really good ideas in here… the soul document… he dialed the sycophancy fairly well… memory system… single WhatsApp portal."
- **Sources**: Transcript lines 240-314.

### KP-11
- **Point**: program.md / AGENTS.md style markdown *is* the meta-layer. A research org (or dev team) is "a set of markdown files that describe all the roles and how the whole thing connects." You can A/B test orgs by tuning their markdown.
- **Why it matters to our goals**: Direct validation of specflow's skill/markdown architecture. The team's process *is* its markdown. Improving the markdown improves the team. Suggests running contests/A-B on different prompt versions to find optimal team docs.
- **Evidence**: "A research organization is a set of markdown files that describe all the roles and how the whole thing connects… you can imagine tuning the code… let people write different program MDs… write a better program MD."
- **Sources**: Transcript lines 718-775.

### KP-12
- **Point**: Demand for software/engineering is going *up* (Jevons paradox), not down — cheaper code → more code wanted. Software engineering remains a good bet near term.
- **Why it matters to our goals**: Reassures the dev team — their jobs aren't disappearing, they're being amplified. Counter-narrative to "AI replacing devs" anxiety that may be slowing adoption inside the team.
- **Evidence**: "It does seem to me like the demand for software will be extremely large… something becomes cheaper, so there's a lot of unlocked demand for it."
- **Sources**: Transcript lines 1448-1505.

### KP-13
- **Point**: Education and docs are being redirected through agents. Don't write HTML docs for humans — write markdown docs for agents, who then explain to humans in their preferred style.
- **Why it matters to our goals**: Critical for the docs-creator role. Their output should target agents as the primary audience. specflow plugin docs, internal team docs, even client deliverables should be agent-readable first. This is a structural rework of the docs-creator's job description.
- **Evidence**: "Instead of HTML documents for humans, you have markdown documents for agents. Cuz if agents get it, then they can just explain all the different parts of it… I'm not explaining to people anymore. I'm explaining it to agents."
- **Sources**: Transcript lines 2186-2253.

### KP-14
- **Point**: A skill is a curriculum/instruction for the agent on how to teach a thing. Karpathy gives the example of a NanoGPT skill that scripts the progression an agent should walk a learner through.
- **Why it matters to our goals**: Direct match to specflow's existing skill model. Skills aren't just task automation — they are micro-curricula. Reframes the test/release/review skills as teaching the agent how to teach the team.
- **Evidence**: "Skill is just a way to instruct the agent how to teach the thing… I could just script the curriculum a little bit as a skill."
- **Sources**: Transcript lines 2204-2217.

### KP-15
- **Point**: Karpathy still hasn't given his agents access to email/calendar — privacy/security cautious. Even the most aggressive AI user keeps a perimeter.
- **Why it matters to our goals**: For a small team, set explicit boundaries on what agents can touch. Don't feel behind for not handing over everything. Aligns with goal 3 (fewer errors) — wide blast radius means worse errors.
- **Evidence**: "I'm still a little bit suspicious and it's still very new and rough around the edges. So I didn't want to give it like full access to my digital life yet."
- **Sources**: Transcript lines 533-544.

### KP-16
- **Point**: Open-source models are now ~6-8 months behind frontier and closing the gap. For most consumer/internal use cases, open or older closed models are already enough; reserve frontier intelligence for "Nobel-Prize-grade" hard problems.
- **Why it matters to our goals**: For a small team, default to good-enough cheaper models for the bulk of work; escalate to frontier only for hard reasoning. Cost discipline without sacrificing throughput.
- **Evidence**: "Maybe like 8 months, 6 months… for the vast majority of consumer use cases… open-source models are actually quite good… frontier closed intelligence is where Nobel Prize kind of work."
- **Sources**: Transcript lines 1714-1797.

### KP-17
- **Point**: The unit of contribution from an expert is "a few bits" — the irreducible insight. Everything downstream (explanation, formatting, variants) is now agent work.
- **Why it matters to our goals**: Tells team members where to spend energy: not on docs polish, not on boilerplate — on the hard-won 1% insight. "Things agents can't do is your job now." A clear filter for prioritization across the team.
- **Evidence**: "My contribution is kind of like these few bits, but everything else in terms of the education that goes on after that is not my domain anymore… The things that agents can't do is your job now."
- **Sources**: Transcript lines 2260-2287.

### KP-18
- **Point**: AutoResearch could go SETI@home / Folding@home style — distributed untrusted workers proposing commits, trusted verifiers checking them, a leaderboard of training improvements.
- **Why it matters to our goals**: Speculative but interesting framing — specflow's pipeline could expose verification gates and let many agents (or contributors) propose changes that are filtered through gates. Mirrors PR-review patterns.
- **Evidence**: "A swarm of agents on the internet could collaborate to improve LLMs and could potentially even like run circles around frontier labs."
- **Sources**: Transcript lines 1213-1265.

## Tools / repos / frameworks mentioned
- Claude Code (Anthropic) — Karpathy's daily driver agent
- Codex (OpenAI) — used for parallel agent fleets (Peter Steinberger setup)
- "Open Claw" / Claude (capitalized as "Claude" but transcribed as "claw") — persistent agent layer with soul document + WhatsApp portal, built by Peter Steinberger (referred to as "Peter")
- NanoChat — Karpathy's hand-tuned LLM training repo, used as AutoResearch testbed
- NanoGPT, MicroGPT, MicroGrad, MakeMore — Karpathy's "boil to essentials" repo lineage; MicroGPT is the latest 200-line essence
- AutoResearch — Karpathy's autonomous research loop project
- program.md / AGENTS.md — markdown describing the org/loop
- Sonos, HVAC, smart home APIs — reverse-engineered by his "Dobby the elf" home agent
- Qwen vision model — used for change-detection on home security camera
- WhatsApp — single chat portal to the Dobby agent
- Periodic (CEO Liam) — auto-research for materials science (sensor-driven)
- SETI@home, Folding@home — analogues for distributed AutoResearch
- Polymarket — referenced as info-market analogue
- Bureau of Labor Statistics jobs data — Karpathy's recent jobs visualization
- Daemon (book, Daniel Suarez) — referenced re: agentic web

## Verification log
- Step 1 (extract): `jq -r '.[0].data[].text' …json > /tmp/transcript-13.txt` → 2299 lines extracted.
- Step 2 (identify): Internal evidence — name "Andre" (transcribed), self-reference to "OAI"/"OpenAI" tenure, NanoChat/MicroGPT projects, Tesla Autopilot lineage ("self-driving 10 years ago"), No Priors podcast intro by host. Speaker is unambiguously Andrej Karpathy. Hosts identified as Sarah Guo (Conviction) and Elad Gil from podcast convention; Conviction is mentioned by name (line 104).
- Step 3 (verify via WebSearch): Confirmed. Episode title "Skill Issue: Andrej Karpathy on Code Agents, AutoResearch, and the Loopy Era of AI." YouTube URL: https://www.youtube.com/watch?v=kwSVtQ7dziU. Spotify and podscripts.co confirm. Fortune, Benzinga, and StocksFoundry corroborate the "haven't typed a line of code since December" and "state of psychosis" quotes. Episode aired ~March 2026.
- Step 4 (extract): 18 KPs above.
- Step 5 (write): this file.
