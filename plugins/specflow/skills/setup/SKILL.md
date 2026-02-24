---
name: "specflow:setup"
description: >
  Set up the specflow plugin in a repository. Use this skill whenever the user
  says "set up specflow", "configure specflow", "initialize specflow", or runs
  /specflow:setup. Also trigger when a user has just installed the plugin and
  wants to get started. This skill creates the required directories, verifies
  Linear MCP configuration, scans the codebase to detect the tech stack, and
  installs relevant specialist agents from the wshobson/agents marketplace.
---

# Specflow Setup

Configure specflow in the current repository. This skill handles first-run setup: directory creation, Linear MCP verification, tech stack detection, and specialist agent installation.

## Process

### Phase 1: Create Directories

Check if the required specflow directories exist and create any that are missing.

Required directories:
- `docs/specflow/prd/`
- `docs/specflow/task/`

Steps:
1. Check which directories already exist
2. If any are missing, confirm with the user before creating them
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
6. If Linear MCP is configured, ask the user which Linear team they primarily work with and store the answer for Phase 5

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
3. Build a summary of detected technologies
4. Present findings to the user: "Detected: Next.js, TypeScript, Tailwind, Prisma" (example)
5. Ask the user to confirm the detected stack is accurate, and if anything is missing

### Phase 4: Install Specialist Agents

Based on the detected tech stack, recommend and install specialist agents from the `wshobson/agents` marketplace.

Steps:
1. Add the wshobson/agents marketplace: run `claude plugin add wshobson/agents` via Bash

**Base Plugins (always installed):**

2. Install these 3 base plugins before any tech-stack plugins:
   - `agent-teams@wshobson-agents` — multi-agent coordination (team-lead, team-reviewer)
   - `comprehensive-review@wshobson-agents` — architecture and code quality review (architect-review, code-reviewer)
   - `agent-orchestration@wshobson-agents` — context management (context-manager)

   Install each via: `claude plugin install {name}@wshobson-agents`
   If any base plugin fails to install, warn the user but continue with the remaining plugins.

**Tech-Stack Plugins:**

3. Map detected tech stack to relevant plugins:

| Detected Stack | Plugin to Install |
|---|---|
| Python / Django / FastAPI / Flask | `python-development` |
| JavaScript / TypeScript / React / Next.js | `javascript-typescript` |
| Frontend (React, Vue, Angular, Tailwind) | `frontend-development` |
| Backend (Express, FastAPI, Django, Rails) | `backend-development` |
| Full-stack (frontend + backend detected) | `full-stack-orchestration` |
| Docker / Kubernetes | `kubernetes-operations` |
| CI/CD workflows | `ci-cd-pipeline` |
| Database (Prisma, SQLAlchemy, etc.) | `database-operations` |

4. Present the recommended tech-stack plugins to the user for approval before installing
5. Install each approved plugin via: `claude plugin install {name}@wshobson-agents`
6. If a plugin fails to install, report the error but continue with the remaining plugins
7. Note to the user: they can remove any installed plugin later with `claude plugin uninstall {name}@wshobson-agents` or remove the entire marketplace with `claude plugin remove wshobson/agents`

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
    "cicd": true
  },
  "linear": {
    "configured": true,
    "team": "Engineering"
  },
  "agents": {
    "marketplace": "wshobson/agents",
    "base": [
      "agent-teams@wshobson-agents",
      "comprehensive-review@wshobson-agents",
      "agent-orchestration@wshobson-agents"
    ],
    "techStack": [
      "javascript-typescript@wshobson-agents",
      "frontend-development@wshobson-agents"
    ],
    "installed": [
      "agent-teams@wshobson-agents",
      "comprehensive-review@wshobson-agents",
      "agent-orchestration@wshobson-agents",
      "javascript-typescript@wshobson-agents",
      "frontend-development@wshobson-agents"
    ],
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
- `techStack`: categorize detected technologies into languages, frameworks, databases, and infrastructure
- `linear.configured`: whether Linear MCP was found or set up
- `linear.team`: the team name from Phase 2 (omit if Linear is not configured)
- `agents.base`: the 3 base plugins (always the same list)
- `agents.techStack`: only the tech-stack plugins that were installed
- `agents.installed`: combined list of all successfully installed plugins (base + tech-stack)
- `agents.roles`: mapping of role names to `plugin:subagent` pairs — this tells PRD and task skills which agent to invoke for each purpose via the Task tool's `subagent_type` parameter

This file is the single reference for what setup detected and installed. Other skills (`prd`, `task`, `linear`, `prime`) can read it to skip redundant questions (e.g., Linear team name). Users can delete this file and re-run `/specflow:setup` to reconfigure.

### Phase 5b: Prime Configuration (Optional)

After saving the base config, ask the user if they want to configure session priming for `/specflow:prime`.

Steps:
1. Ask: "Would you like to configure `/specflow:prime`? It loads project context at the start of each session — config files, recent specflow state, and git status."
2. If yes, ask: "Are there any repo-specific files you want always loaded at the start of a session? (e.g., `src/lib/db.ts`, `src/shared/utils.ts`) — up to 5 files, or skip if none."
3. If the user provides files, add a `prime` section to the saved `docs/specflow/config.json`:

```json
{
  "...existing fields...",
  "prime": {
    "files": ["src/lib/db.ts", "src/shared/utils.ts"]
  }
}
```

4. If the user skips or says no to either question, don't add the `prime` section — `/specflow:prime` will still work using auto-detection from `techStack`

### Phase 6: Summary

Print a setup summary covering everything that was done:

- **Directories**: created / already existed
- **Linear MCP**: configured / not configured (with warning if not configured)
- **Tech stack**: list of detected technologies
- **Agents installed**: list of installed plugins (with note on how to remove)
- **Config saved**: `docs/specflow/config.json`
- **Prime**: configured with {n} custom files / not configured (auto-detection only)

Include these tips:
- To remove an agent: `claude plugin uninstall {name}@wshobson-agents`
- To reconfigure: delete `docs/specflow/config.json` and re-run `/specflow:setup`

End with: "Run `/specflow:prime` to load project context, or `/specflow:prd` to start creating a PRD."

## Guidance

- Always confirm with the user before creating directories or installing plugins
- If any phase fails, continue with the remaining phases -- partial setup is better than no setup
- The config.json file is optional for other skills -- they all work without it, they just benefit from it
- If the user has already run setup before, check for an existing `docs/specflow/config.json` and ask if they want to reconfigure or keep the existing setup
- Keep the summary concise -- the user wants to know what happened, not read a novel
