# pi-telegram-multiacc

A Telegram DM bridge for [pi](https://pi.dev) with support for multiple authorized Telegram accounts and a separate bot for each pi session.

Use it when you want, for example:

- one bot for a work pi session and another for personal work
- two pi sessions running at once, each controlled through a different bot
- multiple trusted Telegram accounts that can message the same bot

This independent extension is based on [badlogic/pi-telegram](https://github.com/badlogic/pi-telegram).

## Install

Install globally for your user:

```bash
pi install git:github.com/musichen/pi-telegram-multiacc
```

Install only for the current project:

```bash
pi install -l git:github.com/musichen/pi-telegram-multiacc
```

Try it for one pi run without installing it:

```bash
pi -e git:github.com/musichen/pi-telegram-multiacc
```

Update installed pi packages later with:

```bash
pi update --extensions
```

> Extensions run with your full system permissions. Review the source before installing it.

## Create Telegram bots

1. Open [@BotFather](https://t.me/BotFather).
2. Run `/newbot` once for each bot you want to use.
3. Choose each bot's name and username.
4. Copy each bot token.

## First bot

Start pi and configure the first bot:

```text
/telegram-setup
```

Paste the token when prompted, then connect it:

```text
/telegram-connect
```

From Telegram, open that bot's DM and send `/start`. The first account to do this is authorized automatically.

## Add another bot

For every additional bot, run this in any pi session:

```text
/telegram-add
```

Paste the new bot's token. The extension asks Telegram's official `getMe` API for the bot username and creates a configuration automatically:

```text
~/.pi/agent/telegram-<bot-username>.json
```

For example, two configured bots create:

```text
~/.pi/agent/telegram-codyagent1_bot.json
~/.pi/agent/telegram-codyagent2_bot.json
```

The existing legacy `telegram.json` file is renamed automatically on startup once its saved bot username is available.

## Connect two pi sessions to two bots

Open two terminals and start pi in each.

### Terminal 1

```bash
pi
```

```text
/telegram-connect

? Connect Telegram bot
❯ @codyagent1_bot (telegram-codyagent1_bot.json)
  @codyagent2_bot (telegram-codyagent2_bot.json)
```

Select `@codyagent1_bot`.

### Terminal 2

```bash
pi
```

```text
/telegram-connect

? Connect Telegram bot
  @codyagent1_bot (telegram-codyagent1_bot.json)
❯ @codyagent2_bot (telegram-codyagent2_bot.json)
```

Select `@codyagent2_bot`, then DM it in Telegram and send `/start` from the account that should control it.

Each bot must be connected to only one pi session at a time. Telegram's polling API is not designed for two sessions to poll the same bot concurrently.

Check a session's assignment with:

```text
/telegram-status

bot: @codyagent2_bot | config: telegram-codyagent2_bot.json | allowed users: 123456789 | polling: running
```

## Allow multiple Telegram accounts for one bot

Each bot config has an allowlist. Add every trusted Telegram numeric user ID to `allowedUserIds`:

```json
{
  "allowedUserIds": [123456789, 987654321]
}
```

Keep the existing `botToken`, `botId`, `botUsername`, and `lastUpdateId` fields in that config file. Existing configurations using the original single-user form remain supported and are migrated automatically:

```json
{
  "allowedUserId": 123456789
}
```

After editing the allowlist, reconnect the bot:

```text
/telegram-disconnect
/telegram-connect
```

Use numeric Telegram user IDs, not usernames. Usernames can change. You can obtain an account ID from a bot such as [@userinfobot](https://t.me/userinfobot).

## Fixed bot assignment for scripts

Normally `/telegram-connect` displays a menu when multiple configured bots exist. For a script or fixed assignment, start pi with a specific configuration:

```bash
PI_TELEGRAM_CONFIG="$HOME/.pi/agent/telegram-codyagent1_bot.json" pi
```

That pi process always uses `@codyagent1_bot` and bypasses the selection menu.

## Commands

```text
/telegram-setup       Configure the current bot
/telegram-add         Add another bot from its token
/telegram-connect     Select and connect a configured bot
/telegram-disconnect  Stop polling in the current pi session
/telegram-status      Show the assigned bot, config, and connection state
```

## License and attribution

This project is released under the MIT License. It is derived from [pi-telegram](https://github.com/badlogic/pi-telegram) by Mario Zechner and retains its license and attribution.
