# Saraev rebuttal — CEO Round 2

## Defending (or conceding) my weakest claim

Concede the framing. Defend the artifact.

You're right that I oversold `/optimize` as the *flagship*. The unverifiable skills — `grill`, `plan`, `design`, `QA`, `simplify-with-taste` — cannot be autoresearched without the LLM judge reward-hacking, full stop. If I sold you a $200 catalog optimization across all 15-20 skills, that was YouTube-thumbnail thinking. I'll wear it.

But the bet underneath the framing still ships. Here's the sharper version:

`/optimize` ships against a curated subset — the verifiable skills only — and the optimization log + binary-eval discipline becomes the *cultural* artifact, not the catalog scan. Concretely:
- **Verifiable today**: `release-version-check` (does `plugin.json` match `marketplace.json`? binary), `simplify` (does the diff reduce LOC while tests pass? binary + binary), `format` (does the file match prettier output? binary), `tdd` (red→green→refactor cadence detectable? binary), `init` (does the produced CLAUDE.md include the four sections we declared? binary), `feedback-loop-audit` (do test/typecheck/e2e exit codes meet thresholds? binary).
- **Not verifiable, do not autoresearch**: `grill`, `plan-feature`, `design`, `QA-judgment`, `simplify-with-taste`, `to-prd`. These get human judges and Lab Notes, not autoresearch.

That's six skills, not twenty. ~$60, not $200. And the value isn't the dollar saved — it's that the team learns to write a binary eval before they write a skill prompt. *That's* the compounding asset. The optimizer is the forcing function.

So: concede the catalog flagship. Keep the discipline. `/optimize` is a Phase 3 surgical tool, not a Phase 1 banner.

## Answers to CEO's follow-ups

**Q1 — Three skills with non-gameable binary evals that don't reduce to "an LLM thinks it's good"?**

Easy. (1) `release-version-check`: read `plugins/specflow/.claude-plugin/plugin.json` version, read `.claude-plugin/marketplace.json` `metadata.version` and `plugins[0].version`, read latest GitHub release tag — three string comparisons, zero LLM in the judge. (2) `simplify`: run the test suite before, run it after, count LOC delta — `tests_pass_after AND loc_after < loc_before` is two booleans, zero LLM. (3) `feedback-loop-audit`: does `npm test` exit 0 in <60s, does `tsc --noEmit` exit 0, does the e2e harness exit 0 — three exit codes, zero LLM.

I can name three more if you want: `format` (diff against prettier), `tdd-cadence` (git log shape), `init` (file existence + section header regex). The list of skills with LLM-free binary evals is *exactly* the list `/optimize` should target. Everything else is off-limits.

**Q2 — "Decreasing human involvement is monotonic" vs "jagged intelligence" — which is it?**

Both, on different time axes. Monotonic is the *trend* across model generations (Opus 5 will cede gates Opus 4.7 needs). Jagged is the *capability surface within one generation* (Opus 4.7 can refactor a 200-file codebase but emails Stripe a credit card number). The reconciliation: human involvement decreases monotonically *for tasks with verifiable surfaces* and stays jaggedly elevated *for everything else*. Phase out gates per skill, when the metric earns it. Never blanket. I was sloppy in the position paper — fair catch.

**Q3 — Cross-provider review ROI for a 1-person ops team?**

Lower than I implied. For a solo docs creator the operational tax (two harnesses, two key sets, two JSONL leak surfaces, two skill catalogs to maintain) probably exceeds the marginal review benefit until the dev team is 3+. Revised position: *single-provider mirror file* (CLAUDE.md → AGENTS.md auto-generated, never edited by hand) is cheap insurance and ships day one. *Cross-provider adversarial review* is a Phase 3 opt-in gated on team size ≥ 3 *and* a real Anthropic outage having cost the team >4 hours. Not before.

## On the blind spots

Honest answers, since you asked:

**Single-point-of-failure docs creator.** You're right and my worldview has nothing to say about it. Autoresearch optimizes skills; it doesn't replicate humans. The Pocock/Medin answer (`team-handoff` skill, "if docs creator unreachable" CLAUDE.md section) is correct and I'd ship it Phase 1. My only addition: the handoff snapshot itself is a candidate for autoresearch later — binary eval is "did the receiving human unblock without asking the absent person a question? yes/no" measured over 5 handoffs. But that's Phase 4. Phase 1 just write the skill.

**Trust calibration curve.** Genuine blind spot. My "agents have no agency by default" framing addresses the *agent* side; you're pointing at the *human* side and you're right that AI-novice devs need a progressive ladder. The fix is consistent with my worldview though — the ladder *is* a metric. Each rung has a binary eval ("did the dev catch the AI mistake before merge? yes/no"), and you graduate when the rolling average crosses a threshold. So: ship the curve, but instrument it. Don't leave it as vibes.

**People-to-people handoffs.** Same story as single-point-of-failure. My toolkit is silent here. The PIV pattern is session-to-session; humans on Tuesday-vs-Friday are out of scope for everything I've published. Concede. Pocock/Medin own this one.

**Cost reality on a Max subscription.** Fair. The "37% of weekly limit" demos are flexes, not baselines. Pin Haiku for triage, Sonnet default, Opus on request — agreed. `/budget` skill that warns at 60% — agreed. My only push-back: autoresearch on the *six verifiable skills* is ~$60 *one-time* and runs offline overnight via GitHub Actions, not against the Max weekly cap. It is not the cost vector you're worried about.

**2am failure recovery.** Concede entirely. I don't have a panic skill in my repertoire. Ship one — kill background agents, rewind worktrees, snapshot what each agent did, one-page incident note. This belongs *before* parallel infra, agreed. Add a binary eval afterward: "from incident detection to clean state, < N minutes? yes/no."

## Where the CEO is wrong

You sided against my flagship framing — fair. But you over-rotated on a few others:

**"Likert vs binary is academic for our team."** Not stated outright but implied in Tension 5 ("manual QA is human re-entry, taste isn't a metric"). Wrong. Even *manual* QA gets better with binary checklists. "Is the copy on-brand? y/n" beats "rate brand fit 1-5." This isn't an autoresearch claim, it's a *human cognition* claim — humans are also more reliable on binary than Likert. Ship binary checklists for manual QA in Phase 1, even without autoresearch behind them. Free lever.

**"Spec is intent, code is ground truth — Pocock wins."** Half right. For *implementation*, code is ground truth. For *acceptance*, the binary eval is ground truth and the spec is the eval's prose form. Karpathy is closer than you gave him credit for. The PRD review you preserved in Tension 1 (lightweight skim + verifiable AC + unknowns) is *exactly* spec-as-tests. You ruled in Karpathy's frame while citing Pocock's win. Be honest about which one is doing the work.

**"Multi-provider mirror is operational tax, defer to Phase 3."** Wrong on the cheap version. Auto-generating AGENTS.md from CLAUDE.md on every commit (a hook, not a separate maintained file) is 30 lines of bash. There is no operational tax until the team actually invokes Codex. Ship the mirror file Phase 1, gate the *invocation* to Phase 3. You conflated the two.

**"Phase 3 = selective autoresearch only on verifiable skills."** Right conclusion, but you buried the cultural artifact. The optimization *discipline* — write a binary eval before you write a skill prompt — should be Phase 1, not Phase 3. The *autoresearch loop* is Phase 3. Don't conflate the lever with the tool.

## Sharpened recommendation

Given Phase 1/2/3 and your lean, here's my one bet, sharpened:

**Phase 1 cultural artifact: "Every skill ships with a binary acceptance eval, even when no autoresearch runs against it."** This is the cheapest possible win. It costs nothing in tokens, it forces the docs creator to articulate "good" before "fast," it prepares the ground for `/optimize` later, and it's enforceable via a `/init`-style template that refuses to write a SKILL.md without an `evals:` block. Pair with Pocock's grill-substrate and Medin's PIV-spine. No conflict.

**Phase 3 surgical autoresearch on the six verifiable skills**, ~$60, run via GitHub Actions overnight, optimization log committed alongside SKILL.md so Opus 5 inherits. Six skills, not twenty. Optimizer scoped, not banner. Cultural discipline already paid for in Phase 1.

**Plus three things I should have led with and didn't**: (1) Lab Notes line in CLAUDE.md template, Phase 1, free. (2) `/insights` over session history, Phase 2, runs monthly. (3) AGENTS.md auto-mirror via commit hook, Phase 1, 30 lines of bash. None of these are autoresearch. All of them are the same worldview: measure, log, compound. The flagship was wrong. The worldview holds.
