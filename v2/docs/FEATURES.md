# Specflow features — inventory

**Status:** v2.4.0 shipped. Sprint 2 next (target v2.5.0). All v2.x original PRD open questions resolved.
**Companion docs:** `ROADMAP.md` (what to do), `SESSION-HANDOFF.md` (current state + locked decisions).

---

## Shipped (v2.0–v2.4)

High-level only — full detail in CHANGELOG + worked examples under `plugins/specflow/examples/docs/specflow/features/`.

| ID | Feature | Version | Worked example |
|----|---------|---------|----------------|
| 001 | design skill | v2.0.0 | `001-design-skill/` |
| 002 | develop skill | v2.0.0 | `002-develop-skill/` |
| 003 | complete skill (retro) | v2.1.0 | `003-complete-skill/` |
| 004 | decision skill | v2.1.0 | `004-decision-skill/` |
| 005 | scope-change skill | v2.1.0 | `005-scope-change-skill/` |
| 006 | insights skill | v2.3.0 | `006-insights-skill/` |
| 007 | prune skill | v2.3.0 | `007-prune-skill/` |
| 008 | optimize skill | v2.3.0 | `008-optimize-skill/` |
| 009 | pages-policy | v2.4.0 (decide-not-build) | n/a — `pages.json` is lazy-populated by `specflow:test` |
| 010 | design-readback | v2.4.0 | `010-design-readback/` |
| 011 | brief-commit-policy | v2.4.0 | `011-brief-commit-policy/` |
| 012 | config-skill-toggles | v2.4.0 | `012-config-skill-toggles/` |
| 013 | example-migration-policy | v2.4.0 | `013-example-migration-policy/` |
| 014 | team-bridge-spec | v2.4.0 | `014-team-bridge-spec/` |

---

## Sprint 2 — template + doctrine churn (target v2.5.0)

One-shot disruption window: every feature touches a primary contract. One MIGRATIONS entry, one `specflow:upgrade` impact pass, one round of worked-example backfill (using the 013 versioning policy from Sprint 1).

### 015 — key-features-in-brief *(merged into 016 — see below)*
**Status:** Folded into 016-brief-enhancements as a fourth section. PRD stays purely technical; the brief carries the non-technical Key Features overview for the user's own scanning. ID 015 stays allocated (allocation rule: never reuse) but the work is delivered as part of 016.

### 016 — brief-enhancements
**Drive:** chat feedback — "the brief should be presented to a minor technical level" + "we need to list key features within a feature in non-technical format" (the Key Features section, originally 015, lives in the brief — not the PRD — because it's for the user's own overview, not a build-chain contract).
**Inspiration:** `knowledge/pocock-real-feature-build.md` (PRD sketches modules/interfaces first; ubiquitous-language updates inline) — reinforces the brief as a layered, interface-first artefact rather than a flat narrative.
**Scope:** Extend `specflow:brief` to add four sections — each with a **visual rendering in the HTML brief** (the brief is the user-facing visual artefact; sections are not just text dumps):
1. **Key Features** — non-technical overview of what's being built and how it'll look. Lives at the top of the brief (after Vision / scope-at-a-glance). HTML rendering: visual card grid with one card per key feature (title + one-line description + optional thumbnail/icon). Not in the PRD.
2. **Resources** — links to docs, design files, prior briefs, related Linear issues. HTML rendering: link cards with favicon + title + source-type pill (e.g. "Linear", "Design", "Doc").
3. **Key decisions** — choices made during PRD synthesis paired with one-liner rationale. HTML rendering: a decision table (Decision · Why · Source) — visually distinct from the comparison block so reviewers can scan choices vs alternatives at a glance.
4. **This phase / next phase** — split scope. HTML rendering: a two-column visual split (left = "this phase, shipping now"; right = "next phase, parked") so the iteration boundary is visible at a glance.

Existing brief has 3-column scope-at-a-glance (`brief/SKILL.md:166`) and `:::comparison`, `:::flow`, `:::scope`, `:::tree` structured blocks. Add four new blocks: `:::key-features`, `:::resources`, `:::key-decisions`, `:::phase-split` — each with deterministic HTML output and inline CSS (no JS, self-contained, per the existing brief contract).
**Files:** `plugins/specflow/skills/brief/SKILL.md`; brief HTML template + CSS for the four new blocks; structured-block vocabulary.
**Done when:** brief renders all four new sections **visually in the HTML output** (not just markdown); worked example shows the rendered HTML with each block populated; PRD remains purely technical (no Key Features section); brief stays self-contained (inline CSS, no JS).

### 017 — tdd-discipline
**Drive:** chat feedback — "Development needs to implore TDD; we must pass off work to another agent to review externally then provide feedback." Plus: "TDD should inherit Matt Pocock's Red, Green approach."
**Inspiration:** `knowledge/pocock-software-fundamentals-matter-more.md` (canonical Red/Green/Refactor definition with "forces small steps" justification — strongest single source); `knowledge/pocock-ai-coding-real-engineers.md` (TDD as the implementation contract for AI agents); `knowledge/pocock-real-feature-build.md` (AFK Ralph + edge-cases-surfaced-only-in-QA).
**Scope:** Extend `specflow:develop` Phase D to invoke `specflow:test --plan-only` BEFORE Phase E code execution. Tests-first becomes mandatory for Yellow lane; configurable for Green. Adopt Pocock's **Red → Green → Refactor** framing as the canonical TDD shape: Phase D writes a failing test (Red); Phase E makes it pass minimally (Green); Phase F refactors with the test as guard. Each lane has explicit Red/Green/Refactor sub-phase markers in the develop manifest.
**Files:** `plugins/specflow/skills/develop/SKILL.md` (Phase D, E, F); `plugins/specflow/skills/test/SKILL.md` (`--plan-only` flag); `plugins/specflow/CORE_PRINCIPLES.md` (TDD section).
**Done when:** Yellow-lane tasks block Phase E without a test plan artefact; Green lane respects `config.json.develop.tddRequired`; develop manifest carries `red`, `green`, `refactor` markers per task; worked example demonstrates the full Red/Green/Refactor flow.

### 022 — cross-task-review
**Drive:** features.md item 1.
**Inspiration:** `knowledge/pocock-real-feature-build.md` (issue-sizing heuristic + merging-too-small + PRD-to-issues judgment); `knowledge/medin-piv-loop.md` (drift detection: comparing PR back to ticket); `knowledge/medin-parallel-agentic-playbook.md` (reviewer-never-sees-writer's-chat — pairs with 027).
**Scope:** Extend `specflow:task` Phase D / Gate 3 with a **whole-set coherence + improvement pass** in a **fresh context window** (no exposure to the writer's chat that produced the task list — per 027-reviewer-context-isolation). Two lenses applied to the entire task list as a single artefact:
1. **Coherence** — do tasks work together? Flow correctly? Not overlap? All trace back to PRD requirements + goal?
2. **Better arrangement** — given the whole set, is there a *better* way to handle this? Could two tasks merge? Could one split? Is there a different ordering that reduces dependency tangles? Is there a missing task the per-task lens couldn't see?

Currently Gate 3 reviews each task individually; no whole-set lens, no fresh-context discipline, no improvement pass.
**Files:** `plugins/specflow/skills/task/SKILL.md` (Phase D); Gate 3 manifest reviewer prompts; reviewer-isolation contract (per 027).
**Done when:** Gate 3 produces a whole-set finding (coherence + improvement) alongside per-task findings; the reviewer ran in a fresh context (manifest's `writer_id` ≠ `cross_task_reviewer_id`); worked example shows both a coherence issue AND a better-arrangement suggestion surfaced + resolved.

### 025 — sprint-task-flagging
**Drive:** features.md item 2; enables 020-sprint-skill input format.
**Inspiration:** `knowledge/medin-parallel-agentic-playbook.md` (issue-as-spec / PR-as-validation); `knowledge/medin-archon-livestream.md` (artifact directory passed between nodes).
**Scope:** When `specflow:task` synthesises tasks, flag which can be batched into a sprint together via a `sprint-bucket: N` field on each task entry.
**Files:** `plugins/specflow/skills/task/SKILL.md`; task synthesis output schema.
**Done when:** every task in a fresh synthesis carries `sprint-bucket: N`; documented bucket-assignment heuristic; worked example shows multi-bucket task list.

### 029 — single-context-window per task implementation
**Drive:** chat feedback citing a Cole Medin video — "for the implementation of a task, it needs to be done all within one context window so it has all of the information that it needs to complete the task." Elevated to locked-in architectural decision in SESSION-HANDOFF (no re-litigation).
**Inspiration:** `knowledge/medin-wisc-framework.md` (context rot as the dominant failure mode; "just in time, not just in case"; sub-agents for research-only isolation so the writer's context stays intact); `knowledge/pocock-ai-coding-real-engineers.md` (smart zone vs. dumb zone — ~100K token cliff regardless of context window; clear-don't-compact discipline); `knowledge/medin-piv-loop.md` (fresh session per phase — implementation IS the single window for that task).
**Scope:** Hard rule: every task's implementation (`specflow:develop` Phases D Red → E Green → F Refactor — per 017) runs in a **single agent context window**. No mid-task compaction. No cross-session resumption mid-implementation. The agent loads everything it needs once at the start of execution and runs the task to completion (or escalates) within that window.

**Implications across the pipeline:**
1. **Task synthesis (`specflow:task`)** — every task carries a `context-budget-estimate` field at synthesis time. The estimate sums: PRD slice (R/AC anchors) + task spec + relevant lessons (018 query result) + per-task manifest scaffold (019) + codebase-context payload (files the task will read) + test plan (017). Tasks whose estimate exceeds the smart-zone budget (default 80K tokens, configurable via `config.json.task.contextBudget`) are auto-flagged for split — they cannot proceed to develop without re-synthesis.
2. **Cross-task review (022)** — its "better arrangement" lens explicitly checks: are any tasks oversized? Should two tasks merge to share context? Should one split because its budget overflows?
3. **Sprint-bucket assignment (025)** — `sprint-bucket: N` respects context budgets too. Tasks in the same bucket don't share context (each runs in its own window), but the bucket sizing is informed by per-task budgets so the developer doesn't approve a sprint where any single task can't fit.
4. **Sprint-plan gate (020)** — visualises per-task `context-budget-estimate` so the developer sees at-a-glance whether anything is risky.
5. **Develop Phase A pre-flight** — re-checks the budget against the actual loaded context before opening Phase D. If estimate-vs-actual diverges by ≥20%, develop pauses and asks the developer to either approve the over-run, drop optional context, or re-route the task to scope-change for splitting.
6. **No mid-task compaction** — develop's running rule: if context approaches the cliff during D/E/F, the agent escalates to the developer rather than compacting. Compaction during implementation is treated as a defect signal (the task was sized wrong).

**Files:** `plugins/specflow/skills/task/SKILL.md` (synthesis adds `context-budget-estimate`); `plugins/specflow/skills/develop/SKILL.md` (Phase A budget pre-flight; Phase D/E/F no-compaction rule); `plugins/specflow/skills/sprint/SKILL.md` (per-task budget surfaced in the sprint-plan gate; per 020); `config.json.task.contextBudget` block; `plugins/specflow/templates/admin/single-context-task.md` (new doctrine doc spelling out the rule + the compaction-is-a-defect-signal stance).
**Done when:** every task in a fresh synthesis carries `context-budget-estimate`; develop's Phase A blocks oversized tasks pre-emptively; develop's D/E/F refuses to compact (escalates instead); sprint-plan gate surfaces budgets; doctrine doc is the citation point; worked example demonstrates a deliberately-oversized task being caught at synthesis (split via scope-change) and the resulting two-task pair fitting cleanly through develop.

---

## Sprint 3 — operational runtime instrumentation (target v2.6.0)

All three extend the develop/test runtime layer. Co-design schemas so `task-manifest.json` + `lessons.json` + brand-check question set share tag vocabularies and lifecycle conventions.

### 018 — lessons-registry
**Drive:** uncommitted in-flight work; ~90% spec-complete in `task` + `test` skills. The user's emphasis: continuous auto-research / self-improving / self-learning is only half the story — the other half is **passing prior learnings forward into early stages** so the system knows what's been successful previously, especially at PRD time and task synthesis.
**Inspiration:** `knowledge/saraev-autoresearch-cold-email.md` (rolling `resources.md` as accumulating knowledge file — direct architectural pattern); `knowledge/saraev-autoresearch-self-improving-skills.md` (autoresearch loop with binary eval criteria — the conceptual basis for /optimize feeding lessons forward); `knowledge/saraev-claude-code-advanced-course.md` (4-meaning CLAUDE.md model: failure-success log); `knowledge/karpathy-no-priors-loopy-era.md` (memory beyond context-window compaction).
**Scope:** Anchor the in-flight `lessons.json` self-learning registry as a formal feature with **a closed read-write loop across the pipeline**, not just a write log.

**Schema:** each lesson entry carries `id`, `created`, `tags[]`, `surface[]` (e.g. `auth`, `migration`, `ui`, `external-api`, `prd-shape`, `task-decomposition`), `outcome` (`worked` | `failed` | `mixed`), `context` (1-2 sentences — what was being attempted), `lesson` (the load-bearing claim — what to do or avoid), `source` (feature ID + retro link), `confidence` (`single-occurrence` | `repeated` | `validated`), `superseded_by` (optional later lesson ID).

**Write path (already in scope; keep):**
1. `specflow:test` Phase D feedback-mode appends edge-case-discovered + brand-coverage lessons.
2. `specflow:complete` retro appends what-worked + what-failed lessons (the bulk of writes).
3. `specflow:insights` (already shipped) clusters lessons by tag + outcome and promotes ≥3-occurrence patterns to `admin/rules/guidelines.md`.

**Read path (the self-learning loop the user emphasised):**
1. `specflow:prd` Phase A.4 — after codebase context (A.3) and before requirements drafting (B), compute the feature's tag profile from the interview + design folder + linked Linear issues. Query lessons.json by tag-match + recency + confidence weighting. Surface the top N (default 5, configurable via `config.json.prd.maxLessonsSurfaced`) inline in the interview as a **"What we've learned before that applies here"** section so the user sees what's being pulled and can correct/dismiss any non-applicable ones. Lessons that survive sign-off propagate into Phase B as constraints.
2. `specflow:task` Phase A.2 — when synthesising tasks, re-query lessons.json with the now-finalised PRD's tag profile. Surface in the coverage-matrix output: each task that touches a tagged surface gets a `prior-lessons: [id1, id2]` field linking the lessons that shaped it. Influences task shape (e.g. "for migration tasks, prior runs flagged data-shape mismatch — task synthesis must include a pre-migration validation step") and ordering (e.g. "for refactors of legacy modules, seam-cut task must precede behavioural change task").
3. Cross-task review (022, Sprint 2) consults lessons too — its "better arrangement" lens gets the same tag-based query so it can suggest re-orderings prior runs proved valuable.
4. `specflow:develop` does NOT query lessons directly — by design. Lessons must influence the *plan* upstream (PRD + tasks); develop executes the plan. This keeps the develop stage Claude-Code-native and avoids cross-context contamination during code execution.

**Retrieval contract:**
- Query input: `{tags: string[], surfaces: string[], outcome_filter?, recency_weeks?: int, confidence_min?: 'single' | 'repeated' | 'validated'}`.
- Default: `recency_weeks: 26` (≈2 quarters), `confidence_min: 'single'`, no outcome filter (both worked + failed are useful).
- Scoring: tag-overlap × confidence-weight × recency-decay. Top N returned with full body for prompt inclusion.
- Superseded lessons are filtered out unless explicitly requested.

**Files:** `plugins/specflow/templates/admin/lessons.json` (schema + seed; already present, untracked); `plugins/specflow/skills/setup/SKILL.md` (Phase 8.x — seed empty registry + `config.json.prd.maxLessonsSurfaced`); `plugins/specflow/skills/prd/SKILL.md` (Phase A.4 — query + surface inline in interview); `plugins/specflow/skills/task/SKILL.md` (Phase A.2 — re-query + `prior-lessons` field on tasks); `plugins/specflow/skills/test/SKILL.md` (Phase D — append); `plugins/specflow/skills/complete/SKILL.md` (retro — append); `plugins/specflow/skills/insights/SKILL.md` (already shipped — clarify it consumes lessons.json, not just task-history.json).

**Done when:** setup seeds `lessons.json`; prd + task + test + complete skills all hit the registry (read + write per the contract above); worked example demonstrates a full loop — a lesson is appended at retro of feature N, then surfaces inline at PRD time of feature N+1 and influences a task at task synthesis; the surfaced lessons are visible to the user (not just hidden context injection).

### 019 — task-manifest
**Drive:** chat feedback (#9), audit ✗. Expanded by the user during review: not just a develop-time write log — a **per-task lifecycle workspace** where every agent that touches the task reads prior context before contributing, then appends in a standardised format. "Each task can have its own space within the system for the LLMs or the agents to be able to track their progress."
**Inspiration:** `knowledge/medin-archon-livestream.md` (artifact directory passed between nodes as cross-skill bus; "continue vs fresh" session toggle per node); `knowledge/medin-archon-harness-builder.md` (workflows as YAML node graphs with deterministic + prompt nodes); `knowledge/medin-parallel-agentic-playbook.md` (issue-vs-PR drift comparison; reviewer-never-sees-writer's-chat); `knowledge/medin-piv-loop.md` (planning artefact loaded into a fresh implementation session — the manifest IS that artefact).
**Scope:** A standardised **per-task manifest** at `debate-log/tasks/T-NN-manifest.md` that accumulates entries from the moment the task is born (`specflow:task` synthesis) through to validation (`specflow:complete` retro). Replaces the original narrow develop-only framing.

**Read-first contract:** every agent invoked against a task **must read the manifest** before contributing — this is how agents see what others have already proposed, found, or decided. Per 027-reviewer-context-isolation, the manifest is part of the artefact-context handed to fresh-context reviewers; the writer's chat is not.

**Standardised entry format** (one block per entry, append-only):
```
---
timestamp: 2026-05-07T12:34:56Z
agent_id: {role-name or principle name, e.g. "task-synthesiser", "goal-driven-reviewer", "implementer-yellow", "edge-case-reviewer", "verifier"}
phase: {task-creation | task-review (Gate 3) | develop-red | develop-green | develop-refactor | test | gate-4 | gate-5 | complete}
event_type: {proposal | finding | decision | iteration-result | escalation | sign-off}
input_ref: {paths to artefacts the agent read — PRD, prior manifest entries, design folder, lessons.json query result}
output_ref: {paths/links to what the agent produced — code diff, test result, finding json}
body: |
  {the actual content — proposal text, finding details, decision rationale, iteration outcome}
outcome: {open | accepted | rejected | deferred-to-misc | superseded-by-T-NN-entry-M}
---
```

**Lifecycle phases captured:**
1. **task-creation** — synthesiser appends the initial task spec + the lessons.json query result that shaped it (per 018-lessons-registry).
2. **task-review (Gate 3)** — Gate 3 reviewers (per-task lens AND cross-task lens from 022) append findings; orchestrator appends the closing decision.
3. **develop-red / develop-green / develop-refactor** — per Pocock's framing from 017-tdd-discipline. Each sub-phase logs the test written (Red), the minimal code (Green), the structural improvement (Refactor).
4. **gate-4 (plan-vs-PRD)** + **gate-5 (code-vs-plan)** — principle reviewers + edge-case-reviewer (028) append findings with `recommendation` + `reasoning`; orchestrator appends accept/reject/defer per finding.
5. **test** — `specflow:test` Phase D appends AC pass/fail + brand-consistency lens findings (per 023).
6. **complete** — `specflow:complete` retro appends the final sign-off + the lessons.json entries this task generated.

**Cross-feature integration:**
- **017 (TDD)** — Red/Green/Refactor markers land as `phase` values on develop entries.
- **018 (lessons-registry)** — task-creation entries cite the lessons that shaped the task; complete entries cite the lessons being appended.
- **022 (cross-task review)** — cross-task findings are entries in EVERY affected task's manifest (not a separate doc).
- **027 (reviewer isolation)** — each entry's `agent_id` is the writer/reviewer ID; isolation is auditable from the manifest alone.
- **028 (edge-case reviewer)** — its `recommendation` + `reasoning` payload is the entry body; orchestrator's accept/reject decision is the next entry's `body` referencing it.

**Files:** `plugins/specflow/skills/task/SKILL.md` (open the manifest at synthesis); `plugins/specflow/skills/develop/SKILL.md` (append per Red/Green/Refactor and per gate); `plugins/specflow/skills/test/SKILL.md` (append at Phase D); `plugins/specflow/skills/complete/SKILL.md` (close the manifest at retro); `plugins/specflow/templates/admin/task-manifest-schema.md` (new doctrine doc — entry format + read-first contract).
**Done when:** every task in a fresh feature run has a manifest opened at synthesis and closed at completion; every agent invoked against the task reads it before contributing; entries follow the standardised format; worked example shows a single task's manifest accumulating entries through all six lifecycle phases (synthesis → Gate 3 review → Red/Green/Refactor → Gate 4/5 → test → complete) with cross-references to 017, 018, 022, 027, 028 visible in the entries.

### 023 — test-brand-consistency
**Drive:** features.md item 4.
**Inspiration:** `knowledge/pocock-real-feature-build.md` (edge cases that grilling cannot catch — surface only in QA); `knowledge/pocock-ai-coding-real-engineers.md` (fresh-context review pass).
**Scope:** Extend `specflow:test` to ask brand/consistency/coverage-gap questions beyond binary AC pass/fail (font correctness, on-brand, what-might-have-been-missed).
**Files:** `plugins/specflow/skills/test/SKILL.md` (question set extension).
**Done when:** test plan output includes a brand-consistency lens section; worked example shows brand findings logged separately from AC findings.

### 027 — reviewer-context-isolation
**Drive:** chat feedback — "Never validate your own work. The reviewer should never see the writer's chat. Must be in a fresh context."
**Inspiration:** `knowledge/medin-parallel-agentic-playbook.md` (DIRECT — "reviewer should never see the writer's chat" is Medin's exact framing); `knowledge/pocock-ai-coding-real-engineers.md` (fresh-context review — avoid reviewing in the dumb zone after ~100K token cliff); `knowledge/medin-wisc-framework.md` (sub-agents for research-only isolation; preserve the writer's full context for develop/complete).
**Scope:** Hard rule across every multi-agent debate gate (Gates 2, 3, 4, 5): every reviewer runs in a **fresh context** with **zero exposure** to the writer's chat history. Reviewer input is the artefact under review (PRD, task list, plan, code) plus its declared context inputs (interview, prior gate manifest) — never the writer's reasoning trace. Same agent identity may not write AND validate the same artefact within a feature. Implementation: each reviewer invocation is a fresh subagent spawn; the orchestrator passes only the explicitly-listed input files. Doctrine doc spells out the contract; gate manifest carries a `writer_id` + `reviewer_ids` block to make violations auditable.
**Files:** `plugins/specflow/templates/admin/reviewer-isolation.md` (new doctrine doc); `plugins/specflow/templates/orchestrator-pattern.md` (extension); skill bodies for `prd`, `task`, `develop` (gate-fire syntax updates); manifest schema (`writer_id` / `reviewer_ids` fields).
**Done when:** every gate manifest in a fresh feature run carries `writer_id` + `reviewer_ids` and they don't intersect; doctrine doc is the citation point for "why fresh context"; worked example demonstrates the isolation contract.

### 028 — edge-case-reviewer
**Drive:** chat feedback — "Edge case reviewer but must submit their recommendation and reasoning to the main agent for acceptance." Sharpened during review: the edge-case reviewer is the **counter-balance to Goal-Driven**. The user's framing: "we don't want to be too hyper-focused on what the goal is, because then we can forget about all the other parts and all the other elements that may come into play. Have something that just specifically is not worried about the goal but worried about the edge cases that we might *inherit*."
**Inspiration:** `knowledge/pocock-real-feature-build.md` (edge cases surfaced only in QA — non-git-repo rollback example; argues for an explicit QA-finds-new-work loop, not just `/complete`); `knowledge/karpathy-vibe-to-agentic.md` (council-of-LLM-judges pattern to make soft domains verifiable); `knowledge/pocock-deslop-codebase-deep-modules.md` (seams + adapters — interaction surfaces are where edge cases live).
**Scope:** New reviewer category dedicated to surfacing edge cases the writer didn't consider. Lives at Gate 4 (plan-vs-PRD) and Gate 5 (code-vs-plan). Joins the principle-reviewer roster (Simplicity, Surgical, Think-Before-Coding, Goal-Driven, **Edge-Case**). Runs in fresh context per 027.

**Lens — deliberately NOT goal-aware:**
The edge-case reviewer is the only principle reviewer whose primary pass **does not consult the PRD goal statement** to decide what's relevant. Its job is to look at the artefact (plan or code) and ask: what does this work *inherit* from the codebase, the runtime, the user environment, that the goal-focused reviewers won't catch?

Specifically, it asks (in this order, none of which mention the goal):
1. **Collateral surface** — what files / modules / database tables / external services does this touch beyond the ones the PRD names?
2. **Failure modes** — what happens at the boundary (empty input, max input, partial input, malformed input, concurrent input)? What happens when a dependency (file, network, lock, env var) is missing or stale?
3. **Inheritance** — what does this change inherit from existing patterns in the codebase that the writer may have copied without noticing? (e.g. inherited error-handling that swallows failures; inherited assumptions about user permissions; inherited timing assumptions.)
4. **Interaction** — what other features / skills / agents in the system touch the same surface, and could this change break them silently? (Looks at `task-history.json`, `lessons.json`, and the per-feature debate logs to identify recent neighbouring work.)
5. **State / environment** — what assumptions about state does this make? (Empty repo? Existing config? User logged in? Network available? Specific OS / shell?)

**Why deliberately not goal-aware:** Goal-Driven's reverse-traceability lens is necessary but creates a blindspot — it certifies "every R/AC has a task / every plan step traces to an R" but cannot see what's *missing* from the R/AC list. The edge-case reviewer's job is exactly that gap. If it consults the goal, it collapses into Goal-Driven and the blindspot returns.

**Output shape:** each finding carries `recommendation:` + `reasoning:` fields. **Findings are advisory, not auto-applied** — the Orchestrator decides accept / reject / defer-to-misc per finding. Decisions land as the next entry in the per-task manifest (019).

**Files:** `plugins/specflow/admin/agents/standard/principles/edge-case-reviewer.md` (new — agent body explicitly forbids consulting `goal:` fields in its primary pass); manifest schema (`recommendation` + `reasoning` fields, accept/reject/defer outcomes); Gate 4 + Gate 5 skill bodies in `develop/SKILL.md`.
**Done when:** edge-case-reviewer fires at Gate 4 and Gate 5 in a fresh feature run; agent body documents the not-goal-aware contract with the five-question lens; worked example demonstrates an edge-case finding that Goal-Driven could not have produced (because the missing concern wasn't in any R or AC); orchestrator's accept/reject decision is the next manifest entry.

---

## Sprint 4 — planner & creative integrations (target v2.7.0)

Highest novelty, lowest coupling — they benefit from every prior sprint's output.

### 020 — sprint-skill *(with 024-sprint-worktree absorbed)*
**Drive:** parked memory + chat feedback (#5) + features.md item 3 + restructure during review: develop is the entry point, but the sprint logic stays a **separate skill (or agent) chained from develop** — keep skills lightweight (≤500 lines), chain don't absorb (per `feedback_skill_size_ceiling`).
**Inspiration:** `knowledge/medin-piv-loop.md` (Plan/Implement/Validate loop with fresh sessions per phase — planning artefact in one session, implementation in a fresh session); `knowledge/medin-parallel-agentic-playbook.md` (git work-trees for isolation); `knowledge/saraev-skill-systems.md` (orchestrator skill with five required understanding points); `knowledge/saraev-skill-chaining.md` (skill chaining as the antidote to mega-skills — the explicit case for keeping develop and sprint separate).
**Scope:** New lightweight `specflow:sprint` skill (or `sprint-planner` agent — open during build) **invoked by `specflow:develop` Phase A.x**, not absorbed into it. When the user runs `/specflow:develop {feature}`, develop calls sprint as a sub-step.

**Sprint-skill responsibilities (single-purpose, ≤500 lines):**
1. **Pull Linear project** mapped to this feature (Linear project ↔ specflow feature is 1:1; mapping recorded in feature manifest at setup time or first-run).
2. **Read all open issues** in the project + reconcile with local `tasks.md`. Drift between Linear and local is flagged for the developer; Linear is treated as the issue source of truth, local `tasks.md` carries the synthesis context (lessons, design-decision links, sprint-bucket flags).
3. **Filter to in-scope batch** using the `sprint-bucket: N` flags from 025 + the configurable cap (`config.json.develop.maxIssuesPerSprint`, default 5).
4. **Synthesise sprint plan** for this session: ordering with dependencies respected, parallelism opportunities identified, **per-stage agent-team assignments per 026** (Plan / Build / Test / Iterate / Validate teams resolved).
5. **Present sprint-plan gate** — visual plan to the developer for approve / adjust / defer. Adjustments include: move issues in/out of the sprint, change team assignments, change order, defer to next session.
6. **On approval — create git work-tree(s)** isolated for this sprint (024-sprint-worktree absorbed). One work-tree per sprint by default; opt-in per-issue work-trees if the developer flags parallel-execution at the gate.
7. **Return approved sprint plan** to develop, which then proceeds with lane-triage and execution.

**Develop's slim addition (≤30 lines, no absorption):** Phase A.x calls `specflow:sprint`, awaits the approved plan, then iterates Phase B-F across the approved batch instead of one issue.

**Open design (deferred to feature build):** skill vs persistent agent — skill recommended (one-shot synthesis, returns artefact, no long-running role). Agent only if a recurring "sprint-planner voice" emerges across sessions worth preserving. Cross-feature sprint planning (multi-Linear-project batches) stays parked under the existing parked-memory entry — out of scope for 020.

**Files:** new `plugins/specflow/skills/sprint/SKILL.md` (the skill body — keep ≤500 lines); `plugins/specflow/admin/agents/specialised/sprint-planner.md` (optional companion agent if pattern emerges); `plugins/specflow/skills/develop/SKILL.md` (slim Phase A.x addition — call sprint, await approved plan); `config.json.develop.maxIssuesPerSprint` block; feature manifest extension for Linear project mapping.

**Done when:** sprint skill produces a valid plan against a feature's Linear project; sprint-plan gate fires for developer approval; work-tree creation is idempotent; develop calls sprint without absorbing its logic (line-count audit confirms develop didn't grow past its current footprint by more than ~30 lines); per-stage team assignments (026) are filled in; worked example shows a multi-issue session with the sprint-plan gate, approval, and execution against the work-tree.

### 026 — agent-teams-per-stage
**Drive:** chat feedback — "Add the option to have designated Agent Teams that are called upon at their respective stages... Plan, Build, Test (code quality, result, simplicity, security), Iterate, Validate."
**Inspiration:** `knowledge/medin-archon-harness-builder.md` (harness as the layer above the coding agent; workflows as YAML node graphs with deterministic + prompt nodes; per-node model selection — Haiku for classification, Sonnet for review, Opus for implementation); `knowledge/medin-archon-livestream.md` (hybrid secret: take decisions away from the agent where determinism matters; "continue vs fresh" session toggle per node); `knowledge/saraev-claude-code-advanced-course.md` (parent + researcher + QA delegation pattern); `knowledge/saraev-skill-systems.md` (orchestrator + N composable component skills).
**Scope:** Promote the canonical pipeline shape — **Plan → Build → Test → Iterate → Validate** — to first-class doctrine. Each stage has a designated agent team configurable per project via `config.json.teams.{stage}`. Default rosters:
- **Plan** — sprint-planner (020) + goal-driven-reviewer + devils-advocate.
- **Build** — implementer + simplicity-reviewer + surgical-reviewer.
- **Test** — code-quality-reviewer + result-reviewer + simplicity-reviewer + security-reviewer (the four lenses called out in the user's brief).
- **Iterate** — verifier + edge-case-reviewer (028).
- **Validate** — orchestrator + (when codex available) codex-reviewer + (when team-review fires) `agent-teams:team-review`.

Sprint planner (020) emits per-task team assignments; develop / test skills consume them at their respective gates. Cross-cuts with 027-reviewer-context-isolation (every team member runs in fresh context).
**Files:** `plugins/specflow/templates/admin/stage-teams.md` (new doctrine doc); `plugins/specflow/skills/sprint/SKILL.md` (team-assignment output); `plugins/specflow/skills/develop/SKILL.md` + `plugins/specflow/skills/test/SKILL.md` (team-consumption); `config.json.teams.{stage}` schema.
**Done when:** doctrine doc spells out default rosters + override path; sprint plan output includes team assignments per task per stage; develop/test consume the assignments without falling back to defaults silently; worked example shows a multi-task plan with explicit per-stage teams.

### 021 — design-image-gen *(removed 2026-05-07)*
**Status:** Removed during review. No clear use case identified; skipped to keep Sprint 4 focused on 020 + 026. ID 021 stays allocated (no-reuse rule). Re-open if a concrete need surfaces in a future feature.

---

## Parked

These are recorded for future consideration. Not on the current sprint roadmap. When promoted, they slot into the next-available sprint and gain a feature ID.

### HTML annotation feature
**Drive:** direct client feedback — clients want to comment on delivered HTML mockups and return feedback in-band.
**Inspiration:** chat feedback only (no transcript trace).
**Recommended approach:** self-contained annotation HTML — inject a JS overlay into the mockup so clients click elements, leave comments, and export a single JSON/markdown file. Plugin then ingests into Linear/PRD revisions. Preserves the one-file deliverable.
**Fallbacks (if export-and-send is too high-friction):** Cloudflare Pages/Vercel + Workers/KV; or GitHub issues API.
**Rejected:** third-party tools (Markup.io, Pastel, BugHerd) — per-seat cost + breaks the self-contained deliverable.

### 006 — feature-model (working design captured 2026-05-06)
**Drive:** maximum context preservation across sessions; agents/humans starting cold should find everything they need.
**Inspiration:** `knowledge/medin-piv-loop.md` (planning artefact loaded into a fresh implementation session); `knowledge/saraev-skill-systems.md` (orchestrator's five required understanding points); `knowledge/saraev-claude-code-advanced-course.md` (4-meaning CLAUDE.md model — compression / preferences / capability declaration / failure-success log).
**Status:** design notes only; not yet a feature. When implementation starts, this gets converted into a proper feature directory under `features/006-feature-model/` using the very model it describes.

**Per-feature directory layout:**

| File | Purpose | Format |
|------|---------|--------|
| `manifest.json` | Structured metadata. Single source of truth. | JSON |
| `transcript.json` | Verbatim record of all conversation across feature lifecycle. Sessioned. | JSON |
| `interview.md` | Curated interview only (existing convention). | Markdown |
| `prd.md` | Problem statement + PRD body. Opens by restating the goal. | Markdown |
| `tasks.md` | Task breakdown. | Markdown |
| `debate-log/` | Existing decision/debate record. | Folder |

**`manifest.json` shape:**
```json
{
  "schema_version": "1",
  "id": "006",
  "name": "feature-model",
  "goal": "One paragraph max. What we are trying to achieve.",
  "references": [
    { "url": "https://...", "note": "context", "tags": ["api"] }
  ],
  "status": "active"
}
```
- `goal`: paragraph max. Narrative context — provides the lens for all downstream work.
- `references`: flat array; freeform `tags` (e.g. `api`, `inspiration`, `docs`, `slack`, `screenshot`).
- `status`: `active | blocked | shipped | parked`. Sub-states inferred from which files exist.

**`transcript.json` shape:** sessioned to keep size bounded.
```json
{
  "schema_version": "1",
  "sessions": [
    { "session_id": "...", "started_at": "...", "ended_at": "...",
      "turns": [{ "role": "user", "text": "..." }] }
  ]
}
```

**Goal mutability:** goal can sharpen as discovery deepens. Manifest holds the *current* goal as a plain string. When the goal changes, append a one-liner to `prd.md` explaining the sharpening.

**System-level manifest** at `plugins/specflow/examples/docs/specflow/manifest.json` (next to `features/`):
```json
{
  "schema_version": "1",
  "active_feature": "006",
  "features": [
    { "id": "001", "name": "design-skill", "status": "shipped", "path": "features/001-design-skill" }
  ]
}
```
No `next_id` counter — derive from `max(features.id) + 1` to avoid concurrent-write contention.

**Backfill plan for 001–005:** write full `manifest.json` for each, extracting `goal` from existing interview/PRD docs. Stub `transcript.json` with `{ "schema_version": "1", "legacy": true, "note": "predates verbatim transcript capture", "see_also": ["interview.md", "debate-log/"] }`.

**Adversarial review (Codex) extension:**
- **PRD gate:** Codex added to standard reviewer lineup. Cross-provider lens, not fungible with same-model reviewers. Sees PRD draft + interview log.
- **Task gate:** Codex reviews the **whole task list per round**, not per-task. Catches cross-task failure modes (missing tasks, wrong ordering, AC-without-task). Sees PRD + task list + checkpoints.
- **Develop gate:** already integrated.
- **Stopping rule:** up to 3 rounds. Stop when Codex returns no new substantive findings OR round 3 reached. **Critical:** if Claude rejects all of Codex's feedback in a round AND Codex re-fires the same findings → escalate to human review.
- **Failure mode:** if Codex unavailable, gate proceeds with remaining reviewers (existing degraded-coverage sentinel pattern).
- **Auto-promotion:** existing develop-gate Codex-only auto-promotion stays. Skip at PRD/task gates (those findings are feature-specific, not code-layer rules).
- **No new artefact formats** — findings land in existing `debate-log/{gate}/findings/round-N/codex-reviewer.json` path.

**Checkpoints (HITL pacing primitive):**
- Created at **task gate**, alongside the task list — not at develop. Boundaries reveal hidden cross-task dependencies.
- Each checkpoint = named group of tasks with a one-line **deliverable**, an `Anchor:` line tracing to a PRD goal, member tasks, implicit ordering (no checkpoint depends on a later one), sized for **one human's review tractability** in a single sitting.
- Location: headers in `tasks.md`; mirrored in `manifest.json.checkpoints`.
- `develop-gate5` fires **once per checkpoint completion**, not whole-feature. Human verifies + signs off; no auto-flow.
- Rollback unit: the whole checkpoint.
- Mid-feature replanning: route through explicit `specflow:scope-change --checkpoint`. Silent edits to `tasks.md` mid-run forbidden.
- Escape hatch: small features (≤3 tasks) may use a single checkpoint or skip the primitive.
- Terminology note: "checkpoint" deliberately, not "sprint" — avoids Scrum baggage and disambiguates from Linear cycles or PRD-level milestones.

**Open / deferred:**
- Plugin-wide reference links — skipped for now (YAGNI).
- Whether goal-sharpening rationale lives in `prd.md` (current decision) or a new `decisions.md` (rejected for now).
- Codex-vs-same-model disagreement weighting — log empirically across several features before baking in.
- Lane interaction with checkpoints (Red/Green HITL gating) — moot under HITL-pacing principle: every checkpoint is a human gate regardless of lane mix.

---

## Feature ID allocation

Next free ID: **030**. Allocation rule: `max(existing IDs) + 1`. IDs are not reused, even when a feature is decided-not-built (e.g. 009 stays allocated).

---

## Reference reading

Each transcript in `v2/docs/knowledge/` is captured as a Markdown brief in the project's house format (Title → Executive Summary → main sections → Key Exact Extracts with timestamps). Verbatim quotes are preserved inside the briefs. Citations on each feature above point to the relevant brief.

### Karpathy
- `knowledge/karpathy-no-priors-loopy-era.md` — auto-research as recursive self-improvement; macro-actions over a repo; skills as scripted curricula; memory beyond context-window compaction.
- `knowledge/karpathy-vibe-to-agentic.md` — Software 3.0; spec-first co-authoring; verifiability as capability map; council-of-LLM-judges to make soft domains verifiable.

### Medin
- `knowledge/medin-archon-harness-builder.md` — harness engineering as the layer above context engineering; workflows as YAML node graphs (deterministic + prompt nodes); per-node model selection.
- `knowledge/medin-archon-livestream.md` — hybrid secret (take decisions away from the agent where determinism matters); continue-vs-fresh per node; artifact directory between nodes.
- `knowledge/medin-parallel-agentic-playbook.md` — issue-as-spec / PR-as-validation; git work-trees for isolation; **"reviewer should never see the writer's chat"**; self-healing layer.
- `knowledge/medin-piv-loop.md` — Plan / Implement / Validate loop with fresh sessions per phase; two-layer planning; system-evolution outer loop.
- `knowledge/medin-wisc-framework.md` — context rot is the dominant failure mode; just-in-time context loading; sub-agents for research-only isolation.

### Pocock
- `knowledge/pocock-ai-coding-real-engineers.md` — smart zone vs dumb zone (~100K token cliff); grill-me skill; Red/Green/Refactor TDD; vertical slices; AFK Ralph; fresh-context review.
- `knowledge/pocock-deslop-codebase-deep-modules.md` — deep vs shallow modules; locality + leverage as the only two targets; tactical (sergeant) vs strategic (general) framing.
- `knowledge/pocock-real-feature-build.md` — full grill → PRD → issues → AFK → QA pipeline demo; ubiquitous-language updates inline; issue-sizing heuristic; edge cases surfaced only in QA.
- `knowledge/pocock-software-fundamentals-matter-more.md` — anti-specs-to-code thesis; canonical Red/Green/Refactor definition; deep modules as the prerequisite for testable codebases; "design the interface, delegate the implementation."

### Saraev
- `knowledge/saraev-autoresearch-cold-email.md` — autoresearch on a business metric; rolling `resources.md` as accumulating knowledge file; cron-driven harvest → challenger → deploy.
- `knowledge/saraev-autoresearch-self-improving-skills.md` — autoresearch loop on skill outputs with binary eval criteria; meta-skill that auto-researches every skill in repo (the conceptual ancestor of `/optimize`).
- `knowledge/saraev-claude-code-advanced-course.md` — 4-meaning CLAUDE.md model; parent + researcher + QA delegation; stochastic-consensus pattern; workspace hygiene.
- `knowledge/saraev-skill-chaining.md` — three layers of context efficiency (`context: fork`, file-handoff, `!` shell-substitution); skill vs sub-skill vs agent; 85% context-burn reduction.
- `knowledge/saraev-skill-systems.md` — orchestrator skill + N composable component skills; sequential workflow orchestration; component reuse across multiple skill systems.
