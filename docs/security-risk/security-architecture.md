# Security Architecture

## Overview

This document outlines the comprehensive security architecture for a Polymarket-like prediction market platform. It incorporates lessons learned from major DeFi security incidents and follows industry best practices from leading security frameworks.

### Document Purpose

- Define security principles and philosophy
- Establish threat models and risk assessment frameworks
- Document defense-in-depth security layers
- Provide implementation guidance for security controls

### Key Statistics (2024-2025)

- **$4 billion+** lost to crypto hacks in 2025 (37% increase from 2024)
- **$2 billion** stolen by North Korean state actors alone in 2025
- **67%** of smart contract losses attributed to access control vulnerabilities
- **83.3%** of DeFi exploits in 2024 involved flash loans

---

## 1. Security Principles and Philosophy

### 1.1 Core Security Principles

#### Defense in Depth
Multiple layers of security controls ensure that if one layer fails, others remain to protect assets. No single security measure is relied upon exclusively.

#### Least Privilege
Every component, user, and process operates with the minimum permissions necessary to perform its function. Admin capabilities are strictly limited and monitored.

#### Zero Trust Architecture
Never trust, always verify. All requests are authenticated and authorized regardless of source, including internal services.

#### Security by Design
Security is integrated from the earliest stages of development, not added as an afterthought. All code undergoes security review before deployment.

#### Fail Secure
When systems fail, they fail in a secure state. If authentication fails, access is denied. If monitoring fails, alerts are raised.

### 1.2 Security Philosophy

```
┌─────────────────────────────────────────────────────────────────┐
│                    SECURITY PHILOSOPHY                          │
├─────────────────────────────────────────────────────────────────┤
│  1. Assume breach - design for resilience                       │
│  2. Minimize attack surface - only expose what's necessary      │
│  3. Continuous monitoring - detect anomalies early              │
│  4. Rapid response - contain and remediate quickly              │
│  5. Transparency - publish audits, bug bounties, incident reports│
│  6. Community engagement - leverage collective security wisdom   │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 Lessons from Major DeFi Incidents

| Incident | Year | Loss | Root Cause | Lesson Learned |
|----------|------|------|------------|----------------|
| Bybit Hack | 2025 | $1.4B | Supply chain attack on Safe wallet UI | Treat UI code with same rigor as smart contracts |
| Balancer v2 | 2025 | $129M | Logic vulnerability in 5-year-old pools | Continuous security assessment of legacy code |
| DMM Bitcoin | 2024 | $300M | Compromised private keys | Multi-party computation for key management |
| WazirX | 2024 | $230M | Hot wallet compromise | Cold storage separation and multisig |
| Penpie | 2024 | $27M | Reentrancy vulnerability | Checks-Effects-Interactions pattern |
| Sonne Finance | 2024 | $20M | Flash loan + governance exploit | Time delays on governance actions |

---

## 2. Threat Model

### 2.1 Asset Identification

#### Critical Assets (Tier 1)
| Asset | Description | Sensitivity | Impact if Compromised |
|-------|-------------|-------------|----------------------|
| User Funds | USDC/collateral in smart contracts | Critical | Direct financial loss |
| Private Keys | Signing keys for admin operations | Critical | Total system compromise |
| Smart Contracts | Core prediction market contracts | Critical | Protocol manipulation |
| Oracle Data | Price feeds and outcome resolution | Critical | Incorrect market settlements |
| User Credentials | Wallet signatures, session tokens | High | Account takeover |

#### Important Assets (Tier 2)
| Asset | Description | Sensitivity | Impact if Compromised |
|-------|-------------|-------------|----------------------|
| Order Book Data | Pending orders and trade history | High | Front-running, market manipulation |
| User PII | KYC data (if applicable) | High | Regulatory violations, privacy breach |
| API Keys | Service authentication tokens | High | Unauthorized API access |
| Infrastructure Secrets | Database passwords, cloud credentials | High | Data breach, service disruption |

#### Supporting Assets (Tier 3)
| Asset | Description | Sensitivity | Impact if Compromised |
|-------|-------------|-------------|----------------------|
| Application Logs | System and transaction logs | Medium | Audit trail loss |
| Configuration Data | Service configurations | Medium | Misconfiguration attacks |
| Analytics Data | Market statistics, user behavior | Low | Competitive intelligence leak |

### 2.2 Threat Actors

#### Nation-State Actors (APT Groups)
- **Lazarus Group / Slow Pisces (North Korea)**
  - Capabilities: Advanced persistent threats, supply chain attacks, social engineering
  - Motivation: Financial theft (stole $2B+ in 2025 alone)
  - TTPs: Spear phishing, watering hole attacks, fake job offers to developers
  - Recent activity: Created fake US companies (Blocknovas LLC, Softglide LLC) to target crypto developers

- **Other State Actors**
  - Iranian groups (Evasive Serpens): Credential harvesting, DNS hijacking
  - Russian actors: Sanctions evasion infrastructure

#### Organized Crime
- Professional hacking groups with DeFi expertise
- Ransomware-as-a-service operators
- Money laundering networks

#### Opportunistic Attackers
- Script kiddies using public exploit code
- Automated scanning and exploitation bots
- MEV searchers exploiting public transactions

#### Insider Threats
- Malicious employees with privileged access
- Compromised developer credentials
- Social engineering of team members

#### Competitors
- Market manipulation for competitive advantage
- DDoS attacks to disrupt service
- Intelligence gathering on trading strategies

### 2.3 Attack Vectors

#### Smart Contract Attack Vectors
```
┌─────────────────────────────────────────────────────────────────┐
│              SMART CONTRACT ATTACK VECTORS                      │
├─────────────────────────────────────────────────────────────────┤
│ 1. Reentrancy Attacks                                           │
│    - Single-function reentrancy                                 │
│    - Cross-function reentrancy                                  │
│    - Cross-contract reentrancy                                  │
│    - Read-only reentrancy                                       │
│                                                                 │
│ 2. Access Control Vulnerabilities ($953M in 2024)               │
│    - Missing access modifiers                                   │
│    - Improper role management                                   │
│    - Unprotected admin functions                                │
│                                                                 │
│ 3. Oracle Manipulation ($52M in 2024)                           │
│    - Flash loan price manipulation                              │
│    - TWAP manipulation                                          │
│    - Stale price data exploitation                              │
│                                                                 │
│ 4. Flash Loan Attacks ($33.8M in 2024)                          │
│    - Collateral manipulation                                    │
│    - Governance attacks                                         │
│    - Arbitrage exploitation                                     │
│                                                                 │
│ 5. Logic Errors ($63.8M in 2024)                                │
│    - Integer overflow/underflow                                 │
│    - Incorrect calculations                                     │
│    - State manipulation                                         │
│                                                                 │
│ 6. Signature Replay                                             │
│    - Missing nonce validation                                   │
│    - Cross-chain replay                                         │
│    - EIP-712 implementation errors                              │
└─────────────────────────────────────────────────────────────────┘
```

#### Infrastructure Attack Vectors
- Compromised private keys (>50% of 2024-2025 attacks)
- Supply chain attacks on dependencies
- Cloud infrastructure misconfiguration
- DNS hijacking
- BGP hijacking

#### Application Attack Vectors
- Front-end code injection (as in Bybit attack)
- API exploitation
- Session hijacking
- Phishing and approval phishing
- Social engineering

### 2.4 Risk Assessment Matrix

#### Likelihood Scale
| Level | Description | Probability |
|-------|-------------|-------------|
| 5 - Almost Certain | Expected to occur | >90% |
| 4 - Likely | Will probably occur | 60-90% |
| 3 - Possible | Might occur | 30-60% |
| 2 - Unlikely | Could occur | 10-30% |
| 1 - Rare | May occur in exceptional circumstances | <10% |

#### Impact Scale
| Level | Description | Financial Impact |
|-------|-------------|------------------|
| 5 - Catastrophic | Complete loss of user funds | >$10M |
| 4 - Major | Significant financial loss | $1M-$10M |
| 3 - Moderate | Notable financial or reputational damage | $100K-$1M |
| 2 - Minor | Limited impact | $10K-$100K |
| 1 - Negligible | Minimal impact | <$10K |

#### Risk Matrix

| | Impact 1 | Impact 2 | Impact 3 | Impact 4 | Impact 5 |
|---|:---:|:---:|:---:|:---:|:---:|
| **Likelihood 5** | Medium | Medium | High | Critical | Critical |
| **Likelihood 4** | Low | Medium | High | High | Critical |
| **Likelihood 3** | Low | Medium | Medium | High | High |
| **Likelihood 2** | Low | Low | Medium | Medium | High |
| **Likelihood 1** | Low | Low | Low | Medium | Medium |

#### Risk Register (Top 10)

| Risk ID | Risk Description | Likelihood | Impact | Risk Level | Mitigation |
|---------|-----------------|------------|--------|------------|------------|
| R-001 | Private key compromise | 4 | 5 | Critical | MPC, HSM, multisig |
| R-002 | Smart contract exploit | 3 | 5 | High | Audits, bug bounty |
| R-003 | Oracle manipulation | 3 | 4 | High | Multiple oracles, TWAP |
| R-004 | Supply chain attack | 3 | 5 | High | Dependency auditing |
| R-005 | Front-end compromise | 3 | 4 | High | CSP, SRI, code signing |
| R-006 | Flash loan attack | 3 | 4 | High | Circuit breakers, time delays |
| R-007 | Admin key abuse | 2 | 5 | High | Timelocks, multisig |
| R-008 | DDoS attack | 4 | 2 | Medium | CDN, rate limiting |
| R-009 | Phishing attack | 4 | 3 | High | User education, warnings |
| R-010 | Regulatory action | 3 | 4 | High | Compliance program |

---

## 3. Defense in Depth Layers

### 3.1 Network Security

#### Network Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        INTERNET                                  │
├─────────────────────────────────────────────────────────────────┤
│                    CDN / DDoS Protection                         │
│                    (Cloudflare / AWS Shield)                     │
├─────────────────────────────────────────────────────────────────┤
│                    Web Application Firewall                      │
│                         (WAF Rules)                              │
├─────────────────────────────────────────────────────────────────┤
│                     Load Balancer (TLS)                          │
├─────────────────────────────────────────────────────────────────┤
│ ┌─────────────┐   ┌─────────────┐   ┌─────────────┐            │
│ │ Public Zone │   │ App Zone    │   │ Data Zone   │            │
│ │             │   │             │   │             │            │
│ │ - API GW    │──▶│ - App Svcs  │──▶│ - Databases │            │
│ │ - Web UI    │   │ - Workers   │   │ - Secrets   │            │
│ │             │   │             │   │             │            │
│ └─────────────┘   └─────────────┘   └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                    Management Zone                               │
│          (Bastion hosts, monitoring, CI/CD)                     │
└─────────────────────────────────────────────────────────────────┘
```

#### Network Security Controls

**Perimeter Security**
- [ ] DDoS protection enabled (AWS Shield Advanced / Cloudflare)
- [ ] Web Application Firewall with custom rules
- [ ] Geographic blocking for high-risk regions
- [ ] Rate limiting at edge

**Network Segmentation**
- [ ] VPC with isolated subnets (public, private, data)
- [ ] Security groups with least-privilege rules
- [ ] Network ACLs as additional layer
- [ ] Private subnets for all backend services

**Traffic Encryption**
- [ ] TLS 1.3 for all external traffic
- [ ] mTLS for service-to-service communication
- [ ] VPN for administrative access
- [ ] Encrypted connections to databases

### 3.2 Application Security

#### Secure Development Lifecycle

```
┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐
│ Design │──▶│  Code  │──▶│  Test  │──▶│ Deploy │──▶│ Monitor│
└────────┘   └────────┘   └────────┘   └────────┘   └────────┘
     │            │            │            │            │
     ▼            ▼            ▼            ▼            ▼
  Threat      Static       Dynamic      Config       Runtime
  Modeling    Analysis     Testing      Scanning     Security
```

#### Application Security Controls

**Code Security**
- [ ] Static Application Security Testing (SAST)
- [ ] Software Composition Analysis (SCA)
- [ ] Secret scanning in CI/CD pipeline
- [ ] Code review requirements (2+ approvers for security-critical code)

**Input Validation**
- [ ] Server-side validation for all inputs
- [ ] Parameterized queries (prevent SQL injection)
- [ ] Content-Type validation
- [ ] File upload restrictions and scanning

**Output Encoding**
- [ ] Context-aware output encoding
- [ ] Content Security Policy (CSP) headers
- [ ] Subresource Integrity (SRI) for external scripts
- [ ] X-Frame-Options to prevent clickjacking

**Session Management**
- [ ] Secure session token generation
- [ ] Session timeout policies
- [ ] Session invalidation on logout
- [ ] Concurrent session limits

### 3.3 Smart Contract Security

#### Smart Contract Security Checklist

**Design Phase**
- [ ] Threat modeling for contract interactions
- [ ] Formal specification of invariants
- [ ] Access control matrix defined
- [ ] Upgrade strategy documented

**Development Phase**
- [ ] Use OpenZeppelin contracts for standard functionality
- [ ] Implement Checks-Effects-Interactions pattern
- [ ] Use ReentrancyGuard on state-changing functions
- [ ] Include comprehensive NatSpec documentation

**Testing Phase**
- [ ] 100% test coverage target
- [ ] Fuzz testing with Echidna
- [ ] Symbolic execution with Manticore
- [ ] Integration tests simulating attack scenarios

**Pre-Deployment**
- [ ] Static analysis with Slither
- [ ] External security audit
- [ ] Bug bounty program active
- [ ] Formal verification for critical functions

**Deployment**
- [ ] Timelocks on admin functions (minimum 24-48 hours)
- [ ] Multisig for privileged operations
- [ ] Pausability for emergency response
- [ ] Monitoring and alerting active

#### OWASP Smart Contract Top 10 (2025) Mitigations

| Vulnerability | Mitigation Strategy |
|--------------|---------------------|
| Access Control | RBAC with OpenZeppelin AccessControl, multisig, timelocks |
| Price Oracle Manipulation | Chainlink oracles, TWAP, multiple oracle sources |
| Logic Errors | Formal verification, extensive testing, invariant checks |
| Lack of Input Validation | Require statements, bounds checking, type safety |
| Reentrancy | ReentrancyGuard, CEI pattern, state locks |
| Unchecked External Calls | Check return values, use SafeERC20 |
| Flash Loan Attacks | Time delays, borrowing caps, circuit breakers |
| Integer Issues | Solidity 0.8+ (built-in overflow checks) |
| Insecure Randomness | Chainlink VRF for on-chain randomness |
| Denial of Service | Gas limits, pull over push patterns |

### 3.4 Operational Security (OpSec)

#### Key Management

**Multi-Party Computation (MPC)**
```
┌─────────────────────────────────────────────────────────────────┐
│                    MPC KEY ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    Key Share 1        Key Share 2        Key Share 3           │
│    ┌─────────┐        ┌─────────┐        ┌─────────┐           │
│    │  HSM 1  │        │  HSM 2  │        │  HSM 3  │           │
│    │ (US-E1) │        │ (EU-W1) │        │ (AP-S1) │           │
│    └────┬────┘        └────┬────┘        └────┬────┘           │
│         │                  │                  │                 │
│         └──────────────────┼──────────────────┘                 │
│                            │                                    │
│                            ▼                                    │
│                   ┌─────────────────┐                           │
│                   │ Threshold Sign  │                           │
│                   │    (2 of 3)     │                           │
│                   └─────────────────┘                           │
│                                                                 │
│  Benefits:                                                      │
│  - No single point of compromise                                │
│  - Geographic distribution                                      │
│  - Hardware-backed security                                     │
│  - Operational resilience                                       │
└─────────────────────────────────────────────────────────────────┘
```

**Key Management Checklist**
- [ ] MPC implementation for signing keys (e.g., Fireblocks, Sepior)
- [ ] HSM backing for key material (FIPS 140-2 Level 3)
- [ ] Geographic distribution of key shares
- [ ] Regular key rotation schedule
- [ ] Secure key ceremony procedures documented
- [ ] Key recovery procedures tested

#### Admin Operations

**Privileged Access Management**
- [ ] Just-in-time access provisioning
- [ ] Session recording for admin actions
- [ ] Separation of duties (no single admin for critical ops)
- [ ] Break-glass procedures for emergencies

**Change Management**
- [ ] All changes through version control
- [ ] Peer review for all deployments
- [ ] Staged rollouts (dev → staging → canary → production)
- [ ] Rollback procedures tested

---

## 4. Authentication and Authorization

### 4.1 Wallet-Based Authentication

#### Authentication Flow
```
┌─────────┐                    ┌─────────┐                    ┌─────────┐
│  User   │                    │  dApp   │                    │ Backend │
└────┬────┘                    └────┬────┘                    └────┬────┘
     │                              │                              │
     │ 1. Connect Wallet            │                              │
     │─────────────────────────────▶│                              │
     │                              │                              │
     │                              │ 2. Request Nonce             │
     │                              │─────────────────────────────▶│
     │                              │                              │
     │                              │ 3. Return Nonce + Message    │
     │                              │◀─────────────────────────────│
     │                              │                              │
     │ 4. Sign Message (EIP-712)    │                              │
     │◀─────────────────────────────│                              │
     │                              │                              │
     │ 5. Return Signature          │                              │
     │─────────────────────────────▶│                              │
     │                              │                              │
     │                              │ 6. Verify Signature          │
     │                              │─────────────────────────────▶│
     │                              │                              │
     │                              │ 7. Issue Session Token       │
     │                              │◀─────────────────────────────│
     │                              │                              │
     │ 8. Authenticated Session     │                              │
     │◀─────────────────────────────│                              │
```

#### EIP-712 Implementation

```solidity
// Domain separator for signature binding
bytes32 constant DOMAIN_TYPEHASH = keccak256(
    "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
);

// Authentication message type
bytes32 constant AUTH_TYPEHASH = keccak256(
    "Authentication(address wallet,uint256 nonce,uint256 expiry)"
);

// Verify signature with replay protection
function verifyAuth(
    address wallet,
    uint256 nonce,
    uint256 expiry,
    bytes memory signature
) external returns (bool) {
    require(block.timestamp < expiry, "Signature expired");
    require(nonces[wallet] == nonce, "Invalid nonce");

    bytes32 structHash = keccak256(abi.encode(
        AUTH_TYPEHASH,
        wallet,
        nonce,
        expiry
    ));

    bytes32 digest = _hashTypedDataV4(structHash);
    address signer = ECDSA.recover(digest, signature);

    require(signer == wallet, "Invalid signature");

    nonces[wallet]++;  // Increment nonce to prevent replay
    return true;
}
```

#### Wallet Authentication Security Checklist
- [ ] Use EIP-712 typed data signing
- [ ] Include nonce in signed message
- [ ] Include expiration timestamp
- [ ] Include chain ID to prevent cross-chain replay
- [ ] Validate signature server-side
- [ ] Rate limit authentication attempts

### 4.2 Session Management

#### Session Token Requirements
| Requirement | Implementation |
|-------------|----------------|
| Token Generation | Cryptographically secure random (256 bits) |
| Token Storage | HttpOnly, Secure, SameSite cookies |
| Token Lifetime | 24 hours maximum, sliding window |
| Token Refresh | Refresh token rotation |
| Token Revocation | Redis-backed revocation list |

#### Session Security Controls
- [ ] Bind session to wallet address
- [ ] Bind session to IP range (optional, may cause issues)
- [ ] Bind session to user agent
- [ ] Implement absolute timeout (24 hours)
- [ ] Implement idle timeout (1 hour)
- [ ] Single session per wallet (optional)
- [ ] Session invalidation on sensitive actions

### 4.3 Role-Based Access Control (RBAC)

#### Role Hierarchy
```
┌─────────────────────────────────────────────────────────────────┐
│                        ROLE HIERARCHY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                        ┌──────────┐                             │
│                        │  Owner   │                             │
│                        │(Multisig)│                             │
│                        └────┬─────┘                             │
│                             │                                   │
│              ┌──────────────┼──────────────┐                    │
│              ▼              ▼              ▼                    │
│        ┌──────────┐  ┌──────────┐  ┌──────────┐                │
│        │  Admin   │  │ Guardian │  │ Operator │                │
│        └────┬─────┘  └────┬─────┘  └────┬─────┘                │
│             │             │             │                       │
│             ▼             ▼             ▼                       │
│        - Upgrade     - Pause/      - Create                     │
│          contracts     Unpause       markets                    │
│        - Set params  - Emergency   - Resolve                    │
│        - Grant roles   withdraw      outcomes                   │
│                                                                 │
│        ┌──────────────────────────────────────┐                │
│        │              User                     │                │
│        │  - Trade, deposit, withdraw           │                │
│        └──────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────────────┘
```

#### Access Control Matrix

| Action | User | Operator | Guardian | Admin | Owner |
|--------|:----:|:--------:|:--------:|:-----:|:-----:|
| Trade | X | X | X | X | X |
| Deposit/Withdraw | X | X | X | X | X |
| Create Market | | X | | X | X |
| Resolve Outcome | | X | | X | X |
| Pause Protocol | | | X | X | X |
| Emergency Withdraw | | | X | | X |
| Update Parameters | | | | X | X |
| Upgrade Contracts | | | | | X |
| Grant/Revoke Roles | | | | | X |

#### Smart Contract RBAC Implementation

```solidity
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/governance/TimelockController.sol";

contract PredictionMarket is AccessControl {
    bytes32 public constant OPERATOR_ROLE = keccak256("OPERATOR_ROLE");
    bytes32 public constant GUARDIAN_ROLE = keccak256("GUARDIAN_ROLE");
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");

    TimelockController public timelock;

    constructor(address _timelock) {
        timelock = TimelockController(payable(_timelock));

        // Owner is the timelock (multisig controlled)
        _grantRole(DEFAULT_ADMIN_ROLE, _timelock);
    }

    function createMarket(bytes32 questionId)
        external
        onlyRole(OPERATOR_ROLE)
    {
        // Market creation logic
    }

    function pause()
        external
        onlyRole(GUARDIAN_ROLE)
    {
        _pause();
        emit ProtocolPaused(msg.sender);
    }

    function updateFee(uint256 newFee)
        external
        onlyRole(ADMIN_ROLE)
    {
        // Must go through timelock
        require(msg.sender == address(timelock), "Must use timelock");
        fee = newFee;
    }
}
```

---

## 5. Data Protection

### 5.1 Encryption at Rest

#### Data Classification and Encryption Requirements

| Data Type | Classification | Encryption | Key Management |
|-----------|---------------|------------|----------------|
| Private Keys | Critical | AES-256-GCM | HSM/MPC |
| User PII (KYC) | Confidential | AES-256-GCM | AWS KMS |
| Session Tokens | Confidential | AES-256-GCM | AWS KMS |
| Transaction Data | Internal | AES-256-GCM | AWS KMS |
| Log Data | Internal | AES-256-GCM | AWS KMS |
| Public Blockchain Data | Public | N/A | N/A |

#### Database Encryption

**PostgreSQL/RDS Configuration**
```yaml
# RDS Encryption Configuration
storage_encrypted: true
kms_key_id: "alias/prediction-market-db-key"

# Application-level encryption for sensitive fields
encrypted_columns:
  - users.kyc_data
  - users.email
  - api_keys.secret_hash
```

### 5.2 Encryption in Transit

#### TLS Configuration

**Minimum Requirements**
- TLS 1.2 minimum, TLS 1.3 preferred
- Strong cipher suites only
- Perfect Forward Secrecy (ECDHE)
- HSTS enabled with 1-year max-age

**Recommended Cipher Suites**
```
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
TLS_AES_128_GCM_SHA256
ECDHE-RSA-AES256-GCM-SHA384
ECDHE-RSA-AES128-GCM-SHA256
```

**Certificate Management**
- [ ] Automated certificate renewal (Let's Encrypt / AWS ACM)
- [ ] Certificate pinning for mobile apps
- [ ] CAA records configured
- [ ] CT logging enabled

### 5.3 Key Management

#### Key Hierarchy
```
┌─────────────────────────────────────────────────────────────────┐
│                      KEY HIERARCHY                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    ┌─────────────────┐                          │
│                    │   Master Key    │  (HSM-protected)         │
│                    │    (KEK)        │                          │
│                    └────────┬────────┘                          │
│                             │                                   │
│         ┌───────────────────┼───────────────────┐               │
│         ▼                   ▼                   ▼               │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │ Signing Key │    │   DB Key    │    │   API Key   │         │
│  │   (MPC)     │    │  (KMS)      │    │    (KMS)    │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Key Rotation Policy

| Key Type | Rotation Frequency | Procedure |
|----------|-------------------|-----------|
| Master Key (KEK) | Annual | Key ceremony |
| Signing Keys | 90 days | Automated MPC re-sharing |
| Database Keys | 90 days | AWS KMS automatic |
| API Keys | On demand | User-initiated |
| TLS Certificates | 90 days | Automated (ACME) |

---

## 6. API Security

### 6.1 Rate Limiting

#### Rate Limit Tiers

| Tier | Requests/Second | Requests/Minute | Requests/Hour |
|------|----------------|-----------------|---------------|
| Anonymous | 1 | 30 | 100 |
| Authenticated | 10 | 300 | 5,000 |
| Premium | 50 | 1,500 | 25,000 |
| Institutional | 200 | 6,000 | 100,000 |

#### Rate Limiting Implementation

```python
# Redis-based rate limiting with sliding window
import redis
from functools import wraps

class RateLimiter:
    def __init__(self, redis_client):
        self.redis = redis_client

    def is_allowed(self, key: str, limit: int, window: int) -> bool:
        """
        Sliding window rate limiting

        Args:
            key: Unique identifier (IP, user_id, api_key)
            limit: Maximum requests allowed
            window: Time window in seconds
        """
        now = time.time()
        window_start = now - window

        pipe = self.redis.pipeline()
        pipe.zremrangebyscore(key, 0, window_start)
        pipe.zadd(key, {str(now): now})
        pipe.zcard(key)
        pipe.expire(key, window)

        results = pipe.execute()
        request_count = results[2]

        return request_count <= limit
```

#### Endpoint-Specific Limits

| Endpoint | Limit | Window | Reason |
|----------|-------|--------|--------|
| POST /auth/login | 5 | 1 min | Brute force prevention |
| POST /orders | 100 | 1 min | Order spam prevention |
| GET /markets | 60 | 1 min | Standard API access |
| POST /withdraw | 3 | 1 hour | Financial protection |

### 6.2 Input Validation

#### Validation Rules

```python
from pydantic import BaseModel, Field, validator
from eth_utils import is_address

class CreateOrderRequest(BaseModel):
    market_id: str = Field(..., regex=r'^0x[a-fA-F0-9]{64}$')
    side: str = Field(..., regex=r'^(BUY|SELL)$')
    amount: int = Field(..., ge=1, le=10**18)  # 1 wei to 1 token
    price: int = Field(..., ge=0, le=10**18)   # 0 to 100%

    @validator('market_id')
    def validate_market_exists(cls, v):
        # Check market exists in database
        if not market_exists(v):
            raise ValueError('Market does not exist')
        return v

    @validator('amount')
    def validate_amount(cls, v):
        # Must be greater than minimum order size
        if v < MIN_ORDER_SIZE:
            raise ValueError(f'Amount must be >= {MIN_ORDER_SIZE}')
        return v

class WithdrawRequest(BaseModel):
    recipient: str
    amount: int = Field(..., ge=1)

    @validator('recipient')
    def validate_address(cls, v):
        if not is_address(v):
            raise ValueError('Invalid Ethereum address')
        return v.lower()  # Normalize to lowercase
```

### 6.3 Signature Verification (HMAC)

#### API Request Signing

```python
import hmac
import hashlib
import time

class APIAuthenticator:
    def sign_request(self,
                     api_secret: str,
                     method: str,
                     path: str,
                     timestamp: int,
                     body: str = "") -> str:
        """
        Generate HMAC-SHA256 signature for API request

        Message format: timestamp + method + path + body
        """
        message = f"{timestamp}{method}{path}{body}"
        signature = hmac.new(
            api_secret.encode('utf-8'),
            message.encode('utf-8'),
            hashlib.sha256
        ).hexdigest()
        return signature

    def verify_request(self,
                       api_key: str,
                       signature: str,
                       method: str,
                       path: str,
                       timestamp: int,
                       body: str = "") -> bool:
        """
        Verify API request signature
        """
        # Check timestamp is within acceptable window (5 seconds)
        now = int(time.time() * 1000)
        if abs(now - timestamp) > 5000:
            raise ValueError("Request timestamp too old")

        # Get API secret from database
        api_secret = get_api_secret(api_key)
        if not api_secret:
            raise ValueError("Invalid API key")

        # Calculate expected signature
        expected = self.sign_request(
            api_secret, method, path, timestamp, body
        )

        # Constant-time comparison to prevent timing attacks
        return hmac.compare_digest(signature, expected)
```

#### Request Headers

```http
POST /api/v1/orders HTTP/1.1
Host: api.predictionmarket.com
Content-Type: application/json
X-API-Key: pk_live_abc123...
X-Timestamp: 1706054400000
X-Signature: 8a3f9b2c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a
```

---

## 7. Infrastructure Security

### 7.1 Cloud Security (AWS)

#### AWS Security Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    AWS SECURITY LAYERS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                AWS Organizations                         │   │
│  │  - SCPs (Service Control Policies)                       │   │
│  │  - Consolidated billing                                  │   │
│  │  - Multi-account strategy                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   IAM                                    │   │
│  │  - Least privilege policies                              │   │
│  │  - MFA required for all users                            │   │
│  │  - Role-based access                                     │   │
│  │  - No long-term access keys                              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Network Security                            │   │
│  │  - VPC with private subnets                              │   │
│  │  - Security groups (stateful)                            │   │
│  │  - NACLs (stateless)                                     │   │
│  │  - VPC Flow Logs enabled                                 │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Data Protection                             │   │
│  │  - S3 bucket policies (no public access)                 │   │
│  │  - RDS encryption enabled                                │   │
│  │  - KMS for key management                                │   │
│  │  - Secrets Manager for credentials                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Detection & Response                        │   │
│  │  - GuardDuty enabled                                     │   │
│  │  - CloudTrail logging                                    │   │
│  │  - Config rules                                          │   │
│  │  - Security Hub                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

#### AWS Security Checklist

**Identity & Access Management**
- [ ] MFA enforced for all IAM users
- [ ] Root account secured and never used
- [ ] IAM policies follow least privilege
- [ ] Service accounts use IAM roles (not keys)
- [ ] IAM Access Analyzer enabled

**Network Security**
- [ ] Default VPC deleted
- [ ] Custom VPCs with proper CIDR planning
- [ ] Private subnets for all workloads
- [ ] VPC endpoints for AWS services
- [ ] VPC Flow Logs enabled and analyzed

**Data Protection**
- [ ] S3 Block Public Access enabled (account-level)
- [ ] S3 buckets encrypted (SSE-KMS)
- [ ] RDS encryption enabled
- [ ] EBS volumes encrypted
- [ ] KMS key rotation enabled

**Detection**
- [ ] GuardDuty enabled in all regions
- [ ] CloudTrail enabled (multi-region)
- [ ] Config rules for compliance
- [ ] Security Hub aggregating findings

### 7.2 Container Security

#### Container Security Best Practices

**Image Security**
- [ ] Use minimal base images (distroless, Alpine)
- [ ] Scan images for vulnerabilities (Trivy, ECR scanning)
- [ ] Sign and verify images
- [ ] Use specific image tags (not `latest`)
- [ ] Regular base image updates

**Runtime Security**
- [ ] Run as non-root user
- [ ] Read-only root filesystem
- [ ] Drop unnecessary capabilities
- [ ] No privileged containers
- [ ] Resource limits defined

**Example Secure Dockerfile**
```dockerfile
# Use distroless base image
FROM gcr.io/distroless/nodejs:18

# Copy only necessary files
COPY --chown=nonroot:nonroot dist/ /app/
COPY --chown=nonroot:nonroot node_modules/ /app/node_modules/

# Use non-root user
USER nonroot

# Set working directory
WORKDIR /app

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD ["/nodejs/bin/node", "healthcheck.js"]

# Start application
CMD ["/nodejs/bin/node", "server.js"]
```

**Kubernetes Security**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prediction-market-api
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
  containers:
  - name: api
    image: prediction-market-api:v1.2.3
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
    resources:
      limits:
        cpu: "1"
        memory: "512Mi"
      requests:
        cpu: "100m"
        memory: "256Mi"
```

### 7.3 Network Segmentation

#### Network Zone Architecture

| Zone | Purpose | Ingress | Egress |
|------|---------|---------|--------|
| DMZ | Load balancers, WAF | Internet | App Zone |
| App Zone | Application services | DMZ only | Data Zone, External APIs |
| Data Zone | Databases, secrets | App Zone only | None (except updates) |
| Management | CI/CD, monitoring | VPN only | All zones |

---

## 8. Security Monitoring

### 8.1 Monitoring Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  SECURITY MONITORING STACK                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    DATA SOURCES                           │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │  │
│  │  │ Smart    │ │ App      │ │ Infra    │ │ Network  │     │  │
│  │  │ Contracts│ │ Logs     │ │ Metrics  │ │ Flows    │     │  │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘     │  │
│  └───────┼────────────┼────────────┼────────────┼───────────┘  │
│          │            │            │            │               │
│          ▼            ▼            ▼            ▼               │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                  COLLECTION LAYER                         │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐                  │  │
│  │  │ Forta    │ │ DataDog/ │ │ AWS      │                  │  │
│  │  │ Network  │ │ Splunk   │ │ CloudWatch│                 │  │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘                  │  │
│  └───────┼────────────┼────────────┼────────────────────────┘  │
│          │            │            │                            │
│          ▼            ▼            ▼                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                  ANALYSIS LAYER                           │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │              SIEM / Security Hub                    │  │  │
│  │  │  - Correlation rules                                │  │  │
│  │  │  - Anomaly detection                                │  │  │
│  │  │  - Threat intelligence                              │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                  RESPONSE LAYER                           │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │  │
│  │  │ PagerDuty│ │ Slack    │ │ Auto     │ │ Playbooks│     │  │
│  │  │          │ │ Alerts   │ │ Response │ │          │     │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘     │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 On-Chain Monitoring with Forta

#### Forta Detection Bots

| Bot Type | Purpose | Alert Threshold |
|----------|---------|-----------------|
| Large Transfer | Detect unusual fund movements | >$100K |
| Governance | Monitor proposal/voting changes | Any change |
| Oracle | Detect price deviation | >5% deviation |
| Access Control | Role changes | Any change |
| Pause Events | Protocol pause/unpause | Any event |
| Upgrade Events | Contract upgrades | Any event |

#### OpenZeppelin Defender Integration

```javascript
// Defender Sentinel Configuration
const sentinelConfig = {
  name: 'Prediction Market Monitor',
  network: 'polygon',
  addresses: [
    '0x1234...', // Market Factory
    '0x5678...', // Exchange
    '0x9abc...'  // Oracle
  ],
  conditions: {
    function: ['pause', 'unpause', 'transferOwnership', 'upgradeTo'],
    event: ['Paused', 'Unpaused', 'OwnershipTransferred', 'Upgraded']
  },
  notification: {
    channels: ['slack', 'pagerduty', 'email']
  },
  autotask: {
    trigger: 'sentinel',
    action: 'emergency-response'  // Auto-pause on critical alerts
  }
};
```

### 8.3 Alert Definitions

| Alert ID | Description | Severity | Response SLA |
|----------|-------------|----------|--------------|
| SEC-001 | Unusual large withdrawal | Critical | 5 min |
| SEC-002 | Multiple failed auth attempts | High | 15 min |
| SEC-003 | Admin action detected | High | 15 min |
| SEC-004 | Oracle price deviation >5% | High | 5 min |
| SEC-005 | Contract upgrade initiated | Critical | Immediate |
| SEC-006 | New admin role granted | Critical | Immediate |
| SEC-007 | DDoS attack detected | High | 5 min |
| SEC-008 | API abuse pattern | Medium | 30 min |

---

## 9. Penetration Testing Plan

### 9.1 Testing Scope

#### In-Scope Components

| Component | Testing Type | Frequency |
|-----------|--------------|-----------|
| Smart Contracts | Code audit, fuzzing | Per release |
| Web Application | OWASP Top 10, API testing | Quarterly |
| Mobile App | OWASP Mobile Top 10 | Quarterly |
| Infrastructure | Cloud config review | Quarterly |
| Network | External/internal pentest | Annual |

#### Out-of-Scope
- Third-party services (Polygon network, Chainlink)
- Physical security
- Social engineering of non-consenting parties

### 9.2 Testing Methodology

#### Smart Contract Testing
1. **Static Analysis**: Slither, Mythril, Securify
2. **Dynamic Analysis**: Echidna fuzzing, Foundry tests
3. **Manual Review**: Business logic, economic attacks
4. **Formal Verification**: Critical functions only

#### Web Application Testing
1. **Reconnaissance**: Subdomain enumeration, technology fingerprinting
2. **Authentication Testing**: Session management, wallet auth
3. **Authorization Testing**: RBAC bypass attempts
4. **Input Validation**: Injection attacks, boundary testing
5. **Business Logic**: Trading manipulation, order book attacks

#### Infrastructure Testing
1. **External Assessment**: Port scanning, service enumeration
2. **Cloud Configuration**: IAM, S3, security groups
3. **Container Security**: Image vulnerabilities, runtime config
4. **Network Segmentation**: Lateral movement attempts

### 9.3 Testing Schedule

| Quarter | Focus Area | Deliverables |
|---------|------------|--------------|
| Q1 | Smart Contract Audit | Audit report, remediation plan |
| Q2 | Web/API Penetration Test | Pentest report, fix verification |
| Q3 | Infrastructure Assessment | Cloud security report |
| Q4 | Red Team Exercise | Full-scope attack simulation |

### 9.4 Reporting Requirements

**Vulnerability Report Format**
```markdown
## Vulnerability: [Title]

**Severity**: Critical / High / Medium / Low / Informational
**CVSS Score**: X.X
**CWE**: CWE-XXX

### Description
[Detailed description of the vulnerability]

### Impact
[What an attacker could achieve]

### Reproduction Steps
1. [Step 1]
2. [Step 2]
3. [Step 3]

### Proof of Concept
[Code/screenshots demonstrating the issue]

### Remediation
[Recommended fix with code examples]

### References
[CVE, CWE, OWASP references]
```

---

## 10. Incident Response

### 10.1 Incident Classification

| Severity | Description | Example | Response Time |
|----------|-------------|---------|---------------|
| P1 - Critical | Active exploitation, fund loss | Smart contract exploit | Immediate |
| P2 - High | Imminent threat, partial service impact | Credential compromise | 15 min |
| P3 - Medium | Potential threat, minor impact | Vulnerability discovered | 4 hours |
| P4 - Low | Minor issue, no immediate threat | Misconfiguration | 24 hours |

### 10.2 Response Playbooks

#### Smart Contract Exploit Response

```
┌─────────────────────────────────────────────────────────────────┐
│              SMART CONTRACT EXPLOIT RESPONSE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. DETECT (0-5 min)                                            │
│     □ Forta alert received                                      │
│     □ Verify alert is genuine                                   │
│     □ Assess initial scope                                      │
│                                                                 │
│  2. CONTAIN (5-15 min)                                          │
│     □ Activate incident commander                               │
│     □ Execute emergency pause (if available)                    │
│     □ Block attacker addresses                                  │
│     □ Disable affected functions                                │
│                                                                 │
│  3. ASSESS (15-60 min)                                          │
│     □ Determine attack vector                                   │
│     □ Calculate fund loss                                       │
│     □ Identify affected users                                   │
│     □ Document evidence                                         │
│                                                                 │
│  4. REMEDIATE (1-24 hours)                                      │
│     □ Develop and test fix                                      │
│     □ Deploy patched contracts                                  │
│     □ Restore service                                           │
│     □ Monitor for follow-up attacks                             │
│                                                                 │
│  5. RECOVER (24-72 hours)                                       │
│     □ User communication                                        │
│     □ Fund recovery efforts                                     │
│     □ Insurance claim (if applicable)                           │
│     □ Law enforcement coordination                              │
│                                                                 │
│  6. POST-INCIDENT (1-2 weeks)                                   │
│     □ Root cause analysis                                       │
│     □ Lessons learned                                           │
│     □ Process improvements                                      │
│     □ Public incident report                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. Compliance and Governance

### 11.1 Security Governance

#### Security Team Structure
- **CISO**: Overall security strategy and risk management
- **Security Engineers**: Implementation and operations
- **Security Researchers**: Vulnerability assessment and red team
- **Incident Response Team**: 24/7 on-call rotation

#### Security Meetings
- **Weekly**: Security review, vulnerability triage
- **Monthly**: Risk assessment, metrics review
- **Quarterly**: Security roadmap, audit planning

### 11.2 Security Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Mean Time to Detect (MTTD) | <5 min | Monitoring alerts |
| Mean Time to Respond (MTTR) | <30 min | Incident tickets |
| Vulnerability Remediation | 7 days (critical) | Issue tracker |
| Audit Finding Closure | 30 days | Audit tracker |
| Security Training Completion | 100% | LMS |

---

## References

### Research Sources
- [Halborn - Top 100 DeFi Hacks 2025](https://www.halborn.com/reports/top-100-defi-hacks-2025)
- [OWASP Smart Contract Top 10](https://scs.owasp.org/sctop10/)
- [Hacken - DeFi Security Lessons](https://hacken.io/discover/defi-security/)
- [OpenZeppelin Defender Documentation](https://docs.openzeppelin.com/defender/)
- [Forta Network](https://forta.org/)
- [Fireblocks MPC Documentation](https://www.fireblocks.com/what-is-mpc)
- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)
- [Chainalysis 2026 Crypto Crime Report](https://www.chainalysis.com/blog/2026-crypto-crime-report-introduction/)

### Standards and Frameworks
- NIST Cybersecurity Framework
- ISO 27001
- SOC 2 Type II
- EEA DeFi Risk Assessment Guidelines

---

*Document Version: 1.0*
*Last Updated: January 2025*
*Classification: Internal*
