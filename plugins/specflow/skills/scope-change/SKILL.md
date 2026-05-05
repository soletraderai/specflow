---
name: specflow:scope-change
description: Mid-development scope change — capture the why, update PRD, regenerate affected tasks, flag impacts on in-flight work. Prevents scope drift from going undocumented.
status: v2-new
phase: 3
requires: [docs/specflow/features/{NNN-slug}/NNN-{slug}-prd.md, docs/specflow/features/{NNN-slug}/NNN-{slug}-tasks.md, docs/specflow/features/{NNN-slug}/NNN-{slug}-interview.md]
produces: [docs/specflow/features/{NNN-slug}/NNN-{slug}-prd.md, docs/specflow/features/{NNN-slug}/NNN-{slug}-prd.html, docs/specflow/features/{NNN-slug}/NNN-{slug}-tasks.md, docs/specflow/features/{NNN-slug}/NNN-{slug}-interview.md, docs/specflow/admin/decision-log.md]
eval: PRD diff is reviewable; affected tasks have updated coverage; impact list cites every in-flight artefact; decision-log entry created.
---

# specflow:scope-change

Mid-development scope change. When the intent of a feature changes during execution, this skill makes the change explicit instead of letting it accumulate as silent drift.

**Triggers:**
- "/specflow:scope-change" — explicit user invocation.
- Auto-suggested by `specflow:develop` when it detects the work in flight has drifted from the PRD anchor.

**Behaviour:**
1. **Capture the *why*.** Why is the scope changing? New constraint? Discovery during implementation? Stakeholder ask? The *why* is the most important field — it justifies the change to future readers.
2. **Update the PRD.** Modify `features/NNN-{slug}/prd.md` with the new requirement(s). Re-render `prd.html`.
3. **Regenerate affected tasks.** Re-run `specflow:task` for the impacted scope. Coverage matrix re-computed.
4. **Flag in-flight impact.** List every artefact in flight (open PRs, in-progress tasks, draft test plans) that the scope change touches. Surface to the user for triage.
5. **Decision-log entry.** Append to `admin/decision-log.md` with title, context, decision, rationale (the *why*), date, references.

**Why this exists:** Without this skill, mid-development scope changes happen silently — the PRD goes stale, tasks drift, decisions are lost. Phase 3 self-learning relies on the PRD/task/decision triad staying in sync; `scope-change` is what keeps it honest.

**Verify steps:**
1. PRD updated with the new requirement(s); diff is reviewable.
2. Tasks regenerated; coverage matrix matches the updated PRD.
3. In-flight artefact list is comprehensive (every open PR, every active branch checked).
4. Decision-log entry exists with the *why*.
5. `prd.html` re-rendered.

**Reference:** PRD Phase 3 scope item 4.
