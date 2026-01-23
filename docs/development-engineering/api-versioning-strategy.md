# API Versioning Strategy

## Comprehensive API Version Management for Prediction Market Platform

This document outlines the API versioning strategy, deprecation policies, and migration practices for maintaining backward compatibility while enabling platform evolution.

---

## Table of Contents

1. [Versioning Approaches](#versioning-approaches)
2. [Semantic Versioning for APIs](#semantic-versioning-for-apis)
3. [Breaking vs Non-Breaking Changes](#breaking-vs-non-breaking-changes)
4. [Deprecation Policy and Timeline](#deprecation-policy-and-timeline)
5. [Migration Guides for Clients](#migration-guides-for-clients)
6. [SDK Versioning Alignment](#sdk-versioning-alignment)
7. [API Changelog Format](#api-changelog-format)
8. [Backward Compatibility Patterns](#backward-compatibility-patterns)

---

## Versioning Approaches

### Comparison of Versioning Methods

| Method | Example | Pros | Cons |
|--------|---------|------|------|
| **URL Path** | `/api/v2/markets` | Clear, cacheable, easy routing | URL clutter, harder redirects |
| **Header** | `API-Version: 2` | Clean URLs, flexible | Less visible, proxy issues |
| **Query Param** | `/api/markets?version=2` | Easy testing | Cache key issues, URL pollution |
| **Content Negotiation** | `Accept: application/vnd.pm.v2+json` | RESTful, granular | Complex, less discoverable |

### Recommended: URL Path Versioning (Primary)

We recommend URL path versioning as the primary strategy for its clarity and ease of implementation.

```
https://api.predictionmarket.com/v1/markets
https://api.predictionmarket.com/v2/markets
https://api.predictionmarket.com/v2/markets/{id}/trades
```

### Implementation

```typescript
// api/src/routes/index.ts
import { Router } from 'express';
import v1Routes from './v1';
import v2Routes from './v2';

const router = Router();

// Version routing
router.use('/v1', v1Routes);
router.use('/v2', v2Routes);

// Default to latest stable version for unversioned requests
router.use('/', (req, res, next) => {
  // Redirect to v2 or return error
  if (req.headers['x-api-version']) {
    // Header-based version override
    const version = req.headers['x-api-version'];
    req.url = `/v${version}${req.url}`;
    return router(req, res, next);
  }

  // Default redirect to latest
  res.redirect(307, `/v2${req.url}`);
});

export default router;
```

```typescript
// api/src/routes/v2/markets.ts
import { Router } from 'express';
import { marketsController } from '../../controllers/v2/markets';
import { validateRequest } from '../../middleware/validation';
import { marketSchemas } from '../../schemas/v2/markets';

const router = Router();

router.get('/',
  validateRequest(marketSchemas.list),
  marketsController.list
);

router.get('/:id',
  validateRequest(marketSchemas.get),
  marketsController.get
);

router.post('/',
  validateRequest(marketSchemas.create),
  marketsController.create
);

router.get('/:id/orderbook',
  validateRequest(marketSchemas.orderbook),
  marketsController.getOrderbook
);

export default router;
```

### API Gateway Configuration (Kong)

```yaml
# kong.yml
_format_version: "3.0"

services:
  - name: prediction-market-api
    url: http://api-service:3000
    routes:
      # V1 routes
      - name: api-v1
        paths:
          - /v1
        strip_path: false
        plugins:
          - name: rate-limiting
            config:
              minute: 100
              policy: local
          - name: request-transformer
            config:
              add:
                headers:
                  - X-API-Version:1

      # V2 routes
      - name: api-v2
        paths:
          - /v2
        strip_path: false
        plugins:
          - name: rate-limiting
            config:
              minute: 200
              policy: local
          - name: request-transformer
            config:
              add:
                headers:
                  - X-API-Version:2

      # Redirect unversioned to v2
      - name: api-default
        paths:
          - /
        strip_path: true
        plugins:
          - name: request-transformer
            config:
              replace:
                uri: /v2$(uri)
```

---

## Semantic Versioning for APIs

### Version Format

```
MAJOR.MINOR.PATCH

v2.3.1
│ │ │
│ │ └── Patch: Bug fixes, security patches
│ └──── Minor: New features, backward compatible
└────── Major: Breaking changes
```

### Version Rules

| Change Type | Version Bump | Example |
|-------------|--------------|---------|
| Breaking changes | Major | v1 -> v2 |
| New endpoints | Minor | v2.1 -> v2.2 |
| New optional fields | Minor | v2.2 -> v2.3 |
| Bug fixes | Patch | v2.3.0 -> v2.3.1 |
| Security fixes | Patch | v2.3.1 -> v2.3.2 |
| Documentation | None | - |

### Version Headers

```typescript
// middleware/version-headers.ts
export function addVersionHeaders(req: Request, res: Response, next: NextFunction) {
  // API version
  res.setHeader('X-API-Version', '2.3.1');

  // Deprecation warnings
  const deprecatedEndpoints = ['/v1/markets', '/v1/trades'];
  if (deprecatedEndpoints.some(e => req.path.startsWith(e))) {
    res.setHeader('Deprecation', 'true');
    res.setHeader('Sunset', 'Sat, 01 Jun 2025 00:00:00 GMT');
    res.setHeader('Link', '</v2/markets>; rel="successor-version"');
  }

  next();
}
```

### Version Response Envelope

```typescript
// Response envelope with version info
interface APIResponse<T> {
  data: T;
  meta: {
    version: string;
    deprecated?: boolean;
    deprecationDate?: string;
    successorVersion?: string;
    requestId: string;
    timestamp: string;
  };
  pagination?: {
    page: number;
    pageSize: number;
    totalItems: number;
    totalPages: number;
  };
}

// Example response
{
  "data": {
    "id": "market_abc123",
    "title": "Will BTC reach $100k by 2025?",
    "status": "ACTIVE"
  },
  "meta": {
    "version": "2.3.1",
    "requestId": "req_xyz789",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

---

## Breaking vs Non-Breaking Changes

### Non-Breaking Changes (Safe)

These changes can be made without version bump:

```typescript
// 1. Adding new optional fields to responses
// Before
interface Market {
  id: string;
  title: string;
  status: string;
}

// After (non-breaking)
interface Market {
  id: string;
  title: string;
  status: string;
  category?: string;        // New optional field
  featuredUntil?: string;   // New optional field
}

// 2. Adding new endpoints
router.get('/v2/markets/:id/analytics', ...);  // New endpoint

// 3. Adding new optional query parameters
// GET /v2/markets?category=sports  // New filter

// 4. Adding new enum values (with caution)
enum MarketStatus {
  ACTIVE = 'ACTIVE',
  RESOLVED = 'RESOLVED',
  PAUSED = 'PAUSED',        // New value - clients should handle unknown
}

// 5. Relaxing validation (accepting more input)
// Before: required field
// After: optional field with default

// 6. Increasing rate limits

// 7. Adding new headers
```

### Breaking Changes (Require Major Version)

```typescript
// 1. Removing or renaming fields
// Before
interface Market {
  id: string;
  title: string;
  endDate: string;  // Will be removed/renamed
}

// After (BREAKING)
interface Market {
  id: string;
  title: string;
  resolutionDate: string;  // Renamed from endDate
}

// 2. Changing field types
// Before
interface Trade {
  amount: number;  // number type
}

// After (BREAKING)
interface Trade {
  amount: string;  // Changed to string for precision
}

// 3. Removing endpoints
// DELETE /v1/users/:id/preferences  // Removed

// 4. Changing URL structure
// Before: /v1/users/{userId}/trades
// After:  /v2/trades?userId={userId}

// 5. Changing authentication requirements
// Before: No auth required
// After: Auth required

// 6. Changing error response format
// Before
{ "error": "Not found" }

// After (BREAKING)
{
  "error": {
    "code": "MARKET_NOT_FOUND",
    "message": "Market not found"
  }
}

// 7. Tightening validation
// Before: email field accepts any string
// After: email field validates format

// 8. Changing default values
// Before: pageSize default = 10
// After: pageSize default = 20

// 9. Removing enum values
```

### Change Compatibility Matrix

```
┌─────────────────────────────────────────────────────────────────┐
│                    Change Compatibility Matrix                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Change Type              │ Response │ Request │ Both           │
│  ─────────────────────────┼──────────┼─────────┼──────────────  │
│  Add optional field       │    ✅    │    ✅   │    ✅          │
│  Add required field       │    ✅    │    ❌   │    ❌          │
│  Remove field             │    ❌    │    ✅   │    ❌          │
│  Rename field             │    ❌    │    ❌   │    ❌          │
│  Change type              │    ❌    │    ❌   │    ❌          │
│  Add enum value           │    ⚠️    │    ✅   │    ⚠️          │
│  Remove enum value        │    ✅    │    ❌   │    ❌          │
│  Relax validation         │    ✅    │    ✅   │    ✅          │
│  Tighten validation       │    ✅    │    ❌   │    ❌          │
│  Change default           │    ⚠️    │    ⚠️   │    ⚠️          │
│                                                                  │
│  ✅ = Safe  ❌ = Breaking  ⚠️ = Potentially Breaking            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Deprecation Policy and Timeline

### Deprecation Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│                    API Deprecation Lifecycle                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ACTIVE ──► DEPRECATED ──► SUNSET ──► REMOVED                   │
│    │            │            │            │                      │
│    │      + 6 months    + 3 months    + 1 month                 │
│    │            │            │            │                      │
│    ▼            ▼            ▼            ▼                      │
│  Normal    Warnings      Limited       Gone                     │
│  operation  in docs      support                                │
│             + headers    Read-only?                             │
│                                                                  │
│  Timeline: 10 months from deprecation to removal                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Deprecation Phases

| Phase | Duration | Actions |
|-------|----------|---------|
| **Active** | Until deprecation | Normal operation |
| **Deprecated** | 6 months | Deprecation headers, documentation updates, migration guides |
| **Sunset** | 3 months | Reduced support, strong warnings, usage monitoring |
| **Removed** | After sunset | Endpoint returns 410 Gone |

### Deprecation Headers (RFC 8594)

```typescript
// middleware/deprecation.ts
interface DeprecationConfig {
  endpoint: string;
  deprecationDate: Date;
  sunsetDate: Date;
  successorEndpoint: string;
  reason: string;
}

const deprecations: DeprecationConfig[] = [
  {
    endpoint: '/v1/markets',
    deprecationDate: new Date('2024-06-01'),
    sunsetDate: new Date('2025-01-01'),
    successorEndpoint: '/v2/markets',
    reason: 'Migrated to v2 with improved response format',
  },
  {
    endpoint: '/v1/trades',
    deprecationDate: new Date('2024-06-01'),
    sunsetDate: new Date('2025-01-01'),
    successorEndpoint: '/v2/trades',
    reason: 'Migrated to v2 with pagination support',
  },
];

export function deprecationMiddleware(req: Request, res: Response, next: NextFunction) {
  const deprecation = deprecations.find(d =>
    req.path.startsWith(d.endpoint)
  );

  if (deprecation) {
    const now = new Date();

    // Check if past sunset date
    if (now > deprecation.sunsetDate) {
      return res.status(410).json({
        error: {
          code: 'ENDPOINT_REMOVED',
          message: `This endpoint was removed on ${deprecation.sunsetDate.toISOString()}`,
          successor: deprecation.successorEndpoint,
        },
      });
    }

    // Add deprecation headers
    res.setHeader('Deprecation', `@${Math.floor(deprecation.deprecationDate.getTime() / 1000)}`);
    res.setHeader('Sunset', deprecation.sunsetDate.toUTCString());
    res.setHeader('Link', `<${deprecation.successorEndpoint}>; rel="successor-version"`);

    // Add warning header
    res.setHeader('Warning', `299 - "This endpoint is deprecated. Please migrate to ${deprecation.successorEndpoint}"`);

    // Log deprecation usage for monitoring
    logger.warn({
      category: 'api.deprecation',
      endpoint: deprecation.endpoint,
      successorEndpoint: deprecation.successorEndpoint,
      clientId: req.headers['x-client-id'],
      userAgent: req.headers['user-agent'],
    }, 'Deprecated endpoint accessed');
  }

  next();
}
```

### Deprecation Communication

```typescript
// Deprecation announcement template
interface DeprecationAnnouncement {
  version: string;
  deprecationDate: string;
  sunsetDate: string;
  changes: {
    type: 'endpoint' | 'field' | 'parameter';
    path: string;
    description: string;
    replacement?: string;
    migrationGuide?: string;
  }[];
}

// Example announcement
const announcement: DeprecationAnnouncement = {
  version: 'v1',
  deprecationDate: '2024-06-01',
  sunsetDate: '2025-01-01',
  changes: [
    {
      type: 'endpoint',
      path: '/v1/markets',
      description: 'The v1 markets endpoint is deprecated',
      replacement: '/v2/markets',
      migrationGuide: 'https://docs.predictionmarket.com/migration/v1-to-v2',
    },
    {
      type: 'field',
      path: '/v1/markets/{id}.endDate',
      description: 'The endDate field is renamed to resolutionDate',
      replacement: 'resolutionDate',
    },
  ],
};
```

### Monitoring Deprecated Endpoint Usage

```typescript
// Track deprecated endpoint usage
import { Counter, Gauge } from 'prom-client';

const deprecatedEndpointUsage = new Counter({
  name: 'api_deprecated_endpoint_usage_total',
  help: 'Usage of deprecated endpoints',
  labelNames: ['endpoint', 'client_id', 'version'],
});

const deprecatedEndpointUniqueClients = new Gauge({
  name: 'api_deprecated_endpoint_unique_clients',
  help: 'Number of unique clients using deprecated endpoints',
  labelNames: ['endpoint'],
});

// Usage report
async function generateDeprecationReport(): Promise<DeprecationReport> {
  const results = await db.query(`
    SELECT
      endpoint,
      COUNT(*) as request_count,
      COUNT(DISTINCT client_id) as unique_clients,
      MAX(timestamp) as last_used
    FROM api_requests
    WHERE endpoint LIKE '/v1/%'
      AND timestamp > NOW() - INTERVAL '7 days'
    GROUP BY endpoint
    ORDER BY request_count DESC
  `);

  return {
    period: '7 days',
    endpoints: results.rows.map(row => ({
      endpoint: row.endpoint,
      requestCount: row.request_count,
      uniqueClients: row.unique_clients,
      lastUsed: row.last_used,
      daysUntilSunset: calculateDaysUntilSunset(row.endpoint),
    })),
  };
}
```

---

## Migration Guides for Clients

### Migration Guide Structure

```markdown
# Migration Guide: v1 to v2

## Overview

This guide helps you migrate from API v1 to v2. The v2 API includes improved
response formats, better pagination, and new features.

**Timeline:**
- v1 Deprecation: June 1, 2024
- v1 Sunset: January 1, 2025
- v1 Removal: February 1, 2025

## Breaking Changes Summary

| Change | v1 | v2 | Impact |
|--------|----|----|--------|
| Response envelope | Direct data | Wrapped with meta | High |
| Pagination | Offset-based | Cursor-based | Medium |
| Date format | Unix timestamp | ISO 8601 | Medium |
| Amount fields | Number | String (decimal) | High |

## Step-by-Step Migration

### 1. Update Base URL

```diff
- const API_BASE = 'https://api.predictionmarket.com/v1';
+ const API_BASE = 'https://api.predictionmarket.com/v2';
```

### 2. Handle New Response Format

```diff
// Before (v1)
- const markets = await fetch('/v1/markets').then(r => r.json());
- console.log(markets[0].title);

// After (v2)
+ const response = await fetch('/v2/markets').then(r => r.json());
+ const markets = response.data;
+ console.log(markets[0].title);
```

### 3. Update Pagination

```diff
// Before (v1) - Offset pagination
- const page2 = await fetch('/v1/markets?offset=20&limit=20');

// After (v2) - Cursor pagination
+ const page1 = await fetch('/v2/markets?pageSize=20');
+ const cursor = page1.pagination.nextCursor;
+ const page2 = await fetch(`/v2/markets?cursor=${cursor}&pageSize=20`);
```

### 4. Handle Decimal Amounts

```diff
// Before (v1)
- interface Trade {
-   amount: number;
- }
- const total = trades.reduce((sum, t) => sum + t.amount, 0);

// After (v2)
+ interface Trade {
+   amount: string;
+ }
+ import Decimal from 'decimal.js';
+ const total = trades.reduce(
+   (sum, t) => sum.plus(new Decimal(t.amount)),
+   new Decimal(0)
+ );
```

### 5. Update Date Handling

```diff
// Before (v1)
- const date = new Date(market.endDate * 1000);

// After (v2)
+ const date = new Date(market.resolutionDate);
```

## Testing Your Migration

1. Run against staging: `https://staging-api.predictionmarket.com/v2`
2. Use the migration validator tool
3. Test all endpoints your application uses

## Getting Help

- Documentation: https://docs.predictionmarket.com/v2
- Migration support: migration-support@predictionmarket.com
- Discord: #api-migration channel
```

### Migration Validator Tool

```typescript
// tools/migration-validator.ts
interface ValidationResult {
  endpoint: string;
  v1Response: any;
  v2Response: any;
  compatible: boolean;
  issues: string[];
}

async function validateMigration(
  v1BaseUrl: string,
  v2BaseUrl: string,
  endpoints: string[]
): Promise<ValidationResult[]> {
  const results: ValidationResult[] = [];

  for (const endpoint of endpoints) {
    const v1Response = await fetch(`${v1BaseUrl}${endpoint}`).then(r => r.json());
    const v2Response = await fetch(`${v2BaseUrl}${endpoint}`).then(r => r.json());

    const issues: string[] = [];

    // Check data structure
    if (v2Response.data === undefined) {
      issues.push('v2 response missing data wrapper');
    }

    // Compare field presence
    const v1Fields = Object.keys(Array.isArray(v1Response) ? v1Response[0] : v1Response);
    const v2Data = v2Response.data;
    const v2Fields = Object.keys(Array.isArray(v2Data) ? v2Data[0] : v2Data);

    const missingFields = v1Fields.filter(f => !v2Fields.includes(f));
    if (missingFields.length > 0) {
      issues.push(`Missing fields in v2: ${missingFields.join(', ')}`);
    }

    results.push({
      endpoint,
      v1Response,
      v2Response,
      compatible: issues.length === 0,
      issues,
    });
  }

  return results;
}

// CLI usage
// npx ts-node tools/migration-validator.ts --v1 https://api.pm.com/v1 --v2 https://api.pm.com/v2
```

---

## SDK Versioning Alignment

### SDK Version Strategy

```
API Version    SDK Version
v1.0.0    ──►  sdk-js@1.0.x, sdk-python@1.0.x
v1.1.0    ──►  sdk-js@1.1.x, sdk-python@1.1.x
v2.0.0    ──►  sdk-js@2.0.x, sdk-python@2.0.x
```

### SDK Package Structure

```
@predictionmarket/sdk
├── package.json
├── src/
│   ├── index.ts
│   ├── client.ts
│   ├── v1/
│   │   ├── index.ts
│   │   ├── markets.ts
│   │   └── trades.ts
│   └── v2/
│       ├── index.ts
│       ├── markets.ts
│       └── trades.ts
└── README.md
```

### SDK Implementation

```typescript
// sdk/src/client.ts
export interface ClientConfig {
  apiKey: string;
  baseUrl?: string;
  version?: 'v1' | 'v2';
  timeout?: number;
}

export class PredictionMarketClient {
  private config: Required<ClientConfig>;

  constructor(config: ClientConfig) {
    this.config = {
      baseUrl: config.baseUrl || 'https://api.predictionmarket.com',
      version: config.version || 'v2',
      timeout: config.timeout || 30000,
      apiKey: config.apiKey,
    };
  }

  // Version-specific client accessors
  get v1() {
    return {
      markets: new V1MarketsClient(this.config),
      trades: new V1TradesClient(this.config),
    };
  }

  get v2() {
    return {
      markets: new V2MarketsClient(this.config),
      trades: new V2TradesClient(this.config),
    };
  }

  // Default to configured version
  get markets() {
    return this.config.version === 'v1'
      ? this.v1.markets
      : this.v2.markets;
  }

  get trades() {
    return this.config.version === 'v1'
      ? this.v1.trades
      : this.v2.trades;
  }
}

// Usage
const client = new PredictionMarketClient({
  apiKey: 'pk_live_xxx',
  version: 'v2',
});

// Uses v2 by default
const markets = await client.markets.list();

// Explicit version access
const v1Markets = await client.v1.markets.list();
const v2Markets = await client.v2.markets.list();
```

### SDK Deprecation Warnings

```typescript
// sdk/src/v1/markets.ts
import { deprecate } from 'util';

export class V1MarketsClient {
  constructor(private config: ClientConfig) {
    // Warn on instantiation
    console.warn(
      '[DEPRECATED] V1MarketsClient is deprecated and will be removed in SDK v3.0.0. ' +
      'Please migrate to V2MarketsClient. See: https://docs.pm.com/migration'
    );
  }

  list = deprecate(
    async (params?: ListParams): Promise<Market[]> => {
      const response = await this.fetch('/v1/markets', params);
      return response;
    },
    'V1MarketsClient.list() is deprecated. Use V2MarketsClient.list() instead.',
    'PM-SDK-001'
  );
}
```

---

## API Changelog Format

### Changelog Structure

```markdown
# API Changelog

All notable changes to the API will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this API adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.3.0] - 2024-01-15

### Added
- New `GET /v2/markets/{id}/analytics` endpoint for market analytics
- `category` field to market response
- `cursor` parameter for pagination on list endpoints
- WebSocket support for real-time price updates

### Changed
- Improved rate limit headers with `X-RateLimit-Remaining`
- Enhanced error responses with `error.details` field

### Deprecated
- `offset` pagination parameter (use `cursor` instead)
- `GET /v2/markets/{id}/history` (use `/v2/markets/{id}/trades` instead)

### Fixed
- Incorrect timezone handling in `createdAt` field
- Rate limiting not applying correctly to WebSocket connections

## [2.2.0] - 2024-01-01

### Added
- `featured` filter for markets endpoint
- Support for market categories

### Security
- Fixed potential XSS in market description rendering

## [2.0.0] - 2023-12-01

### Breaking Changes
- Response format changed to envelope with `data` and `meta`
- `amount` fields changed from `number` to `string` for precision
- `endDate` renamed to `resolutionDate`
- Pagination changed from offset-based to cursor-based

### Migration Guide
See [v1 to v2 Migration Guide](./migration/v1-to-v2.md)
```

### Machine-Readable Changelog

```json
{
  "apiName": "Prediction Market API",
  "versions": [
    {
      "version": "2.3.0",
      "releaseDate": "2024-01-15",
      "changes": {
        "added": [
          {
            "type": "endpoint",
            "path": "GET /v2/markets/{id}/analytics",
            "description": "Market analytics endpoint"
          },
          {
            "type": "field",
            "path": "Market.category",
            "description": "Category field added to market response"
          }
        ],
        "deprecated": [
          {
            "type": "parameter",
            "path": "offset",
            "description": "Use cursor pagination instead",
            "replacement": "cursor",
            "sunsetDate": "2024-07-15"
          }
        ],
        "breaking": []
      }
    },
    {
      "version": "2.0.0",
      "releaseDate": "2023-12-01",
      "changes": {
        "breaking": [
          {
            "type": "response",
            "description": "Response format changed to envelope",
            "migrationGuide": "https://docs.pm.com/migration/v1-to-v2#response-format"
          },
          {
            "type": "field",
            "path": "Trade.amount",
            "description": "Type changed from number to string",
            "reason": "Precision for large decimal values"
          }
        ]
      }
    }
  ]
}
```

### Changelog Automation

```typescript
// tools/generate-changelog.ts
import { Octokit } from '@octokit/rest';

interface ChangelogEntry {
  type: 'added' | 'changed' | 'deprecated' | 'removed' | 'fixed' | 'security' | 'breaking';
  scope: string;
  description: string;
  pr?: number;
  author?: string;
}

async function generateChangelog(
  fromTag: string,
  toTag: string
): Promise<string> {
  const octokit = new Octokit({ auth: process.env.GITHUB_TOKEN });

  // Get commits between tags
  const comparison = await octokit.repos.compareCommits({
    owner: 'org',
    repo: 'prediction-market-api',
    base: fromTag,
    head: toTag,
  });

  const entries: ChangelogEntry[] = [];

  for (const commit of comparison.data.commits) {
    // Parse conventional commit message
    const match = commit.commit.message.match(
      /^(feat|fix|docs|style|refactor|perf|test|chore|breaking)(\(.+\))?: (.+)/
    );

    if (match) {
      const [, type, scope, description] = match;
      entries.push({
        type: mapCommitType(type),
        scope: scope?.replace(/[()]/g, '') || 'general',
        description,
        author: commit.author?.login,
      });
    }
  }

  return formatChangelog(toTag, entries);
}

function mapCommitType(type: string): ChangelogEntry['type'] {
  const mapping: Record<string, ChangelogEntry['type']> = {
    feat: 'added',
    fix: 'fixed',
    breaking: 'breaking',
    perf: 'changed',
    refactor: 'changed',
    docs: 'changed',
  };
  return mapping[type] || 'changed';
}
```

---

## Backward Compatibility Patterns

### Pattern 1: Field Aliasing

```typescript
// Support both old and new field names
interface MarketResponseV1 {
  endDate: number;  // Unix timestamp
}

interface MarketResponseV2 {
  resolutionDate: string;  // ISO 8601
}

function transformMarketResponse(market: DBMarket, version: string) {
  const base = {
    id: market.id,
    title: market.title,
    status: market.status,
  };

  if (version === 'v1') {
    return {
      ...base,
      endDate: Math.floor(market.resolution_date.getTime() / 1000),
    };
  }

  return {
    ...base,
    resolutionDate: market.resolution_date.toISOString(),
    // Include deprecated field with warning
    endDate: Math.floor(market.resolution_date.getTime() / 1000),
  };
}
```

### Pattern 2: Graceful Degradation

```typescript
// Accept both old and new request formats
interface CreateMarketRequestV1 {
  title: string;
  endDate: number;  // Unix timestamp
}

interface CreateMarketRequestV2 {
  title: string;
  resolutionDate: string;  // ISO 8601
}

function parseCreateMarketRequest(body: any): CreateMarketParams {
  // Support both formats
  const resolutionDate = body.resolutionDate
    ? new Date(body.resolutionDate)
    : body.endDate
      ? new Date(body.endDate * 1000)
      : null;

  if (!resolutionDate) {
    throw new ValidationError('Either resolutionDate or endDate is required');
  }

  return {
    title: body.title,
    resolutionDate,
  };
}
```

### Pattern 3: Response Transformation Layer

```typescript
// middleware/response-transformer.ts
interface TransformConfig {
  version: string;
  endpoint: string;
  transform: (data: any) => any;
}

const transforms: TransformConfig[] = [
  {
    version: 'v1',
    endpoint: '/markets',
    transform: (data) => {
      // v1 returns array directly
      if (Array.isArray(data.data)) {
        return data.data.map(transformMarketV1);
      }
      return transformMarketV1(data.data);
    },
  },
  {
    version: 'v2',
    endpoint: '/markets',
    transform: (data) => {
      // v2 returns wrapped response
      return {
        data: Array.isArray(data.data)
          ? data.data.map(transformMarketV2)
          : transformMarketV2(data.data),
        meta: data.meta,
        pagination: data.pagination,
      };
    },
  },
];

export function responseTransformer(req: Request, res: Response, next: NextFunction) {
  const originalJson = res.json.bind(res);

  res.json = (body: any) => {
    const version = req.path.split('/')[1];  // Extract v1 or v2
    const endpoint = req.path.replace(/^\/v\d+/, '');

    const transform = transforms.find(
      t => t.version === version && endpoint.startsWith(t.endpoint)
    );

    if (transform) {
      body = transform.transform(body);
    }

    return originalJson(body);
  };

  next();
}
```

### Pattern 4: Version-Specific Controllers

```typescript
// controllers/markets/base.ts
export abstract class MarketsController {
  protected marketService: MarketService;

  constructor() {
    this.marketService = new MarketService();
  }

  abstract list(req: Request, res: Response): Promise<void>;
  abstract get(req: Request, res: Response): Promise<void>;
  abstract create(req: Request, res: Response): Promise<void>;
}

// controllers/markets/v1.ts
export class MarketsControllerV1 extends MarketsController {
  async list(req: Request, res: Response) {
    const { offset = 0, limit = 20 } = req.query;
    const markets = await this.marketService.list({ offset, limit });

    // V1 returns array directly
    res.json(markets.map(this.transformMarket));
  }

  private transformMarket(market: Market) {
    return {
      id: market.id,
      title: market.title,
      endDate: Math.floor(market.resolutionDate.getTime() / 1000),
    };
  }
}

// controllers/markets/v2.ts
export class MarketsControllerV2 extends MarketsController {
  async list(req: Request, res: Response) {
    const { cursor, pageSize = 20 } = req.query;
    const result = await this.marketService.listWithCursor({ cursor, pageSize });

    // V2 returns wrapped response
    res.json({
      data: result.items.map(this.transformMarket),
      meta: {
        version: '2.0.0',
        requestId: req.id,
      },
      pagination: {
        nextCursor: result.nextCursor,
        hasMore: result.hasMore,
      },
    });
  }

  private transformMarket(market: Market) {
    return {
      id: market.id,
      title: market.title,
      resolutionDate: market.resolutionDate.toISOString(),
    };
  }
}
```

### Pattern 5: API Gateway Versioning

```yaml
# api-gateway/routes.yaml
routes:
  # V1 routes with transformation
  - path: /v1/markets
    service: markets-service
    plugins:
      - response-transformer:
          remove_wrapper: true
          transform_dates: unix

  # V2 routes (native)
  - path: /v2/markets
    service: markets-service
    plugins:
      - response-transformer:
          add_meta: true
          transform_dates: iso8601

  # Header-based versioning support
  - path: /markets
    service: markets-service
    plugins:
      - version-router:
          header: X-API-Version
          default: v2
          routes:
            v1: /v1/markets
            v2: /v2/markets
```

---

## References

- [API Versioning Best Practices 2024](https://liblab.com/blog/api-versioning-best-practices)
- [REST API Versioning Strategies](https://restfulapi.net/versioning/)
- [Stripe API Versioning](https://stripe.com/docs/api/versioning)
- [GitHub API Versioning](https://docs.github.com/en/rest/overview/api-versions)
- [Google AIP-185: API Versioning](https://google.aip.dev/185)
- [RFC 8594 - Sunset Header](https://www.rfc-editor.org/rfc/rfc8594)
- [Postman API Versioning Guide](https://www.postman.com/api-platform/api-versioning/)
