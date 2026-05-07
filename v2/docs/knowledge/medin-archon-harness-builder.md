# Archon — The Open-Source Harness Builder for AI Coding

**Author:** Medin
**Source:** YouTube tutorial (Archon launch / overview)
**Duration:** 30:48

## Executive Summary
Archon is Medin's pitch for a new layer of the AI coding stack: not a coding agent and not a single opinionated harness like B-MAD or the Ralph Loop, but a *harness builder* that sits above Claude Code and Codex. The argument: prompt engineering → context engineering → harness engineering is the trajectory the industry is on (Stripe Minion, Anthropic's own agent-team work), and harnesses are the layer that turns a 6.7% PR acceptance rate into something closer to 70%. Archon lets you encode your own software-development life cycle as a workflow of nodes (prompt or deterministic), run them across any registered repo, and parallelise execution.

## What a Harness Is, and Why It Matters
A harness is the orchestration layer that wraps the coding agent — the tooling, prompting, and chaining of multiple sessions that elevates a model beyond what you get from a single chat.

> **Direct from video:** "Harnesses are the future. It's the layer on top of your coding agents that orchestrates the different sessions. It's what makes AI coding deterministic and repeatable."

The progression Medin draws:
- **Prompt engineering (2022–2024):** craft a single best output.
- **Context engineering:** curate the perfect context window for one agent doing larger work.
- **Harness engineering:** orchestrate many coding-agent sessions together to handle much larger sets of work.

Evidence cited: studies showing PR acceptance moving from 6.7% (raw model) to ~70% (with a harness around it); Stripe shipping ~1,300 AI-generated PRs/week through Stripe Minion (a closed-source internal harness); the Claude Code source-code leak revealing roughly 40% of the codebase is harness-style code (agent teams, sub-agent infrastructure). Up until Archon, the open-source options were either single opinionated harnesses (Ralph Loop, B-MAD, GitHub Spec Kit, GSD) or rolling your own from scratch.

## Workflows as YAML Nodes
Every Archon workflow is a list of nodes. Each node is either a prompt sent into a coding-agent session or a deterministic command (bash, Python, TypeScript). This hybrid is the point.

> **Direct from video:** "There are certain steps of the workflow that we don't want the coding agent to decide. Like sometimes we want to curate context in a specific way, or run our tests in a specific way. We have nodes for that that we can build into Archon workflows."

Workflows can:
- Specify the model per node (Haiku for classification, Sonnet for review, Opus for implementation) for token efficiency.
- Continue a session or open a fresh one between nodes (deliberate context resets).
- Reference inline prompts or external command markdown files.
- Loop, branch on classification (e.g. bug vs. feature), and pause for human approval gates.

The example workflow Medin uses throughout: extract issue number → classify (bug/feature) → web research → investigate or plan → implement → validate → open PR → review.

> **Definition (workflow description):** "Just like Claude Code's skills, where the description is what we first give to Claude Code. We don't want to load the entire workflow into context for the coding agent. That's way too much. It only needs this brief description up front. So, it uses this to determine if it should analyze and run this entire workflow."

## Setup in Five Minutes
The install flow is itself an Archon-style workflow: clone the repo, run Claude Code in it, say "set up Archon," and the bundled Archon skill walks the agent through everything. Steps it covers: prerequisite checks (Bun installed if needed), choosing the first registered project, picking adapters (CLI, web UI, GitHub, Slack, Telegram), selecting database (SQLite default or Postgres), authenticating with Claude (Anthropic subscription via OAuth is supported), and copying the Archon skill into the target repo so any future Claude Code session there can invoke workflows.

A separate terminal handles credential entry so API keys never pass through the coding agent.

Once installed, invoking a workflow is one sentence: "use Archon to fix issue number one in GitHub." The agent loads the Archon skill, picks the right workflow from the bundled defaults, and dispatches it as a background process.

## Default Workflows and Parallel Execution
Archon ships with a substantial library of default workflows — Medin shows: `archon-fix-github-issue`, `archon-create-pr-from-idea`, `archon-validate-pr`, comprehensive PR review, an interactive PRD with human-in-the-loop, idea-to-PR, the Ralph Loop ported as an Archon workflow, and a workflow-builder workflow (used to build new workflows). These can all be run in parallel — Medin demos six fixes invoked simultaneously, each running as a background process while Claude Code monitors them.

> **Direct from video:** "Six times in a row, all of them are running as background processes."

The web UI mirrors the YAML: the registered Archon agent has the registered projects and available workflows injected into context, so requests like "fix GitHub issue 3 in the rag YouTube chat project" route automatically.

## Building Custom Workflows
Two ways to make new workflows:
1. Hand-edit the YAML in `.archon/`.
2. Use the workflow-builder workflow: `use the workflow builder workflow to help me make an Archon workflow` — describe what you want, the builder runs research, asks questions, and emits the YAML. Medin demos building a workflow that incorporates ideas from Beads (persistent structured memory for coding agents) in a single live invocation.

> **Direct from video:** "Literally like no matter what you want to do with your AI coding assistants, it doesn't matter how many coding agent sessions you need, you can bundle it together into an Archon workflow, adding in reliability through deterministic nodes."

## Key Exact Extracts
> **[00:20]** "Harnesses are the future. It's the layer on top of your coding agents that orchestrates the different sessions. It's what makes AI coding deterministic and repeatable."

> **[04:09]** "What Archon unlocks for you is being able to create your own custom harness wrapping up your own software development life cycle."

> **[06:32]** "Define once, run forever, reusable across projects."

> **[07:18]** "We're building in reliability through deterministic steps, enforcing validation at certain steps of the way, and human approvals."

> **[08:01]** "There are certain steps of the workflow that we don't want the coding agent to decide... We have nodes for that that we can build into Archon workflows."

> **[18:48]** "All of the workflows in Archon are simply defined as YAML files."

> **[20:53]** "One of the powerful things we can do with Archon is specify the model we want to use for the individual nodes."

> **[22:54]** "This is sort of a harness wrapping many different Claude Code sessions to work together to fix a GitHub issue."
