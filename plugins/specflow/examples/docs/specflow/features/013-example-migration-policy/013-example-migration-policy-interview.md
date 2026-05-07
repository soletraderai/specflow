# PRD interview — features/013-example-migration-policy

## Original request
> Sprint 2's template churn (PRD + brief shape changes) silently breaks every prior worked example unless we pre-decide a versioning policy. Schema-tag each example with the template version it was authored against; provide a check tool path that lists examples with stale templateVersion. (Risk A from the v2.4+ master plan, surfaced by the Plan agent.)

## Codebase context (pre-grilling)
- `plugins/specflow/examples/docs/specflow/features/00[1-8]-*-prd.md` exist with frontmatter that lacks any version field; they were all authored against the v2.3 template.
- `v2/docs/PRD.md` § Open questions — Sprint 1 work surfaces three "decide-or-build" decisions; Risk A was added by the Plan agent during master-plan validation.
- `plugins/specflow/CHANGELOG.md` releases 2.0 / 2.1 / 2.2 / 2.3 ship without any worked-example versioning hook.
- The master plan's iteration discipline lists `/specflow:prune` and `/specflow:insights` at sprint close; neither currently audits worked examples.

## Round 1 — manual vs automated

**Question.** Should the audit be manual (bash script run at sprint close) or automated (GH Action / CI hook)?

**Answer.** Manual for v2.4. CI automation is a v2.5+ enhancement if dogfood surfaces the need. Reasons: (a) the audit is one-shot per release, not continuous — a CI hook would fire on every commit; (b) the per-stale-example decision (backfill / grandfather / retire) is human; auto-rejection on missing field would force a noisy enforcement that doesn't help the release author make the decision.

## Round 2 — backfill scope

**Question.** Should Sprint 1 backfill all 001-008 examples to `templateVersion: v2.3` proactively, or wait for Sprint 2 to flag them as stale?

**Answer.** Backfill proactively in Sprint 1. Reasons: (a) the script needs every example to carry the field for the audit output to be useful; missing field is a different signal than stale field, and we want the v2.4 release to ship with the field universally present; (b) "informational tag" backfill is mechanical and quick — a single `awk` command. Waiting for Sprint 2 would require *both* a backfill audit AND a stale audit at the same release boundary, doubling the work.

## Round 3 — interview file exemption

**Question.** Should interview files carry `templateVersion` too, or only PRD/tasks/test?

**Answer.** Only PRD/tasks/test. Interview files are open-ended Q&A; they have no canonical schema that could go stale. Adding the field to interviews would be noise without value.

## Topics not discussed

- Whether `templateVersion` should constrain (rather than just inform) — e.g. block PR merges of examples whose tag doesn't match the current release. Out of scope; would require CI enforcement (Round 1 answer).
- Whether to version individual sections of an example (e.g. "PRD section is v2.5 but the manifest is v2.4"). Considered, rejected as over-engineering.
- Whether to record backfill *history* (e.g. `templateVersionHistory: [v2.3, v2.5]` to track that the example was backfilled at Sprint 2). Sufficient for v2.4 to record only the *current* version; history can be reconstructed from git blame on the frontmatter line.
