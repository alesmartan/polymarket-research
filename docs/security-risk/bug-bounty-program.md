# Bug Bounty Program

## Overview

This document outlines the comprehensive bug bounty program structure for a Polymarket-like prediction market platform. It is designed based on industry best practices from leading programs on Immunefi, including Uniswap ($15.5M), Aave ($1M), and Compound Finance ($1M).

### Program Goals

1. **Proactive Security**: Identify vulnerabilities before malicious actors
2. **Community Engagement**: Leverage the global security researcher community
3. **Continuous Protection**: Complement audits with ongoing security testing
4. **Trust Building**: Demonstrate commitment to security through transparency

### Key Statistics

- **Immunefi Platform**: Protects over $190 billion in user funds across 330+ projects
- **Record Bounty**: Uniswap v4 - $15.5 million maximum reward
- **Average Critical Payout**: 10% of funds at risk (capped)
- **Largest Single Payout**: $10 million (Wormhole)

---

## 1. Program Structure

### 1.1 Program Type

We recommend a **managed bug bounty program** hosted on Immunefi with the following structure:

```
┌─────────────────────────────────────────────────────────────────┐
│                    BUG BOUNTY PROGRAM STRUCTURE                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    IMMUNEFI PLATFORM                     │   │
│  │  - Managed submission portal                             │   │
│  │  - Automated triage support                              │   │
│  │  - KYC/payment processing                                │   │
│  │  - Legal safe harbor framework                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    INTERNAL TEAM                         │   │
│  │  - Security Lead: Primary triage owner                   │   │
│  │  - Smart Contract Engineers: Technical validation        │   │
│  │  - Product Team: Business impact assessment              │   │
│  │  - Legal: Safe harbor and payment approval               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    GOVERNANCE                            │   │
│  │  - DAO/Multisig: Final payout approval for >$50K         │   │
│  │  - Emergency Committee: Critical issue response          │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Program Tiers

| Tier | TVL Range | Max Critical Reward | Recommended |
|------|-----------|---------------------|-------------|
| Starter | <$10M | $100,000 | New protocols |
| Standard | $10M-$100M | $500,000 | Growing protocols |
| Enterprise | $100M-$1B | $1,000,000 | Established protocols |
| Premium | >$1B | $2,000,000+ | Major protocols |

### 1.3 Budget Allocation

**Recommended Annual Budget**: 0.5-1% of TVL or $500K minimum

| Category | Allocation | Purpose |
|----------|------------|---------|
| Critical Rewards | 60% | Major vulnerability payouts |
| High/Medium Rewards | 25% | Standard vulnerability payouts |
| Platform Fees | 10% | Immunefi service fees |
| Operational | 5% | Triage, communication, legal |

---

## 2. Scope Definition

### 2.1 In-Scope Assets

#### Smart Contracts (Primary Scope)

| Contract | Network | Address | Max Severity |
|----------|---------|---------|--------------|
| Exchange | Polygon | `0x...` | Critical |
| MarketFactory | Polygon | `0x...` | Critical |
| ConditionalTokens (CTF) | Polygon | `0x...` | Critical |
| CollateralManager | Polygon | `0x...` | Critical |
| OracleAdapter | Polygon | `0x...` | Critical |
| Governance | Polygon | `0x...` | Critical |
| TimelockController | Polygon | `0x...` | High |

**Repository**: https://github.com/[org]/prediction-market-contracts

**Specific Branches/Commits**:
- Main branch (production)
- Release branches (staged for deployment)

#### Web Application (Secondary Scope)

| Asset | URL | Max Severity |
|-------|-----|--------------|
| Trading Interface | https://app.predictionmarket.com | High |
| API Endpoints | https://api.predictionmarket.com | High |
| Authentication System | https://auth.predictionmarket.com | High |

**Note**: Web vulnerabilities max out at High severity. Critical is reserved for smart contracts.

#### Infrastructure (Tertiary Scope)

| Asset | Description | Max Severity |
|-------|-------------|--------------|
| Order Matching Engine | Off-chain matching service | Medium |
| Indexing Service | Blockchain data indexer | Low |
| Price Feed Service | Oracle data aggregation | Medium |

### 2.2 Out-of-Scope Items

The following are explicitly **NOT** in scope:

#### Contracts/Code
- [ ] Third-party contracts not deployed by the project
- [ ] Test contracts and mock implementations
- [ ] Contracts on testnets (Goerli, Mumbai, etc.)
- [ ] Forked code that hasn't been modified
- [ ] Third-party dependencies (OpenZeppelin, Chainlink, etc.)
- [ ] Legacy/deprecated contracts

#### Vulnerabilities
- [ ] Issues already disclosed in previous audits
- [ ] Issues currently being addressed (disclosed in 30 days)
- [ ] Theoretical attacks without proof of concept
- [ ] Attacks requiring >51% of network hashrate
- [ ] Social engineering attacks
- [ ] Physical attacks on infrastructure
- [ ] DoS attacks against infrastructure (unless <$100 attack cost)

#### Web/API
- [ ] Clickjacking without demonstrated impact
- [ ] Self-XSS
- [ ] CSRF on unauthenticated endpoints
- [ ] Missing security headers without exploit
- [ ] SSL/TLS best practice issues
- [ ] Rate limiting issues (unless severe)
- [ ] Content injection without security impact
- [ ] Open redirects without additional impact

#### Other
- [ ] Best practice deviations without security impact
- [ ] Governance attacks requiring unrealistic token amounts
- [ ] Issues with third-party oracles (report to Chainlink)
- [ ] Gas optimization suggestions
- [ ] Typos or grammar issues

### 2.3 Scope Clarifications

```markdown
## Scope Clarification Examples

### IN SCOPE:
- A bug that allows draining funds from the Exchange contract
- A vulnerability that enables unauthorized market resolution
- An access control flaw allowing privilege escalation
- A signature replay attack enabling double-spending
- A front-end vulnerability enabling transaction manipulation

### OUT OF SCOPE:
- A theoretical reentrancy with no practical exploit
- Polygon network congestion causing delayed transactions
- Chainlink oracle providing stale prices (report to Chainlink)
- User losing funds due to approving a malicious contract
- Market manipulation through legitimate large trades
```

---

## 3. Severity Classification

### 3.1 Severity Levels

We follow the **Immunefi Vulnerability Severity Classification System V2.3**.

#### Critical Severity

**Definition**: Vulnerabilities that can lead to direct loss of funds or permanent freezing of funds, or that can be exploited to compromise the entire protocol.

**Examples**:
- Direct theft of user funds
- Permanent freezing of funds (>1 week)
- Unauthorized minting of tokens
- Protocol insolvency
- Governance takeover
- Oracle manipulation leading to significant loss

**Criteria**:
- Exploitable with reasonable cost/complexity
- Affects significant portion of users/funds
- No significant external dependencies for exploitation

#### High Severity

**Definition**: Vulnerabilities that can lead to significant impact but with limitations (time-limited, requires specific conditions, partial fund loss).

**Examples**:
- Temporary freezing of funds (1-7 days)
- Theft of unclaimed/pending rewards
- Theft of protocol fees
- Unauthorized access to privileged functions
- Manipulation of non-critical contract state
- Data exposure of sensitive user information

**Criteria**:
- Requires specific conditions or timing
- Limited scope of affected users/funds
- May require elevated privileges

#### Medium Severity

**Definition**: Vulnerabilities with limited impact that don't directly result in fund loss but could be part of a larger attack chain.

**Examples**:
- Griefing attacks (no financial gain for attacker)
- Temporary denial of service (<1 hour)
- Contract state manipulation without fund impact
- Information disclosure of contract internals
- Circumvention of non-critical controls

**Criteria**:
- No direct financial impact
- Limited to specific edge cases
- May affect user experience

#### Low Severity

**Definition**: Minor issues that don't pose immediate risk but should be addressed.

**Examples**:
- Events not being emitted correctly
- Inconsistent error messages
- Minor gas inefficiencies (>10% waste)
- Outdated compiler version
- Missing input validation without exploit
- Centralization risks (documented)

**Criteria**:
- No security impact
- Best practice violations
- Code quality issues

### 3.2 Impact Assessment Matrix

| | Funds at Risk | Likelihood High | Likelihood Medium | Likelihood Low |
|--|:-------------:|:---------------:|:-----------------:|:--------------:|
| **Direct Loss** | >$100K | Critical | Critical | High |
| **Direct Loss** | $10K-$100K | Critical | High | High |
| **Direct Loss** | <$10K | High | High | Medium |
| **Temporary Freeze** | Any | High | Medium | Medium |
| **No Fund Impact** | N/A | Medium | Low | Low |

---

## 4. Reward Structure

### 4.1 Smart Contract Rewards

| Severity | Minimum Reward | Maximum Reward | Calculation |
|----------|---------------|----------------|-------------|
| Critical | $50,000 | $500,000 | 10% of funds at risk (capped) |
| High | $10,000 | $50,000 | Fixed + impact assessment |
| Medium | $2,500 | $10,000 | Fixed tier |
| Low | $500 | $2,500 | Fixed tier |

#### Critical Reward Calculation

For Critical severity vulnerabilities affecting smart contracts:

```
Reward = min(
    MAX_CRITICAL_REWARD,
    max(
        MIN_CRITICAL_REWARD,
        Funds_at_Risk × 0.10
    )
)

Where:
- MAX_CRITICAL_REWARD = $500,000
- MIN_CRITICAL_REWARD = $50,000
- Funds_at_Risk = Value of funds directly affected
```

**Example Calculations**:
| Funds at Risk | Calculation | Reward |
|---------------|-------------|--------|
| $50,000 | max($50K, $5K) | $50,000 (minimum) |
| $500,000 | min($500K, $50K) | $50,000 |
| $2,000,000 | min($500K, $200K) | $200,000 |
| $10,000,000 | min($500K, $1M) | $500,000 (maximum) |

### 4.2 Web/API Rewards

| Severity | Reward Range |
|----------|--------------|
| Critical | N/A (not applicable to web) |
| High | $5,000 - $20,000 |
| Medium | $1,000 - $5,000 |
| Low | $250 - $1,000 |

### 4.3 Payment Terms

#### Currency
- **Primary**: USDC on Ethereum/Polygon
- **Alternative**: Protocol token (if applicable) at 7-day TWAP
- **Denomination**: All rewards denominated in USD

#### Payment Timeline
| Severity | Validation Time | Payment Time | Total |
|----------|-----------------|--------------|-------|
| Critical | 48 hours | 7 days | 9 days |
| High | 5 days | 14 days | 19 days |
| Medium | 7 days | 21 days | 28 days |
| Low | 14 days | 30 days | 44 days |

#### KYC Requirements
- **Critical/High**: Government-issued ID required
- **Medium/Low**: Basic identity verification
- **Threshold**: KYC required for rewards >$5,000

### 4.4 Bonus Rewards

| Bonus Type | Multiplier | Criteria |
|------------|------------|----------|
| First Report | +10% | First valid report for a vulnerability class |
| Quality Report | +10% | Exceptional documentation and PoC |
| Fix Suggestion | +5% | Includes well-tested fix |
| Responsible Disclosure | +10% | Followed all program rules |

### 4.5 Reward Comparison (Industry Benchmarks)

| Protocol | Max Critical | Max High | Platform |
|----------|-------------|----------|----------|
| **Uniswap v4** | $15,500,000 | $1,000,000 | Cantina |
| **Immunefi** | $10,000,000 | $200,000 | Immunefi |
| **Aave** | $1,000,000 | $250,000 | Immunefi |
| **Compound** | $1,000,000 | $50,000 | Immunefi |
| **Lido** | $2,000,000 | $100,000 | Immunefi |
| **Our Program** | $500,000 | $50,000 | Immunefi |

---

## 5. Responsible Disclosure Policy

### 5.1 Disclosure Requirements

#### Researcher Obligations

1. **Confidentiality**: Keep all vulnerability details confidential until authorized
2. **Good Faith**: Act in good faith and avoid privacy violations, data destruction, or service disruption
3. **No Exploitation**: Do not exploit the vulnerability beyond proof of concept
4. **Timely Reporting**: Report within 24 hours of discovery
5. **Cooperation**: Respond to requests for additional information within 72 hours

#### Project Obligations

1. **Acknowledgment**: Confirm receipt within 24 hours
2. **Communication**: Provide status updates every 7 days
3. **Fair Assessment**: Evaluate severity fairly using published criteria
4. **Timely Payment**: Process rewards within stated timelines
5. **Credit**: Offer public credit (if desired by researcher)

### 5.2 Disclosure Timeline

```
┌─────────────────────────────────────────────────────────────────┐
│                 RESPONSIBLE DISCLOSURE TIMELINE                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Day 0:    Vulnerability discovered                             │
│            ├── Researcher submits report within 24h             │
│            └── Project acknowledges within 24h                  │
│                                                                 │
│  Day 1-3:  Initial Triage                                       │
│            ├── Severity assessment                              │
│            └── Validity confirmation                            │
│                                                                 │
│  Day 3-7:  Technical Validation                                 │
│            ├── Reproduce vulnerability                          │
│            ├── Assess impact                                    │
│            └── Develop mitigation plan                          │
│                                                                 │
│  Day 7-14: Remediation                                          │
│            ├── Develop and test fix                             │
│            ├── Internal security review                         │
│            └── Prepare for deployment                           │
│                                                                 │
│  Day 14-21: Deployment                                          │
│            ├── Deploy fix (coordinated if needed)               │
│            ├── Monitor for exploitation                         │
│            └── Verify fix effectiveness                         │
│                                                                 │
│  Day 21-30: Disclosure                                          │
│            ├── Process reward payment                           │
│            ├── Public disclosure (coordinated)                  │
│            └── Credit researcher (if desired)                   │
│                                                                 │
│  NOTE: Critical vulnerabilities may have accelerated timelines  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 Coordinated Disclosure

For vulnerabilities affecting multiple protocols or external dependencies:

1. **Notify affected parties** before public disclosure
2. **Coordinate patch deployment** across protocols
3. **Agree on disclosure timeline** (typically 90 days maximum)
4. **Share credit** appropriately among all parties

---

## 6. Submission Requirements

### 6.1 Required Information

Every submission must include:

```markdown
## Bug Bounty Report Template

### Summary
[One paragraph summary of the vulnerability]

### Vulnerability Details

**Type**: [Reentrancy / Access Control / Oracle Manipulation / etc.]
**Severity**: [Critical / High / Medium / Low]
**Affected Asset**: [Contract name and address]
**Affected Function(s)**: [Function names]

### Description
[Detailed description of the vulnerability including:]
- Root cause
- Attack vector
- Prerequisites for exploitation

### Impact
[Describe what an attacker could achieve:]
- Financial impact (estimated funds at risk)
- Users affected
- Protocol impact

### Proof of Concept

[Detailed steps to reproduce OR working exploit code]

**Environment**:
- Network: [Mainnet / Fork / Testnet]
- Block number: [If relevant]
- Tools used: [Foundry / Hardhat / etc.]

**Steps**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Code**:
```solidity
// Exploit code or test case
contract Exploit {
    function attack() external {
        // ...
    }
}
```

### Recommended Fix
[If you have a suggestion for fixing the vulnerability]

### Supporting Materials
- [ ] Screenshots
- [ ] Transaction hashes
- [ ] Contract addresses used in PoC
- [ ] Video demonstration (optional)

### Contact Information
- **Immunefi Username**: [username]
- **Email**: [optional]
- **Wallet Address**: [for payment]
```

### 6.2 Proof of Concept Requirements

| Severity | PoC Requirement |
|----------|-----------------|
| Critical | Working exploit code required |
| High | Detailed steps OR code |
| Medium | Clear reproduction steps |
| Low | Description sufficient |

#### Good PoC Example (Critical)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "forge-std/Test.sol";

interface IExchange {
    function withdraw(uint256 amount) external;
    function balanceOf(address) external view returns (uint256);
}

contract ReentrancyExploit is Test {
    IExchange exchange = IExchange(0x...);
    uint256 public attackCount;

    function setUp() public {
        // Fork mainnet at specific block
        vm.createSelectFork("polygon", 50000000);
    }

    function testExploit() public {
        // Initial state
        uint256 initialBalance = address(exchange).balance;
        console.log("Exchange balance before:", initialBalance);

        // Execute attack
        attack();

        // Verify exploit success
        uint256 finalBalance = address(exchange).balance;
        console.log("Exchange balance after:", finalBalance);
        assertLt(finalBalance, initialBalance, "Exploit failed");
    }

    function attack() public {
        // Deposit initial amount
        exchange.deposit{value: 1 ether}();

        // Trigger reentrancy
        exchange.withdraw(1 ether);
    }

    receive() external payable {
        if (attackCount < 10 && address(exchange).balance >= 1 ether) {
            attackCount++;
            exchange.withdraw(1 ether);
        }
    }
}
```

### 6.3 Submission Don'ts

- [ ] **Don't** submit without a proof of concept for Critical/High
- [ ] **Don't** submit duplicate reports for the same vulnerability
- [ ] **Don't** submit theoretical attacks without practical exploitation
- [ ] **Don't** disclose publicly before coordinated disclosure
- [ ] **Don't** exploit beyond what's necessary for the PoC
- [ ] **Don't** submit automated scanner output without analysis
- [ ] **Don't** submit issues already reported in audits

---

## 7. Triage Process

### 7.1 Triage Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                      TRIAGE WORKFLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STAGE 1: INITIAL REVIEW (24 hours)                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Immunefi Triage                                         │   │
│  │  - Check report completeness                             │   │
│  │  - Verify in-scope                                       │   │
│  │  - Initial severity assessment                           │   │
│  │  - Forward to project team                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  STAGE 2: VALIDATION (48-72 hours)                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Security Team                                           │   │
│  │  - Reproduce the issue                                   │   │
│  │  - Verify impact assessment                              │   │
│  │  - Determine root cause                                  │   │
│  │  - Classify severity                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│         ┌────────────────────┼────────────────────┐            │
│         ▼                    ▼                    ▼            │
│    ┌─────────┐         ┌─────────┐         ┌─────────┐        │
│    │ VALID   │         │ INVALID │         │ NEEDS   │        │
│    │         │         │         │         │  INFO   │        │
│    └────┬────┘         └────┬────┘         └────┬────┘        │
│         │                   │                   │              │
│         ▼                   ▼                   ▼              │
│    Proceed to          Close with          Request from       │
│    remediation         explanation         researcher         │
│                                                                 │
│  STAGE 3: RESOLUTION                                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  - Develop fix                                           │   │
│  │  - Internal review                                       │   │
│  │  - Deploy fix                                            │   │
│  │  - Verify resolution                                     │   │
│  │  - Process payment                                       │   │
│  │  - Close report                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Triage Response Templates

#### Acknowledgment (Within 24 hours)

```markdown
Hi [Researcher],

Thank you for your submission to the [Protocol] Bug Bounty Program.

We have received your report regarding [brief description] and our security team is currently reviewing it.

**Report ID**: [ID]
**Submitted**: [Date/Time]
**Initial Status**: Under Review

We will provide an update within [X] business days. If we need additional information, we will reach out.

Thank you for helping secure our protocol.

Best regards,
[Protocol] Security Team
```

#### Request for Information

```markdown
Hi [Researcher],

Thank you for your submission. We've reviewed your report and need some additional information to proceed:

1. [Specific question 1]
2. [Specific question 2]
3. Can you provide [specific artifact]?

Please respond within 7 days to keep your submission active.

Best regards,
[Protocol] Security Team
```

#### Validity Confirmation

```markdown
Hi [Researcher],

We have validated your report regarding [vulnerability description].

**Status**: Confirmed
**Severity**: [Critical/High/Medium/Low]
**Estimated Reward**: $[Amount]

We are currently working on a fix and will update you on our progress. The estimated timeline for remediation is [X] days.

Please maintain confidentiality until we coordinate public disclosure.

Best regards,
[Protocol] Security Team
```

#### Closure (Invalid)

```markdown
Hi [Researcher],

Thank you for your submission regarding [vulnerability description].

After thorough review, we have determined that this issue is [out of scope / not a vulnerability / already known].

**Reason**: [Detailed explanation]

We encourage you to continue participating in our bug bounty program. If you believe this decision is incorrect, please provide additional context or evidence.

Best regards,
[Protocol] Security Team
```

### 7.3 SLA Commitments

| Action | Critical | High | Medium | Low |
|--------|----------|------|--------|-----|
| Acknowledgment | 4 hours | 24 hours | 24 hours | 48 hours |
| Initial Assessment | 24 hours | 48 hours | 5 days | 7 days |
| Validation | 48 hours | 5 days | 7 days | 14 days |
| Remediation Start | Immediate | 48 hours | 7 days | 14 days |
| Payment Processing | 7 days | 14 days | 21 days | 30 days |

---

## 8. Communication Guidelines

### 8.1 Communication Channels

| Channel | Purpose | Response Time |
|---------|---------|---------------|
| Immunefi Portal | Primary submissions and updates | Per SLA |
| security@protocol.com | Alternative contact | 48 hours |
| Discord #security | General security questions | Best effort |
| Emergency Hotline | Active exploits only | Immediate |

### 8.2 Communication Best Practices

**For the Project:**
- Be professional and respectful
- Provide clear, detailed explanations
- Keep researchers informed of progress
- Be transparent about decisions
- Thank researchers for their contributions

**For Researchers:**
- Be patient during triage
- Provide requested information promptly
- Avoid hostile or threatening communication
- Keep all communication confidential
- Use official channels only

### 8.3 Dispute Resolution

If a researcher disagrees with a severity assessment or validity determination:

1. **Appeal Request**: Researcher submits written appeal with additional evidence
2. **Internal Review**: Independent team member reviews the case
3. **Immunefi Mediation**: If unresolved, Immunefi mediates (optional)
4. **Final Decision**: Project makes final determination with documented reasoning

---

## 9. Legal Safe Harbor

### 9.1 Safe Harbor Agreement

The following activities are authorized and protected under our Safe Harbor policy:

#### Authorized Activities
- Security testing of in-scope assets
- Discovering and reporting vulnerabilities
- Creating proof-of-concept exploits
- Accessing data necessary to demonstrate impact

#### Protections Provided
- We will not pursue legal action for good-faith security research
- We will not report activities to law enforcement
- We will advocate on behalf of researchers if third parties pursue action

### 9.2 Conditions for Safe Harbor

To be protected under Safe Harbor, researchers must:

1. **Act in Good Faith**: Conduct research to improve security
2. **Avoid Harm**: Do not cause damage beyond proof of concept
3. **Protect Privacy**: Do not access or exfiltrate user data
4. **Report Promptly**: Submit findings within 24 hours
5. **Maintain Confidentiality**: Do not disclose until authorized
6. **Follow Rules**: Comply with all program rules

### 9.3 Limitations

Safe Harbor does NOT cover:

- Activities that violate applicable law beyond security testing
- Physical attacks or social engineering of employees
- Denial of service attacks against production systems
- Accessing user data beyond what's needed for PoC
- Disclosure of vulnerability details before coordination
- Activities after being explicitly asked to stop

### 9.4 Immunefi Safe Harbor (For Active Exploits)

For whitehat rescue operations during active exploits:

```markdown
## Immunefi Safe Harbor - Active Exploit Rescue

If you discover an ACTIVE exploitation in progress, you may:

1. Intervene to rescue funds
2. Return funds to the protocol vault within 6 hours
3. Submit a Safe Harbor report on Immunefi

**Reward**: 10% of rescued funds (up to 60% of max critical reward)

**Requirements**:
- Funds must be returned before submitting report
- Intervention must occur within 6 hours of detection
- Full KYC required for payment
- Detailed incident report required

**Note**: This does not provide immunity from criminal prosecution.
The project agrees not to pursue civil action if conditions are met.
```

---

## 10. Program Rules

### 10.1 Eligibility

**Eligible Participants:**
- Independent security researchers
- Security firms and teams
- Any individual acting in good faith

**Ineligible Participants:**
- Current employees or contractors of the project
- Auditors who audited the in-scope contracts (within 6 months)
- Individuals on OFAC sanctions lists
- Residents of restricted jurisdictions
- Minors (under 18)

### 10.2 Submission Rules

1. **One Vulnerability Per Report**: Submit separate reports for separate issues
2. **First Come, First Served**: First valid report receives the reward
3. **No Duplicate Rewards**: Same vulnerability, same root cause = one reward
4. **Quality Over Quantity**: Low-quality spam may result in ban
5. **Rate Limits**: Maximum 5 reports per 48-hour period
6. **PoC Required**: Critical and High require working proof of concept

### 10.3 Prohibited Actions

The following actions will result in disqualification:

- [ ] Public disclosure without authorization
- [ ] Attempting to exploit beyond proof of concept
- [ ] Attacking production systems (use forks/testnets)
- [ ] Social engineering employees
- [ ] Accessing real user data
- [ ] Denial of service attacks
- [ ] Spam submissions or automated scanning
- [ ] Attempting to extort the project
- [ ] Violating any applicable laws

### 10.4 Program Changes

- We reserve the right to modify program terms at any time
- Changes will be announced on the Immunefi page
- Existing valid reports are not affected by changes
- Significant changes will have 7-day notice

---

## 11. Platform Selection

### 11.1 Platform Comparison

| Feature | Immunefi | HackerOne | Code4rena |
|---------|----------|-----------|-----------|
| Web3 Focus | Primary | Secondary | Primary |
| Smart Contract Expertise | High | Medium | High |
| Researcher Pool | 40,000+ | 1,000,000+ | 15,000+ |
| Triage Support | Yes | Yes | Community |
| Safe Harbor Framework | Yes | No | No |
| KYC/Payment | Included | Included | Included |
| Platform Fee | 10% | 20-25% | 10-20% |
| Audit Competitions | No | No | Yes |

### 11.2 Recommendation: Immunefi

**Why Immunefi:**
- Largest Web3-focused platform
- Built-in Safe Harbor framework
- Lower fees than traditional platforms
- Deep smart contract expertise
- 330+ active DeFi programs
- $100M+ paid to whitehats

### 11.3 Implementation Steps

```
┌─────────────────────────────────────────────────────────────────┐
│                  PROGRAM LAUNCH CHECKLIST                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WEEK 1: SETUP                                                  │
│  [ ] Sign up for Immunefi                                       │
│  [ ] Complete project verification                              │
│  [ ] Set up payment wallet (multisig recommended)               │
│  [ ] Designate security contacts                                │
│                                                                 │
│  WEEK 2: CONFIGURATION                                          │
│  [ ] Define scope (contracts, web, infrastructure)              │
│  [ ] Set reward tiers                                           │
│  [ ] Write program rules                                        │
│  [ ] Create response templates                                  │
│  [ ] Configure notification channels                            │
│                                                                 │
│  WEEK 3: REVIEW                                                 │
│  [ ] Legal review of terms                                      │
│  [ ] Internal team training                                     │
│  [ ] Test submission workflow                                   │
│  [ ] Fund reward pool                                           │
│                                                                 │
│  WEEK 4: LAUNCH                                                 │
│  [ ] Publish program on Immunefi                                │
│  [ ] Announce on social media                                   │
│  [ ] Post in security communities                               │
│  [ ] Monitor initial submissions                                │
│                                                                 │
│  ONGOING                                                        │
│  [ ] Weekly triage meetings                                     │
│  [ ] Monthly program review                                     │
│  [ ] Quarterly reward pool replenishment                        │
│  [ ] Annual scope and reward review                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 12. Program Promotion

### 12.1 Announcement Channels

| Channel | Audience | Timing |
|---------|----------|--------|
| Twitter/X | General crypto community | Launch day |
| Discord | Protocol community | Launch day |
| Immunefi Boost | Security researchers | Week 1 |
| Security Newsletters | Infosec community | Week 1-2 |
| Reddit (r/ethsecurity) | Ethereum security | Week 1 |
| Telegram Groups | DeFi security groups | Week 1-2 |

### 12.2 Promotional Content

#### Launch Announcement (Twitter)

```
🔒 Introducing the [Protocol] Bug Bounty Program

Up to $500,000 for Critical vulnerabilities

Powered by @immunefi

What's in scope:
• Smart contracts
• Web application
• API endpoints

Start hunting: [link]

Full details 🧵👇
```

#### Press Release Key Points

1. Program launch with specific reward amounts
2. Total funds protected (TVL)
3. Commitment to security
4. Partnership with Immunefi
5. Call to action for researchers

### 12.3 Researcher Engagement

**Outreach to Top Researchers:**
- Direct invitations to known smart contract auditors
- Engagement with past Immunefi top earners
- Participation in security community events
- Sponsorship of CTF competitions

**Ongoing Engagement:**
- Monthly security updates
- Researcher appreciation posts
- Leaderboard recognition
- Swag for top contributors

---

## 13. Metrics and Reporting

### 13.1 Key Metrics

| Metric | Target | Tracking |
|--------|--------|----------|
| Time to First Response | <24 hours | Immunefi dashboard |
| Valid Report Rate | >20% | Monthly report |
| Critical Issues Found | Minimize | Quarterly report |
| Average Payout Time | <14 days | Payment records |
| Researcher Satisfaction | >4/5 | Survey |

### 13.2 Monthly Report Template

```markdown
# Bug Bounty Program Monthly Report

**Period**: [Month Year]
**Prepared by**: [Security Team]

## Summary
- Total Submissions: [X]
- Valid Reports: [X]
- Rewards Paid: $[X]

## Severity Breakdown
| Severity | Count | Total Paid |
|----------|-------|------------|
| Critical | X | $X |
| High | X | $X |
| Medium | X | $X |
| Low | X | $X |

## Response Times
- Average First Response: [X] hours
- Average Resolution Time: [X] days

## Notable Findings
1. [Summary of significant finding]
2. [Summary of significant finding]

## Remediation Status
- [ ] [Issue 1]: Fixed
- [ ] [Issue 2]: In Progress
- [ ] [Issue 3]: Scheduled

## Budget Status
- Starting Balance: $[X]
- Spent This Month: $[X]
- Remaining Balance: $[X]

## Recommendations
- [Any suggested changes to program]
```

---

## 14. Appendix

### 14.1 Example Programs (Reference)

#### Aave Bug Bounty
- **Max Reward**: $1,000,000
- **Scope**: Aave V2, V3, Governance, Safety Module, GHO
- **Payment**: Mix of AAVE and USDC
- **Platform**: Immunefi

#### Compound Finance
- **Max Reward**: $1,000,000
- **Scope**: Compound III, Governance
- **Payment**: COMP token
- **Platform**: Immunefi

#### Uniswap v4
- **Max Reward**: $15,500,000
- **Scope**: v4 Core and Periphery contracts
- **Payment**: USDC
- **Platform**: Cantina

### 14.2 Resources

**Platforms:**
- [Immunefi](https://immunefi.com/)
- [HackerOne](https://www.hackerone.com/)
- [Code4rena](https://code4rena.com/)
- [Cantina](https://cantina.xyz/)

**Guidelines:**
- [Immunefi Bug Bounty Best Practices](https://immunefi.com/learn/)
- [OWASP Vulnerability Disclosure Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Vulnerability_Disclosure_Cheat_Sheet.html)
- [ISO 29147: Vulnerability Disclosure](https://www.iso.org/standard/72311.html)

**Communities:**
- Immunefi Discord
- DeFi Security Alliance
- Security Alliance (SEAL)

### 14.3 Contact Information

```
Security Team: security@predictionmarket.com
Immunefi Program: https://immunefi.com/bug-bounty/predictionmarket/
Emergency Hotline: [For active exploits only]
Discord: [#security channel]
```

---

*Document Version: 1.0*
*Last Updated: January 2025*
*Classification: Public*
