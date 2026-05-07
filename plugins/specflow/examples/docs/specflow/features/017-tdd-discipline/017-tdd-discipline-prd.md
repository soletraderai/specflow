---
feature: 017-tdd-discipline
status: draft
created: 2026-05-07
interview: ./017-tdd-discipline-interview.md
---

# TDD Discipline

## Vision

`specflow:develop` adopts Pocock's Red → Green → Refactor cycle as the canonical implementation pattern inside whatever lane (D.1 Green / D.2 Yellow / D.3 Red) a task lands in. The cycle is *internal* to a task's single context window (per 029): the Red sub-step writes a failing test before any code lands; the Green sub-step makes it pass with the simplest change; the Refactor sub-step is bounded structural improvement under the green test as guard. The cycle's contract lives in `plugins/specflow/templates/admin/tdd-discipline.md` (chain-don't-absorb, mirroring 029's doctrine doc); `develop/SKILL.md` carries one-line citations per lane sub-phase. The cycle is mandatory on Yellow, configurable on Green via `config.develop.tddRequired` (default `true`), and skipped on Red lane (PROTECTED_PATHS_REQUIRE_RED_LANE — human-led). Cycle markers (red / green / refactor) write to per-task scratch (`admin/scratch/{NNN-slug}-develop/`) so retro readers can audit; Phase E (Gate 5 code-vs-plan) and Phase F (Verifier + PR) remain unchanged in scope and do not see the markers.

**Blast radius.** Five files + one doctrine doc + one scratch path: `develop/SKILL.md` (sub-step citations), `test/SKILL.md` (`--plan-only --task` flag extension), `CORE_PRINCIPLES.md` (one-paragraph TDD principle citing the doctrine doc), `admin/config.json` schema (one new field), the new `templates/admin/tdd-discipline.md`, and the per-task scratch path. Lane stays Yellow: CORE_PRINCIPLES.md gets one paragraph (not 30 lines), the doctrine doc absorbs the cycle contract, and Red lane bypasses TDD enforcement so PROTECTED_PATHS_REQUIRE_RED_LANE is unaffected.

## Problem

`specflow:develop` Phase D today (`develop/SKILL.md:397-444`) expresses lane execution as "execute the change set then review the diff" — there is no explicit failing-test-first discipline at the task level. AI agents in this position routinely cheat the test-write step by writing the test *after* the implementation, against the implementation's actual behaviour, which produces tests that are tautological with the code (`knowledge/pocock-ai-coding-real-engineers.md` § Phase 5 names this failure mode verbatim). The tests then pass on first run for the wrong reason — they encode what the code does, not what it should do.

`CORE_PRINCIPLES.md` carries Think-Before-Coding, Simplicity, Surgical, and Goal-Driven, but no TDD principle. The 029 doctrine landed the single-context-per-task contract (`templates/admin/single-context-task.md`) but did not specify the cycle structure inside the window. This feature adds the cycle contract as the missing operational principle, hosts it in a doctrine doc mirroring 029's pattern, and binds `specflow:develop` to honour it.

## Goals

- Add Pocock's Red → Green → Refactor cycle as the canonical implementation pattern inside `specflow:develop`'s lane sub-phases (D.1 Green / D.2 Yellow / D.3 Red, where appropriate). The cycle is internal to a task's single context window — it does NOT replace or rename the lane sub-phases.
- Host the cycle contract in `plugins/specflow/templates/admin/tdd-discipline.md` (chain-don't-absorb pattern, mirrors 029's `templates/admin/single-context-task.md`). `develop/SKILL.md` adds one-line citations per lane sub-phase; CORE_PRINCIPLES.md adds a one-paragraph principle entry citing the doctrine.
- Make the cycle mandatory on Yellow (paired-HITL); configurable on Green via `config.develop.tddRequired` (default `true`); skipped on Red lane (human-led, PROTECTED_PATHS_REQUIRE_RED_LANE).
- Surface explicit `red` / `green` / `refactor` markers in per-task scratch (`admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md`, one file per task) so retro readers can audit whether the cycle held. 019-per-task-manifest absorbs from this scratch path.
- Extend `specflow:test` so `--plan-only --task T{N}` writes the per-task plan section into `{NNN-slug}-test.md` and marks the primary AC's test case as `Status: red (failing)` by default. The `--task` co-flag SKIPS Phase B.5 (Codex pass + user prompt) — the per-task slice inherits the feature-level B.5 outcome.
- Bound the Refactor sub-step at the documentation level: no new behaviour, no new files, no scope creep — Refactor wanting to add a file or change scope routes to `specflow:scope-change`. Violations are documented post-hoc in the manifest as `refactor: failed (new-file-attempted)`.
- Pre-implementation test must actually fail: after the Red sub-step writes the test, the targeted test command runs against the pre-implementation state and the failing exit + stderr is captured before Green sub-step entry. This is the Pocock failure-mode prevention.

## Non-goals

- Changing test runners. Vitest stays for unit + integration; Playwright stays for e2e (per `admin/environment.json`). (Goal-level out-of-scope.)
- Introducing a new test format. Existing test conventions (`__tests__/` co-location per `rules/guidelines.md` PREFER_LOCAL_TESTS) are unchanged.
- Changing lane-precedence rules. Red lane is human-led; Yellow lane is paired-HITL; Green lane is batched parallel. The TDD configurable knob applies inside Green only.
- Renaming or renumbering Phase D's lane sub-phases (D.1 Green / D.2 Yellow / D.3 Red, currently at `develop/SKILL.md:403, 424, 430`). The cycle steps (Red / Green / Refactor) are *internal* to a task's execution within whatever lane it landed on — they are NOT phase-numbered headings, they are cycle steps documented in the doctrine doc.
- Per-task overrides for `tddRequired`. The config is project-wide.
- Surfacing the Red artefact in the PR description. The Red artefact lives in `{NNN-slug}-test.md` and is linked from the manifest scratch.
- Splitting the Phase E (Gate 5) review into Red / Green / Refactor sub-reviews. Phase E reviews the final shipped diff; the cycle markers are retro-audit only.
- Adding runtime detection logic for "Refactor created a new file" inside D.3. The bound is documented; violations are recorded post-hoc as a manifest outcome value (R11), not detected mid-cycle.

## Users

- **Power user — operations coordinator.** Indirect: power users benefit from higher test quality.
- **Admin — platform operator.** Reads the per-task manifest scratch during retro to audit whether the cycle held; values the explicit markers as a synthesis trace.
- **Engineer using the develop skill.** Direct user. The Red sub-step forces a failing-test-first commit; the Green sub-step narrows their decision space; the Refactor sub-step has the green test as guardrail.

## Requirements

- **R1.** Create `plugins/specflow/templates/admin/tdd-discipline.md` as the cycle's canonical home. The doctrine doc states: (a) the Red → Green → Refactor cycle is internal to a task's single context window (per 029); (b) Red writes a failing test before any code lands and the failing exit must be captured pre-implementation; (c) Green writes the simplest change that makes the test pass; (d) Refactor is bounded structural improvement (no new behaviour, no new files, no scope creep); (e) markers `red` / `green` / `refactor` carry outcome + ISO-8601 timestamp + one-line note. Pattern mirrors 029's `templates/admin/single-context-task.md`.
  - Trace: interview Round 4 sign-off; DA-2 chain-don't-absorb (mirror 029's pattern).
  - Serves goal: Outcome (one canonical home) + Driving value (skill-size ceiling respected).

- **R2.** `develop/SKILL.md` Phase D body adds three one-line citations to the doctrine doc — one inside D.1 (Green-lane batched execution), one inside D.2 (Yellow-lane paired HITL), and one inside the Phase B.5/B.6 task-execution prelude that fires per task. Each citation reads `(see templates/admin/tdd-discipline.md for the Red → Green → Refactor cycle contract)`. Phase D's existing lane sub-phase numbering (D.1/D.2/D.3) is unchanged.
  - Trace: interview Round 2; DA-1 (D.X collision avoidance — cycle steps are NOT phase-numbered).
  - Serves goal: Outcome (cycle is invoked at the right insertion points without replacing phase structure).

- **R3.** The Refactor cycle step is bounded by three clauses, documented in `templates/admin/tdd-discipline.md`: (a) no new behaviour — every test that passed at Green still passes at Refactor end; (b) no new files — Refactor only modifies files Green touched; (c) no scope creep — if Refactor wants to add a file or change scope, the agent halts and routes to `specflow:scope-change`. Violations are recorded post-hoc in the per-task manifest scratch as `refactor: failed (new-file-attempted)` (or analogous outcome) — no runtime file-set diff detection.
  - Trace: interview Round 3; DA-5 (no host for runtime detection — demote to manifest-marker outcome).
  - Serves goal: Outcome + Driving value (Surgical Changes binds Refactor; cycle bounded inside the single context window).

- **R4.** Refactor is optional for trivial tasks (one-line changes that do not benefit from a structural pass). When omitted, the per-task manifest records `refactor: skipped (trivial)`.
  - Trace: interview Round 3.
  - Serves goal: Outcome (structural pass is a tool, not a tax).

- **R5.** Yellow lane refuses to enter the Green cycle step without the Red artefact (the per-task plan section in `{NNN-slug}-test.md` containing at least one case marked `Status: red (failing)` AND the captured pre-implementation failing exit per R14). Yellow is mandatory tests-first.
  - Trace: interview Round 1, Round 2.
  - Serves goal: Audience (engineer using develop in Yellow gets enforced TDD).

- **R6.** Add `develop.tddRequired` (boolean, default `true`) to `develop/SKILL.md` Phase D body. When `tddRequired: true` (default), Green lane behaves identically to Yellow. When `tddRequired: false`, Green lane may skip the Red artefact, and the per-task manifest records `red: skipped (config)`.
  
  **Strong CI signal — operational definition.** Per the interview Round 1 phrase, "strong CI signal" means: (a) every PR runs `vitest`, `typecheck`, and `lint` as required checks; (b) merge is blocked on red checks; (c) the project enforces a binary-AC convention in PRDs (every AC has a binary pass/fail criterion). The skill does NOT validate the precondition; the operator self-attests by setting `tddRequired: false`. The manifest records the attestation alongside `red: skipped (config)` so the choice is auditable.
  - Trace: interview Round 1; TBC-2 (operationalise the phrase).
  - Serves goal: Driving value (knob respects projects with strong CI signal; precondition is documented, not vibes).

- **R7.** Red lane (human-led, per `rules/non-negotiable.md` PROTECTED_PATHS_REQUIRE_RED_LANE) does not have its TDD discipline enforced by `specflow:develop` — the human leads. **Extension** (per codex-r1-f1): when AI assists by drafting or changing testable code in a Red-lane task, EITHER the human supplies a Red artefact before any AI Green-like implementation assistance OR the manifest records `red: skipped (human-led Red lane)` with **no `green: passed` marker emitted by the skill**. This prevents Red-lane work from quietly bypassing the TDD audit signal.
  - Trace: interview Round 1; codex-r1-f1.
  - Serves goal: Outcome (lane semantics preserved; TDD audit signal honest in Red).

- **R8.** Extend `test/SKILL.md` `--plan-only` mode (currently at `test/SKILL.md:39, 51, 214`) so that `--plan-only --task T{N}` writes the per-task plan section into `{NNN-slug}-test.md` and marks the primary AC's test case as `Status: red (failing)` by default. **The `--task T{N}` mode SKIPS Phase B.5** (Codex pass + user prompt) — the per-task slice inherits whatever B.5 outcome the feature-level test plan recorded; Gate 5's Codex reviewer covers cross-provider concerns at code-vs-plan time. **Edits bounded** to lines 39, 51, 214; Phase B's case-derivation logic and B.3's write template unchanged in shape — only the per-task filter, the `Status: red (failing)` default, and the B.5 skip are added.
  - Trace: interview Round 2; TBC-5 (B.5 collision); SURG-5 (edit-bound clause).
  - Serves goal: Outcome (Red artefact contract is operational without prompt-fatigue inside Red sub-step).

- **R9.** The per-task manifest scratch at `admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md` (one file per task) carries three markers in this exact shape (the **interim stub schema** 017 owns):
  ```
  red: {outcome} ({YYYY-MM-DDTHH:MM:SSZ}) — {one-line note}
  green: {outcome} ({YYYY-MM-DDTHH:MM:SSZ}) — {one-line note}
  refactor: {outcome} ({YYYY-MM-DDTHH:MM:SSZ}) — {one-line note}
  ```
  Where `{outcome}` ∈ `{passed, failed, skipped (config), skipped (trivial), skipped (human-led Red lane), failed (new-file-attempted)}`. Markers are appended in cycle order. **Schema ownership** (per codex-r1-f2): 017 owns *only* this interim stub contract — semantic fields (`red` / `green` / `refactor`), outcome enum, ISO timestamp, one-line note. **019 owns the canonical per-task manifest schema** and provides a migration/adapter from this stub when 019 lands; 019 is NOT required to read the three-line block "unchanged" — 019 may transform the stub into its own format. The stub path lives in scratch (NOT in `{NNN-slug}-tasks.md`) so it does not broadcast into every downstream skill that reads tasks.md.
  - Trace: interview Round 4; SURG-1, DA-3 (move out of tasks.md); codex-r1-f2 (017 owns interim only, 019 owns canonical).
  - Serves goal: Audience (admin auditing the cycle in retro) + Driving value (binary-cycle audit signal without artefact-broadcast).

- **R10.** Phase E (Gate 5 code-vs-plan, `develop/SKILL.md:446-533`) reviews the combined Red+Green+Refactor diff as a single change set. **Phase E.3 reviewer-dispatch input list does NOT include the per-task manifest scratch file or the cycle markers** — Phase E reviewers see only the code diff, the per-task plan, the tasks-file entry, and their own role definition. Phase F (Verifier + PR) is unchanged in scope.
  - Trace: interview Round 4; TBC-3 (positive-input-list test, not absence-of-string grep).
  - Serves goal: Outcome (Phase E stays lean per 027 fresh-context; cycle structure does not leak into dumb-zone review).

- **R11.** Refactor's "no new files" rule is documented-discipline-only (no runtime detection in D.3). When violated, the per-task manifest records `refactor: failed (new-file-attempted)` so retros catch the violation post-hoc. The cycle contract states "no new files in Refactor" as a discipline; agents that want to add a file route to `specflow:scope-change`.
  - Trace: interview Topics not discussed (hard-block decision); DA-5 (no runtime host); simplicity-r1-f5.
  - Serves goal: Outcome (cycle bounds enforced via retro audit, not runtime detection).

- **R12.** Add a "TDD" section to `CORE_PRINCIPLES.md`. Section is **one paragraph** (≤8 lines, not 30) summarising Pocock's Red → Green → Refactor framing with the canonical Pocock quote from `knowledge/pocock-software-fundamentals-matter-more.md` § Failure Mode 3 cited verbatim, and citing `templates/admin/tdd-discipline.md` as the canonical contract. Pattern: principle entry is short; doctrine doc is the home for the operational details.
  - Trace: original user request — *"TDD should inherit Matt Pocock's Red, Green approach"*; codebase context bullet 3.
  - Serves goal: Outcome (principle adopted; doctrine doc keeps the file tight) + Driving value (downstream skills cite the principle by name).

- **R13.** Extend the `eval:` field of both `develop/SKILL.md` and `test/SKILL.md` to exercise the new TDD discipline.
  - `develop/SKILL.md` eval (line 26) gains: "for every Yellow-lane task and every Green-lane task with `config.develop.tddRequired === true`, the per-task manifest scratch at `admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md` contains a line matching the regex `^red:\\s+(passed|failed|skipped \\((config|trivial|human-led Red lane)\\))\\s+\\([0-9T:Z-]+\\)\\s+—\\s+.+$` and equivalent regexes for `green:` and `refactor:`."
  - `test/SKILL.md` eval (line 19) gains: "when invoked with `--plan-only --task T{N}`, the per-task plan section written into `{NNN-slug}-test.md` contains at least one test case marked `Status: red (failing)`."
  - Trace: GD-r1-f6 (mandatory eval-coverage).
  - Serves goal: Outcome (the eval is the binary contract; without these clauses, AC-1 through AC-13 can pass on a SKILL.md that no longer behaves correctly at runtime).

- **R14.** **Pre-implementation test execution.** After authoring the test in the Red sub-step and before Green entry, the targeted test command (e.g. `vitest run path/to/test`) runs against the pre-implementation state and captures the failing exit + stderr in `admin/scratch/{NNN-slug}-develop/red-test-trace-{task-id}.log`. Green may flip `Status: green (passing)` only when the same command subsequently exits 0. **A pre-implementation pass blocks as an invalid Red artefact** — the test was written after the implementation (the Pocock failure mode `knowledge/pocock-ai-coding-real-engineers.md` § Phase 5 names verbatim).
  - Trace: codex-r1-f4.
  - Serves goal: Outcome (cycle integrity is verifiable, not a doc convention).

## Acceptance criteria

- **AC-1.** `develop/SKILL.md` Phase D body cites `templates/admin/tdd-discipline.md` exactly three times (one citation each in D.1 Green-lane body, D.2 Yellow-lane body, and the Phase B.5/B.6 prelude). The citations appear in canonical cycle order — the file's first three matches of `tdd-discipline.md` reference Red, then Green, then Refactor in that order. Verified:
  ```sh
  grep -nE 'tdd-discipline\.md|Red sub-step|Green sub-step|Refactor sub-step' plugins/specflow/skills/develop/SKILL.md | head -10
  # Manual check: first three cycle-step references appear in Red/Green/Refactor order with strictly increasing line numbers.
  [ "$(grep -c 'templates/admin/tdd-discipline.md' plugins/specflow/skills/develop/SKILL.md)" -ge 3 ]
  ```
  - Verifies: R2 (citations exist) + R1 (doctrine doc home).

- **AC-2.** `develop/SKILL.md` D.1 (Green-lane) body or the Phase B.5/B.6 prelude contains the literal invocation `specflow:test {NNN-slug} --plan-only --task T{N}` within 10 lines of a verb anchor (`invoke`, `fire`, `run`, `execute`):
  ```sh
  grep -B 0 -A 10 -E '\b(invoke|fire|run|execute)\b' plugins/specflow/skills/develop/SKILL.md | grep -qE 'specflow:test \{NNN-slug\} --plan-only --task T\{N\}'
  ```
  - Verifies: R8.

- **AC-3.** `develop/SKILL.md` Green-cycle-step entry is gated by a verify clause: within 30 lines after the Green sub-step citation, the file contains BOTH the literal substring `Status: red (failing)` AND a refusal anchor (`refuse`, `refuses`, `refusal`, `block`) AND a `tddRequired` conditional clause:
  ```sh
  awk '/Green sub-step/,/Refactor sub-step|## Phase E/' plugins/specflow/skills/develop/SKILL.md > /tmp/green-block
  grep -q 'Status: red (failing)' /tmp/green-block
  grep -qE 'refuse|refuses|refusal|block' /tmp/green-block
  grep -qE 'tddRequired.*(true|false)' /tmp/green-block
  ```
  - Verifies: R5, R6.

- **AC-4.** `templates/admin/tdd-discipline.md` Refactor section lists three explicit bounds — (a) no new behaviour, (b) no new files, (c) no scope creep / route to `specflow:scope-change`. Each bound on its own bulleted line:
  ```sh
  awk '/^## Refactor/,/^## /' plugins/specflow/templates/admin/tdd-discipline.md > /tmp/refactor-block
  grep -qE '^- .*no new behaviour' /tmp/refactor-block
  grep -qE '^- .*no new files' /tmp/refactor-block
  grep -qE '^- .*specflow:scope-change' /tmp/refactor-block
  ```
  - Verifies: R3, R11.

- **AC-5.** `config.develop.tddRequired` is documented in `develop/SKILL.md` Phase D body (matching the existing `greenBatchCap` precedent at line 405). The doc names the boolean type, the default `true`, and the Green-lane scope of the knob. Verified by grep against `develop/SKILL.md`:
  ```sh
  grep -qE 'config\.develop\.tddRequired.*boolean.*default[: ]+true' plugins/specflow/skills/develop/SKILL.md
  ```
  No `admin/config.json` schema declaration required.
  - Verifies: R6.

- **AC-6.** `test/SKILL.md` `--plan-only` mode documentation lists `--task T{N}` as a valid co-flag and names the Red-artefact contract:
  ```sh
  grep -qE -- '--plan-only.*--task T\{N\}' plugins/specflow/skills/test/SKILL.md
  grep -q 'Status: red (failing)' plugins/specflow/skills/test/SKILL.md
  grep -qE 'Phase B\.5.*skip' plugins/specflow/skills/test/SKILL.md
  ```
  - Verifies: R8.

- **AC-7a.** A worked example fixture exists at `plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/yellow-happy-path.md` showing a Yellow-lane manifest stub with three `passed` markers carrying R9's exact schema-shape:
  ```sh
  test -f plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/yellow-happy-path.md
  grep -qE '^red: passed \([0-9T:Z-]+\) — ' plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/yellow-happy-path.md
  grep -qE '^green: passed \([0-9T:Z-]+\) — ' plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/yellow-happy-path.md
  grep -qE '^refactor: passed \([0-9T:Z-]+\) — ' plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/yellow-happy-path.md
  ```
  - Verifies: R9 (cycle-held happy path).

- **AC-7b.** A worked example fixture exists at `plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/green-skip-config.md` showing a Green-lane task with `tddRequired: false` carrying `red: skipped (config) (...)`, `green: passed (...)`, `refactor: skipped (trivial) (...)`:
  ```sh
  test -f plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/green-skip-config.md
  grep -qE '^red: skipped \(config\) \([0-9T:Z-]+\) — ' plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/green-skip-config.md
  grep -qE '^refactor: skipped \(trivial\) \([0-9T:Z-]+\) — ' plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/green-skip-config.md
  ```
  Exercises both `skipped (config)` (R6) and `skipped (trivial)` (R4).
  - Verifies: R4, R6, R9.

- **AC-7c.** The per-task manifest scratch path is `admin/scratch/{NNN-slug}-develop/manifest-stub-{task-id}.md` (one file per task). Verified by grep against the doctrine doc:
  ```sh
  grep -qE 'admin/scratch/\{NNN-slug\}-develop/manifest-stub-\{task-id\}\.md' plugins/specflow/templates/admin/tdd-discipline.md
  ```
  Tasks.md is NOT touched — verified by grep absence:
  ```sh
  ! grep -qE 'Per-task manifest stub|manifest-stub' plugins/specflow/skills/task/SKILL.md
  ```
  - Verifies: R9 (scratch-path home; tasks.md untouched).

- **AC-8.** Phase E (Gate 5, `develop/SKILL.md:446-533`) reviewer-dispatch input list (the bullets passed to each forked reviewer sub-agent) does NOT include the per-task manifest scratch or the cycle markers:
  ```sh
  awk '/^## Phase E/,/^## Phase F/' plugins/specflow/skills/develop/SKILL.md > /tmp/phase-e
  ! grep -qE 'manifest-stub|per-task manifest scratch|cycle marker|red:.*passed|green:.*passed' /tmp/phase-e
  ```
  Phase E.3 reviewers see only: code diff, per-task plan, tasks-file entry, role definition.
  - Verifies: R10.

- **AC-9.** `CORE_PRINCIPLES.md` carries a "TDD" principle entry that is one paragraph (≤8 lines) and cites `templates/admin/tdd-discipline.md` as the doctrine home. The verbatim Pocock quote from `knowledge/pocock-software-fundamentals-matter-more.md` § Failure Mode 3 appears in the entry:
  ```sh
  awk '/^## TDD/,/^## /' plugins/specflow/CORE_PRINCIPLES.md > /tmp/tdd-section
  [ "$(wc -l < /tmp/tdd-section)" -le 10 ]
  grep -q 'templates/admin/tdd-discipline.md' /tmp/tdd-section
  grep -q 'TDD forces the LLM to really take small steps' /tmp/tdd-section
  ```
  - Verifies: R12.

- **AC-10.** `develop/SKILL.md` Red-lane handoff path (the existing flow under PROTECTED_PATHS_REQUIRE_RED_LANE) is unchanged in scope, AND the cycle-marker contract for AI-assisted Red-lane tasks is documented:
  ```sh
  awk '/^### D\.3 Red-lane/,/^### |^## Phase E/' plugins/specflow/skills/develop/SKILL.md > /tmp/d3
  ! grep -qE 'tddRequired|Status: red \(failing\) check' /tmp/d3
  grep -qE 'human-led Red lane' /tmp/d3
  ```
  - Verifies: R7.

- **AC-11a.** `templates/admin/tdd-discipline.md` Refactor section contains the literal halt message AND a halt anchor:
  ```sh
  awk '/^## Refactor/,/^## /' plugins/specflow/templates/admin/tdd-discipline.md > /tmp/refactor-block
  grep -q 'Refactor cannot add files; route to specflow:scope-change' /tmp/refactor-block
  grep -qE 'halt|halts|refuse|refuses' /tmp/refactor-block
  ```
  - Verifies: R11 (documented discipline).

- **AC-11b.** A worked-fixture file at `plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/refactor-new-file-block.md` shows a worked task narrative ending in the halt message:
  ```sh
  test -f plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/refactor-new-file-block.md
  grep -q 'Refactor cannot add files; route to specflow:scope-change' plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/refactor-new-file-block.md
  grep -qE 'refactor: failed \(new-file-attempted\)' plugins/specflow/examples/docs/specflow/features/017-tdd-discipline/fixtures/refactor-new-file-block.md
  ```
  - Verifies: R11 (post-hoc audit + worked fixture).

- **AC-12.** `develop/SKILL.md` and `test/SKILL.md` `eval:` fields are extended per R13.
  ```sh
  grep -qE 'eval:.*manifest-stub|eval:.*\^red:\\s\+\(passed' plugins/specflow/skills/develop/SKILL.md
  grep -qE 'eval:.*--task T\{N\}|eval:.*Status: red \(failing\)' plugins/specflow/skills/test/SKILL.md
  ```
  - Verifies: R13.

- **AC-13.** Pre-implementation test trace exists. Given a fixture Yellow-lane task that authors a failing test and runs the targeted test command pre-implementation:
  ```sh
  ls admin/scratch/{NNN-slug}-develop/red-test-trace-{task-id}.log
  grep -qE 'exit (1|[1-9][0-9]*)' admin/scratch/{NNN-slug}-develop/red-test-trace-{task-id}.log
  ```
  And given the same fixture progressed through Green:
  ```sh
  grep -q 'exit 0' admin/scratch/{NNN-slug}-develop/red-test-trace-{task-id}.log  # appended after Green
  ```
  An invalid Red artefact (test passes pre-implementation) blocks Green entry:
  ```sh
  # Fixture: test that passes against pre-implementation state
  specflow:develop NNN-slug; rc=$?; [ $rc -ne 0 ] && grep -q 'invalid Red artefact' stderr.log
  ```
  - Verifies: R14.

## Open questions

- **OQ-1: 017→019 absorption contract.** R9 declares 017 owns the interim stub schema and 019 owns the canonical per-task manifest. The migration adapter from stub to canonical lives in 019's PRD. **Decision needed before 019's PRD synthesis:** the absorption contract must be documented in 019 citing 017's R9 schema verbatim from `templates/admin/tdd-discipline.md`. If 019 lands with a structurally different schema, R9's stub becomes legacy noise; the doctrine doc reopens.

## See also

- Interview: [`./017-tdd-discipline-interview.md`](./017-tdd-discipline-interview.md)
- Doctrine doc: [`../../templates/admin/tdd-discipline.md`](../../templates/admin/tdd-discipline.md) (created by this PRD; canonical home for the cycle contract)
- Tasks: [`./017-tdd-discipline-tasks.md`](./017-tdd-discipline-tasks.md) (generated by `specflow:task`)
- Tests: [`./017-tdd-discipline-test.md`](./017-tdd-discipline-test.md) (generated by `specflow:test`)
