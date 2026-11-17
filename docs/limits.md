# Limits and missing pieces

| Area | Limit / status | Source |
|---|---|---|
| Upload a file to Telegram | 50 MB per file, 10 MB per photo | Official docs (Bot API) |
| Download a user's file (`getFile`, `api.getFileContent`) | 20 MB | Official docs (Bot API) |
| `fetch` response | 30 MB total (streaming does not raise it) | Official docs |
| Batch insert | SQLite variable limit (rows × columns) | Official docs |
| Foreign keys | Not supported (throw at declaration) | Official docs |
| Scheduled jobs / cron | **None** | Official docs (absence) — verified in practice |
| npm packages at runtime | None | Official docs |
| CPU / wall time per invocation | Not published | — |
| Requests per second | Not published | — |
| Database size | Not published | — |
| Mini App hosting | Static files only; `/api/` reserved | Official docs |
| CLI login | Needs a TTY; use `TGCLOUD_TOKEN` in automation | Verified |
| `migrate` | Needs a TTY unless `--safe`, `--yes` or `--dry-run` | Official docs + verified |

## Things that would help (wish list)

- Scheduled handlers (cron-like).
- A documented deploy API (today: [`management-api.md`](management-api.md) is reverse-engineered).
- Published execution limits.
