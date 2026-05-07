# Deslopping a Codebase With Deep Modules

**Author:** Matt Pocock
**Source:** YouTube tutorial / walkthrough
**Duration:** 11:16

## Executive Summary
AI has accelerated software entropy: every change that ignores the broader codebase introduces "weird things" until the codebase is a ball of mud. The cure is a shared vocabulary (modules, interfaces, depth, seams, adapters, locality, leverage) plus a repeatable refactoring skill. Pocock walks his "improve codebase architecture" skill across a real React Router codebase and argues the skill demands a strategic human above the tactical AI — agents are good sergeants, but you must be the general.

## Why Codebases Fall Apart Under AI

> **Direct from video:** "AI has simply accelerated software entropy. In other words, code bases are falling apart faster than they ever have before. Because every time that you make a change that doesn't take into account the entire codebase, you are likely to introduce little things, weird things that make the codebase harder to change."

The "ball of mud" is the failure state. Prevention is the previous video's topic; this one is the cure.

## Shared Vocabulary With the AI
A glossary lives in the skill itself. The point is precision: when you and the AI use the same terms, you can be specific about what you're asking for.

- **Module** — a unit of something in your application (a page's React components, a set of auth functions, a logger).
- **Interface** — everything a caller must know to use the module correctly. Includes methods *and* documentation/conventions for how to call it.
- **Implementation** — what's inside the module; what `signIn` actually does.
- **Deep module** — hides lots of implementation behind a relatively simple interface (Tanstack Query is the canonical example).
- **Shallow module** — complex interface, not much implementation behind it.
- **Depth** — the amount of behavior a caller can exercise per unit of interface they have to learn.
- **Seam** — the location where a module's interface lives in the application; the gap between modules. Seams are where unit/integration tests sit.
- **Adapter** (from hexagonal architecture) — a concrete thing that satisfies the interface at a seam (real clock vs. fake clock in tests).
- **Locality** — keeping things that change together in one place. High locality good; low locality bad.
- **Leverage** — capability per unit of interface the caller has to learn. Deeper module = more leverage.

> **Definition (depth):** "The amount of behavior a caller can exercise per unit of interface that they have to learn."

These ideas are from John Ousterhout's *A Philosophy of Software Design* — Pocock recommends buying it.

## The Two Properties You're Optimising For
When you "improve a codebase," you're chasing two things, and only two:

1. **Locality** for maintainers — changes and bugfixes for a behavior concentrate in one module instead of being scattered.
2. **Leverage** for callers — more capability per unit of interface they have to learn.

> **Principle:** "When we're talking about improving our code bases, these are the two attributes that we're aiming at."

## The Improve-Codebase-Architecture Skill in Practice
Pocock runs the skill against his ~1,500-commit course video manager (React Router + effect.ts).

1. Open a fresh Claude session in the repo.
2. Turn off auto mode — auto mode "does some funny things with these human-in-the-loop style flows."
3. Run the skill. It explores the codebase first, looking for shallow modules, poor leverage, or poor locality.
4. The skill returns several "deepening opportunities" as candidates.
5. Pick one candidate; the skill enters a **grilling session** to ground the proposal in concrete code on both sides of the seam.
6. Iteratively answer (or let it pick recommended answers). It proposes a module shape and a TypeScript interface sketch.
7. Convert the resulting design into a GitHub issue for an AFK agent to implement, or apply directly.

In the demo, the skill found a concept with two parallel implementations and **no single seam** — front-end and back-end could drift out of sync. The fix: collapse them into one module, gaining locality.

## Why This Is Not an AFK Skill
This is the central operational claim of the video.

> **Direct from video:** "This requires a judgment call from you, the programmer, sitting above the LLM. I think of agents as really, really good tactical programmers. They're able to get on the ground and make changes quickly, but they need someone on the level above them who is the strategic programmer."

Run the skill every couple of days on fast-moving codebases. The deeper your modules, the better the leverage *and* the better the test surface.

## Legacy Codebases
A "legacy" codebase usually means a *bad* codebase — hard to change. Before making AI-driven changes, you need a **harness**: tests around deep modules with real leverage and locality. So running improve-codebase-architecture is itself the right starting point on a legacy codebase.

## Key Exact Extracts
> **[00:09]** "AI has simply accelerated software entropy."

> **[00:25]** "It just snowballs and snowballs until you end up with a huge ball of mud. Sloppy, sloppy mud that is incredibly hard to reverse."

> **[02:48]** "A deep module hides lots of implementation behind a relatively simple interface. A shallow module has a complex interface and kind of not much implementation actually behind it."

> **[03:14]** "The amount of behavior a caller can exercise per unit of interface that they have to learn."

> **[03:43]** "These gaps between these modules are called the seams. It's the location at which the module's interface lives inside the application."

> **[04:13]** "Figuring out where your seams are going to live in your application is crucial to getting a good architecture."

> **[05:04]** "You want high locality, grouping and colloccating the things that matter and that often change together."

> **[09:30]** "I think of agents as really, really good tactical programmers… they need someone on the level above them who is the strategic programmer. And that's you."

> **[10:35]** "When we talk about legacy code bases, what we really mean are bad code bases. Code bases that are hard to make changes in. And what you really need before you start making changes in a legacy codebase is a harness around the codebase."

> **[11:00]** "The deeper you get those modules, the higher leverage you're going to get out of them. And leverage as well means testing."
