# mattpocock — repos overview

## Who he is
Matt Pocock is a TypeScript educator (Total TypeScript) and developer advocate (formerly XState/Stately, now AI Hero) with one of the most-followed AI/TS audiences on the web (~60k newsletter subscribers). His public work spans two clear arcs: a long history of TypeScript pedagogy and tooling (`ts-reset`, `ts-error-translator`, `total-typescript-monorepo`), and a recent, very heavy investment in *agentic coding workflows* — most notably his 44k-star [`mattpocock/skills`](https://github.com/mattpocock/skills) repo and `@ai-hero/sandcastle` (a TypeScript orchestrator for sandboxed coding agents). He explicitly positions his skills as "engineering for real engineers — not vibe coding," which is exactly the niche specflow occupies.

## Top relevant repos

### 1. `mattpocock/skills` (44,886 stars) — **most relevant**
- **What it is**: A curated `.claude/skills` directory shipped as `npx skills@latest add mattpocock/skills`. Engineering skills (`grill-with-docs`, `to-prd`, `to-issues`, `tdd`, `triage`, `diagnose`, `improve-codebase-architecture`, `zoom-out`, `setup-matt-pocock-skills`), productivity skills (`grill-me`, `caveman`, `write-a-skill`), and misc (`git-guardrails`, `setup-pre-commit`).
- **Why relevant**: This is a direct competitor/cousin to specflow's pipeline (Setup → PRD → Linear/Prime → Task → Test). Same audience, same Claude Code substrate, same Markdown-skill ergonomic. Many gaps in specflow's pipeline are filled by Matt's skills.
- URL: https://github.com/mattpocock/skills

### 2. `mattpocock/sandcastle` (1,526 stars)
- **What it is**: TypeScript SDK that runs Claude Code (and other agents) inside Docker / Podman / Vercel Firecracker sandboxes, with branch-strategy management and merge-back. `await run({ agent, sandbox, promptFile })`.
- **Why relevant**: Exactly the orchestration layer specflow's dev team would want for AFK / parallel agent runs. Mirrors specflow's "structured pipeline" instinct but at the *agent execution* layer.
- URL: https://github.com/mattpocock/sandcastle

### 3. `mattpocock/evalite` (1,497 stars)
- **What it is**: TypeScript-native, local-first eval framework for LLM-powered apps. Vitest-style: write evals as code, watch mode, web UI.
- **Why relevant**: Specflow's `test` skill could borrow evalite's posture (TS-native, watch-mode, local-first) for "test the spec → code transformation," not just app code.
- URL: https://github.com/mattpocock/evalite

### 4. `mattpocock/graph-docs-cli` (101 stars)
- **What it is**: Builds a knowledge graph from markdown frontmatter (`deps: [foo]`). Forbids circular dependencies. Designed for docs that are read in *learning order*, not narrative order.
- **Why relevant**: Specflow has a documentation-creator user. This is a near-perfect mental model for organizing specs/PRDs/ADRs as a DAG — which is exactly what specflow's outputs need.
- URL: https://github.com/mattpocock/graph-docs-cli

### 5. `mattpocock/ts-error-translator` (2,443 stars)
- **What it is**: VS Code extension that turns TypeScript compiler errors into plain English.
- **Why relevant**: Pattern for "translate machine output → human-readable explanation." Specflow could apply this to test failures, PRD validation errors, or pipeline-step rejections — turning agent-readable status into onboarding-friendly text for the doc-creator persona.
- URL: https://github.com/mattpocock/ts-error-translator

### 6. `mattpocock/ts-reset` (8,459 stars)
- **What it is**: A "CSS reset for TypeScript" — sharpens the standard library types (`JSON.parse → unknown`, `.filter(Boolean)` works as expected, etc.).
- **Why relevant**: Tangential. Useful as a *recommended dependency* for any TypeScript project specflow scaffolds, not as a pattern.
- URL: https://github.com/mattpocock/ts-reset

### 7. `mattpocock/agent-browser` (6 stars)
- **What it is**: Headless browser automation CLI for AI agents (Vercel Labs project Matt forked/contributes to). Snapshot-based accessibility tree with `@e2`-style refs designed for LLMs.
- **Why relevant**: If specflow ever needs browser-driven verification (e.g. "did the docs site render?", e2e for the spec'd feature), this is the agent-friendly primitive — accessibility refs, not CSS selectors.
- URL: https://github.com/mattpocock/agent-browser

### 8. `mattpocock/total-typescript-monorepo` (354 stars)
- **What it is**: "The home of all Matt's internal tooling" — a monorepo of his everyday CLIs and helpers.
- **Why relevant**: Reference example of how a single-author TypeScript practitioner organizes and maintains a tool monorepo. Useful for specflow's own internal tooling shape.
- URL: https://github.com/mattpocock/total-typescript-monorepo

### 9. `mattpocock/ai-hero-cli` (49 stars)
- **What it is**: CLI for navigating numbered exercise folders (`exercises/1-section/1-exercise/main.ts`) with `n`/`p`/Enter shortcuts and per-exercise `readme.md`.
- **Why relevant**: Pattern for shipping incremental, numbered learning content — directly applicable if specflow ever wants to ship onboarding tutorials for AI-new devs.
- URL: https://github.com/mattpocock/ai-hero-cli

### 10. `mattpocock/course-video-manager` (307 stars)
- **What it is**: Tool for managing course video publishing — metadata, descriptions, thumbnails, social posts via Dropbox→Zapier→Buffer pipeline. Notable for being the *worked example* Matt cites in his `skills` README for `CONTEXT.md`.
- **Why relevant**: Real example of `CONTEXT.md` in production — Matt links to it as the canonical "shared language" demo. Gold-standard reference for how a domain glossary file should read.
- URL: https://github.com/mattpocock/course-video-manager

## Patterns or content worth borrowing

### A. The four named failure modes (from `skills` README)
Matt names the four problems his skills fight, with literal book quotes:
1. **"The agent didn't do what I want"** → fix with `/grill-me` / `/grill-with-docs` (interview the user before any work).
2. **"The agent is way too verbose"** → fix with a `CONTEXT.md` shared-language file ("materialization cascade" beats "when a lesson inside a section of a course is made real").
3. **"The code doesn't work"** → fix with `/tdd` (red-green-refactor, vertical slices, no horizontal "all tests then all code") and `/diagnose` (build a feedback loop *first*, hypothesise *second*).
4. **"We built a ball of mud"** → fix with `/improve-codebase-architecture` and `/zoom-out` (deep modules > shallow modules; the *deletion test*).

These four labels are a far better marketing/onboarding spine than specflow's current step-by-step pipeline framing. Each one is a felt user pain.

### B. `CONTEXT.md` + ADRs as durable agent memory
- **`CONTEXT.md`** = project glossary. Created lazily on first term resolution. Owned by humans, read by agents every session. Reduces token usage *and* makes code, variables, and files name themselves consistently.
- **`docs/adr/NNNN-decision.md`** = architectural decisions. Created only when a decision is (1) hard to reverse, (2) surprising without context, (3) the result of a real trade-off. Three-criterion gate prevents ADR spam.
- **`CONTEXT-MAP.md`** at root if the repo has multiple bounded contexts, with per-context `CONTEXT.md` files inside subdirectories.

This is a complete, drop-in spec for "agent-readable persistent project knowledge." Specflow's `prime` skill should produce/maintain exactly this.

### C. Skill file structure (from `write-a-skill`)
```
skill-name/
├── SKILL.md           # required — under 100 lines
├── REFERENCE.md       # detail docs if needed
├── EXAMPLES.md        # examples if needed
└── scripts/           # deterministic helpers (validation, formatting)
```
Frontmatter: `name`, `description` (max 1024 chars, third person, "Use when [specific triggers]"). Description is the *only* thing the orchestrator sees when picking a skill — must include trigger keywords. Add scripts whenever the operation is deterministic or repeated (saves tokens, improves reliability).

### D. The `to-prd` template (drop-in for specflow's PRD skill)
Sections: Problem Statement → Solution → User Stories (numbered, "As a/I want/so that") → Implementation Decisions (modules, interfaces, schema, contracts — *not* file paths or code) → Testing Decisions (what makes a good test, which modules, prior art) → Out of Scope → Further Notes. Every PRD ends tagged `needs-triage`, entering a state machine.

### E. Triage as a state machine
Two **category** roles: `bug`, `enhancement`. Five **state** roles: `needs-triage` → `needs-info` ↔ `ready-for-agent` / `ready-for-human` / `wontfix`. Every issue carries exactly one category + one state. Every triage comment opens with `> *This was generated by AI during triage.*` — explicit AI-attribution. AGENT-BRIEF.md and OUT-OF-SCOPE.md as separate reference files.

### F. The `diagnose` six-phase loop
1. **Build a feedback loop** (failing test, curl, CLI snapshot, headless browser, replay, throwaway harness, fuzz, bisect, differential) — "the loop is the skill, everything else is mechanical."
2. **Reproduce** — confirm same failure mode the user described.
3. **Hypothesise** — 3–5 ranked falsifiable hypotheses *before* testing any. Show the list to the user (cheap checkpoint).
4. **Instrument** — debugger > targeted logs > "log everything"; tag every debug log `[DEBUG-a4f2]` for grep-cleanup.
5. **Fix + regression test** at the *correct seam* (or document why no seam exists).
6. **Cleanup + post-mortem** — original repro gone, regression test passes, all `[DEBUG-...]` removed, hypothesis stated in commit message.

The "loop is the skill" principle is the unifying insight: build the feedback loop *first*, optimize it *second*, and only then start hypothesising. Specflow's `test` skill could adopt this verbatim.

### G. `tdd`'s "vertical slices, not horizontal" anti-pattern
Explicitly named: do NOT write all tests then all implementation. Do `RED→GREEN: test1→impl1, test2→impl2, ...`. Reason given: bulk-written tests test *imagined* behavior, not actual; they end up testing data shapes, not user-facing capability. This single insight, packaged as a one-screen anti-pattern callout, would massively improve specflow's `test` skill.

### H. `caveman` mode — token-saving response style
A skill whose entire purpose is to drop articles, filler, and pleasantries while keeping technical accuracy intact. Auto-clarity exception: revert to normal prose for security warnings, irreversible-action confirms, multi-step sequences. Persistent across turns once triggered.

### I. The "deep modules" vocabulary (`improve-codebase-architecture`)
A controlled glossary the agent must use *exactly*: Module / Interface / Implementation / Depth / Seam / Adapter / Leverage / Locality. Plus three load-bearing principles: **deletion test** (would removing this concentrate or just move complexity?), **the interface is the test surface**, **one adapter = hypothetical seam, two adapters = real seam**. The skill explicitly forbids drift into "service / boundary / component."

### J. Knowledge-graph docs (`graph-docs-cli`)
Each markdown module declares its prerequisites in frontmatter (`deps: [foo, bar]`). Forbidding circular deps forces extraction or merging. Output is a graphviz of optimal learning paths. Far better mental model for spec/PRD organization than chronological folders.

### K. `sandcastle`'s "branch strategy + merge-to-head" pattern
Programmatic agent runs land on `agent/<tag>` branches, then merge back. Lifecycle hooks split into `host` and `sandbox`. `copyToWorktree` for files like `.env` that shouldn't live in git. `promptArgs` for `{{KEY}}` substitution into prompt templates.

## Direct relevance to specflow's goals

- **(1) More productive — for the dev team new to AI coding.**
  - The `grill-with-docs` / `to-prd` / `to-issues` chain is a fully-formed productivity loop specflow could absorb almost intact. Matt's framing of "ask first, write later" + the PRD template + the triage state machine answers the AI-new dev's biggest fear ("I don't know what to ask the agent"). Adopting the four named failure modes as specflow's high-level marketing/onboarding narrative would make the value prop legible in 30 seconds, vs. the current pipeline-as-list framing.

- **(2) Better product in shorter time — for the documentation creator.**
  - `CONTEXT.md` is the single highest-leverage borrow. Once a doc creator + dev team share a glossary, every downstream artifact (spec, PRD, code comment, test name, commit) names the same things the same way. The doc creator gets a canonical vocabulary to *write from*; the agent stops translating jargon every session.
  - `graph-docs-cli` solves the documentation-creator's structural problem: docs that are read in *learning order*. Specflow could ship a `--graph` mode for any doc skill output.
  - `to-prd`'s "numbered user stories + implementation decisions (modules, not files) + testing decisions" template is more durable than free-form PRDs — it survives codebase reshuffles.

- **(3) Fewer errors — for both personas.**
  - `diagnose`'s "build the feedback loop first" reframes debugging as a meta-skill. For an AI-new dev team, this is the single most important shift: the bug isn't fixed by reading code, it's fixed by *making the bug observable on demand*.
  - `tdd`'s vertical-slice rule prevents the most common failure mode of agent-driven test writing (bulk tests that pass while behavior is broken).
  - The ADR three-criterion gate prevents documentation rot — fewer stale rules means fewer stale errors.
  - `git-guardrails-claude-code` (block `push --force`, `reset --hard`, `clean -f`) is a one-line install that prevents an entire class of catastrophic agent errors. Specflow's setup skill should install this by default.
  - `caveman` mode reduces hallucination surface area by reducing token output — fewer words, fewer chances to drift.

- **Caution / non-fit.**
  - Matt's skills are designed to be *small and composable*; they explicitly call out GSD/BMAD/Spec-Kit as taking "control away." Specflow's positioning as a **pipeline** is closer to the latter camp. Borrowing patterns is fine; copying the philosophy wholesale would dilute specflow's "structured workflow" value prop. The right move is to absorb individual skills (CONTEXT.md, to-prd, diagnose) as *reusable primitives inside* specflow's pipeline, not replace the pipeline.

## Cross-references

- **Internal — specflow.**
  - `repo-karpathy-autoresearch.md` (sibling analysis): Karpathy's `program.md`-as-skill insight pairs with Matt's "instructions are themselves a versioned, human-edited Markdown file" — same idea, different domain. Both treat the prose as the program.
  - Specflow's existing skills directory (`plugins/specflow/skills/{linear, prd, prime, setup, task, test}`) maps roughly: `prime` ↔ `setup-matt-pocock-skills` + `CONTEXT.md`, `prd` ↔ `to-prd`, `linear` ↔ `to-issues` + `triage`, `test` ↔ `tdd` + `diagnose`. Each pairing is a borrow opportunity.
  - Memory rule "no premature pipeline CTAs" aligns with Matt's "create files lazily — only when you have something to write" (`grill-with-docs`). Both push back on speculative scaffolding.

- **External — Matt's ecosystem.**
  - Matt's newsletter: https://www.aihero.dev/s/skills-newsletter (60k+ devs; primary distribution channel for new skills).
  - Total TypeScript: https://totaltypescript.com (the educational platform behind `ts-reset`, `ts-error-translator`, etc.).
  - Books cited in `skills` README: *The Pragmatic Programmer* (Thomas & Hunt), *Domain-Driven Design* (Eric Evans), *Extreme Programming Explained* (Kent Beck), *A Philosophy of Software Design* (John Ousterhout). All four directly inform skill design — Ousterhout's "deep modules" is the central pillar of `improve-codebase-architecture`.
  - `@ai-hero/sandcastle` package: https://www.npmjs.com/package/@ai-hero/sandcastle.
  - `evalite` docs: https://www.evalite.dev/.
