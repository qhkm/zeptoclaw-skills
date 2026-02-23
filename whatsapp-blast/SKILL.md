---
name: whatsapp-blast
version: 1.0.0
description: Send bulk WhatsApp messages via WhatsApp Cloud API — orders, alerts, promotions.
author: ZeptoClaw
license: MIT
tags:
  - whatsapp
  - messaging
  - sea
  - ecommerce
env_needed:
  - name: WHATSAPP_TOKEN
    description: WhatsApp Cloud API permanent access token
    required: true
  - name: WHATSAPP_PHONE_ID
    description: Phone number ID from Meta Business (not the phone number itself)
    required: true
metadata: {"zeptoclaw":{"emoji":"💬","requires":{"anyBins":["curl","jq"]}}}
---

# WhatsApp Blast Skill

Send WhatsApp messages to customers via the official Meta WhatsApp Cloud API. No third-party bridge required.

## Setup

1. Create a Meta Business app at [developers.facebook.com](https://developers.facebook.com)
2. Add WhatsApp product → get a test/permanent phone number ID
3. Generate a permanent system user token (not the temporary one)

```bash
export WHATSAPP_TOKEN="EAAxxxxx..."
export WHATSAPP_PHONE_ID="123456789"
```

## Send a Text Message

```bash
curl -s -X POST "https://graph.facebook.com/v19.0/$WHATSAPP_PHONE_ID/messages" \
  -H "Authorization: Bearer $WHATSAPP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "to": "60123456789",
    "type": "text",
    "text": {"body": "Your order #1234 has been shipped! 🚚"}
  }' | jq .
```

Phone numbers must include country code, no `+` prefix (e.g. `60123456789` for Malaysia).

## Send a Template Message

Templates must be pre-approved in Meta Business Manager before use.

```bash
curl -s -X POST "https://graph.facebook.com/v19.0/$WHATSAPP_PHONE_ID/messages" \
  -H "Authorization: Bearer $WHATSAPP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "to": "60123456789",
    "type": "template",
    "template": {
      "name": "order_confirmation",
      "language": {"code": "en"},
      "components": [{
        "type": "body",
        "parameters": [
          {"type": "text", "text": "Ahmad"},
          {"type": "text", "text": "#ORD-5678"},
          {"type": "text", "text": "RM 49.90"}
        ]
      }]
    }
  }' | jq .
```

## Bulk Send from a List

```bash
# numbers.txt — one per line, no + prefix
while IFS= read -r number; do
  curl -s -X POST "https://graph.facebook.com/v19.0/$WHATSAPP_PHONE_ID/messages" \
    -H "Authorization: Bearer $WHATSAPP_TOKEN" \
    -H "Content-Type: application/json" \
    -d "{\"messaging_product\":\"whatsapp\",\"to\":\"$number\",\"type\":\"text\",\"text\":{\"body\":\"Hi! Our Raya sale starts tomorrow — up to 50% off!\"}}" | jq -r '.messages[0].id'
  sleep 1  # stay within rate limits
done < numbers.txt
```

## Tips

- Free tier: 1,000 conversations/month; paid tiers scale to millions
- Rate limit: ~80 messages/second on Cloud API
- Only send to users who have opted in — spam will get your number banned
- Use templates for first-contact messages; free-form text only in 24h reply window
