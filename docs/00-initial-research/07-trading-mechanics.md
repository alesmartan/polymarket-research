# Trading & Market Mechanics

This document provides comprehensive documentation for understanding and implementing trading mechanics in a Polymarket-like prediction market platform.

---

## Table of Contents

1. [How Prediction Markets Work](#how-prediction-markets-work)
2. [Trading Mechanics](#trading-mechanics)
3. [Order Book Implementation](#order-book-implementation)
4. [Position Management](#position-management)
5. [Market Lifecycle](#market-lifecycle)
6. [Market Types](#market-types)
7. [Liquidity Mechanics](#liquidity-mechanics)
8. [Fee Structure](#fee-structure)
9. [Mathematical Formulas](#mathematical-formulas)
10. [Risk Management](#risk-management)

---

## How Prediction Markets Work

### Basic Concept

Prediction markets are exchange-traded markets where participants buy and sell shares representing the likelihood of future events. Unlike traditional betting platforms, prediction markets harness collective intelligence to discover probabilities.

**Core Principles:**

| Concept | Description |
|---------|-------------|
| Share Types | Users trade "Yes" or "No" shares based on event outcomes |
| Price Range | Share prices range from $0.01 to $1.00 |
| Price = Probability | A share price of $0.70 implies a 70% probability |
| Correct Prediction | Winning shares are redeemed for $1.00 |
| Incorrect Prediction | Losing shares become worthless ($0.00) |

**Example Scenario:**

```
Market: "Will Team A win the championship?"

Current Prices:
  - Yes shares: $0.65
  - No shares: $0.35

Interpretation:
  - The market implies a 65% probability of Team A winning
  - Buying 100 Yes shares costs $65.00
  - If Team A wins: You receive $100.00 (profit: $35.00)
  - If Team A loses: You receive $0.00 (loss: $65.00)
```

### Market-Driven Pricing

Unlike traditional bookmakers who set odds, prediction market prices emerge organically from trading activity:

```
Traditional Betting          Prediction Markets
      │                            │
      ▼                            ▼
┌─────────────┐            ┌─────────────────┐
│  Bookmaker  │            │   Collective    │
│  Sets Odds  │            │ Trading Activity│
└─────────────┘            └─────────────────┘
      │                            │
      ▼                            ▼
  Fixed Prices              Dynamic Prices
  (+ margin)                (market-driven)
```

**Price Discovery Mechanism:**

1. **Demand Drives Prices**: More buyers for "Yes" → Yes price rises
2. **Supply-Demand Balance**: Prices adjust until buyers and sellers match
3. **Information Aggregation**: Prices reflect collective market knowledge
4. **Continuous Updates**: Real-time price adjustments as new information emerges

**Wisdom of the Crowd:**

The fundamental principle is that aggregated information from many independent participants often produces more accurate predictions than individual experts.

---

## Trading Mechanics

### Order Types

#### 1. Market Orders

Execute immediately at the best available price in the order book.

```typescript
interface MarketOrder {
  side: 'BUY' | 'SELL';
  outcome: 'YES' | 'NO';
  size: number;           // Number of shares
  marketId: string;
}
```

**Characteristics:**
- Guaranteed execution (if liquidity exists)
- Price is not guaranteed
- May experience slippage on large orders
- Best for urgent trades or highly liquid markets

**Execution Example:**

```
Order Book (Yes Shares):
Ask Side: $0.72 (50 shares), $0.73 (100 shares), $0.74 (200 shares)

Market Buy Order: 120 shares

Execution:
  - 50 shares @ $0.72 = $36.00
  - 70 shares @ $0.73 = $51.10
  - Total: 120 shares for $87.10
  - Average price: $0.726
```

#### 2. Limit Orders

Execute only at a specified price or better.

```typescript
interface LimitOrder {
  side: 'BUY' | 'SELL';
  outcome: 'YES' | 'NO';
  size: number;           // Number of shares
  price: number;          // Limit price ($0.01 - $0.99)
  marketId: string;
  timeInForce: 'GTC' | 'GTD' | 'FOK' | 'IOC';
}
```

**Time in Force Options:**

| Type | Name | Description |
|------|------|-------------|
| GTC | Good Till Cancelled | Remains active until filled or cancelled |
| GTD | Good Till Date | Expires at specified date/time |
| FOK | Fill or Kill | Must fill entirely immediately or cancel |
| IOC | Immediate or Cancel | Fill what's possible immediately, cancel rest |

**Characteristics:**
- Price guaranteed (or better)
- Execution not guaranteed
- May be partially filled
- Best for price-sensitive traders

### Order Matching Priority

Orders are matched using **price-time priority**:

1. **Price Priority**: Better prices execute first
2. **Time Priority**: Earlier orders at same price execute first

```
Order Book State at T=0:
Bids: [A] $0.65 (100) @ T=1, [B] $0.65 (50) @ T=3, [C] $0.64 (200) @ T=2

Incoming Sell Order: 120 shares @ Market

Matching Sequence:
1. Fill 100 shares from [A] @ $0.65 (highest bid, earliest time)
2. Fill 20 shares from [B] @ $0.65 (same price, next earliest)
Result: 120 shares sold, [B] has 30 shares remaining
```

---

## Order Book Implementation

### Central Limit Order Book (CLOB) Structure

```
                    ORDER BOOK

     BID SIDE                    ASK SIDE
  (Buy Orders)                 (Sell Orders)

Price   Size                   Price   Size
─────────────                  ─────────────
$0.68   150  ◄── Best Bid      $0.71   80   ◄── Best Ask
$0.67   200                    $0.72   150
$0.66   100                    $0.73   300
$0.65   500                    $0.74   200
$0.64   250                    $0.75   100

        └──────── SPREAD ────────┘
              ($0.71 - $0.68 = $0.03)
```

### Key Order Book Metrics

```typescript
interface OrderBookMetrics {
  bestBid: number;          // Highest buy price
  bestAsk: number;          // Lowest sell price
  spread: number;           // bestAsk - bestBid
  spreadPercentage: number; // spread / midPrice * 100
  midPrice: number;         // (bestBid + bestAsk) / 2

  // Depth analysis
  bidDepth: Map<number, number>;  // Price -> Total volume
  askDepth: Map<number, number>;  // Price -> Total volume
  totalBidVolume: number;
  totalAskVolume: number;

  // Imbalance
  bookImbalance: number;    // (bidVolume - askVolume) / (bidVolume + askVolume)
}
```

### Spread Analysis

The spread indicates market liquidity and trading costs:

```
Tight Spread ($0.01):          Wide Spread ($0.10):
├─ High liquidity              ├─ Low liquidity
├─ Low transaction cost        ├─ High transaction cost
├─ Active market making        ├─ Limited market making
└─ Easy to enter/exit          └─ Difficult to trade

Formula:
  Spread = Best Ask - Best Bid
  Spread % = (Spread / Mid Price) × 100
  Mid Price = (Best Bid + Best Ask) / 2
```

### Order Book Data Structure

```typescript
class OrderBook {
  private bids: SortedMap<Price, PriceLevel>;  // Descending order
  private asks: SortedMap<Price, PriceLevel>;  // Ascending order

  interface PriceLevel {
    price: number;
    orders: Order[];      // FIFO queue
    totalSize: number;
  }

  interface Order {
    id: string;
    userId: string;
    size: number;
    remainingSize: number;
    timestamp: number;
    side: 'BUY' | 'SELL';
  }
}
```

---

## Position Management

### Position Types

```
LONG YES Position:
├─ Bought Yes shares
├─ Profit if event HAPPENS
├─ Maximum profit: (1 - entry_price) × shares
└─ Maximum loss: entry_price × shares

LONG NO Position:
├─ Bought No shares
├─ Profit if event DOES NOT happen
├─ Maximum profit: (1 - entry_price) × shares
└─ Maximum loss: entry_price × shares
```

### Position Tracking

```typescript
interface Position {
  marketId: string;
  outcome: 'YES' | 'NO';
  size: number;              // Total shares held
  averageEntryPrice: number; // Volume-weighted average
  currentValue: number;      // size × currentPrice
  unrealizedPnL: number;     // currentValue - (size × averageEntryPrice)
  realizedPnL: number;       // From closed portions
}

interface PositionUpdate {
  // When adding to position
  newAveragePrice = (oldSize × oldAvgPrice + newSize × newPrice) / (oldSize + newSize);

  // When reducing position
  realizedPnL += (exitPrice - averageEntryPrice) × soldSize;
}
```

### P&L Calculations

```
Profit/Loss Before Resolution:
──────────────────────────────
Entry: Buy 100 Yes shares @ $0.60 = $60.00
Exit:  Sell 100 Yes shares @ $0.75 = $75.00
P&L:   $75.00 - $60.00 = $15.00 profit (25% return)


Profit/Loss After Resolution:
─────────────────────────────
Entry: Buy 100 Yes shares @ $0.60 = $60.00

If Yes wins:
  Payout: 100 × $1.00 = $100.00
  P&L:    $100.00 - $60.00 = $40.00 profit (66.7% return)

If No wins:
  Payout: 100 × $0.00 = $0.00
  P&L:    $0.00 - $60.00 = -$60.00 loss (100% loss)
```

### Exiting Positions Before Resolution

Traders can exit positions at any time by selling their shares:

```typescript
// Method 1: Direct sale
sellOrder = {
  side: 'SELL',
  outcome: 'YES',
  size: positionSize,
  price: limitPrice  // or market order
};

// Method 2: Offsetting position
// Buy equivalent No shares to lock in value
// Yes + No at same quantity = guaranteed $1.00 redemption
```

**Complete Shares Mechanism:**

```
If you hold:
  100 Yes shares + 100 No shares for same market

You can redeem for:
  100 × $1.00 = $100.00 (regardless of outcome)

This arbitrage opportunity keeps Yes + No prices ≈ $1.00
```

---

## Market Lifecycle

### Overview

```
┌─────────────┐    ┌────────────────┐    ┌────────────┐    ┌────────────┐
│  CREATION   │───►│ ACTIVE TRADING │───►│ RESOLUTION │───►│ SETTLEMENT │
└─────────────┘    └────────────────┘    └────────────┘    └────────────┘
     │                    │                    │                 │
     ▼                    ▼                    ▼                 ▼
  Define event      Open trading         Determine          Distribute
  Set criteria      Price discovery      outcome            payouts
  Add liquidity     Volume builds        Oracle verify      Close market
```

### Phase 1: Market Creation

```typescript
interface MarketCreation {
  // Required fields
  question: string;              // Clear, unambiguous question
  description: string;           // Detailed context
  resolutionCriteria: string;    // Specific conditions for resolution
  endDate: Date;                 // When trading closes
  resolutionDate: Date;          // When outcome is determined

  // Market configuration
  type: 'BINARY' | 'MULTI_OUTCOME';
  outcomes: string[];            // ['Yes', 'No'] or multiple options
  category: string;
  tags: string[];

  // Liquidity
  initialLiquidity: {
    amount: number;
    distribution: number[];      // Initial probability distribution
  };

  // Resolution
  resolutionSource: string;      // Official source for outcome
  oracle: string;                // Oracle/resolver address
}
```

**Best Practices for Market Creation:**

1. **Clear Question**: Unambiguous, single interpretation
2. **Specific Criteria**: Exactly how/when market resolves
3. **Verifiable Source**: Publicly accessible resolution source
4. **Reasonable Timeline**: Adequate time for trading and resolution

### Phase 2: Active Trading

```typescript
interface ActiveTradingPhase {
  status: 'ACTIVE';

  // Trading enabled
  ordersAccepted: true;
  orderTypes: ['MARKET', 'LIMIT'];

  // Market data
  currentPrices: { yes: number; no: number };
  volume24h: number;
  totalVolume: number;
  openInterest: number;

  // Price bounds
  minPrice: 0.01;
  maxPrice: 0.99;
  tickSize: 0.01;
}
```

**Price Discovery Process:**

```
Time    Event                           Yes Price   No Price
────────────────────────────────────────────────────────────
T+0     Market opens                    $0.50       $0.50
T+1     News favors outcome             $0.55       $0.45
T+2     Large buy order (Yes)           $0.62       $0.38
T+3     Counter-information emerges     $0.58       $0.42
T+4     Expert analysis published       $0.65       $0.35
...     Continuous price discovery      ...         ...
```

### Phase 3: Resolution

```typescript
interface ResolutionPhase {
  status: 'RESOLVING';

  // Trading halted
  ordersAccepted: false;

  // Resolution process
  proposedOutcome: 'YES' | 'NO' | null;
  proposer: string;
  proposalTime: Date;
  challengePeriod: Duration;      // Time for disputes

  // Dispute mechanism
  disputes: Dispute[];
  disputeBond: number;            // Required stake to dispute
}

interface Resolution {
  outcome: 'YES' | 'NO';
  resolvedAt: Date;
  resolvedBy: string;             // Oracle address
  resolutionSource: string;       // Evidence/proof
  finalPrices: {
    yes: 1.00 | 0.00;
    no: 0.00 | 1.00;
  };
}
```

**Oracle Resolution Flow:**

```
                    ┌─────────────┐
                    │   Oracle    │
                    │  Proposes   │
                    │   Outcome   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
              ┌─────│  Challenge  │─────┐
              │     │   Period    │     │
              │     └─────────────┘     │
              │                         │
       No Dispute                  Dispute Filed
              │                         │
              ▼                         ▼
     ┌─────────────┐           ┌─────────────┐
     │   Resolve   │           │  Arbitration│
     │  as Proposed│           │   Process   │
     └─────────────┘           └──────┬──────┘
                                      │
                               ┌──────▼──────┐
                               │   Final     │
                               │  Resolution │
                               └─────────────┘
```

### Phase 4: Settlement

```typescript
interface SettlementPhase {
  status: 'SETTLED';

  // Final state
  winningOutcome: 'YES' | 'NO';

  // Payout processing
  payouts: {
    totalWinners: number;
    totalPayout: number;
    averagePayout: number;
  };

  // Automatic actions
  positionsLiquidated: true;
  fundsDistributed: true;
  marketClosed: true;
}
```

**Settlement Calculation:**

```
For each winning position:
  Payout = Position Size × $1.00

For each losing position:
  Payout = Position Size × $0.00

Example:
  User A: 500 Yes shares → Outcome: Yes
  Payout: 500 × $1.00 = $500.00

  User B: 300 No shares → Outcome: Yes
  Payout: 300 × $0.00 = $0.00
```

---

## Market Types

### Binary Markets

The simplest prediction market with two mutually exclusive outcomes.

```
Structure:
┌─────────────────────────────────────┐
│     "Will X happen by Y date?"      │
├──────────────────┬──────────────────┤
│       YES        │        NO        │
│    Price: $0.65  │   Price: $0.35   │
└──────────────────┴──────────────────┘

Constraint: P(Yes) + P(No) = 1.00 (approximately)
```

**Examples:**
- "Will Bitcoin reach $100k by December 2024?"
- "Will Company X announce layoffs this quarter?"
- "Will the Fed raise interest rates?"

```typescript
interface BinaryMarket {
  type: 'BINARY';
  outcomes: ['YES', 'NO'];

  // Pricing constraint
  // yesPrice + noPrice ≈ 1.00 (accounting for spread)

  // Resolution
  possibleResults: ['YES', 'NO'];
}
```

### Multi-Outcome Markets

Markets with more than two possible outcomes.

```
Structure:
┌───────────────────────────────────────────────┐
│        "Who will win the election?"           │
├───────────┬───────────┬───────────┬───────────┤
│ Candidate │ Candidate │ Candidate │   Other   │
│     A     │     B     │     C     │           │
│   $0.45   │   $0.35   │   $0.15   │   $0.05   │
└───────────┴───────────┴───────────┴───────────┘

Constraint: Σ P(outcome_i) = 1.00
```

**NegRisk Framework:**

For multi-outcome markets, Polymarket uses the NegRisk framework:

```typescript
interface NegRiskMarket {
  type: 'MULTI_OUTCOME';

  // Each outcome is a separate binary market
  outcomes: {
    id: string;
    name: string;
    yesPrice: number;
    noPrice: number;
  }[];

  // Constraint enforcement
  // Sum of all Yes prices should equal approximately $1.00
  impliedProbabilitySum: number;  // Target: 1.00

  // Resolution
  // Exactly one outcome resolves to Yes ($1.00)
  // All others resolve to No ($0.00)
}
```

**Trading Multi-Outcome Markets:**

```
To bet on Candidate A winning:
  → Buy "Candidate A - Yes" shares

To bet against Candidate A:
  → Buy "Candidate A - No" shares
  OR
  → Buy Yes shares of other candidates

Arbitrage opportunity if sum of Yes prices ≠ $1.00:
  If Σ(Yes prices) > $1.00: Sell all Yes, profit = Σ - $1.00
  If Σ(Yes prices) < $1.00: Buy all Yes, profit = $1.00 - Σ
```

### Conditional Markets

Markets that depend on another market's outcome.

```
Primary Market: "Will Candidate A win nomination?"
Conditional Market: "If A wins nomination, will A win general election?"

The conditional market only activates if the condition is met.
```

---

## Liquidity Mechanics

### Understanding Liquidity

```
High Liquidity:                    Low Liquidity:
├─ Tight spreads                   ├─ Wide spreads
├─ Deep order book                 ├─ Thin order book
├─ Low price impact                ├─ High price impact
├─ Easy to trade large sizes       ├─ Difficult to trade
└─ More accurate prices            └─ Less reliable prices
```

### Market Making

Market makers provide liquidity by continuously quoting both bid and ask prices.

```typescript
interface MarketMakerStrategy {
  // Quote management
  bidPrice: number;      // Price to buy
  askPrice: number;      // Price to sell
  spread: number;        // askPrice - bidPrice
  size: number;          // Quantity on each side

  // Risk parameters
  inventoryLimit: number;     // Maximum position size
  skewFactor: number;         // Adjust quotes based on inventory

  // P&L sources
  spreadCapture: number;      // Earned from bid-ask spread
  inventoryPnL: number;       // From position value changes
}
```

**Market Making Example:**

```
Market Maker Quotes:
  Bid: $0.64 for 1000 shares
  Ask: $0.66 for 1000 shares
  Spread: $0.02

Trade 1: Buyer takes 500 @ $0.66
  MM inventory: -500 Yes shares
  MM receives: $330

Trade 2: Seller hits 500 @ $0.64
  MM inventory: 0 (flat)
  MM pays: $320

Net P&L: $330 - $320 = $10 (spread capture)
```

**Inventory Risk Management:**

```
When MM is long (bought more than sold):
  → Lower bid price (less eager to buy)
  → Lower ask price (more eager to sell)

When MM is short (sold more than bought):
  → Raise bid price (more eager to buy)
  → Raise ask price (less eager to sell)

Skewed Quotes = Mid Price ± (Base Spread/2) ± (Inventory × Skew Factor)
```

### CLOB vs AMM Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CLOB (Polymarket's Choice)                       │
├─────────────────────────────────────────────────────────────────────┤
│ Pros:                          │ Cons:                              │
│ ├─ Capital efficient           │ ├─ Requires active market makers  │
│ ├─ Better price discovery      │ ├─ Can have low liquidity periods │
│ ├─ Lower slippage for large    │ ├─ More complex implementation    │
│ │   orders (with depth)        │ ├─ Order management overhead      │
│ ├─ Familiar trading interface  │ └─ May have empty order books     │
│ └─ Professional-grade trading  │                                    │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    AMM (Alternative Approach)                        │
├─────────────────────────────────────────────────────────────────────┤
│ Pros:                          │ Cons:                              │
│ ├─ Always provides liquidity   │ ├─ Capital inefficient             │
│ ├─ Simple implementation       │ ├─ Higher slippage                 │
│ ├─ No active management needed │ ├─ Impermanent loss for LPs        │
│ ├─ Permissionless LP           │ ├─ Less precise pricing            │
│ └─ 24/7 liquidity              │ └─ Arbitrage extraction            │
└─────────────────────────────────────────────────────────────────────┘
```

**AMM Pricing Formula (for reference):**

```
Constant Product AMM: x × y = k

Where:
  x = Yes token reserves
  y = No token reserves
  k = constant

Price of Yes = y / (x + y)
Price of No = x / (x + y)

Example:
  Reserves: 1000 Yes, 1000 No
  k = 1,000,000
  Yes price = 1000 / 2000 = $0.50
  No price = 1000 / 2000 = $0.50

After buying 100 Yes tokens:
  New Yes reserves: 900
  New No reserves: 1000000 / 900 = 1111.11
  New Yes price = 1111.11 / 2011.11 = $0.55
```

---

## Fee Structure

### Polymarket-Style Fee Model

```
┌─────────────────────────────────────────────────────────────────┐
│                        FEE STRUCTURE                            │
├─────────────────────────────────────────────────────────────────┤
│  Order Placement:        FREE                                   │
│  Order Cancellation:     FREE                                   │
│  Trading:                FREE                                   │
│  Winning Payouts:        ~2% fee on profits                     │
│  Withdrawals:            Network gas fees only                  │
└─────────────────────────────────────────────────────────────────┘
```

### Fee Calculation

```typescript
interface FeeStructure {
  // Trading fees
  makerFee: 0;           // No fee for providing liquidity
  takerFee: 0;           // No fee for taking liquidity

  // Settlement fees
  winningPayoutFee: 0.02; // 2% on winning position profits

  // Calculation
  grossPayout: number;    // shares × $1.00
  cost: number;           // shares × averageEntryPrice
  grossProfit: number;    // grossPayout - cost
  fee: number;            // grossProfit × winningPayoutFee
  netPayout: number;      // grossPayout - fee
}
```

**Fee Example:**

```
Position: 1000 Yes shares @ $0.60 average price
Outcome: Yes wins

Calculation:
  Gross Payout:   1000 × $1.00 = $1,000.00
  Cost Basis:     1000 × $0.60 = $600.00
  Gross Profit:   $1,000.00 - $600.00 = $400.00
  Fee (2%):       $400.00 × 0.02 = $8.00
  Net Payout:     $1,000.00 - $8.00 = $992.00
  Net Profit:     $992.00 - $600.00 = $392.00
```

### Gas Fees (Blockchain Costs)

```
On Polygon (Polymarket's network):
  Average transaction: $0.01 - $0.10

Operations requiring gas:
  ├─ Depositing funds
  ├─ Withdrawing funds
  ├─ Claiming winnings
  └─ Some order operations (depending on implementation)

Note: Order matching often occurs off-chain to minimize gas costs
```

---

## Mathematical Formulas

### Probability and Pricing

```
Basic Price-Probability Relationship:
──────────────────────────────────────
Implied Probability = Share Price / $1.00

P(Yes) = Yes Price
P(No) = No Price

Market Efficiency Constraint:
P(Yes) + P(No) ≈ 1.00

With spread:
P(Yes) + P(No) = 1.00 + spread_cost
```

### Expected Value (EV)

```
Expected Value Formula:
───────────────────────
EV = Σ (Probability_i × Payout_i) - Cost

For Yes share purchase:
EV = P(correct) × $1.00 + P(incorrect) × $0.00 - Price
EV = True_Probability × $1.00 - Market_Price

Positive EV Trade:
If you believe true probability > market price → Buy Yes
If you believe true probability < market price → Buy No (or sell Yes)
```

**EV Example:**

```
Market Price: Yes @ $0.60 (implies 60% probability)
Your Estimate: 75% probability

EV Calculation:
  EV = 0.75 × $1.00 - $0.60
  EV = $0.75 - $0.60
  EV = +$0.15 per share

This is a positive EV bet - worth taking if your estimate is accurate.
```

### Kelly Criterion (Optimal Bet Sizing)

```
Kelly Formula:
──────────────
f* = (bp - q) / b

Where:
  f* = Fraction of bankroll to bet
  b = Net odds (payout - 1)
  p = Probability of winning
  q = Probability of losing (1 - p)

For prediction markets:
  b = (1 / price) - 1

Example:
  Price: $0.60, Your estimate: 75% win probability
  b = (1/0.60) - 1 = 0.667
  f* = (0.667 × 0.75 - 0.25) / 0.667
  f* = (0.50 - 0.25) / 0.667
  f* = 0.375 (37.5% of bankroll)

Fractional Kelly (safer):
  Use f*/2 or f*/4 to reduce variance
```

### Return on Investment (ROI)

```
ROI Formula:
────────────
ROI = (Net Profit / Initial Investment) × 100%

For winning trade:
  Net Profit = Payout - Cost - Fees
  ROI = ((Payout - Cost - Fees) / Cost) × 100%

Example:
  Cost: $600 (1000 shares @ $0.60)
  Payout: $1000
  Fees: $8
  Net Profit: $1000 - $600 - $8 = $392
  ROI = ($392 / $600) × 100% = 65.3%
```

### Break-Even Analysis

```
Break-Even Price:
─────────────────
For a position to break even on exit before resolution:
  Break-Even = Entry Price + Transaction Costs

For resolution:
  Expected break-even requires:
  Entry Price = True Probability

With fees on winning:
  Adjusted Entry = Entry Price / (1 - fee_rate × (1 - Entry Price))
```

### Price Impact Estimation

```
For market orders, estimate price impact:
─────────────────────────────────────────
Simple Model:
  Average Execution Price ≈ Mid Price + (Order Size / Order Book Depth) × Spread

More accurate: Walk through order book
  For each price level until order filled:
    Executed_i = min(remaining_order, level_size)
    Cost_i = Executed_i × Price_i

  Average Price = Σ Cost_i / Total Executed
  Slippage = Average Price - Initial Best Price
```

---

## Risk Management

### Position Risk Metrics

```typescript
interface RiskMetrics {
  // Position-level risk
  maxLoss: number;           // Entry price × position size
  maxGain: number;           // (1 - entry price) × position size
  breakEvenProbability: number;  // Entry price

  // Portfolio-level risk
  totalExposure: number;     // Sum of all position costs
  correlatedRisk: number;    // Risk from related markets
  diversificationScore: number;

  // Value at Risk (VaR)
  var95: number;             // 95% confidence loss limit
  var99: number;             // 99% confidence loss limit
}
```

### Risk Limits

```
Recommended Position Limits:
────────────────────────────
Single Market:     Max 5-10% of portfolio
Correlated Group:  Max 15-20% of portfolio
Total Exposure:    Based on risk tolerance

Kelly Criterion Limits:
  Full Kelly:      Aggressive (high variance)
  Half Kelly:      Moderate
  Quarter Kelly:   Conservative
```

### Hedging Strategies

```
1. Direct Hedge:
   Long Yes + Long No = Locked value (minus spread cost)

2. Cross-Market Hedge:
   Long Yes on Market A
   Long No on correlated Market B

3. Time-Based Hedge:
   Reduce position size as uncertainty decreases
   Take profits as price moves favorably
```

---

## Implementation Checklist

### Core Trading System

```
□ Order Types
  □ Market orders with immediate execution
  □ Limit orders with price specification
  □ Time-in-force options (GTC, GTD, FOK, IOC)
  □ Order cancellation

□ Order Book
  □ Price-time priority matching
  □ Efficient data structures (sorted maps/heaps)
  □ Real-time best bid/ask updates
  □ Depth aggregation

□ Position Tracking
  □ Average entry price calculation
  □ Unrealized P&L computation
  □ Realized P&L on partial closes
  □ Position history

□ Market Lifecycle
  □ Market creation workflow
  □ Trading phase management
  □ Resolution process
  □ Settlement and payout
```

### Price and Risk

```
□ Pricing
  □ Real-time price updates
  □ Price bounds enforcement ($0.01 - $0.99)
  □ Spread calculation
  □ Market-wide Yes+No ≈ $1.00 constraint

□ Risk Management
  □ Position limits
  □ Exposure calculations
  □ Portfolio risk metrics
  □ Margin/collateral management
```

### Fees and Settlement

```
□ Fee System
  □ Winning payout fee calculation
  □ Fee collection on settlement
  □ Fee reporting

□ Settlement
  □ Oracle integration
  □ Outcome verification
  □ Automatic payout distribution
  □ Position liquidation
```

---

## Glossary

| Term | Definition |
|------|------------|
| **Ask** | The lowest price at which someone is willing to sell |
| **Bid** | The highest price at which someone is willing to buy |
| **CLOB** | Central Limit Order Book - traditional exchange matching |
| **Depth** | Total volume available at each price level |
| **Fill** | Execution of an order |
| **Implied Probability** | Probability derived from market price |
| **Liquidity** | Ease of buying/selling without price impact |
| **Long** | Holding a position expecting price increase |
| **Market Maker** | Entity providing liquidity on both sides |
| **Oracle** | Entity that determines market outcome |
| **Slippage** | Difference between expected and executed price |
| **Spread** | Difference between best bid and best ask |
| **Tick Size** | Minimum price increment ($0.01) |

---

## References

- [Polymarket Documentation](https://docs.polymarket.com/)
- [Polymarket CLOB API](https://docs.polymarket.com/#clob-api)
- [UMA Oracle System](https://docs.umaproject.org/)
- [Prediction Market Theory](https://en.wikipedia.org/wiki/Prediction_market)
