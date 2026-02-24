---
name: prd
description: >
  Create a structured Product Requirements Document (PRD) through guided discovery.
  Use this skill whenever the user wants to create a PRD, write product requirements,
  plan a new feature with formal documentation, or says things like "let's write a PRD",
  "I need requirements for...", "create a product spec", "document this feature",
  or runs /specflow:prd. Also trigger when a user describes a problem and wants to
  turn it into a formal plan before implementation.
---

# PRD Creator

Guide the user through a structured discovery process to produce a comprehensive PRD, then save it to the `docs/specflow/prd/` directory for review.

The goal is to deeply understand the problem before writing anything. A good PRD eliminates ambiguity so that implementation can proceed without constant back-and-forth. The interview phase is the most important part -- rush it and the PRD will have gaps that cost far more to fix later.

## Process

### Phase 1: Problem Discovery

Ask the user for a long, detailed description of:
- The problem they want to solve
- Who is affected and how
- Any ideas they already have for solutions

Let them talk. Don't interrupt with structure yet -- raw context is valuable. After they've shared their initial thoughts, summarize what you heard back to them to confirm understanding.

### Phase 2: Codebase Exploration

Before diving deeper into discussion, explore the repository to ground the conversation in reality:

- Verify any assertions the user made about current behavior
- Understand the relevant parts of the architecture
- Identify existing patterns that the solution should follow
- Note any constraints or dependencies that might affect the design

Share what you found with the user -- they may not be aware of all the implications.

### Phase 3: Deep Interview

This is where the real work happens. Interview the user relentlessly about every aspect of the plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one.

Key areas to probe:

**Users and actors**
- Who are all the actors involved? (end users, admins, systems, third parties)
- What are their goals and constraints?
- How does this change their current workflow?

**Behavior and edge cases**
- What happens in the happy path?
- What are the error states? How should each be handled?
- What are the boundary conditions?
- Are there race conditions or concurrency concerns?
- What happens if this feature is partially complete?

**Scope boundaries**
- What is explicitly NOT included?
- What's the minimum viable version vs. the full vision?
- Are there phases or milestones?

**Integration points**
- What existing systems does this touch?
- Are there API contracts to define or modify?
- Are there schema changes needed?
- How does this affect existing features?

**Non-functional requirements**
- Performance expectations
- Security considerations
- Accessibility requirements
- Browser/device compatibility

Don't accept vague answers. If the user says "it should be fast," ask "what latency is acceptable?" If they say "users can manage their settings," ask "which settings, specifically?" Ambiguity in a PRD becomes bugs in implementation.

### Phase 4: Module Design

Sketch out the major modules that need to be built or modified. Actively look for opportunities to extract deep modules -- ones that encapsulate a lot of functionality behind a simple, testable interface that rarely changes. A deep module is preferable to a shallow one because it hides complexity and provides a stable contract.

For each module, identify:
- Its responsibility (single sentence)
- Its public interface (inputs and outputs)
- What it hides from the rest of the system
- Dependencies on other modules

Share this module map with the user and check:
- Do these modules match their mental model?
- Are any missing or unnecessary?
- Which modules should have tests written for them?

### Phase 5: Write the PRD

Once you have complete understanding, write the PRD using the template below. Every section should be substantive -- if a section feels thin, you probably need to go back and ask more questions.

After writing, review the PRD with the user. Make revisions until they're satisfied.

### Phase 6: Save PRD

Save the PRD to the `docs/specflow/prd/` directory:

1. Derive a kebab-case filename from the PRD title (e.g., "User Authentication Flow" -> `prd-user-authentication-flow.md`)
2. Create the `docs/specflow/prd/` directory if it doesn't exist
3. Fill in the YAML frontmatter with the actual title and today's date
4. Write the file to `docs/specflow/prd/prd-{feature-name}.md`
5. Confirm the saved file path with the user

### Phase 7: Self-Review (Devil's Advocate)

Before presenting the PRD to the user, re-read the saved file and verify everything against this checklist:

**Format & naming:**
- Filename follows `prd-{kebab-case-feature-name}.md` convention
- File is saved in `docs/specflow/prd/`
- YAML frontmatter is present with `title`, `status: Draft`, and `created: YYYY-MM-DD`

**Content completeness:**
- Every section in the template is present and substantive (not placeholder text)
- User stories are specific and actionable (no vague "as a user, I want it to work" stories)
- Implementation decisions capture rationale, not just choices
- Out of scope explicitly lists things that were discussed but deferred
- No GitHub references anywhere in the document

**Consistency:**
- Decisions made during the interview are accurately reflected in the PRD
- No contradictions between sections (e.g., something listed as out of scope but described in implementation)
- Technical terms and module names are used consistently throughout

**Devil's advocate -- challenge your own work:**
- Are there gaps where you made assumptions instead of asking the user?
- Is anything ambiguous enough that two developers could interpret it differently?
- Would a developer be able to implement this without coming back to ask clarifying questions?

If any issues are found, fix them before presenting the PRD to the user. If you made assumptions that should have been questions, flag them to the user for confirmation.

End with: "Run `/specflow:task` to break this PRD into actionable tasks."

## PRD Template

Use this exact structure for every PRD:

```markdown
---
title: "<PRD Title>"
status: Draft
created: YYYY-MM-DD
---

## Problem Statement

[The problem from the user's perspective. What pain exists today? Why does it matter? Include context about who is affected and how frequently.]

## Solution

[The proposed solution from the user's perspective. What will change for them? How will their workflow improve? Keep this focused on outcomes, not implementation details.]

## User Stories

[A focused numbered list covering the most important aspects of the feature. Each story follows the format:]

1. As a <actor>, I want <feature>, so that <benefit>

[Example:]
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending

[Keep this list tight and high-impact. Aim for 6-10 stories that cover the core happy paths, key error states, and the most important actor perspectives. Avoid redundant stories — combine related behaviors into a single story where possible. Quality over quantity.]

## Implementation Decisions

[Decisions made during the interview process. Include:]

- Modules to be built or modified, with their responsibilities
- Public interfaces of those modules
- Architectural decisions and their rationale
- Schema changes
- API contracts
- Key interactions between modules
- Technology choices and why

[Do NOT include specific file paths or code snippets -- they become outdated quickly.]

## Testing Decisions

[Include:]

- What makes a good test for this feature (test external behavior, not implementation details)
- Which modules will have tests and what those tests verify
- Prior art -- similar tests in the codebase that can serve as patterns
- Any special testing considerations (mocking, fixtures, integration tests)

## Out of Scope

[Explicitly list what this PRD does NOT cover. This prevents scope creep and sets clear expectations. Include things that were discussed but deliberately deferred.]

## Further Notes

[Anything else relevant: links to related issues, design mockups, research, open questions that don't block implementation, future considerations.]
```

## Guidance

- Spend most of your time in Phase 3. A thorough interview prevents rework.
- Push back on scope creep during the interview -- help the user find the smallest useful version.
- The user stories section is the heart of the PRD. If it's thin, the PRD isn't ready.
- Implementation decisions should capture the "why" not just the "what."
- When in doubt, ask another question rather than making an assumption.
