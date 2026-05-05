# coleam00/Archon

## What it is

Archon is an **open-source harness builder for AI coding** — a workflow engine that turns AI development processes into deterministic, repeatable YAML workflows. Tagline: "Like Dockerfiles for infrastructure and GitHub Actions for CI/CD — Archon does for AI coding workflows."

Workflows are DAGs of nodes. Each node is either deterministic (`bash:` script, git op, test runner) or AI-driven (`prompt:` or `command:`, optionally inside a `loop:` with a stop condition). Each AI node can request `context: fresh` to escape context rot, choose a model (`opus[1m]`, `sonnet`, `haiku`), and emit structured `output_format` JSON. Every workflow run executes inside its own git worktree so multiple runs can happen in parallel without conflicts.

The product ships:
- 17 default workflows (idea-to-PR, fix-github-issue, PIV loop, smart PR review, refactor-safely, architect, etc.)
- A reusable command library (`.archon/commands/`) — markdown prompt fragments composed into workflows
- CLI, web dashboard ("Mission Control"), Slack/Telegram/Discord/GitHub adapters
- A SQLite/Postgres backend tracking codebases, conversations, sessions, workflow runs, isolation envs

## Author / origin
- **Author:** Cole Medin (`@coleam00`) — "Generative AI specialist," runs **Dynamous** (an AI consultancy / community). Bio: AI Agents, RAG, local AI, gen-AI libraries. 6,772 GitHub followers.
- **URL:** https://github.com/coleam00/Archon — docs at https://archon.diy
- **Last activity:** pushed 2026-04-29 (yesterday); updated 2026-04-30 (today). Very active.
- **Stars:** 20,210 stars / 3,092 forks. License MIT. TypeScript + Bun.
- **Note:** v1 of Archon was a Python task-management + RAG system; that's now archived on `archive/v1-task-management-rag`. Current v2 is a complete rewrite as a workflow engine.

## Core ideas

1. **Determinism via structure, intelligence in the gaps.** The shape of the work (phases, dependencies, gates, artifacts) is owned by the human in YAML; the LLM only fills in the cognitive work at each node. "Same workflow, same sequence, every time." This directly attacks the "every AI run is different" problem.
2. **DAG of nodes with explicit dependencies.** `depends_on:` produces a directed acyclic graph. Some nodes fan out in parallel (5 review agents), then a `synthesize` node joins them with `trigger_rule: one_success`.
3. **Loop nodes with stop conditions.** `loop:` runs an AI node repeatedly with conditions like `until: ALL_TASKS_COMPLETE` or `until: APPROVED`, plus `interactive: true` for human-in-the-loop pauses and `fresh_context: true` for clean state per iteration.
4. **Context isolation per node.** `context: fresh` on each node prevents context rot across long workflows — every phase starts clean and only loads the artifacts it needs.
5. **Worktree-per-run isolation.** Every workflow run gets its own git worktree, enabling parallel "fire-and-forget" runs and clean rollback.
6. **Default-but-overridable.** Bundled workflows live in `.archon/workflows/defaults/`; same-named files in the user's repo override them. Workflows and commands are committed to the repo so the whole team runs the same process.
7. **Composable command library.** Workflows reference reusable `command:` blocks (e.g., `archon-create-plan`, `archon-validate`, `archon-pr-review-scope`) — like functions for prompts.
8. **Multi-agent code review by dimension.** Reviews split into specialized agents (code, error-handling, test-coverage, comment-quality, docs-impact) running in parallel, then a synthesizer reconciles. Self-fixes findings rather than just reporting.
9. **PIV loop (Plan-Implement-Validate)** is treated as the foundational AI coding methodology, with arbitrary-round explore and review phases bracketing an autonomous implement phase.
10. **Workflow router.** The agent picks the right workflow from a description ("use archon to fix issue #42") rather than the user memorizing names.
11. **Privacy-first telemetry.** Single anonymous `workflow_invoked` event with workflow name/description/platform/version + random UUID. Nothing else. Honors `DO_NOT_TRACK=1`.

## Specific patterns or files worth borrowing

- **`.archon/workflows/defaults/archon-idea-to-pr.yaml`** — clean reference for an 8-phase DAG: create-plan → setup → confirm → implement (Opus 1M) → validate → finalize-pr → review (5 parallel agents) → synthesize → fix → summary. Almost every node uses `context: fresh`. Phase comments are big banner blocks — readable.
- **`.archon/workflows/defaults/archon-fix-github-issue.yaml`** — shows the **classify-then-route** pattern: a Haiku-model classifier with strict `output_format` (JSON enum) decides bug vs feature, then downstream nodes branch (`investigate` for bugs, `plan` for features). Cheap model for routing, expensive model for the work.
- **`.archon/workflows/defaults/archon-piv-loop.yaml`** — explicitly interactive loop nodes, with two-mode prompts ("if first iteration … / if subsequent iteration …") and `$LOOP_USER_INPUT` available inside the loop. Excellent template for human-in-the-loop with arbitrary rounds.
- **`.archon/commands/`** — reusable prompt files (markdown). Workflows compose them by name. This is the "skill"-like layer beneath workflows.
- **`output_format: { type: object, properties, required }`** for AI nodes that need structured output downstream — eliminates regex parsing.
- **`trigger_rule: one_success`** on join nodes — workflow continues even if some parallel branches fail.
- **Inline `bash:` verification gates** between AI phases, e.g., the `verify-pr-base` node that re-targets a PR if its base branch drifted. Cheap, deterministic safety nets.
- **README "What Can You Automate?" table** — a one-line description per workflow, written so a non-expert can pick the right one. Good model for specflow's user-facing skill index.
- **Telemetry section of the README** — exemplary disclosure (what's collected, what's NOT, opt-out vars). Good template for any specflow telemetry the team adds later.

## Direct relevance to specflow's goals

specflow is a Claude Code plugin used by a documentation creator + a dev team new to AI. North-star goals: more productive, better product faster, fewer errors. Archon maps almost 1:1.

**(1) More productive** — Archon's premise is "fire-and-forget" workflows: kick off a feature, walk away, come back to a reviewed PR. Worktree isolation lets multiple runs go in parallel. specflow can borrow the worktree-per-run pattern and the workflow router so users don't have to remember which skill to invoke.

**(2) Better product in shorter time** — The multi-agent review pattern (5 parallel reviewers across code/errors/tests/comments/docs, then synthesis + auto-fix) is a stronger quality bar than a single review pass and runs in roughly the same wall-clock time. specflow's existing test/QA skills could be split into parallel dimension-specific reviewers with a synthesis step.

**(3) Fewer errors** — Archon's core thesis is that determinism comes from **structure owned by the human**, not from prompting harder. specflow's pipeline already has phase ordering; adopting Archon's vocabulary (`context: fresh`, `depends_on`, `trigger_rule`, `loop … until`, `interactive: true`) would make the existing pipeline more legible and robust. The `bash:` verification gates between AI phases (like `verify-pr-base`) are cheap insurance against AI drift — specflow could add similar checks (e.g., verifying plan files exist, branch names match conventions) without spending tokens.

For the **documentation-creator persona** specifically, the `archon-create-issue` workflow (26KB!) and the dedicated `docs-impact-agent` show that docs-aware checks deserve their own first-class node, not a bullet inside a generic review.

The "no premature pipeline CTAs" feedback in user memory is consistent with Archon's `interactive: true` / approval-loop pattern — Archon explicitly pauses for human confirmation rather than racing forward.

## Cross-references

- **Internal:** specflow's existing skills under `plugins/specflow/` and the readiness-check / QA-label / release-rule machinery added in recent commits (25ca296, c3d4f01, 89e994d, 31f9b88) are the analog of Archon "commands." A specflow workflow file (YAML or markdown DAG) that strings these together would be the analog of an Archon workflow.
- **External — same author:** Cole Medin runs Dynamous (https://dynamous.ai-ish — see `dynamous.youcanbook.me` in bio) and produces extensive YouTube content on AI coding patterns. Several of the YouTube transcripts already in `knowledge/` are likely his — worth cross-checking.
- **External — Archon docs to follow up on (one level deep, not yet fetched):**
  - https://archon.diy/guides/authoring-workflows/ — full YAML schema reference
  - https://archon.diy/guides/authoring-commands/ — command authoring spec
  - https://archon.diy/book/ — "The Book of Archon," 10-chapter narrative tutorial; likely the best single source for the design rationale
  - https://archon.diy/reference/architecture/ — system internals
- **Archived v1 branch** `archive/v1-task-management-rag` — if specflow ever wants RAG over a docs corpus, the old Python implementation is a reference point.
- **Comparable concepts:** n8n (workflow engine, called out in the README), GitHub Actions (CI DAG), Dockerfiles (declarative determinism). Archon explicitly positions itself in this lineage.
