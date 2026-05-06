---
name: insights
description: Surfaces recurring patterns from admin/task-history.json on a monthly cadence. Two-pass deterministic clustering (field-shape exact-match + token-frequency n-grams). Proposes rule-registry promotions (observation → guideline → non-negotiable) with at-least-3-observation evidence per proposal. Read-only on the corpus; every promotion requires explicit user accept-edit-reject. Auto-fires never; manual /insights and user-wired cron only.
requires:
  - docs/specflow/admin/task-history.json
  - docs/specflow/admin/rules/non-negotiable.md
  - docs/specflow/admin/rules/guidelines.md
  - docs/specflow/admin/rules/glossary.md
  - docs/specflow/admin/decision-log.md (optional secondary signal)
  - docs/specflow/admin/config.json (insights.cadence, insights.minCorpusSize)
produces:
  - docs/specflow/admin/insights/{YYYY-MM}-report.md (replaced-in-place on within-month re-run)
  - docs/specflow/admin/insights/{YYYY-MM}-runs.jsonl (append-only)
  - docs/specflow/admin/rules/{guidelines.md, non-negotiable.md, glossary.md} (only on user-confirmed promotions)
  - docs/specflow/admin/scratch/insights-{YYYY-MM}.lock (per-month concurrent-trigger guard)
eval: |
  every promotion proposal cites >=3 unique contributing task-history.json ids; every cluster in the report has
  len(unique_contributing_ids) >= 3; no cluster_source value is "semantic" in v1 output; no auto-promotion (every
  rule-registry write requires explicit user choice 1|2|3); refuses to run on corpus_size <
  config.json.insights.minCorpusSize (default 10); refuses on schema-drift in task-history.json; lock file at
  admin/scratch/insights-{YYYY-MM}.lock removed on every exit path.
---

# specflow:insights

You are the self-evolution skill that closes the loop between the corpus `specflow:complete` writes and the rules registry the project lives by. Each month the read-mostly pattern miner runs over `admin/task-history.json`, surfaces clusters above a 3-observation threshold via two deterministic passes, and proposes rule-registry promotions (observation → guideline → non-negotiable) the user accepts, edits, or rejects in chat. Without this skill the corpus is write-only — six months of retros land in `task-history.json`, the lessons are captured but never aggregated, and the next time the same gotcha surfaces nobody connects it to the prior occurrences.

This is a **7-phase orchestrator** (A → B → C → D → E → F → G). The skill has no sub-agent forking — clustering is deterministic and runs in the parent context; the heavy discipline is the schema-stability check, the byte-deterministic clustering passes, the lock-file race protection (per `templates/orchestrator-pattern.md` per-month lock at `admin/scratch/insights-{YYYY-MM}.lock`), and the user-confirmed promotion-write surface. Parent context budget ≤8K tokens for a typical 15-entry corpus with five clusters; smaller than `specflow:scope-change` because no multi-skill orchestration fires.

The four core principles bind here as everywhere: *Think Before Coding* (every cluster cites its contributing ids verbatim, every promotion proposal renders the full rule body — no hidden inference); *Simplicity First* (deterministic clusters, no embedding store, no LLM judgement inside the loop — Karpathy autoresearch read-only-eval-inside-the-loop discipline); *Surgical Changes* (the active report replaces in place, the runs log appends only, the rules registry mutates only on explicit user accept); *Goal-Driven Execution* (every phase has an inline binary verify step, every refusal exits with a sentinel chat line — zero silent failures).

---

## Inputs

The skill is invoked via one of (per R6 + AC-6):

- **Manual:** `/insights [--feature {NNN-slug}]` — user-elected, runs against the full corpus by default; the optional `--feature` flag scopes the cluster passes to entries whose `feature == {NNN-slug}` (the report still writes to the active month's path; the scope is recorded in the runs.jsonl entry).
- **Scheduled cron** — user-wired GH Actions cron / system cron / Linear automation invokes the same CLI form. The skill itself NEVER installs a scheduler; the cadence knob `config.json.insights.cadence` is informational (per R7's load-bearing-assumption note: cluster detection and refusal logic are cadence-independent).

Tell the user explicitly which trigger you detected: *"`/insights` for `{YYYY-MM}` — trigger: `{manual | cron}`. Reading config and corpus."*

---

## Phase A — Pre-flight: config, corpus check, lock acquisition

### A.1 Read `config.json.insights` (per R6, R10)

Read `docs/specflow/admin/config.json`. Resolve two knobs:

- `insights.cadence` — must be one of `"weekly" | "monthly" | "quarterly" | "manual-only"`. Default `"monthly"`. Any other value refuses with the literal sentinel chat line:

  *"`config.json.insights.cadence` value `{value}` is not one of the documented values (`weekly | monthly | quarterly | manual-only`). Update the config and re-fire."*

  Exit without writing the report or the runs.jsonl. The threshold is read fresh on every invocation; no caching.

- `insights.minCorpusSize` — positive integer. Default `10`. Used at A.3.

If `config.json` itself is missing, treat as defaults (`monthly` + `10`) and surface the chat-line `[insights: config.json not found — using defaults cadence=monthly minCorpusSize=10]`.

### A.2 Acquire the per-month lock (per R15)

Compute `{YYYY-MM}` from the system date. Check for an existing lock file at `admin/scratch/insights-{YYYY-MM}.lock`:

- **No lock present** → create the lock atomically using a write-with-O_EXCL pattern. Lock body is a single line: ISO-8601 timestamp of acquisition. Proceed to A.3.
- **Lock present, age < 60 minutes** (compare lock body's timestamp to now) → refuse with the literal sentinel chat line:

  *"`/insights` for `{YYYY-MM}` is in flight (started `{timestamp}`). Wait for it to complete, then re-invoke if you have additions."*

  Where `{timestamp}` is read from the lock file body. Exit without writing. Do NOT remove the lock (the in-flight path owns it).
- **Lock present, age ≥ 60 minutes** → treat as stale (covers crashed-orchestration cleanup AND extended interactive accept-edit-reject sessions). Overwrite the lock with a fresh timestamp and proceed. Surface the chat-line: *"[stale lock detected for `insights-{YYYY-MM}` — proceeding]"*.

The 60-minute threshold is hard-coded for v1 (longer than `specflow:complete`'s 30-minute because `/insights` walks potentially many promotion proposals interactively). **v2 candidate (per PRD Open Questions):** surface as `config.json.insights.staleLockMinutes` if real consumers report stale-lock false-positives.

The lock is released atomically at the END of Phase G on every exit path (successful write, refused exit, schema-drift exit, threshold exit, concurrent-trigger exit). A path that completes without removing its own lock is a failed run.

### A.3 Read the corpus + corpus-size refusal (per R10, AC-10)

Read `docs/specflow/admin/task-history.json`. Resolve `corpus_size` as the count of entries whose `superseded_by_retro: true` AND no `superseded_by` (pruned/scope-change-superseded entries are excluded — they're not closed-task lessons).

If `corpus_size < minCorpusSize`, refuse with the literal sentinel chat line:

*"Corpus too small for meaningful pattern detection (`{N}` entries; threshold `{threshold}`, configurable via `config.json.insights.minCorpusSize`). Run more tasks through `specflow:complete` and re-invoke `/insights` when the corpus has grown."*

Where `{N}` is the actual entry count and `{threshold}` is the configured value. Release the lock; exit without writing the report or appending to the runs.jsonl. Manual-mode and cron-mode refuse identically.

### A.4 TOCTOU snapshot against concurrent `specflow:complete` writes (per Gate 2 da-r1-f1)

Capture `task-history.json`'s mtime AND a content-snapshot at this moment. Re-read mtime at Phase D before promotion synthesis; if mtime differs, refuse with: *"`task-history.json` was modified mid-run (concurrent `specflow:complete` write detected). Re-invoke `/insights` to read the updated corpus."* Release the lock; exit. The read-then-snapshot pattern handles the cross-skill race without holding a shared lock with `specflow:complete`.

### A.5 Verify before continuing

- `cadence` resolves to one of the four documented values; `minCorpusSize` is a positive integer.
- Lock acquired; lock body carries the acquisition timestamp.
- `corpus_size >= minCorpusSize`; corpus content snapshot held in working memory.

Tell the user: *"Pre-flight passed: corpus `{N}` entries, threshold `{threshold}`, cadence `{cadence}`, lock acquired at `{timestamp}`. Validating schema."*

Hand off to Phase B.

---

## Phase B — Schema-stability check on the corpus (per R14, AC-14)

### B.1 Walk every entry against the v1 contract field-set

For every entry in the corpus snapshot, validate the presence of the nine load-bearing fields the cluster passes read: `id`, `lane_assigned`, `ai_assistance_level`, `blast_radius_outcome`, `what_didnt_work`, `regressions_caught`, `escaped_issues`, `addenda`, `superseded_by_retro`. These are the subset of `specflow:complete`'s 25-field Schema Appendix that R1 + R2 + R4 actually consume. A missing field on any entry is a hard refusal.

### B.2 Refuse on schema drift

If any entry is missing any required field, refuse with the literal sentinel chat line:

*"Schema drift detected: `task-history.json` entry `{id}` missing required field `{name}`. The corpus must match the v1 contract documented in `003-complete-skill-prd.md`'s Schema Appendix before `/insights` can run. Re-fire `specflow:complete` for the affected entry."*

Release the lock; exit without writing the report or the runs.jsonl. Cross-skill dependency: this validation tracks `specflow:complete`'s Schema Appendix as the authoritative contract; if that schema evolves, R14 + this phase's required-field list MUST be revised in lockstep (per PRD R14 schema-dependency clause).

### B.3 Exclude pruned + superseded entries

From the validated corpus, drop entries with `superseded_by_retro: false` (development-time placeholders that never closed via a retro write), entries with `superseded_by: <T-id>` set (scope-change-superseded; the new entry carries the lesson), and entries with the `pruned: true` marker (when `/prune` is shipped separately). The remaining set is the **clusterable corpus** for Phase C.

### B.4 Verify before continuing

- Every entry in the validated corpus has the nine required fields.
- The clusterable corpus is the validated set minus the three exclusion classes.
- `clusterable_corpus_size` is captured for the runs.jsonl record.

Hand off to Phase C.

---

## Phase C — Two-pass deterministic clustering (per R1, R2, R3)

The two passes are deterministic — same corpus produces same clusters byte-for-byte. Same input → same output is the contract; two consecutive runs on the same corpus produce byte-identical report bodies (per AC-1). Reserved cluster-source label `semantic` for v2 embedding-clustering — Phase C never emits a cluster with `cluster_source: "semantic"`; the runs.jsonl `semantic_clusters` counter is always `0` for v1 (per R3, AC-3).

### C.1 Pass 1 — Field-shape exact-match grouping (per R1, AC-1)

For every entry in the clusterable corpus, compute the field-shape tuple:

```
(lane_assigned, ai_assistance_level, blast_radius_outcome)
```

Group entries by exact tuple match. For each group with `len(unique_contributing_ids) >= 3` (per R2 — task-id-level, deduplicated by `id` field; within-entry repetitions count as 1), emit a cluster object with `cluster_source: "field-shape"`, the tuple, `token_ngram: null`, the contributing ids, and one ≤140-char excerpt per contributing id (default source field `what_didnt_work`; fall back to `escaped_issues.descriptions[0]` then `regressions_caught.descriptions[0]`; an entry with all three empty contributes `{excerpt: "(no retro text)"}` — flagged for the user at promotion synthesis).

Tag-intersection (per R1's "plus tag intersection from the I3 `tags` field where present") is a SECONDARY pass within field-shape: when two or more entries in a tuple group share ≥1 tag, the tag-set intersection becomes a sub-cluster rendered under the tuple cluster in the report.

### C.2 Pass 2 — Token-frequency n-grams (per R1, AC-1)

For every entry in the clusterable corpus, build the **token corpus** by concatenating:

- `what_didnt_work` (string).
- Each `escaped_issues.descriptions[i]` (concatenated with single-space separators).
- Each `regressions_caught.descriptions[i]` (concatenated with single-space separators).

Apply the deterministic transformation:

1. **Unicode NFC normalisation** — converts visually-equivalent codepoints to a canonical form so "café" (composed) and "café" (decomposed) match.
2. **Case-fold to lowercase** — "BaseService" and "baseservice" collapse to `baseservice`. (Per interview Round 1 user edit — without case-folding the token-frequency pass will fragment real patterns into near-duplicates that miss the threshold.)
3. **Stop-word filter** — drop tokens in the fixed ~40-token English stop-word list (`the, and, was, for, but, are, not, you, your, with, from, that, this, have, has, had, will, would, could, should, can, did, does, just, into, over, then, than, when, what, why, how, where, who, which, were, been, being, here, there`) before n-gram extraction. Hard-coded for v1; v2 may surface as `admin/insights/stop-words.txt` if a non-English-corpus consumer surfaces (per PRD Open Questions).
4. **No stemming** — preserves the audit trail's lexical fidelity ("racing" and "raced" don't collapse — surfacing both would be a user-elected widening at edit-time).

Extract n-grams for `n ∈ {2, 3, 4}` (bigrams through quadgrams). Count distinct entries (by `id`) containing each n-gram at least once; deduplicate within-entry repetitions before counting (per R2). Emit a cluster object for every n-gram with `len(unique_contributing_ids) >= 3`: `cluster_source: "token-ngram"`, `field_shape_tuple: null`, the `token_ngram` string, contributing ids, one ≤140-char excerpt per contributing id (source field = the field whose token corpus contained the n-gram; preference order `what_didnt_work` > `escaped_issues.descriptions[0]` > `regressions_caught.descriptions[0]` when multiple fields contained it).

### C.3 Sub-cluster nesting (per R2, AC-2)

When a token-ngram cluster's `contributing_ids` is a subset of a field-shape cluster's `contributing_ids`, render the token cluster as a nested sub-cluster under the field-shape cluster in the report:

> *Lane=green ∧ AI=full ∧ blast=high (4 entries) → token cluster `service worker race` (3 of 4 entries)*

Both clusters still emit independently in the cluster object array (the runs.jsonl `field_shape_clusters` and `token_ngram_clusters` counters track them separately); the report's body folds the token sub-cluster under the field-shape parent for readability.

### C.4 Empty-cluster sentinel (per AC-1, Gate 2 goal-r1-f2)

If Pass 1 surfaces zero clusters meeting the threshold, hold `field_shape_clusters_empty: true` for Phase D. If Pass 2 surfaces zero clusters meeting the threshold, hold `token_ngram_clusters_empty: true`. Both sections are ALWAYS present in the report (even when empty); when empty, the section's body contains the literal sentinel:

*"No clusters above the 3-observation threshold this run."*

A report missing either section header is a failed render (per AC-1).

### C.5 Verify before continuing

- Every emitted cluster has `len(unique_contributing_ids) >= 3`.
- No cluster has `cluster_source: "semantic"` (per R3 + AC-3).
- Every cluster carries one excerpt per contributing id, each ≤140 characters.
- Within-entry repetitions did NOT inflate any cluster's count (a single task with three escaped_issues citing the same n-gram contributes 1, not 3).
- Two consecutive runs on the same corpus would produce byte-identical cluster lists (deterministic).

Hand off to Phase D.

---

## Phase D — Promotion proposal synthesis (per R4, R5, R8, R9)

### D.1 TOCTOU re-check (per Gate 2 da-r1-f1)

Re-read `task-history.json`'s mtime. If it differs from the snapshot captured at A.4, refuse with the concurrent-write sentinel from A.4. Release the lock; exit. (The window between A.4 and D.1 is the entire cluster pass — concurrent writes during that window invalidate the cluster basis.)

### D.2 Read the rules registry (per R9, AC-9)

Read `admin/rules/glossary.md` (the canonical surface listing every rule's `id` + `paths`). Read `admin/rules/guidelines.md` and `admin/rules/non-negotiable.md` (for tier-routing and substring-match-on-rule-id detection per R9). Hold the rule set in working memory.

### D.3 Tier-routing per cluster (per R9, AC-9)

For each cluster passing the threshold, decide the promotion tier:

- **`observation→guideline`** — when the cluster's auto-suggested `id` (SCREAMING_SNAKE_CASE of the dominant n-gram) AND the auto-suggested `paths:` glob (longest common prefix of contributing scopes) have NO matching entry in `glossary.md`. Target file: `admin/rules/guidelines.md`.
- **`guideline→non-negotiable`** — when an existing guideline matches (id substring OR paths glob overlap) AND the cluster's contributing entries include ≥3 distinct entries citing post-promotion violations of that guideline. The violation signal is substring-match on the existing rule's `id` in each entry's `escaped_issues.descriptions[i]` OR `what_didnt_work` (per R9 substring-match tradeoff: implementation-cheap; works against the existing schema; v1 false-positive risk mitigated by D.4's three-option human-confirmed prompt; v1 false-negative risk is the cost of substring matching's lexical-not-semantic limit). Target file: `admin/rules/non-negotiable.md`.
- **`null` (no promotion)** — when the cluster surfaces a pattern but no tier transition warrants. The cluster still appears in the report body for trend-signal but no proposal renders.

The tier is recorded explicitly on the cluster object as `proposes: "observation→guideline" | "guideline→non-negotiable" | null`.

### D.4 Compose the auto-suggested rule fields (per R4, R5, AC-4, AC-5)

For each cluster with `proposes != null`, compose the proposed rule:

- **`id`** — SCREAMING_SNAKE_CASE of the cluster's dominant n-gram (highest cross-entry frequency; for field-shape clusters derive from the union of contributing entries' `what_didnt_work`; for token-ngram clusters use the cluster's own `token_ngram`). Strip non-alphanumeric; collapse underscore runs. Example: `service worker race` → `SERVICE_WORKER_RACE`.
- **`tier`** — `"guideline"` for `observation→guideline`; `"non-negotiable"` for `guideline→non-negotiable`.
- **`paths`** — longest common prefix across contributing entries' `scope` arrays as a glob. De-duplicate scopes; intersect path prefixes (split on `/`); render as `["{prefix}/**/*.{ext}"]` where `{ext}` is the most common file extension (multi-glob `["{prefix}/**/*.{ts,tsx,js,jsx}"]` when extensions are mixed). Never aggressively widen (per AC-5 — the user widens at edit-time if appropriate).
- **`Rule:`** body — one short paragraph stating the rule. For `observation→guideline`: synthesise from the cluster's contributing excerpts (the rule is the inverse of the failure mode). For `guideline→non-negotiable`: lift the existing guideline's `Rule:` body verbatim (promotion is tier-strengthening, not rule-rewording).
- **`Why:`** body — verbatim ≤140-char excerpt per contributing id, each on its own line prefixed `- (id `{contributing_id}`):`. The verbatim-excerpt requirement is non-negotiable per AC-4 — a proposal whose `Why:` is missing any contributing id's excerpt is a failed render.
- **`On violation:`** body (non-negotiable tier ONLY) — *"CI gate fails on detection; PR cannot merge until the violation is resolved or an exception is filed via `specflow:misc --auto`."* (Conservative default; user widens at edit-time if project gate machinery differs.)
- **`Exceptions:`** body — *"None at promotion time. File exceptions via `specflow:misc --auto` with a one-line `why-deviated` rationale."*

### D.5 Verify before continuing (AC-4, AC-5, AC-9)

For each proposal:

- The proposed rule has frontmatter `id` matching `^[A-Z][A-Z0-9_]*$`, `tier` matching the destination file, `paths` as a non-empty array of glob strings.
- The `Why:` body contains one verbatim excerpt per contributing id; each excerpt ≤140 chars.
- The `paths:` glob is the longest common prefix (not aggressively widened).
- The tier transition string is one of the two documented forms (`observation→guideline` or `guideline→non-negotiable`).

A proposal failing any verify is dropped from the proposal list and recorded as a synthesis-failure in the runs.jsonl `synthesis_failures` field (does not block the run; surfaces as a chat-line warning).

Hand off to Phase E.

---

## Phase E — Report synthesis + decision-log secondary signal (per R11, R13)

### E.1 Read `decision-log.md` for cross-link signal (per R13, AC-13)

Read `admin/decision-log.md`. For each cluster, scan every decision-log entry's `**Related:**` field for substring matches against the cluster's contributing ids (accept both the literal `{NNN-slug}-T{N}` form AND prose mentions like `task T{N} of feature {NNN-slug}`). Capture each matching entry's `{date, title}` in the cluster's `related_decision_log_entries` list (may be empty).

The skill NEVER writes to `decision-log.md` (per R13, AC-13). A run that writes to `decision-log.md` (any append, edit, or new entry) is a failed run. Cross-task abstractions are *rules*, not decisions; per-task significant patterns elevate via `specflow:complete`'s R6 path; manual decision entries land via `specflow:decision`.

### E.2 Compose the active report (per R11, AC-1)

Compute the active report path: `admin/insights/{YYYY-MM}-report.md` where `{YYYY-MM}` is the current month from system date. Create `admin/insights/` if missing.

Compose the report body with this section structure (markdown):

- `# Insights — {YYYY-MM}` H1, then a metadata block (`**Run:**`, `**Corpus:**`, `**Clusters:**`, `**Promotions proposed:**`).
- `## Field-shape clusters` — one `### Lane={lane} ∧ AI={ai} ∧ blast={blast} ({N} entries)` subsection per field-shape cluster, each with `**Contributing entries:**` (comma-separated ids), `**Excerpts:**` (one bullet per contributing id, format `- (id `{id}`, field `{source_field}`): "{≤140-char excerpt}"`), `**Nested token sub-clusters:**` when applicable, `**Related decision-log entries:**` (list of `{date, title}` or `(none)`), `**Proposes:**` line, then the full frontmatter + body block from D.4 when `proposes != null`. When the section is empty, body contains the literal sentinel: *"No clusters above the 3-observation threshold this run."* — section header always present (per AC-1).
- `## Token-frequency clusters` — same shape per token-ngram cluster (heading reads `### Token cluster: `{token_ngram}` ({N} entries)`). Standalone clusters only — clusters nested under a field-shape parent at C.3 do NOT re-appear here. Empty-section sentinel as above.
- `## Proposed-rejected` — populated by Phase F option-3 (within-run append; cleared on next within-month re-run since the report is replace-in-place per R11).
- `## Calibration note` — one short paragraph stating: v1 ships deterministic two-pass clustering; embedding-based clustering reserved for v2 under the `semantic` label; every cluster ≥3 observations appears in this report; no auto-promotion fires.

### E.3 Write the report (replace-in-place per R11, AC-11)

Write the composed report to `admin/insights/{YYYY-MM}-report.md`. If the file already exists from a prior within-month run, OVERWRITE it (replace-in-place semantic — the report represents the *current* state of the month's pattern detection). User-accepted promotion writes to `admin/rules/*` from prior runs are independent and never reverted by this skill (per R12, AC-12).

### E.4 Cross-month rollover (per R12, AC-12)

When the system date's `{YYYY-MM}` differs from the most recent `*-report.md`'s `{YYYY-MM}` in `admin/insights/`, the skill writes a fresh `{YYYY-MM}-report.md` AND a fresh `{YYYY-MM}-runs.jsonl`. The prior month's files are NOT touched (no read, no write, no delete). User-accepted promotion writes to `admin/rules/*` from prior months remain in place; the skill never reverts a previously accepted promotion as part of cadence rollover.

### E.5 Verify before continuing (AC-1, AC-2, AC-13)

- `admin/insights/{YYYY-MM}-report.md` exists with both `## Field-shape clusters` and `## Token-frequency clusters` section headers.
- Empty sections carry the literal sentinel body line.
- Every cluster in the report lists its contributing ids verbatim AND its excerpts (one per contributing id, ≤140 chars).
- Every proposal renders the full frontmatter + body block per D.4.
- `decision-log.md` was read but NOT written (re-read mtime; confirm unchanged).
- For cross-month rollover: prior month's files untouched.

Hand off to Phase F.

---

## Phase F — Interactive promotion review (per R8, AC-8)

### F.1 Walk each proposal with the user

For each cluster whose `proposes != null`, surface a chat block titled *"Proposal {i}/{P}: {tier transition} — {auto-suggested id}"* containing: cluster basis (field-shape tuple OR token n-gram + entry count), contributing ids, full frontmatter + body block per D.4, then the three options:

- **`1` — accept-and-write** — append to `{target_file}`; add glossary entry.
- **`2` — edit-and-write** — user edits any of `id`, `paths`, `Rule:`, `Why:`; skill re-validates frontmatter shape and writes.
- **`3` — reject** — no write; cluster recorded in the report's `Proposed-rejected` section.

The skill MUST NOT proceed without an explicit user choice per proposal — empty input or any input not matching `1|2|3` re-prompts. Auto-default is a failed run (per AC-8: *"The skill exits with a failed run if it auto-defaults (no human choice captured for any proposal)"*).

### F.2 Option 1 — accept-and-write

Append the rule's frontmatter + body to the target file (`admin/rules/guidelines.md` for `observation→guideline`; `admin/rules/non-negotiable.md` for `guideline→non-negotiable`). The append matches the target file's existing format byte-for-byte (canonical frontmatter shape; canonical body sections; trailing `---` separator if the file uses one).

Add a one-line entry to `admin/rules/glossary.md` under the matching tier section header:

```markdown
- `{id}` ({tier}, paths: `{paths-glob}`) — {one-line summary of the Rule body}
```

After the rule-registry write, increment the per-run `promotions_accepted` counter. Surface the chat-line: *"Accepted `{id}` → `{target_file}` and `glossary.md`."*

### F.3 Option 2 — edit-and-write

Prompt the user for edits:

*"Edit any of: `id`, `paths`, `Rule:`, `Why:`. Type the field name + new value (one per line); type `done` when finished, or `cancel` to drop."*

Validate each edit:

- `id` — must match `^[A-Z][A-Z0-9_]*$`. Reject and re-prompt on failure.
- `paths` — must be a non-empty array of glob strings (parse as JSON or comma-separated list). Reject and re-prompt on failure.
- `Rule:` — non-empty string. Reject empty.
- `Why:` — non-empty string. Reject empty.

`tier` is NOT user-editable at edit-time (the tier is the routing decision from D.3; if the user wants a different tier, they reject and re-invoke). `On violation:` and `Exceptions:` carry the conservative defaults from D.4; user can edit at the file level after the write.

On `done`: re-run the D.5 verify (regex + non-empty checks); on pass, write per F.2 with the edited fields. On `cancel`: route to F.4 (reject). Increment `promotions_accepted` on success.

### F.4 Option 3 — reject

No write to `admin/rules/*`. Record the rejection in the report's `## Proposed-rejected` section:

```markdown
- {rejection_timestamp} — `{auto_suggested_id}` (tier `{tier}`; contributing_ids `{list}`) — rejected.
```

Re-write the report file with the appended rejection line (replace-in-place per R11; this is a within-run mutation of the report file the skill itself just wrote). Increment the per-run `promotions_rejected` counter. Surface the chat-line: *"Rejected `{auto_suggested_id}` — recorded in `Proposed-rejected` section."*

The proposed-rejected section carries the trend-signal for the NEXT month's run — if the same cluster surfaces and is rejected again, the registry-watcher pattern is the user-facing pattern signal that the project doesn't want this rule.

### F.5 Verify before continuing (AC-8)

- Every proposal received an explicit user choice from `{1, 2, 3}`.
- Option-1 writes appended to the target rule file + glossary; option-2 writes applied the validated edits per F.2; option-3 appended a `Proposed-rejected` line to the report.
- `promotions_accepted + promotions_rejected == promotions_proposed` (binary user-choice contract holds).

Hand off to Phase G.

---

## Phase G — Run-record append + lock release + chat-line summary (per R7, R11, R12, AC-7, AC-11, AC-16)

### G.1 Compute next-suggested-run date (per R7, AC-7)

Read the most recent record from any `admin/insights/*-runs.jsonl` (the `last_successful_run_at` is its `completed_at`). On first-ever run, fall back to the current run's `started_at`. Compute `next_suggested_run_at` per the cadence interval: weekly = +7 days; monthly = +30 days; quarterly = +90 days; `manual-only` = `null`.

The `next_suggested_run_at` is informational only — the skill never blocks subsequent invocations on "too soon" or "too late" relative to the suggested date. Per R7's load-bearing-assumption note: the cadence knob is purely informational inside this skill; it changes only this chat-line and the runs.jsonl record. Cluster detection and refusal logic are cadence-independent.

### G.2 Append to runs.jsonl (per R11, AC-11)

Append one JSON object as a single line to `admin/insights/{YYYY-MM}-runs.jsonl` matching the PRD Schema Appendix v1 contract: `run_id` (ULID or timestamp-shaped), `started_at` + `completed_at` (ISO-8601 UTC), `trigger` (`manual | cron`), `corpus_size`, `minCorpusSize_used`, `cadence_used`, `clusters_surfaced`, `field_shape_clusters`, `token_ngram_clusters`, `semantic_clusters` (always `0` in v1 per R3 — the field is the additivity surface for v2 embedding clustering), `promotions_proposed`, `promotions_accepted`, `promotions_rejected`, `next_suggested_run_at` (ISO-8601 UTC OR `null` when `cadence=manual-only`), `report_path`, `rules_files_written` (every rule file mutated in Phase F, deduplicated; empty array if all proposals were rejected).

The runs.jsonl file is APPEND-ONLY — never overwritten or edited (per R11, AC-11). A run that overwrites or edits any line is a failed run; a run that fails to append a record on successful completion is a failed run. The diff of `runs.jsonl` before vs after a successful run shows exactly one line added.

### G.3 Release the lock atomically

Remove `admin/scratch/insights-{YYYY-MM}.lock`. The remove fires on every exit path the orchestrator reaches (successful write, threshold refusal, schema-drift refusal, concurrent-trigger refusal, config-value refusal, auto-default detection). A path that reaches Phase G without removing its own lock is a failed run.

### G.4 Emit the chat-line summary (per AC-7, AC-16)

Emit two informational chat lines on every successful run:

*"`/insights` `{YYYY-MM}` run complete. Corpus `{N}` entries → `{C}` clusters → `{P}` promotions proposed (`{A}` accepted, `{R}` rejected). Report: `admin/insights/{YYYY-MM}-report.md`."*

*"Next suggested `/insights` run: `{date}` (cadence `{cadence}`)."* OR (when `cadence: "manual-only"`) *"Next suggested `/insights` run: no scheduled cadence — invoke manually."*

The five resolved tokens (N corpus size, C clusters surfaced, P proposals, A accepted count, R rejected count) carry the load-bearing signals for trend-analysis in subsequent runs. (Per Gate 2 goal-r1-f1 the deferred-state token was dropped — R8's three-option prompt forbids deferral; the orphan-AC counter was removed.)

Failed runs (refusal exits) emit one of the five sentinel failure lines from R10 / R14 / R15 / R6-config-invalid / R8-auto-default; no silent exits (per AC-16).

### G.5 Optional Linear ticket per rejected proposal (best-effort)

If `admin/environment.json` indicates `mcp.linear.available: true` AND any proposals were rejected: prompt *"Open a Linear ticket per rejected proposal? Each is confirmed individually (y/n per proposal)."* For each `y`, fire `specflow:linear` to create a ticket capturing the rejection rationale. For each `n`, skip. Best-effort; failures log a chat-line and continue. (This optional surface is informational — it does NOT block completion.)

If MCP unavailable, skip with the chat-line `[insights: Linear MCP not detected — Phase G optional ticketing skipped]`.

### G.6 Verify before declaring done (AC-1 through AC-16)

1. Active report at `admin/insights/{YYYY-MM}-report.md` carries both section headers; clusters list contributing ids verbatim; full frontmatter + body block per proposal; deterministic re-run produces byte-identical body (AC-1, AC-2, AC-4, AC-5).
2. No cluster has `cluster_source: "semantic"` (AC-3); tier-routing is one of the two documented transitions (AC-9).
3. Every promotion proposal received an explicit user choice from `{1, 2, 3}` (AC-8).
4. Runs.jsonl has exactly one line appended; report file replaced in full on within-month re-run (AC-11); cross-month rollover left prior-month files untouched (AC-12).
5. Decision-log read but never written (AC-13); `decision-log.md` mtime unchanged across the run.
6. Closing chat-line summary emitted with the five load-bearing tokens (AC-16); next-suggested-run date emitted (AC-7).
7. Lock file at `admin/scratch/insights-{YYYY-MM}.lock` no longer exists.
8. On refused exits: one of the documented sentinel lines (R10 / R14 / R15 / R6-config-invalid / R8-auto-default) was emitted instead of a successful summary (AC-10, AC-14, AC-15, AC-16).

If any verify step fails, surface the failure and refuse to claim success.

---

## Failure modes

Each maps to a documented sentinel refusal exit; never silent retry.

- **Corpus too small (A.3 / R10 / AC-10)** — threshold sentinel naming count + threshold + knob name. Release lock; exit. Manual + cron refuse identically.
- **Schema drift (B.2 / R14 / AC-14)** — schema-drift sentinel naming the offending entry id + missing field. Cross-skill dependency: `specflow:complete`'s Schema Appendix is the authoritative contract.
- **Concurrent trigger (A.2 / R15 / AC-15)** — in-flight sentinel naming lock acquisition timestamp. First-to-start owns the lock; second-to-start exits without writing.
- **Concurrent `specflow:complete` write mid-run (A.4 + D.1 / Gate 2 da-r1-f1)** — mtime-mismatch sentinel; re-invoke to read updated corpus.
- **Invalid `cadence` config value (A.1 / R6 / AC-6)** — config-invalid sentinel naming the documented enum.
- **User auto-default on promotion prompt (F.1 / R8 / AC-8)** — *"`/insights` requires an explicit choice (`1|2|3`) for every promotion proposal. Auto-default is a failed run."* The proposal is NOT recorded as rejected (no half-state writes).
- **Empty cluster set on a passing corpus (C.4 / Gate 2 goal-r1-f2)** — report renders with zero proposals; empty-section sentinel surfaces; runs.jsonl carries `clusters_surfaced: 0`. Chat-line: *"Registry stable; no promotions proposed this cycle (corpus `{N}` entries; threshold `{threshold}`)."* — positive signal, not failure.
- **All proposals rejected (F.4)** — runs.jsonl records full-rejection cycle (`promotions_accepted: 0`); registry unchanged. Chat-line: *"All `{P}` proposals rejected — registry unchanged. Trend-signal recorded for next month's run."*
- **Synthesis failure on a single proposal (D.5)** — drop the failing proposal; record in runs.jsonl `synthesis_failures` field; chat-line warning naming the cluster id. Run continues; remaining proposals walk normally.

A refused exit without a documented sentinel line is a failed run.

---

## Anti-patterns (refuse to do)

- **Auto-promote without user confirmation.** Phase F's three-option prompt is the ONLY path to a rule-registry write; no silent escalation, no auto-default, no `--yes` flag (R8 + AC-8).
- **Mutate rule files outside Phase F's option-1 / option-2 paths.** A run that writes to `admin/rules/*` outside the explicit-confirm path is a failed run.
- **Cluster on free-text only.** The field-shape pass (Pass 1) is the deterministic anchor; running Pass 2 alone collapses the field-shape signal hierarchy. Both passes ship in v1.
- **Use embedding-based clustering in v1.** Deferred to v2 via the reserved `semantic` cluster-source label (R3 + AC-3 + Gate 2 simplicity-r1-f1 push-back). The runs.jsonl `semantic_clusters` counter is always `0` in v1.
- **Write to `decision-log.md`.** Read-only signal per R13 + AC-13. Cross-task abstractions are *rules*, not decisions.
- **Stem the token corpus or extend the stop-word list silently.** No stemming preserves lexical fidelity; the stop-word list is the fixed ~40-token v1 constant.
- **Aggressively widen the auto-suggested `paths:` glob.** Longest common prefix is the conservative anchor (R5 + AC-5). User widens at edit-time if appropriate.
- **Auto-install a scheduler.** v1 ships only the skill body (R6 + AC-6). Cadence knob is informational; users wire their own scheduler.
- **Bypass the lock.** No path may write to `admin/insights/*` for `{YYYY-MM}` without holding the corresponding lock. Stale-lock proceed (A.2 case 3) overwrites and proceeds; never bypasses.
- **Edit existing rules.** Promotion is additive; demotion / removal is `/prune`'s contract. A run that mutates an existing rule's body or frontmatter is a failed run.
- **Aggregate across repos / projects.** v1 reads only the local repo's `task-history.json` (Goal Out-of-scope item 2).
- **Name any AI vendor or tooling in user-facing output.** The report, proposals, runs.jsonl record, rule writes, and every chat line stay vendor-neutral per the project's CLAUDE.md attribution rule.

---

## Cross-skill integration

- **`specflow:complete`** — primary producer of `task-history.json`. Phase A.4 + D.1 read-then-snapshot mtime-check handles the cross-skill TOCTOU race (Gate 2 da-r1-f1) without a shared lock. Schema dependency per R14 + AC-14: `specflow:complete`'s Schema Appendix is the authoritative v1 contract; if it evolves, R14's required-field list MUST be revised in lockstep.
- **`specflow:decision`** — read-only consumer of `decision-log.md` per R13 + AC-13. Cross-link signal at E.1 surfaces decision-log entries whose `Related` field cites a contributing id. (Note: `specflow:scope-change` writes `SC-{NNN}` decision-log entries via `specflow:decision`; `/insights` reads those the same way it reads any other entry.)
- **`/prune`** — quarterly counterpart. Pruned + scope-change-superseded entries are excluded at B.3 (`pruned: true` and `superseded_by` markers respected). v1 of `/insights` does NOT invoke `/prune` (Goal Out-of-scope item 4); the two consume the same corpus on independent cadences.
- **`specflow:misc`** — out-of-scope rule violations surfaced during Phase F edit-and-write may be routed by the user to `specflow:misc --auto`; the skill itself does NOT auto-fire `specflow:misc`.
- **`specflow:linear`** — best-effort optional ticket creation per rejected proposal at G.5; user-confirmed per proposal.
- **`specflow:scope-change`** — `superseded_by` marker on prior entries is the exclusion signal at B.3 (the new entry carries the lesson; counting both inflates contributing-ids).

---

## Reference

- `docs/specflow/features/006-insights-skill/006-insights-skill-prd.md` — R1-R15 + AC-1 to AC-16; Schema Appendix (runs.jsonl + cluster object v1 contract).
- `docs/specflow/features/006-insights-skill/006-insights-skill-interview.md` — seven rounds; user-confirmed resolutions.
- `docs/specflow/features/006-insights-skill/debate-log/prd-gate2/manifest.md` — Gate 2 (passed-with-revisions); load-bearing constraints: R3 `semantic` reservation (simplicity-r1-f1 push-back held); R7 cadence-knob informational-status (tbc-r1-f1 partial); R9 substring-match tradeoff (tbc-r1-f2); R14 + AC-14 cross-skill schema dependency (surgical-r1-f1); R15 + AC-15 concurrent-trigger guard (da-r1-f1); AC-16 deferred-counter dropped (goal-r1-f1); AC-1 empty-cluster sentinel (goal-r1-f2).
- `templates/orchestrator-pattern.md` — seven sequential phases; no sub-agent forking.
- `CORE_PRINCIPLES.md` — the four principles bound to every phase verify step.
- `skills/complete/SKILL.md` — producer of `task-history.json`; Schema Appendix v1 contract.
- `skills/decision/SKILL.md` — sister Phase 3 skill; canonical `decision-log.md` writer (read-only here).
- `skills/scope-change/SKILL.md` — orchestrator-pattern calibration anchor; writes `SC-{NNN}` entries this skill reads at E.1.
- `examples/docs/specflow/admin/{task-history.json, rules/*.md}` — worked-example corpus + rule registry.
