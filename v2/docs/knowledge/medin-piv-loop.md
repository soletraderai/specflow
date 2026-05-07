# The PIV Loop — Plan, Implement, Validate (with System Evolution)

**Author:** Medin
**Source:** YouTube tutorial (polished AI Transformation Workshop session with Leor Weinstein)
**Duration:** 66:58

## Executive Summary
A condensed version of Medin's full corporate AI-coding training, boiled down to a foundation that engineers and PMs can adopt without ripping out their existing process. The shape: a three-phase methodology for working with coding agents — ideation, the PIV loop (plan/implement/validate) for individual tickets, and system evolution to make the agent more reliable over time. The argument throughout: keep the foundation simple, then mold it to your software-development life cycle. Frameworks like B-MAD, GSD, and GitHub Spec Kit are inspirational but over-engineered for most teams.

## Why Not Just Use B-MAD or Spec Kit
Medin spends time up front explaining why he teaches a foundational methodology rather than handing teams an off-the-shelf framework.

> **Direct from video:** "A lot of these frameworks are very overengineered. They try to do too much at once and it's really difficult to take an existing off-the-shelf framework and mold it to your software development life cycle."

The corporate trainings he runs explicitly preserve existing team conventions — sprint structures, PRD templates, QA processes — and bake them into commands and skills the agent can run.

## Phase 1 — Ideation
The starting point is intentionally unstructured: open Claude Code, brain-dump (Medin uses speech-to-text), then transition to clarifying questions.

> **Direct from video:** "The most important part when you're first planning work with a coding agent is to reduce the number of assumptions that it is making. Because honestly, most of the time when a coding agent does a bad job, it's not like the code is just broken. It's that it's not aligned with what you are actually looking to build."

Two procedural tools that turn the unstructured conversation into structured artifacts:

1. **`/create-prd` command.** Enforces a fixed PRD structure (executive summary, mission, target users, in/out of scope, etc.) as a single markdown source-of-truth. The team's existing PRD conventions get encoded into this command so the agent produces what the team already expects.
2. **`/create-stories` command.** Takes the PRD path plus Jira project/epic IDs as arguments, breaks the PRD into user stories with acceptance criteria, and pushes them as Jira tickets via the Atlassian MCP server. Works with markdown-only output if there's no Jira integration.

Important: Medin separates `/create-prd` and `/create-stories` deliberately so a human reviews and edits the PRD between them. The PRD is high-stakes; the worst place for bad assumptions to land is in tickets.

> **Direct from video:** "Once you have iterated on the plan and you're confident in everything, you actually don't do the implementation right here, we want to start a brand new session with Claude code."

## Phase 2 — The PIV Loop (per Ticket)
For every Jira ticket, GitHub issue, or Linear task, the developer runs the same three-step loop. Two layers of planning matter here:

> **Definition — two-layer planning:** "Layer one planning is higher level. Here are the features that we want to build or the bugs we want to fix at this point we are not digging into the code. Now that we're picking a single ticket for layer 2, this is where we get more in the weeds of things. This is where we're going to analyze the codebase, the documentation, figure out what parts of the codebase we actually have to touch."

The PIV loop steps:

1. **Plan.** Start with a fresh session. Run `/prime` (or a specialised variant like `/prime-workflows`) with the issue ID as an argument; it loads the relevant codebase context and recent git log for long-term memory. Then explore — Medin uses parallel sub-agents for exploration to keep the main window clean. Once aligned with the agent on approach, run `/plan` to produce a `plan.md` with summary, locked decisions, files to touch, task list, and self-validation strategy.
2. **Implement.** Open a *brand-new* Claude Code session. Run `/implement <plan-path>`. The fresh session has no bias from the planning conversation, and the plan markdown carries everything needed. The implement command also handles admin work — branching, running unit/lint/type checks, optional end-to-end testing via the agent-browser skill, comparing the diff back to the plan to detect drift, posting a comment to the Jira ticket, and opening the PR.
3. **Validate.** Human review of the code, plus the agent's own self-validation. The agent should take care of as much as possible (linting, unit tests, integration tests, end-to-end if applicable) so the human is not the bottleneck.

> **Direct from video:** "We want it to take care of as much validation as possible so that by the time control passes back to us for our human validation, there's less that needs to be corrected."

## Phase 3 — System Evolution (the Outer Loop)
The most powerful part of the methodology. After every PIV loop, if a problem surfaced, fix the *system that allowed it*, not just the bug.

> **Direct from video:** "We don't have to just treat the bug as a one-off fix that we address and then move on to the next pivot loop... We can spend some time to fix to also fix the system that allowed the bug."

The four things Medin updates as part of system evolution:
- Commands
- On-demand context (including team docs in Confluence/Drive — these can be optimised for AI consumption too)
- Global rules
- Plan and PRD templates

There are effectively two loops: the inner loop is the PIV cycle when things are running smoothly; the outer loop is system evolution when the agent makes a mistake worth learning from. Outer-loop changes flow through source control like any other code change — branch, PR, review, merge — so the team's AI layer evolves alongside the codebase.

> **Direct from video:** "Every single time you improve a command or a skill, it might save engineers dozens and dozens of hours going forward because you've now made the validation process more reliable or you've made the style conventions respected more often."

## What Stays Human, What Stays Agent
The framing throughout: delegate writing, but stay in the driver's seat for planning and validation. PMs touch the system at the front end (ideation, PRD generation, story creation in Jira/Linear/GitHub). Engineers pick up tickets and run the PIV loop. Both groups review artifacts at every step.

> **Direct from video:** "Our job as an engineer is to no longer write the code, but to do the higher leverage tasks like the planning and validating."

## Key Exact Extracts
> **[02:56]** "We're not vibe coding because we are putting ourselves in the driver's seat along the way through all of the planning and validation that we do."

> **[05:35]** "A lot of these frameworks are very overengineered. They try to do too much at once and it's really difficult to take an existing off-the-shelf framework and mold it to your software development life cycle."

> **[09:35]** "The most important part when you're first planning work with a coding agent is to reduce the number of assumptions that it is making."

> **[39:09]** "For planning with AI coding, you always have two layers. You have the project level planning... and then you have the task planning and that is the individual ticket level."

> **[48:00]** "Just because you can fit a million tokens into a large language model does not mean that you should because they get overwhelmed just like people do."

> **[53:53]** "When you are working with AI coding assistants, you want to make sure that they are as focused as possible. And it's important to be focused in order to be focused to do your planning and implementing in separate sessions because also the coding agent has probably built up a lot of bias throughout this conversation."

> **[57:40]** "We can have a sort of you know retroactive session with the coding agent where we say okay Claude you allowed this problem to creep into my codebase. I want you to dive into your AI layer."

> **[60:02]** "The entire team can reuse these things and you can even create pull requests to update commands just like you create pull requests to update your codebase."

> **[61:40]** "Basically two loops here. You have the inner loop when everything's working well and you're just chugging through the work with the help of your coding agent. And then you have the outer loop when you're taking some time to reflect and make your system better."
