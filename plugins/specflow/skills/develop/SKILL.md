---
name: specflow:develop
description: Orchestrate implementation of tasks generated from a PRD. Lane-based execution (green/yellow/red); integrates agent teams; runs Gates 4 and 5 of the adversarial review chain.
status: v2-new
phase: 2
requires: [docs/specflow/features/{NNN-slug}/NNN-{slug}-tasks.md, docs/specflow/features/{NNN-slug}/NNN-{slug}-prd.md, docs/specflow/features/{NNN-slug}/NNN-{slug}-interview.md, docs/specflow/admin/agents/, docs/specflow/admin/config.json, docs/specflow/admin/rules/, docs/specflow/admin/environment.json]
produces: [docs/specflow/features/{NNN-slug}/debate-log/develop-gate4/manifest.md, docs/specflow/features/{NNN-slug}/debate-log/develop-gate5/manifest.md, docs/specflow/admin/task-history.json, docs/specflow/admin/scratch/{orchestration-id}/]
eval: Verifier confirms task acceptance criteria pass; Gate 5 (Codex code review) signs off; PR opened with PRD anchor in description; debate manifests closed by Orchestrator.
---

# specflow:develop

Phase 2 centrepiece. Orchestrates the *implementation* phase — the gap between task creation (Linear-ready) and test. This is where specflow gains its hands.

**Triggers:** "develop {task-id}", "/specflow:develop", or invoked automatically when a Linear task is moved into "In Progress".

**Skill behaviour (high level):**
1. **Context primer.** Read the task, related PRD, decision-log entries, project's available agent set from `admin/environment.json`, rules registry slice for the touched paths.
2. **Lane assignment + team composition.** Lane assigned by triage tuple (verifiability + blast radius + dependency state + confidentiality classification). Orchestrator picks specialised agents based on lane and scope; agent teams from `config.json` compose the working group.
3. **Plan emission with PRD anchor.** Every plan starts with *"We're doing X because of PRD requirement Y. This aligns with Z."* Then technical plan with verify-steps inline.
4. **Implementation loop.** Agents execute, Orchestrator coordinates, Devil's Advocate intervenes at decision points. Testing-as-cadence — verification fires after every step.
5. **Gate 5 — code vs plan.** Cross-provider Codex review on the diff (3-iteration debate loop). Findings Codex catches and Claude missed get promoted to new entries in `admin/rules/guidelines.md`.
6. **Verification.** Verifier runs at completion vs the original task acceptance criteria.
7. **Handoff.** Emits a summary the test skill can pick up; updates Linear status; logs the run for Phase 3 retro consumption.

**Green / Yellow / Red lanes:**
- **Green** (verifiable + low blast + non-confidential): batched, AFK-eligible, single batched human sign-off per batch.
- **Yellow** (one axis weak): HITL — agent + human paired in real time.
- **Red** (high blast OR low verifiability OR confidential): human-led; AI assists on bounded subtasks only.
- Confidentiality classification is rule-based (path globs in `config.json.confidentialPaths`), not AI-rated.
- Initial target ratio: 60/30/10 (G/Y/R). 30/40/30 means the PRD needs to be re-cut — itself a learning signal logged to `task-history.json`.

**Rule violation auto-flagging:**
- Out-of-scope rule violations spotted during develop → `misc-task` with rule reference, file:line, observation, *why* (citing the rule's why-line).
- In-scope violations → blocker, must be addressed before shipping.

**Verify steps:**
1. Plan was emitted with a PRD anchor citing a specific requirement ID.
2. Lane assignment recorded; rule-based confidentiality check ran.
3. Gate 4 and Gate 5 debate transcripts saved.
4. Verifier confirms acceptance criteria pass.
5. Linear status updated.

**Reference:** PRD Phase 2 scope item 1; Appendix L (full skill spec), N (Gates 4-5).
