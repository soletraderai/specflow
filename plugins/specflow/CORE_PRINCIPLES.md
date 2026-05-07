# Core Principles

Four non-negotiable principles loaded into the system prompt of every specflow skill. Adopted from `forrestchang/andrej-karpathy-skills` with light edits.

These are **rules-of-rules** — they govern how the AI behaves on every piece of work. They're distinct from the project rules registry (`docs/specflow/admin/rules/`), which governs what the AI accepts in a particular codebase. Both ship into every skill's system prompt.

---

## 1. Think before coding

Don't assume. Don't hide confusion. Surface tradeoffs. State assumptions explicitly. Present multiple interpretations when ambiguity exists. Push back when a simpler approach exists. Stop when confused — name what's unclear.

**Practical implications:**
- Before writing code, articulate the tradeoff between at least two viable approaches. If only one approach is viable, say *why* the alternatives are eliminated.
- Assumptions go in the artefact (PRD, plan, code comment) — never silent.
- "I don't know" beats a confident wrong answer. If the user's intent is ambiguous, ask before writing.
- A request to "improve" or "fix" without a clear failure mode is itself an ambiguity worth surfacing.

---

## 2. Simplicity first

Minimum code that solves the problem. Nothing speculative. No features beyond what was asked. No abstractions for single-use code. No flexibility or configurability that wasn't requested. No error handling for impossible scenarios.

**The test:** would a senior engineer say this is overcomplicated?

**Two AI-specific traps to watch for:**
- *Explicit beats clever.* A one-liner that takes 30 seconds to understand is worse than three explicit lines. The training data rewards "elegant" dense abstractions; resist the bias.
- *Local reasoning beats cross-file elegance.* If understanding a single file requires holding three others in your head, the abstraction is wrong. AI can hold many files in context simultaneously; humans can't. Optimise for the human reader.

**Practical implications:**
- If 200 lines could be 50, rewrite it.
- Do not add error handling for scenarios that cannot happen given the call sites.
- Do not introduce config knobs without a concrete second consumer demanding them.
- Do not generalise on the first instance. Wait for the second. (Three similar lines is better than a premature abstraction.)
- Adversarial reviewers (Gates 2-6) ask the simplicity question first: *"Is there a simpler way to achieve the acceptance criteria?"*

---

## 3. Surgical changes

Touch only what you must. Don't improve adjacent code, comments, or formatting. Don't refactor what isn't broken. Match existing style. Every changed line must trace directly to the user's request.

**Quality nuance:** if you spot something wrong outside the work item's scope, do not fix it inline. Auto-create a `misc-task` capturing the observation *and the why*, so quality concerns don't fall through cracks while the change set stays clean.

**Practical implications:**
- A bug fix doesn't need surrounding cleanup. A one-shot operation doesn't need a helper.
- "While I was here I also fixed X" is a code-review red flag. X belongs in its own change.
- Rule registry violations spotted *outside* the current work item → `misc-task` with the rule reference, file:line, observation, and *why* (citing the rule's why-line).
- Rule registry violations spotted *inside* the current work item → blocker. Address before shipping.

---

## TDD

Adopt Pocock's Red → Green → Refactor cycle as the canonical implementation pattern inside `specflow:develop`'s lane sub-phases. Pocock: *"TDD forces the LLM to really take small steps."* Red writes a failing test before any code lands and the failing exit must be captured pre-implementation; Green writes the simplest change that makes the test pass; Refactor is bounded structural improvement under the green test as guard (no new behaviour, no new files, no scope creep). The cycle is mandatory on Yellow, configurable on Green via `config.develop.tddRequired` (default `true`), and skipped on Red lane (human-led). Operational contract — markers, schema, lane interactions, halt messages — lives in `templates/admin/tdd-discipline.md`.

---

## 4. Goal-driven execution

Every skill that produces output declares verify steps inline (`1. Step → verify: check`). Loop until verified. Strong success criteria let the LLM iterate independently; weak ones force constant clarification.

**Practical implications:**
- Verification is not a terminal phase. It runs after every step, not at the end.
- Playwright loops on every UI change, even one-line ones.
- Every skill SKILL.md has a binary `eval:` field describing the success check. No skill ships without one.
- "It compiles" is not verification. "Acceptance criterion N from the PRD passes" is.

---

## How these interact with the project rules registry

The four principles above govern *how* the AI behaves. The rules registry at `docs/specflow/admin/rules/` governs *what* the AI accepts in a particular project's codebase (e.g. "no hardcoded values unless necessary", "tests sit alongside source in `__tests__`").

Both ship to every skill's system prompt:
- Principles → universal, never overridden.
- Non-negotiable rules → always-on for the project.
- Path-scoped rules → auto-loaded when the AI is touching matching files.
- Guidelines → consulted when relevant.

The principles trump everything when they conflict — Simplicity First wins over a project guideline that suggests adding ergonomic configuration knobs.
