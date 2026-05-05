---
name: specflow:feedback-loop-audit
description: Audit the rate of feedback the codebase already provides — test coverage, type strictness, e2e health, lint/format gating, build pipeline. Generates the slim admin/CONTEXT.md from the audit findings as a seed for the project's live context document. Five-phase orchestrator — A detect stack, B audit each dimension, C generate CONTEXT.md, D length-cap check, E report.
status: v2-new
phase: 1
requires:
  - docs/specflow/admin/environment.json
produces:
  - docs/specflow/admin/CONTEXT.md
  - docs/specflow/admin/feedback-loop-audit-{timestamp}.md
eval: CONTEXT.md exists; every reference (file path, config key, command) resolves on disk; length within target 500 lines, hard cap 700; audit report covers all five dimensions with concrete signals; rerun is idempotent (regenerates without orphan content).

---

# specflow:feedback-loop-audit

Audit the rate of feedback the codebase provides. Generates `admin/CONTEXT.md` — the slim live document the human/AI maintains as the project evolves. The faster the feedback loop, the more autonomy the AI can exercise safely; auditing the loop's *current* speed is the first step toward improving it.

This is a **5-phase skill**: detect stack → audit five dimensions → generate CONTEXT.md → enforce length cap → report.

---

## Inputs

The user invokes you with one of:
- `/specflow:feedback-loop-audit` — full audit, regenerate CONTEXT.md.
- `specflow:feedback-loop-audit --report-only` — audit but do NOT touch CONTEXT.md (preview mode).

Auto-invocation: `specflow:setup` and `specflow:upgrade` invoke this skill at the end of their flows to seed CONTEXT.md.

---

## Phase A — Detect stack

### A.1 Read environment

Read `admin/environment.json` for already-detected tools.

### A.2 Detect language(s) + framework(s)

Use Glob + Read to detect:
- **JavaScript/TypeScript** — `package.json`, `tsconfig.json`, `tsconfig.*.json`. Note: `strict`, `noImplicitAny`, `noUncheckedIndexedAccess`, etc.
- **Python** — `pyproject.toml`, `setup.cfg`, `mypy.ini`. Note strict mode flags.
- **Go** — `go.mod`. Lint via `golangci-lint`?
- **Ruby** — `Gemfile`. RBS / Sorbet?
- **Other** — record what's detected; don't fail if a language doesn't match.

### A.3 Detect test runners + frameworks

Glob for typical configs: `vitest.config.*`, `jest.config.*`, `playwright.config.*`, `cypress.config.*`, `pytest.ini`, `tox.ini`, `.rspec`, etc.

For each detected runner: capture which `npm`/`pnpm`/`make`/`pytest`/`go test` command runs the suite.

### A.4 Detect CI

Glob for `.github/workflows/*.yml`, `.gitlab-ci.yml`, `.circleci/config.yml`, `azure-pipelines.yml`, `Jenkinsfile`. Read one workflow file to learn the gate set.

---

## Phase B — Audit five dimensions

For each dimension, capture concrete signals (file paths, config keys, current values, last-run state where available).

### B.1 Test coverage

- Unit, integration, e2e — what's measured, what's not.
- Coverage thresholds in config (if any). Cite the file:line.
- Brittleness signals: tests skipped, `xit`/`describe.skip`, recent flaky-test labels.
- Output: `tests.unit`, `tests.integration`, `tests.e2e`, each with `runner`, `command`, `coverage_threshold`, `skipped_count`, `notes`.

### B.2 Type strictness

- TypeScript: read `tsconfig.json`. Capture `strict`, `noImplicitAny`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noFallthroughCasesInSwitch`. Each as boolean + the file:line where it's set.
- Python: read `pyproject.toml` `[tool.mypy]`. Capture `strict`, `disallow_untyped_defs`, `warn_return_any`.
- Go: implicit (statically typed), but capture `gosec` / `govet` config.
- Output: `types.strictness_score = N/M flags enabled`.

### B.3 E2E health

- Last green run (CI history): when did the e2e suite last fully pass?
- Flake rate: read CI logs if accessible; otherwise mark "unknown — manual reconciliation".
- Scope: what fraction of the live UI is exercised? Quick proxy: count of `test()` blocks in e2e specs vs. count of routes in `pages.json`.

### B.4 Lint and format gating

- What runs in CI vs. only locally? Read the CI workflow.
- ESLint / Prettier / Ruff / Rubocop / golangci-lint — config files, gate config, rules turned on.
- Pre-commit hooks: `.husky/`, `.pre-commit-config.yaml`. Are they enforced or skippable?

### B.5 Build pipeline

- What signals go red on a bad commit? CI workflow gates list (typecheck, test, lint, build, e2e).
- What signals only fire locally and could silently land bad code?
- Build time (rough): read CI workflow's last duration if accessible.

### B.6 Write the audit report

Write `admin/feedback-loop-audit-{YYYY-MM-DD-HHMMSS}.md`:

```markdown
# Feedback-loop audit — {YYYY-MM-DD}

## Stack detected
- Language(s): {TypeScript 5.4, Python 3.12, ...}
- Framework(s): {Next.js 14, FastAPI, ...}
- Test runners: {Vitest, Playwright, pytest}
- CI: {GitHub Actions: .github/workflows/ci.yml}

## Dimension scores

### Test coverage
- Unit: vitest, command `pnpm test`, threshold {x}% (vitest.config.ts:14), {N} skipped tests.
- Integration: {present | absent}.
- E2E: playwright, command `pnpm e2e`, threshold not set.

### Type strictness
- TypeScript: {N/M strict flags} enabled (tsconfig.json:6-12).
- Notable gaps: {noUncheckedIndexedAccess off — implicit `any` on array access}.

### E2E health
- Last green: {date or unknown}.
- Flake rate: {%, or unknown — manual reconciliation needed}.
- Surface coverage: {N test blocks vs M routes in pages.json — ~{x}% covered}.

### Lint and format gating
- ESLint: enforced in CI (`.github/workflows/ci.yml:32`). Rules: {extends list}.
- Prettier: format-only, NOT gated in CI.
- Pre-commit: {present + enforced | present + skippable | absent}.

### Build pipeline
- Gates in CI: typecheck, test, lint, build.
- Gaps: e2e runs nightly only, can land broken e2e on main.
- Build time: ~{x}s on average.

## Summary
- Strong signals: {list}.
- Weak signals (improve before granting more autonomy): {list}.
- Recommendations: {2-4 concrete, prioritised}.
```

---

## Phase C — Generate CONTEXT.md

If `--report-only`, skip this phase.

### C.1 Compose CONTEXT.md

Write `admin/CONTEXT.md` with this structure (slim by design — lifestyle template, not exhaustive):

```markdown
# Project context

Live document. Updated by human or AI as the project evolves. Read by every specflow skill. Keep it current; keep it slim.

## Snapshot
- **Updated:** {YYYY-MM-DD}
- **Last audit:** `admin/feedback-loop-audit-{YYYY-MM-DD-HHMMSS}.md`
- **Stack:** {one-line summary from B.1}

## How this project ships

- **Test command:** `{primary command}`
- **Typecheck command:** `{command}` ({strict mode summary})
- **Lint + format:** `{commands}`
- **E2E:** `{command, cadence — on-PR / nightly / on-demand}`
- **CI:** `{workflow file path}`. Gates: {list}.
- **Deploy:** {brief — link out to the deploy doc if it exists}.

## What "done" looks like here

- Tests pass: {threshold}% coverage on changed lines (or "not gated").
- Type-check clean: `{command}` exits 0.
- Lint clean: `{command}` exits 0.
- E2E for touched flows green: {policy — required for PR / required for merge / nightly}.
- {Domain-specific gate the project uses, e.g. "Storybook story exists for new component"}.

## Conventions detected

- **Tests live:** {alongside source / in `__tests__` / in `tests/` — cite file:line example}.
- **State management:** {React Context / Redux / Zustand / etc. — cite file:line example}.
- **Error handling:** {pattern observed, with one file:line example}.
- **Logging:** {library + convention}.
- **API style:** {REST / GraphQL / RPC — cite a representative file}.

## Known weak spots

(From the audit — kept short. Each line points at a real signal.)

- {weak spot 1 from Phase B summary}
- {weak spot 2}

## Things this document deliberately does NOT cover

- Detailed architecture diagrams — those live in `docs/specflow/docs/` if/when they're written.
- Style guides — those live in `admin/rules/`.
- Decision history — those live in `admin/decision-log.md`.
- Per-feature context — those live in `features/NNN-{slug}/`.
```

### C.2 Validate every reference

Before writing, walk every file path / command in the new CONTEXT.md and verify it resolves. If a referenced path doesn't exist, mark it `(check needed)` rather than fabricating. The verify step (below) requires every reference to resolve.

---

## Phase D — Length-cap check

### D.1 Count lines

```bash
wc -l docs/specflow/admin/CONTEXT.md
```

### D.2 Enforce caps

- **≤500 lines** — target. Surface in Phase E as healthy.
- **501-700 lines** — warn. Recommend trimming non-essentials.
- **>700 lines** — hard cap. The skill MUST trim before exiting. Trim from "Conventions detected" first (move overflow to inline comments in the relevant rule files), then from "Known weak spots" (move to the audit report only).

CONTEXT.md is meant to be the *fast* read every skill does on every invocation. Past 700 lines, it stops being fast.

---

## Phase E — Report

Surface in chat:

```
specflow:feedback-loop-audit complete.

Audit report: admin/feedback-loop-audit-{ts}.md
CONTEXT.md: admin/CONTEXT.md ({N} lines, {within target | warn | trimmed to fit cap})

Strong signals:
- {top 2}

Weak signals (consider before granting more autonomy):
- {top 2-3}

Next step: review CONTEXT.md and edit anything that's wrong or missing — it's a live document, not a freeze.
```

---

## What you MUST NOT do

- **Do not fabricate references.** Every file path / config key / command in the audit and in CONTEXT.md must resolve on disk. If you can't verify, mark `(check needed)` — never invent.
- **Do not exceed 700 lines.** The whole point of CONTEXT.md is fast reads.
- **Do not blow away user edits.** Re-running this skill regenerates CONTEXT.md, but if the user has edited sections, preserve their edits in marked sections (`<!-- user-maintained: do not regenerate -->` blocks). Detect those blocks and skip them.
- **Do not invoke `specflow:simplify` or `/optimize` automatically** to "fix" weak signals. Suggest in the recommendations; never execute.
- **Do not mention Claude, Anthropic, or any AI tooling** in CONTEXT.md or the audit report. Per the project's CLAUDE.md.

---

## Verify before declaring done

1. `admin/feedback-loop-audit-{timestamp}.md` exists with all five dimensions populated.
2. `admin/CONTEXT.md` exists (unless `--report-only`).
3. Every file path / command in CONTEXT.md resolves on disk (or is marked `(check needed)`).
4. Length is ≤700 (target ≤500). If it had to be trimmed, the report names what was trimmed and where it went.
5. User-maintained blocks (if any) preserved verbatim.

If any verify step fails, fix it before returning.

---

## Reference

- `docs/PRD.md` Phase 1 scope item 17 — `feedback-loop-audit` + CONTEXT.md.
- `docs/PRD.md` Appendix I — admin folder + self-learning memory loop (CONTEXT.md is part of the live-document layer).
- `skills/setup/SKILL.md` — primary auto-invoker.
- `skills/upgrade/SKILL.md` — secondary auto-invoker (post-migration).
