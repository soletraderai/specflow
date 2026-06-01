# SpecFlow speed-up implementation spec (apply on branch: speedup-conditional-debate)

Plugin v2.14.1 -> 2.15.0. Apply Changes 1-5, bump plugin.json + marketplace.json to 2.15.0, add a MIGRATIONS.md v2.15.0 entry. Branch only - do NOT touch main or ClaimXPro. Owner reviews diff + tests on a real feature before merge.

KEY RISK: specflow:develop (Gates 4+5) does NOT currently read mode: at all. Mode is set in skills/feature/SKILL.md Phase C.1 and read by prd/task/grill only. So Change 2 MUST add a new "mode read" step to develop (A.0.6, mirroring task A.0) or standard mode silently leaves Gates 4/5 at full cost. Most important edit to get right.

The canonical gate shape exists 4x and all must be edited identically:
- Gate 2 = skills/prd/SKILL.md D.2-D.7
- Gate 3 = skills/task/SKILL.md E.2-E.6 (+ cross-task E.4.5)
- Gate 4 = skills/develop/SKILL.md C.2-C.6
- Gate 5 = skills/develop/SKILL.md E.2-E.7
- Closer rules = templates/agents/standard/lifecycle/orchestrator.md (Pass/fail decision rules)

## Change 1 - Conditional Round 2/3 in every gate (highest leverage, mode-independent)
In each gate, after "Wait for all reviewers to return their finding paths" (Round 1), insert this identical block in all 4 gates:
"Round-1 escalation check (per 034-conditional-rounds v2.15.0). After all Round-1 findings land, scan for severity. A finding is load-bearing if severity==block OR (severity==concern AND it touches a load-bearing field - a requirement/AC/trace at Gate 2; a coverage-matrix/anchor/binary-acceptance entry at Gate 3; a plan PRD-anchor/lane/scope entry at Gate 4; an acceptance-clause/contract/schema entry at Gate 5). If NO Round-1 finding is load-bearing (all note or non-load-bearing concern): the AI applies any trivially-accepted note/concern revisions in one pass, SKIPS Round 2 and Round 3 entirely, and the closer records 'Closing decision: passed (Round 1 clean - no load-bearing findings; Rounds 2-3 skipped per 034)'. If ANY Round-1 finding is load-bearing: run Round 2 and Round 3 as documented. A block ALWAYS forces full multi-round debate - safety net intact."
- Gate 2: insert after skills/prd/SKILL.md:451; gate Round 2 (D.4) + Round 3 (D.5) behind it.
- Gate 3: insert after skills/task/SKILL.md:447; gate Round 2 (E.4), cross-task E.4.5, Round 3 (E.5) behind it.
- Gate 4: insert after skills/develop/SKILL.md C.3; gate C.4/C.5 behind it.
- Gate 5: insert after develop E.3; gate E.4/E.5 behind it.
- orchestrator.md Pass/fail rules: add rule 4 "PASS (Round-1 clean) - when no Round-1 finding was load-bearing, gate closes at Round 1 with PASS; Rounds 2-3 not run; recorded as 'passed (Round 1 clean)'. A fast-path of rule 3, not a new disposition."

## Change 2 - Add standard mode, make it default, auto-classify by size/surface
Edit skills/feature/SKILL.md Phase C.1 (lines 136-142): replace 2-bucket light/full with 3 buckets + size/surface auto-classifier (not just verbs):
- light: single-surface, <=1 layer, no assets/design content, goal fits one paragraph, verbs remove/rename/tweak/hide/copy. Caps as today (grill 0-2, Gates 2/3 skipped).
- standard (NEW DEFAULT): 1-2 layers, a handful of requirements, goal <=2 paragraphs, no cross-cutting arch change. Gates 2+3 run 1 round (Change 1 escalation still applies); cross-task only on 5+ tasks; pre-gate Codex folded (Change 3); reviewers capped by Layers Touched (Change 4).
- full: 3+ layers OR new arch/integration OR reference content present OR classifier uncertain. Today's uncapped flow.
Update reflection block + mode: {light|standard|full} frontmatter (line 199). Default on ambiguity = standard (C.2 user-confirm still lets bump to full).
Thread standard through consumers:
- skills/grill/SKILL.md step 48-54: add standard branch (~2-4 rounds, between light 0-2 and full uncapped).
- skills/prd/SKILL.md A.0 (lines 70-78): add standard - Gate 2 runs 1 round, pre-gate Codex folded.
- skills/task/SKILL.md A.0 (lines 57-73): add standard - Gate 3 runs 1 round, cross-task threshold 5, pre-gate Codex folded.
- skills/develop/SKILL.md: NEW A.0.6 "Mode read" step (mirror task A.0) binding MODE from feature.md; in standard run Gates 4/5 at 1 round. THIS IS THE NEW WIRING.

## Change 3 - Fold the two pre-gate Codex passes into the in-gate Codex slot (standard only)
- skills/prd/SKILL.md C.4 (pre-Gate-2 Codex, lines 380-397): wrap whole sub-phase in "When MODE==full:". In light/standard write one-line skip stub ("Codex folded into in-gate reviewer slot (standard mode) - see D.2") and proceed to D.1. In-gate Codex reviewer (D.2, already env-gated) carries it.
- skills/task/SKILL.md B.5 (pre-Gate-3 Codex, lines 298-314): same wrap; standard folds into E.2 in-gate Codex slot.
Keep both pre-gate passes intact in full.

## Change 4 - Cap reviewers per gate by the task's Layers Touched (standard mode)
Tasks carry a Layers Touched field (skills/task/SKILL.md line 231-233; enum Database/Schema, Backend, API, Frontend/UI, Infra, Docs, Tests). Add to the "Identify reviewers" step of Gates 3/4/5 (task-scoped; leave Gate 2 whole - it reviews the PRD):
"Reviewer cap by Layers Touched (per 034, standard mode). In standard, drop reviewer lenses irrelevant to the task's Layers Touched. ALWAYS keep devils-advocate + goal-driven-reviewer (universal). Add surgical-reviewer when >1 file; simplicity-reviewer when a new module/abstraction; think-before-coding-reviewer when Layers includes Backend/API/Database; edge-case-reviewer (Gate 4/5) when Layers includes API/Backend/Database/Frontend-with-state. A pure Docs or pure Frontend/UI-copy task runs 2-3 relevant lenses, not all 5-6. In full, the whole set always fires. NEVER cap when the task scope matches config.confidentialPaths."
Locations: skills/task/SKILL.md E.2 (419-427), skills/develop/SKILL.md C.2 (387-393) + E.2 (559-565).

## Change 5 - Lower default ceilings at the TEMPLATE level (not any project config)
Edit defaults in skills/setup/SKILL.md Phase 8.2 (seeded config.json, lines 340-383) + the 8.2 prompt copy:
- task.maxDurationHours: keep default 1, but reframe the prompt so 8 is not a casual option (offer 1|2|4, with 8/auto as "only if you know you need it").
- task.contextBudget: lower seeded default 80000 -> 60000 (tightens split-prompt before the context cliff). Update prose at line 389.
- Add seeded knobs task.maxReviewRounds (default 3) + develop.crossTaskTaskThreshold (default 5) so mode logic reads from config, not hard-coded.
Do NOT touch any project's admin/config.json (e.g. examples/) beyond the seed/template.

## Also: raise cross-task-review threshold 3 -> 5 (folded into Change 2/5). Do NOT do the combined-Gate-2+3 redesign (item 6/11) now - it changes the produces/requires handoff contract; leave for a dedicated PRD.

## VERIFY (new MIGRATIONS.md v2.14.1->v2.15.0 entry with these greps):
- grep -n 'Round-1 escalation check' plugins/specflow/skills/prd/SKILL.md plugins/specflow/skills/task/SKILL.md plugins/specflow/skills/develop/SKILL.md  (returns matches in all)
- grep -n 'standard' plugins/specflow/skills/feature/SKILL.md  (3rd bucket present)
- grep -n 'A.0.6' plugins/specflow/skills/develop/SKILL.md  (mode-read wired)
- plugin.json + marketplace.json both report 2.15.0
- Structural sanity: trace feature->prd->task->develop; confirm mode binds at each stage.

## GIT (after applying):
git checkout -b speedup-conditional-debate
# apply Changes 1-5 + MIGRATIONS entry + version bumps
git add -A
git commit -m "feat(specflow): conditional debate rounds + standard mode - cut PRD-to-tasks from ~8h toward 2-3h"   (NO AI attribution)
git push -u origin speedup-conditional-debate
# if push fails on osxkeychain, use the gh auth git-credential helper for github.com

## BENCHMARK (structural, already computed - for the owner):
- PRD+tasks today: ~35 reviewer sub-agent forks (Gate2 ~13 + Gate3 ~13 + cross-task ~7 + 2 Codex), ~8h.
- Modified best case (standard + Round-1 clean): ~10 forks PRD+tasks (~70% cut), ~2-2.5h.
- Modified worst case (block at every gate, full escalation): ~5-6h - still below 8h via folded Codex + reviewer caps; gates earn their cost because a real block escalated.
- Quality net intact: a block ALWAYS triggers full debate.
