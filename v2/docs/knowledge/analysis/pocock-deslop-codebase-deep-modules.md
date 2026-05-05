# De-Slopping AI-Ruined Codebases with Deep Modules

## Identity
- Title: How To De-Slop A Codebase Ruined By AI (with one skill)
- Speaker/Channel: Matt Pocock (AI Hero / @mattpocockuk)
- Likely URL: https://www.youtube.com/watch?v=3MP8D-mdheA
- Suggested slug: pocock-deslop-codebase-deep-modules
- Confidence: high — transcript references "my GitHub skills repo, currently sitting at 41.5K stars" (matches mattpocock/skills), the "improve codebase architecture" skill (matches `skills/engineering/improve-codebase-architecture/SKILL.md` in that repo), Ousterhout's *A Philosophy of Software Design*, the AI Hero newsletter, and the "San Castle" video reference (sandcastle / dexhorthy AFK agents). Web search confirmed the exact title.

## Thesis
AI hasn't broken software fundamentals — it has accelerated software entropy, so codebases rot into "balls of mud" faster than ever. The cure is 20-year-old design discipline: rebuild around **deep modules** (small interfaces hiding lots of implementation) connected at well-defined **seams**. An LLM can scout for deepening opportunities ("tactical sergeant"), but a human must make the strategic calls. Tests live at seams; the better the seams, the better the agent output.

## Key points

### KP-1: AI accelerates entropy unless you actively counter it
- **Point**: "AI has simply accelerated software entropy. Codebases are falling apart faster than they ever have before." Every change made without full-codebase context introduces friction that snowballs.
- **Why it matters to our goals**: Directly threatens goals (2) better product and (3) fewer errors. A new-to-AI dev team is the highest-risk profile for entropy buildup. specflow needs guardrails that force codebase-aware changes.
- **Evidence**: Transcript lines 1–17.
- **Sources**: Pocock's framing of "ball of mud" and the prevention-vs-cure framing.

### KP-2: Shared vocabulary with the AI is a productivity multiplier
- **Point**: Pocock added a **glossary of terminology** (module, interface, implementation, deep/shallow, seam, adapter, locality, leverage) to his skill so human and AI "talk using the same language ... a lot more precise with what you're asking for."
- **Why it matters to our goals**: Goal (1) productivity. specflow's docs creator + dev team would benefit from a canonical glossary in CLAUDE.md / a skill so prompts and reviews are unambiguous.
- **Evidence**: Lines 36–51.
- **Sources**: Glossary section of `improve-codebase-architecture/SKILL.md`.

### KP-3: Deep modules — the core primitive
- **Point**: A *deep module* hides lots of implementation behind a small interface (e.g. TanStack Query). Shallow modules — complex interface, thin implementation — are the antipattern. Idea is from John Ousterhout's *A Philosophy of Software Design*.
- **Why it matters to our goals**: Deep modules give callers high leverage per unit of API learned — the leverage AI agents need to write correct code without re-reading the whole repo every time. Direct lever for goals (1) and (3).
- **Evidence**: Lines 79–108.
- **Sources**: Ousterhout, APoSD.

### KP-4: Seams are where you test, mock, and replace
- **Point**: A *seam* is the location where a module's interface lives — the boundary between modules. Seams are where unit/integration tests and mocks attach. Choosing seam locations is "crucial to getting a good architecture."
- **Why it matters to our goals**: Test harnesses around seams convert "scary changes" into safe ones — exactly what specflow's test/QA pipeline should encode.
- **Evidence**: Lines 117–129.
- **Sources**: —

### KP-5: Adapters satisfy seams (hexagonal architecture)
- **Point**: Once a seam exists, you need a concrete *adapter* satisfying its interface. Example: a real clock vs. a fake clock — both implement the same seam interface; the fake unblocks tests that would otherwise need to wait two weeks of wall time.
- **Why it matters to our goals**: Goal (3) fewer errors and goal (1) productivity — fake adapters at seams keep tests fast and deterministic, which agents rely on.
- **Evidence**: Lines 130–147.
- **Sources**: Hexagonal architecture / ports-and-adapters.

### KP-6: Locality + leverage are the two health metrics
- **Point**: Maintainers want **high locality** (changes and bugs concentrate in one module, not scattered). Callers want **high leverage** (more capability per unit of interface learned). These are the *two attributes* to optimise when refactoring.
- **Why it matters to our goals**: Two concrete, measurable design heuristics specflow can encode in review checklists and skills.
- **Evidence**: Lines 148–167.
- **Sources**: —

### KP-7: Agent = tactical sergeant, human = strategic general
- **Point**: "Agents are really, really good tactical programmers ... but they need someone on the level above them who is the strategic programmer." The skill demands judgment calls; it is explicitly **not an AFK skill**.
- **Why it matters to our goals**: This is the core operating model specflow should bake in — auto-mode is dangerous for architecture decisions. Aligns with our memory-feedback "no premature pipeline CTAs" — humans gate strategic transitions.
- **Evidence**: Lines 290–311.
- **Sources**: —

### KP-8: Run the skill on a recurring cadence (every couple of days)
- **Point**: "I recommend you run this skill every couple of days, especially in a fast-moving codebase. You'll come up with tons of opportunities for deepening."
- **Why it matters to our goals**: A scheduled architecture-review ritual prevents the entropy snowball. Candidate for a specflow skill or scheduled trigger.
- **Evidence**: Lines 311–318.
- **Sources**: —

### KP-9: Auto mode breaks human-in-the-loop skills
- **Point**: Pocock explicitly turns auto mode off for this skill: "Auto mode does some funny things with these human-in-the-loop style flows."
- **Why it matters to our goals**: specflow skills that require approval gates must explicitly disable / warn against auto/yolo modes, otherwise the gate is bypassed.
- **Evidence**: Lines 183–186.
- **Sources**: —

### KP-10: The "grilling session" pattern — AI proposes, human interrogates
- **Point**: The skill enters a **grilling session** where it surfaces concrete code on both sides of a candidate, asks design questions, and only then proposes a module shape and TypeScript interface. The user answers each question thoughtfully.
- **Why it matters to our goals**: A reusable interaction pattern for any specflow skill that touches design decisions — pose discrete questions, don't just generate a plan.
- **Evidence**: Lines 230–278.
- **Sources**: —

### KP-11: Output flows into an issue tracker, then to AFK agents
- **Point**: Once design is settled, "you can take that and put that in as a GitHub issue ... which can then be picked up by an AFK agent" (he points to his Sandcastle/dexhorthy video). Workflow: human reasons → issue → autonomous agent implements.
- **Why it matters to our goals**: Matches specflow's intended pipeline shape. The strategic decision (where the human cost is highest) happens in chat; mechanical implementation is delegated.
- **Evidence**: Lines 277–290.
- **Sources**: dexhorthy / Sandcastle; `to-prd` / `to-issues` skills.

### KP-12: First detection pattern — duplicated parallel implementations across a missing seam
- **Point**: Concrete example flagged by the skill: an "insertion point" concept implemented twice (front-end and back-end) with no shared seam, "the seam where they must agree is untested" — front-end can drift from back-end silently.
- **Why it matters to our goals**: A specific smell to teach specflow's test/review skills to look for — duplicated logic across boundaries with no contract test is a high-value refactor target.
- **Evidence**: Lines 196–224.
- **Sources**: —

### KP-13: Tests are the entry ticket for AI on a legacy codebase
- **Point**: For legacy/bad codebases, "what you really need before you start making changes is a harness around the codebase ... you need tests, testing really nice deep modules that have a lot of leverage and locality." Better tests → better agent output.
- **Why it matters to our goals**: Direct prescription for goal (3) fewer errors. specflow's onboarding-to-AI flow for any existing repo should start with: identify deep modules → add seam tests → only then let the agent loose.
- **Evidence**: Lines 318–342.
- **Sources**: —

### KP-14: "Legacy code" is just "code that's hard to change"
- **Point**: "When we talk about legacy codebases, what we really mean are bad codebases. Codebases that are hard to make changes in." Reframes legacy as a structural property, not an age property.
- **Why it matters to our goals**: Useful framing for the docs creator — quality criterion for docs is "does this make change easier or harder?"
- **Evidence**: Lines 328–333.
- **Sources**: Michael Feathers' *Working Effectively with Legacy Code* (implicit).

### KP-15: "Tactically improve" is a step, not a one-shot
- **Point**: The skill explores → proposes candidates (six in the demo) → user picks one → grilling → proposed shape → design decisions → issue. It's a deliberate, sequential ritual, not "fix everything at once."
- **Why it matters to our goals**: specflow's complex skills should be sequenced this way — one candidate, one decision, one issue per pass — to keep humans in control.
- **Evidence**: Lines 196–278.
- **Sources**: —

## Tools / repos / frameworks mentioned
- **mattpocock/skills** GitHub repo (~41.5K stars at recording; the `improve-codebase-architecture` skill is the focus)
- **Claude Code** (skill runtime; auto mode discussed)
- **John Ousterhout — *A Philosophy of Software Design*** (deep modules, depth/leverage)
- **Hexagonal architecture** (ports & adapters terminology)
- **TanStack Query** (cited as exemplar of a deep module)
- **React Router** + **effect.ts** (the demoed codebase: Pocock's course-video-manager)
- **Sandcastle / dexhorthy AFK agent** (referenced as the downstream picker of generated issues)
- `to-prd` / `to-issues` (Pocock's companion skills for converting design discussion into trackable issues)
- **AI Hero newsletter / skills documentation site** (forthcoming at recording)

## Verification log
- WebSearch "Matt Pocock improve codebase architecture skill deep modules YouTube" → top result: "How To De-Slop A Codebase Ruined By AI (with one skill)" at https://www.youtube.com/watch?v=3MP8D-mdheA. Confirms title and speaker.
- Repo `mattpocock/skills` confirmed; `skills/engineering/improve-codebase-architecture/SKILL.md` exists — matches transcript's "improve codebase architecture skill."
- Star count "41.5K" in transcript is consistent with current scale of mattpocock/skills.
- Ousterhout reference cross-checked: *A Philosophy of Software Design* is the canonical source for deep modules / depth-as-leverage.
- "San Castle" in transcript is auto-caption for **Sandcastle** (Hamel/dexhorthy ecosystem of AFK / autonomous coding agents); not verified independently but consistent with Pocock's known network.
