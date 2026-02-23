---
name: send-email
version: 1.0.0
description: Send transactional emails via Resend — order confirmations, alerts, reports.
author: ZeptoClaw
license: MIT
tags:
  - email
  - notifications
  - transactional
env_needed:
  - name: RESEND_API_KEY
    description: Resend API key from resend.com/api-keys
    required: true
  - name: EMAIL_FROM
    description: Verified sender address (e.g. noreply@yourdomain.com)
    required: true
metadata: {"zeptoclaw":{"emoji":"📧","requires":{"anyBins":["curl","jq"]}}}
---

# Send Email Skill

Send transactional emails using [Resend](https://resend.com) — the simplest modern email API. Free tier: 3,000 emails/month.

## Setup

1. Sign up at [resend.com](https://resend.com) → API Keys → Create Key
2. Add and verify your sending domain (or use `onboarding@resend.dev` for testing)

```bash
export RESEND_API_KEY="re_xxx..."
export EMAIL_FROM="noreply@yourdomain.com"
```

## Send a Plain Text Email

```bash
curl -s -X POST "https://api.resend.com/emails" \
  -H "Authorization: Bearer $RESEND_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"from\": \"$EMAIL_FROM\",
    \"to\": [\"customer@example.com\"],
    \"subject\": \"Your order has shipped!\",
    \"text\": \"Hi Ahmad,\n\nYour order #1234 is on its way. Expected delivery: 2-3 days.\n\nThank you!\"
  }" | jq -r '.id'
```

## Send an HTML Email

```bash
curl -s -X POST "https://api.resend.com/emails" \
  -H "Authorization: Bearer $RESEND_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"from\": \"$EMAIL_FROM\",
    \"to\": [\"customer@example.com\"],
    \"subject\": \"Order Confirmation #1234\",
    \"html\": \"<h1>Thank you for your order!</h1><p>Order <strong>#1234</strong> is confirmed.</p><p>Total: <strong>RM 49.90</strong></p>\"
  }" | jq -r '.id'
```

## Send to Multiple Recipients

```bash
curl -s -X POST "https://api.resend.com/emails" \
  -H "Authorization: Bearer $RESEND_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"from\": \"$EMAIL_FROM\",
    \"to\": [\"alice@example.com\", \"bob@example.com\"],
    \"cc\": [\"manager@yourcompany.com\"],
    \"subject\": \"Weekly Report\",
    \"text\": \"Please find this week's summary attached.\"
  }" | jq .
```

## Send with Attachment

```bash
# Base64-encode the file
ATTACHMENT=$(base64 -i report.pdf)

curl -s -X POST "https://api.resend.com/emails" \
  -H "Authorization: Bearer $RESEND_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"from\": \"$EMAIL_FROM\",
    \"to\": [\"customer@example.com\"],
    \"subject\": \"Your Invoice\",
    \"text\": \"Please find your invoice attached.\",
    \"attachments\": [{
      \"filename\": \"invoice.pdf\",
      \"content\": \"$ATTACHMENT\"
    }]
  }" | jq -r '.id'
```

## Tips

- Test domain: use `delivered@resend.dev` to test without a real address
- Rate limit: 10 emails/second on free tier
- Always include both `text` and `html` — some clients prefer plain text
- Check delivery status: `curl https://api.resend.com/emails/{id} -H "Authorization: Bearer $RESEND_API_KEY"`
