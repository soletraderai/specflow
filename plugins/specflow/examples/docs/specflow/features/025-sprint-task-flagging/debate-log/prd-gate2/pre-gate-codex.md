# PRD Adversarial Review — 025-sprint-task-flagging

Input note: `/Users/marepomana/Web/specflow/plugins/specflow/examples/docs/specflow/v2/docs/FEATURES.md` could not be read (`No such file or directory`); fallback `v2/docs/FEATURES.md` was read and used.

## Axis 1: Vision-to-Goal Trace
[MAJOR] The Vision states that the heuristic is documented so "two synthesis runs produce the same buckets given the same task graph" (Vision), and the Goals repeat that "two synthesis runs over the same task graph produce identical bucket assignments" (Goals). The stated goals add the field and require documentation, but they do not require the missing deterministic tie-break needed to make that reproducibility real. This is a conceptual gap between the vision's deterministic-artifact claim and the goal-level work actually specified.

## Axis 2: Requirements (R1–R10) — Trace + Serves-goal
[BLOCKER] R2 claims the heuristic is reproducible, but its dependency rule does not define deterministic ordering or numbering when several tasks have the same dependency depth and disjoint scope. R2 says tasks land in the "lowest bucket" and that parallel tasks "share a bucket," but it never specifies how task traversal order is chosen when multiple valid topological linearisations exist. The Trace and Serves-goal rationale are therefore underspecified for the reproducibility goal they claim to serve.

[MAJOR] R2's "topological floor" wording permits same-bucket dependencies: "predecessors all sit in lower or same-numbered buckets" and "only on tasks in bucket N or N+1." That conflicts with the Vision/R4 explanation that bucket 1 means "no predecessors" and bucket 2 means "depends only on bucket 1." If dependent tasks can share a bucket, the sprint-skill could fan out work that actually has an ordering dependency.

[MAJOR] R3 says `sprint-bucket` is computed from `Depends on:` plus scope-overlap, but the PRD never defines how free-form `Scope:` surfaces are normalized or compared. The Trace repeats the interview answer, but does not make "scope-overlap" mechanically evaluable. The requirement therefore serves the cached-topology goal only if a deterministic scope-surface comparison already exists, which is not established in R3.

[MAJOR] R7 states that oversized tasks "are split into smaller sub-tasks before bucketing," but the requirement does not define a required verify step that blocks bucket assignment until the 029 split flow has completed. The Serves-goal text says bucketing "inherits a clean task list," but the PRD does not specify the observable condition that proves the list is clean. This leaves the phase-ordering claim underspecified rather than enforceable.

[MAJOR] R9's binary topological-floor invariant allows every dependency to have `sprint-bucket <= N`. That permits a task in bucket N to depend on another task also in bucket N, which undercuts the bucket-as-parallel-fan-out meaning stated in Vision, R2, and R3. The rationale is not just underspecified; the invariant appears too weak for the goal it is meant to verify.

[MAJOR] R10 says `sprint-bucket` is read-only for downstream consumers and that hand edits are a `specflow:scope-change` concern, but its Trace only says the field is derived. "Derived" does not by itself establish a skill-level enforcement path, a validation failure, or a required re-synthesis behavior if a developer manually edits `{NNN-slug}-tasks.md`. The Serves-goal rationale depends on downstream consumers trusting provenance, but R10 does not define how provenance is protected.

## Axis 3: Acceptance Criteria (AC-1 to AC-11) — Binary testability
[MINOR] AC-1's proposed verification, `grep -E 'sprint-bucket' plugins/specflow/skills/task/SKILL.md`, is not binary for the stated criterion. That grep can pass if `sprint-bucket` appears in commentary, a verify step, or an unrelated note, without proving Phase B's per-task entry shape lists it as a sibling field. The AC needs a more targeted structural check to be pass/fail without judgment.

[MAJOR] AC-2 verifies that the three-input heuristic is documented, but the expected documentation is not specific enough to prove deterministic bucket assignment. It does not require any tie-break for same-depth tasks, any stable task ordering rule, or any deterministic normalization of `Scope:` surfaces. A reviewer can pass AC-2 while AC-9 remains impossible to guarantee.

[MINOR] AC-4 allows the verify-step label to be "topological-floor invariant" "or equivalent." "Equivalent" requires human interpretation and weakens the binary nature of the AC. Separately, the check itself inherits R9's same-bucket-dependency weakness by allowing dependency buckets `<= N`.

[MAJOR] AC-5 is not binary because it says an oversized task "triggers the existing 029 R4 split flow before bucket assignment" but does not name the artifact, state transition, or failure condition that proves the split flow occurred first. A synthesized output containing sub-tasks with buckets is observable, but the ordering between split and bucket assignment is not. This makes the core R7 sequencing claim hard to test without reading the implementer's intent.

[MAJOR] AC-6's verification by absence of `sprint-bucket: 0` does not prove that bucket numbers "start at 1." A faulty implementation could assign only `sprint-bucket: 2` and still pass the absence-of-zero check. The AC needs an explicit check that at least one task in the feature has `sprint-bucket: 1` and that no bucket value is below 1.

[MAJOR] AC-7 relies on "absence of any cross-feature bucket-merging logic in `task/SKILL.md`," which is not a binary check without a defined search surface and forbidden terms. Cross-feature merging could be implied indirectly without using that exact phrase. The behavioral claim about two independently synthesized features also needs a concrete fixture or output comparison to be testable.

[BLOCKER] AC-9 asserts identical bucket values across two synthesis runs, but R2 does not define a deterministic tie-break when the graph admits multiple valid topological linearisations. The suggested `cmp` or targeted diff is binary as a test command, but the PRD does not specify enough algorithmic detail for an implementation to reliably pass it. This is the acceptance-level expression of the R2 determinism gap.

[MINOR] AC-10 verifies absence of auto-tuning and DAG visualization by "absence of those concepts." Without specific forbidden strings or a defined reviewer rule, this requires human interpretation and can be bypassed with synonyms. It is directionally aligned with the Non-goals, but not fully binary.

[MAJOR] AC-11 says any change to `sprint-bucket` requires re-synthesis or `specflow:scope-change`, but verifies only the absence of a hand-editing affordance in documentation. Absence of permission is not the same as presence of enforcement or a documented rejection path. A downstream consumer could still accept a manually edited field and AC-11 would pass.

## Axis 4: Out-of-scope Conflicts
No issues found.

## Targeted Challenges
### R2 — Bucket numbering uniqueness
[BLOCKER] R2 does not specify what tie-breaks the bucket number or task traversal order when multiple tasks have no `Depends on:` and disjoint `Scope:` surfaces. The PRD says those tasks "share a bucket" when parallel, but does not state whether identical-depth non-overlapping tasks must all be bucket 1, whether scope grouping can create later buckets, or how bucket numbers are assigned when several groupings are valid. This leaves the grouping-vs-numbering distinction unresolved.

### AC-9 — Determinism under multiple valid linearisations
[BLOCKER] AC-9 requires identical `sprint-bucket` values across runs, but the PRD never mandates a stable topological sort tie-break such as lexicographic task ID order. If the task list or graph traversal order changes across runs, the same graph can have multiple valid linearisations. The PRD needs an explicit deterministic rule before AC-9 is achievable rather than aspirational.

### R3/R10 — Read-only enforcement gap
[MAJOR] R3 says `sprint-bucket` is computed, and R10 says it is read-only for downstream consumers, but neither requirement states what the skill does if a developer manually annotates or edits the field in `{NNN-slug}-tasks.md`. R10 redirects hand edits to `specflow:scope-change`, yet AC-11 only checks that documentation does not invite manual edits. The enforceable rule is missing: reject, overwrite on re-synthesis, warn, or require scope-change before consumption.

### R7 — Phase-ordering vs sequencing observation
[MAJOR] R7 reads like a hard phase-ordering requirement, but the PRD does not require a verify step proving the 029 R4 budget split has completed before bucket assignment proceeds. AC-5 repeats the sequencing claim but does not name an observable precondition or artifact. As written, an implementation could bucket first, split later, and still be hard to disprove from the final tasks file alone.

## Unstated Assumptions
[ASSUMPTION] The PRD assumes free-form `Scope:` text can be compared deterministically enough to decide "disjoint Scope surfaces" in R2/R3.

[ASSUMPTION] The PRD assumes task IDs and task ordering are stable across synthesis runs before `sprint-bucket` assignment occurs, but AC-9 only constrains the final bucket lines.

[ASSUMPTION] The PRD assumes "same input PRD" in AC-9 also means the same synthesized task decomposition, not merely the same source PRD text.

[ASSUMPTION] The PRD assumes 029's budget split flow is already available as a callable pre-bucketing phase, but R7/AC-5 do not specify the handoff artifact or completion marker.

[ASSUMPTION] The PRD assumes downstream consumers will treat `sprint-bucket` as read-only because the documentation says so, without specifying validation or provenance checks.

[ASSUMPTION] The PRD assumes "no shared `Depends on:` edges" in R2 is sufficient to prove safe parallelism, even though a direct dependency between two tasks in the same bucket is still permitted by R2/R9's `<= N` invariant.

## Verdict
FAIL — the PRD's central deterministic-bucket promise is not enforceable until same-depth tie-breaking, same-bucket dependency handling, and read-only/phase-order verification are specified.


— User reviewed; revisions applied autonomously per session-start authorisation, 2026-05-07.
