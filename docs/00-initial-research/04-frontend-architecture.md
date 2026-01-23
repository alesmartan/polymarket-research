# Frontend Architecture Documentation

## Polymarket-Style Prediction Market Platform

This document outlines the comprehensive frontend architecture for building a production-ready prediction market platform similar to Polymarket.

---

## Table of Contents

1. [Tech Stack Overview](#tech-stack-overview)
2. [Project Structure](#project-structure)
3. [Component Architecture](#component-architecture)
4. [Core UI Components](#core-ui-components)
5. [Web3 Integration Patterns](#web3-integration-patterns)
6. [State Management](#state-management)
7. [Real-Time Data Architecture](#real-time-data-architecture)
8. [Responsive Design System](#responsive-design-system)
9. [Performance Optimization](#performance-optimization)
10. [Testing Strategy](#testing-strategy)

---

## Tech Stack Overview

### Core Framework

| Technology | Purpose | Rationale |
|------------|---------|-----------|
| **Next.js 14+** | React Framework | SSR/SSG, App Router, API routes, excellent DX |
| **TypeScript** | Type Safety | Catch errors early, better IDE support |
| **React 18+** | UI Library | Concurrent features, Suspense, transitions |

### Web3 Integration

| Technology | Purpose | Rationale |
|------------|---------|-----------|
| **wagmi v2** | React Hooks for Ethereum | Type-safe, modular, great DX |
| **viem** | Ethereum Library | Modern ethers.js alternative, tree-shakeable |
| **web3.js** | Legacy Support | For specific contract interactions |
| **@rainbow-me/rainbowkit** | Wallet Connection UI | Beautiful, customizable, multi-wallet |

### State Management

| Technology | Purpose | Rationale |
|------------|---------|-----------|
| **Zustand** | Global State | Lightweight, simple API, great TypeScript support |
| **TanStack Query** | Server State | Caching, background updates, optimistic updates |
| **Jotai** | Atomic State | For complex derived state (optional) |

### Styling

| Technology | Purpose | Rationale |
|------------|---------|-----------|
| **Tailwind CSS** | Utility-First CSS | Rapid development, consistent design |
| **Radix UI** | Headless Components | Accessible, unstyled primitives |
| **Framer Motion** | Animations | Smooth, performant animations |
| **class-variance-authority** | Component Variants | Type-safe variant management |

### Data Visualization

| Technology | Purpose | Rationale |
|------------|---------|-----------|
| **Lightweight Charts** | Price Charts | TradingView's library, financial-grade |
| **Recharts** | General Charts | Composable, React-native |
| **D3.js** | Custom Visualizations | Order book, depth charts |

### Development Tools

| Technology | Purpose |
|------------|---------|
| **ESLint** | Code linting |
| **Prettier** | Code formatting |
| **Husky** | Git hooks |
| **Vitest** | Unit testing |
| **Playwright** | E2E testing |
| **Storybook** | Component documentation |

---

## Project Structure

```
src/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth route group
│   │   ├── login/
│   │   └── signup/
│   ├── (main)/                   # Main app route group
│   │   ├── markets/
│   │   │   ├── [marketId]/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── loading.tsx
│   │   │   │   └── error.tsx
│   │   │   └── page.tsx
│   │   ├── portfolio/
│   │   │   └── page.tsx
│   │   ├── activity/
│   │   │   └── page.tsx
│   │   └── layout.tsx
│   ├── api/                      # API routes
│   │   ├── markets/
│   │   ├── orders/
│   │   └── user/
│   ├── layout.tsx
│   ├── page.tsx
│   └── providers.tsx
│
├── components/
│   ├── ui/                       # Base UI components
│   │   ├── button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   ├── Button.stories.tsx
│   │   │   └── index.ts
│   │   ├── input/
│   │   ├── card/
│   │   ├── modal/
│   │   ├── dropdown/
│   │   ├── tabs/
│   │   ├── tooltip/
│   │   └── skeleton/
│   │
│   ├── market/                   # Market-specific components
│   │   ├── MarketCard/
│   │   ├── MarketGrid/
│   │   ├── MarketDetail/
│   │   ├── PriceChart/
│   │   ├── OrderBook/
│   │   ├── MarketStats/
│   │   └── ResolutionInfo/
│   │
│   ├── trading/                  # Trading components
│   │   ├── TradingPanel/
│   │   ├── OrderForm/
│   │   ├── OutcomeSelector/
│   │   ├── AmountInput/
│   │   ├── SlippageSettings/
│   │   ├── OrderConfirmation/
│   │   └── PayoutCalculator/
│   │
│   ├── portfolio/                # Portfolio components
│   │   ├── PositionsList/
│   │   ├── PositionCard/
│   │   ├── PnLChart/
│   │   ├── TradeHistory/
│   │   └── PendingOrders/
│   │
│   ├── wallet/                   # Wallet components
│   │   ├── ConnectButton/
│   │   ├── WalletModal/
│   │   ├── AccountInfo/
│   │   ├── DepositWithdraw/
│   │   └── TransactionHistory/
│   │
│   ├── layout/                   # Layout components
│   │   ├── Header/
│   │   ├── Footer/
│   │   ├── Sidebar/
│   │   ├── MobileNav/
│   │   └── PageContainer/
│   │
│   └── shared/                   # Shared components
│       ├── CategoryFilter/
│       ├── SearchBar/
│       ├── Pagination/
│       ├── LoadingStates/
│       └── ErrorBoundary/
│
├── hooks/                        # Custom React hooks
│   ├── useMarkets.ts
│   ├── useMarketDetail.ts
│   ├── useOrders.ts
│   ├── usePositions.ts
│   ├── usePriceHistory.ts
│   ├── useOrderBook.ts
│   ├── useWallet.ts
│   ├── useTransaction.ts
│   ├── useWebSocket.ts
│   └── useDebounce.ts
│
├── lib/                          # Utility libraries
│   ├── web3/
│   │   ├── config.ts             # wagmi/viem config
│   │   ├── contracts.ts          # Contract ABIs and addresses
│   │   ├── actions.ts            # Contract write functions
│   │   └── utils.ts              # Web3 utilities
│   ├── api/
│   │   ├── client.ts             # API client setup
│   │   ├── markets.ts            # Market API functions
│   │   ├── orders.ts             # Order API functions
│   │   └── user.ts               # User API functions
│   ├── utils/
│   │   ├── formatting.ts         # Number/date formatting
│   │   ├── calculations.ts       # Trading calculations
│   │   └── validation.ts         # Input validation
│   └── constants/
│       ├── markets.ts
│       ├── chains.ts
│       └── errors.ts
│
├── stores/                       # Zustand stores
│   ├── useMarketStore.ts
│   ├── useOrderStore.ts
│   ├── useUIStore.ts
│   └── useUserStore.ts
│
├── types/                        # TypeScript types
│   ├── market.ts
│   ├── order.ts
│   ├── position.ts
│   ├── user.ts
│   └── api.ts
│
├── styles/                       # Global styles
│   ├── globals.css
│   └── themes/
│
└── config/                       # App configuration
    ├── site.ts
    ├── navigation.ts
    └── features.ts
```

---

## Component Architecture

### Component Hierarchy Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              RootLayout                                      │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                           Providers                                    │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────────┐  │  │
│  │  │ WagmiConfig │ │ QueryClient │ │ ThemeProvider│ │ WebSocketProvider│ │  │
│  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────────┘  │  │
│  │                                                                        │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │                         AppShell                                 │  │  │
│  │  │  ┌─────────────────────────────────────────────────────────┐    │  │  │
│  │  │  │                        Header                            │    │  │  │
│  │  │  │  ┌──────┐ ┌─────────┐ ┌────────┐ ┌───────────────────┐  │    │  │  │
│  │  │  │  │ Logo │ │ NavMenu │ │ Search │ │ WalletConnectBtn  │  │    │  │  │
│  │  │  │  └──────┘ └─────────┘ └────────┘ └───────────────────┘  │    │  │  │
│  │  │  └─────────────────────────────────────────────────────────┘    │  │  │
│  │  │                                                                  │  │  │
│  │  │  ┌─────────────────────────────────────────────────────────┐    │  │  │
│  │  │  │                      MainContent                         │    │  │  │
│  │  │  │                    {children}                            │    │  │  │
│  │  │  └─────────────────────────────────────────────────────────┘    │  │  │
│  │  │                                                                  │  │  │
│  │  │  ┌─────────────────────────────────────────────────────────┐    │  │  │
│  │  │  │                  MobileNavigation                        │    │  │  │
│  │  │  └─────────────────────────────────────────────────────────┘    │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Market Discovery Page Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           MarketsPage                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         PageHeader                                     │  │
│  │  ┌─────────────────┐  ┌─────────────────────────────────────────────┐ │  │
│  │  │  Title/Subtitle │  │              SearchBar                      │ │  │
│  │  └─────────────────┘  └─────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                       CategoryTabs                                     │  │
│  │  ┌─────┐ ┌────────┐ ┌─────────┐ ┌────────┐ ┌────────┐ ┌──────────┐   │  │
│  │  │ All │ │Politics│ │ Finance │ │ Sports │ │ Crypto │ │   More   │   │  │
│  │  └─────┘ └────────┘ └─────────┘ └────────┘ └────────┘ └──────────┘   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        FilterBar                                       │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐  │  │
│  │  │  SortSelect  │ │ VolumeFilter │ │ StatusFilter │ │  ViewToggle  │  │  │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        MarketGrid                                      │  │
│  │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐          │  │
│  │  │   MarketCard    │ │   MarketCard    │ │   MarketCard    │          │  │
│  │  │ ┌─────────────┐ │ │ ┌─────────────┐ │ │ ┌─────────────┐ │          │  │
│  │  │ │   Image     │ │ │ │   Image     │ │ │ │   Image     │ │          │  │
│  │  │ ├─────────────┤ │ │ ├─────────────┤ │ │ ├─────────────┤ │          │  │
│  │  │ │   Title     │ │ │ │   Title     │ │ │ │   Title     │ │          │  │
│  │  │ ├─────────────┤ │ │ ├─────────────┤ │ │ ├─────────────┤ │          │  │
│  │  │ │ ProbDisplay │ │ │ │ ProbDisplay │ │ │ │ ProbDisplay │ │          │  │
│  │  │ │  [YES: 65%] │ │ │ │  [YES: 42%] │ │ │ │  [YES: 78%] │ │          │  │
│  │  │ ├─────────────┤ │ │ ├─────────────┤ │ │ ├─────────────┤ │          │  │
│  │  │ │ MarketStats │ │ │ │ MarketStats │ │ │ │ MarketStats │ │          │  │
│  │  │ │ Vol│Traders │ │ │ │ Vol│Traders │ │ │ │ Vol│Traders │ │          │  │
│  │  │ └─────────────┘ │ │ └─────────────┘ │ │ └─────────────┘ │          │  │
│  │  └─────────────────┘ └─────────────────┘ └─────────────────┘          │  │
│  │                                                                        │  │
│  │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐          │  │
│  │  │   MarketCard    │ │   MarketCard    │ │   MarketCard    │          │  │
│  │  └─────────────────┘ └─────────────────┘ └─────────────────┘          │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        Pagination                                      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Market Detail Page Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MarketDetailPage                                     │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                       MarketHeader                                     │  │
│  │  ┌────────────────────────────────────────────────────────────────┐   │  │
│  │  │  [Image] Market Title                            [Share] [Star]│   │  │
│  │  │  Category Badge  •  End Date  •  Resolution Source             │   │  │
│  │  └────────────────────────────────────────────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌─────────────────────────────────────┐ ┌───────────────────────────────┐  │
│  │          MainContent (2/3)          │ │      TradingSidebar (1/3)     │  │
│  │  ┌───────────────────────────────┐  │ │  ┌─────────────────────────┐  │  │
│  │  │      ProbabilityDisplay       │  │ │  │     TradingPanel        │  │  │
│  │  │  ┌─────────┐   ┌─────────┐    │  │ │  │  ┌───────────────────┐  │  │  │
│  │  │  │   YES   │   │   NO    │    │  │ │  │  │  OutcomeSelector  │  │  │  │
│  │  │  │   65¢   │   │   35¢   │    │  │ │  │  │  [YES]    [NO]    │  │  │  │
│  │  │  │  ▲ 2.3% │   │  ▼ 2.3% │    │  │ │  │  └───────────────────┘  │  │  │
│  │  │  └─────────┘   └─────────┘    │  │ │  │  ┌───────────────────┐  │  │  │
│  │  └───────────────────────────────┘  │ │  │  │   OrderTypeToggle │  │  │  │
│  │                                      │ │  │  │  Market │ Limit   │  │  │  │
│  │  ┌───────────────────────────────┐  │ │  │  └───────────────────┘  │  │  │
│  │  │         PriceChart            │  │ │  │  ┌───────────────────┐  │  │  │
│  │  │  ┌─────────────────────────┐  │  │ │  │  │    AmountInput    │  │  │  │
│  │  │  │   TimeframeSelector     │  │  │ │  │  │  [$] Amount       │  │  │  │
│  │  │  │  1H│24H│7D│30D│ALL      │  │  │ │  │  │  [25%][50%][MAX]  │  │  │  │
│  │  │  └─────────────────────────┘  │  │ │  │  └───────────────────┘  │  │  │
│  │  │  ┌─────────────────────────┐  │  │ │  │  ┌───────────────────┐  │  │  │
│  │  │  │                         │  │  │ │  │  │  LimitPriceInput  │  │  │  │
│  │  │  │    [Price History       │  │  │ │  │  │  (if limit order) │  │  │  │
│  │  │  │         Chart]          │  │  │ │  │  └───────────────────┘  │  │  │
│  │  │  │                         │  │  │ │  │  ┌───────────────────┐  │  │  │
│  │  │  └─────────────────────────┘  │  │ │  │  │ PayoutCalculator  │  │  │  │
│  │  └───────────────────────────────┘  │ │  │  │  Shares: 153.8    │  │  │  │
│  │                                      │ │  │  │  Payout: $153.80  │  │  │  │
│  │  ┌───────────────────────────────┐  │ │  │  │  Profit: +$53.80  │  │  │  │
│  │  │        OrderBook              │  │ │  │  └───────────────────┘  │  │  │
│  │  │  ┌───────────┬───────────┐    │  │ │  │  ┌───────────────────┐  │  │  │
│  │  │  │   BIDS    │   ASKS    │    │  │ │  │  │ SlippageSettings  │  │  │  │
│  │  │  │  65¢ 500  │  66¢ 300  │    │  │ │  │  │  [0.5%][1%][2%]   │  │  │  │
│  │  │  │  64¢ 800  │  67¢ 450  │    │  │ │  │  └───────────────────┘  │  │  │
│  │  │  │  63¢ 1200 │  68¢ 600  │    │  │ │  │  ┌───────────────────┐  │  │  │
│  │  │  │  62¢ 2000 │  69¢ 800  │    │  │ │  │  │  [  BUY YES  ]    │  │  │  │
│  │  │  └───────────┴───────────┘    │  │ │  │  └───────────────────┘  │  │  │
│  │  └───────────────────────────────┘  │ │  └─────────────────────────┘  │  │
│  │                                      │ │                               │  │
│  │  ┌───────────────────────────────┐  │ │  ┌─────────────────────────┐  │  │
│  │  │      ContentTabs              │  │ │  │    MarketStats          │  │  │
│  │  │  [Description][Activity]      │  │ │  │  Volume: $1.2M          │  │  │
│  │  │  [Comments][Related]          │  │ │  │  Liquidity: $450K       │  │  │
│  │  │  ┌─────────────────────────┐  │  │ │  │  Traders: 2,341         │  │  │
│  │  │  │                         │  │  │ │  │  24h Change: +5.2%      │  │  │
│  │  │  │   Tab Content Area      │  │  │ │  └─────────────────────────┘  │  │
│  │  │  │                         │  │  │ │                               │  │
│  │  │  └─────────────────────────┘  │  │ └───────────────────────────────┘  │
│  │  └───────────────────────────────┘  │                                     │
│  └─────────────────────────────────────┘                                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Portfolio Dashboard Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PortfolioPage                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                      PortfolioHeader                                   │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │  Portfolio Overview                         [Export] [Settings] │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                       PortfolioSummary                                 │  │
│  │  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐ ┌─────────┐ │  │
│  │  │  Total Value   │ │  Total P&L     │ │  Open Orders   │ │  Win %  │ │  │
│  │  │   $12,450.00   │ │   +$2,340.50   │ │      12        │ │  67.3%  │ │  │
│  │  │   ▲ 3.2%       │ │   ▲ 23.1%      │ │                │ │         │ │  │
│  │  └────────────────┘ └────────────────┘ └────────────────┘ └─────────┘ │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        PnLChart                                        │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │  [Cumulative P&L over time graph]                               │  │  │
│  │  │                                                                  │  │  │
│  │  │   ^                                                              │  │  │
│  │  │   │            ╭──────╮                                          │  │  │
│  │  │   │     ╭─────╯      ╰───────────────╮                          │  │  │
│  │  │   │────╯                              ╰────                      │  │  │
│  │  │   └──────────────────────────────────────────────────>          │  │  │
│  │  │   1W          1M          3M          6M          1Y             │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                      PositionsTabs                                     │  │
│  │  [Active Positions] [Pending Orders] [Trade History] [Resolved]       │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │                    PositionsList                                 │  │  │
│  │  │  ┌───────────────────────────────────────────────────────────┐  │  │  │
│  │  │  │  PositionCard                                              │  │  │  │
│  │  │  │  ┌──────┐ Market Title          YES @ 65¢    [Sell][Hold] │  │  │  │
│  │  │  │  │ Img  │ 500 shares            +$125.00     Exp: Mar 15  │  │  │  │
│  │  │  │  └──────┘ Avg: 55¢  Current: 65¢  P&L: +18.2%             │  │  │  │
│  │  │  └───────────────────────────────────────────────────────────┘  │  │  │
│  │  │  ┌───────────────────────────────────────────────────────────┐  │  │  │
│  │  │  │  PositionCard                                              │  │  │  │
│  │  │  │  ┌──────┐ Market Title          NO @ 42¢     [Sell][Hold] │  │  │  │
│  │  │  │  │ Img  │ 200 shares            -$15.00      Exp: Apr 1   │  │  │  │
│  │  │  │  └──────┘ Avg: 38¢  Current: 42¢  P&L: +10.5%             │  │  │  │
│  │  │  └───────────────────────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Core UI Components

### 1. Market Discovery Components

#### MarketCard Component

```typescript
// components/market/MarketCard/MarketCard.tsx
import { FC } from 'react';
import Image from 'next/image';
import Link from 'next/link';
import { Market } from '@/types/market';
import { formatCurrency, formatPercentage } from '@/lib/utils/formatting';
import { ProbabilityBar } from './ProbabilityBar';
import { MarketBadge } from './MarketBadge';

interface MarketCardProps {
  market: Market;
  variant?: 'default' | 'compact' | 'featured';
  showStats?: boolean;
}

export const MarketCard: FC<MarketCardProps> = ({
  market,
  variant = 'default',
  showStats = true,
}) => {
  const { id, title, image, category, yesPrice, volume24h, tradersCount, endDate } = market;

  return (
    <Link href={`/markets/${id}`}>
      <article className="group relative rounded-xl border border-gray-200 bg-white p-4
                         transition-all hover:border-gray-300 hover:shadow-lg
                         dark:border-gray-800 dark:bg-gray-900">
        {/* Market Image */}
        <div className="relative mb-3 aspect-video overflow-hidden rounded-lg">
          <Image
            src={image}
            alt={title}
            fill
            className="object-cover transition-transform group-hover:scale-105"
          />
          <MarketBadge category={category} className="absolute left-2 top-2" />
        </div>

        {/* Title */}
        <h3 className="mb-3 line-clamp-2 text-sm font-semibold text-gray-900
                      dark:text-white">
          {title}
        </h3>

        {/* Probability Display */}
        <ProbabilityBar yesPrice={yesPrice} />

        {/* Stats */}
        {showStats && (
          <div className="mt-3 flex items-center justify-between text-xs text-gray-500">
            <span>{formatCurrency(volume24h)} Vol</span>
            <span>{tradersCount.toLocaleString()} traders</span>
          </div>
        )}

        {/* End Date */}
        <div className="mt-2 text-xs text-gray-400">
          Ends {new Date(endDate).toLocaleDateString()}
        </div>
      </article>
    </Link>
  );
};
```

#### CategoryFilter Component

```typescript
// components/shared/CategoryFilter/CategoryFilter.tsx
import { FC } from 'react';
import { cn } from '@/lib/utils';

const CATEGORIES = [
  { id: 'all', label: 'All', icon: '🌐' },
  { id: 'politics', label: 'Politics', icon: '🏛️' },
  { id: 'finance', label: 'Finance', icon: '📈' },
  { id: 'sports', label: 'Sports', icon: '⚽' },
  { id: 'crypto', label: 'Crypto', icon: '₿' },
  { id: 'entertainment', label: 'Entertainment', icon: '🎬' },
  { id: 'science', label: 'Science', icon: '🔬' },
] as const;

interface CategoryFilterProps {
  selected: string;
  onChange: (category: string) => void;
  counts?: Record<string, number>;
}

export const CategoryFilter: FC<CategoryFilterProps> = ({
  selected,
  onChange,
  counts,
}) => {
  return (
    <nav className="flex gap-2 overflow-x-auto pb-2 scrollbar-hide">
      {CATEGORIES.map((category) => (
        <button
          key={category.id}
          onClick={() => onChange(category.id)}
          className={cn(
            'flex items-center gap-2 whitespace-nowrap rounded-full px-4 py-2',
            'text-sm font-medium transition-colors',
            selected === category.id
              ? 'bg-blue-600 text-white'
              : 'bg-gray-100 text-gray-700 hover:bg-gray-200 dark:bg-gray-800 dark:text-gray-300'
          )}
        >
          <span>{category.icon}</span>
          <span>{category.label}</span>
          {counts?.[category.id] && (
            <span className="ml-1 text-xs opacity-70">
              ({counts[category.id]})
            </span>
          )}
        </button>
      ))}
    </nav>
  );
};
```

### 2. Market Detail Components

#### PriceChart Component

```typescript
// components/market/PriceChart/PriceChart.tsx
import { FC, useEffect, useRef, useState } from 'react';
import { createChart, IChartApi, ISeriesApi, LineData } from 'lightweight-charts';
import { usePriceHistory } from '@/hooks/usePriceHistory';

interface PriceChartProps {
  marketId: string;
  height?: number;
}

const TIMEFRAMES = [
  { label: '1H', value: '1h' },
  { label: '24H', value: '24h' },
  { label: '7D', value: '7d' },
  { label: '30D', value: '30d' },
  { label: 'ALL', value: 'all' },
] as const;

export const PriceChart: FC<PriceChartProps> = ({ marketId, height = 300 }) => {
  const chartContainerRef = useRef<HTMLDivElement>(null);
  const chartRef = useRef<IChartApi | null>(null);
  const seriesRef = useRef<ISeriesApi<'Area'> | null>(null);

  const [timeframe, setTimeframe] = useState<string>('24h');
  const { data, isLoading } = usePriceHistory(marketId, timeframe);

  useEffect(() => {
    if (!chartContainerRef.current) return;

    const chart = createChart(chartContainerRef.current, {
      height,
      layout: {
        background: { type: 'solid', color: 'transparent' },
        textColor: '#9CA3AF',
      },
      grid: {
        vertLines: { color: '#E5E7EB20' },
        horzLines: { color: '#E5E7EB20' },
      },
      crosshair: {
        mode: 1,
      },
      rightPriceScale: {
        borderColor: '#E5E7EB',
        scaleMargins: { top: 0.1, bottom: 0.1 },
      },
      timeScale: {
        borderColor: '#E5E7EB',
        timeVisible: true,
      },
    });

    const areaSeries = chart.addAreaSeries({
      lineColor: '#3B82F6',
      topColor: '#3B82F640',
      bottomColor: '#3B82F610',
      lineWidth: 2,
    });

    chartRef.current = chart;
    seriesRef.current = areaSeries;

    const handleResize = () => {
      if (chartContainerRef.current) {
        chart.applyOptions({ width: chartContainerRef.current.clientWidth });
      }
    };

    window.addEventListener('resize', handleResize);
    handleResize();

    return () => {
      window.removeEventListener('resize', handleResize);
      chart.remove();
    };
  }, [height]);

  useEffect(() => {
    if (seriesRef.current && data) {
      seriesRef.current.setData(data);
      chartRef.current?.timeScale().fitContent();
    }
  }, [data]);

  return (
    <div className="rounded-xl border border-gray-200 bg-white p-4 dark:border-gray-800 dark:bg-gray-900">
      {/* Timeframe Selector */}
      <div className="mb-4 flex gap-2">
        {TIMEFRAMES.map((tf) => (
          <button
            key={tf.value}
            onClick={() => setTimeframe(tf.value)}
            className={cn(
              'rounded-md px-3 py-1 text-sm font-medium transition-colors',
              timeframe === tf.value
                ? 'bg-blue-600 text-white'
                : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
            )}
          >
            {tf.label}
          </button>
        ))}
      </div>

      {/* Chart Container */}
      <div ref={chartContainerRef} className="relative">
        {isLoading && (
          <div className="absolute inset-0 flex items-center justify-center bg-white/50">
            <Spinner />
          </div>
        )}
      </div>
    </div>
  );
};
```

#### OrderBook Component

```typescript
// components/market/OrderBook/OrderBook.tsx
import { FC, useMemo } from 'react';
import { useOrderBook } from '@/hooks/useOrderBook';
import { formatPrice, formatShares } from '@/lib/utils/formatting';

interface OrderBookProps {
  marketId: string;
  outcome: 'YES' | 'NO';
  onPriceClick?: (price: number) => void;
}

export const OrderBook: FC<OrderBookProps> = ({ marketId, outcome, onPriceClick }) => {
  const { bids, asks, spread, isLoading } = useOrderBook(marketId, outcome);

  const maxTotal = useMemo(() => {
    const maxBid = Math.max(...bids.map((b) => b.total), 0);
    const maxAsk = Math.max(...asks.map((a) => a.total), 0);
    return Math.max(maxBid, maxAsk);
  }, [bids, asks]);

  if (isLoading) {
    return <OrderBookSkeleton />;
  }

  return (
    <div className="rounded-xl border border-gray-200 bg-white dark:border-gray-800 dark:bg-gray-900">
      <div className="border-b border-gray-200 p-3 dark:border-gray-800">
        <h3 className="text-sm font-semibold">Order Book</h3>
        <div className="mt-1 text-xs text-gray-500">
          Spread: {formatPrice(spread)} ({((spread / bids[0]?.price) * 100).toFixed(2)}%)
        </div>
      </div>

      <div className="grid grid-cols-2 gap-px bg-gray-200 dark:bg-gray-800">
        {/* Bids (Buy orders) */}
        <div className="bg-white dark:bg-gray-900">
          <div className="grid grid-cols-3 border-b border-gray-100 px-3 py-2 text-xs font-medium text-gray-500">
            <span>Price</span>
            <span className="text-right">Size</span>
            <span className="text-right">Total</span>
          </div>
          <div className="max-h-48 overflow-y-auto">
            {bids.slice(0, 10).map((bid, index) => (
              <div
                key={index}
                onClick={() => onPriceClick?.(bid.price)}
                className="relative grid cursor-pointer grid-cols-3 px-3 py-1.5 text-xs hover:bg-gray-50"
              >
                {/* Depth visualization */}
                <div
                  className="absolute inset-y-0 left-0 bg-green-500/10"
                  style={{ width: `${(bid.total / maxTotal) * 100}%` }}
                />
                <span className="relative text-green-600">{formatPrice(bid.price)}</span>
                <span className="relative text-right">{formatShares(bid.size)}</span>
                <span className="relative text-right text-gray-500">
                  {formatShares(bid.total)}
                </span>
              </div>
            ))}
          </div>
        </div>

        {/* Asks (Sell orders) */}
        <div className="bg-white dark:bg-gray-900">
          <div className="grid grid-cols-3 border-b border-gray-100 px-3 py-2 text-xs font-medium text-gray-500">
            <span>Price</span>
            <span className="text-right">Size</span>
            <span className="text-right">Total</span>
          </div>
          <div className="max-h-48 overflow-y-auto">
            {asks.slice(0, 10).map((ask, index) => (
              <div
                key={index}
                onClick={() => onPriceClick?.(ask.price)}
                className="relative grid cursor-pointer grid-cols-3 px-3 py-1.5 text-xs hover:bg-gray-50"
              >
                {/* Depth visualization */}
                <div
                  className="absolute inset-y-0 right-0 bg-red-500/10"
                  style={{ width: `${(ask.total / maxTotal) * 100}%` }}
                />
                <span className="relative text-red-600">{formatPrice(ask.price)}</span>
                <span className="relative text-right">{formatShares(ask.size)}</span>
                <span className="relative text-right text-gray-500">
                  {formatShares(ask.total)}
                </span>
              </div>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
};
```

### 3. Trading Interface Components

#### TradingPanel Component

```typescript
// components/trading/TradingPanel/TradingPanel.tsx
import { FC, useState, useCallback, useMemo } from 'react';
import { useAccount, useBalance } from 'wagmi';
import { OutcomeSelector } from '../OutcomeSelector';
import { OrderTypeToggle } from '../OrderTypeToggle';
import { AmountInput } from '../AmountInput';
import { LimitPriceInput } from '../LimitPriceInput';
import { SlippageSettings } from '../SlippageSettings';
import { PayoutCalculator } from '../PayoutCalculator';
import { OrderConfirmation } from '../OrderConfirmation';
import { useCreateOrder } from '@/hooks/useCreateOrder';
import { Market } from '@/types/market';

interface TradingPanelProps {
  market: Market;
}

type Outcome = 'YES' | 'NO';
type OrderType = 'market' | 'limit';

export const TradingPanel: FC<TradingPanelProps> = ({ market }) => {
  const { address, isConnected } = useAccount();
  const { data: balance } = useBalance({ address, token: USDC_ADDRESS });

  const [outcome, setOutcome] = useState<Outcome>('YES');
  const [orderType, setOrderType] = useState<OrderType>('market');
  const [amount, setAmount] = useState<string>('');
  const [limitPrice, setLimitPrice] = useState<string>('');
  const [slippage, setSlippage] = useState<number>(0.5);
  const [showConfirmation, setShowConfirmation] = useState(false);

  const { createOrder, isLoading, error } = useCreateOrder();

  const currentPrice = outcome === 'YES' ? market.yesPrice : market.noPrice;

  const calculatedShares = useMemo(() => {
    const price = orderType === 'limit' ? parseFloat(limitPrice) : currentPrice;
    if (!amount || !price) return 0;
    return parseFloat(amount) / price;
  }, [amount, limitPrice, currentPrice, orderType]);

  const potentialPayout = useMemo(() => {
    return calculatedShares; // Each share pays $1 if correct
  }, [calculatedShares]);

  const potentialProfit = useMemo(() => {
    return potentialPayout - parseFloat(amount || '0');
  }, [potentialPayout, amount]);

  const handleSubmit = useCallback(async () => {
    if (!isConnected) {
      // Trigger wallet connection
      return;
    }

    const orderParams = {
      marketId: market.id,
      outcome,
      orderType,
      amount: parseFloat(amount),
      limitPrice: orderType === 'limit' ? parseFloat(limitPrice) : undefined,
      slippage,
    };

    setShowConfirmation(true);
  }, [market.id, outcome, orderType, amount, limitPrice, slippage, isConnected]);

  const handleConfirm = useCallback(async () => {
    try {
      await createOrder({
        marketId: market.id,
        outcome,
        orderType,
        amount: parseFloat(amount),
        limitPrice: orderType === 'limit' ? parseFloat(limitPrice) : undefined,
        slippage,
      });

      // Reset form on success
      setAmount('');
      setLimitPrice('');
      setShowConfirmation(false);
    } catch (err) {
      console.error('Order failed:', err);
    }
  }, [createOrder, market.id, outcome, orderType, amount, limitPrice, slippage]);

  return (
    <div className="rounded-xl border border-gray-200 bg-white p-4 dark:border-gray-800 dark:bg-gray-900">
      {/* Outcome Selector */}
      <OutcomeSelector
        selected={outcome}
        onChange={setOutcome}
        yesPrice={market.yesPrice}
        noPrice={market.noPrice}
      />

      {/* Order Type Toggle */}
      <OrderTypeToggle
        selected={orderType}
        onChange={setOrderType}
        className="mt-4"
      />

      {/* Amount Input */}
      <AmountInput
        value={amount}
        onChange={setAmount}
        maxAmount={balance?.formatted}
        currency="USDC"
        className="mt-4"
      />

      {/* Limit Price Input (conditional) */}
      {orderType === 'limit' && (
        <LimitPriceInput
          value={limitPrice}
          onChange={setLimitPrice}
          currentPrice={currentPrice}
          outcome={outcome}
          className="mt-4"
        />
      )}

      {/* Payout Calculator */}
      <PayoutCalculator
        shares={calculatedShares}
        potentialPayout={potentialPayout}
        potentialProfit={potentialProfit}
        className="mt-4"
      />

      {/* Slippage Settings (for market orders) */}
      {orderType === 'market' && (
        <SlippageSettings
          value={slippage}
          onChange={setSlippage}
          className="mt-4"
        />
      )}

      {/* Submit Button */}
      <button
        onClick={handleSubmit}
        disabled={!amount || isLoading}
        className={cn(
          'mt-6 w-full rounded-lg py-3 text-sm font-semibold text-white transition-colors',
          outcome === 'YES'
            ? 'bg-green-600 hover:bg-green-700 disabled:bg-green-600/50'
            : 'bg-red-600 hover:bg-red-700 disabled:bg-red-600/50'
        )}
      >
        {isLoading ? (
          <Spinner className="mx-auto" />
        ) : isConnected ? (
          `Buy ${outcome}`
        ) : (
          'Connect Wallet'
        )}
      </button>

      {/* Error Display */}
      {error && (
        <div className="mt-2 text-center text-sm text-red-500">
          {error.message}
        </div>
      )}

      {/* Order Confirmation Modal */}
      <OrderConfirmation
        isOpen={showConfirmation}
        onClose={() => setShowConfirmation(false)}
        onConfirm={handleConfirm}
        order={{
          market,
          outcome,
          orderType,
          amount: parseFloat(amount),
          limitPrice: orderType === 'limit' ? parseFloat(limitPrice) : currentPrice,
          shares: calculatedShares,
          potentialPayout,
          slippage,
        }}
        isLoading={isLoading}
      />
    </div>
  );
};
```

#### OutcomeSelector Component

```typescript
// components/trading/OutcomeSelector/OutcomeSelector.tsx
import { FC } from 'react';
import { cn } from '@/lib/utils';
import { formatPrice, formatPriceChange } from '@/lib/utils/formatting';

interface OutcomeSelectorProps {
  selected: 'YES' | 'NO';
  onChange: (outcome: 'YES' | 'NO') => void;
  yesPrice: number;
  noPrice: number;
  yesChange24h?: number;
  noChange24h?: number;
  className?: string;
}

export const OutcomeSelector: FC<OutcomeSelectorProps> = ({
  selected,
  onChange,
  yesPrice,
  noPrice,
  yesChange24h,
  noChange24h,
  className,
}) => {
  return (
    <div className={cn('grid grid-cols-2 gap-2', className)}>
      {/* YES Option */}
      <button
        onClick={() => onChange('YES')}
        className={cn(
          'flex flex-col items-center rounded-lg border-2 p-4 transition-all',
          selected === 'YES'
            ? 'border-green-500 bg-green-50 dark:bg-green-900/20'
            : 'border-gray-200 hover:border-gray-300 dark:border-gray-700'
        )}
      >
        <span className="text-sm font-medium text-gray-600 dark:text-gray-400">
          Yes
        </span>
        <span className="mt-1 text-2xl font-bold text-green-600">
          {formatPrice(yesPrice)}
        </span>
        {yesChange24h !== undefined && (
          <span
            className={cn(
              'mt-1 text-xs font-medium',
              yesChange24h >= 0 ? 'text-green-500' : 'text-red-500'
            )}
          >
            {formatPriceChange(yesChange24h)}
          </span>
        )}
      </button>

      {/* NO Option */}
      <button
        onClick={() => onChange('NO')}
        className={cn(
          'flex flex-col items-center rounded-lg border-2 p-4 transition-all',
          selected === 'NO'
            ? 'border-red-500 bg-red-50 dark:bg-red-900/20'
            : 'border-gray-200 hover:border-gray-300 dark:border-gray-700'
        )}
      >
        <span className="text-sm font-medium text-gray-600 dark:text-gray-400">
          No
        </span>
        <span className="mt-1 text-2xl font-bold text-red-600">
          {formatPrice(noPrice)}
        </span>
        {noChange24h !== undefined && (
          <span
            className={cn(
              'mt-1 text-xs font-medium',
              noChange24h >= 0 ? 'text-green-500' : 'text-red-500'
            )}
          >
            {formatPriceChange(noChange24h)}
          </span>
        )}
      </button>
    </div>
  );
};
```

### 4. Portfolio Components

#### PositionCard Component

```typescript
// components/portfolio/PositionCard/PositionCard.tsx
import { FC } from 'react';
import Image from 'next/image';
import Link from 'next/link';
import { Position } from '@/types/position';
import { formatCurrency, formatPercentage, formatShares } from '@/lib/utils/formatting';
import { cn } from '@/lib/utils';

interface PositionCardProps {
  position: Position;
  onSell?: () => void;
}

export const PositionCard: FC<PositionCardProps> = ({ position, onSell }) => {
  const {
    market,
    outcome,
    shares,
    averageCost,
    currentPrice,
    pnl,
    pnlPercentage,
  } = position;

  const isProfit = pnl >= 0;

  return (
    <div className="flex items-center gap-4 rounded-xl border border-gray-200 bg-white p-4
                    transition-colors hover:border-gray-300 dark:border-gray-800 dark:bg-gray-900">
      {/* Market Image */}
      <Link href={`/markets/${market.id}`} className="shrink-0">
        <div className="relative h-16 w-16 overflow-hidden rounded-lg">
          <Image src={market.image} alt={market.title} fill className="object-cover" />
        </div>
      </Link>

      {/* Position Info */}
      <div className="min-w-0 flex-1">
        <Link href={`/markets/${market.id}`}>
          <h3 className="truncate font-medium text-gray-900 hover:text-blue-600 dark:text-white">
            {market.title}
          </h3>
        </Link>
        <div className="mt-1 flex items-center gap-2 text-sm">
          <span
            className={cn(
              'rounded px-2 py-0.5 text-xs font-semibold',
              outcome === 'YES'
                ? 'bg-green-100 text-green-700 dark:bg-green-900/30 dark:text-green-400'
                : 'bg-red-100 text-red-700 dark:bg-red-900/30 dark:text-red-400'
            )}
          >
            {outcome}
          </span>
          <span className="text-gray-500">
            {formatShares(shares)} shares @ {formatCurrency(averageCost)}
          </span>
        </div>
        <div className="mt-1 text-xs text-gray-400">
          Current: {formatCurrency(currentPrice)} | Ends: {market.endDate}
        </div>
      </div>

      {/* P&L */}
      <div className="text-right">
        <div
          className={cn(
            'text-lg font-semibold',
            isProfit ? 'text-green-600' : 'text-red-600'
          )}
        >
          {isProfit ? '+' : ''}{formatCurrency(pnl)}
        </div>
        <div
          className={cn(
            'text-sm',
            isProfit ? 'text-green-500' : 'text-red-500'
          )}
        >
          {isProfit ? '+' : ''}{formatPercentage(pnlPercentage)}
        </div>
      </div>

      {/* Actions */}
      <div className="flex gap-2">
        <button
          onClick={onSell}
          className="rounded-lg bg-gray-100 px-3 py-2 text-sm font-medium text-gray-700
                     transition-colors hover:bg-gray-200 dark:bg-gray-800 dark:text-gray-300"
        >
          Sell
        </button>
      </div>
    </div>
  );
};
```

### 5. Wallet Integration Components

#### ConnectButton Component

```typescript
// components/wallet/ConnectButton/ConnectButton.tsx
import { FC } from 'react';
import { useAccount, useDisconnect } from 'wagmi';
import { ConnectButton as RainbowConnectButton } from '@rainbow-me/rainbowkit';
import { formatAddress } from '@/lib/utils/formatting';
import { AccountDropdown } from './AccountDropdown';

interface ConnectButtonProps {
  variant?: 'default' | 'compact';
}

export const ConnectButton: FC<ConnectButtonProps> = ({ variant = 'default' }) => {
  return (
    <RainbowConnectButton.Custom>
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
          <div
            {...(!ready && {
              'aria-hidden': true,
              style: {
                opacity: 0,
                pointerEvents: 'none',
                userSelect: 'none',
              },
            })}
          >
            {(() => {
              if (!connected) {
                return (
                  <button
                    onClick={openConnectModal}
                    className="rounded-lg bg-blue-600 px-4 py-2 text-sm font-semibold
                               text-white transition-colors hover:bg-blue-700"
                  >
                    Connect Wallet
                  </button>
                );
              }

              if (chain.unsupported) {
                return (
                  <button
                    onClick={openChainModal}
                    className="rounded-lg bg-red-600 px-4 py-2 text-sm font-semibold
                               text-white transition-colors hover:bg-red-700"
                  >
                    Wrong Network
                  </button>
                );
              }

              return (
                <AccountDropdown
                  account={account}
                  chain={chain}
                  onOpenAccountModal={openAccountModal}
                  onOpenChainModal={openChainModal}
                />
              );
            })()}
          </div>
        );
      }}
    </RainbowConnectButton.Custom>
  );
};
```

---

## Web3 Integration Patterns

### Wallet Configuration

```typescript
// lib/web3/config.ts
import { getDefaultConfig } from '@rainbow-me/rainbowkit';
import { polygon, polygonMumbai, mainnet } from 'wagmi/chains';
import { http } from 'viem';

export const config = getDefaultConfig({
  appName: 'Prediction Market',
  projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
  chains: [polygon, polygonMumbai, mainnet],
  transports: {
    [polygon.id]: http(process.env.NEXT_PUBLIC_POLYGON_RPC_URL),
    [polygonMumbai.id]: http(process.env.NEXT_PUBLIC_MUMBAI_RPC_URL),
    [mainnet.id]: http(process.env.NEXT_PUBLIC_MAINNET_RPC_URL),
  },
  ssr: true,
});

// Contract addresses by chain
export const CONTRACT_ADDRESSES = {
  [polygon.id]: {
    ctfExchange: '0x...',
    negRiskAdapter: '0x...',
    negRiskExchange: '0x...',
    usdc: '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174',
  },
  [polygonMumbai.id]: {
    ctfExchange: '0x...',
    negRiskAdapter: '0x...',
    negRiskExchange: '0x...',
    usdc: '0x...',
  },
} as const;
```

### Transaction Hook Pattern

```typescript
// hooks/useTransaction.ts
import { useState, useCallback } from 'react';
import { useAccount, usePublicClient, useWalletClient } from 'wagmi';
import { Hash, TransactionReceipt } from 'viem';

interface TransactionState {
  status: 'idle' | 'pending' | 'confirming' | 'confirmed' | 'error';
  hash?: Hash;
  receipt?: TransactionReceipt;
  error?: Error;
}

export function useTransaction() {
  const { address } = useAccount();
  const publicClient = usePublicClient();
  const { data: walletClient } = useWalletClient();

  const [state, setState] = useState<TransactionState>({ status: 'idle' });

  const execute = useCallback(
    async (
      transactionFn: () => Promise<Hash>,
      options?: {
        onSuccess?: (receipt: TransactionReceipt) => void;
        onError?: (error: Error) => void;
      }
    ) => {
      if (!walletClient || !publicClient) {
        throw new Error('Wallet not connected');
      }

      setState({ status: 'pending' });

      try {
        // Execute the transaction
        const hash = await transactionFn();
        setState({ status: 'confirming', hash });

        // Wait for confirmation
        const receipt = await publicClient.waitForTransactionReceipt({
          hash,
          confirmations: 1,
        });

        setState({ status: 'confirmed', hash, receipt });
        options?.onSuccess?.(receipt);

        return receipt;
      } catch (error) {
        const err = error instanceof Error ? error : new Error('Transaction failed');
        setState({ status: 'error', error: err });
        options?.onError?.(err);
        throw err;
      }
    },
    [walletClient, publicClient]
  );

  const reset = useCallback(() => {
    setState({ status: 'idle' });
  }, []);

  return {
    ...state,
    execute,
    reset,
    isIdle: state.status === 'idle',
    isPending: state.status === 'pending',
    isConfirming: state.status === 'confirming',
    isConfirmed: state.status === 'confirmed',
    isError: state.status === 'error',
    isLoading: state.status === 'pending' || state.status === 'confirming',
  };
}
```

### Contract Interaction Hooks

```typescript
// hooks/useCreateOrder.ts
import { useCallback } from 'react';
import { useAccount, usePublicClient, useWalletClient } from 'wagmi';
import { parseUnits, encodeFunctionData } from 'viem';
import { useTransaction } from './useTransaction';
import { CTF_EXCHANGE_ABI } from '@/lib/web3/abis';
import { CONTRACT_ADDRESSES } from '@/lib/web3/config';

interface CreateOrderParams {
  marketId: string;
  outcome: 'YES' | 'NO';
  orderType: 'market' | 'limit';
  amount: number;
  limitPrice?: number;
  slippage: number;
}

export function useCreateOrder() {
  const { address, chain } = useAccount();
  const publicClient = usePublicClient();
  const { data: walletClient } = useWalletClient();
  const transaction = useTransaction();

  const createOrder = useCallback(
    async (params: CreateOrderParams) => {
      if (!walletClient || !publicClient || !address || !chain) {
        throw new Error('Wallet not connected');
      }

      const contracts = CONTRACT_ADDRESSES[chain.id];
      if (!contracts) {
        throw new Error('Unsupported chain');
      }

      // Build order parameters
      const tokenId = params.outcome === 'YES'
        ? params.marketId // YES token ID
        : `${params.marketId}_NO`; // NO token ID

      const amountInWei = parseUnits(params.amount.toString(), 6); // USDC has 6 decimals

      // For limit orders, use the specified price; for market orders, use current + slippage
      const pricePerShare = params.orderType === 'limit' && params.limitPrice
        ? parseUnits(params.limitPrice.toString(), 6)
        : parseUnits('1', 6); // Market order fills at best available

      return transaction.execute(async () => {
        // First, approve USDC spending if needed
        const allowance = await publicClient.readContract({
          address: contracts.usdc,
          abi: ERC20_ABI,
          functionName: 'allowance',
          args: [address, contracts.ctfExchange],
        });

        if (allowance < amountInWei) {
          const approvalHash = await walletClient.writeContract({
            address: contracts.usdc,
            abi: ERC20_ABI,
            functionName: 'approve',
            args: [contracts.ctfExchange, amountInWei],
          });

          await publicClient.waitForTransactionReceipt({ hash: approvalHash });
        }

        // Place the order
        const hash = await walletClient.writeContract({
          address: contracts.ctfExchange,
          abi: CTF_EXCHANGE_ABI,
          functionName: 'fillOrder',
          args: [
            tokenId,
            amountInWei,
            pricePerShare,
            params.orderType === 'market' ? 0n : 1n, // Order type flag
          ],
        });

        return hash;
      });
    },
    [walletClient, publicClient, address, chain, transaction]
  );

  return {
    createOrder,
    ...transaction,
  };
}
```

### MagicLink (Email) Authentication Integration

```typescript
// lib/web3/magicConnector.ts
import { Magic } from 'magic-sdk';
import { createConnector } from 'wagmi';

export function magicConnector() {
  let magicInstance: Magic | null = null;

  return createConnector((config) => ({
    id: 'magic',
    name: 'Email',
    type: 'magic' as const,

    async connect({ chainId } = {}) {
      const magic = getMagic();

      // Check if already logged in
      const isLoggedIn = await magic.user.isLoggedIn();

      if (!isLoggedIn) {
        // Show Magic Link modal for email login
        await magic.auth.loginWithMagicLink({
          email: '', // Will be collected in modal
        });
      }

      const userInfo = await magic.user.getInfo();
      const provider = await magic.wallet.getProvider();

      return {
        accounts: [userInfo.publicAddress as `0x${string}`],
        chainId: chainId ?? polygon.id,
      };
    },

    async disconnect() {
      const magic = getMagic();
      await magic.user.logout();
    },

    async getAccounts() {
      const magic = getMagic();
      const userInfo = await magic.user.getInfo();
      return [userInfo.publicAddress as `0x${string}`];
    },

    async getChainId() {
      return polygon.id;
    },

    async getProvider() {
      const magic = getMagic();
      return magic.wallet.getProvider();
    },

    async isAuthorized() {
      const magic = getMagic();
      return magic.user.isLoggedIn();
    },

    onAccountsChanged() {},
    onChainChanged() {},
    onDisconnect() {},
  }));

  function getMagic() {
    if (!magicInstance) {
      magicInstance = new Magic(process.env.NEXT_PUBLIC_MAGIC_PUBLISHABLE_KEY!, {
        network: {
          rpcUrl: process.env.NEXT_PUBLIC_POLYGON_RPC_URL!,
          chainId: polygon.id,
        },
      });
    }
    return magicInstance;
  }
}
```

---

## State Management

### Zustand Store Pattern

```typescript
// stores/useMarketStore.ts
import { create } from 'zustand';
import { devtools, subscribeWithSelector } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';
import { Market, MarketFilters } from '@/types/market';

interface MarketState {
  // Data
  markets: Market[];
  selectedMarket: Market | null;

  // Filters
  filters: MarketFilters;
  searchQuery: string;
  category: string;
  sortBy: 'volume' | 'newest' | 'ending_soon' | 'popular';

  // UI State
  viewMode: 'grid' | 'list';
  isLoading: boolean;
  error: string | null;

  // Actions
  setMarkets: (markets: Market[]) => void;
  setSelectedMarket: (market: Market | null) => void;
  setFilters: (filters: Partial<MarketFilters>) => void;
  setSearchQuery: (query: string) => void;
  setCategory: (category: string) => void;
  setSortBy: (sort: MarketState['sortBy']) => void;
  setViewMode: (mode: 'grid' | 'list') => void;
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
  reset: () => void;
}

const initialState = {
  markets: [],
  selectedMarket: null,
  filters: {},
  searchQuery: '',
  category: 'all',
  sortBy: 'popular' as const,
  viewMode: 'grid' as const,
  isLoading: false,
  error: null,
};

export const useMarketStore = create<MarketState>()(
  devtools(
    subscribeWithSelector(
      immer((set) => ({
        ...initialState,

        setMarkets: (markets) =>
          set((state) => {
            state.markets = markets;
          }),

        setSelectedMarket: (market) =>
          set((state) => {
            state.selectedMarket = market;
          }),

        setFilters: (filters) =>
          set((state) => {
            state.filters = { ...state.filters, ...filters };
          }),

        setSearchQuery: (query) =>
          set((state) => {
            state.searchQuery = query;
          }),

        setCategory: (category) =>
          set((state) => {
            state.category = category;
          }),

        setSortBy: (sortBy) =>
          set((state) => {
            state.sortBy = sortBy;
          }),

        setViewMode: (mode) =>
          set((state) => {
            state.viewMode = mode;
          }),

        setLoading: (loading) =>
          set((state) => {
            state.isLoading = loading;
          }),

        setError: (error) =>
          set((state) => {
            state.error = error;
          }),

        reset: () => set(initialState),
      }))
    ),
    { name: 'market-store' }
  )
);
```

### TanStack Query Configuration

```typescript
// lib/api/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60, // 1 minute
      gcTime: 1000 * 60 * 5, // 5 minutes (formerly cacheTime)
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: 1,
    },
  },
});

// Query key factory
export const queryKeys = {
  markets: {
    all: ['markets'] as const,
    lists: () => [...queryKeys.markets.all, 'list'] as const,
    list: (filters: MarketFilters) => [...queryKeys.markets.lists(), filters] as const,
    details: () => [...queryKeys.markets.all, 'detail'] as const,
    detail: (id: string) => [...queryKeys.markets.details(), id] as const,
    priceHistory: (id: string, timeframe: string) =>
      [...queryKeys.markets.detail(id), 'priceHistory', timeframe] as const,
    orderBook: (id: string, outcome: string) =>
      [...queryKeys.markets.detail(id), 'orderBook', outcome] as const,
  },
  positions: {
    all: ['positions'] as const,
    list: (address: string) => [...queryKeys.positions.all, address] as const,
  },
  orders: {
    all: ['orders'] as const,
    list: (address: string) => [...queryKeys.orders.all, address] as const,
    pending: (address: string) => [...queryKeys.orders.list(address), 'pending'] as const,
  },
} as const;
```

### React Query Hook Example

```typescript
// hooks/useMarkets.ts
import { useQuery, useInfiniteQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { queryKeys } from '@/lib/api/queryClient';
import { fetchMarkets, fetchMarketDetail } from '@/lib/api/markets';
import { MarketFilters, Market } from '@/types/market';

export function useMarkets(filters: MarketFilters) {
  return useInfiniteQuery({
    queryKey: queryKeys.markets.list(filters),
    queryFn: ({ pageParam = 0 }) => fetchMarkets({ ...filters, offset: pageParam }),
    getNextPageParam: (lastPage, pages) => {
      if (lastPage.markets.length < lastPage.limit) return undefined;
      return pages.length * lastPage.limit;
    },
    initialPageParam: 0,
  });
}

export function useMarketDetail(marketId: string) {
  return useQuery({
    queryKey: queryKeys.markets.detail(marketId),
    queryFn: () => fetchMarketDetail(marketId),
    enabled: !!marketId,
  });
}

export function usePrefetchMarket() {
  const queryClient = useQueryClient();

  return (marketId: string) => {
    queryClient.prefetchQuery({
      queryKey: queryKeys.markets.detail(marketId),
      queryFn: () => fetchMarketDetail(marketId),
      staleTime: 1000 * 60 * 5, // 5 minutes
    });
  };
}
```

---

## Real-Time Data Architecture

### WebSocket Provider

```typescript
// providers/WebSocketProvider.tsx
import { createContext, useContext, useEffect, useRef, useState, ReactNode } from 'react';

interface WebSocketContextValue {
  subscribe: (channel: string, callback: (data: any) => void) => () => void;
  isConnected: boolean;
}

const WebSocketContext = createContext<WebSocketContextValue | null>(null);

export function WebSocketProvider({ children }: { children: ReactNode }) {
  const wsRef = useRef<WebSocket | null>(null);
  const subscriptionsRef = useRef<Map<string, Set<(data: any) => void>>>(new Map());
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    const connect = () => {
      const ws = new WebSocket(process.env.NEXT_PUBLIC_WS_URL!);

      ws.onopen = () => {
        setIsConnected(true);
        // Resubscribe to all channels
        subscriptionsRef.current.forEach((_, channel) => {
          ws.send(JSON.stringify({ type: 'subscribe', channel }));
        });
      };

      ws.onclose = () => {
        setIsConnected(false);
        // Reconnect after delay
        setTimeout(connect, 3000);
      };

      ws.onmessage = (event) => {
        const message = JSON.parse(event.data);
        const { channel, data } = message;

        const callbacks = subscriptionsRef.current.get(channel);
        callbacks?.forEach((callback) => callback(data));
      };

      ws.onerror = (error) => {
        console.error('WebSocket error:', error);
      };

      wsRef.current = ws;
    };

    connect();

    return () => {
      wsRef.current?.close();
    };
  }, []);

  const subscribe = (channel: string, callback: (data: any) => void) => {
    // Add to local subscriptions
    if (!subscriptionsRef.current.has(channel)) {
      subscriptionsRef.current.set(channel, new Set());
    }
    subscriptionsRef.current.get(channel)!.add(callback);

    // Subscribe on WebSocket
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify({ type: 'subscribe', channel }));
    }

    // Return unsubscribe function
    return () => {
      const callbacks = subscriptionsRef.current.get(channel);
      callbacks?.delete(callback);

      if (callbacks?.size === 0) {
        subscriptionsRef.current.delete(channel);
        wsRef.current?.send(JSON.stringify({ type: 'unsubscribe', channel }));
      }
    };
  };

  return (
    <WebSocketContext.Provider value={{ subscribe, isConnected }}>
      {children}
    </WebSocketContext.Provider>
  );
}

export function useWebSocket() {
  const context = useContext(WebSocketContext);
  if (!context) {
    throw new Error('useWebSocket must be used within a WebSocketProvider');
  }
  return context;
}
```

### Real-Time Price Hook

```typescript
// hooks/useRealtimePrice.ts
import { useEffect, useState, useCallback } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { useWebSocket } from '@/providers/WebSocketProvider';
import { queryKeys } from '@/lib/api/queryClient';

interface PriceUpdate {
  marketId: string;
  yesPrice: number;
  noPrice: number;
  timestamp: number;
}

export function useRealtimePrice(marketId: string) {
  const { subscribe, isConnected } = useWebSocket();
  const queryClient = useQueryClient();
  const [price, setPrice] = useState<PriceUpdate | null>(null);

  useEffect(() => {
    if (!marketId) return;

    const unsubscribe = subscribe(`market:${marketId}:price`, (data: PriceUpdate) => {
      setPrice(data);

      // Optimistically update the query cache
      queryClient.setQueryData(
        queryKeys.markets.detail(marketId),
        (old: any) => old ? {
          ...old,
          yesPrice: data.yesPrice,
          noPrice: data.noPrice,
        } : old
      );
    });

    return unsubscribe;
  }, [marketId, subscribe, queryClient]);

  return {
    price,
    isConnected,
  };
}
```

---

## Responsive Design System

### Breakpoint Configuration

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: [
    './src/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    screens: {
      'xs': '475px',
      'sm': '640px',
      'md': '768px',
      'lg': '1024px',
      'xl': '1280px',
      '2xl': '1536px',
    },
    extend: {
      colors: {
        // Brand colors
        primary: {
          50: '#eff6ff',
          100: '#dbeafe',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        },
        // Semantic colors
        success: '#10b981',
        warning: '#f59e0b',
        error: '#ef4444',
        // Trading colors
        bid: '#22c55e',
        ask: '#ef4444',
      },
      spacing: {
        'safe-top': 'env(safe-area-inset-top)',
        'safe-bottom': 'env(safe-area-inset-bottom)',
        'safe-left': 'env(safe-area-inset-left)',
        'safe-right': 'env(safe-area-inset-right)',
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
};

export default config;
```

### Responsive Layout Component

```typescript
// components/layout/ResponsiveLayout/ResponsiveLayout.tsx
import { FC, ReactNode } from 'react';
import { useMediaQuery } from '@/hooks/useMediaQuery';
import { Header } from '../Header';
import { Sidebar } from '../Sidebar';
import { MobileNav } from '../MobileNav';
import { BottomSheet } from '@/components/ui/BottomSheet';

interface ResponsiveLayoutProps {
  children: ReactNode;
  sidebar?: ReactNode;
  showSidebar?: boolean;
}

export const ResponsiveLayout: FC<ResponsiveLayoutProps> = ({
  children,
  sidebar,
  showSidebar = true,
}) => {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(max-width: 1024px)');

  return (
    <div className="min-h-screen bg-gray-50 dark:bg-gray-950">
      <Header />

      <div className="flex">
        {/* Desktop Sidebar */}
        {!isTablet && showSidebar && sidebar && (
          <aside className="sticky top-16 h-[calc(100vh-4rem)] w-80 shrink-0
                          overflow-y-auto border-r border-gray-200 bg-white
                          dark:border-gray-800 dark:bg-gray-900">
            {sidebar}
          </aside>
        )}

        {/* Main Content */}
        <main className="flex-1 px-4 py-6 md:px-6 lg:px-8">
          <div className="mx-auto max-w-7xl">
            {children}
          </div>
        </main>
      </div>

      {/* Mobile Bottom Navigation */}
      {isMobile && <MobileNav />}

      {/* Mobile Sidebar as Bottom Sheet */}
      {isMobile && showSidebar && sidebar && (
        <BottomSheet>{sidebar}</BottomSheet>
      )}
    </div>
  );
};
```

### Mobile Trading Interface

```typescript
// components/trading/MobileTradingSheet/MobileTradingSheet.tsx
import { FC } from 'react';
import { Sheet, SheetContent, SheetTrigger } from '@/components/ui/Sheet';
import { TradingPanel } from '../TradingPanel';
import { Market } from '@/types/market';

interface MobileTradingSheetProps {
  market: Market;
  trigger: ReactNode;
}

export const MobileTradingSheet: FC<MobileTradingSheetProps> = ({
  market,
  trigger,
}) => {
  return (
    <Sheet>
      <SheetTrigger asChild>{trigger}</SheetTrigger>
      <SheetContent
        side="bottom"
        className="h-[85vh] rounded-t-2xl pb-safe-bottom"
      >
        <div className="mx-auto mb-4 h-1.5 w-12 rounded-full bg-gray-300" />
        <div className="overflow-y-auto">
          <TradingPanel market={market} />
        </div>
      </SheetContent>
    </Sheet>
  );
};

// Fixed bottom trading bar for mobile
export const MobileTradingBar: FC<{ market: Market }> = ({ market }) => {
  return (
    <div className="fixed inset-x-0 bottom-0 z-50 border-t border-gray-200
                    bg-white pb-safe-bottom dark:border-gray-800 dark:bg-gray-900
                    md:hidden">
      <div className="flex items-center justify-between p-4">
        <div>
          <div className="text-sm text-gray-500">Current Price</div>
          <div className="text-lg font-bold text-green-600">
            {formatPrice(market.yesPrice)} YES
          </div>
        </div>

        <MobileTradingSheet
          market={market}
          trigger={
            <button className="rounded-lg bg-blue-600 px-6 py-3 font-semibold text-white">
              Trade
            </button>
          }
        />
      </div>
    </div>
  );
};
```

---

## Performance Optimization

### Code Splitting Strategy

```typescript
// app/(main)/markets/[marketId]/page.tsx
import { Suspense } from 'react';
import dynamic from 'next/dynamic';
import { MarketHeader } from '@/components/market/MarketHeader';
import { LoadingSkeleton } from '@/components/shared/LoadingStates';

// Lazy load heavy components
const PriceChart = dynamic(
  () => import('@/components/market/PriceChart').then((mod) => mod.PriceChart),
  {
    loading: () => <LoadingSkeleton variant="chart" />,
    ssr: false, // Charts don't need SSR
  }
);

const OrderBook = dynamic(
  () => import('@/components/market/OrderBook').then((mod) => mod.OrderBook),
  {
    loading: () => <LoadingSkeleton variant="orderbook" />,
    ssr: false,
  }
);

const TradingPanel = dynamic(
  () => import('@/components/trading/TradingPanel').then((mod) => mod.TradingPanel),
  {
    loading: () => <LoadingSkeleton variant="trading" />,
  }
);

export default async function MarketDetailPage({
  params,
}: {
  params: { marketId: string };
}) {
  const market = await fetchMarket(params.marketId);

  return (
    <div className="grid gap-6 lg:grid-cols-3">
      <div className="lg:col-span-2">
        <MarketHeader market={market} />

        <Suspense fallback={<LoadingSkeleton variant="chart" />}>
          <PriceChart marketId={market.id} />
        </Suspense>

        <Suspense fallback={<LoadingSkeleton variant="orderbook" />}>
          <OrderBook marketId={market.id} outcome="YES" />
        </Suspense>
      </div>

      <div className="lg:col-span-1">
        <Suspense fallback={<LoadingSkeleton variant="trading" />}>
          <TradingPanel market={market} />
        </Suspense>
      </div>
    </div>
  );
}
```

### Optimistic Updates

```typescript
// hooks/useOptimisticOrder.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { queryKeys } from '@/lib/api/queryClient';
import { submitOrder } from '@/lib/api/orders';
import { Order, Position } from '@/types';

export function useOptimisticOrder() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: submitOrder,

    onMutate: async (newOrder) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({
        queryKey: queryKeys.positions.all,
      });

      // Snapshot previous value
      const previousPositions = queryClient.getQueryData(
        queryKeys.positions.list(newOrder.userAddress)
      );

      // Optimistically update positions
      queryClient.setQueryData(
        queryKeys.positions.list(newOrder.userAddress),
        (old: Position[] = []) => {
          const existingPosition = old.find(
            (p) => p.marketId === newOrder.marketId && p.outcome === newOrder.outcome
          );

          if (existingPosition) {
            return old.map((p) =>
              p.id === existingPosition.id
                ? {
                    ...p,
                    shares: p.shares + newOrder.shares,
                    averageCost:
                      (p.averageCost * p.shares + newOrder.price * newOrder.shares) /
                      (p.shares + newOrder.shares),
                  }
                : p
            );
          }

          return [
            ...old,
            {
              id: `temp-${Date.now()}`,
              marketId: newOrder.marketId,
              outcome: newOrder.outcome,
              shares: newOrder.shares,
              averageCost: newOrder.price,
              status: 'pending',
            },
          ];
        }
      );

      return { previousPositions };
    },

    onError: (err, newOrder, context) => {
      // Rollback on error
      if (context?.previousPositions) {
        queryClient.setQueryData(
          queryKeys.positions.list(newOrder.userAddress),
          context.previousPositions
        );
      }
    },

    onSettled: (data, error, variables) => {
      // Refetch to ensure consistency
      queryClient.invalidateQueries({
        queryKey: queryKeys.positions.list(variables.userAddress),
      });
    },
  });
}
```

### Virtual List for Large Data Sets

```typescript
// components/portfolio/VirtualizedPositionsList/VirtualizedPositionsList.tsx
import { FC, useCallback } from 'react';
import { useVirtualizer } from '@tanstack/react-virtual';
import { PositionCard } from '../PositionCard';
import { Position } from '@/types/position';

interface VirtualizedPositionsListProps {
  positions: Position[];
  onSell: (position: Position) => void;
}

export const VirtualizedPositionsList: FC<VirtualizedPositionsListProps> = ({
  positions,
  onSell,
}) => {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: positions.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 120, // Estimated row height
    overscan: 5,
  });

  return (
    <div
      ref={parentRef}
      className="h-[600px] overflow-auto"
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          width: '100%',
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map((virtualItem) => {
          const position = positions[virtualItem.index];
          return (
            <div
              key={virtualItem.key}
              style={{
                position: 'absolute',
                top: 0,
                left: 0,
                width: '100%',
                height: `${virtualItem.size}px`,
                transform: `translateY(${virtualItem.start}px)`,
              }}
            >
              <PositionCard
                position={position}
                onSell={() => onSell(position)}
              />
            </div>
          );
        })}
      </div>
    </div>
  );
};
```

### Image Optimization

```typescript
// components/market/MarketImage/MarketImage.tsx
import { FC, useState } from 'react';
import Image from 'next/image';
import { cn } from '@/lib/utils';

interface MarketImageProps {
  src: string;
  alt: string;
  priority?: boolean;
  className?: string;
}

export const MarketImage: FC<MarketImageProps> = ({
  src,
  alt,
  priority = false,
  className,
}) => {
  const [isLoading, setIsLoading] = useState(true);
  const [hasError, setHasError] = useState(false);

  return (
    <div className={cn('relative overflow-hidden bg-gray-100', className)}>
      {hasError ? (
        <div className="flex h-full w-full items-center justify-center bg-gray-200">
          <span className="text-4xl">📊</span>
        </div>
      ) : (
        <>
          <Image
            src={src}
            alt={alt}
            fill
            priority={priority}
            sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            className={cn(
              'object-cover transition-opacity duration-300',
              isLoading ? 'opacity-0' : 'opacity-100'
            )}
            onLoad={() => setIsLoading(false)}
            onError={() => setHasError(true)}
          />
          {isLoading && (
            <div className="absolute inset-0 animate-pulse bg-gray-200" />
          )}
        </>
      )}
    </div>
  );
};
```

---

## Testing Strategy

### Component Testing with Vitest

```typescript
// components/trading/TradingPanel/TradingPanel.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { TradingPanel } from './TradingPanel';
import { createWrapper } from '@/test/utils';

const mockMarket = {
  id: '1',
  title: 'Test Market',
  yesPrice: 0.65,
  noPrice: 0.35,
};

describe('TradingPanel', () => {
  it('renders outcome selector with correct prices', () => {
    render(<TradingPanel market={mockMarket} />, {
      wrapper: createWrapper(),
    });

    expect(screen.getByText('Yes')).toBeInTheDocument();
    expect(screen.getByText('65¢')).toBeInTheDocument();
    expect(screen.getByText('No')).toBeInTheDocument();
    expect(screen.getByText('35¢')).toBeInTheDocument();
  });

  it('calculates payout correctly', async () => {
    const user = userEvent.setup();
    render(<TradingPanel market={mockMarket} />, {
      wrapper: createWrapper(),
    });

    const amountInput = screen.getByPlaceholderText('Enter amount');
    await user.type(amountInput, '100');

    // At 65¢ per share, $100 buys ~153.8 shares
    await waitFor(() => {
      expect(screen.getByText(/153\.8/)).toBeInTheDocument();
    });
  });

  it('shows connect wallet button when not connected', () => {
    render(<TradingPanel market={mockMarket} />, {
      wrapper: createWrapper({ connected: false }),
    });

    expect(screen.getByText('Connect Wallet')).toBeInTheDocument();
  });

  it('opens confirmation modal on submit', async () => {
    const user = userEvent.setup();
    render(<TradingPanel market={mockMarket} />, {
      wrapper: createWrapper({ connected: true }),
    });

    const amountInput = screen.getByPlaceholderText('Enter amount');
    await user.type(amountInput, '100');

    const buyButton = screen.getByRole('button', { name: /buy yes/i });
    await user.click(buyButton);

    await waitFor(() => {
      expect(screen.getByRole('dialog')).toBeInTheDocument();
    });
  });
});
```

### E2E Testing with Playwright

```typescript
// e2e/trading.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Trading Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Mock wallet connection
    await page.addInitScript(() => {
      window.ethereum = {
        isMetaMask: true,
        request: async ({ method }) => {
          if (method === 'eth_accounts') {
            return ['0x1234567890123456789012345678901234567890'];
          }
          if (method === 'eth_chainId') {
            return '0x89'; // Polygon
          }
        },
      };
    });
  });

  test('completes a market order', async ({ page }) => {
    await page.goto('/markets/test-market-id');

    // Select YES outcome
    await page.click('[data-testid="outcome-yes"]');

    // Enter amount
    await page.fill('[data-testid="amount-input"]', '50');

    // Check calculated shares
    await expect(page.locator('[data-testid="calculated-shares"]')).toContainText(/\d+\.\d+/);

    // Submit order
    await page.click('[data-testid="submit-order"]');

    // Confirm in modal
    await page.click('[data-testid="confirm-order"]');

    // Wait for transaction
    await expect(page.locator('[data-testid="transaction-pending"]')).toBeVisible();

    // Check success
    await expect(page.locator('[data-testid="transaction-success"]')).toBeVisible({
      timeout: 30000,
    });
  });

  test('handles insufficient balance', async ({ page }) => {
    await page.goto('/markets/test-market-id');

    // Enter amount larger than balance
    await page.fill('[data-testid="amount-input"]', '1000000');

    // Check for error message
    await expect(page.locator('[data-testid="insufficient-balance"]')).toBeVisible();

    // Submit button should be disabled
    await expect(page.locator('[data-testid="submit-order"]')).toBeDisabled();
  });
});
```

---

## Recommended Libraries Summary

### Core

| Library | Version | Purpose |
|---------|---------|---------|
| next | ^14.0 | React framework |
| react | ^18.0 | UI library |
| typescript | ^5.0 | Type safety |

### Web3

| Library | Version | Purpose |
|---------|---------|---------|
| wagmi | ^2.0 | React hooks for Ethereum |
| viem | ^2.0 | Ethereum library |
| @rainbow-me/rainbowkit | ^2.0 | Wallet UI |
| magic-sdk | ^21.0 | Email authentication |

### State & Data

| Library | Version | Purpose |
|---------|---------|---------|
| zustand | ^4.0 | Global state |
| @tanstack/react-query | ^5.0 | Server state |
| @tanstack/react-virtual | ^3.0 | Virtual lists |

### UI

| Library | Version | Purpose |
|---------|---------|---------|
| tailwindcss | ^3.4 | Styling |
| @radix-ui/react-* | latest | Headless components |
| framer-motion | ^10.0 | Animations |
| class-variance-authority | ^0.7 | Variant management |

### Charts

| Library | Version | Purpose |
|---------|---------|---------|
| lightweight-charts | ^4.0 | Price charts |
| recharts | ^2.0 | General charts |
| d3 | ^7.0 | Custom visualizations |

### Forms & Validation

| Library | Version | Purpose |
|---------|---------|---------|
| react-hook-form | ^7.0 | Form handling |
| zod | ^3.0 | Schema validation |
| @hookform/resolvers | ^3.0 | Zod integration |

### Testing

| Library | Version | Purpose |
|---------|---------|---------|
| vitest | ^1.0 | Unit testing |
| @testing-library/react | ^14.0 | Component testing |
| playwright | ^1.40 | E2E testing |
| msw | ^2.0 | API mocking |

---

## Security Considerations

### Frontend Security Checklist

1. **Input Validation**: Always validate user inputs on both client and server
2. **XSS Prevention**: Use React's built-in escaping, avoid `dangerouslySetInnerHTML`
3. **CSRF Protection**: Implement CSRF tokens for state-changing operations
4. **Secure Storage**: Never store private keys in localStorage
5. **Transaction Verification**: Always show users what they're signing
6. **Rate Limiting**: Implement client-side rate limiting for API calls
7. **Content Security Policy**: Configure strict CSP headers

### Web3 Security

```typescript
// lib/web3/security.ts

// Verify transaction details before signing
export function verifyTransaction(tx: TransactionRequest): boolean {
  // Check recipient is a known contract
  const knownContracts = Object.values(CONTRACT_ADDRESSES).flat();
  if (!knownContracts.includes(tx.to?.toLowerCase())) {
    console.warn('Transaction to unknown contract');
    return false;
  }

  // Check for reasonable gas limits
  if (tx.gasLimit && BigInt(tx.gasLimit) > BigInt(1000000)) {
    console.warn('Unusually high gas limit');
    return false;
  }

  return true;
}

// Decode and display transaction data to user
export function decodeTransactionData(
  data: `0x${string}`,
  abi: Abi
): DecodedTransaction {
  // Use viem to decode function call
  const decoded = decodeFunctionData({ abi, data });
  return {
    functionName: decoded.functionName,
    args: decoded.args,
    humanReadable: formatTransactionForDisplay(decoded),
  };
}
```

---

## Deployment Checklist

### Pre-Deployment

- [ ] Environment variables configured
- [ ] API endpoints updated for production
- [ ] Contract addresses verified for target network
- [ ] Error tracking (Sentry) configured
- [ ] Analytics integrated
- [ ] Performance monitoring enabled

### Build Optimization

- [ ] Bundle size analyzed and optimized
- [ ] Images optimized and using CDN
- [ ] Fonts optimized (subset, preload)
- [ ] Critical CSS inlined
- [ ] Service worker configured (if PWA)

### Testing

- [ ] Unit tests passing
- [ ] E2E tests passing
- [ ] Cross-browser testing complete
- [ ] Mobile testing complete
- [ ] Accessibility audit passed

### Security

- [ ] Security headers configured
- [ ] CSP policy defined
- [ ] Rate limiting enabled
- [ ] WAF configured
- [ ] SSL/TLS configured

---

This architecture provides a solid foundation for building a production-ready prediction market platform. The modular component structure, robust state management, and comprehensive Web3 integration patterns ensure scalability and maintainability as the platform grows.
