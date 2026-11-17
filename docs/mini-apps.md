# Mini Apps on Serverless

Official: <https://core.telegram.org/bots/serverless.md#mini-app-front-end> and <https://core.telegram.org/bots/webapps.md> (*Serverless*).

## Hosting

- `tgcloud.jsonc` → `"static": { "source": "<built folder>" }`. The site is served at `https://app<id>.tgcloud.ai/` (printed by `push`).
- `push` uploads the folder **as it is**: build first. A framework-free Mini App (plain HTML/CSS/JS) needs no build at all — ideal for AI-generated projects.
- Useful config (verified):
  ```jsonc
  { "static": { "source": "webapp",
      "headers": [ { "path": "/", "set": { "Cache-Control": "no-store" } },
                   { "path": "/index.html", "set": { "Cache-Control": "no-store" } } ] } }
  ```
  Telegram's WebView caches aggressively: serve `index.html` with `no-store` and add `?v=N` to your CSS/JS URLs when they change (or use hashed file names + `"immutable"`).
- `.svg` files are served as `image/svg+xml` (verified).
- `/api/` is reserved for endpoints. Don't add `X-Frame-Options` (Telegram Web uses an iframe).

## Calling endpoints

```js
Telegram.WebApp.Serverless.call('getState', { page: 1 }, (err, result) => {
  if (err) { /* err.message, err.type ('ENDPOINT_ERROR', 'UNAUTHORIZED'…), err.status, err.parameters */ return; }
  render(result);
});
```

- The platform verifies `initData`; your endpoint gets `ctx.initData.user`. No login to build.
- `throw new EndpointError('Text for the user', { code: 'X' })` → the client receives `err.type === 'ENDPOINT_ERROR'`, `err.message`, `err.parameters`. Any other exception becomes a generic error (details only in logs).
- `err.status === 0` means the request did not complete (network): retry with backoff and show "Reconnecting…".
- Return the new state from mutating endpoints so the UI can re-render without a second call. Skip expensive parts (statistics) in those responses.

## Practical notes

- Opened in a normal browser (not from Telegram), `Telegram.WebApp.initData` is empty, so endpoint calls cannot be authorized (the docs say they are rejected with `UNAUTHORIZED`). Check `initData` at start-up and show a friendly "open it from the bot" message.
- `Intl.DateTimeFormat().resolvedOptions().timeZone` gives the device time zone: a nice default for per-user time zone settings.
- Set the menu button from the bot (`setChatMenuButton` with `web_app: { url }`) instead of asking users to configure BotFather.
