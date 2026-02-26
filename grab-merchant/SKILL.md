---
name: grab-merchant
version: 1.0.0
description: Manage GrabFood/GrabMart orders, menus, and store hours via the Merchant API.
author: ZeptoClaw
license: MIT
tags:
  - grab
  - food-delivery
  - sea
  - ecommerce
env_needed:
  - name: GRAB_CLIENT_ID
    description: OAuth2 client ID from Grab Developer Portal
    required: true
  - name: GRAB_CLIENT_SECRET
    description: OAuth2 client secret
    required: true
  - name: GRAB_MERCHANT_ID
    description: Your Grab merchant ID
    required: true
metadata: {"zeptoclaw":{"emoji":"🏍️","requires":{"anyBins":["curl","jq"]}}}
---

# Grab Merchant Skill

Manage GrabFood and GrabMart operations — orders, menus, and store availability.

## Setup

1. Register at [developer.grab.com](https://developer.grab.com)
2. Create an app with `food.partner_api` scope
3. Get OAuth2 credentials

```bash
export GRAB_CLIENT_ID="your_client_id"
export GRAB_CLIENT_SECRET="your_client_secret"
export GRAB_MERCHANT_ID="your_merchant_id"
```

## Get Access Token

```bash
export GRAB_ACCESS_TOKEN=$(curl -s -X POST "https://partner-api.grab.com/grabid/v1/oauth2/token" \
  -H "Content-Type: application/json" \
  -d "{
    \"client_id\": \"$GRAB_CLIENT_ID\",
    \"client_secret\": \"$GRAB_CLIENT_SECRET\",
    \"grant_type\": \"client_credentials\",
    \"scope\": \"food.partner_api\"
  }" | jq -r '.access_token')
```

## List Recent Orders

```bash
curl -s "https://partner-api.grab.com/merchant/v2/orders?merchantID=$GRAB_MERCHANT_ID&limit=20" \
  -H "Authorization: Bearer $GRAB_ACCESS_TOKEN" \
  | jq '.orders[] | {id: .orderID, status, total: .price.totalPrice, items: [.items[].name]}'
```

## Get Store Hours

```bash
curl -s "https://partner-api.grab.com/merchant/v2/store/hours?merchantID=$GRAB_MERCHANT_ID" \
  -H "Authorization: Bearer $GRAB_ACCESS_TOKEN" \
  | jq '.schedules[] | {day: .dayOfWeek, open: .openTime, close: .closeTime}'
```

## Pause/Resume Store

```bash
curl -s -X POST "https://partner-api.grab.com/merchant/v2/store/pause" \
  -H "Authorization: Bearer $GRAB_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"merchantID\": \"$GRAB_MERCHANT_ID\", \"duration\": 60}" \
  | jq '.success'
```

## Cancel an Order

```bash
ORDER_ID="order-id"
curl -s -X POST "https://partner-api.grab.com/merchant/v2/orders/cancel" \
  -H "Authorization: Bearer $GRAB_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"orderID\": \"$ORDER_ID\", \"merchantID\": \"$GRAB_MERCHANT_ID\", \"cancelCode\": 1001}" \
  | jq '.success'
```

## Tips

- OAuth2 access tokens expire in 1 hour — refresh using client_credentials flow
- Cancel codes: 1001 (can't fulfill), 1002 (out of stock), 1003 (store closed)
- API is region-scoped — base URL may vary by country
- Rate limit depends on partnership tier
- Custom MCP server planned for `qhkm/zeptoclaw-sea` repo
