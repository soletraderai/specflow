# Building a Real Feature with Claude Code, End to End

## Identity
- Title: Building a REAL feature with Claude Code: every step explained
- Speaker/Channel: Matt Pocock (AI Hero / @mattpocockuk)
- Likely URL: https://www.youtube.com/watch?v=hX7yG1KVYhI
- Suggested slug: pocock-real-feature-build-claude-code
- Confidence: high — speaker self-identifies via the "course video manager" repo (mattpocock/course-video-manager on GitHub), the "Grill Me" skill (from his "5 Agent Skills I Use Every Day" post on aihero.dev), and his "Claude Code for Real Engineers" cohort. Web search returned the exact video title and ID.

## Thesis
AI coding is not a paradigm shift — it amplifies the same fundamentals devs have used for 20 years: clear architecture, tight feedback loops, ubiquitous language, modular design, and human-in-the-loop thinking. The best leverage comes from spending unhurried human time at the front of the loop (grilling a vague idea into a hardened spec), then letting autonomous "Ralph" loops chew through scoped GitHub issues while you do QA in parallel. You review inputs, outputs, and interfaces — not most of the code.

## Key points

### KP-1: Treat the LLM as a junior teammate you delegate to
- **Point**: Get the most out of Claude Code by treating it like someone you delegate to: focus on architecture, feedback loops, and clear interfaces — not on planning every line up front.
- **Why it matters to our goals**: Sets the right mental model for the dev team new to AI. Stops them either over-prompting or treating Claude as a magic spec-to-code box.
- **Evidence**: Opens the video stating "you treat it like someone you would delegate to in your team... you focus on the architecture... good feedback loops... all the things we've been doing for the last 20 years."
- **Sources**: transcript lines 9-17

### KP-2: The "Grill Me" skill — adversarial spec hardening before any code
- **Point**: Pocock starts every feature by dumping a rough, dictated set of vague ideas at Claude and invoking a "Grill Me" skill that asks pointed clarifying questions, surfaces edge cases, and challenges woolly framing — for ~22 minutes before any code is written.
- **Why it matters to our goals**: Directly maps to specflow's "Discover" phase. Reduces errors by surfacing ambiguity early (cheapest place to catch bugs) and produces rich Q&A context the LLM can later reuse. Matches our "no premature pipeline CTAs" memory — humans must finish thinking before tooling moves on.
- **Evidence**: ~22 minutes of grilling produced an 8-bullet scoped feature list (lines 691-694). Skill is published at github.com/mattpocock/skills.
- **Sources**: transcript lines 132-694

### KP-3: Always tell the LLM the WHY, not just the WHAT
- **Point**: When stating requirements, explaining the why lets the LLM suggest alternatives; only the what means it can only build what you said.
- **Why it matters to our goals**: A small, repeatable rule the docs creator and devs can adopt immediately. Prevents the classic "built exactly what I asked, not what I needed" failure mode.
- **Evidence**: "Sure, if the LLM has the what, then it understands what you want to build. But if it doesn't know the why, then it can't suggest alternatives."
- **Sources**: transcript lines 169-184

### KP-4: Maintain a Ubiquitous Language file (DDD applied to LLMs)
- **Point**: Keep a glossary of domain terms (e.g., "ghost lesson", "materialize", "materialization cascade") that the LLM updates after each grilling session. The LLM is the dev; you're the domain expert; the glossary is your shared language.
- **Why it matters to our goals**: Massive productivity multiplier. Once "materialization cascade" is in the glossary, future prompts collapse from paragraphs to four words. Reduces miscommunication errors across a team that grows.
- **Evidence**: References Domain-Driven Design book; lives in repo, gets auto-updated after Grill Me, includes verbs, aliases-to-avoid, and named flows like "materialization cascade" (lines 247-273, 702-732).
- **Sources**: transcript lines 247-273, 700-732

### KP-5: Day shift / night shift — humans grill, agents implement AFK
- **Point**: The human's slow work (extracting ideas from your brain via grilling) happens while AFK agents in the background implement the previous session's PRD. "I'm doing the day shift, Claude does the night shift" (credit: Jamon on Twitter).
- **Why it matters to our goals**: Reframes "AI coding is slow" critique. Productivity gain comes from parallelism, not raw speed. For a small team, this means the docs creator can be grilling the next feature while devs and agents work the current one.
- **Evidence**: lines 1007-1027. After grilling+PRD+issues, he runs Ralph and goes for a walk; returns 90 minutes later to 6 commits.
- **Sources**: transcript lines 1007-1046

### KP-6: PRD -> GitHub issues -> Ralph loop pipeline
- **Point**: Workflow is grill -> auto-write PRD -> auto-break into 4-6 GitHub issues with blocking relationships -> Ralph loop (autonomous Claude with max 100 iterations) picks issues and closes them via commits.
- **Why it matters to our goals**: A concrete reference architecture for specflow. Issues are sized "not too big, not too small" — small enough the agent doesn't lose context, big enough the agent-spinup cost amortizes.
- **Evidence**: 6 issues generated, merged to 4 slices; 5 iterations produced 6 commits (lines 884-934, 1035-1046).
- **Sources**: transcript lines 750-934

### KP-7: Sandcastle — Docker-isolated agent runs with patch extraction
- **Point**: His "Sandcastle" setup runs Claude inside a Docker container with the working dir mounted; commits made inside are pulled out as patches and applied to local repo. Lets him run Ralph loops repeatedly with different inputs.
- **Why it matters to our goals**: Pattern for safe AFK execution — devs new to AI coding need a sandbox so Claude can't damage the host repo or leak secrets. Reduces a major class of errors.
- **Evidence**: lines 962-989.
- **Sources**: transcript lines 962-989

### KP-8: Use sub-agents (`explore`) for token-efficient context gathering
- **Point**: The `explore` skill spawns a sub-agent in its own context window to read tons of files, then returns only a summary to the parent. Used multiple times per session.
- **Why it matters to our goals**: Token efficiency = cost savings + longer effective context = better output. This is a pattern specflow can encode as a default skill.
- **Evidence**: lines 209-227. After all the grilling, parent context still only at ~40k tokens (line 773).
- **Sources**: transcript lines 209-227, 770-776

### KP-9: Don't review the PRD or issues — trust LLM summarization
- **Point**: He explicitly skips reviewing the auto-written PRD and the auto-generated issues, because (a) LLMs are good at summarization, (b) he already pre-reviewed via the Q&A in grilling.
- **Why it matters to our goals**: Counterintuitive but practical. Saves time. The trade-off: front-load review effort into the grilling, not the artefacts.
- **Evidence**: "Am I going to review this PRD? No, I'm not going to. LLMs are really good at summarizing things." (lines 884-890, 935-940).
- **Sources**: transcript lines 884-940

### KP-10: Review interfaces, not implementations
- **Point**: When the PRD-writer skill sketches modules, he checks the interface changes (new methods, API routes) but not the inside of modules. He cares about testability and how future agents will read the code.
- **Why it matters to our goals**: This is the discipline that makes the whole pipeline scale. Devs new to AI coding tend to either rubber-stamp everything or read every line; neither scales.
- **Evidence**: "Notice how I'm thinking about the interface more than I'm actually thinking about the implementation here" (lines 793-828).
- **Sources**: transcript lines 793-828, 1295-1316

### KP-11: Question + Answer is high-attention LLM food
- **Point**: He prefers Q&A grilling format (over pure spec docs) because LLM attention mechanisms hot-spot collocated content. The transcript of the grilling session becomes input to the PRD writer.
- **Why it matters to our goals**: Justification for why specflow's discover phase should preserve Q&A pairs verbatim, not just a cleaned summary.
- **Evidence**: "I freaking love question and answer because it collocates the question with the answer... attention mechanisms... hot spot" (lines 758-770).
- **Sources**: transcript lines 758-770

### KP-12: "Look harder" — push back when the LLM gets it wrong
- **Point**: When Claude wrongly claimed there were no test harnesses, he replied "look harder" — and Claude found them. Don't accept the first wrong answer.
- **Why it matters to our goals**: Simple, teachable behaviour. Reduces errors from premature LLM agreement / hallucinated codebase facts.
- **Evidence**: "It's saying there's no existing test harness... That is rubbish. So I'm going to do a rafiki. I'm going to say look harder. And there we go." (lines 866-873).
- **Sources**: transcript lines 866-873

### KP-13: Tests + types must run on every commit in the Ralph loop
- **Point**: Crucial to Ralph-loop success: every iteration runs tests and types before commit. Without this feedback loop, autonomous agents drift.
- **Why it matters to our goals**: Single biggest "fewer errors" lever for AFK agent runs. Should be a non-negotiable in any specflow autonomous mode.
- **Evidence**: "Something that's crucial to the success of these Ralph loops is making sure it runs tests and types on every single commit." (lines 1251-1257).
- **Sources**: transcript lines 1251-1257

### KP-14: QA in parallel with new fixes via in-app feedback button -> issue
- **Point**: His app has a "feedback" button that captures the current route and the user's dictated feedback, uses Haiku to title it, files a GitHub issue. Ralph picks up issues continuously — so QA findings get fixed while he keeps QAing.
- **Why it matters to our goals**: A pattern for the "iterate in QA" phase. The bottleneck becomes human QA throughput, not implementation. For a docs-creator + devs team, an analogous in-product feedback channel could be huge.
- **Evidence**: lines 1086-1127.
- **Sources**: transcript lines 1086-1127

### KP-15: Human-in-the-loop label keeps Ralph from breaking things
- **Point**: Issues labelled `human-in-the-loop` (or filenamed AFK-incompatible) are skipped by Ralph. The Ralph prompt is taught to ignore them.
- **Why it matters to our goals**: Concrete safety primitive for autonomous loops — there must be a way to mark "humans only" so the agent doesn't barge in.
- **Evidence**: "I've got something in my prompt that says if there's a human in the loop like label on it... don't work on it." (lines 1117-1122).
- **Sources**: transcript lines 1117-1122

### KP-16: Specs-to-code alone never works — edge cases only emerge in QA
- **Point**: Pure pre-planning will miss edge cases (e.g., directory creation succeeds but git init fails, leaving DB and FS out of sync). You only find them once you're iterating in QA.
- **Why it matters to our goals**: Important counterweight to over-rigid spec-driven workflows. specflow should plan for an iterate-in-QA loop, not assume the PRD is final.
- **Evidence**: lines 1190-1212. Real bug found: "if the course repo is not a git repository... we should walk back the creation of the directory."
- **Sources**: transcript lines 1190-1212

### KP-17: Don't use the built-in AskUserQuestion tool — token cost
- **Point**: His Grill Me skill deliberately doesn't call the AskUserQuestion tool because tool calls cost JSON-wrapping overhead. Plain conversation is cheaper.
- **Why it matters to our goals**: A nuanced cost-tuning insight. May or may not apply to specflow, but worth knowing when designing skills.
- **Evidence**: lines 777-790.
- **Sources**: transcript lines 777-790

### KP-18: Rough size issues to amortize agent spinup
- **Point**: Issues should be neither tiny ("we pay the cost of kicking up an entire agent") nor huge. ~4-6 issues per PRD, each spanning UI/schema/API as a coherent slice.
- **Why it matters to our goals**: Calibration heuristic for the slicing step in specflow. Bad slicing wastes either tokens or context.
- **Evidence**: "If they're too small then we pay the cost of like having to kick up an entire agent... ghost course creation maybe... seems decently sized because it touches the UI, schema, API." (lines 902-934).
- **Sources**: transcript lines 902-934

## Tools / repos / frameworks mentioned
- Claude Code (Anthropic CLI)
- Claude Skills: Grill Me, Explore (sub-agent), PRD writer, PRD-to-issues — at github.com/mattpocock/skills
- Sandcastle — Pocock's Docker-based AFK harness (provisional, in his repo)
- Ralph loop — autonomous loop running Claude with max ~100 iterations against a backlog of issues
- "Claude Code for Real Engineers" cohort — aihero.dev
- mattpocock/course-video-manager (GitHub) — the live demo repo (~1,200 commits, 637 closed issues)
- Stack: React Router, TypeScript, Node, Drizzle ORM, Postgres, Vitest, Effect (TypeScript)
- Domain-Driven Design (Eric Evans) — book referenced for ubiquitous language
- Haiku (Anthropic) — used to title feedback-form issues
- "Side question" feature in Claude Code — ask without polluting chat history
- Dictation tool (unnamed) — for voice-to-text spec dumps

## Verification log
- WebSearch query 1: "Matt Pocock Claude code 'grill me' 'course video manager' cohort YouTube" — returned exact video URL https://www.youtube.com/watch?v=hX7yG1KVYhI, GitHub repo, and aihero.dev cohort page. High confidence on identity.
- WebSearch query 2: "'Building a REAL feature with Claude Code' YouTube hX7yG1KVYhI date" — confirmed title, creator, upload date 2026-03-18, and accompanying article at aihero.dev/real-world-feature-build-with-claude-code.
- Cross-check: speaker mentions "course video manager" (lines 45-65), "Claude Code cohort" (line 33), "Willow Reagan my old boss" (line 484), "Effect" (lines 320-331), all consistent with public Pocock content.
