---
name: cloudflare-ops
version: 1.0.0
description: Manage Cloudflare DNS records, Workers, KV, and R2 via the API v4.
author: ZeptoClaw
license: MIT
tags:
  - cloudflare
  - dns
  - workers
  - devops
env_needed:
  - name: CF_API_TOKEN
    description: API token from dash.cloudflare.com/profile/api-tokens
    required: true
  - name: CF_ZONE_ID
    description: Zone ID from the domain overview page
    required: false
  - name: CF_ACCOUNT_ID
    description: Account ID from the domain overview page
    required: false
metadata: {"zeptoclaw":{"emoji":"☁️","requires":{"anyBins":["curl","jq"]}}}
---

# Cloudflare Ops Skill

Manage DNS records, Workers, KV namespaces, and R2 buckets using the Cloudflare API v4.

## Setup

1. Go to [dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens) → Create Token
2. Use the "Edit zone DNS" template for DNS, or create custom for Workers/KV/R2
3. Get Zone ID and Account ID from any domain's overview page (right sidebar)

```bash
export CF_API_TOKEN="xxx..."
export CF_ZONE_ID="zone-id-from-dashboard"
export CF_ACCOUNT_ID="account-id-from-dashboard"
```

## List DNS Records

```bash
curl -s "https://api.cloudflare.com/client/v4/zones/$CF_ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  | jq '.result[] | {id, name, type, content, ttl}'
```

## Create a DNS Record

```bash
curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$CF_ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "A",
    "name": "app.example.com",
    "content": "1.2.3.4",
    "ttl": 3600,
    "proxied": true
  }' | jq '.result | {id, name, type, content}'
```

## Delete a DNS Record

```bash
RECORD_ID="record-id-from-list"
curl -s -X DELETE "https://api.cloudflare.com/client/v4/zones/$CF_ZONE_ID/dns_records/$RECORD_ID" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  | jq '.success'
```

## List Workers

```bash
curl -s "https://api.cloudflare.com/client/v4/accounts/$CF_ACCOUNT_ID/workers/scripts" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  | jq '.result[] | {id, modified_on}'
```

## KV: Read/Write

```bash
# List namespaces
curl -s "https://api.cloudflare.com/client/v4/accounts/$CF_ACCOUNT_ID/storage/kv/namespaces" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  | jq '.result[] | {id, title}'

# Write a key
KV_NS_ID="namespace-id"
curl -s -X PUT "https://api.cloudflare.com/client/v4/accounts/$CF_ACCOUNT_ID/storage/kv/namespaces/$KV_NS_ID/values/mykey" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: text/plain" \
  -d "my value"

# Read a key
curl -s "https://api.cloudflare.com/client/v4/accounts/$CF_ACCOUNT_ID/storage/kv/namespaces/$KV_NS_ID/values/mykey" \
  -H "Authorization: Bearer $CF_API_TOKEN"
```

## Tips

- `proxied: true` routes traffic through Cloudflare (orange cloud)
- Rate limit: 1,200 requests per 5 minutes
- API tokens are more secure than global API keys — always prefer tokens
- Zone ID and Account ID are on the domain overview page (right sidebar)
