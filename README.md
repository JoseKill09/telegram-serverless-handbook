# Telegram Serverless Handbook

**An unofficial handbook and AI-agent kit for [Telegram Serverless](https://core.telegram.org/bots/serverless) (`tgcloud`).**
What the official docs don't tell you, tested patterns, and prompts that let an AI build and ship a Serverless bot **on its own**.

> Goal: anyone, with no programming knowledge, can ask an AI for a Telegram bot and get it running.
> The person only has to (1) create the bot in @BotFather, (2) turn on **Serverless**, and (3) give the AI the information it asks for.

## Pick your path

| You have… | Use | What happens |
|---|---|---|
| An AI **agent that can run commands** (Claude Code — also in the browser at claude.ai/code —, Cursor, Codex, opencode…) | [`prompts/autonomous-agent.md`](prompts/autonomous-agent.md) | The AI writes the code, tests it on Telegram's servers, deploys it and checks the result. Fully autonomous. |
| Only an **AI chat** (Claude, ChatGPT, Gemini…) and Telegram | [Builder Hub](https://github.com/JoseKill09/builder-hub-bot) | The AI gives you a ZIP; you send it to the Builder Hub bot, which validates and publishes it. |

Either way, **regenerate the Serverless access token in @BotFather when you are done**: you gave it to someone else.

## What's inside

| File | About |
|---|---|
| [`AGENTS.md`](AGENTS.md) | Rules for AI agents working on a Serverless project (auto-loaded by most agents) |
| [`docs/platform.md`](docs/platform.md) | Mental model, project layout, what gets deployed, valid handler names |
| [`docs/cli.md`](docs/cli.md) | The `tgcloud` CLI in practice: non-interactive use, project detection, `run`, `migrate`, output formats |
| [`docs/management-api.md`](docs/management-api.md) | The (undocumented) HTTP API behind the CLI |
| [`docs/database.md`](docs/database.md) | The Drizzle-like database: gotchas and migrations |
| [`docs/runtime.md`](docs/runtime.md) | What exists inside the V8 isolate (SDK, `fetch`, files, `Intl`…) |
| [`docs/bot-api.md`](docs/bot-api.md) | Calling the Bot API from Serverless, Rich Messages, entities |
| [`docs/mini-apps.md`](docs/mini-apps.md) | Hosting a Mini App and calling endpoints |
| [`docs/patterns.md`](docs/patterns.md) | Multi-user bots, testing without breaking real data, life without cron… |
| [`docs/limits.md`](docs/limits.md) | Known limits and missing features |
| [`DISCOVERIES.md`](DISCOVERIES.md) | Log of findings, each with date, CLI version and how it was verified |

## Official sources (always check them first)

- Serverless: <https://core.telegram.org/bots/serverless.md> (raw Markdown — easier for AIs than the HTML page)
- Bot API: <https://core.telegram.org/bots/api.md>, one page per method at `https://core.telegram.org/bots/api/methods/<method>.md` (lowercase)
- Mini Apps: <https://core.telegram.org/bots/webapps.md> (section *Serverless*)
- Every project created with `npm create @tgcloud/bot` also ships `AGENTS.md` and `docs/tgcloud-sdk.md`.

This handbook **complements** those pages; when they disagree, the official docs win — and please open an issue so we can update this one.

## Conventions

- Every non-obvious claim is in [`DISCOVERIES.md`](DISCOVERIES.md) with the **date** and **CLI version** it was observed with. The platform is young and changes fast.
- "Verified" means it was reproduced on a real bot. "Unverified" means it is an inference that still needs a test.
- No tokens, IDs or personal data in this repository — ever.

Not affiliated with Telegram.
