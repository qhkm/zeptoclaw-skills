---
name: supabase
version: 1.0.0
description: Query tables, insert rows, and manage auth users via Supabase PostgREST API.
author: ZeptoClaw
license: MIT
tags:
  - supabase
  - database
  - postgres
  - backend
env_needed:
  - name: SUPABASE_URL
    description: "Project URL (https://xxx.supabase.co)"
    required: true
  - name: SUPABASE_ANON_KEY
    description: Anon/public key from Project Settings > API
    required: true
  - name: SUPABASE_SERVICE_KEY
    description: Service role key (for admin operations, bypasses RLS)
    required: false
metadata: {"zeptoclaw":{"emoji":"⚡","requires":{"anyBins":["curl","jq"]}}}
---

# Supabase Skill

Query and modify data in Supabase using the PostgREST API. Also manage auth users.

## Setup

1. Go to your project at [supabase.com/dashboard](https://supabase.com/dashboard)
2. Project Settings → API → Copy the URL and keys

```bash
export SUPABASE_URL="https://xxx.supabase.co"
export SUPABASE_ANON_KEY="eyJhbGci..."
export SUPABASE_SERVICE_KEY="eyJhbGci..."  # optional, for admin ops
```

## Query a Table

```bash
curl -s "$SUPABASE_URL/rest/v1/todos?select=*&order=created_at.desc&limit=20" \
  -H "apikey: $SUPABASE_ANON_KEY" \
  -H "Authorization: Bearer $SUPABASE_ANON_KEY" \
  | jq '.[] | {id, title, done, created_at}'
```

Filter with operators:

```bash
curl -s "$SUPABASE_URL/rest/v1/todos?done=eq.false&select=id,title" \
  -H "apikey: $SUPABASE_ANON_KEY" \
  -H "Authorization: Bearer $SUPABASE_ANON_KEY" \
  | jq .
```

## Insert a Row

```bash
curl -s -X POST "$SUPABASE_URL/rest/v1/todos" \
  -H "apikey: $SUPABASE_ANON_KEY" \
  -H "Authorization: Bearer $SUPABASE_ANON_KEY" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=representation" \
  -d '{"title": "New task from agent", "done": false}' \
  | jq '.[0] | {id, title}'
```

## Update a Row

```bash
curl -s -X PATCH "$SUPABASE_URL/rest/v1/todos?id=eq.1" \
  -H "apikey: $SUPABASE_ANON_KEY" \
  -H "Authorization: Bearer $SUPABASE_ANON_KEY" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=representation" \
  -d '{"done": true}' | jq .
```

## List Auth Users (requires service key)

```bash
curl -s "$SUPABASE_URL/auth/v1/admin/users" \
  -H "apikey: $SUPABASE_SERVICE_KEY" \
  -H "Authorization: Bearer $SUPABASE_SERVICE_KEY" \
  | jq '.users[] | {id, email, created_at, last_sign_in_at}'
```

## Tips

- Supabase requires BOTH `apikey` header AND `Authorization: Bearer` header
- Use `Prefer: return=representation` to get the inserted/updated row back
- PostgREST operators: `eq`, `neq`, `gt`, `lt`, `gte`, `lte`, `like`, `ilike`, `in`
- Service key bypasses Row Level Security — use only for admin operations
- Rate limit depends on your plan (free: 500 req/min)
