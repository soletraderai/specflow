# Cole Medin's PIV Loop: A Foundational AI Coding System (Plan, Implement, Validate)

## Identity
- Title: AI Transformation Workshop — foundational system for reliable AI coding (PIV loop) [recorded live with Leor Weinstein]
- Speaker/Channel: Cole Medin (YouTube: @ColeMedin; GitHub: coleam00; founder of Dynamous AI, CTO Ottomator.ai)
- Likely URL: Posted on Cole Medin's channel as a polished re-cut of the live "AI Transformation Workshop" with Leor Weinstein. Closest matches in search: https://www.youtube.com/@ColeMedin and the related "From PRD to Jira in Minutes" video https://www.youtube.com/watch?v=2Kc658aIVNQ. Exact URL not confirmed in search results.
- Suggested slug: cole-medin-piv-loop-ai-coding-system
- Confidence: High that the speaker is Cole Medin (PIV loop terminology, three-phase framework — ideate / iterate / evolve, references to his repo of skills/commands, opposition to "overengineered" frameworks like BMAD, GSD, Cloudflow, his standard mention of GitHub Spec Kit, and the workshop format with Leor Weinstein). Medium on the exact video URL.

## Thesis
You don't need a heavyweight framework (BMAD, GSD, Spec Kit, Cloudflow) to ship reliable AI-assisted code. A simple, foundational three-phase system — (1) ideate from brain dump to PRD to tickets, (2) run a per-ticket "PIV loop" (Plan, Implement, Validate), and (3) evolve your AI layer over time — gives a small team repeatable, trustworthy results while keeping the human "in the driver's seat" through planning and validation. The engineer's job shifts from writing code to higher-leverage planning and validation; PMs become first-class participants because they own the upstream PRD/story creation step.

## Key points

### KP-1 Engineer's role shifts from typing code to planning and validating
- **Point**: "Our job as an engineer is to no longer write the code, but to do the higher leverage tasks like the planning and validating."
- **Why it matters to our goals**: Directly supports productivity (1) and fewer errors (3). For a docs creator + dev team new to AI coding, repositioning the human as orchestrator/validator is the single biggest mindset shift; it sets the expectation that quality comes from the spec/plan and review, not from the LLM "just being smart."
- **Evidence**: Lines 33–36 of the transcript; reiterated throughout (e.g. lines 80–84, "we are not vibe coding because we are putting ourselves in the driver's seat").
- **Sources**: Transcript lines 33–84.

### KP-2 Three phases: Ideate → PIV loop → System Evolution
- **Point**: The whole system is just three phases — ideate around the work, run a per-ticket PIV (Plan/Implement/Validate) loop, and evolve the agents over time.
- **Why it matters to our goals**: Gives specflow a concrete pipeline shape we can map to (we already have Plan / Build / Test / Release skills). The "system evolution" outer loop is a continuous-improvement engine for the plugin itself.
- **Evidence**: Lines 41–60 ("threepart process … ideulate … piv loop … system evolution mindset"). Inner vs outer loop framing at 1677–1684.
- **Sources**: Transcript lines 41–60, 1677–1700.

### KP-3 Off-the-shelf frameworks are overengineered; build your own minimal foundation
- **Point**: BMAD, GSD, Cloudflow, GitHub Spec Kit etc. are respected but "very overengineered… try to do too much at once" and are hard to mold to an existing SDLC.
- **Why it matters to our goals**: Validates specflow's philosophy of being a small, opinionated plugin rather than a heavyweight framework. Gives us a concrete competitive positioning: "foundational, moldable, simple on purpose."
- **Evidence**: Lines 134–183 ("simple on purpose because I want to show you the foundation that you can then build on top of to mold it into your process").
- **Sources**: Transcript lines 134–183.

### KP-4 Reduce assumptions by making the agent ask clarifying questions one at a time
- **Point**: "Most of the time when a coding agent does a bad job, it's not like the code is just broken. It's that it's not aligned with what you are actually looking to build." Fix: explicitly tell it to ask clarifying questions one at a time (Claude Code's `ask_user_question` tool supports this with multiple-choice answers).
- **Why it matters to our goals**: This is the single highest-leverage move for fewer errors (3) on a team new to AI coding. Cheap to add to specflow's plan/spec skills as a default behavior.
- **Evidence**: Lines 261–290 and 470–522 ("before you write anything ask me clarifying questions one at a time using the ask user question tool"). Recommends spending 20–30+ minutes here.
- **Sources**: Transcript lines 261–290, 470–522.

### KP-5 Two layers of planning: project-level (PRD) and task-level (plan.md)
- **Point**: PMs/initial planning produces a PRD (high-level: features, scope, target users) — no code-level detail. Then a separate task-level plan.md is created per ticket containing files to touch, patterns to follow, task order, and validation strategy.
- **Why it matters to our goals**: Maps cleanly to specflow's plan/spec stages and prevents the common failure of agents diving into code before scope is locked. Better product (2) by avoiding rework; productivity (1) by reusing PRDs across tickets.
- **Evidence**: Lines 1077–1109 ("you always have two layers… project level planning… task planning"). Plan structure shown 1387–1422.
- **Sources**: Transcript lines 1077–1109, 1387–1448.

### KP-6 Anything prompted >3 times becomes a command or skill
- **Point**: "Anytime you find yourself prompting something more than three times, you should turn it into a command or skill."
- **Why it matters to our goals**: Direct rule for specflow content strategy and for our team's adoption — gives us a clear threshold for what should become a packaged workflow.
- **Evidence**: Lines 332–351.
- **Sources**: Transcript lines 332–351.

### KP-7 Separate sessions for planning vs. implementation
- **Point**: Always start a fresh Claude Code session before running `/implement` against the plan. Long planning sessions accumulate bias and waste context; the plan.md should contain everything the implementer needs.
- **Why it matters to our goals**: Concrete reliability tactic. We can encode this in specflow as guidance ("end planning, open new session, then build").
- **Evidence**: Lines 1473–1504 ("we want to start a brand new session… coding agent has probably built up a lot of bias").
- **Sources**: Transcript lines 1473–1504.

### KP-8 Sub-agents for research to protect main context window
- **Point**: Spin up sub-agents to explore the codebase and do web research; they may burn 100k+ tokens and return a few-thousand-token summary. "Just because you can fit a million tokens into a large language model does not mean that you should because they get overwhelmed just like people do."
- **Why it matters to our goals**: Concrete pattern for our team that's new to AI coding — explains the "why" behind sub-agent delegation, not just the "how." Better product (2), fewer errors (3) because the planning agent stays focused.
- **Evidence**: Lines 1275–1316 (with token figures around 1289, 1300).
- **Sources**: Transcript lines 1275–1316.

### KP-9 Prime the agent with codebase + recent git history at the start of every session
- **Point**: A `/prime` command loads external context (Jira ticket, Confluence) plus the codebase + recent git commits — git is treated as long-term memory. Tunable lever: how much context to load up front.
- **Why it matters to our goals**: Cheap, generic step that makes every subsequent prompt better. Maps to a candidate specflow skill or convention.
- **Evidence**: Lines 1140–1228 ("I love using git as long-term memory for my coding agents").
- **Sources**: Transcript lines 1140–1228.

### KP-10 Have the agent self-validate before passing control back
- **Point**: After implementation the agent runs unit tests, integration tests, linting, type checks, and (optionally) end-to-end browser tests via an "agent browser" skill. Goal: "reduce us being the bottleneck for actually shipping."
- **Why it matters to our goals**: Shrinks human-review surface (productivity 1) and catches regressions before review (errors 3). specflow's test skill already does some of this — this validates the design.
- **Evidence**: Lines 1404–1469 and 1535–1568.
- **Sources**: Transcript lines 1404–1469, 1535–1568.

### KP-11 Plan-vs-implementation drift check
- **Point**: After implementation, the agent compares the produced code against plan.md to detect deviation, then updates the Jira ticket via MCP.
- **Why it matters to our goals**: Cheap automated guardrail we could add to specflow (a "verify-against-plan" step) — directly addresses goal (3).
- **Evidence**: Lines 1712–1724 ("after it does the implementation, it looks at the code and compares it to the plan to make sure that we didn't deviate").
- **Sources**: Transcript lines 1712–1724.

### KP-12 System Evolution: every bug is also a system bug
- **Point**: When the agent ships a bug, don't just fix the bug — open a retrospective session asking the agent to inspect its own AI layer (rules, commands, skills, plan/PRD templates) and propose changes so the class of bug can't recur.
- **Why it matters to our goals**: This is the compounding-returns mechanism. For a small team this is how the plugin gets better with use rather than rotting.
- **Evidence**: Lines 1572–1700. Four artifacts to evolve: commands, on-demand context (incl. Confluence), global rules, plan/PRD templates (1652–1661).
- **Sources**: Transcript lines 1572–1700.

### KP-13 Check AI layer artifacts into source control like code
- **Point**: Rules, commands, skills go in source control; updates go through PRs and code review like any other change. "Every single time you improve a command or a skill, it might save engineers dozens and dozens of hours going forward."
- **Why it matters to our goals**: Direct support for specflow's model of versioned skills + plugin.json. Validates that team adoption needs PR review of prompt/skill changes — not a free-for-all.
- **Evidence**: Lines 1628–1651.
- **Sources**: Transcript lines 1628–1651.

### KP-14 PMs are first-class users of the coding agent
- **Point**: PMs are "the first ones that have a touch point with the coding agent" — they own the brain dump → PRD → tickets stage. Ticket descriptions written via this pipeline are often better than human-only descriptions because the agent enriches them with codebase context.
- **Why it matters to our goals**: Perfect fit for our "docs creator + dev team" shape. Suggests specflow should explicitly market a PM/docs entry point, not just a developer one.
- **Evidence**: Lines 184–207 and 982–1011 ("for a product manager usually your description isn't even going to be this good because you don't have full context for the more technical details").
- **Sources**: Transcript lines 184–207, 982–1011.

### KP-15 Use MCP servers (Jira/Atlassian, Linear, GitHub) to eliminate "backstage" admin work
- **Point**: The Atlassian MCP creates Jira issues, posts technical-notes comments, assigns developers, updates ticket status, and even attaches PRs. Same pattern works with Linear MCP or `gh` CLI.
- **Why it matters to our goals**: Removes a meaningful chunk of human grunt work in our team's existing tools (productivity 1). Suggests specflow should ship/recommend MCP integrations rather than reinventing ticketing.
- **Evidence**: Lines 376–392, 856–928, 1718–1734.
- **Sources**: Transcript lines 376–392, 856–928, 1718–1734.

### KP-16 Two artifacts as gates: PRD.md and plan.md
- **Point**: The pipeline produces explicit human-reviewable artifacts at each gate. PRD before stories. Plan before implementation. Each is reviewed and edited by a human before the next stage runs.
- **Why it matters to our goals**: This is the concrete "human in the loop" shape — gives our team review checkpoints that aren't onerous. Maps to specflow's existing artifact-driven flow.
- **Evidence**: Lines 644–688 ("it's not good enough to just immediately create stories from that… important for us to review the artifact"); plan iteration 1409–1473.
- **Sources**: Transcript lines 644–688, 1409–1473.

### KP-17 Start unstructured, then move to structure (twice)
- **Point**: Both at PRD-time and plan-time, start with a free-form conversation/brain dump, then run a command that converts the conversation into a structured artifact. Don't over-template the entry point.
- **Why it matters to our goals**: Reduces the friction of starting — important for a team new to AI coding. Lowers barrier to entry while still producing structured output.
- **Evidence**: Lines 232–260 (PRD) and 1110–1135 (plan).
- **Sources**: Transcript lines 232–260, 1110–1135.

### KP-18 Don't mistake "1M tokens supported" for "1M tokens advisable"
- **Point**: Even with Opus/Codex/Copilot 1M-token windows, large contexts degrade performance. Sub-agents and per-stage session resets exist to keep working contexts small.
- **Why it matters to our goals**: Important guardrail to set early in our team's habits — prevents the natural beginner mistake of stuffing context.
- **Evidence**: Lines 1303–1316.
- **Sources**: Transcript lines 1303–1316.

### KP-19 Voice/speech-to-text for brain dumps
- **Point**: Cole uses speech-to-text for the initial brain dump. The point is to lower friction so people actually do the upfront context-loading step.
- **Why it matters to our goals**: Tiny adoption tip with outsized impact for non-developers (PMs/docs creator) on our team.
- **Evidence**: Lines 251–256, 1267.
- **Sources**: Transcript lines 251–256, 1267.

### KP-20 Process is tool-agnostic
- **Point**: "It doesn't matter in the end what tool you're actually using." Claude Code + Jira is the demo, but Codex + GitHub or Copilot + Linear works identically. The only requirements are one place to manage work and one place to talk to an LLM.
- **Why it matters to our goals**: Tells us specflow's value prop should be the workflow shape, not the tool coupling — important for a small team that may use mixed tools.
- **Evidence**: Lines 86–103, 824–831.
- **Sources**: Transcript lines 86–103, 824–831.

## Tools / repos / frameworks mentioned
- Claude Code (primary AI coding assistant used in demo); `ask_user_question` tool; Plan Mode (distinct from his `/plan` command); sub-agents; skills; commands; global rules; `MCP.json`.
- Atlassian MCP server (Jira + Confluence); Linear MCP server; GitHub CLI as alternatives.
- Codex / Codex CLI, GitHub Copilot — mentioned as interchangeable agents.
- Frameworks named only to contrast with his minimal approach: GitHub Spec Kit, BMAD, Cloudflow, GSD (Gastown).
- Cole's open-source workshop repo (linked in the original video description) containing his rules, commands (`/prime`, `/plan`, `/implement`, `create-prd`, `create-stories`), and skills (including an "agent browser" skill for end-to-end browser testing).
- Demo app: a simple poll builder (web app) used to showcase the full flow.
- Confluence and Google Drive mentioned as PRD storage targets.
- 1M-token context (Opus, Codex, Copilot) as a relevant model capability.

## Verification log
- Searched: `Cole Medin "Leor Weinstein" AI transformation workshop "PRP" OR "pivot loop" Claude Code` — confirmed Cole Medin teaches the PIV loop and contrasts it with frameworks like GitHub Spec Kit and BMAD.
- Searched: `"Cole Medin" "Leor" workshop poll builder Jira PRD "create stories" YouTube` — surfaced his "From PRD to Jira in Minutes: Automate User Stories with AI" video, which matches the create-stories portion of this transcript exactly.
- Searched: `"Leor Weinstein" AI transformation workshop Cole Medin coding` — confirmed Cole Medin's Dynamous AI / Ottomator.ai context and his "context engineering" thesis. Did not return a direct hit naming both presenters in one URL; the live AI Transformation Workshop with Leor Weinstein appears to be a relatively recent collaboration not yet broadly indexed.
- Internal evidence in transcript: speaker says "polished up version of a super valuepacked live workshop … with Leor Weinstein" (lines 1–6); uses "piv loop" terminology consistently (Cole's signature framework); references his GitHub repo of rules/commands/skills (line 108–124); rejects BMAD/GSD/Cloudflow/Spec Kit by name (lines 137–172). Combined signals make speaker identification high-confidence.
