# Building a Real Feature End-to-End — Grill, PRD, Issues, AFK Ralph, QA

**Author:** Matt Pocock
**Source:** YouTube long-form walkthrough (live coding session)
**Duration:** ~44:18

## Executive Summary
A real-time, ~44-minute build of a non-trivial feature ("ghost courses" — courses that exist in the database with no on-disk file path until materialised) on Pocock's actual course-management codebase. He demonstrates the full pipeline: grill-me to reach a shared design concept, ubiquitous-language updates as new terms emerge ("materialize", "materialization cascade", "ghost course"), write-PRD to capture the destination, PRD-to-issues to break the work into independently-grabbable vertical slices, an AFK Ralph loop running in a Docker sandbox to implement them, and a QA loop where a feedback button feeds bugs straight back into Ralph while he keeps testing. The big lesson: real features surface edge cases that grilling never catches — you cannot specs-to-code your way past QA.

## The Feature Being Built
A "ghost course" is a course you can plan inside the app without yet committing it to disk. The work covers:

- DB schema: `courses.file_path` becomes nullable.
- New "create ghost course" flow that asks only for a name.
- A **materialization cascade**: creating a real lesson inside a ghost course assigns a file path to the course, materialises the section, and creates the lesson on disk — all in one flow.
- New direct create-real-lesson and direct delete-real-lesson actions.
- UI: hide publish/export on ghost courses; create-real-lesson button available everywhere.
- Existing `convert to ghost` action stays for real lessons you want to keep planning.

## Step 1 — Grilling for a Shared Design Concept
Open a clean Claude session. Run `/grill-me` against the feature description. The skill explores the repo, asks ~22 questions one at a time, each with a recommended answer.

> **Principle (why grilling beats plan mode):** "I needed to reach a shared understanding. I didn't need an asset, I didn't need a plan. I needed to be on the same wavelength as the AI as my agent."

Examples of questions the AI generated (all real, all load-bearing):

- "When you say a ghost course could have real lessons, what does *real* mean without a file system?" — forces him to be specific.
- "Does direct-create apply inside ghost courses too?" — surfaces an interaction Pocock hadn't considered.
- "You have a real course with a file path, real lessons and ghost lessons. You delete all the real lessons. Does the real course become a ghost?" — answer: no, once a course has a file path it stays real forever.

The AI also probes flow choices — modal vs. inline, repo creation vs. pointing at existing directory — and Pocock keeps narrowing.

## Step 2 — Updating the Ubiquitous Language *During* Grilling
A core technique. As soon as a new term emerges ("materialize", "materialization cascade"), he stops the grilling and asks the AI to update the ubiquitous-language doc.

> **Direct from video:** "I get it to update my ubiquitous language document to basically just keep it up to date with any of the ideas that I've got in here."

The doc gains entries like:

- *materialize* — "the act of transitioning a ghost entity to a real entity by creating its on-disk representation," with `create on disk` and `realize` listed as aliases to avoid.
- *materialization cascade* — "the chain reaction when materializing a lesson inside a ghost course: assigns file path to course → materializes section → materializes lesson."

Why it matters: later he can say "there's a bug in the materialization cascade" and the AI knows exactly what he means. The shared term collapses ambiguity.

> **Direct from video:** "Look how clean, like how much cleaner that language is because I've got a concept of materializing and the word agreed on between me and the LLM."

## Step 3 — write-PRD
Once satisfied (~22 questions in, ~25k tokens used), invoke `/write-prd`. The skill:

1. Asks for a long detailed problem description (already in context from grilling).
2. Re-explores the repo briefly.
3. Sketches the **major modules first**, before writing the PRD body — Pocock baked module-thinking into the skill.
4. Quizzes him on which modules need tests and produces a PRD with problem, solution, user stories (~18 here), implementation decisions, and testing decisions.

The PRD writing uncovers a sloppy proposal — adding `materializeCourseAndLesson` to `courseWriteService`. Pocock weighs whether to extend an existing method with a parameter; decides a new method is cleaner.

> **Operational note:** "Notice how I'm thinking about the interface more than I'm actually thinking about the implementation here. The implementation I don't really care about, but I want to make sure that this is testable."

He also catches the AI claiming "no existing test harness for course-write-service" when there clearly is one — does a "Rafiki, look harder" — and the AI finds the suite split by concern.

> **Principle on whether to read the PRD:** "Am I going to review this PRD? And no, I'm not going to. LLMs are really, really good at summarizing things… I'm just going to accept it on faith."

## Step 4 — PRD to Independently-Grabbable Issues
`/prd-to-issues` breaks the PRD into camp-board issues. The first cut produced six; Pocock judged the lesson-creation buttons too small to justify spinning up a whole agent and merged two and three. Final count: four issues.

Sizing rule he applies in real time: not too big, not too small. Too small = agent startup overhead dominates. Too big = falls out of the smart zone. Issues link to the parent PRD, list acceptance criteria, list what blocks them, and reference the user stories they address.

He does not review the issues either — they're "expanding out stuff that's in the PRD."

## Step 5 — The AFK Ralph Loop
Pocock built a small library called **sand castle** for this:

- A Dockerfile spins up a container.
- Mounts the working directory.
- Claude runs inside the container.
- Any commits Claude makes are extracted as patches and applied to the host repo.

`pnpm ralph` kicks it off with `maxIterations=100`. Ralph picks issues from the local backlog, closes them as it commits. Every commit runs tests and types.

> **Direct from video:** "I'm going to take a little break. I might even go for a little walk and I'm going to just wait and check back in with this once it's done."

Total elapsed: ~90 minutes for five iterations producing six commits. Output: detailed commit messages, tests added alongside features.

> **Day shift / night shift:** "I'm doing the day shift… thinking of ideas, grilling, turning into PRDs, into issues. And then the LLM takes the night shift. Claude goes and actually implements this stuff AFK."

## Step 6 — QA With a Feedback Button That Feeds Ralph
After Ralph reports done, Pocock asks a fresh Claude session: "Take the last five commits and create a QA plan." It writes a step-by-step QA guide, saved as a GitHub issue.

He walks the QA plan manually. Every bug or missing-feature he finds, he uses the app's built-in feedback button:

- Describe the issue in detail.
- The button creates a GitHub issue with a Haiku-generated title + the originating route + his description.
- Ralph picks it up next iteration.

He kicks off Ralph in the background and continues QA in parallel. The two streams converge.

Some bugs found in QA that grilling missed:
- "When I create a ghost course it doesn't redirect me; it shows a minified React error; no loading state."
- "If the course directory isn't a git repo, the database/filesystem state diverges and there's no rollback."

> **Direct from video:** "When you're in the QA loop, when you're iterating towards something, you are going to find little weird edge cases like this that is really hard to plan for before."

This is the central anti-specs-to-code argument: real-world QA finds things you cannot pre-specify.

## Step 7 — Re-design Decisions Found in QA
The right-click menu showed both "create ghost lesson" and "create real lesson" — confusing. Pocock decides he wants a single "add lesson" with a "also create on file system" checkbox. He couldn't decide between flows during grilling, but seeing it in reality made it obvious.

> **Principle:** "We could have had an extra design phase or an extra prototype phase, but… I don't mind just jumping to code and fixing it there."

Flexibility of the backlog approach: queue more issues anytime; Ralph keeps grinding.

## Operational Rules Surfaced
- **Always parallelise QA and bug-fixing.** While Ralph fixes a bug, you QA the next thing.
- **Mark human-in-the-loop tasks explicitly.** Ralph's prompt skips issues labeled human-in-the-loop or with AFK omitted. Pocock renames issues to gate them.
- **Don't review summarised artifacts.** PRDs and issue files are summarisations. Trust them.
- **Do review interfaces.** When PRDs propose new methods, scrutinise the API shape, not the implementation.
- **Tests on every commit.** Non-negotiable for Ralph loops.
- **The interface is the work.** Pocock spends his attention on module boundaries; implementations get delegated.

> **Direct from video:** "What I'm doing here is I'm reviewing inputs and outputs. I'm interested in the code. Absolutely. I'm interested in how the interfaces are changing. I'm interested in how the modules are sort of looking like."

## Key Exact Extracts
> **[14:42]** "Look how clean, like how much cleaner that language is because I've got a concept of materializing and the word agreed on between me and the LLM."

> **[26:35]** "I get it to update my ubiquitous language document to basically just keep it up to date with any of the ideas that I've got in here."

> **[27:10]** "It's even got a concept for the materialization cascade — the chain reaction when materializing a lesson inside a ghost course."

> **[27:08]** "I freaking love this because later on I can say, yeah, there's a bug inside the materialization cascade and it knows exactly what I'm talking about."

> **[26:48]** "Notice how I'm thinking about the interface more than I'm actually thinking about the implementation here. The implementation I don't really care about, but I want to make sure that this is testable."

> **[29:13]** "Am I going to review this PRD? And no, I'm not going to. LLMs are really really good at summarizing things."

> **[33:08]** "I'm doing the day shift. I'm thinking of ideas, I'm grilling with the LM. I'm turning this into PRDs and turning those PRDs into issues. And then the LM takes the night shift."

> **[39:50]** "When you're in the QA loop, when you're iterating towards something, you are going to find little weird edge cases like this that is really hard to plan for before."

> **[43:14]** "What I'm doing here is I'm reviewing inputs and outputs. I'm interested in how the interfaces are changing. I'm interested in how the modules are sort of looking like."
