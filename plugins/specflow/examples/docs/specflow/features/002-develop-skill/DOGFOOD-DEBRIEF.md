# Dogfood debrief — 002-develop-skill

**Date:** 2026-05-06
**Pass:** recursive bootstrap; specflow chain run on a Phase 2 specflow skill (`specflow:develop`).
**Outcome:** PRD passed Gate 2 after PRD revisions for two `block` findings and four `concern` findings; one push-back defended.

This debrief is the deliverable from the dogfood pass per `v2/docs/SESSION-HANDOFF.md` § "Watch for". It captures what the chain surfaced, what worked, and concrete prompt edits recommended for `skills/{prd,grill,task}/SKILL.md` based on observed friction. Edits are recommendations only — the human applies after review.

---

## What broke (friction surfaced)

### F1. Goal field synthesis in the interview asked for "Outcome" but the PRD body's Vision section then re-states it in different words.

The interview's confirmed Goal Outcome field was a tightly-scoped single sentence ("specflow:develop turns a closed-Gate-3 task list into shipped code via lane-based execution..."). The PRD's Vision section (per `skills/prd/SKILL.md` Phase C.2 template) re-derives a paragraph version. The instruction in `skills/prd/SKILL.md:240` says "Synthesise directly from the interview's Goal section. Vision is one paragraph that says: what the world looks like when this feature exists, who it serves, why it matters." — but the prompt does not say *whether* the Vision should reuse the Outcome's exact words or re-write. In this run the PRD ended up with a long Vision paragraph that overlapped substantially with the Goal but didn't quote it verbatim; a reviewer auditing trace integrity could reasonably ask whether the Vision is a faithful synthesis or a slight re-interpretation. **Friction:** the prompt is silent on the verbatim-vs-paraphrase contract.

### F2. AC formatting under uncertainty — when an AC depends on a Phase 1 surface that hasn't been confirmed.

AC-10 in this PRD says `specflow:develop` invokes `specflow:misc --auto` with two specific named fields (`manifest_path`, `gate_finding_id`). The current Phase 1 `skills/misc/SKILL.md` schema doesn't document these fields. The PRD synthesis prompt (`skills/prd/SKILL.md` Phase C.2) does not have a sub-step for "when an AC depends on a Phase 1 schema, verify the schema first or surface the dependency". Surgical Reviewer caught this at Gate 2 — but ideally the AC author (the AI doing PRD synthesis) would have caught it before writing the AC, by reading `skills/misc/SKILL.md` during synthesis. **Friction:** PRD synthesis doesn't cross-check ACs against the existing skill schemas they depend on.

### F3. Adversarial review with `block` findings under-specified — the AI didn't know when to escalate.

The two `block` findings (tbc-r1-f1, goal-r1-f1) were both accepted in Round 2 with concrete revisions; both converged in Round 3. But `skills/prd/SKILL.md` Phase E.6 (Closer — Orchestrator collates) doesn't specify the disposition rule for `block` findings that ARE accepted-and-revised: is the manifest status `passed`, `passed-with-revisions`, or something else? The 001-design-skill calibration anchor used `passed` for a manifest with 0 blocks; this manifest has 2 blocks both resolved and is also `passed`. The convention is implicit but not codified — a future PRD with `block` findings that don't converge would need the convention spelled out. **Friction:** the orchestrator's status taxonomy at Gate 2 doesn't distinguish "passed clean" from "passed after block-finding revisions".

### F4. Goal-Driven Reviewer's lens for "every requirement is covered by at least one AC" applied symmetrically — but the inverse ("every AC verifies a requirement") is not in the reviewer's documented lens.

Goal-Driven Reviewer's Round-1 finding `goal-r1-f1` flagged AC-13 as an orphan AC (verifies a contract no R establishes). The reviewer applied the inverse rule correctly, but the documented lens in `templates/agents/standard/principles/goal-driven-reviewer.md` (per skills glossary) emphasises forward coverage (every R has an AC), not reverse traceability (every AC verifies an R). The reviewer would have to extrapolate the inverse rule from the documented forward rule. **Friction:** Goal-Driven Reviewer's lens is documented one-directionally; both directions are load-bearing.

### F5. `specflow:task` Gate 3 pre-flight check is forward-looking ("does the PRD have closed Gate 2?") but doesn't surface the `block` resolution shape.

When `specflow:task` runs against this dogfood PRD, its Phase A pre-flight check (`skills/task/SKILL.md:39-42`) verifies Gate 2 closing decision is `passed` or `passed-with-escalations`. It will pass against this manifest. But Phase A doesn't read the Round-1 `block` findings to surface them as context for task synthesis — even though `block` findings often establish architectural constraints the tasks should respect (e.g. R5.1's mechanical lane recheck implies a task that builds the mechanical-recheck step, distinct from R5's catch-all path). **Friction:** Gate 3 task synthesis doesn't read the Gate 2 block-finding resolutions as input; if it did, downstream tasks would inherit the resolution shape automatically.

---

## What worked

### W1. Lens distinctness — every reviewer fired a different concern.

Simplicity flagged speculative configurability (two new `config.json.develop.*` knobs); Surgical flagged cross-skill creep (AC-10 implicitly editing the `specflow:misc` schema); Think-Before-Coding flagged an unstated load-bearing assumption (lane-recheck lens) plus a self-contradicting rationale (DA covering cross-provider in same-provider); Goal-Driven flagged an orphan AC and a goal-coverage gap; Devil's Advocate flagged a degraded-path coverage gap (plugin present-but-failing). No two reviewers flagged the same finding. This matches the calibration note from `001-design-skill/debate-log/prd-gate2/manifest.md:89` ("each reviewer should fire its lens distinctly"). The lens-distinct prompts in `templates/agents/standard/principles/*.md` are doing their job.

### W2. The interview file's Resolved-line discipline made PRD synthesis traceable.

Every requirement in the first-pass PRD traced to a Resolved line in the interview (or to the codebase context bullet, with explicit reason for the pattern-break). The Surgical Reviewer's trace-integrity lens worked well at Gate 2 — surgical-r1-f1 was the only trace-integrity-adjacent finding, and it caught a cross-skill schema dependency, not a missing trace. The discipline established in Phase 1 (`skills/grill/SKILL.md` Step 5 mandates a non-empty Resolved line) cascades into clean PRD-synthesis traceability.

### W3. Round-2 split push-back ("partial accept on R11, defend R6") was authored cleanly.

The Simplicity finding lumped two knobs together; the Round-2 response split them with cited evidence (interview Round 2 line 78 grounding R6, no equivalent grounding for R11). The Round-3 reviewer accepted the split as the right calibration. The 001-design-skill calibration anchor's two push-backs were single-finding all-defends; this run's split push-back is a slightly more complex shape that the prompt handled correctly. The pattern is portable to future runs.

### W4. The orchestrator pattern target (≤2K parent context per gate) held.

Per the skill's R16, Gate 2's parent context grew by approximately 1.8K tokens during this run (5 round-1 findings × ~250 tokens, 1 round-2 responses file at ~600 tokens, 5 round-3 responses × ~150 tokens, 1 manifest at ~1.2K tokens — but all were file-on-disk first; the parent saw paths). The target from `templates/orchestrator-pattern.md:849` is met. `specflow:budget` was not run during this dogfood (budget tracking is a Phase 1 skill that this dogfood didn't exercise), but the manual estimate is within bounds.

---

## Recommended prompt edits

These are recommendations. The human applies after review. **Do NOT auto-apply.** All file:line citations are against current `v2/specflow/skills/`.

### E1. `skills/prd/SKILL.md:240` — codify Vision verbatim-vs-paraphrase contract.

Current text:
> {Synthesise directly from the interview's Goal section. Vision is one paragraph that says: what the world looks like when this feature exists, who it serves, why it matters. Do NOT re-derive from the rounds — the goal IS the precedent.}

**Recommended addition** (immediately after the existing text):

> The Vision should incorporate the Goal Outcome's load-bearing phrases verbatim where possible — paraphrase only for prose flow. Reviewers at Gate 2 will check Vision-to-Goal trace integrity; if the Vision drops or rephrases an Outcome phrase, the prose change should be defensible (e.g. "lane-based execution" appears verbatim from the Outcome; "trustworthy AFK-eligible green-lane execution" can be a Vision-only paraphrase since "trustworthy" is implied by the Outcome's "lane discipline + PRD anchor + cross-provider review are the three legs that make AFK-eligible green-lane execution trustworthy").

**Why:** F1 above. Closes the verbatim-vs-paraphrase ambiguity that surfaced during this run.

### E2. `skills/prd/SKILL.md:298` — add Phase C.2 sub-step "cross-check ACs against Phase 1 skill schemas".

Current text (Phase C.3 self-check, item 4):
> 4. **No requirement contradicts the goal's Out-of-scope-at-goal-level.** Cross-check.

**Recommended addition** (new item 5 inserted after item 4):

> 5. **Cross-check ACs against Phase 1 skill schemas they depend on.** For every AC that names a Phase 1 skill (e.g. `specflow:misc --auto`, `specflow:linear`, `specflow:render`), open that skill's SKILL.md and verify the AC's named fields exist in the skill's documented payload schema. If the AC references fields the schema doesn't include, EITHER edit the AC to use only existing fields OR add an explicit "Schema dependency" note naming the Phase 1 schema gap and the proposed enhancement PRD that must land first.

**Why:** F2 above. Surgical Reviewer caught this manually at Gate 2 in this run; better to have the PRD author catch it during synthesis.

### E3. `skills/prd/SKILL.md:454` — clarify Gate 2 manifest status taxonomy when blocks are resolved.

Current text:
> Gate 2 status: **passed | passed-with-escalations | failed**
>
> {One paragraph closing rationale by the Orchestrator. If passed: PRD is fit to proceed
> to specflow:task. If escalations: list the items the human must resolve before proceeding.
> If failed: list the blocking findings and what must change.}

**Recommended addition** (replace the existing taxonomy with):

> Gate 2 status: **passed | passed-with-revisions | passed-with-escalations | failed**
>
> Status determination:
> - `passed` — zero `block` findings landed (or all `block` findings were rejected with reviewer-accepted defences); zero accepted findings forced revisions to load-bearing requirements.
> - `passed-with-revisions` — `block` or load-bearing-`concern` findings landed and were accepted; PRD revisions applied; reviewers converged in Round 3. List the revisions applied under "PRD revisions applied".
> - `passed-with-escalations` — at least one finding did not converge in 3 rounds; surfaced for human decision before proceeding.
> - `failed` — at least one `block` finding was not resolved (rejected without reviewer acceptance, OR no convergence after Round 3 escalation).

**Why:** F3 above. Distinguishes a clean pass (001-design-skill) from a passes-after-block-resolutions pass (this dogfood), which a downstream consumer (Phase 3 self-learning) would want to filter on.

### E4. `templates/agents/standard/principles/goal-driven-reviewer.md` — add reverse-traceability lens.

(File path inferred from skills glossary; verify before editing.)

**Recommended addition** to the Common Findings section:

> - **Orphan AC** — an AC that doesn't verify any stated requirement. Reverse traceability: every AC must trace to ≥1 R, just as every R must be covered by ≥1 AC. An AC verifying a contract the PRD didn't make is itself a coverage gap (the contract is unstated at the R-level).

**Why:** F4 above. The reviewer caught the orphan-AC pattern in this run by extrapolating from the forward rule, but documenting the inverse explicitly makes the lens reliable across reviewers.

### E5. `skills/task/SKILL.md:62` — surface Gate 2 block-finding resolutions as task synthesis input.

Current text (Phase A.1 read list):
> - `features/NNN-{slug}/debate-log/prd-gate2/manifest.md`

**Recommended addition** (new Phase A.2 sub-step, inserted before the existing A.2):

> ### A.2 Surface Gate 2 block-finding resolutions
>
> Read the Gate 2 manifest's "PRD revisions applied" section (if present). Each revision listed there is a load-bearing constraint the tasks should respect. For example, if a Gate 2 `block` finding added a new requirement R5.1 (mechanical pre-Gate-4 lane-recheck step), the tasks should include a distinct task for the mechanical recheck *separate from* the catch-all reviewer-driven re-lane (R5). Do not merge tasks that the Gate 2 process deliberately separated.
>
> Note any revisions that produced new R-level requirements; these should each have ≥1 task per the forward-coverage rule.

**Why:** F5 above. The Gate 2 manifest's revision history is a richer signal than the closing decision alone; tasks that respect the revision shape inherit the resolution implicitly.

---

## Next-session priority

**Recommended:** ship Phase 1 (Priority 2 from `v2/docs/SESSION-HANDOFF.md`), then start Phase 2 build off this PRD.

The dogfood pass demonstrates that the chain handles a real first-pass PRD with `block` findings cleanly — convergence in Round 3, no escalations, PRD revisions applied authentically. The surfaced friction (E1-E5 above) is incremental prompt-tuning, not architectural rework. None of the friction blocks a v2.0.0 release.

Concretely:
1. Apply E1-E5 prompt edits (after human review).
2. Promote `v2/specflow/` to `plugins/specflow/` per `v2/docs/SESSION-HANDOFF.md` § Priority 2 release steps.
3. Cut v2.0.0 release.
4. Phase 2 build picks up `002-develop-skill-prd.md` as the input — `specflow:task {002-develop-skill}` synthesises tasks; Gate 3 fires; implementation begins. The PRD's R5.1 (mechanical lane recheck) is the most architecturally distinctive addition from Gate 2; the Phase 2 tasks should respect that shape per E5.

The recursive-bootstrap nature of this run means the Phase 2 build has a tested PRD + a manifest demonstrating the chain handles Phase-2-domain features correctly. That's the strongest possible foundation for Phase 2 implementation.
