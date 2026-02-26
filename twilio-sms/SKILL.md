---
name: twilio-sms
version: 1.0.0
description: Send and manage SMS messages via the Twilio API.
author: ZeptoClaw
license: MIT
tags:
  - twilio
  - sms
  - messaging
  - communication
env_needed:
  - name: TWILIO_ACCOUNT_SID
    description: Account SID from twilio.com/console
    required: true
  - name: TWILIO_AUTH_TOKEN
    description: Auth Token from twilio.com/console
    required: true
  - name: TWILIO_FROM_NUMBER
    description: "Your Twilio phone number (E.164 format: +1234567890)"
    required: true
metadata: {"zeptoclaw":{"emoji":"📱","requires":{"anyBins":["curl","jq"]}}}
---

# Twilio SMS Skill

Send, check, and list SMS messages using the Twilio API.

## Setup

1. Go to [twilio.com/console](https://www.twilio.com/console)
2. Copy **Account SID** and **Auth Token**
3. Buy or use a trial phone number

```bash
export TWILIO_ACCOUNT_SID="ACxxx..."
export TWILIO_AUTH_TOKEN="xxx..."
export TWILIO_FROM_NUMBER="+1234567890"
```

## Send an SMS

```bash
curl -s -X POST "https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Messages.json" \
  -u "$TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN" \
  -d "From=$TWILIO_FROM_NUMBER" \
  -d "To=+60123456789" \
  -d "Body=Hello from ZeptoClaw!" \
  | jq '{sid, status, to, body}'
```

## Check Message Status

```bash
MSG_SID="SMxxx..."
curl -s "https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Messages/$MSG_SID.json" \
  -u "$TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN" \
  | jq '{sid, status, to, dateSent: .date_sent, errorCode: .error_code}'
```

## List Recent Messages

```bash
curl -s "https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Messages.json?PageSize=10" \
  -u "$TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN" \
  | jq '.messages[] | {sid, from, to, body, status, date_sent}'
```

## Tips

- Phone numbers must be E.164 format: `+[country code][number]`
- Malaysia country code: `+60` (e.g. `+60123456789`)
- Trial accounts can only send to verified numbers
- Status values: `queued`, `sent`, `delivered`, `failed`, `undelivered`
- Rate limit: 1 message/second per number (higher with toll-free/short codes)
- Uses Basic Auth (`-u SID:token`) not Bearer tokens
