# Rich Messages for Telegram bots (cheat sheet for AI models)

Rich Messages are a newer kind of Telegram message with real structure: headings, lists, tables, collapsible blocks, footnotes, maps, math, media collages and **buttons placed inside the text**. They are sent with `sendRichMessage` and edited with `editMessageText`. Official reference: <https://core.telegram.org/bots/api/rich-messages.md> (read it if you can browse; this page is the short, practical version).

Models often get them wrong because they treat them like classic `parse_mode: "HTML"` messages. They are **not** the same: the tag set, the escaping rules and the way you send them are different.

## 1. Sending and editing

```js
// Telegram Serverless (JavaScript). In a classic Bot API library use the same JSON.
await api.sendRichMessage({
  chat_id,
  rich_message: { html: '<h3>Title</h3><p>Text</p>' },   // or { markdown: '# Title\n\nText' }
});
// Edit (the target must already be a rich message):
await api.editMessageText({ chat_id, message_id, rich_message: { html } });
```

- Content goes in `rich_message.html` **or** `rich_message.markdown` (pick one per message). The markdown style is GitHub-flavoured and can contain HTML tags; inside block HTML tags (other than `<details>`, `<tg-collage>`, `<tg-slideshow>`) Markdown is **not** parsed.
- `reply_markup` (a normal inline keyboard) also works, but buttons can live **inside** the HTML (section 4).
- Optional: `skip_entity_detection: true` inside `rich_message` stops Telegram from auto-linking URLs, @usernames, #hashtags, $cashtags, /commands, phone numbers and card numbers.
- Other `sendRichMessage` parameters are the usual ones: `disable_notification`, `protect_content`, `message_thread_id`, `reply_parameters`, `reply_markup`, `message_effect_id` (private chats)…

## 2. Limits

- 32,768 UTF-8 characters per message · 500 blocks (nested ones count) · 16 nesting levels · 50 media items · 20 table columns.
- Classic messages are limited to 4,096 characters. Do not reuse classic code that truncates at 4096 for rich messages, but do truncate to 32,768.

## 3. Everything you can put in `html`

Inline text (can nest):

| Tag | Result |
|---|---|
| `<b>`, `<strong>` | bold |
| `<i>`, `<em>` | italic |
| `<u>`, `<ins>` | underline |
| `<s>`, `<strike>`, `<del>` | strikethrough |
| `<code>` | inline code (no language) |
| `<mark>` | highlighted text |
| `<sub>`, `<sup>` | subscript, superscript |
| `<tg-spoiler>` | spoiler |
| `<a href="https://…">` | link; also `mailto:`, `tel:`, `tg://user?id=…` (mention) |
| `<a href="#id">` | link to an anchor or reference in the same message |
| `<tg-reference name="id">` | text that `<a href="#id">` can jump to (footnote-like) |
| `<tg-emoji emoji-id="…">👍</tg-emoji>` | custom emoji (the inner emoji is the fallback) |
| `<img src="tg://emoji?id=…" alt="👍"/>` | custom emoji, alternative syntax |
| `<tg-time unix="1647531900" format="wDT">22:45 tomorrow</tg-time>` | date/time shown in each reader's own time zone and language |
| `<tg-math>x^2 + y^2</tg-math>` | inline formula (raw LaTeX) |

Blocks:

| Tag | Result |
|---|---|
| `<h1>`…`<h6>` | headings |
| `<p>` | paragraph |
| `<pre>` | preformatted block; `<pre><code class="language-python">…</code></pre>` sets the language |
| `<footer>` | small footer text |
| `<hr/>` | divider |
| `<ul><li>`, `<ol><li>` | lists. `<ol start="3" type="a" reversed>`, `<li value="7" type="i">` |
| `<li><input type="checkbox" checked>` | task list item |
| `<blockquote>…<cite>Author</cite></blockquote>` | quotation. `<blockquote expandable>` collapses long quotes |
| `<aside>…<cite>Author</cite></aside>` | pull quote (centered) |
| `<details open><summary>Title</summary>content</details>` | collapsible block; the body can hold any rich content |
| `<table bordered striped compact><caption>…</caption><tr><th>…</th></tr><tr><td colspan rowspan align valign>…</td></tr></table>` | table; cells hold inline formatting only |
| `<tg-math-block>E = mc^2</tg-math-block>` | formula block (LaTeX) |
| `<tg-map lat="41.9" long="12.5" zoom="14"/>` | map |
| `<a name="id"></a>` | anchor target |

Media (each one must be **its own block**, with `http(s)` URLs only; the type is detected from the file):

```html
<img src="https://…/photo.jpg"/>
<video src="https://…/clip.mp4"></video>          <!-- a .gif here becomes an animation -->
<audio src="https://…/song.mp3"></audio>          <!-- .ogg → voice note -->
<tg-document src="https://…/file.zip"></tg-document>
<figure><img src="…" tg-spoiler/><figcaption>Caption<cite>Credit</cite></figcaption></figure>
<tg-collage><img src="…"/><video src="…"/><figcaption>Caption</figcaption></tg-collage>
<tg-slideshow><img src="…"/><img src="…"/></tg-slideshow>
```
Also `tg://photo?id=…`, `tg://video?id=…`, `tg://document?id=…`, `tg://audio?id=…` reuse already uploaded files. The bot needs permission to send media to the chat.

Entities: numeric entities (`&#233;`) work; only these **named** ones are accepted: `&lt; &gt; &amp; &quot; &apos; &nbsp; &hellip; &mdash; &ndash; &lsquo; &rsquo; &ldquo; &rdquo;`. Any tag not listed here is not supported.

## 4. Buttons inside the message

```html
<tg-button-row align="center">          <!-- align: left | center | right (optional) -->
  <tg-button type="callback_data" data="menu:home" style="primary">Open</tg-button>
  <tg-button type="url" url="https://example.org" style="success">Website</tg-button>
</tg-button-row>
```

- `style` is optional: `primary`, `success`, `danger`, `link`.
- Types: `callback_data` (`data`, ≤64 bytes), `url` (`url`), `web_app` (`url`, **private chats only**), `login_url` (`url`, `forward-text`, `request-write-access`), `switch_inline_query`, `switch_inline_query_current_chat`, `switch_inline_query_chosen_chat` (`query`, `allow-user-chats`, `allow-bot-chats`, `allow-group-chats`, `allow-channel-chats`), `copy_text` (`text`, ≤256 chars; a tap copies it), `disabled` (greyed out).
- A `tg-button` inside a `<p>` is an inline button in the middle of a sentence; inside `<tg-button-row>` it is a real row.
- Button labels may contain `<tg-time>` and `<tg-emoji>`.
- Escape dynamic text in labels and attributes (`& < > "`).
- Callbacks arrive as normal `callback_query` updates.

## 5. Gotchas that break messages

1. **Escape every dynamic value** (`&`, `<`, `>`, and `"` in attributes) before putting it in HTML. User text left unescaped breaks the message or injects tags.
2. **Line breaks:** in rich HTML use `<p>` per paragraph, or `<br>` inside `<blockquote>` and `<details>`. Classic `parse_mode: "HTML"` does **not** accept `<br>`; use `\n` there.
3. **Media only as separate blocks**, never inside a paragraph or a table cell.
4. **Table cells** accept inline formatting only (no lists, no paragraphs).
5. `editMessageText` with `rich_message` only edits rich messages, and returns "message is not modified" if nothing changed: treat that as success. Don't rely on turning a classic text message into a rich one by editing: if the edit fails, send a new message.
6. Not every client or chat type is guaranteed to accept rich messages (older clients, channels, business chats). **Always keep a fallback.**
7. Unsupported named entities (`&copy;`, `&euro;`…) are rejected: write the character itself or a numeric entity.
8. Don't forget that Telegram shows an "Open this link?" alert for inline links.

## 6. Recommended pattern: three-level fallback

Try the best format first and degrade without crashing the bot:

1. `sendRichMessage` with buttons embedded (`<tg-button-row>`);
2. `sendRichMessage` + `reply_markup` (inline keyboard);
3. classic `sendMessage` with `parse_mode: "HTML"` (convert headings to `<b>`, lists to `• `, tables to `label: value` lines).

```js
import { api } from 'sdk';

// screen = { html: '<h3>…</h3>…', classic: '<b>…</b>\n…', rows: [[{ t: 'Open', d: 'menu:home' }]] }
export async function sendScreen(chat_id, screen) {
  const esc = (s) => String(s).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;');
  const embedded = screen.rows.map((r) =>
    '<tg-button-row>' + r.map((b) => b.url
      ? `<tg-button type="url" url="${esc(b.url)}">${esc(b.t)}</tg-button>`
      : `<tg-button type="callback_data" data="${esc(b.d)}">${esc(b.t)}</tg-button>`).join('') + '</tg-button-row>').join('');
  const keyboard = { inline_keyboard: screen.rows.map((r) => r.map((b) => b.url ? { text: b.t, url: b.url } : { text: b.t, callback_data: b.d })) };
  try {
    return await api.sendRichMessage({ chat_id, rich_message: { html: (screen.html + embedded).slice(0, 32000) } });
  } catch (e1) {
    try {
      return await api.sendRichMessage({ chat_id, rich_message: { html: screen.html.slice(0, 32000) }, reply_markup: keyboard });
    } catch (e2) {
      return await api.sendMessage({ chat_id, text: screen.classic.slice(0, 4000), parse_mode: 'HTML', reply_markup: keyboard });
    }
  }
}
```

If an error means the user blocked the bot or the chat does not exist, do not fall back: stop. Optionally remember (in the database) the level that worked for an hour to avoid repeating failing calls.

## 7. Design tips

- One screen = one message: a heading, 1–3 short paragraphs or a table, then one or two button rows. Move details into `<details>`.
- Use `<table>` for key/value data, `<blockquote expandable>` for long quotes, `<tg-time>` for any date so each user sees their own time zone.
- Keep button labels short; use `style` sparingly (`danger` only for destructive actions, `success`/`primary` for the main action).
- Prefer editing the current screen (`editMessageText`) when the user navigates with buttons, instead of sending a new message each time.
