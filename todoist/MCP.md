# Todoist — MCP Server

## Config

Add to `~/.zeptoclaw/config.json` under `mcp_servers`:

```json
{
  "name": "todoist",
  "command": "npx",
  "args": ["-y", "@anthropic/mcp-todoist"],
  "env": {
    "TODOIST_API_TOKEN": "${TODOIST_API_TOKEN}"
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `list_tasks` | List active tasks with optional filters |
| `create_task` | Create a new task |
| `complete_task` | Mark a task as complete |
| `list_projects` | List all projects |

## Notes

- Community MCP server — verify package name at [MCP Registry](https://registry.modelcontextprotocol.io/)
- Requires Todoist API token (free tier available)
