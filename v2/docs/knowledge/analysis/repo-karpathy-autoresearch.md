# karpathy/autoresearch

## What it is
A minimal experiment harness that lets a coding agent autonomously iterate on a single-GPU LLM training script (a stripped-down nanochat) overnight. The agent edits one file (`train.py`), runs a fixed 5-minute training job, compares `val_bpb`, and keeps or discards the change in git — repeating ~12 times an hour, ~100 times overnight. The "research org code" lives in a Markdown file (`program.md`) that the human iterates on; the Python is only ever touched by the agent.

## Author / origin
- Author: Andrej Karpathy (@karpathy)
- URL: https://github.com/karpathy/autoresearch
- Last activity (approx): pushed 2026-03-26; created 2026-03-06
- Stars (approx): ~77.9k

## Core ideas (3-7 bullets)
- **One editable file, one read-only file**: agents only modify `train.py`; `prepare.py` (data, eval, constants) is locked. Constraining the surface keeps diffs reviewable and guarantees a stable, fair benchmark across experiments.
- **Fixed time budget as the universal yardstick**: every run is exactly 5 minutes, so model size, batch size, optimizer, and architecture are directly comparable on a single scalar (`val_bpb`). No need for the agent to reason about training time — it's a constant.
- **Single objective metric, vocab-independent**: `val_bpb` (validation bits per byte) — lower is better, comparable across tokenizer changes. One number to optimize avoids agent paralysis.
- **Git-as-experiment-log**: each experiment is a commit on a dated branch (`autoresearch/<tag>`). "Keep" = advance branch; "discard" = `git reset`. The branch IS the running best; experiment history is the git log.
- **Append-only TSV ledger** (`results.tsv`, untracked): 5 columns — `commit`, `val_bpb`, `memory_gb`, `status` (keep/discard/crash), `description`. Tab-separated specifically because descriptions contain commas. Analysis notebook plots running-min over experiment index.
- **`program.md` as a "lightweight skill"**: the agent's instructions are themselves a versioned, human-edited Markdown file. The README explicitly calls this "the research org code." The human programs the org by editing prose, not Python.
- **Hard-coded autonomy norms**: explicit "NEVER STOP" clause, explicit "do NOT ask 'should I continue?'", crash-handling heuristic ("dumb fix → fix; fundamentally broken → log crash, move on"), simplicity criterion (a 0.001 win for 20 lines of hacky code is not worth it; an equal result with deleted code is a win).
- **Output discipline against context bloat**: `uv run train.py > run.log 2>&1` (explicitly NOT tee), then `grep "^val_bpb:" run.log` to extract the metric. The agent never reads the full training log unless the run crashed (`tail -n 50 run.log`).

## Specific patterns or files worth borrowing
- **`program.md` (skill prompt)** — Six-step setup checklist, then explicit CAN/CANNOT lists, then a numbered LOOP FOREVER. The structure is: (1) Setup contract, (2) Constraints, (3) Output format, (4) Logging schema with example, (5) The loop, (6) Autonomy norms. This is a near-perfect template for a self-running agent skill — every section is something specflow's pipeline skills already need or imply.
- **CAN/CANNOT framing**: instead of policy paragraphs, two bullet lists. Agents follow this much more reliably than narrative rules.
- **TSV ledger with a status enum**: `keep` / `discard` / `crash` is a beautiful three-valued outcome. Specflow's research/test/QA loops could log to a similar append-only ledger keyed by commit hash.
- **Branch-per-run convention**: `autoresearch/<tag>` where `<tag>` is the date. Forces parallel-safety, makes "the experiment" a first-class git object, and gives a clean throw-away if a session goes off the rails.
- **Baseline-first rule**: "Your very first run should always be to establish the baseline." Locks in a reference number before any change is made — a discipline specflow's evaluation/QA steps could adopt.
- **Simplicity criterion paragraph** (literal text in `program.md`): "A 0.001 val_bpb improvement that adds 20 lines of hacky code? Probably not worth it. A 0.001 val_bpb improvement from deleting code? Definitely keep." Worth quoting almost verbatim into specflow's review/simplify skill.
- **Crash-handling decision tree**: dumb-and-easy → fix and re-run; fundamentally broken → log and skip. Cheap heuristic that prevents agents from rat-holing on a single failure.
- **Notable-forks section in README**: invites platform forks (MacOS, MLX, Windows, AMD) and lists them. A community-extension pattern specflow could mirror for plugin variants.
- **`analysis.ipynb` as the human-facing morning report**: `pd.read_csv("results.tsv", sep="\t")`, plot running-minimum kept-bpb vs experiment index, list every KEEP with its description. Specflow could ship a default review-the-overnight-run notebook/skill that turns the TSV into a readable diff.

## Direct relevance to specflow's goals
- **Productivity (overnight throughput)**: the "100 experiments while you sleep" framing is exactly what specflow wants for documentation and dev iteration. The pattern — fixed budget per iteration + single metric + keep/discard via git — generalizes cleanly: e.g. "run the doc generator, score it against the spec, keep if score improved." Specflow could borrow the LOOP FOREVER structure verbatim for any iterative refinement skill.
- **Better product in shorter time**: the `program.md`-as-skill insight is the highest-leverage idea here. Karpathy explicitly says the human's job is to iterate on the *prose instructions*, not the code. Specflow's plugin already centers on Markdown skills — the autoresearch pattern proves out a discipline of "tune the skill, not the agent" that maps perfectly. A skill that itself gets refined against a measurable outcome is a self-improving asset.
- **Fewer errors**: three error-reduction mechanisms transfer directly — (1) **read-only files** (lock the eval harness so the agent can't game its own metric — applies to specflow's QA skill), (2) **git-as-checkpoint** (every experiment is a reset point; nothing is ever lost), (3) **explicit CAN/CANNOT lists** (constrains the action space, reduces drift). For a doc-creator + AI-new dev team, the third is especially valuable: bounded freedom prevents the agent from wandering.
- **Onboarding fit for AI-new devs**: the entire setup is "edit one Markdown file, watch the agent loop." That's the lowest possible cognitive overhead for a team learning AI-assisted workflows — and it's the same shape as specflow's existing skill-driven design.
- **Caution / non-fit**: autoresearch assumes a single objective scalar (`val_bpb`). Specflow's outputs (specs, code, docs) are multi-dimensional and partly subjective; the keep/discard decision needs a richer judge (LLM rubric, human gate, or composite score). The *pattern* transfers; the *metric* doesn't.

## Cross-references
- Parent project: https://github.com/karpathy/nanochat (full multi-platform training stack; autoresearch is a stripped-down single-GPU subset).
- Karpathy launch tweets: https://x.com/karpathy/status/2029701092347630069 and https://x.com/karpathy/status/2031135152349524125.
- "Dummy's Guide" referenced in README: https://x.com/hooeem/status/2030720614752039185.
- Notable forks (platform variants): miolini/autoresearch-macos, trevin-creator/autoresearch-mlx, jsegov/autoresearch-win-rtx, andyluo7/autoresearch.
- Specflow internal: feedback rule on premature pipeline CTAs aligns with autoresearch's "NEVER STOP / don't ask permission" autonomy norm — both push back on agents inserting human-checkpoint friction inside an autonomous loop.
