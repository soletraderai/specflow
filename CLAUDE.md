# Project Rules

## Branding / Attribution

- NEVER mention Claude, Anthropic, or any AI tooling in commits, PRs, code comments, documentation, or any user-facing output.
- Do NOT add "Co-Authored-By" lines referencing Claude or Anthropic in git commits.
- Do NOT include phrases like "Generated with Claude Code" in PR descriptions.
- This applies everywhere: commit messages, pull request titles/bodies, changelogs, issue comments, and code.

## Releases

- When creating or updating a GitHub release, always update these files to match the new version:
  - `plugins/specflow/.claude-plugin/plugin.json` — plugin version
  - `.claude-plugin/marketplace.json` — both `metadata.version` and `plugins[0].version`
- The GitHub release tag, plugin version, and marketplace version must always be the same number.
- The `description` field must also match across `plugin.json` and `marketplace.json` (both occurrences).
