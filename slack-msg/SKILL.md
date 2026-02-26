---
name: slack-msg
version: 1.0.0
description: Post messages, list channels, and reply to threads via the Slack Web API.
author: ZeptoClaw
license: MIT
tags:
  - slack
  - messaging
  - communication
  - notifications
env_needed:
  - name: SLACK_BOT_TOKEN
    description: Bot token (xoxb-) from api.slack.com/apps
    required: true
metadata: {"zeptoclaw":{"emoji":"💬","requires":{"anyBins":["curl","jq"]}}}
---

# Slack Message Skill

Post messages, list channels, and reply to threads using the Slack Web API.

## Setup

1. Go to [api.slack.com/apps](https://api.slack.com/apps) → Create New App
2. Add Bot Token Scopes: `chat:write`, `channels:read`, `files:write`
3. Install app to workspace → Copy **Bot User OAuth Token**

```bash
export SLACK_BOT_TOKEN="xoxb-xxx..."
```

## Post a Message

```bash
curl -s -X POST "https://slack.com/api/chat.postMessage" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "C0XXXXXX",
    "text": "Deploy completed successfully"
  }' | jq '{ok, ts: .ts, channel: .channel}'
```

## List Channels

```bash
curl -s "https://slack.com/api/conversations.list?types=public_channel&limit=50" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  | jq '.channels[] | {id, name, num_members}'
```

## Reply to a Thread

```bash
curl -s -X POST "https://slack.com/api/chat.postMessage" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "C0XXXXXX",
    "thread_ts": "1234567890.123456",
    "text": "Thread reply from agent"
  }' | jq '{ok, ts}'
```

## Post with Blocks (Rich Formatting)

```bash
curl -s -X POST "https://slack.com/api/chat.postMessage" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "C0XXXXXX",
    "blocks": [
      {"type": "header", "text": {"type": "plain_text", "text": "Daily Report"}},
      {"type": "section", "text": {"type": "mrkdwn", "text": "*Errors:* 3\n*Uptime:* 99.9%"}}
    ]
  }' | jq '{ok, ts}'
```

## Tips

- Channel IDs start with `C` (public) or `G` (private) — use IDs not names
- Bot must be invited to a channel before posting: `/invite @botname`
- Rate limit: ~1 message/second per channel (burst: 100 per minute)
- `thread_ts` is the timestamp of the parent message
- Slack uses `mrkdwn` (not markdown) — bold is `*text*`, italic is `_text_`
