## Axis 1 — Vision vs Interview Goal

[MAJOR] The Vision mostly traces to the interview Goal, but it imports a stronger auto-application story than the Goal's success text: PRD Vision says the applier "applies accepted changes to tasks.md," while interview Goal success only requires findings surfaced and resolved in a worked example, and interview Goal out-of-scope says "auto-applying cross-task suggestions" is out of scope. Fix by explicitly tying the applier model to interview Rounds 5-7, not the Goal paragraph alone, and acknowledge the Goal-level shift.

[MINOR] The Goals section says the mini-debate exists "so reviewer and writer have the standard convergence path," but R4 says "The original writer never responds to cross-task findings." Replace "writer" with "applier" or explain the role substitution.

## Axis 2 — R Trace + Serves-Goal Pairs

[INFO] R1-R11 each include a Trace and Serves-goal pair with concrete interview/feature evidence: R1 Round 1, R2 Rounds 2/11, R3 Rounds 3/7, R4 Round 5, R5 Round 6, R6 Round 9, R7 Round 8, R8 Round 4, R9 Round 10, R10 Round 2, R11 FEATURES.md Done-when. No fix needed.

[MINOR] R12's Trace cites "skill-size-ceiling memory feedback" and precedents, but not the interview Goal or a resolved interview round, so it is weaker than the PRD's normal trace standard. Add the source section/line for the memory feedback or mark it as implementation-constraint trace rather than goal trace.

[MAJOR] R13's Trace is circular: "eval-coverage mandate (every R must trace to a binary AC; the eval is the binary contract)" justifies adding eval coverage by saying eval coverage is required. Replace with a concrete source, such as the task skill's existing `eval:` field contract or a Gate 2 reviewer finding mandating eval coverage.

## Axis 3 — AC Binary / Executable

[BLOCKER] AC-4 is executable but appears unpassable because `awk '/^### E\.4 /,/^### E\.4\.5/'` includes the terminating E.4.5 header, and that header necessarily contains `Cross-task`, so `! grep -qE 'cross-task|cross_task' /tmp/e4-block` fails even for a correct implementation. Change the awk range to stop before the E.4.5 line or strip the final line before grepping.

[MAJOR] AC-3's command does not prove the four E.4.5 sub-steps exist; if grep returns zero or only one matching heading, `lines == sorted(lines)` still passes. Assert the exact heading suffix list is `[1,2,3,4]` and that count is four.

[MAJOR] AC-14 is not self-contained: it says "Fixture: a feature with 2 tasks" but runs against ambient `debate-log/tasks-gate3/manifest.md` and checks only absence of round-1 `cross-task-reviewer.json`, not round-2/round-3 cross-task response files. Make the fixture path explicit and assert no cross-task files exist in all three rounds.

[BLOCKER] AC-15's deliberate collision half is a prose assertion plus grep against an already-generated manifest; it gives no executable way to force `writer_id == applier_id` under normal dispatch. Provide a fixture manifest/closer input that deterministically injects the collision, or downgrade this to a unit test for the closer parser.

## Axis 4 — R vs Goal-Level Out-of-Scope

[MAJOR] R7 risks contradicting the Goal out-of-scope "Building 027-reviewer-context-isolation itself" because it requires dispatch-time agent IDs and a gate-failing `FRESH-CONTEXT-VIOLATION` enforcement path before 027 exists. Reframe R7 as manifest-field plumbing only unless the current Agent tool exposes stable run IDs.

[MINOR] R10 is close to the Non-goal "Changing the per-task review behaviour" because per-task Round 3 now reviews post-cross-task-applied tasks rather than only the post-E.4 artefact. Add a sentence that the reviewer lenses and response schema are unchanged while only the artefact snapshot changes.

## A — Phase E.4.5 Numbering

[MAJOR] The PRD does not prove E.4.5 is safe downstream: current `task/SKILL.md` has contiguous headings E.4 Round 2, E.5 Round 3, E.6 Closer, E.7 Final disposition, E.8 Clean up, and existing manifests describe "Round 2" then "Round 3" directly. Add a compatibility check for downstream consumers that grep or slice `^### E\.5` / `Round 3`, especially `scope-change/SKILL.md` E.6 re-firing task Phase E and `brief/SKILL.md` reading `tasks-gate3/manifest.md`.

## B — Applier as `specflow:task --apply-cross-task-feedback`

[MAJOR] The mode-flag shape is only partially supported by current `task/SKILL.md`: Inputs list `specflow:task {NNN}` and `/specflow:task` only, while the only comparable mode flag in nearby skills is `develop --task T{N}` and scope-change's explicit `mode: manual | auto-handoff...` routing. Add an Inputs/Resume branch for `--apply-cross-task-feedback` before Phase A so it is a first-class mode, not a hidden re-entry into a five-phase skill.

## C — R7 Agent IDs

[BLOCKER] R7 treats `agent_id` as solved, but current skill docs only say "dispatch a forked sub-agent" and "return only the file path" (`task/SKILL.md` E.3), with no observable run ID in the reviewer output contract. Specify the source of `writer_id`, forked reviewer run IDs, and re-invoked applier ID, or reduce the diagnostic to a best-effort manifest field until 027 lands.

## D — AC-15 Deliberate ID Collision

[BLOCKER] AC-15 is not testable under normal dispatch because R7 requires all IDs be "populated at dispatch time" and distinct on successful close, while AC-15 asks for a "Fixture: forced writer_id == applier_id" without a sanctioned way to force dispatch identity collision. Add a closer-unit fixture that bypasses dispatch and feeds a manifest input with duplicate IDs, or remove the collision fixture from Gate 3 acceptance.

## E — Cross-Task R3 vs Per-Task Round 3 Cycle

[MAJOR] R10 states per-task reviewers re-fire after E.4.5 against post-applier tasks.md, but R3.4 allows a final applier pass after cross-task R3 sharpens and does not explicitly require Phase B.4 self-check after that final pass. Add an ordering guarantee that E.4.5.4 completes, writes final tasks.md, and reruns B.4 before E.5 starts.

## F — 029 Hard-Cap Interaction

[MAJOR] R9 explicitly allows a merge suggestion where "T3 (60K) and T4 (65K); merging would exceed the 80K cap," but R6 only says coverage-changing findings route to scope-change and is silent on budget-cap-violating but coverage-preserving merges. State that the applier must decline or route any accepted merge that would exceed `context-budget-estimate` hard cap to scope-change/re-synthesis, never apply it in place.

## Unstated Assumptions

[MAJOR] The PRD assumes 022 can observe stable Agent tool IDs before 027, but neither `task/SKILL.md` nor the reviewer role files expose such IDs today. Make this an explicit dependency or implement an interim run-ID generation scheme owned by the orchestrator.

[MINOR] The PRD assumes downstream skills consume only final tasks.md despite R8 adding manifest structure, but `brief/SKILL.md` optionally reads `tasks-gate3/manifest.md`. Add a brief-compatible manifest shape check or state that brief ignores the new section.


— User reviewed; revisions applied autonomously per session-start authorisation, 2026-05-07.
