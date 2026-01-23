# Technical Architecture Documentation

## Polymarket-Like Prediction Market Platform Blueprint

This document provides a comprehensive technical architecture for building a decentralized prediction market platform similar to Polymarket. It serves as a blueprint for developers implementing the system.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Blockchain Layer](#2-blockchain-layer)
3. [Hybrid Trading Architecture](#3-hybrid-trading-architecture)
4. [Smart Contract Architecture](#4-smart-contract-architecture)
5. [Oracle System](#5-oracle-system)
6. [Wallet Architecture](#6-wallet-architecture)
7. [Settlement System](#7-settlement-system)
8. [Frontend Architecture](#8-frontend-architecture)
9. [System Diagrams](#9-system-diagrams)
10. [Technology Stack](#10-technology-stack)
11. [Security Considerations](#11-security-considerations)
12. [Scalability Patterns](#12-scalability-patterns)

---

## 1. Architecture Overview

### High-Level System Design

The platform follows a **hybrid architecture** combining off-chain efficiency with on-chain security:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                    │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │   Web Client    │  │  Mobile Client  │  │   API Clients   │             │
│  │   (Next.js)     │  │  (React Native) │  │   (REST/WS)     │             │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
└───────────┼────────────────────┼────────────────────┼───────────────────────┘
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           APPLICATION LAYER                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    Off-Chain Services                                │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│  │  │ Order Book   │  │  Matching    │  │   Market     │              │   │
│  │  │   Engine     │  │   Engine     │  │   Service    │              │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           BLOCKCHAIN LAYER                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    Polygon Network (L2)                              │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│  │  │ CTF Exchange │  │  NegRisk     │  │  UMA Oracle  │              │   │
│  │  │  Contract    │  │  Exchange    │  │   Adapter    │              │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Core Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Decentralization** | On-chain settlement, decentralized oracles |
| **Efficiency** | Off-chain order matching, L2 scaling |
| **Security** | Non-custodial wallets, cryptographic signatures |
| **Transparency** | All settlements verifiable on-chain |
| **Scalability** | Horizontal scaling of off-chain components |

---

## 2. Blockchain Layer

### 2.1 Network Selection: Polygon (Ethereum L2)

**Why Polygon?**

| Feature | Benefit |
|---------|---------|
| **Proof-of-Stake Consensus** | Energy efficient, fast finality |
| **EVM Compatibility** | Reuse Ethereum tooling and contracts |
| **Low Transaction Fees** | ~$0.001-0.01 per transaction |
| **Fast Block Times** | ~2 second block time |
| **Ethereum Security** | Inherits security through checkpoints |

### 2.2 Network Configuration

```javascript
// network-config.js
const POLYGON_MAINNET = {
  chainId: 137,
  chainIdHex: '0x89',
  name: 'Polygon Mainnet',
  rpcUrls: [
    'https://polygon-rpc.com',
    'https://rpc-mainnet.matic.network',
    'https://rpc-mainnet.maticvigil.com'
  ],
  blockExplorer: 'https://polygonscan.com',
  nativeCurrency: {
    name: 'MATIC',
    symbol: 'MATIC',
    decimals: 18
  }
};

const POLYGON_MUMBAI = {
  chainId: 80001,
  chainIdHex: '0x13881',
  name: 'Polygon Mumbai Testnet',
  rpcUrls: [
    'https://rpc-mumbai.maticvigil.com',
    'https://matic-mumbai.chainstacklabs.com'
  ],
  blockExplorer: 'https://mumbai.polygonscan.com',
  nativeCurrency: {
    name: 'MATIC',
    symbol: 'MATIC',
    decimals: 18
  }
};
```

### 2.3 Gas Optimization Strategies

```solidity
// Gas-efficient patterns for Polygon
contract GasOptimized {
    // Pack storage variables (32 bytes per slot)
    struct PackedData {
        uint128 amount;      // 16 bytes
        uint64 timestamp;    // 8 bytes
        uint32 marketId;     // 4 bytes
        bool isActive;       // 1 byte
        // Total: 29 bytes - fits in one slot
    }

    // Use events for historical data (cheaper than storage)
    event OrderPlaced(
        bytes32 indexed orderId,
        address indexed trader,
        uint256 amount,
        uint256 price
    );

    // Batch operations to amortize base gas costs
    function batchSettle(
        bytes32[] calldata orderIds,
        uint256[] calldata amounts
    ) external {
        uint256 length = orderIds.length;
        for (uint256 i = 0; i < length;) {
            _settle(orderIds[i], amounts[i]);
            unchecked { ++i; } // Gas optimization
        }
    }
}
```

---

## 3. Hybrid Trading Architecture

### 3.1 CLOB (Central Limit Order Book) - Off-Chain

The CLOB handles order management and matching off-chain for performance.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLOB Architecture                             │
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   Order     │───▶│  Matching   │───▶│  Settlement │         │
│  │   Queue     │    │   Engine    │    │    Queue    │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│         ▲                  │                   │                │
│         │                  ▼                   ▼                │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   Order     │    │   Order     │    │  On-Chain   │         │
│  │  Validator  │    │    Book     │    │  Executor   │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Order Book Data Structure

```typescript
// order-book.ts
interface Order {
  id: string;
  marketId: string;
  side: 'BUY' | 'SELL';
  outcome: 'YES' | 'NO';
  price: number;          // 0.01 to 0.99
  size: number;           // Number of shares
  filled: number;         // Shares filled
  status: OrderStatus;
  maker: string;          // Wallet address
  signature: string;      // EIP-712 signature
  timestamp: number;
  expiry: number;
  salt: string;           // Unique nonce
}

type OrderStatus = 'OPEN' | 'PARTIAL' | 'FILLED' | 'CANCELLED';

interface OrderBook {
  marketId: string;
  outcome: 'YES' | 'NO';
  bids: PriceLevel[];     // Buy orders (descending by price)
  asks: PriceLevel[];     // Sell orders (ascending by price)
}

interface PriceLevel {
  price: number;
  size: number;           // Total size at this price
  orders: Order[];        // Individual orders
}

class OrderBookEngine {
  private books: Map<string, OrderBook> = new Map();

  addOrder(order: Order): MatchResult {
    const book = this.getOrCreateBook(order.marketId, order.outcome);

    if (order.side === 'BUY') {
      return this.matchBuyOrder(book, order);
    } else {
      return this.matchSellOrder(book, order);
    }
  }

  private matchBuyOrder(book: OrderBook, order: Order): MatchResult {
    const matches: Match[] = [];
    let remaining = order.size;

    // Match against asks (sell orders)
    while (remaining > 0 && book.asks.length > 0) {
      const bestAsk = book.asks[0];

      // Check if prices cross
      if (order.price < bestAsk.price) break;

      // Match orders at this price level
      for (const askOrder of bestAsk.orders) {
        const matchSize = Math.min(remaining, askOrder.size - askOrder.filled);

        if (matchSize > 0) {
          matches.push({
            buyOrder: order,
            sellOrder: askOrder,
            price: bestAsk.price,
            size: matchSize
          });

          askOrder.filled += matchSize;
          remaining -= matchSize;
        }

        if (remaining === 0) break;
      }

      // Remove filled orders and empty price levels
      this.cleanPriceLevel(book.asks, 0);
    }

    // Add remaining to order book
    if (remaining > 0) {
      this.addToBids(book, { ...order, size: remaining });
    }

    return { matches, remaining };
  }
}
```

#### EIP-712 Order Signing

```typescript
// order-signing.ts
const ORDER_TYPEHASH = {
  Order: [
    { name: 'maker', type: 'address' },
    { name: 'taker', type: 'address' },
    { name: 'tokenId', type: 'uint256' },
    { name: 'makerAmount', type: 'uint256' },
    { name: 'takerAmount', type: 'uint256' },
    { name: 'side', type: 'uint8' },
    { name: 'expiration', type: 'uint256' },
    { name: 'nonce', type: 'uint256' },
    { name: 'feeRateBps', type: 'uint256' },
    { name: 'signatureType', type: 'uint8' }
  ]
};

const DOMAIN = {
  name: 'Prediction Market Exchange',
  version: '1',
  chainId: 137, // Polygon
  verifyingContract: '0x...' // CTF Exchange address
};

async function signOrder(order: Order, signer: ethers.Signer): Promise<string> {
  const signature = await signer._signTypedData(
    DOMAIN,
    ORDER_TYPEHASH,
    order
  );
  return signature;
}

function verifyOrderSignature(order: Order, signature: string): boolean {
  const recoveredAddress = ethers.utils.verifyTypedData(
    DOMAIN,
    ORDER_TYPEHASH,
    order,
    signature
  );
  return recoveredAddress.toLowerCase() === order.maker.toLowerCase();
}
```

### 3.2 CTF (Conditional Token Framework) - On-Chain

The CTF handles token minting, transfers, and settlement on-chain.

```
┌─────────────────────────────────────────────────────────────────┐
│              Conditional Token Framework                         │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    ERC-1155 Tokens                       │   │
│  │                                                          │   │
│  │   Market: "Will X happen?"                               │   │
│  │   ┌─────────────────┐    ┌─────────────────┐            │   │
│  │   │   YES Token     │    │    NO Token     │            │   │
│  │   │   Token ID: 1   │    │   Token ID: 2   │            │   │
│  │   │   Price: $0.65  │    │   Price: $0.35  │            │   │
│  │   └─────────────────┘    └─────────────────┘            │   │
│  │                                                          │   │
│  │   Invariant: YES + NO = $1.00                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Token ID Calculation

```solidity
// TokenIdCalculation.sol
library TokenIdLib {
    /// @notice Calculate condition ID from oracle and question
    function getConditionId(
        address oracle,
        bytes32 questionId,
        uint256 outcomeSlotCount
    ) internal pure returns (bytes32) {
        return keccak256(
            abi.encodePacked(oracle, questionId, outcomeSlotCount)
        );
    }

    /// @notice Calculate collection ID for a specific outcome
    function getCollectionId(
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint256 indexSet
    ) internal pure returns (bytes32) {
        return keccak256(
            abi.encodePacked(parentCollectionId, conditionId, indexSet)
        );
    }

    /// @notice Calculate position ID (ERC-1155 token ID)
    function getPositionId(
        address collateralToken,
        bytes32 collectionId
    ) internal pure returns (uint256) {
        return uint256(keccak256(
            abi.encodePacked(collateralToken, collectionId)
        ));
    }
}
```

---

## 4. Smart Contract Architecture

### 4.1 Contract Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                   Smart Contract Architecture                    │
│                                                                  │
│  ┌─────────────────┐         ┌─────────────────┐               │
│  │  CTF Exchange   │◄───────▶│   NegRisk CTF   │               │
│  │  (Binary)       │         │   Exchange      │               │
│  └────────┬────────┘         └────────┬────────┘               │
│           │                           │                         │
│           ▼                           ▼                         │
│  ┌─────────────────────────────────────────────┐               │
│  │         Conditional Token Contract          │               │
│  │              (ERC-1155)                     │               │
│  └─────────────────────────────────────────────┘               │
│           │                           │                         │
│           ▼                           ▼                         │
│  ┌─────────────────┐         ┌─────────────────┐               │
│  │  UMA CTF        │         │     USDC        │               │
│  │  Adapter        │         │   (Collateral)  │               │
│  └────────┬────────┘         └─────────────────┘               │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                           │
│  │   UMA Oracle    │                                           │
│  │   (External)    │                                           │
│  └─────────────────┘                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 CTF Exchange Contract (Binary Markets)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import "@openzeppelin/contracts/utils/cryptography/EIP712.sol";

/**
 * @title CTFExchange
 * @notice Exchange contract for binary prediction market trading
 * @dev Handles order matching and settlement for YES/NO markets
 */
contract CTFExchange is EIP712, ReentrancyGuard {
    using ECDSA for bytes32;

    // ============ Structs ============

    struct Order {
        address maker;
        address taker;         // Address(0) for open orders
        uint256 tokenId;       // ERC-1155 token ID
        uint256 makerAmount;   // Amount maker provides
        uint256 takerAmount;   // Amount maker receives
        Side side;             // BUY or SELL
        uint256 expiration;
        uint256 nonce;
        uint16 feeRateBps;
        SignatureType signatureType;
    }

    enum Side { BUY, SELL }
    enum SignatureType { EOA, POLY_PROXY, POLY_GNOSIS_SAFE }

    // ============ State Variables ============

    IERC1155 public immutable ctf;           // Conditional Token contract
    IERC20 public immutable collateral;       // USDC
    address public operator;                  // Backend operator

    mapping(bytes32 => uint256) public orderFilled;  // Order hash => filled amount
    mapping(address => uint256) public nonces;       // Maker => nonce
    mapping(address => bool) public validSigners;    // Approved signers

    uint16 public constant MAX_FEE_BPS = 500;  // 5% max fee

    // ============ Events ============

    event OrderFilled(
        bytes32 indexed orderHash,
        address indexed maker,
        address indexed taker,
        uint256 makerAssetId,
        uint256 takerAssetId,
        uint256 makerAmountFilled,
        uint256 takerAmountFilled,
        uint256 fee
    );

    event OrderCancelled(bytes32 indexed orderHash);

    // ============ Constructor ============

    constructor(
        address _ctf,
        address _collateral,
        address _operator
    ) EIP712("CTF Exchange", "1") {
        ctf = IERC1155(_ctf);
        collateral = IERC20(_collateral);
        operator = _operator;
    }

    // ============ External Functions ============

    /**
     * @notice Fill a signed order
     * @param order The order to fill
     * @param signature The maker's signature
     * @param fillAmount Amount to fill
     */
    function fillOrder(
        Order calldata order,
        bytes calldata signature,
        uint256 fillAmount
    ) external nonReentrant {
        bytes32 orderHash = _hashOrder(order);

        // Validate order
        require(block.timestamp < order.expiration, "Order expired");
        require(
            order.taker == address(0) || order.taker == msg.sender,
            "Invalid taker"
        );

        // Verify signature
        require(
            _verifySignature(order, orderHash, signature),
            "Invalid signature"
        );

        // Check fill amount
        uint256 remainingAmount = order.makerAmount - orderFilled[orderHash];
        require(fillAmount <= remainingAmount, "Exceeds remaining");

        // Calculate amounts
        uint256 takerAmount = (fillAmount * order.takerAmount) / order.makerAmount;
        uint256 fee = (takerAmount * order.feeRateBps) / 10000;

        // Update state
        orderFilled[orderHash] += fillAmount;

        // Execute trade
        if (order.side == Side.BUY) {
            // Maker buys tokens, taker sells tokens
            // Taker sends tokens to maker
            ctf.safeTransferFrom(
                msg.sender,
                order.maker,
                order.tokenId,
                fillAmount,
                ""
            );
            // Maker sends collateral to taker (minus fee)
            collateral.transferFrom(
                order.maker,
                msg.sender,
                takerAmount - fee
            );
        } else {
            // Maker sells tokens, taker buys tokens
            // Maker sends tokens to taker
            ctf.safeTransferFrom(
                order.maker,
                msg.sender,
                order.tokenId,
                fillAmount,
                ""
            );
            // Taker sends collateral to maker (minus fee)
            collateral.transferFrom(
                msg.sender,
                order.maker,
                takerAmount - fee
            );
        }

        // Collect fee
        if (fee > 0) {
            collateral.transferFrom(
                order.side == Side.BUY ? order.maker : msg.sender,
                operator,
                fee
            );
        }

        emit OrderFilled(
            orderHash,
            order.maker,
            msg.sender,
            order.tokenId,
            0, // Collateral
            fillAmount,
            takerAmount,
            fee
        );
    }

    /**
     * @notice Cancel an order
     * @param order The order to cancel
     */
    function cancelOrder(Order calldata order) external {
        require(msg.sender == order.maker, "Not maker");
        bytes32 orderHash = _hashOrder(order);
        orderFilled[orderHash] = order.makerAmount; // Mark as fully filled
        emit OrderCancelled(orderHash);
    }

    /**
     * @notice Cancel all orders below a nonce
     * @param minNonce The minimum valid nonce
     */
    function cancelOrdersBelow(uint256 minNonce) external {
        require(minNonce > nonces[msg.sender], "Invalid nonce");
        nonces[msg.sender] = minNonce;
    }

    // ============ Internal Functions ============

    function _hashOrder(Order calldata order) internal view returns (bytes32) {
        return _hashTypedDataV4(keccak256(abi.encode(
            keccak256(
                "Order(address maker,address taker,uint256 tokenId,"
                "uint256 makerAmount,uint256 takerAmount,uint8 side,"
                "uint256 expiration,uint256 nonce,uint16 feeRateBps,"
                "uint8 signatureType)"
            ),
            order.maker,
            order.taker,
            order.tokenId,
            order.makerAmount,
            order.takerAmount,
            order.side,
            order.expiration,
            order.nonce,
            order.feeRateBps,
            order.signatureType
        )));
    }

    function _verifySignature(
        Order calldata order,
        bytes32 orderHash,
        bytes calldata signature
    ) internal view returns (bool) {
        if (order.signatureType == SignatureType.EOA) {
            address signer = orderHash.recover(signature);
            return signer == order.maker && order.nonce >= nonces[order.maker];
        }
        // Handle other signature types (proxy, gnosis safe)
        return false;
    }
}
```

### 4.3 NegRisk CTF Exchange (Multi-Outcome Markets)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title NegRiskCTFExchange
 * @notice Exchange for multi-outcome markets with negative risk
 * @dev Supports markets with 3+ outcomes where prices must sum to 1
 */
contract NegRiskCTFExchange {

    struct Market {
        bytes32 conditionId;
        uint256 outcomeCount;
        bool resolved;
        uint256 winningOutcome;
    }

    struct Position {
        uint256 marketId;
        uint256 outcome;
        uint256 shares;
    }

    mapping(uint256 => Market) public markets;
    mapping(bytes32 => Position[]) public userPositions;

    /**
     * @notice Calculate token IDs for all outcomes in a market
     * @param marketId The market identifier
     * @return tokenIds Array of token IDs for each outcome
     */
    function getOutcomeTokenIds(
        uint256 marketId
    ) external view returns (uint256[] memory tokenIds) {
        Market storage market = markets[marketId];
        tokenIds = new uint256[](market.outcomeCount);

        for (uint256 i = 0; i < market.outcomeCount; i++) {
            tokenIds[i] = _calculateTokenId(market.conditionId, i);
        }
        return tokenIds;
    }

    /**
     * @notice Mint complete sets (one of each outcome)
     * @dev Cost = 1 USDC per complete set
     * @param marketId The market to mint for
     * @param amount Number of complete sets
     */
    function mintCompleteSet(
        uint256 marketId,
        uint256 amount
    ) external {
        Market storage market = markets[marketId];
        require(!market.resolved, "Market resolved");

        // Transfer collateral
        collateral.transferFrom(msg.sender, address(this), amount);

        // Mint one token of each outcome
        for (uint256 i = 0; i < market.outcomeCount; i++) {
            uint256 tokenId = _calculateTokenId(market.conditionId, i);
            ctf.mint(msg.sender, tokenId, amount, "");
        }
    }

    /**
     * @notice Redeem complete sets for collateral
     * @param marketId The market to redeem from
     * @param amount Number of complete sets
     */
    function redeemCompleteSet(
        uint256 marketId,
        uint256 amount
    ) external {
        Market storage market = markets[marketId];

        // Burn one token of each outcome
        for (uint256 i = 0; i < market.outcomeCount; i++) {
            uint256 tokenId = _calculateTokenId(market.conditionId, i);
            ctf.burn(msg.sender, tokenId, amount);
        }

        // Return collateral
        collateral.transfer(msg.sender, amount);
    }

    /**
     * @notice Redeem winning tokens after market resolution
     * @param marketId The resolved market
     * @param amount Amount of winning tokens to redeem
     */
    function redeemWinning(
        uint256 marketId,
        uint256 amount
    ) external {
        Market storage market = markets[marketId];
        require(market.resolved, "Not resolved");

        uint256 tokenId = _calculateTokenId(
            market.conditionId,
            market.winningOutcome
        );

        // Burn winning tokens
        ctf.burn(msg.sender, tokenId, amount);

        // Pay out collateral
        collateral.transfer(msg.sender, amount);
    }

    function _calculateTokenId(
        bytes32 conditionId,
        uint256 outcome
    ) internal pure returns (uint256) {
        return uint256(keccak256(abi.encodePacked(conditionId, outcome)));
    }
}
```

### 4.4 Conditional Token Contract (ERC-1155)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";

/**
 * @title ConditionalTokens
 * @notice ERC-1155 implementation for prediction market outcome tokens
 */
contract ConditionalTokens is ERC1155, AccessControl {

    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant OPERATOR_ROLE = keccak256("OPERATOR_ROLE");

    // Condition: Oracle + Question => Outcomes
    struct Condition {
        address oracle;
        bytes32 questionId;
        uint256 outcomeSlotCount;
        uint256[] payoutNumerators;
        uint256 payoutDenominator;
    }

    mapping(bytes32 => Condition) public conditions;
    mapping(bytes32 => mapping(uint256 => uint256)) public payoutNumerators;

    event ConditionPreparation(
        bytes32 indexed conditionId,
        address indexed oracle,
        bytes32 indexed questionId,
        uint256 outcomeSlotCount
    );

    event ConditionResolution(
        bytes32 indexed conditionId,
        address indexed oracle,
        bytes32 indexed questionId,
        uint256 outcomeSlotCount,
        uint256[] payoutNumerators
    );

    constructor(string memory uri) ERC1155(uri) {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
    }

    /**
     * @notice Prepare a new condition (market)
     * @param oracle The oracle that will resolve this condition
     * @param questionId Unique identifier for the question
     * @param outcomeSlotCount Number of possible outcomes
     */
    function prepareCondition(
        address oracle,
        bytes32 questionId,
        uint256 outcomeSlotCount
    ) external returns (bytes32 conditionId) {
        require(outcomeSlotCount > 1, "Min 2 outcomes");
        require(outcomeSlotCount <= 256, "Max 256 outcomes");

        conditionId = keccak256(
            abi.encodePacked(oracle, questionId, outcomeSlotCount)
        );

        require(
            conditions[conditionId].outcomeSlotCount == 0,
            "Condition exists"
        );

        conditions[conditionId] = Condition({
            oracle: oracle,
            questionId: questionId,
            outcomeSlotCount: outcomeSlotCount,
            payoutNumerators: new uint256[](outcomeSlotCount),
            payoutDenominator: 0
        });

        emit ConditionPreparation(
            conditionId,
            oracle,
            questionId,
            outcomeSlotCount
        );
    }

    /**
     * @notice Resolve a condition (only callable by oracle)
     * @param questionId The question being resolved
     * @param outcomeSlotCount Number of outcomes
     * @param payoutNumerators Payout distribution (usually [1,0] or [0,1])
     */
    function reportPayouts(
        bytes32 questionId,
        uint256 outcomeSlotCount,
        uint256[] calldata payoutNumerators
    ) external {
        bytes32 conditionId = keccak256(
            abi.encodePacked(msg.sender, questionId, outcomeSlotCount)
        );

        Condition storage condition = conditions[conditionId];
        require(condition.outcomeSlotCount > 0, "Condition not found");
        require(condition.oracle == msg.sender, "Not oracle");
        require(condition.payoutDenominator == 0, "Already resolved");
        require(
            payoutNumerators.length == outcomeSlotCount,
            "Invalid payouts length"
        );

        uint256 denominator = 0;
        for (uint256 i = 0; i < outcomeSlotCount; i++) {
            denominator += payoutNumerators[i];
        }
        require(denominator > 0, "Invalid payouts");

        condition.payoutDenominator = denominator;
        for (uint256 i = 0; i < outcomeSlotCount; i++) {
            condition.payoutNumerators[i] = payoutNumerators[i];
        }

        emit ConditionResolution(
            conditionId,
            msg.sender,
            questionId,
            outcomeSlotCount,
            payoutNumerators
        );
    }

    /**
     * @notice Mint outcome tokens
     * @dev Only callable by authorized minters (exchange contracts)
     */
    function mint(
        address to,
        uint256 tokenId,
        uint256 amount,
        bytes memory data
    ) external onlyRole(MINTER_ROLE) {
        _mint(to, tokenId, amount, data);
    }

    /**
     * @notice Burn outcome tokens
     * @dev Only callable by authorized operators
     */
    function burn(
        address from,
        uint256 tokenId,
        uint256 amount
    ) external onlyRole(OPERATOR_ROLE) {
        _burn(from, tokenId, amount);
    }

    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(ERC1155, AccessControl)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }
}
```

---

## 5. Oracle System

### 5.1 UMA Optimistic Oracle Integration

```
┌─────────────────────────────────────────────────────────────────┐
│                    Oracle Resolution Flow                        │
│                                                                  │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │
│  │ Market  │───▶│ Request │───▶│ Propose │───▶│ Dispute │     │
│  │ Expires │    │ Price   │    │ Answer  │    │ Period  │     │
│  └─────────┘    └─────────┘    └─────────┘    └────┬────┘     │
│                                                     │           │
│                      ┌──────────────────────────────┤           │
│                      │                              │           │
│                      ▼                              ▼           │
│              ┌─────────────┐              ┌─────────────┐       │
│              │  Disputed   │              │ No Dispute  │       │
│              │ (DVM Vote)  │              │  (Settle)   │       │
│              └──────┬──────┘              └──────┬──────┘       │
│                     │                            │              │
│                     ▼                            ▼              │
│              ┌─────────────────────────────────────┐           │
│              │         Market Resolved             │           │
│              │     (Payouts Distributed)           │           │
│              └─────────────────────────────────────┘           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 UMA CTF Adapter Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "./interfaces/IOptimisticOracleV2.sol";
import "./interfaces/IConditionalTokens.sol";

/**
 * @title UmaCtfAdapter
 * @notice Adapter to use UMA's Optimistic Oracle for CTF resolution
 */
contract UmaCtfAdapter {

    // ============ Constants ============

    bytes32 public constant YES_KECCAK = keccak256("YES");
    bytes32 public constant NO_KECCAK = keccak256("NO");

    // ============ Structs ============

    struct Question {
        bytes32 questionId;
        bytes ancillaryData;
        uint256 requestTimestamp;
        uint256 resolutionTimestamp;
        address creator;
        bool resolved;
        bool emergencyResolved;
        int256 proposedPrice;
    }

    // ============ State ============

    IOptimisticOracleV2 public immutable optimisticOracle;
    IConditionalTokens public immutable ctf;
    IERC20 public immutable currency; // Bond currency

    uint256 public constant DEFAULT_LIVENESS = 7200; // 2 hours
    uint256 public constant BOND_AMOUNT = 1000e6;    // 1000 USDC

    mapping(bytes32 => Question) public questions;

    // ============ Events ============

    event QuestionInitialized(
        bytes32 indexed questionId,
        bytes ancillaryData,
        uint256 requestTimestamp
    );

    event QuestionResolved(
        bytes32 indexed questionId,
        int256 settledPrice,
        uint256[] payoutNumerators
    );

    // ============ Constructor ============

    constructor(
        address _optimisticOracle,
        address _ctf,
        address _currency
    ) {
        optimisticOracle = IOptimisticOracleV2(_optimisticOracle);
        ctf = IConditionalTokens(_ctf);
        currency = IERC20(_currency);
    }

    // ============ External Functions ============

    /**
     * @notice Initialize a question for resolution
     * @param ancillaryData The question text and metadata
     * @param resolutionTimestamp When the question can be resolved
     */
    function initializeQuestion(
        bytes memory ancillaryData,
        uint256 resolutionTimestamp
    ) external returns (bytes32 questionId) {
        require(
            resolutionTimestamp > block.timestamp,
            "Resolution must be future"
        );

        questionId = keccak256(
            abi.encodePacked(ancillaryData, resolutionTimestamp)
        );

        require(
            questions[questionId].requestTimestamp == 0,
            "Question exists"
        );

        questions[questionId] = Question({
            questionId: questionId,
            ancillaryData: ancillaryData,
            requestTimestamp: 0,
            resolutionTimestamp: resolutionTimestamp,
            creator: msg.sender,
            resolved: false,
            emergencyResolved: false,
            proposedPrice: 0
        });

        emit QuestionInitialized(
            questionId,
            ancillaryData,
            resolutionTimestamp
        );
    }

    /**
     * @notice Request price from UMA Oracle
     * @param questionId The question to resolve
     */
    function requestResolution(bytes32 questionId) external {
        Question storage question = questions[questionId];

        require(question.questionId != bytes32(0), "Question not found");
        require(
            block.timestamp >= question.resolutionTimestamp,
            "Too early"
        );
        require(question.requestTimestamp == 0, "Already requested");

        // Request price from UMA
        optimisticOracle.requestPrice(
            bytes32("YES_OR_NO_QUERY"),
            question.resolutionTimestamp,
            question.ancillaryData,
            currency,
            0 // Reward (handled separately)
        );

        question.requestTimestamp = block.timestamp;
    }

    /**
     * @notice Propose an answer to the question
     * @param questionId The question
     * @param proposedPrice 1e18 for YES, 0 for NO
     */
    function proposeAnswer(
        bytes32 questionId,
        int256 proposedPrice
    ) external {
        Question storage question = questions[questionId];
        require(question.requestTimestamp > 0, "Not requested");
        require(!question.resolved, "Already resolved");

        // Validate price (must be 0 or 1e18)
        require(
            proposedPrice == 0 || proposedPrice == 1e18,
            "Invalid price"
        );

        // Transfer bond
        currency.transferFrom(msg.sender, address(this), BOND_AMOUNT);
        currency.approve(address(optimisticOracle), BOND_AMOUNT);

        // Propose to UMA
        optimisticOracle.proposePrice(
            address(this),
            bytes32("YES_OR_NO_QUERY"),
            question.resolutionTimestamp,
            question.ancillaryData,
            proposedPrice
        );

        question.proposedPrice = proposedPrice;
    }

    /**
     * @notice Settle the question after liveness period
     * @param questionId The question to settle
     */
    function settle(bytes32 questionId) external {
        Question storage question = questions[questionId];
        require(!question.resolved, "Already resolved");

        // Settle with UMA
        int256 settledPrice = optimisticOracle.settle(
            address(this),
            bytes32("YES_OR_NO_QUERY"),
            question.resolutionTimestamp,
            question.ancillaryData
        );

        // Convert price to payouts
        uint256[] memory payoutNumerators = new uint256[](2);
        if (settledPrice == 1e18) {
            // YES wins
            payoutNumerators[0] = 1;
            payoutNumerators[1] = 0;
        } else {
            // NO wins
            payoutNumerators[0] = 0;
            payoutNumerators[1] = 1;
        }

        // Report to CTF
        ctf.reportPayouts(questionId, 2, payoutNumerators);

        question.resolved = true;

        emit QuestionResolved(questionId, settledPrice, payoutNumerators);
    }

    /**
     * @notice Dispute a proposed answer
     * @param questionId The question being disputed
     */
    function dispute(bytes32 questionId) external {
        Question storage question = questions[questionId];
        require(question.requestTimestamp > 0, "Not requested");
        require(!question.resolved, "Already resolved");

        // Transfer bond for dispute
        currency.transferFrom(msg.sender, address(this), BOND_AMOUNT);
        currency.approve(address(optimisticOracle), BOND_AMOUNT);

        // Dispute with UMA
        optimisticOracle.disputePrice(
            address(this),
            bytes32("YES_OR_NO_QUERY"),
            question.resolutionTimestamp,
            question.ancillaryData
        );
    }
}
```

### 5.3 Oracle Economics

| Role | Action | Stake | Outcome |
|------|--------|-------|---------|
| **Proposer** | Submits answer | Bond (e.g., 1000 USDC) | Correct: Bond returned + reward |
| **Disputer** | Challenges answer | Bond (e.g., 1000 USDC) | Correct: Proposer's bond + own bond |
| **DVM Voters** | Vote on disputed answers | UMA tokens | Aligned with consensus: Rewards |

---

## 6. Wallet Architecture

### 6.1 Proxy Wallet System

```
┌─────────────────────────────────────────────────────────────────┐
│                    Wallet Architecture                           │
│                                                                  │
│  User's EOA                                                      │
│  (MetaMask/MagicLink)                                           │
│         │                                                        │
│         │ Controls (1-of-1 Multisig)                            │
│         ▼                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              User's Proxy Wallet                         │   │
│  │              (Deployed to Polygon)                       │   │
│  │                                                          │   │
│  │   ┌──────────────┐    ┌──────────────┐                  │   │
│  │   │    USDC      │    │  ERC-1155    │                  │   │
│  │   │   Balance    │    │  Positions   │                  │   │
│  │   │   $5,000     │    │  YES: 100    │                  │   │
│  │   │              │    │  NO: 50      │                  │   │
│  │   └──────────────┘    └──────────────┘                  │   │
│  │                                                          │   │
│  │   Features:                                              │   │
│  │   - Atomic multi-step transactions                      │   │
│  │   - Gas abstraction                                     │   │
│  │   - Batch operations                                    │   │
│  │   - Approval management                                 │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Proxy Wallet Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Holder.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

/**
 * @title ProxyWallet
 * @notice 1-of-1 multisig wallet for user positions
 */
contract ProxyWallet is Initializable, ERC1155Holder {

    address public owner;
    address public operator;  // Platform operator for meta-tx

    uint256 public nonce;

    event Executed(address indexed target, uint256 value, bytes data);
    event OwnerChanged(address indexed oldOwner, address indexed newOwner);

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    modifier onlyAuthorized() {
        require(
            msg.sender == owner || msg.sender == operator,
            "Not authorized"
        );
        _;
    }

    function initialize(
        address _owner,
        address _operator
    ) external initializer {
        owner = _owner;
        operator = _operator;
    }

    /**
     * @notice Execute a single transaction
     */
    function execute(
        address target,
        uint256 value,
        bytes calldata data
    ) external onlyAuthorized returns (bytes memory) {
        (bool success, bytes memory result) = target.call{value: value}(data);
        require(success, "Execution failed");

        emit Executed(target, value, data);
        return result;
    }

    /**
     * @notice Execute multiple transactions atomically
     * @dev All must succeed or entire batch reverts
     */
    function executeBatch(
        address[] calldata targets,
        uint256[] calldata values,
        bytes[] calldata datas
    ) external onlyAuthorized returns (bytes[] memory results) {
        require(
            targets.length == values.length &&
            targets.length == datas.length,
            "Length mismatch"
        );

        results = new bytes[](targets.length);

        for (uint256 i = 0; i < targets.length; i++) {
            (bool success, bytes memory result) = targets[i].call{
                value: values[i]
            }(datas[i]);
            require(success, "Batch execution failed");
            results[i] = result;

            emit Executed(targets[i], values[i], datas[i]);
        }
    }

    /**
     * @notice Execute with meta-transaction (gasless for user)
     */
    function executeWithSignature(
        address target,
        uint256 value,
        bytes calldata data,
        uint256 _nonce,
        bytes calldata signature
    ) external returns (bytes memory) {
        require(_nonce == nonce, "Invalid nonce");

        bytes32 hash = keccak256(
            abi.encodePacked(
                "\x19\x01",
                _domainSeparator(),
                keccak256(abi.encode(target, value, data, _nonce))
            )
        );

        address signer = _recoverSigner(hash, signature);
        require(signer == owner, "Invalid signature");

        nonce++;

        (bool success, bytes memory result) = target.call{value: value}(data);
        require(success, "Execution failed");

        emit Executed(target, value, data);
        return result;
    }

    /**
     * @notice Approve token spending in a single atomic operation
     * @dev Useful for first-time trades
     */
    function approveAndExecute(
        address token,
        address spender,
        uint256 amount,
        address target,
        bytes calldata data
    ) external onlyAuthorized returns (bytes memory) {
        // Approve
        IERC20(token).approve(spender, amount);

        // Execute
        (bool success, bytes memory result) = target.call(data);
        require(success, "Execution failed");

        emit Executed(target, 0, data);
        return result;
    }

    /**
     * @notice Change wallet owner
     */
    function changeOwner(address newOwner) external onlyOwner {
        require(newOwner != address(0), "Invalid owner");
        emit OwnerChanged(owner, newOwner);
        owner = newOwner;
    }

    /**
     * @notice Withdraw ERC20 tokens
     */
    function withdrawERC20(
        address token,
        address to,
        uint256 amount
    ) external onlyOwner {
        IERC20(token).transfer(to, amount);
    }

    /**
     * @notice Withdraw native currency
     */
    function withdrawNative(
        address payable to,
        uint256 amount
    ) external onlyOwner {
        to.transfer(amount);
    }

    function _domainSeparator() internal view returns (bytes32) {
        return keccak256(
            abi.encode(
                keccak256(
                    "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
                ),
                keccak256("ProxyWallet"),
                keccak256("1"),
                block.chainid,
                address(this)
            )
        );
    }

    function _recoverSigner(
        bytes32 hash,
        bytes calldata signature
    ) internal pure returns (address) {
        (bytes32 r, bytes32 s, uint8 v) = _splitSignature(signature);
        return ecrecover(hash, v, r, s);
    }

    function _splitSignature(
        bytes calldata sig
    ) internal pure returns (bytes32 r, bytes32 s, uint8 v) {
        require(sig.length == 65, "Invalid signature length");
        assembly {
            r := calldataload(sig.offset)
            s := calldataload(add(sig.offset, 32))
            v := byte(0, calldataload(add(sig.offset, 64)))
        }
    }

    receive() external payable {}
}
```

### 6.3 Wallet Factory

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/proxy/Clones.sol";

/**
 * @title ProxyWalletFactory
 * @notice Factory for deploying user proxy wallets
 */
contract ProxyWalletFactory {

    address public immutable implementation;
    address public operator;

    mapping(address => address) public wallets;

    event WalletCreated(address indexed owner, address indexed wallet);

    constructor(address _implementation, address _operator) {
        implementation = _implementation;
        operator = _operator;
    }

    /**
     * @notice Create a new proxy wallet for a user
     * @param owner The EOA that will control the wallet
     */
    function createWallet(address owner) external returns (address wallet) {
        require(wallets[owner] == address(0), "Wallet exists");

        // Create deterministic clone
        bytes32 salt = keccak256(abi.encodePacked(owner));
        wallet = Clones.cloneDeterministic(implementation, salt);

        // Initialize
        ProxyWallet(payable(wallet)).initialize(owner, operator);

        wallets[owner] = wallet;

        emit WalletCreated(owner, wallet);
    }

    /**
     * @notice Get wallet address for an owner (even if not deployed)
     */
    function getWalletAddress(address owner) external view returns (address) {
        bytes32 salt = keccak256(abi.encodePacked(owner));
        return Clones.predictDeterministicAddress(implementation, salt);
    }

    /**
     * @notice Check if wallet exists for owner
     */
    function hasWallet(address owner) external view returns (bool) {
        return wallets[owner] != address(0);
    }
}
```

---

## 7. Settlement System

### 7.1 Settlement Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    Settlement Flow                               │
│                                                                  │
│  1. Order Matching (Off-chain)                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Maker Order          Taker Order                        │   │
│  │  ┌──────────────┐    ┌──────────────┐                   │   │
│  │  │ Buy 100 YES  │◄──▶│ Sell 100 YES │                   │   │
│  │  │ @ $0.65      │    │ @ $0.65      │                   │   │
│  │  └──────────────┘    └──────────────┘                   │   │
│  │                  ▼                                       │   │
│  │           MATCH FOUND                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          │                                      │
│                          ▼                                      │
│  2. Settlement Preparation (Backend)                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  - Validate signatures                                   │   │
│  │  - Check balances                                        │   │
│  │  - Prepare transaction data                              │   │
│  │  - Bundle multiple trades                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          │                                      │
│                          ▼                                      │
│  3. On-chain Settlement                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  CTF Exchange Contract                                   │   │
│  │  ┌──────────────────────────────────────────────────┐   │   │
│  │  │  1. Verify signatures                             │   │   │
│  │  │  2. Transfer USDC: Buyer → Seller                 │   │   │
│  │  │  3. Transfer YES tokens: Seller → Buyer           │   │   │
│  │  │  4. Collect fees                                  │   │   │
│  │  │  5. Emit events                                   │   │   │
│  │  └──────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Settlement Service

```typescript
// settlement-service.ts
import { ethers } from 'ethers';

interface MatchedTrade {
  buyOrder: SignedOrder;
  sellOrder: SignedOrder;
  fillAmount: bigint;
  price: number;
  timestamp: number;
}

interface SettlementBatch {
  trades: MatchedTrade[];
  totalGas: bigint;
  priority: 'HIGH' | 'MEDIUM' | 'LOW';
}

class SettlementService {
  private provider: ethers.Provider;
  private signer: ethers.Signer;
  private exchange: ethers.Contract;
  private pendingBatch: MatchedTrade[] = [];
  private batchThreshold = 10;
  private batchTimeout = 5000; // 5 seconds

  constructor(
    provider: ethers.Provider,
    signer: ethers.Signer,
    exchangeAddress: string,
    exchangeAbi: any[]
  ) {
    this.provider = provider;
    this.signer = signer;
    this.exchange = new ethers.Contract(
      exchangeAddress,
      exchangeAbi,
      signer
    );
  }

  /**
   * Add a matched trade to the settlement queue
   */
  async queueSettlement(trade: MatchedTrade): Promise<void> {
    // Validate trade before queuing
    await this.validateTrade(trade);

    this.pendingBatch.push(trade);

    // Settle immediately if batch threshold reached
    if (this.pendingBatch.length >= this.batchThreshold) {
      await this.settleBatch();
    }
  }

  /**
   * Settle a batch of trades atomically
   */
  async settleBatch(): Promise<ethers.TransactionReceipt> {
    if (this.pendingBatch.length === 0) {
      throw new Error('No trades to settle');
    }

    const batch = [...this.pendingBatch];
    this.pendingBatch = [];

    // Prepare batch data
    const orders = batch.map(t => [t.buyOrder, t.sellOrder]).flat();
    const signatures = batch.map(t => [
      t.buyOrder.signature,
      t.sellOrder.signature
    ]).flat();
    const fillAmounts = batch.map(t => t.fillAmount);

    // Estimate gas
    const gasEstimate = await this.exchange.matchOrders.estimateGas(
      orders,
      signatures,
      fillAmounts
    );

    // Get current gas price
    const feeData = await this.provider.getFeeData();

    // Submit transaction
    const tx = await this.exchange.matchOrders(
      orders,
      signatures,
      fillAmounts,
      {
        gasLimit: gasEstimate * 120n / 100n, // 20% buffer
        maxFeePerGas: feeData.maxFeePerGas,
        maxPriorityFeePerGas: feeData.maxPriorityFeePerGas
      }
    );

    // Wait for confirmation
    const receipt = await tx.wait(1);

    // Process events
    await this.processSettlementEvents(receipt, batch);

    return receipt;
  }

  /**
   * Validate a trade can be settled
   */
  private async validateTrade(trade: MatchedTrade): Promise<void> {
    // Verify signatures
    const buyerValid = await this.exchange.isValidSignature(
      trade.buyOrder,
      trade.buyOrder.signature
    );
    const sellerValid = await this.exchange.isValidSignature(
      trade.sellOrder,
      trade.sellOrder.signature
    );

    if (!buyerValid || !sellerValid) {
      throw new Error('Invalid order signature');
    }

    // Check USDC balance
    const buyer = trade.buyOrder.maker;
    const usdcNeeded = trade.fillAmount * BigInt(trade.price * 100) / 100n;
    const buyerBalance = await this.getUSDCBalance(buyer);

    if (buyerBalance < usdcNeeded) {
      throw new Error('Insufficient USDC balance');
    }

    // Check token balance
    const seller = trade.sellOrder.maker;
    const tokenId = trade.sellOrder.tokenId;
    const sellerTokens = await this.getTokenBalance(seller, tokenId);

    if (sellerTokens < trade.fillAmount) {
      throw new Error('Insufficient token balance');
    }
  }

  /**
   * Process settlement events and update order status
   */
  private async processSettlementEvents(
    receipt: ethers.TransactionReceipt,
    batch: MatchedTrade[]
  ): Promise<void> {
    const events = receipt.logs
      .map(log => {
        try {
          return this.exchange.interface.parseLog(log);
        } catch {
          return null;
        }
      })
      .filter(e => e !== null);

    for (const event of events) {
      if (event.name === 'OrderFilled') {
        // Update order status in database
        await this.updateOrderStatus(
          event.args.orderHash,
          event.args.makerAmountFilled
        );

        // Notify users
        await this.notifyTradeSettled(
          event.args.maker,
          event.args.taker,
          event.args.makerAmountFilled
        );
      }
    }
  }

  private async getUSDCBalance(address: string): Promise<bigint> {
    // Implementation
    return 0n;
  }

  private async getTokenBalance(
    address: string,
    tokenId: bigint
  ): Promise<bigint> {
    // Implementation
    return 0n;
  }

  private async updateOrderStatus(
    orderHash: string,
    filled: bigint
  ): Promise<void> {
    // Implementation
  }

  private async notifyTradeSettled(
    maker: string,
    taker: string,
    amount: bigint
  ): Promise<void> {
    // Implementation
  }
}
```

---

## 8. Frontend Architecture

### 8.1 Application Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    Frontend Architecture                         │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     Next.js App                          │   │
│  │                                                          │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐           │   │
│  │  │   Pages   │  │Components │  │   Hooks   │           │   │
│  │  │           │  │           │  │           │           │   │
│  │  │ /markets  │  │ OrderBook │  │ useWallet │           │   │
│  │  │ /market/  │  │ TradeForm │  │ useOrders │           │   │
│  │  │ /profile  │  │ Portfolio │  │ useMarket │           │   │
│  │  └───────────┘  └───────────┘  └───────────┘           │   │
│  │                                                          │   │
│  │  ┌───────────────────────────────────────────────────┐  │   │
│  │  │                State Management                    │  │   │
│  │  │  ┌──────────────┐    ┌──────────────┐            │  │   │
│  │  │  │   Zustand    │    │ React Query  │            │  │   │
│  │  │  │   (Client)   │    │  (Server)    │            │  │   │
│  │  │  └──────────────┘    └──────────────┘            │  │   │
│  │  └───────────────────────────────────────────────────┘  │   │
│  │                                                          │   │
│  │  ┌───────────────────────────────────────────────────┐  │   │
│  │  │              Web3 Integration                      │  │   │
│  │  │  ┌──────────────┐    ┌──────────────┐            │  │   │
│  │  │  │    wagmi     │    │   ethers.js  │            │  │   │
│  │  │  │  (Wallet)    │    │  (Contract)  │            │  │   │
│  │  │  └──────────────┘    └──────────────┘            │  │   │
│  │  └───────────────────────────────────────────────────┘  │   │
│  │                                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Directory Structure

```
src/
├── app/                      # Next.js App Router
│   ├── layout.tsx
│   ├── page.tsx
│   ├── markets/
│   │   ├── page.tsx          # Market listing
│   │   └── [id]/
│   │       └── page.tsx      # Individual market
│   ├── portfolio/
│   │   └── page.tsx
│   └── api/
│       ├── markets/
│       │   └── route.ts
│       └── orders/
│           └── route.ts
│
├── components/
│   ├── markets/
│   │   ├── MarketCard.tsx
│   │   ├── MarketList.tsx
│   │   └── MarketDetail.tsx
│   ├── trading/
│   │   ├── OrderBook.tsx
│   │   ├── TradePanel.tsx
│   │   ├── OrderForm.tsx
│   │   └── PriceChart.tsx
│   ├── portfolio/
│   │   ├── PositionList.tsx
│   │   └── OrderHistory.tsx
│   └── wallet/
│       ├── ConnectButton.tsx
│       └── WalletInfo.tsx
│
├── hooks/
│   ├── useWallet.ts
│   ├── useOrders.ts
│   ├── useMarket.ts
│   ├── usePositions.ts
│   └── useOrderBook.ts
│
├── lib/
│   ├── contracts/
│   │   ├── abis/
│   │   └── addresses.ts
│   ├── api/
│   │   ├── client.ts
│   │   └── types.ts
│   └── utils/
│       ├── formatting.ts
│       └── signatures.ts
│
├── stores/
│   ├── orderStore.ts
│   └── walletStore.ts
│
└── types/
    ├── market.ts
    ├── order.ts
    └── position.ts
```

### 8.3 Key React Hooks

```typescript
// hooks/useWallet.ts
import { useAccount, useConnect, useDisconnect } from 'wagmi';
import { useQuery, useMutation } from '@tanstack/react-query';

export function useWallet() {
  const { address, isConnected, chain } = useAccount();
  const { connect, connectors } = useConnect();
  const { disconnect } = useDisconnect();

  // Fetch proxy wallet address
  const { data: proxyWallet } = useQuery({
    queryKey: ['proxyWallet', address],
    queryFn: async () => {
      if (!address) return null;
      const response = await fetch(`/api/wallet/${address}`);
      return response.json();
    },
    enabled: !!address
  });

  // Fetch USDC balance
  const { data: usdcBalance } = useQuery({
    queryKey: ['usdcBalance', proxyWallet?.address],
    queryFn: async () => {
      if (!proxyWallet?.address) return 0;
      // Fetch from contract
    },
    enabled: !!proxyWallet?.address,
    refetchInterval: 10000
  });

  // Create proxy wallet mutation
  const createWallet = useMutation({
    mutationFn: async () => {
      const response = await fetch('/api/wallet/create', {
        method: 'POST',
        body: JSON.stringify({ owner: address })
      });
      return response.json();
    }
  });

  return {
    address,
    isConnected,
    chain,
    proxyWallet,
    usdcBalance,
    connect,
    disconnect,
    createWallet,
    connectors
  };
}

// hooks/useOrders.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { signTypedData } from 'wagmi/actions';

interface PlaceOrderParams {
  marketId: string;
  side: 'BUY' | 'SELL';
  outcome: 'YES' | 'NO';
  price: number;
  size: number;
}

export function useOrders(marketId: string) {
  const queryClient = useQueryClient();
  const { address, proxyWallet } = useWallet();

  // Fetch open orders
  const { data: openOrders } = useQuery({
    queryKey: ['openOrders', marketId, address],
    queryFn: async () => {
      const response = await fetch(
        `/api/orders?marketId=${marketId}&maker=${address}`
      );
      return response.json();
    },
    enabled: !!address
  });

  // Place order mutation
  const placeOrder = useMutation({
    mutationFn: async (params: PlaceOrderParams) => {
      // Build order
      const order = {
        maker: proxyWallet.address,
        taker: ethers.constants.AddressZero,
        tokenId: getTokenId(params.marketId, params.outcome),
        makerAmount: params.size,
        takerAmount: Math.floor(params.size * params.price * 1e6),
        side: params.side === 'BUY' ? 0 : 1,
        expiration: Math.floor(Date.now() / 1000) + 86400, // 24 hours
        nonce: Date.now(),
        feeRateBps: 0,
        signatureType: 0
      };

      // Sign order
      const signature = await signTypedData({
        domain: EXCHANGE_DOMAIN,
        types: ORDER_TYPES,
        value: order
      });

      // Submit to API
      const response = await fetch('/api/orders', {
        method: 'POST',
        body: JSON.stringify({ order, signature })
      });

      return response.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries(['openOrders', marketId]);
      queryClient.invalidateQueries(['orderBook', marketId]);
    }
  });

  // Cancel order mutation
  const cancelOrder = useMutation({
    mutationFn: async (orderId: string) => {
      const response = await fetch(`/api/orders/${orderId}`, {
        method: 'DELETE'
      });
      return response.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries(['openOrders', marketId]);
    }
  });

  return {
    openOrders,
    placeOrder,
    cancelOrder
  };
}

// hooks/useOrderBook.ts
import { useQuery } from '@tanstack/react-query';
import { useWebSocket } from './useWebSocket';

export function useOrderBook(marketId: string, outcome: 'YES' | 'NO') {
  // Initial fetch
  const { data: initialData } = useQuery({
    queryKey: ['orderBook', marketId, outcome],
    queryFn: async () => {
      const response = await fetch(
        `/api/orderbook/${marketId}?outcome=${outcome}`
      );
      return response.json();
    }
  });

  // Real-time updates via WebSocket
  const { data: wsData } = useWebSocket(
    `wss://api.example.com/ws/orderbook/${marketId}/${outcome}`
  );

  // Merge initial and real-time data
  const orderBook = useMemo(() => {
    if (!initialData) return null;
    if (!wsData) return initialData;

    // Apply incremental updates
    return applyOrderBookUpdate(initialData, wsData);
  }, [initialData, wsData]);

  const bestBid = orderBook?.bids[0]?.price ?? 0;
  const bestAsk = orderBook?.asks[0]?.price ?? 1;
  const spread = bestAsk - bestBid;
  const midPrice = (bestBid + bestAsk) / 2;

  return {
    orderBook,
    bestBid,
    bestAsk,
    spread,
    midPrice
  };
}
```

### 8.4 Trading Component

```typescript
// components/trading/TradePanel.tsx
import { useState } from 'react';
import { useOrders } from '@/hooks/useOrders';
import { useOrderBook } from '@/hooks/useOrderBook';
import { useWallet } from '@/hooks/useWallet';

interface TradePanelProps {
  marketId: string;
}

export function TradePanel({ marketId }: TradePanelProps) {
  const [outcome, setOutcome] = useState<'YES' | 'NO'>('YES');
  const [side, setSide] = useState<'BUY' | 'SELL'>('BUY');
  const [orderType, setOrderType] = useState<'MARKET' | 'LIMIT'>('LIMIT');
  const [price, setPrice] = useState<number>(0.5);
  const [amount, setAmount] = useState<number>(10);

  const { isConnected, usdcBalance } = useWallet();
  const { placeOrder } = useOrders(marketId);
  const { bestBid, bestAsk, midPrice } = useOrderBook(marketId, outcome);

  // Calculate costs
  const effectivePrice = orderType === 'MARKET'
    ? (side === 'BUY' ? bestAsk : bestBid)
    : price;

  const totalCost = amount * effectivePrice;
  const potentialReturn = side === 'BUY'
    ? amount * (1 - effectivePrice) // Profit if YES wins
    : amount * effectivePrice;       // Profit if NO wins

  const handleSubmit = async () => {
    await placeOrder.mutateAsync({
      marketId,
      side,
      outcome,
      price: effectivePrice,
      size: amount
    });
  };

  return (
    <div className="trade-panel">
      {/* Outcome Selection */}
      <div className="outcome-tabs">
        <button
          className={outcome === 'YES' ? 'active' : ''}
          onClick={() => setOutcome('YES')}
        >
          YES
        </button>
        <button
          className={outcome === 'NO' ? 'active' : ''}
          onClick={() => setOutcome('NO')}
        >
          NO
        </button>
      </div>

      {/* Side Selection */}
      <div className="side-tabs">
        <button
          className={side === 'BUY' ? 'active buy' : ''}
          onClick={() => setSide('BUY')}
        >
          Buy
        </button>
        <button
          className={side === 'SELL' ? 'active sell' : ''}
          onClick={() => setSide('SELL')}
        >
          Sell
        </button>
      </div>

      {/* Order Type */}
      <div className="order-type">
        <label>
          <input
            type="radio"
            checked={orderType === 'LIMIT'}
            onChange={() => setOrderType('LIMIT')}
          />
          Limit
        </label>
        <label>
          <input
            type="radio"
            checked={orderType === 'MARKET'}
            onChange={() => setOrderType('MARKET')}
          />
          Market
        </label>
      </div>

      {/* Price Input (Limit orders only) */}
      {orderType === 'LIMIT' && (
        <div className="price-input">
          <label>Price</label>
          <input
            type="number"
            min={0.01}
            max={0.99}
            step={0.01}
            value={price}
            onChange={(e) => setPrice(parseFloat(e.target.value))}
          />
          <span className="hint">
            Best: {side === 'BUY' ? bestAsk.toFixed(2) : bestBid.toFixed(2)}
          </span>
        </div>
      )}

      {/* Amount Input */}
      <div className="amount-input">
        <label>Shares</label>
        <input
          type="number"
          min={1}
          value={amount}
          onChange={(e) => setAmount(parseInt(e.target.value))}
        />
      </div>

      {/* Order Summary */}
      <div className="order-summary">
        <div className="row">
          <span>Total Cost</span>
          <span>${totalCost.toFixed(2)}</span>
        </div>
        <div className="row">
          <span>Potential Return</span>
          <span className="positive">+${potentialReturn.toFixed(2)}</span>
        </div>
        <div className="row">
          <span>Max Loss</span>
          <span className="negative">-${totalCost.toFixed(2)}</span>
        </div>
      </div>

      {/* Submit Button */}
      <button
        className="submit-btn"
        disabled={!isConnected || totalCost > usdcBalance}
        onClick={handleSubmit}
      >
        {!isConnected
          ? 'Connect Wallet'
          : totalCost > usdcBalance
          ? 'Insufficient Balance'
          : `${side} ${amount} ${outcome} @ $${effectivePrice.toFixed(2)}`}
      </button>
    </div>
  );
}
```

---

## 9. System Diagrams

### 9.1 Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA FLOW DIAGRAM                                  │
│                                                                              │
│  ┌─────────────┐                                           ┌─────────────┐  │
│  │   User      │                                           │  Polygon    │  │
│  │   Browser   │                                           │  Network    │  │
│  └──────┬──────┘                                           └──────▲──────┘  │
│         │                                                         │         │
│         │ 1. Place Order                                          │         │
│         ▼                                                         │         │
│  ┌─────────────────────────────────────────────────────────┐     │         │
│  │                    API Gateway                           │     │         │
│  │                                                          │     │         │
│  │  ┌──────────────┐    ┌──────────────┐                   │     │         │
│  │  │ REST API     │    │ WebSocket    │                   │     │         │
│  │  │ /api/orders  │    │ Server       │                   │     │         │
│  │  └──────┬───────┘    └───────▲──────┘                   │     │         │
│  └─────────┼────────────────────┼───────────────────────────┘     │         │
│            │                    │                                  │         │
│            │ 2. Validate        │ 6. Real-time                    │         │
│            │    & Queue         │    Updates                       │         │
│            ▼                    │                                  │         │
│  ┌─────────────────────────────────────────────────────────┐     │         │
│  │                   Order Service                          │     │         │
│  │                                                          │     │         │
│  │  ┌──────────────┐    ┌──────────────┐    ┌──────────┐  │     │         │
│  │  │ Validation   │───▶│   Order      │───▶│ Match    │  │     │         │
│  │  │ Engine       │    │   Queue      │    │ Engine   │  │     │         │
│  │  └──────────────┘    └──────────────┘    └────┬─────┘  │     │         │
│  └───────────────────────────────────────────────┼─────────┘     │         │
│                                                   │               │         │
│                                    3. Match       │               │         │
│                                       Found       │               │         │
│                                                   ▼               │         │
│  ┌─────────────────────────────────────────────────────────┐     │         │
│  │                 Settlement Service                       │     │         │
│  │                                                          │     │ 5. Tx   │
│  │  ┌──────────────┐    ┌──────────────┐    ┌──────────┐  │     │   Hash  │
│  │  │ Trade        │───▶│   Batch      │───▶│ Submit   │──┼─────┘         │
│  │  │ Validator    │    │   Builder    │    │ Tx       │  │               │
│  │  └──────────────┘    └──────────────┘    └──────────┘  │               │
│  └─────────────────────────────────────────────────────────┘               │
│            │                                                                │
│            │ 4. Update                                                      │
│            ▼                                                                │
│  ┌─────────────────────────────────────────────────────────┐               │
│  │                     Database                             │               │
│  │                                                          │               │
│  │  ┌──────────────┐    ┌──────────────┐    ┌──────────┐  │               │
│  │  │   Orders     │    │   Markets    │    │  Users   │  │               │
│  │  │   Table      │    │   Table      │    │  Table   │  │               │
│  │  └──────────────┘    └──────────────┘    └──────────┘  │               │
│  └─────────────────────────────────────────────────────────┘               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 9.2 Component Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        COMPONENT ARCHITECTURE                                │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         PRESENTATION LAYER                             │  │
│  │                                                                        │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │  │
│  │  │   Market UI     │  │   Trading UI    │  │  Portfolio UI   │       │  │
│  │  │   Components    │  │   Components    │  │  Components     │       │  │
│  │  │                 │  │                 │  │                 │       │  │
│  │  │  - MarketList   │  │  - OrderBook    │  │  - Positions    │       │  │
│  │  │  - MarketCard   │  │  - TradePanel   │  │  - History      │       │  │
│  │  │  - PriceChart   │  │  - OrderForm    │  │  - PnL          │       │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│                                      ▼                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                          STATE LAYER                                   │  │
│  │                                                                        │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │  │
│  │  │   React Query   │  │     Zustand     │  │    WebSocket    │       │  │
│  │  │   (Server)      │  │    (Client)     │  │    (Real-time)  │       │  │
│  │  │                 │  │                 │  │                 │       │  │
│  │  │  - Markets      │  │  - UI State     │  │  - Order Book   │       │  │
│  │  │  - Orders       │  │  - Forms        │  │  - Trades       │       │  │
│  │  │  - Positions    │  │  - Preferences  │  │  - Prices       │       │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│                                      ▼                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         SERVICE LAYER                                  │  │
│  │                                                                        │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │  │
│  │  │   API Client    │  │   Web3 Client   │  │  Wallet Service │       │  │
│  │  │                 │  │                 │  │                 │       │  │
│  │  │  - REST calls   │  │  - Contract     │  │  - Connect      │       │  │
│  │  │  - Auth         │  │    interaction  │  │  - Sign         │       │  │
│  │  │  - Caching      │  │  - Events       │  │  - Transactions │       │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│                                      ▼                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        EXTERNAL SERVICES                               │  │
│  │                                                                        │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │  │
│  │  │  REST API       │  │  Polygon RPC    │  │  IPFS Gateway   │       │  │
│  │  │  (Backend)      │  │  (Blockchain)   │  │  (Storage)      │       │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 9.3 Trading Flow Sequence

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          TRADING FLOW SEQUENCE                               │
│                                                                              │
│   User          Frontend        API           Matching        Blockchain    │
│    │               │             │             Engine              │         │
│    │               │             │               │                 │         │
│    │  1. Connect   │             │               │                 │         │
│    │──────────────▶│             │               │                 │         │
│    │               │             │               │                 │         │
│    │  2. Sign msg  │             │               │                 │         │
│    │◀──────────────│             │               │                 │         │
│    │               │             │               │                 │         │
│    │  3. Submit    │             │               │                 │         │
│    │──────────────▶│             │               │                 │         │
│    │               │  4. Sign    │               │                 │         │
│    │               │  Order      │               │                 │         │
│    │               │────────────▶│               │                 │         │
│    │               │             │ 5. Validate   │                 │         │
│    │               │             │──────────────▶│                 │         │
│    │               │             │               │                 │         │
│    │               │             │  6. Match     │                 │         │
│    │               │             │◀──────────────│                 │         │
│    │               │             │               │                 │         │
│    │               │             │      7. Prepare Settlement      │         │
│    │               │             │─────────────────────────────────▶│        │
│    │               │             │               │                 │         │
│    │               │             │      8. Tx Hash                 │         │
│    │               │             │◀─────────────────────────────────│        │
│    │               │             │               │                 │         │
│    │               │ 9. Confirm  │               │                 │         │
│    │◀──────────────│◀────────────│               │                 │         │
│    │               │             │               │                 │         │
│    │               │      10. WebSocket: Trade Executed            │         │
│    │◀──────────────│◀────────────│               │                 │         │
│    │               │             │               │                 │         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Technology Stack

### 10.1 Complete Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Blockchain** | Polygon (Matic) | L2 scaling, low fees |
| **Smart Contracts** | Solidity ^0.8.19 | Contract development |
| **Contract Framework** | Hardhat / Foundry | Testing, deployment |
| **Frontend** | Next.js 14 | React framework with SSR |
| **State Management** | Zustand + React Query | Client & server state |
| **Styling** | Tailwind CSS | Utility-first CSS |
| **Web3 Integration** | wagmi + viem | Wallet connection |
| **Contract Interaction** | ethers.js v6 | Contract calls |
| **Backend** | Node.js / TypeScript | API server |
| **API Framework** | Express / Fastify | REST API |
| **WebSocket** | Socket.io | Real-time updates |
| **Database** | PostgreSQL | Relational data |
| **Cache** | Redis | Order book, sessions |
| **Message Queue** | RabbitMQ / Kafka | Event processing |
| **Storage** | IPFS | Decentralized storage |
| **Oracle** | UMA Protocol | Market resolution |
| **Monitoring** | DataDog / Grafana | System monitoring |

### 10.2 Development Tools

```json
// package.json (partial)
{
  "dependencies": {
    "next": "^14.0.0",
    "@tanstack/react-query": "^5.0.0",
    "zustand": "^4.4.0",
    "wagmi": "^2.0.0",
    "viem": "^2.0.0",
    "ethers": "^6.9.0",
    "socket.io-client": "^4.7.0",
    "tailwindcss": "^3.4.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "hardhat": "^2.19.0",
    "@nomicfoundation/hardhat-toolbox": "^4.0.0",
    "eslint": "^8.56.0",
    "prettier": "^3.2.0",
    "vitest": "^1.2.0"
  }
}
```

---

## 11. Security Considerations

### 11.1 Smart Contract Security

```solidity
// Security patterns implemented
contract SecureExchange is ReentrancyGuard, Pausable, Ownable {

    // 1. Reentrancy protection
    function fillOrder(Order calldata order) external nonReentrant {
        // ...
    }

    // 2. Integer overflow protection (Solidity 0.8+)
    // Automatic checks, no SafeMath needed

    // 3. Access control
    modifier onlyOperator() {
        require(msg.sender == operator, "Not operator");
        _;
    }

    // 4. Emergency pause
    function pause() external onlyOwner {
        _pause();
    }

    // 5. Signature validation
    function _validateSignature(
        Order calldata order,
        bytes calldata signature
    ) internal view returns (bool) {
        bytes32 orderHash = _hashOrder(order);
        address signer = ECDSA.recover(orderHash, signature);

        // Check signer matches maker
        require(signer == order.maker, "Invalid signer");

        // Check nonce
        require(order.nonce >= nonces[order.maker], "Stale nonce");

        // Check expiration
        require(block.timestamp < order.expiration, "Expired");

        return true;
    }

    // 6. Amount validation
    function _validateAmounts(Order calldata order) internal pure {
        require(order.makerAmount > 0, "Zero maker amount");
        require(order.takerAmount > 0, "Zero taker amount");
        require(order.feeRateBps <= MAX_FEE_BPS, "Fee too high");
    }
}
```

### 11.2 Security Checklist

| Category | Check | Status |
|----------|-------|--------|
| **Smart Contracts** | | |
| | Reentrancy guards | Required |
| | Integer overflow checks | Required |
| | Access control | Required |
| | Emergency pause | Required |
| | Formal verification | Recommended |
| | Multiple audits | Required |
| **API Security** | | |
| | Rate limiting | Required |
| | Input validation | Required |
| | SQL injection prevention | Required |
| | CORS configuration | Required |
| | API key authentication | Required |
| **Frontend Security** | | |
| | XSS prevention | Required |
| | CSRF protection | Required |
| | Secure wallet connection | Required |
| | Transaction verification UI | Required |
| **Infrastructure** | | |
| | DDoS protection | Required |
| | Key management (HSM) | Recommended |
| | Monitoring & alerting | Required |
| | Incident response plan | Required |

### 11.3 Audit Recommendations

1. **Pre-deployment**
   - Internal security review
   - External audit (Trail of Bits, OpenZeppelin, Consensys Diligence)
   - Bug bounty program setup

2. **Post-deployment**
   - Continuous monitoring
   - Gradual rollout with limits
   - Emergency response procedures

---

## 12. Scalability Patterns

### 12.1 Horizontal Scaling

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        HORIZONTAL SCALING ARCHITECTURE                       │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         Load Balancer                                │   │
│  │                    (AWS ALB / Cloudflare)                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│           ┌────────────────────────┼────────────────────────┐              │
│           │                        │                        │              │
│           ▼                        ▼                        ▼              │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐      │
│  │   API Server    │     │   API Server    │     │   API Server    │      │
│  │   Instance 1    │     │   Instance 2    │     │   Instance N    │      │
│  └────────┬────────┘     └────────┬────────┘     └────────┬────────┘      │
│           │                       │                        │               │
│           └───────────────────────┼────────────────────────┘               │
│                                   │                                        │
│                                   ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                     Message Queue (Kafka)                            │   │
│  │                                                                      │   │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                │   │
│  │   │  orders     │  │  matches    │  │  events     │                │   │
│  │   │  topic      │  │  topic      │  │  topic      │                │   │
│  │   └─────────────┘  └─────────────┘  └─────────────┘                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                   │                                        │
│           ┌───────────────────────┼───────────────────────┐               │
│           │                       │                       │               │
│           ▼                       ▼                       ▼               │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐       │
│  │    Matching     │    │   Settlement    │    │    Indexer      │       │
│  │    Engine       │    │    Service      │    │    Service      │       │
│  │   (Stateful)    │    │  (Stateless)    │    │  (Stateless)    │       │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘       │
│                                                                            │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 12.2 Caching Strategy

```typescript
// Redis caching for order books
class OrderBookCache {
  private redis: Redis;

  async getOrderBook(
    marketId: string,
    outcome: string
  ): Promise<OrderBook | null> {
    const key = `orderbook:${marketId}:${outcome}`;
    const cached = await this.redis.get(key);

    if (cached) {
      return JSON.parse(cached);
    }

    return null;
  }

  async setOrderBook(
    marketId: string,
    outcome: string,
    orderBook: OrderBook
  ): Promise<void> {
    const key = `orderbook:${marketId}:${outcome}`;

    // Short TTL since order books update frequently
    await this.redis.setex(key, 1, JSON.stringify(orderBook));
  }

  async updateOrderBook(
    marketId: string,
    outcome: string,
    update: OrderBookUpdate
  ): Promise<void> {
    // Use Lua script for atomic update
    const script = `
      local key = KEYS[1]
      local update = cjson.decode(ARGV[1])
      local book = cjson.decode(redis.call('GET', key) or '{}')

      -- Apply update
      if update.type == 'ADD' then
        -- Add order logic
      elseif update.type == 'REMOVE' then
        -- Remove order logic
      end

      redis.call('SETEX', key, 1, cjson.encode(book))
      return 'OK'
    `;

    await this.redis.eval(
      script,
      1,
      `orderbook:${marketId}:${outcome}`,
      JSON.stringify(update)
    );
  }
}
```

### 12.3 Performance Targets

| Metric | Target | Notes |
|--------|--------|-------|
| Order submission latency | < 50ms | 95th percentile |
| Order matching latency | < 10ms | In-memory matching |
| Settlement latency | < 30 seconds | Polygon block time |
| Order book updates | < 100ms | WebSocket push |
| API throughput | > 10,000 req/s | Per instance |
| WebSocket connections | > 100,000 | With Redis pub/sub |

---

## Appendix A: Contract Addresses (Mainnet)

```typescript
// contracts/addresses.ts
export const POLYGON_MAINNET = {
  CTF: '0x...', // Conditional Token Framework
  CTF_EXCHANGE: '0x...', // Binary market exchange
  NEGRISK_EXCHANGE: '0x...', // Multi-outcome exchange
  UMA_ADAPTER: '0x...', // Oracle adapter
  USDC: '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174',
  PROXY_FACTORY: '0x...',
};

export const POLYGON_MUMBAI = {
  CTF: '0x...',
  CTF_EXCHANGE: '0x...',
  NEGRISK_EXCHANGE: '0x...',
  UMA_ADAPTER: '0x...',
  USDC: '0x...',
  PROXY_FACTORY: '0x...',
};
```

---

## Appendix B: API Reference

### Markets API

```typescript
// GET /api/markets
interface MarketsResponse {
  markets: Market[];
  total: number;
  page: number;
  pageSize: number;
}

// GET /api/markets/:id
interface MarketResponse {
  market: Market;
  orderBook: {
    yes: OrderBook;
    no: OrderBook;
  };
  recentTrades: Trade[];
}

// Market interface
interface Market {
  id: string;
  question: string;
  description: string;
  category: string;
  endDate: string;
  resolved: boolean;
  outcome?: 'YES' | 'NO';
  volume24h: number;
  liquidity: number;
  yesPrice: number;
  noPrice: number;
}
```

### Orders API

```typescript
// POST /api/orders
interface PlaceOrderRequest {
  order: SignedOrder;
  signature: string;
}

interface PlaceOrderResponse {
  orderId: string;
  status: 'OPEN' | 'PARTIAL' | 'FILLED';
  filled: number;
  remaining: number;
}

// DELETE /api/orders/:id
interface CancelOrderResponse {
  orderId: string;
  status: 'CANCELLED';
}

// GET /api/orders?maker=0x...&marketId=...
interface OrdersResponse {
  orders: Order[];
}
```

---

## Appendix C: WebSocket Events

```typescript
// Client -> Server
interface SubscribeMessage {
  type: 'subscribe';
  channel: 'orderbook' | 'trades' | 'positions';
  marketId?: string;
  outcome?: 'YES' | 'NO';
}

// Server -> Client
interface OrderBookUpdateMessage {
  type: 'orderbook_update';
  marketId: string;
  outcome: 'YES' | 'NO';
  bids: PriceLevel[];
  asks: PriceLevel[];
  timestamp: number;
}

interface TradeMessage {
  type: 'trade';
  marketId: string;
  outcome: 'YES' | 'NO';
  price: number;
  size: number;
  side: 'BUY' | 'SELL';
  timestamp: number;
}

interface PositionUpdateMessage {
  type: 'position_update';
  marketId: string;
  outcome: 'YES' | 'NO';
  shares: number;
  avgPrice: number;
  unrealizedPnL: number;
}
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2024-01-XX | Architecture Team | Initial release |

---

*This document serves as the technical blueprint for implementing a Polymarket-like prediction market platform. All code examples are illustrative and should be adapted for production use with proper security reviews and audits.*
