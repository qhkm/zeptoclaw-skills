---
name: vercel
version: 1.0.0
description: Manage Vercel deployments, projects, and domains via the REST API.
author: ZeptoClaw
license: MIT
tags:
  - vercel
  - deployment
  - devops
  - hosting
env_needed:
  - name: VERCEL_TOKEN
    description: Personal token from vercel.com/account/tokens
    required: true
  - name: VERCEL_TEAM_ID
    description: Team ID (optional, for team-scoped operations)
    required: false
metadata: {"zeptoclaw":{"emoji":"▲","requires":{"anyBins":["curl","jq"]}}}
---

# Vercel Skill

Manage Vercel deployments, projects, and domains using the REST API.

## Setup

1. Go to [vercel.com/account/tokens](https://vercel.com/account/tokens) → Create Token
2. Set scope (full account or specific team)

```bash
export VERCEL_TOKEN="xxx..."
```

## List Recent Deployments

```bash
curl -s "https://api.vercel.com/v6/deployments?limit=10" \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  | jq '.deployments[] | {uid, name, state, url, created: .created}'
```

## List Projects

```bash
curl -s "https://api.vercel.com/v9/projects" \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  | jq '.projects[] | {id, name, framework, updatedAt}'
```

## Get Deployment Logs

```bash
DEPLOYMENT_ID="dpl_xxx"
curl -s "https://api.vercel.com/v2/deployments/$DEPLOYMENT_ID/events" \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  | jq '.[] | {type, created, text: .payload.text}' | head -40
```

## List Domains

```bash
curl -s "https://api.vercel.com/v5/domains" \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  | jq '.domains[] | {name, verified, expiresAt}'
```

## Tips

- Rate limit: 100 requests/minute per token
- Deployment states: `READY`, `ERROR`, `BUILDING`, `QUEUED`, `CANCELED`
- Add `?teamId=team_xxx` for team-scoped operations
- Vercel has an **official** MCP server — see MCP.md for native integration
