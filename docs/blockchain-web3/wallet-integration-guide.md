# Wallet Integration Guide

## Comprehensive Wallet Solutions for Prediction Markets

This guide covers the full spectrum of wallet technologies for integrating with a Polymarket-like prediction market platform, from traditional EOA wallets to cutting-edge account abstraction solutions.

---

## Table of Contents

1. [Wallet Landscape Overview](#wallet-landscape-overview)
2. [EOA Wallets](#eoa-wallets)
3. [Smart Contract Wallets](#smart-contract-wallets)
4. [MPC Wallets](#mpc-wallets)
5. [Account Abstraction Wallets](#account-abstraction-wallets)
6. [Social Login Wallets](#social-login-wallets)
7. [Email Wallets](#email-wallets)
8. [Embedded Wallets](#embedded-wallets)
9. [WalletConnect Integration](#walletconnect-integration)
10. [Multi-Signature Setups](#multi-signature-setups)
11. [Recovery Mechanisms](#recovery-mechanisms)
12. [Security Considerations](#security-considerations)
13. [Implementation Code Examples](#implementation-code-examples)

---

## Wallet Landscape Overview

### Wallet Technology Evolution

```
WALLET TECHNOLOGY EVOLUTION
═══════════════════════════════════════════════════════════════════

2015-2018: EOA ERA
┌─────────────────────────────────────────────────────────────────┐
│  • Single private key controls account                          │
│  • Seed phrase backup                                           │
│  • Examples: MetaMask, MyEtherWallet                            │
│  • Problem: Lose key = lose everything                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
2019-2021: SMART WALLET ERA
┌─────────────────────────────────────────────────────────────────┐
│  • Smart contract controls account                              │
│  • Multi-sig support                                            │
│  • Social recovery                                              │
│  • Examples: Gnosis Safe, Argent                                │
│  • Problem: Still need EOA for gas                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
2022-2023: MPC & SOCIAL LOGIN ERA
┌─────────────────────────────────────────────────────────────────┐
│  • Key sharding across parties                                  │
│  • Web2-like authentication                                     │
│  • No seed phrase needed                                        │
│  • Examples: Privy, Web3Auth, Magic                             │
│  • Problem: Trust in third parties                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
2024-2025: ACCOUNT ABSTRACTION ERA
┌─────────────────────────────────────────────────────────────────┐
│  • ERC-4337 standard                                            │
│  • Gas paid by anyone (paymasters)                              │
│  • Programmable validation                                      │
│  • Session keys, batching                                       │
│  • Examples: Safe, ZeroDev, Biconomy                            │
│  • Current: Best UX with strong security                        │
└─────────────────────────────────────────────────────────────────┘
```

### Wallet Type Comparison Matrix

```
WALLET COMPARISON MATRIX
═══════════════════════════════════════════════════════════════════

                    │Security│ UX  │Gasless│Recovery│ Cost │Enterprise
────────────────────┼────────┼─────┼───────┼────────┼──────┼──────────
EOA (MetaMask)      │  ★★★  │ ★★  │  ❌   │   ❌   │ Free │    ❌
────────────────────┼────────┼─────┼───────┼────────┼──────┼──────────
Smart Wallet (Safe) │ ★★★★★ │ ★★★ │  ✅   │   ✅   │ Med  │    ✅
────────────────────┼────────┼─────┼───────┼────────┼──────┼──────────
MPC (Fireblocks)    │ ★★★★★ │ ★★★ │  ❌   │   ✅   │ High │   ✅✅
────────────────────┼────────┼─────┼───────┼────────┼──────┼──────────
AA Wallet (ZeroDev) │ ★★★★  │★★★★ │  ✅   │   ✅   │ Med  │    ✅
────────────────────┼────────┼─────┼───────┼────────┼──────┼──────────
Social Login(Privy) │  ★★★  │★★★★★│  ✅   │   ✅   │ Med  │    ✅
────────────────────┼────────┼─────┼───────┼────────┼──────┼──────────
Email (Magic)       │  ★★★  │★★★★★│  ✅   │   ✅   │ Low  │    ❌
────────────────────┼────────┼─────┼───────┼────────┼──────┼──────────
Embedded (Privy)    │  ★★★  │★★★★★│  ✅   │   ✅   │ Med  │    ✅

Rating: ★ = Basic, ★★★★★ = Excellent
═══════════════════════════════════════════════════════════════════
```

### Recommended Wallet Strategy

```
WALLET INTEGRATION STRATEGY FOR PREDICTION MARKETS
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                    USER SEGMENTS                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CRYPTO NATIVES (30%)          MAINSTREAM USERS (60%)           │
│  ────────────────────          ────────────────────             │
│  • Have existing wallets       • No crypto experience           │
│  • Prefer MetaMask/Rabby       • Want email/social login        │
│  • Value self-custody          • Don't want seed phrases        │
│  • Understand gas fees         • Expect gasless experience      │
│                                                                  │
│  → Support EOA wallets         → Embedded + Social wallets      │
│  → WalletConnect               → Account abstraction            │
│                                                                  │
│                  POWER USERS (10%)                              │
│                  ────────────────────                           │
│                  • High-value accounts                          │
│                  • Need multi-sig                               │
│                  • Institutional requirements                   │
│                                                                  │
│                  → Safe multi-sig                               │
│                  → MPC solutions                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

RECOMMENDED STACK:
┌─────────────────────────────────────────────────────────────────┐
│  PRIMARY: Privy                                                  │
│  • Handles all user types in one SDK                            │
│  • Social login + wallet connect + embedded                     │
│  • Account abstraction built-in                                 │
│                                                                  │
│  SECONDARY: WalletConnect + RainbowKit                          │
│  • For advanced users who prefer their own wallets              │
│  • Supports 300+ wallets                                        │
│                                                                  │
│  ENTERPRISE: Safe (Multi-sig)                                   │
│  • For treasury and high-value operations                       │
│  • Required for institutional users                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## EOA Wallets

### Overview

Externally Owned Accounts (EOAs) are the traditional Ethereum wallet type, controlled by a single private key.

```
EOA WALLET ARCHITECTURE
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                     PRIVATE KEY                                  │
│            (256-bit random number)                               │
│                         │                                        │
│                         ▼                                        │
│                ┌────────────────┐                               │
│                │  ECDSA Sign    │                               │
│                └────────┬───────┘                               │
│                         │                                        │
│                         ▼                                        │
│                ┌────────────────┐                               │
│                │  PUBLIC KEY    │                               │
│                │  (Compressed)  │                               │
│                └────────┬───────┘                               │
│                         │                                        │
│                         ▼                                        │
│                ┌────────────────┐                               │
│                │    ADDRESS     │                               │
│                │ 0x742d35Cc...  │                               │
│                └────────────────┘                               │
│                                                                  │
│  Pros: Simple, Universal, Self-custody                          │
│  Cons: Single point of failure, No recovery, Gas required       │
└─────────────────────────────────────────────────────────────────┘
```

### Popular EOA Wallets

| Wallet | Type | Chains | Features | Best For |
|--------|------|--------|----------|----------|
| **MetaMask** | Browser/Mobile | EVM | Most popular, Snaps | General use |
| **Coinbase Wallet** | Browser/Mobile | EVM + Solana | Coinbase integration | Beginners |
| **Rainbow** | Mobile | EVM | Beautiful UX | Mobile users |
| **Rabby** | Browser | EVM | Multi-chain view | Power users |
| **Trust Wallet** | Mobile | 70+ chains | Wide support | Multi-chain |
| **Frame** | Desktop | EVM | Hardware wallet focus | Security |

### MetaMask Integration

```typescript
// Basic MetaMask detection and connection
async function connectMetaMask(): Promise<string | null> {
  // Check if MetaMask is installed
  if (typeof window.ethereum === "undefined") {
    console.log("MetaMask not installed");
    window.open("https://metamask.io/download/", "_blank");
    return null;
  }

  try {
    // Request account access
    const accounts = await window.ethereum.request({
      method: "eth_requestAccounts",
    });

    // Get the connected account
    const account = accounts[0];
    console.log("Connected:", account);

    // Listen for account changes
    window.ethereum.on("accountsChanged", (accounts: string[]) => {
      if (accounts.length === 0) {
        console.log("Disconnected");
      } else {
        console.log("Account changed:", accounts[0]);
      }
    });

    // Listen for chain changes
    window.ethereum.on("chainChanged", (chainId: string) => {
      console.log("Chain changed:", parseInt(chainId, 16));
      // Reload the page on chain change (recommended by MetaMask)
      window.location.reload();
    });

    return account;
  } catch (error: any) {
    if (error.code === 4001) {
      console.log("User rejected connection");
    } else {
      console.error("Connection error:", error);
    }
    return null;
  }
}

// Switch to correct network
async function switchToPolygon(): Promise<boolean> {
  const POLYGON_CHAIN_ID = "0x89"; // 137 in hex

  try {
    await window.ethereum.request({
      method: "wallet_switchEthereumChain",
      params: [{ chainId: POLYGON_CHAIN_ID }],
    });
    return true;
  } catch (error: any) {
    // Chain not added, add it
    if (error.code === 4902) {
      try {
        await window.ethereum.request({
          method: "wallet_addEthereumChain",
          params: [
            {
              chainId: POLYGON_CHAIN_ID,
              chainName: "Polygon Mainnet",
              nativeCurrency: {
                name: "POL",
                symbol: "POL",
                decimals: 18,
              },
              rpcUrls: ["https://polygon-rpc.com"],
              blockExplorerUrls: ["https://polygonscan.com"],
            },
          ],
        });
        return true;
      } catch (addError) {
        console.error("Failed to add network:", addError);
        return false;
      }
    }
    console.error("Failed to switch network:", error);
    return false;
  }
}
```

---

## Smart Contract Wallets

### Architecture

```
SMART CONTRACT WALLET ARCHITECTURE
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                    SMART CONTRACT WALLET                         │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    PROXY CONTRACT                          │  │
│  │                  (User's Address)                          │  │
│  │                                                            │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │  │
│  │  │   Assets    │  │   Logic     │  │   State     │       │  │
│  │  │   (ETH,     │  │  (Delegated │  │  (Owners,   │       │  │
│  │  │   Tokens)   │  │   to impl)  │  │   Config)   │       │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              IMPLEMENTATION CONTRACT                       │  │
│  │                                                            │  │
│  │  • execute() - Execute transactions                       │  │
│  │  • validateSignature() - Verify owner                     │  │
│  │  • addOwner() / removeOwner()                             │  │
│  │  • changeThreshold()                                      │  │
│  │  • enableModule() - Add plugins                           │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Benefits:                                                       │
│  • Multi-signature support                                      │
│  • Social recovery                                              │
│  • Spending limits                                              │
│  • Session keys                                                 │
│  • Batch transactions                                           │
│  • Upgradeable logic                                            │
└─────────────────────────────────────────────────────────────────┘
```

### Safe (formerly Gnosis Safe)

Safe is the most battle-tested smart contract wallet, securing over $100B in assets.

```typescript
import Safe from "@safe-global/protocol-kit";
import { EthersAdapter } from "@safe-global/protocol-kit";
import { ethers } from "ethers";

// Initialize Safe
async function initializeSafe(
  safeAddress: string,
  signer: ethers.Signer
): Promise<Safe> {
  const ethAdapter = new EthersAdapter({
    ethers,
    signerOrProvider: signer,
  });

  const safe = await Safe.create({
    ethAdapter,
    safeAddress,
  });

  return safe;
}

// Create a new Safe
async function createNewSafe(
  owners: string[],
  threshold: number,
  signer: ethers.Signer
): Promise<string> {
  const ethAdapter = new EthersAdapter({
    ethers,
    signerOrProvider: signer,
  });

  const safeFactory = await SafeFactory.create({ ethAdapter });

  const safeAccountConfig = {
    owners,
    threshold,
  };

  const safe = await safeFactory.deploySafe({ safeAccountConfig });
  const safeAddress = await safe.getAddress();

  console.log("Safe deployed at:", safeAddress);
  return safeAddress;
}

// Create and execute a transaction
async function executeSafeTransaction(
  safe: Safe,
  to: string,
  value: string,
  data: string
): Promise<string> {
  // Create transaction
  const safeTransaction = await safe.createTransaction({
    safeTransactionData: {
      to,
      value,
      data,
    },
  });

  // Sign transaction (for single-owner or collecting signatures)
  const signedTransaction = await safe.signTransaction(safeTransaction);

  // Execute transaction (if threshold met)
  const executeTxResponse = await safe.executeTransaction(signedTransaction);
  const receipt = await executeTxResponse.transactionResponse?.wait();

  console.log("Transaction executed:", receipt?.hash);
  return receipt?.hash || "";
}
```

---

## MPC Wallets

### Multi-Party Computation Overview

```
MPC WALLET ARCHITECTURE
═══════════════════════════════════════════════════════════════════

TRADITIONAL KEY:                    MPC KEY:
┌───────────────┐                  ┌───────────────────────────────┐
│               │                  │                               │
│  PRIVATE KEY  │                  │   KEY SHARE 1   KEY SHARE 2  │
│    (Full)     │                  │   (Party A)     (Party B)     │
│               │                  │       │             │         │
│   ████████    │                  │      ███          ███        │
│               │                  │       │             │         │
└───────────────┘                  │       └──────┬──────┘         │
       │                           │              │                │
       │ Signs                     │     MPC Protocol              │
       ▼                           │     (Threshold)               │
   Signature                       │              │                │
                                   │              ▼                │
                                   │         Signature             │
                                   │   (No full key ever exists)  │
                                   └───────────────────────────────┘

KEY FEATURES:
• No single party ever has full key
• Threshold signatures (e.g., 2-of-3)
• Key refresh without address change
• Distributed trust
```

### MPC Providers Comparison

| Provider | Type | Target | Key Features | Pricing |
|----------|------|--------|--------------|---------|
| **Fireblocks** | Enterprise | Institutions | Full custody suite, $10T+ secured | Enterprise |
| **Fordefi** | Enterprise | Funds/DAOs | Policy engine, DeFi native | Enterprise |
| **ZenGo** | Consumer | Retail | 3FA, no seed phrase | Free tier |
| **Coinbase WaaS** | B2B | Startups | Coinbase infrastructure | Usage-based |
| **Turnkey** | Developer | Apps | API-first, low latency | Usage-based |

### Fireblocks Integration

```typescript
import { FireblocksSDK } from "fireblocks-sdk";

// Initialize Fireblocks client
const fireblocks = new FireblocksSDK(
  process.env.FIREBLOCKS_API_SECRET!,
  process.env.FIREBLOCKS_API_KEY!,
  process.env.FIREBLOCKS_API_URL
);

// Create a vault account for a user
async function createUserVault(userId: string): Promise<string> {
  const vault = await fireblocks.createVaultAccount({
    name: `User-${userId}`,
    hiddenOnUI: false,
    autoFuel: true, // Automatically fund gas
  });

  // Create an Ethereum wallet in the vault
  const wallet = await fireblocks.createVaultAsset(vault.id, "ETH_POLYGON");

  console.log(`Vault ${vault.id} created with address: ${wallet.address}`);
  return wallet.address;
}

// Execute a transaction
async function executeTransaction(
  sourceVaultId: string,
  destinationAddress: string,
  amount: string,
  assetId: string = "USDC_POLYGON"
): Promise<string> {
  const txResult = await fireblocks.createTransaction({
    assetId,
    source: {
      type: "VAULT_ACCOUNT" as any,
      id: sourceVaultId,
    },
    destination: {
      type: "ONE_TIME_ADDRESS" as any,
      oneTimeAddress: {
        address: destinationAddress,
      },
    },
    amount,
    note: "Prediction Market Transaction",
  });

  console.log("Transaction ID:", txResult.id);

  // Wait for transaction to complete
  let status = txResult.status;
  while (status !== "COMPLETED" && status !== "FAILED") {
    await new Promise((resolve) => setTimeout(resolve, 2000));
    const tx = await fireblocks.getTransactionById(txResult.id);
    status = tx.status;
  }

  if (status === "FAILED") {
    throw new Error("Transaction failed");
  }

  return txResult.txHash || txResult.id;
}

// Sign a message
async function signMessage(
  vaultId: string,
  message: string
): Promise<string> {
  const result = await fireblocks.createTransaction({
    operation: "RAW",
    assetId: "ETH_POLYGON",
    source: {
      type: "VAULT_ACCOUNT" as any,
      id: vaultId,
    },
    extraParameters: {
      rawMessageData: {
        messages: [
          {
            content: Buffer.from(message).toString("hex"),
          },
        ],
      },
    },
  });

  // Wait for signing to complete
  let tx = await fireblocks.getTransactionById(result.id);
  while (tx.status !== "COMPLETED") {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    tx = await fireblocks.getTransactionById(result.id);
  }

  return tx.signedMessages![0].signature.fullSig;
}
```

---

## Account Abstraction Wallets

### ERC-4337 Overview

```
ERC-4337 ACCOUNT ABSTRACTION FLOW
═══════════════════════════════════════════════════════════════════

┌──────────────────┐     UserOperation      ┌──────────────────┐
│                  │ ─────────────────────▶ │                  │
│   User/dApp      │                        │   Bundler        │
│                  │                        │   (Collects &    │
│  Signs UserOp    │                        │   Bundles UserOps│
│  (off-chain)     │                        │                  │
└──────────────────┘                        └────────┬─────────┘
                                                     │
                                            Bundle of UserOps
                                                     │
                                                     ▼
┌──────────────────────────────────────────────────────────────┐
│                        ENTRYPOINT                             │
│                 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789   │
│                                                               │
│  handleOps(UserOperation[] ops, address payable beneficiary) │
│                                                               │
│  For each UserOp:                                            │
│  1. validateUserOp() on Smart Account                        │
│  2. If Paymaster: validatePaymasterUserOp()                  │
│  3. Execute the operation                                    │
│  4. postOp() if Paymaster                                    │
│  5. Pay bundler                                              │
└──────────────────────────────────────────────────────────────┘
         │                              │
         ▼                              ▼
┌──────────────────┐         ┌──────────────────┐
│  SMART ACCOUNT   │         │   PAYMASTER      │
│                  │         │   (Optional)     │
│  • Validates sig │         │                  │
│  • Executes call │         │  • Sponsors gas  │
│  • Custom logic  │         │  • ERC-20 gas    │
└──────────────────┘         └──────────────────┘
```

### ZeroDev Integration

ZeroDev provides a Kernel smart account with modular plugins.

```typescript
import { createKernelAccount, createKernelAccountClient } from "@zerodev/sdk";
import { signerToEcdsaValidator } from "@zerodev/ecdsa-validator";
import { ENTRYPOINT_ADDRESS_V07, bundlerActions } from "permissionless";
import { createPublicClient, http } from "viem";
import { polygon } from "viem/chains";

// Configuration
const BUNDLER_URL = process.env.ZERODEV_BUNDLER_URL!;
const PAYMASTER_URL = process.env.ZERODEV_PAYMASTER_URL!;
const PROJECT_ID = process.env.ZERODEV_PROJECT_ID!;

// Create a smart account from an EOA signer
async function createSmartAccount(privateKey: `0x${string}`) {
  const publicClient = createPublicClient({
    chain: polygon,
    transport: http(),
  });

  // Create signer from private key
  const signer = privateKeyToAccount(privateKey);

  // Create ECDSA validator (uses EOA as signer)
  const ecdsaValidator = await signerToEcdsaValidator(publicClient, {
    signer,
    entryPoint: ENTRYPOINT_ADDRESS_V07,
  });

  // Create Kernel account
  const account = await createKernelAccount(publicClient, {
    plugins: {
      sudo: ecdsaValidator,
    },
    entryPoint: ENTRYPOINT_ADDRESS_V07,
  });

  console.log("Smart Account Address:", account.address);

  // Create account client for sending transactions
  const kernelClient = createKernelAccountClient({
    account,
    chain: polygon,
    entryPoint: ENTRYPOINT_ADDRESS_V07,
    bundlerTransport: http(BUNDLER_URL),
    middleware: {
      sponsorUserOperation: async ({ userOperation }) => {
        // Get sponsorship from paymaster
        const response = await fetch(PAYMASTER_URL, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            jsonrpc: "2.0",
            method: "pm_sponsorUserOperation",
            params: [userOperation, ENTRYPOINT_ADDRESS_V07],
            id: 1,
          }),
        });
        const data = await response.json();
        return data.result;
      },
    },
  });

  return { account, client: kernelClient };
}

// Send a gasless transaction
async function sendGaslessTransaction(
  client: any,
  to: `0x${string}`,
  data: `0x${string}`,
  value: bigint = 0n
) {
  const txHash = await client.sendTransaction({
    to,
    data,
    value,
  });

  console.log("Transaction sent:", txHash);

  // Wait for receipt
  const receipt = await client.waitForTransactionReceipt({ hash: txHash });
  console.log("Transaction confirmed:", receipt.status);

  return txHash;
}

// Batch multiple operations in one transaction
async function batchOperations(
  client: any,
  operations: Array<{
    to: `0x${string}`;
    data: `0x${string}`;
    value?: bigint;
  }>
) {
  const txHash = await client.sendTransactions({
    transactions: operations.map((op) => ({
      to: op.to,
      data: op.data,
      value: op.value || 0n,
    })),
  });

  return txHash;
}

// Create a session key for limited permissions
async function createSessionKey(
  account: any,
  permissions: {
    target: `0x${string}`;
    selector: `0x${string}`;
    validUntil: number;
  }[]
) {
  // Generate a new session key
  const sessionPrivateKey = generatePrivateKey();
  const sessionSigner = privateKeyToAccount(sessionPrivateKey);

  // Create session key validator with permissions
  const sessionKeyValidator = await createSessionKeyValidator({
    account,
    signer: sessionSigner,
    permissions: permissions.map((p) => ({
      target: p.target,
      selector: p.selector,
      rules: [],
      validUntil: p.validUntil,
      validAfter: Math.floor(Date.now() / 1000),
    })),
  });

  return {
    privateKey: sessionPrivateKey,
    validator: sessionKeyValidator,
  };
}
```

### Biconomy Smart Accounts

```typescript
import {
  createSmartAccountClient,
  BiconomySmartAccountV2,
  PaymasterMode,
} from "@biconomy/account";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { polygon } from "viem/chains";

const BUNDLER_URL = `https://bundler.biconomy.io/api/v2/137/${process.env.BICONOMY_BUNDLER_KEY}`;
const PAYMASTER_URL = `https://paymaster.biconomy.io/api/v1/137/${process.env.BICONOMY_PAYMASTER_KEY}`;

// Create Biconomy Smart Account
async function createBiconomyAccount(privateKey: `0x${string}`) {
  const account = privateKeyToAccount(privateKey);

  const walletClient = createWalletClient({
    account,
    chain: polygon,
    transport: http(),
  });

  const smartAccount = await createSmartAccountClient({
    signer: walletClient,
    bundlerUrl: BUNDLER_URL,
    paymasterUrl: PAYMASTER_URL,
  });

  const address = await smartAccount.getAccountAddress();
  console.log("Smart Account:", address);

  return smartAccount;
}

// Send sponsored transaction
async function sendSponsoredTx(
  smartAccount: BiconomySmartAccountV2,
  to: `0x${string}`,
  data: `0x${string}`
) {
  const userOpResponse = await smartAccount.sendTransaction(
    {
      to,
      data,
    },
    {
      paymasterServiceData: {
        mode: PaymasterMode.SPONSORED,
      },
    }
  );

  const receipt = await userOpResponse.wait();
  return receipt.receipt.transactionHash;
}

// Send transaction with ERC-20 gas payment
async function sendERC20GasTx(
  smartAccount: BiconomySmartAccountV2,
  to: `0x${string}`,
  data: `0x${string}`,
  feeToken: `0x${string}` // e.g., USDC address
) {
  const userOpResponse = await smartAccount.sendTransaction(
    {
      to,
      data,
    },
    {
      paymasterServiceData: {
        mode: PaymasterMode.ERC20,
        preferredToken: feeToken,
      },
    }
  );

  const receipt = await userOpResponse.wait();
  return receipt.receipt.transactionHash;
}
```

---

## Social Login Wallets

### Provider Comparison

```
SOCIAL LOGIN WALLET PROVIDERS
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                          PRIVY                                   │
├─────────────────────────────────────────────────────────────────┤
│  Funding: $33M (Paradigm, Sequoia)                              │
│  Pricing: Free tier, then usage-based                           │
│                                                                  │
│  Supported Login Methods:                                        │
│  ✅ Email    ✅ SMS     ✅ Google    ✅ Apple                   │
│  ✅ Twitter  ✅ Discord  ✅ Farcaster ✅ Telegram               │
│  ✅ Wallet Connect  ✅ Coinbase  ✅ MetaMask                    │
│                                                                  │
│  Features:                                                       │
│  • Embedded wallets (MPC-based)                                 │
│  • Account abstraction support                                  │
│  • Cross-app wallet portability                                 │
│  • User analytics                                               │
│                                                                  │
│  Best For: All-in-one solution, mainstream apps                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                         DYNAMIC                                  │
├─────────────────────────────────────────────────────────────────┤
│  Funding: $20M                                                  │
│  Pricing: Free tier up to 1,000 MAU                             │
│                                                                  │
│  Supported Login Methods:                                        │
│  ✅ Email    ✅ Google    ✅ Apple    ✅ Farcaster              │
│  ✅ Twitter  ✅ Discord   ✅ Telegram ✅ Passkeys               │
│  ✅ Wallet Connect  ✅ 300+ Wallets                             │
│                                                                  │
│  Features:                                                       │
│  • Multi-chain (EVM, Solana, Bitcoin)                          │
│  • Embedded wallets (TSS-MPC coming)                            │
│  • Custom UI components                                         │
│  • On-chain verification                                        │
│                                                                  │
│  Best For: Multi-chain apps, customization needs                │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        WEB3AUTH                                  │
├─────────────────────────────────────────────────────────────────┤
│  Funding: $18M                                                  │
│  Pricing: Free tier 1,000 MAU, then $0.05/MAU                   │
│                                                                  │
│  Supported Login Methods:                                        │
│  ✅ Email  ✅ Google  ✅ Apple  ✅ Twitter  ✅ Discord          │
│  ✅ Facebook  ✅ Twitch  ✅ Reddit  ✅ Line                     │
│  ✅ LinkedIn  ✅ GitHub  ✅ Kakao  ✅ WeChat                    │
│                                                                  │
│  Features:                                                       │
│  • MPC-TSS key management                                       │
│  • White-label customization                                    │
│  • Self-hosted option available                                 │
│  • Blockchain agnostic                                          │
│                                                                  │
│  Best For: Global apps (most OAuth providers), cost-sensitive   │
└─────────────────────────────────────────────────────────────────┘
```

### Privy Integration

```typescript
import { PrivyProvider, usePrivy, useWallets } from "@privy-io/react-auth";
import { useSmartWallets } from "@privy-io/react-auth/smart-wallets";

// 1. Configure Privy Provider
function App() {
  return (
    <PrivyProvider
      appId={process.env.NEXT_PUBLIC_PRIVY_APP_ID!}
      config={{
        // Appearance
        appearance: {
          theme: "dark",
          accentColor: "#6366f1",
          logo: "/logo.png",
        },
        // Login methods
        loginMethods: ["email", "google", "twitter", "wallet"],
        // Embedded wallet config
        embeddedWallets: {
          createOnLogin: "users-without-wallets",
          requireUserPasswordOnCreate: false,
        },
        // Smart wallet config (Account Abstraction)
        smartWallets: {
          createOnLogin: "all-users",
        },
        // Default chain
        defaultChain: polygon,
        supportedChains: [polygon, arbitrum, base],
      }}
    >
      <YourApp />
    </PrivyProvider>
  );
}

// 2. Use Privy hooks in components
function WalletButton() {
  const {
    ready,
    authenticated,
    login,
    logout,
    user,
  } = usePrivy();

  const { wallets } = useWallets();
  const { client: smartWalletClient } = useSmartWallets();

  if (!ready) {
    return <div>Loading...</div>;
  }

  if (!authenticated) {
    return <button onClick={login}>Connect</button>;
  }

  // User is authenticated
  const embeddedWallet = wallets.find((w) => w.walletClientType === "privy");
  const smartWallet = user?.smartWallet;

  return (
    <div>
      <p>Embedded: {embeddedWallet?.address}</p>
      <p>Smart Wallet: {smartWallet?.address}</p>
      <button onClick={logout}>Disconnect</button>
    </div>
  );
}

// 3. Send transactions
function TransactionComponent() {
  const { wallets } = useWallets();
  const { client: smartWalletClient } = useSmartWallets();

  // Send from embedded wallet (EOA)
  async function sendFromEmbedded() {
    const embeddedWallet = wallets.find((w) => w.walletClientType === "privy");
    if (!embeddedWallet) return;

    const provider = await embeddedWallet.getEthereumProvider();

    const txHash = await provider.request({
      method: "eth_sendTransaction",
      params: [
        {
          from: embeddedWallet.address,
          to: "0x...",
          value: "0x0",
          data: "0x...",
        },
      ],
    });

    console.log("TX Hash:", txHash);
  }

  // Send gasless from smart wallet
  async function sendGasless() {
    if (!smartWalletClient) return;

    const txHash = await smartWalletClient.sendTransaction({
      to: "0x...",
      data: "0x...",
    });

    console.log("Gasless TX:", txHash);
  }

  return (
    <div>
      <button onClick={sendFromEmbedded}>Send (EOA)</button>
      <button onClick={sendGasless}>Send Gasless (AA)</button>
    </div>
  );
}
```

### Dynamic Integration

```typescript
import {
  DynamicContextProvider,
  DynamicWidget,
  useDynamicContext,
  useIsLoggedIn,
} from "@dynamic-labs/sdk-react-core";
import { EthereumWalletConnectors } from "@dynamic-labs/ethereum";

// 1. Configure Dynamic Provider
function App() {
  return (
    <DynamicContextProvider
      settings={{
        environmentId: process.env.NEXT_PUBLIC_DYNAMIC_ENV_ID!,
        walletConnectors: [EthereumWalletConnectors],
        // Enable email login
        enableEmailLogin: true,
        // Enable social logins
        socialProvidersFilter: (providers) =>
          providers.filter((p) => ["google", "twitter", "discord"].includes(p.provider)),
        // Embedded wallet
        embeddedWallets: {
          enabled: true,
        },
      }}
    >
      <YourApp />
    </DynamicContextProvider>
  );
}

// 2. Use Dynamic widget
function ConnectButton() {
  return <DynamicWidget />;
}

// 3. Access wallet data
function WalletInfo() {
  const { primaryWallet, user, handleLogOut } = useDynamicContext();
  const isLoggedIn = useIsLoggedIn();

  if (!isLoggedIn) {
    return <DynamicWidget />;
  }

  return (
    <div>
      <p>Email: {user?.email}</p>
      <p>Wallet: {primaryWallet?.address}</p>
      <button onClick={handleLogOut}>Logout</button>
    </div>
  );
}

// 4. Send transaction
async function sendTransaction(primaryWallet: any) {
  const walletClient = await primaryWallet.getWalletClient();

  const hash = await walletClient.sendTransaction({
    to: "0x...",
    value: parseEther("0.01"),
    data: "0x...",
  });

  return hash;
}
```

---

## Email Wallets

### Magic Link Integration

```typescript
import { Magic } from "magic-sdk";
import { ethers } from "ethers";

// Initialize Magic
const magic = new Magic(process.env.NEXT_PUBLIC_MAGIC_API_KEY!, {
  network: {
    rpcUrl: "https://polygon-rpc.com",
    chainId: 137,
  },
});

// Login with email
async function loginWithEmail(email: string): Promise<string | null> {
  try {
    // This sends a magic link to the user's email
    const didToken = await magic.auth.loginWithMagicLink({ email });

    // Get the user's wallet address
    const userInfo = await magic.user.getInfo();
    console.log("Logged in:", userInfo.publicAddress);

    return userInfo.publicAddress;
  } catch (error) {
    console.error("Login failed:", error);
    return null;
  }
}

// Check if user is logged in
async function isLoggedIn(): Promise<boolean> {
  return await magic.user.isLoggedIn();
}

// Logout
async function logout(): Promise<void> {
  await magic.user.logout();
}

// Get ethers provider
function getMagicProvider(): ethers.BrowserProvider {
  return new ethers.BrowserProvider(magic.rpcProvider as any);
}

// Send transaction
async function sendTransaction(
  to: string,
  value: string,
  data: string
): Promise<string> {
  const provider = getMagicProvider();
  const signer = await provider.getSigner();

  const tx = await signer.sendTransaction({
    to,
    value: ethers.parseEther(value),
    data,
  });

  const receipt = await tx.wait();
  return receipt!.hash;
}

// Sign message
async function signMessage(message: string): Promise<string> {
  const provider = getMagicProvider();
  const signer = await provider.getSigner();

  return await signer.signMessage(message);
}
```

---

## Embedded Wallets

### Architecture Overview

```
EMBEDDED WALLET ARCHITECTURE
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                     YOUR APPLICATION                             │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                   EMBEDDED WALLET SDK                      │  │
│  │                                                            │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │  │
│  │  │   Login     │  │   Wallet    │  │Transaction  │       │  │
│  │  │   Module    │  │   Module    │  │   Module    │       │  │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘       │  │
│  │         │                │                │               │  │
│  └─────────┼────────────────┼────────────────┼───────────────┘  │
│            │                │                │                   │
└────────────┼────────────────┼────────────────┼───────────────────┘
             │                │                │
             ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SECURE ENCLAVE (TEE)                          │
│                                                                  │
│  Key operations never leave the secure environment               │
│                                                                  │
│  ┌──────────────────┐    ┌──────────────────┐                   │
│  │   Key Share 1    │    │   Key Share 2    │                   │
│  │   (Device)       │    │   (Provider)     │                   │
│  └──────────────────┘    └──────────────────┘                   │
│            │                      │                              │
│            └──────────┬───────────┘                              │
│                       │                                          │
│                       ▼                                          │
│              ┌────────────────┐                                  │
│              │  Sign/Decrypt  │                                  │
│              │   (In TEE)     │                                  │
│              └────────────────┘                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

Benefits:
• Seamless UX (no popups, no external wallets)
• Self-custodial (user controls keys)
• Recovery possible (social/email recovery)
• Invisible to end users
```

### Full Implementation with Privy Embedded Wallets

```typescript
import {
  PrivyProvider,
  usePrivy,
  useWallets,
  useCreateWallet,
} from "@privy-io/react-auth";
import { useSmartWallets } from "@privy-io/react-auth/smart-wallets";
import { encodeFunctionData, parseEther, parseUnits } from "viem";

// Complete Privy integration for prediction market
function PredictionMarketApp() {
  const { ready, authenticated, login, logout, user } = usePrivy();
  const { wallets } = useWallets();
  const { client: smartWalletClient } = useSmartWallets();
  const { createWallet } = useCreateWallet();

  // Get embedded wallet
  const embeddedWallet = wallets.find(
    (wallet) => wallet.walletClientType === "privy"
  );

  // Get smart wallet address
  const smartWalletAddress = user?.smartWallet?.address;

  // Place a bet (gasless via smart wallet)
  async function placeBet(
    marketId: bigint,
    isYes: boolean,
    amount: bigint
  ): Promise<string> {
    if (!smartWalletClient) {
      throw new Error("Smart wallet not ready");
    }

    // First approve USDC spending (if needed)
    const approveTx = {
      to: USDC_ADDRESS,
      data: encodeFunctionData({
        abi: erc20Abi,
        functionName: "approve",
        args: [PREDICTION_MARKET_ADDRESS, amount],
      }),
    };

    // Then place the bet
    const betTx = {
      to: PREDICTION_MARKET_ADDRESS,
      data: encodeFunctionData({
        abi: predictionMarketAbi,
        functionName: "buyShares",
        args: [marketId, isYes, amount],
      }),
    };

    // Batch both transactions (gasless!)
    const txHash = await smartWalletClient.sendTransaction([approveTx, betTx]);

    console.log("Bet placed:", txHash);
    return txHash;
  }

  // Withdraw winnings
  async function claimWinnings(marketId: bigint): Promise<string> {
    if (!smartWalletClient) {
      throw new Error("Smart wallet not ready");
    }

    const txHash = await smartWalletClient.sendTransaction({
      to: PREDICTION_MARKET_ADDRESS,
      data: encodeFunctionData({
        abi: predictionMarketAbi,
        functionName: "claimWinnings",
        args: [marketId],
      }),
    });

    return txHash;
  }

  // Export wallet (for advanced users)
  async function exportWallet() {
    if (!embeddedWallet) return;

    // This shows a secure modal to export private key
    await embeddedWallet.export();
  }

  // Sign a message (for verification)
  async function signMessage(message: string): Promise<string> {
    if (!embeddedWallet) {
      throw new Error("Wallet not ready");
    }

    const provider = await embeddedWallet.getEthereumProvider();
    const signature = await provider.request({
      method: "personal_sign",
      params: [message, embeddedWallet.address],
    });

    return signature as string;
  }

  // Fund wallet (redirect to on-ramp)
  async function fundWallet() {
    if (!embeddedWallet) return;

    // Use Privy's built-in funding
    await embeddedWallet.fund({
      chain: polygon,
    });
  }

  if (!ready) {
    return <div>Loading...</div>;
  }

  if (!authenticated) {
    return (
      <div>
        <h1>Prediction Market</h1>
        <button onClick={login}>
          Sign in with Email, Google, or Wallet
        </button>
      </div>
    );
  }

  return (
    <div>
      <h1>Welcome!</h1>
      <p>Email: {user?.email?.address}</p>
      <p>Smart Wallet: {smartWalletAddress}</p>
      <p>EOA Wallet: {embeddedWallet?.address}</p>

      {/* Your app UI */}
      <MarketList onPlaceBet={placeBet} />

      <button onClick={fundWallet}>Add Funds</button>
      <button onClick={exportWallet}>Export Wallet</button>
      <button onClick={logout}>Sign Out</button>
    </div>
  );
}
```

---

## WalletConnect Integration

### WalletConnect V2 with wagmi

```typescript
import { WagmiProvider, createConfig, http, useConnect, useAccount, useDisconnect } from "wagmi";
import { polygon, arbitrum, base } from "wagmi/chains";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { walletConnect, injected, coinbaseWallet } from "wagmi/connectors";

// Configure wagmi
const config = createConfig({
  chains: [polygon, arbitrum, base],
  connectors: [
    walletConnect({
      projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
      metadata: {
        name: "Prediction Market",
        description: "Decentralized prediction market platform",
        url: "https://yourapp.com",
        icons: ["https://yourapp.com/icon.png"],
      },
      showQrModal: true,
    }),
    injected(),
    coinbaseWallet({
      appName: "Prediction Market",
    }),
  ],
  transports: {
    [polygon.id]: http(),
    [arbitrum.id]: http(),
    [base.id]: http(),
  },
});

const queryClient = new QueryClient();

// Provider wrapper
function App() {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        <YourApp />
      </QueryClientProvider>
    </WagmiProvider>
  );
}

// Connect button component
function ConnectWallet() {
  const { connectors, connect, isPending } = useConnect();
  const { address, isConnected } = useAccount();
  const { disconnect } = useDisconnect();

  if (isConnected) {
    return (
      <div>
        <p>Connected: {address}</p>
        <button onClick={() => disconnect()}>Disconnect</button>
      </div>
    );
  }

  return (
    <div>
      {connectors.map((connector) => (
        <button
          key={connector.id}
          onClick={() => connect({ connector })}
          disabled={isPending}
        >
          {connector.name}
        </button>
      ))}
    </div>
  );
}
```

### RainbowKit Integration

```typescript
import "@rainbow-me/rainbowkit/styles.css";
import {
  getDefaultConfig,
  RainbowKitProvider,
  ConnectButton,
} from "@rainbow-me/rainbowkit";
import { WagmiProvider } from "wagmi";
import { polygon, arbitrum, base } from "wagmi/chains";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

// Configure with RainbowKit defaults
const config = getDefaultConfig({
  appName: "Prediction Market",
  projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
  chains: [polygon, arbitrum, base],
  ssr: true,
});

const queryClient = new QueryClient();

// Provider wrapper
function App() {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        <RainbowKitProvider
          modalSize="compact"
          theme={darkTheme({
            accentColor: "#6366f1",
            borderRadius: "medium",
          })}
        >
          <YourApp />
        </RainbowKitProvider>
      </QueryClientProvider>
    </WagmiProvider>
  );
}

// Pre-built connect button
function Header() {
  return (
    <header>
      <h1>Prediction Market</h1>
      <ConnectButton />
    </header>
  );
}

// Custom connect button
function CustomConnectButton() {
  return (
    <ConnectButton.Custom>
      {({
        account,
        chain,
        openAccountModal,
        openChainModal,
        openConnectModal,
        mounted,
      }) => {
        const ready = mounted;
        const connected = ready && account && chain;

        return (
          <div>
            {!connected ? (
              <button onClick={openConnectModal}>Connect Wallet</button>
            ) : (
              <div>
                <button onClick={openChainModal}>{chain.name}</button>
                <button onClick={openAccountModal}>
                  {account.displayName}
                </button>
              </div>
            )}
          </div>
        );
      }}
    </ConnectButton.Custom>
  );
}
```

---

## Multi-Signature Setups

### Treasury Multi-Sig with Safe

```typescript
import Safe, { SafeFactory, SafeAccountConfig } from "@safe-global/protocol-kit";
import SafeApiKit from "@safe-global/api-kit";
import { ethers } from "ethers";

// Initialize Safe API Kit
const safeApiKit = new SafeApiKit({
  chainId: 137n, // Polygon
});

// Create a multi-sig treasury
async function createTreasury(
  owners: string[],
  threshold: number,
  deployer: ethers.Signer
): Promise<string> {
  const ethAdapter = new EthersAdapter({
    ethers,
    signerOrProvider: deployer,
  });

  const safeFactory = await SafeFactory.create({ ethAdapter });

  const safeAccountConfig: SafeAccountConfig = {
    owners,
    threshold,
  };

  const safe = await safeFactory.deploySafe({
    safeAccountConfig,
    saltNonce: Date.now().toString(),
  });

  const treasuryAddress = await safe.getAddress();
  console.log("Treasury deployed:", treasuryAddress);

  return treasuryAddress;
}

// Propose a transaction
async function proposeTransaction(
  safeAddress: string,
  to: string,
  value: string,
  data: string,
  proposer: ethers.Signer
): Promise<string> {
  const ethAdapter = new EthersAdapter({
    ethers,
    signerOrProvider: proposer,
  });

  const safe = await Safe.create({
    ethAdapter,
    safeAddress,
  });

  // Create transaction
  const safeTransaction = await safe.createTransaction({
    safeTransactionData: {
      to,
      value,
      data,
    },
  });

  // Sign the transaction
  const signedTx = await safe.signTransaction(safeTransaction);

  // Get transaction hash
  const safeTxHash = await safe.getTransactionHash(safeTransaction);

  // Propose to Safe API
  await safeApiKit.proposeTransaction({
    safeAddress,
    safeTransactionData: safeTransaction.data,
    safeTxHash,
    senderAddress: await proposer.getAddress(),
    senderSignature: signedTx.signatures.get(
      (await proposer.getAddress()).toLowerCase()
    )!.data,
  });

  console.log("Transaction proposed:", safeTxHash);
  return safeTxHash;
}

// Confirm a pending transaction
async function confirmTransaction(
  safeAddress: string,
  safeTxHash: string,
  signer: ethers.Signer
): Promise<void> {
  const ethAdapter = new EthersAdapter({
    ethers,
    signerOrProvider: signer,
  });

  const safe = await Safe.create({
    ethAdapter,
    safeAddress,
  });

  // Get pending transaction
  const pendingTx = await safeApiKit.getTransaction(safeTxHash);

  // Create Safe transaction from pending
  const safeTransaction = await safe.createTransaction({
    safeTransactionData: pendingTx,
  });

  // Sign
  const signedTx = await safe.signTransaction(safeTransaction);
  const signerAddress = await signer.getAddress();

  // Submit confirmation
  await safeApiKit.confirmTransaction(
    safeTxHash,
    signedTx.signatures.get(signerAddress.toLowerCase())!.data
  );

  console.log("Transaction confirmed by:", signerAddress);
}

// Execute a fully signed transaction
async function executeTransaction(
  safeAddress: string,
  safeTxHash: string,
  executor: ethers.Signer
): Promise<string> {
  const ethAdapter = new EthersAdapter({
    ethers,
    signerOrProvider: executor,
  });

  const safe = await Safe.create({
    ethAdapter,
    safeAddress,
  });

  // Get transaction with all signatures
  const pendingTx = await safeApiKit.getTransaction(safeTxHash);

  // Check if threshold is met
  const threshold = await safe.getThreshold();
  if (pendingTx.confirmations!.length < threshold) {
    throw new Error(
      `Not enough confirmations: ${pendingTx.confirmations!.length}/${threshold}`
    );
  }

  // Execute
  const executeTxResponse = await safe.executeTransaction(pendingTx);
  const receipt = await executeTxResponse.transactionResponse?.wait();

  console.log("Transaction executed:", receipt?.hash);
  return receipt?.hash || "";
}

// Get pending transactions
async function getPendingTransactions(
  safeAddress: string
): Promise<any[]> {
  const pendingTxs = await safeApiKit.getPendingTransactions(safeAddress);
  return pendingTxs.results;
}
```

---

## Recovery Mechanisms

### Social Recovery Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract SocialRecoveryModule {
    struct RecoveryConfig {
        address[] guardians;
        uint256 threshold;
        uint256 recoveryDelay;
    }

    struct RecoveryRequest {
        address newOwner;
        uint256 confirmations;
        uint256 executeAfter;
        bool executed;
        mapping(address => bool) isConfirmed;
    }

    mapping(address => RecoveryConfig) public configs;
    mapping(address => RecoveryRequest) public recoveryRequests;

    uint256 public constant MIN_DELAY = 2 days;
    uint256 public constant MAX_GUARDIANS = 10;

    event GuardiansSet(address indexed wallet, address[] guardians, uint256 threshold);
    event RecoveryInitiated(address indexed wallet, address newOwner, uint256 executeAfter);
    event RecoveryConfirmed(address indexed wallet, address guardian);
    event RecoveryExecuted(address indexed wallet, address newOwner);
    event RecoveryCancelled(address indexed wallet);

    /// @notice Set up guardians for social recovery
    function setupGuardians(
        address[] calldata guardians,
        uint256 threshold,
        uint256 delay
    ) external {
        require(guardians.length >= threshold, "Threshold too high");
        require(guardians.length <= MAX_GUARDIANS, "Too many guardians");
        require(threshold > 0, "Threshold must be > 0");
        require(delay >= MIN_DELAY, "Delay too short");

        // Check for duplicates
        for (uint256 i = 0; i < guardians.length; i++) {
            require(guardians[i] != address(0), "Invalid guardian");
            for (uint256 j = i + 1; j < guardians.length; j++) {
                require(guardians[i] != guardians[j], "Duplicate guardian");
            }
        }

        configs[msg.sender] = RecoveryConfig({
            guardians: guardians,
            threshold: threshold,
            recoveryDelay: delay
        });

        emit GuardiansSet(msg.sender, guardians, threshold);
    }

    /// @notice Initiate recovery (called by a guardian)
    function initiateRecovery(address wallet, address newOwner) external {
        RecoveryConfig storage config = configs[wallet];
        require(_isGuardian(config, msg.sender), "Not a guardian");
        require(newOwner != address(0), "Invalid new owner");

        RecoveryRequest storage request = recoveryRequests[wallet];
        require(!request.executed, "Already executed");

        // Reset if this is a new recovery attempt
        if (request.newOwner != newOwner) {
            request.newOwner = newOwner;
            request.confirmations = 0;
            request.executeAfter = block.timestamp + config.recoveryDelay;
            request.executed = false;

            // Reset confirmations
            for (uint256 i = 0; i < config.guardians.length; i++) {
                request.isConfirmed[config.guardians[i]] = false;
            }
        }

        // Confirm from initiating guardian
        if (!request.isConfirmed[msg.sender]) {
            request.isConfirmed[msg.sender] = true;
            request.confirmations++;
        }

        emit RecoveryInitiated(wallet, newOwner, request.executeAfter);
        emit RecoveryConfirmed(wallet, msg.sender);
    }

    /// @notice Confirm a pending recovery
    function confirmRecovery(address wallet) external {
        RecoveryConfig storage config = configs[wallet];
        require(_isGuardian(config, msg.sender), "Not a guardian");

        RecoveryRequest storage request = recoveryRequests[wallet];
        require(request.newOwner != address(0), "No pending recovery");
        require(!request.executed, "Already executed");
        require(!request.isConfirmed[msg.sender], "Already confirmed");

        request.isConfirmed[msg.sender] = true;
        request.confirmations++;

        emit RecoveryConfirmed(wallet, msg.sender);
    }

    /// @notice Execute recovery after delay
    function executeRecovery(address wallet) external {
        RecoveryConfig storage config = configs[wallet];
        RecoveryRequest storage request = recoveryRequests[wallet];

        require(request.newOwner != address(0), "No pending recovery");
        require(!request.executed, "Already executed");
        require(
            request.confirmations >= config.threshold,
            "Not enough confirmations"
        );
        require(
            block.timestamp >= request.executeAfter,
            "Delay not passed"
        );

        request.executed = true;

        // Execute the ownership transfer
        // This would call the wallet's ownership transfer function
        IWallet(wallet).transferOwnership(request.newOwner);

        emit RecoveryExecuted(wallet, request.newOwner);
    }

    /// @notice Cancel recovery (only original owner)
    function cancelRecovery() external {
        RecoveryRequest storage request = recoveryRequests[msg.sender];
        require(request.newOwner != address(0), "No pending recovery");
        require(!request.executed, "Already executed");

        delete recoveryRequests[msg.sender];

        emit RecoveryCancelled(msg.sender);
    }

    function _isGuardian(
        RecoveryConfig storage config,
        address account
    ) internal view returns (bool) {
        for (uint256 i = 0; i < config.guardians.length; i++) {
            if (config.guardians[i] == account) {
                return true;
            }
        }
        return false;
    }

    function getGuardians(address wallet) external view returns (address[] memory) {
        return configs[wallet].guardians;
    }
}

interface IWallet {
    function transferOwnership(address newOwner) external;
}
```

---

## Security Considerations

### Security Checklist

```
WALLET SECURITY CHECKLIST
═══════════════════════════════════════════════════════════════════

PRE-INTEGRATION:
☐ Audit wallet SDK/provider security documentation
☐ Verify provider's security certifications (SOC2, etc.)
☐ Review incident history and response procedures
☐ Understand key management architecture
☐ Evaluate recovery mechanisms

IMPLEMENTATION:
☐ Use HTTPS everywhere (required for WebCrypto)
☐ Implement proper CSP headers
☐ Never log or store private keys/signatures
☐ Validate all transaction parameters
☐ Implement transaction simulation before signing
☐ Set reasonable gas limits
☐ Add spending limits for embedded wallets

USER PROTECTION:
☐ Display clear transaction details before signing
☐ Implement transaction preview with human-readable data
☐ Show destination address verification
☐ Warn on unusual transactions (high value, new addresses)
☐ Implement session timeouts
☐ Support hardware wallet connections

MONITORING:
☐ Track failed transaction patterns
☐ Monitor for unusual activity
☐ Implement rate limiting
☐ Log security-relevant events (sanitized)
☐ Set up alerts for suspicious patterns
```

### Transaction Security

```typescript
// Transaction validation and security
interface TransactionValidation {
  isValid: boolean;
  warnings: string[];
  errors: string[];
}

async function validateTransaction(
  to: string,
  value: bigint,
  data: string,
  userAddress: string
): Promise<TransactionValidation> {
  const warnings: string[] = [];
  const errors: string[] = [];

  // 1. Check destination address
  if (!ethers.isAddress(to)) {
    errors.push("Invalid destination address");
  }

  // 2. Check for contract interaction
  const code = await provider.getCode(to);
  if (code !== "0x") {
    // It's a contract - decode and validate
    try {
      const decoded = decodeTransactionData(to, data);
      warnings.push(`Contract interaction: ${decoded.method}`);

      // Check if it's a known safe contract
      if (!KNOWN_SAFE_CONTRACTS.includes(to.toLowerCase())) {
        warnings.push("Interacting with unverified contract");
      }
    } catch {
      warnings.push("Could not decode contract interaction");
    }
  }

  // 3. Check value
  if (value > 0n) {
    const balance = await provider.getBalance(userAddress);
    if (value > balance) {
      errors.push("Insufficient balance");
    }

    // Large value warning
    const valueInUSD = await getValueInUSD(value);
    if (valueInUSD > 1000) {
      warnings.push(`Large transaction: $${valueInUSD.toFixed(2)}`);
    }
  }

  // 4. Simulate transaction
  try {
    await provider.estimateGas({
      from: userAddress,
      to,
      value,
      data,
    });
  } catch (error: any) {
    if (error.message.includes("execution reverted")) {
      errors.push("Transaction will likely fail");
    }
  }

  // 5. Check for approval to unknown spender
  if (data.startsWith("0x095ea7b3")) {
    // approve(address,uint256)
    const spender = "0x" + data.slice(34, 74);
    if (!KNOWN_SAFE_CONTRACTS.includes(spender.toLowerCase())) {
      warnings.push("Approving tokens to unverified address");
    }
  }

  return {
    isValid: errors.length === 0,
    warnings,
    errors,
  };
}

// Transaction preview component
function TransactionPreview({
  to,
  value,
  data,
  onConfirm,
  onCancel,
}: {
  to: string;
  value: bigint;
  data: string;
  onConfirm: () => void;
  onCancel: () => void;
}) {
  const [validation, setValidation] = useState<TransactionValidation | null>(null);
  const { address } = useAccount();

  useEffect(() => {
    validateTransaction(to, value, data, address!).then(setValidation);
  }, [to, value, data, address]);

  if (!validation) {
    return <div>Validating transaction...</div>;
  }

  return (
    <div className="transaction-preview">
      <h3>Confirm Transaction</h3>

      <div className="details">
        <p>To: {to}</p>
        <p>Value: {formatEther(value)} ETH</p>
        {data !== "0x" && <p>Data: {truncate(data, 20)}</p>}
      </div>

      {validation.warnings.map((warning, i) => (
        <div key={i} className="warning">
          ⚠️ {warning}
        </div>
      ))}

      {validation.errors.map((error, i) => (
        <div key={i} className="error">
          ❌ {error}
        </div>
      ))}

      <div className="actions">
        <button onClick={onCancel}>Cancel</button>
        <button
          onClick={onConfirm}
          disabled={!validation.isValid}
        >
          Confirm
        </button>
      </div>
    </div>
  );
}
```

---

## Implementation Code Examples

### Complete Wallet Integration Hook

```typescript
// hooks/useWallet.ts
import { usePrivy, useWallets } from "@privy-io/react-auth";
import { useSmartWallets } from "@privy-io/react-auth/smart-wallets";
import { useAccount, useConnect, useDisconnect, useWalletClient } from "wagmi";
import { encodeFunctionData, formatEther, parseEther } from "viem";

export interface WalletState {
  isConnected: boolean;
  isLoading: boolean;
  address: string | null;
  smartWalletAddress: string | null;
  balance: string | null;
  walletType: "privy" | "external" | null;
}

export interface WalletActions {
  connect: () => Promise<void>;
  disconnect: () => Promise<void>;
  sendTransaction: (params: {
    to: string;
    data: string;
    value?: bigint;
    gasless?: boolean;
  }) => Promise<string>;
  signMessage: (message: string) => Promise<string>;
  switchChain: (chainId: number) => Promise<void>;
}

export function useWallet(): WalletState & WalletActions {
  // Privy hooks
  const {
    ready: privyReady,
    authenticated: privyAuthenticated,
    login: privyLogin,
    logout: privyLogout,
    user: privyUser,
  } = usePrivy();
  const { wallets: privyWallets } = useWallets();
  const { client: smartWalletClient } = useSmartWallets();

  // Wagmi hooks (for external wallets)
  const { address: wagmiAddress, isConnected: wagmiConnected } = useAccount();
  const { connectAsync, connectors } = useConnect();
  const { disconnectAsync } = useDisconnect();
  const { data: walletClient } = useWalletClient();

  // Determine wallet state
  const embeddedWallet = privyWallets.find(
    (w) => w.walletClientType === "privy"
  );

  const isConnected = privyAuthenticated || wagmiConnected;
  const isLoading = !privyReady;

  // Determine address based on wallet type
  let address: string | null = null;
  let walletType: "privy" | "external" | null = null;

  if (privyAuthenticated && embeddedWallet) {
    address = embeddedWallet.address;
    walletType = "privy";
  } else if (wagmiConnected && wagmiAddress) {
    address = wagmiAddress;
    walletType = "external";
  }

  const smartWalletAddress = privyUser?.smartWallet?.address || null;

  // Balance (you'd fetch this separately)
  const [balance, setBalance] = useState<string | null>(null);

  useEffect(() => {
    if (address) {
      // Fetch balance
      publicClient.getBalance({ address: address as `0x${string}` })
        .then((bal) => setBalance(formatEther(bal)));
    }
  }, [address]);

  // Connect function
  async function connect() {
    // Try Privy first (for social/email login)
    if (privyReady && !privyAuthenticated) {
      await privyLogin();
      return;
    }

    // Fall back to wagmi (for external wallets)
    const connector = connectors.find((c) => c.ready);
    if (connector) {
      await connectAsync({ connector });
    }
  }

  // Disconnect function
  async function disconnect() {
    if (privyAuthenticated) {
      await privyLogout();
    }
    if (wagmiConnected) {
      await disconnectAsync();
    }
  }

  // Send transaction
  async function sendTransaction(params: {
    to: string;
    data: string;
    value?: bigint;
    gasless?: boolean;
  }): Promise<string> {
    const { to, data, value = 0n, gasless = true } = params;

    // Use smart wallet for gasless transactions
    if (gasless && smartWalletClient) {
      const txHash = await smartWalletClient.sendTransaction({
        to: to as `0x${string}`,
        data: data as `0x${string}`,
        value,
      });
      return txHash;
    }

    // Use embedded wallet
    if (embeddedWallet) {
      const provider = await embeddedWallet.getEthereumProvider();
      const txHash = await provider.request({
        method: "eth_sendTransaction",
        params: [
          {
            from: embeddedWallet.address,
            to,
            value: value ? `0x${value.toString(16)}` : "0x0",
            data,
          },
        ],
      });
      return txHash as string;
    }

    // Use external wallet
    if (walletClient) {
      const txHash = await walletClient.sendTransaction({
        to: to as `0x${string}`,
        data: data as `0x${string}`,
        value,
      });
      return txHash;
    }

    throw new Error("No wallet available");
  }

  // Sign message
  async function signMessage(message: string): Promise<string> {
    if (embeddedWallet) {
      const provider = await embeddedWallet.getEthereumProvider();
      return (await provider.request({
        method: "personal_sign",
        params: [message, embeddedWallet.address],
      })) as string;
    }

    if (walletClient) {
      return await walletClient.signMessage({ message });
    }

    throw new Error("No wallet available");
  }

  // Switch chain
  async function switchChain(chainId: number) {
    if (embeddedWallet) {
      await embeddedWallet.switchChain(chainId);
      return;
    }

    if (walletClient) {
      await walletClient.switchChain({ id: chainId });
      return;
    }

    throw new Error("No wallet available");
  }

  return {
    // State
    isConnected,
    isLoading,
    address,
    smartWalletAddress,
    balance,
    walletType,
    // Actions
    connect,
    disconnect,
    sendTransaction,
    signMessage,
    switchChain,
  };
}
```

### Usage in Components

```typescript
// components/BetButton.tsx
import { useWallet } from "@/hooks/useWallet";
import { predictionMarketAbi } from "@/abis";
import { encodeFunctionData, parseUnits } from "viem";

interface BetButtonProps {
  marketId: bigint;
  isYes: boolean;
  amount: string; // In USDC
}

export function BetButton({ marketId, isYes, amount }: BetButtonProps) {
  const { isConnected, connect, sendTransaction, smartWalletAddress } = useWallet();
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleBet() {
    if (!isConnected) {
      await connect();
      return;
    }

    setIsLoading(true);
    setError(null);

    try {
      const amountWei = parseUnits(amount, 6); // USDC has 6 decimals

      // Encode the bet transaction
      const data = encodeFunctionData({
        abi: predictionMarketAbi,
        functionName: "buyShares",
        args: [marketId, isYes, amountWei],
      });

      // Send gasless transaction
      const txHash = await sendTransaction({
        to: PREDICTION_MARKET_ADDRESS,
        data,
        gasless: true, // Use smart wallet for gasless
      });

      console.log("Bet placed:", txHash);
      // Show success UI
    } catch (err: any) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  }

  return (
    <div>
      <button
        onClick={handleBet}
        disabled={isLoading}
        className={`bet-button ${isYes ? "yes" : "no"}`}
      >
        {isLoading ? "Processing..." : isConnected ? `Bet ${amount} USDC` : "Connect Wallet"}
      </button>
      {error && <p className="error">{error}</p>}
    </div>
  );
}
```

---

## References

- [Privy Documentation](https://docs.privy.io/)
- [Dynamic Documentation](https://docs.dynamic.xyz/)
- [Web3Auth Documentation](https://web3auth.io/docs/)
- [Magic Documentation](https://magic.link/docs)
- [Safe SDK Documentation](https://docs.safe.global/)
- [ZeroDev Documentation](https://docs.zerodev.app/)
- [Biconomy Documentation](https://docs.biconomy.io/)
- [Wagmi Documentation](https://wagmi.sh/)
- [Viem Documentation](https://viem.sh/)
- [RainbowKit Documentation](https://rainbowkit.com/docs/)
- [WalletConnect Documentation](https://docs.walletconnect.com/)
- [ERC-4337 Specification](https://eips.ethereum.org/EIPS/eip-4337)
- [Fireblocks Documentation](https://docs.fireblocks.com/)
