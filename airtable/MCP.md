# Airtable — MCP Server

## Config

Add to `~/.zeptoclaw/config.json` under `mcp_servers`:

```json
{
  "name": "airtable",
  "command": "npx",
  "args": ["-y", "@anthropic/mcp-airtable"],
  "env": {
    "AIRTABLE_TOKEN": "${AIRTABLE_TOKEN}",
    "AIRTABLE_BASE_ID": "${AIRTABLE_BASE_ID}"
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `list_records` | List records with optional filter |
| `create_record` | Create a new record |
| `update_record` | Update record fields |
| `delete_record` | Delete a record |

## Notes

- Community MCP server — verify package name at [MCP Registry](https://registry.modelcontextprotocol.io/)
- Composio also offers Airtable MCP — check both options
