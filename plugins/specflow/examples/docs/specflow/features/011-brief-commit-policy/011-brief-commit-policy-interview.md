# PRD interview — features/011-brief-commit-policy

## Original request
> The brief composes PRD + interview + manifests into a single self-contained HTML. Default recommendation is committed (diffable for review). Projects with sensitive surfaces or repo-size pressure may prefer gitignored-as-derived. Should the choice surface as a setup-time prompt or a `config.json.brief.commitPolicy` knob?

## Codebase context (pre-grilling)
- `plugins/specflow/skills/brief/SKILL.md` step 9 writes the brief HTML; current version has no policy awareness.
- `plugins/specflow/skills/setup/SKILL.md` Phase 8.2 writes `config.json`; current schema has `specflowVersion`, `linear`, `confidentialPaths`, `teams`.
- `v2/docs/SESSION-HANDOFF.md` flagged this as PRD open question #3, recommended either (a) config knob + setup prompt, or (b) lock to committed-only and document gitignored alternative as a CONTEXT.md recipe.
- No prior worked example demonstrates a config-driven brief policy.

## Round 1 — knob vs convention

**Question.** Lock to `committed` and document gitignored alternative in CONTEXT.md, OR ship the knob?

**Answer.** Ship the knob. The convention-only path leaves projects to discover the override via documentation hunting; the knob makes the choice explicit at setup time and self-documents in the rendered HTML banner. Cost is low (one config field, one prompt, one banner line); benefit is a bounded user-visible policy that does not require memorising a recipe.

## Round 2 — per-feature override?

**Question.** Should briefs be able to override per-feature (e.g. one feature's brief is committed even when project default is `derived`)?

**Answer.** No. Per-feature overrides multiply the policy surface for marginal benefit. Project-uniform is cleaner: reviewers know what to expect from any brief in the repo. If a project has a one-off need, they can hand-edit `.gitignore` exclusions (`!features/special/*-brief.html`) — out-of-band but supported by gitignore semantics, not by specflow.

## Round 3 — switch-policy semantics

**Question.** When a user changes `commitPolicy` post-setup (e.g. `committed` → `derived`), should the skill clean up existing committed briefs?

**Answer.** No. The skill warns about pre-existing committed briefs needing manual `git rm` but does not act. Two reasons: (a) destructive cleanup of committed files is the kind of action that wants explicit user confirmation per file, not auto-fire; (b) git history retains the prior brief versions regardless, so even after `git rm` the bytes-on-history are unchanged — the cleanup is mostly cosmetic. Letting the user own that step keeps the skill in the additive-only camp.

## Topics not discussed

- Whether `derived` mode should also gitignore the brief's CSS/JS dependencies. Not applicable — briefs are self-contained, no external assets per `brief/SKILL.md` rule.
- Whether the policy should affect bulk regeneration (`specflow:brief --all`). Not applicable — the policy is about commit semantics, not about regeneration mechanics.
- Whether there's a third mode (`gitignored-but-cached`). Two-mode design is sufficient; a third complicates the prompt without obvious user demand.
