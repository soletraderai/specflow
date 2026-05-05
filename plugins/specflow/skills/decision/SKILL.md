---
name: specflow:decision
description: Lightweight skill for users to manually log a decision out-of-band. Appends to admin/decision-log.md.
status: v2-new
phase: 3
requires: []
produces: [docs/specflow/admin/decision-log.md]
eval: new entry in decision-log.md has title, context, decision, rationale, date, and related files/tasks references.
---

# specflow:decision

Lightweight skill for users to manually log a decision without leaving the flow. Phase 3.

**Triggers:** "/specflow:decision".

**Captures:**
- Title — short, distinctive ("use Postgres triggers for audit log").
- Context — what was the situation that prompted the decision.
- Decision — what was chosen.
- Rationale — *why*. The most important field — survives refactors, justifies the choice years later.
- Date — auto-populated.
- Related files / tasks — for traceability.

**Output:**
- New entry in `admin/decision-log.md`.

**Why a separate skill (not just edit the file):**
- Schema discipline. Manual edits drift; a skill keeps the format consistent.
- Quick capture. The user is mid-flow when they make the decision; the skill is one command.
- Phase 3 self-learning reads this file for patterns — schema consistency makes that work.

**Verify steps:**
1. New entry exists with all required fields.
2. Date format consistent with prior entries.
3. Related files/tasks references are real (paths exist, task IDs resolve).

**Reference:** PRD Phase 3 scope item 3; Appendix I5.
