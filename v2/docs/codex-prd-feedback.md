# Codex Adversarial Review — PRD Feedback

**Job ID:** review-mol11eo2-4ozhf7
**Codex session:** `019ddccd-1b3a-7e03-9da5-621c99cae75b`
**Date:** 2026-04-30
**Target:** working tree (PRD.md, SESSION-HANDOFF.md, knowledge/ artefacts)
**Verdict:** **needs-attention**

---

## Summary

> No-ship: the PRD has real safety intent, but Phase 1 is overloaded and two trust-boundary choices would let AI overrule or misclassify work in exactly the confidential-repo paths this roadmap is meant to protect.

---

## Findings

### 1. [HIGH] Claude is allowed to overrule adversarial review after only three rounds

**Where:** `PRD.md:729-733` (Appendix N — 3-iteration debate loop)

**Issue:** The debate loop makes the writer/developer the final tiebreaker after the reviewer counters. For Green-lane low-risk work this may be acceptable, but the same rule is specified for every gate, including code-vs-plan and tests-vs-requirements. In a confidential repo, this creates a failure path where the agent that produced the artifact can dismiss unresolved reviewer objections and proceed, while the human only spot-checks the saved transcript later.

**Impact:** High-risk or confidential changes can ship with known-but-overruled concerns. The final decision should depend on lane and severity, not on artifact ownership.

**Recommendation:** Change the debate contract so unresolved high/critical findings block by default. Let Claude make final calls only for Green-lane non-critical disagreements; require human tiebreaker for Yellow, Red, confidential paths, auth/billing/schema/public-surface changes, and any Codex/Verifier high-confidence finding.

**Feedback from Human:** No, this stays with Claude. The reason why this stays with Claude is because this is mainly just for the tasks and to make sure that the task plan is the right way to do it. The reason we're going to go with Claude and the reason why we're going to just yank it there and not force this any longer is that we still need to be productive here. If you guys can't come to a decision between Codex and Claude, then it just needs to stay with Claude, because Claude's the one that's going to be developing in the end. Once the development has been through, we will have a human review everything anyway. We should not be overthinking this. We still need to be productive at the end of the day.

**Codex Re-assessment:** *Status — still open (HIGH).* The response sidesteps the underlying risk. The PRD applies the 3-round loop to **every** gate, not just task plans — that includes code-vs-plan (Gate 5) and tests-vs-requirements (Gate 6). Framing it as "mainly task-plan review" understates the surface area where Claude can dismiss unresolved adversarial findings. "Human review later" is not equivalent to blocking semantics at the point of disagreement, especially for Yellow/Red/confidential/auth/schema/public-surface work — by the time later review fires, the work is already done and the override is buried in a transcript.

Compromise that preserves productivity: keep Claude as tiebreaker only for **Green-lane, non-critical disputes**. Require human tiebreaker for: unresolved high/critical findings, confidential paths, auth/billing/schema/public-endpoint changes, and any Yellow/Red lane dispute. This blocks where it matters without slowing the routine work.

---

### 2. [HIGH] Confidentiality classification relies on path globs that cannot catch semantic blast radius

**Where:** `PRD.md:609-613` (Appendix L — green/yellow/red lane execution)

**Issue:** The lane model says confidentiality is rule-based via `config.json.confidentialPaths`, never AI-rated. That is safer than confidence scoring, but path globs alone miss sensitive behavior routed through generic files such as shared utilities, middleware helpers, validation code, API clients, logging, serialization, or test fixtures. A task touching `src/utils` could alter auth, secrets, PII handling, billing, or schema semantics without matching a protected path and be batched as Green if it appears verifiable and low-blast.

**Impact:** Protected work can escape the Red human-led lane under ordinary refactors.

**Recommendation:** Keep path globs as the first layer, but add deterministic semantic triggers before lane assignment: changed imports/references to auth, billing, schemas, secrets, PII, permissions, migrations, public endpoints, encryption, logging of user data, and config/env access. Any match should force Yellow/Red unless a human explicitly downgrades it with a recorded reason.

**Feedback from Human:** I don't know what this all means, to be honest.

**Codex Re-assessment:** *Status — still open (MEDIUM).* Plain-language reframe: **protected work is not only protected files**. Ordinary files can change protected behaviour. Examples — a shared utility, a middleware helper, a logger, a serializer, an API client, or a validation function can alter auth, PII handling, billing, schema, or permissions *without ever touching a file under `confidentialPaths`*. If path globs are the only classifier, an ordinary refactor that imports an auth helper into `src/utils` gets batched as Green and ships with one batched sign-off — even though it changed how authentication works.

Concrete rule that solves it: any task that touches (or *imports/references*) auth checks, user identity, permissions, billing, schemas, migrations, secrets, env vars, PII, logging of user data, public endpoints, or encryption is **automatically Yellow/Red, regardless of file path**. Path globs become layer one (fast first filter); semantic triggers are layer two (catches the indirect cases). A human can still explicitly downgrade with a recorded reason, but the default tilts toward safety.

---

### 3. [MEDIUM] Phase 1 is too large to be a foundation and will delay the safety primitives it depends on

**Where:** `PRD.md:134-154` (Phase 1 — Foundation, 17 priority items)

**Issue:** Phase 1 includes setup, migration, rules, `/grill`, PRD/task enhancements, three adversarial gates, standard agents, trust-ladder primitives, design tooling with Playwright, path-scoped rules, AGENTS mirroring, budget tracking, SKILLS glossary, testing cadence, bounded autoresearch, feedback-loop audit, and CONTEXT generation. This contradicts the stated goal of shipping only the substrate and also conflicts with the later claim that existing skill changes in Phase 1 are mechanical path updates only. The likely failure is not elegance; it is that the team spends Phase 1 building optional machinery while the critical day-one controls remain unfinished or under-tested.

**Recommendation:** Cut Phase 1 to the smallest safety substrate:
- setup/upgrade migration
- `/grill`
- PRD misalignment scan
- persisted task coverage review
- feedback-loop audit
- CONTEXT glossary
- trust-ladder primitives
- rules starter set

Defer to later phases: design, AGENTS mirroring, `/budget`, `simplify` autoresearch, full path-scoped rule infrastructure, and broad testing-cadence enforcement. Ship the core workflow and dogfood it before adding the optional machinery.

**Feedback from Human:** Happy to split this into smaller phases if need be. It probably wouldn't be a bad idea, so we can develop, double-check to make sure that we got the files and everything is all looking correct, and then move to the next one.

**Codex Re-assessment:** *Status — directionally accepted, not yet committed (LOW).* The acceptance is sound and the trade-off is reasonable, but it isn't yet an edit to the PRD. Without a committed rescope, the overloaded Phase 1 list can remain unchanged in `PRD.md` while the team believes the concern is resolved.

Concrete edit to make: split into **Phase 1A — Minimum safety substrate** and **Phase 1B — Optional machinery**.

- **Phase 1A:** setup/upgrade migration, `/grill`, PRD misalignment-scan, persisted task coverage review (per Finding 4), feedback-loop audit, CONTEXT glossary, trust-ladder primitives (`panic`, `confidence-check`, `getting-started`), rules registry starter set, behavioural principles, standard agents.
- **Phase 1B:** `specflow:design`, AGENTS auto-mirror, `/budget`, `simplify` autoresearch, full path-scoped rule infrastructure, broad testing-cadence enforcement.
- **Hard dependency rule:** Phase 1B cannot start until Phase 1A is shipped, dogfooded on at least one real PRD, and reviewed.

This converts the acceptance into structure the team can actually execute against.

---

### 4. [MEDIUM] Task coverage sign-off is intentionally ephemeral during the phase that needs auditability most

**Where:** `PRD.md:140-144` (Phase 1 item 7 — `specflow:task` enhancement)

**Issue:** `specflow:task` produces the coverage matrix and intent summaries in chat, then the docs creator signs off once; only recuts/corrections are logged for later self-learning. For a docs creator already overwhelmed by 50+ tasks, this removes the review artifact that would later explain why a task existed, which PRD requirement it covered, and what the human approved. Phase 3 later records task intent summary and PRD anchor in `task-history.json`, but that means Phase 1 task generation has no durable sign-off trail when the process is most immature.

**Impact:** Wrong task slicing or missed requirements become hard to reconstruct after implementation starts.

**Recommendation:** Persist a compact task-review artifact in Phase 1:
- PRD requirement ID
- Generated task IDs
- Highlighted intent summaries
- Reviewer sign-off
- Unresolved assumptions
- Recut decisions

It can be small and excluded from default context loading, but it should not exist only in chat.

**Feedback from Human:** Yeah, I get what you're saying, and I completely agree, but what do we actually do here? Every task should be reviewed. The task is generated by Claude, then reviewed by itself using a Devil's Advocate Agent. Once it's gone through that process, it needs to be passed on to Codex Adversarial Review. That then looks at it with a fresh set of eyes, provides its feedback if there's anything. If there is feedback, it needs to be flagged with Claude, where Claude looks at that feedback, looks at its initial intent, and it should ask its questions:
- What was I trying to achieve?
- Why did I decide to do it this way?
- Is this feedback of help?
- Does this feedback provide me with some sort of context to be able to complete this task better that are in line with our core concepts?

I guess, reading this out, it's probably the way to do it that if there is feedback, then Claude should run through a series of questions that we define, and it would be those questions:
- What was I trying to achieve?
- Why did I decide this way etc

**Codex Re-assessment:** *Status — still open (MEDIUM).* The proposed loop (Claude generates → Devil's Advocate self-reviews → Codex adversarial-reviews → Claude reflects via questions) is genuinely useful as a **repair step**, but it does not address the original auditability risk. After the loop runs, there is still no durable record of: the coverage matrix, highlighted task intent summaries, reviewer findings, Claude's response, human sign-off, unresolved assumptions, or recut decisions. When something goes wrong six weeks later, there's nothing to reconstruct *why* a task existed and what was approved.

Worse: the loop terminates with "Claude reflects and decides" — which **repeats the Finding 1 failure mode**. If Codex flags a high-severity issue and the reflection questions don't satisfy Claude, Claude still proceeds. Reflection without blocking semantics is just a more polite override.

What actually solves Finding 4 — combine both:

1. **Persist a compact `task-review` artefact** per PRD/task run. Suggested fields (small file, excluded from default context loading):
   - PRD requirement IDs covered
   - Generated task IDs + 2-sentence intent summaries (the highlighted ones)
   - Reviewer findings (Devil's Advocate + Codex)
   - Claude's response to each finding (using your reflection questions as the *response shape*)
   - Human sign-off line (with date)
   - Unresolved assumptions
   - Recut decisions
2. **Apply blocking semantics:** high/critical unresolved findings block the artefact from being signed off. Claude's reflection questions are the **mechanism for engaging with feedback**, not the mechanism for dismissing it.

This keeps everything you proposed — including the reflection-questions process — but adds the durable trail and the safety stop.

---

## Next steps (Codex's recommendations — original review)

1. **Re-scope Phase 1** around the minimum safety substrate before implementation planning.
2. **Revise Appendix N** so reviewer findings have blocking semantics by lane and severity.
3. **Replace path-only confidentiality** with layered path plus semantic risk classification.

---

## Next steps (Codex's recommendations — after re-assessment of human feedback)

After reviewing the human's responses, the open work narrowed to four concrete edits:

1. **Convert each human response into either a committed PRD edit or an explicit accepted risk** — "I'll think about it" leaves the PRD unchanged.
2. **Update PRD Appendix N** with lane/severity-based blocking semantics. Claude tiebreaker for Green-lane non-critical disputes only; human tiebreaker required for unresolved high/critical findings, confidential paths, auth/billing/schema/public-surface changes, and any Yellow/Red dispute.
3. **Replace path-only confidentiality classification** with two layers: path globs (layer 1) plus deterministic semantic triggers on imports/references to auth, identity, permissions, billing, schemas, migrations, secrets, env vars, PII, logging of user data, public APIs, encryption (layer 2). Either layer triggers Yellow/Red.
4. **Change `specflow:task` from chat-only coverage/sign-off to a persisted compact `task-review` artefact** containing requirement IDs, task IDs, intent summaries, reviewer findings, Claude's response (using the human's reflection questions as the response shape), human sign-off, unresolved assumptions, and recut decisions. High/critical unresolved findings block sign-off — they don't return to Claude as final judge.
5. **Split Phase 1 into 1A (minimum safety substrate) and 1B (optional machinery)** with a hard dependency rule: 1B cannot start until 1A is shipped and dogfooded on at least one real PRD.
4. **Persist task coverage sign-off** as a small audit artifact instead of chat-only output.
