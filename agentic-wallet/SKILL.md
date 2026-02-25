---
name: agentic-wallet
version: 1.0.0
description: Give your agent a standalone on-chain wallet for gasless trades, funding, and autonomous payments.
author: aimaneth
license: MIT
tags:
  - crypto
  - wallet
  - coinbase
  - base
  - trading
metadata: {"zeptoclaw":{"emoji":"💳","requires":{"anyBins":["npx"]}}}
---

# Agentic Wallet Skill

Give a ZeptoClaw agent its own standalone on-chain wallet natively controlled by the agent. Powered by Coinbase Developer Platform (CDP).

With this skill, an agent can:
- Hold stablecoins (USDC) and other tokens on the Base network
- Execute trades and swaps autonomously, completely **gasless**
- Send funds to other addresses
- Pay for external AI APIs via the x402 machine-to-machine payment protocol

## Check Wallet Status

Check if the agent is already authenticated and has a wallet instance. 

```bash
npx awal status
```

If it returns "Not configured", you need to authenticate first.

## Authentication

Create a new wallet or log into an existing one via Email OTP.

```bash
npx awal login
```

The CLI will prompt you to enter an email address and then an OTP code sent to that email. **Note:** As an agent, you must ask the user for their email and the OTP code to complete this step, or use an email you have access to.

## Fund the Wallet

Get the wallet's Base network address to send funds to.

```bash
# Returns the 0x address of the agent's wallet
npx awal address
```

Tell the user to send Base ETH, USDC, or other Base network tokens to this address.

## Send Funds

Send tokens to any EVM address on Base. **Gasless**.

```bash
# Send 5 USDC to vitalik.eth
npx awal send 5 usdc vitalik.eth

# Send 0.01 WETH to a specific address
npx awal send 0.01 weth 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
```

## Trade / Swap Tokens

Swap exact amounts of tokens directly from the agent's wallet on the Base network. **Gasless**.

```bash
# Swap 10 USDC into ETH
npx awal trade 10 usdc eth

# Swap 0.05 WETH into DEGEN
npx awal trade 0.05 weth degen
```

## Pay for APIs (x402)

Agentic wallets support the **x402** protocol for machine-to-machine commerce. When an agent encounters an API that returns a HTTP `402 Payment Required` status code with an x402 header, the agent can use `awal` to pay for the invoice autonomously.

```bash
# Pay an x402 invoice to access a paid API
npx awal pay <invoice_string>
```

## Tips

- All operations through `awal` occur on the **Base** network. Make sure the wallet is funded on Base.
- Transactions like sending or trading tokens are **gasless** (sponsored by Coinbase), so the agent doesn't need to hold ETH just to pay for gas to send USDC.
