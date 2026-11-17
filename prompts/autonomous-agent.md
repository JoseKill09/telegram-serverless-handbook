# Prompt: build and deploy a Telegram Serverless bot autonomously

For AI agents that can run commands and access the internet (Claude Code — including claude.ai/code in a browser —, Cursor agent, Codex, opencode…).
Copy everything below the line, paste it to the agent, and add one sentence describing the bot you want.

**Network the agent needs:** `registry.npmjs.org` (to install the CLI), `cloud.telegram.org` (deploy) and `core.telegram.org` (docs). In sandboxes with an allow-list, add those domains.

---

You are an autonomous software agent. Build, test and deploy a **Telegram bot on Telegram Serverless** (`tgcloud`) from start to finish, with as little interaction with me as possible. I am not a programmer.

## Step 0 — Learn the platform (do not skip)

Read, in this order, and follow what they say:
1. https://raw.githubusercontent.com/JoseKill09/telegram-serverless-handbook/main/AGENTS.md — rules for agents (the most important page).
2. https://core.telegram.org/bots/serverless.md — official docs.
3. The handbook docs you need: https://github.com/JoseKill09/telegram-serverless-handbook/tree/main/docs (`cli.md`, `database.md`, `patterns.md` at least).
4. For every Bot API method you use: `https://core.telegram.org/bots/api/methods/<method>.md` (lowercase). For a Mini App: https://core.telegram.org/bots/webapps.md.

## Step 1 — Ask me, once, everything you need

In a single message, ask me for:
- What the bot must do (if my description is not enough), its language, and whether I want a Mini App.
- The bot's @username.
- The **Serverless access token**: in @BotFather → my bot → **Serverless** (turn it on if it is off) → **CLI Access** → Access token. It looks like `app123:…`. Tell me it is *not* the Bot API token and that you don't need that one.
- My numeric Telegram ID, only if the bot needs an owner/admin.
- Permission to deploy when it is ready (say that deploying makes the bot live at once).

Then work alone. Make reasonable product decisions yourself and list them at the end.

## Step 2 — Build

- Create the project with `npm create @tgcloud/bot <folder>` (needs Node 18+). Work only inside it.
- Export the token as `TGCLOUD_TOKEN` (never `tgcloud login`, never write it to a file, never print it, never put it in a command-line argument or a commit).
- Follow every rule in the handbook's AGENTS.md: `tgcloud/` layout, ES modules, relative imports ending in `.js`, only `sdk*` imports, `export default` handlers/endpoints, awaited DB calls, no foreign keys, no cron.
- Make the bot pleasant: a `/start` with clear buttons, short texts, errors explained to the user.

## Step 3 — Test before deploying

- Syntax: `node --check` every module (copy it to a `.mjs` file first if Node complains about modules).
- Logic: `npx tgcloud run handlers/<type> '<json5 payload>' --no-pager` runs your local code on Telegram's servers. It uses the **real database** and really calls the Bot API: use fake IDs (e.g. `chat: { id: 1 }`); an error like *chat not found* at the very end means your logic ran. Delete any test rows you created.
- Endpoints: `npx tgcloud run endpoints/<name> '{…}' --ctx '{ initData: { user: { id: 1, first_name: "Test" } } }' --no-pager`.
- Schema: `npx tgcloud migrate --local --dry-run --no-pager` to see the plan.

## Step 4 — Deploy (after my permission)

1. `npx tgcloud push --no-pager`
2. If the schema changed: `npx tgcloud migrate --safe --no-pager`. Only use `--yes` (destructive) if I explicitly agreed to delete data.
3. `npx tgcloud webhook --no-pager` must say *in sync* and list all your handler types. If not: `npx tgcloud webhook sync`.
4. With a Mini App: open the URL `push` printed and check it loads; set it as the menu button from the code (`setChatMenuButton`) or tell me to set it in @BotFather.

## Step 5 — Hand over

Tell me, in plain language:
- What the bot does and how to try it (what to send it).
- The decisions you made and anything I should configure.
- **Regenerate the access token now** (@BotFather → bot → Serverless → CLI Access) because you used it.
- If you learned something about Serverless that is not in the handbook, show it to me as a short note so I can add it to DISCOVERIES.md.
