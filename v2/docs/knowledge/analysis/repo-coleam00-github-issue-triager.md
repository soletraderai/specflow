# coleam00/GitHubIssueTriager

## What it is

A Next.js 15 + React 19 + TypeScript dashboard that ingests every issue from a target GitHub repo (via `gh`), classifies each one (category/priority/complexity), embeds it into Neon Postgres `pgvector` (HNSW cosine), and generates an on-demand markdown implementation plan plus a "dispatch" record with a generated branch name. Schema lives in `migrations/001_init.sql`: `issues`, `classifications` (append-only — latest row wins in UI), `similar_issues` (vector(1536)), `plans`, `runs`. Endpoints are all `POST` under `src/app/api/{sync,classify,similar,plan,dispatch}/[id]/route.ts` keyed by GitHub issue number, not DB PK. Has a rule-based classifier and deterministic-hash embedding fallback so the whole pipeline runs without `OPENAI_API_KEY`.

But the README is explicit that the app is a *vehicle* for the real product: a parallel-AI-coding lab. The repo's title is "a lab for parallel AI coding."

## Author / origin

Cole Medin (`coleam00`) — known for AI coding content, "10x AI Coding Playbook" diagram is committed at the root (`10xAICodingPlaybookDiagram.png`). Has a `.archon/` directory (his own Archon agent platform) alongside `.claude/`. License MIT. The README is unusually well-written and reads like a teaching artifact — the explicit framing is "point your coding agent at this repo and tell it to port these patterns into your setup."

## Core ideas

1. **Worktree-per-task with full isolation.** `scripts/w.sh` (and Windows wrapper `w.ps1`) provisions in one command: a sibling worktree dir, a new git branch, a Neon Postgres branch (copy-on-write, sub-second), a `.env` with `DATABASE_URL` rewritten to that branch, and a linked `pnpm install`. `w.ps1 issue-1 open` even launches a Claude Code session inside it with `--dangerously-skip-permissions`. `w.ps1 issue-1 rm` tears all of it down (worktree, branch, Neon branch).
2. **Database branching per agent.** Three parallel Claude sessions can migrate, seed, drop tables, with zero collisions. The DB branch *is* the unit of isolation, not just a git branch. (`scripts/worktree-setup.sh`, `worktree-teardown.sh`.)
3. **Stateless deterministic port assignment.** `scripts/assign-port.ts` MD5-hashes `process.cwd()`, takes 4 bytes mod 100, plus 4100. No registry, no race conditions on teardown. Main repo is special-cased to 4000. ~15 lines.
4. **Plan / Implement / Validate are SEPARATE Claude sessions.** The thesis: same-session review catches a meaningfully smaller fraction of bugs than fresh-context review. `/plan` writes to `.agents/plans/{name}.plan.md`; the implementer reads that file cold; the validator reads the diff (and optionally plan) cold. Isolation defeats sycophancy and prevents the implementer from "defending decisions it didn't witness."
5. **Fan-out PR review with specialized subagents.** `.claude/commands/review-pr.md` dispatches up to four reviewers in one `Task` batch — `code-reviewer`, `silent-failure-hunter`, `pr-test-analyzer`, `code-simplifier` — each with its own system prompt under `.claude/agents/`. Aggregator buckets findings into Critical / Important / Suggestions / Verdict. Reviewers must reference `file:line`.
6. **Cross-provider review with feedback loop into CLAUDE.md.** `.claude/commands/cross-review.md` layers Codex (GPT) on top via the Codex plugin's `/codex:adversarial-review`. The interesting move: findings Codex catches that Claude missed become *new CLAUDE.md rules* — the blind spot is patched at the prompt layer rather than rediscovered. The command is a two-phase human-in-the-loop because the Codex plugin uses `disable-model-invocation: true` (so the Claude agent can't auto-invoke it).
7. **Graceful degradation as a first-class design choice.** Missing `OPENAI_API_KEY` triggers a rule-based classifier and hash-based embedding — the README and CLAUDE.md both treat the fallback as "the happy path in tests." CLAUDE.md explicitly says "don't wrap those call sites in try/catch."

## Specific patterns or files worth borrowing

- **`.claude/commands/review-pr.md`** — the parallel fan-out template. Selects reviewers conditionally based on what changed (errors → silent-failure-hunter; tests → pr-test-analyzer). One `Task` batch. Final aggregation is a fixed markdown skeleton with a hard verdict (APPROVE / APPROVE_WITH_CHANGES / REQUEST_CHANGES). "Do not apply fixes automatically" is stated explicitly. ~95 lines.
- **`.claude/commands/cross-review.md`** — two-phase command pattern for working *around* a plugin policy constraint. Phase 1: tell the user the exact verbatim command to run. Phase 2: re-invocation triggers the comparison. Includes the brilliant detail of **"draft a one-line candidate CLAUDE.md rule"** for each cross-provider blind-spot win.
- **`CLAUDE.md`** itself — short, sectioned `Stack / Run / Where things live / Non-obvious`. The "Non-obvious" section is gold: it captures contracts that aren't visible from reading code (e.g. "the silent fallback IS the happy path; don't add try/catch"; "classifications is one-to-many, latest row wins; if you add a field, write a view, never mutate history").
- **`scripts/assign-port.ts`** — 15-line stateless coordination via hashing. Generalizable far beyond ports (any time you'd reach for a registry, ask: can a hash do it?).
- **`scripts/w.sh` + `worktree-setup.sh` + `worktree-teardown.sh`** — single-command lifecycle for an isolated environment. The composability of (git worktree) × (Neon branch) × (`.env` rewrite) × (linked pnpm install) is what makes parallel agents tractable.
- **The "different model, different blind spots" mental model** — applies anywhere there's a review or critique step. Two cheap reviewers from different distributions beat one expensive reviewer from one distribution.

## Direct relevance to specflow's goals

specflow already has a `/test` skill, a release rule, and `feedback_no_premature_pipeline_ctas.md`. This repo offers concrete patterns for the misc-task / triage gap:

- **More productive (G1).** A specflow `/triage-misc` skill could mirror the `classify → similar → plan → dispatch` chain at small scale: an inbox of "stuff to do" gets classified (skill-build vs bug vs doc vs research), checked against a similarity index of past tasks (so the dev team doesn't re-solve solved problems), and emits a plan file. The dispatch step (writing a `runs` row with a generated branch name) maps cleanly onto specflow's plugin-versioning workflow.
- **Better product in shorter time (G2).** The plan/implement/validate split is directly portable: specflow could ship a paired `/plan` (writes `.agents/plans/{name}.plan.md`, no code edits) and `/implement` (reads plan from disk, no chat history). The doc-creator persona becomes the planner; the AI-new dev team becomes the implementer following a written-down spec. This is the cleanest answer to the "how do we keep AI-new devs from drifting" question — the plan IS the spec, and it's reviewable by the documentation creator before any code is written.
- **Fewer errors (G3).** Three concrete loss-reduction patterns:
  - The fresh-context fan-out review (`review-pr.md`) — drop straight into specflow's `/test` or as a separate `/review` skill.
  - The cross-provider blind-spot loop (`cross-review.md`) — run something Codex-like, harvest disagreements, promote them to CLAUDE.md rules. This is the *self-improving rules* loop specflow is missing.
  - "Plan-only" mode — Claude writes plans without touching code. Eliminates an entire class of "the model edited the wrong thing during research" errors that AI-new devs especially suffer from.
- **Worktree isolation** is overkill for specflow today (no DB; specflow IS a plugin), but the *pattern* of "one command provisions a complete isolated env, one command tears it down" is the right shape for any future per-task experimentation.
- **Fallback-as-happy-path** is the right framing for specflow when API/model availability degrades — write the fallback to be a real path, document it in CLAUDE.md as such, and don't try/catch around it.

## Cross-references

- specflow `CLAUDE.md` — release-version sync rule (mirrors this repo's discipline of capturing "Non-obvious" contracts in CLAUDE.md).
- specflow `feedback_no_premature_pipeline_ctas.md` — aligns with this repo's choice to keep `/review-pr` from auto-applying fixes ("Do not apply fixes automatically in this command").
- specflow `/test` skill — direct precedent for a fresh-context fan-out reviewer skill modeled on `review-pr.md`.
- specflow `repo.txt` (sibling research targets) — when reviewing the other 5 repos, look specifically for: (a) parallel-agent isolation patterns, (b) plan-on-disk handoff patterns, (c) self-updating rule files. This repo sets the bar.
- `10xAICodingPlaybookDiagram.png` at the root of this repo — worth fetching separately; it's likely the canonical visual for Cole Medin's playbook and may map onto specflow's pipeline directly.
