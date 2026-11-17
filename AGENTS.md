# AGENTS.md — rules for AI agents building Telegram Serverless bots

Read this file completely before writing code. Then read, in this order:
1. <https://core.telegram.org/bots/serverless.md> (official, raw Markdown)
2. The project's own `AGENTS.md` and `docs/tgcloud-sdk.md` (created by `npm create @tgcloud/bot`)
3. The `docs/` folder of this handbook — especially [`docs/cli.md`](docs/cli.md), [`docs/database.md`](docs/database.md) and [`docs/patterns.md`](docs/patterns.md)
4. Bot API pages for every method you call: `https://core.telegram.org/bots/api/methods/<method>.md`

## What you need from the user (ask once, at the start)

| Ask | Why |
|---|---|
| What the bot must do | The spec |
| Bot username | To mention it in texts and links |
| **Serverless access token** (`app<id>:<secret>`) — @BotFather → bot → Serverless → CLI Access | To deploy. It is **not** the Bot API token, which you do **not** need |
| Their numeric Telegram user ID (optional) | If the bot needs an owner/admin |
| Bot language, whether a Mini App is wanted | Product decisions |
| Permission to deploy | Deploying publishes the bot immediately |

Never ask for the Bot API token, passwords or anything else. Never print the token secret, never write it into project files, commit it, or put it in command-line arguments: pass it as the `TGCLOUD_TOKEN` environment variable. When you finish, tell the user to **regenerate the access token**.

## Hard rules of the platform

- Deployed code = `.js` files under `tgcloud/`: `schema.js`, `lib/**`, flat `handlers/<update_type>.js`, flat `endpoints/<identifier>.js`. Nothing else is deployed as code.
- ES modules only. Imports: `sdk`, `sdk/db`, `sdk/api`, `sdk/fetch`, and project files by **relative path ending in `.js`**. No npm packages, no Node built-ins, no `process.env`, no `require`.
- Handler: `export default async function (payload, ctx)`; endpoint: `export default async function (input, ctx)` with the verified user in `ctx.initData.user`.
- Every DB call is `await`ed. No foreign keys. Drops only via `.deprecated()` + `migrate`.
- **Deploying (`push`) never changes the database. Run `migrate` after schema changes.**
- No cron / scheduled jobs. Do periodic work opportunistically inside handlers.

## Working without a human at the keyboard

The CLI has interactive parts. In an agent (no TTY):
- Do **not** run `tgcloud login` (needs a TTY). Export `TGCLOUD_TOKEN` instead.
- `migrate` refuses to run without a TTY unless you pass a flag: use `--safe` (safe changes only) or, after reviewing `--dry-run`, `--yes` (also destructive).
- Add `--no-pager` to every command.
- Node 18+ is enough for the CLI. If `npm` is missing, you can still run the CLI with `node node_modules/@tgcloud/cli/bin/tgcloud.js`.

## The loop

1. `npm create @tgcloud/bot <folder>` (or reuse an existing project).
2. Write the code. Keep `schema.js`, handlers and endpoints small; share code in `lib/`.
3. Test **before deploying** with `tgcloud run handlers/<name> '<json5 payload>'` — it runs your **local** files on Telegram's servers. ⚠️ It uses the **real database** of the bot and really calls the Bot API: use fake chat IDs (`chat: { id: 1 }` → the call fails with "chat not found" *after* your logic ran), and clean up any rows you create.
4. If `schema.js` changed: `tgcloud migrate --local --dry-run` to see the plan, then `tgcloud migrate --local --safe` (works before pushing).
5. `tgcloud push --no-pager`, then `tgcloud migrate --safe` if it reports pending changes.
6. Verify: `tgcloud webhook` (must say *in sync*, `allowed updates` must list your handlers) and `curl` the Mini App URL if there is one.
7. Report to the user: what was deployed, how to try it, and that they should regenerate the token.

## Quality bar

- Validate inputs in endpoints; check ownership on every action in multi-user bots (see `docs/patterns.md`).
- Handle `BotApiError` where failure is expected (blocked users, deleted messages, missing rights).
- Escape any user text you put into HTML messages.
- Keep handlers fast: do one update's work and return.
