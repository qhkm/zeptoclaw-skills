---
name: discord-webhook
version: 1.0.0
description: Send messages and rich embeds to Discord channels via webhooks.
author: ZeptoClaw
license: MIT
tags:
  - discord
  - messaging
  - notifications
  - webhooks
env_needed:
  - name: DISCORD_WEBHOOK_URL
    description: "Webhook URL from Discord channel settings (Server Settings > Integrations > Webhooks)"
    required: true
metadata: {"zeptoclaw":{"emoji":"🎮","requires":{"anyBins":["curl"]}}}
---

# Discord Webhook Skill

Send messages and rich embeds to Discord channels via webhooks. No bot setup needed.

## Setup

1. In Discord: Server Settings → Integrations → Webhooks → New Webhook
2. Choose the target channel
3. Copy Webhook URL

```bash
export DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/123456/abcdef..."
```

## Send a Simple Message

```bash
curl -s -X POST "$DISCORD_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"content": "Deploy completed"}' \
  -w "\nHTTP Status: %{http_code}\n"
```

## Send a Rich Embed

```bash
curl -s -X POST "$DISCORD_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "embeds": [{
      "title": "Build Report",
      "color": 3066993,
      "fields": [
        {"name": "Status", "value": "Success", "inline": true},
        {"name": "Duration", "value": "2m 34s", "inline": true},
        {"name": "Branch", "value": "main", "inline": true}
      ],
      "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
    }]
  }' -w "\nHTTP Status: %{http_code}\n"
```

## Send with Custom Username/Avatar

```bash
curl -s -X POST "$DISCORD_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "ZeptoClaw Bot",
    "avatar_url": "https://github.com/qhkm/zeptoclaw/raw/main/landing/zeptoclaw/mascot-no-bg.png",
    "content": "Hello from ZeptoClaw!"
  }' -w "\nHTTP Status: %{http_code}\n"
```

## Tips

- No auth token needed — the webhook URL itself is the credential (keep it secret!)
- Color is decimal: green=3066993, red=15158332, blue=3447003, yellow=16776960
- Rate limit: 5 requests per 2 seconds per webhook
- Max 10 embeds per message, 2000 chars per content field
- Returns 204 on success (no response body)
