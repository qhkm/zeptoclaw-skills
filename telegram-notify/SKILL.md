---
name: telegram-notify
version: 1.0.0
description: Send Telegram messages, alerts, and files to users or groups via Bot API.
author: ZeptoClaw
license: MIT
tags:
  - telegram
  - notifications
  - alerts
env_needed:
  - name: TELEGRAM_BOT_TOKEN
    description: Bot token from @BotFather
    required: true
  - name: TELEGRAM_CHAT_ID
    description: Target chat/group/channel ID (use @channelusername or numeric ID)
    required: true
metadata: {"zeptoclaw":{"emoji":"✈️","requires":{"anyBins":["curl","jq"]}}}
---

# Telegram Notify Skill

Send messages, alerts, files, and formatted reports to any Telegram chat using your bot.

## Setup

1. Message [@BotFather](https://t.me/BotFather) → `/newbot` → copy the token
2. Add the bot to your target group/channel and make it an admin
3. Get the chat ID: send a message, then `curl https://api.telegram.org/bot$TOKEN/getUpdates | jq '.result[0].message.chat.id'`

```bash
export TELEGRAM_BOT_TOKEN="7123456789:AAFxxx..."
export TELEGRAM_CHAT_ID="-1001234567890"
```

## Send a Text Alert

```bash
curl -s "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
  -d chat_id="$TELEGRAM_CHAT_ID" \
  -d text="🚨 Server CPU > 90% on prod-1" \
  -d parse_mode="HTML" | jq -r '.ok'
```

## Send Formatted HTML Message

```bash
curl -s "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
  -d chat_id="$TELEGRAM_CHAT_ID" \
  -d parse_mode="HTML" \
  -d text="<b>Daily Report</b>
Orders: <code>142</code>
Revenue: <code>RM 4,210</code>
Status: ✅ All systems normal" | jq -r '.ok'
```

## Send a File

```bash
curl -s "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendDocument" \
  -F chat_id="$TELEGRAM_CHAT_ID" \
  -F document=@"/path/to/report.pdf" \
  -F caption="Weekly sales report" | jq -r '.ok'
```

## Pin a Message

```bash
MSG_ID=$(curl -s "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
  -d chat_id="$TELEGRAM_CHAT_ID" \
  -d text="⚠️ Maintenance window tonight 2-4AM" | jq -r '.result.message_id')

curl -s "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/pinChatMessage" \
  -d chat_id="$TELEGRAM_CHAT_ID" \
  -d message_id="$MSG_ID" | jq -r '.ok'
```

## Tips

- HTML tags allowed: `<b>`, `<i>`, `<code>`, `<pre>`, `<a href="...">` — escape `<>&` in content
- Max message length: 4096 characters; split long messages
- For channels: use `@channelusername` or numeric ID starting with `-100`
- Silence notifications: add `-d disable_notification=true`
