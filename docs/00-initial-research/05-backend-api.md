# Backend Architecture & API Documentation

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Core Backend Services](#core-backend-services)
3. [REST API Reference](#rest-api-reference)
4. [WebSocket API](#websocket-api)
5. [Database Schema](#database-schema)
6. [Infrastructure](#infrastructure)
7. [Security](#security)
8. [Error Handling](#error-handling)

---

## Architecture Overview

### Hybrid Model Architecture

Polymarket employs a **hybrid off-chain/on-chain architecture** that balances performance with decentralization:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Web App   │  │ Mobile App  │  │   API SDK   │  │    Bots     │        │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘        │
└─────────┼────────────────┼────────────────┼────────────────┼────────────────┘
          │                │                │                │
          ▼                ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              API GATEWAY                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Load Balancer → Rate Limiter → Auth Middleware → Request Router    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           BACKEND SERVICES                                   │
│                                                                              │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐                   │
│  │  Order Book   │  │    Market     │  │     User      │                   │
│  │   Service     │  │   Service     │  │   Service     │                   │
│  │   (CLOB)      │  │               │  │               │                   │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘                   │
│          │                  │                  │                            │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐                   │
│  │    Oracle     │  │   Indexer     │  │ Notification  │                   │
│  │   Service     │  │   Service     │  │   Service     │                   │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘                   │
│          │                  │                  │                            │
└──────────┼──────────────────┼──────────────────┼────────────────────────────┘
           │                  │                  │
           ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            DATA LAYER                                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  PostgreSQL │  │    Redis    │  │   RabbitMQ  │  │ TimescaleDB │        │
│  │   (Primary) │  │   (Cache)   │  │   (Queue)   │  │ (Analytics) │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          BLOCKCHAIN LAYER                                    │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │                        Polygon Network                                 │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │ │
│  │  │   CTF       │  │   USDC      │  │    UMA      │  │   Gnosis    │  │ │
│  │  │  Exchange   │  │   Token     │  │   Oracle    │  │   Safe      │  │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Off-chain Speed** | Order matching happens off-chain for sub-second execution |
| **On-chain Security** | All settlements are finalized on-chain for trustless guarantees |
| **Horizontal Scaling** | Microservices architecture allows independent scaling |
| **Event-Driven** | Asynchronous processing via message queues |
| **Cache-First** | Redis caching for frequently accessed data |

### Request Flow Example

```
User Places Order
       │
       ▼
┌──────────────────┐
│  API Gateway     │  ← Rate limit check, Auth validation
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Order Service   │  ← Validate order, check balance
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Matching Engine │  ← Match against order book (off-chain)
└────────┬─────────┘
         │
    ┌────┴────┐
    │ Match?  │
    └────┬────┘
         │
    ┌────┴────┐         ┌──────────────────┐
    │   Yes   │────────▶│  Execute Trade   │
    └─────────┘         │  (off-chain)     │
         │              └────────┬─────────┘
         │                       │
    ┌────┴────┐         ┌────────▼─────────┐
    │   No    │         │  Update Balances │
    └────┬────┘         │  (off-chain)     │
         │              └────────┬─────────┘
         ▼                       │
┌──────────────────┐            ▼
│  Add to Book     │    ┌──────────────────┐
│  (Resting Order) │    │  Batch Settlement│
└──────────────────┘    │  (on-chain)      │
                        └──────────────────┘
```

---

## Core Backend Services

### 1. Order Book Service (CLOB)

The Central Limit Order Book (CLOB) is the heart of the trading system.

#### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    ORDER BOOK SERVICE                            │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   MATCHING ENGINE                        │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │   Price     │  │    Time     │  │   Order     │     │   │
│  │  │  Priority   │  │  Priority   │  │   Validator │     │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────────────────────┐  ┌──────────────────────┐            │
│  │      BID BOOK        │  │      ASK BOOK        │            │
│  │  ┌────────────────┐  │  │  ┌────────────────┐  │            │
│  │  │ Price │ Orders │  │  │  │ Price │ Orders │  │            │
│  │  ├───────┼────────┤  │  │  ├───────┼────────┤  │            │
│  │  │ 0.65  │ [...]  │  │  │  │ 0.66  │ [...]  │  │            │
│  │  │ 0.64  │ [...]  │  │  │  │ 0.67  │ [...]  │  │            │
│  │  │ 0.63  │ [...]  │  │  │  │ 0.68  │ [...]  │  │            │
│  │  └────────────────┘  │  │  └────────────────┘  │            │
│  └──────────────────────┘  └──────────────────────┘            │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   ORDER TYPES                            │   │
│  │  • Market Order: Execute immediately at best price       │   │
│  │  • Limit Order: Execute at specified price or better     │   │
│  │  • GTC: Good Till Cancelled                              │   │
│  │  • IOC: Immediate Or Cancel                              │   │
│  │  • FOK: Fill Or Kill                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

#### Order Matching Algorithm

```typescript
interface Order {
  id: string;
  marketId: string;
  userId: string;
  side: 'BUY' | 'SELL';
  type: 'MARKET' | 'LIMIT';
  price: number;        // 0.01 to 0.99 for binary markets
  size: number;         // Amount in shares
  filledSize: number;
  status: OrderStatus;
  timeInForce: 'GTC' | 'IOC' | 'FOK';
  createdAt: Date;
  expiresAt?: Date;
}

enum OrderStatus {
  PENDING = 'PENDING',
  OPEN = 'OPEN',
  PARTIALLY_FILLED = 'PARTIALLY_FILLED',
  FILLED = 'FILLED',
  CANCELLED = 'CANCELLED',
  EXPIRED = 'EXPIRED'
}

class MatchingEngine {
  private bidBook: PriceLevel[] = [];  // Sorted descending by price
  private askBook: PriceLevel[] = [];  // Sorted ascending by price

  matchOrder(incomingOrder: Order): MatchResult {
    const oppositeBook = incomingOrder.side === 'BUY'
      ? this.askBook
      : this.bidBook;

    const trades: Trade[] = [];
    let remainingSize = incomingOrder.size;

    for (const level of oppositeBook) {
      if (remainingSize === 0) break;

      // Price check for limit orders
      if (incomingOrder.type === 'LIMIT') {
        if (incomingOrder.side === 'BUY' && level.price > incomingOrder.price) break;
        if (incomingOrder.side === 'SELL' && level.price < incomingOrder.price) break;
      }

      // Match against orders at this price level (FIFO)
      for (const restingOrder of level.orders) {
        if (remainingSize === 0) break;

        const matchSize = Math.min(remainingSize, restingOrder.remainingSize);

        trades.push({
          takerOrderId: incomingOrder.id,
          makerOrderId: restingOrder.id,
          price: restingOrder.price,  // Maker's price
          size: matchSize,
          timestamp: Date.now()
        });

        remainingSize -= matchSize;
        restingOrder.remainingSize -= matchSize;
      }

      // Remove filled orders from level
      level.orders = level.orders.filter(o => o.remainingSize > 0);
    }

    // Handle remaining size based on order type
    if (remainingSize > 0) {
      if (incomingOrder.timeInForce === 'FOK') {
        return { trades: [], status: 'CANCELLED', reason: 'FOK not fully filled' };
      }
      if (incomingOrder.timeInForce === 'IOC') {
        return { trades, status: 'PARTIALLY_FILLED' };
      }
      if (incomingOrder.type === 'LIMIT') {
        this.addToBook(incomingOrder, remainingSize);
        return { trades, status: 'OPEN' };
      }
    }

    return { trades, status: 'FILLED' };
  }
}
```

#### Order Book Data Structure

```typescript
interface OrderBook {
  marketId: string;
  outcomeId: string;
  bids: PriceLevel[];
  asks: PriceLevel[];
  lastTradePrice: number;
  lastTradeTime: Date;
  spread: number;
  midPrice: number;
}

interface PriceLevel {
  price: number;
  size: number;        // Total size at this level
  orderCount: number;  // Number of orders
  orders: RestingOrder[];
}

interface RestingOrder {
  id: string;
  userId: string;
  size: number;
  remainingSize: number;
  timestamp: Date;
}
```

### 2. Market Service

Handles market lifecycle from creation to resolution.

#### Market Lifecycle

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   DRAFT     │───▶│   PENDING   │───▶│   ACTIVE    │───▶│   CLOSED    │
│             │    │  APPROVAL   │    │             │    │             │
└─────────────┘    └─────────────┘    └──────┬──────┘    └──────┬──────┘
                                             │                   │
                                             ▼                   ▼
                                      ┌─────────────┐    ┌─────────────┐
                                      │   PAUSED    │    │  RESOLVING  │
                                      │             │    │             │
                                      └─────────────┘    └──────┬──────┘
                                                                │
                                             ┌──────────────────┼──────────────────┐
                                             ▼                  ▼                  ▼
                                      ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
                                      │  RESOLVED   │    │  DISPUTED   │    │   VOIDED    │
                                      │   (YES/NO)  │    │             │    │             │
                                      └─────────────┘    └─────────────┘    └─────────────┘
```

#### Market Model

```typescript
interface Market {
  id: string;
  slug: string;
  question: string;
  description: string;
  category: MarketCategory;
  outcomes: Outcome[];

  // Resolution
  resolutionSource: string;
  resolutionCriteria: string;
  resolvedAt?: Date;
  resolvedOutcome?: string;

  // Timing
  createdAt: Date;
  startDate: Date;
  endDate: Date;

  // Status
  status: MarketStatus;

  // Trading
  volume: number;
  liquidity: number;

  // Oracle
  oracleId: string;
  conditionId: string;  // Gnosis CTF condition

  // Metadata
  tags: string[];
  imageUrl: string;
  featured: boolean;
}

interface Outcome {
  id: string;
  marketId: string;
  name: string;           // "Yes" or "No"
  tokenId: string;        // ERC1155 token ID
  price: number;          // Current price 0-1
  volume: number;
}

enum MarketStatus {
  DRAFT = 'DRAFT',
  PENDING_APPROVAL = 'PENDING_APPROVAL',
  ACTIVE = 'ACTIVE',
  PAUSED = 'PAUSED',
  CLOSED = 'CLOSED',
  RESOLVING = 'RESOLVING',
  RESOLVED = 'RESOLVED',
  DISPUTED = 'DISPUTED',
  VOIDED = 'VOIDED'
}

enum MarketCategory {
  POLITICS = 'POLITICS',
  SPORTS = 'SPORTS',
  CRYPTO = 'CRYPTO',
  ENTERTAINMENT = 'ENTERTAINMENT',
  SCIENCE = 'SCIENCE',
  ECONOMICS = 'ECONOMICS',
  WEATHER = 'WEATHER',
  OTHER = 'OTHER'
}
```

### 3. User Service

Manages user accounts, wallets, and KYC verification.

#### User Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      USER SERVICE                                │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   ACCOUNT MANAGER                        │   │
│  │  • Registration     • Profile Management                 │   │
│  │  • Authentication   • Preferences                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   WALLET MANAGER                         │   │
│  │  • Proxy Wallet Deployment (Gnosis Safe)                 │   │
│  │  • External Wallet Linking                               │   │
│  │  • Balance Tracking                                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   KYC/VERIFICATION                       │   │
│  │  • Identity Verification                                 │   │
│  │  • Geo-restrictions                                      │   │
│  │  • Compliance Checks                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

#### User Model

```typescript
interface User {
  id: string;
  email?: string;
  username?: string;

  // Wallet
  walletAddress: string;        // Primary EOA
  proxyWalletAddress?: string;  // Gnosis Safe

  // Verification
  kycStatus: KYCStatus;
  kycLevel: number;
  verifiedAt?: Date;

  // Trading limits based on KYC level
  dailyLimit: number;
  monthlyLimit: number;

  // Preferences
  settings: UserSettings;

  // Stats
  totalVolume: number;
  totalTrades: number;
  pnl: number;

  createdAt: Date;
  lastActiveAt: Date;
}

enum KYCStatus {
  NONE = 'NONE',
  PENDING = 'PENDING',
  BASIC = 'BASIC',
  VERIFIED = 'VERIFIED',
  REJECTED = 'REJECTED'
}

interface UserSettings {
  theme: 'light' | 'dark';
  notifications: NotificationSettings;
  tradingDefaults: TradingDefaults;
}

interface Position {
  id: string;
  userId: string;
  marketId: string;
  outcomeId: string;

  // Position details
  shares: number;
  avgPrice: number;
  currentPrice: number;

  // P&L
  unrealizedPnl: number;
  realizedPnl: number;

  // History
  openedAt: Date;
  lastUpdatedAt: Date;
}
```

### 4. Oracle Integration Service

Handles market resolution through UMA's optimistic oracle.

#### Resolution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    ORACLE INTEGRATION                            │
│                                                                  │
│  ┌─────────────┐                                                │
│  │   Market    │                                                │
│  │   Closes    │                                                │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              RESOLUTION REQUEST                          │   │
│  │  • Submit outcome assertion to UMA                       │   │
│  │  • Bond posted (typically in USDC)                       │   │
│  │  • Challenge period begins (typically 2 hours)           │   │
│  └───────────────────────┬─────────────────────────────────┘   │
│                          │                                       │
│         ┌────────────────┴────────────────┐                     │
│         ▼                                 ▼                     │
│  ┌─────────────┐                   ┌─────────────┐              │
│  │ No Dispute  │                   │  Disputed   │              │
│  │             │                   │             │              │
│  └──────┬──────┘                   └──────┬──────┘              │
│         │                                 │                      │
│         ▼                                 ▼                      │
│  ┌─────────────────┐             ┌─────────────────┐            │
│  │ Auto-resolved   │             │ DVM Vote        │            │
│  │ after challenge │             │ (UMA holders)   │            │
│  │ period          │             │                 │            │
│  └────────┬────────┘             └────────┬────────┘            │
│           │                               │                      │
│           └───────────────┬───────────────┘                     │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                 SETTLEMENT                               │   │
│  │  • Winning outcome tokens redeemable for $1             │   │
│  │  • Losing outcome tokens worth $0                        │   │
│  │  • CTF contract handles redemption                       │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

#### Oracle Service Implementation

```typescript
interface OracleService {
  // Request resolution
  requestResolution(marketId: string, proposedOutcome: string): Promise<ResolutionRequest>;

  // Monitor disputes
  checkDisputeStatus(requestId: string): Promise<DisputeStatus>;

  // Execute settlement
  settleMarket(marketId: string): Promise<SettlementResult>;
}

interface ResolutionRequest {
  id: string;
  marketId: string;
  proposedOutcome: string;
  proposer: string;
  bondAmount: string;
  challengePeriodEnd: Date;
  status: 'PENDING' | 'DISPUTED' | 'SETTLED';
}

interface DisputeStatus {
  isDisputed: boolean;
  disputer?: string;
  disputeReason?: string;
  dvmVoteScheduled?: Date;
}
```

### 5. Indexer Service

Indexes blockchain events for fast querying.

#### Indexed Events

```typescript
// Events indexed from CTF Exchange contract
const indexedEvents = [
  'OrderFilled',
  'OrderCancelled',
  'PositionTransfer',
  'ConditionResolution',
  'PayoutRedemption'
];

interface IndexerConfig {
  startBlock: number;
  contracts: {
    ctfExchange: string;
    conditionalTokens: string;
    umaOracle: string;
  };
  batchSize: number;
  confirmations: number;
}

// Indexed data structure
interface IndexedTrade {
  id: string;
  blockNumber: number;
  transactionHash: string;
  logIndex: number;

  marketId: string;
  makerAddress: string;
  takerAddress: string;
  side: 'BUY' | 'SELL';
  price: number;
  size: number;

  timestamp: Date;
}
```

---

## REST API Reference

### Base URL

```
Production: https://api.polymarket.com/v1
Staging: https://api-staging.polymarket.com/v1
```

### Authentication

All authenticated endpoints require a signature-based authentication:

```typescript
// Request headers for authenticated endpoints
{
  "POLY_ADDRESS": "0x...",           // User's wallet address
  "POLY_SIGNATURE": "0x...",         // EIP-712 signature
  "POLY_TIMESTAMP": "1699123456789", // Unix timestamp (ms)
  "POLY_NONCE": "abc123"             // Unique request nonce
}

// EIP-712 typed data for signing
const typedData = {
  types: {
    EIP712Domain: [
      { name: 'name', type: 'string' },
      { name: 'version', type: 'string' },
      { name: 'chainId', type: 'uint256' }
    ],
    Request: [
      { name: 'method', type: 'string' },
      { name: 'path', type: 'string' },
      { name: 'body', type: 'string' },
      { name: 'timestamp', type: 'uint256' },
      { name: 'nonce', type: 'string' }
    ]
  },
  primaryType: 'Request',
  domain: {
    name: 'Polymarket',
    version: '1',
    chainId: 137
  },
  message: {
    method: 'POST',
    path: '/orders',
    body: JSON.stringify(orderData),
    timestamp: Date.now(),
    nonce: crypto.randomUUID()
  }
};
```

### Markets API

#### List Markets

```http
GET /markets
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | `ACTIVE` | Filter by status: ACTIVE, CLOSED, RESOLVED |
| `category` | string | - | Filter by category |
| `sortBy` | string | `volume` | Sort by: volume, liquidity, endDate, createdAt |
| `order` | string | `desc` | Sort order: asc, desc |
| `limit` | number | 20 | Results per page (max 100) |
| `offset` | number | 0 | Pagination offset |
| `search` | string | - | Search in question and description |

**Response:**

```json
{
  "data": [
    {
      "id": "0x1234...5678",
      "slug": "will-btc-reach-100k-2024",
      "question": "Will Bitcoin reach $100,000 by end of 2024?",
      "description": "This market will resolve to Yes if...",
      "category": "CRYPTO",
      "status": "ACTIVE",
      "outcomes": [
        {
          "id": "yes",
          "name": "Yes",
          "price": 0.45,
          "tokenId": "1234567890"
        },
        {
          "id": "no",
          "name": "No",
          "price": 0.55,
          "tokenId": "1234567891"
        }
      ],
      "volume": 2500000.00,
      "liquidity": 150000.00,
      "startDate": "2024-01-01T00:00:00Z",
      "endDate": "2024-12-31T23:59:59Z",
      "createdAt": "2024-01-01T00:00:00Z",
      "imageUrl": "https://...",
      "tags": ["bitcoin", "crypto", "price"]
    }
  ],
  "pagination": {
    "total": 150,
    "limit": 20,
    "offset": 0,
    "hasMore": true
  }
}
```

#### Get Market Details

```http
GET /markets/:id
```

**Response:**

```json
{
  "id": "0x1234...5678",
  "slug": "will-btc-reach-100k-2024",
  "question": "Will Bitcoin reach $100,000 by end of 2024?",
  "description": "This market will resolve to Yes if Bitcoin (BTC) trades at or above $100,000 USD on any major exchange before December 31, 2024 23:59:59 UTC.",
  "category": "CRYPTO",
  "status": "ACTIVE",
  "outcomes": [
    {
      "id": "yes",
      "name": "Yes",
      "price": 0.45,
      "tokenId": "1234567890",
      "volume": 1125000.00
    },
    {
      "id": "no",
      "name": "No",
      "price": 0.55,
      "tokenId": "1234567891",
      "volume": 1375000.00
    }
  ],
  "resolutionSource": "CoinGecko",
  "resolutionCriteria": "Market resolves YES if BTC/USD price >= $100,000 on CoinGecko at any point before end date.",
  "volume": 2500000.00,
  "volumeHistory": {
    "24h": 125000.00,
    "7d": 450000.00,
    "30d": 1200000.00
  },
  "liquidity": 150000.00,
  "startDate": "2024-01-01T00:00:00Z",
  "endDate": "2024-12-31T23:59:59Z",
  "createdAt": "2024-01-01T00:00:00Z",
  "oracle": {
    "id": "uma-optimistic",
    "conditionId": "0xabcd...efgh"
  },
  "priceHistory": [
    {"timestamp": "2024-01-01T00:00:00Z", "yesPrice": 0.30},
    {"timestamp": "2024-01-15T00:00:00Z", "yesPrice": 0.35},
    {"timestamp": "2024-02-01T00:00:00Z", "yesPrice": 0.45}
  ],
  "stats": {
    "uniqueTraders": 1250,
    "totalTrades": 8500,
    "avgTradeSize": 294.12
  }
}
```

#### Get Order Book

```http
GET /markets/:id/orderbook
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `outcome` | string | required | Outcome ID (yes/no) |
| `depth` | number | 10 | Number of price levels |

**Response:**

```json
{
  "marketId": "0x1234...5678",
  "outcomeId": "yes",
  "timestamp": "2024-02-15T12:00:00Z",
  "bids": [
    {"price": 0.44, "size": 5000.00, "orders": 3},
    {"price": 0.43, "size": 12000.00, "orders": 7},
    {"price": 0.42, "size": 8500.00, "orders": 4},
    {"price": 0.41, "size": 15000.00, "orders": 12},
    {"price": 0.40, "size": 25000.00, "orders": 18}
  ],
  "asks": [
    {"price": 0.46, "size": 4500.00, "orders": 2},
    {"price": 0.47, "size": 10000.00, "orders": 5},
    {"price": 0.48, "size": 7500.00, "orders": 6},
    {"price": 0.49, "size": 20000.00, "orders": 15},
    {"price": 0.50, "size": 30000.00, "orders": 22}
  ],
  "spread": 0.02,
  "midPrice": 0.45,
  "lastTradePrice": 0.45,
  "lastTradeTime": "2024-02-15T11:58:32Z"
}
```

#### Get Market Trades

```http
GET /markets/:id/trades
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `outcome` | string | - | Filter by outcome |
| `limit` | number | 50 | Results per page |
| `before` | string | - | Cursor for pagination |

**Response:**

```json
{
  "data": [
    {
      "id": "trade_123",
      "marketId": "0x1234...5678",
      "outcomeId": "yes",
      "price": 0.45,
      "size": 500.00,
      "side": "BUY",
      "timestamp": "2024-02-15T11:58:32Z",
      "txHash": "0xabcd...efgh"
    }
  ],
  "pagination": {
    "before": "cursor_xyz",
    "hasMore": true
  }
}
```

### Orders API

#### Place Order (Authenticated)

```http
POST /orders
```

**Request Body:**

```json
{
  "marketId": "0x1234...5678",
  "outcomeId": "yes",
  "side": "BUY",
  "type": "LIMIT",
  "price": 0.45,
  "size": 1000.00,
  "timeInForce": "GTC",
  "expiresAt": "2024-02-16T12:00:00Z"
}
```

**Response:**

```json
{
  "id": "order_456",
  "marketId": "0x1234...5678",
  "outcomeId": "yes",
  "side": "BUY",
  "type": "LIMIT",
  "price": 0.45,
  "size": 1000.00,
  "filledSize": 0,
  "remainingSize": 1000.00,
  "status": "OPEN",
  "timeInForce": "GTC",
  "createdAt": "2024-02-15T12:00:00Z",
  "expiresAt": "2024-02-16T12:00:00Z"
}
```

#### Get User Orders (Authenticated)

```http
GET /orders
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `marketId` | string | - | Filter by market |
| `status` | string | - | Filter by status: OPEN, FILLED, CANCELLED |
| `limit` | number | 50 | Results per page |

**Response:**

```json
{
  "data": [
    {
      "id": "order_456",
      "marketId": "0x1234...5678",
      "outcomeId": "yes",
      "side": "BUY",
      "type": "LIMIT",
      "price": 0.45,
      "size": 1000.00,
      "filledSize": 250.00,
      "remainingSize": 750.00,
      "status": "PARTIALLY_FILLED",
      "timeInForce": "GTC",
      "fills": [
        {"price": 0.45, "size": 250.00, "timestamp": "2024-02-15T12:05:00Z"}
      ],
      "createdAt": "2024-02-15T12:00:00Z"
    }
  ],
  "pagination": {
    "total": 25,
    "limit": 50,
    "offset": 0
  }
}
```

#### Cancel Order (Authenticated)

```http
DELETE /orders/:id
```

**Response:**

```json
{
  "id": "order_456",
  "status": "CANCELLED",
  "cancelledAt": "2024-02-15T12:30:00Z"
}
```

#### Cancel All Orders (Authenticated)

```http
DELETE /orders
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `marketId` | string | Cancel only orders for this market |

**Response:**

```json
{
  "cancelledCount": 5,
  "cancelledOrders": ["order_1", "order_2", "order_3", "order_4", "order_5"]
}
```

### User API

#### Get User Profile (Authenticated)

```http
GET /users/me
```

**Response:**

```json
{
  "id": "user_789",
  "walletAddress": "0x1234...5678",
  "proxyWalletAddress": "0xabcd...efgh",
  "username": "trader123",
  "email": "trader@example.com",
  "kycStatus": "VERIFIED",
  "kycLevel": 2,
  "limits": {
    "dailyLimit": 100000.00,
    "monthlyLimit": 1000000.00,
    "dailyUsed": 25000.00,
    "monthlyUsed": 150000.00
  },
  "stats": {
    "totalVolume": 500000.00,
    "totalTrades": 250,
    "pnl": 15000.00,
    "winRate": 0.58
  },
  "settings": {
    "theme": "dark",
    "notifications": {
      "email": true,
      "push": true,
      "priceAlerts": true
    }
  },
  "createdAt": "2024-01-01T00:00:00Z"
}
```

#### Get User Positions (Authenticated)

```http
GET /users/me/positions
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter: OPEN, CLOSED, ALL |
| `marketId` | string | Filter by market |

**Response:**

```json
{
  "data": [
    {
      "id": "pos_123",
      "market": {
        "id": "0x1234...5678",
        "question": "Will Bitcoin reach $100,000 by end of 2024?",
        "status": "ACTIVE"
      },
      "outcome": {
        "id": "yes",
        "name": "Yes",
        "currentPrice": 0.45
      },
      "shares": 2000.00,
      "avgPrice": 0.40,
      "currentValue": 900.00,
      "costBasis": 800.00,
      "unrealizedPnl": 100.00,
      "unrealizedPnlPercent": 0.125,
      "openedAt": "2024-01-15T10:00:00Z"
    }
  ],
  "summary": {
    "totalPositions": 5,
    "totalValue": 5500.00,
    "totalUnrealizedPnl": 450.00
  }
}
```

#### Get Trade History (Authenticated)

```http
GET /users/me/trades
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `marketId` | string | - | Filter by market |
| `startDate` | string | - | ISO date string |
| `endDate` | string | - | ISO date string |
| `limit` | number | 50 | Results per page |

**Response:**

```json
{
  "data": [
    {
      "id": "trade_abc",
      "market": {
        "id": "0x1234...5678",
        "question": "Will Bitcoin reach $100,000 by end of 2024?"
      },
      "outcome": {
        "id": "yes",
        "name": "Yes"
      },
      "side": "BUY",
      "price": 0.40,
      "size": 500.00,
      "total": 200.00,
      "fee": 0.40,
      "orderId": "order_456",
      "txHash": "0x...",
      "timestamp": "2024-01-15T10:00:00Z"
    }
  ],
  "pagination": {
    "total": 250,
    "limit": 50,
    "offset": 0
  }
}
```

#### Get Wallet Balances (Authenticated)

```http
GET /users/me/balances
```

**Response:**

```json
{
  "usdc": {
    "available": 10000.00,
    "locked": 2500.00,
    "total": 12500.00
  },
  "positions": {
    "totalValue": 5500.00,
    "count": 5
  },
  "openOrders": {
    "lockedValue": 2500.00,
    "count": 3
  },
  "portfolio": {
    "totalValue": 18000.00,
    "pnl": 1500.00,
    "pnlPercent": 0.091
  }
}
```

---

## WebSocket API

### Connection

```javascript
const ws = new WebSocket('wss://ws.polymarket.com/v1');

// Authentication (optional, for private channels)
ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'auth',
    address: '0x1234...5678',
    signature: '0x...',
    timestamp: Date.now()
  }));
};
```

### Subscription Channels

#### Market Price Updates

```javascript
// Subscribe
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'market',
  marketId: '0x1234...5678'
}));

// Incoming message
{
  "type": "price_update",
  "marketId": "0x1234...5678",
  "outcomes": {
    "yes": {"price": 0.46, "change24h": 0.02},
    "no": {"price": 0.54, "change24h": -0.02}
  },
  "timestamp": "2024-02-15T12:00:00Z"
}
```

#### Order Book Updates

```javascript
// Subscribe
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'orderbook',
  marketId: '0x1234...5678',
  outcomeId: 'yes'
}));

// Snapshot (initial)
{
  "type": "orderbook_snapshot",
  "marketId": "0x1234...5678",
  "outcomeId": "yes",
  "bids": [...],
  "asks": [...],
  "timestamp": "2024-02-15T12:00:00Z"
}

// Delta update
{
  "type": "orderbook_delta",
  "marketId": "0x1234...5678",
  "outcomeId": "yes",
  "changes": [
    {"side": "bid", "price": 0.45, "size": 1500.00},
    {"side": "ask", "price": 0.46, "size": 0}  // Size 0 = removed
  ],
  "timestamp": "2024-02-15T12:00:01Z"
}
```

#### Trade Feed

```javascript
// Subscribe
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'trades',
  marketId: '0x1234...5678'
}));

// Incoming message
{
  "type": "trade",
  "marketId": "0x1234...5678",
  "outcomeId": "yes",
  "price": 0.45,
  "size": 500.00,
  "side": "BUY",
  "timestamp": "2024-02-15T12:00:00Z"
}
```

#### User Orders (Authenticated)

```javascript
// Subscribe (requires auth)
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'user_orders'
}));

// Order update
{
  "type": "order_update",
  "order": {
    "id": "order_456",
    "status": "PARTIALLY_FILLED",
    "filledSize": 250.00,
    "remainingSize": 750.00
  },
  "timestamp": "2024-02-15T12:00:00Z"
}

// Order fill
{
  "type": "order_fill",
  "orderId": "order_456",
  "fill": {
    "price": 0.45,
    "size": 250.00
  },
  "timestamp": "2024-02-15T12:00:00Z"
}
```

#### Market Resolution

```javascript
// Subscribe
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'resolutions'
}));

// Incoming message
{
  "type": "market_resolved",
  "marketId": "0x1234...5678",
  "outcome": "yes",
  "resolutionDetails": {
    "source": "Official announcement",
    "resolvedAt": "2024-12-31T23:59:59Z"
  }
}
```

### WebSocket Message Types Summary

| Channel | Message Types | Auth Required |
|---------|---------------|---------------|
| `market` | `price_update`, `volume_update` | No |
| `orderbook` | `orderbook_snapshot`, `orderbook_delta` | No |
| `trades` | `trade` | No |
| `user_orders` | `order_update`, `order_fill`, `order_cancelled` | Yes |
| `user_positions` | `position_update` | Yes |
| `resolutions` | `market_resolved`, `dispute_raised` | No |

---

## Database Schema

### Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATABASE SCHEMA                                    │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│     users       │       │    markets      │       │   categories    │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ id (PK)         │       │ id (PK)         │       │ id (PK)         │
│ wallet_address  │       │ slug            │       │ name            │
│ proxy_wallet    │       │ question        │       │ slug            │
│ username        │       │ description     │       │ description     │
│ email           │       │ category_id(FK) │───────│ icon_url        │
│ kyc_status      │       │ status          │       │ created_at      │
│ kyc_level       │       │ volume          │       └─────────────────┘
│ created_at      │       │ liquidity       │
│ updated_at      │       │ start_date      │
└────────┬────────┘       │ end_date        │
         │                │ resolved_at     │
         │                │ condition_id    │
         │                │ created_at      │
         │                │ updated_at      │
         │                └────────┬────────┘
         │                         │
         │                         │
         │    ┌────────────────────┼────────────────────┐
         │    │                    │                    │
         │    ▼                    ▼                    ▼
         │ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
         │ │    outcomes     │ │   market_tags   │ │  price_history  │
         │ ├─────────────────┤ ├─────────────────┤ ├─────────────────┤
         │ │ id (PK)         │ │ market_id (FK)  │ │ id (PK)         │
         │ │ market_id (FK)  │ │ tag_id (FK)     │ │ market_id (FK)  │
         │ │ name            │ └─────────────────┘ │ outcome_id (FK) │
         │ │ token_id        │                     │ price           │
         │ │ price           │                     │ timestamp       │
         │ │ volume          │                     └─────────────────┘
         │ └────────┬────────┘
         │          │
         │          │
┌────────┴──────────┴────────────────────────────────────────────┐
│                                                                 │
▼                    ▼                    ▼                       ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│     orders      │ │     trades      │ │   positions     │ │   order_fills   │
├─────────────────┤ ├─────────────────┤ ├─────────────────┤ ├─────────────────┤
│ id (PK)         │ │ id (PK)         │ │ id (PK)         │ │ id (PK)         │
│ user_id (FK)    │ │ market_id (FK)  │ │ user_id (FK)    │ │ order_id (FK)   │
│ market_id (FK)  │ │ outcome_id (FK) │ │ market_id (FK)  │ │ trade_id (FK)   │
│ outcome_id (FK) │ │ maker_order(FK) │ │ outcome_id (FK) │ │ price           │
│ side            │ │ taker_order(FK) │ │ shares          │ │ size            │
│ type            │ │ price           │ │ avg_price       │ │ timestamp       │
│ price           │ │ size            │ │ cost_basis      │ └─────────────────┘
│ size            │ │ side            │ │ realized_pnl    │
│ filled_size     │ │ tx_hash         │ │ status          │
│ status          │ │ timestamp       │ │ opened_at       │
│ time_in_force   │ └─────────────────┘ │ closed_at       │
│ expires_at      │                     │ updated_at      │
│ created_at      │                     └─────────────────┘
│ updated_at      │
└─────────────────┘
```

### Table Definitions

#### users

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL UNIQUE,
    proxy_wallet_address VARCHAR(42),
    username VARCHAR(50) UNIQUE,
    email VARCHAR(255) UNIQUE,
    kyc_status VARCHAR(20) DEFAULT 'NONE',
    kyc_level INTEGER DEFAULT 0,
    daily_limit DECIMAL(18, 2) DEFAULT 10000.00,
    monthly_limit DECIMAL(18, 2) DEFAULT 100000.00,
    total_volume DECIMAL(18, 2) DEFAULT 0,
    total_trades INTEGER DEFAULT 0,
    pnl DECIMAL(18, 2) DEFAULT 0,
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_active_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_users_wallet ON users(wallet_address);
CREATE INDEX idx_users_kyc_status ON users(kyc_status);
```

#### markets

```sql
CREATE TABLE markets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    slug VARCHAR(255) NOT NULL UNIQUE,
    question TEXT NOT NULL,
    description TEXT,
    category_id UUID REFERENCES categories(id),
    status VARCHAR(20) DEFAULT 'DRAFT',

    -- Trading stats
    volume DECIMAL(18, 2) DEFAULT 0,
    liquidity DECIMAL(18, 2) DEFAULT 0,

    -- Timing
    start_date TIMESTAMP WITH TIME ZONE NOT NULL,
    end_date TIMESTAMP WITH TIME ZONE NOT NULL,

    -- Resolution
    resolution_source TEXT,
    resolution_criteria TEXT,
    resolved_at TIMESTAMP WITH TIME ZONE,
    resolved_outcome_id UUID,

    -- On-chain references
    condition_id VARCHAR(66),
    oracle_id VARCHAR(66),

    -- Metadata
    image_url TEXT,
    featured BOOLEAN DEFAULT FALSE,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_markets_status ON markets(status);
CREATE INDEX idx_markets_category ON markets(category_id);
CREATE INDEX idx_markets_end_date ON markets(end_date);
CREATE INDEX idx_markets_volume ON markets(volume DESC);
CREATE INDEX idx_markets_featured ON markets(featured) WHERE featured = TRUE;
```

#### outcomes

```sql
CREATE TABLE outcomes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    market_id UUID NOT NULL REFERENCES markets(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    token_id VARCHAR(78) NOT NULL,  -- ERC1155 token ID
    price DECIMAL(5, 4) DEFAULT 0.5,
    volume DECIMAL(18, 2) DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    UNIQUE(market_id, name)
);

CREATE INDEX idx_outcomes_market ON outcomes(market_id);
CREATE INDEX idx_outcomes_token ON outcomes(token_id);
```

#### orders

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    market_id UUID NOT NULL REFERENCES markets(id),
    outcome_id UUID NOT NULL REFERENCES outcomes(id),

    side VARCHAR(4) NOT NULL CHECK (side IN ('BUY', 'SELL')),
    type VARCHAR(10) NOT NULL CHECK (type IN ('MARKET', 'LIMIT')),
    price DECIMAL(5, 4),
    size DECIMAL(18, 2) NOT NULL,
    filled_size DECIMAL(18, 2) DEFAULT 0,

    status VARCHAR(20) DEFAULT 'PENDING',
    time_in_force VARCHAR(3) DEFAULT 'GTC',

    expires_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    cancelled_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_market ON orders(market_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_open ON orders(market_id, outcome_id, side, price)
    WHERE status IN ('OPEN', 'PARTIALLY_FILLED');
```

#### trades

```sql
CREATE TABLE trades (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    market_id UUID NOT NULL REFERENCES markets(id),
    outcome_id UUID NOT NULL REFERENCES outcomes(id),

    maker_order_id UUID NOT NULL REFERENCES orders(id),
    taker_order_id UUID NOT NULL REFERENCES orders(id),
    maker_user_id UUID NOT NULL REFERENCES users(id),
    taker_user_id UUID NOT NULL REFERENCES users(id),

    price DECIMAL(5, 4) NOT NULL,
    size DECIMAL(18, 2) NOT NULL,
    side VARCHAR(4) NOT NULL,  -- Taker's side

    maker_fee DECIMAL(18, 6) DEFAULT 0,
    taker_fee DECIMAL(18, 6) DEFAULT 0,

    tx_hash VARCHAR(66),
    block_number BIGINT,

    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_trades_market ON trades(market_id);
CREATE INDEX idx_trades_maker ON trades(maker_user_id);
CREATE INDEX idx_trades_taker ON trades(taker_user_id);
CREATE INDEX idx_trades_timestamp ON trades(timestamp DESC);
```

#### positions

```sql
CREATE TABLE positions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    market_id UUID NOT NULL REFERENCES markets(id),
    outcome_id UUID NOT NULL REFERENCES outcomes(id),

    shares DECIMAL(18, 2) DEFAULT 0,
    avg_price DECIMAL(5, 4) DEFAULT 0,
    cost_basis DECIMAL(18, 2) DEFAULT 0,
    realized_pnl DECIMAL(18, 2) DEFAULT 0,

    status VARCHAR(10) DEFAULT 'OPEN',
    opened_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    closed_at TIMESTAMP WITH TIME ZONE,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    UNIQUE(user_id, market_id, outcome_id)
);

CREATE INDEX idx_positions_user ON positions(user_id);
CREATE INDEX idx_positions_market ON positions(market_id);
CREATE INDEX idx_positions_open ON positions(user_id, status) WHERE status = 'OPEN';
```

#### price_history (TimescaleDB)

```sql
CREATE TABLE price_history (
    id BIGSERIAL,
    market_id UUID NOT NULL,
    outcome_id UUID NOT NULL,
    price DECIMAL(5, 4) NOT NULL,
    volume DECIMAL(18, 2) DEFAULT 0,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,

    PRIMARY KEY (id, timestamp)
);

-- Convert to TimescaleDB hypertable for time-series optimization
SELECT create_hypertable('price_history', 'timestamp');

CREATE INDEX idx_price_history_market ON price_history(market_id, timestamp DESC);
```

---

## Infrastructure

### System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         INFRASTRUCTURE OVERVIEW                              │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌─────────────────┐
                              │   CloudFlare    │
                              │   (CDN + WAF)   │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │  Load Balancer  │
                              │   (AWS ALB)     │
                              └────────┬────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              │                        │                        │
              ▼                        ▼                        ▼
    ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
    │   API Server    │      │   API Server    │      │   API Server    │
    │   (Node.js)     │      │   (Node.js)     │      │   (Node.js)     │
    │   Instance 1    │      │   Instance 2    │      │   Instance N    │
    └────────┬────────┘      └────────┬────────┘      └────────┬────────┘
             │                        │                        │
             └────────────────────────┼────────────────────────┘
                                      │
        ┌─────────────────────────────┼─────────────────────────────┐
        │                             │                             │
        ▼                             ▼                             ▼
┌───────────────┐           ┌─────────────────┐           ┌─────────────────┐
│    Redis      │           │   PostgreSQL    │           │   RabbitMQ      │
│   Cluster     │           │    Primary      │           │    Cluster      │
│               │           │       │         │           │                 │
│  • Session    │           │       ▼         │           │  • Order Queue  │
│  • Cache      │           │   Read Replicas │           │  • Event Queue  │
│  • Rate Limit │           │                 │           │  • Notifications│
└───────────────┘           └─────────────────┘           └─────────────────┘

        ┌─────────────────────────────────────────────────────────────┐
        │                     BACKGROUND SERVICES                      │
        │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │
        │  │   Matching    │  │   Indexer     │  │   Oracle      │   │
        │  │   Engine      │  │   Service     │  │   Monitor     │   │
        │  └───────────────┘  └───────────────┘  └───────────────┘   │
        │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │
        │  │  Settlement   │  │  Analytics    │  │  Notification │   │
        │  │   Service     │  │   Worker      │  │   Service     │   │
        │  └───────────────┘  └───────────────┘  └───────────────┘   │
        └─────────────────────────────────────────────────────────────┘

        ┌─────────────────────────────────────────────────────────────┐
        │                      MONITORING                              │
        │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │
        │  │  Prometheus   │  │    Grafana    │  │   PagerDuty   │   │
        │  │   (Metrics)   │  │  (Dashboards) │  │   (Alerts)    │   │
        │  └───────────────┘  └───────────────┘  └───────────────┘   │
        │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │
        │  │   ELK Stack   │  │    Jaeger     │  │   Sentry      │   │
        │  │   (Logging)   │  │  (Tracing)    │  │   (Errors)    │   │
        │  └───────────────┘  └───────────────┘  └───────────────┘   │
        └─────────────────────────────────────────────────────────────┘
```

### Caching Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                      CACHING LAYERS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LAYER 1: Application Cache (In-Memory)                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • Hot market data                                       │   │
│  │  • User session data                                     │   │
│  │  • TTL: 1-5 seconds                                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  LAYER 2: Distributed Cache (Redis)                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • Order book snapshots (TTL: 100ms)                     │   │
│  │  • Market summaries (TTL: 1 minute)                      │   │
│  │  • User balances (TTL: 5 seconds)                        │   │
│  │  • Rate limit counters                                   │   │
│  │  • Session tokens                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  LAYER 3: CDN Cache (CloudFlare)                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • Static assets (TTL: 1 year)                          │   │
│  │  • Market list (TTL: 30 seconds)                        │   │
│  │  • Public market data (TTL: 10 seconds)                 │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Redis Data Structures

```typescript
// Order Book Cache
interface OrderBookCache {
  key: `orderbook:${marketId}:${outcomeId}`;
  type: 'sorted_set';
  // Bids: score = -price (negative for descending order)
  // Asks: score = price
}

// Rate Limiting
interface RateLimitCache {
  key: `ratelimit:${userId}:${endpoint}`;
  type: 'string';
  value: number;  // Request count
  ttl: 60;        // Seconds
}

// Session Cache
interface SessionCache {
  key: `session:${sessionId}`;
  type: 'hash';
  fields: {
    userId: string;
    walletAddress: string;
    expiresAt: number;
  };
}

// Market Price Cache
interface MarketPriceCache {
  key: `price:${marketId}`;
  type: 'hash';
  fields: {
    yes: string;  // Price as string
    no: string;
    updatedAt: string;
  };
}
```

### Message Queue Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│                    MESSAGE QUEUE TOPOLOGY                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  EXCHANGE: orders                                                │
│  Type: Direct                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                          │   │
│  │  ──▶ Queue: order.new ──▶ Matching Engine               │   │
│  │  ──▶ Queue: order.cancel ──▶ Order Service              │   │
│  │  ──▶ Queue: order.fill ──▶ Settlement Service           │   │
│  │                                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  EXCHANGE: trades                                                │
│  Type: Fanout                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                          │   │
│  │  ──▶ Queue: trade.indexer ──▶ Indexer Service           │   │
│  │  ──▶ Queue: trade.analytics ──▶ Analytics Worker        │   │
│  │  ──▶ Queue: trade.notify ──▶ Notification Service       │   │
│  │                                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  EXCHANGE: events                                                │
│  Type: Topic                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                          │   │
│  │  market.* ──▶ Queue: market.events                      │   │
│  │  user.* ──▶ Queue: user.events                          │   │
│  │  oracle.* ──▶ Queue: oracle.events                      │   │
│  │                                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Security

### Authentication Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    AUTHENTICATION FLOW                           │
└─────────────────────────────────────────────────────────────────┘

┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  User   │     │ Frontend│     │   API   │     │  Redis  │
└────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘
     │               │               │               │
     │  Connect      │               │               │
     │  Wallet       │               │               │
     │──────────────▶│               │               │
     │               │               │               │
     │               │  Get Nonce    │               │
     │               │──────────────▶│               │
     │               │               │               │
     │               │               │  Store Nonce  │
     │               │               │──────────────▶│
     │               │               │               │
     │               │  Nonce        │               │
     │               │◀──────────────│               │
     │               │               │               │
     │  Sign Message │               │               │
     │  (EIP-712)    │               │               │
     │◀──────────────│               │               │
     │               │               │               │
     │  Signature    │               │               │
     │──────────────▶│               │               │
     │               │               │               │
     │               │  Verify       │               │
     │               │  Signature    │               │
     │               │──────────────▶│               │
     │               │               │               │
     │               │               │  Validate     │
     │               │               │  Nonce        │
     │               │               │──────────────▶│
     │               │               │               │
     │               │               │  Create       │
     │               │               │  Session      │
     │               │               │──────────────▶│
     │               │               │               │
     │               │  Session      │               │
     │               │  Token        │               │
     │               │◀──────────────│               │
     │               │               │               │
     │  Logged In    │               │               │
     │◀──────────────│               │               │
     │               │               │               │
```

### Rate Limiting Configuration

```typescript
interface RateLimitConfig {
  // Global limits
  global: {
    windowMs: 60000,           // 1 minute
    max: 1000,                 // requests per window
  },

  // Per-endpoint limits
  endpoints: {
    // Public endpoints
    'GET /markets': { windowMs: 60000, max: 100 },
    'GET /markets/:id': { windowMs: 60000, max: 200 },
    'GET /orderbook': { windowMs: 1000, max: 10 },

    // Authenticated endpoints
    'POST /orders': { windowMs: 1000, max: 5 },
    'DELETE /orders': { windowMs: 1000, max: 10 },
    'GET /users/me': { windowMs: 60000, max: 60 },

    // WebSocket
    'ws:subscribe': { windowMs: 60000, max: 100 },
    'ws:message': { windowMs: 1000, max: 50 },
  },

  // User-based limits (by KYC level)
  userLimits: {
    NONE: { multiplier: 1.0 },
    BASIC: { multiplier: 2.0 },
    VERIFIED: { multiplier: 5.0 },
  }
}
```

### Request Validation

```typescript
// Order validation schema
const orderSchema = z.object({
  marketId: z.string().uuid(),
  outcomeId: z.string(),
  side: z.enum(['BUY', 'SELL']),
  type: z.enum(['MARKET', 'LIMIT']),
  price: z.number()
    .min(0.01)
    .max(0.99)
    .optional()
    .refine((val, ctx) => {
      if (ctx.parent.type === 'LIMIT' && val === undefined) {
        return false;
      }
      return true;
    }, { message: 'Price required for limit orders' }),
  size: z.number()
    .positive()
    .max(1000000),
  timeInForce: z.enum(['GTC', 'IOC', 'FOK']).default('GTC'),
  expiresAt: z.string().datetime().optional(),
});

// Signature verification
async function verifySignature(
  address: string,
  signature: string,
  message: TypedData,
  timestamp: number
): Promise<boolean> {
  // Check timestamp freshness (5 minute window)
  const now = Date.now();
  if (Math.abs(now - timestamp) > 300000) {
    throw new AuthError('Signature expired');
  }

  // Recover signer address
  const recoveredAddress = ethers.verifyTypedData(
    message.domain,
    message.types,
    message.value,
    signature
  );

  return recoveredAddress.toLowerCase() === address.toLowerCase();
}
```

### Security Headers

```typescript
// Security middleware configuration
const securityHeaders = {
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'X-XSS-Protection': '1; mode=block',
  'Content-Security-Policy': "default-src 'self'; script-src 'self'",
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Permissions-Policy': 'geolocation=(), microphone=(), camera=()',
};
```

---

## Error Handling

### Error Response Format

```typescript
interface ErrorResponse {
  error: {
    code: string;           // Machine-readable error code
    message: string;        // Human-readable message
    details?: object;       // Additional context
    requestId: string;      // For support/debugging
  };
}
```

### Error Codes

| HTTP Status | Error Code | Description |
|-------------|------------|-------------|
| 400 | `INVALID_REQUEST` | Request validation failed |
| 400 | `INVALID_ORDER` | Order parameters invalid |
| 400 | `INSUFFICIENT_BALANCE` | Not enough funds |
| 401 | `UNAUTHORIZED` | Authentication required |
| 401 | `INVALID_SIGNATURE` | Signature verification failed |
| 401 | `SESSION_EXPIRED` | Session token expired |
| 403 | `FORBIDDEN` | Action not permitted |
| 403 | `KYC_REQUIRED` | Higher KYC level needed |
| 403 | `GEO_RESTRICTED` | Location not supported |
| 404 | `NOT_FOUND` | Resource not found |
| 404 | `MARKET_NOT_FOUND` | Market doesn't exist |
| 404 | `ORDER_NOT_FOUND` | Order doesn't exist |
| 409 | `ORDER_ALREADY_CANCELLED` | Order was already cancelled |
| 409 | `MARKET_CLOSED` | Market not accepting orders |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Server error |
| 503 | `SERVICE_UNAVAILABLE` | Maintenance mode |

### Example Error Responses

```json
// 400 Bad Request - Invalid Order
{
  "error": {
    "code": "INVALID_ORDER",
    "message": "Order price must be between 0.01 and 0.99",
    "details": {
      "field": "price",
      "value": 1.5,
      "constraint": "0.01 <= price <= 0.99"
    },
    "requestId": "req_abc123"
  }
}

// 401 Unauthorized - Invalid Signature
{
  "error": {
    "code": "INVALID_SIGNATURE",
    "message": "Request signature verification failed",
    "details": {
      "expectedAddress": "0x1234...5678",
      "recoveredAddress": "0xabcd...efgh"
    },
    "requestId": "req_def456"
  }
}

// 429 Too Many Requests
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Rate limit exceeded. Try again in 30 seconds.",
    "details": {
      "limit": 10,
      "window": "1s",
      "retryAfter": 30
    },
    "requestId": "req_ghi789"
  }
}
```

### Retry Strategy

```typescript
interface RetryConfig {
  // Errors that should be retried
  retryableErrors: [
    'RATE_LIMITED',
    'SERVICE_UNAVAILABLE',
    'INTERNAL_ERROR'
  ];

  // Retry configuration
  maxRetries: 3;
  baseDelayMs: 1000;
  maxDelayMs: 30000;

  // Exponential backoff with jitter
  getDelay(attempt: number): number {
    const delay = Math.min(
      this.baseDelayMs * Math.pow(2, attempt),
      this.maxDelayMs
    );
    // Add jitter (0-25% of delay)
    return delay + Math.random() * delay * 0.25;
  }
}
```

---

## API SDK Example

### JavaScript/TypeScript SDK

```typescript
import { PolymarketClient } from '@polymarket/sdk';

// Initialize client
const client = new PolymarketClient({
  apiUrl: 'https://api.polymarket.com/v1',
  wsUrl: 'wss://ws.polymarket.com/v1',
});

// Authenticate with wallet
await client.authenticate(wallet);

// Get markets
const markets = await client.markets.list({
  status: 'ACTIVE',
  category: 'CRYPTO',
  limit: 20
});

// Get order book
const orderbook = await client.markets.getOrderBook(
  marketId,
  'yes',
  { depth: 10 }
);

// Place limit order
const order = await client.orders.create({
  marketId: '0x1234...5678',
  outcomeId: 'yes',
  side: 'BUY',
  type: 'LIMIT',
  price: 0.45,
  size: 100
});

// Subscribe to price updates
client.ws.subscribe('market', marketId, (update) => {
  console.log('Price update:', update);
});

// Get positions
const positions = await client.users.getPositions({
  status: 'OPEN'
});

// Cancel order
await client.orders.cancel(orderId);
```

---

## Appendix

### Environment Variables

```bash
# API Configuration
API_PORT=3000
API_HOST=0.0.0.0
NODE_ENV=production

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/polymarket
DATABASE_POOL_SIZE=20

# Redis
REDIS_URL=redis://localhost:6379
REDIS_CLUSTER_ENABLED=true

# RabbitMQ
RABBITMQ_URL=amqp://localhost:5672

# Blockchain
POLYGON_RPC_URL=https://polygon-rpc.com
CTF_EXCHANGE_ADDRESS=0x...
CONDITIONAL_TOKENS_ADDRESS=0x...

# Security
JWT_SECRET=xxx
SIGNATURE_EXPIRY_MS=300000

# Rate Limiting
RATE_LIMIT_ENABLED=true
RATE_LIMIT_REDIS_PREFIX=rl:

# Monitoring
PROMETHEUS_ENABLED=true
SENTRY_DSN=https://xxx@sentry.io/xxx
```

### Health Check Endpoint

```http
GET /health
```

```json
{
  "status": "healthy",
  "version": "1.5.0",
  "timestamp": "2024-02-15T12:00:00Z",
  "services": {
    "database": "healthy",
    "redis": "healthy",
    "rabbitmq": "healthy",
    "blockchain": "healthy"
  },
  "metrics": {
    "uptime": 86400,
    "requestsPerSecond": 150,
    "activeConnections": 5000
  }
}
```

---

*Last updated: 2024*
