# Software Fundamentals Matter More Than Ever

**Author:** Matt Pocock
**Source:** YouTube conference talk
**Duration:** 18:23

## Executive Summary
Pocock's thesis: in the AI age, software fundamentals matter *more*, not less. Specs-to-code (treat code as cheap, regenerate from a spec on every change) produces worse code each iteration — a textbook case of software entropy. Good codebases get more out of AI; bad codebases get garbage. He maps five common AI failure modes to old-book disciplines (Ousterhout, Pragmatic Programmer, Brooks's *The Design of Design*, DDD, Kent Beck, TDD) and presents the skills he's built to enforce each: grill-me, ubiquitous-language, TDD/red-green-refactor, deep modules, and continual interface design.

## The Specs-to-Code Failure
Pocock tried specs-to-code. Each compile produced worse code, then worse, then worse. His diagnosis: when you ignore the code and only edit the spec, you're vibe coding by another name, and you stop investing in the design of the system.

> **Direct from video:** "Code is not cheap. In fact, bad code is the most expensive it's ever been. Because if you have a codebase that's hard to change, you're not able to take all of the bounty that AI can offer because AI in a good codebase actually does really, really well."

Definition he leans on, from Ousterhout's *A Philosophy of Software Design*:

> **Definition (complexity):** "Complexity is anything related to the structure of a software system that makes it hard to understand and modify the system."

A bad codebase is one that's hard to change. Good codebases are easy to change.

And from *The Pragmatic Programmer* — software entropy. Every change made without thinking about whole-system design pushes the codebase further toward collapse. That's exactly what specs-to-code produces.

## Failure Mode 1 — The AI Built the Wrong Thing
Cause: no shared design concept (Frederick P. Brooks, *The Design of Design*). The "design concept" is the invisible, ephemeral idea of the thing being built — it is *not* an asset, not a markdown file.

Pocock's fix is the **grill-me skill**:

> **Exact quote:** "Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one."

Two short lines turn the AI into an adversary that asks 40, 60, sometimes 100 questions. The conversation itself becomes the destination — turn it into a PRD afterward, or pass it straight to issues. He prefers this to Claude's default plan mode, which is "extremely eager to create an asset" before alignment is reached.

## Failure Mode 2 — The AI Is Too Verbose
Cause: no shared language. Same gap a developer has when working with a domain expert.

Fix from DDD: a **ubiquitous language**.

> **Direct from video:** "With ubiquitous language, conversations among developers and expressions of the code and conversations with domain experts are all derived from the same domain model."

His ubiquitous-language skill scans the codebase, extracts terminology, and produces a markdown file (tables of terms) that's open during every grilling and planning session. It improves the AI's *thinking traces*, not just its outputs — implementations align more closely with what was planned.

## Failure Mode 3 — The AI Built the Right Thing But It Doesn't Work
Obvious fixes: TypeScript, browser access for front-end agents, automated tests. The deeper problem: LLMs don't use feedback loops well — they "outrun their headlights" (Pragmatic Programmer):

> **Principle:** "The rate of feedback is your speed limit, which means that you should be testing as you go, taking small deliberate steps. And the AI by default is really not very good at that."

Solution: **TDD** — red, green, refactor. Force the LLM to take small steps:

> **Direct from video:** "TDD forces the LLM to really take small steps. You create a test first. You make that test pass and then you refactor the code to make it nicer and consider the design."

## Failure Mode 4 — Your Codebase Is Hard to Test (Shallow Modules Everywhere)
TDD only works on a testable codebase. Back to Ousterhout: deep modules over shallow modules.

- **Deep module:** lots of functionality hidden behind a simple interface. You *can* look inside, but you don't have to.
- **Shallow module:** not much functionality, complex interface.

A codebase of shallow modules is hard for AI to navigate — it can't find the right module, doesn't grasp dependencies. A codebase of deep modules with simple, well-designed interfaces is testable at the boundary and explorable. His **improve-codebase-architecture skill** is the recurring tool for deepening modules.

## Failure Mode 5 — Your Brain Can't Keep Up
Even with feedback loops working, shipping more than ever is exhausting because *you* have to hold the model in your head too.

Deep modules let you treat each module as a **gray box**: design the interface, delegate the implementation. You don't need to review what's inside finance/auth-grade modules excepted — as long as the boundary is testable and the purpose is understood.

> **Tip 5:** "Design the interface, delegate the implementation."

Pocock baked this into his write-PRD skill — PRDs specify *which modules* are changing and *how their interfaces* are modified. Implementations are an afterthought; interfaces are the work.

## Invest in Design Every Day
The closing principle, from Kent Beck:

> **Direct from video:** "Invest in the design of the system every day. And this is the core of it right because specs the code we are not investing in the design of the system we are divesting from it."

The mental model Pocock leaves you with: AI is a brilliant tactical programmer (sergeant on the ground). You are the strategic programmer above it. That requires fundamentals that have been around for 20+ years.

## Key Exact Extracts
> **[02:46]** "A bad codebase is a codebase that's hard to change. If you can't change a codebase without causing bugs, then it's a bad codebase. Good code bases are easy to change."

> **[03:55]** "Code is cheap… Well, I don't think this is right. I think code is not cheap. In fact, bad code is the most expensive it's ever been."

> **[04:20]** "Good code bases matter more than ever, which means software fundamentals matter more than ever. That's the thesis of this talk."

> **[05:53]** "Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one."

> **[10:51]** "The rate of feedback is your speed limit."

> **[11:12]** "TDD forces the LLM to really take small steps. You create a test first. You make that test pass and then you refactor the code."

> **[13:00]** "Deep modules, lots of functionality hidden behind a simple interface, hiding the complexity. You can look inside the deep module if you want to, but you don't need to."

> **[16:27]** "Design the interface, delegate the implementation."

> **[17:00]** "Invest in the design of the system every day… specs the code we are not investing in the design of the system we are divesting from it."

> **[17:43]** "If we think about AI as a really great on the ground programmer, a kind of tactical programmer, a sergeant on the ground… you need someone above that. You need someone thinking on the strategic level and that's you."
