---
name: agentic-wallet
version: 2.0.0
description: Give your agent a standalone on-chain wallet for gasless trades, funding, and autonomous x402 M2M payments.
author: aimaneth
license: MIT
tags:
  - crypto
  - wallet
  - coinbase
  - base
  - trading
  - x402
metadata: {"zeptoclaw":{"emoji":"💳","requires":{"anyBins":["npx", "node"]}}}
---

# Agentic Wallet Skill

Give a ZeptoClaw agent its own standalone on-chain wallet natively controlled by the agent. Powered by Coinbase Developer Platform (CDP).

Unlike wallet SDKs that need to be coded into the agent's source code, **Agentic Wallet** provides a straightforward CLI (`awal`) that your agent can use via standard shell execution to manage funds, execute gasless trades, and engage in agent-to-agent commerce dynamically.

## 1. Authentication

The wallet requires authentication. The agent must log in using an email OTP flow.

```bash
# 1. Initiate login (agent must ask user for the email)
npx awal@latest auth login agent@example.com

# 2. The API returns a flowId. The agent must ask the user for the OTP code sent to their email.
# 3. Verify OTP:
npx awal@latest auth verify <flowId> <otp>
```

Check auth status at any time:
```bash
npx awal@latest status
```

## 2. Wallet Info & Funding

All Agentic Wallet operations occur on the **Base** network. The wallet must be funded with USDC or ETH on Base.

```bash
# Get the wallet's Base network address to receive funds
npx awal@latest address

# Check the wallet's balances
npx awal@latest balance
```

To fund the wallet via fiat (credit card/Apple Pay), generate a funding link for the user:
```bash
# Opens the Coinbase Onramp UI (or provides a URL if running headless)
npx awal@latest show
```

## 3. Send Funds (Gasless)

Send tokens to any EVM address or ENS name on Base. These transactions are sponsored and do not require the agent to hold ETH for gas.

```bash
# Send exact USDC amounts to an address
npx awal@latest send 1.50 0x1234...abcd

# Send using a fiat value prefix
npx awal@latest send "$5.00" 0x1234...abcd

# Send to an ENS name (awal resolves it automatically)
npx awal@latest send 10 vitalik.eth
```

## 4. Trade Tokens (Gasless)

Swap tokens directly from the agent's wallet. Trades are also gasless.

```bash
# Trade exactly 1 USDC for ETH
npx awal@latest trade 1 usdc eth

# Trade $5 worth of USDC for DEGEN, with 2% slippage (200 bps)
npx awal@latest trade $5 usdc degen --slippage 200

# Trade using contract addresses directly
npx awal@latest trade 100 0x83358... 0x42000...
```

## 5. Pay for Services (x402)

Agentic wallets natively support the **x402** protocol for machine-to-machine (M2M) commerce. When the agent needs to access a paid API, it can pay the invoice autonomously.

```bash
# Basic GET request to a paid API. Automatically detects the x402 requirement and pays it.
npx awal@latest x402 pay https://example.com/api/weather

# Prevent overspending by setting a hard cap (e.g., max $0.10)
npx awal@latest x402 pay https://example.com/api/weather --max-amount 100000

# Advanced POST request paying for a specialized AI model
npx awal@latest x402 pay https://example.com/api/sentiment \
  -X POST \
  -d '{"text": "Analyze this text"}'
```

*(Note: `--max-amount 100000` equals $0.10 because x402 uses 6 decimal places for USDC).*

## 6. Discover Paid APIs (x402 Bazaar)

Agents can search the **x402 Bazaar** to discover paid APIs that other agents are hosting.

```bash
# Search for weather APIs in the bazaar (returns top 5 results)
npx awal@latest x402 bazaar search "weather"

# Search for AI models and return the top 10
npx awal@latest x402 bazaar search "sentiment analysis" -k 10

# List all available bazaar resources with full details
npx awal@latest x402 bazaar list --full

# Inspect the exact payment requirements (price) for a specific API before calling it
npx awal@latest x402 details https://example.com/api/weather
```

## 7. Monetize Services

Agents can also *earn* money by hosting their own paid x402 APIs. The agent can write and run a simple Node.js Express server using the `x402-express` middleware.

```javascript
// index.js (Agent writes this file to start earning)
const express = require("express");
const { paymentMiddleware } = require("x402-express");

const app = express();
app.use(express.json());

// 1. Get the agent's wallet address using `npx awal@latest address`
const PAY_TO = "0xYOUR_AGENT_WALLET_ADDRESS";

// 2. Set up the payment requirements ($0.05 per request)
const payment = paymentMiddleware(PAY_TO, {
  "POST /api/analyze": { 
      price: "$0.05", 
      network: "base" 
  },
});

// 3. Protect the route with the middleware
app.post("/api/analyze", payment, (req, res) => {
  res.json({ result: "Here is your paid analysis" });
});

app.listen(3000, () => console.log("Agent API running on port 3000"));
```

Other agents can now call your agent's API using `npx awal@latest x402 pay http://<your-ip>:3000/api/analyze` and the USDC will flow directly into your agent's wallet!

## Environment Notes

This skill requires Node.js and `npx` to be installed in the agent's execution environment. No global npm installs are required since `npx awal@latest` fetches the tool dynamically.
