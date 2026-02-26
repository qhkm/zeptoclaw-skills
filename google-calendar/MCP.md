# Google Calendar — MCP Server

## Install

```bash
npx @anthropic/mcp-google-calendar
```

## Config

Add to `~/.zeptoclaw/config.json` under `mcp_servers`:

```json
{
  "name": "google-calendar",
  "command": "npx",
  "args": ["-y", "@anthropic/mcp-google-calendar"],
  "env": {
    "GOOGLE_CLIENT_ID": "${GOOGLE_CLIENT_ID}",
    "GOOGLE_CLIENT_SECRET": "${GOOGLE_CLIENT_SECRET}",
    "GOOGLE_REFRESH_TOKEN": "${GOOGLE_REFRESH_TOKEN}"
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `list_events` | List upcoming calendar events |
| `create_event` | Create a new event |
| `update_event` | Update an existing event |
| `delete_event` | Delete an event |
| `get_event` | Get event details |
| `list_calendars` | List all calendars |

## Notes

- Community MCP server — verify package name at [MCP Registry](https://registry.modelcontextprotocol.io/)
- Requires Google OAuth2 credentials (client ID, secret, refresh token)
- Server handles token refresh automatically
