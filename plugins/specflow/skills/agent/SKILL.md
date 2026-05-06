---
name: agent
description: Per-repo agent registry — add, remove, list, or refresh agents in admin/agents/. Standard agents (lifecycle + principles) ship with the plugin; specialised agents are matched to the project's stack and snapshotted in-repo for review.
requires:
  - docs/specflow/admin/agents/index.json
  - docs/specflow/admin/environment.json
produces:
  - docs/specflow/admin/agents/{specialised|standard}/{name}.md  (snapshotted bodies)
  - docs/specflow/admin/agents/index.json  (updated)
eval: |
  every agent listed in admin/agents/index.json resolves to a file on disk;
  every snapshotted file has a non-empty body;
  index.json validates against the documented schema.
---

# specflow:agent

Per-repo agent registry. Lists, adds, removes, and refreshes the agents available on this project. The registry lives at `docs/specflow/admin/agents/` — a browsable folder of markdown files plus an `index.json` manifest. Standard agents (lifecycle + principle reviewers) ship with the plugin; specialised agents are pulled from installed marketplaces and **snapshotted** into the repo so they're pinned, diffable, and reviewable in PRs. The skill is grounded in the four principles: *Think Before Coding* (every add asks the user to confirm collisions and stack-match reasoning); *Simplicity First* (four verbs only, no auto-magic refresh); *Surgical Changes* (snapshots are pinned — `refresh` never overwrites without consent); *Goal-Driven Execution* (every verb ends with binary verify steps).

---

## Triggers

- `/specflow:agent` — defaults to `list`.
- `/specflow:agent list`
- `/specflow:agent add {name}` / `/specflow:agent add {name} --source {plugin-name}`
- `/specflow:agent remove {name}`
- `/specflow:agent refresh`

Auto-invocations:
- `specflow:setup` calls `refresh` once at end of setup to seed the global agent index.
- `specflow:upgrade` calls `refresh` post-migration to surface any new standard agents shipped with the plugin.
- `specflow:doctor` validates the index via the eval check above; refuses to pass if any indexed agent's file is missing.

---

## Index schema

`docs/specflow/admin/agents/index.json`:

```json
{
  "schema_version": 1,
  "agents": [
    {
      "name": "orchestrator",
      "category": "lifecycle",
      "source": "specflow",
      "version": "2.0.0",
      "path": "admin/agents/standard/lifecycle/orchestrator.md",
      "snapshot_date": "2026-05-06",
      "upstream_path": "templates/agents/standard/lifecycle/orchestrator.md"
    },
    {
      "name": "frontend-developer",
      "category": "specialised",
      "source": "frontend-mobile-development",
      "version": "1.4.2",
      "path": "admin/agents/specialised/frontend-developer.md",
      "snapshot_date": "2026-04-12",
      "upstream_path": "~/.claude/plugins/frontend-mobile-development/agents/frontend-developer.md",
      "stack_match_reason": "Detected React + TypeScript stack via environment.json"
    }
  ]
}
```

`category` is one of `lifecycle`, `principles`, `specialised`. Lifecycle and principles together form the "standard" set; they live under `admin/agents/standard/{lifecycle|principles}/`. Specialised agents live under `admin/agents/specialised/`.

---

## Verb: `list`

The default verb. Read-only. Print the index grouped by category.

**Behaviour:**
1. Read `admin/agents/index.json`. If absent, print "Agent index missing — run `/specflow:setup` or `/specflow:upgrade` to seed it." and exit.
2. Read `admin/environment.json` for source-plugin version cross-reference (informational only).
3. Emit a markdown table per category: `name | source | version | snapshot_date | path`.
4. End with a one-line summary: `{N} lifecycle, {M} principles, {K} specialised`.

**Inputs:** none.

**Outputs:** chat-only. No file writes.

**Verify:**
1. Output is well-formed Markdown table.
2. Every row's `path` resolves on disk (`test -f docs/specflow/{path}`).
3. Summary count matches the row count per category.

**Failure modes:**
- Index missing → suggest `setup` / `upgrade`. Do not synthesise an index.
- A listed path doesn't resolve → mark the row with `MISSING` and surface as a `doctor` follow-up. List still completes.

---

## Verb: `add {name}` / `add {name} --source {plugin-name}`

Add a specialised agent from an installed plugin. Snapshot the agent body into `admin/agents/specialised/{name}.md` (pinned — no future drift) and append an entry to `index.json`.

**Behaviour:**
1. Resolve the source plugin from the global agent index (built by `setup` and refreshed by `refresh` — consult `admin/environment.json` `agents` section for the canonical mapping of agent → source plugin).
2. If `{name}` matches exactly one source, proceed. If it matches multiple sources (collision across marketplaces), print all candidates with their source plugin and require the user to re-run with `--source {plugin-name}`. Refuse without disambiguation.
3. If `{name}` matches zero sources, refuse with: "Agent `{name}` not found in any installed plugin. Run `/specflow:agent refresh` if you've recently installed a new marketplace, or `gh plugin install {plugin-name}` to add the source plugin."
4. If `--source {plugin-name}` is provided, validate that the plugin is installed and exposes an agent named `{name}`. Refuse if not.
5. Read the upstream agent file from the plugin's agent directory.
6. If `admin/agents/specialised/{name}.md` already exists, prompt before overwrite: "Specialised agent `{name}` already snapshotted from `{existing-source}` on `{existing-date}`. Overwrite with `{new-source}` snapshot? (y/n)".
7. Write the upstream body verbatim to `admin/agents/specialised/{name}.md`.
8. Append the entry to `index.json` under `agents` with `category: "specialised"`, `snapshot_date: {today}`, `version` = source plugin's installed version, `upstream_path` = absolute path read.
9. Print confirmation: `Added {name} from {plugin-name}@{version}. Snapshot pinned at {path}.`

**Inputs:** `{name}` (required), `--source {plugin-name}` (optional; required on collision).

**Outputs:** `admin/agents/specialised/{name}.md`, updated `admin/agents/index.json`.

**Verify:**
1. `admin/agents/specialised/{name}.md` exists and is non-empty.
2. `index.json` has a matching entry with `name`, `source`, `version`, `path`, `snapshot_date`, `upstream_path` populated.
3. Re-read `index.json` — every listed agent resolves on disk.

**Failure modes:**
- **Source plugin not installed** — refuse; suggest `gh plugin install {plugin-name}` then retry.
- **Name collision across plugins** — confirm with user; require `--source` to disambiguate.
- **Upstream file missing or empty** — refuse; report the `upstream_path` and suggest `refresh` to re-scan.
- **`index.json` write fails (e.g., schema violation)** — roll back the `.md` write so the registry stays consistent.

---

## Verb: `remove {name}`

Remove a specialised agent. Backs up the file before deleting; updates `index.json`.

**Behaviour:**
1. Read `index.json`. Locate the entry for `{name}`.
2. If `{name}` has `category: "lifecycle"` or `"principles"`, refuse with: "Standard agents are plugin-shipped and cannot be removed via this skill. To remove one upstream, edit `templates/agents/standard/{category}/` in the plugin source and ship a new plugin version. To suppress one project-side, file a `misc-task` documenting the reason."
3. If `{name}` is not in the index, refuse with: "Agent `{name}` is not in the registry. Run `/specflow:agent list` to see what's registered."
4. Read the snapshot body from `admin/agents/specialised/{name}.md`.
5. Write the body to `admin/scratch/agent-removed-{name}-{YYYY-MM-DD-HHMM}.md` (timestamp suffix prevents collisions across same-day removals).
6. Delete `admin/agents/specialised/{name}.md`.
7. Remove the entry from `index.json`.
8. Print confirmation: `Removed {name}. Backup at {scratch-path}. Reversible until scratch is cleaned.`

**Inputs:** `{name}` (required).

**Outputs:** removed snapshot file; backup at `admin/scratch/agent-removed-{name}-{timestamp}.md`; updated `index.json`.

**Verify:**
1. `admin/agents/specialised/{name}.md` no longer exists.
2. `admin/scratch/agent-removed-{name}-{timestamp}.md` exists with the original body (non-empty).
3. `index.json` has no entry for `{name}`.

**Failure modes:**
- **Trying to remove a standard agent** — refuse with explanation that standard agents are plugin-shipped.
- **Snapshot file missing but index entry exists** (registry drift) — log a warning, remove the index entry only, skip the backup write. Surface as a `doctor` follow-up.
- **Scratch write fails** — refuse to delete the snapshot. Removal must be reversible.

---

## Verb: `refresh`

Re-scan installed plugins; refresh the global agent index that `setup` builds; surface orphans and new standards. Does NOT auto-update specialised snapshots — those are pinned for review.

**Behaviour:**
1. Scan `~/.claude/plugins/*/` for agent definitions. Build an in-memory map of `agent-name → source-plugin@version → upstream-path`.
2. Update the `agents` section of `admin/environment.json` with the fresh map.
3. **For standard agents** (any agent shipped by the `specflow` plugin under `templates/agents/standard/{lifecycle,principles}/`): if a new agent has appeared upstream that isn't in `admin/agents/standard/{category}/`, copy it in and add an entry to `index.json`. This is additive — standards are plugin-shipped, not user-tuned, so refresh applies new ones automatically.
4. **For specialised agents** (everything in `admin/agents/specialised/`): for each entry in `index.json` with `category: "specialised"`:
   - If the source plugin is no longer installed → mark as **orphan** in the drift report. Do not delete the snapshot.
   - If the source plugin is installed but the upstream body differs from the snapshot → mark as **drifted** with a unified diff in the drift report.
   - If the source plugin is installed and the upstream body matches → mark as **clean**.
5. Write the drift report to `admin/scratch/agent-refresh-{YYYY-MM-DD-HHMM}.md` with three sections: `Orphans`, `Drifted`, `New standards added`.
6. For each drifted specialised agent, prompt the user: `Re-snapshot {name} from {source}@{new-version}? (y/n/skip-all)`. On `y`, overwrite the snapshot, update `version` and `snapshot_date` in `index.json`. On `n` or `skip-all`, leave the snapshot pinned.
7. For orphans, do nothing automatically — surface them in chat with the suggestion: `Run /specflow:agent remove {name} to delete the orphan, or keep the snapshot as a read-only reference.`
8. Print summary: `{N} standards refreshed (auto), {M} specialised drifted ({K} re-snapshotted, {L} kept), {P} orphans surfaced.`

**Inputs:** none.

**Outputs:** updated `admin/environment.json` agents section; updated `admin/agents/index.json`; possibly new files under `admin/agents/standard/`; drift report at `admin/scratch/agent-refresh-{timestamp}.md`.

**Verify:**
1. `index.json` `schema_version` unchanged (`1`).
2. Every previously-listed standard agent is still listed (or upgraded with bumped `version`).
3. Drift report exists at `admin/scratch/agent-refresh-{timestamp}.md` if any specialised agents diverged from upstream.
4. Every standard agent file in the plugin's `templates/agents/standard/` has a corresponding entry in `index.json`.

**Failure modes:**
- **No plugins installed** — refresh becomes a no-op for the index; environment.json still updates with an empty `agents` array. Print: "No installed plugins detected. Standard agent set unchanged."
- **Upstream read fails on a specific plugin** (corrupt install) — log the plugin, skip its agents, continue with the rest. Surface in summary.
- **User declines all drift prompts** — refresh still completes; pinned snapshots remain pinned. The drift report is the audit trail for next time.

---

## Cross-skill integration

- `skills/setup/SKILL.md` — invokes `agent refresh` once at end of setup to seed the index. The standard agent set is copied into `admin/agents/standard/` during setup; specialised matching happens during tech detection (separate step) but the registry index is finalised by this skill's `refresh`.
- `skills/upgrade/SKILL.md` — invokes `agent refresh` post-migration to surface any new standard agents shipped with the plugin upgrade. Drift on specialised agents is reported but not auto-applied.
- `skills/doctor/SKILL.md` — runs the `agents.index_integrity` check: for each entry in `index.json`, the file at `path` must exist and be non-empty. If any fail, `doctor` reports FAIL with fix `Run /specflow:agent refresh, or /specflow:agent remove {name} to drop the broken entry.`

---

## What you MUST NOT do

- **Do not modify standard agents project-side.** Standards are plugin-shipped. To change one, edit `plugins/specflow/templates/agents/standard/{category}/{name}.md` upstream and ship a new plugin version. (Surgical Changes — adjacent edits to plugin-shipped files are out of scope for this skill.)
- **Do not auto-overwrite specialised snapshots on `refresh`.** Snapshots are pinned for review. The user owns the decision to re-snapshot. (Think Before Coding.)
- **Do not delete a snapshot without a backup.** `remove` writes to `admin/scratch/` first, then deletes. (Goal-Driven Execution — reversibility is a verify step.)
- **Do not invent agent sources.** If `{name}` doesn't resolve to an installed plugin, refuse and suggest installation. Do not guess the source plugin from the name.
- **Do not skip verify steps because the operation looks "obvious".** Every verb ends with binary verify; partial success without verification is failure.

---

## Verify before declaring done

Per-verb verify steps are listed inline above. The skill-level verify is the `eval:` field at the top: every agent listed in `index.json` resolves to a file on disk; every snapshotted file has a non-empty body; `index.json` validates against the schema. If any of these fail after a verb runs, surface the failure and point at the next remediation step (usually `doctor` or another verb).

---

## Open questions

- **Cross-marketplace name collision policy.** Current rule: require `--source` to disambiguate. Long-term may add a marketplace-priority config so common collisions resolve automatically (`config.agentSourcePriority: ["specflow", "frontend-mobile-development", ...]`). Not in scope for Phase 2.
- **Snapshot refresh cadence.** Currently manual via `agent refresh`. May add a `config.autoRefreshOnSetup: true` flag so re-running `setup` triggers a refresh before tech detection. Trade-off: more network/disk work on every setup vs the user forgetting to refresh after a plugin upgrade.
- **`remove --keep-snapshot` flag.** Useful to mark an agent as orphaned (drop the index entry) but keep the file as a read-only reference. Not implemented yet — current `remove` always deletes the file. File a `misc-task` if a concrete use case appears.
- **`specflow:doctor --clean-scratch` flag.** Phase 1 doctor doesn't have this flag. Until it lands, `admin/scratch/agent-removed-*.md` files accumulate; users prune manually with `rm`. Tracked as a Phase 2 doctor enhancement.

---

## Reference

- `v2/docs/PRD.md` Appendix K — full spec for the agents directory and this skill.
- `v2/docs/PRD.md` Appendix M — environment inventory schema (the `agents` section is what `refresh` updates).
- `examples/docs/specflow/admin/agents/index.json` — reference index file.
- `templates/agents/standard/` — source-of-truth for the standard agent set.
- `skills/setup/SKILL.md`, `skills/upgrade/SKILL.md`, `skills/doctor/SKILL.md` — wired-in callers.
