# Karpathy's Autoresearch Loop Applied to Cold Email and Marketing

## Identity
- Title: Claude Code + Karpathy Autoresearch = The New Meta
- Speaker/Channel: Nick Saraev (founder, Maker School / LeftClick AI; 287K YouTube subs)
- Likely URL: https://www.youtube.com/watch?v=4Cb_l2LJAW8
- Suggested slug: saraev-karpathy-autoresearch-cold-email
- Confidence: High. Strong textual signals match Saraev: "Maker School" course, cold email + Instantly API focus, agency/PPC framing, "anti-gravity" IDE, Whisper Flow dictation, casual "I do a lot of cold email" tone. Video ID 4Cb_l2LJAW8 confirmed via web search to match the title quoted in the upload metadata; Nick Saraev runs the dominant Maker School community and uses Instantly for cold email pipelines.

## Thesis
Andrej Karpathy's open-source `autoresearch` repo (an agent that autonomously edits a `train.py`, runs a 5-minute experiment, keeps or discards the change, and loops overnight) generalises beyond ML. Any workflow with (a) an objective metric, (b) an API or tool that can change inputs, and (c) a tight feedback loop can be wrapped in the same modify -> measure -> keep/discard loop using Claude Code as the orchestrator. Saraev demonstrates this on cold email copy with Instantly as the metrics API, scheduled hourly via GitHub Actions cron, with a `resources.md` that accumulates learnings across runs.

## Key points

### KP-1
- **Point**: Autoresearch is a domain-agnostic autonomous experimentation loop: hypothesis -> baseline vs challenger -> measure -> harvest winner -> log learning -> repeat.
- **Why it matters to our goals**: Maps cleanly onto specflow's "spec -> implement -> test -> iterate" pipeline. We can stop treating each spec run as one-shot and start letting the agent re-run with delta changes against an objective signal (test pass rate, lint count, performance). Directly serves goals 1 and 3 (productivity, fewer errors).
- **Evidence**: "give an AI agent a small but real LLM training setup and just let it experiment autonomously overnight... modify the code, train for 5 minutes, check if the results improved, keep or discard."
- **Sources**: lines 16-24, 152-180

### KP-2
- **Point**: Three minimum requirements for the loop: an objective metric, an API to mutate inputs, and a tight feedback cycle. Without all three the pattern breaks.
- **Why it matters to our goals**: This is a sober gating checklist for which specflow workflows we can automate. Test-pass-rate works (objective, fast, scriptable). "Better-feeling docs" doesn't (subjective). Saves us from gold-plating useless automations.
- **Evidence**: "things that work really well are things that have fast feedback loops... you need a clear metric... you need some sort of API access to change the inputs."
- **Sources**: lines 681-731

### KP-3
- **Point**: A persistent `resources.md` (or similar memory file) accumulates learnings across runs and is the actual source of compounding intelligence — not the orchestrator prompt itself.
- **Why it matters to our goals**: Mirrors specflow's own approach to spec memory. Reinforces that knowledge files should be structured as append-and-consolidate logs, with explicit consolidation around 500-1000 entries to avoid context bloat.
- **Evidence**: "they log all of their learnings to a resources.md that significantly improves future models abilities... eventually after something like 500 to maybe a 1,000 runs you'll probably have to consolidate."
- **Sources**: lines 121-128, 638-645

### KP-4
- **Point**: Schedule the loop with GitHub Actions cron (hourly / 4-hourly). Three steps per tick: harvest (collect prior results), generate (new challenger), deploy (activate).
- **Why it matters to our goals**: Cheap, version-controlled, no extra infra. Specflow ships through plugin.json — adding a workflow file to validate skills nightly against a metric (e.g. example output diff vs golden) is trivial and low-risk.
- **Evidence**: "I'm using a service called GitHub actions, which allows me to store this in the cloud and then run this on regular intervals... three steps. It's going to harvest... generate... deploy."
- **Sources**: lines 329-335, 534-545

### KP-5
- **Point**: The orchestrator agent's prompt quality is secondary; Karpathy himself flags his prompt as "probably pretty crappy." The repeatable scaffolding (test.md + metric + APIs + memory file) is what works.
- **Why it matters to our goals**: Cuts against the temptation to over-engineer specflow's skill prompts. Invest in the harness (state, metric capture, persistence) over hand-tuned wording — better leverage for a small team.
- **Evidence**: "his prompt is probably pretty crappy and that it'd be very easy to make a better one."
- **Sources**: lines 354-358

### KP-6
- **Point**: Most challengers lose to the human-written baseline at first; the value comes from the long tail. Don't kill the loop after a few losing rounds.
- **Why it matters to our goals**: Sets realistic expectations when introducing autonomous iteration to a dev team new to AI coding. Early "the AI made it worse" reactions are normal; success is statistical, not per-iteration.
- **Evidence**: "most challengers are not up to the task of the baseline. Usually the baseline is better because I wrote it, but eventually the challengers do become better."
- **Sources**: lines 608-617

### KP-7
- **Point**: A 5-minute loop yields ~12 experiments/hour; a 4-hour loop yields ~6/day. Iteration rate is the dominant variable in time-to-improvement.
- **Why it matters to our goals**: Argues for ruthlessly minimising the test/eval cycle in specflow (fast unit tests, mocked deps, narrow scope) over richer-but-slower validation. Goal 2 (better product, shorter time) is gated by loop tightness.
- **Evidence**: "if you have a five-minute loop... in 60 minutes you could run 12 experiments... your iteration loop will be much faster."
- **Sources**: lines 681-700

### KP-8
- **Point**: Always pipe loop events to a side channel (Slack webhook in his case) so a human can spot-check without being in the loop.
- **Why it matters to our goals**: Important hedge given the "no premature pipeline CTAs" memory and a docs-creator-plus-junior-dev team. Autonomous loops need observability so the docs creator can intervene at the right moment without owning every step.
- **Evidence**: "I would recommend you always have a way to visualize or at least keep track of things as they go... I've set up a little Slack ping via web hook."
- **Sources**: lines 554-563

### KP-9
- **Point**: When no API exists, fall back to Chrome DevTools MCP or CLI scripting to mutate inputs (e.g. updating an Amazon FBA product page).
- **Why it matters to our goals**: Useful pattern for cases where specflow needs to drive external state (deploy targets, dashboards) that lack a clean API. MCP is the bridge.
- **Evidence**: "if you don't have the API access, you could build some sort of Chrome dev tools or CLI based flow."
- **Sources**: lines 290-300, 720-730

### KP-10
- **Point**: Sub-agents handle bounded tasks (writing copy, calling Instantly, parsing JSON); the orchestrator only routes. Utility scripts (purge leads, deploy, parse) live as small one-off tools.
- **Why it matters to our goals**: Aligns with specflow's existing agent-team patterns. Reinforces that orchestrator + narrow tool-calling sub-agents beats one mega-prompt for reliability — matters for goal 3 (fewer errors).
- **Evidence**: "the orchestrator which orchestrates... the function of a bunch of lower level agents or tools... utility scripts are just little one-off API calls."
- **Sources**: lines 466-497

### KP-11
- **Point**: Each orchestrator run is technically a fresh agent, but it inherits all prior context via the persistent memory file — so the system is stateless per-call but stateful per-loop.
- **Why it matters to our goals**: This is the architectural pattern specflow already implies via skill memory; this transcript validates it as the right shape for compounding agent work.
- **Evidence**: "even though every time I run the orchestrator, it's technically like a different agent. It has all the context from all of the previous runs."
- **Sources**: lines 633-640

### KP-12
- **Point**: Listed transferable use cases beyond cold email: landing-page CRO, ad creatives (Meta/Google), chatbot CSAT scripts, ecom product descriptions, YouTube titles (via Data API v3), newsletter subject lines, pricing pages.
- **Why it matters to our goals**: Useful inventory if specflow ever ships marketing-flavoured skills, but also a template for *internal* metrics: skill quality scores, plugin adoption rate, time-to-green CI. Each is a candidate for an autoresearch wrapper.
- **Evidence**: "you could optimize subject lines for your newsletters... pricing pages... literally whatever you want."
- **Sources**: lines 238-310

## Tools / repos / frameworks mentioned
- karpathy/autoresearch — the source repo (https://github.com/karpathy/autoresearch). Single-GPU nanochat training agent with `program.md`, `train.py`, `resources.md`.
- Claude Code — orchestrator runtime; speaker uses Opus 4.6.
- Anti-Gravity IDE — speaker's editor of choice; says VS Code or "100 different apps" also fine.
- Whisper Flow — voice-to-text dictation for prompting agents (Fn-key push-to-talk).
- Instantly — cold email platform with API for campaigns and reply-rate metrics.
- GitHub Actions (cron) — scheduling the autonomous loop. Modal mentioned as alternative.
- Slack webhooks — observability side-channel.
- Chrome DevTools MCP — fallback when target system has no API.
- YouTube Data API v3 — example for optimising titles.
- Maker School — speaker's own course/community (used as source of cold-email best-practice docs fed into the agent).

## Verification log
- WebSearch query 1: "Karpathy auto research nano GPT cold email optimizer Maker School YouTube Claude Code" — returned the karpathy/autoresearch repo and several derivative posts plus the YouTube video "Claude Code + Karpathy Autoresearch = The New Meta" (4Cb_l2LJAW8). Repo confirms the modify -> verify -> keep/discard loop and `program.md` / single-file `train.py` shape.
- WebSearch query 2: "Maker School cold email Claude Code YouTube creator agency PPC" — returned Maker School / Nick Saraev results plus Claude Code marketing-skill resources.
- WebSearch query 3: "4Cb_l2LJAW8 YouTube channel creator name" — confirmed the video ID maps to the title quoted in the transcript context, dated March 2026.
- WebSearch query 4: "Maker School founder skool.com cold email YouTube creator name" — confirmed Nick Saraev as founder of Maker School and LeftClick AI, 287K YouTube subs, Bulgarian-Canadian, automation/cold-email focus.
- WebSearch query 5: "Nick Saraev LeftClick AI cold email instantly Maker School YouTube" — confirmed Saraev's stack matches every speaker tic in the transcript (Instantly API, agency cold email, Maker School community, YouTube channel @nicksaraev).
- Note: The "Calgary dental marketing firm" line in the example email is sample copy, not the speaker's location — Saraev is based in Canada but markets to North America broadly. Not a confidence-reducing signal.
