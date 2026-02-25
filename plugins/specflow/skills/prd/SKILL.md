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

**If the user provides solution-focused input** (describing what they want built rather than what problem it solves): accept their input as-is — do not ask them to reframe it as a problem. Instead, extract the implied problem yourself and present it in your summary for confirmation: "It sounds like the underlying problem is [your inference]. Is that right, or is there a different motivation?" This keeps Phase 1 efficient while ensuring the Problem Statement section of the PRD has a genuine problem framing, not a restated solution.

### Phase 2: Codebase Exploration

Before diving deeper into discussion, explore the repository to ground the conversation in reality:

- Verify any assertions the user made about current behavior
- Understand the relevant parts of the architecture
- Identify existing patterns that the solution should follow
- Note any constraints or dependencies that might affect the design

Share what you found with the user -- they may not be aware of all the implications.

#### Agent-Assisted Analysis

After the initial exploration above, check for specialist agents:

1. Read `docs/specflow/config.json` (if it exists)
2. If agents are configured (`agents.roles` exists), spawn the `roles.orchestrator` agent (e.g., `agent-teams:team-lead`) via the Task tool. Pass it:
   - The problem domain from Phase 1
   - Key files and patterns found during initial exploration
   - The full list of available agents from config.json (`agents.roles` and `agents.techStack`)
   - Task: "Analyze the codebase architecture relevant to this feature. Identify patterns, constraints, and dependencies. Then recommend which specialist agents (from the available list) should be spawned for deeper analysis, and provide a specific prompt for each recommended agent."
3. The orchestrator will return:
   - Its own architectural analysis findings (using its Read, Glob, Grep, Bash tools)
   - Recommendations for which specialist agents to spawn (if any), with specific prompts
4. Spawn the orchestrator's recommended specialist agents in parallel via the Task tool (max 2, since the orchestrator used 1 of the 3 agent slots)
5. Collect all findings (orchestrator + specialists) and integrate them into the exploration summary you share with the user
6. If `docs/specflow/config.json` doesn't exist or has no agents configured, note "Run `/specflow:setup` for enhanced specialist analysis" and continue with manual exploration only

#### Greenfield vs. Brownfield Classification

After exploration is complete, classify the project type:

1. Auto-detect using heuristics:
   - **Brownfield signals:** User language includes words like "refactor", "migrate", "replace", "upgrade", "rewrite", "modernize", "consolidate", or "deprecate"; codebase exploration found existing code in the problem domain (e.g., there's already an auth system and the user wants to change it)
   - **Greenfield signals:** No existing code in the problem domain; user language focuses on "build", "create", "add new", "implement from scratch"
2. Present the classification to the user: "This looks like a **brownfield** change (modifying existing functionality) / **greenfield** feature (building something new). Is that correct?"
3. The user always gets final say — they can override the classification
4. Store the classification for use in Phase 3 (interview) and Phase 5 (template)

### Phase 3: Deep Interview

This is where the real work happens. Interview the user relentlessly about every aspect of the plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one.

#### Constitution-Informed Constraints

Before starting the interview, load the project constitution from `docs/specflow/config.json` (if it exists and has a `constitution` key). Keep the principles active throughout the interview:

- Proactively surface relevant principles when the user's proposal conflicts with them. For example, if the constitution says "prefer composition over inheritance" and the user proposes a deep class hierarchy, raise it immediately: "Your constitution says 'prefer composition over inheritance' — should we explore a composition-based approach here, or is this an approved exception?"
- Don't lecture — flag the tension, let the user decide, and record the decision
- If no constitution exists, skip this entirely and proceed normally

Key areas to probe:

**Users and actors**
- Who are all the actors involved? (end users, admins, systems, third parties)
- What are their goals and constraints?
- How does this change their current workflow?

**Behavior and edge cases**
- What happens in the happy path?
- What are the error states? For every user action that involves a network call or external resource (form submit, API call, iframe/embed load, third-party script), ask: what does the user see when it fails? What does the user see while it loads? Is there retry behavior?
- What are the boundary conditions?
- Are there race conditions or concurrency concerns? (e.g., double-submit on forms, rapid toggle on expand/collapse)
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

**Brownfield-specific areas** (only probe if classified as brownfield in Phase 2):

- **Current state analysis** — What does the existing system do today? What works well and should be preserved? What are the known pain points? Are there undocumented behaviors or edge cases that users depend on?
- **Migration and transition** — Should migration be incremental or all-or-nothing? Will old and new systems need to co-exist during a transition period? Is there data to migrate? What's the reversibility requirement?
- **Backward compatibility** — Are there existing APIs, contracts, or interfaces that must continue working? Who are the external consumers? What's the deprecation strategy and timeline?
- **Rollback and risk** — What does rollback look like if things go wrong? Can feature flags gate the change? What monitoring detects problems? What's the blast radius of a failure?

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

For features with 3 or fewer modules where the user's own description maps one-to-one to the proposed modules, present the module map inline with a brief confirmation prompt ("Here are the modules I'll use — does this match your thinking?") rather than requiring a separate interaction round. For larger or more ambiguous module designs, wait for explicit user approval before proceeding to Phase 5.

#### Constitution Validation

If a project constitution exists in config.json, check each module's design against it after presenting the module map:

- Do any modules violate architectural boundaries? (e.g., a UI module that directly accesses the database when the constitution forbids it)
- Do quality standards impose constraints on any module? (e.g., "100% test coverage for business logic" means certain modules must be designed for testability)
- Surface any tensions as explicit trade-off decisions for the user to resolve: "Module X crosses the boundary 'no direct DB access from UI' — should we add a service layer, or is this an approved exception?"

### Phase 5: Write the PRD

Once you have complete understanding, write the PRD using the template below. Every section should be substantive -- if a section feels thin, you probably need to go back and ask more questions.

After writing, review the PRD with the user. Make revisions until they're satisfied.

### Phase 6: Save PRD

Save the PRD to the `docs/specflow/prd/` directory:

1. Derive a kebab-case slug from the PRD title (e.g., "User Authentication Flow" → `user-authentication-flow`)
2. Auto-detect the next PRD number:
   a. Glob for `docs/specflow/prd/*.md`
   b. Extract 3-digit numbers from filenames matching pattern `^(\d{3})-`
   c. Next number = max found + 1 (zero-padded to 3 digits)
   d. If no numbered PRDs exist, start at `001`
3. Create the `docs/specflow/prd/` directory if it doesn't exist
4. Fill in the YAML frontmatter with the title, `prd_id: PRD-{NNN}`, type, status, and today's date
5. Write the file to `docs/specflow/prd/{NNN}-{slug}.md` (e.g., `001-user-authentication-flow.md`)
6. Confirm the saved file path and PRD ID with the user

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
- For every user action that triggers a network call or loads an external resource: are both the error state and the loading state defined in the user stories or implementation decisions? If not, flag as a Warning with the specific interaction that is underspecified.
- If brownfield: are all Migration Strategy sub-sections (Current State, Transition Plan, Backward Compatibility, Rollback Plan) substantive? Is the rollback plan realistic and not hand-wavy?

**Consistency:**
- Any contradictions between sections (e.g., something listed as out of scope but described in implementation)?
- Are technical terms and module names used consistently throughout?
- Do module interfaces match the behavior described in user stories?

**Feasibility:**
- Are the proposed modules realistic given the existing codebase?
- Any hidden dependencies not captured?
- Are non-functional requirements achievable with the chosen approach?
- If brownfield: is the migration strategy realistic given codebase complexity? Does the rollback plan assume reversibility that doesn't actually exist (e.g., assuming data migration can be reversed when it can't)?

**Assumptions:**
- Where did the author assume instead of asking the user?
- Any ambiguities that could cause different interpretations by different developers?
- Could a developer implement this without needing to ask clarifying questions?

**Constitution compliance** (only if a constitution exists in config.json):
- Do any implementation decisions violate coding principles, architectural boundaries, or quality standards?
- Is the Constitution Alignment section present and accurate?
- Are all approved exceptions documented with rationale?
- Would a developer following this PRD inadvertently break a constitution principle?

Produce structured findings:
- **Critical Issues** — must fix before presenting (gaps, contradictions, missing requirements)
- **Warnings** — should fix or explicitly flag to the user
- **Observations** — worth noting but not blocking

Address all critical issues immediately by editing the saved PRD file.

#### Layer 2: Agent-Assisted Review

1. Read `docs/specflow/config.json` (if it exists)
2. If agents are configured (`agents.roles` exists), spawn the `roles.orchestrator` agent via the Task tool. This is a separate pass from Phase 2 — the Phase 2 orchestrator analyzed the codebase, this one reviews the PRD document itself. Pass it:
   - The full PRD content
   - The available agents from config.json
   - Task: "Review this PRD for architectural soundness. Are the proposed modules feasible? Are there risks or contradictions not addressed? Recommend which specialist agents (from the available list) should provide additional review perspectives, and provide a specific prompt for each."
3. The orchestrator will return its own review findings and may recommend specialist agents for additional perspectives
4. Spawn any recommended specialist agents and collect their findings
5. Integrate all agent findings into the review — these supplement the built-in framework but do not replace it
6. If `docs/specflow/config.json` doesn't exist or has no agents configured, skip this layer

#### Layer 3: Format & Naming Checklist

Final verification pass:
- Filename follows `{NNN}-{kebab-case-feature-name}.md` convention
- File is saved in `docs/specflow/prd/`
- YAML frontmatter includes `prd_id: PRD-{NNN}` matching the filename number

#### Present Results

If assumptions were found that should have been questions, go back to the user for clarification before finalizing.

Present the review summary to the user with any warnings or observations, then confirm the PRD is ready.

End with: "Run `/specflow:task` to break this PRD into actionable tasks."

## PRD Template

Use this exact structure for every PRD:

```markdown
---
title: "<PRD Title>"
prd_id: PRD-NNN
type: greenfield | brownfield
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

   **Verification Scenarios:**
   - Given <precondition>, When <action>, Then <expected outcome>
   - Given <precondition>, When <action>, Then <expected outcome>

[Each story must include 2-4 verification scenarios in Given/When/Then format. These are the scenarios a human QA tester will execute to verify the story is complete. Cover the happy path first, then key error/edge paths.]

[Example:]
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending

   **Verification Scenarios:**
   - Given I am logged in and have 2 accounts, When I navigate to the accounts page, Then I see the current balance for each account displayed in my local currency
   - Given I am logged in and the balance service is unavailable, When I navigate to the accounts page, Then I see a "Balance temporarily unavailable" message instead of stale data
   - Given I am logged in and have 0 accounts, When I navigate to the accounts page, Then I see a prompt to open my first account

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

### Constitution Alignment

[Only include this sub-section if a project constitution exists in config.json. Omit entirely if no constitution is defined.]

[Document which constitution principles each implementation decision supports. If any approved exceptions were granted during the interview, document them here with rationale.]

- **Supported principles:** [List principles that the design actively satisfies]
- **Approved exceptions:** [List any principles that this design intentionally deviates from, with the rationale discussed during the interview. If none, state "None — all principles upheld."]

## Migration Strategy

[Only include this section for brownfield PRDs. Omit entirely for greenfield.]

### Current State

[Detailed description of the existing system being modified. What it does, how it works, who uses it, and what its known limitations are.]

### Transition Plan

[How the migration will be executed:]
- Deployment strategy (incremental rollout, blue-green, canary, etc.)
- Co-existence period — how long old and new systems run simultaneously, and how traffic is routed
- Data migration — what data needs to move, in what order, and how integrity is verified
- Feature flags — which flags gate the change and how they're managed

### Backward Compatibility

[What must continue working during and after the transition:]
- Existing API contracts and their consumers
- External integrations that depend on current behavior
- Deprecation timeline for old interfaces

### Rollback Plan

[What happens if things go wrong:]
- Steps to reverse the change
- Data migration reversal (if applicable)
- Monitoring and alerting that trigger rollback
- Rollback window — how long after deployment can we still cleanly roll back

## Testing Decisions

[Include:]

- What makes a good test for this feature (test external behavior, not implementation details)
- Which modules will have tests and what those tests verify
- Prior art -- similar tests in the codebase that can serve as patterns
- Any special testing considerations (mocking, fixtures, integration tests)
- Human QA strategy: which verification scenarios require manual testing by a QA tester, and what environment setup or test data they need to execute those scenarios

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
