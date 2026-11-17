# Patterns that work

Tested in real bots in production on Telegram Serverless (CLI 0.2.0, November 2026).

## Multi-user bots (each user has a private space)

- Every table that holds user data gets a `tenant_id` (the Telegram user ID) and an index on it.
- **All authorization lives in one module** (e.g. `lib/actions.js`) used by both handlers and endpoints: `ownRule(uid, id)`, `ownChat(uid, id)`… throw a typed error when the resource is not theirs. The Mini App only *hides* buttons; the server decides.
- When a user adds the bot to a channel/group, assign the chat to them **only after** checking `getChatMember(chat, user)` is `administrator`/`creator`. If the chat already belongs to someone who is still admin, refuse.
- A fixed bot owner: a constant `OWNER_ID` (there is no `.env` on Serverless) plus settings in the database for everything else (open sign-ups, limits, blocked users).

## Testing without breaking real data

- `tgcloud run` hits the real database and Bot API. Use **fake IDs** (`111111`, `222222`) as users and chats.
- Seed fixtures with a **temporary endpoint** (`endpoints/tmpSeed.js`), run it with `tgcloud run endpoints/tmpSeed '{}'`, and delete the file before the next `push`. Clean only the rows of your fake IDs; never empty tables of a live bot.
- If your code messages the owner, swap `OWNER_ID` for a fake value during tests and restore it before deploying.
- Test isolation between users explicitly: user B must not see or modify user A's data.

## No cron

- Housekeeping (delete old history, expired states) runs **opportunistically**: e.g. `if (Math.random() < 0.05) await housekeeping()` at the end of a frequent handler.
- Time-window rules (only between 22:00 and 08:00, weekdays…) are evaluated when the update arrives, using the user's time zone with `Intl.DateTimeFormat(…, { timeZone })`.
- Anything that must happen *later* (reminders, digests) has no clean solution yet: you need an external trigger or user activity.

## Conversations in a private chat

- Keep a `states` table (`user_id` primary key, `kind`, `data` JSON, the menu `message_id`). A button sets the state ("waiting for a name"); the next text message consumes it.
- Any button press clears the pending state (so a stale state never captures unrelated text).
- Edit the menu message in place (`editMessageText`) instead of sending new messages.

## Channel → forum topics (forwarding bots)

- Albums arrive as **separate updates**, one per item, possibly concurrently. Decide once per `media_group_id` with an atomic insert (`onConflictDoNothing().returning()`): the first item decides, the others follow its decision.
- Duplicates (two channels posting the same thing): fingerprint posts by the largest `file_unique_id` of their media or by a hash of the normalized text, and skip if the same fingerprint reached the same topic in the last 24 h.
- To propagate edits (`edited_channel_post`), store the **destination message IDs** of each copy; forwarded messages cannot be edited, copies can (`editMessageText` / `editMessageCaption`).

## Text matching for rules

- Accent/case-insensitive: `s.normalize('NFD').replace(/\p{M}/gu, '').toLowerCase()`.
- Whole words with Unicode: `new RegExp(\`(?<![\\p{L}\\p{N}_])${escaped}(?![\\p{L}\\p{N}_])\`, 'u')`.
- User-supplied regexes: cap their length and reject nested quantifiers like `(a+)+` before compiling (catastrophic backtracking would block your handler).

## Rich Messages with fallback

Try `sendRichMessage` with embedded buttons → rich with `reply_markup` → classic HTML. When a level fails and the next one works, remember it for an hour (in the database) to avoid repeating the failure.

## Moving a project to another bot

Code moves; data doesn't (one database per bot). Steps: new access token, update any hard-coded `app<id>` URLs, deploy into a folder **without** the old `.tgcloud/` (or with a fresh `fetch`), `migrate`, then re-run your one-time setup (`setMyCommands`, menu button…). Users re-create their data or use your export/import.
