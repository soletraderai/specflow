# Worked-example versioning policy

Worked examples in `plugins/specflow/examples/docs/specflow/features/NNN-{slug}/` demonstrate skill behaviour. When a SKILL.md changes — adding a new Phase, mutating a frontmatter requirement, changing the canonical PRD shape — the examples authored against the prior contract may go stale. The PRD bullets, the manifest reviewer roster, the task frontmatter fields can all silently diverge from what the skill actually produces.

The fix is mechanical: every worked example's primary artefact frontmatter (PRD, tasks, test) carries a `templateVersion: vX.Y` field naming the plugin minor version the example was authored against. Sprint-close audits flag examples whose `templateVersion` is older than the most-recent template-changing release.

Introduced in v2.4.0 (`013-example-migration-policy`). Mitigates Risk A from the v2.4+ master plan ("Worked-example debt cascade").

## Why this exists

Sprint 2 of the v2.4+ plan changes the PRD template (adds a non-technical "Key Features" section, `015-key-features-section`) and the brief template (adds resources/decisions/phase-split framing, `016-brief-enhancements`). Without versioning, every worked example authored before Sprint 2 silently violates the new template contract. Setup users reading the examples for guidance get the *old* shape; reviewers checking new examples against the *new* shape see drift. Versioning lets us either backfill (rewrite the old examples) or grandfather (mark them with their original template-version so the divergence is documented, not accidental).

## The policy

Every worked example's PRD frontmatter MUST carry `templateVersion: vX.Y` where `vX.Y` is the plugin minor version (e.g. `v2.3`, `v2.4`) the example was authored against. The same field appears on the example's tasks file and test file when those exist. Interview files do NOT carry the field — the interview shape is open-ended and template-version-independent.

The field is informational, not constraining. A `v2.3`-tagged example can still demonstrate v2.4 behaviour if its content happens to match; the field documents *intent*, not *correctness*.

## Backfill rules

When a Sprint 2-style template-changing release ships:

1. The release author audits every worked example's `templateVersion`.
2. Examples whose `templateVersion` is older than the new minor version are marked **stale-template** in the sprint-close report (audit tooling below).
3. The author chooses one of three remediations per stale example:
   - **Backfill** — rewrite the example against the new template, bump `templateVersion` to the new minor version.
   - **Grandfather** — leave the example as-is; document the divergence in `examples/README.md` § "Template-version index".
   - **Retire** — delete the example if it's superseded by a newer worked example (e.g. 002-develop-skill is superseded by a more recent develop dogfood).

Default for chain-skill examples (PRD/task/test/develop/complete): **backfill**. Default for one-pager-spec examples (013, 014): **grandfather** (their content is not template-driven).

## Audit tooling

Bash audit script (run at sprint close):

```bash
# Find all worked-example PRDs with templateVersion frontmatter
find plugins/specflow/examples/docs/specflow/features -name "*-prd.md" \
  | while read f; do
      version=$(awk '/^---$/{flag=!flag; next} flag && /^templateVersion:/{print $2}' "$f" | head -1)
      if [ -z "$version" ]; then
        echo "MISSING $f"
      else
        echo "$version $f"
      fi
    done | sort
```

Output groups examples by `templateVersion`; missing fields appear first as `MISSING`. The release author cross-references the output against the current minor version and chooses backfill / grandfather / retire per stale entry.

## Migration to v2.4

The pre-v2.4 worked examples (`001-design-skill` through `008-optimize-skill`) lack `templateVersion`. Backfill on the **first sprint-close audit that runs the script**:

- 001 through 008 → mark `templateVersion: v2.3` (the minor version they were authored under).
- 009 through 014 (Sprint 1 features) → mark `templateVersion: v2.3` (authored against the v2.3 template; v2.4 ships their PRDs but the template did not change in v2.4).

Only Sprint 2 (template-changing) triggers a `v2.5` template version and a real backfill audit. Sprint 1's `v2.3` tagging is informational — it asserts the existing template applies.

## Worked example

See `examples/docs/specflow/features/013-example-migration-policy/` — its own PRD demonstrates the policy by carrying the field.

## Resolution citation

`v2/docs/PRD.md` § Resolved decisions — 013-example-migration-policy v2.4.0.
