# Medin rebuttal — CEO Round 2

## Defending (or conceding) my weakest claim

The CEO picked the right target. I conflated two things in Round 1 and that conflation deserves a clean concession.

**Concede**: Shipping a YAML DAG schema — `depends_on:`, `trigger_rule: one_success`, `loop … until: APPROVED` as declarative primitives our team authors and debugs — is exactly the BMAD-shape failure I correctly call out elsewhere. The CEO is right. The moment the docs creator has to debug *why* a `trigger_rule` didn't fire, we have shipped Archon-junior, and we have failed every audience specflow serves.

**Defend**: The *underlying* claim — that determinism belongs in scripts, not prompts — survives. What I was actually arguing for, and what I should have said cleanly the first time, is:

- **Fresh-session boundaries enforced by the skill, not by YAML.** `/execute` opens a new session and reads only `plan.md`. That's a 5-line skill instruction, not a DAG.
- **Path-scoped rules via `paths:` frontmatter.** This is already a Claude Code primitive. We're using it, not inventing it.
- **Per-skill model pinning in skill frontmatter.** Same — leverage the existing harness; don't build a new one.
- **`interactive: true` as a skill-level flag, not a YAML control-flow node.** A boolean on a skill is not a DAG.

The difference is real: skill frontmatter is *configuration the harness already understands*; a YAML DAG is *a new language the team has to learn to debug*. I conflated them in the position paper and I shouldn't have. Concede the DAG; keep the discipline.

So: drop "ship `depends_on:`/`trigger_rule:`/`loop … until: APPROVED` as primitives" from my recommendation. Replace with: "enforce phase boundaries inside skills, using the frontmatter Claude Code already parses."

## Answers to CEO's follow-ups

**Q1 — "Show the docs creator debugging a failed `trigger_rule: one_success` alone at 11pm. What's the diagnostic ladder?"**

There isn't one. That's the concession. If we shipped it, the ladder would be: read YAML spec, inspect run-state JSON, reason about which upstream node returned what, eyeball the trigger expression, ask Claude to explain its own DAG. Five rungs of state the docs creator never signed up for. Don't ship it. Skill-level `interactive: true` flags fail visibly in the chat — a skill that requires approval blocks and asks. That's a debuggable surface for a non-engineer.

**Q2 — "If the plan is wrong, the fresh implementer can't push back because they have no chat history. Doesn't PIV entrench misalignment?"**

Real risk, partial answer. PIV does *not* solve "wrong plan" by itself. It solves "biased reviewer." The guard against wrong plans is upstream: Pocock's grilling step, plus a plan.md review by the docs creator before `/execute` runs. The fresh implementer's job isn't to push back on the plan — it's to detect *plan-vs-codebase* contradictions. Cole's drift-check (KP-11 in the PIV analysis) is the formal step: after implementation, the agent compares the produced code against `plan.md` and flags deviations. That catches "the plan said store user IDs, but the schema is keyed on email" — which a biased same-session implementer would silently paper over. PIV doesn't fix wrong plans; it fixes *wrong reviews of wrong plans*. Pocock's grill is upstream of PIV, not in tension with it.

**Q3 — "On a Max plan that hits weekly limits Tuesday, what's the token cost of `ask_user_question` across a quarter?"**

Fair challenge. I waved at it. Honest answer: I don't have the measurement, and neither does anyone else in this debate. What I can defend with numbers is the *upper bound*. JSON-wrapping a multi-choice question costs ~150–300 tokens vs ~50–100 for plain prose. A 22-question grilling session at the high end is 22 × 250 ≈ 5,500 extra tokens. A weekly limit on Max is on the order of millions. Even if we ran ten grills a week, that's <1% of the limit. The CEO's split-the-baby in Tension 6 — "plain conversation for grilling, structured for short mid-execution clarifications" — is the right answer and I accept it. My Round 1 framing ("token-cost objection optimises a rounding error") was correct on the math but wrong on the rhetoric: I should have proposed the same split.

## On the blind spots

The CEO is right that none of the four advisors built primitives for the actual operating reality. PIV/harness thinking partially answers some, not others.

- **Single-point-of-failure docs creator.** PIV helps: `plan.md` and enriched commits are durable artifacts that survive the docs creator going on holiday. The drift check + postmortem gives someone else a paper trail. PIV does **not** solve "PM has no one to grill with on Tuesday." That's a `team-handoff` skill, and the CEO is right to surface it. Concede: ship `team-handoff` in Phase 1, ahead of any of my five recommendations.
- **Trust calibration curve.** PIV doesn't address this at all. The CEO's progressive ladder (week 1 suggests, week 4 fan-out) is a *training intervention*, not a harness intervention. Per-skill `interactive: true` is the lever — but you have to flip it deliberately as the team graduates. Add: ship a `getting-started` profile with `interactive: true` *everywhere* by default, with one explicit upgrade ritual per skill.
- **People-to-people handoffs.** PIV solves session-to-session. The Tuesday-PM-Wednesday-docs-creator-Thursday-dev gap is a different shape. Enriched commits + plan.md help; a stale-tag on artifacts > N days old is the missing primitive the CEO names. I accept the addition.
- **Cost on Max.** Per-node model selection is exactly the answer here. Pin Haiku for triage and you stretch the Max plan. But I overclaimed when I said "37% of 5-hour limit running 4 parallel" generalised. It generalises *for content workflows*, where parallel agents read mostly-static research material. Code workflows churn larger contexts. Concede: the 37% is Cole's number, not ours. Replace with: ship `/budget` warning at 60% weekly, default to Sonnet, escalate to Opus only on `interactive: true` request.
- **2am failure recovery.** PIV does not answer this. The CEO is right that `panic` belongs *before* parallel infra. Worktrees without a kill-switch is an own-goal. Move my "parallel agents on git worktrees" recommendation behind `panic`.

## Where the CEO is wrong

I'll pick two battles and concede a third.

**Tension 3 — Code as ground truth, spec as intent.** I largely agree with the CEO here, against Karpathy's strong form. But the framing "code is ground truth" understates what `plan.md` is for. The plan isn't a parallel canonical reality; it's the *contract* the implementer is held to and the validator checks against. If we let "code is ground truth" devolve into "the diff is self-explanatory," we lose the drift-check that catches "the plan said X, the code does Y." Refinement, not reversal: code is ground truth for *what shipped*; plan.md is ground truth for *what we agreed to ship*. Both matter. The drift check is the bridge. Don't throw it out.

**Tension 5 — Human QA.** Concede mostly. Pocock's right for our team's product surface (taste, UX, brand fit). But I want to retain *function-based fan-out reviewers* — code-simplifier, error-handling, test-coverage running in parallel — feeding into the human QA gate. Saraev was wrong about org-chart fan-out; he wasn't wrong about dimensional reviewers. The CEO's adjudication should read: human is the QA gate; multi-reviewer-by-dimension is upstream of the human, accelerating their job, not replacing it.

**Tension 8 — Own thin harness, don't import Archon's YAML.** The CEO is right and this is the cleanest restatement of my concession above. specflow IS the harness. Stealing PIV phase boundaries and `paths:` frontmatter ≠ shipping a YAML DSL. I should have written the position paper this way from the start.

The CEO is also right that I underweighted *deletion*. Every artifact-as-durable-memory framing accumulates. Pocock's doc-rot warning is real. Add a `/prune` quarterly ritual to my recommendation.

## Sharpened recommendation

Phase fit, with the concessions baked in.

**Phase 1 — Pocock substrate + one Medin primitive.** Ship `/grill`, `feedback-loop-audit`, `CONTEXT.md`, and `team-handoff` (the CEO's missing primitive). Add the *one* Medin item that fits Phase 1's "light foundation" framing: **migrate to `paths:`-frontmatter rules and cap CLAUDE.md at 500–700 lines**. This is harness hygiene using existing primitives, no new schema, no team retraining. Everything else of mine waits.

**Phase 2 — PIV as convention, not DAG.** This is my one bet, sharpened:

1. `/plan-feature` writes `.specflow/plans/<slug>.md`. No code edits. Borrow WISC's 5-phase template.
2. `/execute` opens fresh session, reads only `plan.md`. Hard-coded skill instruction, not YAML.
3. `/review` opens third fresh session, reads diff + plan for drift check.
4. `/postmortem` turns escaped bugs into rule edits.
5. Per-skill model pinning in frontmatter (Haiku/Sonnet/Opus declared per skill). `interactive: true` on every skill in the `getting-started` profile.

No `depends_on:`, no `trigger_rule:`, no DAG. Phase boundaries are conventions inside skills, enforced by what each skill chooses to read. The harness primitive Claude Code already gives us is enough.

**Phase 3 — earn parallelism and cross-provider.** `panic` skill ships *before* worktree script. Worktree `w` script gated on the bottleneck-on-review trigger. `/optimize` only on the genuinely verifiable skills. Codex adversarial review as opt-in for high-stakes PRs. `/prune` quarterly.

**The one bet, restated**: PIV as convention, enforced inside skills via the frontmatter Claude Code already parses, with `team-handoff` and `paths:`-scoped rules as preconditions. No new DSL. No DAG. The harness *is* the lever — but the lever is the discipline of fresh-context handoffs, not a YAML schema.
