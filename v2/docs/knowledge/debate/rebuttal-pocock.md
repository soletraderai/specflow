# Pocock rebuttal — CEO Round 2

## Defending (or conceding) my weakest claim

The CEO picked "skip reviewing the auto-written PRD" as my weakest claim. I'm going to **partially concede** — and then sharpen the part worth defending, because the CEO's framing collapses two different things.

**What I concede.** The CEO is right that "skip the PRD" generalises from my own seniority. I have a misalignment-radar built from years of grilling vague briefs into shipped code; a docs creator working alone and AI-novice devs do not. Telling them "trust the LLM summary" is telling a junior to drive without mirrors because I drive fine without them. So: yes, the PRD review stays. Not as a polishing exercise — as a 5-minute *misalignment scan*. One pass. Skim. Look for the thing you didn't say in grilling that the LLM invented anyway.

**What I do not concede.** The CEO frames PRD review as "the second-cheapest catch point." That's true only if the review is the *right* review. The trap I was warning against is the Medin trap: PRD review as line-edit, prose-polish, comment-thread-on-paragraph-2. That kind of review *is* worse than useless — it converts attention into busywork, and the team learns to rubber-stamp because they've been trained to find typos and they find typos. The defensible version of my claim:

> The PRD review is a 5-minute misalignment scan, not a quality bar. The quality bar is the running code in QA. If a team is spending more than 10 minutes reviewing a PRD, they have substituted PRD-comfort for code-truth and they will ship slop on schedule.

So: keep the PRD review, **constrain its shape**. Specflow should ship the review as a checklist with a hard time budget, not a free-form artefact-edit ritual. Three questions: (1) Does this match what we agreed in grilling? (2) Are the acceptance criteria verifiable? (3) Is there an assumption in here we never discussed? If yes-yes-no, ship it. The artefact is a hint of direction, not a contract — but it is a *checked* hint.

The PRD review is the safety net. Grilling is the trapeze. I was wrong to argue against the net; I was right to argue against turning the net into a hammock.

## Answers to CEO's follow-ups

**1. "If grilling fully substitutes for PRD review, why does your `to-prd` skill produce a structured PRD at all?"**

It doesn't substitute. The PRD has three legitimate jobs: (a) it's high-attention LLM food for downstream sessions that didn't attend grilling — KP-11 from the feature build, attention mechanisms hot-spot collocated content; (b) it's the input to the issue-slicer; (c) it's a one-page snapshot a teammate can read in 5 minutes. None of those jobs require humans to *polish* it. They require a human to *check it didn't drift* from grilling. The PRD exists as a thin byproduct, not as a deliverable. Concession noted; the artefact stays, the polishing ritual doesn't.

**2. "How does a junior dev — who didn't attend grilling — get aligned before implementing?"**

This is the strongest question the CEO asked. My answer in the position paper assumed the docs creator IS the implementer or in the chair with them. That's wrong for specflow's actual team shape.

The fix is twofold. First, the grilling Q&A transcript ships *with* the PRD into `.specflow/discoveries/<slug>.md` — verbatim, not summarised, because Q&A is high-attention LLM food and high-attention human food too (the junior reads "should points be retroactive? Yes, because retroactive deletes feel like punishment" and gets the *why*, KP-3 from feature build). Second, the implementer's first action is a 60-second `align-check` skill: "given this PRD and Q&A, name the one thing you'd push back on if you were the docs creator." If they can't name something, they didn't read it. That's the junior version of my "look harder" reflex (KP-12 feature build), made operational. The CEO's blind spot 6 ("when to override the AI") is the same skill applied in the other direction.

**3. "QA is after implementation cost is sunk. Why throw away the PRD review as the second-cheapest catch?"**

I'm not throwing it away — see concession above. But the CEO is implying a cost ordering that I disagree with: grilling cheap → PRD review next-cheapest → QA expensive. That's wrong. **PRD review is only second-cheapest if it catches misalignment**. Most PRD reviews don't — they catch typos and missing AC. The grilling Q&A is where 80% of catchable misalignment lives; the PRD review catches maybe 15% (drift between Q&A and PRD); QA catches the remaining 5% plus everything that *can't* be caught upstream (KP-16 feature build: "directory creation succeeds but git init fails" — no PRD review catches that). The cost-ordering the CEO defends only works if PRD review is sharply scoped. Make the scoping explicit in the skill or it bloats.

## On the blind spots

**Single-point-of-failure docs creator.** My worldview *partly* answers this and partly doesn't. The honest read: I work solo, so I built primitives that assume the chair is staffed. The CEO is right that specflow needs a `team-handoff` skill producing a "what's in flight, what's blocked, where artefacts live" snapshot, plus a CLAUDE.md "if the docs creator is unreachable" section. I'd add: the grilling Q&A transcripts *are* the handoff artefact — they survive the docs creator going on holiday in a way a polished PRD doesn't, because they preserve reasoning not just decisions. That's a strength of my worldview the CEO undersells. But the explicit handoff skill? Yes. Ship it Phase 1.

**Trust calibration curve.** I do not have an answer here. My worldview assumes calibration is implicit ("seniors know what to grill for"). For AI-novice devs that's just hand-waving. What would work: a `getting-started` profile with progressively-unlocked autonomy levels — week 1 suggest-only, week 2 implement-with-reviewer, week 3 self-review, week 4 fan-out. Tie unlocks to a concrete ritual (a postmortem of the previous week's escapes), not a calendar tick. This is closer to Saraev's "agents have no agency by default" inverted onto the human. I'd ship it.

**People-to-people handoffs.** My worldview is silent here. PIV is session-to-session; my framing is human-to-agent. Neither covers PM → docs creator → dev → reviewer over four days with context decay. I don't have a primitive for this and I shouldn't pretend to. The CEO's `handoff-note` skill at the human boundary plus a "stale" tag on artefacts older than N days is the right shape; I'd add that the staleness tag should *gate* the agent (refuses to read a >14-day-old PRD without re-grilling).

**Cost reality on a Max subscription.** My worldview includes this — KP-17 feature build (don't use AskUserQuestion because of JSON token cost), KP-49 elsewhere — but only at the skill-design level, not the budget level. A `/budget` skill that warns at 60% weekly and a model-pinning rule (Haiku triage, Sonnet default, Opus on demand) is a Phase 1 hygiene primitive I should have flagged.

**2am failure recovery.** Silent. My answer of "AFK skills require feedback loops" is necessary-not-sufficient. The CEO's `panic` skill — kill background agents, rewind worktrees to last green, snapshot what each agent did — belongs *before* parallel infra, not after. Conceded.

## Where the CEO is wrong

The CEO sided with me on Tensions 3 (code as truth) and 5 (manual QA). They got Tensions 1, 2, 6, and 9 right enough. Where I push back:

**Tension 4 — "Default to gates, earn the right to remove a gate per skill."** Right verdict, wrong framing. The CEO talks about "earning" gate removal as if it's a graduation. It isn't. A skill is AFK-eligible *iff* it has (a) a binary verifiable judge and (b) a reversible blast radius. Both, always. There's no graduation curve from "gated" to "ungated" — there's a per-skill, per-environment classification. `format-code` is AFK on day 1 because it meets the criteria; `improve-codebase-architecture` is HITL on day 1000 because it never will. Frame it as a property of the skill, not a maturity of the team, or specflow ships skills that "earn" autonomy by surviving long enough to bypass their own safety case.

**Tension 7 — "Function-based fan-outs fine, role-based bad."** Agree on the conclusion, but the CEO accepts function-based fan-out too easily. Multi-reviewer-by-dimension still hits the multiplicative-reliability problem (0.9⁵ = 0.59). Five reviewers each missing different 10% means 41% of issues aren't caught by *any* of them. The escape hatch is consolidation: a single human integrator who reads all five reviews and decides. If specflow ships team-review without the integrator step, it's just role-based fan-out wearing a function-based costume.

**Phase 3 framing.** The CEO has Phase 3 as "selective Karpathy + Saraev + the trust ladder." That bundles two unlike things. The trust ladder (`confidence-check`, `panic`, `tour`) is foundational safety — it should be Phase 1 alongside `/grill`, not Phase 3 alongside autoresearch. You don't introduce panic skills *after* the team has been running parallel agents; you ship them *before*. Move trust-ladder primitives forward; keep `/optimize` and worktree infra in Phase 3.

**The "Pocock-substrate + Medin-spine" framing.** Calling Phase 2 "Medin-spine without the YAML" undersells how much of what survives is mine. PIV phase boundaries enforced as fresh-session conventions = Pocock's HITL/AFK frontmatter applied to sessions. Path-scoped rules = good but pre-existing Claude Code primitive, not a Medin invention. Postmortem skill = Pocock's "QA findings feed new issues" loop. The honest framing: Phase 2 is Pocock with one Medin convention bolted on (fresh-context per phase). Don't credit Medin with the spine; credit him with the convention.

## Sharpened recommendation

**One bet for the quarter, revised: ship `/grill` plus the misalignment-scan PRD review and the human-handoff primitive, as a single connected ritual. Not three separate skills — one ritual with three checkpoints.**

Concretely:

1. `/grill` blocks plan mode until `.specflow/discoveries/<slug>.md` exists with verbatim Q&A.
2. `to-prd` runs against the discovery file and produces `.specflow/prds/<slug>.md` with grilling Q&A appended verbatim — **not** summarised.
3. PRD ships with a 3-question misalignment-scan checklist as a top-of-file frontmatter block. Reviewer ticks three boxes or files an issue. 5-minute time budget enforced by the skill itself ("if you're past 5 minutes, stop and re-grill").
4. `team-handoff` runs at every human-to-human boundary, snapshots in-flight discoveries/PRDs/issues, tags anything >14 days as stale.
5. HITL/AFK frontmatter on every skill, day one. No graduation curve — declared property.

Everything else — feedback-loop audit, CONTEXT.md, deep-modules review, manual-QA-as-feedback — stays Phase 1. Trust-ladder primitives (`confidence-check`, `panic`, `tour`) move *into* Phase 1. `/optimize`, worktrees, multi-provider mirror stay Phase 3.

The bet I'm not making: I am not betting on grilling alone. The CEO talked me out of that. I'm betting on **grilling + a thin misalignment-scan + a human-handoff primitive** as the minimum viable substrate for a docs-creator-led team. That's the ritual. Anything thinner is unsafe for our personas; anything heavier is a YAML harness in disguise.

Code is still the battleground. The spec is still scaffolding. But the scaffolding now has a 5-minute checked gate, because the climbers are new.
