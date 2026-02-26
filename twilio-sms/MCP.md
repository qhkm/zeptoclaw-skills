# Twilio — MCP Server (Official)

## Config

Add to `~/.zeptoclaw/config.json` under `mcp_servers`:

```json
{
  "name": "twilio",
  "command": "npx",
  "args": ["-y", "@twilio-alpha/mcp"],
  "env": {
    "TWILIO_ACCOUNT_SID": "${TWILIO_ACCOUNT_SID}",
    "TWILIO_AUTH_TOKEN": "${TWILIO_AUTH_TOKEN}"
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `send_message` | Send SMS/MMS |
| `get_message` | Get message details |
| `list_messages` | List recent messages |

## Notes

- Official MCP server from Twilio (`@twilio-alpha/mcp`)
- Also community: `@deshartman/twilio-messaging-mcp-server`
- Alpha release — API may change
