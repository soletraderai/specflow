# Examples

Read-only worked example for users to browse. Sanitised — no client identifiers.

The example mirrors the full `docs/specflow/` layout a real project ends up with. Setup never copies *from* this folder — see `templates/` for files setup actively writes. This folder is purely educational: "here's what your project will look like after you've used specflow for a while."

## Layout

```
examples/
└── docs/specflow/
    ├── admin/                                       # (Phase 1 ship target — populated incrementally)
    │   ├── config.json
    │   ├── pages.json
    │   ├── profiles.json                            # 3-5 example personas
    │   ├── environment.json                         # detected stack snapshot
    │   ├── CONTEXT.md                               # filled-in context doc
    │   ├── decision-log.md                          # ~5 entries showing the schema
    │   ├── task-history.json                        # ~5 completed tasks showing the schema
    │   ├── rules/                                   # non-negotiable, guidelines, glossary
    │   └── agents/                                  # standard (lifecycle + principles), specialised, index.json
    └── features/
        └── 001-design-skill/                        # ✅ POPULATED — worked example for specflow:design
            ├── 001-design-skill-interview.md        # Phase A goal + 6 grilling rounds + sign-off + Gate-2 amendments
            ├── 001-design-skill-prd.md              # Phase C synthesis — 12 requirements, 11 ACs (post-Gate-2)
            ├── design/                              # mockup placeholders
            ├── docs/                                # research input placeholders
            ├── assets/                              # capture placeholders
            └── debate-log/prd-gate2/                # ✅ FULL Gate 2 manifest
                ├── manifest.md                      # Orchestrator's closing entry, calibration notes
                └── findings/
                    ├── round-1/                     # 5 reviewer JSONs (7 findings total)
                    ├── round-2/responses.json       # AI's accept/push-back for each finding
                    └── round-3/                     # 5 reviewer JSONs (sharpen or accept)
```

## Status

The **001-design-skill worked example is fully populated** with a complete Gate 2 debate manifest. It's the calibration anchor for what a healthy Gate 2 looks like — neither rubber-stamping nor bikeshedding.

The `admin/` example tree is still incrementally populated as each surface is real. If a subdirectory is empty, that's the current state.

## What the worked example demonstrates

- **PRD interview** with confirmed Goal section + 6 grilling rounds + Topics-not-discussed + sign-off (extended through Gate 2).
- **PRD synthesis** with Vision tracing to Goal, every requirement tracing to a Resolved line + a goal field, every AC binary.
- **Gate 2 multi-agent debate manifest** — 5 reviewers fired in parallel into a shared manifest:
  - Round 1: 7 findings across Simplicity, Surgical, Think-Before-Coding, Goal-Driven, Devil's Advocate.
  - Round 2: AI accepted 5, pushed back on 2 with cited evidence.
  - Round 3: every reviewer accepted (no sharpening).
  - Closing decision: passed. PRD revisions applied (R7/R11/R12 sharpened, AC-11 added, interview's Topics-not-discussed extended).

## How to use the example

Read top-down for the lifecycle: identity → rules → spec → review → execution → memory.

1. Start at `features/001-design-skill/001-design-skill-interview.md` — what a goal-confirmed, fully-grilled interview looks like.
2. Then `features/001-design-skill/001-design-skill-prd.md` — what synthesised requirements + ACs look like, including the post-Gate-2 sharpening.
3. Then `features/001-design-skill/debate-log/prd-gate2/manifest.md` — what a Gate 2 closing manifest looks like end-to-end.
4. Then `findings/round-1/*.json` — what each reviewer's lens-distinct finding looks like.
5. Then `findings/round-2/responses.json` — what the AI accepting and pushing back looks like.
6. Then `findings/round-3/*.json` — what reviewer convergence looks like.
7. (When populated) `admin/CONTEXT.md`, `admin/profiles.json`, `admin/rules/glossary.md` — project memory and rules.
8. (When populated) `admin/decision-log.md`, `admin/task-history.json` — what a project remembers about its own past work.

## Don't edit

If you find yourself editing files in `examples/`, stop and ask: should this be a template (`templates/`) instead? Examples are reference material; templates are the files setup copies into your project. Editing an example doesn't change anyone's project.
