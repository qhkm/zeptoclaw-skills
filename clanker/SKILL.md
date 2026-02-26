---
name: clanker
version: 1.0.0
description: Deploy ERC20 tokens on Base, Ethereum, Arbitrum, and other EVM chains using the Clanker SDK. Use when the user wants to deploy a new token, create a memecoin, set up token vesting, configure airdrops, manage token rewards, claim LP fees, or update token metadata. Supports V4 deployment with vaults, airdrops, dev buys, custom market caps, vanity addresses, and multi-chain deployment.
author: BankrBot
license: MIT
tags:
  - crypto
  - defi
  - token-deployment
  - base
metadata: {"zeptoclaw":{"emoji":"🚀","requires":{"anyBins":["npm", "yarn", "pnpm"]}}}
---

# Clanker SDK

Deploy production-ready ERC20 tokens with built-in liquidity pools using the official Clanker TypeScript SDK.


## 🚨 Financial Safety Guardrails

This skill enables high-risk financial operations (token deployment requires gas, dev buys require funds, and liquidity parameters are immutable). You MUST adhere to the following safety rules:

- **Always confirm with the user before executing the deployment script.**
- **Display all parameters clearly before execution:** Token Name, Symbol, Total Supply, Vault allocations, Dev buy amounts, and target chain.
- **Double-check dev buy amounts:** Do NOT execute large dev buys without explicit double-confirmation from the user.
- **Warn users about irreversibility:** Smart contract deployments are permanent and cannot be deleted once broadcast to the blockchain.

## Overview

Clanker is a token deployment protocol that creates ERC20 tokens with Uniswap V4 liquidity pools in a single transaction. The SDK provides a TypeScript interface for deploying tokens with advanced features like vesting, airdrops, and customizable reward distribution.

## Quick Start

### Installation

```bash
npm install clanker-sdk viem
# or
yarn add clanker-sdk viem
# or
pnpm add clanker-sdk viem
```

### Environment Setup

Create a `.env` file with your private key:

```bash
PRIVATE_KEY=0x...your_private_key_here
```

### Basic Token Deployment

```typescript
import { Clanker } from 'clanker-sdk';
import { createPublicClient, createWalletClient, http, type PublicClient } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

const PRIVATE_KEY = process.env.PRIVATE_KEY as `0x${string}`;
const account = privateKeyToAccount(PRIVATE_KEY);

const publicClient = createPublicClient({
  chain: base,
  transport: http(),
}) as PublicClient;

const wallet = createWalletClient({
  account,
  chain: base,
  transport: http(),
});

const clanker = new Clanker({ wallet, publicClient });

const { txHash, waitForTransaction, error } = await clanker.deploy({
  name: 'My Token',
  symbol: 'TKN',
  image: 'ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi',
  tokenAdmin: account.address,
  metadata: {
    description: 'My awesome token',
  },
  context: {
    interface: 'Clanker SDK',
  },
  vanity: true,
});

if (error) throw error;

const { address: tokenAddress } = await waitForTransaction();
console.log('Token deployed at:', tokenAddress);
```

## Core Capabilities

### 1. Token Deployment

Deploy tokens with full customization including metadata, social links, and pool configuration.

**Basic deployment:**
- Token name, symbol, and image (IPFS)
- Description and social media links
- Vanity address generation
- Custom pool configurations

**Reference:** [references/deployment.md](references/deployment.md)

### 2. Vault (Token Vesting)

Lock a percentage of tokens with lockup and vesting periods:

```typescript
vault: {
  percentage: 10,           // 10% of token supply
  lockupDuration: 2592000,  // 30 days cliff (in seconds)
  vestingDuration: 2592000, // 30 days linear vesting
  recipient: account.address,
}
```

**Reference:** [references/vesting.md](references/vesting.md)

### 3. Airdrops

Distribute tokens to multiple addresses using Merkle tree proofs:

```typescript
import { createAirdrop, registerAirdrop } from 'clanker-sdk/v4/extensions';

const { tree, airdrop } = createAirdrop([
  { account: '0x...', amount: 200_000_000 },
  { account: '0x...', amount: 50_000_000 },
]);

// Include in deployment
airdrop: {
  ...airdrop,
  lockupDuration: 86_400,  // 1 day
  vestingDuration: 86_400, // 1 day
}
```

**Reference:** [references/airdrops.md](references/airdrops.md)

### 4. Rewards Configuration

Configure trading fee distribution:

```typescript
rewards: {
  recipients: [
    {
      recipient: account.address,
      admin: account.address,
      bps: 5000,      // 50% of fees
      token: 'Both',  // Receive both tokens
    },
    {
      recipient: '0x...',
      admin: '0x...',
      bps: 5000,      // 50% of fees
      token: 'Both',
    },
  ],
}
```

#### Token Type Options

Choose which tokens each recipient receives from trading fees:

| Token Type | Description |
|------------|-------------|
| `'Clanker'` | Receive only the deployed token |
| `'Paired'` | Receive only the paired token (e.g., WETH) |
| `'Both'` | Receive both tokens |

#### Default Bankr Interface Fee

When deploying via Bankr, use this default rewards configuration with 20% interface fee:

```typescript
// Bankr interface fee recipient
const BANKR_INTERFACE_ADDRESS = '0xF60633D02690e2A15A54AB919925F3d038Df163e';

rewards: {
  recipients: [
    {
      recipient: account.address,           // Creator
      admin: account.address,
      bps: 8000,                            // 80% to creator
      token: 'Paired',                      // Receive paired token (WETH)
    },
    {
      recipient: BANKR_INTERFACE_ADDRESS,   // Bankr interface
      admin: BANKR_INTERFACE_ADDRESS,
      bps: 2000,                            // 20% to Bankr
      token: 'Paired',                      // Receive paired token (WETH)
    },
  ],
}
```

**Reference:** [references/rewards.md](references/rewards.md)

### 5. Dev Buy

Include an initial token purchase in the deployment:

```typescript
devBuy: {
  ethAmount: 0.1,           // Buy with 0.1 ETH
  recipient: account.address,
}
```

### 6. Custom Market Cap

Set initial token price/market cap:

```typescript
import { getTickFromMarketCap } from 'clanker-sdk';

const customPool = getTickFromMarketCap(5); // 5 ETH market cap

pool: {
  ...customPool,
  positions: [
    {
      tickLower: customPool.tickIfToken0IsClanker,
      tickUpper: -120000,
      positionBps: 10_000,
    },
  ],
}
```

**Reference:** [references/pool-config.md](references/pool-config.md)

### 7. Anti-Sniper Protection

Configure fee decay to protect against snipers:

```typescript
sniperFees: {
  startingFee: 666_777,    // 66.6777% starting fee
  endingFee: 41_673,       // 4.1673% ending fee
  secondsToDecay: 15,      // 15 seconds decay
}
```

## Contract Limits & Constants

| Parameter | Value | Notes |
|-----------|-------|-------|
| Token Supply | 100 billion | Fixed at 100,000,000,000 with 18 decimals |
| Max Extension BPS | 9000 (90%) | Max tokens to extensions, min 10% to LP |
| Max Extensions | 10 | Maximum number of extensions per deployment |
| Vault Min Lockup | 7 days | Minimum lockup duration for vesting |
| Airdrop Min Lockup | 1 day | Minimum lockup duration for airdrops |
| Max LP Fee | 10% | Normal trading fee cap |
| Max Sniper Fee | 80% | Maximum MEV/sniper protection fee |
| Sniper Fee Decay | 2 minutes max | Maximum time for sniper fee decay |
| Max Reward Recipients | 7 | Maximum fee distribution recipients |
| Max LP Positions | 7 | Maximum liquidity positions |

## Supported Chains

| Chain | Chain ID | Native Token | Status |
|-------|----------|--------------|--------|
| Base | 8453 | ETH | ✅ Full support |
| Ethereum | 1 | ETH | ✅ Full support |
| Arbitrum | 42161 | ETH | ✅ Full support |
| Unichain | - | ETH | ✅ Full support |
| Monad | - | MON | ✅ Static fees only |

## Post-Deployment Operations

### Claim Vaulted Tokens

```typescript
const claimable = await clanker.getVaultClaimableAmount({ token: TOKEN_ADDRESS });

if (claimable > 0n) {
  const { txHash } = await clanker.claimVaultedTokens({ token: TOKEN_ADDRESS });
}
```

### Collect Trading Rewards

```typescript
// Check available rewards
const availableFees = await clanker.availableRewards({
  token: TOKEN_ADDRESS,
  rewardRecipient: FEE_OWNER_ADDRESS,
});

// Claim rewards
const { txHash } = await clanker.claimRewards({
  token: TOKEN_ADDRESS,
  rewardRecipient: FEE_OWNER_ADDRESS,
});
```

### Update Token Metadata

```typescript
const metadata = JSON.stringify({
  description: 'Updated description',
  socialMediaUrls: [
    { platform: 'twitter', url: 'https://twitter.com/mytoken' },
    { platform: 'telegram', url: 'https://t.me/mytoken' },
  ],
});

const { txHash } = await clanker.updateMetadata({
  token: TOKEN_ADDRESS,
  metadata,
});
```

### Update Token Image

```typescript
const { txHash } = await clanker.updateImage({
  token: TOKEN_ADDRESS,
  image: 'ipfs://new_image_hash',
});
```

## Common Workflows

### Simple Memecoin Launch

1. Prepare token image (upload to IPFS)
2. Deploy with basic config (name, symbol, image)
3. Enable vanity address for memorable contract
4. Share contract address

### Community Token with Airdrop

1. Compile airdrop recipient list
2. Create Merkle tree with `createAirdrop()`
3. Deploy token with airdrop extension
4. Register airdrop with Clanker service
5. Share claim instructions

### Creator Token with Vesting

1. Deploy with vault configuration
2. Set lockup period (cliff)
3. Set vesting duration
4. Claim tokens as they vest

## Full Deployment Config

```typescript
// Bankr interface fee recipient (20%)
const BANKR_INTERFACE_ADDRESS = '0xF60633D02690e2A15A54AB919925F3d038Df163e';

const tokenConfig = {
  chainId: 8453,                    // Base
  name: 'My Token',
  symbol: 'TKN',
  image: 'ipfs://...',
  tokenAdmin: account.address,
  
  metadata: {
    description: 'Token description',
    socialMediaUrls: [
      { platform: 'twitter', url: '...' },
      { platform: 'telegram', url: '...' },
    ],
  },
  
  context: {
    interface: 'Bankr',
    platform: 'farcaster',
    messageId: '',
    id: '',
  },
  
  vault: {
    percentage: 10,
    lockupDuration: 2592000,
    vestingDuration: 2592000,
    recipient: account.address,
  },
  
  devBuy: {
    ethAmount: 0,
    recipient: account.address,
  },
  
  // Default: 80% creator, 20% Bankr interface (all in paired token)
  rewards: {
    recipients: [
      { 
        recipient: account.address,
        admin: account.address,
        bps: 8000,  // 80% to creator
        token: 'Paired',  // Receive paired token (WETH)
      },
      { 
        recipient: BANKR_INTERFACE_ADDRESS,
        admin: BANKR_INTERFACE_ADDRESS,
        bps: 2000,  // 20% to Bankr
        token: 'Paired',  // Receive paired token (WETH)
      },
    ],
  },
  
  pool: {
    pairedToken: '0x4200000000000000000000000000000000000006', // WETH
    positions: 'Standard',
  },
  
  fees: 'StaticBasic',
  vanity: true,
  
  sniperFees: {
    startingFee: 666_777,
    endingFee: 41_673,
    secondsToDecay: 15,
  },
};
```

## Best Practices

### Security

1. **Never expose private keys** - Use environment variables
2. **Test on testnet first** - Verify configs before mainnet
3. **Simulate transactions** - Use `*Simulate` methods before execution
4. **Verify addresses** - Double-check all recipient addresses

### Token Design

1. **Choose meaningful names** - Clear, memorable token identity
2. **Use quality images** - High-res, appropriate IPFS images
3. **Configure vesting wisely** - Align with project timeline

### Gas Optimization

1. **Use Base or Arbitrum** - Lower gas fees
2. **Batch operations** - Combine when possible
3. **Monitor gas prices** - Deploy during low-traffic periods

## Troubleshooting

### Common Issues

- **"Missing PRIVATE_KEY"** - Set environment variable
- **"Insufficient balance"** - Fund wallet with native token
- **"Transaction reverted"** - Check parameters, simulate first
- **"Invalid image"** - Ensure IPFS hash is accessible

### Debug Steps

1. Check wallet balance
2. Verify chain configuration
3. Use simulation methods
4. Check transaction on block explorer
5. Review error message details

## Resources

- **GitHub**: [github.com/clanker-devco/clanker-sdk](https://github.com/clanker-devco/clanker-sdk)
- **NPM**: [npmjs.com/package/clanker-sdk](https://www.npmjs.com/package/clanker-sdk)
- **Examples**: [github.com/clanker-devco/clanker-sdk/tree/main/examples/v4](https://github.com/clanker-devco/clanker-sdk/tree/main/examples/v4)

---

**💡 Pro Tip**: Always use the `vanity: true` option for memorable contract addresses.

**⚠️ Security**: Never commit private keys. Use `.env` files and add them to `.gitignore`.

**🚀 Quick Win**: Start with the simple deployment example, then add features like vesting and rewards as needed.


---

# 📚 Advanced Reference Guide

> The following sections were inlined from the former `references/` directory.



## 📖 Reference: airdrops.md

# Airdrops

Distribute tokens to multiple addresses using Merkle tree proofs with the Clanker airdrop extension.

## Overview

Airdrops allow you to allocate tokens to specific addresses during deployment. Key features:
- **Merkle tree verification** - Efficient on-chain proof verification
- **Lockup/vesting** - Control when tokens become claimable
- **Clanker service** - Optional proof storage and generation

## Create an Airdrop

```typescript
import { createAirdrop, registerAirdrop } from 'clanker-sdk/v4/extensions';

// Define recipients and amounts
const { tree, airdrop } = createAirdrop([
  {
    account: '0x308112D06027Cd838627b94dDFC16ea6B4D90004',
    amount: 200_000_000, // Amount in token units
  },
  {
    account: '0x1eaf444ebDf6495C57aD52A04C61521bBf564ace',
    amount: 50_000_000,
  },
  {
    account: '0x04F6ef12a8B6c2346C8505eE4Cff71C43D2dd825',
    amount: 10_000_000,
  },
]);
```

## Deploy with Airdrop

Include the airdrop in your token deployment:

```typescript
import { Clanker } from 'clanker-sdk';
import { createAirdrop } from 'clanker-sdk/v4/extensions';

const { tree, airdrop } = createAirdrop([
  { account: '0x...', amount: 200_000_000 },
  { account: '0x...', amount: 50_000_000 },
]);

const { txHash, waitForTransaction, error } = await clanker.deploy({
  name: 'Airdrop Token',
  symbol: 'DROP',
  image: 'ipfs://...',
  tokenAdmin: account.address,
  
  metadata: {
    description: 'Token with an airdrop',
  },
  
  context: {
    interface: 'Clanker SDK',
  },
  
  airdrop: {
    ...airdrop,
    lockupDuration: 86_400,  // 1 day lockup
    vestingDuration: 86_400, // 1 day vesting
  },
  
  vanity: true,
});

if (error) throw error;

const { address } = await waitForTransaction();
console.log('Token deployed at:', address);
```

## Register Airdrop with Clanker Service

Store the Merkle tree with Clanker's service for easy proof generation:

```typescript
import { registerAirdrop, fetchAirdropProofs } from 'clanker-sdk/v4/extensions';
import { sleep } from 'bun'; // or setTimeout

// Wait for token to be indexed (minimum 10 seconds)
await sleep(10_000);

// Register the airdrop tree
await registerAirdrop(tokenAddress, tree);
console.log('Airdrop registered!');
```

## Fetch Airdrop Proofs

Get proofs for a specific address:

```typescript
import { fetchAirdropProofs } from 'clanker-sdk/v4/extensions';

const { proofs } = await fetchAirdropProofs(
  tokenAddress,
  '0x308112D06027Cd838627b94dDFC16ea6B4D90004'
);

console.log('Proofs:', proofs);
// [{ proof: [...], entry: { account: '0x...', amount: 200000000 } }]
```

## Claim Airdrop Tokens

Build and execute a claim transaction:

```typescript
import { getClaimAirdropTransaction } from 'clanker-sdk/v4/extensions';

const { proof, entry } = proofs[0];

const tx = getClaimAirdropTransaction({
  chainId: base.id,
  token: tokenAddress,
  recipient: entry.account,
  amount: entry.amount,
  proof,
});

// Execute the transaction
const hash = await wallet.sendTransaction(tx);
```

## Self-Managed Tree Storage

If you don't want to use the Clanker service, store and manage the tree yourself:

```typescript
import { StandardMerkleTree } from '@openzeppelin/merkle-tree';
import fs from 'fs';

// After creating the airdrop
const { tree, airdrop } = createAirdrop([...]);

// Save the tree to a file
fs.writeFileSync('merkle-tree.json', JSON.stringify(tree.dump()));

// Later, load and use the tree
const loadedTree = StandardMerkleTree.load(
  JSON.parse(fs.readFileSync('merkle-tree.json', 'utf8'))
);
```

## Contract Limits

From the Solidity contracts:

- **Minimum Lockup Duration**: 1 day (86,400 seconds) - enforced on-chain
- **Maximum Extension BPS**: 9000 (90% of supply can go to extensions total)

```typescript
airdrop: {
  ...airdrop,
  lockupDuration: 86_400,  // Minimum 1 day required
  vestingDuration: 0,      // No minimum for vesting
}
```

**Note**: Unlike vault (7 days min), airdrop only requires 1 day minimum lockup.

## Airdrop with Extended Vesting

For gradual distribution:

```typescript
const THIRTY_DAYS = 2592000;

airdrop: {
  ...airdrop,
  lockupDuration: THIRTY_DAYS,      // 30-day cliff
  vestingDuration: THIRTY_DAYS * 3, // 90-day vesting
}
```

## Complete Airdrop Workflow

```typescript
import { sleep } from 'bun';
import { createPublicClient, createWalletClient, http, type PublicClient } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';
import {
  createAirdrop,
  fetchAirdropProofs,
  getClaimAirdropTransaction,
  registerAirdrop,
} from 'clanker-sdk/v4/extensions';
import { Clanker } from 'clanker-sdk/v4';

// Setup
const PRIVATE_KEY = process.env.PRIVATE_KEY as `0x${string}`;
const account = privateKeyToAccount(PRIVATE_KEY);
const publicClient = createPublicClient({ chain: base, transport: http() }) as PublicClient;
const wallet = createWalletClient({ account, chain: base, transport: http() });
const clanker = new Clanker({ publicClient, wallet });

// 1. Create the airdrop
const { tree, airdrop } = createAirdrop([
  { account: '0x308112D06027Cd838627b94dDFC16ea6B4D90004', amount: 200_000_000 },
  { account: '0x1eaf444ebDf6495C57aD52A04C61521bBf564ace', amount: 50_000_000 },
  { account: '0x04F6ef12a8B6c2346C8505eE4Cff71C43D2dd825', amount: 10_000_000 },
]);

// 2. Deploy the token
const { txHash, waitForTransaction, error } = await clanker.deploy({
  name: 'Airdrop Token',
  symbol: 'DROP',
  image: 'ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi',
  tokenAdmin: account.address,
  metadata: { description: 'Token with an airdrop' },
  context: { interface: 'Clanker SDK' },
  airdrop: {
    ...airdrop,
    lockupDuration: 86_400,
    vestingDuration: 86_400,
  },
  vanity: true,
});

if (error) throw error;

const { address, error: txError } = await waitForTransaction();
if (txError) throw txError;

console.log(`Token deployed at: ${address}`);

// 3. Wait for indexing, then register
console.log('Waiting for indexing...');
await sleep(10_000);

await registerAirdrop(address, tree);
console.log('Airdrop registered!');

// 4. Fetch proofs for claiming
const { proofs } = await fetchAirdropProofs(
  address,
  '0x308112D06027Cd838627b94dDFC16ea6B4D90004'
);

console.log('Proofs ready for claiming:', proofs);
```

## Best Practices

1. **Verify recipient addresses** - Double-check all addresses before deployment
2. **Test with small amounts** - Verify the flow on testnet first
3. **Secure tree storage** - Back up the Merkle tree if self-managing
4. **Reasonable lockup** - Balance between anti-dump and user experience
5. **Communicate claim process** - Provide clear instructions to recipients



## 📖 Reference: deployment.md

# Token Deployment

Complete guide to deploying tokens with the Clanker SDK.

## Simple Deployment

The minimal configuration for deploying a token:

```typescript
import { Clanker } from 'clanker-sdk';
import { createPublicClient, createWalletClient, http, type PublicClient } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

const PRIVATE_KEY = process.env.PRIVATE_KEY as `0x${string}`;
const account = privateKeyToAccount(PRIVATE_KEY);

const publicClient = createPublicClient({
  chain: base,
  transport: http(),
}) as PublicClient;

const wallet = createWalletClient({
  account,
  chain: base,
  transport: http(),
});

const clanker = new Clanker({ wallet, publicClient });

const { txHash, waitForTransaction, error } = await clanker.deploy({
  name: 'My Token',
  symbol: 'TKN',
  image: 'ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi',
  tokenAdmin: account.address,
  chainId: base.id,
  metadata: {
    description: 'My really cool token',
  },
  context: {
    interface: 'Clanker SDK',
  },
  vanity: true,
});

if (error) throw error;

console.log(`Deploying... ${base.blockExplorers.default.url}/tx/${txHash}`);
const { address: tokenAddress } = await waitForTransaction();
console.log(`Done! ${base.blockExplorers.default.url}/address/${tokenAddress}`);
```

## Multi-Chain Deployment

Deploy to different chains by changing the chain configuration:

```typescript
import { base, mainnet, arbitrum, unichain } from 'viem/chains';

// Chain-specific RPC URLs (optional, for better rate limits)
const RPC_URLS: Record<number, string | undefined> = {
  [mainnet.id]: process.env.RPC_URL_MAINNET,
  [base.id]: process.env.RPC_URL_BASE,
  [arbitrum.id]: process.env.RPC_URL_ARBITRUM,
  [unichain.id]: process.env.RPC_URL_UNICHAIN,
};

// Select chain
const CHAIN = base; // or mainnet, arbitrum, unichain

const publicClient = createPublicClient({
  chain: CHAIN,
  transport: http(RPC_URLS[CHAIN.id]),
}) as PublicClient;

const wallet = createWalletClient({
  account,
  chain: CHAIN,
  transport: http(RPC_URLS[CHAIN.id]),
});

const clanker = new Clanker({ wallet, publicClient });

// Deploy with chainId
const { txHash, waitForTransaction, error } = await clanker.deploy({
  chainId: CHAIN.id,
  name: 'My Token',
  symbol: 'TKN',
  // ... rest of config
});
```

## Token Metadata

Configure rich metadata for your token:

```typescript
metadata: {
  description: 'Token with custom configuration including vesting and rewards',
  socialMediaUrls: [
    { platform: 'twitter', url: 'https://twitter.com/mytoken' },
    { platform: 'telegram', url: 'https://t.me/mytoken' },
    { platform: 'discord', url: 'https://discord.gg/mytoken' },
  ],
  auditUrls: ['https://example.com/audit'],
}
```

## Context Configuration

Track deployment source and social platform info:

```typescript
context: {
  interface: 'Clanker SDK',     // Your app/interface name
  platform: 'farcaster',        // Social platform (farcaster, X, etc.)
  messageId: '',                // Cast hash, tweet URL, etc.
  id: '',                       // FID, X handle, etc.
}
```

## Vanity Addresses

Enable vanity address generation for memorable contract addresses:

```typescript
const { txHash, waitForTransaction, error } = await clanker.deploy({
  // ... other config
  vanity: true,  // SDK will mine for distinctive address
});
```

## Full Configuration Example

```typescript
// Bankr interface fee recipient (20%)
const BANKR_INTERFACE_ADDRESS = '0xF60633D02690e2A15A54AB919925F3d038Df163e';

const { txHash, waitForTransaction, error } = await clanker.deploy({
  chainId: base.id,
  name: 'My Token',
  symbol: 'TKN',
  image: 'ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi',
  tokenAdmin: account.address,
  
  metadata: {
    description: 'Token with custom configuration including vesting and rewards',
    socialMediaUrls: [
      { platform: 'twitter', url: 'https://twitter.com/mytoken' },
      { platform: 'telegram', url: 'https://t.me/mytoken' },
    ],
  },
  
  context: {
    interface: 'Bankr',
    platform: 'farcaster',
    messageId: '',
    id: '',
  },
  
  vault: {
    percentage: 10,
    lockupDuration: 2592000,
    vestingDuration: 2592000,
    recipient: account.address,
  },
  
  devBuy: {
    ethAmount: 0,
    recipient: account.address,
  },
  
  // Default: 80% creator, 20% Bankr interface (all in paired token)
  // Token options: 'Clanker' | 'Paired' | 'Both'
  rewards: {
    recipients: [
      {
        recipient: account.address,
        admin: account.address,
        bps: 8000,  // 80% to creator
        token: 'Paired',  // Receive paired token (WETH)
      },
      {
        recipient: BANKR_INTERFACE_ADDRESS,
        admin: BANKR_INTERFACE_ADDRESS,
        bps: 2000,  // 20% to Bankr
        token: 'Paired',  // Receive paired token (WETH)
      },
    ],
  },
  
  pool: {
    pairedToken: '0x4200000000000000000000000000000000000006', // WETH on Base
    positions: 'Standard',
  },
  
  fees: 'StaticBasic',
  vanity: true,
  
  sniperFees: {
    startingFee: 666_777,
    endingFee: 41_673,
    secondsToDecay: 15,
  },
});
```

## Deploy with Custom Salt

Use a specific salt for deterministic address generation:

```typescript
import { zeroHash } from 'viem';

const { txHash, waitForTransaction, error } = await clanker.deploy({
  // ... config
  salt: zeroHash, // or custom bytes32 value
});
```

## Deploy with USDC Pair

Create a token paired with USDC instead of WETH:

```typescript
const USDC_BASE = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';

const { txHash, waitForTransaction, error } = await clanker.deploy({
  // ... config
  pool: {
    pairedToken: USDC_BASE,
    positions: 'Standard',
  },
});
```

## Deploy with 20 ETH Initial Liquidity

Configure larger initial market cap:

```typescript
import { getTickFromMarketCap } from 'clanker-sdk';

const customPool = getTickFromMarketCap(20); // 20 ETH

const { txHash, waitForTransaction, error } = await clanker.deploy({
  // ... config
  pool: {
    ...customPool,
    positions: [
      {
        tickLower: customPool.tickIfToken0IsClanker,
        tickUpper: -120000,
        positionBps: 10_000,
      },
    ],
  },
});
```

## Error Handling

Always handle deployment errors:

```typescript
const { txHash, waitForTransaction, error } = await clanker.deploy(config);

if (error) {
  console.error('Deployment failed:', error.message);
  process.exit(1);
}

console.log(`Transaction: ${txHash}`);

const { address, error: txError } = await waitForTransaction();

if (txError) {
  console.error('Transaction failed:', txError.message);
  process.exit(1);
}

console.log('Token deployed at:', address);
```

## Simulation Before Deployment

Test deployment without executing:

```typescript
// Note: Check SDK docs for simulate method availability
try {
  await clanker.deploySimulate(config);
  console.log('Simulation successful');
} catch (error) {
  console.error('Simulation failed:', error);
}
```



## 📖 Reference: pool-config.md

# Pool Configuration

Configure Uniswap V4 liquidity pool settings for your Clanker token.

## Overview

Clanker tokens are deployed with Uniswap V4 liquidity pools. You can customize:
- **Paired token** - WETH, USDC, or other tokens
- **Initial market cap** - Starting price/liquidity
- **Pool positions** - Liquidity distribution
- **Fee structure** - Static or dynamic fees
- **Sniper protection** - Fee decay for early trades

## Default Pool Configuration

By default, tokens are paired with WETH using standard positions:

```typescript
const { txHash, waitForTransaction, error } = await clanker.deploy({
  name: 'My Token',
  symbol: 'TKN',
  image: 'ipfs://...',
  tokenAdmin: account.address,
  // Default pool: WETH pair with standard positions
});
```

## Paired Token Options

### WETH (Default)

```typescript
import { WETH_ADDRESSES } from 'clanker-sdk/constants';

pool: {
  pairedToken: WETH_ADDRESSES[base.id], // 0x4200000000000000000000000000000000000006
  positions: 'Standard',
}
```

### USDC

```typescript
const USDC_BASE = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';

pool: {
  pairedToken: USDC_BASE,
  positions: 'Standard',
}
```

## Pool Positions

Choose from preset position configurations:

```typescript
import { POOL_POSITIONS } from 'clanker-sdk/constants';

pool: {
  pairedToken: WETH_ADDRESSES[base.id],
  positions: POOL_POSITIONS.Standard, // Default distribution
  // or
  positions: POOL_POSITIONS.Project,  // Alternative distribution
}
```

## Custom Market Cap

Set a specific initial market cap:

```typescript
import { getTickFromMarketCap } from 'clanker-sdk';

// Get tick values for 5 ETH market cap
const customPool = getTickFromMarketCap(5);

pool: {
  ...customPool,
  positions: [
    {
      tickLower: customPool.tickIfToken0IsClanker,
      tickUpper: -120000,
      positionBps: 10_000, // 100% of liquidity in this position
    },
  ],
}
```

### Market Cap Examples

```typescript
// 5 ETH market cap
const pool5ETH = getTickFromMarketCap(5);

// 10 ETH market cap
const pool10ETH = getTickFromMarketCap(10);

// 20 ETH market cap
const pool20ETH = getTickFromMarketCap(20);
```

## Fee Configurations

Choose between static and dynamic fee structures:

```typescript
import { FEE_CONFIGS } from 'clanker-sdk/constants';

// Static basic fees
fees: FEE_CONFIGS.StaticBasic,

// Dynamic fees (tier 3)
fees: FEE_CONFIGS.Dynamic3,
```

**Note:** Monad chain only supports static fees (`FEE_CONFIGS.StaticBasic`).

## Sniper Protection

Configure fee decay to protect against snipers:

```typescript
sniperFees: {
  startingFee: 666_777,    // 66.6777% starting fee
  endingFee: 41_673,       // 4.1673% ending fee
  secondsToDecay: 15,      // 15 seconds to decay
}
```

### Contract Limits

From the Solidity contracts:

| Parameter | Limit | Notes |
|-----------|-------|-------|
| MAX_MEV_LP_FEE | 800,000 (80%) | Maximum starting fee |
| MAX_LP_FEE | 100,000 (10%) | Normal LP fee cap |
| MAX_MEV_MODULE_DELAY | 120 seconds | Maximum decay time |
| Fee Denominator | 1,000,000 | 100% = 1,000,000 |

**Important**: 
- `startingFee` cannot exceed 800,000 (80%)
- `secondsToDecay` cannot exceed 120 seconds (2 minutes)
- `startingFee` must be greater than `endingFee`

### How Sniper Fees Work

1. Trade immediately after deployment → Pay up to 80% fee
2. Fee decays parabolically over the decay period
3. After decay period → Normal LP fee applies

### Aggressive Sniper Protection

```typescript
sniperFees: {
  startingFee: 800_000,    // 80% starting fee (MAX)
  endingFee: 30_000,       // 3% ending fee
  secondsToDecay: 120,     // 2 minutes decay (MAX)
}
```

### Moderate Sniper Protection

```typescript
sniperFees: {
  startingFee: 666_777,    // ~66.7% starting fee
  endingFee: 41_673,       // ~4.2% ending fee
  secondsToDecay: 15,      // 15 seconds decay
}
```

### Minimal Sniper Protection

```typescript
sniperFees: {
  startingFee: 100_000,    // 10% starting fee
  endingFee: 30_000,       // 3% ending fee
  secondsToDecay: 5,       // 5 seconds decay
}
```

## Full Pool Configuration Example

```typescript
import { getTickFromMarketCap } from 'clanker-sdk';
import { FEE_CONFIGS, WETH_ADDRESSES } from 'clanker-sdk/constants';
import { base } from 'viem/chains';

const customPool = getTickFromMarketCap(10); // 10 ETH market cap

const { txHash, waitForTransaction, error } = await clanker.deploy({
  name: 'Custom Pool Token',
  symbol: 'CPT',
  image: 'ipfs://...',
  tokenAdmin: account.address,
  
  pool: {
    pairedToken: WETH_ADDRESSES[base.id],
    ...customPool,
    positions: [
      {
        tickLower: customPool.tickIfToken0IsClanker,
        tickUpper: -120000,
        positionBps: 10_000,
      },
    ],
  },
  
  fees: FEE_CONFIGS.StaticBasic,
  
  sniperFees: {
    startingFee: 666_777,
    endingFee: 41_673,
    secondsToDecay: 15,
  },
});
```

## Chain-Specific WETH Addresses

```typescript
import { WETH_ADDRESSES } from 'clanker-sdk/constants';

// Access WETH address by chain ID
const wethBase = WETH_ADDRESSES[8453];      // Base
const wethMainnet = WETH_ADDRESSES[1];      // Ethereum
const wethArbitrum = WETH_ADDRESSES[42161]; // Arbitrum
```

## LP Position Limits

From the Solidity contracts:

- **Maximum LP Positions**: 7
- **Position BPS Must Total**: 10,000 (100%)
- **Tick Bounds**: Must be within MIN_TICK and MAX_TICK
- **Tick Spacing**: Ticks must be multiples of the pool's tick spacing
- **Lower Tick Constraint**: tickLower must be >= tickIfToken0IsClanker

```typescript
pool: {
  positions: [
    { tickLower: -60000, tickUpper: -30000, positionBps: 5000 },  // 50%
    { tickLower: -30000, tickUpper: 0, positionBps: 3000 },       // 30%
    { tickLower: 0, tickUpper: 30000, positionBps: 2000 },        // 20%
  ],
}
```

## Best Practices

1. **Start with defaults** - Use standard positions unless you have specific needs
2. **Reasonable market cap** - Don't set unrealistically high initial valuations
3. **Enable sniper protection** - Protect early liquidity from bots
4. **Consider pair token** - WETH for most tokens, USDC for stables
5. **Test configurations** - Verify on testnet before mainnet
6. **Respect contract limits** - Max 7 LP positions, max 80% sniper fee, max 2 min decay



## 📖 Reference: rewards.md

# Rewards Configuration

Configure trading fee distribution for your Clanker token.

## Overview

Clanker tokens generate trading fees from the Uniswap V4 pool. You can configure how these fees are distributed among different recipients.

## Basic Rewards Configuration

```typescript
rewards: {
  recipients: [
    {
      recipient: account.address,    // Address receiving fees
      admin: account.address,        // Address that can update recipient
      bps: 5000,                     // 50% of fees (5000 basis points)
      token: 'Both',                 // Receive both tokens from pair
    },
    {
      recipient: '0x...other...',
      admin: '0x...other...',
      bps: 5000,                     // 50% of fees
      token: 'Both',
    },
  ],
}
```

## Basis Points (bps)

Fees are specified in basis points where:
- 100 bps = 1%
- 1000 bps = 10%
- 5000 bps = 50%
- 10000 bps = 100%

Total rewards must equal 10000 bps (100%).

## Contract Limits

From the Solidity contracts:

- **Maximum Reward Recipients**: 7
- **Total BPS Must Equal**: 10,000 (100%)
- **No Zero Addresses**: All admin and recipient addresses must be non-zero
- **No Zero Amounts**: Each recipient must have bps > 0

## Token Types

Choose which tokens each recipient receives from trading fees:

| Token Type | Description |
|------------|-------------|
| `'Clanker'` | Receive only the deployed token |
| `'Paired'` | Receive only the paired token (e.g., WETH) |
| `'Both'` | Receive both tokens |

```typescript
recipients: [
  {
    recipient: account.address,
    admin: account.address,
    bps: 10000,
    token: 'Paired',  // 'Clanker', 'Paired', or 'Both'
  },
]
```

**Recommendation**: Use `'Paired'` for simpler fee management since WETH is more liquid and easier to convert.

## Single Recipient

Receive all fees to one address:

```typescript
rewards: {
  recipients: [
    {
      recipient: account.address,
      admin: account.address,
      bps: 10000, // 100%
      token: 'Both',
    },
  ],
}
```

## Multiple Recipients

Split fees between creator and interface:

```typescript
rewards: {
  recipients: [
    {
      recipient: '0x...creator...',
      admin: '0x...creator...',
      bps: 7500, // 75% to creator
      token: 'Both',
    },
    {
      recipient: '0x...interface...',
      admin: '0x...interface...',
      bps: 2500, // 25% to interface
      token: 'Both',
    },
  ],
}
```

## Default Bankr Interface Fee

When deploying via Bankr, use this standard configuration with 20% interface fee:

```typescript
// Bankr interface fee recipient
const BANKR_INTERFACE_ADDRESS = '0xF60633D02690e2A15A54AB919925F3d038Df163e';

rewards: {
  recipients: [
    {
      recipient: account.address,           // Creator receives 80%
      admin: account.address,
      bps: 8000,
      token: 'Paired',                      // Receive paired token (WETH)
    },
    {
      recipient: BANKR_INTERFACE_ADDRESS,   // Bankr receives 20%
      admin: BANKR_INTERFACE_ADDRESS,
      bps: 2000,
      token: 'Paired',                      // Receive paired token (WETH)
    },
  ],
}
```

This is the **default and recommended configuration** for all token deployments via Bankr. Both recipients receive fees in the paired token (e.g., WETH) to simplify fee management.

## Check Available Rewards

Query unclaimed rewards for a recipient:

```typescript
const TOKEN_ADDRESS = '0x...';
const FEE_OWNER_ADDRESS = '0x...';

const availableFees = await clanker.availableRewards({
  token: TOKEN_ADDRESS,
  rewardRecipient: FEE_OWNER_ADDRESS,
});

console.log('Available fees:', availableFees);
```

## Claim Rewards

Claim accumulated trading fees:

```typescript
const TOKEN_ADDRESS = '0x...';
const FEE_OWNER_ADDRESS = '0x...';

const { txHash, error } = await clanker.claimRewards({
  token: TOKEN_ADDRESS,
  rewardRecipient: FEE_OWNER_ADDRESS,
});

if (error) {
  console.error('Claim failed:', error.message);
} else {
  console.log('Rewards claimed:', txHash);
}
```

## Update Reward Recipient

Change where rewards are sent (must be called by admin):

```typescript
const { txHash, error } = await clanker.updateRewardRecipient({
  token: TOKEN_ADDRESS,
  oldRecipient: '0x...old...',
  newRecipient: '0x...new...',
});
```

## Update Reward Admin

Transfer admin rights to a new address:

```typescript
const { txHash, error } = await clanker.updateRewardAdmin({
  token: TOKEN_ADDRESS,
  recipient: '0x...recipient...',
  newAdmin: '0x...newAdmin...',
});
```

## Read-Only Rewards Check

Check rewards without needing a wallet:

```typescript
import { createPublicClient, http, type PublicClient } from 'viem';
import { base } from 'viem/chains';
import { Clanker } from 'clanker-sdk/v4';

const publicClient = createPublicClient({
  chain: base,
  transport: http(),
}) as PublicClient;

// Initialize without wallet (read-only)
const clanker = new Clanker({ publicClient });

const availableFees = await clanker.availableRewards({
  token: TOKEN_ADDRESS,
  rewardRecipient: FEE_OWNER_ADDRESS,
});
```

## Complete Rewards Example

```typescript
import { createPublicClient, createWalletClient, http, type PublicClient } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';
import { Clanker } from 'clanker-sdk/v4';

const PRIVATE_KEY = process.env.PRIVATE_KEY as `0x${string}`;
const account = privateKeyToAccount(PRIVATE_KEY);

const publicClient = createPublicClient({ chain: base, transport: http() }) as PublicClient;
const wallet = createWalletClient({ account, chain: base, transport: http() });

const clanker = new Clanker({ wallet, publicClient });

const TOKEN_ADDRESS = '0x1A84F1eD13C733e689AACffFb12e0999907357F0';
const FEE_OWNER_ADDRESS = '0x46e2c233a4C5CcBD6f48073F8808E0e4b3296477';

// Check available rewards
const availableFees = await clanker.availableRewards({
  token: TOKEN_ADDRESS,
  rewardRecipient: FEE_OWNER_ADDRESS,
});

console.log('Available fees:', availableFees);

// Claim if rewards available
if (availableFees.token0 > 0n || availableFees.token1 > 0n) {
  const { txHash, error } = await clanker.claimRewards({
    token: TOKEN_ADDRESS,
    rewardRecipient: FEE_OWNER_ADDRESS,
  });

  if (error) {
    console.error('Claim failed:', error.message);
  } else {
    console.log('Transaction hash:', txHash);
  }
}
```

## Best Practices

1. **Secure admin addresses** - Use multisig for high-value tokens
2. **Document fee split** - Be transparent with community about distribution
3. **Regular claims** - Don't let rewards accumulate excessively
4. **Test updates** - Verify recipient/admin changes on testnet first
5. **Fair distribution** - Consider community expectations for fee splits



## 📖 Reference: troubleshooting.md

# Troubleshooting

Common issues and solutions when using the Clanker SDK.

## Setup Issues

### "Missing PRIVATE_KEY env var"

**Cause:** Environment variable not set or not in correct format.

**Solution:**
```bash
# Ensure PRIVATE_KEY is set with 0x prefix
export PRIVATE_KEY=0x...your_64_character_hex_key...

# Or in .env file
PRIVATE_KEY=0x...your_64_character_hex_key...
```

### "Invalid private key format"

**Cause:** Private key missing `0x` prefix or incorrect length.

**Solution:**
```typescript
// Validate private key format
const PRIVATE_KEY = process.env.PRIVATE_KEY;
if (!PRIVATE_KEY || !isHex(PRIVATE_KEY)) {
  throw new Error('PRIVATE_KEY must be a hex string starting with 0x');
}
```

### TypeScript Import Errors

**Cause:** Incorrect import paths or missing viem peer dependency.

**Solution:**
```bash
# Ensure both packages installed
npm install clanker-sdk viem
```

```typescript
// Correct imports
import { Clanker } from 'clanker-sdk';
import { Clanker } from 'clanker-sdk/v4'; // V4 specific
import { createAirdrop } from 'clanker-sdk/v4/extensions';
```

## Deployment Issues

### "Insufficient balance"

**Cause:** Wallet doesn't have enough native token for gas.

**Solution:**
```typescript
// Check balance before deployment
const balance = await publicClient.getBalance({ address: account.address });
console.log('Balance:', formatEther(balance), 'ETH');

// Fund wallet if needed
// Minimum ~0.01 ETH recommended for Base, more for Ethereum mainnet
```

### "Transaction reverted"

**Cause:** Invalid configuration or contract state issue.

**Solution:**
1. Simulate first:
```typescript
try {
  await clanker.deploySimulate(config);
  console.log('Simulation passed');
} catch (error) {
  console.error('Would revert:', error);
}
```

2. Check configuration values:
```typescript
// Ensure required fields
if (!config.name || !config.symbol) {
  throw new Error('Name and symbol required');
}

// Validate vault percentage
if (config.vault?.percentage > 30) {
  throw new Error('Vault percentage max is 30%');
}

// Validate rewards total
const totalBps = config.rewards?.recipients?.reduce(
  (sum, r) => sum + r.bps, 0
) || 0;
if (totalBps !== 10000) {
  throw new Error('Rewards must total 10000 bps');
}
```

### "Invalid image"

**Cause:** IPFS hash not accessible or invalid format.

**Solution:**
```typescript
// Ensure valid IPFS URI format
const image = 'ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi';

// Test accessibility
// Visit: https://ipfs.io/ipfs/bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi
```

### "Transaction pending too long"

**Cause:** Gas price too low or network congestion.

**Solution:**
```typescript
// Wait with timeout
const { address, error } = await Promise.race([
  waitForTransaction(),
  new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Timeout')), 120000)
  ),
]);

// Check transaction status manually
const receipt = await publicClient.getTransactionReceipt({ hash: txHash });
```

## Chain-Specific Issues

### Monad: "Dynamic fees not supported"

**Cause:** Monad only supports static fee configurations.

**Solution:**
```typescript
import { FEE_CONFIGS } from 'clanker-sdk/constants';

// Use static fees for Monad
fees: FEE_CONFIGS.StaticBasic, // NOT Dynamic3
```

### Wrong Chain

**Cause:** Wallet and publicClient configured for different chains.

**Solution:**
```typescript
// Ensure consistent chain configuration
const CHAIN = base;

const publicClient = createPublicClient({
  chain: CHAIN, // Same chain
  transport: http(),
});

const wallet = createWalletClient({
  account,
  chain: CHAIN, // Same chain
  transport: http(),
});

const { txHash } = await clanker.deploy({
  chainId: CHAIN.id, // Same chain ID
  // ...
});
```

## Post-Deployment Issues

### "No vault tokens to claim"

**Cause:** Lockup period hasn't passed yet.

**Solution:**
```typescript
// Check current claimable amount
const claimable = await clanker.getVaultClaimableAmount({ token: TOKEN_ADDRESS });
console.log('Claimable:', claimable.toString());

// If 0, lockup hasn't passed
// Check deployment block time + lockupDuration
```

### "Cannot claim rewards"

**Cause:** Not the reward recipient or no rewards accumulated.

**Solution:**
```typescript
// Check available rewards first
const available = await clanker.availableRewards({
  token: TOKEN_ADDRESS,
  rewardRecipient: YOUR_ADDRESS,
});

console.log('Available:', available);

// Ensure you're the correct recipient
```

### "Cannot update metadata"

**Cause:** Not the token admin.

**Solution:**
```typescript
// Only tokenAdmin can update metadata
// Verify you're using the same account that deployed
console.log('Your address:', account.address);
// Compare with tokenAdmin set during deployment
```

## Airdrop Issues

### "Airdrop not registered"

**Cause:** Didn't wait for indexing before registering.

**Solution:**
```typescript
// Wait at least 10 seconds after deployment
await sleep(10_000);
await registerAirdrop(tokenAddress, tree);
```

### "Invalid proof"

**Cause:** Merkle tree mismatch or wrong address.

**Solution:**
```typescript
// Regenerate the tree with same data
const { tree, airdrop } = createAirdrop(originalRecipients);

// Ensure exact address match (case-sensitive)
const proof = getAllowlistMerkleProof(tree, entries, address.toLowerCase(), amount);
```

## RPC Issues

### "Rate limited"

**Cause:** Public RPC rate limits exceeded.

**Solution:**
```typescript
// Use dedicated RPC URL
const publicClient = createPublicClient({
  chain: base,
  transport: http(process.env.RPC_URL_BASE), // Alchemy, Infura, etc.
});
```

### "Request failed"

**Cause:** Network issues or RPC unavailable.

**Solution:**
```typescript
// Add retry logic
async function deployWithRetry(config, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await clanker.deploy(config);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await sleep(1000 * (i + 1)); // Exponential backoff
    }
  }
}
```

## Contract Error Reference

Common revert reasons from the Solidity contracts:

| Error | Cause | Solution |
|-------|-------|----------|
| `VaultLockupDurationTooShort` | Lockup < 7 days | Set lockupDuration ≥ 604800 |
| `AirdropLockupDurationTooShort` | Lockup < 1 day | Set lockupDuration ≥ 86400 |
| `MaxExtensionBpsExceeded` | Extensions > 90% | Reduce total extension percentage |
| `MaxExtensionsExceeded` | More than 10 extensions | Use fewer extensions |
| `TooManyRewardParticipants` | More than 7 recipients | Reduce reward recipients |
| `TooManyPositions` | More than 7 LP positions | Use fewer positions |
| `InvalidRewardBps` | Rewards ≠ 10000 bps | Ensure rewards total exactly 10000 |
| `StartingFeeGreaterThanMaxLpFee` | Sniper fee > 80% | Set startingFee ≤ 800000 |
| `TimeDecayLongerThanMaxMevDelay` | Decay > 2 min | Set secondsToDecay ≤ 120 |
| `StartingFeeMustBeGreaterThanEndingFee` | startingFee ≤ endingFee | Increase starting or decrease ending fee |

## Debug Checklist

1. ✅ Private key set and valid (starts with 0x)
2. ✅ Wallet funded with native token
3. ✅ Chain configuration consistent
4. ✅ IPFS image accessible
5. ✅ Required fields provided (name, symbol, image, tokenAdmin)
6. ✅ Total extension BPS ≤ 9000 (90%)
7. ✅ Vault lockup ≥ 7 days (604800 seconds)
8. ✅ Airdrop lockup ≥ 1 day (86400 seconds)
9. ✅ Rewards total = 10000 bps exactly
10. ✅ Reward recipients ≤ 7
11. ✅ LP positions ≤ 7
12. ✅ Sniper startingFee ≤ 800000 (80%)
13. ✅ Sniper secondsToDecay ≤ 120 (2 minutes)
14. ✅ Static fees for Monad chain
15. ✅ Waited for indexing before airdrop registration
16. ✅ Using correct recipient address for claims

## Getting Help

1. **Check SDK examples**: [github.com/clanker-devco/clanker-sdk/tree/main/examples](https://github.com/clanker-devco/clanker-sdk/tree/main/examples)
2. **Review error messages**: Usually contain specific guidance
3. **Verify on block explorer**: Check transaction status and logs
4. **Test on testnet**: Validate configuration before mainnet



## 📖 Reference: vesting.md

# Token Vesting (Vault)

Configure token vesting with lockup periods and linear vesting using the Clanker vault system.

## Overview

The vault system allows you to lock a percentage of the token supply with:
- **Lockup Duration**: Cliff period before any tokens can be claimed
- **Vesting Duration**: Linear vesting period after lockup
- **Recipient**: Address that receives vested tokens

## Basic Vault Configuration

```typescript
const { txHash, waitForTransaction, error } = await clanker.deploy({
  name: 'My Token',
  symbol: 'TKN',
  image: 'ipfs://...',
  tokenAdmin: account.address,
  
  vault: {
    percentage: 10,           // 10% of token supply
    lockupDuration: 2592000,  // 30 days cliff (in seconds)
    vestingDuration: 2592000, // 30 days linear vesting
    recipient: account.address,
  },
  
  // ... other config
});
```

## Duration Values

Common duration values in seconds:

| Duration | Seconds |
|----------|---------|
| 1 hour | 3600 |
| 1 day | 86400 |
| 7 days | 604800 |
| 14 days | 1209600 |
| 30 days | 2592000 |
| 60 days | 5184000 |
| 90 days | 7776000 |
| 180 days | 15552000 |
| 1 year | 31536000 |

## Contract Limits

From the Solidity contracts:

- **Minimum Lockup Duration**: 7 days (enforced on-chain)
- **Maximum Extension BPS**: 9000 (90% of supply can go to extensions total)
- **Minimum to LP**: 10% of supply must go to liquidity pool

```typescript
vault: {
  percentage: 10,           // Up to 90% combined with other extensions
  lockupDuration: 604800,   // Minimum 7 days (604800 seconds)
  vestingDuration: 2592000, // No minimum for vesting duration
}
```

**Note**: The vault is one of several possible extensions. The total of all extension percentages (vault + airdrop + etc.) cannot exceed 90%.

## Check Claimable Amount

After deployment, check how many tokens are available to claim:

```typescript
const TOKEN_ADDRESS = '0x...'; // Your deployed token

const claimable = await clanker.getVaultClaimableAmount({
  token: TOKEN_ADDRESS,
});

console.log('Claimable amount:', claimable.toString());
```

## Claim Vaulted Tokens

Claim tokens once the lockup period has passed:

```typescript
const TOKEN_ADDRESS = '0x...';

// Check if anything is claimable
const claimable = await clanker.getVaultClaimableAmount({
  token: TOKEN_ADDRESS,
});

if (claimable > 0n) {
  const { txHash, error } = await clanker.claimVaultedTokens({
    token: TOKEN_ADDRESS,
  });
  
  if (error) {
    console.error('Claim failed:', error.message);
  } else {
    console.log('Claim transaction:', txHash);
  }
} else {
  console.log('No tokens available to claim yet');
}
```

## Get Vault Claim Transaction

Get the transaction object for claiming (useful for batching or custom signing):

```typescript
const txObject = await clanker.getVaultClaimTransaction({
  token: TOKEN_ADDRESS,
});

console.log('Transaction object:', txObject);
// { to: '0x...', data: '0x...', value: 0n }
```

## Vesting Timeline Example

For a 10% vault with 30-day lockup and 30-day vesting:

```
Day 0:  Token deployed, 10% locked in vault
        ↓
Day 1-30: Lockup period (nothing claimable)
        ↓
Day 31: Vesting begins
        ~3.33% claimable (1/30 of vaulted amount)
        ↓
Day 45: ~50% of vaulted tokens claimable
        ↓
Day 60: 100% claimable
```

## Custom Recipient

Set a different address to receive vested tokens:

```typescript
vault: {
  percentage: 10,
  lockupDuration: 2592000,
  vestingDuration: 2592000,
  recipient: '0x...treasury_address...', // Different from tokenAdmin
}
```

If not specified, `recipient` defaults to `tokenAdmin`.

## Team Vesting Example

Configure vesting for a team allocation:

```typescript
const THIRTY_DAYS = 2592000;
const SIX_MONTHS = 15552000;

const { txHash, waitForTransaction, error } = await clanker.deploy({
  name: 'Team Token',
  symbol: 'TEAM',
  image: 'ipfs://...',
  tokenAdmin: account.address,
  
  vault: {
    percentage: 20,                  // 20% team allocation
    lockupDuration: SIX_MONTHS,      // 6-month cliff
    vestingDuration: SIX_MONTHS * 2, // 12-month vesting
    recipient: '0x...team_multisig...',
  },
  
  metadata: {
    description: 'Token with team vesting schedule',
  },
  
  context: {
    interface: 'Clanker SDK',
  },
});
```

## No Vesting

To deploy without any vesting, simply omit the `vault` configuration:

```typescript
const { txHash, waitForTransaction, error } = await clanker.deploy({
  name: 'My Token',
  symbol: 'TKN',
  image: 'ipfs://...',
  tokenAdmin: account.address,
  // No vault = no vesting
});
```

## Best Practices

1. **Reasonable lockup periods** - Align with project milestones
2. **Gradual vesting** - Avoid cliff-only (0 vesting duration)
3. **Transparent communication** - Share vesting schedule with community
4. **Secure recipient** - Use multisig for team vesting
5. **Test first** - Verify vesting math before mainnet

