# Pocock perspective — position paper

## TL;DR (3 sentences)

specflow's biggest risk is becoming the thing Pocock spent four talks warning against: a spec-driven pipeline that quietly demotes the codebase to a build artifact, hides the agent's mistakes behind a chain of green checkmarks, and trades observability for the comfort of structure. The single highest-leverage move is to harden the human-AI alignment phase ("grill me" before any plan asset is created) and the codebase shape (deep modules, ubiquitous language, fast feedback loops) — because everything downstream is just amplification, and amplification of a bad mental model or a shallow codebase is what produces slop. Pick one bet for the quarter: ship a `/grill` discovery phase that explicitly precedes (and in some cases replaces) Claude Code's plan mode, and refuse to write a PRD asset until alignment is reached.

## What Pocock gets right

The other camps are arguing about the *machinery* of the pipeline — DAGs of nodes (Medin), self-improving auto-tuners (Saraev), spec-as-program (Karpathy). Pocock is arguing about the *substrate*. His sharp claim, the one that should sit at the top of specflow's roadmap: **the only durable source of truth is the running code in a well-shaped codebase, and the only way to keep AI productive against that codebase is to invest in pre-AI software fundamentals — deep modules, seams, ubiquitous language, fast feedback loops — *before* any pipeline runs.**

This is not a stylistic preference. It is a falsifiable claim with mechanism: a shallow-module codebase forces the agent to track dependencies across the whole graph; a deep-module codebase lets it draw a test boundary around one box and work safely inside it (KP-54, KP-55). Bad feedback loops cap AI quality at "coding blind" no matter how clever your prompt is (KP-58). And every "specs-to-code" regeneration drifts the system further from itself (KP-101) — Pocock's framing is that AI hasn't broken software fundamentals, it has *accelerated software entropy* (KP-68). The pipeline camps either don't see this or assume their gates compensate for it. They don't.

The other thing Pocock gets uniquely right: misalignment, not implementation, is the dominant failure mode, and the cure is not a longer PRD template — it is an interrogator that refuses to produce an asset until shared understanding exists (KP-11, KP-104). "Grill me" is a 30-line skill that has survived contact with the real world. The PIV loop and Archon DAGs have not, at the small-team scale specflow targets.

## The 3-5 highest-leverage items for specflow

### 1. Ship `/grill` as a first-class discovery phase that precedes — and refuses — plan mode

**Why it matters.** For our two personas (solo docs creator + AI-novice dev team), the single most expensive class of error is "agent built the wrong thing." Plan mode is "extremely eager to create an asset" before alignment exists (KP-104). A grilling skill that interviews the human one question at a time, with a recommended answer for each, until shared understanding is reached, catches the assumptions humans skip ("should points be retroactive?") and produces Q&A as the durable artefact — high-attention LLM food for every downstream session.

**How to do it.** Adopt Pocock's `grill-me` shape verbatim: one question at a time, recommended answer per question, plain conversation (not `ask_user_question` — JSON wrapping wastes tokens, KP-49), 22–80 questions, output is the conversation history. Make `/grill` block PRD generation until the user explicitly says "we're done." Front the whole specflow pipeline with it. Add a "WHY not just WHAT" rule to the system prompt (KP-12).

**Failure mode prevented.** Premature scaffolding. Pipelines that ship a confident-looking PRD off a vague brief and then spend three Implementation cycles discovering the brief was wrong.

### 2. Make `CONTEXT.md` (ubiquitous language) and a feedback-loop audit the price of admission

**Why it matters.** Every subsequent agent session pays a tax on these two artefacts being weak. A ubiquitous-language file collapses prompts (the "materialization cascade" example, KP-7) and stops the docs↔code drift that is the docs creator's dominant pain. A feedback-loop audit (`npm test`, `npm typecheck`, browser e2e) raises the ceiling on AI quality more than any prompt tweak (KP-58).

**How to do it.** Two skills, ordered. (a) `prime-context` builds/maintains `CONTEXT.md` (verbs, nouns, named flows, aliases-to-avoid) by scanning the repo and extracting domain terms, then proposes additions after every grilling session. (b) `audit-feedback-loops` runs once on repo onboarding: tests run? Time to green? Type-check fast? Browser e2e? Outputs a one-page report and a list of "fix these before agents run wild." This is Pocock's `diagnose` skill instinct: "the loop is the skill, everything else is mechanical."

**Failure mode prevented.** The "amplifier of mess" failure — AI compounds entropy in a bad codebase. specflow shouldn't sell productivity until the codebase can absorb productivity safely.

### 3. Deep modules with simple interfaces — bake Ousterhout into the test/review skills

**Why it matters.** AI is *very good* at producing shallow-module sprawl (KP-54). Once the codebase is a ball of modules with thick interfaces and thin implementations, the agent loses leverage and the human can no longer review by interface. This compounds: bad codebase → bad agent output → worse codebase. specflow's review/test skills should encode the controlled glossary (Module / Interface / Implementation / Depth / Seam / Adapter / Leverage / Locality) and surface shallow-module candidates for human-gated deepening. Explicitly *not* AFK — judgment calls only humans should make (KP-83).

**How to do it.** Adopt Pocock's `improve-codebase-architecture` skill as a periodic ritual ("every couple of days for fast-moving codebases"). Add a review dimension to specflow's test skill: "duplicated parallel implementations across a missing seam" (front-end + back-end "insertion point" with no shared contract) (KP-55, KP-12 of de-slop notes). Output flows into GitHub issues for the AFK loop to pick up.

**Failure mode prevented.** Reviewer burnout and gray-box review collapsing into rubber-stamping. If interfaces are deep and stable, a human can review the *interface* of a change and trust the implementation as a gray box (KP-56) — the discipline that makes the whole pipeline scale.

### 4. Two task types — HITL vs AFK — encoded in the harness, not relied on as norms

**Why it matters.** Pocock's hardest line: "Planning, this alignment phase has to be human in the loop, has to be" (KP-83). Auto/yolo modes "do funny things with these human-in-the-loop style flows" (KP-103). Memory/preferences cannot enforce this — only the harness can. This is the codified version of the user's existing memory-rule "no premature pipeline CTAs."

**How to do it.** Every specflow skill declares HITL or AFK in frontmatter. AFK skills (Ralph-loop implementation, autoresearch optimizer, multi-reviewer fan-out) are runnable unattended; HITL skills (`/grill`, `improve-codebase-architecture`, manual QA, release) refuse to run in auto/yolo mode. Issue labels (`human-in-the-loop`) tell the AFK loop which tickets to skip (KP-15 of feature-build).

**Failure mode prevented.** The entire failure tree of "I went for coffee and Claude shipped to prod." Also Karpathy's "model is brilliant + 10-year-old simultaneously" (KP-63) — the gates exist precisely because capability is jagged.

### 5. Make manual QA the human re-insertion point — and feed it back as new issues

**Why it matters.** Teams that automate idea→PRD→implement→QA "end up with apps that lack taste and are bad" (KP-85). For a docs creator + small dev team, taste is the moat. QA is not the *end* of the loop — it *feeds* the loop, generating new issues (KP-14 of feature-build, in-app feedback button → Haiku-titled GitHub issue → Ralph picks up).

**How to do it.** specflow ships an "iterate-in-QA" skill that captures human QA findings, files them as issues with appropriate labels (bug/enhancement, ready-for-agent vs human-in-the-loop), and feeds them straight into the next AFK pass. Resist the temptation to add an "automated QA" skill that judges its own output.

**Failure mode prevented.** Slop. The thing every other framework eventually produces because it confused "the pipeline ran green" with "the product is good."

## What Pocock rejects

**Reject KP-13 / KP-31 as primary structure: PRD.md + plan.md as explicit human-reviewable gates (Medin's PIV).** Pocock is explicit (KP-16, KP-20): "I don't think there's a lot of value in [polishing the PRD]... the place that you need to be putting the work is in QA." LLMs are good at summarization; review effort belongs *upstream* in grilling Q&A, not on the artefact. Specflow should not build its core ritual around reviewing PRDs and plans. Build it around grilling and QA, with the PRD as a thin byproduct.

**Reject KP-69 / KP-107 as defaults: keeping PRDs in the repo after merge.** Old PRDs poison future sessions because code drifts. Close them as GitHub issues (visible record, not in active context). specflow's default should be archive-not-retain. Medin's "durable memory" framing (KP-26) is fine for *commit-log* memory, where ground truth is the diff, but not for PRDs, where ground truth has moved on.

**Deprioritize KP-29 / KP-30 / KP-31 as identity: full DAG-of-nodes harness with deterministic + LLM + human gates (Archon-shaped).** Useful inspiration, dangerous as the product. Pocock's worry — and ours should be — is that heavyweight harnesses lose observability ("they don't own the stack and they don't have observability over the whole thing," KP-18 of workshop notes / KP-35). specflow should be small, opinionated, inspectable skills that compose; it should NOT become an Archon competitor. The day specflow needs a YAML schema with `depends_on`, `trigger_rule: one_success`, and a workflow router, we have lost the plot.

**Deprioritize KP-21 / KP-28 (autoresearch meta-skill that tunes every other skill).** Autoresearch only works on verifiable domains (KP-27). Tuning a `/grill` skill against an LLM judge is exactly the "council of judges scoring taste" trap that Karpathy himself flagged (KP-22 reward-hacking). The skills that matter most to specflow's users — discovery, design, QA — are precisely the ones autoresearch can't legitimately optimize. Save autoresearch for skills with binary, verifiable acceptance: `format-code`, `pass-types`, `lighthouse-budget`. Don't sell it as a flagship.

**Reject KP-46 elaborate agent org charts (CEO/CMO/CTO patterns).** Saraev is right that these multiply divergence; Pocock's two-pattern view (parent + research sub-agents; planner/implementer/reviewer/merger) is enough. specflow should not ship `team-spawn` presets for "fullstack teams of 7 agents."

## Direct disagreements with other camps

### vs. Medin: "Both PRD.md and plan.md are explicit human-reviewable gates"

Medin: *"It's not good enough to just immediately create stories from that — important for us to review the artifact"* (KP-13, KP-16, paraphrased from PIV loop). The PIV ritual treats the PRD and plan.md as the locus of human judgment; Archon DAGs codify the gates.

**Rebuttal.** This puts review on the wrong side of the cheap-to-fix curve. The cheapest place to catch a misalignment is in the conversation that produced the brief, *before* an asset exists; the most expensive is reading a polished PRD and missing the bad assumption embedded in paragraph two because the prose flows. Pocock's empirical claim (KP-20) is that LLMs *are* good at summarization, and the bug is not in the summary — the bug is in the input the summary is faithful to. Front-load review into grilling Q&A. PRD becomes a hint of direction, not a contract. Medin's gates are a comfort-blanket disguised as discipline; they catch typos, not misalignments.

There's also a scale argument. Medin's PIV loop is calibrated for teams pumping multiple parallel features through Archon. Our two personas (solo docs creator, AI-novice dev team) cannot afford the per-feature ritual cost. Grilling once + skipping PRD review = real velocity. PRD review + plan.md review + PIV gates = the team learns to rubber-stamp.

### vs. Saraev: "Decreasing human involvement is monotonic — auto mode, automated planning, automated QA"

Saraev (KP-111): *"Decreasing human involvement is monotonic... Role becomes CEO of a fleet."* The implication: build for the world where humans only file issues and observe.

**Rebuttal.** This trades observability for theatre. Pocock's response is the de-slop talk's opening line: AI accelerates entropy unless you actively counter it (KP-1 of de-slop / KP-68). A "CEO of a fleet" framing is exactly how you end up with an Archon-style YAML harness whose owner can't debug it when it breaks (KP-18 / KP-35). The autoresearch flagship skill (KP-28) Saraev sells *is* impressive on verifiable domains — Saraev's own 1100ms→67ms website-speed run (KP-23) is real. But specflow's real users are a docs creator and an AI-novice dev team trying to ship a product with taste. They don't need a self-tuning skill catalog; they need a ubiquitous-language file and a grilling skill that doesn't let them lie to themselves.

The decreasing-human-involvement claim also ignores Karpathy's own jagged-models argument (KP-63) — the same talk Saraev cites elsewhere. The model that refactors 100k-line codebases will simultaneously confuse a Stripe customer ID with an email. Manual QA is where that judgment re-enters (KP-85). It is not optional, and it is not automatable.

### vs. Karpathy: "Markdown for agents, HTML for humans" / "spec is the new code"

Karpathy (KP-99, KP-17 paraphrased): *"What is the piece of text I copy-paste to my agent?"* — agents are the primary audience for docs; specs (verifiable acceptance criteria) are where the human's leverage lives.

**Rebuttal.** Pocock and Karpathy actually agree more than they disagree, but the place they diverge is load-bearing for specflow. Karpathy's "spec is the new code" assumes a verifiable domain — the autoresearch repo, training loops, kernels (KP-27). Pocock's "code is the battleground, not the spec" (KP-101) covers everything else. specflow's users build apps with humans in the loop; their domains are *mostly* not verifiable end-to-end. Treating the spec as the source of truth is then a category error. The spec is a hint of direction; the running code is the ground truth; the QA finding is the judgment.

Where Pocock would reinforce Karpathy: yes, write docs *for* agents (markdown not HTML), yes the docs creator's job description shifts toward agent-readable artefacts, yes verifiability is what decides whether to lean on agents or humans. Where he would push back: don't elevate "spec" to a noun the team worships. Delete PRDs after merge (KP-69). The repo is the program. Everything else is scaffolding.

## The one bet for this quarter

**Ship `/grill` as the canonical discovery phase, and make every PRD/plan-generating skill in specflow refuse to run until `/grill` has produced a Q&A artefact.**

One skill. Two-week build. Replaces or precedes Claude Code's plan mode. Outputs the conversation history (not just a polished doc) into `.specflow/discoveries/<slug>.md`. Future PRD/issue/test skills consume that file as their canonical input. No grilling artefact, no downstream pipeline — the hardest gate in the product, and the one with the highest leverage on every north-star goal: more productive (kills wasted implementation cycles on the wrong feature), better product in shorter time (alignment is the cheapest place to catch misalignment), fewer errors (the #1 failure mode is misalignment, not implementation).

Everything else on the roadmap — CONTEXT.md, deep-modules, HITL/AFK tagging, manual-QA-as-feedback — is amplification. Amplification of a misaligned brief is just faster slop. Grill first, or don't bother.
