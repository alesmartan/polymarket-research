# Smart Contract Architecture

This document provides comprehensive documentation for building a Polymarket-like prediction market platform using smart contracts on Ethereum/Polygon.

## Table of Contents

1. [Overview](#overview)
2. [Core Contracts](#core-contracts)
3. [Token Mechanics](#token-mechanics)
4. [Proxy Wallet System](#proxy-wallet-system)
5. [Implementation Guide](#implementation-guide)
6. [Security Considerations](#security-considerations)
7. [Testing Strategy](#testing-strategy)
8. [Deployment Checklist](#deployment-checklist)

---

## Overview

Polymarket's smart contract architecture is built on the **Conditional Token Framework (CTF)** originally developed by Gnosis. This framework enables the creation of prediction markets where outcomes are tokenized as tradeable assets.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         User Interface                               │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Off-Chain Order Book (CLOB)                       │
│              - Order matching - Price discovery                      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         On-Chain Layer                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │ CTF Exchange │  │ NegRisk      │  │ Conditional Token        │  │
│  │ (Binary)     │  │ Exchange     │  │ Framework (ERC-1155)     │  │
│  └──────────────┘  └──────────────┘  └──────────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │ UMA CTF      │  │ Proxy Wallet │  │ Collateral Token         │  │
│  │ Adapter      │  │ Factory      │  │ (USDC)                   │  │
│  └──────────────┘  └──────────────┘  └──────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Core Contracts

### 1. Conditional Token Framework (CTF)

The CTF is the foundation of the prediction market, implementing the ERC-1155 multi-token standard for outcome tokens.

#### Key Concepts

- **Condition**: A question with multiple possible outcomes
- **Position**: A user's stake in one or more outcomes
- **Outcome Slot**: A single possible outcome of a condition
- **Partition**: A grouping of outcome slots

#### Interface Definition

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";

/**
 * @title IConditionalTokens
 * @notice Interface for the Conditional Token Framework
 * @dev Based on Gnosis CTF - ERC-1155 standard for outcome tokens
 */
interface IConditionalTokens is IERC1155 {
    /// @notice Emitted when a condition is prepared
    event ConditionPreparation(
        bytes32 indexed conditionId,
        address indexed oracle,
        bytes32 indexed questionId,
        uint256 outcomeSlotCount
    );

    /// @notice Emitted when a condition is resolved
    event ConditionResolution(
        bytes32 indexed conditionId,
        address indexed oracle,
        bytes32 indexed questionId,
        uint256 outcomeSlotCount,
        uint256[] payoutNumerators
    );

    /// @notice Emitted when a position is split
    event PositionSplit(
        address indexed stakeholder,
        IERC20 collateralToken,
        bytes32 indexed parentCollectionId,
        bytes32 indexed conditionId,
        uint256[] partition,
        uint256 amount
    );

    /// @notice Emitted when positions are merged
    event PositionsMerge(
        address indexed stakeholder,
        IERC20 collateralToken,
        bytes32 indexed parentCollectionId,
        bytes32 indexed conditionId,
        uint256[] partition,
        uint256 amount
    );

    /// @notice Emitted when payout is redeemed
    event PayoutRedemption(
        address indexed redeemer,
        IERC20 indexed collateralToken,
        bytes32 indexed parentCollectionId,
        bytes32 conditionId,
        uint256[] indexSets,
        uint256 payout
    );

    /**
     * @notice Prepares a condition for outcome reporting
     * @param oracle The address that will report the outcome
     * @param questionId Unique identifier for the question
     * @param outcomeSlotCount Number of possible outcomes (2 for binary)
     */
    function prepareCondition(
        address oracle,
        bytes32 questionId,
        uint256 outcomeSlotCount
    ) external;

    /**
     * @notice Reports payouts for a condition
     * @param questionId The question identifier
     * @param payouts Array of payout values for each outcome
     */
    function reportPayouts(
        bytes32 questionId,
        uint256[] calldata payouts
    ) external;

    /**
     * @notice Splits a position into multiple outcome positions
     * @param collateralToken The collateral token (e.g., USDC)
     * @param parentCollectionId The parent collection (0 for root)
     * @param conditionId The condition to split on
     * @param partition Array of outcome index sets
     * @param amount Amount of collateral to split
     */
    function splitPosition(
        IERC20 collateralToken,
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint256[] calldata partition,
        uint256 amount
    ) external;

    /**
     * @notice Merges multiple outcome positions back into collateral
     * @param collateralToken The collateral token
     * @param parentCollectionId The parent collection
     * @param conditionId The condition to merge
     * @param partition Array of outcome index sets
     * @param amount Amount to merge
     */
    function mergePositions(
        IERC20 collateralToken,
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint256[] calldata partition,
        uint256 amount
    ) external;

    /**
     * @notice Redeems winning positions for collateral
     * @param collateralToken The collateral token
     * @param parentCollectionId The parent collection
     * @param conditionId The resolved condition
     * @param indexSets Outcome index sets to redeem
     */
    function redeemPositions(
        IERC20 collateralToken,
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint256[] calldata indexSets
    ) external;

    /**
     * @notice Gets the outcome slot count for a condition
     * @param conditionId The condition identifier
     * @return Number of outcome slots
     */
    function getOutcomeSlotCount(
        bytes32 conditionId
    ) external view returns (uint256);

    /**
     * @notice Gets the payout numerator for a condition outcome
     * @param conditionId The condition identifier
     * @param outcomeIndex The outcome index
     * @return The payout numerator
     */
    function payoutNumerators(
        bytes32 conditionId,
        uint256 outcomeIndex
    ) external view returns (uint256);

    /**
     * @notice Gets the payout denominator for a condition
     * @param conditionId The condition identifier
     * @return The payout denominator
     */
    function payoutDenominator(
        bytes32 conditionId
    ) external view returns (uint256);

    /**
     * @notice Calculates the condition ID
     * @param oracle The oracle address
     * @param questionId The question identifier
     * @param outcomeSlotCount Number of outcomes
     * @return The condition ID
     */
    function getConditionId(
        address oracle,
        bytes32 questionId,
        uint256 outcomeSlotCount
    ) external pure returns (bytes32);

    /**
     * @notice Calculates the collection ID
     * @param parentCollectionId Parent collection
     * @param conditionId The condition
     * @param indexSet The outcome index set
     * @return The collection ID
     */
    function getCollectionId(
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint256 indexSet
    ) external view returns (bytes32);

    /**
     * @notice Calculates the position ID (ERC-1155 token ID)
     * @param collateralToken The collateral token
     * @param collectionId The collection ID
     * @return The position ID
     */
    function getPositionId(
        IERC20 collateralToken,
        bytes32 collectionId
    ) external pure returns (uint256);
}
```

#### Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

/**
 * @title ConditionalTokens
 * @notice Implementation of the Conditional Token Framework
 * @dev Creates ERC-1155 tokens representing outcome positions in prediction markets
 */
contract ConditionalTokens is ERC1155, ReentrancyGuard {
    using SafeERC20 for IERC20;

    /// @notice Mapping from condition ID to outcome slot count
    mapping(bytes32 => uint256) public outcomeSlotCounts;

    /// @notice Mapping from condition ID to payout numerators
    mapping(bytes32 => uint256[]) public payoutNumerators;

    /// @notice Mapping from condition ID to payout denominator
    mapping(bytes32 => uint256) public payoutDenominator;

    // Events
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

    event PositionSplit(
        address indexed stakeholder,
        IERC20 collateralToken,
        bytes32 indexed parentCollectionId,
        bytes32 indexed conditionId,
        uint256[] partition,
        uint256 amount
    );

    event PositionsMerge(
        address indexed stakeholder,
        IERC20 collateralToken,
        bytes32 indexed parentCollectionId,
        bytes32 indexed conditionId,
        uint256[] partition,
        uint256 amount
    );

    event PayoutRedemption(
        address indexed redeemer,
        IERC20 indexed collateralToken,
        bytes32 indexed parentCollectionId,
        bytes32 conditionId,
        uint256[] indexSets,
        uint256 payout
    );

    constructor() ERC1155("") {}

    /**
     * @notice Prepares a condition for a prediction market
     * @param oracle Address authorized to report the outcome
     * @param questionId Unique identifier for the question
     * @param outcomeSlotCount Number of possible outcomes
     */
    function prepareCondition(
        address oracle,
        bytes32 questionId,
        uint256 outcomeSlotCount
    ) external {
        require(outcomeSlotCount >= 2, "Too few outcomes");
        require(outcomeSlotCount <= 256, "Too many outcomes");

        bytes32 conditionId = getConditionId(oracle, questionId, outcomeSlotCount);
        require(outcomeSlotCounts[conditionId] == 0, "Condition already prepared");

        outcomeSlotCounts[conditionId] = outcomeSlotCount;

        emit ConditionPreparation(conditionId, oracle, questionId, outcomeSlotCount);
    }

    /**
     * @notice Reports the outcome of a condition
     * @param questionId The question identifier
     * @param payouts Payout values for each outcome (numerators)
     */
    function reportPayouts(bytes32 questionId, uint256[] calldata payouts) external {
        uint256 outcomeSlotCount = payouts.length;
        require(outcomeSlotCount > 0, "Empty payouts");

        bytes32 conditionId = getConditionId(msg.sender, questionId, outcomeSlotCount);
        require(outcomeSlotCounts[conditionId] == outcomeSlotCount, "Condition not prepared");
        require(payoutDenominator[conditionId] == 0, "Already resolved");

        uint256 denominator = 0;
        for (uint256 i = 0; i < outcomeSlotCount; i++) {
            denominator += payouts[i];
        }
        require(denominator > 0, "Invalid payouts");

        payoutDenominator[conditionId] = denominator;
        payoutNumerators[conditionId] = payouts;

        emit ConditionResolution(
            conditionId,
            msg.sender,
            questionId,
            outcomeSlotCount,
            payouts
        );
    }

    /**
     * @notice Splits collateral into outcome tokens
     * @dev For binary markets, partition = [1, 2] splits into YES (index 0) and NO (index 1)
     */
    function splitPosition(
        IERC20 collateralToken,
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint256[] calldata partition,
        uint256 amount
    ) external nonReentrant {
        require(partition.length > 0, "Empty partition");
        uint256 outcomeSlotCount = outcomeSlotCounts[conditionId];
        require(outcomeSlotCount > 0, "Condition not prepared");

        // Validate partition covers all outcomes exactly once
        uint256 fullIndexSet = (1 << outcomeSlotCount) - 1;
        uint256 freeIndexSet = fullIndexSet;

        uint256[] memory positionIds = new uint256[](partition.length);

        for (uint256 i = 0; i < partition.length; i++) {
            uint256 indexSet = partition[i];
            require(indexSet > 0 && indexSet <= fullIndexSet, "Invalid index set");
            require((indexSet & freeIndexSet) == indexSet, "Partition overlap");
            freeIndexSet ^= indexSet;

            bytes32 collectionId = getCollectionId(parentCollectionId, conditionId, indexSet);
            positionIds[i] = getPositionId(collateralToken, collectionId);
        }

        if (freeIndexSet != 0) {
            // Partial split - requires parent position
            revert("Partial splits not supported");
        }

        // Transfer collateral from sender
        if (parentCollectionId == bytes32(0)) {
            collateralToken.safeTransferFrom(msg.sender, address(this), amount);
        } else {
            // Burn parent position
            uint256 parentPositionId = getPositionId(
                collateralToken,
                parentCollectionId
            );
            _burn(msg.sender, parentPositionId, amount);
        }

        // Mint outcome tokens
        for (uint256 i = 0; i < partition.length; i++) {
            _mint(msg.sender, positionIds[i], amount, "");
        }

        emit PositionSplit(
            msg.sender,
            collateralToken,
            parentCollectionId,
            conditionId,
            partition,
            amount
        );
    }

    /**
     * @notice Merges outcome tokens back into collateral
     */
    function mergePositions(
        IERC20 collateralToken,
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint256[] calldata partition,
        uint256 amount
    ) external nonReentrant {
        require(partition.length > 0, "Empty partition");
        uint256 outcomeSlotCount = outcomeSlotCounts[conditionId];
        require(outcomeSlotCount > 0, "Condition not prepared");

        uint256 fullIndexSet = (1 << outcomeSlotCount) - 1;
        uint256 freeIndexSet = fullIndexSet;

        uint256[] memory positionIds = new uint256[](partition.length);

        for (uint256 i = 0; i < partition.length; i++) {
            uint256 indexSet = partition[i];
            require(indexSet > 0 && indexSet <= fullIndexSet, "Invalid index set");
            require((indexSet & freeIndexSet) == indexSet, "Partition overlap");
            freeIndexSet ^= indexSet;

            bytes32 collectionId = getCollectionId(parentCollectionId, conditionId, indexSet);
            positionIds[i] = getPositionId(collateralToken, collectionId);
        }

        require(freeIndexSet == 0, "Incomplete partition");

        // Burn outcome tokens
        for (uint256 i = 0; i < partition.length; i++) {
            _burn(msg.sender, positionIds[i], amount);
        }

        // Return collateral
        if (parentCollectionId == bytes32(0)) {
            collateralToken.safeTransfer(msg.sender, amount);
        } else {
            uint256 parentPositionId = getPositionId(
                collateralToken,
                parentCollectionId
            );
            _mint(msg.sender, parentPositionId, amount, "");
        }

        emit PositionsMerge(
            msg.sender,
            collateralToken,
            parentCollectionId,
            conditionId,
            partition,
            amount
        );
    }

    /**
     * @notice Redeems winning positions after resolution
     */
    function redeemPositions(
        IERC20 collateralToken,
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint256[] calldata indexSets
    ) external nonReentrant {
        uint256 denominator = payoutDenominator[conditionId];
        require(denominator > 0, "Condition not resolved");

        uint256 outcomeSlotCount = outcomeSlotCounts[conditionId];
        uint256 totalPayout = 0;

        for (uint256 i = 0; i < indexSets.length; i++) {
            uint256 indexSet = indexSets[i];
            require(indexSet > 0, "Invalid index set");

            bytes32 collectionId = getCollectionId(parentCollectionId, conditionId, indexSet);
            uint256 positionId = getPositionId(collateralToken, collectionId);
            uint256 balance = balanceOf(msg.sender, positionId);

            if (balance > 0) {
                // Calculate payout for this position
                uint256 payoutNumerator = 0;
                for (uint256 j = 0; j < outcomeSlotCount; j++) {
                    if ((indexSet >> j) & 1 == 1) {
                        payoutNumerator += payoutNumerators[conditionId][j];
                    }
                }

                uint256 payout = (balance * payoutNumerator) / denominator;
                totalPayout += payout;

                // Burn the position
                _burn(msg.sender, positionId, balance);
            }
        }

        require(totalPayout > 0, "Nothing to redeem");

        // Transfer collateral
        if (parentCollectionId == bytes32(0)) {
            collateralToken.safeTransfer(msg.sender, totalPayout);
        } else {
            uint256 parentPositionId = getPositionId(
                collateralToken,
                parentCollectionId
            );
            _mint(msg.sender, parentPositionId, totalPayout, "");
        }

        emit PayoutRedemption(
            msg.sender,
            collateralToken,
            parentCollectionId,
            conditionId,
            indexSets,
            totalPayout
        );
    }

    // View functions

    function getConditionId(
        address oracle,
        bytes32 questionId,
        uint256 outcomeSlotCount
    ) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(oracle, questionId, outcomeSlotCount));
    }

    function getCollectionId(
        bytes32 parentCollectionId,
        bytes32 conditionId,
        uint256 indexSet
    ) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(parentCollectionId, conditionId, indexSet));
    }

    function getPositionId(
        IERC20 collateralToken,
        bytes32 collectionId
    ) public pure returns (uint256) {
        return uint256(keccak256(abi.encodePacked(collateralToken, collectionId)));
    }

    function getOutcomeSlotCount(bytes32 conditionId) external view returns (uint256) {
        return outcomeSlotCounts[conditionId];
    }
}
```

---

### 2. CTF Exchange Contract

The CTF Exchange handles trading of binary market outcome tokens. It receives matched orders from the off-chain CLOB (Central Limit Order Book) and settles them on-chain.

#### Interface Definition

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title ICTFExchange
 * @notice Interface for the binary outcome exchange
 * @dev Handles YES/NO market trading with on-chain settlement
 */
interface ICTFExchange {
    /// @notice Order structure for the exchange
    struct Order {
        /// @notice Unique salt to prevent replay
        uint256 salt;
        /// @notice Order creator address
        address maker;
        /// @notice Address to receive the proceeds
        address signer;
        /// @notice Address allowed to fill (0 = anyone)
        address taker;
        /// @notice Token ID of the outcome position
        uint256 tokenId;
        /// @notice Amount of outcome tokens
        uint256 makerAmount;
        /// @notice Amount of collateral
        uint256 takerAmount;
        /// @notice Expiration timestamp
        uint256 expiration;
        /// @notice Nonce for cancellation
        uint256 nonce;
        /// @notice Fee rate in basis points
        uint256 feeRateBps;
        /// @notice Side of the order (0 = BUY, 1 = SELL)
        uint8 side;
        /// @notice Signature type (0 = EOA, 1 = POLY_PROXY, 2 = POLY_GNOSIS_SAFE)
        uint8 signatureType;
    }

    /// @notice Emitted when an order is filled
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

    /// @notice Emitted when orders are cancelled
    event OrdersCancelled(address indexed maker, uint256[] orderHashes);

    /**
     * @notice Fills a single order
     * @param order The order to fill
     * @param fillAmount Amount to fill
     * @param signature Order signature
     */
    function fillOrder(
        Order calldata order,
        uint256 fillAmount,
        bytes calldata signature
    ) external;

    /**
     * @notice Fills multiple orders atomically
     * @param orders Array of orders
     * @param fillAmounts Amounts to fill each order
     * @param signatures Order signatures
     */
    function fillOrders(
        Order[] calldata orders,
        uint256[] calldata fillAmounts,
        bytes[] calldata signatures
    ) external;

    /**
     * @notice Matches a buy and sell order
     * @param buyOrder The buy order
     * @param sellOrder The sell order
     * @param buySignature Buy order signature
     * @param sellSignature Sell order signature
     * @param fillAmount Amount to match
     */
    function matchOrders(
        Order calldata buyOrder,
        Order calldata sellOrder,
        bytes calldata buySignature,
        bytes calldata sellSignature,
        uint256 fillAmount
    ) external;

    /**
     * @notice Cancels orders by incrementing nonce
     */
    function incrementNonce() external;

    /**
     * @notice Cancels specific orders
     * @param orders Orders to cancel
     */
    function cancelOrders(Order[] calldata orders) external;

    /**
     * @notice Gets the hash of an order
     * @param order The order
     * @return The order hash
     */
    function hashOrder(Order calldata order) external view returns (bytes32);

    /**
     * @notice Gets the filled amount of an order
     * @param orderHash The order hash
     * @return Amount filled
     */
    function getOrderStatus(bytes32 orderHash) external view returns (uint256);
}
```

#### Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import "@openzeppelin/contracts/utils/cryptography/EIP712.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title CTFExchange
 * @notice Exchange contract for trading binary outcome tokens
 * @dev Receives matched orders from off-chain CLOB and settles on-chain
 */
contract CTFExchange is ReentrancyGuard, EIP712, Ownable {
    using SafeERC20 for IERC20;
    using ECDSA for bytes32;

    // Constants
    uint256 public constant BPS_DIVISOR = 10000;
    uint8 public constant SIDE_BUY = 0;
    uint8 public constant SIDE_SELL = 1;

    // State
    IERC1155 public immutable ctf;
    IERC20 public immutable collateral;
    address public feeCollector;

    /// @notice Mapping of order hash to filled amount
    mapping(bytes32 => uint256) public orderFills;

    /// @notice Mapping of maker address to nonce
    mapping(address => uint256) public nonces;

    /// @notice Mapping of order hash to cancelled status
    mapping(bytes32 => bool) public cancelledOrders;

    // Order struct
    struct Order {
        uint256 salt;
        address maker;
        address signer;
        address taker;
        uint256 tokenId;
        uint256 makerAmount;
        uint256 takerAmount;
        uint256 expiration;
        uint256 nonce;
        uint256 feeRateBps;
        uint8 side;
        uint8 signatureType;
    }

    // EIP-712 type hash
    bytes32 public constant ORDER_TYPEHASH = keccak256(
        "Order(uint256 salt,address maker,address signer,address taker,uint256 tokenId,uint256 makerAmount,uint256 takerAmount,uint256 expiration,uint256 nonce,uint256 feeRateBps,uint8 side,uint8 signatureType)"
    );

    // Events
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

    event OrdersCancelled(address indexed maker, bytes32[] orderHashes);
    event NonceIncremented(address indexed maker, uint256 newNonce);

    constructor(
        address _ctf,
        address _collateral,
        address _feeCollector
    ) EIP712("CTFExchange", "1") {
        ctf = IERC1155(_ctf);
        collateral = IERC20(_collateral);
        feeCollector = _feeCollector;
    }

    /**
     * @notice Fills a single order
     * @param order The order to fill
     * @param fillAmount Amount of the maker's asset to trade
     * @param signature EIP-712 signature
     */
    function fillOrder(
        Order calldata order,
        uint256 fillAmount,
        bytes calldata signature
    ) external nonReentrant {
        _fillOrder(order, fillAmount, signature, msg.sender);
    }

    /**
     * @notice Fills multiple orders
     */
    function fillOrders(
        Order[] calldata orders,
        uint256[] calldata fillAmounts,
        bytes[] calldata signatures
    ) external nonReentrant {
        require(
            orders.length == fillAmounts.length &&
            orders.length == signatures.length,
            "Array length mismatch"
        );

        for (uint256 i = 0; i < orders.length; i++) {
            _fillOrder(orders[i], fillAmounts[i], signatures[i], msg.sender);
        }
    }

    /**
     * @notice Matches a buy order with a sell order
     * @dev Used by the operator to settle matched orders from CLOB
     */
    function matchOrders(
        Order calldata buyOrder,
        Order calldata sellOrder,
        bytes calldata buySignature,
        bytes calldata sellSignature,
        uint256 fillAmount
    ) external nonReentrant {
        require(buyOrder.side == SIDE_BUY, "Invalid buy order");
        require(sellOrder.side == SIDE_SELL, "Invalid sell order");
        require(buyOrder.tokenId == sellOrder.tokenId, "Token mismatch");

        // Verify both orders
        bytes32 buyHash = _hashOrder(buyOrder);
        bytes32 sellHash = _hashOrder(sellOrder);

        _verifyOrder(buyOrder, buyHash, buySignature);
        _verifyOrder(sellOrder, sellHash, sellSignature);

        // Calculate fill amounts
        uint256 buyMakerAmount = fillAmount;
        uint256 buyTakerAmount = (buyMakerAmount * buyOrder.takerAmount) / buyOrder.makerAmount;

        uint256 sellMakerAmount = (fillAmount * sellOrder.makerAmount) / sellOrder.takerAmount;
        uint256 sellTakerAmount = fillAmount;

        // Update fills
        orderFills[buyHash] += buyMakerAmount;
        orderFills[sellHash] += sellMakerAmount;

        require(orderFills[buyHash] <= buyOrder.makerAmount, "Buy overfill");
        require(orderFills[sellHash] <= sellOrder.makerAmount, "Sell overfill");

        // Calculate fees
        uint256 buyFee = (buyTakerAmount * buyOrder.feeRateBps) / BPS_DIVISOR;
        uint256 sellFee = (sellTakerAmount * sellOrder.feeRateBps) / BPS_DIVISOR;

        // Execute transfers
        // Buyer: sends collateral, receives tokens
        collateral.safeTransferFrom(buyOrder.maker, sellOrder.maker, buyTakerAmount - sellFee);
        collateral.safeTransferFrom(buyOrder.maker, feeCollector, buyFee + sellFee);

        // Seller: sends tokens, receives collateral
        ctf.safeTransferFrom(sellOrder.maker, buyOrder.maker, buyOrder.tokenId, fillAmount, "");

        emit OrderFilled(buyHash, buyOrder.maker, sellOrder.maker, 0, buyOrder.tokenId, buyTakerAmount, fillAmount, buyFee);
        emit OrderFilled(sellHash, sellOrder.maker, buyOrder.maker, sellOrder.tokenId, 0, fillAmount, sellTakerAmount - sellFee, sellFee);
    }

    /**
     * @notice Increments the sender's nonce, cancelling all orders with lower nonces
     */
    function incrementNonce() external {
        nonces[msg.sender]++;
        emit NonceIncremented(msg.sender, nonces[msg.sender]);
    }

    /**
     * @notice Cancels specific orders
     */
    function cancelOrders(Order[] calldata orders) external {
        bytes32[] memory hashes = new bytes32[](orders.length);

        for (uint256 i = 0; i < orders.length; i++) {
            require(orders[i].maker == msg.sender, "Not order maker");
            bytes32 orderHash = _hashOrder(orders[i]);
            cancelledOrders[orderHash] = true;
            hashes[i] = orderHash;
        }

        emit OrdersCancelled(msg.sender, hashes);
    }

    /**
     * @notice Gets the EIP-712 hash of an order
     */
    function hashOrder(Order calldata order) external view returns (bytes32) {
        return _hashOrder(order);
    }

    /**
     * @notice Gets the fill status of an order
     */
    function getOrderStatus(bytes32 orderHash) external view returns (uint256) {
        return orderFills[orderHash];
    }

    // Internal functions

    function _fillOrder(
        Order calldata order,
        uint256 fillAmount,
        bytes calldata signature,
        address taker
    ) internal {
        bytes32 orderHash = _hashOrder(order);
        _verifyOrder(order, orderHash, signature);

        // Check taker restriction
        if (order.taker != address(0)) {
            require(order.taker == taker, "Invalid taker");
        }

        // Calculate amounts
        uint256 makerFillAmount;
        uint256 takerFillAmount;

        if (order.side == SIDE_BUY) {
            // Maker is buying tokens, taker provides tokens
            takerFillAmount = fillAmount;
            makerFillAmount = (fillAmount * order.makerAmount) / order.takerAmount;
        } else {
            // Maker is selling tokens, taker provides collateral
            makerFillAmount = fillAmount;
            takerFillAmount = (fillAmount * order.takerAmount) / order.makerAmount;
        }

        // Update fills
        orderFills[orderHash] += makerFillAmount;
        require(orderFills[orderHash] <= order.makerAmount, "Order overfilled");

        // Calculate fee
        uint256 fee = (takerFillAmount * order.feeRateBps) / BPS_DIVISOR;

        // Execute transfers based on order side
        if (order.side == SIDE_BUY) {
            // Maker buys: transfer collateral from maker, tokens from taker
            collateral.safeTransferFrom(order.maker, taker, makerFillAmount - fee);
            collateral.safeTransferFrom(order.maker, feeCollector, fee);
            ctf.safeTransferFrom(taker, order.maker, order.tokenId, takerFillAmount, "");
        } else {
            // Maker sells: transfer tokens from maker, collateral from taker
            ctf.safeTransferFrom(order.maker, taker, order.tokenId, makerFillAmount, "");
            collateral.safeTransferFrom(taker, order.maker, takerFillAmount - fee);
            collateral.safeTransferFrom(taker, feeCollector, fee);
        }

        emit OrderFilled(
            orderHash,
            order.maker,
            taker,
            order.side == SIDE_BUY ? 0 : order.tokenId,
            order.side == SIDE_BUY ? order.tokenId : 0,
            makerFillAmount,
            takerFillAmount,
            fee
        );
    }

    function _verifyOrder(
        Order calldata order,
        bytes32 orderHash,
        bytes calldata signature
    ) internal view {
        require(block.timestamp < order.expiration, "Order expired");
        require(!cancelledOrders[orderHash], "Order cancelled");
        require(order.nonce >= nonces[order.maker], "Invalid nonce");

        bytes32 digest = _hashTypedDataV4(orderHash);
        address signer = digest.recover(signature);
        require(signer == order.signer, "Invalid signature");
    }

    function _hashOrder(Order calldata order) internal pure returns (bytes32) {
        return keccak256(abi.encode(
            ORDER_TYPEHASH,
            order.salt,
            order.maker,
            order.signer,
            order.taker,
            order.tokenId,
            order.makerAmount,
            order.takerAmount,
            order.expiration,
            order.nonce,
            order.feeRateBps,
            order.side,
            order.signatureType
        ));
    }

    // Admin functions

    function setFeeCollector(address _feeCollector) external onlyOwner {
        feeCollector = _feeCollector;
    }
}
```

---

### 3. NegRisk CTF Exchange

The NegRisk Exchange extends the CTF Exchange to support multi-outcome markets with more complex structures.

#### Key Differences from Binary Exchange

| Feature | CTF Exchange | NegRisk Exchange |
|---------|--------------|------------------|
| Outcomes | Binary (YES/NO) | Multiple (3+) |
| Token Pairs | YES/NO per market | Multiple outcomes |
| Settlement | Simple binary | Complex payout calculation |
| Use Case | Simple predictions | Elections, competitions |

#### Interface

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title INegRiskCTFExchange
 * @notice Interface for multi-outcome prediction markets
 * @dev Extends CTF Exchange for markets with more than 2 outcomes
 */
interface INegRiskCTFExchange {
    /// @notice Multi-outcome market structure
    struct Market {
        bytes32 conditionId;
        uint256 outcomeCount;
        uint256[] tokenIds;
        bool resolved;
        uint256 winningOutcome;
    }

    /// @notice Creates a new multi-outcome market
    function createMarket(
        bytes32 questionId,
        uint256 outcomeCount,
        address oracle
    ) external returns (bytes32 marketId);

    /// @notice Gets the complement token ID (for hedging)
    function getComplementTokenId(
        bytes32 marketId,
        uint256 outcomeIndex
    ) external view returns (uint256);

    /// @notice Converts outcome tokens to their complement
    function convertToComplement(
        bytes32 marketId,
        uint256 outcomeIndex,
        uint256 amount
    ) external;

    /// @notice Gets all token IDs for a market
    function getMarketTokenIds(
        bytes32 marketId
    ) external view returns (uint256[] memory);
}
```

---

### 4. UMA CTF Adapter

The UMA CTF Adapter bridges prediction markets with the UMA (Universal Market Access) oracle for decentralized dispute resolution.

#### How UMA Oracle Resolution Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    Market Resolution Flow                        │
└─────────────────────────────────────────────────────────────────┘

1. Market Creator → Prepares condition with UmaCtfAdapter as oracle

2. Resolution Request:
   ┌──────────────┐     ┌─────────────────┐     ┌──────────────┐
   │ Anyone can   │────▶│ UMA Optimistic  │────▶│ Proposed     │
   │ request      │     │ Oracle          │     │ outcome      │
   └──────────────┘     └─────────────────┘     └──────────────┘

3. Challenge Period (typically 2 hours):
   ┌──────────────┐                             ┌──────────────┐
   │ No dispute   │────────────────────────────▶│ Finalized    │
   └──────────────┘                             └──────────────┘
        OR
   ┌──────────────┐     ┌─────────────────┐     ┌──────────────┐
   │ Dispute      │────▶│ UMA DVM Vote    │────▶│ Final result │
   │ raised       │     │ (48-96 hours)   │     │              │
   └──────────────┘     └─────────────────┘     └──────────────┘

4. Adapter → Reports payout to CTF → Users redeem positions
```

#### Interface

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title IUmaCtfAdapter
 * @notice Adapter that bridges CTF markets with UMA's Optimistic Oracle
 * @dev Handles market resolution through UMA's dispute resolution mechanism
 */
interface IUmaCtfAdapter {
    /// @notice Market initialization data
    struct MarketData {
        bytes32 questionId;
        bytes ancillaryData;
        uint256 requestTimestamp;
        uint256 reward;
        bool initialized;
        bool resolved;
        int256 resolvedPrice;
    }

    /// @notice Emitted when a market is initialized
    event MarketInitialized(
        bytes32 indexed questionId,
        bytes ancillaryData,
        uint256 requestTimestamp,
        uint256 reward
    );

    /// @notice Emitted when resolution is requested
    event ResolutionRequested(
        bytes32 indexed questionId,
        uint256 timestamp
    );

    /// @notice Emitted when a market is resolved
    event MarketResolved(
        bytes32 indexed questionId,
        int256 resolvedPrice,
        uint256[] payouts
    );

    /**
     * @notice Initializes a new market for resolution
     * @param questionId Unique identifier matching CTF condition
     * @param ancillaryData Description of the question for UMA voters
     * @param reward Reward for the resolution proposer
     * @return True if initialization successful
     */
    function initializeMarket(
        bytes32 questionId,
        bytes calldata ancillaryData,
        uint256 reward
    ) external returns (bool);

    /**
     * @notice Requests resolution from UMA oracle
     * @param questionId The question to resolve
     */
    function requestResolution(bytes32 questionId) external;

    /**
     * @notice Resolves the market after UMA oracle settlement
     * @param questionId The question to finalize
     */
    function resolve(bytes32 questionId) external;

    /**
     * @notice Gets the resolution status
     * @param questionId The question identifier
     * @return resolved Whether the market is resolved
     * @return price The resolved price (0 = NO, 1 = YES, 0.5 = INVALID)
     */
    function getResolution(
        bytes32 questionId
    ) external view returns (bool resolved, int256 price);

    /**
     * @notice Checks if a market can be resolved
     * @param questionId The question identifier
     * @return True if resolution is available
     */
    function canResolve(bytes32 questionId) external view returns (bool);
}
```

#### Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

// UMA interfaces
interface IOptimisticOracleV2 {
    function requestPrice(
        bytes32 identifier,
        uint256 timestamp,
        bytes calldata ancillaryData,
        IERC20 currency,
        uint256 reward
    ) external returns (uint256);

    function settleAndGetPrice(
        bytes32 identifier,
        uint256 timestamp,
        bytes calldata ancillaryData
    ) external returns (int256);

    function hasPrice(
        address requester,
        bytes32 identifier,
        uint256 timestamp,
        bytes calldata ancillaryData
    ) external view returns (bool);

    function getRequest(
        address requester,
        bytes32 identifier,
        uint256 timestamp,
        bytes calldata ancillaryData
    ) external view returns (Request memory);

    struct Request {
        address proposer;
        address disputer;
        IERC20 currency;
        bool settled;
        int256 proposedPrice;
        int256 resolvedPrice;
        uint256 expirationTime;
        uint256 reward;
        uint256 finalFee;
        uint256 bond;
        uint256 customLiveness;
    }
}

interface IConditionalTokens {
    function reportPayouts(bytes32 questionId, uint256[] calldata payouts) external;
}

/**
 * @title UmaCtfAdapter
 * @notice Bridges Polymarket CTF with UMA's Optimistic Oracle
 */
contract UmaCtfAdapter is ReentrancyGuard, Ownable {
    using SafeERC20 for IERC20;

    // Constants
    bytes32 public constant YES_OR_NO_IDENTIFIER = "YES_OR_NO_QUERY";
    int256 public constant PRICE_YES = 1e18;
    int256 public constant PRICE_NO = 0;
    int256 public constant PRICE_INVALID = 5e17; // 0.5 for invalid/ambiguous

    // State
    IOptimisticOracleV2 public immutable optimisticOracle;
    IConditionalTokens public immutable conditionalTokens;
    IERC20 public immutable rewardToken;

    struct MarketData {
        bytes32 questionId;
        bytes ancillaryData;
        uint256 requestTimestamp;
        uint256 reward;
        bool initialized;
        bool resolved;
        int256 resolvedPrice;
    }

    mapping(bytes32 => MarketData) public markets;

    // Events
    event MarketInitialized(
        bytes32 indexed questionId,
        bytes ancillaryData,
        uint256 requestTimestamp,
        uint256 reward
    );

    event ResolutionRequested(bytes32 indexed questionId, uint256 timestamp);

    event MarketResolved(
        bytes32 indexed questionId,
        int256 resolvedPrice,
        uint256[] payouts
    );

    constructor(
        address _optimisticOracle,
        address _conditionalTokens,
        address _rewardToken
    ) {
        optimisticOracle = IOptimisticOracleV2(_optimisticOracle);
        conditionalTokens = IConditionalTokens(_conditionalTokens);
        rewardToken = IERC20(_rewardToken);
    }

    /**
     * @notice Initializes a market for UMA oracle resolution
     * @param questionId Must match the CTF condition questionId
     * @param ancillaryData Human-readable question for UMA voters
     * @param reward Reward for the resolution proposer
     */
    function initializeMarket(
        bytes32 questionId,
        bytes calldata ancillaryData,
        uint256 reward
    ) external nonReentrant returns (bool) {
        require(!markets[questionId].initialized, "Already initialized");
        require(ancillaryData.length > 0, "Empty ancillary data");

        markets[questionId] = MarketData({
            questionId: questionId,
            ancillaryData: ancillaryData,
            requestTimestamp: 0,
            reward: reward,
            initialized: true,
            resolved: false,
            resolvedPrice: 0
        });

        emit MarketInitialized(questionId, ancillaryData, 0, reward);
        return true;
    }

    /**
     * @notice Requests price resolution from UMA
     * @param questionId The market to resolve
     */
    function requestResolution(bytes32 questionId) external nonReentrant {
        MarketData storage market = markets[questionId];
        require(market.initialized, "Market not initialized");
        require(market.requestTimestamp == 0, "Already requested");

        uint256 timestamp = block.timestamp;
        market.requestTimestamp = timestamp;

        // Transfer reward if set
        if (market.reward > 0) {
            rewardToken.safeTransferFrom(msg.sender, address(this), market.reward);
            rewardToken.safeApprove(address(optimisticOracle), market.reward);
        }

        // Request price from UMA
        optimisticOracle.requestPrice(
            YES_OR_NO_IDENTIFIER,
            timestamp,
            market.ancillaryData,
            rewardToken,
            market.reward
        );

        emit ResolutionRequested(questionId, timestamp);
    }

    /**
     * @notice Resolves the market after UMA settlement
     * @param questionId The market to finalize
     */
    function resolve(bytes32 questionId) external nonReentrant {
        MarketData storage market = markets[questionId];
        require(market.initialized, "Market not initialized");
        require(market.requestTimestamp > 0, "Not requested");
        require(!market.resolved, "Already resolved");

        // Get settled price from UMA
        int256 resolvedPrice = optimisticOracle.settleAndGetPrice(
            YES_OR_NO_IDENTIFIER,
            market.requestTimestamp,
            market.ancillaryData
        );

        market.resolved = true;
        market.resolvedPrice = resolvedPrice;

        // Convert UMA price to CTF payouts
        uint256[] memory payouts = new uint256[](2);

        if (resolvedPrice == PRICE_YES) {
            // YES wins
            payouts[0] = 1; // YES outcome
            payouts[1] = 0; // NO outcome
        } else if (resolvedPrice == PRICE_NO) {
            // NO wins
            payouts[0] = 0;
            payouts[1] = 1;
        } else {
            // Invalid/ambiguous - split 50/50
            payouts[0] = 1;
            payouts[1] = 1;
        }

        // Report to CTF
        conditionalTokens.reportPayouts(questionId, payouts);

        emit MarketResolved(questionId, resolvedPrice, payouts);
    }

    /**
     * @notice Checks if a market can be resolved
     */
    function canResolve(bytes32 questionId) external view returns (bool) {
        MarketData storage market = markets[questionId];
        if (!market.initialized || market.resolved || market.requestTimestamp == 0) {
            return false;
        }

        return optimisticOracle.hasPrice(
            address(this),
            YES_OR_NO_IDENTIFIER,
            market.requestTimestamp,
            market.ancillaryData
        );
    }

    /**
     * @notice Gets resolution status
     */
    function getResolution(bytes32 questionId) external view returns (bool, int256) {
        MarketData storage market = markets[questionId];
        return (market.resolved, market.resolvedPrice);
    }
}
```

---

## Token Mechanics

### Share Pricing Model

Outcome shares in Polymarket trade between **$0.01 and $1.00**, where the price directly represents the market's probability assessment.

```
Price Range: $0.01 ─────────────────────────────────────── $1.00
             │                                              │
             ▼                                              ▼
        1% probability                              100% probability
```

#### Price-Probability Relationship

| Share Price | Implied Probability | Example |
|-------------|--------------------:|---------|
| $0.01 | 1% | Highly unlikely event |
| $0.25 | 25% | Underdog candidate |
| $0.50 | 50% | Coin flip / uncertain |
| $0.70 | 70% | Likely outcome |
| $0.95 | 95% | Near certainty |

#### Resolution Outcomes

```solidity
// After market resolution:
// - Winning shares → $1.00 each (100% of collateral returned)
// - Losing shares → $0.00 each (no collateral returned)

// Example: User holds 100 YES shares at $0.70 average cost
// If YES wins:
//   Redemption = 100 * $1.00 = $100
//   Profit = $100 - $70 = $30

// If NO wins:
//   Redemption = 100 * $0.00 = $0
//   Loss = $70
```

### Complete Set Invariant

A fundamental property of binary markets is that **YES + NO = $1.00** always.

```solidity
/**
 * @notice The complete set invariant ensures market integrity
 *
 * For any binary market:
 *   Price(YES) + Price(NO) = $1.00
 *
 * This enables arbitrage-free pricing and minting/redemption:
 *   - Mint: $1.00 USDC → 1 YES token + 1 NO token
 *   - Redeem: 1 YES + 1 NO → $1.00 USDC
 */

// Minting creates equal YES and NO tokens
function mintCompleteSet(uint256 amount) external {
    // Transfer 1 USDC per complete set
    usdc.transferFrom(msg.sender, address(this), amount);

    // Mint both YES and NO tokens
    _mint(msg.sender, YES_TOKEN_ID, amount, "");
    _mint(msg.sender, NO_TOKEN_ID, amount, "");
}

// Redeeming burns both tokens for collateral
function redeemCompleteSet(uint256 amount) external {
    // Burn both YES and NO tokens
    _burn(msg.sender, YES_TOKEN_ID, amount);
    _burn(msg.sender, NO_TOKEN_ID, amount);

    // Return collateral
    usdc.transfer(msg.sender, amount);
}
```

### Token ID Calculation

```solidity
/**
 * @notice Token IDs are deterministically calculated from market parameters
 * @dev This ensures consistency across the system
 */

// Condition ID from oracle, question, and outcome count
bytes32 conditionId = keccak256(abi.encodePacked(
    oracle,      // UMA adapter address
    questionId,  // Unique question identifier
    uint256(2)   // Binary = 2 outcomes
));

// Collection IDs for each outcome
bytes32 yesCollectionId = keccak256(abi.encodePacked(
    bytes32(0),  // Root collection (no parent)
    conditionId,
    uint256(1)   // Index set for YES (binary: 01)
));

bytes32 noCollectionId = keccak256(abi.encodePacked(
    bytes32(0),
    conditionId,
    uint256(2)   // Index set for NO (binary: 10)
));

// Final token IDs (ERC-1155)
uint256 yesTokenId = uint256(keccak256(abi.encodePacked(usdc, yesCollectionId)));
uint256 noTokenId = uint256(keccak256(abi.encodePacked(usdc, noCollectionId)));
```

---

## Proxy Wallet System

Polymarket uses a proxy wallet architecture to improve user experience while maintaining security.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    User's External Wallet                        │
│              (MetaMask, WalletConnect, etc.)                     │
│                         │                                        │
│                         │ Controls (1-of-1 multisig)            │
│                         ▼                                        │
│              ┌─────────────────────┐                            │
│              │   Proxy Wallet      │                            │
│              │   (Per User)        │                            │
│              │                     │                            │
│              │  - ERC-1155 tokens  │◄──── Outcome positions     │
│              │  - ERC-20 (USDC)    │◄──── Collateral            │
│              │  - Approvals set    │◄──── Pre-approved exchange │
│              └─────────────────────┘                            │
│                         │                                        │
│                         │ Atomic transactions                    │
│                         ▼                                        │
│              ┌─────────────────────┐                            │
│              │   CTF Exchange      │                            │
│              └─────────────────────┘                            │
└─────────────────────────────────────────────────────────────────┘
```

### Benefits

1. **Atomic Transactions**: Multiple operations in single tx
2. **Gas Optimization**: Batch operations save gas
3. **Pre-set Approvals**: No repeated approval transactions
4. **Recovery Options**: Potential for social recovery
5. **Meta-transactions**: Gasless UX possibilities

### Proxy Wallet Factory

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/proxy/Clones.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title ProxyWalletFactory
 * @notice Factory for creating user proxy wallets
 * @dev Uses minimal proxy pattern (EIP-1167) for gas efficiency
 */
contract ProxyWalletFactory is Ownable {
    using Clones for address;

    /// @notice Implementation contract for proxies
    address public implementation;

    /// @notice Mapping of user to their proxy wallet
    mapping(address => address) public userProxies;

    /// @notice Exchange contract address for pre-approval
    address public exchange;

    /// @notice Collateral token for pre-approval
    address public collateralToken;

    /// @notice CTF contract for pre-approval
    address public ctfContract;

    event ProxyCreated(address indexed user, address indexed proxy);
    event ImplementationUpdated(address indexed newImplementation);

    constructor(
        address _implementation,
        address _exchange,
        address _collateralToken,
        address _ctfContract
    ) {
        implementation = _implementation;
        exchange = _exchange;
        collateralToken = _collateralToken;
        ctfContract = _ctfContract;
    }

    /**
     * @notice Creates a proxy wallet for the caller
     * @return proxy The created proxy address
     */
    function createProxy() external returns (address proxy) {
        require(userProxies[msg.sender] == address(0), "Proxy exists");

        // Clone implementation
        proxy = implementation.clone();

        // Initialize the proxy
        IProxyWallet(proxy).initialize(
            msg.sender,
            exchange,
            collateralToken,
            ctfContract
        );

        userProxies[msg.sender] = proxy;
        emit ProxyCreated(msg.sender, proxy);
    }

    /**
     * @notice Creates a proxy wallet for a specific user (admin only)
     * @param user The user to create proxy for
     * @return proxy The created proxy address
     */
    function createProxyFor(address user) external onlyOwner returns (address proxy) {
        require(userProxies[user] == address(0), "Proxy exists");

        proxy = implementation.clone();
        IProxyWallet(proxy).initialize(user, exchange, collateralToken, ctfContract);

        userProxies[user] = proxy;
        emit ProxyCreated(user, proxy);
    }

    /**
     * @notice Gets or creates a proxy for the caller
     */
    function getOrCreateProxy() external returns (address) {
        if (userProxies[msg.sender] != address(0)) {
            return userProxies[msg.sender];
        }
        return this.createProxy();
    }

    /**
     * @notice Computes the deterministic address for a user's proxy
     */
    function computeProxyAddress(address user) external view returns (address) {
        bytes32 salt = keccak256(abi.encodePacked(user));
        return implementation.predictDeterministicAddress(salt);
    }

    function updateImplementation(address _implementation) external onlyOwner {
        implementation = _implementation;
        emit ImplementationUpdated(_implementation);
    }
}

interface IProxyWallet {
    function initialize(
        address owner,
        address exchange,
        address collateralToken,
        address ctfContract
    ) external;
}
```

### Proxy Wallet Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";
import "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Holder.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/proxy/utils/Initializable.sol";

/**
 * @title ProxyWallet
 * @notice User proxy wallet for prediction market interactions
 * @dev 1-of-1 multisig controlled by user's external wallet
 */
contract ProxyWallet is ERC1155Holder, ReentrancyGuard, Initializable {
    using SafeERC20 for IERC20;

    /// @notice Owner of this proxy wallet
    address public owner;

    /// @notice Authorized operators (exchange contracts)
    mapping(address => bool) public operators;

    /// @notice Whether the wallet has been initialized
    bool private _initialized;

    // Events
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    event OperatorUpdated(address indexed operator, bool authorized);
    event Executed(address indexed target, uint256 value, bytes data);

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    modifier onlyOwnerOrOperator() {
        require(msg.sender == owner || operators[msg.sender], "Not authorized");
        _;
    }

    /**
     * @notice Initializes the proxy wallet
     * @param _owner The wallet owner
     * @param _exchange Exchange contract to approve
     * @param _collateralToken Collateral token to approve
     * @param _ctfContract CTF contract to approve
     */
    function initialize(
        address _owner,
        address _exchange,
        address _collateralToken,
        address _ctfContract
    ) external initializer {
        require(!_initialized, "Already initialized");
        _initialized = true;

        owner = _owner;

        // Set up approvals for seamless trading
        IERC20(_collateralToken).safeApprove(_exchange, type(uint256).max);
        IERC1155(_ctfContract).setApprovalForAll(_exchange, true);

        operators[_exchange] = true;

        emit OwnershipTransferred(address(0), _owner);
        emit OperatorUpdated(_exchange, true);
    }

    /**
     * @notice Executes a transaction from the proxy
     * @param target Contract to call
     * @param value ETH value to send
     * @param data Calldata
     */
    function execute(
        address target,
        uint256 value,
        bytes calldata data
    ) external onlyOwner nonReentrant returns (bytes memory) {
        (bool success, bytes memory result) = target.call{value: value}(data);
        require(success, "Execution failed");

        emit Executed(target, value, data);
        return result;
    }

    /**
     * @notice Executes multiple transactions atomically
     * @param targets Contracts to call
     * @param values ETH values
     * @param datas Calldatas
     */
    function executeBatch(
        address[] calldata targets,
        uint256[] calldata values,
        bytes[] calldata datas
    ) external onlyOwner nonReentrant returns (bytes[] memory results) {
        require(
            targets.length == values.length && targets.length == datas.length,
            "Length mismatch"
        );

        results = new bytes[](targets.length);

        for (uint256 i = 0; i < targets.length; i++) {
            (bool success, bytes memory result) = targets[i].call{value: values[i]}(datas[i]);
            require(success, "Batch execution failed");
            results[i] = result;

            emit Executed(targets[i], values[i], datas[i]);
        }
    }

    /**
     * @notice Withdraws ERC20 tokens to owner
     * @param token Token to withdraw
     * @param amount Amount to withdraw (0 = all)
     */
    function withdrawERC20(IERC20 token, uint256 amount) external onlyOwner {
        uint256 balance = token.balanceOf(address(this));
        uint256 withdrawAmount = amount == 0 ? balance : amount;
        require(withdrawAmount <= balance, "Insufficient balance");

        token.safeTransfer(owner, withdrawAmount);
    }

    /**
     * @notice Withdraws ERC1155 tokens to owner
     * @param token Token contract
     * @param tokenId Token ID
     * @param amount Amount to withdraw (0 = all)
     */
    function withdrawERC1155(
        IERC1155 token,
        uint256 tokenId,
        uint256 amount
    ) external onlyOwner {
        uint256 balance = token.balanceOf(address(this), tokenId);
        uint256 withdrawAmount = amount == 0 ? balance : amount;
        require(withdrawAmount <= balance, "Insufficient balance");

        token.safeTransferFrom(address(this), owner, tokenId, withdrawAmount, "");
    }

    /**
     * @notice Updates operator authorization
     */
    function setOperator(address operator, bool authorized) external onlyOwner {
        operators[operator] = authorized;
        emit OperatorUpdated(operator, authorized);
    }

    /**
     * @notice Transfers ownership
     */
    function transferOwnership(address newOwner) external onlyOwner {
        require(newOwner != address(0), "Invalid owner");
        emit OwnershipTransferred(owner, newOwner);
        owner = newOwner;
    }

    /**
     * @notice Approves a token for a spender
     */
    function approveERC20(
        IERC20 token,
        address spender,
        uint256 amount
    ) external onlyOwner {
        token.safeApprove(spender, amount);
    }

    /**
     * @notice Sets approval for ERC1155
     */
    function setApprovalForAllERC1155(
        IERC1155 token,
        address operator,
        bool approved
    ) external onlyOwner {
        token.setApprovalForAll(operator, approved);
    }

    // Allow receiving ETH
    receive() external payable {}
}
```

---

## Implementation Guide

### Required Contracts to Build

For a complete Polymarket-like prediction market, you need to implement:

#### 1. Market Factory Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/Pausable.sol";

/**
 * @title MarketFactory
 * @notice Factory for creating prediction markets
 */
contract MarketFactory is AccessControl, Pausable {
    bytes32 public constant MARKET_CREATOR_ROLE = keccak256("MARKET_CREATOR_ROLE");

    IConditionalTokens public immutable ctf;
    address public immutable oracle;
    IERC20 public immutable collateral;

    struct Market {
        bytes32 questionId;
        bytes32 conditionId;
        string question;
        uint256 createdAt;
        uint256 resolutionTime;
        address creator;
        bool resolved;
    }

    /// @notice All created markets
    mapping(bytes32 => Market) public markets;

    /// @notice List of market IDs
    bytes32[] public marketIds;

    event MarketCreated(
        bytes32 indexed marketId,
        bytes32 indexed conditionId,
        string question,
        uint256 resolutionTime,
        address indexed creator
    );

    constructor(
        address _ctf,
        address _oracle,
        address _collateral
    ) {
        ctf = IConditionalTokens(_ctf);
        oracle = _oracle;
        collateral = IERC20(_collateral);

        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MARKET_CREATOR_ROLE, msg.sender);
    }

    /**
     * @notice Creates a new binary prediction market
     * @param question The question being predicted
     * @param resolutionTime When the market can be resolved
     * @return marketId The unique market identifier
     */
    function createBinaryMarket(
        string calldata question,
        uint256 resolutionTime
    ) external onlyRole(MARKET_CREATOR_ROLE) whenNotPaused returns (bytes32 marketId) {
        require(resolutionTime > block.timestamp, "Resolution time must be future");
        require(bytes(question).length > 0, "Empty question");

        // Generate unique question ID
        bytes32 questionId = keccak256(abi.encodePacked(
            question,
            resolutionTime,
            block.timestamp,
            msg.sender
        ));

        // Prepare condition in CTF
        ctf.prepareCondition(oracle, questionId, 2);

        bytes32 conditionId = ctf.getConditionId(oracle, questionId, 2);

        // Store market data
        marketId = conditionId;
        markets[marketId] = Market({
            questionId: questionId,
            conditionId: conditionId,
            question: question,
            createdAt: block.timestamp,
            resolutionTime: resolutionTime,
            creator: msg.sender,
            resolved: false
        });

        marketIds.push(marketId);

        emit MarketCreated(
            marketId,
            conditionId,
            question,
            resolutionTime,
            msg.sender
        );
    }

    /**
     * @notice Gets market token IDs
     * @param marketId The market to query
     * @return yesTokenId Token ID for YES outcome
     * @return noTokenId Token ID for NO outcome
     */
    function getMarketTokenIds(bytes32 marketId) external view returns (
        uint256 yesTokenId,
        uint256 noTokenId
    ) {
        Market storage market = markets[marketId];
        require(market.createdAt > 0, "Market not found");

        // YES = index set 1 (binary: 01)
        bytes32 yesCollectionId = ctf.getCollectionId(bytes32(0), market.conditionId, 1);
        yesTokenId = ctf.getPositionId(collateral, yesCollectionId);

        // NO = index set 2 (binary: 10)
        bytes32 noCollectionId = ctf.getCollectionId(bytes32(0), market.conditionId, 2);
        noTokenId = ctf.getPositionId(collateral, noCollectionId);
    }

    /**
     * @notice Gets all market IDs
     */
    function getAllMarkets() external view returns (bytes32[] memory) {
        return marketIds;
    }

    /**
     * @notice Gets market count
     */
    function getMarketCount() external view returns (uint256) {
        return marketIds.length;
    }

    // Admin functions
    function pause() external onlyRole(DEFAULT_ADMIN_ROLE) {
        _pause();
    }

    function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) {
        _unpause();
    }
}
```

#### 2. Complete Contract Architecture Summary

```
contracts/
├── core/
│   ├── ConditionalTokens.sol      # ERC-1155 outcome tokens
│   ├── CTFExchange.sol            # Binary market exchange
│   └── NegRiskExchange.sol        # Multi-outcome exchange
│
├── oracle/
│   ├── UmaCtfAdapter.sol          # UMA oracle integration
│   └── interfaces/
│       └── IOptimisticOracle.sol  # UMA interface
│
├── factory/
│   ├── MarketFactory.sol          # Market creation
│   └── ProxyWalletFactory.sol     # User wallet factory
│
├── proxy/
│   └── ProxyWallet.sol            # User proxy wallet
│
├── interfaces/
│   ├── IConditionalTokens.sol
│   ├── ICTFExchange.sol
│   ├── INegRiskExchange.sol
│   └── IUmaCtfAdapter.sol
│
└── libraries/
    ├── OrderLib.sol               # Order encoding/decoding
    └── MathLib.sol                # Safe math operations
```

---

## Security Considerations

### 1. Reentrancy Protection

```solidity
/**
 * @notice All state-changing functions must be protected
 * @dev Use OpenZeppelin's ReentrancyGuard
 */

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureExchange is ReentrancyGuard {
    // WRONG: Vulnerable to reentrancy
    function unsafeFill(Order calldata order) external {
        // External call before state update
        ctf.safeTransferFrom(order.maker, msg.sender, tokenId, amount, "");
        orderFills[orderHash] += amount; // State updated after external call
    }

    // CORRECT: Protected with nonReentrant modifier
    function safeFill(Order calldata order) external nonReentrant {
        // State updated first (Checks-Effects-Interactions pattern)
        orderFills[orderHash] += amount;

        // External call after state update
        ctf.safeTransferFrom(order.maker, msg.sender, tokenId, amount, "");
    }
}
```

### 2. Access Control

```solidity
/**
 * @notice Role-based access control for sensitive operations
 */

import "@openzeppelin/contracts/access/AccessControl.sol";

contract SecureMarketFactory is AccessControl {
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    bytes32 public constant MARKET_CREATOR_ROLE = keccak256("MARKET_CREATOR_ROLE");
    bytes32 public constant OPERATOR_ROLE = keccak256("OPERATOR_ROLE");

    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(ADMIN_ROLE, msg.sender);
    }

    // Only admins can update critical parameters
    function updateOracle(address newOracle) external onlyRole(ADMIN_ROLE) {
        require(newOracle != address(0), "Invalid oracle");
        oracle = newOracle;
    }

    // Only authorized creators can make markets
    function createMarket(string calldata question)
        external
        onlyRole(MARKET_CREATOR_ROLE)
    {
        // Market creation logic
    }

    // Operators can execute matched orders
    function executeMatch(Order calldata buy, Order calldata sell)
        external
        onlyRole(OPERATOR_ROLE)
    {
        // Order matching logic
    }
}
```

### 3. Oracle Manipulation Risks

```solidity
/**
 * @notice Protections against oracle manipulation
 */

contract SecureOracleAdapter {
    /// @notice Minimum bond required for disputes
    uint256 public constant MIN_BOND = 1000e6; // 1000 USDC

    /// @notice Minimum dispute period
    uint256 public constant MIN_DISPUTE_PERIOD = 2 hours;

    /// @notice Maximum markets per question (prevent spam)
    uint256 public constant MAX_MARKETS_PER_QUESTION = 1;

    mapping(bytes32 => uint256) public questionMarketCount;

    /**
     * @notice Time-weighted resolution for manipulation resistance
     * @dev Prevents last-minute oracle attacks
     */
    function requestResolution(bytes32 questionId) external {
        Market storage market = markets[questionId];

        // Cannot resolve before resolution time
        require(block.timestamp >= market.resolutionTime, "Too early");

        // Grace period to allow for disputes
        require(
            block.timestamp <= market.resolutionTime + 7 days,
            "Resolution window closed"
        );
    }

    /**
     * @notice Emergency pause for suspicious oracle behavior
     */
    function emergencyPause(bytes32 questionId) external onlyRole(EMERGENCY_ROLE) {
        markets[questionId].paused = true;
        emit MarketPaused(questionId, msg.sender);
    }
}
```

### 4. Flash Loan Attack Prevention

```solidity
/**
 * @notice Protections against flash loan attacks
 */

contract FlashLoanProtected {
    /// @notice Block number of last interaction per user
    mapping(address => uint256) public lastInteractionBlock;

    /// @notice Prevents same-block interactions
    modifier noFlashLoan() {
        require(
            lastInteractionBlock[msg.sender] < block.number,
            "Flash loan detected"
        );
        lastInteractionBlock[msg.sender] = block.number;
        _;
    }

    /**
     * @notice Large trades require multi-block execution
     */
    function executeLargeTrade(Order calldata order) external noFlashLoan {
        uint256 tradeValue = calculateTradeValue(order);

        if (tradeValue > LARGE_TRADE_THRESHOLD) {
            // Require time delay for large trades
            require(
                block.timestamp >= pendingTrades[msg.sender].timestamp + DELAY,
                "Large trade delay not met"
            );
        }

        // Execute trade
    }

    /**
     * @notice Snapshot-based voting for governance decisions
     * @dev Uses historical balances to prevent flash loan governance attacks
     */
    function vote(uint256 proposalId, bool support) external {
        uint256 snapshotBlock = proposals[proposalId].snapshotBlock;
        uint256 votingPower = getHistoricalBalance(msg.sender, snapshotBlock);

        // Vote with historical balance
    }
}
```

### 5. Input Validation

```solidity
/**
 * @notice Comprehensive input validation
 */

contract ValidatedExchange {
    uint256 public constant MAX_FEE_BPS = 500; // 5% max fee
    uint256 public constant MIN_ORDER_SIZE = 1e6; // 1 USDC minimum
    uint256 public constant MAX_ORDER_SIZE = 1000000e6; // 1M USDC maximum

    function validateOrder(Order calldata order) internal view {
        // Check order hasn't expired
        require(block.timestamp < order.expiration, "Order expired");

        // Check order size bounds
        require(order.makerAmount >= MIN_ORDER_SIZE, "Order too small");
        require(order.makerAmount <= MAX_ORDER_SIZE, "Order too large");

        // Check fee is reasonable
        require(order.feeRateBps <= MAX_FEE_BPS, "Fee too high");

        // Check addresses are valid
        require(order.maker != address(0), "Invalid maker");
        require(order.signer != address(0), "Invalid signer");

        // Check token exists
        require(validTokenIds[order.tokenId], "Invalid token");

        // Check nonce is valid
        require(order.nonce >= nonces[order.maker], "Invalid nonce");

        // Check order isn't cancelled
        bytes32 orderHash = hashOrder(order);
        require(!cancelledOrders[orderHash], "Order cancelled");
    }
}
```

### 6. Signature Security

```solidity
/**
 * @notice Secure signature verification
 */

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import "@openzeppelin/contracts/utils/cryptography/EIP712.sol";

contract SecureSignatures is EIP712 {
    using ECDSA for bytes32;

    // Signature types
    uint8 public constant SIG_TYPE_EOA = 0;
    uint8 public constant SIG_TYPE_EIP1271 = 1;
    uint8 public constant SIG_TYPE_PROXY = 2;

    /**
     * @notice Verifies signature based on type
     */
    function verifySignature(
        bytes32 orderHash,
        address signer,
        bytes calldata signature,
        uint8 signatureType
    ) internal view returns (bool) {
        bytes32 digest = _hashTypedDataV4(orderHash);

        if (signatureType == SIG_TYPE_EOA) {
            // Standard EOA signature
            address recovered = digest.recover(signature);
            return recovered == signer;
        } else if (signatureType == SIG_TYPE_EIP1271) {
            // Smart contract signature (EIP-1271)
            return IERC1271(signer).isValidSignature(digest, signature) == 0x1626ba7e;
        } else if (signatureType == SIG_TYPE_PROXY) {
            // Proxy wallet signature
            address proxyOwner = IProxyWallet(signer).owner();
            address recovered = digest.recover(signature);
            return recovered == proxyOwner;
        }

        return false;
    }

    /**
     * @notice Prevents signature replay across chains
     */
    function _domainSeparatorV4() internal view override returns (bytes32) {
        return keccak256(abi.encode(
            keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
            keccak256(bytes("CTFExchange")),
            keccak256(bytes("1")),
            block.chainid,
            address(this)
        ));
    }
}
```

---

## Testing Strategy

### 1. Unit Tests

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "forge-std/Test.sol";
import "../src/ConditionalTokens.sol";
import "../src/CTFExchange.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MockUSDC is ERC20 {
    constructor() ERC20("Mock USDC", "USDC") {
        _mint(msg.sender, 1000000e6);
    }

    function mint(address to, uint256 amount) external {
        _mint(to, amount);
    }

    function decimals() public pure override returns (uint8) {
        return 6;
    }
}

contract ConditionalTokensTest is Test {
    ConditionalTokens public ctf;
    MockUSDC public usdc;

    address public oracle = address(0x1);
    address public alice = address(0x2);
    address public bob = address(0x3);

    bytes32 public questionId = keccak256("Will ETH reach $10k by end of 2025?");
    bytes32 public conditionId;

    function setUp() public {
        ctf = new ConditionalTokens();
        usdc = new MockUSDC();

        // Fund test accounts
        usdc.mint(alice, 10000e6);
        usdc.mint(bob, 10000e6);

        // Prepare condition
        ctf.prepareCondition(oracle, questionId, 2);
        conditionId = ctf.getConditionId(oracle, questionId, 2);

        // Approve CTF
        vm.prank(alice);
        usdc.approve(address(ctf), type(uint256).max);

        vm.prank(bob);
        usdc.approve(address(ctf), type(uint256).max);
    }

    function testPrepareCondition() public {
        bytes32 newQuestionId = keccak256("New question");

        ctf.prepareCondition(oracle, newQuestionId, 2);

        bytes32 newConditionId = ctf.getConditionId(oracle, newQuestionId, 2);
        assertEq(ctf.getOutcomeSlotCount(newConditionId), 2);
    }

    function testSplitPosition() public {
        uint256[] memory partition = new uint256[](2);
        partition[0] = 1; // YES
        partition[1] = 2; // NO

        uint256 amount = 100e6;

        vm.prank(alice);
        ctf.splitPosition(
            IERC20(address(usdc)),
            bytes32(0),
            conditionId,
            partition,
            amount
        );

        // Check YES token balance
        bytes32 yesCollectionId = ctf.getCollectionId(bytes32(0), conditionId, 1);
        uint256 yesTokenId = ctf.getPositionId(IERC20(address(usdc)), yesCollectionId);
        assertEq(ctf.balanceOf(alice, yesTokenId), amount);

        // Check NO token balance
        bytes32 noCollectionId = ctf.getCollectionId(bytes32(0), conditionId, 2);
        uint256 noTokenId = ctf.getPositionId(IERC20(address(usdc)), noCollectionId);
        assertEq(ctf.balanceOf(alice, noTokenId), amount);

        // Check USDC was transferred
        assertEq(usdc.balanceOf(alice), 10000e6 - amount);
    }

    function testMergePositions() public {
        // First split
        uint256[] memory partition = new uint256[](2);
        partition[0] = 1;
        partition[1] = 2;
        uint256 amount = 100e6;

        vm.prank(alice);
        ctf.splitPosition(IERC20(address(usdc)), bytes32(0), conditionId, partition, amount);

        uint256 balanceBefore = usdc.balanceOf(alice);

        // Then merge
        vm.prank(alice);
        ctf.mergePositions(IERC20(address(usdc)), bytes32(0), conditionId, partition, amount);

        // Check USDC returned
        assertEq(usdc.balanceOf(alice), balanceBefore + amount);

        // Check tokens burned
        bytes32 yesCollectionId = ctf.getCollectionId(bytes32(0), conditionId, 1);
        uint256 yesTokenId = ctf.getPositionId(IERC20(address(usdc)), yesCollectionId);
        assertEq(ctf.balanceOf(alice, yesTokenId), 0);
    }

    function testRedeemWinningPosition() public {
        // Split positions
        uint256[] memory partition = new uint256[](2);
        partition[0] = 1;
        partition[1] = 2;
        uint256 amount = 100e6;

        vm.prank(alice);
        ctf.splitPosition(IERC20(address(usdc)), bytes32(0), conditionId, partition, amount);

        // Resolve: YES wins
        uint256[] memory payouts = new uint256[](2);
        payouts[0] = 1; // YES wins
        payouts[1] = 0; // NO loses

        vm.prank(oracle);
        ctf.reportPayouts(questionId, payouts);

        // Redeem YES position
        uint256[] memory indexSets = new uint256[](1);
        indexSets[0] = 1; // YES

        uint256 balanceBefore = usdc.balanceOf(alice);

        vm.prank(alice);
        ctf.redeemPositions(IERC20(address(usdc)), bytes32(0), conditionId, indexSets);

        // Should receive full amount
        assertEq(usdc.balanceOf(alice), balanceBefore + amount);
    }

    function testCannotRedeemLosingPosition() public {
        // Split positions
        uint256[] memory partition = new uint256[](2);
        partition[0] = 1;
        partition[1] = 2;
        uint256 amount = 100e6;

        vm.prank(alice);
        ctf.splitPosition(IERC20(address(usdc)), bytes32(0), conditionId, partition, amount);

        // Resolve: YES wins, NO loses
        uint256[] memory payouts = new uint256[](2);
        payouts[0] = 1;
        payouts[1] = 0;

        vm.prank(oracle);
        ctf.reportPayouts(questionId, payouts);

        // Try to redeem NO position (should revert)
        uint256[] memory indexSets = new uint256[](1);
        indexSets[0] = 2; // NO

        vm.prank(alice);
        vm.expectRevert("Nothing to redeem");
        ctf.redeemPositions(IERC20(address(usdc)), bytes32(0), conditionId, indexSets);
    }

    function testFuzz_SplitAndMerge(uint256 amount) public {
        // Bound amount to reasonable range
        amount = bound(amount, 1e6, 1000000e6);

        // Mint enough tokens
        usdc.mint(alice, amount);

        uint256[] memory partition = new uint256[](2);
        partition[0] = 1;
        partition[1] = 2;

        uint256 balanceBefore = usdc.balanceOf(alice);

        vm.startPrank(alice);
        ctf.splitPosition(IERC20(address(usdc)), bytes32(0), conditionId, partition, amount);
        ctf.mergePositions(IERC20(address(usdc)), bytes32(0), conditionId, partition, amount);
        vm.stopPrank();

        // Balance should be unchanged after split + merge
        assertEq(usdc.balanceOf(alice), balanceBefore);
    }
}
```

### 2. Integration Tests

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "forge-std/Test.sol";
import "../src/ConditionalTokens.sol";
import "../src/CTFExchange.sol";
import "../src/MarketFactory.sol";
import "../src/UmaCtfAdapter.sol";

contract IntegrationTest is Test {
    ConditionalTokens public ctf;
    CTFExchange public exchange;
    MarketFactory public factory;
    UmaCtfAdapter public adapter;
    MockUSDC public usdc;

    address public admin = address(0x1);
    address public alice = address(0x2);
    address public bob = address(0x3);
    address public operator = address(0x4);

    function setUp() public {
        vm.startPrank(admin);

        // Deploy core contracts
        usdc = new MockUSDC();
        ctf = new ConditionalTokens();

        // Deploy adapter (mock UMA oracle for testing)
        adapter = new UmaCtfAdapter(
            address(new MockOptimisticOracle()),
            address(ctf),
            address(usdc)
        );

        // Deploy factory
        factory = new MarketFactory(
            address(ctf),
            address(adapter),
            address(usdc)
        );

        // Deploy exchange
        exchange = new CTFExchange(
            address(ctf),
            address(usdc),
            admin // fee collector
        );

        vm.stopPrank();

        // Setup user accounts
        usdc.mint(alice, 10000e6);
        usdc.mint(bob, 10000e6);

        vm.prank(alice);
        usdc.approve(address(ctf), type(uint256).max);
        vm.prank(alice);
        usdc.approve(address(exchange), type(uint256).max);
        vm.prank(alice);
        ctf.setApprovalForAll(address(exchange), true);

        vm.prank(bob);
        usdc.approve(address(ctf), type(uint256).max);
        vm.prank(bob);
        usdc.approve(address(exchange), type(uint256).max);
        vm.prank(bob);
        ctf.setApprovalForAll(address(exchange), true);
    }

    function testFullMarketLifecycle() public {
        // 1. Create market
        vm.prank(admin);
        factory.grantRole(factory.MARKET_CREATOR_ROLE(), admin);

        vm.prank(admin);
        bytes32 marketId = factory.createBinaryMarket(
            "Will ETH reach $10k by end of 2025?",
            block.timestamp + 365 days
        );

        // 2. Get token IDs
        (uint256 yesTokenId, uint256 noTokenId) = factory.getMarketTokenIds(marketId);

        // 3. Alice mints positions
        bytes32 conditionId = markets[marketId].conditionId;
        uint256[] memory partition = new uint256[](2);
        partition[0] = 1;
        partition[1] = 2;

        vm.prank(alice);
        ctf.splitPosition(IERC20(address(usdc)), bytes32(0), conditionId, partition, 1000e6);

        // 4. Alice sells NO tokens to Bob via exchange
        // ... order creation and matching logic

        // 5. Market resolves (YES wins)
        vm.warp(block.timestamp + 365 days + 1);

        // ... resolution through UMA adapter

        // 6. Winners redeem
        uint256[] memory indexSets = new uint256[](1);
        indexSets[0] = 1;

        vm.prank(alice);
        ctf.redeemPositions(IERC20(address(usdc)), bytes32(0), conditionId, indexSets);

        // 7. Verify final balances
        // ...
    }
}
```

### 3. Mainnet Fork Testing

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "forge-std/Test.sol";

/**
 * @notice Fork tests against live Polygon/Ethereum state
 * @dev Run with: forge test --fork-url $POLYGON_RPC_URL
 */
contract MainnetForkTest is Test {
    // Live Polymarket addresses (Polygon)
    address constant CTF = 0x4D97DCd97eC945f40cF65F87097ACe5EA0476045;
    address constant USDC = 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174;
    address constant EXCHANGE = 0x4bFb41d5B3570DeFd03C39a9A4D8dE6Bd8B8982E;

    IConditionalTokens public ctf;
    IERC20 public usdc;

    // Whale address for impersonation
    address constant USDC_WHALE = 0xF977814e90dA44bFA03b6295A0616a897441aceC;

    function setUp() public {
        // Fork Polygon mainnet
        vm.createSelectFork(vm.envString("POLYGON_RPC_URL"));

        ctf = IConditionalTokens(CTF);
        usdc = IERC20(USDC);
    }

    function testForkInteractWithLiveMarket() public {
        // Get a live market's condition ID
        bytes32 conditionId = 0x...; // Real condition ID

        // Check outcome slot count
        uint256 outcomes = ctf.getOutcomeSlotCount(conditionId);
        assertEq(outcomes, 2);
    }

    function testForkMintPosition() public {
        address testUser = address(0x1234);

        // Fund test user with USDC from whale
        vm.prank(USDC_WHALE);
        usdc.transfer(testUser, 1000e6);

        // Approve CTF
        vm.prank(testUser);
        usdc.approve(address(ctf), type(uint256).max);

        // Get live condition
        bytes32 conditionId = 0x...; // Real condition ID

        // Split position
        uint256[] memory partition = new uint256[](2);
        partition[0] = 1;
        partition[1] = 2;

        vm.prank(testUser);
        ctf.splitPosition(usdc, bytes32(0), conditionId, partition, 100e6);

        // Verify positions minted
        // ...
    }
}
```

### 4. Test Coverage Checklist

```
## Unit Test Coverage

### ConditionalTokens
- [ ] prepareCondition - success
- [ ] prepareCondition - duplicate condition reverts
- [ ] prepareCondition - invalid outcome count reverts
- [ ] splitPosition - success
- [ ] splitPosition - insufficient balance reverts
- [ ] splitPosition - invalid partition reverts
- [ ] mergePositions - success
- [ ] mergePositions - incomplete partition reverts
- [ ] reportPayouts - success (oracle only)
- [ ] reportPayouts - unauthorized reverts
- [ ] reportPayouts - already resolved reverts
- [ ] redeemPositions - winning position
- [ ] redeemPositions - losing position (no payout)
- [ ] redeemPositions - partial payout (invalid resolution)

### CTFExchange
- [ ] fillOrder - buy order success
- [ ] fillOrder - sell order success
- [ ] fillOrder - expired order reverts
- [ ] fillOrder - cancelled order reverts
- [ ] fillOrder - invalid signature reverts
- [ ] fillOrder - overfill reverts
- [ ] matchOrders - success
- [ ] matchOrders - token mismatch reverts
- [ ] cancelOrders - success
- [ ] incrementNonce - success
- [ ] Fee calculation accuracy

### MarketFactory
- [ ] createBinaryMarket - success
- [ ] createBinaryMarket - unauthorized reverts
- [ ] createBinaryMarket - past resolution time reverts
- [ ] getMarketTokenIds - correct IDs returned

### ProxyWallet
- [ ] initialize - success
- [ ] execute - owner only
- [ ] executeBatch - atomic execution
- [ ] withdrawERC20 - success
- [ ] withdrawERC1155 - success
- [ ] transferOwnership - success

## Integration Tests
- [ ] Full market lifecycle
- [ ] Multi-user trading scenario
- [ ] Market resolution flow
- [ ] Emergency pause/unpause

## Fuzz Tests
- [ ] Split/merge any valid amount
- [ ] Order fill any valid amount
- [ ] Fee calculation precision

## Fork Tests
- [ ] Interaction with live markets
- [ ] Gas estimation accuracy
```

---

## Deployment Checklist

### Pre-Deployment

```markdown
## Code Review
- [ ] Internal audit completed
- [ ] External audit completed
- [ ] All audit findings addressed
- [ ] Test coverage > 95%

## Configuration
- [ ] Constructor parameters documented
- [ ] Access control roles defined
- [ ] Fee parameters set correctly
- [ ] Oracle addresses verified

## Security
- [ ] Reentrancy guards on all external functions
- [ ] Access control on admin functions
- [ ] Input validation comprehensive
- [ ] No floating pragma (use exact version)
- [ ] Dependencies pinned to specific versions
```

### Deployment Script

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "forge-std/Script.sol";
import "../src/ConditionalTokens.sol";
import "../src/CTFExchange.sol";
import "../src/MarketFactory.sol";
import "../src/UmaCtfAdapter.sol";
import "../src/ProxyWalletFactory.sol";
import "../src/ProxyWallet.sol";

contract DeployScript is Script {
    // Polygon addresses
    address constant USDC = 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174;
    address constant UMA_ORACLE = 0xeE3Afe347D5C74317041E2618C49534Daf887c24;

    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        address deployer = vm.addr(deployerPrivateKey);
        address admin = vm.envAddress("ADMIN_ADDRESS");
        address feeCollector = vm.envAddress("FEE_COLLECTOR");

        vm.startBroadcast(deployerPrivateKey);

        // 1. Deploy Conditional Tokens
        ConditionalTokens ctf = new ConditionalTokens();
        console.log("ConditionalTokens:", address(ctf));

        // 2. Deploy UMA Adapter
        UmaCtfAdapter adapter = new UmaCtfAdapter(
            UMA_ORACLE,
            address(ctf),
            USDC
        );
        console.log("UmaCtfAdapter:", address(adapter));

        // 3. Deploy Exchange
        CTFExchange exchange = new CTFExchange(
            address(ctf),
            USDC,
            feeCollector
        );
        console.log("CTFExchange:", address(exchange));

        // 4. Deploy Market Factory
        MarketFactory factory = new MarketFactory(
            address(ctf),
            address(adapter),
            USDC
        );
        console.log("MarketFactory:", address(factory));

        // 5. Deploy Proxy Wallet Implementation
        ProxyWallet proxyImpl = new ProxyWallet();
        console.log("ProxyWallet Implementation:", address(proxyImpl));

        // 6. Deploy Proxy Wallet Factory
        ProxyWalletFactory proxyFactory = new ProxyWalletFactory(
            address(proxyImpl),
            address(exchange),
            USDC,
            address(ctf)
        );
        console.log("ProxyWalletFactory:", address(proxyFactory));

        // 7. Configure access control
        factory.grantRole(factory.MARKET_CREATOR_ROLE(), admin);
        factory.grantRole(factory.DEFAULT_ADMIN_ROLE(), admin);
        factory.renounceRole(factory.DEFAULT_ADMIN_ROLE(), deployer);

        vm.stopBroadcast();

        // Output deployment addresses for verification
        console.log("\n=== Deployment Summary ===");
        console.log("Network: Polygon");
        console.log("ConditionalTokens:", address(ctf));
        console.log("UmaCtfAdapter:", address(adapter));
        console.log("CTFExchange:", address(exchange));
        console.log("MarketFactory:", address(factory));
        console.log("ProxyWallet:", address(proxyImpl));
        console.log("ProxyWalletFactory:", address(proxyFactory));
    }
}
```

### Post-Deployment Verification

```bash
#!/bin/bash
# verify-deployment.sh

# Set environment variables
export ETHERSCAN_API_KEY="your-polygonscan-api-key"
export RPC_URL="https://polygon-mainnet.g.alchemy.com/v2/your-key"

# Contract addresses (update after deployment)
CTF="0x..."
ADAPTER="0x..."
EXCHANGE="0x..."
FACTORY="0x..."
PROXY_IMPL="0x..."
PROXY_FACTORY="0x..."

# Verify ConditionalTokens
forge verify-contract $CTF ConditionalTokens \
    --chain polygon \
    --etherscan-api-key $ETHERSCAN_API_KEY

# Verify UmaCtfAdapter
forge verify-contract $ADAPTER UmaCtfAdapter \
    --chain polygon \
    --constructor-args $(cast abi-encode "constructor(address,address,address)" \
        "0xeE3Afe347D5C74317041E2618C49534Daf887c24" \
        "$CTF" \
        "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174") \
    --etherscan-api-key $ETHERSCAN_API_KEY

# Verify CTFExchange
forge verify-contract $EXCHANGE CTFExchange \
    --chain polygon \
    --constructor-args $(cast abi-encode "constructor(address,address,address)" \
        "$CTF" \
        "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174" \
        "$FEE_COLLECTOR") \
    --etherscan-api-key $ETHERSCAN_API_KEY

echo "Verification complete. Check Polygonscan for results."
```

### Post-Deployment Checklist

```markdown
## Verification
- [ ] All contracts verified on block explorer
- [ ] Source code matches deployed bytecode
- [ ] Constructor arguments correct

## Functionality Testing
- [ ] Create test market
- [ ] Mint test positions
- [ ] Execute test trade
- [ ] Test resolution flow

## Monitoring Setup
- [ ] Event indexing configured
- [ ] Alert thresholds set
- [ ] Dashboard created

## Documentation
- [ ] Deployment addresses documented
- [ ] Admin procedures documented
- [ ] Emergency contacts established

## Security
- [ ] Admin keys secured (multisig)
- [ ] Emergency pause tested
- [ ] Rate limits configured
```

---

## Additional Resources

### Reference Implementations

- [Gnosis Conditional Tokens](https://github.com/gnosis/conditional-tokens-contracts)
- [UMA Protocol](https://github.com/UMAprotocol/protocol)
- [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts)

### Further Reading

- [ERC-1155 Multi Token Standard](https://eips.ethereum.org/EIPS/eip-1155)
- [EIP-712 Typed Structured Data Hashing](https://eips.ethereum.org/EIPS/eip-712)
- [UMA Optimistic Oracle Documentation](https://docs.uma.xyz/)

### Community & Support

- Polymarket Discord
- Gnosis Safe Documentation
- Ethereum Stack Exchange
