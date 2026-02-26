---
name: google-calendar
version: 1.0.0
description: List, create, update, and delete Google Calendar events via the Calendar API v3.
author: ZeptoClaw
license: MIT
tags:
  - google
  - calendar
  - productivity
  - scheduling
env_needed:
  - name: GOOGLE_ACCESS_TOKEN
    description: OAuth2 access token (use `gcloud auth print-access-token` or OAuth flow)
    required: true
  - name: CALENDAR_ID
    description: Calendar ID (default "primary" for your main calendar)
    required: false
metadata: {"zeptoclaw":{"emoji":"📅","requires":{"anyBins":["curl","jq"]}}}
---

# Google Calendar Skill

Manage Google Calendar events — list upcoming, create, update, and delete events using the Calendar API v3.

## Setup

1. Create a project at [console.cloud.google.com](https://console.cloud.google.com)
2. Enable the **Google Calendar API**
3. Create OAuth 2.0 credentials or use a service account
4. Get an access token:

```bash
# If you have gcloud CLI installed:
export GOOGLE_ACCESS_TOKEN=$(gcloud auth print-access-token)

# Or set manually from OAuth flow:
export GOOGLE_ACCESS_TOKEN="ya29.xxx..."
export CALENDAR_ID="primary"
```

## List Upcoming Events

```bash
curl -s "https://www.googleapis.com/calendar/v3/calendars/${CALENDAR_ID:-primary}/events?maxResults=10&orderBy=startTime&singleEvents=true&timeMin=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  -H "Authorization: Bearer $GOOGLE_ACCESS_TOKEN" \
  | jq '.items[] | {summary, start: .start.dateTime, end: .end.dateTime, id}'
```

## Create an Event

```bash
curl -s -X POST "https://www.googleapis.com/calendar/v3/calendars/${CALENDAR_ID:-primary}/events" \
  -H "Authorization: Bearer $GOOGLE_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "summary": "Team Standup",
    "start": {"dateTime": "2026-03-01T09:00:00+08:00"},
    "end": {"dateTime": "2026-03-01T09:30:00+08:00"},
    "description": "Daily standup meeting",
    "location": "Google Meet"
  }' | jq '{id, summary, htmlLink}'
```

## Update an Event

```bash
EVENT_ID="event-id-from-list"
curl -s -X PATCH "https://www.googleapis.com/calendar/v3/calendars/${CALENDAR_ID:-primary}/events/$EVENT_ID" \
  -H "Authorization: Bearer $GOOGLE_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"summary": "Updated Meeting Title"}' | jq '{id, summary, updated}'
```

## Delete an Event

```bash
EVENT_ID="event-id-from-list"
curl -s -X DELETE "https://www.googleapis.com/calendar/v3/calendars/${CALENDAR_ID:-primary}/events/$EVENT_ID" \
  -H "Authorization: Bearer $GOOGLE_ACCESS_TOKEN" \
  -w "\nHTTP Status: %{http_code}\n"
```

## Tips

- `CALENDAR_ID` defaults to `primary` if not set
- Use `singleEvents=true` to expand recurring events into individual instances
- Token expires every ~60 minutes — refresh with `gcloud auth print-access-token`
- Rate limit: 1,000,000 queries/day (generous)
- Time format must be RFC 3339 with timezone offset
- All-day events use `date` field instead of `dateTime`
