# openai/codex-plugin-cc

Source: https://github.com/openai/codex-plugin-cc
Version analyzed: 1.0.4 (marketplace + plugin.json both report 1.0.4)
License: present in repo (`LICENSE`, `NOTICE`)

## What it is

Official OpenAI plugin for Claude Code that lets a Claude Code session call the local Codex CLI to perform code reviews and delegate tasks. It is a **bridge plugin** — not a re-implementation of Codex. It wraps the [Codex app server](https://developers.openai.com/codex/app-server) and shells out to the global `codex` binary already installed on the user's machine.

What you get from a single install:
- `/codex:review` — read-only Codex review of working tree or branch.
- `/codex:adversarial-review` — steerable challenge review that questions implementation, design, tradeoffs, and assumptions. Accepts free-text focus.
- `/codex:rescue` — delegates an investigation, fix, or follow-up implementation task to Codex via the `codex:codex-rescue` subagent.
- `/codex:status`, `/codex:result`, `/codex:cancel` — manage background Codex jobs (list, fetch final output, cancel).
- `/codex:setup` — verify Codex install/auth and toggle the optional stop-time review gate.
- A `codex:codex-rescue` Sonnet subagent that is a pure forwarder.
- Three internal skills (`codex-cli-runtime`, `codex-result-handling`, `gpt-5-4-prompting`) marked `user-invocable: false`.
- `SessionStart`, `SessionEnd`, and `Stop` hooks (the Stop hook implements the optional review gate).

## Author / origin

- Owner: OpenAI (`openai-codex` marketplace, owner `OpenAI`).
- Distributed via Claude Code marketplace: `/plugin marketplace add openai/codex-plugin-cc` then `/plugin install codex@openai-codex`.
- Requirements: ChatGPT subscription (incl. Free) **or** OpenAI API key, plus Node.js 18.18+, plus a globally installed `codex` CLI (`npm install -g @openai/codex`). Auth handled by `codex login` (supports ChatGPT login, device-auth, and `--with-api-key`). The plugin uses local Codex CLI authentication — no separate account or runtime.

## Core ideas

1. **Thin bridge over the local Codex CLI.** Everything is a Bash call into `node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" <subcommand>`. The companion script is the single entry point for all subcommands (`setup`, `review`, `adversarial-review`, `task`, `status`, `result`, `cancel`, plus internal helpers like `task-resume-candidate --json`).

2. **The companion is an app-server client, not a CLI shell.** Internally it speaks the Codex app-server JSON-RPC protocol via `lib/app-server.mjs` and `lib/codex.mjs`. There is a broker (`scripts/app-server-broker.mjs`) that keeps a long-lived Codex app server hot and exposed via `BROKER_ENDPOINT_ENV`. Threads are started with `approvalPolicy: "never"`, `sandbox: "read-only"` by default; rescue runs flip to write-capable when `--write` is passed.

3. **Subagent-as-forwarder pattern.** `codex-rescue.md` is explicitly defined as "a thin forwarding wrapper." It is allowed exactly one Bash call (`codex-companion.mjs task ...`) and must return that stdout verbatim. It is not allowed to read files, summarize, or do its own analysis. This avoids "double-thinking" between Claude and Codex.

4. **Verbatim output discipline.** Every command's prompt instructs Claude to "return the command stdout verbatim." `result.md`, `status.md`, and `cancel.md` use the `!`-prefix bash form so the output is shown directly. The `codex-result-handling` skill enforces: present findings ordered by severity, preserve file paths/line numbers, **never auto-apply fixes after a review** ("CRITICAL: After presenting review findings, STOP. … You MUST explicitly ask the user which issues, if any, they want fixed").

5. **Background-by-default for non-trivial work.** `review.md` and `adversarial-review.md` use `git status --short` and `git diff --shortstat` to estimate scope; if more than ~1–2 files, they recommend `--background` via a single `AskUserQuestion`. Backgrounding is implemented with Claude Code's `Bash(..., run_in_background: true)`, while the companion script's own `--background`/`--wait` flags handle in-process job tracking.

6. **Job state lives on disk, scoped to workspace + Claude session.** `lib/state.mjs` writes JSON job records under `${CLAUDE_PLUGIN_DATA}/state/<workspace-slug>-<sha256-hash>/jobs/`. Jobs carry `sessionId` (from `CODEX_COMPANION_SESSION_ID` env var set by the SessionStart hook) so `/codex:status` can filter to the current session. Each job also writes a `.log` file used for the progress preview.

7. **Resume semantics encoded as routing flags, not text.** `--resume` → `task --resume-last`, `--fresh` → fresh `task`. The rescue command runs `task-resume-candidate --json` and uses `AskUserQuestion` to ask whether to continue the latest thread when the user is ambiguous.

8. **Structured review JSON.** `schemas/review-output.schema.json` constrains review output to `{verdict: approve|needs-attention, summary, findings[], next_steps[]}`, where each finding has `severity` (critical/high/medium/low), `file`, `line_start`, `line_end`, `confidence` (0–1), `recommendation`. The companion validates against this schema before rendering.

9. **Adversarial-review prompt template.** `prompts/adversarial-review.md` is an XML-tagged operator-style prompt with `<role>`, `<task>`, `<operating_stance>` (skeptical), `<attack_surface>` (auth, data loss, rollback, races, empty-state, schema drift, observability), `<review_method>`, `<finding_bar>`, `<structured_output_contract>`, `<grounding_rules>`, `<calibration_rules>`, `<final_check>`. Variables: `{{TARGET_LABEL}}`, `{{USER_FOCUS}}`, `{{REVIEW_INPUT}}`, `{{REVIEW_COLLECTION_GUIDANCE}}`.

10. **Optional Stop-hook review gate.** `hooks/hooks.json` registers a 900-second-timeout `Stop` hook that calls `stop-review-gate-hook.mjs`. When enabled (`/codex:setup --enable-review-gate`), Codex reviews the previous Claude turn; if it finds blocking issues, it returns `BLOCK: <reason>` and prevents stop. Prompt template (`prompts/stop-review-gate.md`) uses a `<compact_output_contract>` whose first line must be exactly `ALLOW: …` or `BLOCK: …`. Carries an explicit warning: can drain usage limits by triggering long Claude/Codex loops.

## Specific patterns or files worth borrowing

- **`agents/codex-rescue.md` as a template for "delegate to external reviewer" subagents.** Frontmatter declares `tools: Bash`, `skills: [codex-cli-runtime, gpt-5-4-prompting]`, and the body is a strict list of forwarding rules. Specflow can mirror this for a `codex-reviewer` subagent that delegates to `/codex:adversarial-review` rather than doing its own analysis.
- **`commands/rescue.md`** demonstrates how a slash command can invoke a subagent via `Agent` tool with `subagent_type: "codex:codex-rescue"`, plus the precise warning ("do not call `Skill(codex:rescue)` — that re-enters this command and hangs the session"). Useful precedent for specflow commands that fan out to specialist subagents.
- **`commands/review.md` size-estimation pattern** (`git status --short --untracked-files=all` + `git diff --shortstat --cached` + `git diff --shortstat`) for deciding wait vs. background. Specflow's `develop` step can reuse this verbatim before kicking off Codex.
- **`schemas/review-output.schema.json`** is a directly reusable contract for any specflow command that wants structured review findings (severity-graded, line-anchored, confidence-scored).
- **`prompts/adversarial-review.md`** is a high-quality reference for adversarial review prompt construction. The `<attack_surface>` taxonomy (auth, data loss, rollback, races, empty-state, version skew, observability gaps) maps cleanly to specflow's "fewer errors" north star.
- **`skills/gpt-5-4-prompting/SKILL.md`** + its `references/` (prompt-blocks.md, codex-prompt-recipes.md, codex-prompt-antipatterns.md) document how to author Codex prompts with stable XML tags (`<task>`, `<structured_output_contract>`, `<verification_loop>`, `<grounding_rules>`, etc.). Specflow can copy or import these references when constructing Codex prompts internally.
- **`skills/codex-result-handling/SKILL.md`** rule "**after presenting review findings, STOP — never auto-apply fixes**" is a critical safety pattern specflow should mirror in `specflow:design`/`specflow:develop` when surfacing Codex critique.
- **State directory layout** (`lib/state.mjs`): workspace-slug + SHA256-hash subdirectories of `${CLAUDE_PLUGIN_DATA}/state` is a clean pattern if specflow ever needs durable job state.
- **Session lifecycle hooks** (`SessionStart`/`SessionEnd` calling a single Node script) for setting `CODEX_COMPANION_SESSION_ID` so jobs can be filtered to the current session. Useful template if specflow needs per-session scoping.

## Direct relevance to specflow's goals

Specflow's adversarial-review goal (mentioned for `specflow:design`, `specflow:develop`) is exactly the use case this plugin was built to satisfy. **Specflow does not need to reimplement Codex integration — it should compose with `openai/codex-plugin-cc`.** Concretely:

### How specflow invokes Codex

Three viable invocation paths, each useful in different specflow steps:

1. **Slash command (preferred for user-visible review steps).** From inside any specflow command (`design`, `develop`, etc.), instruct Claude to run the user-installed slash command:
   - `/codex:adversarial-review --background <focus text>` — for design/architecture critique with a steerable focus string. Background-by-default since most specflow change sets are multi-file.
   - `/codex:review --base main --background` — for ship-time read-only review against the base branch.
   - `/codex:rescue --background <task>` — when specflow needs a second-opinion implementation pass or root-cause investigation handed off to Codex.
   - Then `/codex:status` and `/codex:result` to retrieve the structured output.

   Pro: zero new code in specflow, the user already has Codex auth/install, output is schema-validated. Con: requires the user to have installed `codex@openai-codex` separately (specflow should ship a "Run `/plugin install codex@openai-codex`" check in its readiness step).

2. **Direct CLI invocation (fallback).** If specflow detects the codex plugin is not installed but the `codex` binary is present, it can shell out to `codex` directly. The plugin's own scripts show the protocol details (`lib/codex.mjs`: thread params with `approvalPolicy: "never"`, `sandbox: "read-only"`, `serviceName: "claude_code_codex_plugin"`). This is heavier and not recommended — prefer routing users to `/codex:setup` and the official plugin.

3. **Subagent pattern (for inline composition).** If specflow wants its own subagent that wraps Codex (e.g. `specflow-codex-critic`), copy the `codex-rescue.md` shape: declare `tools: Bash`, body = strict forwarding rules calling `node "${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs" task ...`. This is only worth it if specflow needs to inject its own prompt scaffolding (e.g. always pass a specflow-design-doc context block) — otherwise just call `/codex:adversarial-review`.

### Recommended integration for specflow

- **`specflow:readiness`**: add a check for `codex` binary + a `claude /plugin list` lookup for `codex@openai-codex`. Surface `/codex:setup` if missing. Match the patterns already in specflow's existing readiness check.
- **`specflow:design` (after a design doc is drafted)**: call `/codex:adversarial-review --background "challenge the design choices in <design-doc-path> against {auth, data loss, rollback, races, empty-state, schema drift, observability}"`. Use the attack-surface taxonomy from `prompts/adversarial-review.md`. Wait or background based on size.
- **`specflow:develop` (after PR-shaped change set)**: call `/codex:review --base main --background` for the standard review, then `/codex:adversarial-review --background <focus>` if the design is risk-bearing.
- **`specflow:test` failure recovery**: rather than specflow looping itself, suggest `/codex:rescue --background investigate why <test> is failing` to delegate root-cause work.
- **Output handling**: import the verbatim-presentation + severity-ordered + no-auto-fix rules from `skills/codex-result-handling/SKILL.md` into the corresponding specflow skill. This directly serves the "fewer errors" goal — no silent fix-then-ship loops.
- **Cross-plugin coordination**: do **not** double-wrap Codex calls. If specflow's command instructs Claude to run a Codex slash command, treat the output as authoritative and don't have specflow's subagent re-analyze it (that defeats the adversarial-review value).

### Productivity / quality / fewer-errors mapping

- **More productive**: backgrounded adversarial review keeps the human and the Claude main thread free; specflow can fan out review work without blocking.
- **Better product in shorter time**: structured findings with `file`, `line_start`, `line_end`, `severity`, `confidence`, `recommendation` are immediately actionable; specflow can render them as a fix-list with checkboxes.
- **Fewer errors**: the Stop-gate review hook (when enabled), the no-auto-fix rule, the schema-validated output, and the explicit attack-surface taxonomy all directly attack the "AI ships subtle bugs" failure mode that a dev team new to AI coding will hit hardest. Specflow should default to recommending the review gate stays disabled (per the plugin's own warning) but should expose `/codex:adversarial-review` at every checkpoint.

## Cross-references

- Codex CLI docs: https://developers.openai.com/codex/cli/
- Codex app server: https://developers.openai.com/codex/app-server/
- Codex config: https://developers.openai.com/codex/config-basic, https://developers.openai.com/codex/config-advanced, https://developers.openai.com/codex/config-reference
- Codex pricing / ChatGPT subscription coverage: https://developers.openai.com/codex/pricing
- Project-level Codex config (for per-repo model/effort overrides): `.codex/config.toml` at workspace root (only loaded when project is trusted).
- Companion script entry point (when reading source): `plugins/codex/scripts/codex-companion.mjs`. Core lib: `plugins/codex/scripts/lib/codex.mjs`. State layout: `plugins/codex/scripts/lib/state.mjs`. Prompt templates: `plugins/codex/prompts/`. Output schema: `plugins/codex/schemas/review-output.schema.json`.
