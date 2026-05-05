# Archon: Harness Engineering as the Next Evolution of AI Coding

## Identity
- Title: Unveiling the new Archon — open-source harness builder for AI coding
- Speaker/Channel: Cole Medin (coleam00)
- Likely URL: YouTube channel of Cole Medin (companion to LinkedIn announcement dated 2026-04-07 and the live stream "this Saturday 9am Central")
- Suggested slug: cole-medin-archon-harness-builder
- Confidence: High (speaker self-identifies the project as "the new Archon", repository is github.com/coleam00/Archon, video script matches Cole Medin's LinkedIn announcement of the rewrite)

## Thesis
AI coding has evolved from prompt engineering (2022-2024) to context engineering and now to "harness engineering": orchestrating multiple coding-agent sessions with deterministic guardrails. A custom harness wrapping a model like Opus can outperform a stronger model used naively. Archon lets teams encode their entire SDLC as reusable YAML workflows that mix LLM-driven nodes with deterministic nodes (validation, context curation, human approval) and run them in parallel across repos.

## Key points

### KP-1
- **Point**: Harness engineering is the new layer above context engineering — multiple agent sessions chained together with deterministic logic to make AI coding repeatable.
- **Why it matters to our goals**: Frames specflow's purpose. Our pipeline (docs creator + dev team) is itself a harness. Naming this evolution helps the team understand *why* we don't just hand a prompt to Claude.
- **Evidence**: "Prompt engineering ... evolved into context engineering ... now harness engineering, where we're dealing with many different coding agent sessions, all tying that together through a harness."
- **Sources**: Transcript lines 92-128

### KP-2
- **Point**: A well-built harness around a mid-tier model can beat a stronger model used naively — cited as PR acceptance jumping from ~6.7% to ~70%.
- **Why it matters to our goals**: Direct evidence that disciplined process (specflow) > raw model power. Justifies investment in pipeline structure for "fewer errors" and "better product".
- **Evidence**: "If we just use it to create some code, the PR acceptance rate is only 6.7%. But if we create a harness ... we can get even as high as like almost 70%."
- **Sources**: Transcript lines 136-150

### KP-3
- **Point**: Stripe's internal "Stripe Minion" harness ships ~1,300 AI-only-generated PRs per week by enforcing context curation and validation at fixed steps.
- **Why it matters to our goals**: Concrete proof that small/medium teams can dramatically scale dev throughput with the right harness. Directly speaks to "more productive" and "shorter time".
- **Evidence**: "Stripe ... they ship 1,300 AI only generated pull requests every single week ... they actually built something kind of like Archon, but it's not open-source."
- **Sources**: Transcript lines 151-163

### KP-4
- **Point**: ~40% of the leaked Claude Code source is harness code, signalling that even Anthropic is leaning hard into harnesses (sub-agents, agent teams).
- **Why it matters to our goals**: Validates that our roadmap of skills/sub-agents/teams is aligned with where the platform itself is heading.
- **Evidence**: "With the Claude code source code leak, we found that even Anthropic is leaning a lot more into harnesses ... 40% of their code base is just code for harnesses."
- **Sources**: Transcript lines 163-170

### KP-5
- **Point**: Workflows are nodes — each node is either a prompt to a coding agent or a deterministic command. The "hybrid secret" is enforcing the deterministic ones (validation, context curation) so the agent can't skip them.
- **Why it matters to our goals**: Direct prescription for our pipeline. Steps like "run tests", "load PRD", "QA label" should be deterministic nodes, not things we hope the agent remembers.
- **Evidence**: "A node is either a prompt that we send into a coding agent session, or it is a deterministic command ... we don't want to leave up to the coding agent cuz it might forget to do so."
- **Sources**: Transcript lines 41-58, 232-249

### KP-6
- **Point**: Always do planning and implementation in *fresh context windows / different sessions* to remove bias.
- **Why it matters to our goals**: Concrete implementation rule for specflow. Plan→build handoff should reset context, not continue the same chat. Reduces hallucinated assumptions and lowers token cost.
- **Evidence**: "We do that in a fresh context window. You always want to do your planning and implementation in different coding sessions to remove bias."
- **Sources**: Transcript lines 213-218

### KP-7
- **Point**: Build human-approval gates *into* the workflow rather than relying on humans to remember to intervene.
- **Why it matters to our goals**: Aligns with our memory note about "no premature pipeline CTAs" and significant human workflow gaps. Approval gates should be explicit nodes, not implicit norms.
- **Evidence**: "Even adding in a human approval gate so we can address our feedback ... we can build ourselves into Archon workflows."
- **Sources**: Transcript lines 53-58, 222-226, 691-696

### KP-8
- **Point**: Per-node model selection — use Haiku for cheap classification/extraction nodes, Sonnet/Opus only where reasoning is needed.
- **Why it matters to our goals**: Token cost is a real constraint. Mapping each pipeline step to the cheapest sufficient model directly addresses cost and rate-limit pain for the dev team.
- **Evidence**: "Certain nodes, like classification, they don't need a lot of reasoning power. So we can make it a lot more token efficient, a lot cheaper by just using Haiku."
- **Sources**: Transcript lines 612-623, 666-672

### KP-9
- **Point**: Workflows ship with a description (skill-style) so the coding agent can route to the right workflow without loading the full YAML into context.
- **Why it matters to our goals**: Mirrors Claude Code's skill description pattern. Confirms our skill-frontmatter approach is the right shape; descriptions are routing metadata, not tutorials.
- **Evidence**: "The description is what we first give to Claude Code ... it uses this to determine if it should analyze and run this entire workflow."
- **Sources**: Transcript lines 568-583

### KP-10
- **Point**: Inject context (skills, MCP servers) per-node, not globally — e.g. a validation-only skill, a planning-only MCP.
- **Why it matters to our goals**: Today our skills are globally loaded. Scoping context per phase keeps each agent lean and reduces cross-phase confusion.
- **Evidence**: "Maybe you have a skill that you only need during the validation step, or you have an MCP server that you only want during planning. We have that level of control per node."
- **Sources**: Transcript lines 192-200

### KP-11
- **Point**: Pair every CLI with a skill so the coding agent knows how to use it. CLIs without skills are invisible to agents.
- **Why it matters to our goals**: Direct rule for any tooling we ship in specflow. If we add a CLI, we ship a skill alongside it or the team won't actually invoke it through Claude.
- **Evidence**: "It's important for any CLI to have a skill paired with it, so that our coding agent knows how to use it."
- **Sources**: Transcript lines 396-400

### KP-12
- **Point**: Bundled "fix GitHub issue" workflow does extract → classify (bug vs feature) → web research → investigate-or-plan → implement → validate → PR → review.
- **Why it matters to our goals**: A reference SDLC for our dev team to copy. Especially the bug/feature classification branch — different paths for different work types.
- **Evidence**: "Extract the issue number ... classify the issue. Is this a bug that needs to be fixed or a feature that we need to build? ... investigation, fixing, and validation before it creates the pull request."
- **Sources**: Transcript lines 484-491, 598-665

### KP-13
- **Point**: Workflows can run many in parallel against the same repo (demo: 6 GitHub issues fixed concurrently as background processes).
- **Why it matters to our goals**: Throughput multiplier. A 4-person dev team running 6 parallel workflows is effectively a 24-dev team for narrow, well-specified tasks. Big lever for "shorter time".
- **Evidence**: "Use Archon to fix GitHub issues 5 7 8 9 10 and 11 ... six times in a row, all of them are running as background processes."
- **Sources**: Transcript lines 759-812

### KP-14
- **Point**: "Define once, run forever, reusable across projects" — workflows are repo-agnostic; you register a target repo and the same harness applies.
- **Why it matters to our goals**: specflow itself should be portable. A docs-creator → dev-team pipeline shouldn't be re-built per project; it should be a workflow that points at any repo.
- **Evidence**: "Define once, run forever, reusable across projects."
- **Sources**: Transcript lines 188-193, 285-308

### KP-15
- **Point**: Per-node session control — choose to start a *new* session or *continue* the existing one between nodes for token/context management.
- **Why it matters to our goals**: Practical lever to keep context lean. Long sessions waste cache and accumulate confusion; explicit "new session" boundaries are a quality-and-cost win.
- **Evidence**: "We can determine when we go from node to node, do we want to start a brand new session with Claude Code or continue the conversation? ... keep our context lean."
- **Sources**: Transcript lines 539-549

### KP-16
- **Point**: Existing process frameworks (GSD, B-MAD, beads-style persistent memory) can be ported into a harness rather than adopted wholesale.
- **Why it matters to our goals**: Validates specflow's pick-and-mix posture — borrow patterns from the broader ecosystem rather than swallowing one methodology.
- **Evidence**: "If you want to take another framework like GSD or B-MAD and bring it into Archon, or take a strategy like beads for memory ... you can build it as a custom workflow."
- **Sources**: Transcript lines 822-870

### KP-17
- **Point**: Setup wizard for credentials runs in a *separate terminal*, never in the coding-agent context, so API keys don't enter agent transcripts.
- **Why it matters to our goals**: Security pattern worth copying. Any specflow setup that touches secrets must avoid passing them through Claude.
- **Evidence**: "We need to do this in a different window, because we don't want to send our API keys directly into Claude Code."
- **Sources**: Transcript lines 326-345

## Tools / repos / frameworks mentioned
- **Archon** — github.com/coleam00/Archon — the harness builder itself (YAML workflows + web UI + CLI)
- **Claude Code** / **Codex** — primary coding agents Archon wraps
- **Claude Agent SDK** — used so Anthropic-subscription auth works locally
- **Stripe Minion** — Stripe's internal closed-source harness, the inspiration
- **Anthropic Mythos** — upcoming enterprise-tier model (referenced as costly/unavailable to consumers)
- **Ralph loop** — earlier well-known harness pattern
- **GSD**, **B-MAD** — process frameworks portable into Archon
- **beads** — persistent structured memory for coding agents (open source repo)
- **N8N** — visual reference for Archon's upcoming workflow builder UI ("N8N for AI coding")
- **Bun** — runtime prerequisite for Archon
- **SQLite / Postgres** — database options
- **Telegram / Slack / GitHub** — platform integrations
- **PyAgent SDK / Open Code** — planned future agent backends

## Verification log
- WebSearch query "Archon AI command center harness builder Cole Medin coleam00 open source 2026" → confirmed Archon is by Cole Medin (coleam00), described on GitHub as "the first open-source harness builder for AI coding. Make AI coding deterministic and repeatable" — matches transcript wording verbatim.
- LinkedIn announcement post by Cole Medin dated April 2026 corroborates "after months of work behind the scenes" opening of the transcript.
- Repo: https://github.com/coleam00/Archon (17.9k+ stars at time of search).
