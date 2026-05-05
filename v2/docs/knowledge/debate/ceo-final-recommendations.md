# CEO Final Recommendations — specflow roadmap, post-debate

This is the deliverable that closes the debate. It tells the team what to build, when, and why. It overrides Round 1 where the rebuttals moved my view, and it overrides the advisors' position papers everywhere those papers gave bigger answers than our two personas (solo docs creator + AI-novice dev team) can absorb.

Audience: the small team that executes from this doc. Optimised for the three north-star goals — (1) more productive, (2) better product in shorter time, (3) fewer errors.

---

## 1. What changed because of the debate

### Concessions each advisor made

- **Pocock** — conceded "skip the PRD review" generalises from his own seniority. PRD review stays, scoped as a 5-minute *misalignment scan* (3 questions, hard time budget), not a polish ritual. Also conceded he had no answer for the trust calibration curve, the human-to-human handoff gap, the cost-on-Max budget question, or 2am failure recovery — all of which he now agrees should ship in Phase 1.
- **Medin** — conceded the YAML DAG (`depends_on:` / `trigger_rule:` / `loop … until:`) is exactly the BMAD-shape failure he himself warned against. PIV survives as *convention enforced inside skills*, not as a workflow DSL. He also conceded `team-handoff` belongs in Phase 1 ahead of his own five recommendations, that the 37% of-Max-limit demo doesn't generalise to code workflows, and that artifacts need a `/prune` ritual to avoid accumulation.
- **Saraev** — conceded `/optimize` is not a flagship. It ships in Phase 3 against a curated subset of six verifiable skills (~$60, not $200). Also conceded cross-provider adversarial review is operational tax for a 1-person ops team, that the trust calibration curve is a real blind spot autoresearch doesn't help with, and that `panic` belongs before parallel infra.
- **Karpathy** — conceded "code is disposable" is wrong as stated for a product team. Spec is *intent of record*; code is the *running rendering*; both are authored, neither is throwaway. Also conceded NEVER-STOP only holds *inside a fence* (locked eval surface, fixed scalar, branch-per-run, panic kill switch) — without the fence the failure mode eats us.

### Pushbacks that landed (where I updated my Round 1 view)

- **Pocock's trust-ladder primitives in Phase 1.** Round 1 had `confidence-check`, `panic`, and `tour` in Phase 3 alongside `/optimize`. Pocock is right that you ship safety primitives *before* parallelism, not after. Moved to Phase 1.
- **Karpathy's `simplify` autoresearch as a Phase 1 discipline-installer.** Round 1 had all autoresearch in Phase 3. Karpathy convinced me the *forcing function* (every skill ships with a binary acceptance eval; one bounded loop teaches the team the discipline) is a Phase 1 cultural artifact. The loop running at scale is still Phase 3, but installing the discipline is Phase 1.
- **Saraev's binary acceptance eval block in every SKILL.md.** Free lever. Costs nothing in tokens. Forces the docs creator to articulate "good" before "fast." Phase 1 cultural artifact.
- **Saraev's AGENTS.md auto-mirror via commit hook.** Round 1 deferred multi-provider posture to Phase 3. Saraev correctly separated the *file* (cheap, 30 lines of bash, ship Phase 1) from the *invocation* of Codex (operational tax, defer to Phase 3).
- **Medin's PIV-as-convention, not DAG.** Round 1 already leaned this way; the rebuttal made it crisp. Phase boundaries are enforced by what each skill chooses to read, using frontmatter Claude Code already parses. No new schema.
- **Pocock's reframing of HITL/AFK.** Not "earn the right to remove a gate" (graduation) but "AFK iff (a) binary verifiable judge AND (b) reversible blast radius." A property of the skill, not a maturity of the team.

### Items that emerged that nobody had on the table at the start

- **`team-handoff` skill at the human boundary** (with stale-tag on artifacts >14 days). Surfaces the docs-creator-as-single-point-of-failure blind spot. All four advisors conceded this gap.
- **Trust ladder profile** — `getting-started` mode with `interactive: true` everywhere, unlocked per-skill via an explicit ritual, not a calendar tick. Saraev's "ladder is a metric" framing gives it instrumentation.
- **`panic` skill** — kill background agents, rewind worktrees to last green, snapshot what each agent did, one-page incident note. Ships *before* any parallel infra.
- **`confidence-check` ritual** — "before merging, name one thing the AI did that you don't fully understand." Karpathy's jagged-intelligence point made operational.
- **`/budget` skill** — warns at 60% weekly Max usage; pins Haiku for triage / Sonnet default / Opus on demand.
- **AGENTS.md auto-mirror** — Phase 1, 30-line commit hook. Cross-provider invocation deferred.
- **`tour` skill** — 10-minute repo walkthrough for a new dev with no senior present.
- **`/prune` quarterly ritual** — counters Medin's artifact-accumulation tendency.
- **Misalignment-scan PRD checklist** — 3 questions, 5-minute time budget enforced by the skill itself. Sharpened from Pocock's rebuttal.
- **Binary-eval block in SKILL.md frontmatter** as a Phase 1 *cultural* requirement, even where no autoresearch runs. Refuses to ship a SKILL.md without it.

---

## 2. Adjudicated decisions — the 10 headline tensions

| # | Tension | Resolution | Altered by rebuttals? |
|---|---|---|---|
| 1 | PRD review or skip? | **Keep, but as a 3-question, 5-minute misalignment scan, not a polish ritual.** | Yes — Pocock conceded; rebuttal sharpened the scope. |
| 2 | Keep PRDs in repo or delete? | **Archive, don't delete; never auto-load; scout sub-agent decides relevance.** | No (Round 1 stance held). |
| 3 | Specs or code as ground truth? | **Code is ground truth for what shipped; spec is intent of record for what we agreed to ship; drift-check is the bridge.** | Yes — Karpathy dropped "code disposable"; Medin sharpened "intent of record." |
| 4 | NEVER-STOP or human gates? | **Default to gates. AFK eligibility = binary verifiable judge AND reversible blast radius. Property of the skill, not a team graduation.** | Yes — Pocock reframed from "earn it" to a per-skill property; Karpathy added "every gate ships with the scalar that would retire it" tag. |
| 5 | QA fully automated or human re-entry? | **Human owns taste/UX/brand QA. Machine owns verifiable QA (security, payments, a11y, schema, regressions). Multi-reviewer-by-dimension upstream of the human gate.** | Yes — Karpathy correctly called overreach; verifiable QA splits out. |
| 6 | `ask_user_question` or plain prose? | **Plain prose for grilling (22–80 questions); structured tool for short mid-execution clarifications.** | No — both camps accepted the split. |
| 7 | Agent org charts or two patterns? | **By-function fan-out yes (with a single human integrator). By-role fan-out no.** | Yes — Pocock added the integrator requirement to defeat multiplicative reliability decay. |
| 8 | Heavyweight framework, thin harness, or extend Archon? | **specflow IS the harness. Steal patterns (PIV, `paths:`, postmortem) without the YAML DSL.** | Yes — Medin conceded the YAML directly. |
| 9 | Replace or wrap Claude Code's plan mode? | **Wrap. `/grill` blocks plan mode until `.specflow/discoveries/<slug>.md` exists.** | No (Round 1 stance held). |
| 10 | Ship parallel infra or stay lean? | **Stay lean Phase 1–2. Phase 3 worktree script gated on the bottleneck-on-review trigger AND `panic` shipping first.** | Yes — `panic` now precedes worktrees as a hard prereq. |

---

## 3. Roadmap recommendations by phase

### Phase 1 — Foundation (substrate + safety)

The team's first quarter. Light scaffolding. Pocock-substrate, with one Medin convention (paths-frontmatter) and the cultural artifacts from Saraev (binary evals) and Karpathy (one bounded autoresearch loop) bolted on. Plus the trust-ladder primitives Round 1 wrongly deferred.

1. **`/grill` discovery skill (canonical entry point).** Wraps and blocks Claude Code's plan mode until `.specflow/discoveries/<slug>.md` exists with verbatim Q&A. One question at a time, recommended answer per question, plain prose (no `ask_user_question`). *Why:* misalignment is the #1 failure mode, and grilling catches 80% of catchable misalignment for ~$0 in tokens. Direct hit on goals 2 and 3. *Source:* Pocock (KP-11, KP-104). *Audience:* the docs creator's day-1 mental model.

2. **PRD misalignment-scan + `team-handoff` (one connected ritual).** `to-prd` runs against the discovery file, appends the Q&A verbatim, and ships with a 3-question checklist (matches grilling? AC verifiable? assumption we never discussed?). Hard 5-minute time budget enforced by the skill. `team-handoff` runs at every human-to-human boundary, snapshots in-flight discoveries/PRDs/issues, tags anything >14 days as stale and refuses to load it without re-grilling. *Why:* the rebuttals' biggest reframe — PRD review is a safety net, not a hammock; and the docs creator is a single point of failure that needs an artifact-level escape hatch. *Source:* Pocock rebuttal (concession on PRD); CEO Round 1 blind spot (`team-handoff`); Medin agreed in rebuttal. *Audience:* both personas.

3. **HITL/AFK frontmatter on every skill + `getting-started` profile + `panic` + `confidence-check`.** Every skill declares HITL or AFK in frontmatter; AFK requires a binary judge AND reversible blast radius (Pocock's reframe). `getting-started` profile defaults `interactive: true` everywhere with an explicit per-skill upgrade ritual. `panic` kills background agents, rewinds worktrees to last green, snapshots what each agent did, writes a one-page incident note. `confidence-check` runs pre-merge: "name one thing the AI did you don't fully understand." *Why:* trust calibration is a curve, not a slogan, and 2am recovery must precede parallelism. Direct hit on goal 3. *Source:* CEO Round 1 blind spots; all four advisors conceded in rebuttals.

4. **`feedback-loop-audit` + `CONTEXT.md` glossary skill.** One-shot audit of repo onboarding: tests run? time to green? typecheck fast? e2e exists? Output is a one-page report and a "fix these before agents run wild" list. `CONTEXT.md` is the ubiquitous-language file (verbs, nouns, named flows, aliases-to-avoid) seeded by repo scan, extended by `/grill`. *Why:* the rate of feedback is the speed limit on AI quality (KP-58); the glossary collapses prompts and stops docs↔code drift. Direct hit on all three goals. *Source:* Pocock (KP-7, KP-58). *Audience:* AI-novice dev team.

5. **Path-scoped rules via `paths:` frontmatter; CLAUDE.md capped at 500–700 lines; AGENTS.md auto-mirror commit hook; `/budget` skill.** Migrate rules to `.claude/rules/*.md` with `paths:` frontmatter (Tier 2); `.claude/docs/*.md` with `> Purpose / When to use / Size` headers (Tier 3, scout-loaded only). Cap CLAUDE.md with the calibration test ("if removing a line wouldn't cause the AI to make mistakes, cut it"). Auto-mirror CLAUDE.md → AGENTS.md on commit (30 lines of bash, no edit-by-hand). `/budget` warns at 60% weekly Max usage and enforces Haiku-triage / Sonnet-default / Opus-on-demand. *Why:* harness hygiene using existing primitives; cheap insurance against outages; cost reality on Max. *Source:* Medin (KP-4); Saraev rebuttal (auto-mirror); CEO Round 1 (`/budget`).

6. **Binary-eval block required in every SKILL.md (cultural artifact).** `/init` template refuses to write a SKILL.md without an `evals:` block — even when no autoresearch runs against it. Forces the team to articulate "good" before "fast." *Why:* free lever; prepares ground for Phase 3 `/optimize`; teaches discipline early when it's cheap. *Source:* Saraev rebuttal (sharpened). *Audience:* both personas.

7. **One bounded autoresearch loop on `simplify` (Karpathy's discipline-installer).** Scope: lines deleted + tests pass + lints clean + types clean. Read-only eval surface. Branch-per-run. Hard $20/week budget. Merge only after human review. The deliverable is the *template* (`eval.md` per skill, CAN/CANNOT lists, locked surfaces, scalar) — every future skill that gains a verifiable surface drops into this template. *Why:* installs the verifiable-AC discipline now, where it's cheap, instead of retrofitting after Phase 3 ships unverifiable surface. *Source:* Karpathy rebuttal. *Audience:* the dev team learns the loop on a small bounded skill.

**Items NOT making Phase 1 cut:**
- Full `/optimize` catalog scan — too broad; only the 6 verifiable skills earn it, and only in Phase 3.
- Cross-provider adversarial review (Codex invocation) — Phase 3, gated on team size ≥3 and a real outage.
- Worktree script + DB branching + parallel infra — Phase 3, gated on reviewer pile-up trigger and `panic` having shipped.
- YAML DAG harness primitives (`depends_on:`, `trigger_rule:`) — never. Conceded by Medin.
- `improve-codebase-architecture` periodic review — defer to Phase 2; needs PIV spine first.
- `tour` skill — Phase 2 (good idea but not blocking; needs the substrate first).
- Persona/soul docs for Claude — Phase 2/3, rounding error vs verifiability and parallelism.

### Phase 2 — Development (PIV spine, no DAG)

Once substrate works, install the discipline of fresh-context handoffs. PIV as convention enforced inside skills, not as a YAML schema. This phase teaches the team to operate without a senior present.

1. **`/plan-feature` writes `.specflow/plans/<slug>.md` (no code edits) — WISC 5-phase template.** Plan is the contract. Borrow the 4-parallel-sub-agent codebase-intelligence step. *Why:* separates planning bias from implementation. *Source:* Medin (KP-31). *Audience:* docs creator (the artifact they hand to devs).

2. **`/execute` opens fresh session, reads only `plan.md`. `/review` opens third fresh session, reads diff + plan for drift check.** Hard-coded skill instructions, not YAML. Per-skill model pinning in frontmatter (Haiku/Sonnet/Opus). *Why:* fresh-context per phase eliminates self-grading bias; per-node model selection is the real Max-cost lever. *Source:* Medin (KP-31, KP-34, KP-51). *Audience:* AI-novice devs.

3. **`/postmortem` skill — every escaped bug becomes a rule edit.** Proposes specific edits to commands, on-demand context, global rules, and plan/PRD templates. *Why:* compounding-returns mechanism; without it the plugin rots. *Source:* Medin (KP-24); Pocock approves as "QA findings feed new issues" loop.

4. **`specflow:develop` orchestration (PRD-original Phase 2 centrepiece) — composed onto the PIV skills.** Picks the right agents/team for the task, tracks progress, loops in Devil's Advocate at decision points, runs Verifier at completion. Function-based fan-out only (multi-reviewer-by-dimension), with a single human integrator at the end (Pocock's rebuttal addition). *Why:* original PRD goal; now safe because PIV + integrator exist. *Source:* PRD appendix L. *Audience:* dev team.

5. **`tour` skill + agent indexing + agent registry (PRD Phase 2 items).** `tour` produces a 10-minute repo walkthrough specific to *this* repo's CLAUDE.md, glossary, active skills, and "things you cannot touch yet." Agent indexing scans installed marketplaces; agent registry pins per-repo at `admin/agents/specialised/` with snapshots. *Why:* day-1 onboarding with no senior present (Round 1 blind spot); original PRD scope. *Source:* CEO Round 1 + PRD appendix K. *Audience:* new devs joining mid-quarter.

**Items NOT making Phase 2 cut:**
- `/optimize` autoresearch catalog — Phase 3.
- Worktree parallel infra — Phase 3.
- Multi-provider Codex invocation — Phase 3.
- Self-learning memory loop / decision-log automation — Phase 3 (PRD-original Phase 3).

### Phase 3 — Memory, self-learning, earned parallelism

Selective Karpathy + Saraev + the original PRD memory loop. Earn parallelism only after `panic` ships and reviewer pile-up is observed.

1. **Self-learning memory loop (PRD-original Phase 3).** `decision-log.md` + `task-history.json` + `/specflow:complete` retro skill + `/specflow:decision` + similarity search + profile consumption + `docs/` folder integration. *Why:* original PRD goal; now compounds because PIV + binary-evals + postmortem are already producing the inputs. *Source:* PRD appendix I. *Audience:* both personas.

2. **`/optimize` autoresearch on the 6 verifiable skills (~$60 total, run via GitHub Actions overnight).** `release-version-check`, `simplify` (already running from Phase 1), `format`, `tdd-cadence`, `init`, `feedback-loop-audit`. Optimization log committed alongside SKILL.md so future model upgrades inherit. Read-only eval surfaces, fixed scalars, branch-per-run, panic kill switch. *Why:* compounding asset that survives model upgrades. *Source:* Saraev rebuttal (sharpened). *Audience:* docs creator (set-and-forget overnight).

3. **Worktree `w` script + DB-branching pattern, gated on reviewer pile-up trigger.** ~15-line script: sibling worktree, deterministic port (MD5(cwd) % 100 + 4000), pre-installed deps, isolated DB copy. Ships *only after* `panic` has been used at least once and reviewer pile-up is observed (KP-80). *Why:* parallel infra is a tax until you're the bottleneck. *Source:* Medin (KP-70, KP-71, KP-72). *Audience:* dev team once throughput exceeds one human reviewer.

4. **Cross-provider adversarial review (Codex invocation), gated on team size ≥3 AND a real Anthropic outage having cost the team >4 hours.** Different models have different blind spots; findings Codex catches that Claude missed get promoted to CLAUDE.md as new rules (self-healing). *Why:* the operational tax is real, but earned by evidence. *Source:* Saraev rebuttal (revised). *Audience:* full dev team only.

5. **`/insights` (monthly) + `/prune` (quarterly).** `/insights` runs sub-agents over the prior month's session history and proposes CLAUDE.md / rules / template edits. `/prune` deletes skills nobody used, archives PRDs/plans, retires interim gates whose scalar has earned them out. *Why:* counters artifact accumulation; closes the self-healing loop. *Source:* Saraev (KP-24); Medin rebuttal accepted `/prune`. *Audience:* docs creator (plugin owner).

**Items NOT making Phase 3 cut:**
- Autoresearch on unverifiable skills (`grill`, `plan-feature`, `to-prd`, design-judgment, taste-QA) — never. Reward-hacking trap.
- YAML DAG harness primitives — never.
- Agent org charts (CEO/CMO/CTO patterns) — never.
- Karpathy's "code is disposable" / regenerate-from-spec — never.
- Dark Factory (humans only file issues) — never for our team's product surface.

---

## 4. Blind spots — staffed vs still open

| Blind spot (CEO Round 1) | Now staffed by | Status |
|---|---|---|
| Single-point-of-failure docs creator | `team-handoff` skill (Phase 1 #2) + grilling Q&A as handoff artifact | **Staffed** |
| Trust calibration curve | `getting-started` profile + per-skill upgrade ritual + binary instrumentation (Phase 1 #3) | **Staffed** |
| People-to-people handoffs | `team-handoff` + 14-day stale tag that gates the agent (Phase 1 #2) | **Staffed** |
| Cost reality on Max subscription | `/budget` skill + Haiku-triage/Sonnet-default/Opus-on-demand pinning (Phase 1 #5) | **Staffed** |
| 2am failure recovery | `panic` skill (Phase 1 #3), shipped before any parallel infra | **Staffed** |
| Onboarding day-1 with no senior | `tour` skill (Phase 2 #5); minimal `feedback-loop-audit` + glossary (Phase 1 #4) bridge until then | **Partially staffed** — `tour` slips to Phase 2; the substrate gives a new dev *something* on day-1 of Phase 1. Owner: docs creator authors `tour` template. |
| When to override the AI | `confidence-check` ritual (Phase 1 #3) + "look harder" reflex baked into PRD misalignment-scan checklist | **Staffed** |
| Knowledge transfer | `team-handoff` + grilling Q&A (Phase 1 #2) + `decision-log.md` (Phase 3 #1) + `tour` (Phase 2 #5) | **Staffed across phases** |
| Non-engineers in the loop (designers, PMs) | Voice/dictation entry to `/grill`; `/idea` prose-routing entry point | **Still open** — proposed Phase 2 add. Owner: docs creator. |
| Plugin owner is the dogfood team | `/prune` quarterly ritual (Phase 3 #5) | **Partially staffed** — `/prune` lands Phase 3; until then, manual quarterly review by the plugin owner. Owner: docs creator. |

---

## 5. The four watchwords

The whole debate, distilled into four directives the team can hold in their heads.

1. **Code is ground truth. Spec is intent of record.** Both are authored, neither is throwaway. The drift-check is the bridge between them.

2. **Grill the human. Scan the artifact. Ship the loop.** Misalignment lives upstream; review the spec for misalignment, not for prose. The product is the running rendering, not the document.

3. **Default to gates. Earn AFK with a fence.** Autonomy requires a binary judge AND a reversible blast radius — both, always. Without the fence, the loop eats us.

4. **Survive Tuesday before optimising the quarter.** Single-point-of-failure handoffs, panic recovery, and budget hygiene ship before parallelism, autoresearch, and multi-provider posture. Compounding leverage is worthless if the team can't get through a normal week.

---

## 6. Risks and what would change the plan

### Top 3 risks to this plan

1. **The team rubber-stamps the misalignment-scan checklist.** Three checkboxes in 5 minutes is exactly what habitual rubber-stamping looks like. Mitigation: the skill samples and audits — every Nth PRD gets a *re-grilling* prompt that asks the docs creator to defend one assumption against the verbatim Q&A. If they can't, the PRD is sent back. Without this, Pocock's concession ("keep the net but don't make it a hammock") fails on contact with reality.

2. **Phase 1 sprawls and the team never gets to Phase 2.** Seven Phase 1 items is more than my Round 1 lean. The reason is that the rebuttals correctly moved trust-ladder primitives forward; the cost is a wider Phase 1 surface. Mitigation: explicit DONE criteria per item (binary, of course), and a strict rule that no Phase 2 work begins until Phase 1 has been used in anger on at least three real features. If Phase 1 takes >10 weeks, kill the lowest-confidence item (probably the `simplify` autoresearch loop — it's a discipline-installer, not a deliverable, and the discipline can be installed by the binary-eval block alone).

3. **The `/grill` ritual gets bypassed under deadline pressure.** When the dev team is on fire, the cheapest thing to skip is the 30-minute grilling. Once skipped twice, it stops being canon. Mitigation: `/grill` is a hard gate (no `.specflow/discoveries/<slug>.md` → `to-prd` and plan mode refuse to run). The harness enforces the discipline; humans cannot enforce it on themselves under pressure (memory rule: no premature pipeline CTAs is the same shape — encode in the harness, not in norms).

### Top 3 leading indicators that should trigger a re-plan

1. **>50% of specflow skills develop verifiable acceptance surfaces by end of Phase 2.** Round 1 estimate was ~15-20% verifiable. If the binary-eval discipline (Phase 1 #6) drives this past 50%, Saraev's worldview moves up — `/optimize` becomes a Phase 2 deliverable, not a Phase 3 surgical tool, and the cost-per-skill math changes the roadmap.

2. **The dev team grows past 3 people.** Cross-provider adversarial review, full worktree parallel infra, and `team-spawn` agent teams all become earned. The Phase 3 gate on team size flips. Re-plan accordingly.

3. **A real Anthropic outage (or rate-limit collapse) costs the team >4 hours of stoppage in a single incident.** Multi-provider posture moves from Phase 3 opt-in to Phase 2 mandatory. The auto-mirrored AGENTS.md is already in place from Phase 1; the change is upgrading from *file existence* to *active Codex invocation* on a specific skill list.

---

## Update the PRD

The decisions above require edits to `/Users/marepomana/Web/specflow/PRD.md`. Pseudo-diff form, by section:

```diff
## Approach: three phases
- Phase 1 lays the foundation — folders, docs, examples, the upgrade tool, the design tool, and the environment inventory.
+ Phase 1 lays the substrate AND the safety primitives — folders, docs, examples, upgrade tool, design tool, environment inventory,
+ AND the seven debate-installed items: /grill, PRD misalignment-scan + team-handoff, HITL/AFK frontmatter + getting-started + panic + confidence-check,
+ feedback-loop-audit + CONTEXT.md glossary, paths-frontmatter rules + CLAUDE.md cap + AGENTS.md auto-mirror + /budget,
+ binary-eval block in every SKILL.md, one bounded autoresearch loop on simplify.

### Phase 1 — Foundation (lightweight + critical scaffolding)
+ **Debate additions (this section EXPANDED):**
+ - /grill skill blocks Claude Code's plan mode until .specflow/discoveries/<slug>.md exists with verbatim Q&A.
+ - to-prd appends grilling Q&A verbatim and ships with a 3-question misalignment-scan checklist (5-minute hard time budget).
+ - team-handoff skill at human-to-human boundaries; tags artifacts >14 days as stale and gates the agent on stale reads.
+ - HITL/AFK frontmatter on every skill (AFK iff binary judge AND reversible blast radius).
+ - getting-started profile with `interactive: true` everywhere; per-skill upgrade ritual.
+ - panic skill: kill background agents, rewind worktrees to last green, snapshot, one-page incident note. Ships BEFORE any parallel infra.
+ - confidence-check pre-merge ritual.
+ - feedback-loop-audit + CONTEXT.md glossary skill.
+ - paths-frontmatter rules (.claude/rules/*.md), CLAUDE.md capped 500-700 lines, AGENTS.md auto-mirror commit hook.
+ - /budget skill (warn at 60% weekly Max; pin Haiku-triage/Sonnet-default/Opus-on-demand).
+ - Binary-eval block REQUIRED in every SKILL.md (cultural gate; refuses to write SKILL.md without it).
+ - One bounded autoresearch loop on simplify as discipline-installer (not flagship): lines + tests + lints + types, read-only surface, $20/week budget, branch-per-run, merge only after human review.

### Phase 2 — Development (the develop skill + agent orchestration)
+ **Debate additions:**
+ - PIV spine as CONVENTION (not YAML DAG): /plan-feature writes .specflow/plans/<slug>.md, /execute reads only that, /review runs in third fresh session.
+ - Per-skill model pinning in frontmatter.
+ - /postmortem skill turns escaped bugs into rule edits.
+ - tour skill (10-min repo walkthrough for new devs with no senior present).
+ - Multi-reviewer-by-dimension fan-out REQUIRES a single human integrator (no role-based fan-out).

### Phase 3 — Memory and self-learning
+ **Debate additions:**
+ - /optimize autoresearch on the SIX verifiable skills only (release-version-check, simplify, format, tdd-cadence, init, feedback-loop-audit).
+   Run via GitHub Actions overnight. Optimization log committed alongside SKILL.md.
+ - Worktree w-script GATED on reviewer pile-up trigger AND panic having shipped.
+ - Cross-provider Codex adversarial review GATED on team size >=3 AND a real outage costing >4 hours.
+ - /insights monthly + /prune quarterly.
- - Auto-research loop — explicitly **deferred**. Out of scope for these three phases.
+ - Auto-research loop — IN SCOPE for Phase 3, but ONLY on the six verifiable skills above. Never on grill / plan-feature / to-prd / taste-QA.

## Resolved decisions
+ - **Spec vs code source-of-truth** — code is ground truth for what shipped; spec is intent of record for what we agreed to ship; drift-check is the bridge.
+ - **PRD review** — kept, scoped as 3-question 5-minute misalignment scan, not a polish ritual.
+ - **PRD lifecycle** — archived (not deleted, not retained); never auto-loaded; scout sub-agent decides relevance; >14 days = stale tag.
+ - **AFK eligibility** — binary verifiable judge AND reversible blast radius. Property of the skill, not a team graduation.
+ - **YAML DAG harness** — explicitly rejected. PIV phase boundaries are conventions inside skills, using frontmatter Claude Code already parses.
+ - **Agent org charts** — explicitly rejected. By-function fan-out only, with single human integrator.
+ - **Multi-provider posture** — file mirror Phase 1, active invocation Phase 3.
+ - **Plan mode positioning** — wrap, don't replace. /grill blocks plan mode until discovery file exists.

## Open questions
- 1. **Verifier as the third standard agent** — recommend yes ...
+ 1. RESOLVED: Verifier is the third standard agent.
- 3. **Codex usage scope** — design adversarial in Phase 1 only, ...
+ 3. RESOLVED: Codex stays optional Phase 1 (design adversarial review only). Active invocation Phase 3, gated on team size ≥3 + outage evidence.
+ 11. NEW: How does /grill interact with Linear ticket creation — does it block on no discovery file, or warn?
+ 12. NEW: Does the binary-eval block in SKILL.md frontmatter validate at write-time or at /init time?
+ 13. NEW: Does /budget read from the Max subscription API or from local heuristics?
```

Edits to apply (the user runs them):
- **Vision / Approach** — add the substrate-AND-safety framing to Phase 1 description.
- **Phase 1 Scope** — append the 7 debate-installed items to the existing Phase 1 scope list.
- **Phase 2 Scope** — append PIV-as-convention, per-skill model pinning, /postmortem, tour, integrator-required-for-fan-out.
- **Phase 3 Scope** — replace the "auto-research loop deferred" line with the scoped six-skill version; add gated worktrees, gated cross-provider, /insights, /prune.
- **Resolved decisions** — append the eight new resolved items above.
- **Open questions** — close items 1 and 3; add new items 11–13.
