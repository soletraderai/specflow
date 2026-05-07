# PRD interview — features/017-tdd-discipline

## Goal confirmation

The user invoked `specflow:prd 017-tdd-discipline` with the brief: "Development needs to enforce TDD; tests-first must precede code execution. Adopt Matt Pocock's Red, Green, Refactor framing as the canonical TDD shape across `specflow:develop`."

Confirmed: the goal is to reframe `specflow:develop` Phases D / E / F as Pocock's **Red → Green → Refactor**, make tests-first mandatory for the Yellow lane and configurable for the Green lane, surface explicit `red` / `green` / `refactor` markers in the per-task manifest, and extend `specflow:test` so the Red phase can write a failing test plan artefact without executing. Out-of-scope-at-goal-level: changing test runners; introducing a new test format; changing the lane-precedence rules.

## Original request

> "Development needs to implore TDD; we must pass off work to another agent to review externally then provide feedback." Plus: "TDD should inherit Matt Pocock's Red, Green approach." — chat feedback. Pocock's framing is the canonical TDD shape across the specflow knowledge base (`pocock-software-fundamentals-matter-more.md` and `pocock-ai-coding-real-engineers.md`).

## Codebase context (pre-grilling)

- `plugins/specflow/skills/develop/SKILL.md` Phases D (lane execution, lines 397-444), E (Gate 5 code-vs-plan, lines 446-533), F (Verifier + PR, lines 537-633) currently express implementation as "execute the change set then review the diff" — with no explicit failing-test-first discipline. The 029 doctrine landed in v2.5.0 added Phase A.6 (context-budget pre-flight) and a single no-mid-task-compaction reminder at Phase D entry (line 401).
- `plugins/specflow/skills/test/SKILL.md` already has `--plan-only` mode (lines 39, 51, 214) that generates the test plan without executing. The slot exists; this feature anchors it as the Red-phase artefact contract used by develop.
- `plugins/specflow/CORE_PRINCIPLES.md` has four principles (Think-Before-Coding, Simplicity, Surgical, Goal-Driven) but no TDD section. Adding a TDD section keeps it tight and bounded.
- `knowledge/pocock-software-fundamentals-matter-more.md` § Failure Mode 3 carries the canonical Pocock quote: *"TDD forces the LLM to really take small steps. You create a test first. You make that test pass and then you refactor the code to make it nicer and consider the design."* This is the verbatim source.
- `knowledge/pocock-ai-coding-real-engineers.md` § Phase 5 names the four-step shape (write failing test → confirm fails for the right reason → make it pass → refactor) and the AI-cheating failure mode (test written *after* implementation) the rule prevents.
- `knowledge/pocock-real-feature-build.md` § QA carries the AFK Ralph + edge-cases-surfaced-only-in-QA stance — the test-plan artefact catches what grilling missed.
- 029-single-context-task locked the rule that Red → Green → Refactor for one task runs in one agent context window. 017 is the implementation contract that gives the single window its three internal sub-phases.

## Round 1 — what does "tests-first" mean across the lanes

**Question.** Yellow lane has paired-HITL execution; Green lane has batched parallel execution. Is tests-first mandatory for both, or lane-dependent?

**Answer.** Yellow lane: mandatory tests-first; the Yellow paired-HITL boundary already accepts the latency cost of an extra step, and Yellow tasks have weak verifiability axes that benefit most from binary-pass criteria written before code. Green lane: configurable via `config.json.develop.tddRequired` (default `true`). Projects with strong CI signal and binary-AC-heavy task lists may opt out for Green tasks where the test plan would be redundant with the AC itself; projects with looser ACs keep it on. Red lane is human-led — TDD is the human's choice; the skill does not force it. The configurable knob applies only to Green; Yellow and Red are fixed.

## Round 2 — what is the artefact contract for the Red phase

**Question.** What does the Red phase produce, and where does it live?

**Answer.** The Red phase fires `specflow:test {NNN-slug} --plan-only --task T{N}` against the task. The flag re-derives the test plan (Phase B of test) without executing (skips Phase C), writing the per-task plan section into `{NNN-slug}-test.md`. The plan must include at least one failing test case for the task's primary AC and must mark the case as `Status: red (failing)` until the Green sub-phase confirms it passes. The plan artefact is the Red sub-phase's deliverable; Phase E (now Green) cannot enter without the artefact. Yellow lane: blocking — develop refuses to enter Green without the Red artefact. Green lane with `tddRequired: false`: the Red artefact may be skipped, but the manifest records `red: skipped (config)` so the omission is auditable.

## Round 3 — what does "Refactor" mean in a single context window

**Question.** Pocock's Refactor step in a human-IDE workflow is "make the code nicer, consider the design." Inside a single agent context window (per 029), what bounds the Refactor sub-phase?

**Answer.** Refactor is the structural improvement pass with the Green test as guard. Bounded by: (a) no new behaviour — every test that passed at Green must still pass; (b) no new files — Refactor only modifies files Green touched; (c) no scope creep — the simplicity-first and surgical-changes principles bind the same as in Green. If Refactor wants to add a file or change scope, it routes to `specflow:scope-change`, not silent expansion. The manifest entry `refactor` carries the diff delta from Green-end to Refactor-end so the structural change is visible. Refactor is optional for trivial tasks (one-line changes that don't benefit from a structural pass); the manifest records `refactor: skipped (trivial)` when omitted.

## Round 4 — manifest markers and Gate 5 visibility

**Question.** Should Gate 5 (code-vs-plan) review the Red, Green, and Refactor diffs separately, or as a single combined diff?

**Answer.** Single combined diff for Gate 5. The Red/Green/Refactor markers are *internal* to the task's single context window — they help the agent (and the retro reader) see the cycle structure, but Gate 5's reviewers see the assembled change set. The per-task manifest (019, future) carries the three markers (`red`, `green`, `refactor`) with timestamps and outcome notes, so retro consumers can audit whether the cycle held. Gate 5 reviewers do NOT replay the Red→Green transition; they review the final shipped diff against the plan. This keeps Gate 5 lean (per 027 fresh-context contract) and avoids re-running the cycle in dumb-zone review territory.

## Sign-off

User confirmed: Phases D/E/F reframed as Red → Green → Refactor; Yellow mandatory tests-first; Green configurable via `config.json.develop.tddRequired`; manifest carries three markers per task; `specflow:test --plan-only` is the Red artefact contract; Refactor is bounded to no-new-behaviour; Gate 5 reviews the combined diff. Proceed to PRD synthesis.

## Topics not discussed

- Whether `tddRequired: false` should require a per-task override flag for the Green tasks that opt out. Out of scope — the config is project-wide; per-task overrides multiply surface for marginal flexibility.
- Whether to surface the Red artefact in the PR description. Considered, deferred — Gate 5's combined-diff stance keeps the PR description clean; the Red artefact lives in `{NNN-slug}-test.md` and is linked from the manifest.
- Whether Refactor's "no new files" rule should hard-block or warn-and-continue. Decided: hard-block — Refactor adding a new file is by definition new behaviour, not structural improvement; route to `specflow:scope-change`.
