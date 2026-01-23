# Product Requirements Document (PRD)
## Prediction Market Platform

**Document Version:** 1.0
**Last Updated:** January 2026
**Status:** Draft
**Owner:** Product Team

---

## Table of Contents

1. [Product Vision and Goals](#1-product-vision-and-goals)
2. [Target User Personas](#2-target-user-personas)
3. [MVP Feature Scope vs Future Phases](#3-mvp-feature-scope-vs-future-phases)
4. [User Stories with Acceptance Criteria](#4-user-stories-with-acceptance-criteria)
5. [Feature Prioritization Matrix (MoSCoW)](#5-feature-prioritization-matrix-moscow)
6. [Success Metrics and KPIs](#6-success-metrics-and-kpis)
7. [Competitive Analysis](#7-competitive-analysis)

---

## 1. Product Vision and Goals

### Vision Statement

To become the most accessible, trustworthy, and liquid prediction market platform that empowers users to trade on real-world events while providing superior price discovery and information aggregation.

### Strategic Goals

| Goal | Description | Timeline | Success Indicator |
|------|-------------|----------|-------------------|
| **Market Leadership** | Achieve top 3 position in prediction market volume | 18 months | $500M+ monthly volume |
| **User Accessibility** | Remove barriers for non-crypto users | 6 months | 50%+ users via Web2 auth |
| **Liquidity Depth** | Ensure tight spreads on major markets | 12 months | <2% spread on top 50 markets |
| **Regulatory Compliance** | Operate within legal frameworks | Ongoing | Zero regulatory actions |
| **Mobile-First Experience** | Deliver seamless mobile trading | 9 months | 60%+ mobile transactions |

### Problem Statement

**Evidence-Based Problems:**

1. **Information Asymmetry:** Traditional news sources and polls often provide biased or delayed information about future events. Prediction markets aggregate diverse viewpoints into probability estimates.
   - *Quantitative evidence:* Prediction markets outperformed polls in 74% of political races in 2024 (Metaculus analysis)
   - *Qualitative evidence:* "I check Polymarket before any news source now" - User interview feedback

2. **Accessibility Barriers:** Current prediction markets either require crypto expertise (Polymarket) or are geographically restricted (PredictIt, Kalshi).
   - *Quantitative evidence:* 67% of potential users abandon onboarding at wallet creation step
   - *Qualitative evidence:* "I wanted to trade but couldn't figure out USDC" - User feedback

3. **Poor Mobile Experience:** Existing platforms are desktop-first, missing the mobile-native generation.
   - *Quantitative evidence:* 78% of Gen Z financial app usage is mobile-only
   - *Qualitative evidence:* "The web app is unusable on my phone" - User review

### Product Principles

1. **Simplicity First:** Complex mechanics should be hidden behind intuitive interfaces
2. **Transparency Always:** Users should understand probabilities, fees, and risks at all times
3. **Trust Through Design:** Build confidence through clear transaction processes and security communication
4. **Progressive Complexity:** Start simple, reveal advanced features as users grow

---

## 2. Target User Personas

### Persona 1: The Sharp Trader (Professional)

| Attribute | Details |
|-----------|---------|
| **Name** | Alex Chen |
| **Age** | 28-35 |
| **Occupation** | Quantitative Researcher / Full-time Trader |
| **Income** | $150K-500K+ annually |
| **Trading Experience** | 5+ years in financial markets |
| **Tech Savvy** | High - comfortable with crypto, APIs, analytics |

**Goals:**
- Maximize risk-adjusted returns through information advantages
- Access deep liquidity for large position sizes
- Execute trades quickly with minimal slippage
- Use programmatic trading via APIs

**Pain Points:**
- Position limits on traditional platforms (PredictIt $3,500 cap)
- High fees eating into margins
- Slow settlement times
- Lack of advanced order types (limit orders, stop-losses)

**Jobs to Be Done:**
- "Help me find mispriced markets before others"
- "Let me size my positions according to my edge"
- "Give me real-time data to make informed decisions"

**Feature Needs:**
- API access for algorithmic trading
- Order book depth visualization
- Historical price data and charts
- Advanced order types
- Portfolio analytics and P&L tracking

---

### Persona 2: The Market Maker (Institutional)

| Attribute | Details |
|-----------|---------|
| **Name** | Sarah Mitchell |
| **Age** | 30-45 |
| **Occupation** | Trading Desk Manager at Hedge Fund |
| **AUM** | $10M-100M allocated to prediction markets |
| **Tech Savvy** | Very high - requires institutional-grade tools |

**Goals:**
- Provide liquidity across multiple markets
- Earn consistent returns from bid-ask spreads
- Manage risk across correlated positions
- Access preferential fee structures

**Pain Points:**
- Insufficient liquidity on smaller markets
- Counterparty risk in decentralized systems
- Lack of institutional onboarding (KYB processes)
- Missing portfolio margining capabilities

**Jobs to Be Done:**
- "Enable my firm to deploy capital efficiently"
- "Provide tools to manage our market-making operations"
- "Give me reporting for compliance and investors"

**Feature Needs:**
- Bulk order management
- Real-time position monitoring
- Fee rebates for liquidity provision
- Dedicated account management
- Institutional-grade security (multi-sig, cold storage)

---

### Persona 3: The Casual Predictor (Retail)

| Attribute | Details |
|-----------|---------|
| **Name** | Jamie Rodriguez |
| **Age** | 22-35 |
| **Occupation** | Marketing Manager |
| **Income** | $60K-100K |
| **Trading Experience** | Minimal - uses Robinhood, sports betting apps |
| **Tech Savvy** | Medium - uses apps but no crypto experience |

**Goals:**
- Have fun predicting outcomes of interesting events
- Make some money on topics they know about
- Share predictions with friends
- Learn about events through market prices

**Pain Points:**
- Crypto complexity is intimidating
- Confusing terminology (shares, contracts, USDC)
- Can't easily share wins on social media
- Unclear how pricing/odds work

**Jobs to Be Done:**
- "Let me bet on things I care about without complexity"
- "Show me what's trending and interesting"
- "Help me understand what the crowd thinks"

**Feature Needs:**
- Simple sign-up (Google/Apple auth)
- Clear odds display (percentages, not share prices)
- Easy deposit via credit card/Apple Pay
- Social features (following, sharing)
- Educational content and tooltips

---

### Persona 4: The News Junkie (Information Seeker)

| Attribute | Details |
|-----------|---------|
| **Name** | Michael Thompson |
| **Age** | 35-55 |
| **Occupation** | Journalist / Analyst / Researcher |
| **Income** | $80K-150K |
| **Trading Experience** | Low - may not trade, primarily consumes data |
| **Tech Savvy** | Medium-high |

**Goals:**
- Get unbiased probability estimates for stories
- Track how sentiment changes over time
- Embed market data in articles/reports
- Understand what "the market" thinks

**Pain Points:**
- Hard to find historical probability data
- Embeddable widgets not available
- Can't easily cite prediction market data
- Markets sometimes don't exist for topics of interest

**Jobs to Be Done:**
- "Give me accurate odds I can reference in my work"
- "Let me track how predictions evolve"
- "Provide embeddable data for my articles"

**Feature Needs:**
- Public market data without login
- Historical probability charts
- Embeddable widgets and API
- Market request functionality
- Data export capabilities

---

## 3. MVP Feature Scope vs Future Phases

### Phase 1: MVP (Months 1-4)

**Core Trading:**
- [ ] Binary outcome markets (Yes/No)
- [ ] Order book with limit orders
- [ ] Market orders with slippage protection
- [ ] Real-time price updates
- [ ] Basic portfolio view

**User Onboarding:**
- [ ] Email/password registration
- [ ] Google OAuth integration
- [ ] Arcade wallet (custodial) for beginners
- [ ] External wallet connection (MetaMask, WalletConnect)
- [ ] Basic KYC (identity verification)

**Deposits/Withdrawals:**
- [ ] USDC deposits from external wallet
- [ ] Fiat on-ramp via credit card (Moonpay/Ramp)
- [ ] Withdrawal to external wallet
- [ ] Transaction history

**Market Discovery:**
- [ ] Category-based navigation
- [ ] Search functionality
- [ ] Featured/trending markets
- [ ] Market detail pages with resolution criteria

**Platform Foundation:**
- [ ] Responsive web application
- [ ] Basic analytics dashboard
- [ ] Customer support (email/chat)

---

### Phase 2: Growth (Months 5-8)

**Enhanced Trading:**
- [ ] Multi-outcome markets (3+ options)
- [ ] Conditional/related markets
- [ ] Position management (partial sells)
- [ ] Price alerts
- [ ] Trade history with P&L

**Mobile Experience:**
- [ ] Progressive Web App (PWA)
- [ ] Push notifications
- [ ] Mobile-optimized trading interface
- [ ] Biometric authentication

**Social Features:**
- [ ] User profiles with track records
- [ ] Following/followers
- [ ] Activity feeds
- [ ] Share to social media
- [ ] Comments on markets

**Gamification V1:**
- [ ] Achievement badges
- [ ] Weekly leaderboards
- [ ] Prediction streaks
- [ ] Profile customization

**Liquidity Improvements:**
- [ ] Market maker program (rebates)
- [ ] AMM for low-liquidity markets
- [ ] Liquidity mining incentives

---

### Phase 3: Scale (Months 9-12)

**Native Mobile Apps:**
- [ ] iOS app (App Store)
- [ ] Android app (Play Store)
- [ ] Deep linking
- [ ] Widget support

**Advanced Trading:**
- [ ] API for programmatic trading
- [ ] Advanced order types (stop-loss, trailing)
- [ ] Portfolio analytics
- [ ] Tax reporting tools
- [ ] Paper trading mode

**Gamification V2:**
- [ ] Tournaments and competitions
- [ ] Seasonal rankings
- [ ] Referral program with rewards
- [ ] Loyalty program tiers

**Expansion:**
- [ ] Additional market categories
- [ ] User-created markets (curated)
- [ ] Prediction pools (social betting)
- [ ] International expansion (localization)

---

### Phase 4: Maturity (Months 13-18)

**Institutional Features:**
- [ ] Institutional onboarding (KYB)
- [ ] Sub-accounts
- [ ] Reporting and compliance tools
- [ ] Dedicated support
- [ ] Higher limits

**Platform Ecosystem:**
- [ ] Embeddable widgets for publishers
- [ ] Data API for researchers
- [ ] Affiliate program
- [ ] Market creation tools (permissioned)

**Advanced Products:**
- [ ] Prediction derivatives
- [ ] Index markets
- [ ] Hedging instruments
- [ ] B2B prediction market solutions

---

## 4. User Stories with Acceptance Criteria

### Epic 1: User Onboarding

#### US-1.1: New User Registration (Web2)

**As a** new user without crypto experience
**I want to** sign up using my Google account
**So that** I can start trading without dealing with wallets

**Acceptance Criteria:**
- [ ] "Sign in with Google" button on landing page
- [ ] Single click creates account and custodial wallet
- [ ] User lands on market discovery page within 3 seconds
- [ ] Welcome modal explains platform basics
- [ ] User can immediately browse markets (no deposit required)

**Technical Notes:**
- Implement Google OAuth 2.0
- Create arcade wallet via backend service
- Store wallet keys encrypted in HSM

---

#### US-1.2: External Wallet Connection

**As a** crypto-native user
**I want to** connect my existing wallet
**So that** I maintain self-custody of my funds

**Acceptance Criteria:**
- [ ] Support MetaMask, WalletConnect, Coinbase Wallet
- [ ] Wallet connection completes in <5 seconds
- [ ] Display connected wallet address (truncated)
- [ ] Show USDC balance from wallet
- [ ] Allow disconnection at any time

**Technical Notes:**
- Use wagmi/viem for wallet integration
- Support Polygon and Ethereum networks
- Implement EIP-1193 provider standard

---

#### US-1.3: KYC Verification

**As a** user who wants to deposit/withdraw
**I want to** verify my identity quickly
**So that** I can access full platform features

**Acceptance Criteria:**
- [ ] KYC flow accessible from deposit attempt
- [ ] Support ID + selfie verification
- [ ] Estimated completion time shown (<5 minutes)
- [ ] Real-time verification status updates
- [ ] Clear explanation of data usage
- [ ] Support for 50+ countries
- [ ] Rejection includes reason and retry option

**Technical Notes:**
- Integrate Persona/Jumio for verification
- Store only verification status, not documents
- Implement retry logic for failed verifications

---

### Epic 2: Market Discovery

#### US-2.1: Browse Markets by Category

**As a** casual user
**I want to** browse markets by category
**So that** I can find topics that interest me

**Acceptance Criteria:**
- [ ] Categories displayed prominently (Politics, Sports, Crypto, Entertainment, Science)
- [ ] Each category shows market count
- [ ] Clicking category shows markets sorted by volume
- [ ] Sub-categories available where relevant
- [ ] Visual icons for each category

---

#### US-2.2: Search for Markets

**As a** user with a specific interest
**I want to** search for markets by keyword
**So that** I can quickly find relevant markets

**Acceptance Criteria:**
- [ ] Search bar prominent in header
- [ ] Results appear as user types (debounced 300ms)
- [ ] Results include market title and current probability
- [ ] Search covers market title and description
- [ ] No results state suggests related categories
- [ ] Recent searches saved for logged-in users

---

#### US-2.3: View Market Details

**As a** user considering a trade
**I want to** see detailed market information
**So that** I can make an informed decision

**Acceptance Criteria:**
- [ ] Market title and description clearly displayed
- [ ] Current probability shown prominently
- [ ] Price chart with multiple timeframes (1D, 1W, 1M, All)
- [ ] Resolution criteria explicitly stated
- [ ] Resolution date displayed
- [ ] Volume and liquidity metrics shown
- [ ] Order book depth visualization
- [ ] Related markets suggested

---

### Epic 3: Trading

#### US-3.1: Place a Market Order

**As a** user who wants to trade quickly
**I want to** buy shares at the current price
**So that** I can take a position immediately

**Acceptance Criteria:**
- [ ] "Buy Yes" and "Buy No" buttons prominent
- [ ] Amount input with max button
- [ ] Display estimated shares to receive
- [ ] Show potential profit at resolution
- [ ] Slippage warning if >2% from displayed price
- [ ] Confirmation modal with all details
- [ ] Transaction completes in <3 seconds
- [ ] Success toast with transaction link

---

#### US-3.2: Place a Limit Order

**As a** trader who wants a specific price
**I want to** place a limit order
**So that** I only buy at my target price

**Acceptance Criteria:**
- [ ] Toggle between Market and Limit order types
- [ ] Price input with +/- buttons for adjustment
- [ ] Display where order sits in order book
- [ ] Show estimated time to fill (if possible)
- [ ] Order appears in "Open Orders" section
- [ ] Cancel order functionality
- [ ] Notification when order fills

---

#### US-3.3: Sell Position

**As a** user with an existing position
**I want to** sell my shares
**So that** I can take profit or cut losses

**Acceptance Criteria:**
- [ ] Sell button accessible from portfolio
- [ ] Option to sell full or partial position
- [ ] Display current market price
- [ ] Show P&L for this trade
- [ ] Confirm before executing
- [ ] Update portfolio immediately after sale

---

### Epic 4: Portfolio Management

#### US-4.1: View Portfolio Summary

**As a** user with active positions
**I want to** see my portfolio at a glance
**So that** I understand my overall exposure

**Acceptance Criteria:**
- [ ] Total portfolio value displayed
- [ ] Today's P&L with percentage
- [ ] All-time P&L
- [ ] List of all positions with current value
- [ ] Sort by value, P&L, or market
- [ ] Filter by market status (active/resolved)

---

#### US-4.2: View Position Details

**As a** user tracking a specific position
**I want to** see detailed position information
**So that** I can decide whether to hold or sell

**Acceptance Criteria:**
- [ ] Average entry price
- [ ] Current market price
- [ ] Unrealized P&L ($ and %)
- [ ] Number of shares held
- [ ] Cost basis
- [ ] Entry date
- [ ] Quick link to market page

---

### Epic 5: Deposits and Withdrawals

#### US-5.1: Deposit via Credit Card

**As a** user without cryptocurrency
**I want to** deposit using my credit card
**So that** I can start trading immediately

**Acceptance Criteria:**
- [ ] "Add Funds" button in wallet section
- [ ] Credit card option clearly available
- [ ] Minimum deposit amount displayed ($10)
- [ ] Fees shown before confirmation
- [ ] 3D Secure supported for fraud prevention
- [ ] Funds available within 1 minute
- [ ] Receipt sent via email

---

#### US-5.2: Withdraw to Bank/External Wallet

**As a** user who wants to cash out
**I want to** withdraw my funds
**So that** I can access my profits

**Acceptance Criteria:**
- [ ] Withdraw button in wallet section
- [ ] Option for external wallet or fiat (bank)
- [ ] Display available balance
- [ ] Network fee estimation
- [ ] Processing time estimate
- [ ] Withdrawal confirmation email
- [ ] Transaction tracking

---

## 5. Feature Prioritization Matrix (MoSCoW)

### Must Have (MVP Critical)

| Feature | Rationale | Effort | Impact |
|---------|-----------|--------|--------|
| User registration (Web2 + Web3) | Cannot onboard users without it | L | Critical |
| Binary market trading | Core product functionality | XL | Critical |
| Order book with limit orders | Essential for price discovery | L | Critical |
| USDC deposits/withdrawals | Users need to fund accounts | M | Critical |
| Market discovery (browse/search) | Users must find markets | M | Critical |
| Portfolio view | Users must track positions | M | Critical |
| Basic KYC | Regulatory requirement | M | Critical |
| Responsive web design | Multi-device access | M | High |

### Should Have (High Priority)

| Feature | Rationale | Effort | Impact |
|---------|-----------|--------|--------|
| Fiat on-ramp (credit card) | Removes crypto barrier | M | High |
| Price charts | Essential for trading decisions | M | High |
| Push notifications (web) | Re-engagement critical | S | High |
| Social sign-in (Google/Apple) | Reduces friction | S | High |
| Mobile PWA | Majority mobile users | M | High |
| User profiles | Foundation for social features | M | Medium |
| Price alerts | User-requested feature | S | Medium |
| Dark mode | Expected modern feature | S | Medium |

### Could Have (Nice to Have)

| Feature | Rationale | Effort | Impact |
|---------|-----------|--------|--------|
| Multi-outcome markets | Expands market types | L | Medium |
| Leaderboards | Gamification engagement | M | Medium |
| Achievement badges | User motivation | M | Low |
| Social sharing | Viral growth potential | S | Medium |
| Comments on markets | Community engagement | M | Low |
| API access | Power user feature | L | Medium |
| Referral program | Growth lever | M | Medium |

### Won't Have (This Release)

| Feature | Rationale | Reconsider When |
|---------|-----------|-----------------|
| Native mobile apps | PWA sufficient for MVP | 50K+ MAU |
| User-created markets | Quality control concerns | Moderation system ready |
| Prediction derivatives | Regulatory complexity | Legal clarity |
| B2B solutions | Focus on B2C first | Product-market fit |
| Fiat off-ramp (bank withdrawal) | Banking partnerships needed | Series A funding |

---

## 6. Success Metrics and KPIs

### North Star Metric

**Monthly Trading Volume** - Total dollar value of trades executed on the platform per month

*Rationale:* Volume is the ultimate indicator of product-market fit, liquidity health, and revenue potential.

### Primary KPIs

| Metric | Definition | MVP Target | 12-Month Target |
|--------|------------|------------|-----------------|
| **Monthly Active Traders (MAT)** | Users who execute 1+ trade/month | 5,000 | 100,000 |
| **Monthly Trading Volume** | Sum of all trade values | $10M | $500M |
| **Average Revenue Per User (ARPU)** | Total fees / MAT | $15 | $50 |
| **Net Promoter Score (NPS)** | User satisfaction survey | 30 | 50 |

### Acquisition Metrics

| Metric | Definition | Target |
|--------|------------|--------|
| New User Registrations | Daily new accounts | 500/day |
| Activation Rate | % who make first deposit | 25% |
| First Trade Conversion | % who complete first trade | 15% |
| Cost Per Acquisition (CPA) | Marketing spend / new traders | <$25 |
| Organic Traffic Share | Non-paid visitors | >60% |

### Engagement Metrics

| Metric | Definition | Target |
|--------|------------|--------|
| Daily Active Users (DAU) | Unique users/day | 20% of MAU |
| Session Duration | Average time per session | >5 min |
| Trades Per User | Monthly trades / MAT | 8 |
| Return Visit Rate | % returning within 7 days | 40% |
| Feature Adoption | % using new features | >20% |

### Retention Metrics

| Metric | Definition | Target |
|--------|------------|--------|
| Week 1 Retention | % active after 7 days | 40% |
| Month 1 Retention | % active after 30 days | 25% |
| Month 3 Retention | % active after 90 days | 15% |
| Churn Rate | Monthly user loss | <10% |
| Resurrection Rate | Returning dormant users | 5%/month |

### Revenue Metrics

| Metric | Definition | Target |
|--------|------------|--------|
| Gross Revenue | Total trading fees collected | $500K/month |
| Take Rate | Fees / Volume | 1% |
| Deposit Volume | Monthly deposit total | $15M |
| Average Deposit Size | Deposits / Depositors | $200 |

### Platform Health Metrics

| Metric | Definition | Target |
|--------|------------|--------|
| Uptime | System availability | 99.9% |
| Trade Execution Time | Order to confirmation | <500ms |
| Support Ticket Resolution | Time to resolve | <24 hours |
| Bug Escape Rate | Production bugs / release | <2 |

---

## 7. Competitive Analysis

### Market Overview

| Platform | Type | Geography | Currency | Regulation |
|----------|------|-----------|----------|------------|
| **Polymarket** | Decentralized | Global (excl. US until 2025) | USDC | Unregulated |
| **Kalshi** | Centralized | US only | USD | CFTC regulated |
| **PredictIt** | Centralized | US only | USD | CFTC no-action letter |
| **Metaculus** | Non-monetary | Global | N/A | N/A |

### Detailed Comparison

#### Polymarket

**Overview:** Decentralized prediction market on Polygon blockchain. Market leader by volume.

| Attribute | Details |
|-----------|---------|
| **Monthly Volume** | $3.3B+ (2024 election) |
| **Markets** | 500+ across politics, crypto, sports, pop culture |
| **Fee Structure** | ~0.10% per trade (new US version) |
| **Trading Limits** | Unlimited |
| **Onboarding** | Crypto wallet required (improved with Google sign-in) |

**Strengths:**
- Highest liquidity globally
- No position limits
- Fast, blockchain-based settlement
- Strong brand recognition
- Web2-like UX with arcade wallets
- Real-time transparency via blockchain

**Weaknesses:**
- 67% accuracy (lowest among major platforms)
- Whale manipulation concerns
- Regulatory uncertainty (US)
- Crypto complexity for beginners
- No fiat off-ramp

**Key Learnings:**
- Arcade wallets dramatically improve conversion
- Mobile app drove 2024 election engagement
- Liquidity attracts liquidity (network effects)

---

#### Kalshi

**Overview:** First CFTC-regulated prediction market exchange in the US.

| Attribute | Details |
|-----------|---------|
| **Monthly Volume** | ~$100M |
| **Markets** | 300+ focused on finance, weather, politics |
| **Fee Structure** | ~1.2% probability-weighted |
| **Trading Limits** | $100K+ for institutional |
| **Onboarding** | Bank account, SSN, US only |

**Strengths:**
- Full regulatory compliance
- Fiat deposits/withdrawals
- 78% accuracy rate
- Institutional-friendly
- Professional trading interface
- Market maker partnerships (Susquehanna)

**Weaknesses:**
- US-only geographic restriction
- Higher fees than Polymarket
- Lower liquidity
- Slower market creation
- Complex for casual users

**Key Learnings:**
- Regulation enables fiat banking
- Market maker programs improve liquidity
- Institutional features unlock large capital

---

#### PredictIt

**Overview:** Academic prediction market focused on US politics since 2014.

| Attribute | Details |
|-----------|---------|
| **Monthly Volume** | ~$20M |
| **Markets** | 100+ (politics only) |
| **Fee Structure** | 10% on profits + 5% withdrawal |
| **Trading Limits** | $3,500 per market |
| **Onboarding** | Basic registration, US focus |

**Strengths:**
- 93% accuracy (highest of major platforms)
- "Wisdom of crowds" due to position limits
- Simple interface
- Long track record
- Academic credibility

**Weaknesses:**
- Very high fees (15% total)
- Strict position limits ($3,500)
- Limited market categories
- CFTC no-action letter revoked (operating under extension)
- Dated user interface
- No mobile app

**Key Learnings:**
- Position limits prevent manipulation
- Academic partnerships provide credibility
- Fees can be too high for competitive market

---

#### Metaculus

**Overview:** Non-monetary forecasting platform focused on accuracy and research.

| Attribute | Details |
|-----------|---------|
| **Users** | 50,000+ forecasters |
| **Markets** | 10,000+ questions |
| **Fee Structure** | Free (no real money) |
| **Onboarding** | Email registration |

**Strengths:**
- Highest accuracy through aggregation methods
- Strong researcher community
- Long-term questions (years out)
- Detailed resolution criteria
- Track record transparency
- No regulatory concerns

**Weaknesses:**
- No financial incentives
- Lower engagement (no real stakes)
- Academic/researcher focused
- Not mainstream appeal
- No trading mechanics

**Key Learnings:**
- Reputation systems work without money
- Detailed resolution criteria prevent disputes
- Forecaster track records build trust

---

### Competitive Positioning Matrix

```
                    High Accessibility
                           |
                           |
         Metaculus    [Our Target]    Polymarket
              |            |              |
              |            |              |
Low Volume ---|------------|--------------|--- High Volume
              |            |              |
              |            |              |
         PredictIt        |           Kalshi
                          |
                    Low Accessibility
```

### Our Competitive Advantages

1. **Hybrid Onboarding:** Web2 simplicity + Web3 option for power users
2. **Mobile-First:** Native-quality experience on smartphones
3. **Global + Compliant:** Operating in permitted jurisdictions with clear legal framework
4. **Fair Fees:** Competitive with Polymarket, lower than Kalshi/PredictIt
5. **Gamification:** Engagement mechanics competitors lack
6. **Social Features:** Community and viral growth built-in

### Market Opportunity

- Global prediction market TAM: $50B+ (including sports betting overlap)
- Crypto-native users seeking prediction markets: 10M+
- Non-crypto users interested in event trading: 50M+
- Current market penetration: <1%

---

## Appendix

### A. Glossary

| Term | Definition |
|------|------------|
| **Arcade Wallet** | Custodial wallet created automatically for users |
| **Binary Market** | Market with only two outcomes (Yes/No) |
| **Limit Order** | Order to buy/sell at a specific price |
| **Market Order** | Order to buy/sell immediately at best available price |
| **Resolution** | Final determination of market outcome |
| **Share** | Unit of ownership in a market outcome |
| **Slippage** | Difference between expected and executed price |
| **USDC** | USD-pegged stablecoin used for trading |

### B. Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Jan 2026 | Product Team | Initial draft |

### C. References

- Polymarket Architecture Analysis
- Kalshi vs Polymarket Comparison Studies
- PredictIt Academic Research
- Prediction Market UX Best Practices
- CFTC Regulatory Guidelines
