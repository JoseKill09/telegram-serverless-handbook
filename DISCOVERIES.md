# Discoveries log

Findings that the official documentation does not state (or states differently). Newest first.
**Status:** ✅ verified on a real bot · 🔎 read in the CLI source · ❓ unverified inference.

| Date | CLI | Status | Finding | Details |
|---|---|---|---|---|
| 2026-11-17 | 0.2.0 | ✅ | An **empty `.tgcloud/` folder** is enough for the CLI to treat a folder as a project; `fetch` fills `state.json`, `capabilities.json`, `static.json`, `remote/`. | [cli.md](docs/cli.md#what-makes-a-folder-a-project) |
| 2026-11-17 | 0.2.0 | ✅ | `.tgcloud/capabilities.json` contains the **live list of valid handler names** (`directories[].allowed_names`). | [platform.md](docs/platform.md#valid-handler-names) |
| 2026-11-17 | 0.2.0 | 🔎 | The CLI only talks to **`https://cloud.telegram.org/<app_id>/manage/…`** with `Authorization: Bearer <token>`: `/get`, `/deploy`, `/run`, `/migrate`, `/webhook`, `/webhook/sync`, `/static/prepare`, `/static/blob/<hash>`. | [management-api.md](docs/management-api.md) |
| 2026-11-17 | 0.2.0 | ✅ | `migrate --dry-run` marks tables that exist in the database but not in `schema.js` with **`?`** ("not in schema"); they are left alone. | [cli.md](docs/cli.md#output-formats-useful-for-automation) |
| 2026-11-17 | 0.2.0 | ✅ | If `tgcloud.jsonc` has no `static` key, an already deployed Mini App **stays** and `status` warns "the bot has a deployed Mini App that this project does not manage". | [platform.md](docs/platform.md#what-gets-deployed) |
| 2026-11-17 | 0.2.0 | ✅ | In `status`, after the `Static (…)` header the sections `Modified:` / `New…:` refer to Mini App files. | [cli.md](docs/cli.md) |
| 2026-11-17 | 0.2.0 | ✅ | `Intl` with arbitrary IANA **time zones** works in the runtime (`formatToParts`, `hourCycle: 'h23'`). | [runtime.md](docs/runtime.md) |
| 2026-11-17 | 0.2.0 | ✅ | `desc(sql\`…\`)` generates invalid SQL (`no such column: "undefined"`); use ``sql`… DESC` ``. | [database.md](docs/database.md) |
| 2026-11-17 | 0.2.0 | ✅ | `.returning().run()` resolves to an **array** of rows. | [database.md](docs/database.md) |
| 2026-11-17 | 0.2.0 | ✅ | `migrate --local` diffs against the local `schema.js`: tables can be created **before** the first `push`. | [cli.md](docs/cli.md#non-interactive-flags) |
| 2026-11-17 | 0.2.0 | ✅ | `tgcloud run` executes against the **real database and Bot API**; fake chat IDs make the final send fail with `chat not found` after the logic ran. | [cli.md](docs/cli.md#run-touches-real-things) |
| 2026-11-17 | 0.2.0 | ✅ | `tgcloud login` needs a TTY; `TGCLOUD_TOKEN` works everywhere. The CLI runs on **Node 18**. | [cli.md](docs/cli.md) |
| 2026-11-17 | 0.2.0 | ✅ | New Bot API methods such as **`sendRichMessage`** work through the `api` proxy; `editMessageText` accepts `rich_message`. | [bot-api.md](docs/bot-api.md) |
| 2026-11-17 | 0.2.0 | ✅ | Removing a table needs `.deprecated()` + `migrate --yes` (each drop is a warning); deleting the declaration alone leaves it as `?`. | [database.md](docs/database.md#schema-and-migrations) |
| 2026-11-17 | 0.2.0 | ✅ | `.svg` files in the Mini App are served as `image/svg+xml`; custom `headers` (e.g. `Cache-Control: no-store`) apply. | [mini-apps.md](docs/mini-apps.md) |
| 2026-11-17 | 0.2.0 | ✅ | `allowed_updates` is refreshed automatically on `push` when handler files are added or removed (e.g. adding `edited_channel_post`). | [bot-api.md](docs/bot-api.md#webhook) |

## How to add a finding

1. Reproduce it on a test bot (never a production bot's data) and note the CLI version (`tgcloud --version`).
2. Add a row here (newest first) and a short explanation in the matching `docs/` page.
3. No tokens, IDs or personal data.
