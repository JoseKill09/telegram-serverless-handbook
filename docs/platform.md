# The platform in one page

Telegram Serverless runs your bot's backend **on Telegram's own servers**, in a V8 isolate next to the Bot API and a per-bot SQLite database. You never run a server. Official overview: <https://core.telegram.org/bots/serverless.md>.

## Mental model

```
your folder                         Telegram's cloud (app<id>)
├─ tgcloud/                ──push──▶  modules (JS)          ◀── webhook ── Telegram updates
│  ├─ schema.js                       database (SQLite)      ◀── migrate
│  ├─ handlers/<type>.js              Bot API (api.*)
│  ├─ endpoints/<name>.js             Mini App site  https://app<id>.tgcloud.ai/
│  └─ lib/**.js                         └─ /api/<endpoint>  (Telegram.WebApp.Serverless.call)
├─ webapp/ (built Mini App) ──push──▶
└─ tgcloud.jsonc
```

- An update arrives → the platform calls `export default` of `handlers/<update_type>.js` with the update's object (`message`, `callback_query`…) and `ctx` (`ctx.update` = raw update). No handler file → the update type is not even delivered (`allowed_updates` is derived from your handlers).
- The Mini App calls `endpoints/<name>.js` with `Telegram.WebApp.Serverless.call(name, input, cb)`; the platform verifies `initData` first and gives you `ctx.initData.user`.
- Each bot (each `app<id>`) has its **own** database. Moving code to another bot does not move data.

## Valid handler names

The CLI downloads the allowed list from the server into `.tgcloud/capabilities.json` (`directories[].allowed_names`). On 2026-11-17 (CLI 0.2.0) it was:

`message`, `edited_message`, `stopped_message_generation`, `channel_post`, `edited_channel_post`, `business_connection`, `business_message`, `edited_business_message`, `deleted_business_messages`, `inline_query`, `chosen_inline_result`, `callback_query`, `widget_query`, `shared_widget_query`, `guest_message`, `shipping_query`, `pre_checkout_query`, `purchased_paid_media`, `subscription`, `message_reaction`, `message_reaction_count`, `poll`, `poll_answer`, `my_chat_member`, `chat_member`, `chat_join_request`, `chat_boost`, `removed_chat_boost`, `managed_bot`.

Note `message_reaction*` and `chat_member` updates also need the bot to be admin where relevant (normal Bot API rules).

## What gets deployed

- `.js` files under `tgcloud/` in the four allowed places (`schema.js`, `lib/**`, `handlers/*`, `endpoints/*`). Other `.js` inside `tgcloud/` are flagged by `status`.
- The Mini App build folder named in `tgcloud.jsonc` → `"static": { "source": "<folder>" }`.
- A full `push` makes the cloud **mirror** your folder: modules missing locally are **deleted** in the cloud.
- If `tgcloud.jsonc` has **no** `static` key, an already deployed Mini App is left untouched (`status` warns about it). `"static": false` removes it.

## Two places to edit the same project

Everything the CLI manages can also be edited in @BotFather → your bot → **Serverless** (Handlers, Endpoints, Library, Database, Mini App Files, CLI Access) — from a phone. It is the same project: `tgcloud pull` brings phone edits to your folder.
