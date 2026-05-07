# The Parallel Agentic Playbook — Five Pillars for 10x AI Coding

**Author:** Medin
**Source:** YouTube tutorial
**Duration:** 23:54

## Executive Summary
Medin's playbook for running many coding agents in parallel rather than just one or two at a time. The frame: 10x is easier than 2x because it forces you to build a *system* around your agents instead of shepherding them. The system rests on five pillars built around git work trees plus issue/PR-driven workflows, with a deliberate split between planning, building, and validation in separate sessions. A second half addresses the practical headaches that surface once agents do real end-to-end validation in parallel: port conflicts, dependency reinstalls, database state, token blowout, and PR pileup.

## The Five Pillars

### Pillar 1 — The Issue Is the Spec
Every implementation starts from a GitHub issue (or Linear/Jira ticket). Every validation starts from the pull request. The fan-out pattern: one upstream session creates a batch of issues for the sprint; many downstream sessions each pick one issue and implement it. With work pre-scoped as issues, parallel agents have natural boundaries.

### Pillar 2 — Plan/Build/Validate (Separate Sessions)
Each parallel agent runs its own plan → build → validate flow. The key constraint that makes parallelism viable: separate sessions for separate concerns, especially separating writing from reviewing. Medin uses his usual planning/implementation commands but the only prompt change between agents is the issue number.

### Pillar 3 — Git Work Trees for Isolation
Work trees are the codebase-isolation primitive. Claude Code supports them natively with `--worktree` (or `-w`). Medin uses a custom `w.sh` / `w.ps1` script that does more than just create the work tree — it also installs node_modules up front, creates a Neon database branch, and assigns a unique port. This script extends work-tree support to coding agents that don't have it built in.

> **Direct from video:** "When our coding agent works on the feature here, it's not going to be overriding other features that other coding agents are building. Each one of them has their own environment."

### Pillar 4 — Reviewer Never Sees the Writer's Chat
Pillars 4 and 5 aren't *about* parallelism, but they're what unlocks scaling. Code review must happen in a fresh context window — the writing session has built up too much bias.

> **Direct from video:** "There is so much bias that the agent builds up within the conversation that if you tell it to review the code in the same context window, it's like asking a kid to grade their own homework."

Medin's `/review-pr` command pulls the PR diff, compares it to the originating issue, and runs specialized sub-agents. He often layers a second adversarial review using a different coding agent (e.g. Codex via the Codex plugin for Claude Code) on the same PR. In the demo, that adversarial pass flagged real issues the first reviewer missed.

### Pillar 5 — The Self-Healing Layer
Whenever a bug surfaces, do not just fix the bug. Fix the system that allowed it. After validation finds a problem, ask the agent: what could we change in our rules, skills, workflows, or commands so this class of issue doesn't happen again?

> **Direct from video:** "We don't just fix the bug and move on, but we fix the underlying system that allowed for the bug."

This is also where the issue-in / PR-out artifact pair pays off: you can compare the PR to the originating issue and see exactly where the agent drifted from the plan.

## The End-to-End Validation Problems
Once agents run in parallel and do real end-to-end validation (not just static analysis), five operational problems show up. Medin's repo for the video documents the fixes so agents can reference them.

1. **Port conflicts.** Many concurrent app instances would otherwise fight for the same port. The solution: a startup command that hashes the work-tree name into a unique port, base 4000.
2. **Dependency reinstalls.** Each work tree is a fresh checkout. The setup script installs node_modules up front so the agent doesn't waste time during validation.
3. **Database isolation.** Different feature branches will mutate the same database and break each other. Solution: Neon database branching — each work tree gets its own branch carrying production data forward, isolated from others. SQLite per work tree is the free local alternative.
4. **Token blowout.** Comprehensive validation costs tokens. Manage it by switching models per task — Haiku/Sonnet for cheap/fast steps (codebase analysis, web research, code review), reserve top-tier reasoning for the implementation. `/model` in Claude Code; sub-agents and skills can also be invoked with a specific model.
5. **PR pileup (the human bottleneck).** When you start spending time fixing the same kinds of issues, that's the signal to invest in the AI layer (rules, commands, skills, validation steps) — the self-healing pillar.

## Key Exact Extracts
> **[00:50]** "We need a system especially centered around work trees."

> **[04:14]** "Once we have the bug fix or feature scoped out in an issue or ticket, then we're going to go through the stages of planning, building, and validating, but each one of the coding agents working with their own local copy of the codebase so they aren't overwriting each other's changes. That is what git work trees gives us."

> **[10:08]** "There is so much bias that the agent builds up within the conversation that if you tell it to review the code in the same context window, it's like asking a kid to grade their own homework."

> **[13:38]** "Whenever we encounter a bug in a pull request, we don't just fix the bug and move on, but we fix the underlying system that allowed for the bug."

> **[15:43]** "It's very easy to, you know, look back retroactively and see if there's anything in our process that needs to be adjusted because we deviate from our plans frequently."

> **[16:30]** "Static code analysis is not enough. You need your agents to actually start your application and use it as a user would to make sure you're really ready for the pull request merge."

> **[18:10]** "We need a work tree for the codebase, but we need something like a work tree for the database as well."

> **[22:46]** "Anytime that you feel like you're slowing down on these code reviews and you just want to get over to that next batch of issues, if there's any issues that come up where it feels like you're spending a lot of time fixing, that is your signal to do the self-healing layer."
