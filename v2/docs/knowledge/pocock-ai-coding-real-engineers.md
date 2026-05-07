# AI Coding for Real Engineers — The Full Workflow Workshop

**Author:** Matt Pocock
**Source:** YouTube workshop recording (live conference workshop, ~2 hours)
**Duration:** ~120:00

## Executive Summary
This is Pocock's full end-to-end workflow for serious AI-assisted development, taught against the constraints LLMs actually have. Two non-negotiable LLM properties drive everything: a **smart zone / dumb zone** (cognition collapses around 100k tokens regardless of advertised context window) and a **Memento-like amnesia** (every clear resets back to the system prompt). From those constraints he derives a pipeline — grill-me → PRD → camp board of vertical slices → AFK Ralph loop → QA → re-loop — anchored in TDD (red/green/refactor) and deep-module discipline. The thesis: software engineering fundamentals are not obsolete; they're how you stay effective when the LLM is the implementer.

## LLM Constraint 1 — The Smart Zone and the Dumb Zone
Attribution: Dex Hy of Human Layer.

> **Direct from video:** "When you're working with LLMs, they have a smart zone and a dumb zone… every time you add a token to an LLM, it's kind of like you're adding a team to a football league… by around sort of 40% or around I would say around 100k is kind of my new marker for this because it doesn't matter whether you're using 1 million context window or 200k. It's always going to be about this. It starts to just get dumber."

Implication: size every task to fit inside the smart zone. Watch the token count constantly — Pocock keeps a status line showing exact token usage in every session.

## LLM Constraint 2 — They're Like the Guy From *Memento*
Every session has the same shape: system prompt → exploration → implementation → testing/feedback. When you `/clear`, you reset to the system prompt with no memory of what came before.

> **Direct from video:** "I much prefer my AI to behave like the guy from Momento because this state is always the same. Always the same. Every time you do it, you clear and you go back to the beginning."

Pocock dislikes `/compact` for coding. Compacting creates "sediment" — a written history that drags on every later turn. Clearing is cleaner. Optimise for clears, not for compacts.

## Phase 1 — Grilling (Human in the Loop)
Start every piece of work with `/grill-me <client-brief>`. The skill is two lines but causes the AI to ask 40–100 questions, walking down the design tree.

> **Exact quote:** "Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one. For each question, provide your recommended answer. Ask the questions one at a time."

Why it works: you and the AI need a shared **design concept** (Frederick P. Brooks, *The Design of Design*) — an ephemeral, invisible idea of what's being built, not a markdown asset. The grilling produces it.

Practical notes from the demo:

- The skill exhausts a sub-agent for the explore phase (~93k tokens) but only ~25k bubble back up to your main context.
- Each question comes with a recommended answer — you can speed-run by accepting recommendations.
- Critical product questions (e.g. "should points be retroactive?") need humans, possibly multiple humans (mob programming with AI). Planning is human-in-the-loop; implementation is AFK.

## Phase 2 — Two Documents: Destination and Journey
After grilling, Pocock writes two artifacts:

1. **Destination document** — the PRD. Captures problem, solution, user stories, implementation decisions, and *testing decisions*. He does **not** read it: "What am I testing? LLMs are great at summarization. I have reached the same wavelength as the LLM." He trusts the summarization because the design concept is already shared.
2. **Journey document** — the camp board. A set of independently-grabbable issues with blocking relationships, broken into **vertical slices**, not horizontal layers.

The PRD template includes module changes:

> **Principle:** "Inside the PRD I'm specific about the module changes and the interfaces inside those modules how they're being modified."

## Phase 3 — Vertical Slices, Not Horizontal Layers
Tracer bullets / vertical slices, from *The Pragmatic Programmer*. AI's default is to code horizontally — phase 1 = all DB, phase 2 = all API, phase 3 = all front-end — which means no integrated feedback until phase 3.

> **Direct from video:** "AI loves to code horizontally… You don't get feedback on your work until you've really started or completed phase three."

The fix: each issue must be a thin slice that crosses every layer it needs to. "Award points for lesson completion visible on dashboard" is a good vertical slice; "create gamification service" alone is horizontal.

A camp board of vertical slices is a directed acyclic graph — multiple agents can work in parallel along independent edges. A sequential plan can only be picked up by one agent.

## Phase 4 — AFK Implementation (Ralph Loop)
Once the camp board exists, the human leaves the loop. Pocock calls his loop **Ralph** (after Ralph Wiggum / the Ralph Wiggum software practice).

The harness is small — a bash script (`once.sh`) that:
1. Caps all issue markdown files into a variable.
2. Grabs the last 5 commits.
3. Runs Claude with `--permission-mode acceptEdits` and that bundled context.

Run inside Docker so commits are produced in a sandbox and patched out to the host repo.

Ralph's prompt prioritises: critical bug fixes → dev infra → tracer bullets → polishing/quick wins/refactors. It uses TDD to complete each task and runs feedback loops (tests + types) on every commit.

> **Day shift / night shift framing (Jamon, Twitter):** the human does the day shift — ideas, grilling, PRDs, issue creation. The AI takes the night shift — implementation. Once the camp board is queued, your work is done until QA.

## Phase 5 — TDD Is Non-Negotiable

> **Direct from video:** "TDD I found is absolutely essential for getting the most out of agents."

The skill teaches the AI **red → green → refactor**:

1. Write a single failing test.
2. Confirm it fails for the right reason ("confirmed red").
3. Make the implementation pass ("green").
4. Refactor.

Without TDD, AI cheats — it writes the implementation, then writes tests that match the implementation. With TDD, it instruments the code *before* writing it, which makes cheating much harder and yields good tests as a side effect. Pocock warps his entire workflow around making TDD pull-able.

## Phase 6 — Reviewing the Work
Tokens are cheap; AI is good at reviewing. **Always have an AI review step after implementation, in a fresh context.**

> **Principle:** "If you get it to sort of try to do its reviewing, it's going to be doing the reviewing in the dumb zone… whereas if you clear the context, then you're essentially going to be able to just review in the smart zone, which is where you want to be."

Implementation produces a commit; reviewer is a separate, cleared agent reading just that commit's diff.

## QA Loop — Feedback Buttons and Re-Looping
After Ralph closes its issues, you QA. Pocock's app has a "feedback" button: describe a bug in detail, it creates a GitHub issue (Haiku-generated title + the route + the description), Ralph picks it up. While you continue QA, Ralph fixes bugs in parallel.

Edge cases that fall out of QA — "what happens if the directory isn't a git repo?" — are exactly why specs-to-code can't work alone:

> **Direct from video:** "When you're in the QA loop, when you're iterating towards something, you are going to find little weird edge cases like this that is really hard to plan for before."

## Codebases Are the Battleground
A repeated motif. Bad codebase → bad agent output. Shallow modules → AI gets lost in exploration. Deep modules with clear seams → testable, navigable, fast.

> **Direct from video:** "Bad code bases make bad agents. If you have a garbage codebase you're going to get garbage out of the agent that's working in that codebase."

## Operational Rules of Thumb
- **Keep the system prompt tiny.** People who load 250k of context are "in the dump zone without even being able to do anything."
- **Watch tokens like a hawk.** A status line showing exact token count is "essential information on every coding session."
- **Sub-agents are delegation.** They burn tokens in an isolated context window and report a summary back. Use them for explore.
- **Don't use ask-user-question.** Pocock dislikes the UI; not calling a tool is always more token-efficient than calling one.
- **Own your stack.** Don't depend on opaque planning frameworks (Spec-Kit, OpenSpec, Taskmaster). When things break you need observability.
- **1M context windows are mostly more dumb zone.** Good for retrieval, not coding. Smart zone is still ~100k.
- **PRs will be larger and more numerous.** Code review volume goes up; he doesn't have a clean answer for this.

## Key Exact Extracts
> **[03:24]** "By around sort of 40% or around I would say around 100k is kind of my new marker for this because it doesn't matter whether you're using 1 million uh context window or 200k. It's always going to be about this. It starts to just get dumber."

> **[09:00]** "Devs love compacting for some reason, but I hate it. I much prefer my AI to behave like the guy from Momento because this state is always the same."

> **[16:22]** "Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies one by one."

> **[35:33]** "I have reached the same wavelength as the LLM, right? Using the grill meme skill, we have a shared design concept. So if I have a shared design concept, all I'm doing is I'm just essentially checking the LLM's ability to summarize. So I don't tend to read these."

> **[42:51]** "AI loves to code horizontally."

> **[44:00]** "You don't get feedback on your work until you've really started or completed phase three."

> **[1:07:35]** "TDD I found is absolutely essential for getting the most out of agents."

> **[1:08:00]** "It's writing a failing test first… and then I need to make the implementation pass."

> **[1:05:00]** "If you get it to sort of try to do its reviewing, it's going to be doing the reviewing in the dumb zone… whereas if you clear the context, then you're essentially going to be able to just review in the smart zone."

> **[36:55]** "Bad code bases make bad agents. If you have a garbage codebase you're going to get garbage out of the agent that's working in that codebase."
