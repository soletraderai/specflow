# Medin perspective — position paper

## TL;DR (3 sentences)

specflow is already a harness — the team's job this quarter is to admit it, name it, and ship the deterministic primitives (PIV phase boundaries, fresh-context handoffs, path-scoped rules, parallel worktrees, per-node model selection) that turn a pile of skills into a repeatable SDLC. The model isn't the bottleneck and never will be again; the harness is, and a well-built harness with Sonnet beats raw Opus by an order of magnitude in PR acceptance (~6.7% → ~70%). For a docs creator working alone and a dev team new to AI, the *biggest* risks are not under-tooling — they're skipping deterministic gates, conflating planning sessions with implementation sessions, and copy-pasting some heavyweight framework (BMAD, Spec Kit) instead of encoding the team's own process.

## What Medin gets right

The other camps are arguing about parts of the elephant. Pocock is right that fundamentals matter and that bad codebases produce bad AI output. Karpathy is right that verifiability decides what automates. Saraev is right that elaborate org charts are anthropomorphic theatre. All of them are doing **harness engineering** without using the word, and that imprecision hurts their ability to systematise.

Four things the Medin worldview gets right that the others miss or downplay:

**1. The harness IS the product, not the model.** Same model + better harness = ~6.7% → ~70% PR acceptance, and Sonnet-in-Archon often beats Opus-in-raw-Claude-Code (KP-29). This reframes the investment question. Pocock spends time on architectural hygiene; Karpathy spends time on autoresearch loops. Both are harness work, but neither names it as such, so neither builds reusable primitives. specflow has started accidentally — `readiness check`, `QA label`, the release rule are deterministic gates. The question is whether we name and extend the pattern, or stay accidental.

**2. The PIV loop with separate sessions is the correct atomic unit.** Plan in one session (output: `plan.md`), implement in a fresh session that reads only the plan, validate in a third fresh session that reads only the diff (KP-31, KP-33, KP-51). This is not a stylistic preference — it's a *bias-elimination* primitive. A planning session that also implements becomes biased toward defending its own plan; a review session that also wrote the code can't see its own bugs ("asking a kid to grade their own homework"). Pocock's "skip reviewing the PRD" only works *because* his grilling Q&A already burned the planning bias — he just doesn't formalise the boundary. Cole formalises it.

**3. Determinism via scripts, intelligence in the gaps.** The "hybrid secret" (KP-30). The shape of the work — phases, dependencies, gates, artifacts — is owned by humans in YAML/markdown; the LLM only fills cognitive gaps. Don't tell the agent to "remember to run tests"; make the test-run a deterministic node it can't skip. Don't tell the agent to "use the right model"; pin Haiku for routing, Sonnet for research, Opus for hard reasoning. Don't tell the agent to "wait for human approval"; encode `interactive: true` and `loop … until: APPROVED`. Every prompt-based behaviour the agent might forget is a future bug.

**4. Self-evolving rules — every escaped bug is a system bug.** When a defect ships, don't just fix the bug. Open a retrospective session and ask: "what in our rules / skills / workflows / CLAUDE.md should change so this can never recur?" (KP-24). Four artifacts to evolve: commands, on-demand context, global rules, plan/PRD templates. This is the compounding-returns mechanism. Without it, the plugin rots; with it, it gets better with use. Cole's framing beats Karpathy's autoresearch for a small team because it doesn't require an objective metric — just a postmortem habit.

## The 3-5 highest-leverage items for specflow

### 1. Codify PIV as the spine — separate sessions, plan-as-artifact, fresh-context review

Highest-ROI move. Today specflow's skills *could* all be invoked in one session. Force fresh sessions at phase boundaries with `plan.md` (and equivalents for build → test, test → release) as the only context the next phase reads:

- `/plan-feature` writes `.claude/plans/{kebab-name}.md`, no code edits. Borrow the WISC 5-phase template (Feature Understanding → Codebase Intelligence via 4 parallel sub-agents → External Research → Strategic Thinking → Plan Generation).
- `/execute` reads only the plan. "Fresh session, no planning baggage" hard-coded in the skill.
- `/review` runs in a third fresh session, reads only the diff (and optionally the plan for drift check).

This solves three problems at once — context rot, bias in self-review, and the docs-creator-as-PM handoff. The plan.md is the artifact the docs creator can read, edit, and approve; the contract the dev team executes against; the spec the validator checks against. It's also the answer to the "no premature pipeline CTAs" memory: the plan IS the gap, not an unsolicited next step.

### 2. Path-scoped rules with `paths:` frontmatter (3-tier context)

Today specflow's rules likely live in CLAUDE.md and command files, all loaded always. Migrate to WISC's 3-tier structure (KP-4):

- **Tier 1** — `CLAUDE.md`, capped at 500–700 lines. Calibration test: "if removing a line wouldn't cause the AI to make mistakes, cut it."
- **Tier 2** — `.claude/rules/*.md` with YAML `paths:` frontmatter, auto-loaded when matching files are touched (e.g., `releases.md` scoped to `plugin.json`/`marketplace.json`; `skills-authoring.md` scoped to `**/SKILL.md`).
- **Tier 3** — `.claude/docs/*.md` with `> Purpose / When to use / Size` headers; never auto-loaded; a scout sub-agent decides whether to load.

Cuts always-on tokens, makes docs-creator rules invisible to dev sessions and vice versa, gives the harness room to grow without bloating CLAUDE.md.

### 3. Per-node model selection wired into every skill

Stop running Opus by default. Pin model choice at the skill level (KP-34):

- Classification, triage, scout → Haiku.
- Codebase scan, web research, doc-impact review, simple validation → Sonnet.
- Architecture, deep planning, hard implementation, adversarial review → Opus.

Cole hit only 37% of his 5-hour limit running 4 parallel workflows. For a docs creator on a $20–$100 plan and a small dev team on Max, this is the difference between "ran out Tuesday" and "ran productively all week." Also a hedge against rate-limit tightening (KP-95).

### 4. Parallel agents on git worktrees

The foundational unblocker for >1 agent (KP-70, KP-72). Ship a `w` script that, in one command:

- creates a sibling worktree with a new branch
- assigns a deterministic port via `MD5(cwd) % 100 + 4000` (~15 lines, no registry)
- pre-installs deps and provisions an isolated DB copy if the project has one
- launches a Claude session inside it

Parallelism is how a docs creator alone matches a team's throughput. Cole demoed 4–6 parallel issue fixes in one prompt; that's the difference between "1 feature this week" and "6." The DB-branching part is a placeholder for now (specflow has no DB) but the *pattern* — one command provisions an isolated env, one tears it down — is the right shape for future per-task experimentation.

### 5. System Evolution outer loop — `/postmortem` and enriched commits

Every escaped bug becomes a CLAUDE.md / rules / skill update (KP-24):

- A `/postmortem` skill that, given a bug or merged-then-fixed PR, opens a retrospective session and proposes specific edits to commands, rules, plan templates, and CLAUDE.md.
- An enriched `/commit` (borrowed from WISC) with a `Context:` section that logs `.claude/` changes alongside code changes (KP-38). Git log becomes long-term agent memory.

This is what makes specflow appreciate rather than depreciate. Without it, the plugin is a snapshot of someone's preferences in late 2026 that gets stale when Claude Code 5.0 ships.

## What Medin rejects

**1. The "1M-token context window" hype.** "Just because you can fit a million tokens does not mean that you should — they get overwhelmed just like people do" (KP-2). The 1M window is a trap that lulls AI-novice teams into sloppy loading, producing context rot blamed on the model. The discipline is *less* context, more on-demand loading.

**2. Heavyweight off-the-shelf frameworks (BMAD, Spec Kit, GSD, Cloudflow, Stripe Minions).** Cole calls these "respected but very overengineered… they require you to pretty much change how you work fundamentally" (KP-35). Useful as inspiration, not adoption targets. specflow should encode the team's existing SDLC and stay small enough to inspect end-to-end. Specifically reject: deeply nested agent org charts, 50-step bundled workflows the team won't customise, anything that requires a separate web UI.

**3. Pure autonomous loops without checkpoints (Ralph-Loop / Dark Factory).** "If the coding agent makes a mistake in the first iteration… that issue can propagate and blow up from the rest of the loop because it builds on top of a code base that's already wrong" (KP-32). Cole's *own* Dark Factory experiment is explicitly framed as an experiment, not a recommendation (KP-113). For a docs creator alone and a dev team new to AI, hybrid harnesses with explicit `interactive: true` gates. AFK is fine for bounded loops with read-only eval surfaces; not for production code.

**4. The framing that PRDs poison the repo.** Cole treats artifacts — plans, handoffs, enriched commits — as *durable agent memory* (KP-26, KP-38). Closing PRDs as GitHub issues after merge throws away exactly the audit trail the dev team needs. The drift problem is real, but the answer is enriched commits and dated archive folders, not deletion.

**5. Avoiding `ask_user_question` for token cost.** For a docs creator unfamiliar with code-level details, multiple-choice clarification is a clarity win that dwarfs the JSON-wrapping overhead. The token-cost objection optimises a rounding error.

## Direct disagreements with other camps

### vs. Pocock

Pocock: *"Specs-to-code is vibe coding by another name. I tried this. I really tried it and it sucks. … The place that you need to be putting the work is in QA"* (KP-101, KP-16, KP-20).

**Rebuttal**: Pocock is right that *spec-only-edits-then-regenerate* is broken — but he's overgeneralising from a strawman. The PIV loop isn't "edit specs and regenerate." It's "plan in writing, hand the plan to a fresh implementer, validate in a third fresh session." The plan is a contract, not a regeneration trigger. When Pocock says "skip reviewing the PRD, trust LLM summarization," he's relying entirely on his grilling step to have caught misalignment — which works *for him* because he's a senior engineer who knows what to grill for. For a docs creator alone, the PRD review IS the safety net; for a dev team new to AI, the plan.md review is the moment the senior dev catches "you're storing user IDs but joining on email." Skipping artifact review is feasible only when the reviewer has internalised the failure modes — not the audience specflow serves.

Pocock's other big claim: *"The rate of feedback is your speed limit"* (KP-58). True. But fast feedback without phase boundaries gives you fast feedback inside a biased session. Cole's answer: fast feedback **per fresh session**. Pocock solves the speed-of-feedback problem; he doesn't solve the bias-of-feedback problem.

### vs. Saraev

Saraev: *"Skip agent org charts… anthropomorphism that multiplies divergence. Stick to: (a) parent + Sonnet researchers + Opus QA, or (b) developer + QA loop"* (KP-46).

**Rebuttal**: Saraev is right about *role-based* org charts (CEO/CMO/CTO theatre). He's wrong if he generalises to *function-based* fan-outs. The Archon multi-reviewer pattern — code reviewer, error-handling, test-coverage, comment-quality, docs-impact, all running in parallel, then a synthesizer (KP-53) — is a strictly better quality bar at roughly the same wall-clock time. Those agents are specialists by *dimension*, not by anthropomorphic *role*. "Code-simplifier subagent" is not "CMO subagent." Saraev is rejecting the noun-class he was burned by. Cole's hybrid: by-function good, by-role bad.

Where Saraev is dead right and Cole would steal directly: CLAUDE.md as a 4-in-1 asset, and the meta-skill that autoresearches every other skill (KP-28). The autoresearch meta-skill belongs in specflow's roadmap as a P2 once the harness primitives are in.

### vs. Karpathy

Karpathy: *"Markdown for agents, HTML for humans. … Detailed specs are the new code"* (KP-99, KP-17). On autonomy: *"NEVER STOP, don't ask should I continue?"* (KP-50).

**Rebuttal — on specs**: Karpathy is correct that verifiability decides what automates and that the unit of contribution from an expert is "a few bits" (KP-87). But "specs are the new code" is a reach. Specs *gate* code; they don't *replace* it. Cole's PRD/plan.md gates are Karpathy's "few bits" made operational. The disagreement is rhetorical: Karpathy predicts a future state; Cole ships tomorrow.

**Rebuttal — on NEVER-STOP**: Karpathy's autoresearch loop is bounded by a fixed metric and read-only files. It is *safe* because the eval surface can't be corrupted. A general coding pipeline has no fixed metric (no `val_bpb`) and the eval surface IS the codebase being mutated. Applying NEVER-STOP to a coding pipeline gets you Ralph-Loop compounding errors. Conflating the regimes produces specflow users who YOLO production codebases and ship security holes. Cole's framing is more rigorous: human gates *iff* the loop lacks a fixed metric and read-only eval surface.

Karpathy's most useful contribution to specflow is the binary-yes/no eval discipline (KP-22) and the simplicity criterion ("0.001 from deleting code? Definitely keep" — KP-62). Both belong in specflow's review and simplify skills. But autoresearch only applies where verifiability lives — most of specflow's pipeline (spec interpretation, design tradeoffs, UX) is on the wrong side of that line.

## The one bet for this quarter

**Ship the PIV loop as specflow's spine, with fresh-context boundaries enforced as deterministic primitives.**

Concretely, this quarter:

1. `/plan-feature` writes `.claude/plans/{name}.md`. No code edits in this skill. WISC 5-phase template, 4 parallel sub-agents in the codebase-intelligence step.
2. `/execute` reads only the plan. Hard-coded instruction: fresh session, no planning baggage. Uses Opus.
3. `/review` runs in a third fresh session, reads only the diff plus optionally the plan for drift check. Uses Sonnet by default, with `/codex:adversarial-review` available as an opt-in second pass for high-stakes PRs.
4. `/postmortem` closes the loop: when a bug escapes, propose specific edits to rules/skills/templates.
5. Migrate path-scoped rules to `.claude/rules/*.md` with `paths:` frontmatter as a precondition for #1–4 working at scale.

Everything else — worktree parallelism, DB branching, autoresearch meta-skills, multi-reviewer fan-out, MCP integrations — is downstream of this. Without phase boundaries enforced as primitives, those features amplify a flawed substrate. With them, every later feature compounds.

The bet is that disciplined fresh-context handoffs, encoded once at the harness layer, will produce a step-change in PR acceptance and error rate for both the docs creator and the dev team — without anyone having to learn a new framework, change how they work, or wait on a model upgrade. The harness is the lever. This quarter we pull it.
