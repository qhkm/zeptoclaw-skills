---
name: airtable
version: 1.0.0
description: Read, create, update, and delete Airtable records via the REST API.
author: ZeptoClaw
license: MIT
tags:
  - airtable
  - database
  - spreadsheet
  - productivity
env_needed:
  - name: AIRTABLE_TOKEN
    description: Personal access token from airtable.com/create/tokens
    required: true
  - name: AIRTABLE_BASE_ID
    description: Base ID (starts with "app", from URL or API docs)
    required: true
  - name: AIRTABLE_TABLE_NAME
    description: Table name or ID
    required: true
metadata: {"zeptoclaw":{"emoji":"📊","requires":{"anyBins":["curl","jq"]}}}
---

# Airtable Skill

CRUD operations on Airtable bases using the REST API.

## Setup

1. Go to [airtable.com/create/tokens](https://airtable.com/create/tokens) → Create Token
2. Grant `data.records:read` and `data.records:write` scopes
3. Get your Base ID from the URL: `airtable.com/`**`appXXXXXX`**`/tblYYYYYY`

```bash
export AIRTABLE_TOKEN="patXXX.xxx..."
export AIRTABLE_BASE_ID="appXXXXXX"
export AIRTABLE_TABLE_NAME="Tasks"
```

## List Records

```bash
curl -s "https://api.airtable.com/v0/$AIRTABLE_BASE_ID/$AIRTABLE_TABLE_NAME?maxRecords=20" \
  -H "Authorization: Bearer $AIRTABLE_TOKEN" \
  | jq '.records[] | {id, fields: .fields}'
```

Filter with formula:

```bash
curl -s "https://api.airtable.com/v0/$AIRTABLE_BASE_ID/$AIRTABLE_TABLE_NAME" \
  -H "Authorization: Bearer $AIRTABLE_TOKEN" \
  --data-urlencode "filterByFormula={Status}='Active'" \
  | jq '.records[] | {id, name: .fields.Name, status: .fields.Status}'
```

## Create a Record

```bash
curl -s -X POST "https://api.airtable.com/v0/$AIRTABLE_BASE_ID/$AIRTABLE_TABLE_NAME" \
  -H "Authorization: Bearer $AIRTABLE_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "records": [{
      "fields": {
        "Name": "New Item",
        "Status": "Todo",
        "Priority": "High"
      }
    }]
  }' | jq '.records[0] | {id, fields: .fields}'
```

## Update a Record

```bash
RECORD_ID="recXXXXXX"
curl -s -X PATCH "https://api.airtable.com/v0/$AIRTABLE_BASE_ID/$AIRTABLE_TABLE_NAME" \
  -H "Authorization: Bearer $AIRTABLE_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"records\": [{
      \"id\": \"$RECORD_ID\",
      \"fields\": {\"Status\": \"Done\"}
    }]
  }" | jq '.records[0].fields'
```

## Delete a Record

```bash
RECORD_ID="recXXXXXX"
curl -s -X DELETE "https://api.airtable.com/v0/$AIRTABLE_BASE_ID/$AIRTABLE_TABLE_NAME?records[]=$RECORD_ID" \
  -H "Authorization: Bearer $AIRTABLE_TOKEN" \
  | jq '.records[0]'
```

## Tips

- Field names are case-sensitive and must match your table exactly
- Batch operations: up to 10 records per create/update/delete request
- Rate limit: 5 requests per second per base
- Use `offset` from response for pagination (Airtable returns 100 records max)
- Formula syntax: `{FieldName}='value'` (curly braces around field names)
