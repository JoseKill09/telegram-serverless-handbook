# The Bot API from Serverless

`import { api } from 'sdk'`, then `await api.<method>({ snake_case_params })`. The `{ ok, result }` envelope is unwrapped; failures throw `BotApiError` with `.code`, `.description`, `.method`, `.parameters`. **Any** Bot API method works, including brand-new ones, because `api` is a proxy.

## Verified details

- New methods with no SDK helper work as-is, e.g. **Rich Messages**:
  ```js
  await api.sendRichMessage({ chat_id, rich_message: { html: '<h3>Title</h3><p>Text</p>' } });
  await api.editMessageText({ chat_id, message_id, rich_message: { html } });   // edit a rich message
  ```
  Full reference, limits, gotchas and a fallback pattern: [rich-messages.md](rich-messages.md).
  Buttons can live inside the HTML (`<tg-button-row><tg-button type="callback_data" data="x">OK</tg-button></tg-button-row>`) or in `reply_markup`. Keep a fallback to classic `sendMessage` with `parse_mode: 'HTML'`: if rich fails for a user/client, retry one level simpler. Spec: <https://core.telegram.org/bots/api/rich-messages.md>.
  In rich HTML, use `<br>` for line breaks inside `<blockquote>` and `<details>`; classic HTML does **not** accept `<br>` (use `\n`).
- `copyMessage` returns `{ message_id }`; `copyMessages` / `forwardMessages` return arrays; `forwardMessage` returns a `Message`. Keep the new IDs if you will need to edit the copies later.
- `copyMessage` accepts `caption` + `caption_entities` to **replace** a media caption while copying. A text message cannot be "copied with changes": send it again with `sendMessage({ text, entities })`.
- Entity offsets are in **UTF-16 code units** — exactly JavaScript's `string.length`, so shifting entities after prepending text is just `offset + prefix.length`, even with emoji.
- Errors you will meet: `400 chat not found` (fake IDs in tests), `403` (user blocked the bot), `message to edit not found`, `message is not modified` (harmless), `message thread not found` (forum topic deleted).
- `getMe` works; cache the result in the database (it never changes).
- Setting the profile from code works: `setMyName`, `setMyDescription`, `setMyShortDescription`, `setMyCommands` (with scopes), `setChatMenuButton` (with or without `chat_id`). Do it once per version (store a "setup done for vX" flag), not on every update.
- `request_chat` keyboard buttons can ask Telegram to add the bot as admin with specific rights (`bot_administrator_rights`): the user picks a channel/group and you receive `chat_shared` **and** a `my_chat_member` update.
- Before trusting "the user who added the bot", verify with `getChatMember(chat, user)` that they are `administrator`/`creator`.

## Webhook

The platform manages the webhook. `allowed_updates` = the handler files you deployed. If someone sets a webhook with the bot token, `tgcloud webhook` shows it out of sync; `tgcloud webhook sync` fixes it.
