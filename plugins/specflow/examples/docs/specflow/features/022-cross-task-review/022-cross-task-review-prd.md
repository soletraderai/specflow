---
feature: 022-cross-task-review
status: draft
created: 2026-05-07
interview: ./022-cross-task-review-interview.md
---

# Cross-Task Review

## Vision

`specflow:task` Gate 3 gains a **whole-set cross-task review** alongside the existing per-task review. A new principle-aligned reviewer agent (`cross-task-reviewer`) fires in a dedicated Round 2.5 — after the orchestrator has applied per-task revisions, before per-task reviewers' Round 3 sharpen — and applies two lenses to the entire task list as a single artefact: **coherence** (do tasks work together; flow correctly; not overlap; all trace back to PRD requirements + goal) and **better arrangement** (could two tasks merge; could one split; is the dependency ordering optimal; is there a missing task the per-task lens couldn't see). Per interview Rounds 5-7 the review is **feedback-only**: a separate fresh-context **applier agent** (`specflow:task --apply-cross-task-feedback`) reads the feedback in isolation and *decides* which findings warrant action on their own merits before any tasks.md edits land — there is no auto-apply path. The applier's accept / reject / scope-change-required decisions are recorded in the manifest; only `accepted` decisions produce tasks.md edits, and only when those edits preserve PRD coverage AND respect 029's per-task budget cap. The reviewer and applier each run in their own forked sub-agent, never seeing the writer's chat (per 027); the manifest carries `agent_id` fields so writer / reviewer / applier identity is auditable. Gate 3 ships with the Pocock issue-sizing-heuristic and Medin reviewer-isolation-discipline both honoured at the same gate.

## Problem

`specflow:task` Phase E (Gate 3) today (`task/SKILL.md:320-444`) runs the standard reviewer set against *individual tasks*. Per E.3 lens documentation, the existing reviewers each carry one principle: Goal-Driven verifies forward coverage, Surgical verifies reverse traceability, Simplicity flags over-decomposition, Think-Before-Coding flags unstated assumptions, Devil's Advocate flags Gate-2-to-Gate-3 drift. **There is no whole-set lens** today — no reviewer reads the entire task list as a single artefact and asks "is this the right granularity / arrangement / coverage at the *set* level?". Without that lens, oversized tasks can ship undetected (per 029's R4 hard-cap they're auto-flagged at synthesis, but within-cap arrangement issues — e.g. two tasks that are really one concept, one task that's really two concepts, a missing task the per-task lens couldn't see — slip through). Pocock's issue-sizing-heuristic (`knowledge/pocock-real-feature-build.md`) and Medin's reviewer-isolation-discipline (`knowledge/medin-parallel-agentic-playbook.md` § "reviewer should never see the writer's chat") both address gaps the current Gate 3 manifold does not catch. 022 closes the lens gap; 027 (Sprint 3) later formalises the isolation contract; 022's interim agent_id mechanism becomes 027's substrate (chain-don't-absorb).

## Goals

- Add a single new reviewer agent (`cross-task-reviewer`) to Gate 3's standard reviewer set, dedicated to whole-set coherence + better-arrangement lenses.
- Position cross-task review as a dedicated Round 2.5 in Gate 3 — after the orchestrator's Round 2 response to per-task findings, before per-task reviewers' Round 3 sharpen.
- Run a three-round mini-debate inside Round 2.5 (cross-task R1 fire → applier R2 response + apply accepted → cross-task R3 sharpen → applier final pass) so reviewer and applier have the standard convergence path. (The original writer never participates in cross-task review — see R4.)
- Make cross-task review **feedback-only**: a separate fresh-context applier agent (`specflow:task --apply-cross-task-feedback {NNN-slug}`) reads the feedback in isolation, decides which findings to act on, applies accepted changes to tasks.md, and records accept/reject decisions in the manifest. The original writer never responds to cross-task findings.
- Add `writer_id`, `cross_task_reviewer_id`, and `applier_id` fields to the Gate 3 manifest schema; verify all three differ at gate close (interim fresh-context audit signal pending 027).
- Auto-route PRD-coverage-changing cross-task findings (add missing task, drop covering task, merge with coverage ambiguity) to `specflow:scope-change`; coverage-preserving findings (merge of two tasks both anchoring R5; reorder within preserved dependency edges) auto-apply.
- Surface within-budget merge/split signals from `context-budget-estimate` (029) as a soft input to the better-arrangement lens (not a hard-cap re-check; 029 R4 already covers that).
- Skip cross-task review entirely when the synthesised task list has fewer than 3 tasks (record `Cross-task review skipped: task count {N} below threshold (3)` in the manifest).
- The closing Gate 3 manifest gains a dedicated "Cross-task findings" section parallel to the existing per-task sections, with its own Accepted / Rejected / Escalated sub-headings and one shared closing-decision footer covering both lenses.

## Non-goals

- Building 027-reviewer-context-isolation itself. 027 is a Sprint 3 dependency 022 *uses*. 022 ships the interim `agent_id` mechanism + forked-sub-agent dispatch convention; 027 absorbs/extends. (Goal-level out-of-scope.)
- Changing the per-task review behaviour. The five existing reviewers (Simplicity / Surgical / Think-Before-Coding / Goal-Driven / Devil's Advocate / Codex when available) keep their per-task lenses unchanged.
- Auto-applying cross-task findings without an applier review pass. Cross-task is feedback-only by design (interview Round 5).
- Cross-feature task batching. Per 025 R5 (parked at higher orchestration layer).
- Linear cycle integration of cross-task findings. Deferred to 020-sprint-skill family.
- Per-feature opt-out for cross-task review. The threshold (<3 tasks) is mechanical, not configurable. (Per interview Round 11.)
- Concrete lens-checklist prompts (the specific sub-questions the cross-task reviewer asks). The role-def file at `admin/agents/standard/principles/cross-task-reviewer.md` carries those; the PRD names the two lenses, not the per-question text.
- Auto-detecting the optimal task count. The reviewer surfaces merge/split *suggestions*; deciding the right count is the applier's call, not an algorithmic rule.
- 028-edge-case-reviewer interaction. 028 is Sprint 3; both reviewers stand independently. Integration is PRD-level, not 022's surface.

## Users

- **Engineer using the task skill.** Direct user. Sees the cross-task findings in the Gate 3 manifest's new dedicated section alongside per-task findings; reviews the applier's accept/reject decisions; resolves any escalations or scope-change-routed findings before proceeding to `specflow:test` / `specflow:develop`.
- **Admin — platform operator.** Reads the closing Gate 3 manifest for retro / audit; values the dedicated cross-task section as a lens-distinguished synthesis trace; values the `agent_id` triplet as a fresh-context audit signal.
- **Downstream skill consumers.** `specflow:sprint` (020), `specflow:develop`, `specflow:test` read the closed Gate 3 manifest. They treat cross-task findings the same as per-task — the lens distinction is recorded but consumers act on the post-applier `{NNN-slug}-tasks.md`, not the manifest's reviewer-deliberation transcript.

(Profiles per `admin/profiles.json`. The cross-task reviewer and applier are programmatic — no profile applies; treated as tooling agents.)

## Requirements

- **R1.** Add a new reviewer role-def file at `plugins/specflow/examples/docs/specflow/admin/agents/standard/principles/cross-task-reviewer.md`. The reviewer's two lenses are documented in the role-def: (1) **Coherence** — do tasks work together; flow correctly; not overlap; all trace back to PRD requirements + goal. (2) **Better arrangement** — could two tasks merge; could one split; is the dependency ordering optimal; is there a missing task the per-task lens couldn't see. The role-def carries the existing principle-reviewer schema (severity / evidence / claim / proposed_change) plus a `lens` field per finding (`coherence | better-arrangement`) so the manifest can surface the distinction.
  - Trace: interview Round 1 — *cross-task review is implemented as a single new reviewer agent added to Gate 3's standard reviewer set, not as a re-application of existing reviewers and not as a separate gate.*
  - Serves goal: Outcome (single new reviewer slot) + Audience (manifest reader sees the lens distinction).

- **R2.** Extend `task/SKILL.md` Phase E to add a new sub-phase **Phase E.4.5 — Cross-task review** between the existing E.4 (Round 2 — orchestrator response) and E.5 (Round 3 — per-task sharpen). E.4.5 fires only when the synthesised `{NNN-slug}-tasks.md` carries **3 or more tasks** (counted by `### T-` headings). When fewer than 3 tasks exist, Phase E.4.5 is skipped entirely; the manifest's "Cross-task findings" section is populated with a single line: `Cross-task review skipped: task count {N} below threshold (3)`.
  - Trace: interview Round 2 — *cross-task reviewer fires as a dedicated Round 2.5 AFTER the orchestrator's Round 2 response.* Round 11 — *cross-task review fires only when the synthesised task list has 3 or more tasks.*
  - Serves goal: Outcome (Round 2.5 placement) + Driving value (threshold avoids meaningless cross-task review on trivial features).

- **R3.** Phase E.4.5 runs a three-round mini-debate, NOT a single-shot review:
  1. **E.4.5.1 — Cross-task R1 fire.** Dispatch a forked sub-agent reading `cross-task-reviewer.md`, the post-Round-2 `{NNN-slug}-tasks.md`, the PRD, the interview, the Gate 2 manifest. Reviewer writes findings to `debate-log/tasks-gate3/findings/round-1/cross-task-reviewer.json` (per-finding `lens` field per R1).
  2. **E.4.5.2 — Applier R2 response + apply.** Re-invoke `specflow:task --apply-cross-task-feedback {NNN-slug}` in a fresh context window. The applier reads the cross-task R1 findings + the post-Round-2 tasks.md + the PRD/interview, decides accept / reject / scope-change-required per finding, applies accepted changes to tasks.md, and writes its decisions to `debate-log/tasks-gate3/findings/round-2/cross-task-responses.json`.
  3. **E.4.5.3 — Cross-task R3 sharpen.** Re-dispatch the cross-task reviewer (fresh forked sub-agent) reading its R1 findings + the applier's R2 responses + the post-applier tasks.md. May sharpen any rejection with new evidence or escalated severity. Writes to `debate-log/tasks-gate3/findings/round-3/cross-task-reviewer.json`.
  4. **E.4.5.4 — Applier final pass.** If any sharpens, re-invoke the applier on the sharpened findings. Decisions append to `cross-task-responses.json`. Unresolved findings escalate to the manifest's Cross-task Escalations sub-section.
  - Trace: interview Round 3 — *cross-task reviewer gets its own three-round mini-debate ... after three cross-task rounds the existing Gate 3 convergence rules apply.* Round 7 — *full three-round flow with the applier playing the orchestrator's role.*
  - Serves goal: Outcome (three-round convergence with forced decision after three rounds).

- **R4.** Cross-task review is **feedback-only**. The original `specflow:task` orchestrator (Phase E.4 Round 2) does NOT respond to or auto-apply cross-task findings — only the dedicated applier (R3 above) does. The applier runs in a fresh context window: it reads only the feedback findings, the post-Round-2 tasks.md, and the PRD / interview — never the writer's chat or the cross-task reviewer's deliberation transcripts beyond the written findings.
  - Trace: interview Round 5 — *cross-task review is feedback-only ... a separate fresh-context applier agent reads the feedback in isolation.*
  - Serves goal: Driving value (Medin reviewer-isolation discipline; the writer never gets to defend their original choices to the applier).

- **R5.** The applier is `specflow:task` re-invoked in a fresh context window via the new flag `--apply-cross-task-feedback {NNN-slug}`. When invoked this way, `specflow:task` skips Phases A / B / C / D entirely and enters a new Phase **F — Cross-task feedback application**. Phase F takes three inputs: (1) the post-Round-2 `{NNN-slug}-tasks.md`; (2) the cross-task findings JSON from `round-1/cross-task-reviewer.json`; (3) the PRD + interview for coverage-matrix validation. Phase F's output is (a) the post-applier `{NNN-slug}-tasks.md` (in-place edits) and (b) `debate-log/tasks-gate3/findings/round-2/cross-task-responses.json` recording per-finding decisions.
  
  **Precondition check (per DA-2 / tbc-r1-f3).** Phase F's first action is a precondition check on all three inputs. On missing `cross-task-reviewer.json` (the parent Gate 3 invocation never produced findings, e.g. user manually invoked the flag standalone), the applier refuses with: *"Cross-task review has not fired for this feature. Run `specflow:task {NNN-slug}` from Phase E to produce the findings, then re-invoke `--apply-cross-task-feedback`."* On missing tasks.md or PRD, refuse with the analogous diagnostic. The chain-skip-Phase-A behaviour assumes the parent run guaranteed chain verification; manual standalone invocation is an unsupported entry point in v2.5.
  - Trace: interview Round 6; DA-2; tbc-r1-f3 (manual-invocation guard).
  - Serves goal: Outcome (applier inherits synthesis logic for free) + Driving value (fresh context enforces isolation; standalone-invocation failure mode defined).

- **R6.** The applier's per-finding decision is one of: `accepted` (with `revision_applied: <description>`), `rejected` (with `rationale: <text>`), or `scope-change-required` (with `gap: <description>`). For `scope-change-required` findings, the applier halts further cross-task application for that finding and the manifest closes with a directive pointing the user to `specflow:scope-change`. PRD-coverage-changing findings (add missing task with new R/AC anchor; merge-with-coverage-ambiguity; drop-with-coverage-hole) MUST be flagged `scope-change-required`; the applier does NOT modify the PRD's coverage matrix on the fly.
  - Trace: interview Round 9 — *when the applier identifies a cross-task finding as PRD-coverage-changing ... it flags the finding `scope-change-required` ... auto-route to `specflow:scope-change`.*
  - Serves goal: Outcome (Gate 3 / scope-change boundary preserved) + Driving value (applier respects existing skill-boundary contract).

- **R7.** The Gate 3 manifest gains three best-effort opaque `agent_id` fields — `writer_id`, `cross_task_reviewer_id`, `applier_id` — populated at dispatch time. **Format is unspecified at the PRD level**: the orchestrator generates whatever opaque value is convenient (timestamp+suffix; UUIDv4; harness-emitted run ID — implementation chooses) and records it verbatim. 027 (Sprint 3) owns the canonical format contract; 022 ships only the field-presence + per-agent-distinctness convention as substrate. **No echo-back protocol; no closer-side collision check; no FRESH-CONTEXT-VIOLATION escalation surface** — those are 027 concerns. The audit signal is simply: in the green path all three fields are populated and pairwise non-equal; in the skip path only `writer_id` is populated; absent fields short-circuit any pairwise check (the closer does NOT pseudo-error on absent fields). 022's chain-don't-absorb contribution is the field NAMES + the populate-at-dispatch convention; 027 absorbs the format and adds runtime verification.
  - Trace: interview Round 8; surgical-r1-f2 (R7 simplified to manifest-field-only); simplicity-r1-f2; codex-r1-f3 / f4 (echo-back spoofing surface and timestamp collision concerns moot once closer doesn't enforce uniqueness).
  - Serves goal: Outcome (auditable best-effort signal) + Driving value (chain-don't-absorb foundation for 027 without claiming 027's format contract).

- **R8.** The closing Gate 3 manifest at `debate-log/tasks-gate3/manifest.md` gains a dedicated "Cross-task findings" H2 section parallel to the existing per-task sections. The cross-task section has three H3 sub-headings — `Accepted findings`, `Rejected findings`, `Escalated to human` — each entry carrying the finding ID, the reviewer (`cross-task-reviewer`), the lens (`coherence` | `better-arrangement`), the severity, the claim + evidence, and the applier's decision (revision applied, rationale for rejection, or escalation reason). The manifest's single closing-decision footer covers BOTH per-task and cross-task lenses (one Gate 3 status: passed / passed-with-revisions / passed-with-escalations / failed).
  
  **Closer precedence rule (per DA-4).** E.6 closer applies the existing FAIL-on-unresolved-`block` rule to the UNION of per-task and cross-task findings. A `block`-severity finding from EITHER lens triggers the same status. The closing rationale paragraph names which lens(es) drove the status (e.g. "passed-with-escalations: per-task PASSED; cross-task escalated 1 block-severity coherence finding"). A fixture where per-task PASSES but cross-task escalates a `block` closes Gate 3 as `passed-with-escalations`, not `passed`.
  - Trace: interview Round 4; DA-4 (closer precedence).
  - Serves goal: Outcome (single artefact, lens distinction visible, deterministic closer precedence) + Audience (admin reader navigates one manifest, sees both lenses + which drove status).

- **R9.** The cross-task reviewer reads each task's `context-budget-estimate` field (added by 029) as a soft signal for the better-arrangement lens. When two within-budget tasks have operationally similar scope and overlapping concerns, the reviewer surfaces a merge suggestion (`coherence` lens) noting the budget context (e.g. "T3 (60K) and T4 (65K); merging would exceed the 80K cap, so consider whether they're really one concept or two"). When a within-budget task has weak coherence in its own scope (large but operationally fragmented), the reviewer surfaces a split suggestion. The reviewer does NOT re-check 029 R4's hard-cap auto-flag (that fires at synthesis Phase B.4 and never reaches Gate 3). Budget is one input among several; coherence + arrangement remain the load-bearing lenses.
  
  **Hard-cap enforcement at the applier (per Codex pre-gate finding [MAJOR] F).** The applier MUST decline any merge whose combined `context-budget-estimate` would exceed the configured 029 per-task budget cap. **Phase F reads `config.task.contextBudget` from the active config at Phase F entry** (after the fresh applier invocation starts; never uses a value embedded in the Round-1 finding, tasks.md, or synthesis-time snapshot — per codex-r1-f5). Such mergers are flagged `scope-change-required` with rationale `merge would violate 029 R4 hard-cap (combined budget {N}K > cap {C}K) — re-synthesis required`. The applier never applies a merge that would breach the cap in place.
  - Trace: interview Round 10; Codex pre-gate finding [MAJOR] F; codex-r1-f5 (config read at Phase F execution time).
  - Serves goal: Outcome (budget-aware soft signal) + Driving value (029's hard-cap unchanged; cross-task respects the cap with runtime-current config).

- **R10.** Gate 3's existing per-task Round 3 sharpen pass (E.5) fires AFTER Phase E.4.5 closes. Per-task reviewers re-fire against the post-applier `{NNN-slug}-tasks.md`. **Per-task reviewer lenses, schemas, and response surfaces are unchanged** — only the artefact snapshot they review differs.
  
  **Ordering guarantee (per Codex pre-gate finding [MAJOR] E; surgical-r1-f5 trim).** Phase E.4.5.4 must complete writing the post-applier tasks.md before Phase E.5 starts. Phase E.5 reads the post-applier tasks.md as input; if the applier wrote an incomplete file, per-task reviewers surface the issue in their normal lens. (No B4-CHECK-FAILED-POST-APPLIER diagnostic surface — the applier inherits B.4 logic, and per-task reviewers catch any incompleteness anyway.)
  
  **Hybrid R3 sharpen surface (per tbc-r1-f5).** Per-task reviewers operate as a hybrid R3:
  - Sharpen R1 findings whose anchored T-id still exists in the post-applier tasks.md (standard R3 sharpen surface).
  - Treat R1 findings whose T-id was merged-out / dropped as auto-resolved — recorded in manifest as `resolved-by-cross-task-merge: T{N}->T{M}` or `resolved-by-cross-task-drop: T{N}` per finding.
  - Net-new findings on tasks introduced by the applier (e.g. a merged T3+T7 → new T3.5) are recorded as `round-3-net-new` per-task findings AND require a one-pass orchestrator response (no R4 sharpen — net-new findings are themselves an exit lever).
  
  **Assumption made explicit:** per-task lenses remain principle-valid across artefact transformation; they do not require re-baselining to the post-applier state.
  - Trace: interview Round 2; codex pre-gate [MAJOR] E; surgical-r1-f5 (B4 diagnostic dropped); tbc-r1-f5 (hybrid R3 surface).
  - Serves goal: Outcome (per-task lens catches downstream issues; behaviour unchanged per Non-goal; T-id mapping ambiguity resolved).

- **R11.** Add a worked-example fixture demonstrating the full Round 2.5 flow at `plugins/specflow/examples/docs/specflow/features/022-cross-task-review/fixtures/cross-task-worked-example.md`. The fixture shows a 4-task feature where: (a) a coherence-axis finding fires (e.g. "T2 and T3 both anchor R4 with overlapping scope — consider merging") AND (b) a better-arrangement finding fires (e.g. "T1 has weak coherence — its scope spans two distinct concerns; consider splitting"). The fixture demonstrates: applier accepts (a), rejects (b) with rationale; cross-task R3 sharpens (b) with new evidence; applier final pass accepts the sharpened (b); manifest closes with both findings under Accepted, both lenses represented, all three `agent_id` fields populated and distinct.
  - Trace: FEATURES.md § 022 Done when — *worked example shows both a coherence issue AND a better-arrangement suggestion surfaced + resolved.*
  - Serves goal: Outcome (Done-when criterion has a binary fixture) + Driving value (downstream skills can read the fixture as canonical reference).

- **R12.** `task/SKILL.md` total line count after the change set ≤500. Phase E.4.5 sub-phase + R1's role-def file + applier Phase F + R7's manifest-schema agent_id fields together inflate `task/SKILL.md` substantively. Per the **chain-don't-absorb pattern** documented in the user's `feedback_skill_size_ceiling` memory (skills ≤~500 lines; new behaviour becomes a new skill or chained doc, not bolted onto existing skill), extract the cross-task review contract — Phase E.4.5 mini-debate steps, the applier's Phase F operational shape, the manifest schema extension, the agent_id generation scheme — into a doctrine doc at `plugins/specflow/templates/task/cross-task-review.md`. The pattern is established by 029's `templates/admin/single-context-task.md` (shipped commit b2e4ee9) and 025's `templates/task/sprint-bucket-heuristic.md` (shipped this sprint). `task/SKILL.md` Phase E.4.5 carries one-line citations; the doctrine doc holds the operational details.
  - Trace: implementation-constraint trace per `feedback_skill_size_ceiling` user memory (skills ≤500 lines); 029's `templates/admin/single-context-task.md` precedent (commit b2e4ee9); 025's `templates/task/sprint-bucket-heuristic.md` precedent (Sprint 2). Codex pre-gate finding [MINOR] (R12 trace strengthening).
  - Serves goal: Driving value (skill-size ceiling honoured; doctrine doc absorbs detail).

- **R14.** **Downstream-numbering compatibility (simplified per simplicity-r1-f4 + surgical-r1-f1).** Inserting Phase E.4.5 between E.4 and E.5 is internal to `task/SKILL.md`. Two compatibility points only:
  - The cross-task doctrine doc at `templates/task/cross-task-review.md` carries one paragraph noting the scope-change re-fire interaction (when scope-change's E.6 delta-regeneration re-fires `specflow:task` Phase E, the re-fire now traverses cross-task review on the regenerated task list — acceptable by construction; not a scope-change-side edit).
  - `brief/SKILL.md` reads `tasks-gate3/manifest.md` per `brief/SKILL.md:81-83`; the brief composer renders manifest body verbatim, so the new "Cross-task findings" H2 section renders alongside per-task sections without code changes (compatibility-only verification, no edit).
  
  No `scope-change/SKILL.md` edit; no repo-wide grep audit.
  - Trace: Codex pre-gate finding [MAJOR] A; surgical-r1-f1 (drop scope-change edit); simplicity-r1-f4 (drop repo-wide audit).
  - Serves goal: Outcome (insertion is downstream-safe via doctrine-doc note).

- **R15.** *Reserved.* (R15 was the first-class mode entry-point requirement; per surgical-r1-f4 + simplicity-r1-f4, the flag is documented as a Phase A Resume-logic branch under R5's umbrella — not a public Inputs-section entry. The applier flag is an orchestrator-internal contract, not a user-facing entry point.)

- **R16.** **025 cross-cut — sprint-bucket recompute on accepted merge/split (per DA-3 / codex-r1-f6).** When the applier accepts a merge or split via Phase F:
  - **Merge** — the merged task's `sprint-bucket` is recomputed via 025's heuristic at `templates/task/sprint-bucket-heuristic.md` against the merged scope (NOT inherited from either parent). Recompute touches only the merged task's bucket; if the recompute would alter buckets outside the merged component (e.g. tasks downstream now re-leveled), the finding is flagged `scope-change-required` with rationale `merge bucket-recompute creates graph-wide bucket drift — re-synthesis required`.
  - **Split** — each child task re-runs the heuristic. Same scope-change-routing if recomputation creates graph-wide drift.
  - **Audit** — pre-edit and post-edit bucket values for each affected task ID are recorded in `cross-task-responses.json` per finding under a `bucket_audit` field.
  - Trace: DA-3 (silent 025 cross-cut); codex-r1-f6 (sprint-bucket lineage).
  - Serves goal: Outcome (022 + 025 contract gap closed; bucket integrity preserved across cross-task application).

- **R17.** **Sub-agent dispatch failure fallback (per DA-5).** If the cross-task reviewer's forked sub-agent OR the applier's Phase F invocation fails to return a valid finding/response JSON (network drop, harness crash, malformed JSON, missing finding file), Phase E.4.5 logs the failure to `debate-log/tasks-gate3/findings/round-{N}/cross-task-{role}.failure.json` with the failure mode + timestamp. The gate escalates as `passed-with-escalations` (NOT failed — cross-task is the additive lens; per-task review remains authoritative for the run). The manifest closer notes: *"Cross-task review unavailable: {reason}; per-task review remains authoritative for this run."* The user re-runs `specflow:task {NNN-slug}` to retry, OR proceeds with per-task-only review acknowledging the cross-task gap.
  - Trace: DA-5 (sub-agent dispatch failure semantics).
  - Serves goal: Outcome (graceful degradation; harness flakiness does not break Gate 3) + Driving value (per-task lens stays load-bearing; cross-task is best-effort additive).

- **R13.** Extend `task/SKILL.md` `eval:` field (line 20, current text *"tasks file exists with one task per PRD requirement; coverage matrix shows 100% PRD-requirement coverage and zero orphan tasks; every task acceptance is binary; Gate 3 debate manifest closes with Orchestrator sign-off entry; any user-driven recut wrote a record to task-history.json."*) with two clauses that exercise the new contract:
  1. When `{NNN-slug}-tasks.md` has 3+ tasks, the closing Gate 3 manifest contains a "Cross-task findings" H2 section with at least one of `Accepted findings`, `Rejected findings`, `Escalated to human`, or `Cross-task review skipped:` (the latter only when task count <3).
  2. The manifest's `agent_id` triplet (`writer_id`, `cross_task_reviewer_id`, `applier_id`) is populated AND all three differ.
  - Trace: existing `task/SKILL.md:20` `eval:` field shape (the binary-success contract every skill carries per `rules/non-negotiable.md` `TESTS_REQUIRED_FOR_VERIFIABLE_SKILLS`). Without these clauses, R7 + R8 are not exercised by the existing eval — synthesis could pass on a tasks file that omits the new manifest section entirely. Codex pre-gate finding [MAJOR] R13 trace.
  - Serves goal: Outcome (eval enumerates the new contract).

## Acceptance criteria

- **AC-1.** New reviewer role-def file exists at `plugins/specflow/examples/docs/specflow/admin/agents/standard/principles/cross-task-reviewer.md` and documents the two lenses (coherence + better-arrangement) verbatim:
  ```sh
  test -f plugins/specflow/examples/docs/specflow/admin/agents/standard/principles/cross-task-reviewer.md
  grep -qE '^## Coherence|coherence lens' plugins/specflow/examples/docs/specflow/admin/agents/standard/principles/cross-task-reviewer.md
  grep -qE '^## Better arrangement|better-arrangement lens' plugins/specflow/examples/docs/specflow/admin/agents/standard/principles/cross-task-reviewer.md
  grep -qE '"lens".*"coherence|better-arrangement"' plugins/specflow/examples/docs/specflow/admin/agents/standard/principles/cross-task-reviewer.md
  ```
  - Verifies: R1.

- **AC-2.** `task/SKILL.md` Phase E lists a new sub-phase E.4.5 between E.4 (Round 2) and E.5 (Round 3). The sub-phase header AND threshold gate are present:
  ```sh
  grep -qE '^### E\.4\.5 — Cross-task review' plugins/specflow/skills/task/SKILL.md
  grep -qE 'task count.*<.*3|fewer than 3 tasks|3 or more tasks' plugins/specflow/skills/task/SKILL.md
  ```
  - Verifies: R2.

- **AC-3.** Phase E.4.5 documents the four sub-steps (E.4.5.1 / E.4.5.2 / E.4.5.3 / E.4.5.4) — exactly four, in canonical order:
  ```sh
  grep -nE '^#### E\.4\.5\.[1-4]' plugins/specflow/skills/task/SKILL.md > /tmp/sub-steps
  [ "$(wc -l < /tmp/sub-steps)" -eq 4 ]
  python3 -c "
  import re
  lines = open('/tmp/sub-steps').read().splitlines()
  pairs = [(int(re.match(r'(\d+):', l).group(1)), re.search(r'E\.4\.5\.(\d)', l).group(1)) for l in lines]
  suffixes = [p[1] for p in sorted(pairs, key=lambda p: p[0])]
  assert suffixes == ['1', '2', '3', '4'], f'expected sub-steps 1,2,3,4 in order, got {suffixes}'
  "
  ```
  - Verifies: R3.

- **AC-4.** `task/SKILL.md` Phase E.4 (orchestrator response) body does NOT reference cross-task findings — the original orchestrator never responds to or auto-applies cross-task feedback. Verified by extracting the E.4 body strictly excluding the E.4.5 header line that bounds it:
  ```sh
  awk '/^### E\.4 /,/^### E\.4\.5/' plugins/specflow/skills/task/SKILL.md | head -n -1 > /tmp/e4-body
  ! grep -qE 'cross-task|cross_task' /tmp/e4-body
  ```
  (`head -n -1` strips the terminating `### E.4.5` line so the AC tests only the E.4 body, not the boundary line.)
  - Verifies: R4.

- **AC-5.** `task/SKILL.md` documents the new `--apply-cross-task-feedback {NNN-slug}` flag and the new Phase F:
  ```sh
  grep -qE -- '--apply-cross-task-feedback' plugins/specflow/skills/task/SKILL.md
  grep -qE '^## Phase F — Cross-task feedback application' plugins/specflow/skills/task/SKILL.md
  ```
  Phase F lists the three inputs (post-Round-2 tasks.md; cross-task R1 findings JSON; PRD/interview). Awk extraction uses the strict next-heading boundary idiom (per codex-r1-f2 — the `/^## Phase F/,/^## /` range pattern self-defeats):
  ```sh
  awk 'found && /^## /{exit} /^## Phase F/{found=1} found{print}' plugins/specflow/skills/task/SKILL.md > /tmp/phase-f
  grep -qE 'tasks.md' /tmp/phase-f
  grep -qE 'cross-task-reviewer\.json' /tmp/phase-f
  grep -qE 'PRD|prd\.md' /tmp/phase-f
  ```
  - Verifies: R5.

- **AC-6.** The applier's response schema documents the three decision values: `accepted`, `rejected`, `scope-change-required` (using the strict next-heading awk idiom):
  ```sh
  awk 'found && /^## /{exit} /^## Phase F/{found=1} found{print}' plugins/specflow/skills/task/SKILL.md > /tmp/phase-f
  grep -qE '"decision":\s*"accepted"' /tmp/phase-f
  grep -qE '"decision":\s*"rejected"' /tmp/phase-f
  grep -qE '"decision":\s*"scope-change-required"' /tmp/phase-f
  ```
  And explicitly forbids the applier from modifying the PRD coverage matrix on the fly:
  ```sh
  grep -qE 'scope-change-required.*MUST|coverage-changing.*scope-change-required' /tmp/phase-f
  ```
  - Verifies: R6.

- **AC-7.** Manifest schema (documented in Phase E.6 closer) gains three `agent_id` fields:
  ```sh
  awk '/^### E\.6/,/^### |^## /' plugins/specflow/skills/task/SKILL.md > /tmp/closer
  grep -qE 'writer_id' /tmp/closer
  grep -qE 'cross_task_reviewer_id' /tmp/closer
  grep -qE 'applier_id' /tmp/closer
  grep -qE 'FRESH-CONTEXT-VIOLATION' /tmp/closer
  ```
  - Verifies: R7.

- **AC-8.** Closing Gate 3 manifest template includes a dedicated "Cross-task findings" H2 section with three H3 sub-sections:
  ```sh
  awk '/manifest\.md.*template|^## Cross-task findings/,/^## |^---/' plugins/specflow/skills/task/SKILL.md > /tmp/manifest-template
  grep -qE '^## Cross-task findings' /tmp/manifest-template
  grep -qE '^### Accepted findings' /tmp/manifest-template
  grep -qE '^### Rejected findings' /tmp/manifest-template
  grep -qE '^### Escalated to human' /tmp/manifest-template
  ```
  And per-finding entry shape includes a `lens` field:
  ```sh
  grep -qE 'lens.*coherence|better-arrangement' /tmp/manifest-template
  ```
  - Verifies: R8.

- **AC-9.** Cross-task reviewer reads `context-budget-estimate` per task. The role-def file documents this:
  ```sh
  grep -qE 'context-budget-estimate' plugins/specflow/examples/docs/specflow/admin/agents/standard/principles/cross-task-reviewer.md
  grep -qE 'within-budget|hard-cap.*029.*R4' plugins/specflow/examples/docs/specflow/admin/agents/standard/principles/cross-task-reviewer.md
  ```
  - Verifies: R9.

- **AC-10.** Per-task Round 3 sharpen (E.5) fires AFTER Phase E.4.5 closes. The post-E.4.5 tasks.md is the artefact per-task reviewers sharpen on:
  ```sh
  grep -nE '^### E\.4\.5|^### E\.5' plugins/specflow/skills/task/SKILL.md | awk -F: '{print $1, $2}' > /tmp/order-check
  python3 -c "
  lines = [l.split() for l in open('/tmp/order-check')]
  e45 = next(int(l[0]) for l in lines if 'E.4.5' in l[1])
  e5 = next(int(l[0]) for l in lines if 'E.5' in l[1] and 'E.4.5' not in l[1])
  assert e45 < e5, 'E.4.5 must precede E.5'
  "
  ```
  - Verifies: R10.

- **AC-11.** Worked-example fixture exists at `plugins/specflow/examples/docs/specflow/features/022-cross-task-review/fixtures/cross-task-worked-example.md` and demonstrates both lenses + applier accept/reject + sharpen + final pass:
  ```sh
  test -f plugins/specflow/examples/docs/specflow/features/022-cross-task-review/fixtures/cross-task-worked-example.md
  fix=plugins/specflow/examples/docs/specflow/features/022-cross-task-review/fixtures/cross-task-worked-example.md
  grep -qE 'lens.*coherence' "$fix"
  grep -qE 'lens.*better-arrangement' "$fix"
  grep -qE 'decision.*accepted' "$fix"
  grep -qE 'decision.*rejected' "$fix"
  grep -qE 'sharpen|round-3' "$fix"
  grep -qE 'writer_id|cross_task_reviewer_id|applier_id' "$fix"
  ```
  - Verifies: R11.

- **AC-12.** Doctrine doc exists at `plugins/specflow/templates/task/cross-task-review.md` and `task/SKILL.md` Phase E.4.5 cites it at least once:
  ```sh
  test -f plugins/specflow/templates/task/cross-task-review.md
  awk '/^### E\.4\.5/,/^### E\.5/' plugins/specflow/skills/task/SKILL.md > /tmp/e45-block
  grep -qE 'templates/task/cross-task-review\.md' /tmp/e45-block
  ```
  And `task/SKILL.md` line count after the change set ≤500:
  ```sh
  [ "$(wc -l < plugins/specflow/skills/task/SKILL.md)" -le 500 ]
  ```
  - Verifies: R12.

- **AC-13.** `task/SKILL.md` `eval:` field is extended per R13:
  ```sh
  awk '/^eval:/' plugins/specflow/skills/task/SKILL.md > /tmp/eval-line
  grep -qE 'Cross-task findings|cross-task' /tmp/eval-line
  grep -qE 'agent_id|writer_id.*cross_task_reviewer_id' /tmp/eval-line
  ```
  - Verifies: R13.

- **AC-14.** Cross-task review skipped on a 1-task or 2-task feature. Fixture path is explicit:
  ```sh
  FIX=plugins/specflow/examples/docs/specflow/features/022-cross-task-review/fixtures/threshold-skip-2-tasks/
  test -f "${FIX}tasks.md"
  test -f "${FIX}debate-log/tasks-gate3/manifest.md"
  grep -qE 'Cross-task review skipped: task count [12] below threshold \(3\)' "${FIX}debate-log/tasks-gate3/manifest.md"
  # NO cross-task finding files in any of the three rounds
  ! ls "${FIX}debate-log/tasks-gate3/findings/round-1/cross-task-reviewer.json" 2>/dev/null
  ! ls "${FIX}debate-log/tasks-gate3/findings/round-2/cross-task-responses.json" 2>/dev/null
  ! ls "${FIX}debate-log/tasks-gate3/findings/round-3/cross-task-reviewer.json" 2>/dev/null
  ```
  - Verifies: R2 (threshold gate).

- **AC-15.** Fresh-context audit signal verified in the happy-path fixture: writer / reviewer / applier `agent_id` triplet populated AND all three differ. The collision-detection unit-fixture path has been dropped (per codex-r1-f1 / surgical-r1-f2 / simplicity-r1-f2 simplification — no closer-side collision check in 022; 027 owns runtime verification). The audit signal is field-presence + non-collision in the green path:
  ```sh
  FIX=plugins/specflow/examples/docs/specflow/features/022-cross-task-review/fixtures/cross-task-worked-example/
  python3 -c "
  import re, sys
  m = open('${FIX}debate-log/tasks-gate3/manifest.md').read()
  ids = {}
  for k in ['writer_id', 'cross_task_reviewer_id', 'applier_id']:
      match = re.search(rf'\\*\\*{k}:\\*\\*\\s+(\\S+)', m)
      assert match, f'{k} missing from manifest'
      ids[k] = match.group(1)
  assert len(set(ids.values())) == 3, f'agent_id collision: {ids}'
  "
  ```
  - Verifies: R7 (best-effort manifest field; format unspecified; 027 owns runtime verification).

- **AC-16.** **Downstream-numbering compatibility checks** (per R14):
  ```sh
  # No skill outside specflow:task greps for ^### E\.5 or 'Round 3' as skill-internal phase markers
  for skill in plugins/specflow/skills/*/SKILL.md; do
    if [ "$skill" = "plugins/specflow/skills/task/SKILL.md" ]; then continue; fi
    ! grep -qE '\^### E\\\.5|tasks-gate3.*Round 3' "$skill"
  done
  # brief/SKILL.md still reads tasks-gate3/manifest.md (compatibility preserved)
  grep -qE 'tasks-gate3/manifest\.md' plugins/specflow/skills/brief/SKILL.md
  # scope-change/SKILL.md's E.6 re-fire path is documented as cross-task-aware
  grep -qE 'cross-task|E\.4\.5' plugins/specflow/skills/scope-change/SKILL.md || \
    grep -qE 'cross-task|E\.4\.5' plugins/specflow/templates/task/cross-task-review.md
  ```
  - Verifies: R14.

- **AC-17.** **First-class mode entry-point** (per R15):
  ```sh
  # Inputs section documents the new flag
  awk '/^## Inputs/,/^## /' plugins/specflow/skills/task/SKILL.md > /tmp/inputs
  grep -qE -- '--apply-cross-task-feedback' /tmp/inputs
  # Phase A's Resume logic carries an --apply-cross-task-feedback branch
  awk '/^## Phase A/,/^## Phase B/' plugins/specflow/skills/task/SKILL.md > /tmp/phase-a
  grep -qE -- '--apply-cross-task-feedback' /tmp/phase-a
  grep -qE 'jump to Phase F|skip Phase A.*Phase F' /tmp/phase-a
  ```
  - Verifies: R15.

## Open questions

None — all questions resolved during grilling. (The Topics-not-discussed entries — reviewer role-def location, lens-checklist sub-questions, manifest closing-decision passthrough logic, 028 interaction, cross-feature batching, Linear cycle integration, auto-apply without applier, per-feature config opt-out — are either out-of-scope per goal-level or PRD-derivable from the resolved rounds.)

## See also

- Interview: [`./022-cross-task-review-interview.md`](./022-cross-task-review-interview.md)
- Doctrine doc: [`../../templates/task/cross-task-review.md`](../../templates/task/cross-task-review.md) (created by this PRD; canonical home for the Phase E.4.5 + Phase F operational details)
- Reviewer role-def: [`../../admin/agents/standard/principles/cross-task-reviewer.md`](../../admin/agents/standard/principles/cross-task-reviewer.md) (created by this PRD)
- Worked example fixture: [`./fixtures/cross-task-worked-example.md`](./fixtures/cross-task-worked-example.md) (created by this PRD)
- Tasks: [`./022-cross-task-review-tasks.md`](./022-cross-task-review-tasks.md) (generated by `specflow:task`)
- Tests: [`./022-cross-task-review-test.md`](./022-cross-task-review-test.md) (generated by `specflow:test`)
