# Templates

Files `specflow:setup` and `specflow:upgrade` actively copy into the user's project. These are the **operational** templates — when a setup runs, the contents of this folder become the contents of `docs/specflow/admin/` (with substitutions where appropriate).

Distinct from `examples/`, which is read-only educational reference material. If a file lives in `templates/`, the plugin treats it as a source it can copy from. If it lives in `examples/`, the plugin never touches it.

## Layout

```
templates/
├── README.md                      # this file
├── agents/
│   └── standard/                  # copied to admin/agents/standard/ at setup
│       ├── lifecycle/             # plan / challenge / confirm — kept deliberately small
│       │   ├── orchestrator.md
│       │   ├── devils-advocate.md
│       │   └── verifier.md
│       └── principles/            # one reviewer per core principle — scales 1:1 with CORE_PRINCIPLES.md
│           ├── simplicity-reviewer.md
│           ├── surgical-reviewer.md
│           ├── think-before-coding-reviewer.md
│           └── goal-driven-reviewer.md
├── admin/
│   ├── CONTEXT.md                 # copied to admin/CONTEXT.md (then filled in by feedback-loop-audit)
│   └── rules/
│       ├── non-negotiable.md      # seeds admin/rules/non-negotiable.md (user accepts/edits at setup)
│       ├── guidelines.md          # copied empty — guidelines accumulate from real work
│       └── glossary.md            # copied with starter set's glossary lines
└── profile-examples.json          # read by setup's profile interview; user picks/customises before admin/profiles.json is written
```

## How setup uses each template

- **Standard agents** — `setup` copies the lifecycle three to `admin/agents/standard/lifecycle/` and the principle reviewers to `admin/agents/standard/principles/`. Never overwrites if the user has edited any of them; instead surfaces a diff for the user to merge. The principle-reviewer set must stay in sync with `CORE_PRINCIPLES.md` — adding a fifth principle requires adding a fifth reviewer template here.
- **CONTEXT.md** — copied verbatim once. After the initial copy, ownership transfers to the project; `feedback-loop-audit` populates the structure on first run.
- **Rules registry** — `non-negotiable.md` shipped with the starter set. User reviews each rule at setup and accepts or edits. `guidelines.md` ships empty. `glossary.md` ships with starter-set glossary lines and grows as rules are added.
- **Profile examples** — read into memory at setup. Setup proposes a relevant subset based on detected stack; user picks/customises. The result is written to `admin/profiles.json` (this file is never copied verbatim).

## Editing templates

Editing a file here changes what every future `specflow:setup` run produces. Treat changes here like API changes: bump the plugin version, document in `CHANGELOG.md`, add a migration entry in `MIGRATIONS.md` if existing projects need updating.
