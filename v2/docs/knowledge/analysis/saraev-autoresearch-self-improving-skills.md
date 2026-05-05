# Self-Improving Claude Code Skills via Karpathy's Autoresearch Loop

## Identity
- Title: Stop Fixing Your Claude Skills. Autoresearch Does It For You (likely)
- Speaker/Channel: Nick Saraev (@nicksaraev)
- Likely URL: https://www.youtube.com/watch?v=qKU-e0x2EmE
- Suggested slug: saraev-autoresearch-self-improving-skills
- Confidence: High on speaker (Saraev) and topic. Medium on the exact video — Saraev published several closely related autoresearch videos around the same window; the diagram-generator example, the 32 -> 39/40 score arc, the Anti-gravity + Whisper Flow + Opus 4.6 stack, and the "10 diagrams every 2 minutes" framing all match his autoresearch series.

## Thesis
Skills (and prompts in general) are noisy: roughly 70% useful, 30% garbage. Instead of hand-tuning them, wrap each skill in Andrej Karpathy's `autoresearch` loop — define a binary eval suite, let an agent generate N outputs, grade them, mutate the prompt, keep the winner, and repeat. With three ingredients (an objective metric, an automated measurement tool, and something to mutate) any skill, prompt, landing page, or email campaign can be optimized overnight by agents while the human sleeps.

## Key points

### KP-1
- **Point**: The autoresearch loop only needs three ingredients — an objective metric, an automated measurement tool, and something mutable — to self-improve any artifact.
- **Why it matters to our goals**: Gives specflow a generic recipe to harden every skill it ships (test, review, plan, etc.). Once we wrap a skill this way, future model upgrades inherit the accumulated improvements automatically — direct hit on goal #1 (productivity) and #3 (fewer errors).
- **Evidence**: "You need an objective metric... some form of measurement tool... and obviously you need something to change."
- **Sources**: Transcript lines 115-167; https://github.com/karpathy/autoresearch

### KP-2
- **Point**: Skills are just prompts, and prompts are inherently noisy — single-shot evaluation is meaningless. You must run the skill many times and look at the mode/median of outputs.
- **Why it matters to our goals**: Forces specflow's QA culture toward distributional thinking. A docs-creator-plus-junior-dev team can't trust one happy run of `/test` or `/review`; they need an N-run baseline before believing a skill works.
- **Evidence**: "Sometimes you'll run a prompt and it'll do X. Another time... it'll do Y... we have to run them many, many times."
- **Sources**: Transcript lines 178-217

### KP-3
- **Point**: Use binary yes/no evals (does it contain X? is it linear? is text legible?). Likert/0-7 scales compound variability and produce unstable scores.
- **Why it matters to our goals**: Tells us how to write the eval rubrics for specflow skills (PRD checks, spec checks, test-plan checks). Binary keeps the judge stable, reduces error rate, and is cheaper to run.
- **Evidence**: "Yes or no is the simplest way to pitch it... the more variability you give the model at every step along the chain, the more variable it gets in total."
- **Sources**: Transcript lines 444-464

### KP-4
- **Point**: Beware overspecified evals — too many narrow rules cause the model to reward-hack and parrot the criteria back instead of producing real quality.
- **Why it matters to our goals**: Critical for our small team. If we over-engineer the rubrics for a `/spec` or `/test` skill, the agent will Goodhart its way to a passing score on shallow output. Better product means broader, simpler quality bars.
- **Evidence**: "It'll just find a way to parrot every single one of the evaluation points back to you... like a student who doesn't really understand the material but still gets 100% on the test."
- **Sources**: Transcript lines 463-483

### KP-5
- **Point**: Karpathy's repo is deliberately tiny — only `prepare.py`, `train.py`, and `program.md` matter. `program.md` is itself "a super lightweight skill" that drives the whole agent loop.
- **Why it matters to our goals**: Aligns with specflow's philosophy of small, composable skill markdown files. We can drop an autoresearch `program.md` into our plugin and reuse the existing skill format. No new infra needed.
- **Evidence**: "This repo is deliberately kept small and only has three files that matter... program.md is just your agent."
- **Sources**: Transcript lines 31-65

### KP-6
- **Point**: A worked example: starting score 32/40 (80%) on the diagram-generator skill, the loop hit 39/40 (97.5%) within a small number of runs, mutating the prompt each cycle and "keeping the winner."
- **Why it matters to our goals**: Concrete proof a sub-hour loop can raise an existing specflow skill from acceptable to near-perfect. Maps directly to "fewer errors" + "shorter time."
- **Evidence**: "We started with a 32 out of 40... we eventually hit 39 out of 40 on this experiment."
- **Sources**: Transcript lines 369-443

### KP-7
- **Point**: Cost math is tractable: ~2 cents per generation, 10 per cycle, ~$10 to fully optimize a skill across ~50 cycles.
- **Why it matters to our goals**: A docs creator and a small dev team can afford to optimize the entire specflow skill catalogue for low double-digit dollars. Removes the "we can't afford agentic experiments" objection.
- **Evidence**: "About 2 cents per generation... within 50 tests I can get it to a good place, I will have optimized the skill for about $10."
- **Sources**: Transcript lines 336-349

### KP-8
- **Point**: The optimization log itself becomes a durable asset — a list of mutations that future, smarter models can pick up where the previous one left off.
- **Why it matters to our goals**: Suggests specflow should version-control not just skill markdown but the autoresearch transcripts/diffs alongside it. As Opus 5/GPT-6 land, our skills compound rather than reset.
- **Evidence**: "You could take this big list and pass it on to GPT6 or Opus 5.0 and it'll be able to pick up where its predecessors left off... probably soon to be one of the most important and valuable assets of our time."
- **Sources**: Transcript lines 96-114

### KP-9
- **Point**: The pattern is domain-agnostic. Saraev applied it to website load time (1100ms -> 67ms, 81.3% improvement over 67 tests), cold email reply rates, and now skills.
- **Why it matters to our goals**: Anything specflow's team measures (CI duration, lint count, PR review latency, doc clarity scores) can become a target metric. The loop is the same.
- **Evidence**: "This auto research methodology took my load speed from about 1100 milliseconds literally down to 67."
- **Sources**: Transcript lines 73-95

### KP-10
- **Point**: The recommended setup is Anti-gravity (or any Claude Code IDE), Opus 4.6 as the loop driver, Whisper Flow for natural-language goal entry, and Sonnet vision as the evaluator.
- **Why it matters to our goals**: Concrete, low-friction stack the team can adopt today. Matches specflow's existing Claude Code dependency — no new vendors.
- **Evidence**: "I'm doing this in anti-gravity... feed everything into, in my case, Opus 4.6... 10 diagrams every 2 minutes evaluating via cloud sonnet vision."
- **Sources**: Transcript lines 280-407

### KP-11
- **Point**: A "meta skill" can autoresearch every other skill in a repo — one optimizer that walks the catalog and tunes each skill in turn.
- **Why it matters to our goals**: This is essentially a `/optimize-skills` command for specflow. One artifact that compounds quality across the whole plugin without per-skill engineering.
- **Evidence**: "I'm going to create a meta skill that goes through and then performs a sort of optimization for literally every skill in my repo."
- **Sources**: Transcript lines 408-419

### KP-12
- **Point**: Real-time dashboard visibility (live score curves per run) is part of the workflow — humans watch trends, not individual outputs.
- **Why it matters to our goals**: Useful UX hint for specflow: when we ship `/optimize`, surface a streaming score table so the docs creator and devs can sanity-check progress without reading every artifact.
- **Evidence**: "It's now opening up a real time dashboard for me to show me the results... we went from 32 up to 37."
- **Sources**: Transcript lines 366-378

### KP-13
- **Point**: A binary eval set is itself a specification. Saraev's diagram rubric (legible text, pastel palette, linear flow, no ordinals) is effectively a tiny PRD encoded as tests.
- **Why it matters to our goals**: Reinforces specflow's spec-first DNA. Eval criteria should be authored upstream by the docs creator, then handed to the loop — same artifact, two purposes.
- **Evidence**: "I have narrowed down four criteria to make a high-quality diagram... is all of the text legible... does it fit my color palette... is it linear... is it free of numbers, ordinals."
- **Sources**: Transcript lines 244-272

## Tools / repos / frameworks mentioned
- karpathy/autoresearch (GitHub) — the source pattern (`prepare.py`, `train.py`, `program.md`)
- karpathy/nanoGPT / nanochat — what autoresearch was originally built around
- Claude Code (Anthropic) — host environment for the skill
- Anti-gravity (Google's IDE) — wrapping Claude Code extension
- Opus 4.6 — loop driver model
- Claude Sonnet (vision) — eval judge
- Nano Banana Pro 2 — image generator backend for the diagram skill (~$0.02/image)
- Whisper Flow — voice-to-text for entering the goal prompt
- Excalidraw — destination canvas for generated diagrams
- Google Lighthouse — used in his earlier website-speed autoresearch run
- Instantly — cold email platform used as a separate autoresearch target

## Verification log
- 2026-04-30: Searched "Karpathy nanochat auto research repo diagram generator" — confirmed `github.com/karpathy/autoresearch`, `program.md` described as "super lightweight skill," and surfaced the 32 -> 40/40 diagram-generator anecdote attributed to Nick Saraev.
- 2026-04-30: Searched "Nick Saraev YouTube Claude Code skills autoresearch diagram generator" — Saraev's channel `@nicksaraev` confirmed; multiple matching videos exist ("Stop Fixing Your Claude Skills. Autoresearch Does It For You" — qKU-e0x2EmE; "This Claude Code Skills Hack Changed How I Work (Autoresearch)" — N_KXkczJ-FU; "Claude Code + Karpathy Autoresearch = The New Meta" — 4Cb_l2LJAW8). Tone, tooling references (Anti-gravity, Whisper Flow, Opus 4.6), and the diagram-generator + 1100ms->67ms website examples all match Saraev's published material.
- Best single URL match for this transcript: https://www.youtube.com/watch?v=qKU-e0x2EmE (medium confidence; could also be N_KXkczJ-FU).
