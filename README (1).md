# 📧 Email to Telegram — n8n Workflow

An [n8n](https://n8n.io) workflow that watches an email inbox over IMAP and instantly forwards every new unread email to a Telegram chat (sender name, sender address, subject, and a 300-character preview of the body).

## How it works

```
Email Trigger (IMAP)  ──►  Telegram
```

1. **Email Trigger (IMAP)** polls the mailbox for `UNSEEN` emails received `SINCE` a given date.
2. **Telegram** sends a formatted message to your chat:

```
📧 Email jdid
Nom: John Doe
Email: john@example.com
Sujet: Meeting tomorrow

Hi, just a reminder about our meeting...
```

## Requirements

- An n8n instance (self-hosted or cloud), v1.x or later
- An email account with IMAP enabled
- A Telegram bot (created via [@BotFather](https://t.me/BotFather))

## Setup

### 1. Import the workflow
In n8n: **Workflows → Import from File** → select `email-to-telegram.json`.

### 2. Create the credentials

**IMAP account**
| Field | Example |
|-------|---------|
| User | `you@example.com` |
| Password | App password (not your main password for Gmail/Outlook) |
| Host | `imap.gmail.com` |
| Port | `993` |
| SSL/TLS | enabled |

> Gmail: enable 2FA, then create an **App Password**.

**Telegram API**
1. Talk to [@BotFather](https://t.me/BotFather), send `/newbot`, and copy the token.
2. In n8n, create a *Telegram API* credential with that token.

### 3. Set your chat ID
Open the **Telegram** node and replace `YOUR_TELEGRAM_CHAT_ID` with your chat ID.
To find it, send a message to your bot, then open:
`https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates` and read `message.chat.id`
(or use [@userinfobot](https://t.me/userinfobot)).

> Important: you must send `/start` to your bot at least once, otherwise it cannot message you.

### 4. Adjust the date filter
In the **Email Trigger** node, `customEmailConfig` is:

```json
["UNSEEN", ["SINCE", "Nov 17, 2026"]]
```

Change the date to the day you want to start receiving emails from. This prevents old unread emails from flooding your Telegram on the first run.

### 5. Activate
Select your credentials in both nodes, test with **Execute Workflow**, then toggle **Active**.

## Configuration options

| Option | Where | Description |
|--------|-------|-------------|
| `postProcessAction` | IMAP node | `nothing` leaves emails untouched; use `read` to mark them as read |
| `forceReconnect` | IMAP node | Reconnect interval in minutes |
| Preview length | Telegram node | `.slice(0, 300)` — change `300` to show more or less |
| `parse_mode` | Telegram node | `HTML` — see the note below |

## Known issues & tips

- **`parse_mode: HTML`**: if an email body contains characters like `<` or `>`, Telegram may reject the message with a "can't parse entities" error. Remove `parse_mode` from the node (plain text) to avoid this.
- **Duplicates**: with `postProcessAction: nothing`, an email stays unread. Depending on your server, it may be re-sent after a reconnect. Set it to `read` to avoid that.
- **Hardcoded date**: update the `SINCE` date when needed, or replace it with a dynamic expression.

## Security

Never commit real credentials, bot tokens, chat IDs or your n8n `instanceId`. This repo ships with placeholders only — replace them inside n8n, not in the JSON.

## License

MIT
