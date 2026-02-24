---
name: "specflow:prd"
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

#### Agent-Assisted Analysis (if available)

After the initial exploration above, check for specialist agents:

1. Read `docs/specflow/config.json` (if it exists)
2. If agents are configured (`agents.roles` exists), spawn relevant agents via the Task tool in parallel (max 3):
   - Use the `roles.architectureReview` agent to analyze codebase architecture relevant to the PRD — pass it the problem domain from Phase 1 and the key modules/files found during exploration
   - Use the appropriate tech-stack agent (from `agents.techStack`) to analyze framework-specific patterns relevant to the feature
3. Collect subagent findings and integrate them into the exploration summary you share with the user
4. If `docs/specflow/config.json` doesn't exist or has no agents configured, note "Run `/specflow:setup` for enhanced specialist analysis" and continue normally — this is purely additive

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

Re-read the saved PRD file and apply a structured review before presenting to the user. This phase has three layers: the built-in review framework (primary), optional agent review (supplementary), and a final format checklist.

#### Layer 1: Built-in PRD Review Framework

This is specflow's own devil's advocate logic. Apply each lens systematically:

**Completeness:**
- Is every template section substantive (not placeholder text)?
- Are user stories specific, actionable, and testable?
- Do implementation decisions include rationale, not just choices?
- Is out of scope explicit about what was discussed but deferred?
- No GitHub references anywhere in the document

**Consistency:**
- Any contradictions between sections (e.g., something listed as out of scope but described in implementation)?
- Are technical terms and module names used consistently throughout?
- Do module interfaces match the behavior described in user stories?

**Feasibility:**
- Are the proposed modules realistic given the existing codebase?
- Any hidden dependencies not captured?
- Are non-functional requirements achievable with the chosen approach?

**Assumptions:**
- Where did the author assume instead of asking the user?
- Any ambiguities that could cause different interpretations by different developers?
- Could a developer implement this without needing to ask clarifying questions?

Produce structured findings:
- **Critical Issues** — must fix before presenting (gaps, contradictions, missing requirements)
- **Warnings** — should fix or explicitly flag to the user
- **Observations** — worth noting but not blocking

Address all critical issues immediately by editing the saved PRD file.

#### Layer 2: Agent-Assisted Review (if available)

1. Read `docs/specflow/config.json` (if it exists)
2. If agents are configured, spawn review agents via the Task tool for supplementary perspectives:
   - Use `roles.architectureReview` agent to review architectural decisions in the PRD
   - Pass the full PRD content and ask for architecture-focused feedback
3. Integrate any agent findings into the review — these supplement the built-in framework but do not replace it
4. If no agents are available, skip this layer entirely

#### Layer 3: Format & Naming Checklist

Final verification pass:
- Filename follows `prd-{kebab-case-feature-name}.md` convention
- File is saved in `docs/specflow/prd/`
- YAML frontmatter is present with `title`, `status: Draft`, and `created: YYYY-MM-DD`

#### Present Results

If assumptions were found that should have been questions, go back to the user for clarification before finalizing.

Present the review summary to the user with any warnings or observations, then confirm the PRD is ready.

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
