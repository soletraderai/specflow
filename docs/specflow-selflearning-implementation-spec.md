# SpecFlow self-learning implementation spec (apply on branch: speedup-conditional-debate)

Plugin v2.15.0 -> 2.16.0. Apply Changes 1-6, bump plugin.json + marketplace.json to 2.16.0, add MIGRATIONS v2.15.0->v2.16.0 entry. SAME branch as the speed-fix. Branch only - do NOT touch main or any project's docs/specflow/ (ClaimXPro included). Owner reviews diff + runs the LOOP-CLOSES TEST before merge. Full rationale: docs/specflow-improvement-analysis.md Pain-point-2.

PROBLEM (evidence): capture works (lessons.json), reuse does not. ClaimXPro L-001 recurred 4x, L-004 5x. Three defects: (1) specflow:learn reads admin/plugin-findings.jsonl which NO project produces -> auto-promotion never runs; (2) lessons are prose not runnable -> re-derived each time; (3) matched lesson is a hint not a gate -> ignored.

PRE-EXISTING ISSUES (call out in MIGRATIONS Known-stale, do NOT fix here): (a) templates/admin/lessons-registry.md schema diverges from the LIVE schema in skills/test/SKILL.md - treat skills/test/SKILL.md as source of truth (it matches disk); (b) specflow:insights referenced in 2 docs but does NOT exist - the real clusterer is specflow:learn; leave refs, flag them.

## Canonical edit map (verify EVERY anchor against the real file before editing; flag any that don't match, don't guess)
- C1 one-corpus: skills/learn/SKILL.md (requires L7, Inputs L46, schema L52-79, A.2 L102-118, B.3 L159-172, C/D L190-362, cross-skill L507-520); skills/test/SKILL.md D.4.5 L492-496
- C2 test_fragment: skills/test/SKILL.md entry-shape L527-544, field-semantics after L556, D.2 Q-set L397-425, D.4 step1 L465-468, D.6 verify L511-518; templates/admin/lessons-registry.md L11-43
- C3 required-check: skills/test/SKILL.md B.0 L111-129, B.4 L227-235, Verify L642, Query-Inject L595; skills/task/SKILL.md A.4 L142
- C4 auto-promote-CI: skills/test/SKILL.md D.4 prompt L490, status L560, promotion L606-607; skills/learn/SKILL.md Phase D
- C5 prune: skills/test/SKILL.md supersession-B L602-603, status L560, query-filter L592
- C6 version: .claude-plugin/plugin.json L3, .claude-plugin/marketplace.json L10+L17, MIGRATIONS.md new entry above the v2.14.1->v2.15.0 block

## C1 - ONE CORPUS (lessons.json is the single SoT; repoint specflow:learn)
- skills/test/SKILL.md D.4.5 (L492-496): replace body so the auto-fire invokes 'specflow:learn --feature {NNN-slug}' reading admin/lessons.json (NOT plugin-findings.jsonl). Fire-and-forget for the parent; on failure log '[test: :learn auto-fire failed - {reason}; corpus retains the lesson]' and continue D.5. Remove the plugin-findings.jsonl reference entirely.
- skills/learn/SKILL.md: re-aim (not rewrite) the 5-phase orchestrator from plugin-findings.jsonl to lessons.json:
  - frontmatter requires (L7): -> 'docs/specflow/admin/lessons.json (the corpus; mutable JSON array - schema owned by skills/test/SKILL.md)'.
  - Inputs (L46): no-op when lessons.json missing or [].
  - schema block (L52-79): replace with pointer + cluster rule: cluster_key = sorted-lowercased tags joined by +, scoped to status==active; two lessons cluster iff tags overlap >=2 AND share >=1 surface tag (ui|data-model|api|auth|migration|infra|cli|docs); cluster count = distinct occurrences[].feature summed across members (NOT lesson count) so L-001(4)/L-004(5) qualify alone.
  - A.2 (L102-118): Read admin/lessons.json, parse as JSON array; 3 exit branches (missing / [] / >=1).
  - B.3 (L159-172): use cluster_key; qualify at count>=3.
  - C/D (L190-362): keep tier-A auto-apply but a RUNNABLE lesson routes to C4's misc-task CI path, not just guidelines.md. Non-runnable recurrence keeps guidelines.md append with frontmatter id/source_lesson_ids/auto_applied_by.
  - cross-skill (L507-520) + anti-pattern (L507): delete plugin-findings.jsonl/insights lines; 'test Phase D produces lessons.json; learn consumes it; no separate findings file.'

## C2 - test_fragment FIELD (runnable, reusable)
- skills/test/SKILL.md entry-shape (L527-544): add after remediation, before status:
  "test_fragment": { "kind": "grep | testcase | ci-check", "assertion": "<verbatim copy-pasteable line>", "scope": "<glob>", "expect": "<binary pass phrase>", "runnable": true|false }
- field-semantics (after L556): add bullet defining each sub-field; required for every NEW escape lesson; pre-2.16.0 lessons with no fragment read as runnable:false and Phase B falls back to prose remediation (backward-compatible).
- D.2 (L397-425): add Q4 capturing the reusable check (grep line+path / testcase skeleton / CI command). If user says skip, derive kind by keyword (path/grep->grep; exit-0/CI/yarn|npm|pnpm cmd->ci-check; else testcase), set runnable, PRESENT derived fragment for confirm before writing. Never write an unseen fragment.
- D.4 step1 (L465-468): new lesson appends with kind:escape,status:active,first_seen,occurrences,+the Q4 test_fragment. Recurrence on a pre-2.16.0 lesson: prompt Q4 + backfill the fragment (the one permitted post-write addition - it's forward-tooling not audit).
- D.6 verify (L511-518): add check 6 - new/backfilled entry has test_fragment with non-empty kind/assertion/scope/expect + boolean runnable.
- templates/admin/lessons-registry.md (L11-43): add a test_fragment row for parity; do NOT re-converge the divergent field names.

## C3 - MATCHED LESSON = REQUIRED CHECK (the enforcement - stops recurrence)
- skills/test/SKILL.md B.0 (after L125): partition matched active lessons REQUIRED vs advisory. REQUIRED when test_fragment.scope glob overlaps an in-scope task's Scope path AND tags overlap >=1 surface tag. A REQUIRED lesson's test_fragment MUST become a concrete test case in B.1 (ci-check/grep -> a runner/grep case with the assertion verbatim + expect as pass criterion; testcase -> the skeleton as Status:red). Tag 'Source: lesson L-NNN (REQUIRED)'. Surface the REQUIRED-vs-advisory split in chat. Write the REQUIRED set to admin/scratch/test-{slug}-{ts}/required-lessons.json ([{id,test_fragment,derived_tc_id}]) - the file B.4 reads.
- B.4 (after L233): add BLOCKING check 5 - read required-lessons.json; for every entry confirm a plan test case carries 'Source: lesson L-NNN (REQUIRED)' AND its pass criterion contains the assertion verbatim (grep/ci-check) or named skeleton (testcase). If ANY required lesson uncovered, the plan does NOT pass B.4 - refuse to proceed to Phase C (or in --plan-only refuse 'complete') with: 'Plan blocked: required lesson L-NNN ({title}) has no covering test case. Add the case (assertion: {assertion}, scope {scope}) or run specflow:scope-change to retire the lesson before this plan can close.'
- Verify-before-done (L642): replace with terminal check 7 - every REQUIRED matched lesson has a covering case embedding its fragment verbatim; advisory ones surfaced or explicitly declined; a 'done' with an uncovered REQUIRED lesson is a FAILED run.
- Query-Inject (L595): update the test sentence to describe the REQUIRED partition + B.4 gate; for task, a REQUIRED lesson forces a lesson-anchor field or uncovered-lessons decision.
- skills/task/SKILL.md A.4 (after L142): mirror - a matched lesson with fragment.scope overlapping a task Scope is REQUIRED; the covering task MUST carry lesson-anchor: L-NNN + its acceptance references the fragment's expect; if no task covers a REQUIRED lesson the uncovered-lesson prompt becomes BLOCKING.

## C4 - AUTO-PROMOTE recurring RUNNABLE lesson into a CI check (fixes L-004 'still not in CI')
- skills/test/SKILL.md D.4 prompt (L490): when occurrences>=3 distinct features AND test_fragment.runnable==true: prompt 'L-NNN occurred n times + has a runnable check ({assertion}). Promote to CI gate and/or guideline? [ci/rule/both/no]'. On ci/both: write admin/scratch/misc-payload-{ts}.json per skills/misc/SKILL.md auto-invocation schema (trigger:rule-violation, calling_skill:specflow:test, title:'Wire L-NNN check into CI: {assertion}', scope inferred from fragment.scope [apps/web->WEB, apps/expo->MOBILE, apps/backend->BACKEND, else SHARED], priority:P1, description with verbatim assertion/scope/expect/occurrences, verification:'command runs in CI as required PR status check on {scope}, blocks merge on non-zero'), then 'specflow:misc --auto admin/scratch/misc-payload-{ts}.json'; set lesson status:promoted-to-ci + promoted_to_ci:'misc-task MISC-NNN'. On rule/both also do guideline draft. runnable==false/absent -> today's guideline behaviour (status promoted-to-rule).
- status semantics (L560): add 'promoted-to-ci' - stays active-equivalent for querying (still REQUIRED) until prune confirms CI live + no recurrence -> resolved.
- promotion section (L606-607): append para - runnable fragment promotes to CI (specflow:misc --auto) not a prose guideline; prose is read-not-enforced, CI is enforced; this is the L-001/L-004 fix.
- skills/learn/SKILL.md Phase D auto-apply: same CI routing for tier-A clusters whose members have runnable fragments; cap 3/run.

## C5 - PRUNE to resolved (only active queried)
- skills/test/SKILL.md supersession-B (L602-603): when a real test execution on a feature overlapping an active/promoted-to-ci lesson has a case sourced from it that PASSED and this run added no new occurrence, prompt at end of Phase C 'L-NNN check ({assertion}) passed with no recurrence. Mark resolved? [yes/no/skip]'. On yes: status:resolved + append validating feature to occurrences with note. resolved lessons filtered OUT of B.0/task-A.4 query; preserved as audit; if gap recurs, D.3 similarity re-opens (back to active).
- status (L560): tighten resolved definition.
- query-filter (L592): select status in {active, promoted-to-ci} (NOT resolved/superseded) AND tags overlap >=1; cap 5 unchanged.

## C6 - VERSION + MIGRATION
- plugin.json L3 2.15.0->2.16.0; marketplace.json L10+L17 2.15.0->2.16.0.
- MIGRATIONS.md: new '## v2.15.0 -> v2.16.0' entry ABOVE the speed-fix v2.14.1->v2.15.0 block. Cover: scope of C1-C5, Steps (existing lessons keep working - no test_fragment read as runnable:false + prose fallback, NOT REQUIRED until backfilled on next recurrence; no plugin-findings.jsonl to migrate; config untouched, resolved excluded from the cap-5), Backups (lessons.json.bak unchanged), KNOWN-STALE-REFERENCES (the lessons-registry.md schema divergence + the non-existent specflow:insights - flag, don't fix), Verify (the grep block below), Rollback (code-only revert, no data migrated).

## LOOP-CLOSES TEST (run against a /tmp fixture, NOT ClaimXPro - this is the PROOF for the owner)
Part 0 fixture:
  SF=/tmp/sf-loopclose-$(date +%s); mkdir -p "$SF/docs/specflow/admin/rules" "$SF/docs/specflow/admin/scratch" "$SF/docs/specflow/features/050-widget-colors/debate-log/tasks-gate3" "$SF/apps/expo/components"
  printf '{"stack":["expo","react-native","typescript"],"cli":{"codex":{"available":false}}}\n' > "$SF/docs/specflow/admin/environment.json"
  printf '{"audiences":[{"name":"field-tech"}]}\n' > "$SF/docs/specflow/admin/profiles.json"
  for f in glossary guidelines non-negotiable; do printf '# %s\n' "$f" > "$SF/docs/specflow/admin/rules/$f.md"; done
  printf '{"routes":[]}\n' > "$SF/docs/specflow/admin/pages.json"; printf '{"tasks":[]}\n' > "$SF/docs/specflow/admin/task-history.json"
  printf 'export const c = "#ff0000";\n' > "$SF/apps/expo/components/Badge.tsx"
Part 1 seed lesson L-900 (kind escape, tags expo/ui/infra, test_fragment ci-check assertion "rg -n --pcre2 '#[0-9a-fA-F]{6}' apps/expo -g '*.tsx'" scope apps/expo/** expect 'zero matches' runnable true, status active, 3 occurrences across 3 features) into "$SF/docs/specflow/admin/lessons.json"; seed a PRD (050-widget-colors, domain ui, R1 touches apps/expo) + tasks.md (T1 Scope apps/expo/components/Badge.tsx Layers Frontend/UI) + a closed Gate-3 manifest.
Part 2 run 'specflow:test 050-widget-colors --plan-only' against $SF and assert:
  (a) SURFACES+REQUIRED: required-lessons.json exists under admin/scratch/test-050-widget-colors-* and contains L-900.
  (b) FRAGMENT INJECTED: the written test plan contains 'Source: lesson L-900 (REQUIRED)' AND the assertion verbatim.
  (c) GATE BLOCKS: delete the L-900 case from the plan, re-run; the run MUST emit 'Plan blocked: required lesson L-900' and MUST NOT emit 'Test plan synthesised'.
Part 3 auto-promote: trigger D.4 promotion (or specflow:learn --feature 050-widget-colors) over the seeded corpus; assert a misc-payload-*.json emitted with calling_skill specflow:test + the CI command verbatim, and the lesson flips to promoted-to-ci.
Part 4 source grep verify (from /Users/marepomana/Web/specflow):
  grep -n 'test_fragment' plugins/specflow/skills/test/SKILL.md
  grep -n 'lessons.json' plugins/specflow/skills/learn/SKILL.md ; ! grep -q 'plugin-findings.jsonl' plugins/specflow/skills/learn/SKILL.md
  grep -n 'REQUIRED' plugins/specflow/skills/test/SKILL.md ; grep -n 'required-lessons.json' plugins/specflow/skills/test/SKILL.md
  grep -n 'Plan blocked: required lesson' plugins/specflow/skills/test/SKILL.md
  grep -n 'promoted-to-ci\|misc --auto' plugins/specflow/skills/test/SKILL.md
  grep -n 'Mark resolved\|status: "resolved"' plugins/specflow/skills/test/SKILL.md
  grep -n '2.16.0' plugins/specflow/.claude-plugin/plugin.json .claude-plugin/marketplace.json
  grep -n 'specflow:learn --feature' plugins/specflow/skills/test/SKILL.md
All greps hit + both negative checks pass + Part 2 (a/b/c) + Part 3 = loop closes (capture -> retrieve as REQUIRED over single corpus -> enforce via B.4 block + CI promotion).

## GIT (on the existing speed-fix branch)
git checkout speedup-conditional-debate ; (apply C1-6 + MIGRATIONS + version bumps) ; git add -A
git commit -m "feat(specflow): close the self-learning loop - single corpus, runnable test_fragment, required-check enforcement, CI auto-promotion"   (NO AI attribution)
git push origin speedup-conditional-debate   (if push stalls on git-credential-osxkeychain use the gh auth git-credential helper for github.com)
