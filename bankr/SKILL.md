---
name: bankr
version: 1.0.0
description: AI-powered crypto trading agent and LLM gateway via natural language. Use when the user wants to trade crypto, check portfolio balances, view token prices, transfer crypto, manage NFTs, use leverage, bet on Polymarket, deploy tokens, set up automated trading, sign and submit raw transactions, or access LLM models through the Bankr LLM gateway funded by your Bankr wallet. Supports Base, Ethereum, Polygon, Solana, and Unichain.
author: BankrBot
license: MIT
tags:
  - crypto
  - defi
  - trading
  - wallet
metadata: {"zeptoclaw":{"emoji":"📺","homepage":"https://bankr.bot","requires":{"anyBins":["bankr"]}}}
---

# Bankr

Execute crypto trading and DeFi operations using natural language. Two integration options:

1. **Bankr CLI** (recommended) — Install `@bankr/cli` for a batteries-included terminal experience
2. **REST API** — Call `https://api.bankr.bot` directly from any language or tool

Both use the same API key and the same async job workflow under the hood.


## 🚨 Financial Safety Guardrails

These skills enable high-risk financial operations (leverage trading up to 100x, token deployment, Polymarket betting, raw transaction submission). You MUST adhere to the following safety rules:

- **Always confirm with the user before executing any transaction.**
- **Display amounts, fees, slippage, and risk level before execution.**
- **Refuse to execute leverage >10x without explicit double-confirmation.**
- **Warn users about irreversible on-chain operations.**

## Getting an API Key

Before using either option, you need a Bankr API key. Two ways to get one:

**Option A: Headless email login (recommended for agents)**

Two-step flow — send OTP, then verify and complete setup. See "First-Time Setup" below for the full guided flow with user preference prompts.

```bash
# Step 1 — send OTP to email
bankr login email user@example.com

# Step 2 — verify OTP and generate API key (options based on user preferences)
bankr login email user@example.com --code 123456 --accept-terms --key-name "My Agent" --read-write
```

This creates a wallet, accepts terms, and generates an API key — no browser needed. Before running step 2, ask the user whether they need read-only or read-write access, LLM gateway, and their preferred key name.

**Option B: Bankr Terminal**

1. Visit [bankr.bot/api](https://bankr.bot/api)
2. **Sign up / Sign in** — Enter your email and the one-time passcode (OTP) sent to it
3. **Generate an API key** — Create a key with **Agent API** access enabled (the key starts with `bk_...`)

Both options automatically provision **EVM wallets** (Base, Ethereum, Polygon, Unichain) and a **Solana wallet** — no manual wallet setup needed.

## Option 1: Bankr CLI (Recommended)

### Install

```bash
bun install -g @bankr/cli
```

Or with npm:

```bash
npm install -g @bankr/cli
```

### First-Time Setup

#### Headless email login (recommended for agents)

When the user asks to log in with an email, walk them through this flow:

**Step 1 — Send verification code**

```bash
bankr login email <user-email>
```

**Step 2 — Ask the user for the OTP code** they received via email.

**Step 3 — Before completing login, ask the user about their preferences:**

1. **Accept Terms of Service** — Present the [Terms of Service](https://bankr.bot/terms) link and confirm the user agrees. Required for new users — do not pass `--accept-terms` unless the user has explicitly confirmed.
2. **Read-only or read-write API key?**
   - **Read-only** (default) — portfolio, balances, prices, research only
   - **Read-write** (`--read-write`) — enables swaps, transfers, orders, token launches, leverage, Polymarket bets
3. **Enable LLM gateway access?** (`--llm`) — multi-model API at `llm.bankr.bot` (currently limited to beta testers). Skip if user doesn't need it.
4. **Key name?** (`--key-name`) — a display name for the API key (e.g. "My Agent", "Trading Bot")

**Step 4 — Construct and run the step 2 command** with the user's choices:

```bash
# Example with all options
bankr login email <user-email> --code <otp> --accept-terms --key-name "My Agent" --read-write --llm

# Example read-only, no LLM
bankr login email <user-email> --code <otp> --accept-terms --key-name "Research Bot"
```

#### Login options reference

| Option | Description |
|--------|-------------|
| `--code <otp>` | OTP code received via email (step 2) |
| `--accept-terms` | Accept [Terms of Service](https://bankr.bot/terms) without prompting (required for new users) |
| `--key-name <name>` | Display name for the API key (e.g. "My Agent"). Prompted if omitted |
| `--read-write` | Enable write operations: swaps, transfers, orders, token launches, leverage, Polymarket bets. **Without this flag, the key is read-only** (portfolio, balances, prices, research only) |
| `--llm` | Enable [LLM gateway](https://docs.bankr.bot/llm-gateway/overview) access (multi-model API at `llm.bankr.bot`). Currently limited to beta testers |

Any option not provided on the command line will be prompted interactively by the CLI, so you can mix headless and interactive as needed.

#### Login with existing API key

If the user already has an API key:

```bash
bankr login --api-key bk_YOUR_KEY_HERE
```

If they need to create one at the Bankr Terminal:
1. Run `bankr login --url` — prints the terminal URL
2. Present the URL to the user, ask them to generate a `bk_...` key
3. Run `bankr login --api-key bk_THE_KEY`

#### Separate LLM Gateway Key (Optional)

If your LLM gateway key differs from your API key, pass `--llm-key` during login or run `bankr config set llmKey YOUR_LLM_KEY` afterward. When not set, the API key is used for both. See the Advanced Reference Guide below (llm-gateway) for full details.

#### Verify Setup

```bash
bankr whoami
bankr prompt "What is my balance?"
```

## Option 2: REST API (Direct)

No CLI installation required — call the API directly with `curl`, `fetch`, or any HTTP client.

### Authentication

All requests require an `X-API-Key` header:

```bash
curl -X POST "https://api.bankr.bot/agent/prompt" \
  -H "X-API-Key: bk_YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is my ETH balance?"}'
```

### Quick Example: Submit → Poll → Complete

```bash
# 1. Submit a prompt — returns a job ID
JOB=$(curl -s -X POST "https://api.bankr.bot/agent/prompt" \
  -H "X-API-Key: $BANKR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is my ETH balance?"}')
JOB_ID=$(echo "$JOB" | jq -r '.jobId')

# 2. Poll until terminal status
while true; do
  RESULT=$(curl -s "https://api.bankr.bot/agent/job/$JOB_ID" \
    -H "X-API-Key: $BANKR_API_KEY")
  STATUS=$(echo "$RESULT" | jq -r '.status')
  [ "$STATUS" = "completed" ] || [ "$STATUS" = "failed" ] || [ "$STATUS" = "cancelled" ] && break
  sleep 2
done

# 3. Read the response
echo "$RESULT" | jq -r '.response'
```

### Conversation Threads

Every prompt response includes a `threadId`. Pass it back to continue the conversation:

```bash
# Start — the response includes a threadId
curl -X POST "https://api.bankr.bot/agent/prompt" \
  -H "X-API-Key: $BANKR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is the price of ETH?"}'
# → {"jobId": "job_abc", "threadId": "thr_XYZ", ...}

# Continue — pass threadId to maintain context
curl -X POST "https://api.bankr.bot/agent/prompt" \
  -H "X-API-Key: $BANKR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "And what about SOL?", "threadId": "thr_XYZ"}'
```

Omit `threadId` to start a new conversation. CLI equivalent: `bankr prompt --continue` (reuses last thread) or `bankr prompt --thread <id>`.

### API Endpoints Summary

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/agent/prompt` | POST | Submit a prompt (async, returns job ID) |
| `/agent/job/{jobId}` | GET | Check job status and results |
| `/agent/job/{jobId}/cancel` | POST | Cancel a running job |
| `/agent/balances` | GET | Wallet balances across chains (sync, optional `?chains=` filter) |
| `/agent/sign` | POST | Sign messages/transactions (sync) |
| `/agent/submit` | POST | Submit raw transactions (sync) |

For full API details (request/response schemas, job states, rich data, polling strategy), see:

**Reference**: See Advanced Reference Guide below (api-workflow) | the Advanced Reference Guide below (sign-submit-api)

## CLI Command Reference

### Core Commands

| Command | Description |
|---------|-------------|
| `bankr login` | Authenticate with the Bankr API (interactive menu) |
| `bankr login email <address>` | Send OTP to email (headless step 1) |
| `bankr login email <address> --code <otp> [options]` | Verify OTP and complete setup (headless step 2) |
| `bankr login --api-key <key>` | Login with an existing API key directly |
| `bankr login --api-key <key> --llm-key <key>` | Login with separate LLM gateway key |
| `bankr login --url` | Print Bankr Terminal URL for API key generation |
| `bankr logout` | Clear stored credentials |
| `bankr whoami` | Show current authentication info |
| `bankr prompt <text>` | Send a prompt to the Bankr AI agent |
| `bankr prompt --continue <text>` | Continue the most recent conversation thread |
| `bankr prompt --thread <id> <text>` | Continue a specific conversation thread |
| `bankr status <jobId>` | Check the status of a running job |
| `bankr cancel <jobId>` | Cancel a running job |
| `bankr balances` | Show wallet token balances across all chains |
| `bankr balances --chain <chains>` | Filter by chain(s): base, polygon, mainnet, unichain, solana (comma-separated) |
| `bankr balances --json` | Output raw JSON balances |
| `bankr skills` | Show all Bankr AI agent skills with examples |

### Configuration Commands

| Command | Description |
|---------|-------------|
| `bankr config get [key]` | Get config value(s) |
| `bankr config set <key> <value>` | Set a config value |
| `bankr --config <path> <command>` | Use a custom config file path |

Valid config keys: `apiKey`, `apiUrl`, `llmKey`, `llmUrl`

Default config location: `~/.bankr/config.json`. Override with `--config` or `BANKR_CONFIG` env var.

### Environment Variables

| Variable | Description |
|----------|-------------|
| `BANKR_API_KEY` | API key (overrides stored key) |
| `BANKR_API_URL` | API URL (default: `https://api.bankr.bot`) |
| `BANKR_LLM_KEY` | LLM gateway key (falls back to `BANKR_API_KEY` if not set) |
| `BANKR_LLM_URL` | LLM gateway URL (default: `https://llm.bankr.bot`) |

Environment variables override config file values. Config file values override defaults.

### LLM Gateway Commands

| Command | Description |
|---------|-------------|
| `bankr llm models` | List available LLM models |
| `bankr llm setup openclaw [--install]` | Generate or install OpenClaw config |
| `bankr llm setup opencode [--install]` | Generate or install OpenCode config |
| `bankr llm setup claude` | Show Claude Code environment setup |
| `bankr llm setup cursor` | Show Cursor IDE setup instructions |
| `bankr llm claude [args...]` | Launch Claude Code via the Bankr LLM Gateway |

## Core Usage

### Simple Query

For straightforward requests that complete quickly:

```bash
bankr prompt "What is my ETH balance?"
bankr prompt "What's the price of Bitcoin?"
```

The CLI handles the full submit-poll-complete workflow automatically. You can also use the shorthand — any unrecognized command is treated as a prompt:

```bash
bankr What is the price of ETH?
```

### Interactive Prompt

For prompts containing `$` or special characters that the shell would expand:

```bash
# Interactive mode — no shell expansion issues
bankr prompt
# Then type: Buy $50 of ETH on Base

# Or pipe input
echo 'Buy $50 of ETH on Base' | bankr prompt
```

### Conversation Threads

Continue a multi-turn conversation with the agent:

```bash
# First prompt — starts a new thread automatically
bankr prompt "What is the price of ETH?"
# → Thread: thr_ABC123

# Continue the conversation (agent remembers the ETH context)
bankr prompt --continue "And what about BTC?"
bankr prompt -c "Compare them"

# Resume any thread by ID
bankr prompt --thread thr_ABC123 "Show me ETH chart"
```

Thread IDs are automatically saved to config after each prompt. The `--continue` / `-c` flag reuses the last thread.

### Manual Job Control

For advanced use or long-running operations:

```bash
# Submit and get job ID
bankr prompt "Buy $100 of ETH"
# → Job submitted: job_abc123

# Check status of a specific job
bankr status job_abc123

# Cancel if needed
bankr cancel job_abc123
```

## LLM Gateway

The [Bankr LLM Gateway](https://docs.bankr.bot/llm-gateway/overview) is a unified API for Claude, Gemini, GPT, and other models — multi-provider access, cost tracking, automatic failover, and SDK compatibility through a single endpoint.

**Base URL:** `https://llm.bankr.bot`

Uses your `llmKey` if configured, otherwise falls back to your API key.

### Quick Commands

```bash
bankr llm models                           # List available models
bankr llm credits                          # Check credit balance
bankr llm setup openclaw --install         # Install Bankr provider into OpenClaw
bankr llm setup opencode --install         # Install Bankr provider into OpenCode
bankr llm setup claude                     # Print Claude Code env vars
bankr llm setup cursor                     # Cursor setup instructions
bankr llm claude                           # Launch Claude Code through gateway
bankr llm claude --model claude-opus-4.6   # Launch with specific model
```

### Direct SDK Usage

The gateway works with standard OpenAI and Anthropic SDKs — just override the base URL:

```bash
# OpenAI-compatible
curl -X POST "https://llm.bankr.bot/v1/chat/completions" \
  -H "Authorization: Bearer $BANKR_LLM_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "claude-sonnet-4.5", "messages": [{"role": "user", "content": "Hello"}]}'
```

For full model list, provider config JSON shape, SDK examples (Python, TypeScript), all setup commands, and troubleshooting, see:

**Reference**: See Advanced Reference Guide below (llm-gateway)

## Capabilities Overview

### Trading Operations

- **Token Swaps**: Buy/sell/swap tokens across chains
- **Cross-Chain**: Bridge tokens between chains
- **Limit Orders**: Execute at target prices
- **Stop Loss**: Automatic sell protection
- **DCA**: Dollar-cost averaging strategies
- **TWAP**: Time-weighted average pricing

**Reference**: See Advanced Reference Guide below (token-trading)

### Portfolio Management

- Check balances across all chains (`bankr balances` or `GET /agent/balances`)
- View USD valuations
- Track holdings by token or chain
- Real-time price updates
- Multi-chain aggregation
- Filter by chain: `bankr balances --chain base,solana` or `GET /agent/balances?chains=base,solana`

**Reference**: See Advanced Reference Guide below (portfolio)

### Market Research

- Token prices and market data
- Technical analysis (RSI, MACD, etc.)
- Social sentiment analysis
- Price charts
- Trending tokens
- Token comparisons

**Reference**: See Advanced Reference Guide below (market-research)

### Transfers

- Send to addresses, ENS, or social handles
- Multi-chain support
- Flexible amount formats
- Social handle resolution (Twitter, Farcaster, Telegram)

**Reference**: See Advanced Reference Guide below (transfers)

### NFT Operations

- Browse and search collections
- View floor prices and listings
- Purchase NFTs via OpenSea
- View your NFT portfolio
- Transfer NFTs
- Mint from supported platforms

**Reference**: See Advanced Reference Guide below (nft-operations)

### Polymarket Betting

- Search prediction markets
- Check odds
- Place bets on outcomes
- View positions
- Redeem winnings

**Reference**: See Advanced Reference Guide below (polymarket)

### Leverage Trading

- Long/short positions (up to 50x crypto, 100x forex/commodities)
- Crypto, forex, and commodities
- Stop loss and take profit
- Position management via Avantis on Base

**Reference**: See Advanced Reference Guide below (leverage-trading)

### Token Deployment

- **EVM (Base)**: Deploy ERC20 tokens via Clanker with customizable metadata and social links
- **Solana**: Launch SPL tokens via Raydium LaunchLab with bonding curve and auto-migration to CPMM
- Creator fee claiming on both chains
- Fee Key NFTs for Solana (50% LP trading fees post-migration)
- Optional fee recipient designation with 99.9%/0.1% split (Solana)
- Both creator AND fee recipient can claim bonding curve fees (gas sponsored)
- Optional vesting parameters (Solana)
- Rate limits: 1/day standard, 10/day Bankr Club (gas sponsored within limits)

**Reference**: See Advanced Reference Guide below (token-deployment)

### Automation

- Limit orders
- Stop loss orders
- DCA (dollar-cost averaging)
- TWAP (time-weighted average price)
- Scheduled commands

**Reference**: See Advanced Reference Guide below (automation)

### Arbitrary Transactions

- Submit raw EVM transactions with explicit calldata
- Custom contract calls to any address
- Execute pre-built calldata from other tools
- Value transfers with data

**Reference**: See Advanced Reference Guide below (arbitrary-transaction)

## Supported Chains

| Chain    | Native Token | Best For                      | Gas Cost |
| -------- | ------------ | ----------------------------- | -------- |
| Base     | ETH          | Memecoins, general trading    | Very Low |
| Polygon  | MATIC        | Gaming, NFTs, frequent trades | Very Low |
| Ethereum | ETH          | Blue chips, high liquidity    | High     |
| Solana   | SOL          | High-speed trading            | Minimal  |
| Unichain | ETH          | Newer L2 option               | Very Low |

## Safety & Access Control

**Dedicated Agent Wallet**: When building autonomous agents, create a separate Bankr account rather than using your personal wallet. This isolates agent funds — if a key is compromised, only the agent wallet is exposed. Fund it with limited amounts and replenish as needed.

**API Key Types**: Bankr uses a single key format (`bk_...`) with capability flags (`agentApiEnabled`, `llmGatewayEnabled`). You can optionally configure a separate LLM Gateway key via `bankr config set llmKey` or `BANKR_LLM_KEY` — useful when you want independent revocation or different permissions for agent vs LLM access.

**Read-Only API Keys**: Keys with `readOnly: true` filter all write tools (swaps, transfers, staking, token launches, etc.) from agent sessions. The `/agent/sign` and `/agent/submit` endpoints return 403. Ideal for monitoring bots and research agents.

**IP Whitelisting**: Set `allowedIps` on your API key to restrict usage to specific IPs. Requests from non-whitelisted IPs are rejected with 403 at the auth layer.

**Rate Limits**: 100 messages/day (standard), 1,000/day (Bankr Club), or custom per key. Resets 24h from first message (rolling window). LLM Gateway uses a credit-based system.

**Key safety rules:**
- Store keys in environment variables (`BANKR_API_KEY`, `BANKR_LLM_KEY`), never in source code
- Add `~/.bankr/` and `.env` to `.gitignore` — the CLI stores credentials in `~/.bankr/config.json`
- Test with small amounts on low-cost chains (Base, Polygon) before production use
- Use `waitForConfirmation: true` with `/agent/submit` — transactions execute immediately with no confirmation prompt
- Rotate keys periodically and revoke immediately if compromised at [bankr.bot/api](https://bankr.bot/api)

**Reference**: See Advanced Reference Guide below (safety)

## Common Patterns

### Check Before Trading

```bash
# Check balance
bankr prompt "What is my ETH balance on Base?"

# Check price
bankr prompt "What's the current price of PEPE?"

# Then trade
bankr prompt "Buy $20 of PEPE on Base"
```

### Portfolio Review

```bash
# Direct balance check (no AI agent, instant response)
bankr balances
bankr balances --chain base
bankr balances --chain base,solana
bankr balances --json

# Via AI agent (natural language, richer context)
bankr prompt "Show my complete portfolio"

# Chain-specific
bankr prompt "What tokens do I have on Base?"

# Token-specific
bankr prompt "Show my ETH across all chains"
```

### Set Up Automation

```bash
# DCA strategy
bankr prompt "DCA $100 into ETH every week"

# Stop loss protection
bankr prompt "Set stop loss for my ETH at $2,500"

# Limit order
bankr prompt "Buy ETH if price drops to $3,000"
```

### Market Research

```bash
# Price and analysis
bankr prompt "Do technical analysis on ETH"

# Trending tokens
bankr prompt "What tokens are trending on Base?"

# Compare tokens
bankr prompt "Compare ETH vs SOL"
```

## API Workflow

Bankr uses an asynchronous job-based API:

1. **Submit** — Send prompt (with optional `threadId`), get job ID and thread ID
2. **Poll** — Check status every 2 seconds
3. **Complete** — Process results when done
4. **Continue** — Reuse `threadId` for multi-turn conversations

The `bankr prompt` command handles this automatically. When using the REST API directly, implement the poll loop yourself (see Option 2 above or the reference below). For manual job control via CLI, use `bankr status <jobId>` and `bankr cancel <jobId>`.

For details on the API structure, job states, polling strategy, and error handling, see:

**Reference**: See Advanced Reference Guide below (api-workflow)

### Synchronous Endpoints

For direct signing and transaction submission, Bankr also provides synchronous endpoints:

- **POST /agent/sign** - Sign messages, typed data, or transactions without broadcasting
- **POST /agent/submit** - Submit raw transactions directly to the blockchain

These endpoints return immediately (no polling required) and are ideal for:
- Authentication flows (sign messages)
- Gasless approvals (sign EIP-712 permits)
- Pre-built transactions (submit raw calldata)

**Reference**: See Advanced Reference Guide below (sign-submit-api)

## Error Handling

Common issues and fixes:

- **Authentication errors** → Run `bankr login` or check `bankr whoami` (CLI), or verify your `X-API-Key` header (REST API)
- **Insufficient balance** → Add funds or reduce amount
- **Token not found** → Verify symbol and chain
- **Transaction reverted** → Check parameters and balances
- **Rate limiting** → Wait and retry

For comprehensive error troubleshooting, setup instructions, and debugging steps, see:

**Reference**: See Advanced Reference Guide below (error-handling)

## Best Practices

### Security

1. Never share your API key or LLM key
2. Use a dedicated agent wallet with limited funds for autonomous agents
3. Use read-only API keys for monitoring and research-only agents
4. Set IP whitelisting for server-side agents with known IPs
5. Verify addresses before large transfers
6. Use stop losses for leverage trading
7. Store keys in environment variables, not source code — add `~/.bankr/` to `.gitignore`

See the Advanced Reference Guide below (safety) for comprehensive safety guidance.

### Trading

1. Check balance before trades
2. Specify chain for lesser-known tokens
3. Consider gas costs (use Base/Polygon for small amounts)
4. Start small, scale up after testing
5. Use limit orders for better prices

### Automation

1. Test automation with small amounts first
2. Review active orders regularly
3. Set realistic price targets
4. Always use stop loss for leverage
5. Monitor execution and adjust as needed

## Tips for Success

### For New Users

- Start with balance checks and price queries
- Test with $5-10 trades first
- Use Base for lower fees
- Enable trading confirmations initially
- Learn one feature at a time

### For Experienced Users

- Leverage automation for strategies
- Use multiple chains for diversification
- Combine DCA with stop losses
- Explore advanced features (leverage, Polymarket)
- Monitor gas costs across chains

## Prompt Examples by Category

### Trading

- "Buy $50 of ETH on Base"
- "Swap 0.1 ETH for USDC"
- "Sell 50% of my PEPE"
- "Bridge 100 USDC from Polygon to Base"

### Portfolio

- `bankr balances` (direct, no AI processing)
- `bankr balances --chain base` (single chain)
- "Show my portfolio"
- "What's my ETH balance?"
- "Total portfolio value"
- "Holdings on Base"

### Market Research

- "What's the price of Bitcoin?"
- "Analyze ETH price"
- "Trending tokens on Base"
- "Compare UNI vs SUSHI"

### Transfers

- "Send 0.1 ETH to vitalik.eth"
- "Transfer $20 USDC to @friend"
- "Send 50 USDC to 0x123..."

### NFTs

- "Show Bored Ape floor price"
- "Buy cheapest Pudgy Penguin"
- "Show my NFTs"

### Polymarket

- "What are the odds Trump wins?"
- "Bet $10 on Yes for [market]"
- "Show my Polymarket positions"

### Leverage

- "Open 5x long on ETH with $100"
- "Short BTC 10x with stop loss at $45k"
- "Show my Avantis positions"

### Automation

- "DCA $100 into ETH weekly"
- "Set limit order to buy ETH at $3,000"
- "Stop loss for all holdings at -20%"

### Token Deployment

**Solana (LaunchLab):**

- "Launch a token called MOON on Solana"
- "Launch a token called FROG and give fees to @0xDeployer"
- "Deploy SpaceRocket with symbol ROCK"
- "Launch BRAIN and route fees to 7xKXtg..."
- "How much fees can I claim for MOON?"
- "Claim my fees for MOON" (works for creator or fee recipient)
- "Show my Fee Key NFTs"
- "Claim my fee NFT for ROCKET" (post-migration)
- "Transfer fees for MOON to 7xKXtg..."

**EVM (Clanker):**

- "Deploy a token called BankrFan with symbol BFAN on Base"
- "Claim fees for my token MTK"

### Arbitrary Transactions

- "Submit this transaction: {to: 0x..., data: 0x..., value: 0, chainId: 8453}"
- "Execute this calldata on Base: {...}"
- "Send raw transaction with this JSON: {...}"

### Sign API (Synchronous)

Direct message signing without AI processing:

```bash
# Sign a plain text message
curl -X POST "https://api.bankr.bot/agent/sign" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"signatureType": "personal_sign", "message": "Hello, Bankr!"}'

# Sign EIP-712 typed data (permits, orders)
curl -X POST "https://api.bankr.bot/agent/sign" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"signatureType": "eth_signTypedData_v4", "typedData": {...}}'

# Sign a transaction without broadcasting
curl -X POST "https://api.bankr.bot/agent/sign" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"signatureType": "eth_signTransaction", "transaction": {"to": "0x...", "chainId": 8453}}'
```

### Submit API (Synchronous)

Direct transaction submission without AI processing:

```bash
# Submit a raw transaction
curl -X POST "https://api.bankr.bot/agent/submit" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {"to": "0x...", "chainId": 8453, "value": "1000000000000000000"},
    "waitForConfirmation": true
  }'
```

**Reference**: See Advanced Reference Guide below (sign-submit-api)

## Resources

- **Documentation**: https://docs.bankr.bot
- **LLM Gateway Docs**: https://docs.bankr.bot/llm-gateway/overview
- **API Key Management**: https://bankr.bot/api
- **Terminal**: https://bankr.bot/terminal
- **CLI Package**: https://www.npmjs.com/package/@bankr/cli
- **Twitter**: @bankr_bot

## Troubleshooting

### CLI Not Found

```bash
# Verify installation
which bankr

# Reinstall if needed
bun install -g @bankr/cli
```

### Authentication Issues

**CLI:**
```bash
# Check current auth
bankr whoami

# Re-authenticate
bankr login

# Check LLM key specifically
bankr config get llmKey
```

**REST API:**
```bash
# Test your API key
curl -s "https://api.bankr.bot/_health" -H "X-API-Key: $BANKR_API_KEY"
```

### API Errors

See the Advanced Reference Guide below (error-handling) for comprehensive troubleshooting.

### Getting Help

1. Check error message in CLI output or API response
2. Run `bankr whoami` to verify auth (CLI) or test with a curl to `/_health` (REST API)
3. Consult relevant reference document
4. Test with simple queries first (`bankr prompt "What is my balance?"` or `POST /agent/prompt`)

---

**Pro Tip**: The most common issue is not specifying the chain for tokens. When in doubt, always include "on Base" or "on Ethereum" in your prompt.

**Security**: Keep your API key private. Never commit your config file to version control. Only trade amounts you can afford to lose.

**Quick Win**: Start by checking your portfolio (`bankr prompt "Show my portfolio"`) to see what's possible, then try a small $5-10 trade on Base to get familiar with the flow.


---

# 📚 Advanced Reference Guide

> The following sections were inlined from the former `references/` directory.



## 📖 Reference: api-workflow.md

# Bankr API Workflow Reference

Understanding the asynchronous job pattern for Bankr API operations.

**Source**: [Agent API Reference](https://www.notion.so/Agent-API-2e18e0f9661f80cb83ccfc046f8872e3)

## Using the Bankr CLI

The CLI handles submit-poll-complete automatically. For installation and login, see the main [SKILL.md](../SKILL.md).

```bash
bankr prompt "What is my ETH balance?"   # submit + poll + display
bankr status <jobId>                      # check a specific job
bankr cancel <jobId>                      # cancel a running job
```

## Using the REST API Directly

Call the endpoints below with `curl`, `fetch`, or any HTTP client. All requests require an `X-API-Key` header.

### Core Pattern: Submit-Poll-Complete

All operations follow this pattern:

```
1. SUBMIT  → Send prompt, get job ID
2. POLL    → Check status every 2s
3. COMPLETE → Process results
```

## API Endpoints

### POST /agent/prompt
Submit a natural language prompt to start a job.

**CLI equivalent:** `bankr prompt "What is my ETH balance?"`

**Request:**
```bash
curl -X POST "https://api.bankr.bot/agent/prompt" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is my ETH balance?"}'
```

**Continue a conversation:**
```bash
curl -X POST "https://api.bankr.bot/agent/prompt" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "And what about SOL?", "threadId": "thr_ABC123"}'
```

**Request Body:**
- **prompt** (string, required): The prompt to send to the AI agent (max 10,000 characters)
- **threadId** (string, optional): Continue an existing conversation thread. If omitted, a new thread is created.

**Response (202 Accepted):**
```json
{
  "success": true,
  "jobId": "job_abc123",
  "threadId": "thr_XYZ789",
  "status": "pending",
  "message": "Job submitted successfully"
}
```

**Error Responses:**

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `Invalid request` or `Prompt too long` | Bad input or exceeds 10,000 chars |
| 401 | `Authentication required` | Missing or invalid API key |
| 403 | `Agent API access not enabled` | API key lacks agent access |

### GET /agent/job/{jobId}
Check job status and results.

**CLI equivalent:** `bankr status job_abc123`

**Request:**
```bash
curl -X GET "https://api.bankr.bot/agent/job/job_abc123" \
  -H "X-API-Key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "success": true,
  "jobId": "job_abc123",
  "threadId": "thr_XYZ789",
  "status": "completed",
  "prompt": "What is my ETH balance?",
  "response": "You have 0.5 ETH worth approximately $1,825.",
  "richData": [],
  "statusUpdates": [
    {"message": "Checking balances...", "timestamp": "2025-01-26T10:00:00Z"},
    {"message": "Calculating USD values...", "timestamp": "2025-01-26T10:00:02Z"}
  ],
  "createdAt": "2025-01-26T10:00:00Z",
  "completedAt": "2025-01-26T10:00:03Z",
  "processingTime": 3000
}
```

**Error Responses:**

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `Job ID required` | Missing job ID in path |
| 401 | `Authentication required` | Missing or invalid API key |
| 404 | `Job not found` | Job ID doesn't exist or doesn't belong to you |

### POST /agent/job/{jobId}/cancel
Cancel a pending or processing job. Cancel requests are idempotent — cancelling an already-cancelled job returns success.

**CLI equivalent:** `bankr cancel job_abc123`

**Request:**
```bash
curl -X POST "https://api.bankr.bot/agent/job/job_abc123/cancel" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**
```json
{
  "success": true,
  "jobId": "job_abc123",
  "status": "cancelled",
  "prompt": "Swap 0.1 ETH for USDC",
  "createdAt": "2025-01-26T10:00:00Z",
  "cancelledAt": "2025-01-26T10:00:05Z"
}
```

**Error Responses:**

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `Job ID required`, `Job already completed`, or `Job already failed` | Invalid state for cancellation |
| 401 | `Authentication required` | Missing or invalid API key |
| 404 | `Job not found` | Job ID doesn't exist or doesn't belong to you |

## Job Status States

| Status | Description | Action |
|--------|-------------|--------|
| `pending` | Job queued, not yet started | Keep polling |
| `processing` | Job is being processed | Keep polling, show updates |
| `completed` | Job finished successfully | Read response and richData |
| `failed` | Job encountered an error | Check error field |
| `cancelled` | Job was cancelled | No further action |

## Response Fields

### Standard Fields
- **success**: Boolean, true if request succeeded
- **jobId**: Unique job identifier
- **threadId**: Conversation thread ID (reuse to continue the conversation)
- **status**: Current job status (`pending`, `processing`, `completed`, `failed`, `cancelled`)
- **prompt**: Original user prompt
- **createdAt**: ISO 8601 timestamp

### Success Fields (status=completed)
- **response**: Natural language response text
- **richData**: Array of structured data objects (see Rich Data below)
- **completedAt**: When job finished (ISO 8601)
- **processingTime**: Duration in milliseconds

### Progress Fields (status=processing)
- **statusUpdates**: Array of progress messages (`{message, timestamp}`)
- **startedAt**: When processing began (ISO 8601)
- **cancellable**: Boolean, whether the job can still be cancelled

### Error Fields (status=failed)
- **error**: Error message
- **completedAt**: When failure occurred (ISO 8601)

### Cancelled Fields (status=cancelled)
- **cancelledAt**: When the job was cancelled (ISO 8601)

## Rich Data Objects

Completed jobs may include a `richData` array containing structured data (e.g., token info, price quotes, charts). Each entry has:

```typescript
type RichData = {
  type?: string;          // e.g., "token_info", "price_quote"
  [key: string]: unknown; // Additional structured data
};
```

The exact shape depends on the operation performed. The `response` field always contains a human-readable text summary regardless of what `richData` contains.

## Polling Strategy

### Timing
- **Interval**: 2 seconds between requests
- **Typical duration**: 30 seconds to 2 minutes
- **Maximum**: 5 minutes (then suggest cancellation)

### Example Polling Loop
```bash
#!/bin/bash
JOB_ID="job_abc123"
MAX_ATTEMPTS=150  # 5 minutes

for i in $(seq 1 $MAX_ATTEMPTS); do
    sleep 2
    STATUS=$(curl -s "https://api.bankr.bot/agent/job/$JOB_ID" \
        -H "X-API-Key: $API_KEY" | jq -r '.status')

    case "$STATUS" in
        completed|failed|cancelled)
            break
            ;;
        *)
            echo "Polling... ($i/$MAX_ATTEMPTS)"
            ;;
    esac
done
```

## Status Update Handling

**Track what you've shown:**
```bash
LAST_UPDATE_COUNT=0

while true; do
    RESULT=$(get_job_status "$JOB_ID")
    CURRENT_COUNT=$(echo "$RESULT" | jq '.statusUpdates | length')

    if [ "$CURRENT_COUNT" -gt "$LAST_UPDATE_COUNT" ]; then
        # Show new updates only
        echo "$RESULT" | jq -r ".statusUpdates[$LAST_UPDATE_COUNT:] | .[].message"
        LAST_UPDATE_COUNT=$CURRENT_COUNT
    fi

    STATUS=$(echo "$RESULT" | jq -r '.status')
    [ "$STATUS" != "pending" ] && [ "$STATUS" != "processing" ] && break

    sleep 2
done
```

## Output Guidelines

### Response Formatting

**Price queries:**
- Clear, direct answer
- Include symbol and price
- Example: "ETH is currently $3,245.67"

**Trading confirmations:**
- Confirm amounts
- Show transaction hash
- Mention gas costs if significant

**Portfolio display:**
- List token amounts and USD values
- Group by chain if multi-chain
- Show total portfolio value

**Market analysis:**
- Summarize key insights
- Highlight important data points
- Keep concise

**Errors:**
- Explain what went wrong clearly
- Suggest next steps
- Avoid technical jargon

## Error Handling

### Authentication Errors (401)
```json
{
  "error": "Authentication required",
  "message": "API key is missing or invalid"
}
```

**Resolution**: Check API key, ensure "Agent API" access is enabled at https://bankr.bot/api

### Forbidden (403)
```json
{
  "error": "Agent API access not enabled",
  "message": "Enable agent access for your API key"
}
```

**Resolution**: Visit https://bankr.bot/api and enable Agent API access on your key

### Rate Limiting (429)
```json
{
  "error": "Rate limit exceeded",
  "message": "Retry after 60 seconds"
}
```

**Resolution**: Wait and retry, implement exponential backoff

### Job Failures
```json
{
  "success": true,
  "status": "failed",
  "error": "Insufficient balance for trade"
}
```

**Resolution**: Check specific error, guide user to fix

## Best Practices

### Submission
1. **Validate input** before submitting
2. **Handle errors** gracefully
3. **Store job ID** for tracking
4. **Show confirmation** to user

### Polling
1. **Use 2-second interval** - Don't poll too fast
2. **Show progress** - Display status updates
3. **Set timeout** - Don't poll forever
4. **Handle network errors** - Retry with backoff

### Completion
1. **Parse response** carefully
2. **Check richData** for structured results
3. **Format output** nicely
4. **Save to history** for reference

### Error Recovery
1. **Identify error type** quickly
2. **Provide clear explanation** to user
3. **Suggest fixes** when possible
4. **Retry** intelligently

## Example Workflows

### Simple Balance Check
```
1. Submit: "What is my ETH balance?"
2. Poll every 2s (completes in ~5s)
3. Show: "You have 0.5 ETH ($1,825)"
```

### Token Swap
```
1. Submit: "Swap 0.1 ETH for USDC"
2. Poll every 2s
   - Update: "Checking balance..."
   - Update: "Finding best route..."
   - Update: "Executing swap..."
3. Complete after ~45s
4. Show: "Swapped 0.1 ETH for 365 USDC"
5. Display transaction hash
```

### Market Research
```
1. Submit: "Analyze ETH price"
2. Poll every 2s
   - Update: "Fetching price data..."
   - Update: "Running technical analysis..."
   - Update: "Analyzing sentiment..."
3. Complete after ~30s
4. Show formatted analysis with key metrics
```

## Security Notes

### API Key
- Never expose in client code
- Use environment variables or config.json
- Rotate periodically
- Monitor usage
- Revoke immediately if leaked at https://bankr.bot/api

### Validation
- Validate user input
- Sanitize prompts
- Check amounts/addresses
- Confirm before critical operations

### Error Messages
- Don't leak sensitive info
- Be helpful but not revealing
- Log internally, show safely
- Guide users constructively

---

**Remember**: The asynchronous pattern allows Bankr to handle complex operations that may take time, while keeping you informed of progress.



## 📖 Reference: arbitrary-transaction.md

# Arbitrary Transaction Reference

Submit raw EVM transactions with explicit calldata to any supported chain.

## Supported Chains

| Chain | Chain ID |
|-------|----------|
| Ethereum | 1 |
| Polygon | 137 |
| Base | 8453 |
| Unichain | 130 |

## JSON Format

```json
{
  "to": "0x...",
  "data": "0x...",
  "value": "0",
  "chainId": 8453
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `to` | string | Yes | Target contract address (0x + 40 hex chars) |
| `data` | string | Yes | Calldata to execute (0x + hex string, or "0x" for empty) |
| `value` | string | Yes | Amount in wei (e.g., "0", "1000000000000000000" for 1 ETH) |
| `chainId` | number | Yes | Target chain ID (1, 137, 8453, or 130) |

## Validation Rules

| Field | Validation |
|-------|------------|
| `to` | Must be 0x followed by exactly 40 hex characters |
| `data` | Must start with 0x, can be "0x" for empty calldata |
| `value` | Wei amount as string, use "0" for no value transfer |
| `chainId` | Must be a supported chain ID |

## Prompt Examples

**Submit a raw transaction:**
```
Submit this transaction:
{
  "to": "0x1234567890abcdef1234567890abcdef12345678",
  "data": "0xa9059cbb000000000000000000000000recipient00000000000000000000000000000000000000000000000000000000000f4240",
  "value": "0",
  "chainId": 8453
}
```

**Execute calldata on a contract:**
```
Execute this calldata on Base:
{
  "to": "0xContractAddress...",
  "data": "0xFunctionSelector...",
  "value": "0",
  "chainId": 8453
}
```

**Send ETH with calldata:**
```
Submit transaction with value:
{
  "to": "0xRecipientAddress...",
  "data": "0x",
  "value": "1000000000000000000",
  "chainId": 1
}
```

**ERC-20 transfer via calldata:**
```
Submit this ERC-20 transfer:
{
  "to": "0xTokenContractAddress...",
  "data": "0xa9059cbb000000000000000000000000...",
  "value": "0",
  "chainId": 8453
}
```

## Common Issues

| Issue | Resolution |
|-------|------------|
| Unsupported chain | Use chainId 1, 137, 8453, or 130 |
| Invalid address | Ensure 0x + 40 hex chars |
| Invalid calldata | Ensure proper hex encoding with 0x prefix |
| Transaction reverted | Check calldata encoding and contract state |
| Insufficient funds | Ensure wallet has enough ETH/MATIC for gas + value |

## Use Cases

- **Custom contract interactions** - Call any function on any contract
- **Pre-built calldata execution** - Execute calldata generated by other tools
- **Advanced DeFi operations** - Complex multi-step transactions
- **Protocol integrations** - Interact with protocols not yet natively supported

## Best Practices

1. **Verify calldata** - Double-check encoding before submission
2. **Test on testnet first** - If possible, test transactions on testnets
3. **Start with zero value** - Test contract calls without sending ETH first
4. **Check gas estimates** - Ensure sufficient balance for gas costs
5. **Verify contract addresses** - Confirm target address is correct

## Security Notes

- **Irreversible** - Blockchain transactions cannot be undone
- **Verify everything** - Calldata determines exactly what happens
- **Trust the source** - Only execute calldata from trusted sources
- **Check value field** - Ensure you're not sending unintended ETH
- **Contract verification** - Confirm the target contract is legitimate



## 📖 Reference: automation.md

# Automation Reference

Set up automated orders and scheduled trading strategies.

## Order Types

### Limit Orders
Execute trade when price reaches target.

**Examples:**
- "Set a limit order to buy ETH at $3,000"
- "Limit order: sell BNKR when it hits $0.02"
- "Buy 1 SOL if price drops to $100"
- "Sell my PEPE at $0.000015"

**Use cases:**
- Buy the dip
- Take profit at target
- Enter at better price
- Exit at predetermined level

### Stop Loss Orders
Automatically sell to limit losses.

**Examples:**
- "Set stop loss for my ETH at $2,500"
- "Stop loss: sell 50% of BNKR if it drops 20%"
- "Protect my SOL position with stop at $150"

**Use cases:**
- Protect gains
- Limit downside
- Risk management
- Sleep peacefully

### DCA (Dollar Cost Averaging)
Invest fixed amounts at regular intervals.

**Examples:**
- "DCA $100 into ETH every week"
- "Set up daily $50 Bitcoin purchases"
- "Buy $25 of SOL every Monday"
- "Monthly $500 DCA into ETH"

**Use cases:**
- Reduce timing risk
- Build position over time
- Smooth out volatility
- Disciplined investing

**Intervals:**
- Hourly
- Daily
- Weekly
- Monthly

### TWAP (Time-Weighted Average Price)
Spread large orders over time to reduce market impact.

**Examples:**
- "TWAP: buy $1000 of ETH over 24 hours"
- "Spread my sell order over 4 hours"
- "Buy 10 ETH using TWAP over next 6 hours"
- "TWAP sell 50% of position over 12 hours"

**Use cases:**
- Large order execution
- Reduce slippage
- Minimize market impact
- Better average price

### Scheduled Commands
Run any Bankr command on a schedule.

**Examples:**
- "Every morning at 8am, check my portfolio"
- "Daily at 9am, check ETH price"
- "Every Monday, show me trending tokens"
- "Hourly, check my open positions"

**Use cases:**
- Regular monitoring
- Automated reporting
- Price alerts
- Balance checks

## Managing Automations

### View Active Automations
**Examples:**
- "Show my automations"
- "What limit orders do I have?"
- "List my active DCAs"
- "Show all scheduled commands"

**Information shown:**
- Order type
- Asset/pair
- Trigger condition
- Amount
- Status
- Created date

### Cancel Automations
**Examples:**
- "Cancel my ETH limit order"
- "Stop my DCA into Bitcoin"
- "Cancel all my stop losses"
- "Remove my SOL automation"

**Cancellation:**
- Immediate effect
- No fees for canceling
- Can recreate anytime
- To modify an automation, cancel and recreate with new parameters

## Chain Support

### EVM Chains (Base, Polygon, Ethereum)
- ✅ Limit orders
- ✅ Stop loss
- ✅ DCA
- ✅ TWAP
- ✅ Scheduled commands

### Solana
- ✅ Limit orders (via Jupiter Trigger)
- ✅ Stop loss (via Jupiter)
- ✅ DCA (via Jupiter)
- ⚠️ TWAP (limited support)
- ✅ Scheduled commands

## Common Issues

| Issue | Resolution |
|-------|------------|
| Order not triggering | Check price hasn't already passed |
| Insufficient balance | Ensure funds available when executes |
| Order cancelled | May have conflicting orders |
| Slippage on trigger | Market moved quickly |
| DCA not executing | Check balance and gas funds |

## Best Practices

### Setting Up
1. **Start small** - Test with small amounts
2. **Clear conditions** - Be specific about triggers
3. **Check balance** - Ensure funds available
4. **Set alerts** - Get notified on execution
5. **Review regularly** - Update as market changes

### Risk Management
1. **Always use stop loss** - Especially for leverage
2. **Don't set and forget** - Monitor periodically
3. **Adjust targets** - Update as conditions change
4. **Consider gas** - Factor in execution costs
5. **Test first** - Try one small automation first

### DCA Strategy
1. **Consistent amounts** - Stick to the plan
2. **Long timeframe** - At least 3-6 months
3. **Don't pause** - Keep going through volatility
4. **Review quarterly** - Adjust if needed
5. **Compound** - Consider increasing over time

### Limit Order Strategy
1. **Realistic prices** - Check historical support/resistance
2. **Layered orders** - Multiple orders at different prices
3. **Time limits** - Set expiration if needed
4. **Review daily** - Cancel stale orders
5. **Be patient** - Good prices take time

## Tips for Success

### Automation is Not Set-and-Forget
- Check on orders regularly
- Market conditions change
- Update targets as needed
- Cancel outdated orders

### Combine Strategies
- DCA + stop loss = Protected accumulation
- Limit buy + limit sell = Range trading
- TWAP + stop loss = Large position exit

### Use Alerts
- Get notified on execution
- Track automation performance
- Stay informed without constant checking
- Review execution prices

### Keep It Simple
- Start with basic automations
- Add complexity gradually
- Don't over-automate
- Clear naming for tracking

## Cost Considerations

### Execution Costs
- Gas fees on each trigger
- Higher on Ethereum mainnet
- Very low on Base/Polygon
- Factor into profit calculations

### DCA Costs
- Per-transaction gas
- Can add up with frequent DCA
- Daily DCA = 365 transactions/year
- Weekly might be more efficient

### TWAP Costs
- Multiple transactions
- Total gas = intervals × gas per tx
- Balance cost vs slippage savings

## Examples by Use Case

### Build Long-Term Position
```
"DCA $200 into ETH every week for 6 months"
"Set stop loss at -30% to protect"
"Monthly review of strategy"
```

### Take Profit Strategy
```
"Limit order: sell 25% at $4,000"
"Limit order: sell 25% at $4,500"
"Limit order: sell 50% at $5,000"
```

### Downside Protection
```
"Stop loss at -20% for all holdings"
"Set stop loss for my ETH at $2,500"
"Stop loss: sell 50% of BNKR if it drops 20%"
```

### Opportunistic Buying
```
"Buy $100 ETH if drops to $2,500"
"Buy $200 ETH if drops to $2,000"
"Buy $500 ETH if drops to $1,500"
```

## Monitoring & Adjusting

### Weekly Reviews
- Check execution history
- Adjust price targets
- Cancel outdated orders
- Review performance

### Monthly Analysis
- Calculate ROI
- Compare to manual trading
- Adjust DCA amounts
- Refine strategy

### Quarterly Rebalance
- Reassess allocations
- Update long-term targets
- Modify automation strategy
- Consider market changes

---

**Remember**: Automation is a tool to execute your strategy, not a replacement for strategy. Regular review and adjustment is key to success.



## 📖 Reference: error-handling.md

# Error Handling Reference

Resolve Bankr API errors and common issues.

## Authentication Errors

### Symptoms
- HTTP 401 status code
- "Invalid API key" or "Unauthorized" message
- "X-API-Key header is required"

### Resolution Steps

**1. Install the Bankr CLI**
```bash
bun install -g @bankr/cli
```

**2. Authenticate**
```bash
bankr login
```

Or if you already have an API key from https://bankr.bot/api:
```bash
bankr config set apiKey bk_your_actual_key_here
```

**3. Verify Setup**
```bash
bankr whoami
bankr prompt "What is my balance?"
```

### Common API Key Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| "Invalid API key" | Wrong key or revoked | Generate new key |
| "Agent API not enabled" | Missing permission | Enable in API dashboard |
| "API key expired" | Old/inactive key | Create new key |
| "Rate limit exceeded" | Too many requests | Wait or upgrade plan |

## Job Failures

### Transaction Failures

**Insufficient Balance**
- **Error**: "Insufficient balance for trade"
- **Cause**: Not enough tokens or gas
- **Fix**: Add funds or reduce amount

**Token Not Found**
- **Error**: "Token not found on [chain]"
- **Cause**: Wrong symbol, chain, or address
- **Fix**: Verify token exists, specify chain, use contract address

**Slippage Exceeded**
- **Error**: "Slippage tolerance exceeded"
- **Cause**: Price moved too much during execution
- **Fix**: Retry, increase slippage, or use smaller amount

**Transaction Reverted**
- **Error**: "Transaction reverted"
- **Cause**: On-chain failure (various reasons)
- **Fix**: Check transaction details, verify parameters

**Network Congestion**
- **Error**: "Network congestion, transaction failed"
- **Cause**: High network activity
- **Fix**: Increase gas, wait, or try L2

### Market/Query Failures

**Market Not Found**
- **Error**: "Polymarket market not found"
- **Cause**: Market closed, wrong search terms
- **Fix**: Try different search, check if market exists

**NFT Not Available**
- **Error**: "NFT listing no longer available"
- **Cause**: NFT was sold to someone else
- **Fix**: Try another listing, check floor price

**Rate Limit**
- **Error**: "Rate limit exceeded"
- **Cause**: Too many requests in short time
- **Fix**: Wait 60 seconds, implement backoff

**Timeout**
- **Error**: "Job timed out"
- **Cause**: Operation took too long
- **Fix**: Simplify query, retry, or check service status

## HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| **400** | Bad request | Check prompt format, validate parameters |
| **401** | Unauthorized | Fix API key (see Authentication section) |
| **402** | Payment required | Ensure wallet has BNKR on Base for fees |
| **403** | Forbidden | Agent API access not enabled — enable at https://bankr.bot/api |
| **429** | Rate limited | Wait and retry with exponential backoff |
| **500** | Server error | Retry after delay |
| **502** | Bad gateway | Temporary issue, retry after delay |
| **503** | Service unavailable | Service maintenance, retry later |

## Network/Connection Errors

### Symptoms
- "Failed to connect"
- "Network error"
- "Timeout"
- "Connection refused"

### Troubleshooting

**Check Internet Connection**
```bash
ping -c 3 api.bankr.bot
```

**Verify API Endpoint**
```bash
curl -I https://api.bankr.bot
```

**Check DNS Resolution**
```bash
nslookup api.bankr.bot
```

**Test with curl**
```bash
curl -sf https://api.bankr.bot || echo "Connection failed"
```

## Balance/Funding Issues

### Insufficient Native Token
**Symptoms**:
- "Insufficient ETH for gas"
- "Not enough MATIC for transaction"
- "Insufficient SOL for fees"

**Fix**:
- Add native token to wallet
- Amounts needed:
  - Ethereum: 0.01-0.05 ETH
  - Base: 0.001-0.01 ETH
  - Polygon: 1-5 MATIC
  - Solana: 0.01-0.1 SOL

### Insufficient Token Balance
**Symptoms**:
- "Insufficient [TOKEN] balance"
- "Balance too low for trade"

**Fix**:
- Check balance first
- Reduce trade amount
- Add more tokens

## Configuration Issues

### CLI Not Installed
```bash
# Install the Bankr CLI
bun install -g @bankr/cli

# Or with npm
npm install -g @bankr/cli

# Verify installation
which bankr
```

### Not Authenticated
```bash
# Authenticate (opens browser for email/OTP flow)
bankr login

# Or set API key directly
bankr config set apiKey bk_your_key_here

# Set separate LLM key (optional, falls back to API key)
bankr config set llmKey your_llm_key_here

# Verify
bankr whoami
```

Config is stored at `~/.bankr/config.json`. View current values with `bankr config get`.

### REST API Authentication
If using the API directly without the CLI, test your key with:
```bash
curl -s "https://api.bankr.bot/_health" -H "X-API-Key: $BANKR_API_KEY"
```
Set `BANKR_API_KEY` (and optionally `BANKR_LLM_KEY` for the LLM gateway) as environment variables.

## User-Friendly Error Messages

### Template
```
[What went wrong]

This usually means: [Explanation]

To fix this:
1. [Step 1]
2. [Step 2]
3. [Step 3]

Need help? Visit https://bankr.bot/api
```

### Examples

**Balance Error:**
```
You don't have enough ETH to complete this trade.

This usually means: Your wallet balance is too low for the trade amount plus gas fees.

To fix this:
1. Check your balance: "What is my ETH balance?"
2. Either reduce the trade amount
3. Or add more ETH to your wallet

You currently need at least $XX.XX worth of ETH.
```

**Token Not Found:**
```
Couldn't find the token "XYZ" on Base.

This usually means: The token symbol is wrong, the token doesn't exist on this chain, or it hasn't been indexed yet.

To fix this:
1. Double-check the token symbol spelling
2. Try specifying the chain: "Buy XYZ on Ethereum"
3. Or use the contract address instead

Try: "Search for XYZ token" to find it
```

## Debugging Checklist

Before reporting an issue, check:

- [ ] API key is set and correct
- [ ] Config file exists and has valid JSON
- [ ] Internet connection is working
- [ ] api.bankr.bot is reachable
- [ ] Wallet has sufficient balance (tokens + gas)
- [ ] Token/market exists on specified chain
- [ ] Command syntax is correct
- [ ] No typos in token symbols or addresses
- [ ] Recent similar operations worked

## Getting Help

### Check Status
```bash
# Verify authentication
bankr whoami

# Test with a simple query
bankr prompt "What is my balance?"
```

### Gather Information
When reporting issues, include:
- Error message (exact text)
- Command that failed
- Job ID (if available)
- Timestamp
- Chain and tokens involved
- Your config (without API key)

### Resources
- **Agent API Reference**: https://www.notion.so/Agent-API-2e18e0f9661f80cb83ccfc046f8872e3
- **API Key Management**: https://bankr.bot/api
- **Twitter**: @bankr_bot
- **Telegram**: @bankr_ai_bot

## Prevention

### Before Operating
1. **Test with small amounts** first
2. **Verify balance** before trades
3. **Check token exists** on chain
4. **Confirm parameters** are correct
5. **Have enough gas** for transactions

### Best Practices
1. Start small and test
2. Keep some native token for gas
3. Verify addresses/symbols
4. Use limit orders for better prices
5. Monitor your automations
6. Review transactions before confirming
7. Keep API key secure

### Regular Maintenance
1. Check balance weekly
2. Review open orders monthly
3. Update automation rules
4. Monitor gas costs
5. Keep config backed up

## Common Mistake Patterns

### Wrong Chain
- **Mistake**: "Buy TOKEN" (doesn't specify chain)
- **Result**: Token not found or wrong chain selected
- **Fix**: "Buy TOKEN on Base"

### Insufficient Gas Buffer
- **Mistake**: Using all ETH in trade
- **Result**: No gas for future transactions
- **Fix**: Keep 0.01 ETH buffer

### Typos in Symbols
- **Mistake**: "ETHE" instead of "ETH"
- **Result**: Token not found
- **Fix**: Double-check spelling

### Forgetting Decimals
- **Mistake**: "Buy 100 ETH" (wants $100 worth)
- **Result**: Tries to buy 100 ETH ($300,000+)
- **Fix**: "Buy $100 of ETH"

### No Stop Loss
- **Mistake**: Opening leverage without stop loss
- **Result**: Risk of liquidation
- **Fix**: Always set stop loss for leverage

## Error Recovery Workflow

```
1. Error occurs
   ↓
2. Read error message carefully
   ↓
3. Check this guide for known issue
   ↓
4. Apply suggested fix
   ↓
5. Test with small amount
   ↓
6. If still failing:
   - Verify config
   - Test API connectivity
   - Report issue with details
```

---

**Remember**: Most errors have simple fixes. Read the error message carefully, check the basics (API key, balance, connection), and consult this guide.



## 📖 Reference: leverage-trading.md

# Leverage Trading Reference

Trade with leverage using Avantis perpetuals on Base.

## Overview

Avantis offers long/short positions on crypto, forex, and commodities via perpetuals on Base.

**Chain**: Base
**Protocol**: [Avantis](https://docs.avantisfi.com/)

### Leverage Limits

| Asset Class | Max Leverage |
|-------------|-------------|
| Crypto | 50x |
| Forex | 100x |
| Commodities | 100x |

## Supported Markets

### Cryptocurrency
BTC, ETH, SOL, ARB, AVAX, BNB, DOGE, LINK, OP, MATIC

### Forex
- EUR/USD - Euro vs US Dollar
- GBP/USD - British Pound vs US Dollar
- USD/JPY - US Dollar vs Japanese Yen
- AUD/USD - Australian Dollar vs US Dollar
- USD/CAD - US Dollar vs Canadian Dollar

### Commodities
- XAU (Gold)
- XAG (Silver)
- WTI (Crude Oil)
- NATGAS (Natural Gas)

## Prompt Examples

**Open positions:**
- "Open a 5x long on ETH with $100"
- "Short Bitcoin with 10x leverage using $50"
- "Long Gold with 2x leverage"
- "Open 3x long SOL position"

**With risk management:**
- "Long ETH 5x with stop loss at $3000"
- "Short BTC 10x with take profit at 20%"
- "Long SOL 3x with SL at $150 and TP at $200"
- "5x long EUR/USD with stop loss at 1.08"

**View positions:**
- "Show my Avantis positions"
- "What leverage trades do I have open?"
- "My open positions"
- "PnL on my ETH long"

**Close positions:**
- "Close my ETH long"
- "Exit all my Avantis positions"
- "Close 50% of my BTC short"
- "Take profit on my SOL position"

## Position Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| **Leverage** | 1x to 50x crypto, 100x forex/commodities | "5x leverage" |
| **Collateral** | Amount to use as margin | "$100", "0.1 ETH" |
| **Direction** | Long (price up) or Short (price down) | "long", "short" |
| **Stop Loss** | Auto-close to limit losses | "stop loss at $3000" |
| **Take Profit** | Auto-close to lock in gains | "take profit at $4000" |

## How Leverage Works

### Long Position Example
- Open 5x long ETH with $100 at $3,000
- Effective position: $500 worth of ETH
- If ETH → $3,300 (+10%): Gain $50 (50% profit)
- If ETH → $2,700 (-10%): Lose $50 (50% loss)
- If ETH → $2,400 (-20%): Position liquidated

### Short Position Example
- Open 5x short BTC with $100 at $50,000
- If BTC → $45,000 (-10%): Gain $50 (50% profit)
- If BTC → $55,000 (+10%): Lose $50 (50% loss)
- If BTC → $60,000 (+20%): Position liquidated

## Leverage Guidelines

| Risk Level | Leverage | Use Case | Liquidation Risk |
|------------|----------|----------|------------------|
| Conservative | 1-3x | Long-term views | Low |
| Moderate | 3-10x | Swing trading | Medium |
| Aggressive | 10-25x | Short-term scalps | High |
| Extreme | 25x+ | Expert only | Very High |

## Liquidation

**What is liquidation?**
- Position is automatically closed
- Happens when losses approach collateral amount
- You lose all collateral for that position

**Liquidation Price Calculation:**
- Long position: Entry price × (1 - 1/leverage)
- Short position: Entry price × (1 + 1/leverage)

**Examples:**
- 5x long ETH at $3,000: Liquidated ~$2,400 (-20%)
- 10x short BTC at $50,000: Liquidated ~$55,000 (+10%)
- 2x long SOL at $100: Liquidated ~$50 (-50%)

## Risk Management

### Stop Loss (SL)
- Set maximum acceptable loss
- Closes position automatically
- Protects from larger losses
- **Recommended for all positions**

### Take Profit (TP)
- Set profit target
- Closes position when reached
- Locks in gains
- Good for planned exits

### Position Sizing
- Start with small amounts
- Risk only 1-5% of capital per trade
- Don't use full balance as collateral
- Keep reserve for other opportunities

## Funding Rates

**What are funding rates?**
- Periodic fee between longs and shorts
- Usually every 8 hours
- Can be positive or negative
- Affects profitability on long holds

**How they work:**
- If rate is positive: Longs pay shorts
- If rate is negative: Shorts pay longs
- Typically 0.01% - 0.1% per period

**Impact:**
- Short-term trades: Minimal
- Long holds: Can add up
- Check before opening position

## Common Issues

| Issue | Resolution |
|-------|------------|
| Insufficient collateral | Add more funds to wallet |
| Asset not supported | Check available markets list |
| Position liquidated | Reduce leverage or add more collateral |
| High funding rate | Consider closing and reopening |
| Slippage | Use smaller position size |
| Cannot close | Market might be paused (rare) |

## Advanced Strategies

### Hedging
- Open opposite position on same asset
- Protects against adverse moves
- Locks in current value

### Scaling In/Out
- Build position gradually
- Take partial profits
- Average entry price
- Manage risk dynamically

### Cross-Asset Arbitrage
- Long one asset, short related asset
- Example: Long ETH, short BTC
- Profit from spread changes
- Requires market knowledge

## Best Practices

### Before Opening Position
1. **Check price** - Confirm entry is good
2. **Set stop loss** - Decide max loss upfront
3. **Calculate liquidation** - Know your risk
4. **Check funding** - Consider costs
5. **Have a plan** - Know exit strategy

### While Position Open
1. **Monitor regularly** - Markets move fast
2. **Adjust stops** - Trail profits upward
3. **Watch funding** - Rates can change
4. **Stay informed** - Follow news
5. **Don't overtrade** - Be patient

### Closing Position
1. **Take profits** - Don't be greedy
2. **Cut losses** - Accept when wrong
3. **Learn** - Analyze what worked/didn't
4. **Record** - Keep trade journal
5. **Rest** - Don't immediately jump into next trade

## Risk Warnings

⚠️ **High Risk Activity**
- Can lose entire collateral quickly
- Leverage amplifies both gains and losses
- Liquidation is permanent
- Not suitable for beginners
- Only use money you can afford to lose

⚠️ **Market Volatility**
- Crypto is highly volatile
- Flash crashes can liquidate positions
- Weekend markets can be thin
- News can cause rapid moves

⚠️ **Technical Risks**
- Smart contract risk
- Oracle failures (rare)
- Network congestion
- Gas spikes on busy days

## Tips for Beginners

1. **Start small** - Test with $10-20
2. **Low leverage** - Use 2-3x maximum
3. **Always use stop loss** - Non-negotiable
4. **Close daily** - Don't hold overnight initially
5. **Paper trade first** - Practice without real money
6. **Learn from losses** - They will happen
7. **Don't revenge trade** - Take breaks after losses

## Tips for Experienced Traders

1. **Manage multiple positions** - Diversify leverage exposure
2. **Use TP/SL ratios** - 2:1 or 3:1 reward:risk minimum
3. **Consider funding** - Factor into long-term holds
4. **Scale positions** - Don't go all-in at once
5. **Hedge strategically** - Use shorts to protect longs
6. **Monitor correlation** - Related assets move together
7. **Take regular profits** - Lock in winners

## Resources

- **Avantis Documentation**: https://docs.avantisfi.com/
- **Funding Rate History**: Plan long-term holds
- **Market Statistics**: Analyze trading data
- **Liquidation Calculator**: Estimate risk

---

**Remember**: Leverage trading is a tool, not a get-rich-quick scheme. Most traders lose money. Start small, learn continuously, and never risk more than you can afford to lose.



## 📖 Reference: llm-gateway.md

# LLM Gateway Reference

The Bankr LLM Gateway is a unified API for Claude, Gemini, GPT, and other models. It provides multi-provider access, cost tracking, automatic failover, and SDK compatibility through a single endpoint.

**Base URL:** `https://llm.bankr.bot`

The gateway accepts both `https://llm.bankr.bot` and `https://llm.bankr.bot/v1` — it normalizes paths automatically. Works with both OpenAI and Anthropic API formats.

## Authentication

The gateway uses your **LLM key** for authentication. The key resolution order:

1. `BANKR_LLM_KEY` environment variable
2. `llmKey` in `~/.bankr/config.json`
3. Falls back to your Bankr API key (`BANKR_API_KEY` / `apiKey`)

Most users only need a single key for both the agent API and the LLM gateway. Set a separate LLM key only if your keys have different permissions or rate limits.

### Setting the LLM Key

**Via CLI:**
```bash
bankr login --llm-key YOUR_LLM_KEY            # during login
bankr config set llmKey YOUR_LLM_KEY           # after login
```

**Via environment variable:**
```bash
export BANKR_LLM_KEY=your_llm_key_here
```

**Verify:**
```bash
bankr config get llmKey
```

## Available Models

| Model | Provider | Best For |
|-------|----------|----------|
| `claude-opus-4.6` | Anthropic | Most capable, advanced reasoning |
| `claude-opus-4.5` | Anthropic | Complex reasoning, architecture |
| `claude-sonnet-4.5` | Anthropic | Balanced speed and quality |
| `claude-haiku-4.5` | Anthropic | Fast, cost-effective |
| `gemini-3-pro` | Google | Long context (2M tokens) |
| `gemini-3-flash` | Google | High throughput |
| `gemini-2.5-pro` | Google | Long context, multimodal |
| `gemini-2.5-flash` | Google | Speed, high throughput |
| `gpt-5.2` | OpenAI | Advanced reasoning |
| `gpt-5.2-codex` | OpenAI | Code generation |
| `gpt-5-mini` | OpenAI | Fast, economical |
| `gpt-5-nano` | OpenAI | Ultra-fast, lowest cost |
| `kimi-k2.5` | Moonshot AI | Long-context reasoning |
| `qwen3-coder` | Alibaba | Code generation, debugging |

```bash
# Fetch live model list from the gateway
bankr llm models
```

## Credits

Check your LLM gateway credit balance:

```bash
bankr llm credits
```

Returns your remaining USD credit balance. When credits are exhausted, gateway requests will fail with HTTP 402.

## Tool Integrations

### OpenClaw

Auto-install the Bankr provider into your OpenClaw config:

```bash
# Write config to ~/.openclaw/openclaw.json
bankr llm setup openclaw --install

# Preview the config without writing
bankr llm setup openclaw
```

This writes the following provider config (with your key and all available models):

```json
{
  "models": {
    "providers": {
      "bankr": {
        "baseUrl": "https://llm.bankr.bot",
        "apiKey": "your_key_here",
        "api": "openai-completions",
        "models": [
          { "id": "claude-sonnet-4.5", "name": "Claude Sonnet 4.5", "api": "anthropic-messages" },
          { "id": "claude-haiku-4.5", "name": "Claude Haiku 4.5", "api": "anthropic-messages" },
          { "id": "gemini-3-flash", "name": "Gemini 3 Flash" },
          { "id": "gpt-5.2", "name": "GPT 5.2" }
        ]
      }
    }
  }
}
```

Claude models are automatically configured with `"api": "anthropic-messages"` per-model overrides while all other models use the default `"api": "openai-completions"`.

To use a Bankr model as your default in OpenClaw, add to `openclaw.json`:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "bankr/claude-sonnet-4.5"
      }
    }
  }
}
```

### Claude Code

Two ways to use Claude Code with the gateway:

**Option A: Launch directly (recommended)**

```bash
# Launch Claude Code through the gateway
bankr llm claude

# Pass any Claude Code flags through
bankr llm claude --model claude-sonnet-4.5
bankr llm claude --allowedTools Edit,Write,Bash
bankr llm claude --resume
```

All arguments after `claude` are forwarded to the `claude` binary. The CLI sets `ANTHROPIC_BASE_URL` and `ANTHROPIC_AUTH_TOKEN` automatically from your config (using `llmKey` if set, otherwise `apiKey`).

**Option B: Set environment variables**

```bash
# Print the env vars to add to your shell profile
bankr llm setup claude
```

This outputs:
```bash
export ANTHROPIC_BASE_URL="https://llm.bankr.bot"
export ANTHROPIC_AUTH_TOKEN="your_key_here"
```

Add these to `~/.zshrc` or `~/.bashrc` so all Claude Code sessions use the gateway.

### OpenCode

```bash
# Auto-install Bankr provider into ~/.config/opencode/opencode.json
bankr llm setup opencode --install

# Preview without writing
bankr llm setup opencode
```

### Cursor

```bash
# Get step-by-step setup instructions with your API key
bankr llm setup cursor
```

The setup adds your key as the OpenAI API Key, sets `https://llm.bankr.bot/v1` as the base URL override, and registers the available model IDs. When the base URL override is enabled, all model requests go through the gateway.

## Direct SDK Usage

The gateway is compatible with standard OpenAI and Anthropic SDKs — just override the base URL.

### curl (OpenAI format)

```bash
curl -X POST "https://llm.bankr.bot/v1/chat/completions" \
  -H "Authorization: Bearer $BANKR_LLM_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4.5",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### curl (Anthropic format)

```bash
curl -X POST "https://llm.bankr.bot/v1/messages" \
  -H "x-api-key: $BANKR_LLM_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4.5",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### OpenAI SDK (Python)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://llm.bankr.bot/v1",
    api_key="your_bankr_key",
)

response = client.chat.completions.create(
    model="claude-sonnet-4.5",
    messages=[{"role": "user", "content": "Hello"}],
)
```

### OpenAI SDK (TypeScript)

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://llm.bankr.bot/v1",
  apiKey: "your_bankr_key",
});

const response = await client.chat.completions.create({
  model: "gemini-3-flash",
  messages: [{ role: "user", content: "Hello" }],
});
```

### Anthropic SDK (Python)

```python
from anthropic import Anthropic

client = Anthropic(
    base_url="https://llm.bankr.bot",
    api_key="your_bankr_key",
)

message = client.messages.create(
    model="claude-sonnet-4.5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
)
```

## Troubleshooting

### 401 Unauthorized
- Verify key is set: `bankr config get llmKey` or `echo $BANKR_LLM_KEY`
- Check for leading/trailing spaces
- Ensure the key hasn't expired

### 402 Payment Required
- Credits exhausted: `bankr llm credits` shows $0.00
- Top up credits at [bankr.bot/api](https://bankr.bot/api)

### Model not found
- Use exact model IDs (e.g., `claude-sonnet-4.5`, not `claude-3-sonnet`)
- Check available models: `bankr llm models`

### Claude Code not found
- `bankr llm claude` requires Claude Code to be installed separately
- Install: https://docs.anthropic.com/en/docs/claude-code

### Slow responses
- Try `claude-haiku-4.5` or `gemini-3-flash` for faster responses
- The gateway has automatic failover — temporary slowness usually resolves itself

---

**Documentation**: https://docs.bankr.bot/llm-gateway/overview



## 📖 Reference: market-research.md

# Market Research Reference

Research tokens and analyze market data using Bankr's AI-powered analysis.

## Capabilities

- Token search across chains
- Price and market data
- Technical analysis
- Social sentiment analysis
- Price charts
- Trending tokens
- Token comparisons

## Prompt Examples

**Price queries:**
- "What's the price of ETH?"
- "How much is Bitcoin worth?"
- "Current price of BNKR"
- "Price of SOL in USD"

**Market data:**
- "Show me ETH market data"
- "What's the market cap of BNKR?"
- "Trading volume for Bitcoin"
- "SOL fully diluted valuation"

**Technical analysis:**
- "Do technical analysis on ETH"
- "Show RSI for Bitcoin"
- "Is ETH overbought?"
- "Support and resistance levels for BTC"
- "Moving averages for SOL"

**Sentiment analysis:**
- "What's the sentiment on ETH?"
- "Is the community bullish on SOL?"
- "Twitter sentiment for PEPE"
- "Social metrics for DOGE"

**Charts:**
- "Show me ETH price chart"
- "Generate BTC chart for last week"
- "30-day price chart for BNKR"

**Discovery:**
- "What tokens are trending?"
- "Show top gainers today"
- "Top losers in the last 24 hours"
- "New tokens on Base"

**Comparisons:**
- "Compare ETH vs SOL"
- "Which is better: MATIC or ARB?"
- "Show differences between UNI and AAVE"

## Data Available

### Price Data
- Current USD price
- 24h / 7d / 30d change
- All-time high/low
- Historical prices

### Market Metrics
- Market cap
- Fully diluted valuation (FDV)
- 24h trading volume
- Circulating supply
- Total supply
- Max supply
- Number of holders

### Technical Indicators
- RSI (Relative Strength Index)
- MACD (Moving Average Convergence Divergence)
- Moving averages (50-day, 200-day)
- Support/resistance levels
- Bollinger Bands
- Volume analysis

### Social Metrics
- Twitter mentions
- Community sentiment
- Social volume
- Influencer activity

## Supported Chains

Token research works across:
- Base
- Polygon
- Ethereum
- Solana
- Unichain

## Token Search

Find tokens by name, symbol, or address:
- "Search for BNKR token"
- "Find tokens called Bankr"
- "What is the contract for PEPE on Base?"
- "Token address for USDC on Polygon"

## Use Cases

**Before trading:**
- Check current price and trends
- Analyze technical indicators
- Review sentiment

**Investment research:**
- Compare similar tokens
- Check market cap and volume
- Review holder distribution

**Market monitoring:**
- Track trending tokens
- Find top gainers/losers
- Monitor sentiment shifts

## Limitations

- Historical data limited to available timeframes
- Sentiment based on public social data
- New tokens may have limited data
- Analysis is informational, not investment advice
- Some metrics may not be available for all tokens

## Best Practices

1. **Cross-reference** - Check multiple metrics
2. **Context matters** - Consider overall market conditions
3. **Volume** - Low volume tokens have less reliable data
4. **New tokens** - Be cautious with recently launched tokens
5. **DYOR** - Use as one of many research tools



## 📖 Reference: nft-operations.md

# NFT Operations Reference

Browse, purchase, and manage NFTs across chains via OpenSea integration.

**Supported Chains**: Base, Ethereum, Polygon

## Operations

- **Browse** - Search NFT collections
- **View Listings** - Find best deals and floor prices
- **Buy** - Purchase NFTs from marketplace listings
- **View Holdings** - Check your NFT portfolio
- **Transfer** - Send NFTs to another wallet
- **Mint** - Mint from supported platforms (Manifold, SeaDrop)

## Prompt Examples

**Browse NFTs:**
- "Find NFTs from the Bored Ape collection"
- "Show me trending NFT collections"
- "Search for Pudgy Penguins NFTs"
- "What are the top NFT collections on Base?"

**View listings:**
- "What's the floor price for Pudgy Penguins?"
- "Show cheapest NFTs in Azuki collection"
- "List all available Bored Apes under 50 ETH"
- "Show me the rarest items in [collection]"

**Buy NFTs:**
- "Buy the cheapest Bored Ape"
- "Purchase this NFT: [OpenSea URL]"
- "Buy Pudgy Penguin #1234"
- "Get the floor Azuki"

**View holdings:**
- "Show my NFTs"
- "What NFTs do I own on Ethereum?"
- "My NFT collection on Base"
- "Show all my Pudgy Penguins"

**Transfer NFTs:**
- "Send my Bored Ape #123 to 0x..."
- "Transfer Pudgy Penguin to vitalik.eth"
- "Send NFT to @friend"

**Minting:**
- "Mint from [Manifold link]"
- "Mint 5 NFTs from this collection"

## Collection Resolution

Bankr resolves common names and abbreviations:

| Input | Resolved |
|-------|----------|
| "Bored Apes" / "BAYC" | boredapeyachtclub |
| "Pudgy Penguins" | pudgypenguins |
| "CryptoPunks" / "Punks" | cryptopunks |
| "Azuki" | azuki |
| "Doodles" | doodles-official |
| "Cool Cats" | cool-cats-nft |

## Chain Considerations

### Ethereum
- Most valuable blue-chip collections
- Highest liquidity
- Expensive gas fees
- Established marketplace

### Base
- Growing NFT ecosystem
- Very low gas fees
- Newer collections
- Good for emerging artists

### Polygon
- Gaming and metaverse NFTs
- Low gas fees
- Good for frequent trading
- Strong gaming communities

## OpenSea Integration

Bankr uses OpenSea's marketplace:
- Real-time floor prices
- Verified collections
- Direct purchase links
- Rarity data
- Collection stats

## Common Issues

| Issue | Resolution |
|-------|------------|
| Collection not found | Try alternative names or contract address |
| NFT already sold | Try another listing or wait for new listings |
| Insufficient funds | Check balance including gas costs |
| High gas | Wait for lower gas or try L2 (Base/Polygon) |
| Unverified collection | Verify legitimacy before purchasing |

## Safety Tips

1. **Verify collection** - Check official links and social media
2. **Check floor price** - Avoid overpaying, compare to floor
3. **Verified badge** - Look for OpenSea verified collections
4. **Gas costs** - Factor in gas, especially on Ethereum
5. **Research** - DYOR on collection before buying
6. **Scams** - Be wary of too-good-to-be-true deals
7. **Contract address** - Verify it matches official contract

## NFT Portfolio

View your holdings:
- Total NFT count by chain
- Estimated floor value
- Collection breakdown
- Recently acquired
- Rarest pieces

## Minting

For supported mint platforms:
- Manifold mints
- SeaDrop protocol
- Direct contract mints (if supported)

Provide the mint page URL and Bankr handles the transaction.

## Best Practices

1. **Start small** - Learn with cheaper NFTs first
2. **Research collections** - Check roadmap and community
3. **Compare prices** - Look at recent sales and floor
4. **Gas timing** - Mint/buy during low gas periods
5. **Hold long-term** - Most value comes from holding
6. **Diversify** - Don't put everything in one collection



## 📖 Reference: polymarket.md

# Polymarket Reference

Interact with Polymarket prediction markets.

## Overview

Polymarket is a decentralized prediction market where users can search markets, view odds, place bets, and manage positions.

**Chain**: Polygon (uses USDC.e for betting)

## Prompt Examples

**Search markets:**
- "Search Polymarket for election markets"
- "What prediction markets are trending?"
- "Find markets about crypto"
- "Show Polymarket sports markets"

**Check odds:**
- "What are the odds Trump wins the election?"
- "Check the odds on the Eagles game"
- "Polymarket odds for ETF approval"
- "What's the probability of [event]?"

**Place bets:**
- "Bet $10 on Yes for Trump winning"
- "Place $5 on the Eagles to win"
- "Buy $20 of Yes shares on [market]"
- "Bet on No for [event]"

**View positions:**
- "Show my Polymarket positions"
- "What bets do I have active?"
- "My Polymarket portfolio"

**Redeem winnings:**
- "Redeem my Polymarket positions"
- "Cash out my resolved bets"
- "Claim my winnings"

## How Betting Works

### Share-Based System
- You buy shares of "Yes" or "No" outcomes
- Share price reflects market probability
  - $0.60 = 60% chance according to market
  - $0.20 = 20% chance
- If your outcome wins, shares pay $1.00 each
- Profit = $1.00 - purchase price (per share)

### Example
**Bet $10 on "Yes" at $0.60 price:**
- Receive: ~16.67 shares
- If Yes wins: Get $16.67 (profit: $6.67)
- If No wins: Lose $10

### Return on Investment
- Better odds (lower price) = higher potential return
- Price $0.10 → 10x return if wins
- Price $0.90 → 1.11x return if wins

## Auto-Bridging

If you don't have USDC on Polygon:
- Bankr automatically bridges from another chain
- Uses your available stablecoins (USDC/USDT)
- Optimizes for lowest fees
- Typically completes in minutes

## Market Types

| Category | Examples |
|----------|----------|
| Politics | Elections, legislation, appointments |
| Sports | Game outcomes, championships, player stats |
| Crypto | Price predictions, ETF approvals, launches |
| Culture | Awards shows, entertainment events |
| Business | Company earnings, acquisitions, product launches |
| World Events | Geopolitics, natural events, social trends |

## Market Phases

### Active Markets
- Open for betting
- Prices fluctuate with news
- Can buy or sell shares

### Closed Markets
- No new bets accepted
- Outcome determined
- Awaiting resolution

### Resolved Markets
- Outcome confirmed
- Winners can redeem
- Losers get nothing

## Common Issues

| Issue | Resolution |
|-------|------------|
| Market not found | Try different search terms, check spelling |
| Insufficient USDC | Add USDC or let auto-bridge handle it |
| Market closed | Can't bet on closed/resolved markets |
| Low liquidity | May get worse prices on small markets |
| Slippage | Large bets may move price against you |

## Tips for Success

### Research
1. Read market details carefully
2. Check resolution criteria
3. Review similar past markets
4. Follow news about the event

### Strategy
1. **Start small** - Test with small amounts
2. **Diversify** - Spread risk across markets
3. **Think probability** - If you think real odds > market odds, bet Yes
4. **Sell early** - Can sell shares before resolution
5. **Compound** - Reinvest winnings

### Timing
1. **Early bets** - Better odds before news breaks
2. **React fast** - Odds change quickly with news
3. **Redeem promptly** - Claim winnings soon after resolution

### Risk Management
1. Never bet more than you can afford to lose
2. Understand the outcome criteria
3. Consider worst-case scenarios
4. Don't let emotions drive decisions
5. Set a budget and stick to it

## Market Liquidity

- **High liquidity** - Easy to buy/sell, stable prices
- **Low liquidity** - Harder to exit, price slippage
- Check volume before large bets
- Popular markets have better liquidity

## Resolution Process

1. **Event occurs** - Real-world outcome determined
2. **Market closes** - No more betting
3. **Resolution** - Polymarket resolves via UMA oracle based on outcome criteria
4. **Winners paid** - Shares worth $1 each
5. **Losers** - Shares become worthless

## Advanced Features

### Selling Shares
- Can sell before resolution
- Lock in profits or cut losses
- Price depends on current odds

### Partial Positions
- Don't have to go all-in
- Can build position over time
- Average your entry price

### Market Making
- Provide liquidity to earn fees
- Advanced strategy
- Requires understanding of odds

## Responsible Betting

- Set limits before you start
- Don't chase losses
- Take breaks
- Betting is not guaranteed profit
- Only use money you can afford to lose

## Best Practices

1. **Read carefully** - Understand resolution criteria
2. **Check sources** - Official resolution sources
3. **Start small** - Learn with small bets
4. **Track record** - Keep notes on your bets
5. **Stay informed** - Follow news about your markets
6. **Redeem quickly** - Don't leave money on table



## 📖 Reference: portfolio.md

# Portfolio Reference

Query token balances and portfolio across all supported chains.

## Supported Chains

All chains: Base, Polygon, Ethereum, Unichain, Solana

## Prompt Examples

**Full portfolio:**
- "Show my portfolio"
- "What's my total balance?"
- "How much crypto do I have?"
- "Portfolio value"
- "What's my net worth?"

**Chain-specific:**
- "Show my Base balance"
- "What tokens do I have on Polygon?"
- "Ethereum portfolio"
- "Solana holdings"

**Token-specific:**
- "How much ETH do I have?"
- "What's my USDC balance?"
- "Show my ETH across all chains"
- "BNKR balance"

## Features

- **USD Valuation**: All balances include current USD value
- **Multi-Chain Aggregation**: See the same token across all chains
- **Real-Time Prices**: Values reflect current market prices
- **Comprehensive View**: Shows all tokens with meaningful balances

## Common Tokens Tracked

- **Stablecoins**: USDC, USDT, DAI
- **Blue Chips**: ETH, WETH, WBTC
- **DeFi**: UNI, AAVE, LINK, COMP, CRV
- **Memecoins**: DOGE, SHIB, PEPE, BONK
- **Project tokens**: BNKR, ARB, OP, MATIC

## Use Cases

**Before trading:**
- "Do I have enough ETH to swap for 100 USDC?"
- "Check if I have MATIC for gas on Polygon"

**Portfolio review:**
- "What's my largest holding?"
- "Show portfolio breakdown by chain"
- "What percentage of my portfolio is stablecoins?"

**After transactions:**
- "Did my ETH arrive?"
- "Show my new BNKR balance"
- "Verify the swap completed"

## Output Format

Portfolio responses typically include:
- Token name and symbol
- Amount held
- Current USD value
- Chain location
- Price per token
- 24h price change

## Notes

- Balance queries are read-only (no transactions)
- Shows balance of connected wallet address
- Very small balances (dust) may be excluded
- Includes native tokens (ETH, MATIC, SOL) and ERC20/SPL tokens



## 📖 Reference: safety.md

# Safety & Access Control Reference

Comprehensive safety guidance for building agents and integrations with the Bankr API and CLI. Covers API key types, access controls, wallet separation, rate limits, and operational best practices.

## API Key Types & Separation

Bankr uses a single key format (`bk_...`) with **capability flags** that control what each key can access. You can optionally configure a separate key for the LLM Gateway.

### Capability Flags

Each API key has independent toggles managed at [bankr.bot/api](https://bankr.bot/api):

| Flag | Controls Access To | Default |
|------|-------------------|---------|
| `agentApiEnabled` | `/agent/*` endpoints (prompt, sign, submit, job status) | false |
| `llmGatewayEnabled` | LLM Gateway at `llm.bankr.bot` (chat completions, model access) | false |
| `externalOrdersEnabled` | External order submission endpoints | false |
| `readOnly` | When true, restricts agent sessions to read-only tools | false |

A single key can have multiple capabilities enabled (e.g., both Agent API and LLM Gateway).

### Agent API Key vs LLM Gateway Key

For most users, **one key works for both** the Agent API and LLM Gateway. However, you can configure a separate LLM key when you want different permissions or rate limits for each:

| Config | Agent API Key | LLM Gateway Key |
|--------|--------------|-----------------|
| Environment variable | `BANKR_API_KEY` | `BANKR_LLM_KEY` (falls back to `BANKR_API_KEY`) |
| CLI config key | `apiKey` | `llmKey` (falls back to `apiKey`) |
| Used by | `bankr prompt`, `/agent/*` endpoints | `bankr llm claude`, `llm.bankr.bot` |

**When to use separate keys:**
- Your agent API key is read-only but your LLM key needs no such restriction (LLM calls are inherently read-only)
- You want to revoke LLM access without affecting agent operations (or vice versa)
- Different keys for different team members or environments

**Setting a separate LLM key:**
```bash
bankr login --api-key bk_AGENT_KEY --llm-key bk_LLM_KEY   # during login
bankr config set llmKey bk_LLM_KEY                         # after login
```

For full LLM Gateway setup details, see [llm-gateway.md](llm-gateway.md).

## API Key Access Control

Bankr API keys support granular access control configured at [bankr.bot/api](https://bankr.bot/api). Two key security features: **read-only mode** and **IP whitelisting**.

### Read-Only API Keys

When an API key has `readOnly: true`, all write tools are filtered from the agent session. The agent receives a system directive explaining the restriction and will inform users accordingly.

**Behavior by endpoint:**

| Endpoint | Read-Only Behavior |
|----------|-------------------|
| `POST /agent/prompt` | Works — but only read tools are available (balances, prices, analytics, portfolio, research) |
| `POST /agent/sign` | Blocked — returns 403 |
| `POST /agent/submit` | Blocked — returns 403 |
| `GET /agent/job/{jobId}` | Works — unaffected |
| `POST /agent/job/{jobId}/cancel` | Works — unaffected |

**403 error responses:**

For `/agent/sign`:
```json
{
  "error": "Read-only API key",
  "message": "This API key has read-only access and cannot sign messages or transactions. Update your API key permissions at https://bankr.bot/api"
}
```

For `/agent/submit`:
```json
{
  "error": "Read-only API key",
  "message": "This API key has read-only access and cannot submit transactions. Update your API key permissions at https://bankr.bot/api"
}
```

**Write tool categories filtered in read-only mode:**

| Category | Examples |
|----------|----------|
| Swaps | Token buy/sell/swap across all chains |
| Transfers | Send tokens, NFTs |
| NFT Operations | Purchase, mint NFTs |
| Staking | Stake/unstake operations |
| Orders | Limit orders, stop losses |
| Token Launches | Deploy ERC20/SPL tokens |
| Leverage | Open/close/modify positions |
| Polymarket | Place/redeem bets |
| Claims | Claim rewards, fees |

The agent receives a system directive and will explain the restriction if a user requests a write operation:

> *"This session has READ-ONLY API access. You can retrieve information (balances, prices, analytics, portfolio data, market research) but CANNOT execute any transactions."*

### IP Whitelisting

API keys support an `allowedIps` whitelist. When configured, requests from non-whitelisted IPs are rejected at the authentication layer before reaching any endpoint.

- **Empty array** (`[]`) = all IPs allowed (default)
- **Non-empty array** = only listed IPs can use the key

**403 error response:**
```json
{
  "error": "IP address not allowed",
  "message": "IP address not allowed for this API key"
}
```

### Configuring Access Control

Manage API key settings at [bankr.bot/api](https://bankr.bot/api):

| Field | Type | Description |
|-------|------|-------------|
| `readOnly` | boolean | When true, only read tools are available |
| `allowedIps` | string[] | IP whitelist (empty = all allowed) |
| `agentApiEnabled` | boolean | Whether `/agent/*` endpoints are accessible |
| `llmGatewayEnabled` | boolean | Whether LLM Gateway endpoints are accessible |

## CLI Security

The Bankr CLI (`@bankr/cli`) stores credentials locally and provides its own safety considerations alongside the REST API.

### Credential Storage

The CLI stores keys in `~/.bankr/config.json`:

```json
{
  "apiKey": "bk_...",
  "llmKey": "bk_...",
  "apiUrl": "https://api.bankr.bot",
  "llmUrl": "https://llm.bankr.bot"
}
```

**Safety rules for CLI credentials:**
- Add `~/.bankr/` to your global `.gitignore` — never commit this directory
- On shared machines, restrict file permissions: `chmod 600 ~/.bankr/config.json`
- Use `bankr logout` to clear stored credentials when done on a shared machine
- For CI/CD, prefer environment variables (`BANKR_API_KEY`, `BANKR_LLM_KEY`) over config files

### Non-Interactive Login

When running the CLI in automated scripts or AI agent environments where interactive prompts aren't possible:

```bash
# Direct key login — no prompts
bankr login --api-key bk_YOUR_KEY

# With separate LLM key
bankr login --api-key bk_AGENT_KEY --llm-key bk_LLM_KEY

# Verify it worked
bankr whoami
```

### CLI vs REST API Access Controls

Access controls (read-only, IP whitelist) apply identically whether you use the CLI or REST API — they are enforced server-side on the API key itself. The CLI is a convenience wrapper; it submits the same requests as direct API calls.

```bash
# These two are equivalent — same access controls apply
bankr prompt "What is my balance?"
curl -X POST "https://api.bankr.bot/agent/prompt" \
  -H "X-API-Key: bk_YOUR_KEY" \
  -d '{"prompt": "What is my balance?"}'
```

## Dedicated Agent Wallet

When building autonomous agents that execute transactions, use a **separate Bankr account** as the agent's wallet rather than your personal account. This limits blast radius — if an agent key is compromised or the agent misbehaves, only the dedicated wallet's funds are at risk.

### Why Separate Wallets

- **Limited exposure**: A compromised agent key only exposes the agent wallet's funds, not your main holdings
- **Clear accounting**: Agent transactions are isolated from personal activity
- **Independent controls**: Apply stricter access controls (read-only, IP whitelist) without affecting personal use
- **Easy revocation**: Disable the agent account without disrupting your primary wallet

### Setup Steps

1. **Create a new Bankr account** — Sign up at [bankr.bot/api](https://bankr.bot/api) with a different email. This provisions fresh EVM and Solana wallets automatically.
2. **Generate an API key** — Enable **Agent API** access for the key
3. **Configure access controls** — Set `readOnly`, `allowedIps`, or both as appropriate for your use case
4. **Fund with limited amounts** — Transfer only what the agent needs for its operations

### Recommended Funding

Fund the agent wallet with enough for gas and intended operations, not more:

| Chain | Gas Buffer | Trading Capital |
|-------|-----------|-----------------|
| Base | 0.01 - 0.05 ETH | As needed for trades |
| Polygon | 5 - 10 MATIC | As needed for trades |
| Ethereum | 0.05 - 0.1 ETH | As needed for trades |
| Solana | 0.1 - 0.5 SOL | As needed for trades |

Replenish periodically rather than pre-loading large amounts.

### Access Control Combinations

Choose the right combination based on your agent's purpose:

| Use Case | readOnly | allowedIps | Funding Level |
|----------|----------|------------|---------------|
| Monitoring / analytics bot | Yes | Yes (server IP) | None needed |
| Trading bot (server-side) | No | Yes (server IP) | Limited trading capital |
| Development / testing | No | No | Minimal (test amounts) |
| Read-only research agent | Yes | No | None needed |

## Rate Limits

### Daily Message Limits

The `/agent/prompt` endpoint enforces daily message limits per account:

| Tier | Daily Limit |
|------|-------------|
| Standard | 100 messages/day |
| Bankr Club | 1,000 messages/day |
| Custom | Set per API key |

**429 response when limit exceeded:**
```json
{
  "error": "Daily limit exceeded",
  "message": "You have reached your daily API limit of 100 messages. Upgrade to Bankr Club for 1000 messages/day. Resets at 2025-01-15T12:00:00.000Z",
  "resetAt": 1736942400000,
  "limit": 100,
  "used": 100
}
```

The reset window is **24 hours from the first message** (rolling window), not a fixed midnight reset. The `resetAt` field in the response tells you exactly when the counter resets.

### General API Rate Limits

| Scope | Limit | Window |
|-------|-------|--------|
| Public endpoints | 100 requests | 15 minutes per IP |
| General API | 120 requests | 1 minute per IP |
| External orders | 10 requests | 1 second per API key |

For error response handling, retry strategies, and exponential backoff guidance, see [error-handling.md](error-handling.md).

## Transaction Safety

Blockchain transactions are **irreversible** once confirmed. Key safety rules:

- **Test first** — Always test with small amounts before scaling up. Use Base or Polygon for low-cost testing.
- **Verify recipients** — Double-check addresses before transfers. See [transfers.md](transfers.md) for address resolution details.
- **Gas buffer** — Keep enough native tokens for gas on each chain you operate on. See the funding table above for recommended minimums.
- **Wait for confirmation** — Use `waitForConfirmation: true` with `/agent/submit` to ensure transactions are confirmed before proceeding. See [sign-submit-api.md](sign-submit-api.md).
- **Immediate execution** — `/agent/submit` executes transactions immediately with no confirmation prompt. For safety with the prompt API, the AI agent may ask for confirmation on large or unusual operations.
- **Understand calldata** — When using arbitrary transactions, verify the calldata source is trusted. See [arbitrary-transaction.md](arbitrary-transaction.md).

## Key Management

### Storage

- **Environment variables** — Store API keys in `BANKR_API_KEY` and LLM keys in `BANKR_LLM_KEY`, never in source code
- **CLI config** — The CLI stores keys in `~/.bankr/config.json`. Ensure this directory is in `.gitignore` and has restricted permissions
- **Never commit secrets** — Add `~/.bankr/`, `.env`, and credential files to `.gitignore`. Use `bankr logout` to clear CLI credentials on shared machines

### Rotation & Revocation

- **Rotate periodically** — Generate new keys and deactivate old ones at [bankr.bot/api](https://bankr.bot/api). After rotating, update both env vars and CLI config (`bankr login --api-key NEW_KEY`)
- **Revoke immediately** — If any key (API or LLM) is leaked, deactivate it immediately at the dashboard
- **One key per purpose** — Use separate keys for different agents, environments, and services (Agent API vs LLM Gateway) so you can revoke individually without disrupting unrelated systems

### Best Practices

- Prefer environment variables for server-side agents and CI/CD; use CLI config for local development
- If you use separate API and LLM keys, rotate them independently
- When revoking a compromised key, check both `BANKR_API_KEY` and `BANKR_LLM_KEY` — if the same key was used for both, both need updating

For the full API key setup and authentication workflow, see [api-workflow.md](api-workflow.md).

## Safety by Feature

Each feature has specific safety considerations documented in its reference file:

| Feature | Key Safety Points | Reference |
|---------|-------------------|-----------|
| Leverage Trading | Risk warnings, liquidation, position sizing | [leverage-trading.md](leverage-trading.md) |
| Transfers | Verify recipient address, ENS resolution | [transfers.md](transfers.md) |
| NFT Operations | Collection verification, floor price checks | [nft-operations.md](nft-operations.md) |
| Polymarket | Responsible betting, position limits | [polymarket.md](polymarket.md) |
| Token Deployment | Legal considerations, rate limits | [token-deployment.md](token-deployment.md) |
| Automation | Monitoring active orders, execution conditions | [automation.md](automation.md) |
| Arbitrary Transactions | Trust calldata source, verify contract targets | [arbitrary-transaction.md](arbitrary-transaction.md) |
| Sign & Submit API | Immediate execution, no confirmation prompt | [sign-submit-api.md](sign-submit-api.md) |

## Checklist

Before deploying an agent or integration:

- [ ] Use a **dedicated agent wallet** — not your personal account
- [ ] Fund the agent wallet with **limited amounts** appropriate to its purpose
- [ ] Set API key to **read-only** if the agent only needs to query data
- [ ] Configure **IP whitelisting** for server-side agents with known IPs
- [ ] Store keys in **environment variables** (`BANKR_API_KEY`, `BANKR_LLM_KEY`), never in source code or version control
- [ ] If using the CLI, ensure `~/.bankr/` is in `.gitignore` and has restricted file permissions
- [ ] Use **separate keys** for Agent API vs LLM Gateway if they need independent access controls or revocation
- [ ] **Test with small amounts** on low-cost chains (Base, Polygon) before production use
- [ ] Verify **recipient addresses** in any transfer logic before execution
- [ ] Implement **error handling** for rate limits (429) and access control errors (403)
- [ ] Monitor the agent's **daily message usage** against your tier limit
- [ ] Review and **rotate all keys** (API and LLM) periodically; revoke immediately if compromised



## 📖 Reference: sign-submit-api.md

# Sign and Submit API Reference

Synchronous endpoints for signing messages and submitting transactions directly.

## Overview

Unlike the async prompt endpoint, these endpoints are **synchronous** and return immediately:

| Endpoint | Purpose | Returns |
|----------|---------|---------|
| `POST /agent/sign` | Sign messages, typed data, or transactions | Signature |
| `POST /agent/submit` | Submit raw transactions to chain | Transaction hash |

## POST /agent/sign

Sign data without broadcasting to the network.

### Supported Signature Types

| Type | Use Case |
|------|----------|
| `personal_sign` | Sign plain text messages (auth, verification) |
| `eth_signTypedData_v4` | Sign EIP-712 typed data (permits, orders) |
| `eth_signTransaction` | Sign transactions for later broadcast |

### Request Examples

#### personal_sign

```bash
curl -X POST "https://api.bankr.bot/agent/sign" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "signatureType": "personal_sign",
    "message": "Sign in to MyApp\nNonce: abc123"
  }'
```

#### eth_signTypedData_v4

```bash
curl -X POST "https://api.bankr.bot/agent/sign" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "signatureType": "eth_signTypedData_v4",
    "typedData": {
      "domain": {
        "name": "USD Coin",
        "version": "2",
        "chainId": 8453,
        "verifyingContract": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913"
      },
      "types": {
        "Permit": [
          { "name": "owner", "type": "address" },
          { "name": "spender", "type": "address" },
          { "name": "value", "type": "uint256" },
          { "name": "nonce", "type": "uint256" },
          { "name": "deadline", "type": "uint256" }
        ]
      },
      "primaryType": "Permit",
      "message": {
        "owner": "0xYourAddress",
        "spender": "0xSpenderAddress",
        "value": "1000000",
        "nonce": "0",
        "deadline": "1735689600"
      }
    }
  }'
```

#### eth_signTransaction

```bash
curl -X POST "https://api.bankr.bot/agent/sign" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "signatureType": "eth_signTransaction",
    "transaction": {
      "to": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
      "chainId": 8453,
      "value": "0",
      "data": "0xa9059cbb..."
    }
  }'
```

### Success Response (200 OK)

```json
{
  "success": true,
  "signature": "0x...",
  "signer": "0xYourWalletAddress",
  "signatureType": "personal_sign"
}
```

### Error Responses

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `signatureType is required` | Missing signature type |
| 400 | `message is required for personal_sign` | Missing message field |
| 400 | `typedData is required for eth_signTypedData_v4` | Missing typed data |
| 400 | `transaction is required for eth_signTransaction` | Missing transaction |
| 401 | `Authentication required` | Missing or invalid API key |
| 403 | `Agent API access not enabled` | API key lacks agent access |

## POST /agent/submit

Submit raw transactions directly to the blockchain.

### Request Body

```json
{
  "transaction": {
    "to": "0x...",
    "chainId": 8453,
    "value": "1000000000000000000",
    "data": "0x..."
  },
  "description": "Transfer 1 ETH",
  "waitForConfirmation": true
}
```

### Transaction Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `to` | string | Yes | Destination address |
| `chainId` | number | Yes | Chain ID (8453=Base, 1=Ethereum, 137=Polygon) |
| `value` | string | No | Value in wei (as string) |
| `data` | string | No | Calldata (hex string) |
| `gas` | string | No | Gas limit |
| `gasPrice` | string | No | Legacy gas price |
| `maxFeePerGas` | string | No | EIP-1559 max fee |
| `maxPriorityFeePerGas` | string | No | EIP-1559 priority fee |
| `nonce` | number | No | Transaction nonce |

### Options

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `description` | string | - | Human-readable description for logging |
| `waitForConfirmation` | boolean | `true` | Wait for on-chain confirmation |

### Request Examples

#### Simple ETH Transfer

```bash
curl -X POST "https://api.bankr.bot/agent/submit" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {
      "to": "0xRecipientAddress",
      "chainId": 8453,
      "value": "1000000000000000000"
    },
    "description": "Send 1 ETH"
  }'
```

#### ERC20 Transfer

```bash
curl -X POST "https://api.bankr.bot/agent/submit" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {
      "to": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
      "chainId": 8453,
      "value": "0",
      "data": "0xa9059cbb000000000000000000000000recipient000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000f4240"
    },
    "description": "Transfer USDC"
  }'
```

#### Fire-and-Forget

```bash
curl -X POST "https://api.bankr.bot/agent/submit" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {
      "to": "0xRecipientAddress",
      "chainId": 8453,
      "value": "100000000000000000"
    },
    "waitForConfirmation": false
  }'
```

### Success Response (200 OK)

With confirmation (`waitForConfirmation: true`):

```json
{
  "success": true,
  "transactionHash": "0x...",
  "status": "success",
  "blockNumber": "12345678",
  "gasUsed": "21000",
  "signer": "0xYourWalletAddress",
  "chainId": 8453
}
```

Without confirmation (`waitForConfirmation: false`):

```json
{
  "success": true,
  "transactionHash": "0x...",
  "status": "pending",
  "signer": "0xYourWalletAddress",
  "chainId": 8453
}
```

### Transaction Status Values

| Status | Description |
|--------|-------------|
| `success` | Transaction confirmed and succeeded |
| `reverted` | Transaction confirmed but reverted |
| `pending` | Transaction submitted, not yet confirmed |

### Error Responses

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `transaction object is required` | Missing transaction |
| 400 | Submission failed | Insufficient funds, gas estimation failed, etc. |
| 401 | `Authentication required` | Missing or invalid API key |
| 403 | `Agent API access not enabled` | API key lacks agent access |

## Use Cases

### Authentication (Sign)

Use `personal_sign` to verify wallet ownership:

```json
{
  "signatureType": "personal_sign",
  "message": "Sign in to MyApp\n\nNonce: xyz789\nTimestamp: 2025-01-26T10:00:00Z"
}
```

### Gasless Approvals (Sign)

Use `eth_signTypedData_v4` for EIP-2612 permits:

```json
{
  "signatureType": "eth_signTypedData_v4",
  "typedData": { /* permit struct */ }
}
```

### Pre-Built Transactions (Submit)

Submit transactions built by external tools:

```javascript
const tx = await buildSwapTransaction();
await fetch('https://api.bankr.bot/agent/submit', {
  method: 'POST',
  headers: { 'X-API-Key': apiKey, 'Content-Type': 'application/json' },
  body: JSON.stringify({ transaction: tx })
});
```

### Multi-Step Workflows (Submit)

Execute approve + swap in sequence:

```javascript
// 1. Approve
await submit({ transaction: approveTx });

// 2. Swap
await submit({ transaction: swapTx });
```

## Comparison

| Feature | /agent/prompt | /agent/sign | /agent/submit |
|---------|---------------|-------------|---------------|
| Input | Natural language | Structured data | Transaction object |
| Response | Async (job ID) | Sync (signature) | Sync (tx hash) |
| Executes on-chain | Via AI agent | No | Yes |
| Best for | General queries | Auth, permits | Raw transactions |

## Security Notes

- API keys with agent access can submit transactions — keep them secure
- The `submit` endpoint has no confirmation prompts — it executes immediately
- Validate all transaction parameters before submission
- Consider using `waitForConfirmation: true` for important transactions

---

**Full documentation**: [docs.bankr.bot/agent-api](https://docs.bankr.bot/agent-api/overview)



## 📖 Reference: token-deployment.md

# Token Deployment Reference

Deploy and manage tokens on EVM chains (via Clanker) and Solana (via Raydium LaunchLab).

## Supported Chains

| Chain | Protocol | Token Standard | Best For |
|-------|----------|----------------|----------|
| **Base** | Clanker | ERC20 | Memecoins, social tokens |
| **Unichain** | Clanker | ERC20 | Lower fees, newer ecosystem |
| **Solana** | Raydium LaunchLab | SPL | High-speed trading, bonding curves |

---

## Solana Token Launches (Raydium LaunchLab)

Launch SPL tokens on Solana with a bonding curve mechanism that auto-migrates to a Raydium CPMM pool.

### Deployment Parameters

| Parameter | Required | Description | Example |
|-----------|----------|-------------|---------|
| **Name** | Yes | Token name (1-32 chars) | "MoonRocket" |
| **Symbol** | No | Ticker (1-10 chars), defaults to name | "MOON" |
| **Image** | No | Logo URL | "https://example.com/logo.png" |
| **Decimals** | No | Token decimals (0-9), default 6 | 6 |
| **Fee Recipient** | No | Wallet to receive 99.9% of creator fees | "7xKXtg..." |
| **Cliff Period** | No | Vesting cliff in seconds | 2592000 (30 days) |
| **Unlock Period** | No | Vesting period in seconds | 7776000 (90 days) |
| **Locked Amount** | No | Tokens to lock for vesting | 500000000 |

### Prompt Examples

**Launch tokens:**
- "Launch a token called MOON on Solana"
- "Deploy a Solana memecoin called DOGE2"
- "Launch SpaceRocket with symbol ROCK"
- "Create a token with 30 day cliff and 90 day vesting"
- "Launch BRAIN and route fees to 7xKXtg..."

**Check fees:**
- "How much fees can I claim for MOON?"
- "Check fee status for my token"

**Claim fees:**
- "Claim my fees for MOON" (works for both creator and fee recipient)
- "Claim creator fees for my token"

**Fee Key NFTs:**
- "Show my Fee Key NFTs"
- "What tokens do I have fee rights for?"
- "Transfer fees for MOON to 7xKXtg..."

**Claim shared fee NFT (post-migration):**
- "Claim my fee NFT for ROCKET"

### Bonding Curve Mechanics

1. **Launch**: Token starts with a bonding curve that determines price based on supply
2. **Trading**: Early buyers get lower prices; price increases as more tokens are bought
3. **Migration**: When bonding curve fills, token auto-migrates to Raydium CPMM pool
4. **Post-Migration**: Trading continues on standard AMM with LP fee distribution

**Benefits:**
- Fair launch mechanism (no pre-allocation needed)
- Price discovery through market demand
- Automatic liquidity provision
- No rug pull risk (liquidity is locked)

### Fee Structure

**During Bonding Curve Phase:**
| Fee | Recipient | Description |
|-----|-----------|-------------|
| 1% | Bankr Platform | Platform fee |
| 0.5% | Creator | Creator trading fee (or split with fee recipient) |

**Fee Sharing (when feeRecipient specified):**
| Share | Recipient | Description |
|-------|-----------|-------------|
| 99.9% | Fee Recipient | Main share of creator fees |
| 0.1% | Creator | Referrer fee |

**At Migration (when bonding curve completes):**
| LP Share | Recipient | Description |
|----------|-----------|-------------|
| 40% | Bankr Platform | Locked platform LP |
| 50% | Token Creator | Locked creator LP (Fee Key NFT) |
| 10% | Burned | Deflationary mechanism |

**Post-Migration:**
- Token trades on Raydium CPMM pool
- Fee Key NFT holders can claim 50% of LP trading fees

### Fee Claiming

**Checking Fee Status:**
- Use "How much fees can I claim for TOKEN?" to check status
- Shows pool status (bonding curve vs migrated)
- Explains how to claim based on your role

**Standard Tokens (No Fee Sharing):**
- Creator claims all 0.5% trading fees
- Use "Claim my fees for TOKEN"
- Requires ~0.005 SOL gas

**Tokens with Fee Sharing Arrangement:**
- BOTH creator AND fee recipient can initiate claims
- Fees automatically split: 99.9% to recipient, 0.1% to creator
- Gas is sponsored by Bankr (free for users)
- Use "Claim my fees for TOKEN" (works for either party)

**Post-Migration Fee Claiming:**
1. Fee recipient claims Fee Key NFT: "Claim my fee NFT for TOKEN"
2. Then claim ongoing LP fees: "Claim CPMM fees for TOKEN"

### Fee Key NFTs

Fee Key NFTs represent the right to claim LP trading fees after migration.

**How They Work:**
- Created when token migrates from bonding curve to CPMM
- Represent 50% share of LP trading fees
- Standard SPL token (decimals=0, amount=1)
- Transferable (with restrictions for permanent arrangements)

**Managing Fee Rights:**
- View your NFTs: "Show my Fee Key NFTs"
- Transfer to another wallet: "Transfer fees for TOKEN to ADDRESS"
- Claim if designated recipient: "Claim my fee NFT for TOKEN"

### Fee Recipient (Permanent Arrangements)

Specify a `feeRecipient` to route creator fees to a different wallet.

**How It Works:**
1. Launch token with `feeRecipient` address
2. During bonding curve: EITHER party can claim fees
3. Fees split automatically: 99.9% to recipient, 0.1% to creator
4. After migration: recipient claims Fee Key NFT
5. Recipient uses "Claim CPMM fees" for ongoing LP fees

**Important:**
- Creates a PERMANENT arrangement
- Deployer CANNOT transfer their Fee Key NFT
- Only the designated recipient can claim the NFT
- Use for treasuries, DAOs, collaborators, or charity

**Who Can Claim During Bonding Curve:**
- Token creator (deployer)
- Designated fee recipient
- Either party initiates, fees split automatically

### Vesting Parameters

Optional vesting for team tokens or investor allocations.

| Parameter | Description | Example |
|-----------|-------------|---------|
| Cliff Period | Time before any tokens unlock | 30 days = 2592000 seconds |
| Unlock Period | Time for gradual unlock after cliff | 90 days = 7776000 seconds |
| Locked Amount | Total tokens to lock | In token units with decimals |

### Gas Fees

| Operation | Cost | Sponsored? |
|-----------|------|------------|
| Token Launch | ~0.01-0.02 SOL | Yes (within limits) |
| Standard Fee Claim | ~0.005 SOL | No |
| Shared Fee Claim | ~0.005 SOL | Yes (always) |
| Transfer Fee Rights | ~0.005 SOL | No |
| Claim Fee NFT | ~0.005 SOL | No |

Gas is sponsored for token launches within daily limits (1/day standard, 10/day Bankr Club).
Shared fee claims are always sponsored to ensure atomic claim+transfer.

### Rate Limits

| User Type | Daily Limit | Gas Sponsored |
|-----------|-------------|---------------|
| Standard Users | Unlimited | 1 token/day |
| Bankr Club Members | Unlimited | 10 tokens/day |

Users can launch additional tokens beyond sponsored limits by paying ~0.01 SOL gas.

---

## EVM Token Launches (Clanker)

Deploy ERC20 tokens on Base and Unichain using Clanker.

### Deployment Parameters

| Parameter | Required | Description | Example |
|-----------|----------|-------------|---------|
| **Name** | Yes | Full token name | "My Token" |
| **Symbol** | Yes | Ticker, 3-5 chars | "MTK" |
| **Description** | No | Token description | "A community token" |
| **Image** | No | Logo URL or upload | URL or file |
| **Website** | No | Project website | "myproject.com" |
| **Twitter** | No | Twitter/X handle | "@myproject" |
| **Telegram** | No | Telegram group | "@mytoken" |

### Prompt Examples

**Deploy tokens:**
- "Deploy a token called BankrFan with symbol BFAN"
- "Create a memecoin: name=DogeKiller, symbol=DOGEK"
- "Deploy token with website myproject.com and Twitter @myproject"
- "Create a token on Base"
- "Launch new token on Unichain"

**Claim fees:**
- "Claim fees for my token MTK"
- "Check my Clanker fees"
- "Claim legacy Clanker fees"

**Update metadata:**
- "Update description for MyToken"
- "Add Twitter link to my token"
- "Update logo for MyToken"

### Rate Limits

| User Type | Daily Limit |
|-----------|-------------|
| Standard Users | 1 token/day |
| Bankr Club Members | 10 tokens/day |

### Fee Structure

- Small fee on each trade, accumulated for token creator
- Claimable anytime via "Claim fees for my token"
- Legacy fees (older Clanker versions) claimed separately

### Chain Selection

**Base (Recommended):**
- Primary Clanker support
- Low deployment cost
- Growing ecosystem
- Easy liquidity provision

**Unichain:**
- Secondary option
- Low cost
- Newer ecosystem
- Less liquidity

### Deployment Process

1. **Specify Parameters**: Name, symbol (required); description, social links (optional)
2. **Contract Deployment**: Clanker deploys audited ERC20 contract with automatic liquidity
3. **Verification**: Get contract address, view on block explorer

---

## Common Issues

| Issue | Chain | Resolution |
|-------|-------|------------|
| Rate limit reached | Both | Wait 24 hours or upgrade to Bankr Club |
| Name/symbol taken | EVM | Choose different name |
| Insufficient SOL | Solana | Add SOL for gas fees |
| NFT not found | Solana | Token may still be on bonding curve |
| Cannot transfer NFT | Solana | Permanent fee arrangement exists |
| No fees to claim | Solana | No trades yet or recently claimed |
| Token migrated | Solana | Use CPMM fee claiming instead |

## Best Practices

### Before Deploying
1. **Choose unique name/symbol** — Check availability
2. **Prepare branding** — Logo, description ready
3. **Choose right chain** — Solana for bonding curves, Base for ERC20
4. **Understand fees** — Know the fee structure for your chain

### During Deployment
1. **Solana**: Only tokenName is required — don't over-specify
2. **EVM**: Add metadata and social links immediately
3. **Save addresses** — Token address and any NFT mints

### After Deployment
1. **Check fee status** — "How much fees can I claim for TOKEN?"
2. **Claim fees regularly** — Don't leave money unclaimed
3. **Monitor migration** (Solana) — Fee Key NFT created at migration
4. **Engage community** — Marketing and updates

## Security Considerations

### Solana (LaunchLab)
- Bonding curve prevents rug pulls (liquidity locked)
- LP is automatically locked at migration
- Fee Key NFTs are standard SPL tokens
- Permanent fee arrangements are immutable
- Shared fee claims use atomic transactions (claim+transfer)

### EVM (Clanker)
- Uses audited contracts
- Standard ERC20 implementation
- Verifiable on block explorer
- Creator controls metadata

## Legal Considerations

**Disclaimer:**
- Token deployment may have legal implications
- Consider securities laws in your jurisdiction
- Consult legal counsel for serious projects
- Be transparent with community
- Don't make price promises

---

**Solana Tip**: Just say "Launch TOKEN_NAME" — only the name is required. Symbol defaults to name, and the bonding curve handles everything else.

**Fee Claiming Tip**: Both creator and fee recipient can claim fees during bonding curve. Just say "Claim my fees for TOKEN" — the system handles the split automatically.

**EVM Tip**: Add social links during deployment for better discoverability on aggregators.



## 📖 Reference: token-trading.md

# Token Trading Reference

Execute token trades and swaps across multiple blockchains.

## Supported Chains

| Chain | Native Token | Characteristics |
|-------|--------------|-----------------|
| Base | ETH | Low fees, ideal for memecoins |
| Polygon | MATIC | Fast, cheap transactions |
| Ethereum | ETH | Highest liquidity, expensive gas |
| Unichain | ETH | Newer L2 option |
| Solana | SOL | High speed, minimal fees |

## Amount Formats

| Format | Example | Description |
|--------|---------|-------------|
| USD | `$50` | Dollar amount to spend |
| Percentage | `50%` | Percentage of your balance |
| Exact | `0.1 ETH` | Specific token amount |

## Prompt Examples

**Same-chain swaps:**
- "Swap 0.1 ETH for USDC on Base"
- "Buy $50 of BNKR on Base"
- "Sell 50% of my ETH holdings"
- "Purchase 100 USDC worth of PEPE"

**Cross-chain swaps:**
- "Bridge 0.5 ETH from Ethereum to Base"
- "Move 100 USDC from Polygon to Solana"

**ETH/WETH conversion:**
- "Convert 0.1 ETH to WETH"
- "Unwrap 0.5 WETH to ETH"

## Chain Selection

- If no chain specified, Bankr selects the most appropriate chain
- Base is preferred for most operations due to low fees
- Cross-chain routes are automatically optimized
- Include chain name in prompt to specify: "Buy ETH on Polygon"

## Slippage

- Default slippage tolerance is applied automatically
- For volatile tokens, Bankr adjusts slippage as needed
- If slippage is exceeded, the transaction fails safely
- You can specify: "with 1% slippage"

## Common Issues

| Issue | Resolution |
|-------|------------|
| Insufficient balance | Reduce amount or add funds |
| Token not found | Check token symbol/address, specify chain |
| High slippage | Try smaller amounts or use limit orders |
| Network congestion | Wait and retry, or try L2 |
| Gas too high | Use Base/Polygon, or wait for lower gas |

## Best Practices

1. **Start small** - Test with small amounts first
2. **Specify chains** - For lesser-known tokens, always include chain
3. **Check slippage** - Be careful with low-liquidity tokens
4. **Monitor gas** - Ethereum mainnet can be expensive
5. **Use L2s** - Base and Polygon offer much lower fees



## 📖 Reference: transfers.md

# Transfers Reference

Transfer tokens to addresses, ENS names, or social handles.

## Supported Transfers

- **EVM Chains**: Base, Polygon, Ethereum, Unichain
  - Native tokens: ETH, MATIC
  - ERC20 tokens: USDC, USDT, WETH, etc.
- **Solana**: SOL and SPL tokens

## Recipient Formats

| Format | Example | Description |
|--------|---------|-------------|
| Address | `0x1234...abcd` | Direct wallet address (EVM) |
| Address | `9x...abc` | Direct wallet address (Solana) |
| ENS | `vitalik.eth` | Ethereum Name Service |
| Twitter | `@elonmusk` | Twitter/X username |
| Farcaster | `@dwr.eth` | Farcaster username |
| Telegram | `@username` | Telegram handle |

**Social Handle Resolution**: Handles are resolved to linked wallet addresses before sending. User must have linked their wallet to the social platform.

## Amount Formats

| Format | Example | Description |
|--------|---------|-------------|
| USD | `$50` | Dollar amount |
| Percentage | `50%` | Percentage of balance |
| Exact | `0.1 ETH` | Specific amount |

## Prompt Examples

**To addresses:**
- "Send 0.5 ETH to 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
- "Transfer 100 USDC to 9xKc...abc"
- "Send $20 of ETH to 0x1234..."

**To ENS:**
- "Send 1 ETH to vitalik.eth"
- "Transfer $50 of USDC to mydomain.eth"
- "Send 10 USDC to friend.eth"

**To social handles:**
- "Send $20 of ETH to @friend on Twitter"
- "Transfer 0.1 ETH to @user on Farcaster"
- "Send 50 USDC to @buddy on Telegram"

**With chain specified:**
- "Send ETH on Base to vitalik.eth"
- "Send 10% of my ETH to @friend"
- "Transfer USDC on Polygon to 0x..."

## Chain Selection

If not specified, Bankr selects automatically based on:
- Recipient activity patterns
- Gas costs
- Token availability
- Liquidity

Specify chain in prompt if you need a specific network.

## Common Issues

| Issue | Resolution |
|-------|------------|
| ENS not found | Verify the ENS name exists and is registered |
| Social handle not found | Check username spelling and platform |
| No linked wallet | User hasn't linked wallet to their social account |
| Insufficient balance | Reduce amount or ensure enough funds |
| Wrong chain | Specify chain explicitly in prompt |
| Gas required | Ensure you have native token for gas |

## Security Notes

- **Verify recipient** - Always double-check before confirming
- **Address preview** - Social handle resolution shows the resolved address
- **Irreversible** - Blockchain transactions cannot be undone
- **Large transfers** - May require additional confirmation
- **Test first** - Send small amount first for new recipients

## Best Practices

1. **Start small** - Test with small amounts for new recipients
2. **Verify address** - Double-check resolved addresses
3. **Check chain** - Ensure recipient uses the same chain
4. **Gas buffer** - Keep some native token for future transactions
5. **ENS preferred** - More reliable than social handles
6. **Screenshot** - Save transaction hash for records

