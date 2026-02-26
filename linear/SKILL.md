---
name: linear
version: 1.0.0
description: Manage Linear issues, projects, and teams via the GraphQL API.
author: ZeptoClaw
license: MIT
tags:
  - linear
  - issues
  - project-management
  - productivity
env_needed:
  - name: LINEAR_API_KEY
    description: API key from linear.app/settings/api
    required: true
metadata: {"zeptoclaw":{"emoji":"🔷","requires":{"anyBins":["curl","jq"]}}}
---

# Linear Skill

Create and manage Linear issues using the GraphQL API. Linear has no REST API — all operations use a single GraphQL endpoint.

## Setup

1. Go to [linear.app/settings/api](https://linear.app/settings/api) → Personal API Keys
2. Create a new key and copy it

```bash
export LINEAR_API_KEY="lin_api_xxx..."
```

## List My Issues

```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ viewer { assignedIssues(first: 20) { nodes { id identifier title state { name } priority } } } }"}' \
  | jq '.data.viewer.assignedIssues.nodes[] | {identifier, title, state: .state.name, priority}'
```

## Create an Issue

```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "mutation($input: IssueCreateInput!) { issueCreate(input: $input) { success issue { identifier title url } } }",
    "variables": {
      "input": {
        "teamId": "TEAM_UUID_HERE",
        "title": "Fix login timeout",
        "description": "Users report 504 errors on login",
        "priority": 2
      }
    }
  }' | jq '.data.issueCreate.issue | {identifier, title, url}'
```

## Update Issue Status

```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "mutation($id: String!, $input: IssueUpdateInput!) { issueUpdate(id: $id, input: $input) { success issue { identifier title state { name } } } }",
    "variables": {
      "id": "ISSUE_UUID_HERE",
      "input": {"stateId": "STATE_UUID_HERE"}
    }
  }' | jq '.data.issueUpdate.issue | {identifier, title, state: .state.name}'
```

## List Teams

```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ teams { nodes { id name key } } }"}' \
  | jq '.data.teams.nodes[] | {id, name, key}'
```

## Tips

- Linear is **GraphQL-only** — no REST endpoints exist
- UUIDs internally — `identifier` (e.g. "ENG-123") is the human-friendly key
- Priority: 0 (none), 1 (urgent), 2 (high), 3 (medium), 4 (low)
- Rate limit: 1,500 requests per hour per user
- Auth header has no `Bearer` prefix — just the API key directly
- Get team/state IDs first before creating or updating issues
