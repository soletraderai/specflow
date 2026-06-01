---
name: specflow:learn
description: Background loop — auto-fired by specflow:test after each Phase D feedback capture (not a user-facing command). Repo-local self-learning loop. Reads structured findings emitted by specflow:test runs, clusters them deterministically by signal_pattern, and auto-applies Tier-A additive rules to admin/rules/guidelines.md, admin/CONTEXT.md weak-spots, and admin/config.json new keys. Tier-B candidates (plugin-level changes) and Tier-C conflicts log to scratch for manual review. Append-only on the corpus; additive-only on the registries; per-run cap of 3 auto-applies; min cluster size 3. Five-phase orchestrator — A pre-flight + lock, B clustering, C tier routing, D Tier-A auto-apply, E report + lock release. Direct invocation is supported for debugging / re-runs but no user workflow requires it.
status: v2-new
phase: 3
requires:
  - docs/specflow/admin/plugin-findings.jsonl (the corpus; append-only)
  - docs/specflow/admin/rules/guidelines.md
  - docs/specflow/admin/rules/non-negotiable.md
  - docs/specflow/admin/CONTEXT.md
  - docs/specflow/admin/config.json
produces:
  - docs/specflow/admin/learn/{feature_slug}-learn-{YYYY-MM-DD-HHMMSS}.md (end-of-feature report)
  - docs/specflow/admin/learn/runs.jsonl (append-only run log)
  - docs/specflow/admin/rules/guidelines.md (Tier-A append; .bak preserved)
  - docs/specflow/admin/CONTEXT.md (Tier-A weak-spots append; .bak preserved)
  - docs/specflow/admin/config.json (Tier-A new keys only; .bak preserved)
  - docs/specflow/admin/scratch/plugin-candidates-{date}.md (Tier-B log)
  - docs/specflow/admin/scratch/learn-conflicts-{date}.md (Tier-C log)
  - docs/specflow/admin/scratch/learn-{YYYY-MM-DD-HHMMSS}.lock
eval: |
  every Tier-A write cites >=3 contributing finding_ids in the rule's source_finding_ids frontmatter;
  per-run cap of 3 Tier-A auto-applies honoured; no existing rule body mutated (additive-only); no
  existing config key edited (new keys only); every mutating write has a corresponding .bak; lock
  removed on every exit path; report contains the five required sections; same corpus produces
  byte-identical clusters across two consecutive runs (deterministic).
---

# specflow:learn

Repo-local self-learning. Closes the loop between `specflow:test` (which emits findings) and the project's living rules + context registry (which every other skill already reads). The corpus accretes; deterministic clustering surfaces patterns at the 3-observation threshold; additive auto-application means the system gets smarter without manual triage and without ever editing prior decisions.

This skill is the consumer half of the loop. The producer half — `specflow:test` emitting `plugin-findings.jsonl` entries — is intentionally decoupled. Anything that can append a valid JSONL entry to the corpus drives this skill: `specflow:test`, manual append during exploration, future producers under different names. Decoupling means this skill is fully testable on its own corpus.

This is a **5-phase orchestrator** (A → B → C → D → E). No sub-agent forking; no LLM-as-judge inside the loop (clustering is deterministic). Per-run cap of 3 Tier-A writes is the load-bearing safety valve for the 50-finding-burst case where a full-feature test run dumps a wave of new signal.

The four core principles bind here as everywhere: *Think Before Coding* (every cluster cites its contributing finding_ids verbatim); *Simplicity First* (deterministic clustering, no embeddings, no judge); *Surgical Changes* (additive-only writes; never edit prior rules or config keys); *Goal-Driven Execution* (binary verify inline at every phase; sentinel refusal lines on every exit).

---

## Inputs

The skill is invoked via one of:

- **Manual:** `/specflow:learn [--feature {NNN-slug}]` — runs over the full corpus by default; `--feature` scopes the report to findings whose `feature == {NNN-slug}`. The corpus is still read whole (clustering thresholds need cross-feature signal); the scope only filters the report header + the contributing-finding excerpts.
- **End-of-test auto-invocation:** `specflow:test` may invoke `specflow:learn` at the end of Phase C (full + targeted modes only; never `--plan-only`). The invocation is best-effort: if `plugin-findings.jsonl` doesn't exist yet, this skill no-ops cleanly with a single chat line (see A.3).

Tell the user explicitly which trigger you detected: *"`/specflow:learn` — trigger: `{manual | end-of-test}`. Feature scope: `{NNN-slug | full-corpus}`. Reading corpus."*

---

## Findings corpus schema

`docs/specflow/admin/plugin-findings.jsonl` is one JSON object per line, append-only. Required fields per entry:

```json
{
  "finding_id": "F-20260511-0001",
  "captured_at": "2026-05-11T10:23:00Z",
  "source_run": "019-test-2026-05-11",
  "feature": "019-initial-assessment-per-structure-capture",
  "category": "bug | architecture | template | config",
  "severity": "high | medium | low",
  "affected_component": "specflow:test | specflow:task | specflow:linear | PRD-template | task-template | admin/config.json | admin/rules | other:{short-name}",
  "signal_pattern": "MIGRATION_FILE_COMPLETENESS",
  "proposed_fix": "Short imperative — what should the rule say.",
  "source_evidence": ["apps/backend/prisma/schema.prisma:863", "missing migrations/*_form_section_scope/"]
}
```

Field rules:

- `finding_id` — `F-{YYYYMMDD}-{NNNN}`. Stable across runs; deduplication uses this id.
- `signal_pattern` — SCREAMING_SNAKE_CASE token used for clustering. Same logical issue across runs MUST share this token; that is the deterministic clustering anchor.
- `category` — exactly one of the four documented values. `bug` and `architecture` route to Tier B (plugin-level, never auto-applied). `template` and `config` route to Tier A (repo-local, auto-applied).
- `affected_component` — names a single component. For Tier A, the component decides the destination file (`PRD-template` / `task-template` → `admin/rules/guidelines.md`; `admin/config.json` → `admin/config.json`; everything else → `admin/CONTEXT.md` weak-spots).
- `source_evidence` — array of strings, each a `file:line` citation or a short factual phrase. Surfaces in the report verbatim; truncation is forbidden.

A producer that emits a JSONL line missing any required field MUST be rejected at B.1 schema-drift refusal.

---

## Phase A — Pre-flight: lock, corpus check, registry read

### A.1 Acquire the per-run lock

Compute `{run_ts} = {YYYY-MM-DD-HHMMSS}` from the system date. Create the lock atomically at `docs/specflow/admin/scratch/learn-{run_ts}.lock`. Body: ISO-8601 acquisition timestamp.

Check for any other `learn-*.lock` in `admin/scratch/`:

- **No other lock present** → proceed.
- **Other lock present, body timestamp < 30 minutes ago** → refuse with the literal sentinel chat line:

  *"`/specflow:learn` is in flight (started `{timestamp}`, lock `{path}`). Wait for it to complete, then re-invoke."*

  Exit. Do NOT touch the other lock.

- **Other lock present, body timestamp ≥ 30 minutes ago** → treat as stale. Remove the stale lock; surface the chat line *"[stale learn lock detected at `{path}` — removed]"*; proceed.

The lock is released atomically at the END of Phase E on every exit path (success, refusal, schema-drift, conflict-only). A path that completes without removing its own lock is a failed run.

### A.2 Read the corpus

Read `docs/specflow/admin/plugin-findings.jsonl`. Parse line-by-line; ignore blank lines.

- **File missing** → no-op exit with the chat line:

  *"`plugin-findings.jsonl` not found. Self-learning loop has no corpus yet — no action taken. The corpus appears when `specflow:test` (or any producer) emits its first finding."*

  Release the lock; exit cleanly. This is the expected first-run state, NOT a failure.

- **File present, zero parseable entries** → no-op exit with the chat line:

  *"`plugin-findings.jsonl` exists but contains 0 valid entries. No clustering this run."*

  Release the lock; exit cleanly.

- **File present, ≥1 parseable entry** → continue.

### A.3 Read the registry

Read in parallel:
- `docs/specflow/admin/rules/guidelines.md`
- `docs/specflow/admin/rules/non-negotiable.md`
- `docs/specflow/admin/CONTEXT.md`
- `docs/specflow/admin/config.json`

If any registry file is missing, treat as empty and surface a single chat-line warning: `[learn: {path} not found — treating as empty]`. The skill MUST NOT create scaffolding files; `specflow:setup` owns initial registry creation.

Build an in-memory set of existing rule ids (from both `guidelines.md` and `non-negotiable.md` frontmatter blocks). This set drives conflict detection at C.3.

### A.4 Verify before continuing

- Lock acquired at `admin/scratch/learn-{run_ts}.lock`; body carries acquisition timestamp.
- Corpus parsed; `corpus_size` captured.
- Registry set captured (may be empty).

Tell the user: *"Pre-flight passed. Corpus: `{N}` findings. Registry: `{R}` existing rules. Validating schema."*

Hand off to Phase B.

---

## Phase B — Schema validation + deterministic clustering

### B.1 Validate every corpus entry

For each entry, verify the nine required fields from the **Findings corpus schema** section are present and well-typed. On the first invalid entry, refuse with the literal sentinel chat line:

*"Schema drift detected: `plugin-findings.jsonl` entry `{finding_id | line-number}` missing required field `{field}` (or invalid `category` / `severity` value). The producer emitted a malformed entry. Fix the producer and remove the malformed line before re-invoking."*

Release the lock; exit without writing the report. The refusal is hard because every cluster downstream depends on the schema holding; one malformed entry corrupts the clustering pass.

### B.2 Deduplicate

Group entries by `finding_id`. If a `finding_id` appears more than once, keep the first occurrence and surface a chat line warning: `[learn: duplicate finding_id {id} — kept first occurrence]`. The corpus is append-only; duplicates are a producer bug, not a refusal trigger.

### B.3 Cluster by signal_pattern

For each unique `signal_pattern`, collect:

- `contributing_finding_ids` — the list of `finding_id`s sharing this pattern.
- `count` — `len(contributing_finding_ids)`.
- `categories` — the set of `category` values across contributing findings.
- `severities` — the set of `severity` values.
- `affected_components` — the set of `affected_component` values.
- `excerpts` — for each contributing finding, the verbatim `proposed_fix` string (≤140 chars; truncate at 140 with `…` if longer).
- `evidence` — flat union of every contributing finding's `source_evidence` array, deduplicated.

A cluster qualifies for downstream consideration if `count >= 3`. Clusters with `count < 3` are recorded but emit nothing in Phase D; they appear in the report under "Below threshold."

### B.4 Determinism check

Sort the cluster list by:
1. `signal_pattern` ascending (lexicographic).
2. Within ties, `count` descending.

The sort order is byte-deterministic; two consecutive runs on the same corpus MUST produce identical cluster lists. The report at Phase E.3 is the artefact that proves it.

### B.5 Verify before continuing

- Every parsed entry passed the nine-field schema check.
- Every cluster has `count = len(set(contributing_finding_ids))` (set-counted, never list-counted — repeat finding_ids are A.2 deduplicated, but the safety re-check fires here).
- Sort order is deterministic and stable.

Hand off to Phase C.

---

## Phase C — Tier routing

For each cluster with `count >= 3`, decide the tier:

### C.1 Tier A — repo-local, auto-applied

Trigger conditions (ALL must hold):

- Every contributing finding's `category` is `template` or `config`.
- No contributing finding's `category` is `bug` or `architecture`.
- The cluster's `signal_pattern` does NOT match (case-insensitive substring) any existing rule `id` in the registry set from A.3 (additive-only check; promotion of a guideline to non-negotiable is OUT of scope for v1 — `insights` owns tier-strengthening).
- The cluster's `proposed_fix` set, deduplicated, has cardinality ≤ 2 (≥3 wildly different fixes signal that the `signal_pattern` is too coarse; route to Tier C).

Destination file:

- `affected_component == "admin/config.json"` → `docs/specflow/admin/config.json` (new top-level key only; never edit existing).
- `affected_component ∈ {"PRD-template", "task-template"}` → `docs/specflow/admin/rules/guidelines.md` (append a guideline).
- Everything else → `docs/specflow/admin/CONTEXT.md` (append a "weak spot" line under the "Known weak spots" section; create the section if missing).

Tag the cluster with `tier: "A"` and `destination: {path}`.

### C.2 Tier B — plugin-level candidate, logged only

Trigger conditions:

- Any contributing finding has `category` in `{bug, architecture}`.

These touch the plugin itself (`plugins/specflow/skills/*`, PRD template, task template). They are repo-irrelevant for self-learning — fixing them changes the plugin, not the project. Tag with `tier: "B"`; log destination `docs/specflow/admin/scratch/plugin-candidates-{YYYY-MM-DD}.md`. The user manually decides whether to open a plugin-repo issue.

### C.3 Tier C — conflict or low-quality cluster

Trigger conditions:

- Any of Tier A's preconditions failed for a reason OTHER than category (e.g. `signal_pattern` collides with an existing rule id; `proposed_fix` set has cardinality > 2).

Tag with `tier: "C"`; log destination `docs/specflow/admin/scratch/learn-conflicts-{YYYY-MM-DD}.md` with the failure reason captured.

### C.4 Per-run Tier-A cap

Sort all Tier-A clusters by `count` descending, then `signal_pattern` ascending. Take the first 3. These are the **promotable** Tier-A clusters for this run.

Remaining Tier-A clusters (rank 4+) are tagged `tier: "A-pending"` and surface in the report's "Hit threshold but blocked by cap" section. They will promote on a future run when the cap allows (and the corpus may have added more contributing findings by then, raising their rank).

### C.5 Verify before continuing

- Every `count >= 3` cluster has exactly one of `tier ∈ {"A", "A-pending", "B", "C"}`.
- The Tier-A promotable set is ≤ 3 entries.
- No Tier-A promotable cluster has `signal_pattern` matching any existing registry rule id.

Hand off to Phase D.

---

## Phase D — Tier-A auto-apply

For each Tier-A promotable cluster (≤3 per run), perform the write per its destination. Order: process clusters in the sort order from C.4 (count desc, signal_pattern asc). Each write is atomic: `.bak` first, then write, then re-read to verify the append landed.

### D.1 Backup before any write

Before the first Tier-A write of the run, copy the target files to siblings with `.bak` suffix:

- `admin/rules/guidelines.md.bak`
- `admin/CONTEXT.md.bak`
- `admin/config.json.bak`

Only files that will actually be modified need backups; if no Tier-A cluster routes to `config.json`, skip the config backup. The `.bak` is overwritten on every run — it represents the pre-this-run state, not a deep history.

### D.2 Write a new guideline (destination: `admin/rules/guidelines.md`)

Append the following block to `guidelines.md`:

```markdown
---
id: {signal_pattern}
tier: guideline
paths: ["**/*"]
source_finding_ids: [{comma-separated contributing_finding_ids}]
auto_applied_at: {ISO-8601 timestamp}
auto_applied_by: specflow:learn
---

**Rule:** {short-fix synthesised from the cluster's proposed_fix set; if cardinality 1, use it verbatim; if cardinality 2, render both joined by " AND "}

**Why:** Pattern surfaced via {count} `specflow:test` findings across feature(s) {comma-separated unique feature values}.

- (id `{contributing_finding_id_1}`): "{verbatim proposed_fix excerpt}"
- (id `{contributing_finding_id_2}`): "{verbatim proposed_fix excerpt}"
- (id `{contributing_finding_id_3}`): "{verbatim proposed_fix excerpt}"

**On violation:** Surface during the relevant skill; log a misc-task if non-trivial.

**Exceptions:** Edit this entry to narrow `paths:` once you know the right scope. Auto-applied with `paths: ["**/*"]` as a conservative wide default.

---
```

The `paths: ["**/*"]` wide default is deliberate (per Rule R5 of `insights` — never aggressively widen; here we never aggressively narrow either, because the producer didn't tell us the scope). The user narrows at edit-time when they next read the file.

### D.3 Write a CONTEXT.md weak-spot line (destination: `admin/CONTEXT.md`)

Locate the `## Known weak spots` section header. If missing, append it at the end of the file with a one-line intro:

```markdown

## Known weak spots

Surfaced by `specflow:learn`. Each line points at a recurring signal worth designing for. Edit or delete entries that no longer match reality.

```

Append one line per Tier-A cluster routed here, in this format:

```markdown
- `{signal_pattern}` ({count} findings, severity `{max-severity-across-contributors}`): {short synthesis of proposed_fix} (source: F-list `{comma-separated finding_ids}`)
```

### D.4 Write a config.json new key (destination: `admin/config.json`)

Parse `config.json` as JSON. The cluster's `proposed_fix` MUST nominate the new key path and value as a structured suffix: `KEY=path.to.key; VALUE=<json-value>` somewhere in one of the contributing findings' `proposed_fix` strings. If no contributing finding nominates the key/value in this form, demote the cluster to Tier C (log to conflicts scratch with reason `config-cluster-missing-key-value-nomination`) and skip.

If the nominated key path already exists in `config.json`, demote to Tier C with reason `config-key-already-set`. Existing keys are never edited.

If the key is novel, set it (deep-create intermediate objects as needed) and write the result back, preserving 2-space JSON indentation and trailing newline.

### D.5 Verify each write

After each Tier-A write:

1. Re-read the destination file.
2. Confirm the appended block / line / key is present byte-for-byte.
3. Confirm the file still parses (Markdown: file readable; JSON: `JSON.parse` succeeds).
4. If verification fails, restore from `.bak`, surface the chat line *"[learn: write to `{path}` failed verification — restored from .bak]"*, and demote the cluster to Tier C with reason `post-write-verification-failed`.

### D.6 Log Tier B + Tier C clusters to scratch

For every Tier-B cluster, append to `docs/specflow/admin/scratch/plugin-candidates-{YYYY-MM-DD}.md`:

```markdown
## {signal_pattern} ({count} findings)

- Affected: {comma-separated affected_components}
- Severity: max `{max-severity}`
- Proposed fixes:
  - "{verbatim proposed_fix #1}"
  - "{verbatim proposed_fix #2}"
  - "{verbatim proposed_fix #3}"
- Contributing finding_ids: {list}
- Evidence:
  - {evidence_line_1}
  - {evidence_line_2}

```

Create the file if missing; append below if present.

For every Tier-C cluster, append to `docs/specflow/admin/scratch/learn-conflicts-{YYYY-MM-DD}.md`:

```markdown
## {signal_pattern} ({count} findings) — conflict: {reason}

- {short failure-reason narrative}
- Contributing finding_ids: {list}

```

### D.7 Verify before continuing

- Every Tier-A promotable cluster either wrote successfully (D.5 passed) or was demoted to Tier C with a recorded reason.
- Tier-B and Tier-C scratch files exist for every cluster of those tiers (idempotent across same-day re-runs — entries dedupe by `signal_pattern`).
- No existing rule body in `guidelines.md` or `non-negotiable.md` was mutated.
- No existing key in `config.json` was edited.

Hand off to Phase E.

---

## Phase E — Report + run-log + lock release

### E.1 Compose the end-of-feature report

Report path: `docs/specflow/admin/learn/{feature_slug | "full-corpus"}-learn-{run_ts}.md`. Create `admin/learn/` if missing.

Required sections (in this order). Every section header MUST be present even when empty; empty sections render the literal sentinel body line `_(none this run)_`.

```markdown
# Self-learning summary — {feature_slug | "full-corpus"} — {run_ts}

**Trigger:** `{manual | end-of-test}`
**Corpus size:** {N} findings ({delta_new_since_last_run} new since last `specflow:learn` run)
**Clusters above threshold:** {count >= 3 cluster count}

## Auto-applied this run (Tier A, cap 3)

- `{signal_pattern}` → `{destination_path}` ({count} contributing findings: `{id_list}`)
- ...

## Hit threshold but blocked by cap (next run will promote)

- `{signal_pattern}` ({count} findings) — rank {N+1}, awaiting cap slot
- ...

## Plugin-level candidates (Tier B — NOT auto-applied; logged for manual review)

- `{signal_pattern}` ({count} findings, affects `{components}`) → `admin/scratch/plugin-candidates-{date}.md`
- ...

## Conflicts (Tier C — logged, no action)

- `{signal_pattern}` ({count} findings) — reason: `{conflict_reason}` → `admin/scratch/learn-conflicts-{date}.md`
- ...

## Below threshold (need more occurrences)

- {N} clusters at 2 occurrences
- {M} clusters at 1 occurrence
- Total findings sitting under the cluster threshold: {total}

## System learned

{T} new entries now live in `admin/`:
- `admin/rules/guidelines.md`: +{G} guidelines
- `admin/CONTEXT.md`: +{C} weak-spot lines
- `admin/config.json`: +{K} new keys

Future skill invocations will read the updated registries automatically. Read the diff in your next commit to review.
```

Write the report. Overwrite if a same-`run_ts` path exists (a same-second re-run is rare but the second-precision timestamp is enough; tighter precision is overkill).

### E.2 Append to the run log

Append one JSON line to `docs/specflow/admin/learn/runs.jsonl`:

```json
{
  "run_id": "{run_ts}",
  "started_at": "{ISO-8601 of A.1}",
  "completed_at": "{ISO-8601 of now}",
  "trigger": "manual | end-of-test",
  "feature_scope": "{NNN-slug | null}",
  "corpus_size": {N},
  "clusters_total": {C_total},
  "clusters_above_threshold": {C_threshold},
  "tier_a_applied": {A_applied},
  "tier_a_pending": {A_pending},
  "tier_b_logged": {B_logged},
  "tier_c_logged": {C_logged},
  "files_written": ["admin/rules/guidelines.md", "admin/CONTEXT.md"],
  "report_path": "{report_path}"
}
```

Append-only. A run that overwrites or edits any line is a failed run.

### E.3 Release the lock

Remove `docs/specflow/admin/scratch/learn-{run_ts}.lock`. The remove fires on every exit path the orchestrator reaches (success, schema-drift refusal at B.1, concurrent-trigger refusal at A.1, corpus-missing no-op at A.2, post-write demotion at D.5). A path that reaches Phase E without removing its own lock is a failed run.

### E.4 Emit the chat-line summary

```
specflow:learn run complete.

Corpus: {N} findings → {C} clusters → Tier-A applied: {A}, blocked-by-cap: {AP}, Tier-B logged: {B}, Tier-C logged: {C}.

System learned: +{G} guidelines, +{C_lines} CONTEXT weak-spots, +{K} config keys.

Report: admin/learn/{report_filename}
Read the diff in your next commit to review the auto-applied changes.
```

When `A == 0 && AP == 0 && B == 0 && C == 0`, emit instead:

```
specflow:learn run complete. Corpus: {N} findings. No clusters above the 3-observation threshold this run. System unchanged.
```

### E.5 Verify before declaring done

1. Report at `admin/learn/{...}-learn-{run_ts}.md` has all six section headers (empty sections carry the sentinel).
2. Run log has exactly one new line appended.
3. Lock file no longer exists.
4. Every Tier-A write block in the report names ≥3 contributing `finding_id`s.
5. No existing rule body was mutated (re-read `guidelines.md` / `non-negotiable.md`; confirm the bytes before the appended block are identical to the `.bak`).
6. No existing config key was edited (re-read `config.json`; confirm every pre-existing key/value is byte-identical to `.bak`).
7. Same corpus → same clusters: a second consecutive run on the same corpus produces an identical clusters list (deterministic). The skill itself does not run twice automatically; this is a property the report's sort order must satisfy.

If any verify step fails, surface the failure and refuse to claim success.

---

## Failure modes

Each maps to a documented sentinel refusal exit; never silent retry.

- **Concurrent lock present (A.1)** — in-flight sentinel naming lock path + timestamp. First-to-start owns the lock.
- **Corpus missing or empty (A.2)** — no-op exit with explanatory chat line; NOT a failure. The first specflow run on a fresh repo hits this path.
- **Schema drift on any corpus entry (B.1)** — schema-drift sentinel naming the offending `finding_id` or line number + missing field. Producer bug; refuse hard.
- **Duplicate finding_id (B.2)** — single-line warning; kept-first-occurrence semantics; run continues.
- **No clusters above threshold (C → D)** — empty-report path; "System unchanged" chat line; runs.jsonl records zero promotions.
- **Tier-A post-write verification failure (D.5)** — restore from `.bak`; demote cluster to Tier C with reason `post-write-verification-failed`; run continues.
- **Config cluster missing KEY=/VALUE= nomination (D.4)** — demote to Tier C with reason `config-cluster-missing-key-value-nomination`.
- **Config key already exists (D.4)** — demote to Tier C with reason `config-key-already-set`.

A refused exit without a documented sentinel line is a failed run.

---

## Anti-patterns (refuse to do)

- **Edit existing rules.** Promotion is additive. A run that mutates an existing rule's body or frontmatter is a failed run. Demotion / removal lives in `/prune`.
- **Edit existing config keys.** New top-level keys only; existing keys are never touched.
- **Auto-apply more than 3 Tier-A clusters per run.** The cap is the load-bearing safety valve for the 50-finding-burst case. Per-run only; the cap resets on the next invocation.
- **Apply Tier B automatically.** Tier B = plugin-level changes (touch `plugins/specflow/`). Logged to scratch only; the user routes manually.
- **Use embeddings or LLM-as-judge for clustering.** Deterministic field-shape (`signal_pattern`) match only. Same corpus → byte-identical clusters.
- **Auto-promote a guideline to non-negotiable.** Tier-strengthening lives in `insights` (sister Phase 3 skill). This skill only ever creates new `tier: guideline` entries.
- **Skip the `.bak` step.** Every mutating write makes a `.bak` first. A write without a backup is a failed run.
- **Parse the markdown report from `specflow:test`.** This skill reads `plugin-findings.jsonl` only. If a producer wants its findings consumed, it appends to the JSONL.
- **Create registry scaffolding.** If `admin/rules/guidelines.md` doesn't exist, surface a warning and skip — never auto-create. `specflow:setup` owns initial scaffolding.
- **Aggregate across repos.** v1 reads only the local repo's corpus. Cross-repo plugin improvements are a separate, slower, human-driven loop.
- **Name any AI vendor or tooling in user-facing output.** Per the project's CLAUDE.md attribution rule.

---

## Cross-skill integration

- **`specflow:test`** — primary producer. Emits findings to `plugin-findings.jsonl` (when it grows that capability). For now, the corpus can be populated by any producer or manually during exploration.
- **`specflow:setup`** — owns initial creation of `admin/rules/*` and `admin/CONTEXT.md`. This skill assumes those files exist; it warns and skips if they don't.
- **`/insights`** — sister Phase 3 skill. `insights` mines `task-history.json` for project-domain lessons; `specflow:learn` mines `plugin-findings.jsonl` for process-meta findings. Both write to `admin/rules/guidelines.md`, but never to the same `id` (Tier-A's C.3 conflict check filters that case). Cadences are independent: `insights` runs monthly across the full task corpus; `specflow:learn` runs at the end of every `specflow:test` invocation.
- **`/prune`** — quarterly counterpart. This skill never demotes or removes rules; `/prune` owns that contract. The `source_finding_ids` frontmatter field on auto-applied rules is the audit trail `/prune` reads when deciding whether a rule has aged out.
- **`specflow:complete`** — unrelated to this skill's corpus. Project-task retros land in `task-history.json`; plugin-process findings land in `plugin-findings.jsonl`. They share the rules-registry destination only.

---

## Reference

- `skills/insights/SKILL.md` — sister Phase 3 skill; deterministic clustering on a different corpus; the rule-frontmatter shape and `.bak` discipline are inherited verbatim.
- `skills/complete/SKILL.md` — sister producer of project-level lessons; schema-validated corpus pattern adopted here.
- `skills/feedback-loop-audit/SKILL.md` — seeds `admin/CONTEXT.md`; this skill appends weak-spot lines to its "Known weak spots" section.
- `templates/admin/rules/guidelines.md` — destination shape for Tier-A guideline appends.
- `templates/orchestrator-pattern.md` — five sequential phases; no sub-agent forking.
- `CORE_PRINCIPLES.md` — the four principles bound to every phase verify step.
