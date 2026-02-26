---
name: todoist
version: 1.0.0
description: Manage Todoist tasks and projects via the REST API v2.
author: ZeptoClaw
license: MIT
tags:
  - todoist
  - tasks
  - productivity
  - gtd
env_needed:
  - name: TODOIST_API_TOKEN
    description: API token from todoist.com/app/settings/integrations/developer
    required: true
metadata: {"zeptoclaw":{"emoji":"✅","requires":{"anyBins":["curl","jq"]}}}
---

# Todoist Skill

Create, list, and complete tasks in Todoist using the REST API v2.

## Setup

1. Go to [todoist.com/app/settings/integrations/developer](https://todoist.com/app/settings/integrations/developer)
2. Copy your **API token**

```bash
export TODOIST_API_TOKEN="your_token_here"
```

## List Active Tasks

```bash
curl -s "https://api.todoist.com/rest/v2/tasks" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  | jq '.[] | {id, content, due: .due.date, priority, project_id}'
```

Filter by project:

```bash
curl -s "https://api.todoist.com/rest/v2/tasks?project_id=PROJECT_ID" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  | jq '.[] | {id, content, due: .due.date}'
```

## Create a Task

```bash
curl -s -X POST "https://api.todoist.com/rest/v2/tasks" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Review PR #42",
    "due_string": "tomorrow at 10am",
    "priority": 3,
    "project_id": "PROJECT_ID"
  }' | jq '{id, content, due: .due.date}'
```

## Complete a Task

```bash
TASK_ID="task-id-from-list"
curl -s -X POST "https://api.todoist.com/rest/v2/tasks/$TASK_ID/close" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  -w "\nHTTP Status: %{http_code}\n"
```

## List Projects

```bash
curl -s "https://api.todoist.com/rest/v2/projects" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  | jq '.[] | {id, name, color}'
```

## Tips

- `due_string` accepts natural language: "tomorrow", "every monday", "Jan 5 at 3pm"
- Priority: 1 (normal) to 4 (urgent) — note: Todoist UI shows these inverted
- Rate limit: 450 requests per 15 minutes
- Use `X-Request-Id` header for idempotent writes
