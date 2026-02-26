# Vercel — MCP Server (Official)

## Install

```bash
npm install -g @vercel/mcp-server
```

## Config

Add to `~/.zeptoclaw/config.json` under `mcp_servers`:

```json
{
  "name": "vercel",
  "command": "npx",
  "args": ["-y", "@vercel/mcp-server"],
  "env": {
    "VERCEL_TOKEN": "${VERCEL_TOKEN}"
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `list_deployments` | List recent deployments |
| `get_deployment` | Get deployment details |
| `list_projects` | List all projects |
| `list_domains` | List configured domains |
| `get_deployment_logs` | Get build/runtime logs |

## Notes

- **Official** MCP server maintained by Vercel
- Check [npmjs.com/package/@vercel/mcp-server](https://www.npmjs.com/package/@vercel/mcp-server) for latest version
