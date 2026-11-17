# Only an AI chat? Use Shipcast

If your AI cannot run commands (a normal chat in Claude, ChatGPT, Gemini…), it can still write the whole bot, but it cannot deploy it. Use **[Shipcast](https://github.com/JoseKill09/shipcast)**:

1. Give your AI the prompt in [`shipcast/prompts/build-for-shipcast.md`](https://github.com/JoseKill09/shipcast/blob/main/prompts/build-for-shipcast.md) and describe your bot.
2. It gives you a ZIP in the [Shipcast format](https://github.com/JoseKill09/shipcast/blob/main/docs/FORMAT.md).
3. Open the Shipcast bot in Telegram → 🚀 Publish → send your Serverless access token and the ZIP → review → Publish.
4. If Shipcast finds problems, it gives you an error report: paste it back to your AI and send the new ZIP.
5. Regenerate the access token in @BotFather when you are done.

### Could a chat AI deploy by itself?

Only if its code-execution sandbox can reach `cloud.telegram.org`. The CLI is a thin client over a simple HTTP API (see [`docs/management-api.md`](../docs/management-api.md)), so an AI with unrestricted network could deploy even without npm. Most chat sandboxes restrict the network, so this is not something to rely on — **unverified** as of 2026-11-17.

Without any of this, a person can still paste each file by hand into @BotFather (bot → Serverless → Handlers / Endpoints / Library / Database / Mini App Files): it works from a phone, but it is manual.
