# Gas Optimization Guide

## Comprehensive Gas Optimization for Prediction Markets on Polygon/Ethereum

This guide covers gas optimization strategies for deploying and operating a prediction market platform on Ethereum L1 and Layer 2 networks, with a focus on Polygon PoS and zkEVM.

---

## Table of Contents

1. [Smart Contract Gas Optimization Patterns](#smart-contract-gas-optimization-patterns)
2. [Batch Transaction Strategies](#batch-transaction-strategies)
3. [Meta-Transactions (EIP-2771)](#meta-transactions-eip-2771)
4. [Gas Sponsorship and Relayers](#gas-sponsorship-and-relayers)
5. [Account Abstraction (ERC-4337)](#account-abstraction-erc-4337)
6. [Gasless Transactions for Users](#gasless-transactions-for-users)
7. [L2 Gas Pricing Differences](#l2-gas-pricing-differences)
8. [Gas Estimation Strategies](#gas-estimation-strategies)
9. [Transaction Queuing](#transaction-queuing)
10. [Failed Transaction Handling](#failed-transaction-handling)
11. [Cost Analysis and Projections](#cost-analysis-and-projections)

---

## Smart Contract Gas Optimization Patterns

### Gas Cost Fundamentals

```
EVM OPERATION GAS COSTS
═══════════════════════════════════════════════════════════════════

Operation                          Gas Cost      Notes
──────────────────────────────────────────────────────────────────
Transaction base cost              21,000        Every tx pays this
Storage write (new slot)           22,100        SSTORE to zero→non-zero
Storage write (existing)            5,000        SSTORE non-zero→non-zero
Storage write (clear)               2,900+       SSTORE non-zero→zero (refund)
Storage read                        2,100        SLOAD (cold)
Storage read                          100        SLOAD (warm/same tx)
Memory expansion                    3 per word   Quadratic after 724 bytes
Calldata (zero byte)                    4
Calldata (non-zero byte)               16
LOG operation                        375+        Event emission
Contract creation                   32,000       Base cost
Code deployment                    200/byte      Contract bytecode

═══════════════════════════════════════════════════════════════════
```

### 1. Storage Packing

The most impactful optimization - storage operations are the most expensive EVM operations.

```solidity
// BAD: Each variable takes a full 32-byte slot (4 slots = expensive)
contract Unoptimized {
    uint256 public price;        // Slot 0
    bool public isActive;        // Slot 1 (wastes 31 bytes!)
    address public owner;        // Slot 2
    uint256 public timestamp;    // Slot 3
}

// GOOD: Pack variables into fewer slots (2 slots = cheaper)
contract Optimized {
    uint256 public price;        // Slot 0 (32 bytes)
    uint96 public timestamp;     // Slot 1 (12 bytes)
    address public owner;        // Slot 1 (20 bytes) - packed!
    bool public isActive;        // Can fit in slot 1 or separate

    // Alternative: Use a struct for explicit packing
    struct MarketData {
        uint128 price;           // 16 bytes
        uint64 timestamp;        // 8 bytes
        uint32 volume;           // 4 bytes
        bool isActive;           // 1 byte
        bool isResolved;         // 1 byte
        // Total: 30 bytes - fits in 1 slot!
    }
}
```

**Storage Slot Visualization:**

```
STORAGE PACKING DIAGRAM
═══════════════════════════════════════════════════════════════════

Unoptimized (4 slots):
┌────────────────────────────────────────────────────────────────┐
│ Slot 0: price (uint256)                                        │
│ [████████████████████████████████████████████████████████████] │
│                          32 bytes used                          │
├────────────────────────────────────────────────────────────────┤
│ Slot 1: isActive (bool)                                        │
│ [█░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░] │
│  1 byte used, 31 bytes WASTED!                                 │
├────────────────────────────────────────────────────────────────┤
│ Slot 2: owner (address)                                        │
│ [████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░] │
│           20 bytes used, 12 bytes wasted                       │
├────────────────────────────────────────────────────────────────┤
│ Slot 3: timestamp (uint256)                                    │
│ [████████████████████████████████████████████████████████████] │
│                          32 bytes used                          │
└────────────────────────────────────────────────────────────────┘
Total: 4 SSTORE operations = ~88,400 gas for initial writes

Optimized (2 slots):
┌────────────────────────────────────────────────────────────────┐
│ Slot 0: price (uint256)                                        │
│ [████████████████████████████████████████████████████████████] │
│                          32 bytes used                          │
├────────────────────────────────────────────────────────────────┤
│ Slot 1: owner (20) + timestamp (8) + isActive (1)              │
│ [████████████████████████████████████████████████████████░░░░] │
│                    29 bytes used, 3 available                   │
└────────────────────────────────────────────────────────────────┘
Total: 2 SSTORE operations = ~44,200 gas for initial writes
SAVINGS: ~50%
```

### 2. Calldata vs Memory

```solidity
// BAD: Using memory copies the data (costs gas)
function processBets(uint256[] memory betIds) external {
    // Data is copied to memory - expensive for large arrays
    for (uint i = 0; i < betIds.length; i++) {
        // process
    }
}

// GOOD: Using calldata reads directly from transaction data
function processBets(uint256[] calldata betIds) external {
    // Data stays in calldata - no copy, much cheaper
    for (uint i = 0; i < betIds.length; i++) {
        // process
    }
}

// Gas comparison for 100 element array:
// memory: ~5,000 gas for copy + operations
// calldata: ~500 gas (just operations)
```

### 3. Use Constants and Immutables

```solidity
contract GasEfficient {
    // GOOD: Constants are embedded in bytecode (no SLOAD)
    uint256 public constant FEE_DENOMINATOR = 10000;
    uint256 public constant MAX_MARKETS = 1000;

    // GOOD: Immutables are set once and embedded
    address public immutable FACTORY;
    uint256 public immutable DEPLOYMENT_TIME;

    constructor(address _factory) {
        FACTORY = _factory;  // Set once, never changes
        DEPLOYMENT_TIME = block.timestamp;
    }

    // Reading constant: ~3 gas (inline)
    // Reading immutable: ~3 gas (inline after deployment)
    // Reading storage: ~2,100 gas (SLOAD)
}
```

### 4. Short-Circuit Conditions

```solidity
// BAD: Always evaluates both conditions
function checkEligibility(address user) public view returns (bool) {
    // isWhitelisted() might be expensive SLOAD
    return isWhitelisted(user) && hasMinimumBalance(user);
}

// GOOD: Order conditions by likelihood and cost
function checkEligibility(address user) public view returns (bool) {
    // Put cheap/likely-to-fail checks first
    if (!hasMinimumBalance(user)) return false;  // Cheap check first
    if (!isWhitelisted(user)) return false;       // Expensive check second
    return true;
}
```

### 5. Avoid Redundant Storage Reads

```solidity
// BAD: Multiple SLOAD operations for same variable
function updateMarket(uint256 marketId) external {
    require(markets[marketId].isActive, "Not active");      // SLOAD
    require(markets[marketId].endTime > block.timestamp);   // SLOAD
    markets[marketId].lastUpdate = block.timestamp;         // SLOAD + SSTORE
}

// GOOD: Cache storage in memory
function updateMarket(uint256 marketId) external {
    Market storage market = markets[marketId];  // Single reference
    require(market.isActive, "Not active");     // Warm read: 100 gas
    require(market.endTime > block.timestamp);  // Warm read: 100 gas
    market.lastUpdate = block.timestamp;        // Single SSTORE
}

// Even better for read-heavy functions:
function getMarketDetails(uint256 marketId) external view returns (...) {
    Market memory market = markets[marketId];  // Copy once to memory
    // All subsequent reads are from memory (3 gas each)
    return (market.price, market.isActive, market.endTime, ...);
}
```

### 6. Use Unchecked for Safe Math

```solidity
// BAD: Default checked math adds overhead
function sumArray(uint256[] calldata values) external pure returns (uint256) {
    uint256 total = 0;
    for (uint256 i = 0; i < values.length; i++) {
        total += values[i];  // Overflow check on every iteration
    }
    return total;
}

// GOOD: Use unchecked when overflow is impossible
function sumArray(uint256[] calldata values) external pure returns (uint256) {
    uint256 total = 0;
    uint256 length = values.length;
    for (uint256 i = 0; i < length;) {
        unchecked {
            total += values[i];
            ++i;  // Pre-increment is slightly cheaper
        }
    }
    return total;
}

// Gas savings: ~50-100 gas per loop iteration
```

### 7. Custom Errors vs Require Strings

```solidity
// BAD: String error messages stored in bytecode
function oldStyle(uint256 amount) external {
    require(amount > 0, "Amount must be greater than zero");
    require(amount <= maxAmount, "Amount exceeds maximum allowed");
    // Each string adds to deployment and runtime cost
}

// GOOD: Custom errors (Solidity 0.8.4+)
error AmountZero();
error AmountTooHigh(uint256 amount, uint256 max);

function newStyle(uint256 amount) external {
    if (amount == 0) revert AmountZero();
    if (amount > maxAmount) revert AmountTooHigh(amount, maxAmount);
    // Smaller bytecode, cheaper to deploy and revert
}

// Gas savings: ~50 gas per revert, significant deployment savings
```

### 8. Events Optimization

```solidity
// Events are cheaper than storage but still cost gas
// Use indexed parameters wisely (max 3 per event)

// BAD: Logging everything
event MarketCreated(
    uint256 indexed marketId,
    address indexed creator,
    string indexed title,       // Strings can't be indexed efficiently
    uint256 endTime,
    uint256 fee,
    bytes32 description
);

// GOOD: Index only what you'll filter by
event MarketCreated(
    uint256 indexed marketId,
    address indexed creator,
    bytes32 titleHash,          // Store hash instead of string
    uint256 endTime
    // Emit less data, store details off-chain
);
```

### Complete Optimized Prediction Market Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

// Custom errors
error Unauthorized();
error MarketNotActive();
error InvalidAmount();
error MarketExpired();
error InsufficientBalance();

contract OptimizedPredictionMarket {
    // Packed struct (1 slot)
    struct Market {
        uint96 totalYesShares;
        uint96 totalNoShares;
        uint32 endTime;
        uint16 feeBps;
        bool resolved;
        bool outcome;
    }

    // Packed position struct (1 slot)
    struct Position {
        uint128 yesShares;
        uint128 noShares;
    }

    // Constants (no storage cost)
    uint256 private constant FEE_DENOMINATOR = 10000;
    uint256 private constant PRECISION = 1e18;

    // Immutables (no storage cost after deployment)
    address public immutable owner;
    address public immutable collateralToken;

    // Storage
    mapping(uint256 => Market) public markets;
    mapping(uint256 => mapping(address => Position)) public positions;
    uint256 public marketCount;

    constructor(address _collateralToken) {
        owner = msg.sender;
        collateralToken = _collateralToken;
    }

    function createMarket(
        uint32 endTime,
        uint16 feeBps
    ) external returns (uint256 marketId) {
        if (msg.sender != owner) revert Unauthorized();

        unchecked {
            marketId = marketCount++;
        }

        // Single SSTORE for packed struct
        markets[marketId] = Market({
            totalYesShares: 0,
            totalNoShares: 0,
            endTime: endTime,
            feeBps: feeBps,
            resolved: false,
            outcome: false
        });
    }

    function buyShares(
        uint256 marketId,
        bool isYes,
        uint256 amount
    ) external {
        if (amount == 0) revert InvalidAmount();

        // Cache storage reference
        Market storage market = markets[marketId];

        if (market.resolved) revert MarketNotActive();
        if (block.timestamp >= market.endTime) revert MarketExpired();

        // Calculate shares (simplified AMM)
        uint256 shares = _calculateShares(market, isYes, amount);

        // Update storage (single SSTORE due to packing)
        if (isYes) {
            market.totalYesShares += uint96(shares);
        } else {
            market.totalNoShares += uint96(shares);
        }

        // Update position
        Position storage pos = positions[marketId][msg.sender];
        if (isYes) {
            pos.yesShares += uint128(shares);
        } else {
            pos.noShares += uint128(shares);
        }

        // Transfer collateral (external call last - CEI pattern)
        // IERC20(collateralToken).transferFrom(msg.sender, address(this), amount);
    }

    function _calculateShares(
        Market storage market,
        bool isYes,
        uint256 amount
    ) private view returns (uint256) {
        // Simplified constant product formula
        uint256 currentYes = market.totalYesShares == 0 ? PRECISION : market.totalYesShares;
        uint256 currentNo = market.totalNoShares == 0 ? PRECISION : market.totalNoShares;

        unchecked {
            if (isYes) {
                return (amount * currentNo) / (currentYes + currentNo);
            } else {
                return (amount * currentYes) / (currentYes + currentNo);
            }
        }
    }
}
```

---

## Batch Transaction Strategies

### Multicall Pattern

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract MulticallPredictionMarket {
    /// @notice Execute multiple calls in a single transaction
    /// @param calls Array of encoded function calls
    /// @return results Array of return data from each call
    function multicall(
        bytes[] calldata calls
    ) external returns (bytes[] memory results) {
        results = new bytes[](calls.length);

        uint256 length = calls.length;
        for (uint256 i = 0; i < length;) {
            (bool success, bytes memory result) = address(this).delegatecall(calls[i]);

            if (!success) {
                // Bubble up the revert reason
                if (result.length > 0) {
                    assembly {
                        let size := mload(result)
                        revert(add(result, 32), size)
                    }
                }
                revert("Multicall failed");
            }

            results[i] = result;

            unchecked { ++i; }
        }
    }

    /// @notice Batch create multiple markets
    function batchCreateMarkets(
        uint32[] calldata endTimes,
        uint16[] calldata fees
    ) external returns (uint256[] memory marketIds) {
        uint256 length = endTimes.length;
        marketIds = new uint256[](length);

        for (uint256 i = 0; i < length;) {
            marketIds[i] = _createMarket(endTimes[i], fees[i]);
            unchecked { ++i; }
        }
    }

    /// @notice Batch place multiple bets
    function batchBuyShares(
        uint256[] calldata marketIds,
        bool[] calldata isYes,
        uint256[] calldata amounts
    ) external {
        uint256 length = marketIds.length;

        for (uint256 i = 0; i < length;) {
            _buyShares(marketIds[i], isYes[i], amounts[i]);
            unchecked { ++i; }
        }
    }
}
```

### Gas Savings Analysis

```
BATCH TRANSACTION GAS COMPARISON
═══════════════════════════════════════════════════════════════════

Scenario: User wants to place 5 bets

INDIVIDUAL TRANSACTIONS (5 separate txs):
┌─────────────────────────────────────────────────────────────────┐
│  Tx 1: Base (21,000) + Execution (50,000) = 71,000 gas          │
│  Tx 2: Base (21,000) + Execution (50,000) = 71,000 gas          │
│  Tx 3: Base (21,000) + Execution (50,000) = 71,000 gas          │
│  Tx 4: Base (21,000) + Execution (50,000) = 71,000 gas          │
│  Tx 5: Base (21,000) + Execution (50,000) = 71,000 gas          │
│  ─────────────────────────────────────────────────────────────  │
│  TOTAL: 355,000 gas                                             │
└─────────────────────────────────────────────────────────────────┘

BATCHED TRANSACTION (1 multicall tx):
┌─────────────────────────────────────────────────────────────────┐
│  Base cost: 21,000 gas                                          │
│  Multicall overhead: ~5,000 gas                                 │
│  5x Execution: 5 × 50,000 = 250,000 gas                        │
│  ─────────────────────────────────────────────────────────────  │
│  TOTAL: 276,000 gas                                             │
└─────────────────────────────────────────────────────────────────┘

SAVINGS: 79,000 gas (22% reduction)
At 30 gwei: 0.00237 ETH saved (~$7 at $3000/ETH)

For 10 transactions: ~45% savings
For 20 transactions: ~57% savings
```

### Frontend Integration with Multicall3

```typescript
import { encodeFunctionData, Address } from 'viem';
import { multicall3Abi } from './abis';

const MULTICALL3_ADDRESS = '0xcA11bde05977b3631167028862bE2a173976CA11';

interface Call {
  target: Address;
  callData: `0x${string}`;
  allowFailure?: boolean;
}

async function batchPredictions(
  marketContract: Address,
  bets: Array<{ marketId: bigint; isYes: boolean; amount: bigint }>
) {
  const calls: Call[] = bets.map((bet) => ({
    target: marketContract,
    allowFailure: false,
    callData: encodeFunctionData({
      abi: predictionMarketAbi,
      functionName: 'buyShares',
      args: [bet.marketId, bet.isYes, bet.amount],
    }),
  }));

  // Execute all bets in single transaction
  const results = await walletClient.writeContract({
    address: MULTICALL3_ADDRESS,
    abi: multicall3Abi,
    functionName: 'aggregate3',
    args: [calls],
  });

  return results;
}

// Usage
const bets = [
  { marketId: 1n, isYes: true, amount: parseEther('10') },
  { marketId: 2n, isYes: false, amount: parseEther('5') },
  { marketId: 3n, isYes: true, amount: parseEther('15') },
];

await batchPredictions(MARKET_CONTRACT, bets);
```

---

## Meta-Transactions (EIP-2771)

### Overview

```
META-TRANSACTION FLOW (EIP-2771)
═══════════════════════════════════════════════════════════════════

  ┌──────────────┐     1. Sign Message      ┌──────────────────┐
  │    User      │ ────────────────────────▶│   Off-chain      │
  │  (No ETH)    │                          │   Signature      │
  └──────────────┘                          └────────┬─────────┘
                                                     │
                                            2. Send to Relayer
                                                     │
                                                     ▼
                                            ┌──────────────────┐
                                            │    Relayer       │
                                            │  (Pays Gas)      │
                                            └────────┬─────────┘
                                                     │
                                            3. Submit TX
                                                     │
                                                     ▼
  ┌──────────────────────────────────────────────────────────────┐
  │                    Trusted Forwarder                          │
  │  • Verifies signature                                         │
  │  • Appends original sender to calldata                        │
  │  • Forwards to target contract                                │
  └────────────────────────────────┬─────────────────────────────┘
                                   │
                                   │ 4. Forward call
                                   ▼
  ┌──────────────────────────────────────────────────────────────┐
  │              Target Contract (ERC2771 Aware)                  │
  │  • Uses _msgSender() instead of msg.sender                    │
  │  • Extracts original sender from calldata                     │
  │  • Executes user's intended action                            │
  └──────────────────────────────────────────────────────────────┘
```

### ERC2771 Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/metatx/ERC2771Context.sol";

contract MetaTxPredictionMarket is ERC2771Context {
    mapping(uint256 => mapping(address => uint256)) public positions;

    constructor(address trustedForwarder)
        ERC2771Context(trustedForwarder)
    {}

    function buyShares(uint256 marketId, bool isYes, uint256 amount) external {
        // IMPORTANT: Use _msgSender() instead of msg.sender
        address user = _msgSender();

        // Now this works for both direct calls and meta-transactions
        positions[marketId][user] += amount;

        // Process the bet...
    }

    // Override required functions
    function _msgSender() internal view override returns (address sender) {
        if (isTrustedForwarder(msg.sender)) {
            // Extract sender from calldata (last 20 bytes)
            assembly {
                sender := shr(96, calldataload(sub(calldatasize(), 20)))
            }
        } else {
            return super._msgSender();
        }
    }

    function _msgData() internal view override returns (bytes calldata) {
        if (isTrustedForwarder(msg.sender)) {
            return msg.data[:msg.data.length - 20];
        } else {
            return super._msgData();
        }
    }
}
```

### Trusted Forwarder Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import "@openzeppelin/contracts/utils/cryptography/EIP712.sol";

contract TrustedForwarder is EIP712 {
    using ECDSA for bytes32;

    struct ForwardRequest {
        address from;
        address to;
        uint256 value;
        uint256 gas;
        uint256 nonce;
        bytes data;
    }

    bytes32 private constant _TYPEHASH =
        keccak256(
            "ForwardRequest(address from,address to,uint256 value,uint256 gas,uint256 nonce,bytes data)"
        );

    mapping(address => uint256) private _nonces;

    constructor() EIP712("TrustedForwarder", "1") {}

    function getNonce(address from) public view returns (uint256) {
        return _nonces[from];
    }

    function verify(
        ForwardRequest calldata req,
        bytes calldata signature
    ) public view returns (bool) {
        address signer = _hashTypedDataV4(
            keccak256(
                abi.encode(
                    _TYPEHASH,
                    req.from,
                    req.to,
                    req.value,
                    req.gas,
                    req.nonce,
                    keccak256(req.data)
                )
            )
        ).recover(signature);

        return _nonces[req.from] == req.nonce && signer == req.from;
    }

    function execute(
        ForwardRequest calldata req,
        bytes calldata signature
    ) public payable returns (bool, bytes memory) {
        require(verify(req, signature), "Invalid signature");

        _nonces[req.from]++;

        // Append the original sender to the calldata
        (bool success, bytes memory returndata) = req.to.call{
            gas: req.gas,
            value: req.value
        }(abi.encodePacked(req.data, req.from));

        // Validate gas was sufficient
        if (gasleft() <= req.gas / 63) {
            assembly {
                invalid()
            }
        }

        return (success, returndata);
    }
}
```

---

## Gas Sponsorship and Relayers

### Biconomy Integration

```typescript
// Biconomy SDK Integration for Gas Sponsorship
import { createSmartAccountClient, PaymasterMode } from "@biconomy/account";
import { createWalletClient, http, createPublicClient } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { polygon } from "viem/chains";

const BICONOMY_BUNDLER_URL = "https://bundler.biconomy.io/api/v2/137/xxx";
const BICONOMY_PAYMASTER_URL = "https://paymaster.biconomy.io/api/v1/137/xxx";

async function setupBiconomyClient(privateKey: `0x${string}`) {
  const account = privateKeyToAccount(privateKey);

  const walletClient = createWalletClient({
    account,
    chain: polygon,
    transport: http(),
  });

  // Create Biconomy Smart Account
  const smartAccount = await createSmartAccountClient({
    signer: walletClient,
    bundlerUrl: BICONOMY_BUNDLER_URL,
    paymasterUrl: BICONOMY_PAYMASTER_URL,
  });

  return smartAccount;
}

// Execute gasless transaction
async function gaslessBet(
  smartAccount: any,
  marketId: bigint,
  isYes: boolean,
  amount: bigint
) {
  const txData = encodeFunctionData({
    abi: predictionMarketAbi,
    functionName: "buyShares",
    args: [marketId, isYes, amount],
  });

  const transaction = {
    to: PREDICTION_MARKET_ADDRESS,
    data: txData,
  };

  // Send with gas sponsorship
  const userOpResponse = await smartAccount.sendTransaction(transaction, {
    paymasterServiceData: {
      mode: PaymasterMode.SPONSORED, // Gas paid by dApp
    },
  });

  const receipt = await userOpResponse.wait();
  console.log("Gasless tx hash:", receipt.transactionHash);
}
```

### Gelato Relay Integration

```typescript
// Gelato Relay for Gasless Transactions
import { GelatoRelay, SponsoredCallRequest } from "@gelatonetwork/relay-sdk";

const relay = new GelatoRelay();

async function gelatoGaslessBet(
  userAddress: string,
  marketId: bigint,
  isYes: boolean,
  amount: bigint,
  userSignature: `0x${string}`
) {
  // Encode the function call
  const data = encodeFunctionData({
    abi: predictionMarketAbi,
    functionName: "buySharesWithPermit",
    args: [marketId, isYes, amount, userSignature],
  });

  const request: SponsoredCallRequest = {
    chainId: BigInt(137), // Polygon
    target: PREDICTION_MARKET_ADDRESS,
    data: data,
  };

  // Submit to Gelato with API key for sponsorship
  const response = await relay.sponsoredCall(
    request,
    process.env.GELATO_SPONSOR_API_KEY!
  );

  console.log("Task ID:", response.taskId);

  // Poll for completion
  const status = await relay.getTaskStatus(response.taskId);
  return status;
}
```

### Gas Sponsorship Architecture

```
GAS SPONSORSHIP SYSTEM ARCHITECTURE
═══════════════════════════════════════════════════════════════════

                         ┌────────────────────────┐
                         │    Sponsor Dashboard   │
                         │  • Top up gas tank     │
                         │  • Set spending limits │
                         │  • View usage stats    │
                         └───────────┬────────────┘
                                     │
                                     ▼
┌──────────────┐           ┌────────────────────────┐
│    User      │           │    Paymaster Service   │
│  (No Gas)    │           │  ┌──────────────────┐  │
└──────┬───────┘           │  │ Policy Engine    │  │
       │                   │  │ • Whitelist      │  │
       │ 1. Sign UserOp    │  │ • Rate limits    │  │
       │                   │  │ • Max gas/tx     │  │
       ▼                   │  └──────────────────┘  │
┌──────────────┐           │                        │
│   dApp UI    │──────────▶│  ┌──────────────────┐  │
└──────────────┘           │  │   Gas Tank       │  │
       │                   │  │   (USDC/ETH)     │  │
       │ 2. Submit to      │  └──────────────────┘  │
       │    Bundler        └───────────┬────────────┘
       │                               │
       ▼                               │ 3. Verify & Sign
┌──────────────┐                       │
│   Bundler    │◀──────────────────────┘
│              │
└──────┬───────┘
       │
       │ 4. Bundle & Submit
       ▼
┌──────────────────────────────────────────────────────────────┐
│                      EntryPoint Contract                      │
│              (Executes UserOps, pays bundler)                │
└──────────────────────────────────────────────────────────────┘
```

---

## Account Abstraction (ERC-4337)

### ERC-4337 Architecture

```
ERC-4337 COMPONENT DIAGRAM
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│                        USER LAYER                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌─────────────┐                      ┌─────────────┐         │
│    │   EOA       │                      │  Smart      │         │
│    │   Signer    │────────────────────▶ │  Account    │         │
│    │   (key)     │   Controls           │  (Wallet)   │         │
│    └─────────────┘                      └─────────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ UserOperation
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      INFRASTRUCTURE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌─────────────┐      ┌─────────────┐      ┌─────────────┐   │
│    │  Alt        │      │             │      │             │   │
│    │  Mempool    │─────▶│  Bundler    │─────▶│  Paymaster  │   │
│    │             │      │             │      │  (Optional) │   │
│    └─────────────┘      └─────────────┘      └─────────────┘   │
│                                │                                 │
└────────────────────────────────┼────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                       BLOCKCHAIN                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌────────────────────────────────────────────────────────┐   │
│    │                    EntryPoint                           │   │
│    │  0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789            │   │
│    │                                                         │   │
│    │  • handleOps(UserOperation[] ops)                      │   │
│    │  • Validates signatures and nonces                     │   │
│    │  • Calls validateUserOp on Smart Account               │   │
│    │  • Executes operations                                 │   │
│    │  • Compensates bundler                                 │   │
│    └────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### UserOperation Structure

```solidity
struct UserOperation {
    address sender;              // Smart account address
    uint256 nonce;               // Anti-replay
    bytes initCode;              // Account creation code (if new)
    bytes callData;              // What to execute
    uint256 callGasLimit;        // Gas for main execution
    uint256 verificationGasLimit;// Gas for validation
    uint256 preVerificationGas;  // Gas for bundler overhead
    uint256 maxFeePerGas;        // Max gas price
    uint256 maxPriorityFeePerGas;// Priority fee
    bytes paymasterAndData;      // Paymaster address + data
    bytes signature;             // User's signature
}
```

### Smart Account Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@account-abstraction/contracts/core/BaseAccount.sol";
import "@account-abstraction/contracts/interfaces/IEntryPoint.sol";
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract PredictionMarketAccount is BaseAccount {
    using ECDSA for bytes32;

    IEntryPoint private immutable _entryPoint;
    address public owner;

    // Prediction market specific storage
    mapping(uint256 => uint256) public marketPositions;

    event PredictionMarketAccountInitialized(
        IEntryPoint indexed entryPoint,
        address indexed owner
    );

    modifier onlyOwner() {
        require(msg.sender == owner || msg.sender == address(this), "Not owner");
        _;
    }

    constructor(IEntryPoint anEntryPoint) {
        _entryPoint = anEntryPoint;
    }

    function initialize(address anOwner) public {
        require(owner == address(0), "Already initialized");
        owner = anOwner;
        emit PredictionMarketAccountInitialized(_entryPoint, owner);
    }

    function entryPoint() public view override returns (IEntryPoint) {
        return _entryPoint;
    }

    /// @notice Validate the signature of a UserOperation
    function _validateSignature(
        UserOperation calldata userOp,
        bytes32 userOpHash
    ) internal view override returns (uint256 validationData) {
        bytes32 hash = userOpHash.toEthSignedMessageHash();
        address recovered = hash.recover(userOp.signature);

        if (recovered != owner) {
            return SIG_VALIDATION_FAILED;
        }
        return 0; // Valid
    }

    /// @notice Execute a transaction (called by EntryPoint)
    function execute(
        address dest,
        uint256 value,
        bytes calldata func
    ) external onlyOwner {
        (bool success, bytes memory result) = dest.call{value: value}(func);
        if (!success) {
            assembly {
                revert(add(result, 32), mload(result))
            }
        }
    }

    /// @notice Execute a batch of transactions
    function executeBatch(
        address[] calldata dests,
        uint256[] calldata values,
        bytes[] calldata funcs
    ) external onlyOwner {
        require(
            dests.length == values.length && values.length == funcs.length,
            "Length mismatch"
        );

        for (uint256 i = 0; i < dests.length; i++) {
            (bool success, bytes memory result) = dests[i].call{value: values[i]}(
                funcs[i]
            );
            if (!success) {
                assembly {
                    revert(add(result, 32), mload(result))
                }
            }
        }
    }

    /// @notice Deposit to EntryPoint for gas
    function addDeposit() public payable {
        entryPoint().depositTo{value: msg.value}(address(this));
    }

    /// @notice Get current deposit balance
    function getDeposit() public view returns (uint256) {
        return entryPoint().balanceOf(address(this));
    }

    receive() external payable {}
}
```

### Paymaster for ERC-20 Gas Payment

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@account-abstraction/contracts/core/BasePaymaster.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract TokenPaymaster is BasePaymaster {
    IERC20 public immutable token;
    uint256 public constant COST_OF_POST = 35000; // Gas for postOp

    // Price oracle or fixed rate
    uint256 public tokenPricePerGas = 1e15; // Example: 0.001 token per gas

    constructor(
        IEntryPoint _entryPoint,
        IERC20 _token
    ) BasePaymaster(_entryPoint) {
        token = _token;
    }

    /// @notice Validate paymaster can cover the operation
    function _validatePaymasterUserOp(
        UserOperation calldata userOp,
        bytes32 userOpHash,
        uint256 maxCost
    ) internal override returns (bytes memory context, uint256 validationData) {
        // Calculate token cost
        uint256 tokenCost = (maxCost + COST_OF_POST) * tokenPricePerGas / 1e18;

        // Check user has approved enough tokens
        require(
            token.allowance(userOp.sender, address(this)) >= tokenCost,
            "Insufficient token allowance"
        );

        // Return context for postOp
        return (abi.encode(userOp.sender, tokenCost), 0);
    }

    /// @notice Called after operation execution
    function _postOp(
        PostOpMode mode,
        bytes calldata context,
        uint256 actualGasCost
    ) internal override {
        if (mode == PostOpMode.postOpReverted) {
            return; // Don't charge if reverted
        }

        (address sender, uint256 maxTokenCost) = abi.decode(
            context,
            (address, uint256)
        );

        // Calculate actual token cost
        uint256 actualTokenCost = actualGasCost * tokenPricePerGas / 1e18;

        // Transfer tokens from user
        token.transferFrom(sender, address(this), actualTokenCost);
    }
}
```

---

## Gasless Transactions for Users

### Complete Gasless Flow Implementation

```typescript
// Full gasless transaction implementation
import {
  createSmartAccountClient,
  BiconomySmartAccountV2,
  PaymasterMode,
} from "@biconomy/account";
import { encodeFunctionData, parseEther } from "viem";

class GaslessTransactionService {
  private smartAccount: BiconomySmartAccountV2 | null = null;

  async initialize(signer: any) {
    this.smartAccount = await createSmartAccountClient({
      signer,
      bundlerUrl: process.env.BUNDLER_URL!,
      paymasterUrl: process.env.PAYMASTER_URL!,
      chainId: 137, // Polygon
    });

    return this.smartAccount.getAccountAddress();
  }

  // Fully sponsored transaction (dApp pays)
  async sponsoredBet(
    marketId: bigint,
    isYes: boolean,
    amount: bigint
  ): Promise<string> {
    if (!this.smartAccount) throw new Error("Not initialized");

    const tx = {
      to: PREDICTION_MARKET_ADDRESS,
      data: encodeFunctionData({
        abi: predictionMarketAbi,
        functionName: "buyShares",
        args: [marketId, isYes, amount],
      }),
    };

    const response = await this.smartAccount.sendTransaction(tx, {
      paymasterServiceData: {
        mode: PaymasterMode.SPONSORED,
      },
    });

    const receipt = await response.wait();
    return receipt.transactionHash;
  }

  // User pays in ERC-20 token
  async tokenPaidBet(
    marketId: bigint,
    isYes: boolean,
    amount: bigint,
    paymentToken: `0x${string}`
  ): Promise<string> {
    if (!this.smartAccount) throw new Error("Not initialized");

    // First approve token spending (if needed)
    const approveTx = {
      to: paymentToken,
      data: encodeFunctionData({
        abi: erc20Abi,
        functionName: "approve",
        args: [PAYMASTER_ADDRESS, parseEther("1000")],
      }),
    };

    // Then the actual bet
    const betTx = {
      to: PREDICTION_MARKET_ADDRESS,
      data: encodeFunctionData({
        abi: predictionMarketAbi,
        functionName: "buyShares",
        args: [marketId, isYes, amount],
      }),
    };

    // Batch both transactions
    const response = await this.smartAccount.sendTransaction(
      [approveTx, betTx],
      {
        paymasterServiceData: {
          mode: PaymasterMode.ERC20,
          preferredToken: paymentToken,
        },
      }
    );

    const receipt = await response.wait();
    return receipt.transactionHash;
  }

  // Hybrid: Sponsored up to limit, then user pays
  async hybridBet(
    marketId: bigint,
    isYes: boolean,
    amount: bigint
  ): Promise<string> {
    if (!this.smartAccount) throw new Error("Not initialized");

    const tx = {
      to: PREDICTION_MARKET_ADDRESS,
      data: encodeFunctionData({
        abi: predictionMarketAbi,
        functionName: "buyShares",
        args: [marketId, isYes, amount],
      }),
    };

    // Try sponsored first
    try {
      const response = await this.smartAccount.sendTransaction(tx, {
        paymasterServiceData: {
          mode: PaymasterMode.SPONSORED,
        },
      });
      const receipt = await response.wait();
      return receipt.transactionHash;
    } catch (e) {
      // Fall back to user-paid if sponsorship limit exceeded
      console.log("Sponsorship limit reached, using user balance");

      const response = await this.smartAccount.sendTransaction(tx);
      const receipt = await response.wait();
      return receipt.transactionHash;
    }
  }
}
```

### Sponsorship Policy Configuration

```typescript
// Biconomy Dashboard Policy Configuration
interface SponsorshipPolicy {
  name: string;
  chainId: number;
  rules: PolicyRule[];
}

interface PolicyRule {
  type: "contract" | "method" | "wallet" | "rate_limit";
  condition: {
    // Contract whitelist
    addresses?: string[];
    // Method whitelist
    methods?: string[];
    // Wallet whitelist/blacklist
    wallets?: { include?: string[]; exclude?: string[] };
    // Rate limiting
    limit?: {
      maxTxPerDay?: number;
      maxGasPerDay?: string;
      maxTxPerHour?: number;
    };
  };
}

const predictionMarketPolicy: SponsorshipPolicy = {
  name: "Prediction Market Sponsorship",
  chainId: 137,
  rules: [
    {
      type: "contract",
      condition: {
        addresses: [PREDICTION_MARKET_ADDRESS, COLLATERAL_TOKEN_ADDRESS],
      },
    },
    {
      type: "method",
      condition: {
        methods: [
          "buyShares",
          "sellShares",
          "claimWinnings",
          "approve", // For token approval
        ],
      },
    },
    {
      type: "rate_limit",
      condition: {
        limit: {
          maxTxPerDay: 50,
          maxGasPerDay: "5000000", // 5M gas units
          maxTxPerHour: 10,
        },
      },
    },
  ],
};
```

---

## L2 Gas Pricing Differences

### Chain Comparison Matrix

```
L2 GAS PRICING COMPARISON (Post-Dencun 2024)
═══════════════════════════════════════════════════════════════════

Chain          │ Avg Gas  │ Avg Tx    │ Finality  │ TPS    │ Notes
               │ Price    │ Cost      │           │        │
───────────────┼──────────┼───────────┼───────────┼────────┼────────────
Polygon PoS    │ 30 gwei  │ $0.001-   │ ~2 sec    │ ~1000  │ Cheap,fast
               │          │ $0.01     │           │        │ MATIC gas
───────────────┼──────────┼───────────┼───────────┼────────┼────────────
Polygon zkEVM  │ 0.025    │ $0.01-    │ ~30 min*  │ ~40    │ ZK proofs
               │ gwei     │ $0.05     │           │        │ ETH gas
───────────────┼──────────┼───────────┼───────────┼────────┼────────────
Arbitrum One   │ 0.05     │ $0.005-   │ ~1 week** │ ~40    │ Lowest L2
               │ gwei     │ $0.02     │           │        │ fees
───────────────┼──────────┼───────────┼───────────┼────────┼────────────
Optimism       │ 0.12     │ $0.01-    │ ~1 week** │ ~130   │ OP Stack
               │ gwei     │ $0.03     │           │        │
───────────────┼──────────┼───────────┼───────────┼────────┼────────────
Base           │ 0.01     │ $0.001-   │ ~1 week** │ ~100   │ Coinbase
               │ gwei     │ $0.03     │           │        │ backed
───────────────┼──────────┼───────────┼───────────┼────────┼────────────
zkSync Era     │ 0.25     │ $0.02-    │ ~1 hour   │ ~15    │ Native AA
               │ gwei     │ $0.10     │           │        │
───────────────┼──────────┼───────────┼───────────┼────────┼────────────
Ethereum L1    │ 20-50    │ $1-$50    │ ~15 min   │ ~15    │ Most secure
               │ gwei     │           │           │        │

* ZK proof generation time, soft finality faster
** Challenge period for fraud proofs, soft finality ~15 min
```

### Gas Calculation Formulas

```typescript
// L2 Gas Cost Calculation

// Polygon PoS
function polygonGasCost(
  gasUsed: bigint,
  gasPriceGwei: bigint,
  maticPriceUsd: number
): number {
  const gasCostMatic = Number(gasUsed * gasPriceGwei) / 1e9;
  return gasCostMatic * maticPriceUsd;
}

// Optimistic Rollups (Arbitrum, Optimism, Base)
// Cost = L2 Execution + L1 Data
function optimisticRollupCost(
  l2GasUsed: bigint,
  l2GasPrice: bigint,
  callDataBytes: number,
  l1GasPrice: bigint, // Current L1 gas price
  ethPriceUsd: number
): number {
  // L2 execution cost
  const l2Cost = Number(l2GasUsed * l2GasPrice) / 1e18;

  // L1 data cost (compressed calldata posted to L1)
  // ~16 gas per non-zero byte, ~4 gas per zero byte
  const avgByteCost = 12n; // Approximate
  const l1DataGas = BigInt(callDataBytes) * avgByteCost;
  const l1Cost = Number(l1DataGas * l1GasPrice) / 1e18;

  return (l2Cost + l1Cost) * ethPriceUsd;
}

// ZK Rollups (zkSync, Polygon zkEVM)
function zkRollupCost(
  gasUsed: bigint,
  gasPrice: bigint,
  ethPriceUsd: number
): number {
  // ZK rollups have simpler pricing - proof cost amortized
  const gasCostEth = Number(gasUsed * gasPrice) / 1e18;
  return gasCostEth * ethPriceUsd;
}
```

### Cost Projection Tool

```typescript
class GasCostEstimator {
  private prices = {
    eth: 3000,
    matic: 0.5,
  };

  private gasSettings = {
    ethereum: { gasPrice: 30n * 10n ** 9n }, // 30 gwei
    polygon: { gasPrice: 30n * 10n ** 9n }, // 30 gwei MATIC
    arbitrum: { gasPrice: 1n * 10n ** 8n }, // 0.1 gwei
    optimism: { gasPrice: 1n * 10n ** 8n }, // 0.1 gwei
    base: { gasPrice: 1n * 10n ** 7n }, // 0.01 gwei
  };

  estimateCosts(operation: string, gasUsed: bigint): ChainCosts {
    return {
      ethereum: this.calculateEthCost(gasUsed, this.gasSettings.ethereum),
      polygon: this.calculatePolygonCost(gasUsed, this.gasSettings.polygon),
      arbitrum: this.calculateL2Cost(gasUsed, this.gasSettings.arbitrum),
      optimism: this.calculateL2Cost(gasUsed, this.gasSettings.optimism),
      base: this.calculateL2Cost(gasUsed, this.gasSettings.base),
    };
  }

  // Example output for a typical bet transaction (100k gas):
  /*
    {
      ethereum: "$9.00",
      polygon: "$0.0015",
      arbitrum: "$0.03",
      optimism: "$0.03",
      base: "$0.003"
    }
  */
}
```

---

## Gas Estimation Strategies

### Accurate Gas Estimation

```typescript
import { createPublicClient, http, encodeFunctionData } from "viem";
import { polygon } from "viem/chains";

const client = createPublicClient({
  chain: polygon,
  transport: http(),
});

async function estimateGasWithBuffer(
  to: `0x${string}`,
  data: `0x${string}`,
  from: `0x${string}`,
  bufferPercent: number = 20
): Promise<{ gasLimit: bigint; estimatedCost: bigint }> {
  // Get base estimation
  const gasEstimate = await client.estimateGas({
    to,
    data,
    account: from,
  });

  // Add buffer for safety
  const gasLimit = (gasEstimate * BigInt(100 + bufferPercent)) / 100n;

  // Get current gas price
  const gasPrice = await client.getGasPrice();

  return {
    gasLimit,
    estimatedCost: gasLimit * gasPrice,
  };
}

// Batch estimation for multiple transactions
async function estimateBatchGas(
  transactions: Array<{ to: `0x${string}`; data: `0x${string}` }>,
  from: `0x${string}`
): Promise<{
  individual: bigint[];
  total: bigint;
  batchSavings: bigint;
}> {
  // Estimate each individually
  const individualEstimates = await Promise.all(
    transactions.map((tx) =>
      client.estimateGas({
        to: tx.to,
        data: tx.data,
        account: from,
      })
    )
  );

  // Estimate as batch (multicall)
  const multicallData = encodeMulticall(transactions);
  const batchEstimate = await client.estimateGas({
    to: MULTICALL3_ADDRESS,
    data: multicallData,
    account: from,
  });

  const individualTotal = individualEstimates.reduce((a, b) => a + b, 0n);

  return {
    individual: individualEstimates,
    total: batchEstimate,
    batchSavings: individualTotal - batchEstimate,
  };
}
```

### Dynamic Gas Price Strategy

```typescript
interface GasPriceStrategy {
  maxFeePerGas: bigint;
  maxPriorityFeePerGas: bigint;
  estimatedTime: string;
}

async function getGasPriceStrategies(): Promise<{
  slow: GasPriceStrategy;
  standard: GasPriceStrategy;
  fast: GasPriceStrategy;
}> {
  const feeHistory = await client.getFeeHistory({
    blockCount: 10,
    rewardPercentiles: [10, 50, 90],
  });

  const baseFee = feeHistory.baseFeePerGas[feeHistory.baseFeePerGas.length - 1];

  // Calculate priority fees from history
  const rewards = feeHistory.reward!;
  const avgSlow =
    rewards.reduce((a, r) => a + r[0], 0n) / BigInt(rewards.length);
  const avgStandard =
    rewards.reduce((a, r) => a + r[1], 0n) / BigInt(rewards.length);
  const avgFast =
    rewards.reduce((a, r) => a + r[2], 0n) / BigInt(rewards.length);

  return {
    slow: {
      maxFeePerGas: baseFee + avgSlow,
      maxPriorityFeePerGas: avgSlow,
      estimatedTime: "~5 minutes",
    },
    standard: {
      maxFeePerGas: baseFee + avgStandard,
      maxPriorityFeePerGas: avgStandard,
      estimatedTime: "~30 seconds",
    },
    fast: {
      maxFeePerGas: (baseFee * 2n) + avgFast,
      maxPriorityFeePerGas: avgFast * 2n,
      estimatedTime: "~10 seconds",
    },
  };
}
```

---

## Transaction Queuing

### Queue Architecture

```
TRANSACTION QUEUE ARCHITECTURE
═══════════════════════════════════════════════════════════════════

┌────────────────────────────────────────────────────────────────┐
│                     USER INTERFACE                              │
└───────────────────────────┬────────────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────────────┐
│                   TRANSACTION MANAGER                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  Pending     │  │  Submitted   │  │  Confirmed   │         │
│  │  Queue       │─▶│  Queue       │─▶│  Queue       │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│         │                 │                 │                  │
│         │                 │                 │                  │
│         ▼                 ▼                 ▼                  │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                   STATE MACHINE                           │ │
│  │  PENDING → SUBMITTED → CONFIRMING → CONFIRMED/FAILED     │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────────────┐
│                    BLOCKCHAIN                                   │
└────────────────────────────────────────────────────────────────┘
```

### Implementation

```typescript
import { EventEmitter } from "events";

enum TxStatus {
  PENDING = "pending",
  SUBMITTED = "submitted",
  CONFIRMING = "confirming",
  CONFIRMED = "confirmed",
  FAILED = "failed",
  REPLACED = "replaced",
}

interface QueuedTransaction {
  id: string;
  status: TxStatus;
  to: `0x${string}`;
  data: `0x${string}`;
  value?: bigint;
  nonce?: number;
  hash?: `0x${string}`;
  gasLimit?: bigint;
  maxFeePerGas?: bigint;
  maxPriorityFeePerGas?: bigint;
  attempts: number;
  createdAt: Date;
  submittedAt?: Date;
  confirmedAt?: Date;
  error?: string;
}

class TransactionQueue extends EventEmitter {
  private queue: Map<string, QueuedTransaction> = new Map();
  private processing = false;
  private currentNonce: number | null = null;

  constructor(
    private walletClient: any,
    private publicClient: any,
    private maxRetries: number = 3
  ) {
    super();
  }

  async add(tx: Omit<QueuedTransaction, "id" | "status" | "attempts" | "createdAt">): Promise<string> {
    const id = crypto.randomUUID();
    const queuedTx: QueuedTransaction = {
      ...tx,
      id,
      status: TxStatus.PENDING,
      attempts: 0,
      createdAt: new Date(),
    };

    this.queue.set(id, queuedTx);
    this.emit("added", queuedTx);

    // Start processing if not already
    if (!this.processing) {
      this.processQueue();
    }

    return id;
  }

  private async processQueue() {
    if (this.processing) return;
    this.processing = true;

    while (this.hasPendingTransactions()) {
      const tx = this.getNextPending();
      if (!tx) break;

      try {
        await this.submitTransaction(tx);
      } catch (error: any) {
        await this.handleError(tx, error);
      }
    }

    this.processing = false;
  }

  private async submitTransaction(tx: QueuedTransaction) {
    tx.status = TxStatus.SUBMITTED;
    tx.attempts++;
    tx.submittedAt = new Date();
    this.emit("submitted", tx);

    // Get nonce if not set
    if (tx.nonce === undefined) {
      tx.nonce = await this.getNextNonce();
    }

    // Estimate gas if not set
    if (!tx.gasLimit) {
      tx.gasLimit = await this.publicClient.estimateGas({
        to: tx.to,
        data: tx.data,
        value: tx.value,
      });
      tx.gasLimit = (tx.gasLimit * 120n) / 100n; // 20% buffer
    }

    // Get gas price if not set
    if (!tx.maxFeePerGas) {
      const gasPrice = await this.publicClient.getGasPrice();
      tx.maxFeePerGas = (gasPrice * 110n) / 100n;
      tx.maxPriorityFeePerGas = gasPrice / 10n;
    }

    // Submit transaction
    const hash = await this.walletClient.sendTransaction({
      to: tx.to,
      data: tx.data,
      value: tx.value,
      nonce: tx.nonce,
      gas: tx.gasLimit,
      maxFeePerGas: tx.maxFeePerGas,
      maxPriorityFeePerGas: tx.maxPriorityFeePerGas,
    });

    tx.hash = hash;
    tx.status = TxStatus.CONFIRMING;
    this.emit("confirming", tx);

    // Wait for confirmation
    const receipt = await this.publicClient.waitForTransactionReceipt({
      hash,
      confirmations: 1,
    });

    if (receipt.status === "success") {
      tx.status = TxStatus.CONFIRMED;
      tx.confirmedAt = new Date();
      this.emit("confirmed", tx);
    } else {
      throw new Error("Transaction reverted");
    }
  }

  private async handleError(tx: QueuedTransaction, error: any) {
    console.error(`Transaction ${tx.id} failed:`, error.message);

    // Check if we should retry
    if (tx.attempts < this.maxRetries) {
      // Check error type for appropriate action
      if (error.message.includes("nonce too low")) {
        // Reset nonce and retry
        this.currentNonce = null;
        tx.nonce = undefined;
        tx.status = TxStatus.PENDING;
        return;
      }

      if (error.message.includes("underpriced")) {
        // Increase gas price and retry
        tx.maxFeePerGas = tx.maxFeePerGas
          ? (tx.maxFeePerGas * 130n) / 100n
          : undefined;
        tx.status = TxStatus.PENDING;
        return;
      }

      // Generic retry
      tx.status = TxStatus.PENDING;
      return;
    }

    // Max retries exceeded
    tx.status = TxStatus.FAILED;
    tx.error = error.message;
    this.emit("failed", tx);
  }

  private async getNextNonce(): Promise<number> {
    if (this.currentNonce === null) {
      this.currentNonce = await this.publicClient.getTransactionCount({
        address: this.walletClient.account.address,
        blockTag: "pending",
      });
    } else {
      this.currentNonce++;
    }
    return this.currentNonce;
  }

  private hasPendingTransactions(): boolean {
    return [...this.queue.values()].some(
      (tx) => tx.status === TxStatus.PENDING
    );
  }

  private getNextPending(): QueuedTransaction | undefined {
    return [...this.queue.values()]
      .filter((tx) => tx.status === TxStatus.PENDING)
      .sort((a, b) => a.createdAt.getTime() - b.createdAt.getTime())[0];
  }

  getStatus(id: string): QueuedTransaction | undefined {
    return this.queue.get(id);
  }

  getAllTransactions(): QueuedTransaction[] {
    return [...this.queue.values()];
  }
}
```

---

## Failed Transaction Handling

### Error Recovery Strategies

```typescript
interface TransactionError {
  code: string;
  message: string;
  recoverable: boolean;
  action: "retry" | "increase_gas" | "reset_nonce" | "cancel" | "manual";
}

const ERROR_HANDLERS: Record<string, TransactionError> = {
  NONCE_TOO_LOW: {
    code: "NONCE_TOO_LOW",
    message: "Nonce too low - another transaction was confirmed",
    recoverable: true,
    action: "reset_nonce",
  },
  INSUFFICIENT_FUNDS: {
    code: "INSUFFICIENT_FUNDS",
    message: "Not enough ETH/MATIC for gas",
    recoverable: false,
    action: "manual",
  },
  REPLACEMENT_UNDERPRICED: {
    code: "REPLACEMENT_UNDERPRICED",
    message: "Gas price too low to replace pending transaction",
    recoverable: true,
    action: "increase_gas",
  },
  TRANSACTION_REVERTED: {
    code: "TRANSACTION_REVERTED",
    message: "Contract execution failed",
    recoverable: false,
    action: "manual",
  },
  TIMEOUT: {
    code: "TIMEOUT",
    message: "Transaction not mined in time",
    recoverable: true,
    action: "increase_gas",
  },
};

class TransactionRecovery {
  async handleFailedTransaction(
    tx: QueuedTransaction,
    error: Error
  ): Promise<RecoveryAction> {
    const errorType = this.classifyError(error);
    const handler = ERROR_HANDLERS[errorType];

    if (!handler) {
      return { action: "manual", reason: error.message };
    }

    switch (handler.action) {
      case "retry":
        return { action: "retry", delay: 1000 };

      case "reset_nonce":
        const newNonce = await this.publicClient.getTransactionCount({
          address: tx.to,
          blockTag: "pending",
        });
        return { action: "retry", newNonce };

      case "increase_gas":
        const currentGasPrice = await this.publicClient.getGasPrice();
        return {
          action: "retry",
          newGasPrice: (currentGasPrice * 150n) / 100n, // +50%
        };

      case "cancel":
        return {
          action: "cancel",
          cancelTx: await this.createCancelTransaction(tx),
        };

      default:
        return { action: "manual", reason: handler.message };
    }
  }

  private classifyError(error: Error): string {
    const message = error.message.toLowerCase();

    if (message.includes("nonce too low")) return "NONCE_TOO_LOW";
    if (message.includes("insufficient funds")) return "INSUFFICIENT_FUNDS";
    if (message.includes("underpriced")) return "REPLACEMENT_UNDERPRICED";
    if (message.includes("reverted")) return "TRANSACTION_REVERTED";
    if (message.includes("timeout")) return "TIMEOUT";

    return "UNKNOWN";
  }

  private async createCancelTransaction(tx: QueuedTransaction) {
    // Send 0 ETH to self with same nonce but higher gas
    return {
      to: tx.to,
      value: 0n,
      nonce: tx.nonce,
      maxFeePerGas: tx.maxFeePerGas ? tx.maxFeePerGas * 2n : undefined,
    };
  }
}
```

### Stuck Transaction Resolution

```typescript
class StuckTransactionResolver {
  private pendingTxs: Map<string, { hash: string; submittedAt: Date }> =
    new Map();

  async monitorAndResolve(
    maxPendingTime: number = 5 * 60 * 1000 // 5 minutes
  ) {
    for (const [address, txInfo] of this.pendingTxs) {
      const elapsed = Date.now() - txInfo.submittedAt.getTime();

      if (elapsed > maxPendingTime) {
        console.log(`Transaction ${txInfo.hash} stuck for ${elapsed}ms`);

        // Check if still pending
        const receipt = await this.publicClient.getTransactionReceipt({
          hash: txInfo.hash as `0x${string}`,
        });

        if (!receipt) {
          // Still pending - try to speed up
          await this.speedUpTransaction(address, txInfo.hash);
        } else {
          // Confirmed - remove from pending
          this.pendingTxs.delete(address);
        }
      }
    }
  }

  private async speedUpTransaction(address: string, originalHash: string) {
    // Get original transaction
    const originalTx = await this.publicClient.getTransaction({
      hash: originalHash as `0x${string}`,
    });

    if (!originalTx) return;

    // Create replacement with higher gas
    const currentGasPrice = await this.publicClient.getGasPrice();
    const newGasPrice = currentGasPrice > originalTx.gasPrice!
      ? (currentGasPrice * 110n) / 100n
      : (originalTx.gasPrice! * 150n) / 100n;

    const newHash = await this.walletClient.sendTransaction({
      to: originalTx.to,
      data: originalTx.input,
      value: originalTx.value,
      nonce: originalTx.nonce,
      gas: originalTx.gas,
      maxFeePerGas: newGasPrice,
      maxPriorityFeePerGas: newGasPrice / 10n,
    });

    console.log(`Replacement transaction sent: ${newHash}`);
    this.pendingTxs.set(address, {
      hash: newHash,
      submittedAt: new Date(),
    });
  }
}
```

---

## Cost Analysis and Projections

### Monthly Cost Model

```
PREDICTED MONTHLY GAS COSTS
═══════════════════════════════════════════════════════════════════

Assumptions:
- 100,000 daily active users
- Average 5 transactions per user per day
- Average gas per transaction: 150,000

                           Daily Txs: 500,000
                           Monthly Txs: 15,000,000

SCENARIO 1: Ethereum Mainnet
┌─────────────────────────────────────────────────────────────────┐
│  Gas Price: 30 gwei (average)                                   │
│  ETH Price: $3,000                                              │
│                                                                  │
│  Cost per tx: 150,000 × 30 gwei = 0.0045 ETH = $13.50          │
│  Daily cost: 500,000 × $13.50 = $6,750,000                      │
│  Monthly cost: $202,500,000                                     │
│                                                                  │
│  VERDICT: NOT VIABLE ❌                                         │
└─────────────────────────────────────────────────────────────────┘

SCENARIO 2: Polygon PoS
┌─────────────────────────────────────────────────────────────────┐
│  Gas Price: 30 gwei                                             │
│  MATIC Price: $0.50                                             │
│                                                                  │
│  Cost per tx: 150,000 × 30 gwei = 0.0045 MATIC = $0.00225      │
│  Daily cost: 500,000 × $0.00225 = $1,125                        │
│  Monthly cost: $33,750                                          │
│                                                                  │
│  VERDICT: EXCELLENT ✅                                          │
└─────────────────────────────────────────────────────────────────┘

SCENARIO 3: Arbitrum One
┌─────────────────────────────────────────────────────────────────┐
│  Gas Price: 0.05 gwei                                           │
│  ETH Price: $3,000                                              │
│                                                                  │
│  Cost per tx: 150,000 × 0.05 gwei = 0.0000075 ETH = $0.0225    │
│  Daily cost: 500,000 × $0.0225 = $11,250                        │
│  Monthly cost: $337,500                                         │
│                                                                  │
│  VERDICT: GOOD ✅                                               │
└─────────────────────────────────────────────────────────────────┘

SCENARIO 4: Base
┌─────────────────────────────────────────────────────────────────┐
│  Gas Price: 0.01 gwei                                           │
│  ETH Price: $3,000                                              │
│                                                                  │
│  Cost per tx: 150,000 × 0.01 gwei = 0.0000015 ETH = $0.0045    │
│  Daily cost: 500,000 × $0.0045 = $2,250                         │
│  Monthly cost: $67,500                                          │
│                                                                  │
│  VERDICT: EXCELLENT ✅                                          │
└─────────────────────────────────────────────────────────────────┘
```

### Gas Sponsorship Budget Planning

```
SPONSORSHIP BUDGET MODEL
═══════════════════════════════════════════════════════════════════

User Tier System:
┌─────────────────────────────────────────────────────────────────┐
│  NEW USERS (First 30 days)                                      │
│  • Full gas sponsorship                                         │
│  • Up to 50 transactions                                        │
│  • Max $0.50 per transaction                                    │
│  Budget per user: $25                                           │
├─────────────────────────────────────────────────────────────────┤
│  ACTIVE USERS (Trading volume < $1000/month)                    │
│  • 50% gas sponsorship                                          │
│  • Up to 100 transactions                                       │
│  Budget per user: $5/month                                      │
├─────────────────────────────────────────────────────────────────┤
│  POWER USERS (Trading volume > $1000/month)                     │
│  • 25% gas sponsorship                                          │
│  • Unlimited transactions                                       │
│  Budget per user: $10/month                                     │
├─────────────────────────────────────────────────────────────────┤
│  WHALES (Trading volume > $10000/month)                         │
│  • Full gas sponsorship (cost of acquisition)                   │
│  Budget per user: $50/month                                     │
└─────────────────────────────────────────────────────────────────┘

Monthly Budget Projection (100K users on Polygon):
┌─────────────────────────────────────────────────────────────────┐
│  New users (10%):      10,000 × $25 = $250,000                 │
│  Active users (60%):   60,000 × $5  = $300,000                 │
│  Power users (25%):    25,000 × $10 = $250,000                 │
│  Whales (5%):           5,000 × $50 = $250,000                 │
│  ─────────────────────────────────────────────────────────────  │
│  TOTAL MONTHLY BUDGET: $1,050,000                               │
│                                                                  │
│  Revenue offset (if avg fee = 0.5% on $500M volume):           │
│  $2,500,000 in fees                                             │
│                                                                  │
│  NET AFTER SPONSORSHIP: $1,450,000 profit                       │
└─────────────────────────────────────────────────────────────────┘
```

### ROI Analysis

```
GAS OPTIMIZATION ROI
═══════════════════════════════════════════════════════════════════

OPTIMIZATION                  │ IMPLEMENTATION │ ANNUAL    │ ROI
                              │ COST           │ SAVINGS   │
──────────────────────────────┼────────────────┼───────────┼────────
Storage packing               │ $20,000        │ $500,000  │ 2400%
Batch transactions            │ $15,000        │ $300,000  │ 1900%
L2 migration (from L1)        │ $100,000       │ $2,000,000│ 1900%
Account abstraction           │ $50,000        │ $400,000  │ 700%
Gas sponsorship system        │ $80,000        │ $200,000* │ 150%
──────────────────────────────┼────────────────┼───────────┼────────
TOTAL                         │ $265,000       │ $3,400,000│ 1183%

* Gas sponsorship "savings" measured as increased user retention value
```

---

## References

- [Solidity Gas Optimization - Alchemy](https://www.alchemy.com/overviews/solidity-gas-optimization)
- [RareSkills Gas Optimization Guide](https://rareskills.io/post/gas-optimization)
- [ERC-4337 Documentation](https://docs.erc4337.io/)
- [Biconomy Documentation](https://docs.biconomy.io/)
- [Gelato Relay](https://docs.gelato.network/relay)
- [Polygon Gas Optimization](https://www.rapidinnovation.io/post/mastering-gas-efficiency-tips-and-tricks-for-polygon-smart-contracts)
- [L2 Gas Fee Statistics](https://coinlaw.io/gas-fee-markets-on-layer-2-statistics/)
- [Multicall3](https://www.multicall3.com/)
