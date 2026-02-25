---
name: "specflow:setup"
description: >
  Set up the specflow plugin in a repository. Use this skill whenever the user
  says "set up specflow", "configure specflow", "initialize specflow", or runs
  /specflow:setup. Also trigger when a user has just installed the plugin and
  wants to get started. This skill creates the required directories, verifies
  Linear MCP configuration, scans the codebase to detect the tech stack, and
  installs relevant specialist agents from the wshobson/agents marketplace.
  Offers a quick setup mode (recommended) that auto-configures everything with
  minimal questions, or a custom mode for step-by-step control.
---

# Specflow Setup

Configure specflow in the current repository. This skill handles first-run setup: directory creation, Linear MCP verification, tech stack detection, and specialist agent installation.

## Process

### Setup Mode Selection

Before starting, ask the user which setup mode they prefer:

**Quick setup (recommended):**
- Auto-creates directories without per-directory confirmation
- Auto-detects Linear team (skips question if only one team)
- Auto-generates a constitution from the codebase (CLAUDE.md, config files, existing patterns) — presented for one-shot confirmation
- Installs all recommended plugins without per-plugin approval, with brief explanations of what each does
- Configures prime with auto-detected files

**Custom setup:**
- The full step-by-step flow with per-phase confirmation and customization at each step

Ask: "How would you like to set up specflow?"
- Option 1: "Quick setup (recommended)" — "Auto-configure everything based on your codebase. You'll review a summary at the end."
- Option 2: "Custom setup" — "Step-by-step walkthrough with control over each decision."

If the user says anything like "just set it up", "whatever you think", "recommended", or "quick" — use quick setup.

In the phases below, differences between quick and custom mode are noted where applicable.

### Phase 1: Create Directories

Check if the required specflow directories exist and create any that are missing.

Required directories:
- `docs/specflow/prd/`
- `docs/specflow/task/`

Steps:
1. Check which directories already exist
2. **Custom mode:** confirm with the user before creating them. **Quick mode:** create automatically and report what was done.
3. Report what was created vs what already existed

### Phase 2: Check Linear MCP

Verify that Linear MCP is configured so the `/specflow:task` and `/specflow:linear` skills can function.

Steps:
1. Read `.mcp.json` in the project root (project-scope MCP config) and check for a Linear MCP server entry
2. Read `~/.claude.json` (user-scope config) and check for a Linear MCP server entry
3. As a fallback, run `claude mcp list` via Bash to check for any Linear MCP server
4. If Linear MCP is found in any location: confirm to the user ("Linear MCP is configured")
5. If not found:
   - Warn the user that Linear MCP is required for `/specflow:task` and `/specflow:linear`
   - Ask if they'd like to set it up now
   - If yes: run `claude mcp add linear -- npx -y @anthropic/linear-mcp-server` via Bash
   - If no: note that `/specflow:prd` still works without it, but task and linear skills will not function
6. If Linear MCP is configured, auto-detect the user's Linear teams:
   - Call the `list_teams` MCP tool to retrieve available teams
   - If exactly 1 team exists: auto-select it and confirm to the user ("Using team: {name}")
   - If multiple teams exist: present them as options and ask the user to pick their primary team
   - If the MCP call fails or the user wants to decide later: store `null` — downstream skills (`/specflow:task`, `/specflow:linear`) will ask when the team is actually needed
   - A `null` team value means "deferred", not "unconfigured"

### Phase 3: Scan Repository Tech Stack

Detect the project's tech stack by checking for key files and parsing their contents.

Check for the following files/patterns and detect accordingly:

| File/Pattern | Stack Detection |
|---|---|
| `package.json` | Parse `dependencies` and `devDependencies` for React, Next.js, Vue, Angular, Express, Fastify, etc. |
| `tsconfig.json` | TypeScript project |
| `requirements.txt` / `pyproject.toml` / `Pipfile` | Python; parse for Django, FastAPI, Flask |
| `Cargo.toml` | Rust |
| `go.mod` | Go |
| `Gemfile` | Ruby/Rails |
| `docker-compose.yml` / `Dockerfile` | Containerized |
| `.github/workflows/` | CI/CD configured |
| `prisma/` / `drizzle.config.*` | Database ORM |
| `next.config.*` | Next.js specifically |
| `tailwind.config.*` | Tailwind CSS |

Steps:
1. Use Glob and Read tools to check for each file/pattern
2. When a manifest file is found (e.g., `package.json`), read it and parse dependencies to identify specific frameworks
3. Build a summary of detected technologies — include ALL detected libraries, even those that don't fit standard categories (animation, mail, 3D, etc.)
4. Present findings to the user: "Detected: Next.js, TypeScript, Tailwind, Prisma" (example)
5. **Custom mode:** Ask the user to confirm the detected stack is accurate, and if anything is missing. **Quick mode:** present findings as informational and continue.

### Phase 3b: Project Constitution

Ask the user if they want to define project principles that every PRD should follow. These are non-negotiable constraints — hard rules. If something needs judgment, that's what the PRD interview handles.

**Three paths:**

#### Path A: Auto-generate (default in quick mode)

This is the recommended path. Also use this path whenever the user says "you decide", "whatever you think", "recommended", "let Claude decide", or similar.

Steps:
1. Read the project's CLAUDE.md (if it exists), linter config (`.eslintrc.*`, `biome.json`, etc.), test setup (`jest.config.*`, `vitest.config.*`, etc.), and `package.json` scripts
2. Analyze the project context: solo dev vs team (check git log for multiple authors), app type (marketing site, SaaS, API, library), existing test coverage, CI/CD setup
3. Generate contextually appropriate principles:
   - Solo dev on a marketing site gets: "Add tests for new business logic" not "100% test coverage"
   - Project with no tests gets: "Add test coverage for critical paths" not "all public APIs must have integration tests"
   - Project with CLAUDE.md rules gets those rules incorporated rather than duplicated
4. Present the auto-generated principles grouped by category for one-shot confirmation: "Here are recommended principles based on your codebase. Accept, edit, or skip?"
5. If the user accepts: store them. If they want to edit: let them modify in-place. If they skip: don't add a `constitution` key.

#### Path B: Custom (default in custom mode)

Steps:
1. Ask: "Would you like to define project principles that every PRD should follow? (You can skip this and add them later by re-running setup.)"
2. If the user skips, move to Phase 4. Do NOT add a `constitution` key to config.json — downstream skills degrade gracefully when it's absent.
3. If yes, interview across three categories (max 5 principles per category). Adapt examples to the detected project context:

**Coding principles** — rules about how code should be written
- For solo devs / small projects: "prefer composition over inheritance", "use strict TypeScript (no `any`)", "keep functions under 50 lines"
- For teams: "all functions must have return types", "no default exports", "document public APIs"
- Each category offers a "Recommend for me" option that triggers auto-generation for just that category
- Ask: "What coding principles should every developer follow? (Up to 5, or 'recommend for me', or press enter to skip.)"

**Architectural boundaries** — rules about system structure
- For solo devs: "keep API routes thin — business logic in service layer", "no direct DB access from UI components"
- For teams: "all external API calls go through a service layer", "no circular dependencies between modules"
- Ask: "What architectural boundaries should never be crossed? (Up to 5, or 'recommend for me', or press enter to skip.)"

**Quality standards** — rules about testing, review, and release
- For solo devs: "add tests for new business logic", "run linter before committing"
- For teams: "100% test coverage for business logic", "no PR without reviewer", "all public APIs must have integration tests"
- Ask: "What quality standards must every feature meet? (Up to 5, or 'recommend for me', or press enter to skip.)"

4. After collecting principles, summarize them back to the user grouped by category
5. Ask the user to confirm, edit, or remove any principles
6. Store confirmed principles in config.json under the `constitution` key (see Phase 5 schema)
7. If the user provided principles in some categories but skipped others, only include the non-empty categories

#### Path C: Skip

If the user explicitly skips, don't add a `constitution` key. Downstream skills degrade gracefully.

### Phase 4: Install Specialist Agents

Based on the detected tech stack, recommend and install specialist agents from the `wshobson/agents` marketplace.

**Important:** All `claude plugin` commands may not produce visible stdout in the Bash tool. For every `claude plugin` command, use this pattern to capture output:

```bash
claude plugin <subcommand> <args> 1>/tmp/specflow-plugin-output.txt 2>&1; cat /tmp/specflow-plugin-output.txt
```

Steps:

**Step 1: Check pre-existing plugins**

Run `claude plugin list 1>/tmp/specflow-plugin-output.txt 2>&1; cat /tmp/specflow-plugin-output.txt` to see what's already installed. Parse the output to identify:
- Already-installed plugins from any marketplace
- Disabled plugins that can be re-enabled instead of re-installed

**Step 2: Add the marketplace**

Run: `claude plugin marketplace add wshobson/agents 1>/tmp/specflow-plugin-output.txt 2>&1; cat /tmp/specflow-plugin-output.txt`

If the marketplace is already added, this is a no-op — continue.

**Step 3: Detect the registered marketplace name**

Run: `claude plugin marketplace list 1>/tmp/specflow-plugin-output.txt 2>&1; cat /tmp/specflow-plugin-output.txt`

Parse the output to find the registered name for `wshobson/agents`. This is the `{MARKETPLACE_NAME}` used in all subsequent install commands. Do NOT hardcode `wshobson-agents` or `claude-code-workflows` — always use the detected name.

Store this name for use in the config file and all install commands.

**Step 4: Install base plugins**

These 3 base plugins are always installed (they power the specflow workflow):

| Plugin | What It Does in Specflow |
|---|---|
| `agent-teams` | Multi-agent coordination — team-lead orchestrates parallel work, team-reviewer runs code reviews |
| `comprehensive-review` | Architecture and code quality review — architect-review checks system design, code-reviewer checks implementation |
| `agent-orchestration` | Context management — context-manager loads and maintains project context across sessions |

Install each via: `claude plugin install {name}@{MARKETPLACE_NAME} 1>/tmp/specflow-plugin-output.txt 2>&1; cat /tmp/specflow-plugin-output.txt`

If a plugin is already installed (detected in Step 1), skip it and note that it's already present. If a plugin fails to install, add it to the `agents.failed` list and warn the user but continue.

**Step 5: Install tech-stack plugins**

Map detected tech stack to relevant plugins:

| Detected Stack | Plugin to Install | What It Does |
|---|---|---|
| Python / Django / FastAPI / Flask | `python-development` | Python-specific linting, testing patterns, framework guidance |
| JavaScript / TypeScript / React / Next.js | `javascript-typescript` | JS/TS optimization, async patterns, type safety |
| Frontend (React, Vue, Angular, Tailwind) | `frontend-mobile-development` | UI component patterns, accessibility, responsive design |
| Backend (Express, FastAPI, Django, Rails) | `backend-development` | API design, middleware patterns, error handling |
| Full-stack (frontend + backend detected) | `full-stack-orchestration` | Coordinated frontend/backend development |
| Docker / Kubernetes | `kubernetes-operations` | Container orchestration, deployment patterns |
| CI/CD workflows | `cicd-automation` | Pipeline design, build optimization, deployment automation |
| Database (Prisma, SQLAlchemy, etc.) | `database-design` | Schema design, migration patterns, query optimization |

**Quick mode:** Install all recommended tech-stack plugins without per-plugin approval. Present a brief summary of what will be installed and why.

**Custom mode:** Present the recommended tech-stack plugins to the user for approval before installing.

Install each approved plugin via: `claude plugin install {name}@{MARKETPLACE_NAME} 1>/tmp/specflow-plugin-output.txt 2>&1; cat /tmp/specflow-plugin-output.txt`

If a plugin is already installed, skip it. If a plugin fails to install, add it to `agents.failed` and continue.

**Step 6: Report results**

Tell the user:
- Which plugins were installed successfully
- Which were already installed (skipped)
- Which failed (with error details)
- How to remove a plugin: `claude plugin uninstall {name}@{MARKETPLACE_NAME}`
- How to remove the marketplace: `claude plugin marketplace remove {MARKETPLACE_NAME}`

### Phase 5: Save Setup Config

Save a configuration file that captures the setup state for use by other specflow skills.

Save to `docs/specflow/config.json` with this structure:

```json
{
  "created": "YYYY-MM-DD",
  "techStack": {
    "languages": ["TypeScript"],
    "frameworks": ["Next.js", "Tailwind CSS"],
    "databases": ["Prisma"],
    "infrastructure": ["Docker"],
    "cicd": true,
    "other": {
      "animation": ["GSAP", "Framer Motion"],
      "3d": ["Three.js", "React Three Fiber"],
      "mail": ["Nodemailer"]
    }
  },
  "linear": {
    "configured": true,
    "team": "Engineering"
  },
  "constitution": {
    "coding": ["Use strict TypeScript (no `any`)", "Keep functions under 50 lines"],
    "architecture": ["No direct DB access from UI components", "API routes delegate to service layer"],
    "quality": ["Add tests for new business logic", "Run linter before committing"]
  },
  "agents": {
    "marketplace": {
      "source": "wshobson/agents",
      "registeredName": "claude-code-workflows"
    },
    "base": [
      "agent-teams",
      "comprehensive-review",
      "agent-orchestration"
    ],
    "techStack": [
      "javascript-typescript",
      "frontend-mobile-development"
    ],
    "installed": [
      "agent-teams",
      "comprehensive-review",
      "agent-orchestration",
      "javascript-typescript",
      "frontend-mobile-development"
    ],
    "failed": [],
    "roles": {
      "orchestrator": "agent-teams:team-lead",
      "architectureReview": "comprehensive-review:architect-review",
      "codeReview": "comprehensive-review:code-reviewer",
      "contextManager": "agent-orchestration:context-manager"
    }
  }
}
```

Populate each field based on what was detected and installed in the previous phases:
- `created`: today's date in ISO format
- `techStack`: categorize detected technologies into languages, frameworks, databases, and infrastructure. Use `techStack.other` as a catch-all object for technologies that don't fit standard categories — key is the category name, value is an array of library names
- `linear.configured`: whether Linear MCP was found or set up
- `linear.team`: the team name from Phase 2. `null` means "deferred" — downstream skills will ask when the team is actually needed. Omit the `team` key entirely only if Linear is not configured.
- `constitution`: only present if the user defined or accepted principles in Phase 3b. Contains up to three sub-keys (`coding`, `architecture`, `quality`), each an array of strings. Omit the entire key if the user skipped constitution setup. Omit individual sub-keys for categories the user left empty.
- `agents.marketplace`: object with `source` (the GitHub path used to add it) and `registeredName` (the name detected via `claude plugin marketplace list`). The `registeredName` is what downstream commands need for install/uninstall operations.
- `agents.base`: the 3 base plugin names (without `@suffix`)
- `agents.techStack`: only the tech-stack plugin names that were installed (without `@suffix`)
- `agents.installed`: combined list of all successfully installed plugin names (base + tech-stack, without `@suffix`)
- `agents.failed`: array of plugin names that failed to install (empty array if all succeeded)
- `agents.roles`: mapping of role names to `plugin:subagent` pairs — this tells PRD and task skills which agent to invoke for each purpose via the Task tool's `subagent_type` parameter

This file is the single reference for what setup detected and installed. Other skills (`prd`, `task`, `linear`, `prime`) can read it to skip redundant questions (e.g., Linear team name). Users can delete this file and re-run `/specflow:setup` to reconfigure.

### Phase 5b: Prime Configuration (Optional)

After saving the base config, ask the user if they want to configure session priming for `/specflow:prime`.

Steps:
1. Ask: "Would you like to configure `/specflow:prime`? It loads project context at the start of each session — config files, recent specflow state, and git status."
2. If yes, ask: "Are there any repo-specific files you want always loaded at the start of a session? (e.g., `src/lib/db.ts`, `src/shared/utils.ts`) — **up to 5 files**. If you suggest more than 5, I'll ask you to pick your top 5. Or skip if none."
3. If the user suggests more than 5 files, show the full list and ask them to pick their top 5.
4. If the user provides files, add a `prime` section to the saved `docs/specflow/config.json`:

```json
{
  "...existing fields...",
  "prime": {
    "files": ["src/lib/db.ts", "src/shared/utils.ts"]
  }
}
```

5. If the user skips or says no to either question, don't add the `prime` section — `/specflow:prime` will still work using auto-detection from `techStack`

### Phase 6: Summary

Print a setup summary covering everything that was done:

- **Directories**: created / already existed
- **Linear MCP**: configured / not configured (with warning if not configured)
- **Linear team**: {team name} / deferred
- **Tech stack**: list of detected technologies
- **Agents installed**: list of installed plugins (with note on how to manage)
- **Constitution**: {n} coding, {n} architecture, {n} quality principles / skipped
- **Config saved**: `docs/specflow/config.json`
- **Prime**: configured with {n} custom files / not configured (auto-detection only)

Include these tips (using the detected `{MARKETPLACE_NAME}` from Phase 4 — never hardcode):
- To remove an agent: `claude plugin uninstall {name}@{MARKETPLACE_NAME}`
- To add more agents: `claude plugin marketplace list {MARKETPLACE_NAME}` then `claude plugin install {name}@{MARKETPLACE_NAME}`
- To reconfigure: delete `docs/specflow/config.json` and re-run `/specflow:setup`

End with: "Run `/specflow:prime` to load project context, or `/specflow:prd` to start creating a PRD."

## Guidance

- Always confirm with the user before creating directories or installing plugins (in custom mode)
- If any phase fails, continue with the remaining phases -- partial setup is better than no setup
- The config.json file is optional for other skills -- they all work without it, they just benefit from it
- If the user has already run setup before, check for an existing `docs/specflow/config.json` and ask if they want to reconfigure or keep the existing setup
- Keep the summary concise -- the user wants to know what happened, not read a novel
- **Stdout capture:** All `claude plugin` commands must use the `1>/tmp/specflow-plugin-output.txt 2>&1; cat /tmp/specflow-plugin-output.txt` pattern — the Bash tool does not capture their stdout directly
- **Dynamic marketplace name:** Never hardcode `@wshobson-agents` or `@claude-code-workflows` as the marketplace suffix. Always detect the registered name via `claude plugin marketplace list` and use that throughout
- **"You decide" handling:** If the user says "you decide", "whatever you think", "recommended", "let Claude decide", or similar at any point, treat it as a request for auto-generation. For constitution questions, switch to the auto-generate path. For plugin selection, install all recommended plugins. For any other question, make a sensible default choice based on the codebase and confirm it.
- **Auto-switch to quick mode:** If the user chose custom mode but keeps deferring decisions ("you decide", skipping questions), suggest switching to quick mode to save time
