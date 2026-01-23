# Pricing Strategy

## Executive Summary

This document outlines the comprehensive pricing strategy for a Polymarket-like prediction market platform, including fee structures, competitive analysis, premium tiers, B2B pricing, and price testing methodology based on industry research and real-world benchmarks.

---

## 1. Pricing Philosophy

### 1.1 Core Principles

**Growth-First Approach:**
The prediction market industry is characterized by extreme fee variation - from 0.01% (Polymarket US) to over 15% effective cost on some platforms. Our pricing philosophy prioritizes:

1. **Market Share Acquisition**: Low fees attract volume; volume attracts liquidity
2. **Network Effects**: More traders = better prices = more traders
3. **Long-Term Value**: Build user base first, monetize later
4. **Competitive Positioning**: Undercut established players on core fees

**Value-Based Pricing:**
- Core trading should be accessible (low/no fees)
- Premium value-add services command premium prices
- Enterprise/B2B captures willingness to pay for customization

### 1.2 Strategic Positioning

```
┌─────────────────────────────────────────────────────────────┐
│                    PRICING MATRIX                           │
├─────────────────────────────────────────────────────────────┤
│           │   Low Price    │    High Price                  │
├───────────┼────────────────┼────────────────────────────────┤
│   High    │  PENETRATION   │   PREMIUM                      │
│   Value   │  (Our Core)    │   (Enterprise)                 │
├───────────┼────────────────┼────────────────────────────────┤
│   Low     │  ECONOMY       │   SKIMMING                     │
│   Value   │  (Avoid)       │   (Competitors)                │
└───────────┴────────────────┴────────────────────────────────┘
```

---

## 2. Fee Structure Options

### 2.1 Option A: Zero Trading Fees (Polymarket Model)

**Structure:**
- No platform trading fees on core markets
- Revenue from other streams (data, premium, B2B)
- Possible small withdrawal fees

**Pros:**
- Maximum volume attraction
- Tightest spreads (more liquidity)
- Competitive moat against fee-charging platforms
- Users love "free"

**Cons:**
- Requires alternative revenue streams
- Higher funding requirements to reach profitability
- Hard to introduce fees later

**Financial Impact:**
```
Year 1 Revenue Mix (Zero-Fee Model):
- Premium Subscriptions: 40%
- API/Data Sales: 25%
- B2B/Enterprise: 20%
- Market Creation: 10%
- Interest/Float: 5%

Required Alternative Revenue to Replace 0.5% Fee:
- At $500M trading volume: $2.5M needed from other sources
- At $1B trading volume: $5M needed from other sources
```

**Best For:** Early-stage growth, crypto-native markets

### 2.2 Option B: Percentage of Winnings

**Structure:**
- No fee to enter positions
- Fee charged only on net profits when position closes profitably

**Example Fee Schedule:**
| Net Profit | Fee Rate |
|------------|----------|
| < $100 | 0% |
| $100 - $1,000 | 1% |
| $1,000 - $10,000 | 1.5% |
| $10,000 - $100,000 | 2% |
| > $100,000 | 2.5% |

**Pros:**
- Fair - only pay when you win
- Lower barrier to entry
- Aligns platform with user success

**Cons:**
- Revenue volatility
- Complex accounting
- Users may feel "taxed" on success

**Benchmark:** BetFair and Smarkets charge 2% on winnings

### 2.3 Option C: Spread-Based Pricing

**Structure:**
- Platform earns on bid-ask spread
- No explicit trading fee
- Revenue scales with volume

**Example:**
```
Market: "Will X happen?"
- True probability: 50%
- Bid price: 49¢
- Ask price: 51¢
- Spread: 2¢ (2% implied fee)
```

**Spread by Liquidity Level:**
| Liquidity Depth | Typical Spread | Implied Fee |
|-----------------|----------------|-------------|
| High ($1M+) | 0.5-1% | 0.25-0.5% |
| Medium ($100K-$1M) | 1-2% | 0.5-1% |
| Low (< $100K) | 2-5% | 1-2.5% |

**Pros:**
- Invisible to users
- Scales naturally with volume
- Rewards liquidity providers

**Cons:**
- Less transparent
- Requires active market-making
- Spread varies by market

### 2.4 Option D: Tiered Pricing Model

**Structure:**
| Tier | Monthly Volume | Fee Rate | Benefits |
|------|---------------|----------|----------|
| Standard | < $10K | 0.5% | Basic access |
| Active | $10K - $100K | 0.3% | Priority support |
| Pro | $100K - $1M | 0.15% | Advanced tools |
| VIP | > $1M | 0.05% | Dedicated manager |

**Volume Incentives:**
```
Example: $500K monthly trading volume

Standard Rate: 0.5% × $500K = $2,500 fees
Tiered Rate:
  - First $10K @ 0.5% = $50
  - Next $90K @ 0.3% = $270
  - Next $400K @ 0.15% = $600
  - Total: $920 (0.18% effective)

Savings: $1,580 (63% reduction)
```

**Pros:**
- Rewards loyalty
- Predictable for high-volume users
- Encourages more trading

**Cons:**
- Complex to communicate
- May alienate small traders
- Requires tracking infrastructure

### 2.5 Option E: Kalshi-Style Probability-Based Fees

**Structure:**
Fee varies based on outcome probability to ensure no bet is a guaranteed loser.

**Formula:**
```
Fee = Base Rate × Contracts × Price × (1 - Price)

Where:
- Base Rate: 7% (Kalshi's rate)
- Price: Contract price (0-100%)

Example at 50% probability:
Fee = 0.07 × 1 × 0.50 × 0.50 = $0.0175 per contract

Example at 90% probability:
Fee = 0.07 × 1 × 0.90 × 0.10 = $0.0063 per contract
```

**Fee by Probability:**
| Probability | Fee per $1 Contract | Effective % |
|-------------|---------------------|-------------|
| 10%/90% | $0.0063 | 0.63% |
| 20%/80% | $0.0112 | 1.12% |
| 30%/70% | $0.0147 | 1.47% |
| 40%/60% | $0.0168 | 1.68% |
| 50%/50% | $0.0175 | 1.75% |

**Pros:**
- Mathematically fair
- Prevents arbitrage
- Higher revenue from balanced markets

**Cons:**
- Complex to explain
- Fees higher on popular markets
- User confusion

---

## 3. Competitive Pricing Analysis

### 3.1 Market Landscape

| Platform | Fee Model | Effective Rate | Market Focus |
|----------|-----------|----------------|--------------|
| Polymarket (Global) | Zero fees | 0% | Crypto, Global |
| Polymarket (US) | Flat taker | 0.01% | US Regulated |
| Kalshi | Variable | ~1.2% avg | US Regulated |
| BetFair | Commission on wins | 2% | Sports, UK |
| Smarkets | Commission on wins | 2% | Sports, UK |
| PredictIt | Fee + withdrawal | 10%+ effective | US (limited) |
| Crypto.com | Flat fee | $0.02/contract | Crypto |

### 3.2 Fee Comparison (Per $100 Traded)

```
Platform Fee Comparison ($100 traded, 50% probability):

Polymarket Global:  $0.00  ████████████████████████████████████ FREE
Polymarket US:      $0.01  ▌
Crypto.com:         $2.00  ██
Smarkets:           $2.00  ██ (on winnings only)
BetFair:            $2.00  ██ (on winnings only)
Kalshi:             $1.20  █▌ (average)
PredictIt:         $10.00+ ████████████████████████████████████████████████
```

### 3.3 Competitive Response Strategy

**If Competing on Price:**
```
Target: Undercut Kalshi by 50%+
Strategy: 0.5% flat fee or 0.01% Polymarket-style

Position: "Trade more, pay less"
```

**If Competing on Value:**
```
Target: Match Kalshi, exceed on features
Strategy: 1% fee with superior analytics, UX, markets

Position: "Premium experience, fair pricing"
```

**If Competing on Access:**
```
Target: Underserved markets (non-US, specific verticals)
Strategy: Zero fees, geographic/vertical focus

Position: "Prediction markets for everyone"
```

---

## 4. Premium Tier Features

### 4.1 Tier Structure

#### Free Tier
**Price:** $0

**Features:**
- Basic trading (up to $10K/month)
- Standard market access
- Basic mobile app
- Community support
- 2-day settlement
- Basic portfolio view

#### Pro Tier
**Price:** $29/month or $290/year (17% discount)

**Features:**
- Everything in Free, plus:
- Unlimited trading volume
- Advanced charting & analytics
- API access (1,000 calls/day)
- Priority customer support
- Same-day settlement
- Portfolio analytics dashboard
- Price alerts (unlimited)
- Custom watchlists
- Export trade history

#### Premium Tier
**Price:** $99/month or $990/year (17% discount)

**Features:**
- Everything in Pro, plus:
- API access (10,000 calls/day)
- Real-time data feeds
- Advanced order types
- Market creation (5/month)
- Dedicated account manager
- 1-hour settlement
- Custom analytics reports
- Early access to new features
- Discord VIP channel

#### Enterprise Tier
**Price:** $499/month or custom

**Features:**
- Everything in Premium, plus:
- Unlimited API access
- Custom integrations
- White-label options
- Dedicated infrastructure
- SLA guarantees (99.9% uptime)
- Custom market creation
- Bulk data exports
- Compliance reports
- Legal support

### 4.2 Feature Comparison Matrix

| Feature | Free | Pro | Premium | Enterprise |
|---------|------|-----|---------|------------|
| Monthly Trading Limit | $10K | Unlimited | Unlimited | Unlimited |
| API Calls/Day | 0 | 1,000 | 10,000 | Unlimited |
| Market Creation | No | No | 5/month | Unlimited |
| Settlement Speed | 2 days | Same day | 1 hour | Instant |
| Support Level | Community | Priority | Dedicated | 24/7 Phone |
| Analytics | Basic | Advanced | Custom | Enterprise |
| Price Alerts | 5 | Unlimited | Unlimited | Unlimited |
| Watchlists | 1 | 10 | Unlimited | Unlimited |
| Fee Discount | 0% | 10% | 25% | Custom |

### 4.3 Conversion Funnel Targets

| Stage | Conversion Rate | Target Users |
|-------|-----------------|--------------|
| Visitor → Free | 5% | 50,000 |
| Free → Pro | 5% | 2,500 |
| Pro → Premium | 15% | 375 |
| Premium → Enterprise | 5% | 19 |

**Monthly Revenue (at target):**
- Pro: 2,500 × $29 = $72,500
- Premium: 375 × $99 = $37,125
- Enterprise: 19 × $499 = $9,481
- **Total: $119,106/month**

---

## 5. B2B/Enterprise Pricing

### 5.1 White-Label Licensing

**Tier 1: Startup License**
- **Setup Fee:** $25,000
- **Monthly Fee:** $5,000
- **Includes:**
  - Core platform deployment
  - Basic customization (logo, colors)
  - Up to 10,000 users
  - Standard support
  - Basic analytics

**Tier 2: Growth License**
- **Setup Fee:** $75,000
- **Monthly Fee:** $15,000
- **Includes:**
  - Full platform deployment
  - Advanced customization
  - Up to 100,000 users
  - Priority support
  - Full analytics suite
  - Custom domain

**Tier 3: Enterprise License**
- **Setup Fee:** $200,000+
- **Monthly Fee:** $50,000+
- **Includes:**
  - Complete white-label solution
  - Full source code access
  - Unlimited users
  - Dedicated support team
  - Custom feature development
  - On-premise option
  - SLA guarantees

### 5.2 API Partnerships

| Tier | Monthly Price | Rate Limits | Support |
|------|--------------|-------------|---------|
| Developer | $99 | 10K calls/day | Email |
| Business | $499 | 100K calls/day | Priority |
| Enterprise | $2,499 | 1M calls/day | Dedicated |
| Unlimited | Custom | Unlimited | 24/7 |

**Usage-Based Overage:**
- Developer: $0.001 per additional call
- Business: $0.0005 per additional call
- Enterprise: $0.0001 per additional call

### 5.3 Custom Market Creation

| Service | Price | Turnaround |
|---------|-------|------------|
| Standard Market | $500 | 24 hours |
| Complex Market (multi-outcome) | $1,500 | 48 hours |
| Private Market (invite-only) | $2,500 | 24 hours |
| Enterprise Program (10+/month) | $3,000/month | Same day |

**Custom Market Features:**
- Custom resolution criteria
- Branded market pages
- Embedded widgets
- Custom liquidity pools
- API integration

### 5.4 Data Licensing

| Product | Monthly Price | Description |
|---------|--------------|-------------|
| Historical Data | $1,000 | All historical prices, volumes |
| Real-time Feed | $2,500 | Live price updates |
| Research Dataset | $5,000 | Cleaned data for analysis |
| Full Data Package | $7,500 | All data products |
| Custom Data Solutions | Negotiated | Bespoke requirements |

---

## 6. Promotional Pricing

### 6.1 Launch Promotions

**Early Adopter Program:**
```
First 1,000 users:
- 6 months Pro tier free
- Founding member badge
- Lifetime 25% fee discount
- Direct feedback channel to team

First 10,000 users:
- 3 months Pro tier free
- Early adopter badge
- 10% permanent fee discount
```

**Launch Week Specials:**
- Zero fees for first 7 days
- Double referral bonuses
- Free market creation

### 6.2 Loyalty Rewards

**Trading Rewards Program:**
| Monthly Volume | Reward |
|---------------|--------|
| $10K - $50K | 5% fee cashback |
| $50K - $100K | 10% fee cashback |
| $100K - $500K | 15% fee cashback |
| $500K - $1M | 20% fee cashback |
| > $1M | 25% fee cashback + VIP status |

**Streak Bonuses:**
- 7 days active: 5% bonus on next trade
- 30 days active: 10% bonus
- 90 days active: 15% bonus + Pro trial
- 365 days active: Lifetime discount

### 6.3 Volume Discounts

**Tiered Volume Pricing:**
```
Standard Rate: 0.5%

Volume Tiers (Monthly):
$0 - $50K:          0.50%
$50K - $250K:       0.40% (20% off)
$250K - $1M:        0.30% (40% off)
$1M - $5M:          0.20% (60% off)
$5M+:               0.10% (80% off)
```

**Prepaid Volume Packages:**
| Package | Volume | Price | Effective Rate | Savings |
|---------|--------|-------|----------------|---------|
| Starter | $100K | $400 | 0.40% | 20% |
| Trader | $500K | $1,500 | 0.30% | 40% |
| Pro | $2M | $4,000 | 0.20% | 60% |
| Whale | $10M | $10,000 | 0.10% | 80% |

### 6.4 Referral Program

**Two-Sided Rewards:**
```
Referrer Rewards:
- $25 credit when referee completes first trade
- 10% of referee's fees for 6 months
- Bonus tiers: 10+ referrals = VIP status

Referee Rewards:
- $25 trading credit
- First month Pro free
- Zero fees for first $1,000 traded
```

**Referral Tiers:**
| Referrals | Bonus | Status |
|-----------|-------|--------|
| 1-5 | $25 each | Advocate |
| 6-15 | $35 each | Champion |
| 16-50 | $50 each | Ambassador |
| 50+ | $75 each + 1% rev share | Partner |

---

## 7. Price Testing Methodology

### 7.1 A/B Testing Framework

**Test Structure:**
```
Control Group (A): Current pricing
Test Group (B): New pricing variant
Test Group (C): Alternative variant (optional)

Sample Size: Minimum 1,000 users per group
Duration: 2-4 weeks minimum
Significance: 95% confidence level
```

**Key Metrics to Track:**
1. Conversion rate (signup → first trade)
2. Trading volume per user
3. Revenue per user
4. Retention rate (7-day, 30-day)
5. Feature adoption
6. Customer satisfaction (NPS)

### 7.2 Price Sensitivity Analysis

**Van Westendorp Price Sensitivity:**
Survey questions:
1. At what price would you consider the product too expensive?
2. At what price would you consider it a bargain?
3. At what price would you start to doubt quality?
4. At what price is it getting expensive but still acceptable?

**Conjoint Analysis:**
Test feature combinations:
```
Example Conjoint Sets:

Option A:
- Price: $29/month
- API calls: 1,000/day
- Support: Email
- Settlement: Same day

Option B:
- Price: $49/month
- API calls: 5,000/day
- Support: Priority
- Settlement: 1 hour

Option C:
- Price: $19/month
- API calls: 500/day
- Support: Community
- Settlement: 2 days
```

### 7.3 Pricing Experiments Calendar

| Quarter | Experiment | Goal |
|---------|------------|------|
| Q1 | Fee rate testing (0.3% vs 0.5%) | Optimize take rate |
| Q1 | Pro tier pricing ($29 vs $39) | Revenue maximization |
| Q2 | Volume discount thresholds | Increase trading volume |
| Q2 | Annual vs monthly pricing | Improve retention |
| Q3 | Enterprise pricing tiers | B2B revenue |
| Q3 | Referral reward amounts | Acquisition efficiency |
| Q4 | Premium feature bundling | Upsell conversion |
| Q4 | Geographic pricing | International expansion |

### 7.4 Implementation Process

**Phase 1: Hypothesis (Week 1)**
- Define hypothesis (e.g., "Lowering Pro price to $19 will increase conversions by 50%")
- Identify success metrics
- Calculate required sample size
- Get stakeholder alignment

**Phase 2: Setup (Week 2)**
- Configure A/B test infrastructure
- Set up tracking and analytics
- Create pricing variations
- QA testing

**Phase 3: Execute (Weeks 3-6)**
- Launch test to segment
- Monitor for anomalies
- Avoid peeking at results
- Document observations

**Phase 4: Analyze (Week 7)**
- Statistical significance testing
- Segment analysis
- Long-term impact projection
- ROI calculation

**Phase 5: Decide (Week 8)**
- Present findings
- Make go/no-go decision
- Plan rollout or iteration
- Document learnings

### 7.5 Pricing Analytics Dashboard

**Real-Time Metrics:**
```
┌─────────────────────────────────────────────────────────┐
│ PRICING DASHBOARD                                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Average Revenue Per User (ARPU)                        │
│  ├── Free: $0.50/month                                  │
│  ├── Pro: $32.00/month                                  │
│  ├── Premium: $115.00/month                             │
│  └── Blended: $8.50/month                               │
│                                                         │
│  Conversion Rates                                       │
│  ├── Visitor → Free: 5.2%                               │
│  ├── Free → Pro: 4.8%                                   │
│  ├── Pro → Premium: 12.3%                               │
│  └── Trial → Paid: 22.5%                                │
│                                                         │
│  Fee Revenue                                            │
│  ├── Today: $12,450                                     │
│  ├── This Week: $78,320                                 │
│  ├── This Month: $312,450                               │
│  └── Take Rate: 0.42%                                   │
│                                                         │
│  Price Elasticity                                       │
│  └── Current: -1.3 (elastic)                            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 8. Recommended Pricing Strategy

### 8.1 Phase 1: Launch (Months 1-6)

**Strategy: Penetration Pricing**

```
Trading Fees: 0% (zero fees)
Premium Tiers:
- Free: Full access
- Pro: $19/month (introductory)

Focus: User acquisition, volume growth
Target: 50,000 users, $100M+ monthly volume
```

### 8.2 Phase 2: Growth (Months 7-18)

**Strategy: Value-Based Tiering**

```
Trading Fees: 0.25% (introduced)
Premium Tiers:
- Free: Limited features
- Pro: $29/month
- Premium: $99/month

Focus: Revenue activation, premium conversion
Target: 200,000 users, $500M+ monthly volume
```

### 8.3 Phase 3: Maturity (Months 19+)

**Strategy: Optimized Portfolio**

```
Trading Fees: 0.3-0.5% (tiered by volume)
Premium Tiers:
- Free: Basic
- Pro: $39/month
- Premium: $149/month
- Enterprise: $499+/month

Focus: Revenue optimization, enterprise growth
Target: 500,000+ users, $1B+ monthly volume
```

---

## 9. Appendix

### 9.1 Pricing Communication Templates

**Fee Increase Announcement:**
```
Subject: Updates to Our Pricing

Dear [User],

We're making some changes to our pricing effective [Date].

What's changing:
- Trading fees will be [X]% (previously [Y]%)
- Pro tier will be $[X]/month (previously $[Y])

Why:
- Continued investment in platform security
- New features including [X, Y, Z]
- Improved customer support

What stays the same:
- Free tier access
- Your current subscription (grandfathered until [Date])

Questions? Reply to this email or visit [FAQ link].

Thank you for being part of our community.
```

### 9.2 Competitive Pricing Monitor

**Monthly Competitive Check:**
- [ ] Polymarket fee changes
- [ ] Kalshi pricing updates
- [ ] New market entrants
- [ ] Traditional betting site fees
- [ ] Crypto exchange fees

### 9.3 Pricing Decision Matrix

| Scenario | Recommended Action |
|----------|-------------------|
| Competitor lowers fees | Monitor for 30 days, match if volume drops >10% |
| Volume growth slowing | Test promotional pricing or fee reduction |
| High churn in paid tiers | A/B test lower prices or enhanced features |
| Enterprise demand increasing | Accelerate enterprise tier development |
| Regulatory changes | Adjust pricing for compliance costs |

---

## References

- Polymarket fee structure via official documentation
- Kalshi fee revenue ($263.5M in 2025) - Yahoo Finance
- BetFair/Smarkets 2% commission benchmark - Industry analysis
- Prediction market fee impact on prices - Karl Whelan academic research
- SaaS pricing benchmarks - OpenView Partners, Profitwell
