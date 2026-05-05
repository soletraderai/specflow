# It Ain't Broke: Why Software Fundamentals Matter More Than Ever

## Identity
- Title: It Ain't Broke: Why Software Fundamentals Matter More Than Ever
- Speaker/Channel: Matt Pocock (AI Hero) — conference talk, AI Engineer
- Likely URL: https://www.youtube.com/watch?v=v4F1gFy-hqg
- Suggested slug: pocock-software-fundamentals-matter-more
- Confidence: high — speaker self-identifies via the "Mac PCO skills" GitHub repo (mattpocock/skills) and the aihero.dev newsletter; he names his "Claude Code for real engineers" course; signature skills referenced in the talk (grill me, ubiquitous language, improve codebase architecture) match Matt Pocock's open-sourced skills repo. Web search confirmed the talk title and YouTube URL.

## Thesis
"Specs-to-code" / vibe-coding fails because re-running the compiler on a spec produces ever-worse code (software entropy). Good codebases — built with classic fundamentals (deep modules, ubiquitous language, TDD, shared design concept) — are what let AI deliver real leverage. The human's job moves up a layer: strategic architect designing interfaces while AI handles the tactical implementation inside them.

## Key points

### KP-1: Code is not cheap — bad code is now the most expensive it has ever been
- **Point**: The premise behind specs-to-code ("just regenerate from the spec") fails because every regeneration drifts the system further. AI in a *good* codebase amplifies output; in a bad codebase it compounds entropy.
- **Why it matters to our goals**: Directly hits goals (1) productivity, (2) shorter time-to-product, (3) fewer errors. specflow should not sell the team on "ignore the code, edit the spec" — instead the plugin should treat code quality as a first-class input that determines whether AI is a multiplier or a multiplier-of-mess.
- **Evidence**: Speaker's own experiment — re-running specs-to-code produced "garbage"; cites Ousterhout (A Philosophy of Software Design) on complexity, and Pragmatic Programmer on software entropy.
- **Sources**: Transcript lines 41-56, 93-111, 457-468.

### KP-2: Reach a shared "design concept" with the AI before any plan asset
- **Point**: Most AI miss-fires stem from a missing shared mental model. Pocock's "grill me" skill makes the AI interrogate the human with 40-100 questions until alignment is reached, *then* produce a PRD or issues. He prefers this to Claude Code's default plan mode, which he says is "extremely eager to create an asset."
- **Why it matters to our goals**: For a docs-creator + AI-novice dev team, a forced grilling step prevents the most common failure (AI builds the wrong thing) and produces a written PRD as a byproduct that bridges the docs/dev handoff.
- **Evidence**: References Frederick P. Brooks' *Design of Design* — the "design concept" is the invisible shared theory between collaborators. Pocock's grill-me skill repo went viral (~13k stars on the skills repo).
- **Sources**: Transcript lines 121-189; mattpocock/skills GitHub.

### KP-3: Build a ubiquitous language file that AI and humans both use
- **Point**: AI verbosity is a symptom of language mismatch. Borrowing from Domain-Driven Design, maintain a markdown file of domain terms — a ubiquitous language — kept open during planning. AI thinks more concisely and implementation aligns better with the plan.
- **Why it matters to our goals**: Cheap, high-leverage artifact for a small team — one file does triple duty: docs creator's glossary, dev team's naming guide, AI's planning context. Reduces the docs↔code drift that causes errors.
- **Evidence**: Pocock has a "ubiquitous language" skill that scans the codebase, extracts terminology, and writes the file. He notes thinking traces become less verbose and implementations more aligned.
- **Sources**: Transcript lines 190-258.

### KP-4: AI outruns its headlights — TDD is a forcing function for small steps
- **Point**: AI tends to write huge batches before checking. Pragmatic Programmer: "the rate of feedback is your speed limit." TDD forces test-first → minimal pass → refactor, keeping AI in tight feedback loops. Static types, browser access for frontend, automated tests are non-negotiable feedback loops.
- **Why it matters to our goals**: A team new to AI coding will burn time on AI-induced regressions. Baking TDD-style cadence into the specflow pipeline (write test → get green → refactor) directly attacks goal (3) fewer errors and goal (1) productivity by shortening recovery loops.
- **Evidence**: Transcript lines 264-302.

### KP-5: Deep modules with simple interfaces — the codebase shape AI thrives in
- **Point**: Ousterhout: prefer few deep modules (lots of behavior behind a small interface) over many shallow ones. AI is *very good* at producing shallow-module sprawl, which is hard for both AI and humans to navigate. Deep modules are also easier to test (test at the interface).
- **Why it matters to our goals**: Architectural target for any code specflow generates or refactors. Suggests a specflow skill or check that measures interface surface vs. internal behavior and flags shallow-module sprawl. Shorter time + fewer errors because the AI can locate and modify code reliably.
- **Evidence**: Pocock has an "improve codebase architecture" skill: explore, find related code, wrap in deep module. Transcript lines 322-396.

### KP-6: Design the interface, delegate the implementation (gray-box trust)
- **Point**: With deep modules behind clean interfaces, you can review the *interface* carefully and let AI own the implementation as a gray box, verifying via the interface's tests. Saves human cognitive load. Don't do this for high-stakes domains (finance, etc.).
- **Why it matters to our goals**: Direct answer to the "small team, AI-novice devs" reality — they cannot deeply review every diff. Gives a principled rule for *where* to spend review attention. Prevents reviewer burnout (Pocock: "more tired than ever before in your career").
- **Evidence**: Transcript lines 397-436.

### KP-7: Plans must name modules and interface changes explicitly
- **Point**: PRDs should be specific about which modules change and how their interfaces are modified. Modules should be part of the ubiquitous language. This is "investing in the design every day" (Kent Beck) — the opposite of specs-to-code, which divests from design.
- **Why it matters to our goals**: Concrete schema improvement for any specflow PRD/spec template — require a "modules touched / interfaces changed" section. Makes plans reviewable and AI-actionable.
- **Evidence**: Transcript lines 437-456.

### KP-8: AI is a tactical sergeant; you are the strategist
- **Point**: Frame AI as an excellent on-the-ground programmer making local code changes. The human stays at the strategic level — system design, module boundaries, interfaces, domain language. That role requires the same fundamentals devs have used for 20+ years.
- **Why it matters to our goals**: Operating principle for the whole specflow plugin and for how the team is taught to use it. The docs creator + senior dev are strategists; AI executes. Sets correct expectations for the AI-novice devs (don't trust blindly, don't fear obsolescence).
- **Evidence**: Transcript lines 457-468.

### KP-9: Plan mode's eagerness to ship an asset is a failure mode
- **Point**: Direct critique of Claude Code's default plan mode — it rushes to produce a plan asset before alignment exists. Better to grill first (no asset), then write the asset.
- **Why it matters to our goals**: specflow is a Claude Code plugin — should explicitly position its grill/discovery phase *before* plan mode, or replace plan mode for non-trivial work. This is a concrete differentiator the plugin can offer.
- **Evidence**: Transcript lines 181-189.

### KP-10: Software entropy is the default — without design discipline, AI accelerates it
- **Point**: From Pragmatic Programmer: every change considered in isolation degrades the whole. AI making rapid local changes accelerates entropy unless human architectural attention counterbalances.
- **Why it matters to our goals**: Argues for periodic architectural-health passes inside specflow's pipeline (e.g. an "improve codebase architecture" step), not just feature-shipping steps.
- **Evidence**: Transcript lines 80-95.

## Tools / repos / frameworks mentioned
- mattpocock/skills (GitHub) — "Mac PCO skills" repo containing grill-me, ubiquitous-language, improve-codebase-architecture, writer-PRD skills
- Claude Code (Anthropic CLI) — speaker's primary tool; critiques default plan mode
- aihero.dev — speaker's newsletter and "Claude Code for Real Engineers" cohort
- Books: *A Philosophy of Software Design* (John Ousterhout); *The Pragmatic Programmer* (Hunt & Thomas); *The Design of Design* (Frederick P. Brooks); Domain-Driven Design (Eric Evans, implicit); Kent Beck quote on investing in design
- TypeScript (as a feedback loop), browser access for front-end LLM, automated tests, TDD

## Verification log
1. Transcript first-person clues: "Mac PCO skills" GitHub repo + "aihero.dev newsletter" + "Claude Code for real engineers" course → unambiguously Matt Pocock. Repo at github.com/mattpocock/skills confirmed.
2. WebSearch query "Matt Pocock 'Claude Code for real engineers' talk specs to code grill me skill" — confirmed skills, course, and persona.
3. WebSearch query "Matt Pocock conference talk 'code is not cheap' 'ubiquitous language' 'deep modules' Claude Code" — surfaced exact talk title "It Ain't Broke: Why Software Fundamentals Matter More Than Ever" with YouTube URL https://www.youtube.com/watch?v=v4F1gFy-hqg. Title, description, and themes match the transcript line-for-line. High confidence.
4. Conference context: applause + audience interaction in transcript matches AI Engineer / similar live event format; talk surfaced via daily.dev and StartupHub recap articles consistent with this being a talk Pocock gave in early 2026.
