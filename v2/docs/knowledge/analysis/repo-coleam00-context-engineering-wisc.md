# coleam00/context-engineering-intro — WISC framework

## What it is

A practical, opinionated framework for managing AI-assistant context inside a coding session. Lives at `coleam00/context-engineering-intro/use-cases/ai-coding-wisc-framework/`. The parent repo is the popular "Context Engineering Template" (CLAUDE.md + INITIAL.md + PRPs workflow); this sub-folder is the newer, more sophisticated iteration that evolves the template into a four-strategy operational framework with concrete slash commands, a 3-tier context loading system, and example rule/doc files extracted from a real production codebase ("Archon").

It is explicitly built for Claude Code but designed so the patterns transfer to any AI coding tool. The repo positions context engineering as "10x better than prompt engineering and 100x better than vibe coding."

## Author / origin

- **Author:** Cole Medin (`coleam00` on GitHub) — runs the Dynamous AI Mastery community and a popular YouTube channel on AI engineering. The original "Context Engineering Intro" repo is one of the most-starred Claude Code resources on GitHub.
- **Origin / inspiration:** Anthropic's "Effective Context Engineering for AI Agents" article (the four strategies are lifted directly from there) plus Anthropic's "Effective Harnesses for Long-Running Agents." Also references Martin Fowler's knowledge-priming article, Chroma's "Context Rot" research, and GitHub's Spec Kit.
- **Date:** WISC sub-folder is from late 2025 / early 2026, post-dating the original PRP-based template.

## Core ideas (what does WISC stand for?)

WISC is an acronym for the four context engineering strategies, **stated in priority order**:

- **W — Write**: Externalize the agent's memory to files so it survives context resets. Specs, plans, handoff docs, and enriched commits are all "memory writes."
- **I — Isolate**: Use sub-agents to keep research noise out of the main session. Plans spawn parallel research sub-agents; primes use focused exploration; the "scout pattern" sends a sub-agent to read a doc's header before deciding to load it.
- **S — Select**: Load only the context you need for the current task, not everything. Realised through path-scoped rules that auto-load and primes that explore only one subsystem.
- **C — Compress**: When sessions run long, compress with focus or hand off to a fresh session.

The README explicitly notes: "Write and Isolate have the most impact, Select is the force multiplier, and Compress is the safety net."

The framework's central thesis: *most agent failures aren't model failures, they're context failures*. WISC is the operational discipline for managing context as a first-class engineering concern.

### The 3-Tier Context System

Layered "progressive disclosure" of context:

- **Tier 1 — Global Rules (`CLAUDE.md`)**: Always loaded. Hard cap of ~500 lines. Rule of thumb quoted from the README: *"If removing a line wouldn't cause the AI to make mistakes, cut it."*
- **Tier 2 — On-Demand Rules (`.claude/rules/*.md`)**: Auto-loaded based on which files the agent is touching, via YAML frontmatter:

  ```yaml
  ---
  paths:
    - "**/*.test.ts"
    - "**/*.spec.ts"
  ---
  ```
- **Tier 3 — Reference Docs (`.claude/docs/*.md`)**: 200-500 line reference guides. NOT auto-loaded. Each starts with a header block describing purpose / when-to-use / size, so a scout sub-agent can decide whether to load it. Example header from `architecture-deep-dive.md`:

  ```markdown
  > **Purpose**: End-to-end flow traces across the entire Archon system with file:line references.
  > **When to use**: Understanding how data flows between packages, debugging cross-system issues, onboarding.
  > **Size**: ~500 lines — use a scout sub-agent to check relevance before loading.
  ```

## Specific patterns or files worth borrowing

### File structure

```
use-cases/ai-coding-wisc-framework/
├── README.md
├── WISCFrameworkForAICoding.png
└── .claude/
    ├── commands/
    │   ├── prime.md, prime-backend.md, prime-frontend.md,
    │   ├── prime-workflows.md, prime-isolation.md
    │   ├── plan-feature.md
    │   ├── execute.md
    │   ├── handoff.md
    │   └── commit.md
    ├── rules-example/
    │   ├── testing.md, web-frontend.md, database.md,
    │   ├── orchestrator.md, workflows.md, adapters.md,
    │   ├── isolation.md, server-api.md, cli.md
    └── docs-example/
        ├── architecture-deep-dive.md (~324 lines)
        ├── workflow-yaml-reference.md (~309 lines)
        ├── adapter-implementation-guide.md (~248 lines)
        └── isolation-and-worktree-guide.md (~231 lines)
```

### Strategy-to-command mapping

```
WRITE     /plan-feature  /execute  /handoff  /commit
           (specs)        (specs)   (progress) (git memory)
ISOLATE   /plan-feature spawns research sub-agents
          /prime-* commands use focused exploration
          Scout pattern: sub-agents read doc headers first
SELECT    /prime-*   .claude/rules/*.md   .claude/docs/*.md
          (focused)  (auto-loaded)         (on-demand via scouts)
COMPRESS  /handoff   /compact (built-in)
```

### `/plan-feature` — five-phase planning prompt

Frontmatter: `description: Create a comprehensive implementation plan` + `argument-hint: <feature-name-or-description>`. Saves output to `.claude/archon/plans/{kebab-case-name}.md`.

Five phases:
1. **Feature Understanding** — restate problem, success criteria, scope, package impact, interface changes.
2. **Codebase Intelligence** — explicitly spawns 4 parallel sub-agents: (A) affected-package deep-dive, (B) interface and type contracts, (C) test patterns, (D) related prior work (`git log`).
3. **External Research (if needed)** — web search for SDK docs, gotchas, community patterns.
4. **Strategic Thinking** — architecture decisions, interface design, test isolation strategy, lint compliance, rollback plan.
5. **Plan Generation** — writes a plan file with sections: Overview, Success Criteria, Affected Packages, Architecture Notes, Implementation Tasks (numbered, with file path + Create/Modify/Delete + dependency), Validation Steps, Rollback Notes. Includes "Task Ordering Rules" (deps first, group by package, schema before types before impl before tests) and a "Prohibited Patterns" callout list.

### `/execute` — read-plan-then-implement prompt

Frontmatter: `argument-hint: <path-to-plan.md>`. Six explicit steps: (1) read entire plan first, (2) verify clean working tree, (3) execute tasks in dependency order with read-before-edit and incremental type-check, (4) per-package validation, (5) full validation (`bun run validate`), (6) structured completion report (Tasks Completed / Files Created / Files Modified / Validation Results / Manual Verification / Notes).

Crucial design choice: the plan itself is the only context the execute session needs — "fresh session, no planning conversation baggage."

### `/handoff` — Write + Compress combined

Output template (worth borrowing nearly verbatim):

```markdown
# Handoff: [Brief Task Description]
**Date:** … **Branch:** … **Last Commit:** …
## Goal
## Completed
## In Progress / Next Steps
## Key Decisions   ← WHY, not just what
## Dead Ends (Don't Repeat These)   ← prevents wasted re-exploration
## Files Changed
## Current State (tests, type-check, lint, build, manual verify)
## Context for Next Session
**Recommended first action:** [exact command]
```

Quality criteria: "Let a fresh agent continue without asking any clarifying questions" and "Be under 100 lines (concise, not comprehensive — link to files rather than duplicating content)."

### `/commit` — enriched commits with `Context:` section

Conventional tag prefix (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`, `perf:`) with monorepo scope `feat(workflows): …`. Body explains WHY, not what. The novel piece is the `Context:` section that logs changes to AI assets:

```
feat(orchestrator): add retry logic for session recovery

Added exponential backoff when SDK subprocess crashes mid-session.

Context:
- Updated .claude/rules/orchestrator.md with retry conventions
- Added .claude/commands/debug-session.md for session state inspection
- Surfaced issue: mock.module() in retry tests needs isolated batch
```

Rationale quoted from the file: *"Your git log is long-term memory. Future agents and sessions use `git log` to understand project history. If context changes aren't captured in commits, the AI layer's evolution becomes invisible."*

### `/prime` and `/prime-*` — focused codebase priming

`/prime` reads core docs, key entry points, and the dependency graph; outputs a "concise summary (under 300 words) covering Project Overview / Architecture / Current State." The variants (`/prime-backend`, `/prime-frontend`, `/prime-workflows`, `/prime-isolation`) prime only the relevant subsystem instead of the whole codebase, dropping a 30K-token overview to a few thousand.

Uses Claude Code's `!` directive to run shell commands inline at slash-command invocation:
```
List the monorepo packages:
!`ls packages/`
```

### Rule frontmatter pattern

Every rule file in `rules-example/` starts with:
```yaml
---
paths:
  - "**/*.test.ts"
  - "**/*.spec.ts"
---
```
This is the auto-load trigger — the harness loads the rule when the agent reads/edits a matching path. Each rule then drills into very specific, hard-won conventions (e.g., `testing.md` opens with "CRITICAL: mock.module() Pollution Rules" with a linked Bun bug). They are short (~3-6KB), prescriptive, and full of code examples.

## Direct relevance to specflow's goals

This is the single most relevant repo for specflow's context-engineering layer. specflow already has a plugin structure (`plugins/specflow/.claude-plugin/`), a `knowledge/` folder, slash commands, and a release discipline; WISC has the missing operational pieces. Concrete borrowable patterns mapped to specflow's three north-stars:

### (1) More productive

- **Adopt path-scoped rules with `paths:` frontmatter.** Today, specflow's rules likely live in plugin command files or root `CLAUDE.md`. Migrating per-domain rules (testing, releases, plugin packaging, marketplace.json discipline) into `.claude/rules/*.md` with auto-load globs cuts always-on tokens and makes the docs creator's writing-related rules invisible to the dev team and vice versa. This is the single biggest "fewer tokens in the wrong context" lever.
- **Plug in `/prime` variants.** A `/prime-plugin`, `/prime-knowledge`, `/prime-release` set lets the dev team and the docs creator each load only what they need at session start — directly addresses the "new to AI coding" team's tendency to dump everything into context.
- **Use the inline shell directive (`!`).** specflow slash commands can use `` !`...` `` to fold dynamic state (current branch, marketplace version, last release) into the prompt without an extra tool call.

### (2) Better product in shorter time

- **Spec-driven `/plan-feature` → `/execute` two-stage workflow.** This is the most transformative single pattern. The dev team gets a ceremony: feature description → plan file (reviewed by humans before implementation) → fresh-session execution from the plan. It bridges the human-workflow gap that the user's `feedback_no_premature_pipeline_ctas.md` specifically calls out — the plan file IS the human review checkpoint between research and implementation.
- **The 4-parallel-subagent recipe inside `/plan-feature`** (affected-package / interface contracts / test patterns / prior work via `git log`) is a drop-in template. It's both faster than serial exploration and produces better plans because the four perspectives synthesize into a single document.
- **Reference docs with scout-able headers.** specflow's `knowledge/` directory is currently flat. Add the three-line `> Purpose / When to use / Size` header at the top of every doc, and a scout sub-agent can decide whether to load — turning the whole knowledge folder into Tier 3 storage that doesn't bloat sessions.

### (3) Fewer errors

- **`/handoff` with the "Dead Ends" and "Key Decisions (with WHY)" sections.** Both demographics (docs creator, dev team) suffer the same loss of state across sessions. A 100-line `HANDOFF.md` with explicit dead-ends prevents the most common failure mode for AI-coding-novice teams: re-exploring approaches that already failed. This pairs perfectly with specflow's "no premature pipeline CTAs" feedback rule.
- **Enriched `/commit` with `Context:` section.** specflow's CLAUDE.md already enforces release-version sync; extending commits to log `.claude/` changes alongside code changes makes the AI layer's evolution auditable. Future agents reading `git log` understand WHY a rule exists. This directly reduces a class of error where rules drift silently from the codebase.
- **The "If removing a line wouldn't cause the AI to make mistakes, cut it" CLAUDE.md test.** This is a calibration heuristic specflow can adopt today as a quality gate when reviewing changes to the project CLAUDE.md.
- **Rule files in `rules-example/`** are usable templates as-is — `testing.md`, `database.md`, etc., demonstrate how to compress hard-won lessons into 3-6KB of tightly scoped conventions with code examples and external references (e.g., bug links). The `testing.md` `mock.module()` warnings show the "convention with citation" pattern that turns institutional memory into auto-loaded guardrails.

### Specific suggested adaptations for specflow

- Rename: WISC's "Archon" terminology should be stripped and replaced with specflow language; the framework structure carries over cleanly.
- specflow already has `knowledge/`; treat it as the Tier-3 docs layer and add scout-headers to every file.
- Add `.claude/rules/` to the plugin and start with two files: `releases.md` (path-scoped to `plugin.json`/`marketplace.json`) and `branding.md` (path-scoped to commit messages + PR bodies — though path-globs over commit/PR text isn't a thing, so this one belongs in CLAUDE.md or in a hook).
- Pair `/plan-feature` with the existing `agent-teams:team-feature` skill — the WISC plan becomes the "spec" the team uses for parallel decomposition.

## Cross-references

- **Anthropic, "Effective Context Engineering for AI Agents"** — direct source of the four W/I/S/C strategies. The repo's framework is essentially a productisation of this article.
- **Anthropic, "Effective Harnesses for Long-Running Agents"** — companion piece on session lifecycle, referenced in WISC's resources list.
- **Martin Fowler, "Knowledge Priming for AI Agents"** — independent treatment of priming patterns; aligns with the `/prime-*` family.
- **alexop.dev, "Stop Bloating Your Claude.md: Progressive Disclosure for AI Coding Tools"** — direct inspiration for the 3-tier context system.
- **Chroma research, "Context Rot"** — empirical motivation for compression and selective loading.
- **GitHub Spec Kit** — listed as related prior art; uses similar plan-then-execute spec discipline.
- **coleam00/context-engineering-intro (parent repo, root README)** — the original "PRP" (Product Requirements Prompt) pattern: `/generate-prp INITIAL.md` → `/execute-prp PRPs/feature.md`. WISC's `/plan-feature` → `/execute` is the evolved successor; specflow can borrow either or both. The parent README's framing ("Context Engineering vs Prompt Engineering vs Vibe Coding") is the cleanest one-page pitch for the discipline.
- **specflow internal:** the user's memory file `feedback_no_premature_pipeline_ctas.md` (don't add forward references when there's a human workflow gap) — reinforces why the WISC plan-file-as-handoff pattern fits specflow: the plan IS the gap, not an unsolicited next-step suggestion.
- **specflow internal:** existing `agent-teams:*` skills (parallel feature development, multi-reviewer patterns) — natural integration points for the `/plan-feature` sub-agent spawning step.
