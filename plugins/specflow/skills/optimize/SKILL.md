---
name: optimize
description: Autoresearch loop generalising simplify's discipline across the verifiable-skill set. For a target skill with a binary eval, generates sequential prompt variants (default 5, max 10) using six structured mutation operators, evaluates each variant via the target's machine-eval surface, opens a single-skill PR, requires human merge. Branch-per-run, no LLM-as-judge inside the loop, per-target weekly budget cap, declines after a streak of failed runs.
status: v2-new
phase: 3
requires:
  - target skill's SKILL.md (the skill being optimised, e.g. plugins/specflow/skills/simplify/SKILL.md)
  - target skill's eval surface (machine-runnable, binary-output, read-only)
  - docs/specflow/admin/optimize-runs.jsonl (run history, append-only)
  - docs/specflow/admin/budget/per-skill-tokens.json
  - docs/specflow/admin/config.json
produces:
  - branch optimize/{target}-{run-ts} with proposed SKILL.md prompt variant
  - PR on the branch with eval results table + variants table + decline-streak summary
  - docs/specflow/admin/optimize-runs.jsonl entry with merge_decision tracked through human review
  - docs/specflow/admin/scratch/optimize-{target}-{run-ts}/ scratch directory (variants, eval logs, scoring)
  - .github/workflows/optimize-merge-gate.yml (idempotent first-run install)
eval: |
  PR exists with passing CI; weekly per-target spend under cap; human merge decision recorded in
  optimize-runs.jsonl; no auto-merge ever; declined target after streak honoured (no further runs
  on that target without --override-decline {reason}).
---

# optimize

Bounded autoresearch loop on a single target skill's prompt body. The eval surface is **the target's own machine-checkable `eval:` field** — never an LLM-as-judge, never a peer-skill judge, never the orchestrator's taste. **The metric is a signal, not a goal.** That is the load-bearing Goodharting protection inherited verbatim from `simplify`, and the structural reason this skill is trustworthy at the catalog level instead of the one-skill level.

This skill is the **generalisation of `simplify`** — not a replacement. `simplify` continues to ship as the Phase 1 discipline-installer focused on one target (code complexity in a scope). `/optimize` extends the same shape across the verifiable-skill set: branch-per-run (R8), sequential variants (R5), no LLM-as-judge inside the loop (R6 + structural enforcement throughout), human merge owns taste (R12 three-guardrail lockout). Phase 3 self-learning consumers (`/insights`, `/prune`) mine the resulting `optimize-runs.jsonl` corpus for which targets converge, which keep failing variants, which the human always rejects (the eval-is-missing-what-the-human-values signal R13 surfaces).

The four core principles bind here as everywhere: think before coding (eligibility check stops broken targets pre-flight; assumptions about `eval_reads ⊂ produces:` named in the artefact, not silent), simplicity first (six fixed mutation operators, no free-form rewrite; one operator per variant), surgical changes (the branch contains one commit modifying one target's SKILL.md plus the corpus append; pre-PR file-diff check refuses cross-skill smuggling), goal-driven execution (every phase has a binary verify step inline; the pre-flight gate chain refuses on broken baseline rather than papering over it).

---

## Inputs

The skill is invoked via one of:

- `/optimize {target}` — manually run the loop on a named target (e.g. `/optimize simplify`, `/optimize feedback-loop-audit`).
- `/optimize {target} --variants {N}` — clamp variant count to `[1, 10]` (default 5).
- `/optimize {target} --override-budget {reason}` — extends only that target's cap for the current run; reason recorded.
- `/optimize {target} --override-decline {reason}` — proceeds despite a 3-decline streak; reason recorded.
- `/optimize {target} --dry-run` — generate variants and run evals but do NOT push the branch or open the PR.
- Scheduled invocation — GitHub Actions overnight cron picks an under-cap target and fires the same loop. The cron path uses GH's `concurrency: { group: optimize-cron, cancel-in-progress: false }`.

Tell the user explicitly which mode you detected: *"Optimize run on `{target}` — mode: {manual | cron | dry-run}. Pre-flight gate chain starting."*

---

## Phase A — Pre-flight + target eligibility

### A.1 Verify target skill exists (R1, R2.c)

Resolve `plugins/specflow/skills/{target}/SKILL.md`. If missing, refuse: *"Target `{target}` has no SKILL.md at `plugins/specflow/skills/{target}/SKILL.md`. Refusing — `/optimize` requires a real target skill."*

### A.2 Mechanical eligibility check on `eval:` field (R1)

Read the target's frontmatter `eval:` field. Apply the structured check:

- **Clause (a)** — the field exists and is non-empty. Refusal: *"Target `{target}` has no `eval:` field. Refusing — `/optimize` requires an eval surface."*
- **Clause (b)** — every clause in the field can be evaluated to true/false from produced files (path exists, JSON field present, count ≤ N, exit-code 0). Clauses requiring AI judgement fail.
- **Clause (c)** — no clause uses a judgement word from the configurable list. Default list (hard-coded): `appropriately`, `adequately`, `cleanly`, `concrete signals`, `coverage`, `idiomatic`, `well`, `properly`, `correctly`. Project list extends via `config.json.optimize.judgementWords` (the project list extends the default; it does not replace it). Refusal: *"Target `{target}` eval clause `{N}` ('{clause text}') uses judgement word `{word}`. Refusing — `/optimize` requires a fully binary eval surface."*

### A.3 Structured score-direction declaration (R17)

Verify the target's `eval:` field includes a structured score block of the form `score: { signal: "{description}", direction: "maximize|minimize" }` OR has a self-evident maximize-direction shape (lines deleted, count of passing tests). Ambiguous score directions refuse with: *"Target `{target}` has no structured `score:` block in its `eval:` field and the score-direction is not self-evident. Refusing — `/optimize` requires a declared score direction. Add `score: { signal: \"{description}\", direction: \"maximize|minimize\" }` to the target's `eval:` field."*

### A.4 `eval_reads ⊂ produces:` strict contract (R10)

Parse the target's `eval:` field for every file reference. Verify each appears in the target's `produces:` field. Targets whose `eval:` reads files outside `produces:` (e.g. project-level lint output, working-tree state) refuse with: *"Target `{target}` eval reads `{file}` which is not enumerated in `produces:`. Refusing — `/optimize` requires `eval_reads ⊂ produces:` for redirectable per-variant evaluation."*

Targets whose `produces:` is empty refuse with: *"Target `{target}` has no enumerable `produces:` field. Refusing — `/optimize` requires a redirectable eval surface."*

### A.5 Baseline eval health check (R2.e)

Run the target skill at its current SKILL.md (no variants applied) inside a forked sub-agent, with output redirected to `admin/scratch/optimize-{target}-{run-ts}/baseline-run/`. Run the target's `eval:` field clauses against the baseline output. If baseline eval is not green, refuse: *"Target `{target}` baseline eval is not green: `{which clauses failed}`. `/optimize` cannot improve a broken baseline. Fix the baseline before running `/optimize`."*

This matches `simplify` Pre.A.3 verbatim — transposed from the simplification surface to the prompt-eval surface. Without it, the loop Goodharts against an undefined target.

### A.6 Verify before continuing

- Target SKILL.md exists.
- Eligibility clauses (a), (b), (c) pass.
- Score-direction declared OR self-evident.
- `eval_reads ⊂ produces:` contract holds.
- Baseline eval green.

Hand off to Phase B.

---

## Phase B — Concurrency + budget + decline-streak

### B.1 Lockfile gate (R2.a, R14)

Check `admin/scratch/optimize.lock`:
- **No lock present** → create atomically with start timestamp + orchestration ID. Proceed.
- **Lock present, age < 4 hours** → refuse: *"Another `/optimize` run is in progress (id: `{id}`, started: `{ts}`). Wait for it to complete or remove the lockfile if you've confirmed the prior run is dead."* Exit without writing.
- **Lock present, age ≥ 4 hours** → treat as stale (assume the prior run died). Overwrite atomically with fresh timestamp. Surface chat-line: *"[stale optimize.lock detected — proceeding]"*.

The 4-hour threshold is calibrated for typical run duration (10-variant target lands in 30-60 minutes; 4 hours covers any reasonable hang). The lock is released at the END of Phase I on every exit path (success, refused, partial). A path that completes without removing its own lock is a failed run.

### B.2 Single-PR-per-target gate (R2.b)

Run `gh pr list --head 'optimize/{target}-*' --state open`. If ≥1 open PR, refuse: *"Target `{target}` already has an open `/optimize` PR (`#{N}`). Resolve that PR (merge or close) before opening another."* This prevents the "user merged variant 1, then a second variant 2 PR shipped before the merge propagated" race.

### B.3 Budget gate (R2.d, R7)

Read `admin/budget/per-skill-tokens.json`. Filter to the target's invocations within the rolling 7-day window. Compare against `config.json.optimize.targetCapUsd` (default $10/target/week).

- If `weekly_spend < targetCapUsd` → proceed.
- If `weekly_spend >= targetCapUsd` AND `--override-budget {reason}` was NOT passed → refuse: *"Target `{target}` is at ${X.XX} of the ${cap} weekly cap. Refusing to run. Override via `--override-budget {reason}`."* Exit.
- If `--override-budget {reason}` passed → proceed; record `override_budget_reason: "{reason}"` in the run's record at Phase H.

Aggregate envelope is implicit ($10 × 6 targets = $60/week). No aggregate cap is enforced at runtime; per-target caps are the only hard cap.

### B.4 Decline-streak gate (R13)

Read `admin/optimize-runs.jsonl`. Walk the most recent records for `{target}`, filtered to `merge_decision IN (closed-without-merge, merged)` only. Refused runs (`merge_decision: null-failed-run`) are skipped — they neither reset the streak nor count toward it.

- Compute the consecutive-decline count: walk newest-first; count records with `merge_decision: closed-without-merge` until a `merged` is encountered (which resets the count to 0) or the records run out.
- Compute the most-recent-decline `ended_at` timestamp.

Apply the streak rules:

- **Operator-avoid window (7 days):** if the most recent decline's `ended_at` is < 7 days old, the operator that produced that variant is excluded from this run's operator pool. Surface chat-line: *"[operator `{op}` declined within 7 days; excluded from this run's pool]"*.
- **30-day-skip window:** if 3 consecutive declines on `{target}` AND the third decline's `ended_at` is < 30 days old:
  - **Cron mode:** skip this target. Log a no-op record. Cron picks another eligible target on the next pass.
  - **Manual mode without `--override-decline {reason}`:** refuse: *"Target `{target}` has had 3 consecutive PRs closed without merge. The eval is producing improvements the human keeps rejecting — likely the eval is missing what the human values. Inspect `optimize-runs.jsonl` for the rejected variants and consider tightening the eval before the next run. Override via `--override-decline {reason}`."*
  - **Manual mode with `--override-decline {reason}`:** proceed; surface the warning chat-line; record `override_decline_reason: "{reason}"` in Phase H.

A subsequent `merged` decision clears the skip — the cron resumes targeting the skill on its next nightly run. A subsequent `closed-without-merge` does NOT extend the skip — the prior streak's skip continues to expire on its original schedule (computed from the third decline's `ended_at`, not the new one's).

### B.5 Verify before continuing

- Lock acquired (fresh OR stale-overwrite).
- No open PR for `{target}`.
- Budget under cap OR override passed.
- Decline streak resolved (no skip OR override passed).

Hand off to Phase C.

---

## Phase C — Variant generation (sequential)

### C.1 Cut the branch + capture baseline SHA (R8)

```bash
ts=$(date +%Y%m%d-%H%M%S)
slug=$(echo "{target}" | sed 's/[^a-zA-Z0-9]/-/g' | tr -s - | sed 's/-$//')
branch="optimize/${slug}-${ts}"
git checkout -b "${branch}"
mkdir -p admin/scratch/optimize-${slug}-${ts}/{variants,run-output,evals}
git rev-parse HEAD:plugins/specflow/skills/{target}/SKILL.md > admin/scratch/optimize-${slug}-${ts}/baseline-skill-sha.txt
```

The baseline SHA is verified again before opening the PR (Phase F) — if the target SKILL.md was modified mid-run, the run is discarded with a structured message and the scratch is retained.

### C.2 Determine variant count + operator pool

- Variant count: `--variants {N}` flag (clamped to `[1, 10]`); default 5.
- `--variants 12` → clamp to 10 with chat-line warning: *"Variant count clamped to 10 (max). Sequential variants per `simplify` discipline."*
- `--variants 0` → refuse: *"Variant count must be ≥1."*

Operator pool: the six structured mutation operators from R3, minus any operator excluded by B.4's 7-day operator-avoid window. Each variant picks exactly one operator. For targets whose SKILL.md body has no `^## Phase [A-Z]` heading, `split-by-phase` is skipped during operator selection (per AC-3 heuristic).

### C.3 Generate variants sequentially (R3, R5, R6 frontmatter-immutability)

For each of `1..N` variants:

1. Fork a variant-generator sub-agent. Pass it (via command substitution):
   - The target SKILL.md body (everything below the `---` delimiter).
   - The target's `eval:` field as the binary contract the variant must still satisfy.
   - The operator instruction (one of the six; see Operator Set below).
   - The hard rule: the variant MUST NOT change the frontmatter (`requires:` / `produces:` / `eval:` / `name` / `description` / `phase` are immutable).
2. The sub-agent writes the proposed full SKILL.md to `admin/scratch/optimize-{slug}-{ts}/variants/v{N}-{operator}.md` and the diff to `variants/v{N}-{operator}.patch`.
3. Variants run **sequentially** (not in parallel) — exactly matching `simplify`'s primitive. Parallel attempts would explode the budget and create merge headaches.

**No cross-variant context bleed.** Each variant is generated in a fresh forked context whose only inputs are the target body, the eval contract, and the operator instruction. Variants do not see each other.

### C.4 Operator Set — the six structured mutations (R3)

Each operator has a documented signature and an expected line-delta direction:

- **`tighten`** — input: prose with qualifiers / hedge phrases / defensive error handling for impossible scenarios. Output: same instructions with hedges removed. Expected line delta: ≤ 0 (lines decrease or hold).
- **`consolidate`** — input: two adjacent instructions covering overlapping ground. Output: one instruction folding both. Expected line delta: ≤ 0.
- **`clarify`** — input: vague phrasing ("appropriately handle", "ensure correctness"). Output: binary-testable language ("exit code 0", "JSON has field `x`"). Expected line delta: ~ 0 (length hold).
- **`deduplicate`** — input: instruction repeated across sections. Output: kept once, removed elsewhere. Expected line delta: ≤ 0.
- **`reorder`** — input: load-bearing instruction buried mid-prompt. Output: same instruction moved earlier. Expected line delta: 0 (rearrangement, no semantic change).
- **`split-by-phase`** — input: a monolithic phase covering multiple R-IDs. Output: the same phase rewritten as named sub-phases (one R-ID per sub-phase). Only fires on targets whose body has `^## Phase [A-Z]` headings. Expected line delta: ≥ 0 (structure adds light scaffolding).

Variants invoke exactly one operator per variant. Each variant's `eval-{N}.json` records the operator. Variants whose body is byte-identical to the baseline (no actual mutation occurred) record `passes_all_gates: false, reason: "no-op variant"`.

### C.5 Verify before continuing

- N variant files exist at `variants/v{N}-{operator}.md`.
- Each variant declares exactly one operator from the six-operator set.
- No variant's frontmatter differs from the baseline (mechanical diff per R4 — defence-in-depth before Phase D's per-variant check fires again).

Hand off to Phase D.

---

## Phase D — Per-variant evaluation

For each variant (sequentially, not in parallel):

### D.1 Frontmatter immutability re-check (R4)

Mechanically diff the variant's SKILL.md frontmatter against the baseline's frontmatter (everything between the first `---` and the second `---`). If the frontmatter differs in any way, mark the variant `passes_all_gates: false, reason: "frontmatter mutated: {specific diff}"`. The variant is not eligible to win the loop. The loop continues to the next variant — does NOT abort the run.

### D.2 Variant compile validation (R15)

Before running the eval, validate the variant's SKILL.md mechanically:

- (a) YAML frontmatter parses (failure → `passes_all_gates: false, reason: "frontmatter parse error: {detail}"`).
- (b) Body has at least one heading (failure → `passes_all_gates: false, reason: "body has no headings"`).
- (c) Frontmatter unchanged per D.1 (already checked; defence-in-depth).
- (d) Markdown lints pass when `admin/environment.json.lint.markdown` is populated. If unpopulated, this clause is N/A and the variant proceeds.

Validation failures mark the variant fail; the loop continues. Does NOT abort the run.

### D.3 Per-variant scratch redirection (R10)

Create `admin/scratch/optimize-{slug}-{ts}/run-output/v{N}/`. Apply the variant's SKILL.md to a forked sub-agent's working tree (the project's actual working tree is NOT mutated). Invoke the target skill inside the fork with output redirected to the per-variant scratch directory. The eval check reads files from the per-variant scratch directory without modifying the project's actual state.

### D.4 Run target's eval — binary, machine-only (R6, anti-LLM-as-judge)

Run the target's `eval:` field clauses against the per-variant scratch output. The eval is **the target's own machine-eval surface, nothing else**. No LLM-as-judge. No "is this prompt cleaner?" question. No taste-driven scoring. No peer-skill review.

Capture the per-variant eval result to `admin/scratch/optimize-{slug}-{ts}/evals/eval-{N}.json`:

```json
{
  "variant": N,
  "operator": "tighten | consolidate | clarify | deduplicate | reorder | split-by-phase",
  "eval_clauses_passed": ["clause-1", "clause-2"],
  "eval_clauses_failed": [],
  "score": X,
  "score_direction": "maximize | minimize",
  "lines_delta": M,
  "passes_all_gates": true,
  "approach": "{one-line description of what the operator did to the prompt body}"
}
```

The `score` field is target-specific (lines deleted for `simplify`, CONTEXT.md line count for `feedback-loop-audit`, etc.) and read from the target's `eval:` `score:` block (or self-evident shape per A.3).

### D.5 Verify before continuing to next variant

- `eval-{N}.json` exists with all fields populated.
- The project's working tree state outside `admin/scratch/optimize-{slug}-{ts}/` is unchanged.
- Target's eval ran in the forked sub-agent only.

When all variants have been evaluated, hand off to Phase E.

---

## Phase E — Scoring + ranking

### E.1 Filter to passing variants (R11)

From the variants where `passes_all_gates: true`, build the candidate pool. If no variants pass all gates, the run fails — surface explicitly to the user, skip Phase F (no PR opened), and record `winner_variant: null, all_gates_passed: false, reason: "all variants failed validation"` in Phase H.

### E.2 Apply score-direction (R17, R11)

For each candidate, score is read from `eval-{N}.json.score`. Apply the direction declared per A.3:

- `direction: maximize` → highest score wins (matches `simplify`'s lines-deleted shape).
- `direction: minimize` → lowest score wins (matches `feedback-loop-audit`'s CONTEXT.md-line-count shape).

### E.3 Tie-break

If two variants tie on score, pick the one with fewer side-effects (smaller `variants/v{N}-{operator}.patch` byte size). If still tied, pick the lower variant number (the one generated first). The same tie-break applies for both directions.

### E.4 Record runners-up

Record the top-3 variants (winner + two runners-up) in `admin/scratch/optimize-{slug}-{ts}/scoring.json` for the PR description's variants table. Every variant's eval is recorded regardless — that's the corpus Phase 3 mines.

### E.5 Verify

- Winner variant identified OR `winner_variant: null` recorded for the failed run.
- Tie-break applied deterministically.
- Scoring rationale captured for the PR description.

Hand off to Phase F.

---

## Phase F — PR open

### F.1 Mid-run target SKILL.md edit guard (R8)

Re-read the target SKILL.md and compute its current SHA. Compare to `baseline-skill-sha.txt` captured at C.1. If the SHA mismatches, refuse PR open: *"Target `{target}` SKILL.md was modified mid-run (baseline SHA `{X}`, current SHA `{Y}`). Discarding this run; rerun `/optimize {target}` to base the next loop on the current SKILL.md."* Retain the scratch directory for inspection. Record the run as `merge_decision: null-failed-run, refuse_reason: "baseline-sha-mismatch"` in Phase H.

### F.2 Apply the winning variant's SKILL.md diff (R10)

Only the winning variant's SKILL.md diff is applied to the working tree. Nothing the variant produced as side-effect (e.g. an updated `CONTEXT.md` from a `feedback-loop-audit` run) is preserved.

```bash
cp admin/scratch/optimize-{slug}-{ts}/variants/v{winner}-{operator}.md plugins/specflow/skills/{target}/SKILL.md
git add plugins/specflow/skills/{target}/SKILL.md admin/optimize-runs.jsonl
git commit -m "optimize({target}): {operator}-variant; {one-line approach}"
```

Commit message names the target and the operator. Per CLAUDE.md, no AI vendor or tooling is mentioned.

### F.3 Mechanical pre-PR cross-skill-contamination check (R9)

Run `git diff HEAD~1 HEAD --name-only` on the branch. The expected file list is exactly:

- `plugins/specflow/skills/{target}/SKILL.md`
- `docs/specflow/admin/optimize-runs.jsonl`
- Optionally one test file declared in the run config.

Any unrelated file modification fails the pre-PR check. Refuse with: *"Optimize run modified unexpected files: `{list}`. Expected: `{target}/SKILL.md`, `admin/optimize-runs.jsonl`. Aborting — Surgical discipline broken."* Record `refuse_reason: "cross-skill contamination: {list}"` in Phase H. Retain the branch for inspection. Do NOT open the PR.

### F.4 Install the merge-gate workflow if missing (R12 install sub-clause)

Check for `.github/workflows/optimize-merge-gate.yml`. If absent, write the merge-gate workflow content (idempotent — re-runs do not modify an existing file). The workflow blocks merge on `optimize/*` branches without a `human-approved` label, AND verifies the label applier is a human (not a bot/GH Action) by checking the `actor.type` of the label-applied event.

### F.5 Push branch + open PR

```bash
git push -u origin {branch}
```

Open the PR with `gh pr create`. Title: `optimize({target}): {operator}-variant; {one-line summary}`.

Body sections:

1. **R17 anchor paragraph** — *"We're optimising `{target}` because the discipline-installer pattern compounds across the verifiable-skill set. This honours `simplify`'s four tenets: branch-per-run, sequential variants, no LLM-as-judge, human merge owns taste."*
2. **Eval results table** — every variant: `variant | operator | score | passes_all_gates | reason`.
3. **Variants tried table** — top-3 highlighted with runners-up scores; rejected variants with reasons.
4. **Scoring rationale** — winner pick + tie-break path.
5. **Decline-streak summary** — current streak count + most-recent-decline date for `{target}` (informs the reviewer).
6. **Discipline citations** — the four `simplify` tenets the run honoured (cite `simplify/SKILL.md` Phase E + Phase B.3).
7. **Auto-merge lockout footer** — literal HTML comment `<!-- optimize: human-merge-required -->` AND prose: *"Human merge owns taste. This PR will not auto-merge under any path. See CORE_PRINCIPLES.md."*

### F.6 Verify

- PR exists on `optimize/{target}-{ts}` branch.
- PR body includes the literal `<!-- optimize: human-merge-required -->` comment.
- PR has NO `auto-merge` label applied by the skill.
- Workflow file at `.github/workflows/optimize-merge-gate.yml` exists.

Hand off to Phase G.

---

## Phase G — Auto-merge structural lockout (R12)

Three independent guardrails prevent auto-merge on `/optimize` PRs. All three fire on every run:

### G.1 PR-description HTML comment (R12.a, AC-11.a)

The literal `<!-- optimize: human-merge-required -->` comment is written into the PR body at F.5. Any auto-merge wrapper that scans PR descriptions can detect and refuse on it.

### G.2 No auto-merge label, no `gh pr merge --auto` (R12.b, AC-11.b)

The skill explicitly does NOT call `gh pr merge --auto` and never applies an `auto-merge` label even if one exists in the project. Verified by `gh pr view --json labels` returning no `auto-merge` label authored by this run.

### G.3 GH Action merge gate (R12.c, AC-11.c)

The workflow at `.github/workflows/optimize-merge-gate.yml` blocks merge on PRs whose head branch matches `optimize/*` until a `human-approved` label is applied by a real user. The workflow verifies `actor.type` of the label-applied event is `User` (not `Bot` or `GitHubAction`); automation attempts to apply `human-approved` are blocked.

The skill MUST NOT call any auto-merge mechanism. A run that fires `gh pr merge --auto` OR applies the `human-approved` label is a failed run.

### G.4 Verify

- HTML comment present in PR body.
- No `auto-merge` label on PR.
- Workflow file exists and contains the human-actor check.

Hand off to Phase H.

---

## Phase H — Run record + decline-streak update (R16)

### H.1 Append run record to `admin/optimize-runs.jsonl`

Append a single line:

```json
{"id": "optimize-{slug}-{ts}", "target": "{target}", "branch": "{branch|null}", "variants_tried": N, "winner_variant": M|null, "winner_operator": "{operator|null}", "winner_score": X|null, "score_direction": "maximize|minimize", "all_gates_passed": true|false, "pr_url": "{url|null}", "spend_usd": Y, "started_at": "{iso}", "ended_at": "{iso}", "merge_decision": "pending|null-failed-run", "refuse_reason": "{string|null}", "override_budget_reason": "{string|null}", "override_decline_reason": "{string|null}"}
```

- `merge_decision: "pending"` for runs that opened a PR (updated to `merged` or `closed-without-merge` when the human reviews).
- `merge_decision: "null-failed-run"` for refused runs (eligibility, budget, baseline-eval, lockfile, single-PR, cross-skill-contamination, baseline-sha-mismatch).
- `refuse_reason` is non-null exactly when `merge_decision: "null-failed-run"`.
- `winner_operator` is non-null on records that opened a PR; preserved when `merge_decision` later updates to `closed-without-merge` (load-bearing for B.4's operator-avoid window).

### H.2 Corpus integrity invariants (AC-15)

After the append, verify:

- (a) `id` is unique across all records (no two records share the same `id`).
- (b) Records with `merge_decision: "closed-without-merge"` have a non-null `winner_operator`.
- (c) Records with `merge_decision: "null-failed-run"` have a non-null `target` AND a non-null `refuse_reason`.
- (d) Every line of `optimize-runs.jsonl` parses as JSON in isolation (R14's lockfile is the load-bearing mechanism preventing concurrent writes / partial lines).

A run that violates any invariant is a failed run; surface the violation and refuse to release the lock until corrected.

### H.3 Merge-decision watcher (R12 polling)

The `merge_decision` field updates when the human merges or closes the PR. The watcher path is the GH webhook configured at workflow install OR a polling job (cron, every 6 hours) that reads open `optimize/*` PRs and reconciles their state with `optimize-runs.jsonl`. The skill itself does not block on the merge — Phase I closes the run with `merge_decision: "pending"` recorded; the watcher updates the record asynchronously.

### H.4 Verify

- The new line in `optimize-runs.jsonl` parses as JSON.
- All four corpus integrity invariants hold.
- `merge_decision` is `pending` for opened-PR runs OR `null-failed-run` for refused runs.

Hand off to Phase I.

---

## Phase I — Final disposition

### I.1 Release the lock atomically

Remove `admin/scratch/optimize.lock`. The remove fires on every exit path the orchestrator reaches (successful PR open, refused exit, partial run, all of Phase A/B/F's refusal exits). A path that reaches Phase I without removing its own lock is a failed run.

### I.2 Surface the chat-line summary

On every successful run that opened a PR:

*"Optimize run on `{target}` complete. Top variant: `v{N}-{operator}` (score `{X}`, direction `{maximize|minimize}`). Variants passing all gates: `{P}/{N}`. Budget remaining for `{target}` this week: `${R}/${cap}`. Decline streak for `{target}`: `{C}`. PR: `{url}`."*

For refused runs, surface the structured refusal sentinel from the failing phase (A.2 / A.3 / A.4 / A.5 / B.1 / B.2 / B.3 / B.4 / F.1 / F.3) along with the eligible recovery action.

For runs where all variants failed validation (no PR opened), surface: *"Optimize run on `{target}` produced no passing variants. Run recorded for Phase 3 corpus mining. Inspect `admin/scratch/optimize-{slug}-{ts}/evals/` for the per-variant failure reasons."*

### I.3 Verify before declaring done

1. PR exists on `optimize/{target}-{ts}` branch (or run was refused per a documented sentinel).
2. PR description has the eval-results table + variants table + decline-streak summary + auto-merge lockout footer.
3. PR has the `<!-- optimize: human-merge-required -->` HTML comment.
4. PR has NO `auto-merge` label applied by the skill.
5. `.github/workflows/optimize-merge-gate.yml` exists.
6. `admin/optimize-runs.jsonl` has a new entry with all corpus integrity invariants holding.
7. `admin/scratch/optimize.lock` no longer exists.
8. Weekly spend in `admin/budget/per-skill-tokens.json` reflects this run's cost AND remains within `targetCapUsd` (or override is recorded).
9. No AI vendor or tooling name appears in the PR title, body, branch name, commit message, run record, or chat-line summary (per the project's CLAUDE.md attribution rule).

If any verify step fails, surface explicitly. Do NOT claim the run is complete with missing manifests, missing record, or held lock.

---

## Failure modes

The following are explicit failure modes the skill handles without silent retry. Each maps to a documented user-elected response or a sentinel refusal exit.

- **Target ineligibility (non-binary eval, judgement word, missing eval, missing produces, missing score-direction, eval reads outside produces)** — A.2 / A.3 / A.4 refuse with the specific cited clause.
- **Baseline eval broken** — A.5 refuses with the specific clauses that failed; user fixes the baseline before retrying.
- **Lockfile fresh** — B.1 refuses; second-to-start path exits without writing.
- **Open PR for target** — B.2 refuses; user resolves the prior PR.
- **Budget cap reached pre-flight** — B.3 refuses; user passes `--override-budget {reason}` or waits.
- **Decline streak triggered (manual mode)** — B.4 refuses without override; user passes `--override-decline {reason}` after inspecting `optimize-runs.jsonl`.
- **Decline streak triggered (cron mode)** — B.4 skips; cron picks another target on next pass; no-op record appended.
- **Variant compile/parse failure** — D.2 marks variant fail; loop continues to next variant; does NOT abort run.
- **Frontmatter mutated** — D.1 marks variant fail; loop continues.
- **No-op variant (body byte-identical to baseline)** — recorded as `passes_all_gates: false, reason: "no-op variant"`; loop continues.
- **All variants fail validation** — E.1 records `winner_variant: null`; PR is NOT opened; run record retained for Phase 3 mining.
- **Baseline SHA mismatch (target SKILL.md edited mid-run)** — F.1 refuses PR open; scratch retained; record `refuse_reason: "baseline-sha-mismatch"`.
- **Cross-skill contamination (unexpected file in branch diff)** — F.3 refuses PR open; scratch retained; record `refuse_reason: "cross-skill contamination: {list}"`.
- **Budget cap reached mid-run** — abort cleanly at the next variant boundary; record partial run with `aborted_budget` outcome and the variants completed so far.

---

## Anti-patterns (refuse to do)

- **Auto-merge any PR.** R12 three-guardrail lockout is structural, not configurable-off. The skill never calls `gh pr merge --auto` and never applies the `human-approved` label. A run that does is a failed run.
- **LLM-as-judge inside the loop.** Only the target's machine eval scores variants. No "is this prompt cleaner?" question. No peer-skill judgement. No orchestrator taste. The eval is what the eval is — that is the load-bearing Goodharting protection inherited from `simplify`.
- **Modify multiple skills in one PR.** Single-skill PR per run; R8 branch isolation. F.3's pre-PR file-diff check is the structural enforcement.
- **Modify frontmatter.** R6 invariant — frontmatter (`requires:` / `produces:` / `eval:` / `name` / `description` / `phase`) is the eval-surface contract. Variants that mutate it are rejected pre-eval (D.1 + C.5 defence-in-depth).
- **Cross-variant context bleed.** Each variant generated in a fresh forked context with only the target body + eval contract + operator instruction as inputs. Variants do not see each other; ranking happens after all variants close.
- **Override the decline-streak without `--override-decline {reason}`.** B.4 refuses; the override reason is recorded in `optimize-runs.jsonl` for Phase 3 audit.
- **Skip baseline eval.** A.5 fires on every run. A broken baseline cannot be improved; `simplify`'s discipline is non-negotiable here.
- **Free-form rewrite (no operator).** Each variant invokes exactly one of the six operators. Free-form rewrite drifts toward verbosity (the LLM bias toward "helpful" qualifiers that soften the prompt's signal); operators force narrowing not expanding.
- **Name any AI vendor or tooling** in PR descriptions, branch names, commit messages, run records, or chat-line summaries. Per CLAUDE.md, this is non-negotiable.

---

## Cross-skill integration

- **`simplify`** — the Phase 1 discipline-installer that `/optimize` generalises. `simplify` continues to ship as its own skill (focused on one target — code complexity in a scope); `/optimize` is the catalog-wide pattern. The four discipline tenets (`simplify` SKILL.md "Why this matters") are inherited verbatim: branch-per-run (R8), narrow eval / no LLM-as-judge (R6), human merge owns taste (R12), corpus capture (R16). `/optimize` can target `simplify` itself; the recursive case is left to surface naturally in the corpus rather than be specified up front.
- **`feedback-loop-audit`** — one of the verifiable-skill set targets. `direction: minimize` shape (CONTEXT.md line count). Its current eval may include judgement words ("concrete signals") — A.2's eligibility check refuses until the eval is tightened.
- **`specflow:doctor`** — eligible target (binary eval check; `direction: maximize` shape on count of passing checks).
- **`init`, `format`, `tdd-cadence`, `release-version-check`** — other verifiable-skill set targets per PRD R7. Each requires the structured `score:` block in its `eval:` field per R17 before `/optimize` v1 can run on it; this is a cross-skill schema dependency that ships via separate per-target enhancement PRDs.
- **`/insights`** (Phase 3) — consumes `optimize-runs.jsonl` for cross-target pattern detection (which targets converge, which keep failing variants, which the human always rejects). Pattern parallel to `simplify-runs.jsonl` consumption.
- **`/prune`** (Phase 3) — quarterly-cadence consumer; mines the corpus for stale targets and operator-effectiveness signals.
- **`specflow:budget`** — exposes per-target rolling spend; consumed by B.3's per-target weekly cap.
- **Stale-branch housekeeping** — out of scope for `/optimize` v1. Branch retention contract is ≥30 days (lower bound); upper bound deferred to a separate housekeeping skill.

---

## Reference

- `docs/specflow/features/008-optimize-skill/008-optimize-skill-prd.md` — full requirements R1-R17 and acceptance criteria AC-1 to AC-16.
- `docs/specflow/features/008-optimize-skill/debate-log/prd-gate2/manifest.md` — Gate 2 closing decision; PRD revisions applied (R17 score-direction, R13 decline-streak semantics, R10 `eval_reads ⊂ produces:` strict contract, R12 workflow file ownership, AC-15 corpus-integrity sub-clauses, R13 manual-override clause + AC-16).
- `skills/simplify/SKILL.md` — Phase 1 discipline-installer that `/optimize` generalises. The four tenets (branch-per-run, sequential variants, no LLM-as-judge, human merge) are inherited verbatim.
- `templates/orchestrator-pattern.md` — three primitives (forked sub-agent contexts, file handoff, command substitution); load-bearing for variant generation (C.3) and per-variant eval (D.3).
- `templates/agents/standard/principles/goal-driven-reviewer.md` — orphan-AC + orphan-phase reverse-traceability lens (E4 + E9).
- `CORE_PRINCIPLES.md` — the four principles bound to every phase verify step.
- `skills/budget/SKILL.md` — per-target rolling spend source consumed by B.3.
- `admin/optimize-runs.jsonl` — append-only corpus this skill writes; consumed by `/insights` and `/prune`.
- `karpathy/autoresearch` — referenced study for the loop pattern.
- `forrestchang/andrej-karpathy-skills` — referenced study for the discipline.
