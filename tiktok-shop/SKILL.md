---
name: tiktok-shop
version: 1.0.0
description: Manage TikTok Shop products, orders, and inventory via the Open API v2.
author: ZeptoClaw
license: MIT
tags:
  - tiktok
  - ecommerce
  - sea
  - marketplace
env_needed:
  - name: TIKTOK_APP_KEY
    description: App Key from TikTok Shop Partner Center
    required: true
  - name: TIKTOK_APP_SECRET
    description: App Secret from TikTok Shop Partner Center
    required: true
  - name: TIKTOK_ACCESS_TOKEN
    description: Shop access token (from OAuth flow)
    required: true
metadata: {"zeptoclaw":{"emoji":"🎵","requires":{"anyBins":["curl","jq","openssl"]}}}
---

# TikTok Shop Skill

Manage products, orders, and inventory on TikTok Shop using the Open API v2.

## Setup

1. Register at [TikTok Shop Partner Center](https://partner.tiktokshop.com/)
2. Create an app and get App Key + App Secret
3. Complete OAuth flow to get an access token for the target shop

```bash
export TIKTOK_APP_KEY="your_app_key"
export TIKTOK_APP_SECRET="your_app_secret"
export TIKTOK_ACCESS_TOKEN="your_access_token"
```

## Signing Requests

TikTok Shop uses HMAC-SHA256 request signing. Helper function:

```bash
tiktok_sign() {
  local path="$1" timestamp=$(date +%s)
  local sign_str="${TIKTOK_APP_SECRET}${path}${timestamp}${TIKTOK_APP_SECRET}"
  local sign=$(echo -n "$sign_str" | openssl dgst -sha256 -hmac "$TIKTOK_APP_SECRET" | awk '{print $2}')
  echo "timestamp=$timestamp&sign=$sign"
}
```

## List Products

```bash
PARAMS=$(tiktok_sign "/api/products/search")
curl -s -X POST "https://open-api.tiktokglobalshop.com/api/products/search?app_key=$TIKTOK_APP_KEY&access_token=$TIKTOK_ACCESS_TOKEN&$PARAMS" \
  -H "Content-Type: application/json" \
  -d '{"page_size": 20}' \
  | jq '.data.products[] | {id, name: .product_name, status: .product_status}'
```

## List Orders

```bash
PARAMS=$(tiktok_sign "/api/orders/search")
curl -s -X POST "https://open-api.tiktokglobalshop.com/api/orders/search?app_key=$TIKTOK_APP_KEY&access_token=$TIKTOK_ACCESS_TOKEN&$PARAMS" \
  -H "Content-Type: application/json" \
  -d '{"page_size": 20}' \
  | jq '.data.order_list[] | {id: .order_id, status: .order_status, total: .payment_info.total_amount}'
```

## Update Inventory

```bash
PARAMS=$(tiktok_sign "/api/products/stocks")
curl -s -X POST "https://open-api.tiktokglobalshop.com/api/products/stocks?app_key=$TIKTOK_APP_KEY&access_token=$TIKTOK_ACCESS_TOKEN&$PARAMS" \
  -H "Content-Type: application/json" \
  -d '{
    "product_id": "PRODUCT_ID",
    "skus": [{"id": "SKU_ID", "stock_infos": [{"warehouse_id": "WH_ID", "available_stock": 100}]}]
  }' | jq '.code, .message'
```

## Tips

- Signing is required for ALL API calls — use the helper function
- Access tokens expire — implement token refresh in production
- API base URL varies by region (global, US, UK, SEA)
- Rate limit: 10 requests/second per app
- Order statuses: `UNPAID`, `ON_HOLD`, `AWAITING_SHIPMENT`, `SHIPPED`, `COMPLETED`, `CANCELLED`
- Custom MCP server planned for `qhkm/zeptoclaw-sea` repo
