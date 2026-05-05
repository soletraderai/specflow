---
name: specflow:prime
description: Prime the codebase context for an upcoming piece of work — load the relevant files into the conversation so downstream skills can reason about them.
status: shipped
phase: 1
requires: [docs/specflow/admin/environment.json]
produces: []
eval: working set of files relevant to the target feature is loaded; downstream skill (typically prd, design, or develop) can reference them by path without re-reading.
---

# specflow:prime

Prime the codebase context for an upcoming piece of work. Often the first call before `specflow:prd`, `specflow:design`, or `specflow:develop`.

**Triggers:** "prime for X", "/specflow:prime".

**v2 changes from v1:** none for the core flow. Now reads `admin/environment.json` to know which agents and CLIs are available, so primed context can include relevant agent prompts when downstream skills will hand off.

**Verify steps:**
1. Working set loaded → conversation context references the target files.
2. No silent over-loading → primed file count is bounded; warns if the working set exceeds a configurable cap.

**Reference:** v1 SKILL.md at `plugins/specflow/skills/prime/SKILL.md` for the existing implementation that v2 builds on.
