---
name: lazada
version: 1.0.0
description: Lazada Seller Center API — manage products, orders, and inventory across SEA.
author: ZeptoClaw
license: MIT
tags:
  - lazada
  - ecommerce
  - sea
  - seller
env_needed:
  - name: LAZADA_APP_KEY
    description: App key from Lazada Open Platform (open.lazada.com)
    required: true
  - name: LAZADA_APP_SECRET
    description: App secret from Lazada Open Platform
    required: true
  - name: LAZADA_ACCESS_TOKEN
    description: OAuth access token (obtained after seller authorization)
    required: true
  - name: LAZADA_REGION
    description: "Region endpoint: my (Malaysia), sg, th, ph, vn, id"
    required: true
metadata: {"zeptoclaw":{"emoji":"🛍️","requires":{"anyBins":["curl","jq","python3"]}}}
---

# Lazada Seller Skill

Manage your Lazada store across Southeast Asia using the Lazada Open Platform API.

## Setup

1. Register at [open.lazada.com](https://open.lazada.com) → Create App
2. Get your App Key and App Secret
3. Complete OAuth flow to get an Access Token for your seller account
4. Set your region: `my`, `sg`, `th`, `ph`, `vn`, or `id`

```bash
export LAZADA_APP_KEY="12345"
export LAZADA_APP_SECRET="abc123secret"
export LAZADA_ACCESS_TOKEN="50001234|xxx..."
export LAZADA_REGION="my"  # Malaysia
```

## Signature Helper

All Lazada API calls require HMAC-SHA256 signatures. Use this helper:

```bash
lazada_call() {
  local path="$1" params="$2"
  local ts=$(date +%s%3N)
  local base="$path${params}&access_token=$LAZADA_ACCESS_TOKEN&app_key=$LAZADA_APP_KEY&sign_method=sha256&timestamp=$ts"
  # Sort params alphabetically for signing (use Python for reliability)
  local sign=$(echo -n "$base" | python3 -c "
import sys, hmac, hashlib
msg = sys.stdin.read()
print(hmac.new('$LAZADA_APP_SECRET'.encode(), msg.encode(), hashlib.sha256).hexdigest().upper())
")
  curl -s "https://api.lazada.com.$LAZADA_REGION/rest$path?$params&access_token=$LAZADA_ACCESS_TOKEN&app_key=$LAZADA_APP_KEY&sign_method=sha256&timestamp=$ts&sign=$sign"
}
```

## Get Orders

```bash
# Pending orders from last 7 days
curl -s "https://api.lazada.com.$LAZADA_REGION/rest/orders/get" \
  # Use the SDK or sign manually (see helper above)
  # Params: status=pending, created_after=2026-02-01T00:00:00+08:00
  | jq '.data.orders[] | {order_id, status, price}'
```

## Get Order Items

```bash
# Get items for a specific order
# GET /order/items/get?order_id=123456
curl ... | jq '.data[] | {name, sku, quantity, item_price}'
```

## Update Order Status (Mark as Shipped)

```bash
# POST /order/deliver — mark item as delivered for seller-fulfilled orders
# Required params: delivery_type=dropship, order_item_ids=[123,456]
```

## Get Products

```bash
# List active products
# GET /products/get?status=active&limit=50&offset=0
curl ... | jq '.data.products[] | {item_id, name: .attributes.name, quantity: .skus[0].quantity}'
```

## Update Stock Quantity

```bash
# POST /product/price/quantity/update
# Body: {"skuSellerList": [{"skuId": "123", "quantity": 50}]}
```

## Tips

- Use the [Lazada SDK](https://open.lazada.com/doc/sdk.htm) if doing many calls — it handles signing automatically
- Access tokens expire in 30 days — store refresh tokens and renew proactively
- Rate limit: ~100 calls/minute per app
- Test in Lazada Sandbox before running in production
- For Malaysia sellers: use `api.lazada.com.my` endpoint
