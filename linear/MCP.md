# Linear — MCP Server

## Config

Add to `~/.zeptoclaw/config.json` under `mcp_servers`:

```json
{
  "name": "linear",
  "command": "npx",
  "args": ["-y", "@linear/mcp-server"],
  "env": {
    "LINEAR_API_KEY": "${LINEAR_API_KEY}"
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `list_issues` | List issues with filters |
| `create_issue` | Create a new issue |
| `update_issue` | Update issue fields |
| `list_teams` | List all teams |
| `search_issues` | Search issues by text |

## Notes

- Community MCP server — verify package name at [MCP Registry](https://registry.modelcontextprotocol.io/)
- Linear API is GraphQL-only; MCP server abstracts this into simple tool calls
