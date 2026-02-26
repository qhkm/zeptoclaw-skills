---
name: uniswap
version: 1.0.0
description: Get swap quotes, check approvals, execute swaps, and generate deep links using the Uniswap Trading API.
author: aimaneth
license: MIT
tags:
  - crypto
  - defi
  - trading
  - uniswap
env_needed:
  - name: UNISWAP_API_KEY
    description: API key from the Uniswap Developer Portal (https://developers.uniswap.org/)
    required: true
metadata: {"zeptoclaw":{"emoji":"🦄","requires":{"anyBins":["curl","jq"]}}}
---

# Uniswap Skill

Interact with [Uniswap](https://uniswap.org/) — the largest decentralised exchange — via the Trading API. Get swap quotes, check token approvals, execute swaps, and generate deep links that open the Uniswap UI with pre-filled parameters.

Based on [Uniswap AI](https://github.com/Uniswap/uniswap-ai) — adapted for ZeptoClaw agents.

## Setup

1. Get an API key from the [Uniswap Developer Portal](https://developers.uniswap.org/)
2. Set environment variable:

```bash
export UNISWAP_API_KEY="your_api_key_here"
```

**Base URL**: `https://trade-api.gateway.uniswap.org/v1`

**Required headers for all requests**:

```
Content-Type: application/json
x-api-key: $UNISWAP_API_KEY
x-universal-router-version: 2.0
```

## Get a Swap Quote

Get the best price for a token swap. This is the most common operation.

```bash
curl -s -X POST https://trade-api.gateway.uniswap.org/v1/quote \
  -H "Content-Type: application/json" \
  -H "x-api-key: $UNISWAP_API_KEY" \
  -H "x-universal-router-version: 2.0" \
  -d '{
    "swapper": "0xYOUR_WALLET_ADDRESS",
    "tokenIn": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
    "tokenOut": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
    "tokenInChainId": "1",
    "tokenOutChainId": "1",
    "amount": "1000000000000000000",
    "type": "EXACT_INPUT",
    "slippageTolerance": 0.5,
    "routingPreference": "BEST_PRICE"
  }' | jq .
```

The above example quotes swapping **1 WETH → USDC** on Ethereum mainnet.

**Key parameters**:

| Parameter           | Values                                  |
| ------------------- | --------------------------------------- |
| `type`              | `EXACT_INPUT` or `EXACT_OUTPUT`         |
| `slippageTolerance` | 0–100 (percentage)                      |
| `routingPreference` | `BEST_PRICE`, `FASTEST`, or `CLASSIC`   |
| `protocols`         | Optional: `["V2", "V3", "V4"]`          |

**Response** includes `quote.output.amount`, `quote.gasFeeUSD`, and routing details.

> **Tip**: Use the `gasFeeUSD` field to display gas costs. Don't manually convert `gasFee` (wei) — it leads to inaccurate estimates.

## Check Token Approval

Before swapping ERC-20 tokens, check if the token is approved for the Uniswap router.

```bash
curl -s -X POST https://trade-api.gateway.uniswap.org/v1/check_approval \
  -H "Content-Type: application/json" \
  -H "x-api-key: $UNISWAP_API_KEY" \
  -H "x-universal-router-version: 2.0" \
  -d '{
    "walletAddress": "0xYOUR_WALLET_ADDRESS",
    "token": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
    "amount": "1000000000",
    "chainId": 1
  }' | jq .
```

- If `approval` is `null` — token is already approved, proceed to swap.
- If `approval` contains a transaction — sign and submit it before swapping.

## Execute a Swap (3-Step Flow)

The full swap flow is: **check_approval → quote → swap**.

After getting a quote, send it to the swap endpoint to get a ready-to-sign transaction:

```bash
# Step 1: Get quote (see above)
# Step 2: Send quote response to /swap
curl -s -X POST https://trade-api.gateway.uniswap.org/v1/swap \
  -H "Content-Type: application/json" \
  -H "x-api-key: $UNISWAP_API_KEY" \
  -H "x-universal-router-version: 2.0" \
  -d '{
    "routing": "CLASSIC",
    "quote": {
      "input": {"token": "0x...", "amount": "1000000000000000000"},
      "output": {"token": "0x...", "amount": "999000000"},
      "slippage": 0.5
    },
    "swapper": "0xYOUR_WALLET_ADDRESS",
    "tokenIn": "0x...",
    "tokenOut": "0x...",
    "tokenInChainId": "1",
    "tokenOutChainId": "1",
    "amount": "1000000000000000000",
    "type": "EXACT_INPUT"
  }' | jq .
```

**Response** returns a ready-to-sign transaction with `to`, `from`, `data`, `value`, and `gasLimit`.

> **Important**: Spread the quote response into the swap request body — don't wrap it in `{"quote": ...}`. Also strip any `null` fields like `permitData: null` before sending.

## Generate a Uniswap Deep Link

Generate a URL that opens the Uniswap swap interface with parameters pre-filled:

```bash
# ETH → USDC on Ethereum
echo "https://app.uniswap.org/swap?chain=ethereum&inputCurrency=ETH&outputCurrency=0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48&exactAmount=1&exactField=input"

# USDC → ETH on Base
echo "https://app.uniswap.org/swap?chain=base&inputCurrency=0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913&outputCurrency=ETH&exactAmount=100&exactField=input"
```

**Deep link parameters**:

| Parameter        | Description                          |
| ---------------- | ------------------------------------ |
| `chain`          | `ethereum`, `base`, `arbitrum`, etc. |
| `inputCurrency`  | Token address or `ETH`               |
| `outputCurrency` | Token address or `ETH`               |
| `exactAmount`    | Amount in human-readable units       |
| `exactField`     | `input` or `output`                  |

## Supported Chains

| Chain ID | Chain       | Chain ID | Chain       |
| -------- | ----------- | -------- | ----------- |
| 1        | Ethereum    | 8453     | Base        |
| 10       | Optimism    | 42161    | Arbitrum    |
| 56       | BNB Chain   | 42220    | Celo        |
| 130      | Unichain    | 43114    | Avalanche   |
| 137      | Polygon     | 81457    | Blast       |
| 196      | X Layer     | 7777777  | Zora        |
| 324      | zkSync      | 480      | World Chain |

## Common Token Addresses

| Token | Ethereum (1)                                 | Base (8453)                                  |
| ----- | -------------------------------------------- | -------------------------------------------- |
| WETH  | `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` | `0x4200000000000000000000000000000000000006` |
| USDC  | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| USDT  | `0xdAC17F958D2ee523a2206206994597C13D831ec7` | —                                            |
| DAI   | `0x6B175474E89094C44Da98b954EedeAC495271d0F` | —                                            |

## Tips

- **Quote freshness**: Quotes expire quickly — get a fresh quote right before executing a swap
- **Slippage**: Use `0.5` (0.5%) for stablecoins, `1.0`–`3.0` for volatile pairs
- **Chain IDs as strings**: `tokenInChainId` and `tokenOutChainId` must be strings (e.g. `"1"`), not numbers
- **Permit2**: When using Permit2, `signature` and `permitData` must both be present or both absent — never set `permitData: null`
- **Native ETH**: Use `ETH` in deep links. For API calls, use the WETH address
- **Amount format**: Amounts are in the token's smallest unit (wei for ETH = 18 decimals, 6 decimals for USDC)
- **Rate limits**: Be mindful of API rate limits — cache quotes when displaying to users
