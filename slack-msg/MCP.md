# Slack — MCP Server

## Config

Add to `~/.zeptoclaw/config.json` under `mcp_servers`:

```json
{
  "name": "slack",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-slack"],
  "env": {
    "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `post_message` | Post a message to a channel |
| `list_channels` | List workspace channels |
| `reply_to_thread` | Reply in a thread |
| `get_channel_history` | Read recent messages |
| `search_messages` | Search across workspace |

## Notes

- Official MCP server from Anthropic's registry (`@modelcontextprotocol/server-slack`)
- Also available: `@ubie-oss/slack-mcp-server` (community alternative)
- Bot token needs scopes: `chat:write`, `channels:read`, `channels:history`
