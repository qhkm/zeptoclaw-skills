---
name: billplz
version: 1.0.0
description: Create bills and check payments via Billplz — Malaysia's payment gateway.
author: ZeptoClaw
license: MIT
tags:
  - billplz
  - payments
  - malaysia
  - sea
  - fintech
env_needed:
  - name: BILLPLZ_API_KEY
    description: API key from billplz.com/enterprise/setting
    required: true
  - name: BILLPLZ_COLLECTION_ID
    description: Default collection ID for creating bills
    required: false
  - name: BILLPLZ_SANDBOX
    description: "Set to 'true' for sandbox mode (uses staging URL)"
    required: false
metadata: {"zeptoclaw":{"emoji":"💳","requires":{"anyBins":["curl","jq"]}}}
---

# Billplz Skill

Create payment bills, check payment status, and manage collections using Billplz — Malaysia's leading payment gateway.

## Setup

1. Register at [billplz.com](https://www.billplz.com) or [billplz-sandbox.com](https://www.billplz-sandbox.com) (testing)
2. Go to Settings → Keys & Tokens → Copy API Key

```bash
export BILLPLZ_API_KEY="your_api_key"
export BILLPLZ_COLLECTION_ID="your_collection_id"

# For sandbox/testing:
# export BILLPLZ_SANDBOX="true"
```

```bash
# Helper: set base URL
BILLPLZ_URL="${BILLPLZ_SANDBOX:+https://www.billplz-sandbox.com/api/v3}"
BILLPLZ_URL="${BILLPLZ_URL:-https://www.billplz.com/api/v3}"
```

## Create a Bill (Payment Request)

```bash
curl -s -X POST "$BILLPLZ_URL/bills" \
  -u "$BILLPLZ_API_KEY:" \
  -d "collection_id=$BILLPLZ_COLLECTION_ID" \
  -d "email=customer@example.com" \
  -d "name=Ahmad" \
  -d "amount=4990" \
  -d "description=Order #1234" \
  -d "callback_url=https://yourapp.com/webhook/billplz" \
  -d "redirect_url=https://yourapp.com/payment/complete" \
  | jq '{id, url, amount, state}'
```

## Check Bill Status

```bash
BILL_ID="bill-id-from-create"
curl -s "$BILLPLZ_URL/bills/$BILL_ID" \
  -u "$BILLPLZ_API_KEY:" \
  | jq '{id, state, paid, amount, paid_amount, url}'
```

## List Collections

```bash
curl -s "$BILLPLZ_URL/collections" \
  -u "$BILLPLZ_API_KEY:" \
  | jq '.collections[] | {id, title, status}'
```

## Create a Collection

```bash
curl -s -X POST "$BILLPLZ_URL/collections" \
  -u "$BILLPLZ_API_KEY:" \
  -d "title=My Store Payments" \
  | jq '{id, title}'
```

## Tips

- Amount is in **cents** (sen) — RM 49.90 = `4990`
- Bill states: `due` (unpaid), `paid`, `deleted`
- Basic Auth: API key as username, empty password (`-u "key:"`)
- Sandbox URL: `billplz-sandbox.com` — use for testing
- Supported payment methods: FPX, credit card, e-wallets
- Callback URL receives POST with `x_signature` for verification
- Custom MCP server planned for `qhkm/zeptoclaw-sea` repo
