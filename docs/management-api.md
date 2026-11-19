# The management API behind the CLI (undocumented)

> **Unofficial and subject to change.** Prefer the CLI. This page exists so that tools and AIs know what the CLI talks to — for example to allow it through a network allow-list, or to build a deployer.

Observed in `@tgcloud/cli` 0.2.0 source (`src/api/client.js`), 2026-11-17.

- Base URL: **`https://cloud.telegram.org`** (overridable with `TG_CLOUD_API_URL`; `TGCLOUD_BETA=1` appends `/beta`).
- Every request: `https://cloud.telegram.org/<app_id>/manage<path>` where `<app_id>` is the public part of the token (`app123…`).
- Header: `Authorization: Bearer <full token>`; JSON bodies (`Content-Type: application/json`).
- Responses use an envelope that the CLI unwraps (`parseEnvelope`).

| Method | Path | Used by |
|---|---|---|
| GET | `/get` | `fetch`, `status` (snapshot of modules, revision, capabilities) |
| POST | `/deploy` | `push` |
| POST | `/run` | `run` |
| GET / POST | `/migrate` | `migrate` (plan / apply) |
| GET | `/webhook` | `webhook` |
| POST | `/webhook/sync` | `webhook sync` |
| POST | `/static/prepare` | `push` (Mini App: which files must be uploaded) |
| PUT | `/static/blob/<hash>?bundle=<bundle>` | `push` (raw bytes, `Content-Type: application/octet-stream`, explicit `Content-Length`) |

Consequences:
- To deploy, a machine only needs to reach **`cloud.telegram.org`** (plus `registry.npmjs.org` if it installs the CLI).
- A deployer could, in principle, speak this API directly (even from another Serverless bot with `sdk` `fetch`). The body formats of `/deploy` and `/static/*` are not documented — read the CLI source before trying, and expect changes. **Unverified** beyond reading the source.

## Downloading a bot's published code and Mini App (export / migrate to another bot)

- `tgcloud fetch` already stores a **snapshot of the deployed modules** in `.tgcloud/remote/` (same files as `tgcloud pull`, which writes them into `tgcloud/`). `pull` does **not** download the Mini App.
- The Mini App's file list is in `.tgcloud/static.json` (`manifest`: path → `{hash, size}`, plus `config` with `headers`, `spa`, `immutable`, `redirects`). Each file can be downloaded with a plain **`GET https://cloud.telegram.org/<app_id>/manage/static/blob/<hash>`** and `Authorization: Bearer <token>` (verified: HTTP 200 with the raw bytes).
- With both you can rebuild a complete project (`tgcloud/` + `webapp/` + a `tgcloud.jsonc` with `static.source` and the same `headers`) and publish it to a different bot. A rebuilt project re-checked against the original bot shows "nothing to change". Database **data** is not part of any of this (only `schema.js`).

