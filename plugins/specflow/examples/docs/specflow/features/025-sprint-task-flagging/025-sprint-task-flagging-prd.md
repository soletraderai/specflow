---
feature: 025-sprint-task-flagging
status: draft
created: 2026-05-07
interview: ./025-sprint-task-flagging-interview.md
---

# Sprint Task Flagging

## Vision

`specflow:task` synthesis writes a single integer `sprint-bucket: N` field on every per-task entry, derived deterministically from the dependency graph and scope-overlap. Bucket 1 always means "no predecessors"; bucket 2 means "depends only on bucket 1"; etc. The heuristic is documented in `plugins/specflow/templates/task/sprint-bucket-heuristic.md` (chain-don't-absorb, mirroring 029's doctrine doc pattern); `task/SKILL.md` Phase B.4 carries one-line citations. The field is reproducible — given the same dependency graph, scope surfaces, and task IDs, two synthesis runs produce identical bucket assignments — and the future sprint-skill (020) reads it without recomputing topology. The field is read-only at the documentation level; the skill does not police hand-edits.

## Problem

`specflow:task` § Phase B writes per-task entries with `Anchor / Scope / Acceptance / Depends on / context-budget-estimate / Notes` (post-029). 020-sprint-skill (the planner) needs to know which tasks can be batched into a sprint together — i.e. which tasks share a topological layer with no shared `Depends on:` edges and disjoint `Scope:` surfaces, fitting under the project's per-task context budget. Today, 020 would have to recompute that grouping every time it runs by traversing `Depends on:` edges and inferring scope overlap — duplicating logic that lives more naturally at synthesis time, where the dependency graph and per-task budgets are already in hand.

029's § Implications #3 explicitly anticipates this feature: *"Tasks in the same bucket don't share context (each runs in its own window), but the bucket sizing is informed by per-task budgets so the developer doesn't approve a sprint where any single task can't fit."* The bucket field is the deterministic artefact passed from `specflow:task` to `specflow:sprint`, in the same shape as the deterministic-node-passes-artefact-to-coding-agent-node pattern from `knowledge/medin-archon-livestream.md` § Hybrid Secret.

## Goals

- Add a single integer field — `sprint-bucket: N` — to the per-task synthesis schema in `task/SKILL.md` Phase B.
- Host the bucket-assignment heuristic in `plugins/specflow/templates/task/sprint-bucket-heuristic.md` (mirrors 029's `templates/admin/single-context-task.md` chain-don't-absorb pattern); `task/SKILL.md` adds one-line citations from B.3 and B.4.
- Reject malformed dependency graphs (cycles, self-loops, dangling references, duplicate task IDs) with a deterministic `GRAPH-INVALID:` diagnostic before bucket assignment runs.
- Respect 029's per-task budget: tasks whose `context-budget-estimate` exceeds the configured budget are auto-flagged for split (per 029 R4) before bucketing operates.
- Scope bucket numbers to the feature: bucket 1 always means "no predecessors" within the feature.
- Keep `task/SKILL.md` at or under 500 lines after additions, per the skill-size ceiling. The heuristic + worked example land in the doctrine doc, not the skill body.

## Non-goals

- Building the sprint-skill itself (020). 025 produces the field; 020 consumes it. (Goal-level out-of-scope.)
- Enforcing a sprint-wide context budget cap (per 029 § Non-goals).
- Auto-promoting bucket numbers into Linear cycle assignments. Deferred to 020.
- Cross-feature bucket sharing. Cross-feature sprint planning is parked under the existing parked-memory entry; bucket numbers are feature-scoped.
- DAG visualisation. Visualisation belongs to the sprint-plan gate (020), not the synthesis-time field.
- Reserved bucket numbers (e.g. `sprint-bucket: 0` for setup tasks). Bucket 1 is the floor.
- Auto-detecting the optimal number of buckets vs accepting whatever the topology produces. **Whether this lands in 022 or a future feature is a separate planning decision** — FEATURES.md § 022 currently scopes 022 as task-merge / split / re-order, NOT bucket-count tuning. (Per DA-2.)
- **Hand-edit detection**. The field is documented as read-only (R10), but `specflow:task` does NOT recompute-and-compare on every synthesis run to police hand-edits. AC-11 verifies the documentation; runtime enforcement is out of scope. (Per simplicity-r1-f1, surgical-r1-f1, DA-r1-f1.)

## Users

- **Power user — operations coordinator.** Indirect. The sprint-skill (020) consumes the field to plan parallel fan-out batches; power users benefit from the resulting parallelisation.
- **Admin — platform operator.** Reads the bucket assignments at the sprint-plan gate to audit whether the topological grouping makes sense.
- **Engineer using the task skill.** Direct user. Sees `sprint-bucket: N` on each task at synthesis time; can review the grouping before tasks land in `{NNN-slug}-tasks.md`.

**Downstream consumers (informational, NOT contract-shaping stakeholders).** `specflow:sprint` (020, future) reads `sprint-bucket: N` to plan a fan-out batch. 025 writes the field per R1-R12; how 020 reads it is 020's concern. Listing 020 here is informational and does not give it contract authority over 025's surface. (Per DA-r1-f4.)

## Requirements

- **R1.** Add `sprint-bucket: N` (where `N` is a positive integer ≥ 1) as a per-task field in `task/SKILL.md` Phase B.3 write template (currently lines 174-186 — the `- **field:** value` markdown bullet pattern). The field sits alongside the existing `Anchor / Scope / Acceptance / Depends on / context-budget-estimate / Notes`.
  - Trace: codebase context bullet 1; sign-off — single integer field per task.
  - Serves goal: Outcome (the field exists) + Audience (sprint-skill consumes it).

- **R2.** Document the bucket-assignment heuristic in `plugins/specflow/templates/task/sprint-bucket-heuristic.md` (chain-don't-absorb mirror of 029's `templates/admin/single-context-task.md`). The heuristic uses a single-rule fixpoint definition:

  **Canonical task ID grammar.** Task IDs follow `T-<int>(\.<letter>)?` per `task/SKILL.md` Phase B convention (e.g. `T-1`, `T-2`, `T-3.A`, `T-3.B` per 029's split flow). Task IDs are unique within a feature; bucket assignment relies on within-feature ID uniqueness.

  **Typed comparator.** Same-bucket scope-overlap conflicts are resolved by a typed comparator on `(int, optional-letter)` over the ID's grammar — NOT raw string lexicographic. `T-2` < `T-10` (natural-number ordering); `T-3.A` < `T-3.B` (letter ordering after equal int).

  **Single-rule definition (recursive).**
  ```
  bucket(T) = 1 if T has no predecessors AND no same-bucket scope conflicts with an earlier-ID task;
            = 1 + max(bucket(P) for P in Depends-on(T) ∪ EarlierIDSameBucketScopeConflicts(T)) otherwise.
  ```

  **Bump iteration discipline.** Bumps applied iteratively until no same-bucket scope overlap remains. In each pass, walk same-bucket task pairs in `(bucket-asc, T-id-asc, T-id-asc)` order under the typed comparator and bump the later-ID partner: `bucket(T_j) := bucket(T_j) + 1`. Repeat passes until a pass produces zero bumps (fixed point). Termination is guaranteed: each bump strictly increases bucket numbers; the topological-floor invariant bounds the maximum.

  **Topological-floor corollary** (by construction): for every `T_j ∈ Depends-on(T_i)`, `bucket(T_j) < bucket(T_i)` — strictly less-than (per codex-r1-f5). The heuristic does NOT need a separate runtime invariant check.

  **Per-task budget respect.** Tasks whose `context-budget-estimate` exceeds the configured budget are auto-flagged for split at synthesis (per 029 R4) and never reach bucket assignment.
  - Trace: interview Round 1 + Round 2; simplicity-r1-f2 (single-rule); TBC-2 (typed comparator); TBC-5 (iteration discipline); codex-r1-f5 (strict less-than).
  - Serves goal: Outcome (heuristic is documented and reproducible) + Driving value (deterministic field).

- **R3.** `task/SKILL.md` Phase B.4 (Self-check, where 029's budget check already lives at line 208) gains two one-line citations: one to `templates/task/sprint-bucket-heuristic.md` for the heuristic, one to the same doc for graph-validity. No new "Phase D verify-steps" section is created — verify lives in B.4 + the existing 'Verify before declaring done' tail (line 460).
  - Trace: DA-r1-f3 (Phase D doesn't exist; B.4 is the actual home); DA-r1-f5 (chain-don't-absorb).
  - Serves goal: Outcome (heuristic is invoked at the existing verify location).

- **R4.** Bucket numbering starts at 1. Bucket 1 means "no predecessors" within the feature. There is no bucket 0; setup tasks (if present) land in bucket 1 by virtue of having no predecessors.
  - Trace: interview Round 3.
  - Serves goal: Outcome (deterministic numbering convention).

- **R5.** Bucket numbers are scoped to the feature. Two features whose tasks could run in parallel do NOT share bucket numbers.
  
  **Hypothesis tag** (per TBC-4): per the current 020 design hypothesis (FEATURES.md § 020 — open design), one Linear project maps to one specflow feature. If 020's mapping evolves (e.g. one feature → many Linear projects, or many features → one project), 025's feature-scoping should be re-examined.
  - Trace: interview Round 3; TBC-4 (hypothesis tag).
  - Serves goal: Outcome (feature-scoped numbering) + Driving value (avoids smuggling higher-level orchestration into 025).

- **R6.** Bucket count is uncapped. A feature with a deep dependency graph may legitimately have many buckets. The heuristic does not tune the bucket count; it accepts whatever the topology produces.
  - Trace: interview Round 1.
  - Serves goal: Outcome.

- **R7.** Bucket assignment runs *after* per-task budget enforcement (029 R4). Property statement: tasks whose `context-budget-estimate` exceeds the configured budget are split per 029 R4 before bucketing operates. The heuristic doc states this explicitly; no verbatim phase-listing requirement.
  - Trace: codebase context bullet 3; SURG-r1-f2 (property, not control flow).
  - Serves goal: Driving value (029's per-task budget invariant preserved).

- **R8.** `task/SKILL.md` total line count after the change set ≤500. The heuristic + worked example are extracted into `plugins/specflow/templates/task/sprint-bucket-heuristic.md` (chain-don't-absorb). `task/SKILL.md` additions are limited to:
  - One field-emission line in B.3 write template (`- **sprint-bucket:** N`).
  - One one-line citation in B.3 referencing the heuristic doc.
  - One one-line citation in B.4 self-check section.
  - The heuristic, the worked example, the typed-comparator grammar, the iteration discipline, and the graph-validity diagnostic format ALL live in the doctrine doc — NOT in the skill body. **No prose-tightening of unrelated sections.**
  - Trace: codebase context bullet 6; DA-r1-f5; SURG-r1-f5.
  - Serves goal: Driving value (skill-size ceiling; chain-don't-absorb).

- **R9.** *Reserved.* (R9 was the topological-floor invariant verify-step in earlier drafts; dropped per simplicity-r1-f3 / SURG-r1-f3 — R2's rule guarantees the floor by construction.)

- **R10.** The `sprint-bucket` field is documented as read-only for downstream consumers; editing the field by hand changes the topological grouping and is a `specflow:scope-change` concern. **Documentation-level only** — `task/SKILL.md` does NOT recompute-and-compare on every synthesis run. AC-11 verifies the documentation does not advertise hand-edit affordances; runtime enforcement is out of scope (Non-goals).
  - Trace: interview Round 2; simplicity-r1-f1 + SURG-r1-f1 + DA-r1-f1 (drop runtime enforcement).
  - Serves goal: Outcome (deterministic artefact) + Driving value (avoids circular diagnostic with `specflow:scope-change` delta-regeneration).

- **R11.** **Graph validity (pre-bucket).** Before bucket assignment, `task/SKILL.md` Phase B.4 invokes a graph-validity step (cited from `templates/task/sprint-bucket-heuristic.md`) that rejects malformed dependency graphs with a deterministic diagnostic. Failure modes and diagnostics:
  - Cycle (any strongly-connected component of size >1) → `GRAPH-INVALID: cycle: T-X -> T-Y -> ... -> T-X` (lists the cycle members in traversal order).
  - Self-loop (`T.depends_on` contains `T`) → `GRAPH-INVALID: self-loop on T-X`.
  - Duplicate task IDs → `GRAPH-INVALID: duplicate task ID T-X`.
  - Duplicate dependency edge (`T.depends_on` contains `T-Y` more than once) → `GRAPH-INVALID: duplicate edge T-X -> T-Y`.
  - Dangling reference (a `Depends on:` ID not present in the feature) → `GRAPH-INVALID: T-X.depends_on references unknown task T-Y`.

  Diagnostic prefix is verbatim `GRAPH-INVALID:` followed by the subtype. Synthesis aborts before any `sprint-bucket: N` is written; the user is pointed to `specflow:scope-change` for legitimate dependency-graph edits.
  - Trace: TBC-r1-f1; codex-r1-f1; codex-r3-f1 (duplicate-edge case); goal-driven-r1-f5.
  - Serves goal: Outcome (heuristic well-defined on all input shapes) + Driving value (errors are deterministic and actionable).

- **R12.** **Eval extension.** Extend `task/SKILL.md` `eval:` field (line 20) with two clauses:
  1. Every task entry in `{NNN-slug}-tasks.md` carries a `- **sprint-bucket:** N` field with `N >= 1`.
  2. The strict topological-floor corollary holds: for every task `T_i` with `sprint-bucket: N`, every `T_j` listed in `T_i.Depends on:` has `sprint-bucket: M` with `M < N` (strictly less-than, per R2 corollary).

  Verify mechanism: a parser pass over the synthesised tasks file emits zero violations.
  - Trace: GD-r1-f1 (mandatory eval-coverage); codex-r1-f5 (strict-less-than).
  - Serves goal: Outcome (the eval is the binary contract; without these clauses, AC-1 through AC-15 can pass on a tasks file that omits the field).

## Acceptance criteria

- **AC-1.** `task/SKILL.md` Phase B.3 write template includes the `- **sprint-bucket:** {int}` field as a sibling of the existing fields:
  ```sh
  grep -qE '\*\*sprint-bucket:\*\*' plugins/specflow/skills/task/SKILL.md
  ```
  - Verifies: R1.

- **AC-2.** `task/SKILL.md` Phase B.4 contains one-line citations to `templates/task/sprint-bucket-heuristic.md` (heuristic + graph-validity), and Phase B.3 cites the heuristic doc once:
  ```sh
  [ "$(grep -c 'templates/task/sprint-bucket-heuristic.md' plugins/specflow/skills/task/SKILL.md)" -ge 3 ]
  ```
  - Verifies: R3.

- **AC-3.** A worked example in `templates/task/sprint-bucket-heuristic.md` shows three tasks: T-1 with no predecessors → `sprint-bucket: 1`; T-2 with `Depends on: T-1` → `sprint-bucket: 2`; T-10 with no `Depends on:` and disjoint `Scope:` from T-1 → `sprint-bucket: 1` (parallelism); plus a fourth task T-3 with overlapping scope to T-1 → bumped to `sprint-bucket: 2` (typed-comparator tie-break, T-1 < T-3 by integer suffix). The example also demonstrates `T-2 < T-10` ordering (typed comparator, NOT string lex):
  ```sh
  test -f plugins/specflow/templates/task/sprint-bucket-heuristic.md
  grep -qE 'T-1.*sprint-bucket: 1' plugins/specflow/templates/task/sprint-bucket-heuristic.md
  grep -qE 'T-2.*sprint-bucket: 2' plugins/specflow/templates/task/sprint-bucket-heuristic.md
  grep -qE 'T-10.*sprint-bucket: 1' plugins/specflow/templates/task/sprint-bucket-heuristic.md
  grep -qE 'T-2 (<|before) T-10' plugins/specflow/templates/task/sprint-bucket-heuristic.md
  ```
  - Verifies: R2 (heuristic + comparator + worked example).

- **AC-4.** *Reserved.* (AC-4 was the standalone topological-floor invariant verify-step; dropped per simplicity-r1-f3 — R2's rule guarantees the property by construction; R12's eval clause 2 is the runtime check.)

- **AC-5.** A task whose `context-budget-estimate` exceeds the configured budget triggers the existing 029 R4 split flow *before* bucket assignment. Verified by fixture:
  ```sh
  # Fixture: a task whose budget estimate exceeds config.task.taskBudget; with one Depends-on edge.
  specflow:task NNN-slug; rc=$?
  # Synthesis aborts with the budget-overrun warning before writing sprint-bucket on the oversize task:
  grep -qE 'budget.*exceeds' stderr.log
  # The resulting tasks file (if any) does not have sprint-bucket on the oversize task:
  ! grep -E 'T-OVERSIZE.*sprint-bucket' "{NNN-slug}-tasks.md"
  ```
  - Verifies: R7.

- **AC-6.** No task in any feature has `sprint-bucket: 0`:
  ```sh
  ! grep -rE '\*\*sprint-bucket:\*\* 0\b' docs/specflow/features/
  ```
  - Verifies: R4.

- **AC-7.** Two features synthesised independently each carry their own bucket numbering. Positive structural assertion: no shared file under `plugins/specflow/skills/` references both feature IDs alongside a `sprint-bucket`:
  ```sh
  # featureA = 016, featureB = 017 (illustrative)
  ! grep -rlE '(016-brief-enhancements|017-tdd-discipline).*sprint-bucket' plugins/specflow/skills/
  ```
  - Verifies: R5.

- **AC-8.** `task/SKILL.md` line count after the change set is ≤500:
  ```sh
  [ "$(wc -l < plugins/specflow/skills/task/SKILL.md)" -le 500 ]
  ```
  - Verifies: R8.

- **AC-9.** Two synthesis runs of `specflow:task` over the same input PRD produce identical `sprint-bucket: N` assignments. Verified via markdown-aware extraction:
  ```sh
  python3 -c '
  import re, sys
  def extract(path):
      s = open(path).read()
      pairs = re.findall(r"### (T-\d+(?:\.[A-Z])?)[\s\S]*?- \*\*sprint-bucket:\*\* (\d+)", s)
      # typed comparator sort: (int, optional-letter)
      def key(p):
          m = re.match(r"T-(\d+)(?:\.([A-Z]))?", p[0])
          return (int(m.group(1)), m.group(2) or "")
      return sorted(pairs, key=key)
  a, b = extract("run-1.md"), extract("run-2.md")
  sys.exit(0 if a == b else 1)
  '
  ```
  Edge: a task whose `Notes:` field literally contains the string `sprint-bucket:` does NOT pollute the diff because the regex anchors on the markdown bullet `- **sprint-bucket:**` AND on the heading `### T-<id>`.
  - Verifies: R2 (reproducibility), R12 (consistent across runs).

- **AC-10.** `task/SKILL.md` does NOT document auto-tuning, auto-detection, or DAG visualisation. Verified via closed-token-list grep, scoped to outside the explicit Non-goals call-out:
  ```sh
  grep -nE -w 'auto-tune|auto-detect|DAG|visuali[sz]ation|render' plugins/specflow/skills/task/SKILL.md > /tmp/matches
  # All matches must be inside an explicit `<!-- non-goal -->` marker block (or zero matches)
  python3 -c "
  import sys, re
  body = open('plugins/specflow/skills/task/SKILL.md').read()
  # Identify all matches and confirm each is inside a Non-goal block
  for line_no, line_text in [(int(l.split(':')[0]), l.split(':',1)[1]) for l in open('/tmp/matches').read().splitlines() if l]:
      pre = '\n'.join(body.splitlines()[:line_no])
      if pre.rfind('<!-- non-goal -->') < pre.rfind('<!-- /non-goal -->'):
          print(f'leak at line {line_no}: {line_text}'); sys.exit(1)
  "
  ```
  - Verifies: R6 + Non-goals (DAG viz, auto-tuning).

- **AC-11.** `task/SKILL.md` does not document hand-editing of `sprint-bucket: N`. Verified by grep absence:
  ```sh
  ! grep -qE 'edit.*sprint-bucket|hand-edit|manually set sprint-bucket' plugins/specflow/skills/task/SKILL.md
  ```
  - Verifies: R10.

- **AC-12.** *Reserved.* (AC-12 was the recompute-and-compare verify-step for hand-edit detection; dropped per simplicity-r1-f1 / SURG-r1-f1 / DA-r1-f1.)

- **AC-13.** Phase ordering — bucket assignment after budget check — is verified by both documentation AND an execution-order fixture:
  - **Doc** — `templates/task/sprint-bucket-heuristic.md` § Per-task budget respect documents the ordering as a property:
    ```sh
    grep -qE 'budget.*before.*bucket|029 R4.*before' plugins/specflow/templates/task/sprint-bucket-heuristic.md
    ```
  - **Execution** — fixture per AC-5 confirms the abort happens before any `sprint-bucket:` line is written for the oversize task.
  - Verifies: R7.

- **AC-14.** Graph-validity step rejects malformed graphs with the verbatim `GRAPH-INVALID:` prefix per R11. One fixture per case:
  ```sh
  # Self-loop fixture
  specflow:task NNN-slug-self-loop; grep -qE '^GRAPH-INVALID: self-loop on T-' stderr.log
  # Cycle fixture
  specflow:task NNN-slug-cycle; grep -qE '^GRAPH-INVALID: cycle: T-' stderr.log
  # Duplicate-ID fixture
  specflow:task NNN-slug-dup; grep -qE '^GRAPH-INVALID: duplicate task ID T-' stderr.log
  # Duplicate-edge fixture (per codex-r3 sharpen)
  specflow:task NNN-slug-dup-edge; grep -qE '^GRAPH-INVALID: duplicate edge T-.* -> T-' stderr.log
  # Dangling-ref fixture
  specflow:task NNN-slug-dangle; grep -qE '^GRAPH-INVALID: T-.*references unknown task T-' stderr.log
  ```
  Each fixture lives at `plugins/specflow/examples/docs/specflow/features/025-sprint-task-flagging/fixtures/<case>.md`.
  - Verifies: R11.

- **AC-15.** `task/SKILL.md` `eval:` field is extended per R12:
  ```sh
  awk '/^eval:/' plugins/specflow/skills/task/SKILL.md > /tmp/eval-line
  grep -qE 'sprint-bucket' /tmp/eval-line
  grep -qE 'topological-floor|sprint-bucket: M.*M < N' /tmp/eval-line
  ```
  - Verifies: R12.

## Open questions

None — all questions resolved during grilling and Round 2 reviewer synthesis. (R5's hypothesis tag for the 020 mapping is documented inline; if the 020 design lands differently, R5 is reopened by `specflow:scope-change`, not as an open question here.)

## See also

- Interview: [`./025-sprint-task-flagging-interview.md`](./025-sprint-task-flagging-interview.md)
- Heuristic doc: [`../../templates/task/sprint-bucket-heuristic.md`](../../templates/task/sprint-bucket-heuristic.md) (created by this PRD; canonical home for the bucket-assignment heuristic)
- Tasks: [`./025-sprint-task-flagging-tasks.md`](./025-sprint-task-flagging-tasks.md) (generated by `specflow:task`)
- Tests: [`./025-sprint-task-flagging-test.md`](./025-sprint-task-flagging-test.md) (generated by `specflow:test`)
