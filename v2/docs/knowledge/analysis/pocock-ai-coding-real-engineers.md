# Matt Pocock — AI Coding For Real Engineers (Full Workshop)

## Identity
- Title: [FULL WORKSHOP] AI Coding For Real Engineers
- Speaker/Channel: Matt Pocock (AI Hero / mattpocockuk) — delivered at AI Engineer conference
- Likely URL: https://www.youtube.com/watch?v=-QFHIoCo-Ko
- Suggested slug: `pocock-ai-coding-for-real-engineers`
- Confidence: high — speaker self-identifies as "Matt", references his repo `github.com/mattpocock/course-video-manager`, his AI Hero site, his "grill me" skill (which has its own AI Hero post and went viral on HN), and his Sand Castle library. WebSearch matches this exact workshop title at the URL above.

## Thesis
Software engineering fundamentals from the pre-AI era (Pragmatic Programmer, Fowler, Ousterhout) work *better* with LLMs than fancy "specs-to-code" pipelines. The job is to keep the LLM in its "smart zone" (~100k tokens), reach a *shared design concept* with it via aggressive interviewing, then split work into vertical slices on a Kanban board so agents can run AFK while humans stay in the loop on planning and QA.

## Key points

### KP-1: The "smart zone" is ~100k tokens — bigger context windows just ship more dumb zone
- **Point**: LLMs degrade quadratically as context grows; ~100k tokens is the practical ceiling for *coding* quality regardless of whether the window is 200k or 1M. Larger windows help retrieval, not coding.
- **Why it matters to our goals**: Productivity (1) and fewer errors (3) — sizing tasks to stay under ~100k is a hard constraint our workflow must respect. Specflow skills/commands should aim to keep working context lean.
- **Evidence**: "It doesn't matter whether you're using 1 million context window or 200k. It's always going to be about this... around 100k is kind of my new marker"; "they shipped a lot more dumb zone to you essentially."
- **Sources**: Workshop, ~lines 98–110, 965–991. Concept attributed to Dex (HumanLayer) — "smart zone / dumb zone."

### KP-2: Prefer /clear over /compact — treat each session like Memento
- **Point**: Compacting carries "sediment" that pollutes the next phase. Clearing returns to a clean system prompt; pair this with persisted artifacts (PRD, issues) so context is rebuilt deterministically each loop.
- **Why it matters to our goals**: Fewer errors (3). Our skills should write durable artifacts, not rely on long-lived chat memory.
- **Evidence**: "Devs love compacting for some reason, but I hate it. I much prefer my AI to behave like the guy from Memento because this state is always the same."
- **Sources**: ~lines 271–285.

### KP-3: Specs-to-code is vibe coding by another name — keep the code as the battleground
- **Point**: Pocock explicitly rejects the spec-driven workflow where you only edit specs and re-generate code. You must continuously shape the codebase; specs alone rot.
- **Why it matters to our goals**: Better product (2), fewer errors (3). Tension with parts of specflow's positioning — worth debating whether our pipeline over-trusts the spec layer.
- **Evidence**: "This is kind of like vibe coding by another name where you're essentially ignoring the code... I tried this. I really tried it and it sucks."
- **Sources**: ~lines 326–355. **Flag**: this is the most provocative claim for us — directly challenges any "spec is source of truth" framing. Worth a debate point.

### KP-4: The "grill me" skill — relentless interview to reach a shared design concept
- **Point**: Before writing anything, run a tiny skill that interviews the human one question at a time, with a *recommended answer* for each, until shared understanding is reached. Often 22–80+ questions. The output is the conversation history, not a doc.
- **Why it matters to our goals**: Better product (2) + fewer errors (3). Misalignment is the #1 failure mode; a grilling step catches assumptions ("should points be retroactive?") that humans miss.
- **Evidence**: "Interview me relentlessly about every aspect of this plan until we reach a shared understanding... For each question, provide your recommended answer."; cites Frederick Brooks' "design concept."
- **Sources**: ~lines 317–456. Skill is public on `github.com/mattpocock/skills`.

### KP-5: Two task types — Human-in-the-Loop vs AFK
- **Point**: Planning/alignment cannot be looped — humans must sit there. Implementation can be AFK. Designing skills around this distinction is fundamental.
- **Why it matters to our goals**: Productivity (1). Specflow should explicitly tag steps HITL vs AFK and not pretend planning can be automated.
- **Evidence**: "Planning, this alignment phase has to be human in the loop, has to be."
- **Sources**: ~lines 691–707.

### KP-6: Destination doc (PRD) + Journey doc (Kanban of issues), not multi-phase plans
- **Point**: PRD = where we're going (problem, solution, user stories, implementation/testing decisions, out-of-scope). Then a separate skill (`prd-to-issues`) breaks PRD into independently-grabbable issues with blocking relationships → a DAG, not a sequential plan.
- **Why it matters to our goals**: Productivity (1) — DAGs enable parallel agents; sequential plans only one agent.
- **Evidence**: "I prefer a Kanban board set up like this to a sequential plan because a sequential plan can really only be picked up by one agent."
- **Sources**: ~lines 758–823, 1039–1075, 1289–1336.

### KP-7: Vertical slices ("tracer bullets") beat horizontal layers
- **Point**: AI defaults to coding all DB → all API → all frontend. That blocks feedback until phase 3. Force thin vertical slices so each issue produces a runnable, reviewable end-to-end result.
- **Why it matters to our goals**: Fewer errors (3) + shorter time (2). This is a concrete rule we could bake into a `/prd-to-issues` skill prompt.
- **Evidence**: "AI loves to code horizontally... you don't get feedback on your work until you've really started or completed phase three"; cites Pragmatic Programmer.
- **Sources**: ~lines 1085–1218.

### KP-8: TDD is non-negotiable; AI cheats at tests if you let it write them last
- **Point**: Red-green-refactor with the test written *before* implementation. Otherwise the AI writes implementation, then writes tests that match its own (possibly wrong) code.
- **Why it matters to our goals**: Fewer errors (3). Our test skill should enforce TDD ordering, not just "write tests."
- **Evidence**: "It tends to try to cheat at the tests because it's sort of doing it in layers."
- **Sources**: ~lines 1733–1771.

### KP-9: Quality of feedback loops is the ceiling on AI code quality
- **Point**: If `npm test` and `npm typecheck` are slow, flaky, or absent, AI is "coding blind." Investing in fast deterministic feedback loops raises the ceiling more than prompt tweaks.
- **Why it matters to our goals**: Fewer errors (3). Specflow should probably have a "feedback loop audit" skill before anything else runs.
- **Evidence**: "The quality of your feedback loops influences how good your AI can code... that is the ceiling."
- **Sources**: ~lines 1810–1823.

### KP-10: Review in a fresh context, not the implementer's context
- **Point**: If the implementer agent has burned 80k tokens, asking it to review its own work means reviewing in the dumb zone. Clear context first → review in smart zone.
- **Why it matters to our goals**: Fewer errors (3). Our review skills should mandate fresh context.
- **Evidence**: "If you clear the context, then you're essentially going to be able to just review in the smart zone."
- **Sources**: ~lines 1707–1722.

### KP-11: Push vs pull for coding standards
- **Point**: Implementer agents should *pull* coding standards (via skills they choose to invoke) — keeps their context lean. Reviewer agents should have standards *pushed* into context — they need to compare against everything.
- **Why it matters to our goals**: A useful design principle for how we structure our skill files vs CLAUDE.md content.
- **Evidence**: "In the reviewer I would push the coding standards. In the implement I would allow it to pull."
- **Sources**: ~lines 2266–2306, 2370–2378.

### KP-12: Deep modules (Ousterhout) make codebases AI-friendly
- **Point**: Many tiny "shallow" modules force AI to track dependencies across the whole graph and produce bad test boundaries. Deep modules with small interfaces and lots of internal functionality let you draw one test boundary around the whole thing — better tests, better AI work.
- **Why it matters to our goals**: Better product (2), fewer errors (3). For a small dev team adopting AI, this is architectural advice that compounds. Worth referencing in our docs.
- **Evidence**: Cites Ousterhout's *Philosophy of Software Design*; Pocock's `improve-codebase-architecture` skill scans for shallow→deep candidates.
- **Sources**: ~lines 1916–2009, 2096–2141.

### KP-13: Design interfaces, delegate implementations
- **Point**: To stay sane while moving fast, humans own the *shape* (module map, interfaces) and delegate the *insides* to AI. This preserves codebase mental model without reading every line.
- **Why it matters to our goals**: Productivity (1) without sacrificing fewer errors (3). Direct answer to "I know my codebase less well than I used to."
- **Evidence**: "Design the interface for these modules, but then delegate the implementation... they become like gray boxes."
- **Sources**: ~lines 2061–2091.

### KP-14: Don't keep PRDs in the repo after merge — doc-rot poisons future agents
- **Point**: Old PRDs misalign future Claude sessions because code drifts from the original spec. Pocock closes them as GitHub issues (visible record, not in active context). **Note**: this contradicts what some teams do, including possibly our specflow defaults.
- **Why it matters to our goals**: Fewer errors (3). Worth explicit decision in specflow: keep, archive, or delete?
- **Evidence**: "It's almost unrecognizable... this is doc rot... I tend to not keep it around."
- **Sources**: ~lines 2156–2192. **Flag**: Pocock himself frames this as "a question that doesn't have a clear answer." Genuinely contested.

### KP-15: Don't over-optimize the PRD
- **Point**: Polishing the PRD into perfection has low ROI. The real value is in the grilling session (alignment) and downstream QA. PRD is just a hint of direction.
- **Why it matters to our goals**: Productivity (1). Avoid spending iterations refining specs.
- **Evidence**: "I don't think there's a lot of value in that... the place that you need to be putting the work is in QA."
- **Sources**: ~lines 2226–2253. **Flag**: Counter to a lot of "spec quality matters most" thinking.

### KP-16: Manual QA is where human taste re-enters
- **Point**: Teams that automate idea→PRD→implement→QA end up with "slop." Humans must QA manually to inject taste, opinion, and product judgment. QA *generates new issues* that flow back into the Kanban — QA is not the end of the loop, it feeds the loop.
- **Why it matters to our goals**: Better product (2). Specflow should not pitch full automation; QA is the human re-insertion point.
- **Evidence**: "If you try to automate the QA, automate the research... you end up with apps that lack taste and are bad."
- **Sources**: ~lines 1881–1898, 2438–2450.

### KP-17: Sandboxed parallelization with worktrees + Docker (Sand Castle pattern)
- **Point**: For real parallel agent work: use git worktrees in Docker containers, with Planner / Implementer / Reviewer / Merger agents. Pocock uses Sonnet for implementation, Opus for review.
- **Why it matters to our goals**: Productivity (1). If we ever go parallel, this is the architecture. **Flag**: heavy infra; small team may not need this yet.
- **Evidence**: "Sandboxes it in a Docker container... for each issue, we create a sandbox... I'm using Sonnet for implementation and Opus for reviewing."
- **Sources**: ~lines 2310–2378.

### KP-18: Own your stack — beware framework lock-in (taskmaster, openspec, specit)
- **Point**: At this stage of AI coding maturity, there's no clear winner among planning frameworks. Students who lean on a framework lose observability and can't debug when it breaks. Build your own thin layer.
- **Why it matters to our goals**: Better product (2). Validates specflow being skill-based and inspectable rather than wrapping a heavyweight framework. Also a self-check: are *we* a framework people would over-rely on?
- **Evidence**: "You need to own as much of your planning stack as you possibly can... they don't own the stack and they don't have observability over the whole thing."
- **Sources**: ~lines 607–634.

### KP-19: Bad codebases produce bad AI output (mutual reinforcement)
- **Point**: Garbage in, garbage out applies to codebases — AI working in a bad codebase produces bad code. Investing in codebase quality compounds AI productivity.
- **Why it matters to our goals**: All three goals. Our team-onboarding story should include "fix the worst feedback-loop and module-shape issues first."
- **Evidence**: "Bad code bases make bad agents."
- **Sources**: ~lines 951–961.

### KP-20: Track token usage live as a discipline
- **Point**: Pocock keeps a status line showing exact token count visible at all times. Knowing how close you are to ~100k drives decisions to clear/restart.
- **Why it matters to our goals**: Productivity (1) + fewer errors (3). Small habit, big leverage. Could be a `/status` or `/tokens` skill.
- **Evidence**: "Essential information on every coding session... so that you know how close you are to the dumb zone."
- **Sources**: ~lines 248–258.

## Tools / repos / frameworks mentioned
- **`mattpocock/skills`** — public skill collection: grill-me, write-a-prd, prd-to-issues, improve-codebase-architecture, ralph (https://github.com/mattpocock/skills)
- **`mattpocock/course-video-manager`** — his real working repo with 744 closed issues showing the workflow in production
- **Sand Castle** — TypeScript library for AFK agent loops with sandboxed worktrees, planner/implementer/reviewer/merger pattern (Pocock-built)
- **Ralph (Wiggum) loop** — sequential AFK loop pattern; attribution unclear in the talk but predates Pocock
- **HumanLayer (Dex Horthy)** — "smart zone / dumb zone" terminology
- **Beads** — Steve Yegge's framework for managing Kanban/issues; Pocock hasn't tried it but says "seems very good"
- **Books referenced as "gold mines"**: *Pragmatic Programmer* (tracer bullets), Fowler's *Refactoring*, Frederick Brooks' *Design of Design*, Ousterhout's *Philosophy of Software Design*
- **Other planning frameworks (mentioned but not endorsed)**: taskmaster, openspec, specit
- **TLDraw** — what Pocock uses instead of slides

## Verification log
- Searched: "Matt Pocock AI workshop grill me Ralph Wiggum AFK Sand Castle" → confirmed AI Hero (mattpocockuk), grill-me skill viral on HN, Sand Castle is his TypeScript lib for parallel AFK agents.
- Searched: "Matt Pocock AI Hero workshop two hour PRD vertical slices" → confirmed full workshop title is "[FULL WORKSHOP] AI Coding For Real Engineers" at https://www.youtube.com/watch?v=-QFHIoCo-Ko, posted on AI Engineer channel. Workflow tweet confirmed: Idea → /write-a-prd → PRD → /prd-to-issues → Kanban → ralph.sh → Ralph Loop → Manual QA.
- Speaker self-identification at line 5–7 ("My name is Matt. I'm a teacher and now I teach AI") plus reference to "AI Hero" website at line 252 and `github.com/mattpocco/course-video-manager` (typo for mattpocock) at line 879 — converging evidence is unambiguous.
