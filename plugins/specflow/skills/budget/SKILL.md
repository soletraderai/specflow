---
name: specflow:budget
description: Visibility on two cost signals — subscription consumption (LLM spend, Codex calls) and per-skill context-window cost. Skills self-report at completion via a structured log entry; this skill aggregates and surfaces the rolling view. Cultural artefact — installs the metric / judge habit before any optimiser exists. Provides the early-warning signal for orchestrator-pattern violations.
status: v2-new
phase: 1
requires:
  - docs/specflow/admin/budget/skill-invocations.jsonl
  - docs/specflow/admin/environment.json
produces:
  - docs/specflow/admin/budget/usage-summary.md
  - docs/specflow/admin/budget/per-skill-tokens.json
eval: usage-summary.md exists for the requested window with subscription totals AND per-skill table; per-skill table covers every skill invoked in the window (no silent omissions); trending-up Δ tokens are flagged with the orchestrator-pattern audit checklist link; skill-invocations.jsonl is append-only and never rewritten.

---

# specflow:budget

Visibility on two distinct cost signals:

1. **Subscription consumption** — LLM spend, Codex calls. Pulled from the harness's billing surface where available; manually reconciled where not.
2. **Per-skill context-window cost** — how many tokens each skill adds to the parent context per invocation. **The early-warning signal for orchestrator-pattern violations.**

Honesty about limits: this skill is **observability + judgment**, not a meter. Exact token counts depend on the harness exposing them; when they're not exposed, the skill works from self-reported invocation records (tool calls made, scratch files written, sub-skill chain depth) as a leak proxy. The pattern of *Avg Δ trending up over time* is the load-bearing signal — the absolute number is secondary.

---

## Inputs

The user invokes you with one of:

- `/specflow:budget` — full report (subscription + per-skill, default window = current month).
- `/specflow:budget --tokens` — per-skill view only.
- `/specflow:budget --spend` — subscription view only.
- `/specflow:budget --window {7d|30d|90d|all}` — narrow or widen the time range.
- `/specflow:budget --skill {name}` — drill into one skill's history.

---

## The data sources

### Source 1 — `admin/budget/skill-invocations.jsonl` (append-only)

Every specflow skill writes one line to this file when its invocation finishes (success or fail). Standard entry shape:

```json
{"skill": "specflow:prd", "invocation_id": "{uuid}", "started_at": "2026-05-05T14:32:18Z", "ended_at": "2026-05-05T14:38:42Z", "status": "ok|aborted|error", "tool_calls": 47, "scratch_files_written": 3, "sub_skills": ["grill", "specflow:brief"], "feature": "001-design-skill", "self_reported_delta_tokens": 850, "notes": ""}
```

`self_reported_delta_tokens` is the skill's best estimate of how many tokens it added to the parent context. Skills that follow the orchestrator pattern (forked sub-agents + scratch handoff + command substitution) report low single-digit-K values; skills that leak report higher.

If the file doesn't exist, create it with `: > admin/budget/skill-invocations.jsonl` (touch via Bash).

### Source 2 — provider billing surfaces

For subscription totals:
- **Anthropic API spend** — read from `~/.config/claude-code/billing.json` if present (harness convention; tolerate absence).
- **Codex spend** — read from `admin/environment.json.tools.codex.usage_log_path` if set; tolerate absence.
- **Other providers** — listed in `admin/environment.json.tools.*.usage_log_path` if the project uses them.

If a billing surface is unavailable, the report says so explicitly: `Anthropic spend — billing surface not exposed; manual reconciliation needed.` Do NOT fabricate numbers.

---

## Phase A — Read state

Read in parallel:
- `admin/budget/skill-invocations.jsonl` (entire file)
- `admin/environment.json` (for billing surface paths)
- Each provider's billing/usage file from the paths in `environment.json` (when present)

Filter `skill-invocations.jsonl` entries to the requested window.

---

## Phase B — Aggregate

### B.1 Per-skill aggregation

For each unique `skill` in the filtered window:
- Count invocations.
- Compute min / max / avg / total of `self_reported_delta_tokens`.
- Sum `tool_calls`, `scratch_files_written`.
- For orchestrator skills, list the sub-skill chain seen across invocations.

Write the structured aggregation to `admin/budget/per-skill-tokens.json`:

```json
{
  "window": {"from": "2026-04-05", "to": "2026-05-05"},
  "generated_at": "2026-05-05T14:50:00Z",
  "skills": [
    {
      "name": "specflow:prd",
      "invocations": 5,
      "delta_tokens": {"min": 600, "max": 1200, "avg": 800, "total": 4000},
      "tool_calls_total": 235,
      "scratch_files_total": 15,
      "sub_skills_seen": ["grill", "specflow:brief"],
      "trend": "stable",
      "leak_signals": []
    },
    {
      "name": "specflow:develop",
      "invocations": 12,
      "delta_tokens": {"min": 1800, "max": 3200, "avg": 2400, "total": 28800},
      "tool_calls_total": 1140,
      "scratch_files_total": 84,
      "sub_skills_seen": ["specflow:test", "specflow:design"],
      "trend": "rising",
      "leak_signals": ["avg-delta increased 40% over last 3 invocations"]
    }
  ]
}
```

### B.2 Trend detection

For each skill with ≥3 invocations in the window:
- Compare the average of the last third to the average of the first third.
- `>20% increase` → `trend: "rising"`, add a `leak_signal` line.
- `>20% decrease` → `trend: "improving"`.
- Otherwise → `trend: "stable"`.

`rising` is the early-warning signal. The audit checklist for fixing leaks lives in `templates/orchestrator-pattern.md` — the report links to it for any rising skill.

### B.3 Subscription aggregation

For each provider with a billing surface:
- Sum the spend over the window.
- Compute spend-per-week trend.

Skills with no billing surface get marked `unavailable` rather than `0` — the distinction matters.

---

## Phase C — Write the report

Generate `admin/budget/usage-summary.md`:

```markdown
# specflow:budget — {window} report
Generated: {YYYY-MM-DD HH:MM}

## Subscription consumption

| Provider  | Spend (window) | Trend (week-over-week) | Source |
|-----------|----------------|------------------------|--------|
| Anthropic | $42.18 | +5% | ~/.config/claude-code/billing.json |
| Codex     | $3.40  | -12% | admin/environment.json:codex.usage_log_path |
| Other     | unavailable | — | no billing surface configured |

**Total this window: $45.58.**

## Per-skill context cost

| Skill | Invocations | Avg Δ tokens | Min | Max | Total Δ | Trend |
|-------|-------------|--------------|-----|-----|---------|-------|
| specflow:prd | 5 | 800 | 600 | 1200 | 4K | stable |
| specflow:brief | 5 | 90 | 70 | 110 | 0.45K | stable |
| specflow:task | 5 | 1100 | 900 | 1500 | 5.5K | stable |
| specflow:develop | 12 | 2400 | 1800 | 3200 | 28.8K | ⚠️ **rising** |
|   └─ via gate-4 | 12 | 1800 | 1400 | 2100 | 21.6K | stable |
|   └─ via gate-5 | 12 | 1600 | 1300 | 1900 | 19.2K | stable |

## Leak signals

- ⚠️ **specflow:develop** — Avg Δ increased 40% over last 3 invocations.
  Likely causes (audit checklist at `templates/orchestrator-pattern.md`):
  - Sub-skills inheriting parent context instead of forking.
  - Tool responses landing in the parent because sub-skills return raw payloads.
  - Scratch files being read with Read tool calls instead of command substitution.

## What's *not* in this report

- Token counts from skills that didn't write a self-report record (rare, but possible — older versions of a skill, or invocations that crashed before the post-hook). The list of skills with missing records: {if any, list}.
- Subscription surfaces marked `unavailable` above — these need manual reconciliation against the provider's web console.
```

---

## How skills should self-report

Every skill that orchestrates other skills, calls tools, or runs for >1 minute should write an entry to `admin/budget/skill-invocations.jsonl` when it finishes. Pattern (in the skill's "Verify before declaring done" section, as the last step):

```bash
cat >> docs/specflow/admin/budget/skill-invocations.jsonl <<EOF
{"skill": "{name}", "invocation_id": "{uuid}", "started_at": "{iso}", "ended_at": "{iso}", "status": "{ok|aborted|error}", "tool_calls": {n}, "scratch_files_written": {m}, "sub_skills": [{...}], "feature": "{NNN-slug or empty}", "self_reported_delta_tokens": {best-effort number}, "notes": "{optional}"}
EOF
```

`self_reported_delta_tokens` is a **best-effort estimate**. Pragmatic heuristics:
- A skill that wrote N scratch files, made M tool calls, invoked K sub-skills (forked), reports roughly `200 + 50*N + 100*M + 200*K` tokens added to the parent.
- Skills that write to the conversation channel (chat output, not files) add their output length / 4 as token estimate.
- Verbose drift over 5K should already trigger a leak audit before the per-skill table catches it.

The number doesn't need to be perfect. The trend is the signal.

---

## Phase 2 / Phase 3 hooks

- **Phase 2** — `specflow:develop` writes detailed sub-skill attribution so the indented orchestrator view becomes the primary debugging surface.
- **Phase 3** — `/optimize` reads `per-skill-tokens.json` to pick optimisation targets; skills with persistent rising trends become candidates for the bounded-autoresearch loop.

---

## What you MUST NOT do

- **Do not fabricate numbers.** Missing billing surfaces report `unavailable`; missing self-reports list the skills with gaps.
- **Do not aggregate outside the requested window.** A `--window 7d` report only covers the last 7 days, period.
- **Do not modify `skill-invocations.jsonl`.** Append-only by contract. Past entries are the audit trail.
- **Do not flag a skill as leaking without citing the audit checklist.** Every `leak_signal` references `templates/orchestrator-pattern.md` so the user has a path forward.
- **Do not invoke other skills automatically.** Budget reads and reports; remediation is user-driven.
- **Do not mention the underlying AI tooling or vendor** outside provider names where they're a literal data source. Per the project's CLAUDE.md, the surrounding language stays neutral.

---

## Verify before declaring done

1. `admin/budget/usage-summary.md` exists for the requested window with subscription section + per-skill table + leak-signals section.
2. `admin/budget/per-skill-tokens.json` updated with the same data in structured form.
3. Every skill in `skill-invocations.jsonl` for the window appears in the per-skill table.
4. Every skill flagged `rising` has a `leak_signals` entry citing the orchestrator-pattern audit checklist.
5. Subscription rows mark unavailable surfaces explicitly (no `0` masquerading as a real reading).
6. `skill-invocations.jsonl` is unmodified (append-only invariant holds).

If any verify step fails, fix the report or surface the limitation explicitly — don't paper over it.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 13 — budget skill + binary acceptance eval block.
- `templates/orchestrator-pattern.md` — the audit checklist for leak signals.
- `skills/develop/SKILL.md` (Phase 2) — primary consumer of the indented orchestrator view.
- `skills/optimize/SKILL.md` (Phase 3) — reads `per-skill-tokens.json` to pick targets.
