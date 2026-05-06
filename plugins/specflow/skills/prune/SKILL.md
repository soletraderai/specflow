---
name: prune
description: Quarterly pruning of stale rules, decisions, agent snapshots, and task-history entries. Per-surface staleness boundaries, append-only archive at admin/archive/{YYYY-Q}-prune.md, user-confirmed delete with restoration round-trip support. Read-mostly until the user accepts a per-item prune.
requires:
  - docs/specflow/admin/decision-log.md
  - docs/specflow/admin/rules/non-negotiable.md
  - docs/specflow/admin/rules/guidelines.md
  - docs/specflow/admin/rules/glossary.md
  - docs/specflow/admin/task-history.json
  - docs/specflow/admin/agents/index.json
produces:
  - docs/specflow/admin/archive/{YYYY-Q}-prune.md (append-only)
  - rule-registry edits, decision-log edits, task-history edits, agent-index edits (only on user-confirmed prunes)
  - docs/specflow/admin/scratch/prune-history.json (run record + drift continuity tracking)
eval: |
  every pruned item exists verbatim in the quarter's archive entry; restoration from the archive round-trips
  to the original state byte-for-byte; no item is pruned without explicit user confirmation; archive is
  append-only (skill never modifies its own archive).
---

# specflow:prune

You are the quarterly registry-pruning cadence skill. Once a quarter — manually invoked at retro time or fired by scheduled cron — you walk four surfaces (rules, decision-log, agent snapshots, task-history), classify each entry against an explicit per-surface staleness boundary, surface candidates with a one-line *why* per candidate, and accept the user's per-item sign-off. Confirmed prunes land byte-for-byte in `admin/archive/{YYYY-Q}-prune.md` before the source registry is touched; restoration from the archive reproduces the source state byte-identical. You never auto-prune, never modify your own archive, never name an AI vendor in user-facing prose.

The four core principles bind here as everywhere. *Think Before Coding* — every staleness boundary is binary and named at the R-level; no implicit "looks stale" judgement. *Simplicity First* — v1 has no archive-retention knob (forever-by-default; v2 enhancement gated on documented consumer ask) and no batch-accept-all option. *Surgical Changes* — the skill mutates user-edited registry files; archive-before-mutate plus per-item confirmation are non-negotiable; `decision-log.md`, `rules/`, and `task-history.json` use a read-only Stage 2 (the archive file is the single source of truth for archive state); only `agents/` mutates at Stage 2 because the live agent registry is the runtime surface. *Goal-Driven Execution* — every phase has inline binary verify steps; round-trip restoration is the load-bearing property byte-identical Stage 1 capture establishes.

This is an **8-phase orchestrator** (A → B → C → D → E → F → G → H). Phases B-E compute candidate sets per surface in parallel (read-only). Phase F walks surfaces in fixed order (rules → decision-log → agent snapshots → task-history) for per-item interactive review — rules first because rule changes have the largest blast radius; task-history last because it is the most conservative pruner. No sub-agent forking — cross-phase state (lock, candidate sets, restoration records, run ledger) lives in working memory until Phase H's append.

---

## Inputs

Two invocation modes (per R1):

- **Manual:** `/prune` — full interactive flow; per-item prompts re-fire on ambiguous input.
- **Cron:** same orchestration in `cron` mode — any branch that would prompt is treated as `defer` with reason `cron-mode-cannot-prompt`.

Auxiliary verbs:

- **Force-unlock:** `/prune --force-unlock` — escape hatch when a prior run crashed (per R8); confirms `y/N` (default N); writes `admin/scratch/prune-{YYYY-Q}.unlock-marker` for the 5-minute manual-priority window.
- **Restore:** `/prune restore --quarter {YYYY-Q} --item {id}` — reverse a single prune; reads the archive entry verbatim, re-inserts into the source surface, records the restoration in `prune-history.json`. Does NOT delete the archive entry.

Tell the user at entry: *"Prune cadence — mode: {manual | cron} for quarter `{YYYY-Q}`. Starting Phase A pre-flight."*

---

## Phase A — Pre-flight: lock, fresh-project check, registry parse, history read

### A.1 Lock acquisition (per R8)

Compute current quarter `{YYYY-Q}`. Attempt atomic create at `admin/scratch/prune-{YYYY-Q}.lock` with body `{ISO-8601 timestamp} {manual|cron}`.

**Implausible-timestamp gate.** If a lock exists, read its body's timestamp. If it is more than 86400 seconds (24 hours) in the past OR more than 3600 seconds (1 hour) in the future, refuse with: *"Lock file at `admin/scratch/prune-{YYYY-Q}.lock` has implausible timestamp `{ts}` (compared to current time `{now}`). Inspect the lock file by hand and use `--force-unlock` if appropriate."* Exit; no registry touched.

**Three nominal cases** after the implausibility gate:

- **(a) No lock** → create the lock; proceed to A.2.
- **(b) Lock age <5400s (90min)** → compute age via filesystem `mtime` as primary clock and the lock body's timestamp as cross-check; if they disagree by >60s, prefer the lock body. Refuse: *"`/prune` is already in flight (started `{timestamp}` via `{source}`). Wait for it to complete, or run `/prune --force-unlock` if the prior run crashed."* Exit.
- **(c) Lock age ≥5400s** → treat as stale; overwrite with fresh timestamp + source; emit *"[stale lock detected for prune-{YYYY-Q} — proceeding]"*; proceed.

**Force-unlock branch.** If no lock present, refuse with *"`/prune --force-unlock` invoked but no lock present at `admin/scratch/prune-{YYYY-Q}.lock`. No-op."* Otherwise prompt *"Force-unlock will discard the in-flight run's state. Confirm? (y/N)"* (default N). On `y`, remove the lock, write `admin/scratch/prune-{YYYY-Q}.unlock-marker` with the unlock timestamp, emit *"[lock cleared at {timestamp}; re-invoke /prune within 5 minutes to start a fresh run]"*. Exit.

**Unlock-marker priority window.** On nominal entry, if the marker exists with a timestamp within the last 300 seconds AND mode is `cron`, refuse with *"`/prune --force-unlock` was invoked at `{ts}`; awaiting manual re-invoke. Cron deferred."* Marker is consumed by the next manual `/prune` OR auto-expires after 300 seconds. The lock is released atomically on every exit path below.

**Verify A.1:** lock exists with current timestamp + invocation source on the proceed path; or skill has exited via a sentinel with no registry touched.

### A.2 Fresh-project check (per R1)

Read the oldest entry across the four registries: oldest `decision-log.md` `**Date:**`; oldest rule's git creation date; oldest agent snapshot's `snapshot_date`; oldest task-history `completed_at`. If the resulting oldest is less than 91 days before today, refuse: *"`/prune` requires at least one quarter of registry history. Oldest entry is `{date}` (`{N}` days). Re-run after `{date + 91 days}`."* Release lock; exit.

**Verify A.2:** oldest-entry date ≥91 days before today, OR exited via the sentinel.

### A.3 Source-registry parse (per R9)

Parse each surface inline. Auto-repair is forbidden.

- **`decision-log.md`** — same tail-only parse `specflow:decision`'s R9 uses: file ends with canonical preamble + `---` (no entries) OR a complete entry with all five canonical bold-prefixed fields + trailing `---`.
- **`rules/non-negotiable.md` / `rules/guidelines.md`** — every rule has YAML frontmatter (`id`, `tier`, `paths`) followed by *Rule*, *Why*, *On violation*, *Exceptions* sections.
- **`agents/index.json`** — validate against the schema in `skills/agent/SKILL.md` (top-level `schema_version: 1`, `agents` array, each entry with `name`, `category`, `source`, `version`, `path`, `snapshot_date`).
- **`task-history.json`** — top-level `tasks` array; every entry has `id`, `feature_id` (or `feature`), `task_id`, `completed_at`.

On any parse failure: *"`{registry-path}` failed parse: `{description}`. Aborting to avoid compounding the corruption. Run `specflow:doctor --feature admin` to validate, or fix by hand and re-run."* Release lock; exit.

**Verify A.3:** all four registries parsed clean, OR exited via the sentinel.

### A.4 Read prune-history.json (per R4)

Read `admin/scratch/prune-history.json` (append-only ledger of `{run_date, surface, candidate_id, status}` records). If absent or empty, hold an empty in-memory ledger; the agent sub-phase (Phase D) will surface no candidates this run. If present, parse and key by `(surface, candidate_id)` for the agent-snapshot two-consecutive-run threshold.

**Verify A.4:** ledger held in working memory (possibly empty); R4's "first-ever run → no candidates" path is wired.

---

## Phase B — Rules staleness detection (per R3 + R6)

### B.1 Citation-resolution helper (Gate 2 tbc-r1-f1)

For each rule in `rules/non-negotiable.md` and `rules/guidelines.md`, walk the *Why* section and extract citations in two forms:

- **Title citations** — quoted strings (single or double quotes) that exact-match an existing decision-log H2 title.
- **Date citations** — `YYYY-MM-DD` substrings that exact-match an existing decision-log entry's `**Date:**` value.

Resolve each to the corresponding decision-log H2 title. **The worked-example rules (`PREFER_LOCAL_TESTS` cites `2026-02-14`; `PREFER_COMPOSITION_OVER_INHERITANCE` cites `2026-01-08`) use the date form, so date-citation tolerance is non-optional.**

### B.2 Supersession scan (per R3)

Once each citation resolves to a title, scan every decision-log entry's `**Related:**` field for `Supersedes: "{resolved title}"` (case-sensitive on the resolved title; the prefix is documented in `specflow:decision`'s R6 + AC-5). A resolved citation whose title appears after `Supersedes: "..."` is superseded.

### B.3 Per-tier candidacy

- **Non-negotiable** — candidate iff at least one resolved citation is superseded. **No dormancy clause** (Round 1 user-edit) — non-violation is the success state. Rules without resolvable citations are never candidates.
- **Guidelines** — candidate iff (a) at least one resolved citation is superseded, OR (b) zero `task-history.json` `addenda[].description` entries and zero `misc-task` entries cite the rule's `id` within `prune.thresholds.guidelines.dormancyDays` (default 365) days. Guidelines without extractable citations fall through to clause (b) only.

### B.4 Per-candidate *why*

Superseded: *"Cited decision-log entry '{resolved title}' was superseded on {date}."* Dormancy: *"Zero references in {N} days (threshold: {threshold})."*

**Verify B:** every candidate has a non-empty *why*; non-negotiable candidates carry a superseded flag; guideline candidates carry a superseded flag or a dormancy flag; the candidate set is keyed by `{rule-id, source-file, source-line-range}`.

---

## Phase C — Decision-log staleness detection (per R2 + R6)

For each `decision-log.md` entry, mark as candidate iff (a) its `**Date:**` is older than `prune.thresholds.decisionLog.ageDays` (default 365) days before today, AND (b) no `task-history.json` `prd_anchor`, no `task-history.json` `decision_log_links[].title`, and no other decision-log `**Related:**` field, contains the candidate's H2 title as a substring within `prune.thresholds.decisionLog.dormancyDays` (default 182) days. Both clauses must fire — pure-age over-fires on still-relevant foundational decisions; pure-dormancy under-fires on entries that are old AND irrelevant.

Per-candidate *why*: *"Age {N}Q (created {date}); no references in last {M} days."*

**Verify C:** every candidate has a non-empty *why*; H2 title and source-line range held in working memory.

---

## Phase D — Agent-snapshot staleness detection (per R4 + R6)

### D.1 First-ever-run path

If the prune-history ledger from A.4 is empty, emit: *"Agent sub-phase: no candidates this run; persistence threshold not yet met. Drift will be evaluated next run."* Skip to Phase E.

### D.2 Two-consecutive-run drift check

For each agent in `admin/agents/specialised/`, mark as candidate iff `prune-history.json` records the same agent with `surface: agents` AND `status: orphan` OR `status: drifted` for the two most-recent prior `/prune` runs. The two-run threshold avoids over-firing on a one-off install hiccup. Drift status comes from `admin/scratch/agent-refresh-{timestamp}.md` reports written by `/specflow:agent refresh`.

Per-candidate *why*: *"Persistent drift across runs `{run-1 date}` and `{run-2 date}` (status: {orphan|drifted})."*

**Verify D:** first-ever-run path emitted the canonical line + empty candidate set, OR each candidate is held with `{agent-name, source-plugin, snapshot-path, index-entry}`.

---

## Phase E — Task-history staleness detection (per R5 + R6)

For each entry in `task-history.json`'s `tasks` array, mark as candidate iff `completed_at` is older than `prune.thresholds.taskHistory.ageDays` (default 365) days before today, AND `superseded_by_retro: true`, AND the entry's `addenda` array has zero entries with `date` within `prune.thresholds.taskHistory.dormancyDays` (default 182) days. All three clauses must fire — task-history is the most conservative pruner; it's the corpus `/insights` reads.

Per-candidate *why*: *"Age {N}Q (completed {date}); superseded by retro; no addenda in last {M} days."*

**Verify E:** every candidate has a non-empty *why*; `id`, JSON-object slice, and addenda array held in working memory.

---

## Phase F — Round-trip restoration capture + interactive review (per R6 + R10 + R11 Stage 1)

### F.1 Capture verbatim restoration records (Gate 2 goal-r1-f1)

For every candidate from Phases B-E, capture the verbatim original-entry slice from the source registry as the **restoration record** held in working memory:

- **`decision-log.md`** — H2 line through entry's trailing `---` (exclusive).
- **`rules/{tier}.md`** — YAML frontmatter through *Exceptions* section's terminating `---`.
- **`agents/index.json`** — JSON object for the entry, plus the file body of `admin/agents/specialised/{name}.md`.
- **`task-history.json`** — JSON object for the entry.

This is the contract Gate 2's goal-r1-f1 made R-level: every accepted prune must round-trip from its archive entry back to the source surface byte-for-byte.

**Verify F.1:** every candidate has a non-empty restoration record; the captured byte-string round-trips when re-pasted (in-memory test before any disk write).

### F.2 Per-surface sub-phase entry prompt (per R6)

Walk surfaces in fixed order: rules → decision-log → agent snapshots → task-history. At each sub-phase entry: *"{Surface} sub-phase: {N} candidates. `proceed` (p) to review, `skip` (s) to defer all candidates from this surface to the next run."*

Accepted: `proceed | skip | p | s`. Empty or any other input re-prompts: *"Choose `proceed` (p) or `skip` (s)."* Cron-mode treats `skip` as default if no interactive input is available; deferral lands in the run report with reason `cron-mode-cannot-prompt`.

### F.3 Per-item review (per R6)

Surface candidates as a numbered list with the *why*. Per-item prompt: *"{N}. {title} — {why}. `keep` (k), `archive` (a), `defer` (d)?"*

Accepted: `keep | archive | defer | k | a | d`. Empty or any other input re-prompts: *"Choose `keep` (k), `archive` (a), or `defer` (d)."* Never auto-defaults; cron-mode treats every per-item prompt as `defer` and records the deferral.

### F.4 Per-item disposition

Hold per surface: **archive** (restoration record + composed archive entry held for Phase G); **defer** (reason recorded); **keep** (reason `user-override` recorded).

**Verify F:** every candidate has a disposition; every `archive` carries a restoration record + composed archive entry; every `defer` and `keep` has a reason string.

---

## Phase G — Archive write (Stage 1) + source treatment (Stage 2) (per R7 + R10 + R11)

### G.1 Compose archive entries (per R7)

For each `archive`-marked candidate, compose the archive block byte-for-byte against the canonical five-field shape: H2 title; blank line; `**Source surface:** {registry path}`; blank line; `**Prune date:** {YYYY-MM-DD}`; blank line; `**Why:** {one-line description}`; blank line; a fenced code block labelled `Original entry` containing the captured restoration record bytes verbatim; blank line; `---`; blank line. Field order is fixed; bytes inside the `Original entry` fence are byte-identical to the captured slice.

`Source surface` values are exactly one of: `admin/rules/non-negotiable.md`, `admin/rules/guidelines.md`, `admin/decision-log.md`, `admin/agents/index.json`, `admin/task-history.json`.

### G.2 Append to `admin/archive/{YYYY-Q}-prune.md` (per R7 + R10)

Create `admin/archive/` if missing; create the per-quarter file on its first archive write; otherwise append. The file is append-only within its quarter; the skill never reads or modifies prior archive files.

**Stage 1 archive-write per item is the transaction surface** (per R10): compose → append → verify Stage 1 → only then mutate the source registry. On archive-write failure: *"Archive write to `admin/archive/{YYYY-Q}-prune.md` failed: `{error}`. Source registry NOT modified — your prune decisions for this sub-phase are not yet applied. Resolve the underlying issue and re-run; the candidate list is regenerated each run."* Exit the sub-phase. Items earlier in the sub-phase whose Stage 2 already succeeded stay mutated; the failure is per-item, not per-sub-phase. Lock released. Re-run is idempotent because already-archived items are skipped on the next attempt.

### G.3 Stage 1 verification (per R11)

After each archive-block append, re-read and confirm: (i) H2 title matches; (ii) `**Source surface:**` names the source path; (iii) `**Prune date:**` equals today's `YYYY-MM-DD`; (iv) `**Why:**` is non-empty; (v) `Original entry` bytes are byte-identical to the captured restoration record. Any Stage-1 failure aborts the per-item write with a structured error; source registry NOT modified for that item.

### G.4 Stage 2 source-registry treatment (Gate 2 surgical-r1-f1)

**Read-only on three surfaces** (decision-log, rules, task-history): the source registry stays unmodified at Stage 2. The archive file is the single source of truth for archive state; Phase 3 consumers (`/insights`, `specflow:prd`, `specflow:task`) cross-reference the archive file. Stage 2 re-reads the source registry and confirms it is byte-identical to its pre-Stage-2 state (no accidental mutation).

**Bespoke Stage 2 on agent snapshots only** — the live agent registry IS the runtime surface; a read-only Stage 2 here would mean a "pruned" agent stays active at runtime, defeating the prune. For each accepted agent-snapshot candidate: move `admin/agents/specialised/{name}.md` to `admin/archive/agents/{YYYY-Q}/{name}.md`; remove the entry from `admin/agents/index.json`. Stage 2 re-reads `agents/index.json` and confirms the entry is no longer present, AND confirms the file at `admin/archive/agents/{YYYY-Q}/{name}.md` exists with non-empty body. The asymmetry is structural (live agent registry IS the runtime surface), not bespoke per surface.

**Verify G:** every accepted item is archived (Stage 1 verified); three-surface source registries are byte-identical to pre-Stage-2; agent-snapshot Stage 2 mutations landed (index entry absent + archive-agents file exists); round-trip restoration on at least one randomly-selected item produces byte-identical output (paste the archive block's `Original entry` content into the source position; diff against the captured restoration record is empty).

---

## Phase H — Run record + final disposition

### H.1 Append to prune-history.json (per R4)

Append a record for this run keyed by `{run_date, quarter, mode}`, with per-surface counts (`candidates`, `archived`, `kept`, `deferred`), the archive path, and per-item entries `{surface, candidate_id, status, reason}` where status is one of `archived | kept | deferred | user-override`. Per-item status is the surface R4's two-consecutive-run agent-drift threshold reads on subsequent runs.

### H.2 Final chat-line summary

Emit exactly one line: *"`{N}` candidates surfaced; `{K}` archived; `{M}` kept; `{P}` deferred. Archive at `admin/archive/{YYYY-Q}-prune.md`. Round-trip restoration test: `{passed|failed}`."*

### H.3 Lock release

Release `admin/scratch/prune-{YYYY-Q}.lock` atomically. Lock release is on every exit path — success, refusal, cancellation, or any sub-phase failure (per R8).

**Verify H:** every accepted item is archived AND treated per surface (read-only on three; Stage-2 mutation on agents); round-trip restoration on a randomly-selected item produced byte-identical output; prune-history.json carries the run record with per-item status; lock released; summary line emitted exactly once. If any verify step fails, refuse to claim success; surface the partial state and the recovery path (the candidate list is regenerated each run; already-archived items are skipped).

---

## Verb: `restore` — reverse a single prune (per R11 round-trip restoration)

`/prune restore --quarter {YYYY-Q} --item {id}` reverses a single archive entry into its source surface.

1. Read `admin/archive/{YYYY-Q}-prune.md`. Locate the archive block whose H2 title matches `{id}` (or whose internal identifier matches for JSON surfaces). Refuse if not found: *"No archive block in `admin/archive/{YYYY-Q}-prune.md` matches `{id}`."*
2. Read `**Source surface:**` to identify the target.
3. **Refusal-on-modified-target:** if the target surface has been modified at the original entry's position since the archive entry was written, refuse: *"Restoration target surface `{path}` has been modified since the archive entry at `{archive-path}` was written. Diff: `{diff}`. Resolve by hand and re-run, or restore manually."* No automatic conflict resolution.
4. For decision-log, rules, task-history: insert the `Original entry` bytes verbatim at the source position (file end for append-only surfaces; the original line range for in-place surfaces).
5. For agent snapshots: copy `admin/archive/agents/{YYYY-Q}/{name}.md` back to `admin/agents/specialised/{name}.md`; re-add the index entry from the archive block's `Original entry` JSON.
6. Append a record to `prune-history.json` with `restored_at: {ISO-8601 timestamp}` and the source identifier.
7. **Do NOT delete the archive entry.** The archive remains the audit trail (per R7's append-only contract).

**Verify restore:** the source surface's restored slice is byte-identical to the archive block's `Original entry` content; prune-history.json carries the restoration record; the archive block is unchanged after restoration.

---

## Failure modes and anti-patterns

Refusal exits — each emits a documented sentinel; a refused exit without a sentinel is a failed run:

- **Registry younger than 1 quarter (R1).** Refuse with date-arithmetic message; lock not acquired (or released on the refused path).
- **Implausible lock timestamp (R8).** Refuse with the implausible-timestamp sentinel; route at `--force-unlock` if appropriate.
- **Concurrent prune detected (R8).** Refuse with the in-flight sentinel; the prior run holds the lock.
- **Source registry corrupt (R9).** Refuse with the parse-failure sentinel; route at `specflow:doctor --feature admin`. No auto-repair.
- **Stage 1 archive-write fails mid-prune (R10).** Halt; surface partial state; archive entries already written remain (the skill never modifies its own archive — Gate 2 simplicity-r1-f1 conceded knob). Lock released.
- **Stage 1 read-back mismatch (R11).** Abort the per-item write; source registry NOT modified for that item.
- **Restoration target modified (restore verb).** Refuse with the diff; user resolves manually. Archive entry unchanged.

User-elected branches:

- **Per-item disposition (R6).** `keep` / `archive` / `defer` per item; ambiguous input re-prompts; never auto-defaults.
- **Per-sub-phase entry (R6).** `proceed` / `skip` per surface; `skip` defers all candidates from that surface to the next run.
- **Force-unlock confirm (R8).** `y / N` (default N on empty input) before clearing the lock.

Refuse to do (anti-patterns):

- **Modify the archive.** Append-only; the skill never prunes its own archive; meta-cleanup is out of scope (Gate 2 simplicity-r1-f1 concession). v1 has no `prune.archiveRetention` knob.
- **Auto-prune without per-item user confirmation.** Every accepted prune requires explicit `archive` from the user (or `cron-mode-cannot-prompt` deferral, never silent acceptance).
- **Batch-accept-all option.** Every item gets explicit `keep` / `archive` / `defer`; `--accept-all` is not a flag.
- **Use a different write-helper for archive entries than `specflow:decision` uses for decision-log.** Archive entries mirror the canonical schema discipline.
- **Auto-default on ambiguous user input.** Re-prompt with literal options.
- **Edit prior-quarter archive files.** Read-only from this skill's perspective.
- **Name any AI vendor or tooling in user-facing output, archive entries, or run-ledger records.** Per the project's CLAUDE.md attribution rule.

---

## Cross-skill integration

- **`specflow:agent refresh`** — produces the drift report `admin/scratch/agent-refresh-{timestamp}.md` with `orphan` / `drifted` / `clean` classifications. `/prune` reads two consecutive runs' worth (via `prune-history.json`) for the agent-snapshot persistence threshold (R4).
- **`specflow:complete`** — task-history.json writer; `superseded_by_retro: true` is the prune-eligibility flag (R5). The append-only contract is preserved — read-only Stage 2 leaves the live source bytes unchanged for Phase 3 consumers.
- **`specflow:decision`** — pruned decision-log entries go through the same write-helper for archive entries (mirror schema). The H2 title + bold-prefixed fields shape is shared; the canonical write path for `decision-log.md` is the canonical write path for archive blocks.
- **`/insights`** — pruned task-history and decision-log entries are excluded from `/insights` clustering. The skill respects `superseded_by_retro` and the archive-presence marker (Phase 3 readers cross-reference `admin/archive/{YYYY-Q}-prune.md` to identify pruned entries on read-only-Stage-2 surfaces).
- **`specflow:doctor`** — when R9's parse-failure refusal fires, the skill routes at `specflow:doctor --feature admin`. `/prune` does not auto-repair.
- **`specflow:scope-change`** — also appends decision-log entries; `/prune`'s decision-log boundary (R2) reads scope-change-written entries on the same terms as `specflow:complete`-written entries.

---

## Open questions (deferred to v2)

- **Cross-quarter rolling-archive partition.** R7 partitions one archive file per quarter; cross-boundary collisions are non-issues because each quarter has its own lock — but interaction at the boundary is unconfirmed in v1. Recommendation: file-naming follows the run-fire's calendar date.
- **Persistence-ledger schema stability.** `prune-history.json` lives in `admin/scratch/`; v1 validates inline at every read; v2 promotion to `admin/` if a future skill needs to read it.
- **Misc-task surface inclusion.** Out of scope for v1; the per-surface architecture supports it as an additive fifth surface.
- **Archive-retention v2 enhancement.** v1 hard-codes forever; `prune.archiveRetention` and `prune.staleLockMinutes` are both v2 knob candidates gated on documented consumer ask.

---

## Reference

- `docs/specflow/features/007-prune-skill/007-prune-skill-prd.md` — full requirements R1-R11 and acceptance criteria AC-1 to AC-11.
- `docs/specflow/features/007-prune-skill/007-prune-skill-interview.md` — six interview rounds.
- `docs/specflow/features/007-prune-skill/debate-log/prd-gate2/manifest.md` — Gate 2 (passed-with-revisions); load-bearing constraints: R3 citation-resolution helper (tbc-r1-f1 block resolved); R11 round-trip restoration (goal-r1-f1 block resolved); R11 read-only Stage 2 on three surfaces + bespoke on agents (surgical-r1-f1 concern resolved); R8 implausible-timestamp gate + unlock-marker priority window (da-r1-f1 concern resolved); R7 forever-by-default with no v1 knob (simplicity-r1-f1 concession).
- `CORE_PRINCIPLES.md` — the four principles bound to every phase verify step.
- `skills/decision/SKILL.md` — sister Phase 3 skill; canonical write helper for `decision-log.md`; mirror schema for archive blocks.
- `skills/complete/SKILL.md` — task-history.json writer; `superseded_by_retro` flag the R5 boundary reads.
- `skills/agent/SKILL.md` — `refresh` verb produces the drift report R4 reads across two consecutive runs.
- `skills/insights/SKILL.md` — sister Phase 3 cadence skill; read-only-Stage-2 archive-presence is the cross-skill signal.
- `templates/agents/standard/principles/goal-driven-reviewer.md` — orphan-AC and orphan-phase lenses applied to every phase verify step.
- `examples/docs/specflow/admin/` — worked-example registries the staleness boundaries are calibrated against.
