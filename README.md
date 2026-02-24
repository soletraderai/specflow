# Specflow

A Claude Code plugin for structured product requirements workflows — create PRDs, break them into tasks, and export to Linear.

## Overview

Specflow guides you through a complete product planning pipeline:

1. **Prime your session** — Load project context (config, specflow state, codebase config, git status) so every conversation starts informed
2. **Create a PRD** — Structured discovery interview that produces a comprehensive Product Requirements Document
3. **Break into tasks** — Decompose the PRD into vertical-slice issues with dependency tracking, time estimates, and acceptance criteria
4. **Export to Linear** — Push approved tasks to Linear as a project with properly sequenced issues and blocker relationships

## Skills

| Skill | Description |
|-------|-------------|
| `/specflow:setup` | Set up specflow: create directories, verify Linear MCP, detect tech stack, install specialist agents |
| `/specflow:prime` | Load project context into the session — config, specflow state, codebase config, git status |
| `/specflow:prd` | Create a structured PRD through guided discovery |
| `/specflow:task` | Break a PRD into actionable tasks organized as vertical slices |
| `/specflow:linear` | Export an approved task review to Linear as a project with issues |

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- Run `/specflow:setup` after installation to configure Linear MCP, detect your tech stack, and install relevant specialist agents

## Installation

```bash
claude plugin add soletraderai/specflow
```

## Usage

### 0. Set up specflow (recommended)

```
/specflow:setup
```

Specflow creates the required directories, checks your Linear MCP configuration, scans your codebase to detect the tech stack, and installs relevant specialist agents from the `wshobson/agents` marketplace. Configuration is saved to `docs/specflow/config.json` for use by other skills.

### 1. Prime your session (optional)

```
/specflow:prime
```

Loads project context into the conversation — specflow config, recent PRD/task state, codebase configuration files (based on detected tech stack), and git status. Runs at the start of a session so you don't have to re-explain your project. Read-only, no side effects.

### 2. Write a PRD

```
/specflow:prd
```

Specflow interviews you about the problem, explores your codebase for context, then produces a PRD saved to `docs/specflow/prd/`.

### 3. Break into tasks

```
/specflow:task
```

Reads the PRD, explores relevant code, and generates a task review document in `docs/specflow/task/` with vertical-slice issues, estimates, and dependency ordering. Review and adjust tasks before approving.

### 4. Export to Linear

```
/specflow:linear
```

Takes the approved task review and creates a Linear project with issues in dependency order, complete with `blockedBy` relationships and hour estimates.

## How It Works

Specflow uses a file-based workflow:

```
docs/specflow/prd/       → PRD documents (Markdown with YAML frontmatter)
docs/specflow/task/       → Task review documents (parsed during Linear export)
```

Each step reads from the previous step's output, so you always have a reviewable artifact before anything is pushed to Linear.

## License

[MIT](LICENSE)
