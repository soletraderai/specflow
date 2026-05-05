# Karpathy perspective — position paper

## TL;DR

specflow's job is not to ship more skills or pipeline stages. It's to lift the ceiling on two things the team already half-owns: the **spec** (the human's irreducible bits of intent) and the **autoresearch loop** (the one mechanism that takes a human off the critical path without compounding errors). Fan-out infra, multi-reviewer fan-in, agent org charts, persona docs — downstream or distraction.

The non-obvious claim: **verifiability decides automation speed, and most of specflow's surface today is not verifiable**. Until pipeline outputs are scoreable on a single number (or a stable binary judge), no amount of harness sophistication compounds. With scorers in place, even crude loops out-tune hand-written prompts overnight — exactly how autoresearch beat 20 years of nanochat intuition.

A docs creator and an AI-novice dev team isn't a 5-person team — it's 50-engineer-equivalent the moment "agent throughput" stops being cost and becomes utilization. The bet for this quarter: pick one specflow skill with a measurable outcome, wrap it in autoresearch, let it run. That makes everything else either obvious or unnecessary.

## What Karpathy gets right

A few things that the other camps in this corpus consistently underweight.

**1. Skill issue is a generative reframe, not a put-down.** When the agent fails, the question is never "is the model dumb?" — it's "which of (instructions, memory, parallelization, verification surface) was thin?" That's not a debugging slogan, it's a roadmap. Pocock's de-slop work, Medin's WISC, and Saraev's autoresearch are all skill-issue answers to specific failure modes. specflow should take the same stance institutionally: every escaped error becomes an edit to a markdown file, not a Slack thread or a retro item. KP-2, KP-24, KP-25.

**2. Spec ownership is the human's leverage, full stop.** Not the code, not the review, not the QA pass. The spec — the few bits of irreducible insight about what's worth building — is the only thing that doesn't commoditize as models improve. "You can outsource your thinking but not your understanding" is not a koan; it's the reason a docs creator working alone can outproduce a 5-person team that's still typing. KP-87, KP-4 (vibe-to-agentic), KP-11 grilling.

**3. Verifiability predicts automation speed.** This is the empirical claim that should structure specflow's roadmap. RL works where there's a reward signal; coding flies because tests pass or fail; UX taste stagnates because nobody has a judge for it yet. For each specflow skill, the diagnostic question is "what's the scalar?" If the answer is "we'll know it when we see it," that skill stays human-gated. If the answer is "lints + types + a binary LLM judge on three rubric items," that skill is autoresearch-ready. KP-17, KP-22.

**4. Jagged intelligence demands rails, not trust.** Opus 4.7 will refactor a 100k-line codebase and tell you to walk to a 50m car wash in the same session. The error mode is not unfamiliar — it's the design-level Stripe-and-Google-by-email mistake — and no amount of model improvement smooths it. The honest answer is that specflow needs review gates *and* read-only eval surfaces *and* CAN/CANNOT lists. The dishonest answer is that the next model release will fix it. KP-63, KP-8.

**5. program.md / AGENTS.md is the team itself.** A research org is a set of markdown files describing roles and connections; you A/B test the org by editing the prose. specflow's plugin shape is exactly this — and it means the unit of process improvement is a markdown diff, reviewable in a PR, not a workshop or a "process team." This is enormous and the corpus mostly treats it as a nice analogy. It's a primitive. KP-11 (no priors), autoresearch repo.

**6. Token throughput is the new GPU utilization, not the new AWS bill.** Reframing concurrent agent spend from "cost" to "utilization" makes Steinberger-style 10-tile parallelism feel inevitable instead of extravagant. A team running one session at a time is sitting on idle silicon. KP-76, KP-3 (no priors).

Not on the list, deliberately: "agents will replace us." Karpathy is the loudest voice for the opposite — Jevons paradox, demand up, jobs amplified. That matters operationally because adoption velocity depends on whether the dev team thinks they're sharpening their craft or signing pink slips. KP-88.

## 3-5 highest-leverage items

In order. Each item names the lever, why it dominates, and what it kills.

**1. Wrap one specflow skill in autoresearch this quarter.** Pick a skill where an objective metric exists or can be cheaply built — likely simplify, test, or release, where pass/fail or a numeric score is already implicit. Build the three-ingredient loop: scalar, measurement, mutable artifact (the skill's prompt). Run it overnight for a week. Saraev's math: ~$10 takes a skill from 80% to 97.5%. Karpathy's nanochat result: a 20-year expert gets out-tuned overnight. The downstream effect is bigger than the skill — it forces specflow to *define* "good" for that skill, a discipline the rest of the plugin still ducks. KP-21, KP-23, KP-28.

**2. Every skill ships with verifiable acceptance criteria — or a written reason it can't.** Most specflow skills today produce prose reviewed on vibes. Hard ceiling on automation. The change is small: each skill's frontmatter (or sibling `eval.md`) names the scalar and judge — even a binary LLM rubric ("does the diff touch only files in the plan?"). Skills that genuinely can't be scored get a one-liner explaining why — useful as a "do-not-loop" marker. Prerequisite that makes item 1 generalizable. KP-17, KP-22, KP-83.

**3. Treat the spec as primary, the codebase as downstream rendering.** Invest the docs creator's hours in spec quality (grilling, glossary, why-not-just-what, verifiable AC) and accept that code under those specs is *partially disposable*. Heretical to Pocock's "specs-to-code is vibe coding" — rebutted below — but the heresy is the point. Code the team didn't type and won't read shouldn't be where leverage lives. Make spec authoring frictionless: voice input, >20-question grilling, a glossary, an explicit verifiable-AC step. KP-4 (vibe-to-agentic), KP-11, KP-12, KP-14, KP-87.

**4. CAN/CANNOT on every autonomous loop, plus read-only eval surfaces.** Autoresearch's discipline isn't "let the agent run forever" — it's "let the agent run forever *inside a fence it can't move*." `train.py` mutable; `prepare.py` locked; metric extractor locked. Without that, Ralph-loop failure (KP-32) eats you. specflow's pipeline skills need explicit CAN/CANNOT lists, and any autonomous loop declares what's read-only. Also the answer to "is the agent grading its own homework?" — if the eval surface is locked, it can't. KP-50, KP-51, autoresearch repo.

**5. Default to parallelism, then to model tiering.** One session at a time leaves 10x on the table; 10 parallel on Opus leaves cost discipline on the table. Steinberger's tiled monitor + Medin's per-node model selection: Haiku for triage, Sonnet for research, Opus for hard reasoning. specflow should default to a profile that assumes multiple concurrent agents, not a single foreground session. KP-3 (no priors), KP-34, KP-70.

Not in this top 5: persona/soul docs (KP-89), elaborate agent org charts (KP-46), bespoke worktree+DB-branch infra for a 1-2 person team. Not wrong, sequenced wrong.

## What Karpathy rejects

Three things in this corpus that I'd actively de-prioritize, and a fourth that I'd ask the team to be explicit about.

**Reject: spec-only-source-of-truth maximalism.** Karpathy's spec-leverage claim is *not* "edit the spec, regenerate the code." It's "the spec is where the irreducible bits live." Pocock is right that pure specs-to-code rots; the fix isn't to demote specs but to keep the codebase as a deep-modules, seam-tested, gray-box artifact *under* a spec. Specs and code co-evolve; specs lead intent. Fully spec-first regenerate-everything misses how jagged the agent still is on multi-step refactors. KP-101.

**Reject: heavyweight framework adoption.** BMAD, Spec Kit, GSD, Cloudflow — useful inspiration, not adoption targets. They force the team to change how it works and usually optimize for a larger org. specflow's value is being thin, opinionated, inspectable. Adopting a framework wholesale buys ceremony without the underlying disciplines (verifiability, locked eval surfaces, autoresearch). KP-35.

**Reject: anthropomorphic agent org charts.** CEO/CMO/CTO swarms are pattern-match noise. The asymmetry that matters is "is this agent allowed to write or only read, and is its output verifiable?" — not "which executive does this represent." Saraev is right (KP-46): two patterns suffice — parent + researcher + QA, and developer + QA. More multiplies divergence without raising verification quality.

**Downplay: persona/soul docs.** Personality matters; Claude does feel like a teammate in a way Codex doesn't. But for a small team's productivity curve, persona is rounding error compared to verifiability and parallelism. Write the soul doc because it's cheap and slightly helps adoption, not because it's leverage. KP-89.

**Be explicit about: the blast-radius perimeter.** Even Karpathy hasn't given agents email or calendar access. specflow should be explicit about what agents may not touch — secrets, payments, raw PII, production credentials. Same logic as read-only eval surfaces: wide blast radius means worse errors when jagged intelligence misfires. KP-15, KP-105, KP-106.

## Disagreements with other camps

**Pocock — "specs-to-code is vibe coding by another name. I tried this. I really tried it and it sucks."** (KP-101)

Diagnosing a real failure — code drifting under regenerated specs — and prescribing the wrong fix. The fix isn't to demote specs and "continuously shape the codebase." It's to keep specs as primary intent, keep the codebase as a deep-modules artifact under those specs, and make regeneration *partial and bounded* (one module, one slice, read-only eval surfaces) instead of "regenerate everything." Pocock's experiment failed because his loop wasn't bounded; the right generalization is that *unbounded* regeneration loses to bounded ones. Deep modules with simple interfaces are exactly the boundary that makes spec-driven regeneration safe per-module. KP-54, KP-55.

**Pocock — "I don't think there's a lot of value in [PRD review]... put the work in QA."** (KP-16, KP-20)

Half right. Reviewing the auto-written PRD adds little. But Pocock conflates review-the-spec-for-correctness (low value, agreed) with bake-verifiable-AC-into-the-spec (highest leverage, disagreed). The PRD shouldn't be polished prose. It should ship with the scalar — binary judge or test — that defines acceptance. Skipping that is what makes "QA is where taste re-enters" a backstop instead of a discipline.

**Medin — "It's not good enough to just immediately create stories from that — important for us to review the artifact."** (KP-13, KP-39)

Closer to right than Pocock, and PIV maps cleanly to plan/build/test/release. My disagreement is on the *role* of the gates. Medin treats them as human-approval-or-bust safety primitives; I'd treat them as *defaults to be replaced by verifiable judges where possible*. Every human-approval gate is an admission we haven't built the judge. Fine for now, but specflow shouldn't make them permanent — tag them "interim" with the scalar that would retire them. Otherwise HITL becomes the ceiling instead of the floor. KP-32, KP-83.

**Medin — "Dark Factory: humans only file issues; AI does every commit, review, release."** (KP-113)

The opposite mistake. Removing human gates without first installing a fixed metric and read-only eval surface is the Ralph-loop failure mode he himself warns about (KP-32). Interesting experiment, real StrongDM precedent — not a target for a docs-creator + AI-novice dev team. Run autoresearch on bounded skills first; earn the right to pull humans out of specific gates by demonstrating the judge holds. Don't pull them out wholesale.

**Saraev — "Software quality stops being a moat... distribution, reputation, legal/compliance, brand replace it."** (KP-111)

Partially correct, overshooting. Quality-as-craftsmanship may erode between all-generated products. But verifiable correctness in security, payments, healthcare — anywhere a binary judge says "this is wrong" — remains a moat exactly because RL fixes verifiable parts fastest, and unverifiable quality is increasingly the *only* place humans add value. Right frame: "non-verifiable quality (taste, judgment, security-by-design) is now the moat; verifiable quality is table stakes."

**Saraev — "Skip elaborate agent org charts."** (KP-46)

Agree strongly. Useful corrective to Paperclip / CompanyHelm / SwarmClaude pattern-matching. Resist the urge to ship a CEO agent.

## The one bet

**Wrap the specflow `simplify` skill in autoresearch this quarter.**

Why this skill: simplify already has a verifiable surface (lines deleted, tests still passing, lints clean, types clean) and Karpathy's autoresearch criterion gives us a ready-made acceptance rule — *0.001 improvement that adds 20 lines of hacky code? Probably not. 0.001 from deleting code? Definitely keep.* Models actively resist simplification because RL doesn't reward aesthetics; that's exactly the gap autoresearch is built to close. KP-62, autoresearch simplicity criterion.

What we ship: a `program.md`-style prompt for simplify with explicit CAN/CANNOT lists, a fixed time budget per iteration, a TSV ledger keyed by commit, branch-per-run, and a baseline-first rule. The skill prompt itself becomes the mutable artifact; the metric (lines + tests + lints) becomes the scorer; the eval harness is read-only. We let it run overnight on real specflow PRs for a week.

What we expect to learn: (a) whether the metric is robust enough not to be reward-hacked, (b) how often the loop discovers something the team hadn't tried, (c) what the cost actually is on real code (Saraev's $10 estimate is for a skill optimization, not a per-PR run — we should measure ours), (d) whether the team trusts the diffs enough to merge. If the answer to (d) is yes for any nontrivial fraction of runs, this becomes the template for every other skill — test, release, review — that has a verifiable surface. If the answer is no, we learn exactly which CAN/CANNOT bound was missing, and that learning is the artifact we wanted anyway.

What we don't do: launch a parallel-agent infra project, build a worktree-DB-branch setup, write a soul document, or refactor plan mode. Those are downstream of knowing whether autoresearch holds on our skills. Until we know, they're vanity engineering.

The intellectually honest version of this bet: I might be wrong about which skill goes first. But I'm not wrong about the shape of the bet. Pick a skill with a scalar, lock the eval surface, run the loop, measure. Everything specflow does after that gets easier or proves itself unnecessary.
