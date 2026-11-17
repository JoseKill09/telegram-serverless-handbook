# Inside the isolate: what exists

Official SDK reference: `docs/tgcloud-sdk.md` in every scaffolded project, and <https://core.telegram.org/bots/serverless.md#the-sdk>.

## Available (verified)

| Thing | Notes |
|---|---|
| `sdk` → `db`, `api`, `fetch`, `InputFile`, `BotApiError`, `EndpointError`, `FormData` | The whole platform surface |
| `console.log/info/warn/error` | Shown by `tgcloud run` (`!` for warnings) |
| Modern JS: `async/await`, classes, `Map`, `Set`, optional chaining, `??`, spread… | — |
| **`Intl`** with **time zones** (`Intl.DateTimeFormat('es-ES', { timeZone: 'America/Mexico_City' })`, `formatToParts`, `hourCycle`) | Enough to implement per-user time zones without libraries |
| `String.prototype.normalize('NFD')` and Unicode property escapes in regexes (`/\p{L}/u`, `/\p{M}/gu`) | Accent-insensitive matching works |
| Regex lookbehind `(?<!…)` | Whole-word matching with Unicode letters |
| `Math.random()`, `Date.now()` | — |
| `Uint8Array`, `InputFile(bytes, name, { type })` | Upload files built in memory (`api.sendDocument`, `api.sendPhoto`) |
| `api.getFileContent(file_id)` → `Uint8Array` | Download a user's file (≤ 20 MB, Bot API limit) |

## Not available

- npm packages, `require`, Node built-ins (`fs`, `path`, `crypto`, `Buffer`, `process`).
- Filesystem.
- Background work after the handler returns; timers to "do something later"; cron.

## Unverified / use with care

- `TextEncoder` / `TextDecoder`: standard in V8 isolates, but not listed in the docs. Ship a tiny UTF-8 fallback if you depend on it.
- Dynamic `import()`: not documented — use static imports.
- `setTimeout` inside a handler: not documented; there is no guarantee the isolate stays alive.
- CPU/time limits per invocation are not published.

## Module state

Module-level variables may survive between invocations in the same isolate, but nothing guarantees it, and invocations may run concurrently. Use them only as caches, never as the source of truth; never keep per-request data (like "the current user's time zone") in module globals — pass it explicitly.
