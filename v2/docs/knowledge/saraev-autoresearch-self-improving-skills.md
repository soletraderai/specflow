# Autoresearch for Self-Improving Skills — Letting Skills Evolve Overnight

**Author:** Saraev
**Source:** YouTube tutorial
**Duration:** 16:33

## Executive Summary
Saraev maps Andrej Karpathy's recently-released **Autoresearch** repo onto Claude Code skills. The thesis: skills are prompts, prompts are noisy, so the only honest way to know if a skill is good is to (a) run it many times and (b) score the outputs against a fixed eval suite. He then closes the loop — the agent rewrites the skill prompt, re-runs, keeps the winner. He demos this on his `diagram-generator` skill (running every 2 minutes, generating 10 diagrams each round, scored across 4 binary criteria → max 40), watching it climb from 32/40 baseline to 39/40 in a few iterations for ~$10 of compute.

## Where the idea comes from

Karpathy's auto-research repo is "deliberately kept small and only has three files that matter": `prepare.py` (machine-learning specific, ignore), `train.py` (the thing being optimised — pretend this is your `skill.md`), and `program.md` (the agent prompt).

> **Direct from video:** "I want you to pretend for a second that this train.py is actually your skill.md. And then your programm is just your agent."

The agent is told: improve this train/skill, here's how to measure if the new version is better than the last, repeat until great.

## The three ingredients

> **Definition:** "In order for auto research to work, you need three ingredients. You need an objective metric. Okay? Now, that's a number that you can measure... Next, you need some form of measurement tool. This ideally would be automated, reliable. There'd be no human in the loop... Finally, you obviously need something to change."

Mapped to skills:

1. **Objective metric** — your **eval pass rate** (e.g. 32/40).
2. **Measurement tool** — an agent runs the eval test suite automatically.
3. **Thing to change** — the skill's `skill.md` prompt itself.

## Evals — why and how

Skills are prompts; prompts are inherently noisy. One run gives you X, next run Y. To compare prompt versions you must run *many* times and look at mode/median, not single runs. Then evaluate outputs against a standard ("benchmark the performance of our skills").

**Best eval format: binary yes/no questions.** He's emphatic about this:

> **Direct from video:** "Go binary wherever possible... Imagine like a little cone, right? It starts over here really really narrow but then the more variability the more we compound out until eventually my my my answer you know my out of 40 could be at 39 out of 40 or it could be a 2 out of 40."

Likert scales compound probabilities and explode variance. Binary keeps the cone narrow.

**Worked eval set** for the diagram generator (4 binary criteria):

1. Is all the text in the diagram legible and grammatically correct?
2. Does it fit my pastel/soft-colour palette?
3. Is it linear (left-to-right or top-to-bottom)?
4. Is it free of numbers/ordinals/ordering?

10 generations × 4 criteria = score out of 40 per run.

## A trap to avoid: over-specific evals

> **Direct from video:** "Don't go so concrete and so like narrow that the model starts optimizing for silly things like I've seen a lot of people say stuff like hey make sure this is under x words... if you give the model way too many of these evals what it'll eventually do is it'll just like find a way to parrot every single one of the evaluation points back to you. So even if the actual quality of the thing is not very good it'll technically say passes the test."

The student-who-doesn't-understand-but-aces-the-test failure mode. Keep evals broad, simple, and focused on real quality signals.

## The procedure

1. Set up a Claude Code environment.
2. Clone Karpathy's auto-research repo into the workspace so the agent has the convention as context.
3. Write evals — a list of binary criteria for "good output".
4. Prompt the agent: "Use the auto-research convention from the above repo to build a self-improving skill system for my `<skill-name>` skill. Eval suite is the binary criteria above. Every 2 minutes, generate 10 outputs, score against eval suite, mutate the prompt, keep the winner."
5. Walk away.

He notes the agent set up a real-time dashboard showing run-by-run scores; opening run starts at 32/40 and climbs.

## Cost economics

Diagram generation costs ~$0.02 per image via Nano Banana Pro 2 → ~$0.20 per 10-image test → ~$10 to optimise across 50 tests. A frame Saraev uses repeatedly: this is a positive ROI move for any skill that drives revenue.

## The compounding-knowledge angle

Beyond the immediate improvement, the auto-research run produces a **log of every change attempted** — what worked, what didn't. That artifact is what he believes is "soon to be one of the most important and valuable assets of our time": when GPT-6 / Opus 5.0 ships, you hand the next agent the previous research log and it picks up where the predecessor left off.

> **Direct from video:** "You could take this big list and pass it on to GPT6 or Opus 5.0 and it'll be able to pick up where its predecessors left off."

He explicitly suggests building a "**meta skill** that goes through and then performs a sort of optimization for literally every skill in my repo."

## Key Exact Extracts

> **[00:18]** "I wanted to show you how to combine Claude Code skills with a new development in the AI space called Auto Research to achieve significantly higher reliability, accuracy, and allow your skills to quite literally improve themselves overnight."

> **[03:44]** "In order for auto research to work, you need three ingredients. You need an objective metric... some form of measurement tool... something to change."

> **[05:34]** "All machine learning and all AI outputs are distributions of data. And so, in order for us to control against that and allow us to make iterations and improvements on them, we just need to run them many, many times."

> **[06:54]** "The best way to do that is using binary yes or no true or false questions."

> **[14:33]** "The one thing that matters is just how much time you let it run on. So if it runs for, you know, a couple of days, couple of weeks, couple of months, you can imagine you could start basically wherever the hell you want and eventually it's going to be fantastic."

> **[14:55]** "The core thing is just defining the right set of evals. And my recommendation is always just make them simple yes or no answers."

> **[15:25]** "If you give the model way too many of these evals what it'll eventually do is it'll just like find a way to parrot every single one of the evaluation points back to you... That's sort of like a student who you know doesn't really understand the material but then still gets 100% on the test."

> **[13:30]** "I'm going to create a meta skill that goes through and then performs a sort of optimization for literally every skill in my repo just to get it as close as I can to perfect."
