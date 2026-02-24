---
name: "specflow:prime"
description: >
  Bootstrap a session with project context. Use this skill whenever the user
  says "prime", "load context", "bootstrap session", "what's the project status",
  or runs /specflow:prime. Also trigger at the start of a new conversation when
  the user wants to get oriented before doing work. This skill is read-only —
  it loads config, recent specflow state, codebase configuration, and git context
  so subsequent work starts from an informed position.
---

# Session Prime

Load project context into the current conversation so you can start work from an informed position. This skill is **read-only** — it reads files and runs read-only git commands but never modifies anything.

The goal is to front-load ~2,000-5,000 tokens of context so the user doesn't have to re-explain their project every session.

## Process

### Phase 1: Foundation Context

Load the core project configuration.

Steps:
1. Read `docs/specflow/config.json` — this determines what to load in later phases
2. Read `CLAUDE.md` if it exists (project-level instructions)
3. Read `.claude/settings.json` and `.claude/settings.local.json` if they exist
4. If `docs/specflow/config.json` does not exist:
   - Note: "No specflow config found. Using auto-detection. Run `/specflow:setup` for full configuration."
   - Fall back to bare auto-detection in Phase 3 (check for manifest files directly)

### Phase 2: Specflow State

Check what feature work is in flight by reading frontmatter only from recent specflow artifacts.

Steps:
1. Glob for `docs/specflow/prd/*.md` — find the most recently modified file
2. If found, read only the first 15 lines (frontmatter) to get the title, `prd_id`, status, and date
3. Glob for `docs/specflow/task/*-tasks-*.md` — find the most recently modified file
4. If found, read only the first 15 lines (frontmatter) to get the title, `prd_id`, and status
5. If no specflow artifacts exist, note: "No specflow PRDs or task reviews found."

### Phase 3: Codebase Configuration

Use the tech stack from `config.json` (or auto-detect if no config exists) to load relevant configuration files.

#### Tier 2 — Language Manifests

Read files based on detected tech stack. If no config.json exists, check for each file and read whichever are found.

| techStack match | Files to read |
|---|---|
| TypeScript / JavaScript | `package.json`, `tsconfig.json` |
| Python | `pyproject.toml` — if not found, try `requirements.txt` |
| Rust | `Cargo.toml` |
| Go | `go.mod` |
| Ruby | `Gemfile` |

#### Tier 3 — Tooling Configs

Read framework and tooling configs. Use "first found" logic — only read the first match per category.

| techStack match | Files to read (first found) |
|---|---|
| TypeScript / JavaScript | `biome.json`, or `.eslintrc*`, or `eslint.config.*` |
| Next.js | `next.config.*` (glob to find exact filename) |
| Tailwind CSS | `tailwind.config.*` (glob to find exact filename) |
| Prisma | `prisma/schema.prisma` (first 80 lines — enough to capture models without pulling in migrations or seed data) |
| Docker | `docker-compose.yml` or `Dockerfile` (first 40 lines — captures service definitions / base image + key stages without verbose COPY/RUN blocks) |

For auto-detection (no config.json): check files in this exact order to ensure consistent results across sessions:

1. **Tier 2 first**, top to bottom: `package.json` → `tsconfig.json` → `pyproject.toml` → `requirements.txt` → `Cargo.toml` → `go.mod` → `Gemfile`
2. **Then Tier 3**, top to bottom: `biome.json` → `.eslintrc*` → `eslint.config.*` → `next.config.*` → `tailwind.config.*` → `prisma/schema.prisma` → `docker-compose.yml` → `Dockerfile`

Read whatever is found. Infer the tech stack from whichever Tier 2 files exist (e.g., `package.json` present → TypeScript/JavaScript).

### Phase 4: Repo-Specific Files

Load custom files specified in the prime config section.

Steps:
1. Check if `config.json` has a `prime.files` array
2. If present, read each file listed (up to 5 files, up to 80 lines each — these limits keep repo-specific files within ~1,500 tokens, leaving headroom for the other phases)
3. Skip any file that doesn't exist — note which files were skipped but don't error
4. If no `prime` section exists, skip this phase entirely

### Phase 5: Git Context

Run read-only git commands to understand the current state.

Steps:
1. Run `git branch --show-current` to get the current branch
2. Run `git log --oneline -n 5` to get recent commits
3. Run `git status --short` to see uncommitted changes

Do NOT run `git diff` — its output is unbounded and would blow the context budget.

### Phase 6: Summary

Print a concise summary of everything that was loaded. Use this exact format:

```
--- Prime Complete ---
Project: {languages} / {frameworks}
Branch: {branch} | {n} uncommitted changes
Recent: {latest commit hash + message}
Constitution: {n} coding, {n} architecture, {n} quality principles
Specflow: {prd_id}: {prd name} ({status}) | {task review name} ({status})
Files loaded: {n} config, {n} repo-specific
```

Adapt the summary based on what was actually found:
- If no specflow artifacts: omit the Specflow line
- If no uncommitted changes: show "clean working tree"
- If no config.json: show "Project: auto-detected" with whatever was found
- If prime.files were configured: include the repo-specific count
- If constitution exists in config.json: show the principle counts per category. If no constitution: omit the Constitution line entirely.

End with: "Context loaded. Ready to work."

## Guidance

- This skill is strictly read-only. Never create, modify, or delete any files.
- Never run commands that modify state (no git checkout, no npm install, etc.).
- Keep total context under ~5,000 tokens. Use line limits on large files.
- When globbing for config files (e.g., `next.config.*`), read only the first match.
- If a file doesn't exist, skip it silently. Don't ask the user about missing files.
- Speed matters — this runs at the start of every session. Don't over-explain or ask unnecessary questions. Just load and summarize.
- If config.json is missing, auto-detection should still produce useful output. The skill should always complete, even in an empty repo.

## Error Handling

- **config.json corrupted / unparseable JSON**: Skip it entirely and fall back to auto-detection. Note in the summary: "config.json could not be parsed — using auto-detection."
- **Git commands fail** (e.g., not a git repo, git not installed): Skip Phase 5 entirely. In the summary, replace the Branch/Recent lines with "Git: unavailable". Do not ask the user to fix anything.
- **File read fails** (permissions, encoding, binary): Skip that file and continue. Never let a single unreadable file abort the skill.
- **General principle**: The skill must always run to completion and produce a summary. Degrade gracefully — load what you can, skip what you can't, and note omissions in the summary rather than erroring out.
