---
name: tokopedia
version: 1.0.0
description: Manage Tokopedia shop products, orders, and inventory via the Seller API.
author: ZeptoClaw
license: MIT
tags:
  - tokopedia
  - ecommerce
  - indonesia
  - sea
  - marketplace
env_needed:
  - name: TOKOPEDIA_CLIENT_ID
    description: App client ID from Tokopedia Developer Console
    required: true
  - name: TOKOPEDIA_CLIENT_SECRET
    description: App client secret
    required: true
  - name: TOKOPEDIA_SHOP_ID
    description: Your shop ID
    required: true
metadata: {"zeptoclaw":{"emoji":"🇮🇩","requires":{"anyBins":["curl","jq"]}}}
---

# Tokopedia Skill

Manage products, orders, and shop info on Tokopedia — Indonesia's largest marketplace.

## Setup

1. Register at [developer.tokopedia.com](https://developer.tokopedia.com)
2. Create an app and get OAuth2 credentials
3. Get access token:

```bash
export TOKOPEDIA_CLIENT_ID="your_client_id"
export TOKOPEDIA_CLIENT_SECRET="your_client_secret"
export TOKOPEDIA_SHOP_ID="your_shop_id"
```

## Get Access Token

```bash
export TOKOPEDIA_ACCESS_TOKEN=$(curl -s -X POST "https://accounts.tokopedia.com/token" \
  -u "$TOKOPEDIA_CLIENT_ID:$TOKOPEDIA_CLIENT_SECRET" \
  -d "grant_type=client_credentials" \
  | jq -r '.access_token')
```

## List Products

```bash
curl -s "https://fs.tokopedia.net/inventory/v1/fs/$TOKOPEDIA_CLIENT_ID/product/info?shop_id=$TOKOPEDIA_SHOP_ID&page=1&per_page=20" \
  -H "Authorization: Bearer $TOKOPEDIA_ACCESS_TOKEN" \
  | jq '.data[] | {id: .basic.productID, name: .basic.name, price: .price.value, stock: .stock.value}'
```

## List Orders

```bash
curl -s "https://fs.tokopedia.net/v2/order/list?shop_id=$TOKOPEDIA_SHOP_ID&page=1&per_page=20&fs_id=$TOKOPEDIA_CLIENT_ID" \
  -H "Authorization: Bearer $TOKOPEDIA_ACCESS_TOKEN" \
  | jq '.data[] | {order_id, status, buyer: .buyer_info.name, total: .amt.total_amount}'
```

## Accept an Order

```bash
ORDER_ID="order-id"
curl -s -X POST "https://fs.tokopedia.net/v1/order/$ORDER_ID/fs/$TOKOPEDIA_CLIENT_ID/ack" \
  -H "Authorization: Bearer $TOKOPEDIA_ACCESS_TOKEN" \
  | jq '.header.process_time'
```

## Get Shop Info

```bash
curl -s "https://fs.tokopedia.net/v1/shop/fs/$TOKOPEDIA_CLIENT_ID/shop-info?shop_id=$TOKOPEDIA_SHOP_ID" \
  -H "Authorization: Bearer $TOKOPEDIA_ACCESS_TOKEN" \
  | jq '.data[] | {id: .shop_id, name: .shop_name, domain, reputation}'
```

## Tips

- OAuth2 token expires in ~1 hour — implement refresh
- Base URL: `fs.tokopedia.net` (fulfillment service)
- Shop ID is numeric — find it in your seller dashboard URL
- Rate limit varies by endpoint (~50 req/min typical)
- Order statuses: 0 (seller), 100 (payment verified), 220 (shipped), 400 (delivered), 700 (completed)
- Custom MCP server planned for `qhkm/zeptoclaw-sea` repo
