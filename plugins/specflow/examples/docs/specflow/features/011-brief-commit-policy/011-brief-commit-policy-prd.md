---
feature: 011-brief-commit-policy
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
interview: ./011-brief-commit-policy-interview.md
---

# 011 — `brief.html` commit policy

## Vision

Surface the commit policy for `specflow:brief`'s rendered HTML output as a first-class config knob (`config.json.brief.commitPolicy`) rather than an undocumented operator preference. Default `committed` — the brief is the diffable review surface, and most projects benefit from having it land in PR diffs alongside its source PRD/interview/manifests. Alternative `derived` — gitignored as a regeneratable artefact, picked by projects that prefer their repo not carry the rendered HTML (sensitive surfaces, repo-size pressure, or briefs that change frequently enough that diffing them is more noise than signal). The setup skill prompts the user once at install; the brief skill reads the knob and renders a one-line policy banner so the rendered HTML self-documents which choice the project made. No silent defaults, no guessing, no per-feature override (consistency across the project is the point).

## Problem

`specflow:brief` shipped in v2.2 with the rendered HTML committed by convention but no explicit knob. Two project shapes feel the gap. (a) Projects with sensitive surfaces — internal tooling repos where the rendered brief inadvertently surfaces internal terminology in PR diffs that would otherwise stay in the (also-committed but more-rarely-grepped) PRD body. They want the brief regeneratable but absent from the diffable surface. (b) Repo-size-sensitive projects — small monorepos that don't want every feature's 200-300 KB rendered HTML accreting forever. They'd prefer the brief regenerated on demand. The undocumented default forces both groups to discover the convention, then either accept it or hand-write a `.gitignore` line and explain to teammates why their `specflow:brief` outputs aren't appearing in PRs. Neither is great. The fix is a knob, a setup-time prompt, and a brief-rendered banner that surfaces which policy applied so reviewers see at a glance whether to expect the brief committed or regenerable.

## Goals

- New config field `config.json.brief.commitPolicy` ∈ `committed | derived`. Default `committed`.
- `specflow:setup` Phase 8.2 prompts the user with a two-option choice during config write. The prompt explains both modes and recommends `committed` as the default-good answer.
- When user picks `derived`, setup appends `*-brief.html` to `.gitignore` (creates `.gitignore` if absent; idempotent — never duplicates).
- `specflow:brief` reads the knob via `config.json` and renders a one-line policy banner near the top of the HTML, just below the source strip: `committed` → "diffable review surface" framing; `derived` → "regenerate via specflow:brief" framing.
- Banner is part of the deterministic compose: identical inputs (config + source files) produce identical HTML. The policy banner is a function of config, not of fresh state.
- No per-feature override — the knob is project-level. Consistency across briefs is the point of having a config'd policy.
- Migration entry in v2.3 → v2.4 covers the new key + the default + the `.gitignore` snippet.

## Non-goals

- **Per-feature override.** Briefs are uniform across the project. If a project wants a brief committed *for one feature*, that's an `--exception` future enhancement; v2.4 ships project-uniform.
- **Auto-detection of which mode "fits" the repo.** No heuristic on repo size or sensitivity tags; the user picks at setup.
- **Retroactive cleanup.** Switching from `committed` to `derived` post-setup does NOT delete prior committed briefs from the repo's history. Cleanup is the user's responsibility (`git rm` + commit). The skill warns about this on switch but does not act.
- **Brief-rendered banner customisation.** The banner copy is fixed verbatim per the SKILL.md spec. Projects that want different banner text edit the SKILL.md prompt themselves.
- **Linear / external-system sync changes.** This feature only touches the local commit policy; how briefs surface in Linear (or similar) is unchanged.

## Users

- **Setup operators** running `specflow:setup` on a fresh repo. They benefit from being asked once, in plain language, about a policy that affects every brief they'll ever generate. The prompt is short and recommends a default; they can accept-and-move-on or pick `derived` if their context demands.
- **Brief readers** (PR reviewers, future maintainers) opening `NNN-{slug}-brief.html` in a browser. They benefit from the rendered policy banner — at a glance they know whether the brief is the canonical review artefact (committed) or a regenerated view (derived). No ambiguity about whether the file they're reading is up to date with the source PRD.
- **Repo-size auditors** (the rare but real role) running `du -sh` on the repo and wondering where the bytes went. They benefit from `derived` mode being a one-knob switch rather than a manual cleanup project.

## Requirements

- **R1.** `config.json` schema gains the `brief.commitPolicy` field. Default `committed`. Allowed values `committed | derived`. Schema is enforced by `specflow:setup` during the prompt step (any answer outside the two options re-prompts).
  - Trace: skills/setup/SKILL.md Phase 8.2 (v2.4.0).
  - Serves goal: knob exists, default is set.

- **R2.** `specflow:setup` Phase 8.2 prompts the user before writing `config.json`. Prompt copy includes (a) what the brief is, (b) what each mode means, (c) the recommendation. User answer routed through the schema-enforcement check.
  - Trace: skills/setup/SKILL.md Phase 8.2.
  - Serves goal: setup-time choice is explicit, not silent.

- **R3.** When the user picks `derived`, `specflow:setup` appends `*-brief.html` to the project's `.gitignore`. Creates `.gitignore` if absent. Idempotent — never duplicates the line if it's already present from a prior run.
  - Trace: skills/setup/SKILL.md Phase 8.2.
  - Serves goal: `derived` mode is functional out-of-the-box.

- **R4.** `specflow:brief` reads `config.json.brief.commitPolicy` (default `committed` when config or field is absent — backward-compat for v2.2/v2.3 projects). Renders a one-line policy banner near the top of the HTML, below the source strip and above the visual abstract.
  - Trace: skills/brief/SKILL.md step 9 (v2.4.0).
  - Serves goal: rendered HTML self-documents the policy.

- **R5.** Banner copy is fixed:
  - `committed` → "This brief is committed to the repo as the diffable review surface (`config.brief.commitPolicy = "committed"`)."
  - `derived` → "This brief is gitignored as a derived artefact — regenerate via `specflow:brief NNN-{slug}` (`config.brief.commitPolicy = "derived"`)."
  - Trace: skills/brief/SKILL.md step 9.
  - Serves goal: copy is non-customisable so reviewers don't have to parse project-specific phrasing.

- **R6.** Banner inclusion is deterministic: identical inputs (config + source files) produce identical HTML including the banner. Banner is a function of config state, not of when the compose ran.
  - Trace: skills/brief/SKILL.md § Determinism.
  - Serves goal: byte-identical re-compose property is preserved.

- **R7.** MIGRATIONS.md v2.3 → v2.4 entry covers the new key, the default, and the `.gitignore` snippet. Existing v2.3 projects upgrading via `specflow:upgrade` get prompted exactly as fresh setup; their existing committed briefs are not touched.
  - Trace: plugins/specflow/MIGRATIONS.md v2.3 → v2.4 (Sprint 1 close).
  - Serves goal: upgrade path is explicit.

## Acceptance criteria

- **AC-1.** Running `specflow:setup` on a fresh repo and accepting the default produces `config.json` with `brief.commitPolicy: "committed"`. Verifies R1, R2.
- **AC-2.** Running `specflow:setup` on a fresh repo and picking `derived` produces (a) `config.json` with `brief.commitPolicy: "derived"`, (b) a `.gitignore` containing `*-brief.html`. Verifies R1, R2, R3.
- **AC-3.** Running `specflow:setup` twice with `derived` does not duplicate `*-brief.html` in `.gitignore`. Verifies R3 idempotency.
- **AC-4.** Running `specflow:brief` on a feature in a project with `commitPolicy: "committed"` produces HTML containing the verbatim "diffable review surface" banner. Verifies R4, R5.
- **AC-5.** Running `specflow:brief` on a feature in a project with `commitPolicy: "derived"` produces HTML containing the verbatim "regenerate via" banner. Verifies R4, R5.
- **AC-6.** Running `specflow:brief` twice on unchanged inputs produces byte-identical HTML, banner included. Verifies R6.
- **AC-7.** Running `specflow:brief` on a project with no `config.json.brief` field defaults to the `committed` banner with no warning. Verifies R4 backward-compat.
- **AC-8.** v2.3 → v2.4 MIGRATIONS entry references `brief.commitPolicy` with default and `.gitignore` snippet. Verifies R7.

## Open questions

None — the knob is small and bounded.

## See also

- `plugins/specflow/skills/setup/SKILL.md` Phase 8.2 — the producer-side edit
- `plugins/specflow/skills/brief/SKILL.md` step 9 — the consumer-side edit
- `plugins/specflow/MIGRATIONS.md` v2.3 → v2.4 — the migration entry
- `v2/docs/PRD.md` § Resolved decisions — the resolution citation (closes Sprint 1's third PRD question)
