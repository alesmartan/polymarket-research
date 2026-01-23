# User Flows and Wireframes
## Prediction Market Platform

**Version:** 1.0
**Last Updated:** January 2026
**Status:** Active
**Owner:** Design Team

---

## Table of Contents

1. [User Journey Maps](#1-user-journey-maps)
2. [Onboarding Flow](#2-onboarding-flow)
3. [Market Discovery Flow](#3-market-discovery-flow)
4. [Trading Flow](#4-trading-flow)
5. [Deposit/Withdrawal Flows](#5-depositwithdrawal-flows)
6. [Portfolio Management Flow](#6-portfolio-management-flow)
7. [ASCII Wireframes](#7-ascii-wireframes)

---

## 1. User Journey Maps

### Journey Map: New Casual User

**Persona:** Jamie (Casual Predictor)
**Goal:** Place first prediction on a political event
**Timeline:** First 30 minutes

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                           CASUAL USER FIRST PREDICTION JOURNEY                           │
├──────────┬──────────┬──────────┬──────────┬──────────┬──────────┬──────────┬────────────┤
│  Stage   │ Discover │  Sign Up │  Explore │  Deposit │  Trade   │  Track   │   Share    │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│          │ Sees     │ Creates  │ Browses  │ Adds     │ Places   │ Watches  │ Shares     │
│ Actions  │ market   │ account  │ markets  │ $20 via  │ first    │ position │ prediction │
│          │ on X/news│ (Google) │ by topic │ card     │ trade    │ update   │ on social  │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│          │ Curious, │ Easy,    │ Engaged, │ Slight   │ Excited, │ Checking │ Proud,     │
│ Feelings │ skeptical│ fast     │ browsing │ hesitation│ nervous  │ often    │ validated  │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│          │ What's   │ Is this  │ What does│ Is my    │ Did I do │ What do  │ Will       │
│ Thoughts │ this?    │ legit?   │ 65% mean?│ money    │ this     │ numbers  │ friends    │
│          │          │          │          │ safe?    │ right?   │ mean?    │ engage?    │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│          │ Clear    │ Google   │ Intuitive│ Trusted  │ Clear    │ Simple   │ Easy share │
│ Needs    │ value    │ sign-in  │ categories│ payment │ confirm- │ P&L      │ with       │
│          │ prop     │ option   │ & search │ icons    │ ation    │ display  │ context    │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│ Touch-   │ Social   │ Sign up  │ Home     │ Deposit  │ Market   │ Portfolio│ Share      │
│ points   │ media,   │ page     │ page,    │ modal,   │ page,    │ page     │ modal,     │
│          │ news     │          │ search   │ on-ramp  │ order    │          │ social     │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│          │ ★★★★★    │ ★★★★☆    │ ★★★★☆    │ ★★★☆☆    │ ★★★★☆    │ ★★★★☆    │ ★★★★★      │
│ Current  │ Good     │ Fast but │ Categories│ Fees    │ Works    │ Could be │ Native     │
│ State    │ viral    │ KYC slow │ work     │ unclear  │ well     │ simpler  │ sharing    │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│ Opportu- │ Better   │ Instant  │ Add      │ Show     │ Add      │ Add      │ Generate   │
│ nities   │ landing  │ KYC      │ trending │ all fees │ education│ simple   │ shareable  │
│          │ page     │          │ section  │ upfront  │ tooltips │ charts   │ images     │
└──────────┴──────────┴──────────┴──────────┴──────────┴──────────┴──────────┴────────────┘
```

### Journey Map: Power Trader

**Persona:** Alex (Sharp Trader)
**Goal:** Execute a multi-market trading strategy
**Timeline:** Daily routine

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                              POWER TRADER DAILY JOURNEY                                  │
├──────────┬──────────┬──────────┬──────────┬──────────┬──────────┬──────────┬────────────┤
│  Stage   │ Morning  │ Analysis │ Position │ Execute  │ Monitor  │ Adjust   │  Review    │
│          │ Review   │          │ Planning │          │          │          │            │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│          │ Check    │ Review   │ Calculate│ Place    │ Watch    │ Modify   │ Log daily  │
│ Actions  │ overnight│ news &   │ position │ limit    │ fills,   │ orders,  │ P&L,       │
│          │ changes  │ data     │ sizes    │ orders   │ prices   │ hedge    │ learnings  │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│          │ 5 min    │ 30 min   │ 15 min   │ 10 min   │ All day  │ As needed│ 10 min     │
│ Time     │          │          │          │          │          │          │            │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│          │ Portfolio│ Charts,  │ Spread-  │ Order    │ Positions│ Open     │ Analytics  │
│ Tools    │ view,    │ external │ sheet,   │ form,    │ list,    │ orders,  │ dashboard, │
│ Needed   │ alerts   │ sources  │ risk     │ keyboard │ alerts   │ cancel   │ export     │
│          │          │          │ calc     │ shortcuts│          │          │            │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│ Pain     │ Slow     │ No API   │ Manual   │ No bulk  │ Alert    │ Can't    │ No export  │
│ Points   │ load     │ for data │ sizing   │ orders   │ latency  │ edit     │ to CSV     │
│          │ times    │          │          │          │          │ orders   │            │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┼────────────┤
│ Feature  │ Dashboard│ Data API │ Position │ Bulk     │ Instant  │ Order    │ Analytics  │
│ Requests │ preload  │ access   │ sizer    │ orders   │ push     │ modify   │ export     │
│          │          │          │ tool     │          │ alerts   │          │            │
└──────────┴──────────┴──────────┴──────────┴──────────┴──────────┴──────────┴────────────┘
```

### Journey Map: Market Discovery to First Trade (Detailed)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        MARKET DISCOVERY → FIRST TRADE                            │
└─────────────────────────────────────────────────────────────────────────────────┘

User arrives at landing page
         │
         ▼
┌─────────────────┐
│  Browse Markets │ ──────► Can explore without account
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Find Interesting│
│     Market      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐         ┌─────────────────┐
│  View Market    │────────►│ Read resolution │
│    Details      │         │    criteria     │
└────────┬────────┘         └─────────────────┘
         │
         ▼
┌─────────────────┐
│ Click "Buy Yes" │
│  or "Buy No"    │
└────────┬────────┘
         │
         ▼
    ┌────┴────┐
    │Logged in?│
    └────┬────┘
    No   │   Yes
    │    │    │
    ▼    │    ▼
┌───────────┐ │ ┌─────────────────┐
│ Sign Up   │ │ │ Has sufficient  │
│ Modal     │ │ │    balance?     │
└─────┬─────┘ │ └────────┬────────┘
      │       │     No   │   Yes
      ▼       │     │    │    │
┌───────────┐ │     ▼    │    ▼
│ Google    │ │ ┌───────────┐ │ ┌─────────────┐
│ Sign In   │ │ │ Deposit   │ │ │ Enter Trade │
└─────┬─────┘ │ │ Prompt    │ │ │   Amount    │
      │       │ └─────┬─────┘ │ └──────┬──────┘
      │       │       │       │        │
      ▼       │       ▼       │        ▼
┌───────────┐ │ ┌───────────┐ │ ┌─────────────┐
│ Account   │ │ │ Choose    │ │ │ Review      │
│ Created   │ │ │ Method    │ │ │   Order     │
└─────┬─────┘ │ └─────┬─────┘ │ └──────┬──────┘
      │       │       │       │        │
      └───────┼───────┘       │        ▼
              │               │ ┌─────────────┐
              │               │ │  Confirm    │
              │               │ │   Trade     │
              │               │ └──────┬──────┘
              │               │        │
              │               │        ▼
              │               │ ┌─────────────┐
              │               └►│ Success!    │
              │                 │ View in     │
              │                 │ Portfolio   │
              │                 └─────────────┘
              │
              └────────────────────────────────►
```

---

## 2. Onboarding Flow

### 2.1 Flow Diagram: Wallet Connection Options

```
                              ┌─────────────────────┐
                              │   Welcome Screen    │
                              │  "Start Predicting" │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │  Choose Sign Up     │
                              │      Method         │
                              └──────────┬──────────┘
                                         │
                    ┌────────────────────┼────────────────────┐
                    │                    │                    │
                    ▼                    ▼                    ▼
         ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
         │   Google/Apple   │ │ Connect Wallet   │ │  Email/Password  │
         │    (Web2 Auth)   │ │   (Web3 Auth)    │ │   (Traditional)  │
         └────────┬─────────┘ └────────┬─────────┘ └────────┬─────────┘
                  │                    │                    │
                  ▼                    ▼                    ▼
         ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
         │ Create Arcade    │ │ Select Wallet    │ │ Email Verify     │
         │ Wallet (auto)    │ │ (MetaMask, etc)  │ │ + Password Set   │
         └────────┬─────────┘ └────────┬─────────┘ └────────┬─────────┘
                  │                    │                    │
                  │                    ▼                    │
                  │           ┌──────────────────┐          │
                  │           │  Sign Message    │          │
                  │           │  (Wallet Auth)   │          │
                  │           └────────┬─────────┘          │
                  │                    │                    │
                  └────────────────────┼────────────────────┘
                                       │
                                       ▼
                              ┌──────────────────┐
                              │  Account Created │
                              │  Show Welcome    │
                              └────────┬─────────┘
                                       │
                                       ▼
                              ┌──────────────────┐
                              │  Skip KYC for    │──────► Explore with
                              │  Browsing?       │        $0 balance
                              └────────┬─────────┘
                                       │ Want to Trade
                                       ▼
                              ┌──────────────────┐
                              │  KYC Required    │
                              │  (For Deposits)  │
                              └────────┬─────────┘
                                       │
                                       ▼
```

### 2.2 Flow Diagram: KYC Verification

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          KYC VERIFICATION FLOW                          │
└─────────────────────────────────────────────────────────────────────────┘

     ┌────────────────┐
     │ User Triggers  │ ◄── From: Deposit button, Trade attempt,
     │ KYC Flow       │        Settings, or First login
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Country        │
     │ Selection      │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐      ┌────────────────┐
     │ Supported?     │──No──► "Not Available │
     │                │       │  in Your       │
     │                │       │  Region" Error │
     └───────┬────────┘       └────────────────┘
             │ Yes
             ▼
     ┌────────────────┐
     │ Select ID Type │
     │ - Passport     │
     │ - Driver's Lic │
     │ - National ID  │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Capture/Upload │
     │ ID Document    │
     │ (Front)        │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Capture/Upload │ ◄── Skip for passports
     │ ID Document    │
     │ (Back)         │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Selfie with    │
     │ Liveness Check │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Processing...  │ ◄── ~30 seconds
     │ Verify Identity│
     └───────┬────────┘
             │
     ┌───────┴───────┐
     │               │
     ▼               ▼
┌─────────┐    ┌─────────────┐    ┌─────────────┐
│ Success │    │ Needs Review│───►│ Manual      │
│ ✓       │    │ (Edge case) │    │ Review      │
└────┬────┘    └─────────────┘    │ (24-48 hrs) │
     │                            └──────┬──────┘
     │                                   │
     ▼                                   ▼
┌────────────────┐              ┌────────────────┐
│ Unlock Full    │              │ Email When     │
│ Platform Access│              │ Complete       │
└────────────────┘              └────────────────┘
```

### 2.3 Flow Diagram: First Deposit

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           FIRST DEPOSIT FLOW                            │
└─────────────────────────────────────────────────────────────────────────┘

     ┌────────────────┐
     │ Click "Deposit"│
     │ or "Add Funds" │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐      ┌────────────────┐
     │ KYC Complete?  │──No──► Go to KYC Flow │
     │                │       └────────────────┘
     └───────┬────────┘
             │ Yes
             ▼
     ┌────────────────┐
     │ Choose Method  │
     └───────┬────────┘
             │
     ┌───────┼───────┬───────────────┐
     │       │       │               │
     ▼       ▼       ▼               ▼
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐
│ Credit  │ │ Apple   │ │ Crypto  │ │ Bank        │
│ Card    │ │ Pay /   │ │ Transfer│ │ Transfer    │
│         │ │ Google  │ │ (USDC)  │ │ (ACH/Wire)  │
└────┬────┘ └────┬────┘ └────┬────┘ └──────┬──────┘
     │           │           │             │
     ▼           ▼           ▼             ▼
┌─────────┐ ┌─────────┐ ┌─────────────┐ ┌─────────────┐
│ Enter   │ │ Native  │ │ Show Wallet │ │ Show Bank   │
│ Amount  │ │ Payment │ │ Address for │ │ Details     │
│ $10-10K │ │ Sheet   │ │ USDC Deposit│ │ (For wire)  │
└────┬────┘ └────┬────┘ └──────┬──────┘ └──────┬──────┘
     │           │             │               │
     ▼           ▼             │               │
┌─────────┐ ┌─────────┐        │               │
│ Payment │ │ Confirm │        │               │
│ Form    │ │ Payment │        │               │
│ (Stripe)│ │         │        │               │
└────┬────┘ └────┬────┘        │               │
     │           │             │               │
     └───────────┼─────────────┼───────────────┘
                 │             │
                 ▼             ▼
         ┌────────────┐ ┌────────────────┐
         │ Processing │ │ Awaiting       │
         │ ~30 sec    │ │ Blockchain     │
         │            │ │ Confirmation   │
         └─────┬──────┘ └───────┬────────┘
               │                │
               └────────┬───────┘
                        │
                        ▼
               ┌────────────────┐
               │ Funds Credited │
               │ ✓ $XX.XX added │
               │ Start Trading! │
               └────────────────┘
```

---

## 3. Market Discovery Flow

### 3.1 Flow Diagram: Market Discovery

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         MARKET DISCOVERY FLOW                           │
└─────────────────────────────────────────────────────────────────────────┘

                    ┌─────────────────┐
                    │   Home Page     │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ Featured/       │ │ Browse by       │ │ Search by       │
│ Trending        │ │ Category        │ │ Keyword         │
└────────┬────────┘ └────────┬────────┘ └────────┬────────┘
         │                   │                   │
         │                   ▼                   │
         │          ┌─────────────────┐          │
         │          │ Categories:     │          │
         │          │ - Politics      │          │
         │          │ - Sports        │          │
         │          │ - Crypto        │          │
         │          │ - Entertainment │          │
         │          │ - Science       │          │
         │          │ - Economics     │          │
         │          └────────┬────────┘          │
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Market List     │
                    │ (Filterable)    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Apply Filters:  │
                    │ - Volume (H/L)  │
                    │ - End Date      │
                    │ - Probability   │
                    │ - New Markets   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Select Market   │
                    │ Card            │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Market Detail   │
                    │ Page            │
                    └─────────────────┘
```

### 3.2 Flow Diagram: Market Detail Page

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         MARKET DETAIL PAGE FLOW                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                        MARKET DETAIL PAGE                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ Market Title: "Will X win the 2026 election?"                    │  │
│  │                                                                   │  │
│  │ Current Probability: [======65%======]  Yes: 65¢  No: 35¢       │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌──────────────────────────┐  ┌───────────────────────────────────┐  │
│  │ PRICE CHART              │  │ ORDER BOOK                        │  │
│  │                          │  │                                   │  │
│  │     /\    /\             │  │ Bids (Yes)  │  Asks (Yes)        │  │
│  │    /  \  /  \    /\      │  │ 64¢  1,234  │  65¢  2,345        │  │
│  │   /    \/    \  /  \     │  │ 63¢    890  │  66¢  1,456        │  │
│  │  /            \/         │  │ 62¢    567  │  67¢    890        │  │
│  │                          │  │                                   │  │
│  │ [1D] [1W] [1M] [All]     │  │                                   │  │
│  └──────────────────────────┘  └───────────────────────────────────┘  │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ PLACE ORDER                                          [Market][Limit]│ │
│  │                                                                    │ │
│  │  ┌───────────────────┐  ┌───────────────────┐                     │ │
│  │  │    BUY YES        │  │    BUY NO         │                     │ │
│  │  │    65¢            │  │    35¢            │                     │ │
│  │  └───────────────────┘  └───────────────────┘                     │ │
│  │                                                                    │ │
│  │  Amount: [$] [___100___] [Max]                                    │ │
│  │                                                                    │ │
│  │  You receive: 153.84 shares                                       │ │
│  │  Potential profit: $53.84 (if Yes wins)                          │ │
│  │                                                                    │ │
│  │  [        Place Order        ]                                    │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ MARKET INFO                                                       │ │
│  │                                                                    │ │
│  │ Resolution Date: March 15, 2026                                   │ │
│  │ Volume (24h): $234,567                                            │ │
│  │ Total Volume: $1,234,567                                          │ │
│  │ Liquidity: $456,789                                               │ │
│  │                                                                    │ │
│  │ Resolution Source: Official announcement from...                  │ │
│  │ Resolution Criteria: This market resolves Yes if...               │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ RELATED MARKETS                                                   │ │
│  │ • Will Y win primary? (45%)                                       │ │
│  │ • Will Z endorse? (72%)                                           │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Trading Flow

### 4.1 Flow Diagram: Buy Shares (Market Order)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    BUY SHARES - MARKET ORDER FLOW                       │
└─────────────────────────────────────────────────────────────────────────┘

     ┌────────────────┐
     │ On Market Page │
     │ Click "Buy Yes"│
     │ or "Buy No"    │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Select Outcome │
     │ [Yes] / [No]   │
     │ (Highlighted)  │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Enter Amount   │
     │ (USD or Shares)│
     │ [$] [____]     │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Real-time Calc │
     │ • Shares: XXX  │
     │ • Avg Price: $X│
     │ • Payout: $XXX │
     │ • Profit: $XX  │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐      ┌────────────────┐
     │ Check Balance  │──No──► Insufficient   │
     │ Sufficient?    │       │ Funds Modal   │
     │                │       │ [Deposit Now] │
     └───────┬────────┘       └────────────────┘
             │ Yes
             ▼
     ┌────────────────┐
     │ Slippage Check │
     │ > 2%?          │
     └───────┬────────┘
             │
     ┌───────┴───────┐
     │ Warning       │ No Warning
     ▼               ▼
┌────────────┐ ┌────────────┐
│ Show       │ │            │
│ Slippage   │ │            │
│ Warning    │ │            │
└─────┬──────┘ │            │
      │        │            │
      └────────┼────────────┘
               │
               ▼
     ┌────────────────┐
     │ Click "Confirm │
     │ Order"         │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Processing...  │
     │ (Blockchain tx)│
     └───────┬────────┘
             │
     ┌───────┴───────┐
     │               │
     ▼               ▼
┌─────────┐    ┌─────────────┐
│ Success │    │ Failed      │
│ ✓       │    │ (Show error)│
└────┬────┘    └─────────────┘
     │
     ▼
┌────────────────┐
│ Success Toast  │
│ "Bought XX     │
│ shares of Yes" │
│ [View Position]│
└────────────────┘
```

### 4.2 Flow Diagram: Limit Order

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        LIMIT ORDER FLOW                                 │
└─────────────────────────────────────────────────────────────────────────┘

     ┌────────────────┐
     │ Toggle to      │
     │ "Limit Order"  │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Select Outcome │
     │ [Yes] / [No]   │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Enter Limit    │
     │ Price (0-100¢) │
     │ [−] [63¢] [+]  │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Enter Amount   │
     │ (USD or Shares)│
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Show Order     │
     │ Preview:       │
     │ • Shares: XXX  │
     │ • Total: $XX   │
     │ • If filled at │
     │   63¢: $XX     │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Funds Reserved │
     │ from Balance   │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Submit Limit   │
     │ Order          │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Order Added to │
     │ Order Book     │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Show in "Open  │
     │ Orders" Tab    │
     │                │
     │ Options:       │
     │ [Cancel Order] │
     │ [Edit Price]   │
     └───────┬────────┘
             │
     ┌───────┴───────┐
     │               │
     ▼               ▼
┌─────────┐    ┌─────────────┐
│ Filled  │    │ Cancelled   │
│ (Push   │    │ (Funds      │
│ notify) │    │ returned)   │
└─────────┘    └─────────────┘
```

### 4.3 Flow Diagram: Sell Position

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          SELL POSITION FLOW                             │
└─────────────────────────────────────────────────────────────────────────┘

     ┌────────────────┐
     │ From Portfolio │
     │ Click "Sell"   │
     │ on Position    │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Sell Modal     │
     │ Opens          │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Current        │
     │ Position Info: │
     │ • 100 Yes @65¢ │
     │ • Cost: $65    │
     │ • Value: $70   │
     │ • P&L: +$5     │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Enter Sell     │
     │ Quantity       │
     │ [___] / 100    │
     │ [25%][50%][Max]│
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Order Type     │
     │ [Market][Limit]│
     └───────┬────────┘
             │
     ┌───────┴───────┐
     │               │
     ▼               ▼
┌─────────┐    ┌─────────────┐
│ Market: │    │ Limit:      │
│ Sell at │    │ Enter sell  │
│ 70¢     │    │ price       │
└────┬────┘    └──────┬──────┘
     │                │
     └────────┬───────┘
              │
              ▼
     ┌────────────────┐
     │ Review Sale:   │
     │ • Selling: 50  │
     │ • At: 70¢      │
     │ • Receive: $35 │
     │ • P&L: +$2.50  │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Confirm Sell   │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Processing...  │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Success!       │
     │ $35 added to   │
     │ balance        │
     └────────────────┘
```

---

## 5. Deposit/Withdrawal Flows

### 5.1 Withdrawal Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          WITHDRAWAL FLOW                                │
└─────────────────────────────────────────────────────────────────────────┘

     ┌────────────────┐
     │ Click          │
     │ "Withdraw"     │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Show Available │
     │ Balance        │
     │ $1,234.56      │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Choose Method  │
     └───────┬────────┘
             │
     ┌───────┴───────┐
     │               │
     ▼               ▼
┌─────────────┐ ┌─────────────┐
│ External    │ │ Bank        │
│ Wallet      │ │ Transfer    │
│ (Crypto)    │ │ (Fiat)      │
└──────┬──────┘ └──────┬──────┘
       │               │
       ▼               ▼
┌─────────────┐ ┌─────────────┐
│ Enter Wallet│ │ Select Saved│
│ Address     │ │ Bank Account│
│ (or paste)  │ │ or Add New  │
└──────┬──────┘ └──────┬──────┘
       │               │
       ▼               │
┌─────────────┐        │
│ Select      │        │
│ Network     │        │
│ • Polygon   │        │
│ • Ethereum  │        │
└──────┬──────┘        │
       │               │
       └───────┬───────┘
               │
               ▼
     ┌────────────────┐
     │ Enter Amount   │
     │ $[_________]   │
     │ Min: $10       │
     │ Max: $1,234.56 │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Review:        │
     │ • Amount: $500 │
     │ • Network Fee: │
     │   ~$2.50       │
     │ • You receive: │
     │   $497.50      │
     │ • ETA: ~5 min  │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ 2FA Required   │
     │ (If enabled)   │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Confirm        │
     │ Withdrawal     │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Processing...  │
     │ (Show tx hash) │
     └───────┬────────┘
             │
             ▼
     ┌────────────────┐
     │ Withdrawal     │
     │ Initiated      │
     │ Track Status ▶ │
     └────────────────┘
```

---

## 6. Portfolio Management Flow

### 6.1 Portfolio Overview Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       PORTFOLIO MANAGEMENT FLOW                         │
└─────────────────────────────────────────────────────────────────────────┘

     ┌────────────────┐
     │ Navigate to    │
     │ "Portfolio"    │
     └───────┬────────┘
             │
             ▼
     ┌────────────────────────────────────────────────────────────────┐
     │                      PORTFOLIO DASHBOARD                       │
     ├────────────────────────────────────────────────────────────────┤
     │                                                                │
     │  ┌─────────────────────────────────────────────────────────┐  │
     │  │ PORTFOLIO VALUE            TODAY'S P&L    ALL-TIME P&L  │  │
     │  │ $1,234.56                  +$45.67        +$234.56      │  │
     │  │                            (+3.8%)        (+23.4%)      │  │
     │  └─────────────────────────────────────────────────────────┘  │
     │                                                                │
     │  ┌─────────────────────────────────────────────────────────┐  │
     │  │ [Active Positions] [Open Orders] [History] [Resolved]   │  │
     │  └─────────────────────────────────────────────────────────┘  │
     │                                                                │
     │  ACTIVE POSITIONS (5)                                         │
     │  ┌─────────────────────────────────────────────────────────┐  │
     │  │ Market                 │ Side │ Qty │ Avg  │ P&L │ Act  │  │
     │  ├────────────────────────┼──────┼─────┼──────┼─────┼──────┤  │
     │  │ Will X win election?   │ Yes  │ 100 │ 65¢  │ +$5 │ Sell │  │
     │  │ BTC > $100K by March?  │ No   │  50 │ 40¢  │ +$8 │ Sell │  │
     │  │ Will Y happen?         │ Yes  │  75 │ 52¢  │ -$3 │ Sell │  │
     │  └─────────────────────────────────────────────────────────┘  │
     │                                                                │
     │  OPEN ORDERS (2)                                              │
     │  ┌─────────────────────────────────────────────────────────┐  │
     │  │ Market                 │ Side │ Price │ Qty  │ Cancel   │  │
     │  ├────────────────────────┼──────┼───────┼──────┼──────────┤  │
     │  │ Will Z announce?       │ Yes  │  45¢  │  100 │    ✕     │  │
     │  │ Will A > B?            │ No   │  30¢  │   50 │    ✕     │  │
     │  └─────────────────────────────────────────────────────────┘  │
     │                                                                │
     └────────────────────────────────────────────────────────────────┘
```

### 6.2 Position Detail Flow

```
     ┌────────────────┐
     │ Click Position │
     │ Row            │
     └───────┬────────┘
             │
             ▼
     ┌────────────────────────────────────────────────────────────────┐
     │                      POSITION DETAIL                           │
     ├────────────────────────────────────────────────────────────────┤
     │                                                                │
     │  Market: "Will X win the 2026 election?"                      │
     │  ──────────────────────────────────────────────────────────    │
     │                                                                │
     │  YOUR POSITION                                                 │
     │  ┌──────────────────────────────────────────────────────────┐ │
     │  │ Side:           YES                                      │ │
     │  │ Quantity:       100 shares                               │ │
     │  │ Average Price:  65.00¢                                   │ │
     │  │ Cost Basis:     $65.00                                   │ │
     │  │                                                          │ │
     │  │ Current Price:  70.00¢                                   │ │
     │  │ Current Value:  $70.00                                   │ │
     │  │ Unrealized P&L: +$5.00 (+7.7%)                          │ │
     │  │                                                          │ │
     │  │ If YES wins:    +$35.00 profit                          │ │
     │  │ If NO wins:     -$65.00 loss                            │ │
     │  └──────────────────────────────────────────────────────────┘ │
     │                                                                │
     │  TRADE HISTORY FOR THIS MARKET                                │
     │  ┌──────────────────────────────────────────────────────────┐ │
     │  │ Date       │ Action │ Qty │ Price │ Total                │ │
     │  ├────────────┼────────┼─────┼───────┼──────────────────────┤ │
     │  │ Jan 15     │ Buy    │  50 │  60¢  │ $30.00               │ │
     │  │ Jan 18     │ Buy    │  50 │  70¢  │ $35.00               │ │
     │  └──────────────────────────────────────────────────────────┘ │
     │                                                                │
     │  ACTIONS                                                       │
     │  [    Sell Position    ]  [    Add to Position    ]           │
     │  [    Set Price Alert  ]  [    View Market        ]           │
     │                                                                │
     └────────────────────────────────────────────────────────────────┘
```

---

## 7. ASCII Wireframes

### 7.1 Home Page (Desktop)

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│ ┌─────────────────────────────────────────────────────────────────────────────────────┐ │
│ │ [Logo]    Markets   Leaderboard   Learn          🔍 Search...    [Deposit] [Profile]│ │
│ └─────────────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                         │
│  ┌───────────────────────────────────────────────────────────────────────────────────┐ │
│  │                                                                                   │ │
│  │   The World's Prediction Market                                                   │ │
│  │   Trade on the outcome of real-world events                                       │ │
│  │                                                                                   │ │
│  │   [   Get Started   ]    [   Explore Markets   ]                                 │ │
│  │                                                                                   │ │
│  └───────────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                         │
│  TRENDING MARKETS                                                         [See All →] │
│  ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐              │
│  │ 🏛️ Politics         │ │ 🏀 Sports           │ │ 🪙 Crypto            │              │
│  │                     │ │                     │ │                     │              │
│  │ Will X win the     │ │ Will Team A win    │ │ BTC > $100K by     │              │
│  │ 2026 election?     │ │ championship?      │ │ March 2026?        │              │
│  │                     │ │                     │ │                     │              │
│  │ ┌────────────────┐ │ │ ┌────────────────┐ │ │ ┌────────────────┐ │              │
│  │ │ Yes   ████ 65% │ │ │ │ Yes  █████ 72% │ │ │ │ Yes  ███── 45% │ │              │
│  │ │ No    ██── 35% │ │ │ │ No   ██─── 28% │ │ │ │ No   ████─ 55% │ │              │
│  │ └────────────────┘ │ │ └────────────────┘ │ │ └────────────────┘ │              │
│  │                     │ │                     │ │                     │              │
│  │ $1.2M Vol  │ 34d   │ │ $890K Vol │ 12d    │ │ $2.3M Vol │ 65d   │              │
│  └─────────────────────┘ └─────────────────────┘ └─────────────────────┘              │
│                                                                                         │
│  BROWSE BY CATEGORY                                                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐        │
│  │ 🏛️       │ │ 🏀       │ │ 🪙       │ │ 🎬       │ │ 🔬       │ │ 📈       │        │
│  │Politics  │ │ Sports   │ │ Crypto   │ │Entertain │ │ Science  │ │ Economics│        │
│  │  127     │ │   89     │ │   56     │ │   34     │ │   23     │ │   45     │        │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘        │
│                                                                                         │
│  NEW MARKETS                                                             [See All →] │
│  ┌───────────────────────────────────────────────────────────────────────────────────┐ │
│  │ Market                                      │ Probability │ Volume │ Ends In     │ │
│  ├─────────────────────────────────────────────┼─────────────┼────────┼─────────────┤ │
│  │ Will the Fed cut rates in Q1 2026?         │    58%      │ $456K  │ 45 days     │ │
│  │ Will movie X gross $1B worldwide?          │    42%      │ $123K  │ 30 days     │ │
│  │ Will company Y announce layoffs?           │    23%      │ $234K  │ 14 days     │ │
│  └───────────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                         │
│  ┌───────────────────────────────────────────────────────────────────────────────────┐ │
│  │ [Logo]   Markets | Leaderboard | Learn | About | Terms | Privacy      © 2026     │ │
│  └───────────────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Home Page (Mobile)

```
┌─────────────────────────┐
│ [☰]  [Logo]    [🔍] [👤]│
├─────────────────────────┤
│                         │
│ Trade on real-world     │
│ event outcomes          │
│                         │
│ [   Get Started   ]     │
│                         │
├─────────────────────────┤
│ TRENDING                │
│ ┌─────────────────────┐ │
│ │ Will X win 2026?    │ │
│ │                     │ │
│ │ Yes ████████── 65%  │ │
│ │ No  ████────── 35%  │ │
│ │                     │ │
│ │ $1.2M Vol  34d left │ │
│ └─────────────────────┘ │
│                         │
│ ┌─────────────────────┐ │
│ │ Team A wins title?  │ │
│ │                     │ │
│ │ Yes █████████─ 72%  │ │
│ │ No  ███─────── 28%  │ │
│ │                     │ │
│ │ $890K Vol  12d left │ │
│ └─────────────────────┘ │
│                         │
├─────────────────────────┤
│ CATEGORIES              │
│ ┌─────┐┌─────┐┌─────┐   │
│ │🏛️   ││🏀   ││🪙   │   │
│ │Poli-││Sport││Cryp-│   │
│ │tics ││  s  ││  to │   │
│ └─────┘└─────┘└─────┘   │
│ ┌─────┐┌─────┐┌─────┐   │
│ │🎬   ││🔬   ││📈   │   │
│ │Ent. ││Sci. ││Econ │   │
│ └─────┘└─────┘└─────┘   │
│                         │
├─────────────────────────┤
│ [🏠] [📊] [💰] [🏆] [👤]│
└─────────────────────────┘
```

### 7.3 Market Detail Page (Desktop)

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│ [Logo]    Markets   Leaderboard   Learn          🔍 Search...    [Deposit] [Profile]   │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                         │
│ ← Back to Markets                                                                       │
│                                                                                         │
│ ┌────────────────────────────────────────────────────┬──────────────────────────────┐  │
│ │                                                    │                              │  │
│ │  🏛️ Politics                                       │  TRADE                       │  │
│ │                                                    │                              │  │
│ │  Will X win the 2026 Presidential               │  ┌────────────┬───────────┐  │  │
│ │  Election?                                        │  │  BUY YES   │  BUY NO   │  │  │
│ │                                                    │  │    65¢     │    35¢    │  │  │
│ │  Current Probability                              │  └────────────┴───────────┘  │  │
│ │  ┌──────────────────────────────────────────────┐ │                              │  │
│ │  │██████████████████████████████░░░░░░░░░░░░░░│ │  Order Type                  │  │
│ │  │                   65%                        │ │  [● Market] [○ Limit]       │  │
│ │  └──────────────────────────────────────────────┘ │                              │  │
│ │                                                    │  Amount                      │  │
│ │  PRICE CHART                    [1D][1W][1M][All] │  ┌─────────────────────────┐ │  │
│ │  ┌──────────────────────────────────────────────┐ │  │ $  │     100           │ │  │
│ │  │      70│          ╱╲                         │ │  └─────────────────────────┘ │  │
│ │  │        │    ╱╲   ╱  ╲   ╱╲                   │ │  [25%] [50%] [75%] [Max]    │  │
│ │  │      60│   ╱  ╲ ╱    ╲ ╱  ╲                  │ │                              │  │
│ │  │        │  ╱    ╳      ╳    ╲ ╱╲              │ │  ─────────────────────────   │  │
│ │  │      50│ ╱               ╲╱  ╲              │ │  Est. Shares:  153.84       │  │
│ │  │        │╱                                    │ │  Avg Price:    65.00¢       │  │
│ │  │      40├────────────────────────────────────│ │  If YES wins:  +$53.84      │  │
│ │  │        Jan    Feb    Mar    Apr    May       │ │  If NO wins:   -$100.00     │  │
│ │  └──────────────────────────────────────────────┘ │                              │  │
│ │                                                    │  [    Place Order    ]       │  │
│ │  MARKET INFO                                       │                              │  │
│ │  ┌──────────────────────────────────────────────┐ │  ─────────────────────────   │  │
│ │  │ Resolution Date:  November 5, 2026           │ │  YOUR POSITION               │  │
│ │  │ Volume (24h):     $234,567                   │ │  50 Yes @ 60¢               │  │
│ │  │ Total Volume:     $12,345,678                │ │  P&L: +$2.50 (+8.3%)        │  │
│ │  │ Liquidity:        $456,789                   │ │  [    Sell    ]              │  │
│ │  └──────────────────────────────────────────────┘ │                              │  │
│ │                                                    │                              │  │
│ │  RESOLUTION CRITERIA                              │                              │  │
│ │  ┌──────────────────────────────────────────────┐ │                              │  │
│ │  │ This market will resolve to "Yes" if X is   │ │                              │  │
│ │  │ officially declared winner of the 2026       │ │                              │  │
│ │  │ U.S. Presidential Election by certified      │ │                              │  │
│ │  │ results. Resolution source: AP/Reuters.      │ │                              │  │
│ │  └──────────────────────────────────────────────┘ │                              │  │
│ │                                                    │                              │  │
│ └────────────────────────────────────────────────────┴──────────────────────────────┘  │
│                                                                                         │
│ RELATED MARKETS                                                                         │
│ ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐                │
│ │ Will Y win primary? │ │ Electoral votes >300│ │ Control of Senate   │                │
│ │ Yes: 45%  No: 55%   │ │ Yes: 38%  No: 62%   │ │ Dem: 52%  Rep: 48%  │                │
│ └─────────────────────┘ └─────────────────────┘ └─────────────────────┘                │
│                                                                                         │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### 7.4 Market Detail Page (Mobile)

```
┌─────────────────────────┐
│ ←  Will X win 2026?  ⋮ │
├─────────────────────────┤
│                         │
│ Current Probability     │
│ ┌─────────────────────┐ │
│ │████████████░░░░ 65% │ │
│ └─────────────────────┘ │
│                         │
│ ┌──────────┬──────────┐ │
│ │ BUY YES  │  BUY NO  │ │
│ │   65¢    │   35¢    │ │
│ └──────────┴──────────┘ │
│                         │
├─────────────────────────┤
│ [Chart][Orders][Info]   │
├─────────────────────────┤
│                         │
│ PRICE CHART      [1W ▼] │
│ ┌─────────────────────┐ │
│ │    ╱╲    ╱╲         │ │
│ │   ╱  ╲╱╲╱  ╲   ╱╲   │ │
│ │  ╱          ╲ ╱  ╲  │ │
│ │ ╱            ╳    ╲ │ │
│ │╱                    │ │
│ └─────────────────────┘ │
│                         │
│ Resolution: Nov 5, 2026 │
│ Volume: $12.3M          │
│                         │
├─────────────────────────┤
│                         │
│ TRADE                   │
│ [● Market] [○ Limit]    │
│                         │
│ Amount                  │
│ ┌─────────────────────┐ │
│ │ $       100         │ │
│ └─────────────────────┘ │
│                         │
│ Est. Shares: 153.84     │
│ Potential: +$53.84      │
│                         │
│ [    Place Order    ]   │
│                         │
├─────────────────────────┤
│ [🏠] [📊] [💰] [🏆] [👤]│
└─────────────────────────┘
```

### 7.5 Portfolio Page (Desktop)

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│ [Logo]    Markets   Leaderboard   Learn          🔍 Search...    [Deposit] [Profile]   │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                         │
│ MY PORTFOLIO                                                                            │
│                                                                                         │
│ ┌─────────────────────────────────────────────────────────────────────────────────────┐ │
│ │                                                                                     │ │
│ │  PORTFOLIO VALUE              TODAY'S P&L              ALL-TIME P&L                │ │
│ │  $1,234.56                    +$45.67 (+3.8%)          +$234.56 (+23.4%)           │ │
│ │                                                                                     │ │
│ │  ┌───────────────────────────────────────────────────────────────────────────────┐ │ │
│ │  │  $1400 │                                            ╱                         │ │ │
│ │  │        │                                      ╱╲   ╱                          │ │ │
│ │  │  $1200 │                               ╱╲    ╱  ╲ ╱                           │ │ │
│ │  │        │                         ╱╲   ╱  ╲╱╱    ╳                            │ │ │
│ │  │  $1000 │                   ╱╲   ╱  ╲╱╱                                        │ │ │
│ │  │        │            ╱╲╲   ╱  ╲╱╱                                              │ │ │
│ │  │   $800 │      ╱╲   ╱   ╲╱╱                                                    │ │ │
│ │  │        │ ╱╲╲╱╱  ╲╱╱                                                           │ │ │
│ │  │   $600 │╱                                                                     │ │ │
│ │  │        └─────────────────────────────────────────────────────────────────────│ │ │
│ │  │         Jan     Feb     Mar     Apr     May     Jun                          │ │ │
│ │  └───────────────────────────────────────────────────────────────────────────────┘ │ │
│ │                                                                                     │ │
│ └─────────────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                         │
│ ┌──────────────────────────────────────────────────────────────────────────────────────┐│
│ │ [Active Positions (5)]  [Open Orders (2)]  [Trade History]  [Resolved]              ││
│ └──────────────────────────────────────────────────────────────────────────────────────┘│
│                                                                                         │
│ ACTIVE POSITIONS                                                                        │
│ ┌──────────────────────────────────────────────────────────────────────────────────────┐│
│ │ Market                              │ Side │  Qty │  Avg  │ Current │   P&L   │ Act ││
│ ├─────────────────────────────────────┼──────┼──────┼───────┼─────────┼─────────┼─────┤│
│ │ Will X win the 2026 election?      │ Yes  │  100 │ 65.0¢ │  70.0¢  │ +$5.00  │[Sell││
│ │ BTC > $100K by March 2026?         │ No   │   50 │ 40.0¢ │  35.0¢  │ +$2.50  │[Sell││
│ │ Will Team A win championship?      │ Yes  │   75 │ 72.0¢ │  75.0¢  │ +$2.25  │[Sell││
│ │ Fed rate cut in Q1?                │ Yes  │  200 │ 58.0¢ │  55.0¢  │ -$6.00  │[Sell││
│ │ Movie X grosses $1B?               │ No   │   30 │ 58.0¢ │  60.0¢  │ -$0.60  │[Sell││
│ └──────────────────────────────────────────────────────────────────────────────────────┘│
│                                                                                         │
│ OPEN ORDERS                                                                             │
│ ┌──────────────────────────────────────────────────────────────────────────────────────┐│
│ │ Market                              │ Side │ Price │   Qty │ Filled │ Created │ Act ││
│ ├─────────────────────────────────────┼──────┼───────┼───────┼────────┼─────────┼─────┤│
│ │ Will Z make announcement?          │ Yes  │  45¢  │   100 │   0%   │ Jan 20  │  ✕  ││
│ │ Inflation < 3% by July?            │ No   │  30¢  │    50 │   0%   │ Jan 18  │  ✕  ││
│ └──────────────────────────────────────────────────────────────────────────────────────┘│
│                                                                                         │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### 7.6 Sign Up Modal

```
┌───────────────────────────────────────┐
│                                    ✕  │
│                                       │
│         Create Your Account           │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │ G  Continue with Google         │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │   Continue with Apple          │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │ 🦊 Connect Wallet               │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ──────────── or ────────────        │
│                                       │
│   Email                               │
│   ┌─────────────────────────────────┐ │
│   │ email@example.com               │ │
│   └─────────────────────────────────┘ │
│                                       │
│   Password                            │
│   ┌─────────────────────────────────┐ │
│   │ ••••••••••••                    │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │       Create Account            │ │
│   └─────────────────────────────────┘ │
│                                       │
│   Already have an account? Sign in    │
│                                       │
│   By signing up, you agree to our     │
│   Terms of Service and Privacy Policy │
│                                       │
└───────────────────────────────────────┘
```

### 7.7 Order Confirmation Modal

```
┌───────────────────────────────────────┐
│                                    ✕  │
│                                       │
│         Confirm Your Order            │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │ Will X win the 2026 election?   │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │                                 │ │
│   │   Buying: YES                   │ │
│   │                                 │ │
│   │   ─────────────────────────     │ │
│   │                                 │ │
│   │   Amount:        $100.00        │ │
│   │   Shares:        153.84         │ │
│   │   Avg Price:     65.00¢         │ │
│   │                                 │ │
│   │   ─────────────────────────     │ │
│   │                                 │ │
│   │   If YES wins:   +$53.84        │ │
│   │   If NO wins:    -$100.00       │ │
│   │                                 │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ⚠️ Slippage warning: Estimated      │
│      price may vary by up to 2%       │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │       Confirm Order             │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │         Cancel                  │ │
│   └─────────────────────────────────┘ │
│                                       │
└───────────────────────────────────────┘
```

### 7.8 Deposit Modal

```
┌───────────────────────────────────────┐
│ ←  Add Funds                       ✕  │
├───────────────────────────────────────┤
│                                       │
│   Current Balance: $234.56            │
│                                       │
│   ─────────────────────────────────   │
│                                       │
│   Choose Payment Method               │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │ 💳 Credit/Debit Card            │ │
│   │    Instant • 2.9% fee           │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │  Apple Pay                     │ │
│   │    Instant • 2.9% fee           │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │ 🪙 Crypto (USDC)                │ │
│   │    ~5 min • Network fee only    │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │ 🏦 Bank Transfer (ACH)          │ │
│   │    1-3 days • No fee            │ │
│   └─────────────────────────────────┘ │
│                                       │
│   ─────────────────────────────────   │
│                                       │
│   Amount                              │
│   ┌─────────────────────────────────┐ │
│   │ $       100                     │ │
│   └─────────────────────────────────┘ │
│   [$25] [$50] [$100] [$500]           │
│                                       │
│   Min: $10 • Max: $10,000             │
│                                       │
│   ┌─────────────────────────────────┐ │
│   │       Continue                  │ │
│   └─────────────────────────────────┘ │
│                                       │
└───────────────────────────────────────┘
```

---

## Appendix

### A. Flow Diagram Legend

```
┌──────────┐
│  Screen  │   Rectangular box = Screen/Page
└──────────┘

◇──────────◇
│ Decision │   Diamond = Decision point
◇──────────◇

     │
     ▼         Arrow = Flow direction
     │

─────────      Line = Connection

[Button]       Square brackets = Interactive element

(Note)         Parentheses = Annotation
```

### B. Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Jan 2026 | Initial user flows and wireframes |

### C. References

- [Polymarket Mobile Case Study](https://www.lazertechnologies.com/case-studies/polymarket)
- [Prediction Market UX Best Practices](https://iceteasoftware.com/blog/build-a-prediction-market-like-polymarket)
- [Web3 Onboarding Patterns](https://merge.rocks/blog/web3-design-in-2024-best-principles-and-patterns)
