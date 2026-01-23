# Treasury Management

## Executive Summary

This document outlines comprehensive treasury management practices for a Polymarket-like prediction market platform, covering treasury structure, reserve management, yield strategies, multi-sig setup, hedging, reporting, governance, and emergency procedures. Based on best practices from leading crypto protocols including MakerDAO, Uniswap, and industry-standard treasury management frameworks.

---

## 1. Treasury Structure

### 1.1 Overview

A well-designed treasury structure separates funds by purpose, risk profile, and access requirements. This ensures operational efficiency while maintaining security and regulatory compliance.

```
┌─────────────────────────────────────────────────────────────────┐
│                    TREASURY ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  OPERATING   │  │   RESERVE    │  │  INVESTMENT  │          │
│  │   WALLET     │  │   WALLET     │  │   WALLET     │          │
│  │              │  │              │  │              │          │
│  │  Hot Wallet  │  │  Warm Wallet │  │  Cold Wallet │          │
│  │  Daily ops   │  │  User funds  │  │  Long-term   │          │
│  │  5-10% total │  │  70-80%      │  │  15-20%      │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│         │                 │                 │                   │
│         └────────────────┼─────────────────┘                   │
│                          │                                      │
│                  ┌───────┴───────┐                              │
│                  │   MASTER      │                              │
│                  │   TREASURY    │                              │
│                  │   MULTI-SIG   │                              │
│                  └───────────────┘                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Operating Wallet

**Purpose:** Day-to-day operational expenses and automated transactions

**Characteristics:**
- Hot wallet with programmatic access
- Holds 5-10% of total treasury
- Replenished weekly from Reserve
- Lower security threshold (2-of-3 multi-sig)

**Permitted Uses:**
- Gas fees for smart contract operations
- Payroll disbursements
- Vendor payments
- Small user withdrawals (<$10K)
- Marketing expenses
- Automated market making operations

**Balance Targets:**

| Stage | Operating Balance | Monthly Burn Coverage |
|-------|-------------------|----------------------|
| Early | $100K-$500K | 2-3 months |
| Growth | $500K-$2M | 1-2 months |
| Scale | $2M-$10M | 1 month |

**Controls:**
```
Daily spending limit: $50K
Single transaction limit: $25K
Signers required: 2-of-3
Replenishment: Weekly (automated when balance < $100K)
Monitoring: Real-time alerts for transactions > $5K
```

### 1.3 Reserve Wallet

**Purpose:** User deposits, trading collateral, and platform liquidity

**Characteristics:**
- Warm wallet with controlled access
- Holds 70-80% of total treasury
- Higher security threshold (3-of-5 multi-sig)
- Backed 1:1 with user deposits + buffer

**Permitted Uses:**
- User withdrawal processing
- Market settlement payouts
- Liquidity provision
- Emergency operating fund replenishment
- Regulatory reserve requirements

**Reserve Composition:**

| Asset | Target Allocation | Rationale |
|-------|-------------------|-----------|
| USDC | 60-70% | Primary settlement currency |
| USDT | 15-20% | Diversification |
| DAI | 5-10% | Decentralized alternative |
| ETH | 5% | Gas reserves |

**Minimum Reserve Requirements:**
```
User Deposits Coverage: 100% at all times
Platform Liability Buffer: +10% additional
Operating Runway: +3 months expenses
Regulatory Buffer: +5% (jurisdiction dependent)

Example:
- User deposits: $10M
- Buffer (10%): $1M
- Operating (3 mo): $1.5M
- Regulatory (5%): $500K
- Total Reserve Minimum: $13M
```

### 1.4 Investment Wallet

**Purpose:** Long-term value preservation and yield generation on excess funds

**Characteristics:**
- Cold storage with maximum security
- Holds 15-20% of total treasury
- Highest security threshold (4-of-7 multi-sig)
- Time-locks on withdrawals (24-72 hours)

**Permitted Uses:**
- Yield-generating strategies (approved list only)
- Strategic investments
- Protocol token holdings
- Long-term reserves
- Insurance fund

**Investment Policy:**
```
Risk Tolerance: Conservative
Yield Target: 3-8% APY
Maximum single position: 25% of investment wallet
Approved protocols: [Whitelist maintained by Treasury Committee]
Review frequency: Monthly
```

---

## 2. USDC Reserve Management

### 2.1 Minimum Reserve Ratios

**Reserve Ratio Framework:**

| Category | Ratio | Definition |
|----------|-------|------------|
| Minimum Reserve | 100% | Total USDC >= Total User Deposits |
| Operating Reserve | 105% | Includes operational buffer |
| Target Reserve | 110% | Comfortable operating level |
| Maximum Reserve | 130% | Excess above this deployed to yield |

**Real-Time Monitoring:**
```
┌─────────────────────────────────────────────────────────┐
│               RESERVE HEALTH DASHBOARD                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Current Reserve Ratio: 112%      [████████████░░] ✓   │
│                                                         │
│  Total User Deposits:     $15,000,000                   │
│  Total USDC Reserve:      $16,800,000                   │
│  Buffer Amount:           $1,800,000                    │
│                                                         │
│  Status: HEALTHY                                        │
│                                                         │
│  Alerts:                                                │
│  ├── < 105%: Warning (email)                            │
│  ├── < 100%: Critical (all hands)                       │
│  └── > 130%: Deploy excess (treasury team)              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Liquidity Buffer Management

**Withdrawal Demand Modeling:**
```
Historical Analysis:
├── Average daily withdrawals: 2-3% of deposits
├── Peak daily withdrawals: 8-10% of deposits
├── Black swan events: Up to 25% in 24 hours

Buffer Sizing:
├── Normal operations: 5% liquid USDC
├── Elevated alert: 15% liquid USDC
├── Crisis mode: 30%+ liquid USDC
```

**Liquidity Tiers:**

| Tier | Amount | Availability | Location |
|------|--------|--------------|----------|
| Tier 1 | $1M | Instant | Operating wallet |
| Tier 2 | $5M | < 1 hour | Reserve wallet |
| Tier 3 | $10M | < 24 hours | Yield positions (liquid) |
| Tier 4 | $5M+ | 24-72 hours | Investment wallet |

### 2.3 Stablecoin Diversification

**Why Diversify:**
- Single stablecoin risk (USDC brief depeg in March 2023)
- Counterparty risk reduction
- Regulatory risk mitigation
- Liquidity access across chains

**Target Allocation:**

| Stablecoin | Allocation | Rationale |
|------------|------------|-----------|
| USDC | 60% | Primary, Circle-backed |
| USDT | 20% | Highest liquidity, Tether |
| DAI | 15% | Decentralized, MakerDAO |
| FRAX/Other | 5% | Diversification |

**Rebalancing Rules:**
```
Trigger rebalance when:
├── Any stablecoin > 70% of total
├── Any stablecoin < 10% of target
├── Depeg > 0.5% for > 1 hour
└── Regulatory news affecting issuer

Rebalancing constraints:
├── Maximum slippage: 0.1%
├── Execution window: 24-48 hours
├── Use DEX aggregators for best rates
```

---

## 3. Yield Strategies for Idle Funds

### 3.1 Approved Yield Strategies

**Tier 1 - Lowest Risk (Core Reserves):**

| Strategy | Expected APY | Risk Level | Max Allocation |
|----------|--------------|------------|----------------|
| Circle Yield | 3-4% | Very Low | 40% |
| Coinbase USDC | 2-4% | Very Low | 30% |
| Treasury Bills (via protocols) | 4-5% | Very Low | 30% |

**Tier 2 - Low Risk (Excess Reserves):**

| Strategy | Expected APY | Risk Level | Max Allocation |
|----------|--------------|------------|----------------|
| Aave V3 (USDC) | 4-8% | Low | 25% |
| Compound V3 | 4-7% | Low | 25% |
| MakerDAO DSR (DAI) | 5-8% | Low | 20% |
| Morpho Optimizer | 5-10% | Low | 15% |

**Tier 3 - Medium Risk (Investment Wallet Only):**

| Strategy | Expected APY | Risk Level | Max Allocation |
|----------|--------------|------------|----------------|
| Curve + Convex | 5-12% | Medium | 10% |
| Yearn V3 | 5-15% | Medium | 10% |
| Pendle Fixed Rate | 6-12% | Medium | 10% |

### 3.2 Risk Assessment Framework

**Protocol Risk Scoring:**

| Factor | Weight | Scoring Criteria |
|--------|--------|------------------|
| Time in Market | 20% | < 1 year: 0, 1-2 years: 50, 2+ years: 100 |
| TVL | 15% | < $100M: 0, $100M-$1B: 50, > $1B: 100 |
| Audit Status | 25% | None: 0, 1 audit: 50, Multiple top firms: 100 |
| Insurance | 15% | None: 0, Partial: 50, Full: 100 |
| Track Record | 25% | Hacks: 0, Near-misses: 50, Clean: 100 |

**Minimum Score for Deployment:**
- Tier 1 strategies: Score >= 90
- Tier 2 strategies: Score >= 75
- Tier 3 strategies: Score >= 60

### 3.3 Yield Strategy Implementation

**Deployment Process:**
```
1. Proposal
   └── Treasury team proposes strategy
   └── Risk assessment completed
   └── Expected return vs. risk documented

2. Approval
   └── Treasury Committee review (score >= threshold)
   └── Technical review (smart contract interaction)
   └── Multi-sig approval (Investment wallet signers)

3. Execution
   └── Small test transaction ($10K)
   └── 24-hour monitoring
   └── Full deployment in tranches

4. Monitoring
   └── Daily yield tracking
   └── Weekly position review
   └── Monthly strategy reassessment

5. Exit Triggers
   └── APY drops below threshold
   └── Risk score deteriorates
   └── Protocol incident reported
   └── Better opportunity identified
```

### 3.4 Expected Yields Summary

**Blended Yield Target:**
```
Conservative Portfolio:
├── 50% @ 3% (Tier 1) = 1.5%
├── 35% @ 6% (Tier 2) = 2.1%
├── 15% @ 10% (Tier 3) = 1.5%
└── Blended Yield: ~5.1% APY

On $20M excess reserves = ~$1M annual yield
```

---

## 4. Multi-Sig Setup

### 4.1 Multi-Sig Architecture

**Recommended: Gnosis Safe (Safe)**

Safe is the industry standard for institutional crypto treasury management, securing over $100B in assets and used by major protocols including Uniswap, Aave, and MakerDAO.

**Wallet Structure:**

| Wallet | Threshold | Signers | Purpose |
|--------|-----------|---------|---------|
| Operating | 2-of-3 | Core team | Daily operations |
| Reserve | 3-of-5 | Expanded team | User funds |
| Investment | 4-of-7 | Full committee | Long-term holdings |
| Emergency | 2-of-3 | Emergency team | Crisis response |

### 4.2 Signer Requirements

**Operating Wallet (2-of-3):**
- CEO
- CFO
- Head of Operations

**Reserve Wallet (3-of-5):**
- CEO
- CFO
- CTO
- Head of Operations
- Lead Engineer

**Investment Wallet (4-of-7):**
- CEO
- CFO
- CTO
- General Counsel
- Head of Operations
- Board Representative
- External Advisor

### 4.3 Threshold Settings

**Threshold Selection Rationale:**
```
Threshold = n-of-m where:
├── n = minimum signers required
├── m = total signers
├── n should be > 50% of m (majority)
├── m - n = fault tolerance (signers who can be unavailable)

Examples:
├── 2-of-3: Majority control, 1 signer redundancy
├── 3-of-5: Majority control, 2 signer redundancy
├── 4-of-7: Majority control, 3 signer redundancy

Balance:
├── Higher threshold = more security
├── Lower threshold = more operational efficiency
├── Consider: availability zones, time zones, key loss risk
```

**Threshold by Transaction Type:**

| Transaction | Threshold | Delay |
|-------------|-----------|-------|
| < $10K | 2-of-3 | None |
| $10K - $100K | 3-of-5 | None |
| $100K - $1M | 4-of-7 | 24 hours |
| > $1M | 4-of-7 | 48 hours |
| Smart contract upgrade | 5-of-7 | 72 hours |

### 4.4 Key Management

**Hardware Wallet Requirements:**
- All signers must use hardware wallets (Ledger or Trezor)
- No browser extensions for treasury operations
- Firmware kept up to date
- Recovery seeds stored securely (never digitally)

**Key Storage Best Practices:**
```
DO:
├── Store recovery seeds in fireproof safes
├── Use geographic distribution (different cities)
├── Consider bank safety deposit boxes
├── Implement Shamir's Secret Sharing for critical keys
├── Document recovery procedures

DON'T:
├── Store seeds digitally (no photos, no cloud)
├── Keep all seeds in one location
├── Share seeds via any electronic medium
├── Use personal devices for treasury signing
```

**Key Ceremony Protocol:**
```
Initial Setup:
1. Generate keys on airgapped devices
2. Verify addresses with multiple team members
3. Test small transaction before funding
4. Document all addresses and verify checksums
5. Distribute seed phrase backups to secure locations

Key Rotation (Annual):
1. Generate new signer wallets
2. Update Safe signer list (requires current threshold)
3. Remove old signers
4. Verify new setup with test transactions
5. Update documentation
```

### 4.5 Delegate Configuration

**Delegates:** Non-owners authorized to propose (but not approve) transactions

**Use Cases:**
- Operations team can queue regular payments
- Automated systems can propose transactions
- Reduces burden on signers for routine operations

**Setup:**
```
Delegate Roles:
├── Payroll Delegate: Can propose salary payments
├── Operations Delegate: Can propose vendor payments
├── Tech Delegate: Can propose infrastructure costs
└── Emergency Delegate: Can propose emergency transactions

Constraints:
├── Delegates cannot approve transactions
├── All delegate proposals require signer approval
├── Delegate list reviewed quarterly
```

---

## 5. Hedging Strategies

### 5.1 Stablecoin Diversification (Primary Hedge)

**Risk: Single stablecoin failure**

**Mitigation:**
```
Portfolio Approach:
├── 60% USDC (Circle)
├── 20% USDT (Tether)
├── 15% DAI (MakerDAO)
├── 5% Other (FRAX, etc.)

Monitoring:
├── Peg health (< 0.5% deviation acceptable)
├── Issuer financials (quarterly review)
├── Regulatory news (real-time alerts)
├── Reserve composition changes
```

### 5.2 Currency Risk Hedging

**Risk: Operational expenses in different currencies (USD, EUR, etc.)**

**Strategies:**
```
Natural Hedge:
├── Match revenue currency with expense currency
├── Hold reserves in currencies needed for operations
└── Invoice in stablecoins where possible

Forward Hedging (if significant non-USD exposure):
├── Use DeFi forex protocols
├── Traditional FX forwards (if banked)
├── Rolling 3-6 month hedges
└── Target: 50-80% of projected exposure hedged
```

### 5.3 Interest Rate Risk

**Risk: Variable yield strategies underperform in low-rate environments**

**Strategies:**
```
Fixed vs. Variable Mix:
├── 50% in fixed-rate positions (e.g., Pendle PT tokens)
├── 50% in variable positions (Aave, Compound)
└── Review and rebalance quarterly

Duration Management:
├── Short duration (< 3 months) for operating needs
├── Medium duration (3-12 months) for reserve buffer
├── Longer duration OK for investment wallet
```

### 5.4 Smart Contract Risk

**Risk: Protocol hack or exploit affecting deposited funds**

**Mitigation Layers:**
```
1. Protocol Selection
   └── Only audited, battle-tested protocols
   └── Minimum $500M TVL for Tier 1 allocations

2. Diversification
   └── No single protocol > 25% of deployed capital
   └── Spread across protocol types

3. Insurance
   └── Nexus Mutual coverage for major positions
   └── Budget 1-2% of position value annually

4. Monitoring
   └── Real-time protocol health dashboards
   └── DeFi Llama, DeBank alerts
   └── Automated position monitoring
```

### 5.5 Operational Risk Hedging

**Risk: Key person unavailability, system failures**

**Mitigation:**
```
Key Person Risk:
├── 3+ signers in different jurisdictions
├── Written procedures for all treasury operations
├── Cross-training on all critical functions
├── Succession plan documented

System Risk:
├── Multiple RPC providers
├── Backup hardware wallets
├── Redundant monitoring systems
├── Disaster recovery plan tested quarterly
```

---

## 6. Treasury Reporting

### 6.1 Internal Reporting

**Daily Report (Operations Team):**
```
Daily Treasury Status - [Date]

BALANCES:
├── Operating Wallet: $XXX,XXX
├── Reserve Wallet: $X,XXX,XXX
├── Investment Wallet: $X,XXX,XXX
└── Total: $XX,XXX,XXX

RESERVE RATIO: XXX%

TODAY'S ACTIVITY:
├── Inflows: $XXX,XXX
├── Outflows: $XXX,XXX
└── Net: +/-$XXX,XXX

ALERTS:
├── [Any warnings or issues]

YIELD POSITIONS:
├── Protocol A: $XM @ X.X% APY
├── Protocol B: $XM @ X.X% APY
└── Total Yield (24h): $X,XXX
```

**Weekly Report (Leadership):**
```
Weekly Treasury Report - Week of [Date]

SUMMARY:
├── Opening Balance: $XX,XXX,XXX
├── Closing Balance: $XX,XXX,XXX
├── Net Change: +/-$X,XXX,XXX

RESERVE HEALTH:
├── Average Reserve Ratio: XXX%
├── Minimum (week): XXX%
├── Maximum (week): XXX%

OPERATING EXPENSES:
├── Payroll: $XXX,XXX
├── Infrastructure: $XX,XXX
├── Marketing: $XX,XXX
├── Other: $XX,XXX
└── Total: $XXX,XXX

YIELD PERFORMANCE:
├── Gross Yield: $XX,XXX
├── Annualized APY: X.X%
├── vs. Target: +/-X.X%

KEY DECISIONS NEEDED:
├── [Any pending approvals]
```

**Monthly Report (Board/Investors):**
```
Monthly Treasury Report - [Month Year]

EXECUTIVE SUMMARY:
[2-3 sentence overview]

FINANCIAL POSITION:
┌─────────────────────────────────────────┐
│  Total Assets:           $XX,XXX,XXX    │
│  Total Liabilities:      $XX,XXX,XXX    │
│  Net Position:           $XX,XXX,XXX    │
│  Reserve Ratio:          XXX%           │
│  Runway (months):        XX             │
└─────────────────────────────────────────┘

ASSET ALLOCATION:
├── Cash & Equivalents: XX%
├── Yield Positions: XX%
├── Strategic Holdings: XX%

YIELD PERFORMANCE:
├── Monthly Yield: $XXX,XXX
├── YTD Yield: $X,XXX,XXX
├── Annualized Return: X.X%

RISK METRICS:
├── Largest Single Position: XX%
├── Protocol Concentration: [Compliant/Non-compliant]
├── Insurance Coverage: $XXM

REGULATORY:
├── License Status: [Current]
├── Reserve Requirements: [Met/Not Met]

UPCOMING:
├── [Planned major transactions]
├── [Policy changes]
```

### 6.2 External Reporting (If Required)

**Proof of Reserves:**
- Published monthly (minimum)
- Third-party attestation (quarterly)
- On-chain verification where possible

**Regulatory Reports:**
- Per jurisdiction requirements
- Maintain audit trail for all transactions
- Retain records for 7+ years

### 6.3 Dashboard & Monitoring

**Real-Time Monitoring Tools:**
- **DeBank/Zapper:** Portfolio tracking across chains
- **DeFi Llama:** Protocol health and TVL monitoring
- **Safe {Wallet}:** Multi-sig transaction queue
- **Custom Dashboard:** Reserve ratio, alerts, KPIs

**Alert Thresholds:**

| Metric | Warning | Critical |
|--------|---------|----------|
| Reserve Ratio | < 110% | < 100% |
| Single Position | > 20% | > 30% |
| 24h Outflows | > $1M | > $5M |
| Protocol TVL Drop | > 10% | > 25% |
| Stablecoin Depeg | > 0.3% | > 1% |

---

## 7. Governance Over Treasury

### 7.1 Treasury Committee

**Composition:**
- CFO (Chair)
- CEO
- CTO
- General Counsel
- Head of Operations
- External Advisor (optional)

**Responsibilities:**
- Approve treasury policy changes
- Review and approve yield strategies
- Oversee risk management
- Authorize transactions above threshold
- Monthly treasury review

**Meeting Cadence:**
- Weekly: Operations review (30 min)
- Monthly: Full treasury review (1 hour)
- Quarterly: Policy and strategy review (2 hours)
- Ad-hoc: Emergency situations

### 7.2 Decision Framework

**Transaction Approval Matrix:**

| Amount | Required Approval |
|--------|-------------------|
| < $10K | Operations (2-of-3) |
| $10K - $100K | CFO + 1 Committee member |
| $100K - $500K | Treasury Committee (majority) |
| $500K - $1M | Treasury Committee (supermajority) |
| > $1M | Treasury Committee + Board |

**Policy Change Approval:**
```
Tier 1 Changes (Operational):
├── Signer rotation
├── Threshold adjustments
├── Approved protocol list updates
└── Requires: Treasury Committee majority

Tier 2 Changes (Strategic):
├── Reserve ratio changes
├── Investment policy updates
├── New strategy types
└── Requires: Treasury Committee unanimous + CEO

Tier 3 Changes (Structural):
├── New wallet creation
├── Major policy overhaul
├── Regulatory-driven changes
└── Requires: Board approval
```

### 7.3 Audit & Compliance

**Internal Audit:**
- Monthly reconciliation of all wallets
- Quarterly review of access controls
- Annual full treasury audit

**External Audit:**
- Annual financial statement audit
- Smart contract audits (as needed)
- Proof of reserves attestation (quarterly)

**Compliance Requirements:**
```
Regulatory:
├── Maintain required reserve ratios
├── File required reports
├── Respond to regulatory inquiries
├── Update licenses as needed

Internal:
├── Follow all treasury policies
├── Document all decisions
├── Maintain audit trail
├── Regular policy training
```

---

## 8. Emergency Procedures

### 8.1 Emergency Classification

| Level | Description | Examples |
|-------|-------------|----------|
| Level 1 | Minor issue, no immediate risk | Signer unavailable, minor reporting error |
| Level 2 | Significant issue, contained risk | Protocol warning, unusual activity |
| Level 3 | Critical issue, potential fund loss | Protocol exploit, regulatory action |
| Level 4 | Catastrophic, active fund loss | Direct attack, major hack |

### 8.2 Response Protocols

**Level 1 Response:**
```
1. Notify Treasury Committee
2. Implement workaround
3. Document incident
4. Review in weekly meeting
```

**Level 2 Response:**
```
1. Immediate notification to CFO + CEO
2. Assess situation (1 hour max)
3. Prepare contingency actions
4. Convene Treasury Committee if escalating
5. Execute response plan
6. Post-incident review within 48 hours
```

**Level 3 Response:**
```
1. IMMEDIATE: Alert all Committee members
2. WITHIN 15 MINUTES: Emergency call
3. WITHIN 1 HOUR:
   ├── Assess damage/exposure
   ├── Pause affected protocols (if possible)
   ├── Begin fund recovery if needed
4. WITHIN 4 HOURS:
   ├── Execute emergency plan
   ├── Prepare communications
   ├── Engage legal counsel
5. ONGOING: Hourly updates until resolved
6. POST-INCIDENT: Full review, policy updates
```

**Level 4 Response:**
```
1. IMMEDIATE: Emergency wallet activation
2. SIMULTANEOUS:
   ├── Move unaffected funds to emergency wallet
   ├── Pause all protocol interactions
   ├── Alert all signers
3. WITHIN 30 MINUTES:
   ├── Emergency Committee call
   ├── Assess total exposure
   ├── Contact affected protocols
4. WITHIN 2 HOURS:
   ├── Secure all recoverable funds
   ├── Engage security experts
   ├── Prepare public communication
5. ONGOING: Continuous management until resolved
6. POST-INCIDENT: Full forensics, policy overhaul
```

### 8.3 Emergency Contacts

```
Internal:
├── CFO: [Phone] [Telegram] [Email]
├── CEO: [Phone] [Telegram] [Email]
├── CTO: [Phone] [Telegram] [Email]
├── General Counsel: [Phone] [Email]

External:
├── Legal Counsel: [Firm] [Contact] [Phone]
├── Security Partner: [Firm] [Contact] [Phone]
├── Insurance Provider: [Company] [Policy #] [Hotline]
├── Exchange Contacts: [List of key contacts]

Protocol Contacts:
├── Aave: [Security Contact]
├── Compound: [Security Contact]
├── Circle: [Institutional Support]
├── Safe: [Security Team]
```

### 8.4 Emergency Wallet

**Purpose:** Secure funds during crisis situations

**Characteristics:**
- Separate multi-sig (3-of-5)
- Different signers than primary wallets (subset)
- Pre-approved for emergency use only
- Tested quarterly (small test transaction)

**Emergency Wallet Protocol:**
```
Activation Criteria:
├── Level 3 or 4 emergency declared
├── OR primary wallet compromised
├── OR regulatory seizure imminent

Authorized Actions:
├── Receive funds from compromised wallets
├── Hold funds until situation resolved
├── Transfer to new primary wallets (once secure)

Prohibited Actions:
├── Deploy to any yield protocol
├── Send to any external address (except new primary)
├── Use for any operational purpose
```

---

## 9. Insurance Considerations

### 9.1 Coverage Types

**Smart Contract Cover:**
- Covers losses from protocol exploits
- Providers: Nexus Mutual, InsurAce, Unslashed

**Custodial Insurance:**
- Covers losses from custody failures
- Often included with institutional custody
- Providers: Fireblocks, Copper, BitGo

**Crime/Theft Insurance:**
- Covers internal/external theft
- Traditional insurance market
- Lloyd's of London syndicates

**Directors & Officers:**
- Protects leadership from personal liability
- Critical for regulatory situations
- Standard corporate insurance

### 9.2 Coverage Recommendations

| Coverage Type | Recommended Amount | Estimated Cost |
|---------------|-------------------|----------------|
| Smart Contract | $5-10M | 1-2% annually |
| Custodial | $10-20M | Often included |
| Crime/Theft | $5-10M | 0.5-1% annually |
| D&O | $5M | $50-100K annually |

**Total Insurance Budget:** ~$200K-400K annually for $20M+ treasury

### 9.3 Claim Process

```
1. Incident Documentation
   ├── Record all transaction hashes
   ├── Document timeline
   ├── Gather evidence of loss
   └── Calculate total exposure

2. Notification
   ├── Notify insurer within 24-48 hours
   ├── Provide initial incident report
   └── Preserve all evidence

3. Claim Filing
   ├── Complete claim forms
   ├── Submit all documentation
   ├── Cooperate with investigation

4. Resolution
   ├── Work with adjuster
   ├── Negotiate if needed
   └── Receive payout (typically 30-90 days)
```

---

## 10. Implementation Checklist

### 10.1 Initial Setup

**Week 1-2: Foundation**
- [ ] Define treasury structure (wallets, purposes)
- [ ] Select multi-sig solution (recommend Safe)
- [ ] Identify signers for each wallet
- [ ] Order hardware wallets for all signers

**Week 3-4: Wallet Setup**
- [ ] Conduct key generation ceremony
- [ ] Set up Operating wallet (2-of-3)
- [ ] Set up Reserve wallet (3-of-5)
- [ ] Set up Investment wallet (4-of-7)
- [ ] Set up Emergency wallet (3-of-5)
- [ ] Test all wallets with small transactions

**Week 5-6: Policies & Procedures**
- [ ] Document treasury policy
- [ ] Define approval thresholds
- [ ] Create emergency procedures
- [ ] Train all signers

**Week 7-8: Monitoring & Reporting**
- [ ] Set up monitoring dashboards
- [ ] Configure alerts
- [ ] Create reporting templates
- [ ] Test alert system

### 10.2 Ongoing Operations

**Daily:**
- [ ] Check reserve ratio
- [ ] Review pending transactions
- [ ] Monitor yield positions
- [ ] Check for alerts

**Weekly:**
- [ ] Produce weekly report
- [ ] Review large transactions
- [ ] Check protocol health
- [ ] Replenish operating wallet if needed

**Monthly:**
- [ ] Produce monthly report
- [ ] Full reconciliation
- [ ] Review yield strategy performance
- [ ] Treasury Committee meeting

**Quarterly:**
- [ ] External attestation (if applicable)
- [ ] Policy review
- [ ] Insurance review
- [ ] Emergency procedure test
- [ ] Signer access review

**Annually:**
- [ ] Full treasury audit
- [ ] Policy overhaul if needed
- [ ] Key rotation consideration
- [ ] Insurance renewal
- [ ] Strategic review

---

## 11. Appendix: Templates & Tools

### 11.1 Transaction Request Template

```
TREASURY TRANSACTION REQUEST

Date: [Date]
Requestor: [Name]
Wallet: [Operating/Reserve/Investment]

TRANSACTION DETAILS:
Amount: $XXX,XXX
Currency: [USDC/USDT/DAI/ETH]
Recipient: [Address]
Purpose: [Description]

CATEGORY:
[ ] Payroll
[ ] Vendor Payment
[ ] Protocol Deposit
[ ] Protocol Withdrawal
[ ] Rebalancing
[ ] Other: ____________

SUPPORTING DOCUMENTATION:
[Links or attachments]

APPROVALS REQUIRED:
[ ] Signer 1: ____________
[ ] Signer 2: ____________
[ ] Signer 3: ____________ (if needed)

NOTES:
[Any additional context]
```

### 11.2 Reserve Ratio Calculator

```
RESERVE RATIO CALCULATION

Total User Deposits:        $___________
Total Stablecoin Holdings:  $___________

RATIO = (Holdings / Deposits) × 100 = _____%

STATUS:
[ ] Critical (< 100%)
[ ] Warning (100-105%)
[ ] Healthy (105-130%)
[ ] Excess (> 130%)

RECOMMENDED ACTION:
[Based on status]
```

### 11.3 Yield Strategy Proposal Template

```
YIELD STRATEGY PROPOSAL

Date: [Date]
Proposer: [Name]

STRATEGY OVERVIEW:
Protocol: [Name]
Chain: [Ethereum/Polygon/etc.]
Asset: [USDC/DAI/etc.]
Expected APY: X.X%
Amount: $X,XXX,XXX

RISK ASSESSMENT:
Time in Market: [X years]
TVL: $[X]B
Audits: [List audits]
Insurance Available: [Yes/No]
Risk Score: [XX/100]

ALLOCATION:
Tier: [1/2/3]
% of Treasury: X%
Within Policy Limits: [Yes/No]

APPROVAL:
[ ] Technical Review Complete
[ ] Risk Assessment Complete
[ ] Treasury Committee Approval

DECISION: [Approved/Rejected]
Notes: [Any conditions]
```

---

## References

- Safe (Gnosis Safe) multi-sig documentation and best practices
- MakerDAO treasury practices and RWA investments
- Uniswap DAO treasury management ($2B+)
- DeFi protocol yield rates (Aave, Compound, MakerDAO DSR)
- Crypto treasury management guides (Request Finance, TRES Finance)
- Smart contract insurance providers (Nexus Mutual, InsurAce)
- Industry research on DAO treasury management from 101 Blockchains
