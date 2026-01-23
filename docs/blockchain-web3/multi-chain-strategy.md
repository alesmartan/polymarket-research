# Multi-Chain Strategy

## Comprehensive Multi-Chain Architecture for Prediction Markets

This document outlines a strategic approach to deploying a Polymarket-like prediction market across multiple blockchain networks, covering chain selection, bridge integration, cross-chain messaging, and unified liquidity strategies.

---

## Table of Contents

1. [Chain Comparison Matrix](#chain-comparison-matrix)
2. [Selection Criteria](#selection-criteria)
3. [Bridge Integration Options](#bridge-integration-options)
4. [Cross-Chain Messaging](#cross-chain-messaging)
5. [Unified Liquidity Strategies](#unified-liquidity-strategies)
6. [Chain-Specific Considerations](#chain-specific-considerations)
7. [Deployment Priority Roadmap](#deployment-priority-roadmap)
8. [Multi-Chain Architecture Patterns](#multi-chain-architecture-patterns)

---

## Chain Comparison Matrix

### Comprehensive Chain Comparison

```
LAYER 2 & SIDECHAIN COMPARISON MATRIX (2024-2025)
═══════════════════════════════════════════════════════════════════════════════════════════════════

                    │ Polygon  │ Arbitrum │  Base    │ Optimism │ zkSync   │ Avalanche
                    │   PoS    │   One    │          │          │   Era    │  C-Chain
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
TYPE                │ Sidechain│ Optimistic│ Optimistic│ Optimistic│ ZK      │ Sidechain
                    │          │ Rollup   │ Rollup   │ Rollup   │ Rollup   │
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
TVL (Nov 2025)      │ $1.5B    │ $16.6B   │ $10B     │ $6B      │ $570M    │ $2.1B
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
TPS                 │ ~1,000   │ ~40-60   │ ~100     │ ~130     │ ~12-15   │ ~4,500
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
AVG TX COST         │ $0.001-  │ $0.005-  │ $0.001-  │ $0.01-   │ $0.02-   │ $0.03-
                    │ $0.01    │ $0.02    │ $0.03    │ $0.03    │ $0.10    │ $0.05
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
BLOCK TIME          │ 2 sec    │ 0.25 sec │ 2 sec    │ 2 sec    │ 1 sec    │ 2 sec
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
FINALITY            │ ~2 sec   │ ~1 week* │ ~1 week* │ ~1 week* │ ~1 hour  │ ~1 sec
                    │ (soft)   │ (7 days) │ (7 days) │ (7 days) │ (proof)  │ (instant)
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
EVM COMPATIBLE      │ ✅ Full  │ ✅ Full  │ ✅ Full  │ ✅ Full  │ ✅ Full  │ ✅ Full
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
NATIVE AA           │ ❌       │ ❌       │ ❌       │ ❌       │ ✅       │ ❌
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
GAS TOKEN           │ MATIC/POL│ ETH      │ ETH      │ ETH      │ ETH      │ AVAX
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
BRIDGE TO L1        │ 20-30min │ 7 days   │ 7 days   │ 7 days   │ 24 hours │ ~15 min
────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
ECOSYSTEM SIZE      │ 500+     │ 600+     │ 300+     │ 200+     │ 150+     │ 400+
(dApps)             │          │          │          │          │          │

* Soft finality in ~15 minutes, full finality after challenge period
═══════════════════════════════════════════════════════════════════════════════════════════════════
```

### Visual Comparison

```
CHAIN STRENGTHS RADAR
═══════════════════════════════════════════════════════════════════

POLYGON PoS                          ARBITRUM ONE
         TPS                                  TPS
          ▲                                    ▲
         ╱│╲                                  ╱│╲
        ╱ │ ╲                                ╱ │ ╲
    10 ╱  │  ╲                           8  ╱  │  ╲
      ╱   │   ╲                            ╱   │   ╲
Cost─────────────Security         Cost─────────────Security
      ╲   │   ╱                            ╲   │   ╱
    9  ╲  │  ╱ 6                         10 ╲  │  ╱ 9
        ╲ │ ╱                                ╲ │ ╱
         ╲│╱                                  ╲│╱
          ▼                                    ▼
       Ecosystem                            Ecosystem
          8                                    9

BASE                                 OPTIMISM
         TPS                                  TPS
          ▲                                    ▲
         ╱│╲                                  ╱│╲
        ╱ │ ╲                                ╱ │ ╲
     8 ╱  │  ╲                           9  ╱  │  ╲
      ╱   │   ╲                            ╱   │   ╲
Cost─────────────Security         Cost─────────────Security
      ╲   │   ╱                            ╲   │   ╱
    9  ╲  │  ╱ 9                          8 ╲  │  ╱ 9
        ╲ │ ╱                                ╲ │ ╱
         ╲│╱                                  ╲│╱
          ▼                                    ▼
       Ecosystem                            Ecosystem
          7                                    7

zkSync ERA                           AVALANCHE
         TPS                                  TPS
          ▲                                    ▲
         ╱│╲                                  ╱│╲
        ╱ │ ╲                                ╱ │ ╲
     5 ╱  │  ╲                          10 ╱  │  ╲
      ╱   │   ╲                            ╱   │   ╲
Cost─────────────Security         Cost─────────────Security
      ╲   │   ╱                            ╲   │   ╱
     6 ╲  │  ╱ 10                         7 ╲  │  ╱ 8
        ╲ │ ╱                                ╲ │ ╱
         ╲│╱                                  ╲│╱
          ▼                                    ▼
       Ecosystem                            Ecosystem
          5                                    7

Scale: 1-10 (higher is better)
```

### DeFi Ecosystem Comparison

```
DEFI ECOSYSTEM BY CHAIN
═══════════════════════════════════════════════════════════════════

MAJOR PROTOCOLS BY CHAIN:

POLYGON PoS
├── DEXs: QuickSwap, SushiSwap, Uniswap V3
├── Lending: Aave V3, Compound
├── Derivatives: Gains Network
├── NFTs: OpenSea, Rarible
└── Gaming: Sandbox, Decentraland

ARBITRUM ONE
├── DEXs: Uniswap V3, Camelot, SushiSwap
├── Lending: Aave V3, Radiant
├── Derivatives: GMX, dYdX, Gains Network
├── Bridges: Stargate, Hop Protocol
└── Options: Dopex, Lyra

BASE
├── DEXs: Aerodrome, Uniswap V3
├── Lending: Moonwell, Aave V3
├── Social: Friend.tech, Farcaster ecosystem
├── NFTs: Zora, OpenSea
└── Bridges: Base Bridge, Across

OPTIMISM
├── DEXs: Velodrome, Uniswap V3
├── Lending: Aave V3, Sonne Finance
├── Perpetuals: Synthetix, Kwenta
├── Governance: Optimism Collective
└── Bridges: Optimism Bridge, Hop

zkSync ERA
├── DEXs: SyncSwap, Mute.io
├── Lending: ReactorFusion, ZeroLend
├── DEX Aggregators: 1inch (native)
├── Wallets: Argent (native AA)
└── Bridges: zkSync Bridge, LayerSwap

AVALANCHE C-Chain
├── DEXs: Trader Joe, Pangolin
├── Lending: Aave V3, Benqi
├── Derivatives: GMX (multichain)
├── Gaming: DeFi Kingdoms, Crabada
└── Bridges: Avalanche Bridge, LayerZero
```

---

## Selection Criteria

### Decision Framework

```
CHAIN SELECTION DECISION TREE
═══════════════════════════════════════════════════════════════════

                    START
                      │
                      ▼
        ┌─────────────────────────────┐
        │ Primary user demographic?   │
        └─────────────────────────────┘
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
      Crypto       Mainstream   Institutional
      Native       Users        Users
          │           │           │
          ▼           ▼           ▼
     ┌────────┐  ┌────────┐  ┌────────┐
     │ARBITRUM│  │  BASE  │  │ETHEREUM│
     │(DeFi   │  │(Coinbase│  │   or   │
     │users)  │  │onramp) │  │ARBITRUM│
     └────────┘  └────────┘  └────────┘
          │           │           │
          ▼           ▼           ▼
        ┌─────────────────────────────┐
        │    Cost sensitivity?        │
        └─────────────────────────────┘
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       Very         Moderate     Not
       High         Important    Important
          │           │           │
          ▼           ▼           ▼
     ┌────────┐  ┌────────┐  ┌────────┐
     │POLYGON │  │ARBITRUM│  │ETHEREUM│
     │  PoS   │  │  BASE  │  │zkSync  │
     │(lowest │  │OPTIMISM│  │(best   │
     │ cost)  │  │        │  │security│
     └────────┘  └────────┘  └────────┘
          │           │           │
          ▼           ▼           ▼
        ┌─────────────────────────────┐
        │  Liquidity requirements?    │
        └─────────────────────────────┘
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Deep        Medium       Low
          │           │           │
     ┌────────┐  ┌────────┐  ┌────────┐
     │ARBITRUM│  │POLYGON │  │ zkSync │
     │($16B   │  │($1.5B  │  │ or new │
     │ TVL)   │  │ TVL)   │  │ chains │
     └────────┘  └────────┘  └────────┘
```

### Weighted Scoring Matrix

```typescript
interface ChainScore {
  chain: string;
  scores: {
    transactionCost: number;      // 1-10 (10 = cheapest)
    security: number;             // 1-10 (10 = most secure)
    liquidity: number;            // 1-10 (10 = deepest)
    developerExperience: number;  // 1-10 (10 = best)
    userExperience: number;       // 1-10 (10 = best)
    ecosystem: number;            // 1-10 (10 = largest)
    futureProof: number;          // 1-10 (10 = best outlook)
  };
  weightedTotal: number;
}

const WEIGHTS = {
  transactionCost: 0.20,      // Critical for prediction markets
  security: 0.20,             // Handling user funds
  liquidity: 0.15,            // For market making
  developerExperience: 0.10,  // Build speed
  userExperience: 0.15,       // Adoption
  ecosystem: 0.10,            // Integrations
  futureProof: 0.10,          // Long-term viability
};

const chainScores: ChainScore[] = [
  {
    chain: "Polygon PoS",
    scores: {
      transactionCost: 10,
      security: 6,
      liquidity: 7,
      developerExperience: 9,
      userExperience: 9,
      ecosystem: 8,
      futureProof: 7,
    },
    weightedTotal: 7.85, // Calculated
  },
  {
    chain: "Arbitrum One",
    scores: {
      transactionCost: 9,
      security: 9,
      liquidity: 10,
      developerExperience: 9,
      userExperience: 8,
      ecosystem: 9,
      futureProof: 9,
    },
    weightedTotal: 9.05, // HIGHEST
  },
  {
    chain: "Base",
    scores: {
      transactionCost: 10,
      security: 9,
      liquidity: 9,
      developerExperience: 9,
      userExperience: 10,
      ecosystem: 7,
      futureProof: 9,
    },
    weightedTotal: 9.15, // HIGHEST (user experience focus)
  },
  {
    chain: "Optimism",
    scores: {
      transactionCost: 8,
      security: 9,
      liquidity: 8,
      developerExperience: 9,
      userExperience: 8,
      ecosystem: 7,
      futureProof: 9,
    },
    weightedTotal: 8.40,
  },
  {
    chain: "zkSync Era",
    scores: {
      transactionCost: 7,
      security: 10,
      liquidity: 5,
      developerExperience: 7,
      userExperience: 7,
      ecosystem: 5,
      futureProof: 10,
    },
    weightedTotal: 7.50,
  },
  {
    chain: "Avalanche",
    scores: {
      transactionCost: 7,
      security: 8,
      liquidity: 7,
      developerExperience: 8,
      userExperience: 8,
      ecosystem: 7,
      futureProof: 7,
    },
    weightedTotal: 7.45,
  },
];
```

### Selection Recommendation

```
RECOMMENDED DEPLOYMENT STRATEGY
═══════════════════════════════════════════════════════════════════

PRIMARY CHAIN: Polygon PoS
├── Reason: Polymarket's proven infrastructure
├── Lowest transaction costs
├── Largest existing user base for prediction markets
├── Best for high-frequency, low-value trades
└── Established oracle and liquidity infrastructure

SECONDARY CHAIN: Base
├── Reason: Coinbase ecosystem integration
├── Seamless fiat onramp via Coinbase
├── Growing mainstream user base
├── OP Stack benefits (shared security model)
└── Strong backing and long-term support

TERTIARY CHAIN: Arbitrum One
├── Reason: Deepest DeFi liquidity
├── Best for institutional/whale users
├── GMX-style derivatives infrastructure
├── Strongest DeFi ecosystem
└── Natural expansion for power users

FUTURE CONSIDERATION: zkSync Era
├── Reason: Best security guarantees
├── Native account abstraction
├── Future-proof ZK technology
├── Watch for ecosystem growth
└── Consider when TVL exceeds $2B
```

---

## Bridge Integration Options

### Bridge Comparison

```
BRIDGE COMPARISON MATRIX
═══════════════════════════════════════════════════════════════════

                  │ Security │ Speed    │ Cost    │ Chains │ Volume
──────────────────┼──────────┼──────────┼─────────┼────────┼────────
Native Bridges    │ ★★★★★   │ ★★☆☆☆   │ ★★★★★  │ ★☆☆☆☆ │ ★★★★★
(e.g., Arb/OP)    │ Highest  │ 7 days   │ Low     │ 2      │ Highest
──────────────────┼──────────┼──────────┼─────────┼────────┼────────
Stargate          │ ★★★★☆   │ ★★★★☆   │ ★★★☆☆  │ ★★★★★ │ ★★★★☆
(LayerZero)       │ High     │ Minutes  │ Medium  │ 15+    │ High
──────────────────┼──────────┼──────────┼─────────┼────────┼────────
Across Protocol   │ ★★★★☆   │ ★★★★★   │ ★★★★☆  │ ★★★★☆ │ ★★★★☆
                  │ High     │ Seconds  │ Low     │ 8+     │ High
──────────────────┼──────────┼──────────┼─────────┼────────┼────────
Hop Protocol      │ ★★★★☆   │ ★★★★☆   │ ★★★★☆  │ ★★★☆☆ │ ★★★☆☆
                  │ High     │ Minutes  │ Low     │ 6+     │ Medium
──────────────────┼──────────┼──────────┼─────────┼────────┼────────
Synapse           │ ★★★☆☆   │ ★★★★☆   │ ★★★☆☆  │ ★★★★★ │ ★★★☆☆
                  │ Medium   │ Minutes  │ Medium  │ 15+    │ Medium
──────────────────┼──────────┼──────────┼─────────┼────────┼────────
Celer cBridge     │ ★★★☆☆   │ ★★★★☆   │ ★★★☆☆  │ ★★★★★ │ ★★★☆☆
                  │ Medium   │ Minutes  │ Medium  │ 20+    │ Medium
──────────────────┼──────────┼──────────┼─────────┼────────┼────────
Socket/Bungee     │ ★★★★☆   │ ★★★★☆   │ ★★★★★  │ ★★★★★ │ ★★★★☆
(Aggregator)      │ Varies   │ Minutes  │ Best    │ 15+    │ High

Rating: ★ = Poor, ★★★★★ = Excellent
```

### Bridge Integration Architecture

```
BRIDGE INTEGRATION PATTERN
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                        USER INTERFACE                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                  Bridge Aggregator UI                      │  │
│  │  • Auto-selects best bridge for route                     │  │
│  │  • Shows fee comparison                                   │  │
│  │  • Displays estimated time                                │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BRIDGE AGGREGATOR                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Socket / LI.FI API                                       │  │
│  │  • Routes to cheapest/fastest bridge                      │  │
│  │  • Handles multiple hops if needed                        │  │
│  │  • Provides unified status tracking                       │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   Stargate      │ │    Across       │ │   Hop Protocol  │
│   (LayerZero)   │ │   Protocol      │ │                 │
└────────┬────────┘ └────────┬────────┘ └────────┬────────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    DESTINATION CHAIN                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Prediction Market Receiver Contract                      │  │
│  │  • Receives bridged tokens                                │  │
│  │  • Auto-deposits to user account                          │  │
│  │  • Optional: Auto-executes pending orders                 │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation Code

```typescript
// Bridge Aggregator Integration using Socket API
import axios from "axios";

interface BridgeQuote {
  bridgeName: string;
  fromChainId: number;
  toChainId: number;
  fromToken: string;
  toToken: string;
  fromAmount: string;
  toAmount: string;
  estimatedTime: number; // seconds
  fee: string;
  route: any;
}

class BridgeAggregator {
  private socketApiKey: string;
  private baseUrl = "https://api.socket.tech/v2";

  constructor(apiKey: string) {
    this.socketApiKey = apiKey;
  }

  async getQuote(params: {
    fromChainId: number;
    toChainId: number;
    fromToken: string;
    toToken: string;
    amount: string;
    userAddress: string;
  }): Promise<BridgeQuote[]> {
    const response = await axios.get(`${this.baseUrl}/quote`, {
      headers: { "API-KEY": this.socketApiKey },
      params: {
        fromChainId: params.fromChainId,
        toChainId: params.toChainId,
        fromTokenAddress: params.fromToken,
        toTokenAddress: params.toToken,
        fromAmount: params.amount,
        userAddress: params.userAddress,
        uniqueRoutesPerBridge: true,
        sort: "output", // Sort by best output
      },
    });

    return response.data.result.routes.map((route: any) => ({
      bridgeName: route.usedBridgeNames.join(" + "),
      fromChainId: params.fromChainId,
      toChainId: params.toChainId,
      fromToken: params.fromToken,
      toToken: params.toToken,
      fromAmount: params.amount,
      toAmount: route.toAmount,
      estimatedTime: route.serviceTime,
      fee: route.totalGasFeesInUsd,
      route: route,
    }));
  }

  async executeBridge(
    route: any,
    userAddress: string,
    signer: any
  ): Promise<string> {
    // Get build transaction
    const buildResponse = await axios.post(
      `${this.baseUrl}/build-tx`,
      {
        route: route,
        userAddress: userAddress,
      },
      {
        headers: { "API-KEY": this.socketApiKey },
      }
    );

    const txData = buildResponse.data.result;

    // Execute transaction
    const tx = await signer.sendTransaction({
      to: txData.txTarget,
      data: txData.txData,
      value: txData.value,
    });

    return tx.hash;
  }

  async trackBridgeStatus(
    txHash: string,
    fromChainId: number,
    toChainId: number
  ): Promise<{
    status: "pending" | "completed" | "failed";
    destinationTxHash?: string;
  }> {
    const response = await axios.get(`${this.baseUrl}/bridge-status`, {
      headers: { "API-KEY": this.socketApiKey },
      params: {
        transactionHash: txHash,
        fromChainId: fromChainId,
        toChainId: toChainId,
      },
    });

    return {
      status: response.data.result.destinationTxStatus,
      destinationTxHash: response.data.result.destinationTransactionHash,
    };
  }
}
```

---

## Cross-Chain Messaging

### Protocol Comparison

```
CROSS-CHAIN MESSAGING PROTOCOLS
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                        LayerZero V2                              │
├─────────────────────────────────────────────────────────────────┤
│  Security Model: Decentralized Verifier Networks (DVNs)         │
│  Chains Supported: 70+                                          │
│  Message Time: 1-5 minutes                                      │
│  Cost: Medium                                                   │
│                                                                  │
│  Pros:                                                          │
│  • Highly configurable security                                 │
│  • App-level security choices                                   │
│  • Large ecosystem (Stargate, etc.)                            │
│                                                                  │
│  Cons:                                                          │
│  • Complexity in DVN selection                                  │
│  • Recent centralization concerns                               │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                          Axelar                                  │
├─────────────────────────────────────────────────────────────────┤
│  Security Model: PoS Validator Set (75 validators)              │
│  Chains Supported: 64+                                          │
│  Message Time: 2-5 minutes                                      │
│  Cost: Medium                                                   │
│                                                                  │
│  Pros:                                                          │
│  • Strong Cosmos ecosystem integration                          │
│  • General Message Passing (GMP)                               │
│  • Consistent security model                                    │
│                                                                  │
│  Cons:                                                          │
│  • Smaller validator set than Ethereum                          │
│  • Higher latency than some competitors                         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                         Wormhole                                 │
├─────────────────────────────────────────────────────────────────┤
│  Security Model: Guardian Network (19 validators)               │
│  Chains Supported: 30+                                          │
│  Message Time: ~15 minutes                                      │
│  Cost: Low-Medium                                               │
│                                                                  │
│  Pros:                                                          │
│  • Solana native support                                        │
│  • 1B+ messages processed                                       │
│  • Uniswap approved                                             │
│                                                                  │
│  Cons:                                                          │
│  • 2022 exploit history ($325M)                                 │
│  • Smaller Guardian set                                         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       Chainlink CCIP                             │
├─────────────────────────────────────────────────────────────────┤
│  Security Model: Oracle Network + Risk Management               │
│  Chains Supported: 12+ (growing)                                │
│  Message Time: ~20 minutes                                      │
│  Cost: Higher                                                   │
│                                                                  │
│  Pros:                                                          │
│  • Most battle-tested oracle infrastructure                     │
│  • Active Risk Management Network                               │
│  • Enterprise-grade SLAs                                        │
│                                                                  │
│  Cons:                                                          │
│  • Limited chain support currently                              │
│  • Higher costs                                                 │
│  • Slower message delivery                                      │
└─────────────────────────────────────────────────────────────────┘
```

### Cross-Chain Market State Synchronization

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {ILayerZeroEndpointV2} from "@layerzero-v2/protocol/contracts/interfaces/ILayerZeroEndpointV2.sol";
import {OApp} from "@layerzero-v2/oapp/contracts/oapp/OApp.sol";

contract CrossChainMarketSync is OApp {
    // Message types
    uint16 constant MSG_TYPE_MARKET_CREATE = 1;
    uint16 constant MSG_TYPE_MARKET_RESOLVE = 2;
    uint16 constant MSG_TYPE_LIQUIDITY_SYNC = 3;

    // Market data structure
    struct MarketData {
        uint256 marketId;
        bytes32 questionHash;
        uint256 endTime;
        uint256 totalLiquidity;
        bool resolved;
        bool outcome;
    }

    // Chain-specific market states
    mapping(uint256 => MarketData) public markets;

    // Mapping of supported chain endpoints
    mapping(uint32 => bool) public supportedChains;

    event CrossChainMessageSent(uint32 dstEid, uint16 msgType, bytes payload);
    event CrossChainMessageReceived(uint32 srcEid, uint16 msgType, bytes payload);
    event MarketSynced(uint256 marketId, uint32 sourceChain);

    constructor(
        address _endpoint,
        address _owner
    ) OApp(_endpoint, _owner) {}

    /// @notice Broadcast market creation to all chains
    function broadcastMarketCreation(
        uint256 marketId,
        bytes32 questionHash,
        uint256 endTime,
        uint32[] calldata dstEids
    ) external payable onlyOwner {
        bytes memory payload = abi.encode(
            MSG_TYPE_MARKET_CREATE,
            marketId,
            questionHash,
            endTime
        );

        uint256 totalFee = 0;
        for (uint256 i = 0; i < dstEids.length; i++) {
            require(supportedChains[dstEids[i]], "Chain not supported");

            bytes memory options = _buildOptions(200000); // Gas limit
            MessagingFee memory fee = _quote(dstEids[i], payload, options, false);
            totalFee += fee.nativeFee;
        }

        require(msg.value >= totalFee, "Insufficient fee");

        for (uint256 i = 0; i < dstEids.length; i++) {
            bytes memory options = _buildOptions(200000);
            _lzSend(
                dstEids[i],
                payload,
                options,
                MessagingFee(msg.value / dstEids.length, 0),
                payable(msg.sender)
            );

            emit CrossChainMessageSent(dstEids[i], MSG_TYPE_MARKET_CREATE, payload);
        }
    }

    /// @notice Broadcast market resolution to all chains
    function broadcastMarketResolution(
        uint256 marketId,
        bool outcome,
        uint32[] calldata dstEids
    ) external payable onlyOwner {
        require(markets[marketId].endTime > 0, "Market doesn't exist");

        bytes memory payload = abi.encode(
            MSG_TYPE_MARKET_RESOLVE,
            marketId,
            outcome
        );

        for (uint256 i = 0; i < dstEids.length; i++) {
            bytes memory options = _buildOptions(150000);
            _lzSend(
                dstEids[i],
                payload,
                options,
                MessagingFee(msg.value / dstEids.length, 0),
                payable(msg.sender)
            );

            emit CrossChainMessageSent(dstEids[i], MSG_TYPE_MARKET_RESOLVE, payload);
        }
    }

    /// @notice Receive cross-chain messages
    function _lzReceive(
        Origin calldata _origin,
        bytes32 _guid,
        bytes calldata _message,
        address _executor,
        bytes calldata _extraData
    ) internal override {
        (uint16 msgType, ) = abi.decode(_message, (uint16, bytes));

        if (msgType == MSG_TYPE_MARKET_CREATE) {
            _handleMarketCreate(_origin.srcEid, _message);
        } else if (msgType == MSG_TYPE_MARKET_RESOLVE) {
            _handleMarketResolve(_origin.srcEid, _message);
        } else if (msgType == MSG_TYPE_LIQUIDITY_SYNC) {
            _handleLiquiditySync(_origin.srcEid, _message);
        }

        emit CrossChainMessageReceived(_origin.srcEid, msgType, _message);
    }

    function _handleMarketCreate(uint32 srcEid, bytes calldata message) internal {
        (, uint256 marketId, bytes32 questionHash, uint256 endTime) =
            abi.decode(message, (uint16, uint256, bytes32, uint256));

        markets[marketId] = MarketData({
            marketId: marketId,
            questionHash: questionHash,
            endTime: endTime,
            totalLiquidity: 0,
            resolved: false,
            outcome: false
        });

        emit MarketSynced(marketId, srcEid);
    }

    function _handleMarketResolve(uint32 srcEid, bytes calldata message) internal {
        (, uint256 marketId, bool outcome) =
            abi.decode(message, (uint16, uint256, bool));

        MarketData storage market = markets[marketId];
        require(!market.resolved, "Already resolved");

        market.resolved = true;
        market.outcome = outcome;

        // Trigger local payouts
        _processPayouts(marketId, outcome);

        emit MarketSynced(marketId, srcEid);
    }

    function _handleLiquiditySync(uint32 srcEid, bytes calldata message) internal {
        (, uint256 marketId, uint256 newLiquidity) =
            abi.decode(message, (uint16, uint256, uint256));

        markets[marketId].totalLiquidity = newLiquidity;
    }

    function _buildOptions(uint128 gasLimit) internal pure returns (bytes memory) {
        return abi.encodePacked(uint16(1), gasLimit);
    }

    function _processPayouts(uint256 marketId, bool outcome) internal {
        // Implementation: Process winning positions
    }
}
```

### Message Flow Diagram

```
CROSS-CHAIN MARKET RESOLUTION FLOW
═══════════════════════════════════════════════════════════════════

        PRIMARY CHAIN (Polygon)              SECONDARY CHAINS
              │                          ┌─────────┬─────────┐
              │                          │Arbitrum │  Base   │
              │                          └────┬────┴────┬────┘
              │                               │         │
    ┌─────────▼─────────┐                     │         │
    │  Oracle Reports   │                     │         │
    │  Market Outcome   │                     │         │
    └─────────┬─────────┘                     │         │
              │                               │         │
    ┌─────────▼─────────┐                     │         │
    │  Resolution       │                     │         │
    │  Finalized        │                     │         │
    └─────────┬─────────┘                     │         │
              │                               │         │
    ┌─────────▼─────────┐                     │         │
    │  Broadcast via    │─────────────────────┼─────────┤
    │  LayerZero        │                     │         │
    └─────────┬─────────┘                     │         │
              │                               │         │
              │         MSG_MARKET_RESOLVE    │         │
              ├───────────────────────────────▶         │
              │                               │         │
              │         MSG_MARKET_RESOLVE    │         │
              ├─────────────────────────────────────────▶
              │                               │         │
              │                    ┌──────────▼──────┐  │
              │                    │  Verify &       │  │
              │                    │  Process        │  │
              │                    └──────────┬──────┘  │
              │                               │         │
              │                    ┌──────────▼──────┐  │
              │                    │  Local Payouts  │  │
              │                    └─────────────────┘  │
              │                                         │
              │                              ┌──────────▼──────┐
              │                              │  Verify &       │
              │                              │  Process        │
              │                              └──────────┬──────┘
              │                                         │
              │                              ┌──────────▼──────┐
              │                              │  Local Payouts  │
              │                              └─────────────────┘
```

---

## Unified Liquidity Strategies

### Liquidity Architecture Options

```
UNIFIED LIQUIDITY APPROACHES
═══════════════════════════════════════════════════════════════════

OPTION 1: Hub and Spoke Model
───────────────────────────────────────────────────────────────────
                         ┌─────────────────┐
                         │   PRIMARY HUB   │
                         │    (Polygon)    │
                         │                 │
                         │  Main Liquidity │
                         │     Pool        │
                         └────────┬────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
       ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
       │  Arbitrum   │     │    Base     │     │  Optimism   │
       │   Spoke     │     │   Spoke     │     │   Spoke     │
       │             │     │             │     │             │
       │ Local Cache │     │ Local Cache │     │ Local Cache │
       └─────────────┘     └─────────────┘     └─────────────┘

   Pros: Centralized liquidity, simpler management
   Cons: Hub dependency, cross-chain latency for large trades


OPTION 2: Fully Distributed Model
───────────────────────────────────────────────────────────────────
       ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
       │  Polygon    │◄───▶│  Arbitrum   │◄───▶│    Base     │
       │             │     │             │     │             │
       │ Independent │     │ Independent │     │ Independent │
       │  Liquidity  │     │  Liquidity  │     │  Liquidity  │
       └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  │
                         ┌────────▼────────┐
                         │   Arbitrage     │
                         │     Bots        │
                         │ (Keep in sync)  │
                         └─────────────────┘

   Pros: No single point of failure, local execution
   Cons: Liquidity fragmentation, complex arbitrage


OPTION 3: Virtual Aggregated Liquidity
───────────────────────────────────────────────────────────────────
                    ┌─────────────────────────┐
                    │  VIRTUAL LIQUIDITY LAYER │
                    │                          │
                    │  Aggregates quotes from  │
                    │  all chains in real-time │
                    └────────────┬─────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
         ▼                       ▼                       ▼
   ┌───────────┐          ┌───────────┐          ┌───────────┐
   │  Polygon  │          │ Arbitrum  │          │   Base    │
   │   Pool    │          │   Pool    │          │   Pool    │
   │   $10M    │          │   $25M    │          │   $15M    │
   └───────────┘          └───────────┘          └───────────┘

                    Virtual Total: $50M
                    Best execution routing

   Pros: Deep virtual liquidity, best prices
   Cons: Complex implementation, cross-chain settlement
```

### Unified Liquidity Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "./CrossChainMarketSync.sol";

contract UnifiedLiquidityPool is CrossChainMarketSync {
    struct ChainLiquidity {
        uint32 chainId;
        uint256 availableLiquidity;
        uint256 lastUpdate;
    }

    // Aggregated view of liquidity across chains
    mapping(uint256 => mapping(uint32 => ChainLiquidity)) public marketChainLiquidity;

    // Local liquidity for this chain
    mapping(uint256 => uint256) public localLiquidity;

    /// @notice Get best execution route for a trade
    function getBestExecution(
        uint256 marketId,
        uint256 amount,
        bool isBuy
    ) external view returns (
        uint32 bestChain,
        uint256 expectedOutput,
        uint256 slippage
    ) {
        // Check local liquidity first
        uint256 localOutput = _simulateTrade(marketId, amount, isBuy, localLiquidity[marketId]);

        bestChain = uint32(block.chainid);
        expectedOutput = localOutput;
        slippage = _calculateSlippage(localLiquidity[marketId], amount);

        // Check other chains
        uint32[] memory chains = getSupportedChains();
        for (uint256 i = 0; i < chains.length; i++) {
            ChainLiquidity memory cl = marketChainLiquidity[marketId][chains[i]];

            if (cl.lastUpdate + 5 minutes > block.timestamp) {
                uint256 output = _simulateTrade(marketId, amount, isBuy, cl.availableLiquidity);
                uint256 chainSlippage = _calculateSlippage(cl.availableLiquidity, amount);

                if (output > expectedOutput || (output == expectedOutput && chainSlippage < slippage)) {
                    bestChain = chains[i];
                    expectedOutput = output;
                    slippage = chainSlippage;
                }
            }
        }
    }

    /// @notice Execute trade with cross-chain routing if beneficial
    function executeWithRouting(
        uint256 marketId,
        uint256 amount,
        bool isBuy,
        uint256 minOutput,
        uint32 preferredChain
    ) external payable returns (bytes32 orderId) {
        // If preferred chain is local, execute directly
        if (preferredChain == uint32(block.chainid)) {
            return _executeLocalTrade(msg.sender, marketId, amount, isBuy, minOutput);
        }

        // Otherwise, bridge and execute cross-chain
        orderId = _initiateCrossChainTrade(
            msg.sender,
            marketId,
            amount,
            isBuy,
            minOutput,
            preferredChain
        );
    }

    /// @notice Sync local liquidity to other chains
    function syncLiquidityToChains(
        uint256 marketId,
        uint32[] calldata dstChains
    ) external payable {
        uint256 currentLiquidity = localLiquidity[marketId];

        bytes memory payload = abi.encode(
            MSG_TYPE_LIQUIDITY_SYNC,
            marketId,
            currentLiquidity
        );

        for (uint256 i = 0; i < dstChains.length; i++) {
            bytes memory options = _buildOptions(100000);
            _lzSend(
                dstChains[i],
                payload,
                options,
                MessagingFee(msg.value / dstChains.length, 0),
                payable(msg.sender)
            );
        }
    }

    function _simulateTrade(
        uint256 marketId,
        uint256 amount,
        bool isBuy,
        uint256 liquidity
    ) internal pure returns (uint256) {
        // Simplified CPMM calculation
        // In production: use actual AMM formula
        return (amount * liquidity) / (liquidity + amount);
    }

    function _calculateSlippage(
        uint256 liquidity,
        uint256 amount
    ) internal pure returns (uint256) {
        // Returns slippage in basis points
        return (amount * 10000) / liquidity;
    }

    function _executeLocalTrade(
        address user,
        uint256 marketId,
        uint256 amount,
        bool isBuy,
        uint256 minOutput
    ) internal returns (bytes32) {
        // Execute trade on local chain
        // Implementation details...
        return keccak256(abi.encode(user, marketId, amount, block.timestamp));
    }

    function _initiateCrossChainTrade(
        address user,
        uint256 marketId,
        uint256 amount,
        bool isBuy,
        uint256 minOutput,
        uint32 dstChain
    ) internal returns (bytes32) {
        // Bridge tokens and execute on destination
        // Implementation details...
        return keccak256(abi.encode(user, marketId, amount, dstChain, block.timestamp));
    }
}
```

---

## Chain-Specific Considerations

### Polygon PoS

```
POLYGON PoS CONSIDERATIONS
═══════════════════════════════════════════════════════════════════

ADVANTAGES:
├── Lowest transaction costs (~$0.001)
├── Fastest soft finality (~2 seconds)
├── Largest prediction market user base (Polymarket)
├── Strong USDC liquidity
├── Well-established oracle infrastructure
└── Mature developer tooling

CONSIDERATIONS:
├── Security model weaker than L2 rollups
├── Recent POL migration (from MATIC)
├── Reorg risk (though rare)
└── Less decentralized than Ethereum

TECHNICAL NOTES:
┌─────────────────────────────────────────────────────────────────┐
│  RPC Endpoints:                                                  │
│  • Primary: https://polygon-rpc.com                             │
│  • Backup: https://rpc.ankr.com/polygon                         │
│                                                                  │
│  Block Confirmations: 64 blocks recommended for finality        │
│                                                                  │
│  Gas Token: POL (previously MATIC)                              │
│  Gas Price: 30-100 gwei typical                                 │
│                                                                  │
│  Key Contracts:                                                  │
│  • USDC: 0x3c499c542cef5e3811e1192ce70d8cc03d5c3359            │
│  • USDC.e: 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174          │
│  • Chainlink Price Feed: 0xAB594600376Ec9fD91F8e885dADF0CE8fE  │
└─────────────────────────────────────────────────────────────────┘
```

### Arbitrum One

```
ARBITRUM ONE CONSIDERATIONS
═══════════════════════════════════════════════════════════════════

ADVANTAGES:
├── Highest TVL among L2s ($16.6B)
├── Deepest DeFi liquidity
├── Full EVM equivalence
├── Strong security (Ethereum L1 settlement)
├── Battle-tested with major protocols
└── Active governance (ARB token)

CONSIDERATIONS:
├── 7-day withdrawal period to L1
├── Sequencer centralization (being addressed)
├── Higher costs than Polygon PoS
└── Less mainstream user adoption than Base

TECHNICAL NOTES:
┌─────────────────────────────────────────────────────────────────┐
│  RPC Endpoints:                                                  │
│  • Primary: https://arb1.arbitrum.io/rpc                        │
│  • Backup: https://arbitrum.llamarpc.com                        │
│                                                                  │
│  L1 → L2: ~10 minutes                                           │
│  L2 → L1: 7 days (challenge period)                             │
│                                                                  │
│  Gas Token: ETH                                                  │
│  Gas Price: 0.01-0.1 gwei typical                               │
│                                                                  │
│  Key Contracts:                                                  │
│  • USDC: 0xaf88d065e77c8cC2239327C5EDb3A432268e5831            │
│  • USDC.e: 0xFF970A61A04b1cA14834A43f5dE4533eBDDB5CC8          │
│  • Arbitrum Gateway: 0x72Ce9c846789fdB6fC1f34aC4AD25Dd9ef7031  │
│                                                                  │
│  Unique Features:                                                │
│  • Stylus (Rust/C smart contracts) coming                       │
│  • Timeboost (priority ordering auction)                        │
└─────────────────────────────────────────────────────────────────┘
```

### Base

```
BASE CONSIDERATIONS
═══════════════════════════════════════════════════════════════════

ADVANTAGES:
├── Coinbase ecosystem integration
├── Seamless fiat on/off ramp
├── Massive mainstream user potential
├── OP Stack (shared security benefits)
├── Low fees with high throughput
└── Strong social/consumer app focus

CONSIDERATIONS:
├── More centralized (Coinbase sequencer)
├── Younger ecosystem than competitors
├── Less DeFi depth than Arbitrum
└── Reputation tied to Coinbase

TECHNICAL NOTES:
┌─────────────────────────────────────────────────────────────────┐
│  RPC Endpoints:                                                  │
│  • Primary: https://mainnet.base.org                            │
│  • Backup: https://base.llamarpc.com                            │
│                                                                  │
│  L1 → L2: ~5 minutes                                            │
│  L2 → L1: 7 days (challenge period)                             │
│                                                                  │
│  Gas Token: ETH                                                  │
│  Gas Price: 0.001-0.01 gwei typical (very low!)                 │
│                                                                  │
│  Key Contracts:                                                  │
│  • USDC: 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913            │
│  • Base Bridge: 0x49048044D57e1C92A77f79988d21Fa8fAF74E97e      │
│                                                                  │
│  Unique Features:                                                │
│  • Coinbase Smart Wallet native support                         │
│  • Verifications (on-chain identity)                            │
│  • Strong social primitives (Farcaster integration)             │
└─────────────────────────────────────────────────────────────────┘
```

### Optimism

```
OPTIMISM CONSIDERATIONS
═══════════════════════════════════════════════════════════════════

ADVANTAGES:
├── OP Stack foundation (powers Base, others)
├── Strong governance (Optimism Collective)
├── Superchain vision (shared network)
├── Synthetix ecosystem (derivatives)
├── Retroactive public goods funding
└── Well-documented, developer-friendly

CONSIDERATIONS:
├── Lower TVL than Arbitrum
├── Competition from Base (same tech)
├── Sequencer centralization
└── Smaller unique DeFi presence

TECHNICAL NOTES:
┌─────────────────────────────────────────────────────────────────┐
│  RPC Endpoints:                                                  │
│  • Primary: https://mainnet.optimism.io                         │
│  • Backup: https://optimism.llamarpc.com                        │
│                                                                  │
│  L1 → L2: ~5 minutes                                            │
│  L2 → L1: 7 days (challenge period)                             │
│                                                                  │
│  Gas Token: ETH                                                  │
│  Gas Price: 0.001-0.1 gwei typical                              │
│                                                                  │
│  Key Contracts:                                                  │
│  • USDC: 0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85            │
│  • OP Token: 0x4200000000000000000000000000000000000042          │
│                                                                  │
│  Unique Features:                                                │
│  • Superchain interoperability                                   │
│  • Bedrock upgrade (shared with Base)                           │
│  • RetroPGF grant program                                        │
└─────────────────────────────────────────────────────────────────┘
```

### zkSync Era

```
zkSync ERA CONSIDERATIONS
═══════════════════════════════════════════════════════════════════

ADVANTAGES:
├── Native account abstraction (built-in!)
├── ZK-proof security (no fraud proofs)
├── Fastest L1 finality (~1 hour)
├── Future-proof technology
├── Paymaster support out of box
└── Strong cryptographic guarantees

CONSIDERATIONS:
├── Smaller ecosystem currently
├── Higher costs than optimistic rollups
├── EVM differences (requires testing)
├── Newer, less battle-tested
└── Lower TVL ($570M)

TECHNICAL NOTES:
┌─────────────────────────────────────────────────────────────────┐
│  RPC Endpoints:                                                  │
│  • Primary: https://mainnet.era.zksync.io                       │
│  • Backup: https://zksync.drpc.org                              │
│                                                                  │
│  L1 → L2: ~15 minutes                                           │
│  L2 → L1: ~24 hours (proof generation)                          │
│                                                                  │
│  Gas Token: ETH                                                  │
│  Gas Price: 0.25 gwei typical                                   │
│                                                                  │
│  Key Contracts:                                                  │
│  • USDC: 0x3355df6D4c9C3035724Fd0e3914dE96A5a83aaf4            │
│                                                                  │
│  Unique Features:                                                │
│  • Native account abstraction (AA)                              │
│  • Paymasters for gasless transactions                          │
│  • Contract must be deployed differently (zksolc)               │
│                                                                  │
│  EVM Differences:                                                │
│  • Different bytecode (zkEVM vs EVM)                            │
│  • Some opcodes behave differently                              │
│  • Requires zkSync-specific compilation                         │
└─────────────────────────────────────────────────────────────────┘
```

### Avalanche C-Chain

```
AVALANCHE C-CHAIN CONSIDERATIONS
═══════════════════════════════════════════════════════════════════

ADVANTAGES:
├── Sub-second finality (~1 second!)
├── High throughput (4,500 TPS)
├── Subnet architecture for scaling
├── Strong institutional interest
├── Gaming ecosystem (DeFi Kingdoms)
└── Independent consensus (not Ethereum L2)

CONSIDERATIONS:
├── Separate security from Ethereum
├── Different ecosystem than EVM L2s
├── Higher base costs than L2s
├── Less DeFi composability with ETH
└── AVAX gas token (not ETH)

TECHNICAL NOTES:
┌─────────────────────────────────────────────────────────────────┐
│  RPC Endpoints:                                                  │
│  • Primary: https://api.avax.network/ext/bc/C/rpc              │
│  • Backup: https://avalanche.drpc.org                           │
│                                                                  │
│  Cross-chain: Avalanche Bridge (~15 minutes)                    │
│                                                                  │
│  Gas Token: AVAX                                                 │
│  Gas Price: 25-50 nAVAX (very low after Octane upgrade)        │
│                                                                  │
│  Key Contracts:                                                  │
│  • USDC: 0xB97EF9Ef8734C71904D8002F8b6Bc66Dd9c48a6E            │
│  • USDC.e: 0xA7D7079b0FEaD91F3e65f86E8915Cb59c1a4C664          │
│                                                                  │
│  Unique Features:                                                │
│  • Subnets for app-specific chains                              │
│  • Instant finality (no reorgs)                                 │
│  • Octane upgrade (90% gas reduction)                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Deployment Priority Roadmap

### Phased Rollout Strategy

```
MULTI-CHAIN DEPLOYMENT ROADMAP
═══════════════════════════════════════════════════════════════════

PHASE 1: FOUNDATION (Months 1-3)
───────────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────────────┐
│  PRIMARY DEPLOYMENT: Polygon PoS                                 │
│                                                                  │
│  Objectives:                                                     │
│  ☐ Deploy core prediction market contracts                      │
│  ☐ Integrate with Chainlink oracles                             │
│  ☐ Set up USDC liquidity pools                                  │
│  ☐ Launch with 10 initial markets                               │
│  ☐ Implement basic gasless transactions (Biconomy)              │
│                                                                  │
│  Success Metrics:                                                │
│  • 10,000 active users                                          │
│  • $1M TVL                                                      │
│  • 99.9% uptime                                                 │
│                                                                  │
│  Team Focus: 100% on Polygon                                    │
└─────────────────────────────────────────────────────────────────┘

PHASE 2: EXPANSION (Months 4-6)
───────────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────────────┐
│  SECONDARY DEPLOYMENT: Base                                      │
│                                                                  │
│  Objectives:                                                     │
│  ☐ Deploy contracts to Base                                     │
│  ☐ Integrate Coinbase onramp                                    │
│  ☐ Set up LayerZero messaging (Polygon ↔ Base)                 │
│  ☐ Cross-chain market synchronization                           │
│  ☐ Unified UI with chain selector                               │
│                                                                  │
│  Success Metrics:                                                │
│  • 5,000 new Base users                                         │
│  • $500K Base TVL                                               │
│  • <5 minute cross-chain sync                                   │
│                                                                  │
│  Team Focus: 60% Polygon, 40% Base                              │
└─────────────────────────────────────────────────────────────────┘

PHASE 3: POWER USERS (Months 7-9)
───────────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────────────┐
│  TERTIARY DEPLOYMENT: Arbitrum One                               │
│                                                                  │
│  Objectives:                                                     │
│  ☐ Deploy contracts to Arbitrum                                 │
│  ☐ Deep DeFi integrations (GMX, Aave)                          │
│  ☐ Institutional-grade features                                 │
│  ☐ Cross-chain liquidity aggregation                            │
│  ☐ Advanced trading features (limit orders, etc.)               │
│                                                                  │
│  Success Metrics:                                                │
│  • Average trade size >$1,000                                   │
│  • $5M Arbitrum TVL                                             │
│  • 50% of volume from Arbitrum                                  │
│                                                                  │
│  Team Focus: 40% Polygon, 30% Base, 30% Arbitrum               │
└─────────────────────────────────────────────────────────────────┘

PHASE 4: OPTIMIZATION (Months 10-12)
───────────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────────────┐
│  EVALUATE & EXPAND                                               │
│                                                                  │
│  Potential Additions:                                            │
│  ☐ Optimism (if Superchain benefits materialize)               │
│  ☐ zkSync Era (if ecosystem matures)                            │
│  ☐ Avalanche (if gaming markets take off)                       │
│                                                                  │
│  Focus Areas:                                                    │
│  ☐ Unified liquidity optimization                               │
│  ☐ Cross-chain arbitrage bots                                   │
│  ☐ Chain-specific marketing                                     │
│  ☐ Performance benchmarking                                     │
│                                                                  │
│  Decision Criteria for New Chains:                              │
│  • TVL > $1B                                                    │
│  • Unique user demographic                                      │
│  • Technical differentiator                                     │
│  • Partnership opportunity                                       │
└─────────────────────────────────────────────────────────────────┘

TIMELINE VISUALIZATION:
═══════════════════════════════════════════════════════════════════

Month:  1   2   3   4   5   6   7   8   9   10  11  12
        │   │   │   │   │   │   │   │   │   │   │   │
Polygon ████████████████████████████████████████████████
                │
Base            └───────████████████████████████████████
                                │
Arbitrum                        └───────████████████████
                                                │
Others                                          └───────
        │               │               │               │
     Phase 1         Phase 2         Phase 3         Phase 4
    Foundation      Expansion      Power Users    Optimization
```

---

## Multi-Chain Architecture Patterns

### Recommended Architecture

```
MULTI-CHAIN PREDICTION MARKET ARCHITECTURE
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                        USER INTERFACE                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    React/Next.js Frontend                  │  │
│  │                                                            │  │
│  │  • Chain abstraction layer                                │  │
│  │  • Unified wallet connection                              │  │
│  │  • Automatic chain switching                              │  │
│  │  • Cross-chain position aggregation                       │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────┬────────────────────────────────┘
                                 │
┌────────────────────────────────▼────────────────────────────────┐
│                      BACKEND SERVICES                            │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐    │
│  │  API Gateway   │  │  Indexer       │  │  Price Oracle  │    │
│  │  (REST/GQL)    │  │  (TheGraph)    │  │  Aggregator    │    │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘    │
└──────────┼───────────────────┼───────────────────┼──────────────┘
           │                   │                   │
           └───────────────────┼───────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                   CROSS-CHAIN LAYER                              │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    LayerZero / Axelar                      │  │
│  │  • Market state synchronization                           │  │
│  │  • Resolution broadcast                                   │  │
│  │  • Liquidity rebalancing signals                         │  │
│  └──────────────────────────────────────────────────────────┘  │
└──────────┬──────────────────────────────────────┬───────────────┘
           │                                      │
┌──────────▼──────────┐              ┌───────────▼────────────────┐
│   PRIMARY CHAIN     │              │    SECONDARY CHAINS         │
│   (Polygon PoS)     │              │                             │
│                     │              │  ┌─────────┐  ┌─────────┐  │
│  ┌───────────────┐  │              │  │  Base   │  │Arbitrum │  │
│  │MarketFactory  │  │◄────────────▶│  │         │  │         │  │
│  │MarketCore     │  │              │  │ Mirror  │  │ Mirror  │  │
│  │LiquidityPool  │  │              │  │Contracts│  │Contracts│  │
│  │Oracle         │  │              │  │         │  │         │  │
│  │Resolver       │  │              │  └─────────┘  └─────────┘  │
│  └───────────────┘  │              │                             │
│                     │              │  (Receive cross-chain      │
│  (Authoritative     │              │   updates, local execution) │
│   source of truth)  │              │                             │
└─────────────────────┘              └─────────────────────────────┘
```

### Contract Deployment Pattern

```typescript
// Multi-chain deployment configuration
interface ChainConfig {
  chainId: number;
  name: string;
  rpcUrl: string;
  contracts: {
    marketFactory: `0x${string}`;
    marketCore: `0x${string}`;
    liquidityPool: `0x${string}`;
    crossChainSync: `0x${string}`;
    paymaster?: `0x${string}`; // Only on chains with AA
  };
  bridges: {
    layerZeroEndpoint: `0x${string}`;
    nativeBridge?: `0x${string}`;
  };
  tokens: {
    usdc: `0x${string}`;
    nativeWrapper: `0x${string}`;
  };
  config: {
    isPrimary: boolean;
    confirmations: number;
    maxGasPrice: bigint;
  };
}

const CHAIN_CONFIGS: Record<number, ChainConfig> = {
  // Polygon PoS (Primary)
  137: {
    chainId: 137,
    name: "Polygon",
    rpcUrl: "https://polygon-rpc.com",
    contracts: {
      marketFactory: "0x...",
      marketCore: "0x...",
      liquidityPool: "0x...",
      crossChainSync: "0x...",
    },
    bridges: {
      layerZeroEndpoint: "0x1a44076050125825900e736c501f859c50fe728c",
    },
    tokens: {
      usdc: "0x3c499c542cef5e3811e1192ce70d8cc03d5c3359",
      nativeWrapper: "0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270",
    },
    config: {
      isPrimary: true,
      confirmations: 64,
      maxGasPrice: 500n * 10n ** 9n, // 500 gwei max
    },
  },
  // Base
  8453: {
    chainId: 8453,
    name: "Base",
    rpcUrl: "https://mainnet.base.org",
    contracts: {
      marketFactory: "0x...",
      marketCore: "0x...",
      liquidityPool: "0x...",
      crossChainSync: "0x...",
    },
    bridges: {
      layerZeroEndpoint: "0x1a44076050125825900e736c501f859c50fe728c",
      nativeBridge: "0x49048044D57e1C92A77f79988d21Fa8fAF74E97e",
    },
    tokens: {
      usdc: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
      nativeWrapper: "0x4200000000000000000000000000000000000006",
    },
    config: {
      isPrimary: false,
      confirmations: 10,
      maxGasPrice: 1n * 10n ** 9n, // 1 gwei max
    },
  },
  // Arbitrum One
  42161: {
    chainId: 42161,
    name: "Arbitrum",
    rpcUrl: "https://arb1.arbitrum.io/rpc",
    contracts: {
      marketFactory: "0x...",
      marketCore: "0x...",
      liquidityPool: "0x...",
      crossChainSync: "0x...",
    },
    bridges: {
      layerZeroEndpoint: "0x1a44076050125825900e736c501f859c50fe728c",
      nativeBridge: "0x72Ce9c846789fdB6fC1f34aC4AD25Dd9ef7031ef",
    },
    tokens: {
      usdc: "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
      nativeWrapper: "0x82aF49447D8a07e3bd95BD0d56f35241523fBab1",
    },
    config: {
      isPrimary: false,
      confirmations: 20,
      maxGasPrice: 5n * 10n ** 9n, // 5 gwei max
    },
  },
};

// Deployment script pattern
async function deployToChain(chainConfig: ChainConfig) {
  const deployer = new ContractDeployer(chainConfig);

  // 1. Deploy core contracts
  const marketFactory = await deployer.deploy("MarketFactory", [
    chainConfig.tokens.usdc,
    chainConfig.config.isPrimary,
  ]);

  const marketCore = await deployer.deploy("MarketCore", [marketFactory.address]);

  const liquidityPool = await deployer.deploy("LiquidityPool", [
    chainConfig.tokens.usdc,
    marketCore.address,
  ]);

  // 2. Deploy cross-chain sync
  const crossChainSync = await deployer.deploy("CrossChainMarketSync", [
    chainConfig.bridges.layerZeroEndpoint,
    marketCore.address,
  ]);

  // 3. Configure cross-chain peers
  if (!chainConfig.config.isPrimary) {
    // Set primary chain as trusted peer
    await crossChainSync.setPeer(
      CHAIN_CONFIGS[137].chainId, // Polygon
      CHAIN_CONFIGS[137].contracts.crossChainSync
    );
  }

  // 4. Initialize contracts
  await marketCore.setLiquidityPool(liquidityPool.address);
  await marketCore.setCrossChainSync(crossChainSync.address);

  console.log(`Deployed to ${chainConfig.name}:`, {
    marketFactory: marketFactory.address,
    marketCore: marketCore.address,
    liquidityPool: liquidityPool.address,
    crossChainSync: crossChainSync.address,
  });
}
```

---

## References

- [Arbitrum Documentation](https://docs.arbitrum.io/)
- [Base Documentation](https://docs.base.org/)
- [Optimism Documentation](https://docs.optimism.io/)
- [Polygon Documentation](https://docs.polygon.technology/)
- [zkSync Documentation](https://docs.zksync.io/)
- [Avalanche Documentation](https://docs.avax.network/)
- [LayerZero V2 Documentation](https://docs.layerzero.network/)
- [Axelar Documentation](https://docs.axelar.dev/)
- [Wormhole Documentation](https://docs.wormhole.com/)
- [L2Beat - L2 Comparison](https://l2beat.com/)
- [DefiLlama - Chain TVL](https://defillama.com/chains)
- [DWF Labs - Interoperability Research](https://www.dwf-labs.com/research)
