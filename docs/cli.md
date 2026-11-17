# The `tgcloud` CLI in practice

Package `@tgcloud/cli` (observed: **0.2.0**). Official reference: <https://core.telegram.org/bots/serverless.md#command-line-interface>.

## Installing and running

- `npm create @tgcloud/bot <folder>` scaffolds a project and installs the CLI as a dev dependency (`npx tgcloud …`). `npm install -g @tgcloud/cli` gives a global `tgcloud`.
- The CLI runs fine on **Node 18**, even if the project was created with a newer Node.
- No `npm` on the machine? Run it directly: `node node_modules/@tgcloud/cli/bin/tgcloud.js <command>`.
- Its dependencies are small (chalk, commander, diff, json5, ora). `npm install --ignore-scripts` works.

## Authentication without a terminal

- `tgcloud login` asks for the token interactively and **needs a TTY**. Agents and scripts must use the **`TGCLOUD_TOKEN`** environment variable instead (it is never written to disk).
- Token format: `app<digits>:<secret>`. Anything else → `TGCLOUD_TOKEN has invalid format. Expected "app<id>:<secret>".` (this happens before any network call).
- `--debug` logs HTTP requests with the secret masked.

## What makes a folder a "project"

- The CLI looks for a **`.tgcloud/` directory** in the current folder or its parents. That's all: an **empty** `.tgcloud/` is enough.
- `tgcloud fetch` then fills it with `state.json` (`last_known_revision`), `capabilities.json` (layout rules and valid handler names), `static.json` and `remote/` (a snapshot of the deployed modules).
- This makes it easy to deploy any folder that has a `tgcloud/` subfolder: `mkdir .tgcloud && tgcloud fetch && tgcloud status && tgcloud push`.

## Non-interactive flags

| Command | In scripts / agents |
|---|---|
| everything | add `--no-pager` |
| `migrate` | without a TTY it refuses to run unless you pass `--safe` (apply safe changes, skip warnings), `--yes` (apply safe **and** destructive, skip manual) or `--dry-run` |
| `migrate --local` | diff against your **local** `schema.js` instead of the deployed one — you can create tables **before** the first push |
| `push --force` | skip the revision check (overwrites the cloud) — only when you know nobody else deployed |

## Output formats (useful for automation)

`tgcloud status` (offline, after `fetch`):

```
Linked to app123 (from env)
Local revision: 7
Webhook: in sync

Modified:
  tgcloud/handlers/message.js
New (not yet deployed):
  tgcloud/lib/reply.js
Deleted locally (still in snapshot):
  tgcloud/endpoints/old.js

Static (webapp/ → https://app123.tgcloud.ai/)
Modified:
  webapp/index.html
```

After the `Static (…)` header, the same section names refer to Mini App files. When everything matches: `Working directory clean.`

`tgcloud migrate --dry-run` lists one change per line:

```
Schema out of sync — 4 changes:
  +  hits                new table          ← safe
  ✗  admins              drop table         ← warning (destructive)
  ~  users.name          type change        ← manual (example)
  ?  chats               not in schema      ← exists in the DB, not in schema.js (left alone)
```

`tgcloud push` ends with `Done in …`, prints `Static: https://app<id>.tgcloud.ai/` when there is a Mini App, and `Schema out of sync — N changes … Run "tgcloud migrate"` when the database needs a migration.

`tgcloud run` prints `> handler(payload)`, `- Running...`, `console.*` lines (`!` prefix for `console.warn`) and finally `< <return value>  <ms>` or `✗ Uncaught exception: …`.

## `run` touches real things

`tgcloud run handlers/<name> '<json5>'` executes your **local** code on the platform **against the real database and the real Bot API** of the bot. It is the best test tool, but:
- use fake chat IDs (`chat: { id: 1 }`): the logic runs and only the final send fails with `chat not found`;
- clean up rows you create (a temporary endpoint that deletes them works well — and delete that endpoint afterwards);
- if your code notifies an owner/admin by ID, temporarily point that ID to a fake one.

## Revisions and concurrency

Every deploy bumps a revision. `push` sends the revision you last synced; if the cloud moved ahead, it refuses. `fetch` / `pull` resync; `push --force` overwrites.
