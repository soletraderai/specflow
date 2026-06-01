# SpecFlow improvement analysis

**Scope:** the last 10% — making PRD/task creation faster, fixing the self-learning loop, and getting AI to carry more of the implementation.
**Evidence base:** the SpecFlow plugin source at `plugins/specflow/` and ~96 MB of real ClaimXPro usage transcripts at `~/.claude/projects/-Users-marepomana-Web-ClaimXPro/`, plus the live ClaimXPro project state under `docs/specflow/admin/`.
**Status:** analysis and recommendations only. No plugin code has been changed.

---

## Executive summary

The 90% that works is the adversarial review chain — the grilling, the multi-agent debates, the coverage matrices. The quality is real and the transcripts prove it. The 10% that hurts is that the same machinery now runs unconditionally and at full strength on every feature, regardless of size or risk, and two of the supporting loops (self-learning and AI-led development) are wired up but not actually closing.

Three concrete findings:

1. PRD-to-tasks is slow because the adversarial debate runs four to seven times per feature, each time as a full 3-round, 5-6 reviewer fork. A single `/specflow:task` run in the transcripts took roughly 1h40m on its own and spawned 14+ reviewer sub-agents; the PRD run before it consumed a similar block. The debate pattern was originally something the owner hand-built once per PRD. SpecFlow codified that single pass into Gates 2, 3, 4, 5, plus a cross-task mini-debate and two pre-gate Codex passes. The fix is to make the debate conditional and capped, not to remove it.

2. The self-learning loop captures well but reuses weakly, and its two halves are not connected. `lessons.json` holds nine genuinely excellent lessons (L-001 to L-009). But `plugin-findings.jsonl` — the corpus `specflow:learn` consumes — does not exist in ClaimXPro, so the auto-promotion half of the loop has never run. Worse, the lessons that are captured keep recurring: L-001 recurred four times after being promoted to a rule; L-004 recurred five-plus times. Capture works; the reuse path (forward-only injection into prompts, no enforcement gate) does not change behaviour.

3. AI is not carrying the implementation; the dev team marks Linear tickets Done by hand and the work diverges. `task-history.json` contains zero `lane_assigned` / `elapsed_minutes` / `what_worked` entries, meaning `specflow:develop`'s lane-based AI build flow was effectively not the path the dev work took. L-006: "all 28 tickets marked Done in Linear, but 18 of 28 have zero corresponding code"; L-007: guards authored but never wired. The Linear-to-dev-to-Done path has no AI implementation step and no shipped-vs-plan reconciliation gate.

Top recommendation: ship the conditional-debate changes (pain point 1) first — biggest time win and lowest risk because lightweight mode already proves the pattern. Then connect the learning loop (pain point 2). The dev-throughput change (pain point 3) is the largest piece and is partly workflow, not plugin.

---

## Pain point 1 — PRD + task creation is too slow (~8 hours)

### (a) What the transcripts and plugin actually show

The pipeline runs the same heavy debate up to seven times per feature:

- `specflow:prd` Phase B is grilling (Gate 1), Phase C.4 is a pre-Gate-2 Codex pass, Phase D is Gate 2 — a full 3-round debate with 5 reviewers (6 with Codex).
- `specflow:task` Phase B.5 is a pre-Gate-3 Codex pass, Phase E is Gate 3 — another 3-round, 5-6 reviewer debate, and Phase E.4.5 is a cross-task review — a third 3-round mini-debate.
- `specflow:develop` then adds Gate 4 and Gate 5, each another 3-round, 6-reviewer debate, per task.

A 5-reviewer, 3-round gate is up to 15 forked agent runs. For PRD + tasks alone that is Gate 2 + Gate 3 + cross-task + two Codex passes — comfortably 35-45 sub-agent invocations before a single line of code.

The transcripts confirm the cost. Session `f7373f1c` (2026-05-21): a single `/specflow:task` run ran ~1h40m for the task phase alone, with 14 reviewer sub-agent folders and 27 reviewer-dispatch references. Session `02ffe904` (2026-05-25): ~1h45m of PRD/prep before tasking even began. Two such blocks back to back land at the owner's 8-hour figure.

Config permits it: ClaimXPro's `admin/config.json` has `task.maxDurationHours: 8` and `task.contextBudget: 200000`. The big task run hit `cache_read_input_tokens: 402470` — running near the context cliff, where re-reads and retries multiply.

Lightweight mode already exists (`specflow:feature` sets `mode: light|full`; light caps grilling at 0-2 rounds and skips Gates 2/3) but only triggers off verb heuristics at kickoff. The owner's hypothesis — too much debate, too many agents, not syncing — is correct; the lever is already half-built.

The sync friction is the fork-and-reconcile overhead: every gate writes per-reviewer JSON to `findings/round-N/`, then an orchestrator re-reads them all to write a manifest. With 5-6 reviewers x 3 rounds x multiple gates, the orchestrator spends most wall-clock marshalling files between forked contexts.

### (b) Concrete recommended changes

Quick wins:

1. Default to a 3-tier mode, not 2, and auto-classify. Add `standard` between `light` and `full`: standard runs Gate 2 and Gate 3 at 1 round each (no Round-2 rebuttal / Round-3 sharpen unless a `block` lands); cross-task review only fires on 5+ tasks. Auto-classify mode from PRD size and surface, not only kickoff verbs. Most ClaimXPro features should land in standard.
2. Collapse the two Codex passes into the gate, not before it. Drop the separate pre-gate Codex pass in standard mode; let Codex's in-gate reviewer slot carry it. Keep the pre-gate pass only in full.
3. Make Round 2 and Round 3 conditional on a `block`. In every gate, only fire the rebuttal/sharpen rounds if Round 1 produced at least one `block` or load-bearing `concern`. Highest-leverage change: most rounds converge with nothing load-bearing yet pay the full 3-round cost.
4. Cap reviewer count per gate by the task's `Layers Touched`. Drop from 5-6 reviewers to the 2-3 relevant ones in standard mode.
5. Lower the ClaimXPro ceilings: `task.maxDurationHours: 1-2` and tighten `contextBudget` so the split prompt fires before the context cliff.

Bigger redesign:

6. Single combined Gate 2+3 in standard mode — synthesise PRD and tasks, then run one debate over both together rather than two fork-and-reconcile cycles.

### (c) Expected impact

Changes 2 + 3 alone should remove roughly half the sub-agent invocations on a typical feature. Realistic PRD-to-tasks wall-clock: from ~8 hours toward 2-3 hours for a standard feature, with full still available. Quality risk is contained: a real block still triggers full 3-round escalation.

---

## Pain point 2 — the self-learning loop is weak

### (a) What the plugin and project state show

The owner's intent is the right design, and SpecFlow half-implements it twice, in two disconnected systems:

- System A — `lessons.json` (works on capture, weak on reuse). Written by `specflow:test` Phase D, queried by prd/task/test. Holds nine high-quality lessons (L-001 to L-009). This is the knowledge system the owner wants.
- System B — `plugin-findings.jsonl` + `specflow:learn` (does not run). `specflow:learn` clusters and promotes recurring findings; it reads `plugin-findings.jsonl`, which does not exist in ClaimXPro. So the auto-promotion half is dead. `specflow:test` Phase D.4.5 claims to auto-fire `specflow:learn`, but with no producer the consumer no-ops.

The reuse path does not change behaviour — the evidence is recurrence. The registry only injects matched lessons forward as "must consider" lines, with no enforcement. L-001 (narrow check accepted as proof) recurred four times, including after being promoted to a rule. L-004 (zero-tolerance ACs decay without a CI gate) recurred five-plus times ("yarn check:colors still not in CI"). Capture is excellent; reuse is a heads-up the model can and does ignore.

Secondary problem: lessons are written as prose escape narratives, not reusable test fragments. So even when a lesson matches, the model re-derives the test from scratch — the opposite of "faster over time".

### (b) Concrete recommended changes

1. Pick one corpus. Make `lessons.json` the single source of truth; retire `plugin-findings.jsonl`/`specflow:learn` or repoint `specflow:learn` to cluster over `lessons.json`.
2. Add a reusable `test_fragment` field to each lesson — a copy-pasteable assertion (grep pattern + scope, test-case skeleton, or CI command). This is what `specflow:test` injects verbatim, so generation reuses prior work instead of re-deriving.
3. Make matched lessons a gate, not a hint, for their own surface — the fragment becomes a required test case the plan cannot close without.
4. Auto-promote a recurring runnable lesson into a CI check, not just a `guidelines.md` line — when a lesson reaches the promotion threshold and its fragment is runnable, emit a misc-task to wire it into CI.
5. Keep it from bloating: cap the active set (5 per query, tag-ranked); add a prune pass that flips a lesson to `resolved` once a feature ships with the fragment passing and no recurrence.

### (c) Expected impact

"Smarter and faster over time" becomes literal: the second feature on a surface reuses the first's test fragment. Recurrence on captured lessons drops sharply, because a matched lesson now blocks its own test plan.

---

## Pain point 3 — dev-team throughput

### (a) What the transcripts and project state show

The owner wants AI to do most of the implementation, with the team signing off on visual and backend. Today the opposite happens:

- `task-history.json` has no development-time records — no `lane_assigned`, `ai_assistance_level`, `elapsed_minutes`. `specflow:develop`'s lane-based AI build flow was not the path the dev work took; the team builds Linear issues by hand in Claude Code, outside the develop loop.
- Hand-built tickets diverge from the plan. L-006: "all 28 tickets marked Done in Linear, but 18 of 28 have zero corresponding code... a radically descoped, architecturally different feature." L-007: guard class authored but dead code, never registered. These are exactly what Gate 4/Gate 5/Verifier are designed to prevent — but the build happened outside that flow.
- The Linear export is one-directional. `specflow:linear --sync` only pulls status back; a human flipping a ticket to Done is trusted as proof, with no shipped-vs-plan reconciliation before close. L-006 is that gap realised.

### (b) Concrete recommended changes

Plugin changes:

1. Make `specflow:develop` the default build path for exported tasks, triggered from Linear (the "Backlog/Todo to In Progress" hook). The human's job moves from writing code to reviewing the PR.
2. Split sign-off into visual and backend: UI-touching tasks must carry Playwright screenshot artefacts + a human visual approval; backend/API/database tasks need a human contract/schema approval. Route the sign-off to the right human by surface.
3. Add a shipped-vs-plan reconciliation gate before a ticket can be Done — compare the shipped diff against the whole task plan; refuse Done if the architecture diverged (fixes L-006/L-007/L-008/L-009).
4. Close the Linear round-trip honestly: `--sync` refuses to accept a Linear "Done" until the reconciliation gate passes.

Workflow changes (broader than the plugin):

5. Wire the recurring-lesson enforcement scripts into CI as required PR checks (ClaimXPro `cicd` is currently false). Biggest non-plugin lever.
6. Standardise the team on the develop loop — start tickets through `specflow:develop` rather than hand-coding then verifying.

### (c) Expected impact

AI moves from "verifies after humans build" to "builds, humans sign off". The L-006 class of failure cannot occur once Done requires passing reconciliation. Throughput rises because the human bottleneck shifts from authoring to reviewing.

---

## Prioritised list

Quick wins (do first):

1. Conditional Round 2/3 in all gates — only on a `block` or load-bearing `concern`. (Pain point 1)
2. Fold the two pre-gate Codex passes into the in-gate Codex slot in standard mode. (Pain point 1)
3. Add a `standard` mode and auto-classify features into it. (Pain point 1)
4. Cap reviewers per gate by `Layers Touched`. (Pain point 1)
5. Lower ClaimXPro's `task.maxDurationHours` and `contextBudget`. (Pain point 1)
6. Add a `test_fragment` field to lessons and make a matched lesson a required test case. (Pain point 2)

Bigger redesigns (do second):

7. Unify the two learning systems — one corpus, one query path. (Pain point 2)
8. Auto-promote recurring runnable lessons into CI checks. (Pain points 2 + 3)
9. Make `specflow:develop` the default Linear-triggered build path, with visual + backend sign-off. (Pain point 3)
10. Add a shipped-vs-plan reconciliation gate before Done; `--sync` refuses Done until it passes. (Pain point 3)
11. Single combined Gate 2+3 in standard mode. (Pain point 1)

Workflow (owner/team to drive):

12. Wire the recurring-lesson enforcement scripts into CI as required PR checks.
13. Move the dev team onto the develop loop.
