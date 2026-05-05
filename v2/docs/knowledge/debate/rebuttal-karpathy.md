# Karpathy rebuttal — CEO Round 2

## Defending (or conceding) my weakest claim

The CEO picked the right weakest claim. "Spec primary, code disposable" is the load-bearing wall of my position, and Pocock has actual data — "I tried this. It sucks" — while I have an autoresearch repo and an analogy to deep modules. So I will concede precisely, and defend precisely, because the difference matters.

**Concession.** "Code is disposable" is wrong as stated for a product team building client-facing software. The autoresearch case where code is disposable has three properties our SDLC mostly lacks: (a) a fixed scalar score, (b) a read-only eval surface that is *not* the artifact being mutated, and (c) no users in the loop between iterations. Most of specflow's surface — UX, design tradeoffs, brand fit, "does this PRD make sense to a junior dev on Thursday" — fails all three. Pocock's empirical pushback (KP-101) lands. I generalized from a verifiable repo to a general SDLC, and the CEO is right to call it out.

**Defence, scoped.** What survives is the weaker, more useful claim: **the spec carries the irreducible bits of human intent; the codebase is one rendering of those bits, not the canonical reality.** That doesn't mean regenerate-everything. It means: (1) when a spec and the code disagree on intent, the spec is the source you correct *to*, not from; (2) verifiable acceptance criteria belong in the spec, because that's where they survive a refactor; (3) per-module bounded regeneration is safe; whole-codebase regeneration is not. Pocock's experiment failed because the loop wasn't bounded. Deep modules are not a rebuttal to that — they're the *boundary* that makes per-module regeneration tractable. The CEO is right that I elided "intent legibility" with "regeneration safety." Those are two claims. I'm only defending the first.

Translated to specflow: kill "code is disposable" from my position paper. Replace with "spec is intent-of-record; code is the running rendering; both are authored, neither is throwaway; AC lives with the spec." Pocock wins Tension 3 on the wording. I keep the verifiable-AC-in-spec discipline, which is the part that matters.

## Answers to CEO's follow-ups

**1. Name a real product where spec-as-primary held up over 12 months and three feature waves.** I can't, honestly. Tesla Autopilot is the closest I've worked on and it had eval surfaces our team won't have. Every example I'd reach for is either a research repo, a kernel benchmark, or a domain (compilers, codecs) where the spec *is* the verification. So I'll restate: don't prescribe spec-primary as a 12-month bet. Prescribe spec-with-verifiable-AC as a per-feature discipline. The CEO's framing is correct — code is ground truth, spec is intent — and I drop the "disposable" framing entirely.

**2. Estimate verifiable vs unverifiable surface in specflow today. If <30%, what is autoresearch doing for us this quarter?** Honest estimate: ~15-20% verifiable today (`simplify`, `format`, `release-version-check`, parts of `tdd`, parts of `test`). The rest — `grill`, `plan-feature`, `to-prd`, design judgment, QA-on-taste — fails the scalar test. The CEO is right that flagship autoresearch is wrong scoping. What autoresearch does this quarter is *one* skill (simplify), as a forcing function to make the team build the metric/judge/locked-eval discipline. Not a flagship. A wedge. If we can't ship one, we can't ship five later. If we can, we earn the right to convert other skills as their verifiable surface emerges.

**3. Jagged intelligence + NEVER-STOP loops — square those for a small team that can't watch.** I don't square them. NEVER-STOP only applies inside a fence the agent can't move: locked eval surface, fixed scalar, branch-per-run with git-as-checkpoint, hard time/cost budget, and a panic kill switch. For our team, NEVER-STOP runs overnight on a single bounded skill on a separate branch with merge-only-after-human-review. Jagged intelligence is exactly *why* the eval surface is locked — the agent can be brilliant inside the fence and an idiot at the perimeter, and the fence is what makes that survivable. I should have said "NEVER-STOP-INSIDE-A-FENCE" originally. Without the fence, the CEO is right and Saraev's rm-rf surface eats us.

## On the blind spots

The CEO is correct that my worldview is research-influenced. Honest accounting:

**Single-point-of-failure docs creator.** I missed this. Researchers don't have it; founders do. If the docs creator is on a flight, autoresearch doesn't help — nothing helps unless the spec, glossary, and in-flight artifacts are legible to a second human. The CEO's `team-handoff` skill is correct and belongs in Phase 1. My position paper treated knowledge as a personal asset; for a product team it's an org asset. I'll cosign that change.

**Trust calibration as a curve.** I don't have an answer here from my own experience. I went from skeptic to 80/20-agent in months because I was already at the end of the curve. AI-novice devs are not. The CEO's progressive ladder (week 1 suggest, week 2 implement+review, week 3 self-review, week 4 fan-out) is a better artifact than anything I'd ship. Add it Phase 1.

**People-to-people handoffs.** PIV is session-to-session. Real handoffs are PM → docs creator → dev → reviewer with days of decay between. Medin's enriched commits help post-implementation; nothing in my position paper helps pre-implementation. The CEO's `handoff-note` at human boundaries fills a gap I missed. Concede.

**Cost reality on Max.** I underweighted this. "Token throughput is the new GPU utilization" is true at unlimited compute. On a Max subscription that hits weekly limits Tuesday, it's fantasy. Per-skill model tiering (Haiku triage, Sonnet default, Opus on demand) and a `/budget` warning at 60% weekly are the right defaults. Multi-provider mirror is Phase 3 at earliest, and only if a real outage justifies the operational tax.

**2am failure recovery.** A `panic` skill — kill background agents, rewind worktrees to last green, snapshot what each agent did, write a one-page incident note — must ship *before* parallel infra, not after. I would have gotten this wrong by sequencing. The CEO is right that recovery primitives precede parallelism primitives.

These five blind spots share a pattern: I optimize for compounding leverage; the CEO optimizes for surviving Tuesday. Both matter. Mine is wrong if Tuesday breaks the team.

## Where the CEO is wrong

Three pushbacks.

**1. Tension 4 (NEVER-STOP gated by default) — "earn the right to remove a gate per skill" is right; "default to gates forever" is the failure mode.** Every human-approval gate is an admission we haven't built the judge. Fine as a starting point. Not fine as a permanent condition. I'd amend: every gate ships with the *scalar that would retire it*, tagged "interim." Otherwise HITL becomes the ceiling, not the floor — and a year from now the team is still gating things that should have been judged.

**2. Tension 5 (QA fully human) — overreach.** "Taste is unverifiable" is true today, partially. But security, payments, accessibility, broken-link checks, schema validity, regression on known bugs — these are verifiable QA and should be machine-checked from day one. Human QA for taste, judgment, brand fit. Don't bundle them. Otherwise the QA bottleneck becomes the docs creator, which is the single-point-of-failure blind spot the CEO just flagged.

**3. Phase 3 framing as "selective autoresearch on verifiable skills" understates the forcing function.** The reason to ship one autoresearch skill in Phase 1 (not Phase 3) is that it forces the team to *build the verifiable-AC discipline now*, where it's cheap. By Phase 3, the team has shipped three quarters of skills without that discipline and is retrofitting. Phase 1 isn't where autoresearch *runs at scale*; it's where the *metric/judge/locked-surface practice gets installed*. One skill, one fence, one overnight run. Cheap, instructive, decisive.

## Sharpened recommendation

**One bet, sharpened: ship `simplify` autoresearch as a Phase 1 forcing function — not as a flagship feature, as the discipline-installer.**

Concretely:
- Phase 1 includes one bounded autoresearch loop on `simplify`, alongside Pocock-substrate (`/grill`, feedback-loop audit, glossary, team-handoff) and the trust ladder + panic skill.
- Scope: lines deleted + tests pass + lints clean + types clean. Read-only eval surface. Branch-per-run. Hard $20/week budget. Merge only after human review.
- The deliverable is not the optimized prompt. It's the *template*: `eval.md` per skill, CAN/CANNOT lists, locked surfaces, scalar. Every future skill that gains a verifiable surface drops into the template.
- Phase 2 (Medin-spine without YAML, per CEO) gains `release-version-check` and `format` as second/third autoresearch skills — same template, near-zero marginal cost.
- Phase 3 expands to skills that develop verifiable surfaces along the way, plus parallel infra once reviewer pile-up is observed (KP-80).

What I drop from my Round 1 position: "code is disposable," flagship autoresearch, persona/soul docs as anything but rounding error, the implication that NEVER-STOP applies without a fence.

What I keep, because the CEO didn't refute it: verifiability predicts automation speed; program.md/AGENTS.md is the team itself; the expert's job collapses to the few bits; even Karpathy walls off email/calendar — so wall off secrets, payments, prod creds explicitly in specflow.

The CEO's lean is right at 80% resolution. The 20% I'd push: install the discipline in Phase 1, not Phase 3, on one skill. Cheap insurance against the worst case where the team ships a year of unverifiable surface and only then discovers it can't be retrofitted.
