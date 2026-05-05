# Orchestrator pattern

The operational pattern any specflow skill that orchestrates other skills MUST follow. This is how multi-skill chains stay within budget instead of leaking context.

Without this pattern, a 5-step orchestration accumulates ~51K tokens by step 5 (raw tool responses, intermediate scrapes, sub-agent reasoning traces all visible in the parent context). With this pattern, the same chain runs at ~5-8K tokens — **roughly 85% reduction**. The difference is real and load-bearing for Phase 2's `specflow:develop` and the multi-agent debate manifest.

## The three primitives

### 1. Forked sub-agent contexts

Every sub-skill invocation runs in its own forked context, separate from the parent conversation. Tool responses, intermediate reasoning, and raw data stay in the fork. **Only the structured result returns to the parent.**

When to fork:
- Any sub-skill that performs tool calls returning bulky data (web fetch, file reads of large files, database queries, scrape results).
- Any sub-skill whose internal reasoning is not load-bearing for the parent's decision (most reviewers fall into this category).
- Every reviewer in the multi-agent debate manifest — without exception.

When NOT to fork:
- A sub-skill whose entire output IS the artefact being authored (e.g. when the parent is asking the sub-skill to write a paragraph the parent will use verbatim). Forking adds round-trip overhead with no context savings.

### 2. File handoff between steps

Sub-skills write minimal extraction files instead of returning bulky payloads through the conversation channel. The parent reads only what's needed for the next step.

Convention:
- Step output files live in a per-orchestration scratch directory: `admin/scratch/{orchestration-id}/{step-name}.{ext}`.
- File contents are the **distilled** result, not the raw input. A 50KB scrape becomes a 200-token JSON of the fields the next step actually uses.
- Each step declares (in its `requires:` and `produces:` frontmatter) which files it consumes and which it emits, so the orchestrator can validate handoff without inspecting bodies.
- Scratch files are removed after the orchestration completes successfully (or retained for debugging on failure — orchestrator's call).

Bad pattern (V1 — the leak):
```
Step 1 returns: full scrape (4K tokens) → visible in parent context
Step 2 returns: full search results (8K tokens) → visible in parent context
Step 3 returns: synthesis of 1+2 (6K tokens) → visible
... by step 5: ~51K tokens of accumulated raw data
```

Good pattern (V2 — the discipline):
```
Step 1 forks → writes profile.json (200 tokens) → parent sees only "wrote profile.json"
Step 2 forks → reads profile.json via command substitution → writes signals.json (300 tokens)
Step 3 forks → reads signals.json → writes synthesis.md (1200 tokens)
... by step 5: ~5-8K tokens total in parent context
```

### 3. Command substitution for zero-cost file injection

When a step needs to inject the contents of a previous step's output file into its own prompt, use Claude Code's command substitution rather than a Read tool call.

```
!{cat admin/scratch/orchestration-42/profile.json}
```

This substitutes the file's contents into the prompt **before** the LLM sees the prompt. No tool call cost, no model decision about whether to read it, no accumulated tool-response in the conversation. The content is just there.

When to use:
- Step N needs the verbatim content of step N-1's output as part of its own prompt.
- Small files (< 5KB ideal; > 20KB defeats the purpose).
- Inputs the step *always* uses (not conditionally).

When NOT to use:
- Files that might not exist (substitution fails noisily).
- Conditional reads (use a Read tool inside the forked sub-agent instead).
- Large files where you only need a fragment (filter inside the substitution: `!{jq '.signals' admin/scratch/.../profile.json}`).

## Calibration anchor

If you're orchestrating ≥3 sub-skills and the parent context is over 15K tokens after step 3, you're leaking. Audit:
1. Are sub-skills forking? If they're inheriting the parent context, fix that first.
2. Are tool responses landing in the parent? If yes, the sub-skill isn't returning a structured result — it's returning raw data.
3. Are scratch files being read with Read tool calls instead of command substitution? Read tool calls accumulate in conversation history.

The `/specflow:budget` skill (Phase 1) exposes per-skill token consumption — use it to spot which steps are leaking before the bill arrives.

## How the multi-agent debate manifest uses this pattern

Reference: PRD Appendix N.

Every adversarial-review gate runs N reviewers in parallel. Each reviewer:
1. Runs in a forked context (no shared bias from other reviewers, no accumulated context from prior gates).
2. Reads the artefact under review via command substitution (`!{cat features/NNN-{slug}/NNN-{slug}-prd.md}`) — zero tokens spent on "let me read the file."
3. Writes a **minimal finding file** to `features/NNN-{slug}/debate-log/{gate}/findings/round-{N}/{reviewer-name}.json` — just severity, evidence, proposed change. Not the reviewer's internal reasoning trail.
4. Returns the path of the finding file (one line) to the Orchestrator. The Orchestrator collates findings into the manifest summary.

The manifest summary file (`features/NNN-{slug}/debate-log/{gate}/manifest.md`) contains the *collated* findings + AI's responses + reviewers' Round-3 entries — but not the per-reviewer raw context that produced them. The raw context lives in the forked sub-agent and dies with it (unless explicitly retained for Phase 3 self-learning, in which case it's archived to `features/NNN-{slug}/debate-log/{gate}/raw/{reviewer-name}.txt`).

**Why feature-scoped paths:** When a downstream agent works on the same feature (e.g. `specflow:develop` after `specflow:task`), it reads every prior gate's manifest from the same feature folder. No cross-folder chase. The feature folder is a complete record of how the feature was built. Cross-feature gates (misc-task review, `/optimize` runs) keep the legacy `admin/debate-log/{date}-{slug}-{gate}/` location since there's no feature folder to home them in.

This keeps the parent Orchestrator's context bounded regardless of how many reviewers fire or how chatty each one is.

## How `specflow:develop` will use this pattern (Phase 2)

For each task in a feature, `specflow:develop` orchestrates:
1. Plan generation (one forked agent) → writes `plan.md` to scratch.
2. Gate 4 debate manifest (N forked reviewers + Orchestrator collator) → writes `gate-4-manifest.md`.
3. Implementation (forked specialised agents per file) → writes intermediate diffs to scratch, NOT to the conversation.
4. Gate 5 debate manifest (Codex + reviewers) → writes `gate-5-manifest.md`.
5. Verifier confirmation (one forked agent) → writes `verification.md`.
6. Final commit / PR (parent only sees the file paths and pass/fail signals).

A 10-task feature should run end-to-end in ≤30K parent-context tokens. Without this pattern it would run at ≥150K and start hitting context limits mid-feature.

## Skill author checklist

Before shipping a skill that orchestrates others:
- [ ] Every sub-skill invocation forks unless there's a documented reason not to.
- [ ] Every step declares its `requires:` and `produces:` files in SKILL.md frontmatter.
- [ ] Verbatim file content is injected via command substitution, not Read tools.
- [ ] Sub-skills return distilled results (one path or one structured summary), not raw payloads.
- [ ] Scratch directory is named per orchestration (`admin/scratch/{orchestration-id}/`), cleaned up on success.
- [ ] `/specflow:budget` shows the parent context staying under 15K after 3 sub-skill steps.

## Reference

- PRD Appendix N — multi-agent debate manifest (uses this pattern).
- `templates/agents/standard/lifecycle/orchestrator.md` — the agent that owns this pattern's enforcement.
- `skills/budget/SKILL.md` — per-skill token tracking.
- `skills/develop/SKILL.md` (Phase 2) — primary consumer of this pattern.
