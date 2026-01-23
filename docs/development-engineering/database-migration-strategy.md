# Database Migration Strategy

## Comprehensive Database Management for Prediction Market Platform

This document outlines the database migration strategy, versioning approach, and data management practices for a high-performance trading platform.

---

## Table of Contents

1. [Schema Versioning Approach](#schema-versioning-approach)
2. [Migration Tool Selection](#migration-tool-selection)
3. [Zero-Downtime Migration Patterns](#zero-downtime-migration-patterns)
4. [Rollback Procedures](#rollback-procedures)
5. [Data Backup Strategy](#data-backup-strategy)
6. [Point-in-Time Recovery](#point-in-time-recovery)
7. [Read Replica Management](#read-replica-management)
8. [Sharding Considerations](#sharding-considerations)
9. [TimescaleDB for Time-Series Data](#timescaledb-for-time-series-data)

---

## Schema Versioning Approach

### Versioning Philosophy

Our schema versioning follows these principles:

1. **Forward-only migrations**: All changes are additive where possible
2. **Semantic versioning**: Major.Minor.Patch for schema versions
3. **Immutable migrations**: Once applied, migrations are never modified
4. **Backward compatibility**: Maintain compatibility for N-1 application versions

### Version Numbering Convention

```
V{MAJOR}.{MINOR}.{PATCH}__{description}.sql

Examples:
V1.0.0__initial_schema.sql
V1.1.0__add_market_categories.sql
V1.1.1__add_index_market_status.sql
V1.2.0__add_trading_history.sql
V2.0.0__restructure_positions.sql (breaking change)
```

### Migration Types

| Type | Version Impact | Example |
|------|----------------|---------|
| **Additive** | Minor/Patch | Add column, add table, add index |
| **Backward Compatible** | Minor | Add nullable column, add constraint with default |
| **Breaking** | Major | Remove column, change data type, rename |
| **Data Migration** | Minor | Backfill data, transform existing records |

### Migration File Structure

```
database/
├── migrations/
│   ├── V1.0.0__initial_schema.sql
│   ├── V1.1.0__add_market_categories.sql
│   ├── V1.1.1__add_index_market_status.sql
│   ├── V1.2.0__add_trading_history.sql
│   └── V2.0.0__restructure_positions.sql
├── seeds/
│   ├── development/
│   │   └── 001_sample_markets.sql
│   └── test/
│       └── 001_test_fixtures.sql
├── rollbacks/
│   ├── R1.1.0__rollback_market_categories.sql
│   └── R1.2.0__rollback_trading_history.sql
└── scripts/
    ├── backup.sh
    ├── restore.sh
    └── validate-migration.sh
```

---

## Migration Tool Selection

### Prisma (Recommended for Primary)

Prisma offers a declarative schema-first approach with excellent TypeScript integration.

```prisma
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id            String    @id @default(uuid())
  address       String    @unique
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  positions     Position[]
  trades        Trade[]
}

model Market {
  id            String    @id @default(uuid())
  title         String
  description   String
  category      String
  status        MarketStatus @default(ACTIVE)
  createdAt     DateTime  @default(now())
  resolutionTime DateTime?
  outcome       Int?
  positions     Position[]
  trades        Trade[]
  priceHistory  PriceHistory[]

  @@index([status])
  @@index([category])
  @@index([createdAt])
}

model Position {
  id            String    @id @default(uuid())
  user          User      @relation(fields: [userId], references: [id])
  userId        String
  market        Market    @relation(fields: [marketId], references: [id])
  marketId      String
  outcome       Int
  shares        Decimal   @db.Decimal(78, 18)
  costBasis     Decimal   @db.Decimal(78, 18)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@unique([userId, marketId, outcome])
  @@index([userId])
  @@index([marketId])
}

model Trade {
  id            String    @id @default(uuid())
  user          User      @relation(fields: [userId], references: [id])
  userId        String
  market        Market    @relation(fields: [marketId], references: [id])
  marketId      String
  outcome       Int
  side          TradeSide
  shares        Decimal   @db.Decimal(78, 18)
  price         Decimal   @db.Decimal(78, 18)
  cost          Decimal   @db.Decimal(78, 18)
  txHash        String    @unique
  blockNumber   BigInt
  timestamp     DateTime
  createdAt     DateTime  @default(now())

  @@index([userId])
  @@index([marketId])
  @@index([timestamp])
  @@index([txHash])
}

enum MarketStatus {
  PENDING
  ACTIVE
  PAUSED
  RESOLVED
  DISPUTED
}

enum TradeSide {
  BUY
  SELL
}
```

### Prisma Migration Commands

```bash
# Generate migration from schema changes
npx prisma migrate dev --name add_market_categories

# Apply migrations in production
npx prisma migrate deploy

# Reset database (development only)
npx prisma migrate reset

# Generate Prisma Client
npx prisma generate

# View migration status
npx prisma migrate status
```

### Flyway (Alternative for Complex Migrations)

Flyway provides more control for complex SQL migrations.

```yaml
# flyway.conf
flyway.url=jdbc:postgresql://localhost:5432/prediction_market
flyway.user=${DB_USER}
flyway.password=${DB_PASSWORD}
flyway.schemas=public
flyway.locations=filesystem:./migrations
flyway.baselineOnMigrate=true
flyway.validateOnMigrate=true
flyway.outOfOrder=false
flyway.cleanDisabled=true  # Prevent accidental clean in production
```

```sql
-- V1.2.0__add_trading_history.sql
-- Add trading history table with proper indexing

CREATE TABLE IF NOT EXISTS trading_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    market_id UUID NOT NULL REFERENCES markets(id),
    outcome INTEGER NOT NULL,
    side VARCHAR(4) NOT NULL CHECK (side IN ('BUY', 'SELL')),
    shares NUMERIC(78, 18) NOT NULL,
    price NUMERIC(78, 18) NOT NULL,
    cost NUMERIC(78, 18) NOT NULL,
    tx_hash VARCHAR(66) UNIQUE NOT NULL,
    block_number BIGINT NOT NULL,
    timestamp TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create indexes
CREATE INDEX idx_trading_history_user ON trading_history(user_id);
CREATE INDEX idx_trading_history_market ON trading_history(market_id);
CREATE INDEX idx_trading_history_timestamp ON trading_history(timestamp DESC);
CREATE INDEX idx_trading_history_tx ON trading_history(tx_hash);

-- Partition by time (for TimescaleDB)
SELECT create_hypertable('trading_history', 'timestamp',
    chunk_time_interval => INTERVAL '1 day',
    if_not_exists => TRUE
);

COMMENT ON TABLE trading_history IS 'Historical record of all trades';
```

### TypeORM (Legacy/Alternative)

```typescript
// migrations/1705123456789-AddMarketCategories.ts
import { MigrationInterface, QueryRunner, TableColumn, TableIndex } from 'typeorm';

export class AddMarketCategories1705123456789 implements MigrationInterface {
  name = 'AddMarketCategories1705123456789';

  public async up(queryRunner: QueryRunner): Promise<void> {
    // Add category column
    await queryRunner.addColumn(
      'markets',
      new TableColumn({
        name: 'category',
        type: 'varchar',
        length: '100',
        isNullable: true, // Initially nullable for backward compatibility
      })
    );

    // Add index for category queries
    await queryRunner.createIndex(
      'markets',
      new TableIndex({
        name: 'IDX_MARKETS_CATEGORY',
        columnNames: ['category'],
      })
    );

    // Backfill existing data
    await queryRunner.query(`
      UPDATE markets
      SET category = 'uncategorized'
      WHERE category IS NULL
    `);

    // Make column required after backfill
    await queryRunner.changeColumn(
      'markets',
      'category',
      new TableColumn({
        name: 'category',
        type: 'varchar',
        length: '100',
        isNullable: false,
        default: "'uncategorized'",
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropIndex('markets', 'IDX_MARKETS_CATEGORY');
    await queryRunner.dropColumn('markets', 'category');
  }
}
```

### Tool Comparison

| Feature | Prisma | Flyway | TypeORM |
|---------|--------|--------|---------|
| Language | TypeScript | SQL | TypeScript |
| Type Safety | Excellent | N/A | Good |
| Migration Format | Generated SQL | Raw SQL | TypeScript/SQL |
| Rollback Support | Limited | Full | Full |
| Schema Drift Detection | Yes | Yes | Limited |
| Multi-database | PostgreSQL, MySQL, SQLite | 20+ databases | 10+ databases |
| Learning Curve | Low | Medium | Medium |
| Production Ready | Yes | Yes | Yes |

---

## Zero-Downtime Migration Patterns

### Expand-Contract Pattern

The expand-contract (also called parallel change) pattern enables schema changes without downtime.

```
Phase 1: EXPAND
┌─────────────────────────────────────────────────────────────────┐
│ Old Schema                    New Schema (added)                │
│ ┌─────────────┐              ┌─────────────┐                   │
│ │ Column A    │              │ Column A    │                   │
│ │ Column B    │    ──►       │ Column B    │                   │
│ └─────────────┘              │ Column C    │ (new, nullable)   │
│                              └─────────────┘                   │
└─────────────────────────────────────────────────────────────────┘

Phase 2: MIGRATE
┌─────────────────────────────────────────────────────────────────┐
│ Application writes to BOTH columns                              │
│ Background job backfills Column C from Column B                 │
└─────────────────────────────────────────────────────────────────┘

Phase 3: CONTRACT
┌─────────────────────────────────────────────────────────────────┐
│ Application reads from Column C only                            │
│ Remove Column B (if no longer needed)                          │
└─────────────────────────────────────────────────────────────────┘
```

### Example: Renaming a Column

```sql
-- Step 1: Add new column (Deploy 1)
ALTER TABLE markets ADD COLUMN resolution_timestamp TIMESTAMPTZ;

-- Step 2: Create trigger to sync data (Deploy 1)
CREATE OR REPLACE FUNCTION sync_resolution_time()
RETURNS TRIGGER AS $$
BEGIN
  NEW.resolution_timestamp = NEW.resolution_time;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_sync_resolution_time
BEFORE INSERT OR UPDATE ON markets
FOR EACH ROW EXECUTE FUNCTION sync_resolution_time();

-- Step 3: Backfill existing data (Background job)
UPDATE markets
SET resolution_timestamp = resolution_time
WHERE resolution_timestamp IS NULL
  AND resolution_time IS NOT NULL;

-- Step 4: Application uses new column (Deploy 2)
-- Update application to read/write resolution_timestamp

-- Step 5: Remove old column (Deploy 3, after verification)
DROP TRIGGER trg_sync_resolution_time ON markets;
DROP FUNCTION sync_resolution_time();
ALTER TABLE markets DROP COLUMN resolution_time;
```

### Online DDL Operations

```sql
-- Safe operations (no locks on PostgreSQL)
CREATE INDEX CONCURRENTLY idx_markets_category ON markets(category);
DROP INDEX CONCURRENTLY idx_markets_old;

-- Add column with default (PostgreSQL 11+, instant)
ALTER TABLE markets ADD COLUMN is_featured BOOLEAN DEFAULT false;

-- Add NOT NULL constraint safely
ALTER TABLE markets
ADD CONSTRAINT chk_category_not_null CHECK (category IS NOT NULL) NOT VALID;

-- Validate constraint in background
ALTER TABLE markets VALIDATE CONSTRAINT chk_category_not_null;
```

### Blue-Green Database Migration

```yaml
# blue-green-migration.yml
name: Blue-Green Database Migration

stages:
  - name: prepare-green
    steps:
      - create-green-database
      - restore-from-blue-snapshot
      - setup-logical-replication

  - name: apply-migrations
    steps:
      - run-migrations-on-green
      - validate-schema-green
      - run-smoke-tests-green

  - name: sync-and-switch
    steps:
      - wait-for-replication-sync
      - update-application-config
      - switch-traffic-to-green
      - verify-application-health

  - name: cleanup
    steps:
      - stop-replication
      - archive-blue-database
      - update-dns-records
```

### AWS Blue-Green Deployment

```typescript
// blue-green-migration.ts
import { RDSClient, CreateBlueGreenDeploymentCommand, SwitchoverBlueGreenDeploymentCommand } from '@aws-sdk/client-rds';

async function performBlueGreenMigration() {
  const rds = new RDSClient({ region: 'us-east-1' });

  // Step 1: Create Blue-Green deployment
  const createResponse = await rds.send(new CreateBlueGreenDeploymentCommand({
    BlueGreenDeploymentName: 'prediction-market-upgrade',
    Source: 'arn:aws:rds:us-east-1:123456789:db:prediction-market-blue',
    TargetEngineVersion: '15.4',
    TargetDBParameterGroupName: 'prediction-market-pg15',
  }));

  console.log('Blue-Green deployment created:', createResponse.BlueGreenDeployment?.BlueGreenDeploymentIdentifier);

  // Step 2: Wait for green environment to be ready
  await waitForGreenEnvironment(createResponse.BlueGreenDeployment?.BlueGreenDeploymentIdentifier);

  // Step 3: Apply migrations to green
  await applyMigrationsToGreen();

  // Step 4: Validate green environment
  await validateGreenEnvironment();

  // Step 5: Switchover (30 seconds downtime typical)
  const switchoverResponse = await rds.send(new SwitchoverBlueGreenDeploymentCommand({
    BlueGreenDeploymentIdentifier: createResponse.BlueGreenDeployment?.BlueGreenDeploymentIdentifier,
    SwitchoverTimeout: 300, // 5 minutes max
  }));

  console.log('Switchover complete:', switchoverResponse.BlueGreenDeployment?.Status);
}
```

---

## Rollback Procedures

### Automatic Rollback Triggers

```typescript
// migration-runner.ts
interface MigrationResult {
  success: boolean;
  version: string;
  duration: number;
  error?: Error;
}

interface RollbackConfig {
  autoRollbackOnError: boolean;
  healthCheckUrl: string;
  healthCheckTimeout: number;
  rollbackTimeout: number;
}

async function runMigrationWithRollback(
  migration: Migration,
  config: RollbackConfig
): Promise<MigrationResult> {
  const startTime = Date.now();

  try {
    // Run migration
    await migration.up();

    // Health check after migration
    const isHealthy = await performHealthCheck(config.healthCheckUrl, config.healthCheckTimeout);

    if (!isHealthy && config.autoRollbackOnError) {
      console.error('Health check failed, initiating rollback');
      await migration.down();
      throw new Error('Migration rolled back due to failed health check');
    }

    return {
      success: true,
      version: migration.version,
      duration: Date.now() - startTime,
    };
  } catch (error) {
    if (config.autoRollbackOnError) {
      try {
        await migration.down();
        console.log('Rollback completed successfully');
      } catch (rollbackError) {
        console.error('Rollback failed:', rollbackError);
        throw new Error(`Migration failed and rollback failed: ${rollbackError.message}`);
      }
    }

    return {
      success: false,
      version: migration.version,
      duration: Date.now() - startTime,
      error: error as Error,
    };
  }
}
```

### Manual Rollback Scripts

```sql
-- R1.2.0__rollback_trading_history.sql
-- Rollback script for V1.2.0__add_trading_history.sql

-- Drop hypertable (TimescaleDB)
DROP TABLE IF EXISTS trading_history CASCADE;

-- Restore from backup if data needs to be preserved
-- pg_restore -d prediction_market -t trading_history backup.dump
```

### Rollback Checklist

```markdown
## Pre-Rollback Checklist

- [ ] Confirm rollback is necessary
- [ ] Notify stakeholders
- [ ] Document current state
- [ ] Verify backup availability
- [ ] Check application compatibility

## Rollback Steps

1. [ ] Put application in maintenance mode
2. [ ] Stop background jobs
3. [ ] Execute rollback migration
4. [ ] Deploy previous application version
5. [ ] Run smoke tests
6. [ ] Restore application traffic
7. [ ] Monitor for issues

## Post-Rollback

- [ ] Document incident
- [ ] Root cause analysis
- [ ] Update migration for retry
- [ ] Schedule retry window
```

---

## Data Backup Strategy

### Backup Types and Schedule

| Backup Type | Frequency | Retention | Storage |
|-------------|-----------|-----------|---------|
| Full Backup | Daily | 30 days | S3 Glacier |
| Incremental | Hourly | 7 days | S3 Standard |
| WAL Archive | Continuous | 7 days | S3 Standard-IA |
| Snapshot | Before migrations | 7 days | EBS/RDS |

### Automated Backup Configuration

```yaml
# backup-config.yml
backup:
  full:
    schedule: "0 2 * * *"  # 2 AM daily
    retention_days: 30
    storage_class: GLACIER
    compression: true
    encryption: AES-256

  incremental:
    schedule: "0 * * * *"  # Every hour
    retention_days: 7
    storage_class: STANDARD

  wal_archiving:
    enabled: true
    archive_command: "aws s3 cp %p s3://prediction-market-wal/%f"
    retention_days: 7

  pre_migration:
    enabled: true
    snapshot_type: manual
    retention_days: 7
```

### Backup Script

```bash
#!/bin/bash
# backup.sh - Database backup script

set -euo pipefail

# Configuration
DB_HOST="${DB_HOST:-localhost}"
DB_NAME="${DB_NAME:-prediction_market}"
DB_USER="${DB_USER:-postgres}"
BACKUP_DIR="/backups"
S3_BUCKET="s3://prediction-market-backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${DATE}.sql.gz"

# Create backup directory
mkdir -p "${BACKUP_DIR}"

echo "Starting backup of ${DB_NAME}..."

# Full backup with compression
pg_dump -h "${DB_HOST}" -U "${DB_USER}" -d "${DB_NAME}" \
  --format=custom \
  --compress=9 \
  --verbose \
  --file="${BACKUP_FILE}"

# Calculate checksum
CHECKSUM=$(sha256sum "${BACKUP_FILE}" | awk '{print $1}')
echo "${CHECKSUM}" > "${BACKUP_FILE}.sha256"

# Upload to S3
aws s3 cp "${BACKUP_FILE}" "${S3_BUCKET}/full/${DATE}/" \
  --storage-class STANDARD_IA \
  --metadata "checksum=${CHECKSUM}"

aws s3 cp "${BACKUP_FILE}.sha256" "${S3_BUCKET}/full/${DATE}/"

# Cleanup local backup (keep last 3)
find "${BACKUP_DIR}" -name "*.sql.gz" -mtime +3 -delete

echo "Backup completed: ${BACKUP_FILE}"
echo "Checksum: ${CHECKSUM}"
```

### AWS RDS Automated Backups

```terraform
# rds-backup.tf
resource "aws_db_instance" "main" {
  identifier = "prediction-market-prod"

  # Backup configuration
  backup_retention_period = 30
  backup_window          = "03:00-04:00"
  delete_automated_backups = false

  # Enable automated backups to S3
  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]

  # Performance insights for backup monitoring
  performance_insights_enabled = true
  performance_insights_retention_period = 7
}

# Export snapshots to S3
resource "aws_db_snapshot_copy" "cross_region" {
  source_db_snapshot_identifier = aws_db_snapshot.daily.db_snapshot_arn
  target_db_snapshot_identifier = "prediction-market-${var.date}-dr"
  copy_tags                     = true
  kms_key_id                    = aws_kms_key.dr_key.arn

  # Copy to DR region
  destination_region = "us-west-2"
}
```

---

## Point-in-Time Recovery

### PITR Configuration

```sql
-- PostgreSQL WAL configuration for PITR
ALTER SYSTEM SET wal_level = 'replica';
ALTER SYSTEM SET archive_mode = 'on';
ALTER SYSTEM SET archive_command = 'aws s3 cp %p s3://prediction-market-wal/wal/%f';
ALTER SYSTEM SET archive_timeout = '60';  -- Archive every 60 seconds minimum

-- Reload configuration
SELECT pg_reload_conf();
```

### Recovery Script

```bash
#!/bin/bash
# restore-pitr.sh - Point-in-time recovery

set -euo pipefail

# Configuration
RECOVERY_TARGET_TIME="${1:-}"
BASE_BACKUP_DATE="${2:-$(date +%Y%m%d)}"
S3_BUCKET="s3://prediction-market-backups"
RESTORE_DIR="/restore"
PGDATA="/var/lib/postgresql/15/main"

if [ -z "$RECOVERY_TARGET_TIME" ]; then
  echo "Usage: $0 <recovery_target_time> [base_backup_date]"
  echo "Example: $0 '2024-01-15 14:30:00' 20240115"
  exit 1
fi

echo "Starting PITR to: ${RECOVERY_TARGET_TIME}"

# Stop PostgreSQL
systemctl stop postgresql

# Download base backup
aws s3 cp "${S3_BUCKET}/full/${BASE_BACKUP_DATE}/" "${RESTORE_DIR}/" --recursive

# Restore base backup
rm -rf "${PGDATA}"/*
pg_restore -D "${PGDATA}" "${RESTORE_DIR}/prediction_market_*.sql.gz"

# Create recovery configuration
cat > "${PGDATA}/recovery.signal" << EOF
EOF

cat > "${PGDATA}/postgresql.auto.conf" << EOF
restore_command = 'aws s3 cp s3://prediction-market-wal/wal/%f %p'
recovery_target_time = '${RECOVERY_TARGET_TIME}'
recovery_target_action = 'promote'
EOF

# Set permissions
chown -R postgres:postgres "${PGDATA}"

# Start PostgreSQL (will enter recovery mode)
systemctl start postgresql

echo "Recovery initiated. Check logs for progress."
echo "Once recovery is complete, PostgreSQL will promote to primary."
```

### AWS RDS PITR

```typescript
// pitr-restore.ts
import { RDSClient, RestoreDBInstanceToPointInTimeCommand } from '@aws-sdk/client-rds';

async function restoreToPointInTime(targetTime: Date) {
  const rds = new RDSClient({ region: 'us-east-1' });

  const response = await rds.send(new RestoreDBInstanceToPointInTimeCommand({
    SourceDBInstanceIdentifier: 'prediction-market-prod',
    TargetDBInstanceIdentifier: `prediction-market-pitr-${Date.now()}`,
    RestoreTime: targetTime,
    UseLatestRestorableTime: false,
    DBInstanceClass: 'db.r6g.xlarge',
    MultiAZ: true,
    PubliclyAccessible: false,
    VpcSecurityGroupIds: ['sg-12345678'],
    DBSubnetGroupName: 'prediction-market-subnet-group',
    Tags: [
      { Key: 'Environment', Value: 'recovery' },
      { Key: 'RestoreTime', Value: targetTime.toISOString() },
    ],
  }));

  console.log('PITR restore initiated:', response.DBInstance?.DBInstanceIdentifier);
  return response.DBInstance;
}
```

---

## Read Replica Management

### Replica Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Database Architecture                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                     ┌─────────────────┐                         │
│                     │    Primary      │                         │
│                     │   (us-east-1a)  │                         │
│                     │   Read/Write    │                         │
│                     └────────┬────────┘                         │
│                              │                                   │
│              ┌───────────────┼───────────────┐                  │
│              │               │               │                   │
│              ▼               ▼               ▼                   │
│     ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│     │  Replica 1  │  │  Replica 2  │  │  Replica 3  │          │
│     │ (us-east-1b)│  │ (us-east-1c)│  │ (us-west-2) │          │
│     │  Read Only  │  │  Read Only  │  │  DR/Read    │          │
│     └─────────────┘  └─────────────┘  └─────────────┘          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Replica Configuration

```typescript
// database-config.ts
import { Pool } from 'pg';

interface DatabaseConfig {
  primary: Pool;
  replicas: Pool[];
  replicaSelector: ReplicaSelector;
}

class ReplicaSelector {
  private currentIndex = 0;
  private replicas: Pool[];

  constructor(replicas: Pool[]) {
    this.replicas = replicas;
  }

  // Round-robin selection
  getNextReplica(): Pool {
    const replica = this.replicas[this.currentIndex];
    this.currentIndex = (this.currentIndex + 1) % this.replicas.length;
    return replica;
  }

  // Get replica with lowest latency
  async getLowestLatencyReplica(): Promise<Pool> {
    const latencies = await Promise.all(
      this.replicas.map(async (replica) => {
        const start = Date.now();
        await replica.query('SELECT 1');
        return { replica, latency: Date.now() - start };
      })
    );

    latencies.sort((a, b) => a.latency - b.latency);
    return latencies[0].replica;
  }
}

// Configuration
const dbConfig: DatabaseConfig = {
  primary: new Pool({
    host: process.env.DB_PRIMARY_HOST,
    database: process.env.DB_NAME,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    max: 20,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 10000,
  }),

  replicas: [
    new Pool({
      host: process.env.DB_REPLICA_1_HOST,
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      max: 50,
      idleTimeoutMillis: 30000,
    }),
    new Pool({
      host: process.env.DB_REPLICA_2_HOST,
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      max: 50,
      idleTimeoutMillis: 30000,
    }),
  ],

  replicaSelector: null as any, // Initialized below
};

dbConfig.replicaSelector = new ReplicaSelector(dbConfig.replicas);

// Query router
export async function query(
  sql: string,
  params: any[],
  options: { useReplica?: boolean } = {}
) {
  const pool = options.useReplica
    ? dbConfig.replicaSelector.getNextReplica()
    : dbConfig.primary;

  return pool.query(sql, params);
}

// Force primary for writes
export async function writeQuery(sql: string, params: any[]) {
  return dbConfig.primary.query(sql, params);
}

// Use replica for reads
export async function readQuery(sql: string, params: any[]) {
  return query(sql, params, { useReplica: true });
}
```

### Replica Lag Monitoring

```typescript
// replica-monitor.ts
import { Gauge } from 'prom-client';

const replicaLag = new Gauge({
  name: 'db_replica_lag_seconds',
  help: 'Replication lag in seconds',
  labelNames: ['replica'],
});

async function monitorReplicaLag() {
  // Check lag on each replica
  for (const [index, replica] of dbConfig.replicas.entries()) {
    try {
      const result = await replica.query(`
        SELECT
          CASE WHEN pg_last_wal_receive_lsn() = pg_last_wal_replay_lsn()
            THEN 0
            ELSE EXTRACT(EPOCH FROM now() - pg_last_xact_replay_timestamp())
          END AS lag_seconds
      `);

      const lagSeconds = parseFloat(result.rows[0].lag_seconds);
      replicaLag.set({ replica: `replica-${index + 1}` }, lagSeconds);

      // Alert if lag is too high
      if (lagSeconds > 30) {
        console.warn(`High replica lag on replica-${index + 1}: ${lagSeconds}s`);
      }
    } catch (error) {
      console.error(`Error checking replica-${index + 1} lag:`, error);
      replicaLag.set({ replica: `replica-${index + 1}` }, -1);
    }
  }
}

// Run every 10 seconds
setInterval(monitorReplicaLag, 10000);
```

---

## Sharding Considerations

### When to Shard

| Indicator | Threshold | Action |
|-----------|-----------|--------|
| Table Size | > 500 GB | Consider sharding |
| Write TPS | > 10,000 | Consider sharding |
| Query Latency | P99 > 100ms | Optimize, then shard |
| Connection Count | > 500 | Connection pooling first |

### Sharding Strategies

#### 1. Range-Based Sharding (Time)

```sql
-- Partition by time range
CREATE TABLE trades (
    id UUID NOT NULL,
    user_id UUID NOT NULL,
    market_id UUID NOT NULL,
    timestamp TIMESTAMPTZ NOT NULL,
    amount NUMERIC(78, 18) NOT NULL
) PARTITION BY RANGE (timestamp);

-- Create partitions
CREATE TABLE trades_2024_01 PARTITION OF trades
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE trades_2024_02 PARTITION OF trades
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- Auto-create future partitions
CREATE OR REPLACE FUNCTION create_trades_partition()
RETURNS void AS $$
DECLARE
    partition_date DATE;
    partition_name TEXT;
    start_date DATE;
    end_date DATE;
BEGIN
    partition_date := DATE_TRUNC('month', NOW() + INTERVAL '1 month');
    partition_name := 'trades_' || TO_CHAR(partition_date, 'YYYY_MM');
    start_date := partition_date;
    end_date := partition_date + INTERVAL '1 month';

    EXECUTE format(
        'CREATE TABLE IF NOT EXISTS %I PARTITION OF trades FOR VALUES FROM (%L) TO (%L)',
        partition_name, start_date, end_date
    );
END;
$$ LANGUAGE plpgsql;
```

#### 2. Hash-Based Sharding (User)

```sql
-- Partition by user hash
CREATE TABLE positions (
    id UUID NOT NULL,
    user_id UUID NOT NULL,
    market_id UUID NOT NULL,
    shares NUMERIC(78, 18) NOT NULL
) PARTITION BY HASH (user_id);

-- Create hash partitions
CREATE TABLE positions_p0 PARTITION OF positions FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE positions_p1 PARTITION OF positions FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE positions_p2 PARTITION OF positions FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE positions_p3 PARTITION OF positions FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

#### 3. Application-Level Sharding

```typescript
// shard-router.ts
import { createHash } from 'crypto';

interface ShardConfig {
  shardId: string;
  connectionString: string;
  pool: Pool;
}

class ShardRouter {
  private shards: Map<string, ShardConfig>;
  private shardCount: number;

  constructor(shardConfigs: ShardConfig[]) {
    this.shards = new Map(shardConfigs.map(s => [s.shardId, s]));
    this.shardCount = shardConfigs.length;
  }

  // Get shard for a user
  getShardForUser(userId: string): ShardConfig {
    const hash = createHash('md5').update(userId).digest('hex');
    const shardIndex = parseInt(hash.substring(0, 8), 16) % this.shardCount;
    return Array.from(this.shards.values())[shardIndex];
  }

  // Get shard for a market (might use different strategy)
  getShardForMarket(marketId: string): ShardConfig {
    // Markets might be on a single shard or replicated
    return Array.from(this.shards.values())[0]; // Primary shard for markets
  }

  // Execute query on specific shard
  async queryOnShard(shardId: string, sql: string, params: any[]): Promise<any> {
    const shard = this.shards.get(shardId);
    if (!shard) throw new Error(`Shard ${shardId} not found`);
    return shard.pool.query(sql, params);
  }

  // Execute query across all shards (fan-out)
  async queryAllShards(sql: string, params: any[]): Promise<any[]> {
    const results = await Promise.all(
      Array.from(this.shards.values()).map(shard =>
        shard.pool.query(sql, params)
      )
    );
    return results.flatMap(r => r.rows);
  }
}
```

---

## TimescaleDB for Time-Series Data

### Installation and Setup

```sql
-- Enable TimescaleDB extension
CREATE EXTENSION IF NOT EXISTS timescaledb;

-- Create time-series tables

-- Price history (high-frequency updates)
CREATE TABLE price_history (
    time TIMESTAMPTZ NOT NULL,
    market_id UUID NOT NULL,
    outcome INTEGER NOT NULL,
    price NUMERIC(18, 8) NOT NULL,
    volume NUMERIC(78, 18) NOT NULL,
    open_interest NUMERIC(78, 18) NOT NULL
);

-- Convert to hypertable
SELECT create_hypertable('price_history', 'time',
    chunk_time_interval => INTERVAL '1 hour'
);

-- Add compression policy
ALTER TABLE price_history SET (
    timescaledb.compress,
    timescaledb.compress_segmentby = 'market_id, outcome'
);

SELECT add_compression_policy('price_history', INTERVAL '7 days');

-- Add retention policy (keep 1 year of raw data)
SELECT add_retention_policy('price_history', INTERVAL '1 year');

-- Create indexes
CREATE INDEX idx_price_history_market_time ON price_history (market_id, time DESC);
CREATE INDEX idx_price_history_time ON price_history (time DESC);
```

### Continuous Aggregates

```sql
-- Create continuous aggregate for OHLCV data
CREATE MATERIALIZED VIEW price_ohlcv_1m
WITH (timescaledb.continuous) AS
SELECT
    time_bucket('1 minute', time) AS bucket,
    market_id,
    outcome,
    first(price, time) AS open,
    max(price) AS high,
    min(price) AS low,
    last(price, time) AS close,
    sum(volume) AS volume
FROM price_history
GROUP BY bucket, market_id, outcome;

-- Add refresh policy
SELECT add_continuous_aggregate_policy('price_ohlcv_1m',
    start_offset => INTERVAL '1 hour',
    end_offset => INTERVAL '1 minute',
    schedule_interval => INTERVAL '1 minute'
);

-- Hourly aggregates
CREATE MATERIALIZED VIEW price_ohlcv_1h
WITH (timescaledb.continuous) AS
SELECT
    time_bucket('1 hour', bucket) AS bucket,
    market_id,
    outcome,
    first(open, bucket) AS open,
    max(high) AS high,
    min(low) AS low,
    last(close, bucket) AS close,
    sum(volume) AS volume
FROM price_ohlcv_1m
GROUP BY time_bucket('1 hour', bucket), market_id, outcome;

SELECT add_continuous_aggregate_policy('price_ohlcv_1h',
    start_offset => INTERVAL '1 day',
    end_offset => INTERVAL '1 hour',
    schedule_interval => INTERVAL '1 hour'
);

-- Daily aggregates
CREATE MATERIALIZED VIEW price_ohlcv_1d
WITH (timescaledb.continuous) AS
SELECT
    time_bucket('1 day', bucket) AS bucket,
    market_id,
    outcome,
    first(open, bucket) AS open,
    max(high) AS high,
    min(low) AS low,
    last(close, bucket) AS close,
    sum(volume) AS volume
FROM price_ohlcv_1h
GROUP BY time_bucket('1 day', bucket), market_id, outcome;

SELECT add_continuous_aggregate_policy('price_ohlcv_1d',
    start_offset => INTERVAL '7 days',
    end_offset => INTERVAL '1 day',
    schedule_interval => INTERVAL '1 day'
);
```

### Querying Time-Series Data

```typescript
// timescale-queries.ts
import { Pool } from 'pg';

const pool = new Pool({ connectionString: process.env.TIMESCALE_URL });

// Get recent price history
async function getPriceHistory(marketId: string, outcome: number, interval: string, limit: number) {
  const query = `
    SELECT
      bucket,
      open,
      high,
      low,
      close,
      volume
    FROM price_ohlcv_${interval}
    WHERE market_id = $1 AND outcome = $2
    ORDER BY bucket DESC
    LIMIT $3
  `;

  const result = await pool.query(query, [marketId, outcome, limit]);
  return result.rows;
}

// Get volume over time
async function getVolumeTimeSeries(marketId: string, startTime: Date, endTime: Date) {
  const query = `
    SELECT
      time_bucket('1 hour', time) AS bucket,
      sum(volume) AS volume
    FROM price_history
    WHERE market_id = $1
      AND time >= $2
      AND time < $3
    GROUP BY bucket
    ORDER BY bucket
  `;

  const result = await pool.query(query, [marketId, startTime, endTime]);
  return result.rows;
}

// Get market statistics
async function getMarketStats(marketId: string) {
  const query = `
    SELECT
      outcome,
      last(price, time) AS current_price,
      first(price, time) AS start_price,
      max(price) AS high_24h,
      min(price) AS low_24h,
      sum(volume) AS volume_24h,
      count(*) AS trade_count_24h
    FROM price_history
    WHERE market_id = $1
      AND time >= NOW() - INTERVAL '24 hours'
    GROUP BY outcome
  `;

  const result = await pool.query(query, [marketId]);
  return result.rows;
}
```

### Migration from PostgreSQL to TimescaleDB

```sql
-- Migration script for existing data

-- 1. Backup existing table
CREATE TABLE trades_backup AS SELECT * FROM trades;

-- 2. Create new hypertable
CREATE TABLE trades_new (
    id UUID NOT NULL,
    user_id UUID NOT NULL,
    market_id UUID NOT NULL,
    outcome INTEGER NOT NULL,
    side VARCHAR(4) NOT NULL,
    shares NUMERIC(78, 18) NOT NULL,
    price NUMERIC(78, 18) NOT NULL,
    cost NUMERIC(78, 18) NOT NULL,
    tx_hash VARCHAR(66) UNIQUE NOT NULL,
    block_number BIGINT NOT NULL,
    timestamp TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

SELECT create_hypertable('trades_new', 'timestamp',
    chunk_time_interval => INTERVAL '1 day',
    migrate_data => false
);

-- 3. Copy data in batches
DO $$
DECLARE
    batch_size INT := 100000;
    total_rows INT;
    processed INT := 0;
BEGIN
    SELECT COUNT(*) INTO total_rows FROM trades_backup;

    WHILE processed < total_rows LOOP
        INSERT INTO trades_new
        SELECT * FROM trades_backup
        ORDER BY timestamp
        OFFSET processed LIMIT batch_size;

        processed := processed + batch_size;
        RAISE NOTICE 'Processed % of % rows', processed, total_rows;
        COMMIT;
    END LOOP;
END $$;

-- 4. Swap tables
ALTER TABLE trades RENAME TO trades_old;
ALTER TABLE trades_new RENAME TO trades;

-- 5. Recreate indexes
CREATE INDEX idx_trades_user ON trades(user_id, timestamp DESC);
CREATE INDEX idx_trades_market ON trades(market_id, timestamp DESC);

-- 6. Enable compression
ALTER TABLE trades SET (
    timescaledb.compress,
    timescaledb.compress_segmentby = 'market_id, user_id'
);

SELECT add_compression_policy('trades', INTERVAL '7 days');

-- 7. Cleanup after verification
-- DROP TABLE trades_old;
-- DROP TABLE trades_backup;
```

---

## References

- [Prisma Documentation](https://www.prisma.io/docs/orm)
- [Flyway Documentation](https://flywaydb.org/documentation)
- [TypeORM Migration Guide](https://typeorm.io/migrations)
- [TimescaleDB Documentation](https://docs.timescale.com/)
- [AWS RDS Blue-Green Deployments](https://aws.amazon.com/blogs/database/how-wiz-achieved-near-zero-downtime-for-amazon-aurora-postgresql-major-version-upgrades-at-scale-using-aurora-blue-green-deployments/)
- [PostgreSQL Partitioning](https://www.postgresql.org/docs/current/ddl-partitioning.html)
- [Zero-Downtime PostgreSQL Migrations](https://www.instantdb.com/essays/pg_upgrade)
