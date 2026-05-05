---
name: simplify
description: Bounded autoresearch loop on the simplification task. Branch-per-run, read-only machine eval (lines deleted + tests + lints + types), $20/week budget, human merge decision. Karpathy-style discipline-installer — the deliverable is the *template* future skills inherit (Phase 3 /optimize across the verifiable-skill set), not the optimised output of this one skill. Every run is bounded to a single scope; refuses sweeping refactors.
status: v2-new
phase: 1
requires:
  - docs/specflow/admin/budget/usage-summary.md
produces:
  - PR on simplify/{scope}-{timestamp} branch
  - docs/specflow/admin/simplify-runs.jsonl
eval: PR exists on a `simplify/...` branch; eval results (lines deleted, test pass/fail, lint pass/fail, type pass/fail) attached to the PR description; all CI gates green on the branch; weekly spend stays under the $20 cap; human merge decision recorded in simplify-runs.jsonl.

---

# simplify

Bounded autoresearch loop on a single simplification scope. The eval surface is **deliberately narrow and machine-checkable** — lines deleted (the score), tests pass, lints pass, types pass. **No human-judge LLM in the loop.** That's where Goodharting starts.

This skill is a **discipline-installer**. The deliverable that matters most is the *pattern* — branch-per-run, narrow eval, human merge — not the immediate output. Phase 3 `/optimize` extends the same shape across the verifiable-skill set.

---

## Inputs

- `/simplify {scope}` — `{scope}` is a path: a single file, a single module, a single function. Refuses paths that match >50 source files or any path that looks like a sweeping refactor (`src/`, repo root, `**/*`).
- `/simplify {scope} --variants {N}` — number of variants to generate (default 5, max 10).
- `/simplify {scope} --dry-run` — go through the loop but do NOT open a PR; report what would have shipped.
- Scheduled invocation — GitHub Actions can fire `/simplify {scope}` on a cron; same loop applies.

---

## Pre-flight gates (refuse the run if any fail)

### Pre.1 Budget gate

Read `admin/budget/usage-summary.md` for the current week. If `simplify`'s spend ≥ $20 this week, refuse:

```
simplify is at $X.XX of the $20.00 weekly cap. Refusing to run.

The budget refreshes Monday. Override only via /simplify --override-budget {reason}, which records to simplify-runs.jsonl with the override reason.
```

The override exists for the rare case the user knows what they're doing; the override REASON is recorded for Phase 3 audit.

### Pre.2 Scope gate

Validate the scope:
- Resolves to a real path on disk.
- Is ≤50 source files (use `find {scope} -type f \( -name '*.ts' -o -name '*.tsx' -o -name '*.py' -o -name '*.go' \) | wc -l`).
- Does not include the repo root, `node_modules`, build outputs, generated files.

Refuse oversized scopes:

```
{scope} resolves to {N} files. simplify caps at 50 files per run.

Pick a tighter scope:
- A single file: {example file from inside the scope}
- A single subdirectory: {example subdir}
- One function: src/auth/session.ts:42-91
```

### Pre.3 Working-tree gate

`git status` must be clean. simplify creates a branch and needs a clean baseline.

```
Working tree has uncommitted changes. simplify needs a clean baseline.
Stash or commit before running.
```

### Pre.4 Eval-surface gate

The project must have at least:
- A test command in `admin/CONTEXT.md` (or detectable via `package.json` scripts).
- A typecheck command (or — for languages without one — explicitly mark as N/A).
- A lint command.

Without these, the read-only eval can't run, and simplify won't ship.

---

## Phase A — Branch + capture baseline

### A.1 Cut the branch

```bash
ts=$(date +%Y%m%d-%H%M%S)
slug=$(echo "{scope}" | sed 's/[^a-zA-Z0-9]/-/g' | tr -s - | sed 's/-$//')
branch="simplify/${slug}-${ts}"
git checkout -b "${branch}"
```

### A.2 Capture baseline metrics

```bash
mkdir -p admin/scratch/simplify-${ts}
{
  echo "Scope: {scope}"
  echo "Branch: ${branch}"
  echo "Started: $(date -u +%Y-%m-%dT%H:%M:%SZ)"
  echo "---"
  echo "Lines (baseline):"
  wc -l {scope}
  echo "---"
  echo "Test command: $(grep -m1 '^- \*\*Test command:\*\*' admin/CONTEXT.md | sed 's/.*`\(.*\)`.*/\1/')"
  echo "Typecheck command: ..."
  echo "Lint command: ..."
} > "admin/scratch/simplify-${ts}/baseline.txt"
```

### A.3 Run eval on baseline (must pass)

If baseline is broken (tests already failing, types already failing, lints already failing), refuse:

```
Baseline eval is not green. simplify cannot improve a broken baseline.
- Tests: {pass | FAIL with N failures}
- Types: {pass | FAIL with N errors}
- Lints: {pass | FAIL with N warnings}

Fix the baseline before running simplify.
```

Why: simplify's contract is "tests still pass, types still pass, lints still pass." If they don't pass at baseline, "still pass" is meaningless — the loop would Goodhart against an undefined target.

---

## Phase B — Variant generation

### B.1 Generate N variants

For each of `--variants` (default 5) iterations:
- Re-read the scope.
- Apply a different simplification strategy — extract no helper, inline a helper, drop a comment block, fold two functions, remove dead code, narrow a type.
- Make the change in the variant branch using forked sub-agent contexts (each variant attempt runs in isolation; only the diff returns).

Each variant writes its proposed diff to `admin/scratch/simplify-${ts}/variant-{N}.patch` (use `git diff > variant-{N}.patch` after staging the variant).

The variants run **sequentially**, not in parallel — parallel attempts would explode the budget and create merge headaches. One variant at a time, fully evaluated, then the next.

### B.2 Per-variant eval

After each variant lands its diff, **reset the working tree** between variants (each variant is a candidate, not cumulative):

```bash
git stash                       # save the variant's diff
git checkout {scope}            # reset to branch baseline
git stash apply stash@{0}       # bring just this variant in
```

Then run the read-only eval:

1. **Lines deleted** — `git diff --stat` against the branch baseline; capture `+N -M` and compute `delta = M - N`. Higher `delta` = more lines deleted = better score.
2. **Tests pass** — run the test command. Exit 0 = pass.
3. **Types pass** — run typecheck. Exit 0 = pass.
4. **Lints pass** — run lint. Exit 0 = pass.

Record per-variant in `admin/scratch/simplify-${ts}/eval-{N}.json`:

```json
{
  "variant": 3,
  "lines_deleted": 47,
  "lines_added": 8,
  "delta": 39,
  "tests": "pass",
  "types": "pass",
  "lints": "pass",
  "passes_all_gates": true,
  "approach": "Inlined the auth-context helper; dropped two private wrappers."
}
```

If a variant fails any gate, `passes_all_gates: false` — it's recorded but not eligible for the pick.

### B.3 No LLM judge

**The only signal that selects between variants is the `delta` score.** No LLM-as-judge inside the loop. No "is this variant cleaner?" question. No taste-driven scoring.

That's the whole point of the discipline. The eval is what the eval is.

If two variants tie on `delta`, pick the one with fewer side-effects (fewer files touched). If still tied, pick the lower variant number (the one generated first).

---

## Phase C — Pick + apply

### C.1 Pick

From the variants where `passes_all_gates: true`, pick the highest `delta`. If none pass all gates, the run fails — surface it explicitly and skip Phase D.

### C.2 Apply the winning variant

```bash
git checkout {scope}
git stash apply {winner-stash}
git add {scope}
git commit -m "simplify: {scope} ({delta} lines deleted)"
```

Use a commit message that names the scope and the delta. Do NOT mention any AI tooling (per CLAUDE.md).

---

## Phase D — Open the PR

### D.1 Push the branch

```bash
git push -u origin {branch}
```

### D.2 Open the PR

Use `gh pr create` with a body that includes the eval results:

```markdown
## simplify — {scope}

**Branch:** `{branch}`
**Score (lines deleted):** **{delta}** ({lines_deleted} removed, {lines_added} added)

## Eval results

| Gate | Status |
|------|--------|
| Tests | ✅ pass |
| Types | ✅ pass |
| Lints | ✅ pass |

## Approach

{The winning variant's approach line, verbatim from eval-{N}.json}

## Variants tried

| # | Approach | Δ | Passed all gates? |
|---|----------|---|-------------------|
| 1 | {approach} | {n} | yes / no |
| 2 | {approach} | {n} | yes / no |
| ... | | | |

## How to review

This PR is the output of a bounded autoresearch loop on `simplify`. The loop's eval surface is intentionally narrow — lines deleted, tests pass, lints pass, types pass. Reviewer's job is the parts the eval can't see: clarity, intent preservation, taste.

If the change feels worse despite the metrics improving, **the metric is wrong** — close this PR and we'll iterate the loop's eval surface (not its decisions).

## Audit trail

- `admin/scratch/simplify-{ts}/baseline.txt` — pre-run state.
- `admin/scratch/simplify-{ts}/eval-*.json` — every variant's eval (not just the winner).
- `admin/simplify-runs.jsonl` — appended this run's record.
```

### D.3 Wait for human decision

The skill does NOT auto-merge. The human owns the merge decision. The skill's loop ends when the PR opens.

---

## Phase E — Record the run

Append to `admin/simplify-runs.jsonl`:

```json
{"id": "simplify-{ts}", "scope": "{scope}", "branch": "{branch}", "variants_tried": 5, "winner_variant": 3, "winner_delta": 39, "all_gates_passed": true, "pr_url": "{url}", "spend_usd": 1.42, "started_at": "{iso}", "ended_at": "{iso}", "merge_decision": "pending"}
```

`merge_decision` starts as `pending`. The human (or a Phase 3 hook) updates it to `merged | closed-without-merge | open-with-feedback` after the review lands. Phase 3 mines the corpus for patterns: which scopes converged, which kept failing variants, which ones humans always reject (signal that the eval is missing something).

---

## What this skill MUST NOT do

- **Do not auto-merge.** Human merge decision is the contract. No exceptions.
- **Do not run an LLM judge in the loop.** The eval surface is read-only and machine-checkable. Adding "is this cleaner?" as an LLM call is exactly the Goodharting failure mode this skill is designed to prevent.
- **Do not exceed the budget.** $20/week is hard. Override is `--override-budget {reason}` only, and the reason is logged.
- **Do not run on broken baselines.** A baseline that already fails any gate cannot be improved; refuse explicitly.
- **Do not run on sweeping scopes.** 50-file cap is hard. Tighter scopes produce cleaner experiments.
- **Do not skip the variants log.** Even on a failed run, every variant's eval gets recorded — that's the corpus Phase 3 mines.
- **Do not mention Claude, Anthropic, or any AI tooling** in commit messages, PR descriptions, branch names, or any user-facing output. Per the project's CLAUDE.md.

---

## Verify before declaring done

1. PR exists on a `simplify/...` branch.
2. PR description has the full eval table + variants table.
3. CI is green on the branch (tests, types, lints — all the gates the loop enforced).
4. `admin/simplify-runs.jsonl` has a new entry with `merge_decision: "pending"`.
5. Weekly spend in `admin/budget/usage-summary.md` is updated and within the $20 cap.

If any verify step fails, surface the failure explicitly — including, for partial runs, "no variant passed all gates."

---

## Why this matters

Karpathy's autoresearch insight: when you let an LLM evaluate its own output, you get reward hacking. When you constrain the eval to a narrow, machine-checkable surface, you get genuine improvement OR you get nothing — but you don't get plausible-looking-but-worse output.

The pattern this skill installs:
1. Branch-per-run (state stays clean; bad runs are cheap to discard).
2. Narrow eval (no LLM judge; no Goodharting).
3. Human merge (taste stays human-owned).
4. Corpus capture (every run logged; Phase 3 learns from the pattern, not from any one run).

Phase 3's `/optimize` extends this shape across `release-version-check`, `format`, `tdd-cadence`, `init`, `feedback-loop-audit`, and this skill itself.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 16 — bounded autoresearch loop.
- `docs/PRD.md` Phase 3 scope item 7 — `/optimize` extends this pattern.
- `docs/PRD.md` Appendix I — admin folder + self-learning memory loop.
- `karpathy/autoresearch` — referenced study for the pattern.
- `forrestchang/andrej-karpathy-skills` — referenced study for the discipline.
- `skills/budget/SKILL.md` — budget enforcement source.
