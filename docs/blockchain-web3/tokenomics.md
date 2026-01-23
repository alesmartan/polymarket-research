# Tokenomics Design Guide

## Comprehensive Token Economics for Prediction Markets

This document provides a comprehensive guide to designing tokenomics for a Polymarket-like prediction market platform, incorporating research from successful DeFi protocols including UNI, AAVE, dYdX, and GMX.

---

## Table of Contents

1. [Token Utility Design](#token-utility-design)
2. [Token Distribution](#token-distribution)
3. [Vesting Schedules](#vesting-schedules)
4. [Token Emission Schedule](#token-emission-schedule)
5. [Inflation/Deflation Mechanisms](#inflationdeflation-mechanisms)
6. [Buyback and Burn Strategies](#buyback-and-burn-strategies)
7. [Staking Mechanics and Rewards](#staking-mechanics-and-rewards)
8. [Governance Framework](#governance-framework)
9. [Token Launch Strategies](#token-launch-strategies)
10. [Regulatory Considerations](#regulatory-considerations)

---

## Token Utility Design

### Core Utility Principles

A well-designed token should serve multiple functions within the ecosystem while maintaining sustainable value. Based on research from leading DeFi protocols, our token utility framework incorporates:

### 1. Governance Rights

```
┌─────────────────────────────────────────────────────────────────┐
│                    GOVERNANCE UTILITY                           │
├─────────────────────────────────────────────────────────────────┤
│  Token holders can vote on:                                     │
│  • Protocol parameters (fees, collateral requirements)          │
│  • Market categories and listing criteria                       │
│  • Oracle provider selection                                    │
│  • Treasury allocation decisions                                │
│  • Protocol upgrades and improvements                           │
│  • Partnership proposals                                        │
└─────────────────────────────────────────────────────────────────┘
```

**Implementation Pattern (Similar to UNI/AAVE):**
- 1 token = 1 vote
- Delegation supported for passive holders
- Time-locked voting for major decisions (7-14 day periods)
- Quorum requirements: 4% of total supply for standard proposals

### 2. Fee Discounts

| Tier | Token Balance | Trading Fee Discount | Market Creation Discount |
|------|---------------|---------------------|-------------------------|
| Bronze | 100-999 | 10% | 5% |
| Silver | 1,000-9,999 | 20% | 15% |
| Gold | 10,000-49,999 | 35% | 25% |
| Platinum | 50,000-249,999 | 50% | 40% |
| Diamond | 250,000+ | 70% | 60% |

### 3. Staking Benefits

```solidity
// Example staking contract interface
interface ITokenStaking {
    // Stake tokens to earn rewards and unlock benefits
    function stake(uint256 amount, uint256 lockPeriod) external;

    // Lock periods: 30, 90, 180, 365 days
    // Longer locks = higher multipliers

    // Rewards include:
    // - Share of protocol fees (30% of total fees)
    // - Governance weight multiplier
    // - Priority market resolution rights
}
```

### 4. Protocol Fee Sharing

Following the GMX model where 30% of platform fees are distributed to stakers:

```
Protocol Revenue Flow:
┌──────────────────┐
│  Trading Fees    │
│  (0.1-0.5%)      │
└────────┬─────────┘
         │
         ▼
┌──────────────────────────────────────────────────────┐
│                  FEE DISTRIBUTION                     │
├──────────────────────────────────────────────────────┤
│  30% → Token Stakers (real yield)                    │
│  25% → Treasury (protocol growth)                    │
│  20% → Liquidity Providers                           │
│  15% → Insurance Fund                                │
│  10% → Buyback & Burn                                │
└──────────────────────────────────────────────────────┘
```

### 5. Market Creation & Resolution

- **Market Creation:** Stake tokens to create prediction markets
- **Resolution Participation:** Earn rewards for accurate market resolution
- **Dispute Resolution:** Token-weighted voting on contested outcomes

### Comparison with Leading Protocols

| Protocol | Primary Utility | Fee Sharing | Governance | Staking APY |
|----------|-----------------|-------------|------------|-------------|
| UNI | Governance | No (proposed) | Yes | N/A |
| AAVE | Governance + Safety | Yes (Safety Module) | Yes | 5-8% |
| dYdX | Fee Discounts + Governance | Yes (100% v4) | Yes | Variable |
| GMX | Fee Sharing + Governance | Yes (30%) | Yes | 10-20% |
| **Our Token** | All of Above | Yes (30%) | Yes | 8-15% |

---

## Token Distribution

### Allocation Breakdown

```
                    TOKEN DISTRIBUTION
    ═══════════════════════════════════════════════════

              Community/Ecosystem (40%)
                    ████████████████
                   ╱                ╲
                  ╱                  ╲
    Team (15%)   ╱                    ╲   Treasury (15%)
    ██████      ╱                      ╲      ██████
               ╱                        ╲
              ╱                          ╲
             ╱                            ╲
    Investors ╱                            ╲ Liquidity
    (18%)    ╱                              ╲ Mining (12%)
    ███████ ╱                                ╲ █████

    ═══════════════════════════════════════════════════

    TOTAL SUPPLY: 1,000,000,000 TOKENS
```

### ASCII Pie Chart

```
                         TOKEN ALLOCATION

           Community/Ecosystem
                  40%
                ╭─────╮
              ╱    ·    ╲
            ╱   ·    ·    ╲
    Team  ╱  ·         ·   ╲  Treasury
    15%  │ ·             · │   15%
         │·      PIE      ·│
         │·     CHART     ·│
    Inv. │ ·             · │  Liquidity
    18%   ╲  ·         ·  ╱   Mining 12%
            ╲   ·    ·   ╱
              ╲    ·   ╱
                ╰─────╯

    ┌─────────────────────────────────────────────────┐
    │  Community/Ecosystem  ████████████████  40.0%   │
    │  Investors (Seed+A+B) ███████▌          18.0%   │
    │  Team & Advisors      ██████            15.0%   │
    │  Treasury             ██████            15.0%   │
    │  Liquidity Mining     █████             12.0%   │
    └─────────────────────────────────────────────────┘
```

### Detailed Allocation Table

| Category | Percentage | Tokens | Purpose | Vesting |
|----------|------------|--------|---------|---------|
| **Community/Ecosystem** | 40% | 400,000,000 | Airdrops, grants, rewards | Various |
| - Airdrop (Phase 1) | 10% | 100,000,000 | Early user rewards | 25% at TGE, 75% over 12 months |
| - Ecosystem Grants | 15% | 150,000,000 | Developer incentives | Released per milestone |
| - Future Airdrops | 10% | 100,000,000 | Ongoing user acquisition | TBD by governance |
| - Bug Bounties | 5% | 50,000,000 | Security rewards | As needed |
| **Investors** | 18% | 180,000,000 | Funding rounds | 24-36 month vesting |
| - Seed Round | 5% | 50,000,000 | Initial funding | 12mo cliff, 24mo linear |
| - Series A | 8% | 80,000,000 | Growth funding | 12mo cliff, 30mo linear |
| - Series B | 5% | 50,000,000 | Scale funding | 6mo cliff, 24mo linear |
| **Team & Advisors** | 15% | 150,000,000 | Retention & alignment | 48 month vesting |
| - Core Team | 12% | 120,000,000 | Founders & employees | 12mo cliff, 36mo linear |
| - Advisors | 3% | 30,000,000 | Strategic advisors | 6mo cliff, 24mo linear |
| **Treasury** | 15% | 150,000,000 | Protocol owned | Governance controlled |
| **Liquidity Mining** | 12% | 120,000,000 | DEX incentives | 48 month emission |

### Comparison with Successful Projects

| Project | Community | Team | Investors | Treasury | Other |
|---------|-----------|------|-----------|----------|-------|
| Uniswap (UNI) | 60% | 21.5% | 18.5% | - | - |
| AAVE | 80%* | 13% | - | - | 7% Reserve |
| dYdX | 50% | 15.3% | 27.7% | 7% | - |
| GMX | ~100%** | - | - | - | Fair Launch |
| **Ours** | 40% | 15% | 18% | 15% | 12% LM |

*AAVE migrated from LEND with different initial distribution
**GMX converted from another token with majority already circulating

---

## Vesting Schedules

### Visual Vesting Timeline

```
VESTING SCHEDULE TIMELINE (48 MONTHS)
═══════════════════════════════════════════════════════════════════════

Month:    0    6    12   18   24   30   36   42   48
          │    │    │    │    │    │    │    │    │
          ▼    ▼    ▼    ▼    ▼    ▼    ▼    ▼    ▼

TEAM      ░░░░░░░░░░░░░█████████████████████████████████
          │←── Cliff ──│←────── Linear Vesting ────────│
          (12 months)              (36 months)

SEED      ░░░░░░░░░░░░░███████████████████████████
          │←── Cliff ──│←──── Linear Vesting ────│
          (12 months)         (24 months)

SERIES A  ░░░░░░░░░░░░░████████████████████████████████████
          │←── Cliff ──│←────────── Linear Vesting ──────│
          (12 months)              (30 months)

SERIES B  ░░░░░░█████████████████████████████
          │←Cliff│←──── Linear Vesting ────│
          (6 mo)         (24 months)

ADVISORS  ░░░░░░███████████████████████████
          │←Clf─│←──── Linear Vesting ────│
          (6 mo)         (24 months)

AIRDROP   ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
          │25%│←─────── 75% over 12 mo ───│
          TGE

═══════════════════════════════════════════════════════════════════════
Legend: ░ = Locked/Cliff  █ = Vesting/Released
```

### Unlock Schedule Table

| Month | Team | Seed | Series A | Series B | Advisors | Airdrop | Total Unlock |
|-------|------|------|----------|----------|----------|---------|--------------|
| 0 (TGE) | 0% | 0% | 0% | 0% | 0% | 25% | 2.5% |
| 6 | 0% | 0% | 0% | 0% | 0% | 62.5% | 6.25% |
| 12 | 0% | 0% | 0% | 25% | 25% | 100% | 17.5% |
| 18 | 16.7% | 25% | 14.3% | 50% | 50% | 100% | 28.5% |
| 24 | 33.3% | 50% | 28.6% | 75% | 75% | 100% | 41.2% |
| 30 | 50% | 75% | 42.9% | 100% | 100% | 100% | 55.4% |
| 36 | 66.7% | 100% | 57.1% | 100% | 100% | 100% | 67.8% |
| 42 | 83.3% | 100% | 85.7% | 100% | 100% | 100% | 81.4% |
| 48 | 100% | 100% | 100% | 100% | 100% | 100% | 100% |

### Smart Contract Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract TokenVesting {
    struct VestingSchedule {
        uint256 totalAmount;
        uint256 cliffDuration;    // in seconds
        uint256 vestingDuration;  // in seconds
        uint256 startTime;
        uint256 released;
        bool revocable;
    }

    mapping(address => VestingSchedule) public vestingSchedules;

    // Team vesting: 12 month cliff, 36 month linear
    uint256 public constant TEAM_CLIFF = 365 days;
    uint256 public constant TEAM_VESTING = 3 * 365 days;

    // Investor vesting: varies by round
    uint256 public constant SEED_CLIFF = 365 days;
    uint256 public constant SEED_VESTING = 2 * 365 days;

    function calculateVestedAmount(address beneficiary)
        public view returns (uint256)
    {
        VestingSchedule memory schedule = vestingSchedules[beneficiary];

        if (block.timestamp < schedule.startTime + schedule.cliffDuration) {
            return 0;
        }

        uint256 timeFromStart = block.timestamp - schedule.startTime;

        if (timeFromStart >= schedule.cliffDuration + schedule.vestingDuration) {
            return schedule.totalAmount;
        }

        uint256 vestedTime = timeFromStart - schedule.cliffDuration;
        return (schedule.totalAmount * vestedTime) / schedule.vestingDuration;
    }

    function release() external {
        VestingSchedule storage schedule = vestingSchedules[msg.sender];
        uint256 vested = calculateVestedAmount(msg.sender);
        uint256 releasable = vested - schedule.released;

        require(releasable > 0, "No tokens to release");

        schedule.released = vested;
        // Transfer tokens to beneficiary
    }
}
```

---

## Token Emission Schedule

### Emission Curve Visualization

```
CUMULATIVE TOKEN EMISSION (% of Total Supply)
═══════════════════════════════════════════════════════════════════

100% │                                         ████████████████████
     │                                    █████
 90% │                               █████
     │                          █████
 80% │                      ████
     │                  ████
 70% │               ███
     │            ███
 60% │          ██
     │        ██
 50% │      ██
     │     █
 40% │    █
     │   █
 30% │  █
     │  █
 20% │ █
     │ █
 10% │█
     │█
  0% └────────────────────────────────────────────────────────────────
     0    6   12   18   24   30   36   42   48   54   60   66   72
                           MONTHS FROM TGE
```

### Monthly Emission Schedule

```
MONTHLY TOKEN RELEASES
═══════════════════════════════════════════════════════════════════

Month 0  (TGE):     ████████████████████████████████  10.0% (Initial)
Month 1-6:         ███████████████                    ~2.5% per month
Month 7-12:        ██████████████████                 ~3.0% per month
Month 13-24:       ████████████                       ~2.0% per month
Month 25-36:       ████████                           ~1.5% per month
Month 37-48:       ████                               ~1.0% per month
Month 49-72:       ██                                 ~0.5% per month

═══════════════════════════════════════════════════════════════════
```

### Emission by Category Over Time

| Period | Community | Team | Investors | Treasury | LM | Cumulative |
|--------|-----------|------|-----------|----------|-----|------------|
| TGE | 10% | 0% | 0% | 0% | 0% | 10% |
| Year 1 | 25% | 8.3% | 10% | 5% | 6% | 54.3% |
| Year 2 | 35% | 15% | 18% | 10% | 9% | 87% |
| Year 3 | 40% | 15% | 18% | 12% | 11% | 96% |
| Year 4 | 40% | 15% | 18% | 15% | 12% | 100% |

---

## Inflation/Deflation Mechanisms

### Deflationary Mechanisms

```
DEFLATIONARY PRESSURE SOURCES
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                     BURN MECHANISMS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Transaction Fee Burns (like EIP-1559)                       │
│     └── 10% of all protocol fees burned automatically           │
│                                                                  │
│  2. Quarterly Buyback & Burn                                    │
│     └── 20% of treasury profits used for open market buybacks   │
│                                                                  │
│  3. Market Creation Burns                                       │
│     └── 5% of market creation fees burned                       │
│                                                                  │
│  4. Penalty Burns                                               │
│     └── Slashed stakes from bad actors burned                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Supply Reduction Model

```
PROJECTED SUPPLY OVER TIME (with burns)
═══════════════════════════════════════════════════════════════════

Starting:  1,000,000,000 tokens
           ████████████████████████████████████████████████████

Year 1:      990,000,000 tokens (-1.0%)
           ███████████████████████████████████████████████████

Year 2:      975,000,000 tokens (-2.5%)
           ██████████████████████████████████████████████████

Year 3:      955,000,000 tokens (-4.5%)
           █████████████████████████████████████████████████

Year 5:      910,000,000 tokens (-9.0%)
           ███████████████████████████████████████████████

Year 10:     820,000,000 tokens (-18.0%)
           █████████████████████████████████████████

═══════════════════════════════════════════════════════════════════
Target: ~2% annual deflation through combined mechanisms
```

### Inflationary Controls

```solidity
// Emission cap implementation
contract TokenEmission {
    uint256 public constant MAX_SUPPLY = 1_000_000_000 * 1e18;
    uint256 public constant ANNUAL_EMISSION_CAP = 20_000_000 * 1e18; // 2% max

    uint256 public totalEmitted;
    uint256 public lastEmissionYear;
    uint256 public yearlyEmitted;

    modifier withinEmissionCap(uint256 amount) {
        uint256 currentYear = (block.timestamp - deployTime) / 365 days;

        if (currentYear > lastEmissionYear) {
            yearlyEmitted = 0;
            lastEmissionYear = currentYear;
        }

        require(
            yearlyEmitted + amount <= ANNUAL_EMISSION_CAP,
            "Annual emission cap exceeded"
        );
        require(
            totalEmitted + amount <= MAX_SUPPLY,
            "Max supply exceeded"
        );
        _;
    }
}
```

---

## Buyback and Burn Strategies

### Strategy Overview

Based on successful implementations by Binance (BNB), MakerDAO/Sky, and Ethereum (EIP-1559):

```
BUYBACK & BURN WORKFLOW
═══════════════════════════════════════════════════════════════════

  Protocol Revenue
        │
        ▼
┌───────────────────┐
│  Revenue Split    │
│                   │
│  30% → Stakers    │
│  25% → Treasury   │──────┐
│  20% → LPs        │      │
│  15% → Insurance  │      │
│  10% → B&B Fund   │──┐   │
└───────────────────┘  │   │
                       │   │
        ┌──────────────┘   │
        │                  │
        ▼                  │
┌───────────────────┐      │
│  Buyback Fund     │◄─────┘ (Treasury profits also contribute)
│  Accumulation     │
└────────┬──────────┘
         │
         ▼ (Quarterly)
┌───────────────────┐
│  TWAP Buyback     │  Time-Weighted Average Price
│  Execution        │  Minimizes market impact
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│  Burn Address     │  0x000...dead
│  Permanent        │
└───────────────────┘
```

### Implementation Strategies

#### 1. Automatic Fee Burns (Ethereum EIP-1559 Style)

```solidity
contract AutomaticBurn {
    uint256 public constant BURN_RATE = 1000; // 10% in basis points

    function processTransactionFee(uint256 fee) internal {
        uint256 burnAmount = (fee * BURN_RATE) / 10000;
        uint256 treasuryAmount = fee - burnAmount;

        // Burn tokens
        _burn(burnAmount);

        // Send rest to treasury
        _transfer(address(this), treasury, treasuryAmount);

        emit FeeBurned(burnAmount);
    }
}
```

#### 2. Quarterly TWAP Buybacks (Binance Style)

```solidity
contract QuarterlyBuyback {
    uint256 public constant QUARTER = 90 days;
    uint256 public constant BUYBACK_DURATION = 10 days;
    uint256 public lastBuybackTime;

    // Execute buyback over 10 days using TWAP
    function executeBuyback(uint256 budgetInUSD) external onlyGovernance {
        require(
            block.timestamp >= lastBuybackTime + QUARTER,
            "Too early for buyback"
        );

        // Calculate daily buyback amount
        uint256 dailyBudget = budgetInUSD / 10;

        // Execute through DEX aggregator (1inch, 0x)
        // Using TWAP to minimize slippage and front-running

        lastBuybackTime = block.timestamp;
        emit BuybackInitiated(budgetInUSD, BUYBACK_DURATION);
    }
}
```

### Projected Burn Impact

| Year | Protocol Revenue | Buyback Budget | Tokens Burned | Supply Reduction |
|------|------------------|----------------|---------------|------------------|
| 1 | $50M | $5M | 10M tokens | 1.0% |
| 2 | $150M | $15M | 25M tokens | 2.5% |
| 3 | $300M | $30M | 45M tokens | 4.5% |
| 4 | $500M | $50M | 70M tokens | 7.0% |
| 5 | $750M | $75M | 100M tokens | 10.0% |

---

## Staking Mechanics and Rewards

### Staking Tiers System

```
STAKING TIERS AND BENEFITS
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│  TIER 1: Flexible Staking                                       │
│  Lock Period: None (unstake anytime)                            │
│  Base APY: 5%                                                   │
│  Governance Multiplier: 1x                                      │
│  Fee Share: Standard                                            │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  TIER 2: 30-Day Lock                                            │
│  Lock Period: 30 days minimum                                   │
│  Base APY: 8%                                                   │
│  Governance Multiplier: 1.25x                                   │
│  Fee Share: +10% bonus                                          │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  TIER 3: 90-Day Lock                                            │
│  Lock Period: 90 days minimum                                   │
│  Base APY: 12%                                                  │
│  Governance Multiplier: 1.5x                                    │
│  Fee Share: +25% bonus                                          │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  TIER 4: 365-Day Lock (Maximum Benefits)                        │
│  Lock Period: 365 days minimum                                  │
│  Base APY: 15%                                                  │
│  Governance Multiplier: 2x                                      │
│  Fee Share: +50% bonus                                          │
│  Extra: Early access to new features                            │
└─────────────────────────────────────────────────────────────────┘
```

### Reward Sources

```
STAKING REWARD COMPOSITION
═══════════════════════════════════════════════════════════════════

    ┌──────────────────────────────────────────────────────────┐
    │                    TOTAL STAKING APY                      │
    │                      8-15% Target                         │
    └──────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
    ┌──────────┐       ┌──────────┐       ┌──────────┐
    │ Protocol │       │  Token   │       │ Liquidity│
    │   Fees   │       │ Emissions│       │ Incentive│
    │  (USDC)  │       │  (TOKEN) │       │  (TOKEN) │
    │   ~5%    │       │   ~5%    │       │   ~5%    │
    └──────────┘       └──────────┘       └──────────┘
    Real Yield         Inflationary       LP Rewards
```

### Implementation (AAVE Safety Module Inspired)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract StakingModule {
    struct StakeInfo {
        uint256 amount;
        uint256 lockEnd;
        uint256 tier;
        uint256 rewardDebt;
        uint256 lastClaimTime;
    }

    // Tier multipliers in basis points (10000 = 1x)
    uint256[4] public tierMultipliers = [10000, 12500, 15000, 20000];
    uint256[4] public lockDurations = [0, 30 days, 90 days, 365 days];

    mapping(address => StakeInfo) public stakes;

    uint256 public totalStaked;
    uint256 public rewardPerTokenStored;
    uint256 public lastUpdateTime;

    // Protocol fee pool (in USDC)
    uint256 public feePool;

    function stake(uint256 amount, uint256 tier) external {
        require(tier < 4, "Invalid tier");

        _updateReward(msg.sender);

        StakeInfo storage info = stakes[msg.sender];
        info.amount += amount;
        info.tier = tier;
        info.lockEnd = block.timestamp + lockDurations[tier];

        totalStaked += amount;

        // Transfer tokens to contract
        token.transferFrom(msg.sender, address(this), amount);

        emit Staked(msg.sender, amount, tier);
    }

    function calculateReward(address staker) public view returns (uint256) {
        StakeInfo memory info = stakes[staker];
        if (info.amount == 0) return 0;

        uint256 baseReward = _pendingReward(staker);
        uint256 multipliedReward = (baseReward * tierMultipliers[info.tier]) / 10000;

        return multipliedReward;
    }

    function claimRewards() external {
        _updateReward(msg.sender);

        uint256 reward = calculateReward(msg.sender);
        require(reward > 0, "No rewards");

        stakes[msg.sender].rewardDebt = 0;
        stakes[msg.sender].lastClaimTime = block.timestamp;

        // Transfer rewards (mix of token emissions and USDC fees)
        _distributeRewards(msg.sender, reward);

        emit RewardsClaimed(msg.sender, reward);
    }

    function unstake(uint256 amount) external {
        StakeInfo storage info = stakes[msg.sender];
        require(block.timestamp >= info.lockEnd, "Still locked");
        require(info.amount >= amount, "Insufficient stake");

        _updateReward(msg.sender);

        info.amount -= amount;
        totalStaked -= amount;

        token.transfer(msg.sender, amount);

        emit Unstaked(msg.sender, amount);
    }
}
```

### Safety Module (AAVE-Inspired)

For protocol protection, stakers can opt into the Safety Module:

```
SAFETY MODULE MECHANICS
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                      SAFETY MODULE                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Purpose: Backstop protocol against shortfall events             │
│                                                                  │
│  Staked Amount at Risk: Up to 30% can be slashed                │
│                                                                  │
│  Additional Rewards: +5% APY for accepting slashing risk        │
│                                                                  │
│  Cooldown Period: 10 days to unstake                            │
│                                                                  │
│  Unstake Window: 2 days after cooldown                          │
│                                                                  │
│  Current Coverage: $50M in staked value                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Governance Framework

### Governance Structure

```
GOVERNANCE ARCHITECTURE
═══════════════════════════════════════════════════════════════════

                    ┌─────────────────────┐
                    │   TOKEN HOLDERS     │
                    │   (Voting Power)    │
                    └──────────┬──────────┘
                               │
               ┌───────────────┼───────────────┐
               │               │               │
               ▼               ▼               ▼
        ┌──────────┐    ┌──────────┐    ┌──────────┐
        │ Direct   │    │ Delegate │    │ Council  │
        │  Vote    │    │  System  │    │ Members  │
        └────┬─────┘    └────┬─────┘    └────┬─────┘
             │               │               │
             └───────────────┼───────────────┘
                             │
                             ▼
                    ┌─────────────────────┐
                    │  GOVERNANCE FORUM   │
                    │  (Discussion)       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  SNAPSHOT VOTING    │
                    │  (Off-chain Signal) │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  ON-CHAIN VOTING    │
                    │  (Binding)          │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  TIMELOCK (48hrs)   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  EXECUTION          │
                    └─────────────────────┘
```

### Proposal Types and Requirements

| Proposal Type | Quorum | Approval | Timelock | Example |
|---------------|--------|----------|----------|---------|
| **Standard** | 4% | >50% | 48 hours | Fee changes |
| **Emergency** | 1% | >66% | 6 hours | Security fix |
| **Protocol Upgrade** | 10% | >66% | 7 days | Smart contract upgrade |
| **Treasury** | 4% | >50% | 48 hours | Grant allocation |
| **Constitutional** | 20% | >75% | 14 days | Governance rules |

### Voting Power Calculation

```solidity
contract GovernanceVoting {
    // Voting power = tokens + staked tokens * tier multiplier
    function getVotingPower(address account) public view returns (uint256) {
        uint256 walletBalance = token.balanceOf(account);

        StakeInfo memory stake = stakingModule.stakes(account);
        uint256 stakedPower = stake.amount * tierMultipliers[stake.tier] / 10000;

        // Add delegated votes
        uint256 delegatedPower = delegatedVotes[account];

        return walletBalance + stakedPower + delegatedPower;
    }
}
```

### Delegation System

```
DELEGATION FLOW
═══════════════════════════════════════════════════════════════════

  Token Holder A                    Delegate B
  (1000 tokens)                     (Expert Voter)
       │                                 │
       │   delegate(B)                   │
       ├────────────────────────────────▶│
       │                                 │
       │   Voting power transferred      │
       │                                 │
       │◀ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
       │   Rewards still go to A         │
       │                                 │
       │   undelegate()                  │
       ├────────────────────────────────▶│
       │                                 │
       │   Voting power returned         │
       ▼                                 ▼
```

---

## Token Launch Strategies

### Launch Options Comparison

```
TOKEN LAUNCH STRATEGIES COMPARISON
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│  FAIR LAUNCH                                                    │
├─────────────────────────────────────────────────────────────────┤
│  Pros:                          Cons:                           │
│  • Community trust              • No funding raised             │
│  • Equal opportunity            • Less controlled distribution  │
│  • Regulatory friendly          • Requires existing treasury    │
│                                                                  │
│  Examples: Yearn (YFI), Radiant Capital                         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  INITIAL DEX OFFERING (IDO)                                     │
├─────────────────────────────────────────────────────────────────┤
│  Pros:                          Cons:                           │
│  • Immediate liquidity          • Bot front-running             │
│  • Public price discovery       • Whale manipulation            │
│  • Marketing exposure           • Regulatory scrutiny           │
│                                                                  │
│  Platforms: DAO Maker, Polkastarter, CopperLaunch              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  LIQUIDITY BOOTSTRAPPING POOL (LBP)                             │
├─────────────────────────────────────────────────────────────────┤
│  Pros:                          Cons:                           │
│  • Fair price discovery         • Complexity for users          │
│  • Anti-whale design            • Requires understanding        │
│  • Gradual distribution         • Price can drop significantly  │
│                                                                  │
│  Platforms: Balancer, Fjord Foundry                             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  AIRDROP                                                        │
├─────────────────────────────────────────────────────────────────┤
│  Pros:                          Cons:                           │
│  • User acquisition             • Sell pressure at launch       │
│  • Rewards loyalty              • Sybil attacks                 │
│  • Decentralization             • Regulatory questions          │
│                                                                  │
│  Examples: UNI, dYdX, ENS, Arbitrum, Optimism                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  HYBRID APPROACH (RECOMMENDED)                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: Private rounds (completed pre-launch)                 │
│           └── Seed, Series A, Series B                          │
│                                                                  │
│  Phase 2: Community airdrop to active users                     │
│           └── Based on trading volume, market creation          │
│                                                                  │
│  Phase 3: LBP for public price discovery                        │
│           └── 48-72 hour event with anti-whale measures         │
│                                                                  │
│  Phase 4: DEX liquidity mining launch                           │
│           └── Incentivized pools on Uniswap, Balancer           │
│                                                                  │
│  Examples: Jupiter (points → airdrop → incentives)              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Airdrop Design Best Practices

```
AIRDROP ELIGIBILITY CRITERIA
═══════════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────────────┐
│  ACTIVITY-BASED ALLOCATION                                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Criteria                              Weight    Max Tokens       │
│  ─────────────────────────────────────────────────────────────── │
│  Trading Volume (cumulative)           25%       25,000           │
│  Markets Created                       20%       20,000           │
│  Days Active                          15%       15,000           │
│  Winning Positions                    15%       15,000           │
│  Liquidity Provided                   15%       15,000           │
│  Early User Bonus (first 6 months)    10%       10,000           │
│                                                                   │
│  TOTAL MAX PER USER:                           100,000 tokens     │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Anti-Sybil Measures

```solidity
contract AirdropClaim {
    // Sybil resistance mechanisms
    mapping(address => bool) public hasClaimed;
    mapping(bytes32 => bool) public usedProofs;

    // Minimum activity requirements
    uint256 public constant MIN_TRADES = 5;
    uint256 public constant MIN_VOLUME = 100 * 1e6; // $100 USDC
    uint256 public constant MIN_DAYS_ACTIVE = 7;

    function claim(
        uint256 amount,
        bytes32[] calldata merkleProof,
        bytes calldata worldIdProof // Optional: World ID verification
    ) external {
        require(!hasClaimed[msg.sender], "Already claimed");
        require(verifyMerkleProof(msg.sender, amount, merkleProof), "Invalid proof");

        // Optional: Verify World ID for enhanced sybil resistance
        if (worldIdEnabled) {
            require(verifyWorldId(msg.sender, worldIdProof), "World ID failed");
        }

        // Activity verification
        require(getUserTrades(msg.sender) >= MIN_TRADES, "Insufficient trades");
        require(getUserVolume(msg.sender) >= MIN_VOLUME, "Insufficient volume");

        hasClaimed[msg.sender] = true;

        // Transfer with vesting (25% immediate, 75% over 12 months)
        uint256 immediate = amount * 25 / 100;
        uint256 vested = amount - immediate;

        token.transfer(msg.sender, immediate);
        vestingContract.createVesting(msg.sender, vested, 365 days);

        emit AirdropClaimed(msg.sender, amount);
    }
}
```

---

## Regulatory Considerations

### Token Classification Framework

```
SEC TOKEN CLASSIFICATION (2025 Guidance)
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                    HOWEY TEST ANALYSIS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Investment of Money?                                         │
│     └── Token sale = YES                                         │
│     └── Airdrop for activity = Arguable NO                       │
│                                                                  │
│  2. Common Enterprise?                                           │
│     └── Protocol success linked = YES                            │
│     └── Decentralized protocol = Arguable NO                     │
│                                                                  │
│  3. Expectation of Profits?                                      │
│     └── Marketing as investment = YES (bad)                      │
│     └── Utility-focused messaging = Better position              │
│                                                                  │
│  4. Efforts of Others?                                           │
│     └── Centralized team control = YES                           │
│     └── Decentralized DAO = Potentially NO                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

RISK SPECTRUM:
─────────────────────────────────────────────────────────────────
HIGHER RISK ◄───────────────────────────────────────► LOWER RISK
  ICO with          IDO with          Fair Launch      Airdrop to
  profit            marketing         utility          existing
  promises          focus             focus            users
```

### Compliance Checklist

```
REGULATORY COMPLIANCE CHECKLIST
═══════════════════════════════════════════════════════════════════

□ Token Utility Documentation
  ├── Clear utility use cases documented
  ├── Not marketed as investment opportunity
  └── Functional utility at launch (not promised)

□ Decentralization Plan
  ├── Progressive decentralization roadmap
  ├── DAO governance structure
  └── Reduced team control over time

□ Geographic Restrictions
  ├── US persons excluded from token sale
  ├── OFAC sanctioned countries blocked
  └── Geo-blocking on frontend

□ KYC/AML Procedures
  ├── Investor KYC for private rounds
  ├── Airdrop verification (optional)
  └── Suspicious activity monitoring

□ Legal Structure
  ├── Foundation in crypto-friendly jurisdiction
  ├── Legal opinion on token classification
  └── Terms of service and disclaimers

□ Token Sale Documentation
  ├── Not characterized as securities offering
  ├── Risk disclosures provided
  └── No guarantees of returns

□ Secondary Market Considerations
  ├── No market manipulation
  ├── Transparent vesting schedules
  └── Lockup compliance
```

### Jurisdiction Considerations

| Jurisdiction | Regulatory Stance | Key Requirements | Risk Level |
|--------------|-------------------|------------------|------------|
| **USA** | Evolving (2025 guidance) | Howey test, CFTC/SEC split | Medium-High |
| **EU** | MiCA (Dec 2024) | Whitepaper, authorization | Medium |
| **UK** | FCA oversight | Registration, promotions rules | Medium |
| **Singapore** | MAS guidelines | Payment token vs. security token | Low-Medium |
| **Switzerland** | FINMA framework | Utility token classification | Low |
| **UAE** | VARA (Dubai) | Licensing framework | Low |
| **Cayman Islands** | Light touch | Foundation structure | Low |

### Risk Mitigation Strategies

```
REGULATORY RISK MITIGATION
═══════════════════════════════════════════════════════════════════

1. UTILITY-FIRST DESIGN
   ┌────────────────────────────────────────────────────────────┐
   │  • Token has real utility at launch                        │
   │  • Governance, fee discounts, staking are primary uses    │
   │  • Not primarily a speculative asset                       │
   └────────────────────────────────────────────────────────────┘

2. PROGRESSIVE DECENTRALIZATION
   ┌────────────────────────────────────────────────────────────┐
   │  Year 1: Team-led with community input                     │
   │  Year 2: Governance controls major decisions               │
   │  Year 3+: Full DAO control, minimal team involvement       │
   └────────────────────────────────────────────────────────────┘

3. TRANSPARENT COMMUNICATION
   ┌────────────────────────────────────────────────────────────┐
   │  • Never promise returns or price appreciation             │
   │  • Focus on protocol development, not token price          │
   │  • Regular governance and transparency reports             │
   └────────────────────────────────────────────────────────────┘

4. LEGAL STRUCTURE
   ┌────────────────────────────────────────────────────────────┐
   │  • Foundation for token issuance (offshore)                │
   │  • Operating company for development (compliant)           │
   │  • Clear separation of responsibilities                    │
   └────────────────────────────────────────────────────────────┘
```

---

## Appendix: Implementation Timeline

```
TOKEN LAUNCH TIMELINE
═══════════════════════════════════════════════════════════════════

Month -6:  ┌─────────────────────────────────────────────────────┐
           │  • Legal structure setup                            │
           │  • Tokenomics finalization                          │
           │  • Smart contract development                       │
           └─────────────────────────────────────────────────────┘

Month -3:  ┌─────────────────────────────────────────────────────┐
           │  • Smart contract audits (2+ auditors)              │
           │  • Community building                               │
           │  • Governance forum launch                          │
           └─────────────────────────────────────────────────────┘

Month -1:  ┌─────────────────────────────────────────────────────┐
           │  • Airdrop snapshot announcement                    │
           │  • Private round close                              │
           │  • Marketing campaign                               │
           └─────────────────────────────────────────────────────┘

Month 0:   ┌─────────────────────────────────────────────────────┐
           │  • TGE (Token Generation Event)                     │
           │  • Airdrop claims open                              │
           │  • DEX liquidity deployed                           │
           │  • Staking contracts live                           │
           └─────────────────────────────────────────────────────┘

Month 1-3: ┌─────────────────────────────────────────────────────┐
           │  • CEX listings                                     │
           │  • Governance activation                            │
           │  • First proposals                                  │
           └─────────────────────────────────────────────────────┘

Month 3+:  ┌─────────────────────────────────────────────────────┐
           │  • Ongoing development                              │
           │  • Regular governance cycles                        │
           │  • Quarterly burns                                  │
           └─────────────────────────────────────────────────────┘
```

---

## References

- [Uniswap Tokenomics](https://tokenomist.ai/uniswap)
- [AAVE Safety Module](https://docs.aave.com/developers/safety-module)
- [dYdX Tokenomics](https://crypternon.com/en/dydx-tokenomics/)
- [GMX Fee Distribution](https://docs.gmx.io/docs/tokenomics/overview)
- [SEC 2025 Crypto Guidance](https://cointelegraph.com/explained/secs-2025-guidance-what-tokens-are-and-arent-securities)
- [Token Utility Best Practices](https://blog.innmind.com/how-to-design-token-utility-research/)
- [Buyback & Burn Mechanisms](https://www.okx.com/en-us/learn/buyback-burning-supply-tokenomics)
