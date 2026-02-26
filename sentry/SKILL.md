---
name: sentry
version: 1.0.0
description: Monitor and resolve Sentry issues, view project stats, and manage error tracking.
author: ZeptoClaw
license: MIT
tags:
  - sentry
  - monitoring
  - errors
  - devops
env_needed:
  - name: SENTRY_AUTH_TOKEN
    description: Auth token from sentry.io/settings/auth-tokens/
    required: true
  - name: SENTRY_ORG
    description: Organization slug
    required: true
  - name: SENTRY_PROJECT
    description: Project slug
    required: false
metadata: {"zeptoclaw":{"emoji":"🐛","requires":{"anyBins":["curl","jq"]}}}
---

# Sentry Skill

Monitor errors, resolve issues, and check project health using the Sentry API.

## Setup

1. Go to [sentry.io/settings/auth-tokens/](https://sentry.io/settings/auth-tokens/) → Create Token
2. Select scopes: `project:read`, `event:read`, `org:read`
3. Note your organization slug from URL: `sentry.io/organizations/`**`your-org`**

```bash
export SENTRY_AUTH_TOKEN="sntrys_xxx..."
export SENTRY_ORG="your-org"
export SENTRY_PROJECT="your-project"
```

## List Unresolved Issues

```bash
curl -s "https://sentry.io/api/0/projects/$SENTRY_ORG/$SENTRY_PROJECT/issues/?query=is:unresolved" \
  -H "Authorization: Bearer $SENTRY_AUTH_TOKEN" \
  | jq '.[] | {id, title, culprit, count, firstSeen, lastSeen}' | head -40
```

## Get Issue Details

```bash
ISSUE_ID="issue-id-from-list"
curl -s "https://sentry.io/api/0/issues/$ISSUE_ID/" \
  -H "Authorization: Bearer $SENTRY_AUTH_TOKEN" \
  | jq '{id, title, status, count, userCount, firstSeen, lastSeen}'
```

## Resolve an Issue

```bash
ISSUE_ID="issue-id-from-list"
curl -s -X PUT "https://sentry.io/api/0/issues/$ISSUE_ID/" \
  -H "Authorization: Bearer $SENTRY_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"status": "resolved"}' | jq '{id, status}'
```

## Project Stats (24h error count)

```bash
curl -s "https://sentry.io/api/0/projects/$SENTRY_ORG/$SENTRY_PROJECT/stats/?stat=received&resolution=1h" \
  -H "Authorization: Bearer $SENTRY_AUTH_TOKEN" \
  | jq 'map(.[1]) | add'
```

## Tips

- Rate limit: 40 requests/second for organization tokens
- Use `?query=is:unresolved+level:error` to filter by level
- Issue IDs are numeric strings
- `culprit` field shows the function/method where the error occurred
- Pagination: use `Link` header for next page URL
