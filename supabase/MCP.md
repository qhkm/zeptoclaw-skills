# Supabase — MCP Server

## Config

Add to `~/.zeptoclaw/config.json` under `mcp_servers`:

```json
{
  "name": "supabase",
  "command": "npx",
  "args": ["-y", "supabase-mcp"],
  "env": {
    "SUPABASE_URL": "${SUPABASE_URL}",
    "SUPABASE_SERVICE_KEY": "${SUPABASE_SERVICE_KEY}"
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `query` | Run SQL or PostgREST queries |
| `insert` | Insert rows |
| `update` | Update rows |
| `delete` | Delete rows |
| `list_tables` | List all tables |
| `list_users` | List auth users |

## Notes

- Community MCP: [github.com/supabase-community/supabase-mcp](https://github.com/supabase-community/supabase-mcp)
- Uses service role key — has full admin access (bypasses RLS)
