# Testing Strategy

## Comprehensive Testing Framework for Prediction Market Platform

This document outlines the testing strategy, frameworks, and best practices for ensuring quality and security in a blockchain-based prediction market platform.

---

## Table of Contents

1. [Testing Pyramid](#testing-pyramid)
2. [Unit Testing Setup](#unit-testing-setup)
3. [Smart Contract Testing](#smart-contract-testing)
4. [Integration Testing](#integration-testing)
5. [End-to-End Testing](#end-to-end-testing)
6. [Load and Stress Testing](#load-and-stress-testing)
7. [Chaos Engineering](#chaos-engineering)
8. [Security Testing](#security-testing)
9. [Test Coverage Targets](#test-coverage-targets)
10. [Testing in CI/CD](#testing-in-cicd)

---

## Testing Pyramid

### Overview

```
                    ┌─────────────────┐
                    │    E2E Tests    │  ← Slowest, Most Expensive
                    │   (Playwright)  │     5-10% of tests
                    ├─────────────────┤
                    │  Integration    │
                    │    Tests        │     15-25% of tests
                    ├─────────────────┤
               ┌────┤   Contract      │
               │    │    Tests        │     Smart contracts
               │    │   (Foundry)     │     25-35% of tests
               │    ├─────────────────┤
               │    │   Unit Tests    │  ← Fastest, Most Numerous
               │    │  (Jest/Vitest)  │     40-50% of tests
               └────┴─────────────────┘
```

### Test Distribution

| Test Type | Percentage | Execution Time | Frequency |
|-----------|------------|----------------|-----------|
| Unit Tests | 40-50% | < 5 min | Every commit |
| Contract Tests | 25-35% | 5-15 min | Every commit |
| Integration | 15-25% | 10-30 min | Every PR |
| E2E | 5-10% | 30-60 min | Every PR, Daily |
| Load/Security | < 5% | Hours | Weekly, Pre-release |

---

## Unit Testing Setup

### Jest Configuration (API/Backend)

```typescript
// jest.config.ts
import type { Config } from 'jest';

const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/*.test.ts', '**/*.spec.ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/index.ts',
    '!src/**/*.interface.ts',
    '!src/migrations/**',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 85,
      lines: 85,
      statements: 85,
    },
  },
  coverageReporters: ['text', 'lcov', 'html'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  globalSetup: '<rootDir>/jest.global-setup.ts',
  globalTeardown: '<rootDir>/jest.global-teardown.ts',
  testTimeout: 30000,
  verbose: true,
  forceExit: true,
  detectOpenHandles: true,
};

export default config;
```

### Jest Setup File

```typescript
// jest.setup.ts
import { mockDeep, DeepMockProxy } from 'jest-mock-extended';
import { PrismaClient } from '@prisma/client';

// Mock Prisma client
jest.mock('./src/lib/prisma', () => ({
  prisma: mockDeep<PrismaClient>(),
}));

// Mock Redis
jest.mock('./src/lib/redis', () => ({
  redis: {
    get: jest.fn(),
    set: jest.fn(),
    del: jest.fn(),
    setex: jest.fn(),
  },
}));

// Global test utilities
global.createMockUser = () => ({
  id: 'user_test123',
  address: '0x1234567890123456789012345678901234567890',
  createdAt: new Date(),
  updatedAt: new Date(),
});

global.createMockMarket = () => ({
  id: 'market_test456',
  title: 'Test Market',
  description: 'Test description',
  status: 'ACTIVE',
  createdAt: new Date(),
  resolutionDate: new Date(Date.now() + 86400000),
});

// Cleanup after each test
afterEach(() => {
  jest.clearAllMocks();
});
```

### Unit Test Examples

```typescript
// src/services/__tests__/market.service.test.ts
import { MarketService } from '../market.service';
import { prisma } from '@/lib/prisma';
import { redis } from '@/lib/redis';

jest.mock('@/lib/prisma');
jest.mock('@/lib/redis');

describe('MarketService', () => {
  let marketService: MarketService;
  const mockPrisma = prisma as jest.Mocked<typeof prisma>;

  beforeEach(() => {
    marketService = new MarketService();
    jest.clearAllMocks();
  });

  describe('getMarket', () => {
    it('should return market from cache if available', async () => {
      // Arrange
      const cachedMarket = { id: 'market_123', title: 'Cached Market' };
      (redis.get as jest.Mock).mockResolvedValue(JSON.stringify(cachedMarket));

      // Act
      const result = await marketService.getMarket('market_123');

      // Assert
      expect(redis.get).toHaveBeenCalledWith('market:market_123');
      expect(mockPrisma.market.findUnique).not.toHaveBeenCalled();
      expect(result).toEqual(cachedMarket);
    });

    it('should fetch from database and cache if not in cache', async () => {
      // Arrange
      const dbMarket = { id: 'market_123', title: 'DB Market' };
      (redis.get as jest.Mock).mockResolvedValue(null);
      mockPrisma.market.findUnique.mockResolvedValue(dbMarket as any);

      // Act
      const result = await marketService.getMarket('market_123');

      // Assert
      expect(mockPrisma.market.findUnique).toHaveBeenCalledWith({
        where: { id: 'market_123' },
        include: expect.any(Object),
      });
      expect(redis.setex).toHaveBeenCalledWith(
        'market:market_123',
        300,
        JSON.stringify(dbMarket)
      );
      expect(result).toEqual(dbMarket);
    });

    it('should throw NotFoundError if market does not exist', async () => {
      // Arrange
      (redis.get as jest.Mock).mockResolvedValue(null);
      mockPrisma.market.findUnique.mockResolvedValue(null);

      // Act & Assert
      await expect(marketService.getMarket('nonexistent'))
        .rejects
        .toThrow('Market not found');
    });
  });

  describe('calculateSharePrice', () => {
    it('should calculate correct price for binary market', () => {
      // Arrange
      const market = {
        poolYes: BigInt('1000000000000000000'),  // 1e18
        poolNo: BigInt('1000000000000000000'),   // 1e18
      };

      // Act
      const price = marketService.calculateSharePrice(market, 'YES');

      // Assert
      expect(price).toBe(0.5);  // 50% probability
    });

    it('should handle edge cases with very small pools', () => {
      const market = {
        poolYes: BigInt('1'),
        poolNo: BigInt('1000000000000000000'),
      };

      const price = marketService.calculateSharePrice(market, 'YES');

      expect(price).toBeLessThan(0.001);
    });
  });
});

// src/utils/__tests__/validation.test.ts
describe('Validation Utils', () => {
  describe('isValidEthereumAddress', () => {
    it.each([
      ['0x1234567890123456789012345678901234567890', true],
      ['0x0000000000000000000000000000000000000000', true],
      ['1234567890123456789012345678901234567890', false],  // No 0x prefix
      ['0x123456789012345678901234567890123456789', false],  // Too short
      ['0x12345678901234567890123456789012345678901', false], // Too long
      ['0xGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG', false], // Invalid chars
      ['', false],
      [null, false],
    ])('isValidEthereumAddress(%s) should return %s', (address, expected) => {
      expect(isValidEthereumAddress(address as any)).toBe(expected);
    });
  });
});
```

### Vitest Configuration (Frontend)

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./vitest.setup.ts'],
    include: ['src/**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'lcov', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        'src/generated/',
      ],
      thresholds: {
        branches: 75,
        functions: 80,
        lines: 80,
        statements: 80,
      },
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### React Component Testing

```typescript
// src/components/__tests__/MarketCard.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MarketCard } from '../MarketCard';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const mockMarket = {
  id: 'market_123',
  title: 'Will BTC reach $100k?',
  description: 'Test market',
  status: 'ACTIVE',
  yesPrice: 0.65,
  noPrice: 0.35,
  volume24h: '1000000',
  resolutionDate: new Date(Date.now() + 86400000).toISOString(),
};

const renderWithProviders = (component: React.ReactElement) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });

  return render(
    <QueryClientProvider client={queryClient}>
      {component}
    </QueryClientProvider>
  );
};

describe('MarketCard', () => {
  it('renders market information correctly', () => {
    renderWithProviders(<MarketCard market={mockMarket} />);

    expect(screen.getByText('Will BTC reach $100k?')).toBeInTheDocument();
    expect(screen.getByText('65%')).toBeInTheDocument();  // Yes price
    expect(screen.getByText('$1M')).toBeInTheDocument();  // Volume
  });

  it('shows loading state when fetching price', async () => {
    renderWithProviders(<MarketCard market={mockMarket} isLoading />);

    expect(screen.getByTestId('price-skeleton')).toBeInTheDocument();
  });

  it('handles trade button click', async () => {
    const onTrade = vi.fn();
    const user = userEvent.setup();

    renderWithProviders(
      <MarketCard market={mockMarket} onTrade={onTrade} />
    );

    await user.click(screen.getByRole('button', { name: /buy yes/i }));

    expect(onTrade).toHaveBeenCalledWith({
      marketId: 'market_123',
      outcome: 'YES',
    });
  });

  it('displays expired state for past resolution date', () => {
    const expiredMarket = {
      ...mockMarket,
      resolutionDate: new Date(Date.now() - 86400000).toISOString(),
    };

    renderWithProviders(<MarketCard market={expiredMarket} />);

    expect(screen.getByText(/expired/i)).toBeInTheDocument();
    expect(screen.queryByRole('button', { name: /buy/i })).not.toBeInTheDocument();
  });
});
```

---

## Smart Contract Testing

### Foundry Configuration

```toml
# foundry.toml
[profile.default]
src = "contracts"
out = "out"
libs = ["lib"]
optimizer = true
optimizer_runs = 200
via_ir = true

[profile.default.fuzz]
runs = 1000
max_test_rejects = 65536

[profile.default.invariant]
runs = 256
depth = 15
fail_on_revert = false

[profile.ci]
fuzz.runs = 5000
invariant.runs = 500

[fmt]
line_length = 120
tab_width = 4
bracket_spacing = true
```

### Contract Unit Tests

```solidity
// test/MarketFactory.t.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../contracts/MarketFactory.sol";
import "../contracts/Market.sol";
import "../contracts/mocks/MockUSDC.sol";

contract MarketFactoryTest is Test {
    MarketFactory public factory;
    MockUSDC public usdc;

    address public owner;
    address public alice;
    address public bob;

    event MarketCreated(address indexed market, string title, uint256 resolutionTime);

    function setUp() public {
        owner = address(this);
        alice = makeAddr("alice");
        bob = makeAddr("bob");

        usdc = new MockUSDC();
        factory = new MarketFactory(address(usdc));

        // Fund test accounts
        usdc.mint(alice, 1_000_000e6);
        usdc.mint(bob, 1_000_000e6);
    }

    function test_CreateMarket() public {
        string memory title = "Will BTC reach $100k?";
        string memory description = "Bitcoin price prediction";
        uint256 resolutionTime = block.timestamp + 30 days;

        vm.expectEmit(false, true, false, true);
        emit MarketCreated(address(0), title, resolutionTime);

        address marketAddress = factory.createMarket(
            title,
            description,
            resolutionTime
        );

        assertNotEq(marketAddress, address(0));

        Market market = Market(marketAddress);
        assertEq(market.title(), title);
        assertEq(market.description(), description);
        assertEq(market.resolutionTime(), resolutionTime);
    }

    function test_CreateMarket_RevertsPastResolutionTime() public {
        vm.expectRevert("Resolution time must be in future");

        factory.createMarket(
            "Test",
            "Test",
            block.timestamp - 1
        );
    }

    function testFuzz_CreateMarket_ValidResolutionTime(uint256 offset) public {
        // Bound the fuzz input to reasonable values
        offset = bound(offset, 1 hours, 365 days);
        uint256 resolutionTime = block.timestamp + offset;

        address marketAddress = factory.createMarket(
            "Fuzz Test Market",
            "Testing various resolution times",
            resolutionTime
        );

        Market market = Market(marketAddress);
        assertEq(market.resolutionTime(), resolutionTime);
    }
}
```

### Trading Contract Tests

```solidity
// test/TradingEngine.t.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../contracts/TradingEngine.sol";
import "../contracts/Market.sol";
import "../contracts/mocks/MockUSDC.sol";

contract TradingEngineTest is Test {
    TradingEngine public engine;
    Market public market;
    MockUSDC public usdc;

    address public alice;
    address public bob;

    uint256 constant INITIAL_LIQUIDITY = 100_000e6;  // 100k USDC
    uint256 constant TRADE_AMOUNT = 1_000e6;  // 1k USDC

    function setUp() public {
        alice = makeAddr("alice");
        bob = makeAddr("bob");

        usdc = new MockUSDC();
        engine = new TradingEngine(address(usdc));

        // Create a market
        market = Market(engine.createMarket(
            "Test Market",
            "Description",
            block.timestamp + 30 days
        ));

        // Add initial liquidity
        usdc.mint(address(this), INITIAL_LIQUIDITY);
        usdc.approve(address(engine), INITIAL_LIQUIDITY);
        engine.addLiquidity(address(market), INITIAL_LIQUIDITY);

        // Fund test accounts
        usdc.mint(alice, 1_000_000e6);
        usdc.mint(bob, 1_000_000e6);
    }

    function test_BuyYesShares() public {
        vm.startPrank(alice);
        usdc.approve(address(engine), TRADE_AMOUNT);

        uint256 sharesBefore = market.balanceOf(alice, 0);  // YES = 0

        (uint256 shares, uint256 avgPrice) = engine.buy(
            address(market),
            0,  // YES outcome
            TRADE_AMOUNT,
            0   // No slippage protection for test
        );

        uint256 sharesAfter = market.balanceOf(alice, 0);

        assertGt(shares, 0, "Should receive shares");
        assertEq(sharesAfter - sharesBefore, shares, "Balance should increase");
        assertLt(avgPrice, 1e6, "Price should be less than $1");

        vm.stopPrank();
    }

    function test_SellYesShares() public {
        // First buy some shares
        vm.startPrank(alice);
        usdc.approve(address(engine), TRADE_AMOUNT);
        (uint256 shares,) = engine.buy(address(market), 0, TRADE_AMOUNT, 0);

        // Then sell half
        uint256 sharesToSell = shares / 2;
        market.setApprovalForAll(address(engine), true);

        uint256 usdcBefore = usdc.balanceOf(alice);

        (uint256 proceeds,) = engine.sell(
            address(market),
            0,
            sharesToSell,
            0  // No slippage protection
        );

        uint256 usdcAfter = usdc.balanceOf(alice);

        assertGt(proceeds, 0, "Should receive USDC");
        assertEq(usdcAfter - usdcBefore, proceeds, "USDC balance should increase");

        vm.stopPrank();
    }

    function test_RevertOnHighSlippage() public {
        vm.startPrank(alice);
        usdc.approve(address(engine), TRADE_AMOUNT);

        // Set minimum shares very high (will fail)
        uint256 minSharesOut = type(uint256).max;

        vm.expectRevert("Slippage exceeded");
        engine.buy(address(market), 0, TRADE_AMOUNT, minSharesOut);

        vm.stopPrank();
    }

    function test_PriceImpact() public {
        vm.startPrank(alice);
        usdc.approve(address(engine), 50_000e6);

        // Get price before
        uint256 priceBefore = engine.getPrice(address(market), 0);

        // Large trade
        engine.buy(address(market), 0, 50_000e6, 0);

        // Get price after
        uint256 priceAfter = engine.getPrice(address(market), 0);

        assertGt(priceAfter, priceBefore, "Price should increase after large buy");

        vm.stopPrank();
    }

    function testFuzz_TradeDoesNotDrainPool(uint256 tradeSize) public {
        // Bound to reasonable trade sizes
        tradeSize = bound(tradeSize, 1e6, INITIAL_LIQUIDITY);

        vm.startPrank(alice);
        usdc.mint(alice, tradeSize);
        usdc.approve(address(engine), tradeSize);

        engine.buy(address(market), 0, tradeSize, 0);

        // Pool should still have liquidity
        (uint256 yesPool, uint256 noPool) = engine.getPoolState(address(market));
        assertGt(yesPool, 0, "YES pool should not be drained");
        assertGt(noPool, 0, "NO pool should not be drained");

        vm.stopPrank();
    }
}
```

### Invariant Testing

```solidity
// test/invariants/TradingInvariants.t.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "./handlers/TradingHandler.sol";
import "../../contracts/TradingEngine.sol";

contract TradingInvariantsTest is Test {
    TradingEngine public engine;
    TradingHandler public handler;
    Market public market;
    MockUSDC public usdc;

    function setUp() public {
        usdc = new MockUSDC();
        engine = new TradingEngine(address(usdc));

        market = Market(engine.createMarket(
            "Invariant Test Market",
            "Testing invariants",
            block.timestamp + 30 days
        ));

        // Initial liquidity
        usdc.mint(address(this), 1_000_000e6);
        usdc.approve(address(engine), 1_000_000e6);
        engine.addLiquidity(address(market), 1_000_000e6);

        // Create handler
        handler = new TradingHandler(engine, market, usdc);

        // Target the handler for invariant testing
        targetContract(address(handler));
    }

    // Invariant: YES price + NO price should always equal ~1
    function invariant_PricesSumToOne() public view {
        uint256 yesPrice = engine.getPrice(address(market), 0);
        uint256 noPrice = engine.getPrice(address(market), 1);

        // Allow 0.1% deviation due to rounding
        assertApproxEqRel(yesPrice + noPrice, 1e6, 0.001e18);
    }

    // Invariant: Pool product should remain constant (for CPMM)
    function invariant_ConstantProduct() public view {
        (uint256 yesPool, uint256 noPool) = engine.getPoolState(address(market));
        uint256 product = yesPool * noPool;

        // Product should be >= initial (can grow with fees)
        uint256 initialProduct = 500_000e6 * 500_000e6;  // Initial 50/50 split
        assertGe(product, initialProduct);
    }

    // Invariant: Total shares should equal USDC deposited
    function invariant_SharesBackedByCollateral() public view {
        uint256 totalYesShares = market.totalSupply(0);
        uint256 totalNoShares = market.totalSupply(1);
        uint256 collateral = usdc.balanceOf(address(engine));

        // In binary markets, max payout is max(yes, no) shares
        uint256 maxPayout = totalYesShares > totalNoShares
            ? totalYesShares
            : totalNoShares;

        assertGe(collateral, maxPayout, "Insufficient collateral");
    }

    // Invariant: No individual can have negative balance
    function invariant_NoNegativeBalances() public view {
        // This is implicit in ERC1155, but good to verify
        address[] memory actors = handler.getActors();
        for (uint256 i = 0; i < actors.length; i++) {
            assertGe(market.balanceOf(actors[i], 0), 0);
            assertGe(market.balanceOf(actors[i], 1), 0);
            assertGe(usdc.balanceOf(actors[i]), 0);
        }
    }
}

// test/invariants/handlers/TradingHandler.sol
contract TradingHandler is Test {
    TradingEngine public engine;
    Market public market;
    MockUSDC public usdc;

    address[] public actors;
    address internal currentActor;

    modifier useActor(uint256 actorIndexSeed) {
        currentActor = actors[bound(actorIndexSeed, 0, actors.length - 1)];
        vm.startPrank(currentActor);
        _;
        vm.stopPrank();
    }

    constructor(TradingEngine _engine, Market _market, MockUSDC _usdc) {
        engine = _engine;
        market = _market;
        usdc = _usdc;

        // Create test actors
        for (uint256 i = 0; i < 10; i++) {
            address actor = makeAddr(string(abi.encodePacked("actor", i)));
            actors.push(actor);
            usdc.mint(actor, 100_000e6);
        }
    }

    function buy(uint256 actorSeed, uint256 outcome, uint256 amount) public useActor(actorSeed) {
        outcome = bound(outcome, 0, 1);
        amount = bound(amount, 1e6, 10_000e6);

        usdc.approve(address(engine), amount);

        try engine.buy(address(market), outcome, amount, 0) {
            // Success
        } catch {
            // Expected failures are ok
        }
    }

    function sell(uint256 actorSeed, uint256 outcome, uint256 shares) public useActor(actorSeed) {
        outcome = bound(outcome, 0, 1);
        uint256 balance = market.balanceOf(currentActor, outcome);
        shares = bound(shares, 0, balance);

        if (shares == 0) return;

        market.setApprovalForAll(address(engine), true);

        try engine.sell(address(market), outcome, shares, 0) {
            // Success
        } catch {
            // Expected failures are ok
        }
    }

    function getActors() external view returns (address[] memory) {
        return actors;
    }
}
```

---

## Integration Testing

### Database Integration Tests

```typescript
// tests/integration/market.integration.test.ts
import { PrismaClient } from '@prisma/client';
import { MarketService } from '@/services/market.service';
import { setupTestDatabase, teardownTestDatabase } from './helpers/database';

describe('Market Service Integration', () => {
  let prisma: PrismaClient;
  let marketService: MarketService;

  beforeAll(async () => {
    prisma = await setupTestDatabase();
    marketService = new MarketService(prisma);
  });

  afterAll(async () => {
    await teardownTestDatabase(prisma);
  });

  beforeEach(async () => {
    // Clean tables before each test
    await prisma.trade.deleteMany();
    await prisma.position.deleteMany();
    await prisma.market.deleteMany();
    await prisma.user.deleteMany();
  });

  describe('createMarket', () => {
    it('should create market with correct data', async () => {
      const input = {
        title: 'Test Market',
        description: 'Integration test market',
        category: 'crypto',
        resolutionDate: new Date(Date.now() + 86400000),
        creatorId: 'user_123',
      };

      const market = await marketService.createMarket(input);

      expect(market).toMatchObject({
        title: input.title,
        description: input.description,
        category: input.category,
        status: 'ACTIVE',
      });

      // Verify in database
      const dbMarket = await prisma.market.findUnique({
        where: { id: market.id },
      });

      expect(dbMarket).toBeTruthy();
      expect(dbMarket?.title).toBe(input.title);
    });

    it('should initialize pool with correct values', async () => {
      const market = await marketService.createMarket({
        title: 'Pool Test',
        description: 'Testing pool initialization',
        category: 'test',
        resolutionDate: new Date(Date.now() + 86400000),
        creatorId: 'user_123',
        initialLiquidity: '10000000000', // 10k USDC in wei
      });

      expect(market.poolYes.toString()).toBe('5000000000');
      expect(market.poolNo.toString()).toBe('5000000000');
    });
  });

  describe('executeTrade', () => {
    let market: Market;
    let user: User;

    beforeEach(async () => {
      user = await prisma.user.create({
        data: {
          address: '0x1234567890123456789012345678901234567890',
        },
      });

      market = await prisma.market.create({
        data: {
          title: 'Trade Test Market',
          description: 'Testing trades',
          category: 'test',
          status: 'ACTIVE',
          resolutionDate: new Date(Date.now() + 86400000),
          poolYes: BigInt('50000000000'),
          poolNo: BigInt('50000000000'),
        },
      });
    });

    it('should create trade and update position', async () => {
      const trade = await marketService.executeTrade({
        userId: user.id,
        marketId: market.id,
        outcome: 0, // YES
        side: 'BUY',
        amount: '1000000000', // 1k USDC
        txHash: '0xabc123',
        blockNumber: 12345678,
      });

      expect(trade.shares).toBeTruthy();
      expect(BigInt(trade.shares) > BigInt(0)).toBe(true);

      // Check position was created/updated
      const position = await prisma.position.findUnique({
        where: {
          userId_marketId_outcome: {
            userId: user.id,
            marketId: market.id,
            outcome: 0,
          },
        },
      });

      expect(position).toBeTruthy();
      expect(position?.shares.toString()).toBe(trade.shares);
    });

    it('should update pool after trade', async () => {
      const poolYesBefore = market.poolYes;
      const poolNoBefore = market.poolNo;

      await marketService.executeTrade({
        userId: user.id,
        marketId: market.id,
        outcome: 0,
        side: 'BUY',
        amount: '1000000000',
        txHash: '0xdef456',
        blockNumber: 12345679,
      });

      const updatedMarket = await prisma.market.findUnique({
        where: { id: market.id },
      });

      // Buying YES should decrease YES pool and increase NO pool
      expect(updatedMarket!.poolYes < poolYesBefore).toBe(true);
      expect(updatedMarket!.poolNo > poolNoBefore).toBe(true);
    });
  });
});

// tests/integration/helpers/database.ts
import { PrismaClient } from '@prisma/client';
import { execSync } from 'child_process';

export async function setupTestDatabase(): Promise<PrismaClient> {
  // Use test database URL
  process.env.DATABASE_URL = process.env.TEST_DATABASE_URL;

  // Run migrations
  execSync('npx prisma migrate deploy', { stdio: 'inherit' });

  const prisma = new PrismaClient();
  await prisma.$connect();

  return prisma;
}

export async function teardownTestDatabase(prisma: PrismaClient): Promise<void> {
  await prisma.$disconnect();
}
```

### API Integration Tests

```typescript
// tests/integration/api/markets.api.test.ts
import request from 'supertest';
import { app } from '@/app';
import { setupTestDatabase, teardownTestDatabase, createTestUser } from '../helpers';

describe('Markets API', () => {
  let authToken: string;

  beforeAll(async () => {
    await setupTestDatabase();
    const user = await createTestUser();
    authToken = await generateAuthToken(user);
  });

  afterAll(async () => {
    await teardownTestDatabase();
  });

  describe('GET /v2/markets', () => {
    it('should return paginated markets', async () => {
      const response = await request(app)
        .get('/v2/markets')
        .set('Authorization', `Bearer ${authToken}`)
        .query({ pageSize: 10 });

      expect(response.status).toBe(200);
      expect(response.body).toMatchObject({
        data: expect.any(Array),
        meta: expect.objectContaining({
          version: expect.any(String),
        }),
        pagination: expect.objectContaining({
          hasMore: expect.any(Boolean),
        }),
      });
    });

    it('should filter by category', async () => {
      const response = await request(app)
        .get('/v2/markets')
        .set('Authorization', `Bearer ${authToken}`)
        .query({ category: 'crypto' });

      expect(response.status).toBe(200);
      response.body.data.forEach((market: any) => {
        expect(market.category).toBe('crypto');
      });
    });

    it('should handle cursor pagination', async () => {
      // Get first page
      const page1 = await request(app)
        .get('/v2/markets')
        .set('Authorization', `Bearer ${authToken}`)
        .query({ pageSize: 5 });

      // Get second page using cursor
      const page2 = await request(app)
        .get('/v2/markets')
        .set('Authorization', `Bearer ${authToken}`)
        .query({
          pageSize: 5,
          cursor: page1.body.pagination.nextCursor,
        });

      // Should not have overlapping items
      const page1Ids = page1.body.data.map((m: any) => m.id);
      const page2Ids = page2.body.data.map((m: any) => m.id);

      const overlap = page1Ids.filter((id: string) => page2Ids.includes(id));
      expect(overlap).toHaveLength(0);
    });
  });

  describe('POST /v2/markets/:id/trade', () => {
    it('should execute trade with valid input', async () => {
      const marketId = 'market_test123';

      const response = await request(app)
        .post(`/v2/markets/${marketId}/trade`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          outcome: 'YES',
          side: 'BUY',
          amount: '100000000',
          maxSlippage: 0.05,
        });

      expect(response.status).toBe(200);
      expect(response.body.data).toMatchObject({
        id: expect.any(String),
        shares: expect.any(String),
        avgPrice: expect.any(String),
        txHash: expect.any(String),
      });
    });

    it('should reject trade with insufficient balance', async () => {
      const response = await request(app)
        .post('/v2/markets/market_test123/trade')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          outcome: 'YES',
          side: 'BUY',
          amount: '999999999999999999999',  // Very large amount
          maxSlippage: 0.05,
        });

      expect(response.status).toBe(400);
      expect(response.body.error.code).toBe('INSUFFICIENT_BALANCE');
    });

    it('should reject trade exceeding slippage', async () => {
      const response = await request(app)
        .post('/v2/markets/market_test123/trade')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          outcome: 'YES',
          side: 'BUY',
          amount: '50000000000000',  // Large trade causing slippage
          maxSlippage: 0.001,  // Very tight slippage
        });

      expect(response.status).toBe(400);
      expect(response.body.error.code).toBe('SLIPPAGE_EXCEEDED');
    });
  });
});
```

---

## End-to-End Testing

### Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
    ['github'],
  ],
  use: {
    baseURL: process.env.E2E_BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### E2E Test Examples

```typescript
// tests/e2e/trading.spec.ts
import { test, expect } from '@playwright/test';
import { connectWallet, fundWallet } from './helpers/wallet';

test.describe('Trading Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Connect wallet before each test
    await connectWallet(page);
    await fundWallet(page, '1000');  // Fund with 1000 USDC
  });

  test('complete buy flow', async ({ page }) => {
    // Navigate to market
    await page.goto('/markets/btc-100k-2025');

    // Verify market loaded
    await expect(page.getByRole('heading', { name: /BTC.*100k/i })).toBeVisible();

    // Click Buy YES
    await page.getByRole('button', { name: 'Buy YES' }).click();

    // Fill trade amount
    await page.getByLabel('Amount').fill('100');

    // Verify preview
    await expect(page.getByTestId('trade-preview')).toContainText('shares');
    await expect(page.getByTestId('price-impact')).toBeVisible();

    // Confirm trade
    await page.getByRole('button', { name: 'Confirm Trade' }).click();

    // Wait for transaction
    await expect(page.getByText('Transaction submitted')).toBeVisible({ timeout: 10000 });
    await expect(page.getByText('Trade confirmed')).toBeVisible({ timeout: 30000 });

    // Verify portfolio updated
    await page.goto('/portfolio');
    await expect(page.getByText('BTC')).toBeVisible();
    await expect(page.getByTestId('position-shares')).not.toHaveText('0');
  });

  test('handles slippage warning', async ({ page }) => {
    await page.goto('/markets/low-liquidity-market');

    await page.getByRole('button', { name: 'Buy YES' }).click();
    await page.getByLabel('Amount').fill('50000');  // Large amount

    // Should show slippage warning
    await expect(page.getByText(/price impact.*high/i)).toBeVisible();
    await expect(page.getByTestId('price-impact')).toHaveCSS('color', /red|#ff/);

    // Confirm button should require acknowledgment
    await expect(page.getByRole('button', { name: 'Confirm Trade' })).toBeDisabled();
    await page.getByLabel(/I understand the price impact/i).check();
    await expect(page.getByRole('button', { name: 'Confirm Trade' })).toBeEnabled();
  });

  test('sell flow with existing position', async ({ page }) => {
    // First buy some shares
    await page.goto('/markets/btc-100k-2025');
    await page.getByRole('button', { name: 'Buy YES' }).click();
    await page.getByLabel('Amount').fill('100');
    await page.getByRole('button', { name: 'Confirm Trade' }).click();
    await expect(page.getByText('Trade confirmed')).toBeVisible({ timeout: 30000 });

    // Now sell
    await page.getByRole('button', { name: 'Sell YES' }).click();
    await page.getByRole('button', { name: 'Max' }).click();  // Sell all

    await page.getByRole('button', { name: 'Confirm Trade' }).click();
    await expect(page.getByText('Trade confirmed')).toBeVisible({ timeout: 30000 });

    // Verify position is zero
    await page.goto('/portfolio');
    await expect(page.getByText('No positions')).toBeVisible();
  });
});

// tests/e2e/helpers/wallet.ts
import { Page } from '@playwright/test';

export async function connectWallet(page: Page): Promise<void> {
  // Mock MetaMask connection for E2E tests
  await page.evaluate(() => {
    window.ethereum = {
      isMetaMask: true,
      request: async ({ method }: { method: string }) => {
        if (method === 'eth_requestAccounts') {
          return ['0x1234567890123456789012345678901234567890'];
        }
        if (method === 'eth_chainId') {
          return '0x89';  // Polygon
        }
        return null;
      },
      on: () => {},
      removeListener: () => {},
    };
  });

  await page.goto('/');
  await page.getByRole('button', { name: 'Connect Wallet' }).click();
  await page.getByRole('button', { name: 'MetaMask' }).click();
}

export async function fundWallet(page: Page, amount: string): Promise<void> {
  // In E2E tests, mock the balance
  await page.evaluate((amt) => {
    window.__TEST_USDC_BALANCE__ = amt;
  }, amount);
}
```

---

## Load and Stress Testing

### k6 Load Test Configuration

```typescript
// tests/load/trading-load.ts
import http from 'k6/http';
import { check, sleep, group } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const tradeSuccessRate = new Rate('trade_success_rate');
const tradeDuration = new Trend('trade_duration');

export const options = {
  scenarios: {
    // Ramp up to normal load
    normal_load: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '2m', target: 50 },   // Ramp up
        { duration: '5m', target: 50 },   // Stay at 50
        { duration: '2m', target: 100 },  // Ramp to 100
        { duration: '5m', target: 100 },  // Stay at 100
        { duration: '2m', target: 0 },    // Ramp down
      ],
      gracefulRampDown: '30s',
    },
    // Spike test
    spike: {
      executor: 'ramping-vus',
      startTime: '17m',  // Start after normal load
      startVUs: 0,
      stages: [
        { duration: '10s', target: 500 },  // Quick spike
        { duration: '1m', target: 500 },   // Hold
        { duration: '10s', target: 0 },    // Quick drop
      ],
    },
  },
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    http_req_failed: ['rate<0.01'],
    trade_success_rate: ['rate>0.99'],
    trade_duration: ['p(95)<2000'],
  },
};

const BASE_URL = __ENV.BASE_URL || 'https://staging-api.predictionmarket.com';
const API_KEY = __ENV.API_KEY;

export default function () {
  const headers = {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${API_KEY}`,
  };

  group('Browse Markets', () => {
    // List markets
    const marketsRes = http.get(`${BASE_URL}/v2/markets?pageSize=20`, { headers });
    check(marketsRes, {
      'markets list status 200': (r) => r.status === 200,
      'markets list has data': (r) => JSON.parse(r.body as string).data.length > 0,
    });

    sleep(1);

    // Get specific market
    const markets = JSON.parse(marketsRes.body as string).data;
    if (markets.length > 0) {
      const marketId = markets[Math.floor(Math.random() * markets.length)].id;
      const marketRes = http.get(`${BASE_URL}/v2/markets/${marketId}`, { headers });
      check(marketRes, {
        'market detail status 200': (r) => r.status === 200,
      });
    }

    sleep(0.5);
  });

  group('Execute Trade', () => {
    const tradeStart = Date.now();

    const tradeRes = http.post(
      `${BASE_URL}/v2/markets/test-market/trade`,
      JSON.stringify({
        outcome: Math.random() > 0.5 ? 'YES' : 'NO',
        side: 'BUY',
        amount: '1000000',  // 1 USDC
        maxSlippage: 0.05,
      }),
      { headers }
    );

    const tradeDurationMs = Date.now() - tradeStart;
    tradeDuration.add(tradeDurationMs);

    const success = tradeRes.status === 200;
    tradeSuccessRate.add(success);

    check(tradeRes, {
      'trade status 200': (r) => r.status === 200,
      'trade has txHash': (r) => {
        if (r.status !== 200) return false;
        const body = JSON.parse(r.body as string);
        return body.data && body.data.txHash;
      },
    });

    sleep(2);
  });

  group('Check Portfolio', () => {
    const portfolioRes = http.get(`${BASE_URL}/v2/portfolio`, { headers });
    check(portfolioRes, {
      'portfolio status 200': (r) => r.status === 200,
    });

    sleep(1);
  });
}

export function handleSummary(data: any) {
  return {
    'load-test-results.json': JSON.stringify(data),
    stdout: textSummary(data, { indent: ' ', enableColors: true }),
  };
}
```

### Artillery Configuration

```yaml
# tests/load/artillery-config.yml
config:
  target: "https://staging-api.predictionmarket.com"
  phases:
    - duration: 60
      arrivalRate: 10
      name: "Warm up"
    - duration: 300
      arrivalRate: 50
      name: "Sustained load"
    - duration: 60
      arrivalRate: 100
      name: "Peak load"
  defaults:
    headers:
      Content-Type: "application/json"
      Authorization: "Bearer {{ $processEnvironment.API_KEY }}"
  plugins:
    expect: {}
    metrics-by-endpoint: {}

scenarios:
  - name: "Browse and Trade"
    weight: 70
    flow:
      - get:
          url: "/v2/markets"
          capture:
            - json: "$.data[0].id"
              as: "marketId"
          expect:
            - statusCode: 200
            - hasProperty: "data"

      - think: 2

      - get:
          url: "/v2/markets/{{ marketId }}"
          expect:
            - statusCode: 200

      - think: 1

      - post:
          url: "/v2/markets/{{ marketId }}/trade"
          json:
            outcome: "YES"
            side: "BUY"
            amount: "1000000"
            maxSlippage: 0.05
          expect:
            - statusCode: 200
            - hasProperty: "data.txHash"

  - name: "Portfolio Check"
    weight: 20
    flow:
      - get:
          url: "/v2/portfolio"
          expect:
            - statusCode: 200

      - get:
          url: "/v2/portfolio/history"
          expect:
            - statusCode: 200

  - name: "Price Websocket"
    weight: 10
    engine: ws
    flow:
      - send:
          payload: '{"type":"subscribe","channel":"prices","marketId":"test-market"}'
      - think: 30
      - send:
          payload: '{"type":"unsubscribe","channel":"prices"}'
```

---

## Chaos Engineering

### Chaos Mesh Configuration

```yaml
# tests/chaos/network-delay.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: api-network-delay
  namespace: staging
spec:
  action: delay
  mode: all
  selector:
    namespaces:
      - staging
    labelSelectors:
      app: api
  delay:
    latency: "500ms"
    correlation: "50"
    jitter: "100ms"
  duration: "5m"
---
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: api-pod-kill
  namespace: staging
spec:
  action: pod-kill
  mode: one
  selector:
    namespaces:
      - staging
    labelSelectors:
      app: api
  scheduler:
    cron: "@every 10m"
---
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: api-cpu-stress
  namespace: staging
spec:
  mode: all
  selector:
    namespaces:
      - staging
    labelSelectors:
      app: api
  stressors:
    cpu:
      workers: 2
      load: 80
  duration: "5m"
```

### Chaos Testing Script

```typescript
// tests/chaos/run-chaos-tests.ts
import { ChaosMesh } from '@chaos-mesh/client';
import { runLoadTest } from './helpers';

async function runChaosExperiment(name: string, config: any): Promise<TestResult> {
  const chaos = new ChaosMesh({
    apiUrl: process.env.CHAOS_MESH_URL,
  });

  console.log(`Starting chaos experiment: ${name}`);

  // Start chaos
  const experiment = await chaos.create(config);

  // Run load test during chaos
  const loadTestResults = await runLoadTest({
    duration: '5m',
    targetRPS: 100,
  });

  // Stop chaos
  await chaos.delete(experiment.metadata.name);

  // Analyze results
  return {
    name,
    p99Latency: loadTestResults.p99,
    errorRate: loadTestResults.errorRate,
    recoveryTime: await measureRecoveryTime(),
  };
}

async function main() {
  const experiments = [
    {
      name: 'Network Delay',
      config: {
        apiVersion: 'chaos-mesh.org/v1alpha1',
        kind: 'NetworkChaos',
        spec: {
          action: 'delay',
          mode: 'all',
          selector: { labelSelectors: { app: 'api' } },
          delay: { latency: '500ms' },
          duration: '5m',
        },
      },
    },
    {
      name: 'Pod Failure',
      config: {
        apiVersion: 'chaos-mesh.org/v1alpha1',
        kind: 'PodChaos',
        spec: {
          action: 'pod-kill',
          mode: 'one',
          selector: { labelSelectors: { app: 'api' } },
        },
      },
    },
    {
      name: 'Database Partition',
      config: {
        apiVersion: 'chaos-mesh.org/v1alpha1',
        kind: 'NetworkChaos',
        spec: {
          action: 'partition',
          mode: 'all',
          selector: { labelSelectors: { app: 'api' } },
          direction: 'to',
          target: {
            selector: { labelSelectors: { app: 'postgresql' } },
          },
          duration: '2m',
        },
      },
    },
  ];

  const results = [];

  for (const experiment of experiments) {
    const result = await runChaosExperiment(experiment.name, experiment.config);
    results.push(result);

    // Wait for system to stabilize
    await sleep(60000);
  }

  // Generate report
  console.log('\n=== Chaos Test Results ===\n');
  console.table(results);

  // Check SLOs
  for (const result of results) {
    if (result.p99Latency > 2000) {
      console.error(`FAIL: ${result.name} - P99 latency exceeded 2s`);
    }
    if (result.errorRate > 0.05) {
      console.error(`FAIL: ${result.name} - Error rate exceeded 5%`);
    }
    if (result.recoveryTime > 60000) {
      console.error(`FAIL: ${result.name} - Recovery time exceeded 60s`);
    }
  }
}

main();
```

---

## Security Testing

### Static Analysis with Slither

```yaml
# .github/workflows/security.yml
name: Security Analysis

on:
  push:
    paths:
      - 'contracts/**'
  pull_request:
    paths:
      - 'contracts/**'

jobs:
  slither:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Slither
        uses: crytic/slither-action@v0.4.0
        id: slither
        with:
          target: 'contracts/'
          slither-config: 'slither.config.json'
          fail-on: 'medium'
          sarif: 'slither.sarif'

      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: slither.sarif
```

```json
// slither.config.json
{
  "detectors_to_exclude": [
    "naming-convention",
    "solc-version"
  ],
  "exclude_informational": true,
  "exclude_low": false,
  "filter_paths": [
    "node_modules",
    "lib"
  ],
  "compile_force_framework": "foundry"
}
```

### Mythril Analysis

```bash
#!/bin/bash
# scripts/mythril-analysis.sh

# Analyze main contracts
for contract in contracts/*.sol; do
  echo "Analyzing $contract..."

  docker run --rm \
    -v $(pwd):/code \
    mythril/myth:latest \
    analyze /code/$contract \
    --solc-json /code/mythril-config.json \
    --execution-timeout 300 \
    --max-depth 20 \
    -o markdown \
    > reports/mythril/$(basename $contract .sol).md

done
```

### Fuzzing with Echidna

```solidity
// test/fuzz/TradingFuzz.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "../../contracts/TradingEngine.sol";
import "../../contracts/mocks/MockUSDC.sol";

contract TradingFuzz {
    TradingEngine engine;
    MockUSDC usdc;
    Market market;

    constructor() {
        usdc = new MockUSDC();
        engine = new TradingEngine(address(usdc));

        market = Market(engine.createMarket(
            "Fuzz Market",
            "Fuzzing",
            block.timestamp + 365 days
        ));

        // Initialize with liquidity
        usdc.mint(address(this), 1_000_000e6);
        usdc.approve(address(engine), type(uint256).max);
        engine.addLiquidity(address(market), 1_000_000e6);
    }

    // Echidna will call this with random values
    function echidna_pool_not_empty() public view returns (bool) {
        (uint256 yesPool, uint256 noPool) = engine.getPoolState(address(market));
        return yesPool > 0 && noPool > 0;
    }

    function echidna_prices_valid() public view returns (bool) {
        uint256 yesPrice = engine.getPrice(address(market), 0);
        uint256 noPrice = engine.getPrice(address(market), 1);

        // Prices should be between 0 and 1 (scaled to 1e6)
        return yesPrice <= 1e6 && noPrice <= 1e6 && yesPrice > 0 && noPrice > 0;
    }

    function echidna_no_free_money() public view returns (bool) {
        // User balance should not exceed what they deposited
        uint256 totalUserBalance = market.balanceOf(address(this), 0)
                                 + market.balanceOf(address(this), 1);

        // This should always be less than total liquidity
        return totalUserBalance <= 1_000_000e6;
    }

    // Entry points for fuzzing
    function buy(uint256 outcome, uint256 amount) public {
        outcome = outcome % 2;  // 0 or 1
        amount = amount % 100_000e6;  // Cap at 100k

        if (amount < 1e6) return;  // Minimum trade

        usdc.mint(address(this), amount);
        usdc.approve(address(engine), amount);

        try engine.buy(address(market), outcome, amount, 0) {
            // Success
        } catch {
            // Expected failures ok
        }
    }

    function sell(uint256 outcome, uint256 shares) public {
        outcome = outcome % 2;
        uint256 balance = market.balanceOf(address(this), outcome);
        shares = shares % (balance + 1);

        if (shares == 0) return;

        market.setApprovalForAll(address(engine), true);

        try engine.sell(address(market), outcome, shares, 0) {
            // Success
        } catch {
            // Expected failures ok
        }
    }
}
```

```yaml
# echidna.config.yaml
testMode: assertion
testLimit: 50000
seqLen: 100
shrinkLimit: 5000
coverage: true
coverageFormats: ["html", "lcov"]
cryticArgs: ["--foundry-compile-all"]
```

---

## Test Coverage Targets

### Coverage Requirements

| Component | Line Coverage | Branch Coverage | Function Coverage |
|-----------|--------------|-----------------|-------------------|
| Smart Contracts | 95% | 90% | 100% |
| API Services | 85% | 80% | 90% |
| Frontend Components | 80% | 75% | 85% |
| Utilities/Helpers | 90% | 85% | 95% |

### Coverage Configuration

```typescript
// jest.config.ts (API)
coverageThreshold: {
  global: {
    branches: 80,
    functions: 85,
    lines: 85,
    statements: 85,
  },
  './src/services/': {
    branches: 85,
    functions: 90,
    lines: 90,
    statements: 90,
  },
  './src/utils/': {
    branches: 90,
    functions: 95,
    lines: 95,
    statements: 95,
  },
},
```

```toml
# foundry.toml (Contracts)
[profile.default]
coverage_report = "lcov"

# Minimum coverage for CI to pass
# (enforced in CI script)
```

### Coverage Enforcement in CI

```yaml
# .github/workflows/coverage.yml
jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Tests with Coverage
        run: |
          pnpm test:coverage
          forge coverage --report lcov

      - name: Check Coverage Thresholds
        run: |
          # API coverage
          npx nyc check-coverage --lines 85 --branches 80 --functions 85

          # Contract coverage
          node scripts/check-contract-coverage.js --min-lines 95 --min-branches 90

      - name: Upload to Codecov
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/lcov.info,./lcov.info
          flags: unit
          fail_ci_if_error: true
```

---

## Testing in CI/CD

### Complete CI Pipeline

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # Fast unit tests
  unit-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: Run API Unit Tests
        run: cd api && pnpm test:unit

      - name: Run Frontend Unit Tests
        run: cd frontend && pnpm test:unit

      - name: Upload Coverage
        uses: codecov/codecov-action@v4
        with:
          flags: unit

  # Contract tests
  contract-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - uses: foundry-rs/foundry-toolchain@v1
        with:
          version: nightly

      - name: Run Forge Tests
        run: forge test -vvv

      - name: Run Forge Coverage
        run: forge coverage --report lcov

      - name: Upload Coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./lcov.info
          flags: contracts

  # Integration tests
  integration-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    needs: [unit-tests, contract-tests]
    services:
      postgres:
        image: timescale/timescaledb:latest-pg15
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm db:migrate:test

      - name: Run Integration Tests
        run: pnpm test:integration
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/test
          REDIS_URL: redis://localhost:6379

  # E2E tests
  e2e-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 45
    needs: [integration-tests]
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Start Application
        run: |
          docker-compose -f docker-compose.test.yml up -d
          npx wait-on http://localhost:3000

      - name: Run E2E Tests
        run: npx playwright test
        env:
          E2E_BASE_URL: http://localhost:3000

      - name: Upload Artifacts
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/

  # Security tests
  security-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    needs: [contract-tests]
    steps:
      - uses: actions/checkout@v4

      - name: Run Slither
        uses: crytic/slither-action@v0.4.0
        with:
          fail-on: medium

      - name: Run npm audit
        run: pnpm audit --audit-level=high

  # Test Summary
  test-summary:
    runs-on: ubuntu-latest
    needs: [unit-tests, contract-tests, integration-tests, e2e-tests, security-tests]
    if: always()
    steps:
      - name: Check Results
        run: |
          if [[ "${{ needs.unit-tests.result }}" == "failure" ]] ||
             [[ "${{ needs.contract-tests.result }}" == "failure" ]] ||
             [[ "${{ needs.integration-tests.result }}" == "failure" ]] ||
             [[ "${{ needs.e2e-tests.result }}" == "failure" ]] ||
             [[ "${{ needs.security-tests.result }}" == "failure" ]]; then
            echo "One or more test suites failed"
            exit 1
          fi
          echo "All tests passed!"
```

---

## References

- [Smart Contract Unit Testing Complete Guide 2024](https://www.krayondigital.com/blog/smart-contract-unit-testing-complete-guide-2024)
- [Foundry Documentation](https://book.getfoundry.sh/)
- [Hardhat Testing Guide](https://hardhat.org/hardhat-runner/docs/guides/test-contracts)
- [Playwright Documentation](https://playwright.dev/)
- [k6 Load Testing](https://k6.io/docs/)
- [Chaos Mesh](https://chaos-mesh.org/)
- [Slither](https://github.com/crytic/slither)
- [OpenZeppelin Test Helpers](https://docs.openzeppelin.com/test-helpers/)
- [Jest Documentation](https://jestjs.io/)
- [Vitest Documentation](https://vitest.dev/)
