---
name: notion
version: 1.0.0
description: Read, create, and update Notion pages and databases via the official API.
author: ZeptoClaw
license: MIT
tags:
  - notion
  - productivity
  - notes
env_needed:
  - name: NOTION_TOKEN
    description: Notion integration secret token (from notion.so/my-integrations)
    required: true
  - name: NOTION_DATABASE_ID
    description: Target database ID (from the page URL)
    required: false
metadata: {"zeptoclaw":{"emoji":"📝","requires":{"anyBins":["curl","jq"]}}}
---

# Notion Skill

Read and write Notion pages and databases using the official Notion API.

## Setup

1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations) → New Integration
2. Copy the **Internal Integration Secret**
3. Open your Notion page/database → Share → Invite your integration
4. Copy the database ID from the URL: `notion.so/workspace/`**`DATABASE_ID`**`?v=...`

```bash
export NOTION_TOKEN="secret_xxx..."
export NOTION_DATABASE_ID="abc123..."
```

## Query a Database

```bash
curl -s -X POST "https://api.notion.com/v1/databases/$NOTION_DATABASE_ID/query" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"page_size": 20}' | jq '.results[].properties.Name.title[0].text.content'
```

Filter by a property:
```bash
curl -s -X POST "https://api.notion.com/v1/databases/$NOTION_DATABASE_ID/query" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "property": "Status",
      "select": {"equals": "In Progress"}
    }
  }' | jq '.results | length'
```

## Create a Page (Database Row)

```bash
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d "{
    \"parent\": {\"database_id\": \"$NOTION_DATABASE_ID\"},
    \"properties\": {
      \"Name\": {\"title\": [{\"text\": {\"content\": \"New Task\"}}]},
      \"Status\": {\"select\": {\"name\": \"Todo\"}},
      \"Priority\": {\"select\": {\"name\": \"High\"}}
    }
  }" | jq '.id'
```

## Update a Page Property

```bash
PAGE_ID="page-id-from-query"
curl -s -X PATCH "https://api.notion.com/v1/pages/$PAGE_ID" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"properties": {"Status": {"select": {"name": "Done"}}}}' | jq '.id'
```

## Append Content to a Page

```bash
curl -s -X PATCH "https://api.notion.com/v1/blocks/$PAGE_ID/children" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "children": [{
      "object": "block",
      "type": "paragraph",
      "paragraph": {
        "rich_text": [{"type": "text", "text": {"content": "Agent appended this note."}}]
      }
    }]
  }' | jq '.results[0].id'
```

## Tips

- Database IDs have no hyphens in the API URL — add hyphens in the UUID format if needed
- Rate limit: 3 requests/second per integration
- Use `jq` to extract just the data you need — API responses are verbose
- Property names are case-sensitive and must match exactly
