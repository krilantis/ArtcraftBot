# BRIEF — ArtCraft Bot Platform: the base (logger, gateway, adapter)
For: Codex (builder). From: Krilantis via Muse. Date: 2026-11-26.
Hard deadline: **2026-11-30** (your access ends with the ChatGPT plan). Build in the order given; a module is finished — tests green, note written — before the next one starts. The cut line is the module list itself: logger complete beats three modules at eighty percent.

## What this is
The foundation of a standalone, open-source (MIT) bot platform of our own — ground-up, zero Forge dependency. This brief is only the base: the three pieces everything else plugs into. No UI, no supervisor, no bot homes, no evolution track in this brief.

## Where it lives
- This repo: **github.com/krilantis/ArtcraftBot** (private; the founder created it 2026-11-26). MIT LICENSE is already in place from the first commit. Nothing is made public and nothing is pushed to any other remote — public release is the founder's tap, not yours.
- Stack: Node + TypeScript, minimal dependencies, SQLite for the logger's index. No heavy frameworks.

## Module 1 — The untainted logger (build first)
An append-only record of every model call any bot makes. This data is training/evolution capital and can never be back-filled, so it ships first and it does not mutate records after writing them.
- JSONL append log + a SQLite index over it.
- One record per call: timestamp, bot id, ticket id (nullable), run id, provider, model, full request (messages/prompt), full response, tool calls, latency ms, tokens in/out, cost USD (from the gateway's price table), outcome (ok / error / rate-limited).
- A read path: query by bot, by ticket, by day — CLI or small library API.
- Tests: write/read round-trip; concurrent appends do not corrupt the log; a record written is byte-identical when read back.

## Module 2 — The model gateway
An OpenAI-compatible HTTP endpoint that harnesses point at, so every call passes through one door with caps and logging.
- Provider routes (these lanes are ruled; do not add others): **Synthetic** (`https://api.synthetic.new/openai/v1`), **OpenRouter — free models only** (enforce the `:free` suffix; reject anything else), **NVIDIA**. Provider support in code for other vendors is acceptable; enabling a new paid lane is not part of this brief.
- Per-bot gateway keys: the gateway issues a key per bot id; caps are config (daily USD cap, daily request cap per bot). A bot over cap gets a clean refusal, not a silent pass.
- Rate limits: on a provider 429, respect Retry-After with exponential backoff; surface exhaustion as an error the caller can read.
- Every call — success, refusal, or failure — goes through the Module 1 logger.
- Provider credentials come from environment/secret files only. `.env.example` documents them; no real key is ever committed.

## Module 3 — BotAdapter (interface + OpenCode)
- A harness-agnostic adapter interface: open a session in a bot home directory, send a ticket/prompt, stream events back, interrupt a run, close the session, list artifacts produced.
- One real implementation: **OpenCode over `opencode serve`** (its HTTP API). Do NOT build on `opencode run` — in build 1.19.6 its run mode breaks with custom agents, and it blocks on an inherited stdin; if run mode is ever used, stdin must be closed. The serve path is the platform path.
- A fake adapter for tests.
- One integration test, env-gated: adapter → gateway → Synthetic, bare prompt "Reply with exactly: READY", expects READY back.

## Fences
- No credentials in the repo, ever — env only.
- No publishing, no pushing to any remote, no package-registry releases.
- Test calls stay tiny (the READY class of call). No bulk runs.
- If a design choice is genuinely open, pick the simpler one and note it in the builder's note; do not stall.

## Deliverable shape (this is the acceptance test)
- The repo, with: source, green test suite Muse can run with one command, `.env.example`, and `docs/` holding one builder's note per module — what it does, how to run it, its known limits, and every open choice you made and why.
- Muse verifies by running the suite and the integration test on his own bench. Code that only runs in your environment is not done.

## Standing rules that bind this build
- The logger's records are untainted data: exact, dated, never edited after the fact — a wrong record gets a correcting record appended, never a rewrite.
- Mechanical over verbal: if a behavior can be checked by a test, the test is the proof.
- Filed ≠ adopted applies to your own suggestions too: note ideas in the builder's notes; do not build them into the base.
