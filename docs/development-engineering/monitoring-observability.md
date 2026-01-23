# Monitoring and Observability

## Comprehensive Observability Strategy for Prediction Market Platform

This document outlines the monitoring, logging, metrics, and observability practices for a blockchain-based prediction market platform.

---

## Table of Contents

1. [Observability Overview](#observability-overview)
2. [Logging Strategy](#logging-strategy)
3. [Log Aggregation](#log-aggregation)
4. [Metrics Collection](#metrics-collection)
5. [Distributed Tracing](#distributed-tracing)
6. [Alerting Rules and Thresholds](#alerting-rules-and-thresholds)
7. [Dashboard Designs](#dashboard-designs)
8. [On-Chain Monitoring](#on-chain-monitoring)
9. [SLIs, SLOs, and SLAs](#slis-slos-and-slas)
10. [Incident Severity Levels](#incident-severity-levels)
11. [Runbooks](#runbooks)

---

## Observability Overview

### The Three Pillars of Observability

```
┌─────────────────────────────────────────────────────────────────┐
│                    Observability Platform                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────┐   │
│  │    LOGS       │  │    METRICS    │  │     TRACES        │   │
│  │               │  │               │  │                   │   │
│  │  What         │  │  How much     │  │  Where/When       │   │
│  │  happened     │  │  of something │  │  requests flow    │   │
│  │               │  │               │  │                   │   │
│  │  Structured   │  │  Time-series  │  │  Distributed      │   │
│  │  Events       │  │  Data         │  │  Context          │   │
│  └───────────────┘  └───────────────┘  └───────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   CORRELATION                            │   │
│  │  Trace ID → Logs → Metrics → Root Cause Analysis        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Observability Stack

| Component | Primary Tool | Alternative |
|-----------|-------------|-------------|
| Logging | Datadog Logs | ELK Stack |
| Metrics | Prometheus + Grafana | Datadog Metrics |
| Tracing | Datadog APM | Jaeger |
| On-chain | Tenderly | Forta |
| Alerts | PagerDuty | Opsgenie |
| Dashboards | Grafana | Datadog |

---

## Logging Strategy

### Structured Logging Format

All logs should be structured JSON for easy parsing and querying.

```typescript
// logger.ts
import pino from 'pino';

interface LogContext {
  service: string;
  version: string;
  environment: string;
  traceId?: string;
  spanId?: string;
  userId?: string;
  marketId?: string;
  transactionHash?: string;
}

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
    bindings: () => ({}),
  },
  base: {
    service: process.env.SERVICE_NAME,
    version: process.env.APP_VERSION,
    environment: process.env.NODE_ENV,
  },
  timestamp: pino.stdTimeFunctions.isoTime,
  redact: {
    paths: ['req.headers.authorization', 'privateKey', 'password', 'secret'],
    censor: '[REDACTED]',
  },
});

// Example log output:
// {
//   "level": "info",
//   "time": "2024-01-15T10:30:00.000Z",
//   "service": "api",
//   "version": "1.2.3",
//   "environment": "production",
//   "traceId": "abc123",
//   "msg": "Trade executed successfully",
//   "userId": "user_456",
//   "marketId": "market_789",
//   "amount": "100.00",
//   "outcome": "YES",
//   "txHash": "0x..."
// }

export default logger;
```

### Log Levels

| Level | When to Use | Example |
|-------|-------------|---------|
| `fatal` | Application crash | Database connection lost |
| `error` | Operation failed | Trade execution failed |
| `warn` | Unexpected but recoverable | Rate limit approaching |
| `info` | Business events | User placed trade |
| `debug` | Diagnostic information | Cache hit/miss |
| `trace` | Detailed debugging | Function entry/exit |

### Log Categories

```typescript
// Structured log categories
enum LogCategory {
  // Business Events
  TRADE_EXECUTED = 'trade.executed',
  TRADE_FAILED = 'trade.failed',
  MARKET_CREATED = 'market.created',
  MARKET_RESOLVED = 'market.resolved',
  USER_REGISTERED = 'user.registered',

  // System Events
  API_REQUEST = 'api.request',
  API_RESPONSE = 'api.response',
  CACHE_HIT = 'cache.hit',
  CACHE_MISS = 'cache.miss',

  // Blockchain Events
  TX_SUBMITTED = 'blockchain.tx.submitted',
  TX_CONFIRMED = 'blockchain.tx.confirmed',
  TX_FAILED = 'blockchain.tx.failed',
  BLOCK_PROCESSED = 'blockchain.block.processed',

  // Security Events
  AUTH_SUCCESS = 'auth.success',
  AUTH_FAILURE = 'auth.failure',
  RATE_LIMITED = 'security.rate_limited',
  SUSPICIOUS_ACTIVITY = 'security.suspicious',
}

// Example usage
logger.info({
  category: LogCategory.TRADE_EXECUTED,
  traceId: ctx.traceId,
  userId: user.id,
  marketId: market.id,
  side: 'BUY',
  outcome: 'YES',
  amount: trade.amount,
  price: trade.price,
  txHash: receipt.transactionHash,
  gasUsed: receipt.gasUsed,
  blockNumber: receipt.blockNumber,
}, 'Trade executed successfully');
```

### Request/Response Logging

```typescript
// Express middleware for request logging
import { Request, Response, NextFunction } from 'express';
import { v4 as uuidv4 } from 'uuid';

export function requestLogger(req: Request, res: Response, next: NextFunction) {
  const traceId = req.headers['x-trace-id'] as string || uuidv4();
  const startTime = Date.now();

  // Attach trace ID to request
  req.traceId = traceId;
  res.setHeader('x-trace-id', traceId);

  // Log request
  logger.info({
    category: 'api.request',
    traceId,
    method: req.method,
    path: req.path,
    query: req.query,
    userAgent: req.headers['user-agent'],
    ip: req.ip,
    userId: req.user?.id,
  }, `Incoming ${req.method} ${req.path}`);

  // Log response on finish
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    const level = res.statusCode >= 500 ? 'error' :
                  res.statusCode >= 400 ? 'warn' : 'info';

    logger[level]({
      category: 'api.response',
      traceId,
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      duration,
      contentLength: res.get('content-length'),
    }, `${req.method} ${req.path} ${res.statusCode} ${duration}ms`);
  });

  next();
}
```

---

## Log Aggregation

### ELK Stack Configuration

```yaml
# docker-compose.elk.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=true
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    healthcheck:
      test: curl -s http://localhost:9200 >/dev/null || exit 1
      interval: 30s
      timeout: 10s
      retries: 5

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline
      - ./logstash/config:/usr/share/logstash/config
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      elasticsearch:
        condition: service_healthy

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
    ports:
      - "5601:5601"
    depends_on:
      elasticsearch:
        condition: service_healthy

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.0
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    depends_on:
      - logstash

volumes:
  elasticsearch-data:
```

### Logstash Pipeline

```ruby
# logstash/pipeline/prediction-market.conf

input {
  beats {
    port => 5044
  }

  # Direct input from applications
  tcp {
    port => 5000
    codec => json
  }
}

filter {
  # Parse JSON logs
  if [message] =~ /^\{/ {
    json {
      source => "message"
    }
  }

  # Add derived fields
  if [traceId] {
    mutate {
      add_field => { "[@metadata][trace_id]" => "%{traceId}" }
    }
  }

  # Parse blockchain-specific fields
  if [txHash] {
    mutate {
      add_field => {
        "blockchain.txHash" => "%{txHash}"
        "blockchain.network" => "polygon"
      }
    }
  }

  # Categorize by service
  if [service] == "api" {
    mutate {
      add_field => { "[@metadata][index_prefix]" => "api" }
    }
  } else if [service] == "indexer" {
    mutate {
      add_field => { "[@metadata][index_prefix]" => "indexer" }
    }
  }

  # Geoip for user IPs
  if [ip] {
    geoip {
      source => "ip"
      target => "geoip"
    }
  }

  # Remove sensitive fields
  mutate {
    remove_field => ["privateKey", "password", "secret", "authorization"]
  }
}

output {
  elasticsearch {
    hosts => ["${ELASTICSEARCH_HOSTS}"]
    user => "${ELASTICSEARCH_USER}"
    password => "${ELASTICSEARCH_PASSWORD}"
    index => "prediction-market-%{[@metadata][index_prefix]}-%{+YYYY.MM.dd}"
  }
}
```

### Datadog Configuration

```yaml
# datadog-agent.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: datadog-agent-config
  namespace: monitoring
data:
  datadog.yaml: |
    api_key: ${DD_API_KEY}
    site: datadoghq.com

    logs_enabled: true
    logs_config:
      container_collect_all: true
      auto_multi_line_detection: true

    apm_config:
      enabled: true
      apm_non_local_traffic: true

    process_config:
      enabled: true

    network_config:
      enabled: true

    tags:
      - env:${ENV}
      - service:prediction-market
      - team:platform
```

```typescript
// Datadog logger integration
import tracer from 'dd-trace';

tracer.init({
  service: 'prediction-market-api',
  env: process.env.NODE_ENV,
  version: process.env.APP_VERSION,
  logInjection: true,
  profiling: true,
  runtimeMetrics: true,
});

// Logs will automatically include trace context
logger.info({
  userId: user.id,
  action: 'trade',
}, 'Processing trade request');
// Output includes dd.trace_id and dd.span_id
```

---

## Metrics Collection

### Prometheus Metrics

```typescript
// metrics.ts
import { Registry, Counter, Histogram, Gauge, Summary } from 'prom-client';

const register = new Registry();

// Default metrics
import { collectDefaultMetrics } from 'prom-client';
collectDefaultMetrics({ register });

// Custom metrics

// Trading metrics
export const tradesTotal = new Counter({
  name: 'prediction_market_trades_total',
  help: 'Total number of trades executed',
  labelNames: ['market_id', 'outcome', 'status'],
  registers: [register],
});

export const tradeVolume = new Counter({
  name: 'prediction_market_trade_volume_usdc',
  help: 'Total trade volume in USDC',
  labelNames: ['market_id', 'outcome'],
  registers: [register],
});

export const tradeDuration = new Histogram({
  name: 'prediction_market_trade_duration_seconds',
  help: 'Trade execution duration in seconds',
  labelNames: ['market_id', 'status'],
  buckets: [0.1, 0.25, 0.5, 1, 2.5, 5, 10],
  registers: [register],
});

// API metrics
export const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.01, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
  registers: [register],
});

export const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register],
});

// Blockchain metrics
export const blockchainTransactionsTotal = new Counter({
  name: 'blockchain_transactions_total',
  help: 'Total blockchain transactions',
  labelNames: ['type', 'status', 'network'],
  registers: [register],
});

export const blockchainGasUsed = new Histogram({
  name: 'blockchain_gas_used',
  help: 'Gas used per transaction',
  labelNames: ['type', 'network'],
  buckets: [50000, 100000, 200000, 500000, 1000000, 2000000],
  registers: [register],
});

export const blockchainLatency = new Histogram({
  name: 'blockchain_rpc_latency_seconds',
  help: 'RPC call latency in seconds',
  labelNames: ['method', 'provider'],
  buckets: [0.05, 0.1, 0.25, 0.5, 1, 2, 5],
  registers: [register],
});

export const currentBlockHeight = new Gauge({
  name: 'blockchain_current_block_height',
  help: 'Current processed block height',
  labelNames: ['network'],
  registers: [register],
});

export const indexerLag = new Gauge({
  name: 'indexer_block_lag',
  help: 'Number of blocks behind chain head',
  labelNames: ['network'],
  registers: [register],
});

// Business metrics
export const activeMarkets = new Gauge({
  name: 'prediction_market_active_markets',
  help: 'Number of currently active markets',
  registers: [register],
});

export const totalValueLocked = new Gauge({
  name: 'prediction_market_tvl_usdc',
  help: 'Total value locked in USDC',
  registers: [register],
});

export const activeUsers = new Gauge({
  name: 'prediction_market_active_users',
  help: 'Number of active users in last 24h',
  registers: [register],
});

// Cache metrics
export const cacheHits = new Counter({
  name: 'cache_hits_total',
  help: 'Total cache hits',
  labelNames: ['cache_name'],
  registers: [register],
});

export const cacheMisses = new Counter({
  name: 'cache_misses_total',
  help: 'Total cache misses',
  labelNames: ['cache_name'],
  registers: [register],
});

// Export metrics endpoint
export async function metricsHandler(req: Request, res: Response) {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
}
```

### Prometheus Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: prediction-market-prod
    region: us-east-1

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

rule_files:
  - /etc/prometheus/rules/*.yml

scrape_configs:
  # Kubernetes service discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)

  # API service
  - job_name: 'api'
    static_configs:
      - targets: ['api:9090']
    metrics_path: /metrics

  # Indexer service
  - job_name: 'indexer'
    static_configs:
      - targets: ['indexer:9090']

  # PostgreSQL
  - job_name: 'postgresql'
    static_configs:
      - targets: ['postgres-exporter:9187']

  # Redis
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']

  # Node exporter (system metrics)
  - job_name: 'node'
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
```

---

## Distributed Tracing

### OpenTelemetry Setup

```typescript
// tracing.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';
import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';

const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: process.env.SERVICE_NAME,
    [SemanticResourceAttributes.SERVICE_VERSION]: process.env.APP_VERSION,
    [SemanticResourceAttributes.DEPLOYMENT_ENVIRONMENT]: process.env.NODE_ENV,
  }),
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT,
  }),
  spanProcessor: new BatchSpanProcessor(traceExporter, {
    maxQueueSize: 2048,
    maxExportBatchSize: 512,
    scheduledDelayMillis: 5000,
  }),
  instrumentations: [
    getNodeAutoInstrumentations({
      '@opentelemetry/instrumentation-http': {
        ignoreIncomingPaths: ['/health', '/ready', '/metrics'],
      },
      '@opentelemetry/instrumentation-express': {},
      '@opentelemetry/instrumentation-pg': {},
      '@opentelemetry/instrumentation-redis': {},
    }),
  ],
});

sdk.start();

process.on('SIGTERM', () => {
  sdk.shutdown()
    .then(() => console.log('Tracing terminated'))
    .catch((error) => console.error('Error terminating tracing', error))
    .finally(() => process.exit(0));
});
```

### Custom Span Creation

```typescript
// Custom spans for blockchain operations
import { trace, SpanKind, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('prediction-market');

async function executeTradeWithTracing(trade: TradeRequest): Promise<TradeResult> {
  return tracer.startActiveSpan('trade.execute', {
    kind: SpanKind.INTERNAL,
    attributes: {
      'trade.market_id': trade.marketId,
      'trade.outcome': trade.outcome,
      'trade.amount': trade.amount,
      'trade.user_id': trade.userId,
    },
  }, async (span) => {
    try {
      // Validate trade
      const validationSpan = tracer.startSpan('trade.validate');
      await validateTrade(trade);
      validationSpan.end();

      // Submit to blockchain
      const txSpan = tracer.startSpan('blockchain.submit', {
        attributes: {
          'blockchain.network': 'polygon',
          'blockchain.contract': TRADING_CONTRACT_ADDRESS,
        },
      });

      const tx = await submitTransaction(trade);
      txSpan.setAttribute('blockchain.tx_hash', tx.hash);

      // Wait for confirmation
      const receipt = await tx.wait();
      txSpan.setAttribute('blockchain.block_number', receipt.blockNumber);
      txSpan.setAttribute('blockchain.gas_used', receipt.gasUsed.toString());
      txSpan.end();

      // Update database
      const dbSpan = tracer.startSpan('database.update');
      const result = await updateTradeRecord(trade, receipt);
      dbSpan.end();

      span.setStatus({ code: SpanStatusCode.OK });
      return result;
    } catch (error) {
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: error.message,
      });
      span.recordException(error);
      throw error;
    } finally {
      span.end();
    }
  });
}
```

### Trace Context Propagation

```typescript
// Propagate context across services
import { context, propagation } from '@opentelemetry/api';

// HTTP client with trace propagation
async function callService(url: string, data: any): Promise<any> {
  const headers: Record<string, string> = {};

  // Inject trace context into headers
  propagation.inject(context.active(), headers);

  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      ...headers,
    },
    body: JSON.stringify(data),
  });

  return response.json();
}

// Message queue with trace propagation
async function publishMessage(queue: string, message: any): Promise<void> {
  const carrier: Record<string, string> = {};
  propagation.inject(context.active(), carrier);

  await rabbitMQ.publish(queue, {
    ...message,
    _traceContext: carrier,
  });
}

// Consumer extracts context
async function consumeMessage(message: any): Promise<void> {
  const parentContext = propagation.extract(context.active(), message._traceContext);

  await context.with(parentContext, async () => {
    // Processing happens within parent trace context
    await processMessage(message);
  });
}
```

---

## Alerting Rules and Thresholds

### Prometheus Alert Rules

```yaml
# alerts/prediction-market.yml
groups:
  - name: prediction-market-critical
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status_code=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 2m
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} over 5%"
          runbook: "https://runbooks.internal/high-error-rate"

      # API latency
      - alert: HighAPILatency
        expr: |
          histogram_quantile(0.99,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, route)
          ) > 2
        for: 5m
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "High API latency on {{ $labels.route }}"
          description: "P99 latency is {{ $value }}s"

      # Blockchain transaction failures
      - alert: BlockchainTransactionFailures
        expr: |
          sum(rate(blockchain_transactions_total{status="failed"}[5m]))
          / sum(rate(blockchain_transactions_total[5m])) > 0.1
        for: 5m
        labels:
          severity: critical
          team: blockchain
        annotations:
          summary: "High blockchain transaction failure rate"
          description: "{{ $value | humanizePercentage }} of transactions failing"

      # Indexer lag
      - alert: IndexerLagCritical
        expr: indexer_block_lag > 100
        for: 5m
        labels:
          severity: critical
          team: blockchain
        annotations:
          summary: "Indexer is {{ $value }} blocks behind"
          description: "Critical indexer lag detected"

  - name: prediction-market-warning
    rules:
      # Memory usage
      - alert: HighMemoryUsage
        expr: |
          container_memory_usage_bytes
          / container_spec_memory_limit_bytes > 0.85
        for: 10m
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "High memory usage in {{ $labels.container }}"
          description: "Memory usage is {{ $value | humanizePercentage }}"

      # Database connections
      - alert: DatabaseConnectionPoolExhaustion
        expr: |
          pg_stat_activity_count / pg_settings_max_connections > 0.8
        for: 5m
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "Database connection pool near exhaustion"
          description: "{{ $value | humanizePercentage }} of connections used"

      # Cache hit ratio
      - alert: LowCacheHitRatio
        expr: |
          sum(rate(cache_hits_total[5m]))
          / (sum(rate(cache_hits_total[5m])) + sum(rate(cache_misses_total[5m]))) < 0.8
        for: 15m
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "Low cache hit ratio"
          description: "Cache hit ratio is {{ $value | humanizePercentage }}"

      # Trade volume anomaly
      - alert: TradeVolumeAnomaly
        expr: |
          abs(sum(rate(prediction_market_trade_volume_usdc[1h]))
          - sum(rate(prediction_market_trade_volume_usdc[1h] offset 1d)))
          / sum(rate(prediction_market_trade_volume_usdc[1h] offset 1d)) > 0.5
        for: 30m
        labels:
          severity: warning
          team: business
        annotations:
          summary: "Unusual trade volume detected"
          description: "Trade volume changed by {{ $value | humanizePercentage }}"

  - name: prediction-market-slo
    rules:
      # Availability SLO
      - alert: SLOAvailabilityBreach
        expr: |
          1 - (
            sum(increase(http_requests_total{status_code!~"5.."}[30d]))
            / sum(increase(http_requests_total[30d]))
          ) > 0.001  # 99.9% SLO
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "SLO breach: Availability below 99.9%"
          description: "30-day availability is {{ $value | humanizePercentage }}"

      # Latency SLO
      - alert: SLOLatencyBreach
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket[30d])) by (le)
          ) > 0.5
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "SLO breach: P95 latency above 500ms"
```

### PagerDuty Integration

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'

route:
  receiver: 'default'
  group_by: ['alertname', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
      continue: true
    - match:
        severity: warning
      receiver: 'slack-warnings'
    - match:
        team: blockchain
      receiver: 'blockchain-team'

receivers:
  - name: 'default'
    slack_configs:
      - api_url: '${SLACK_WEBHOOK_URL}'
        channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: '${PAGERDUTY_SERVICE_KEY}'
        severity: critical
        description: '{{ .CommonAnnotations.summary }}'
        details:
          alertname: '{{ .GroupLabels.alertname }}'
          description: '{{ .CommonAnnotations.description }}'
          runbook: '{{ .CommonAnnotations.runbook }}'

  - name: 'slack-warnings'
    slack_configs:
      - api_url: '${SLACK_WEBHOOK_URL}'
        channel: '#alerts-warning'
        color: 'warning'

  - name: 'blockchain-team'
    slack_configs:
      - api_url: '${SLACK_WEBHOOK_URL}'
        channel: '#blockchain-alerts'
    pagerduty_configs:
      - service_key: '${PAGERDUTY_BLOCKCHAIN_KEY}'
```

---

## Dashboard Designs

### Main Operations Dashboard

```json
{
  "dashboard": {
    "title": "Prediction Market - Operations Overview",
    "tags": ["production", "operations"],
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "gridPos": {"x": 0, "y": 0, "w": 8, "h": 6},
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (status_code)",
            "legendFormat": "{{status_code}}"
          }
        ]
      },
      {
        "title": "P99 Latency by Endpoint",
        "type": "graph",
        "gridPos": {"x": 8, "y": 0, "w": 8, "h": 6},
        "targets": [
          {
            "expr": "histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, route))",
            "legendFormat": "{{route}}"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "singlestat",
        "gridPos": {"x": 16, "y": 0, "w": 4, "h": 3},
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100"
          }
        ],
        "thresholds": "1,5",
        "colors": ["green", "yellow", "red"]
      },
      {
        "title": "Active Users (24h)",
        "type": "singlestat",
        "gridPos": {"x": 20, "y": 0, "w": 4, "h": 3},
        "targets": [
          {
            "expr": "prediction_market_active_users"
          }
        ]
      },
      {
        "title": "Trade Volume (24h)",
        "type": "graph",
        "gridPos": {"x": 0, "y": 6, "w": 12, "h": 6},
        "targets": [
          {
            "expr": "sum(increase(prediction_market_trade_volume_usdc[24h]))",
            "legendFormat": "Volume (USDC)"
          }
        ]
      },
      {
        "title": "Blockchain Metrics",
        "type": "graph",
        "gridPos": {"x": 12, "y": 6, "w": 12, "h": 6},
        "targets": [
          {
            "expr": "indexer_block_lag",
            "legendFormat": "Indexer Lag"
          },
          {
            "expr": "rate(blockchain_transactions_total[5m])",
            "legendFormat": "TX Rate"
          }
        ]
      }
    ]
  }
}
```

### Business Metrics Dashboard

```yaml
# Grafana dashboard as code
apiVersion: 1
dashboards:
  - name: Business Metrics
    folder: Prediction Market
    type: file
    options:
      path: /var/lib/grafana/dashboards/business.json
---
# business.json structure
panels:
  - title: Total Value Locked (TVL)
    type: stat
    targets:
      - expr: prediction_market_tvl_usdc
    fieldConfig:
      defaults:
        unit: currencyUSD

  - title: Active Markets
    type: stat
    targets:
      - expr: prediction_market_active_markets

  - title: Trade Volume by Market
    type: piechart
    targets:
      - expr: sum(prediction_market_trade_volume_usdc) by (market_id)

  - title: User Acquisition
    type: timeseries
    targets:
      - expr: increase(prediction_market_users_registered_total[1d])
        legendFormat: New Users

  - title: Top Markets by Volume
    type: table
    targets:
      - expr: topk(10, sum(prediction_market_trade_volume_usdc) by (market_id, market_name))
```

---

## On-Chain Monitoring

### Tenderly Integration

```typescript
// tenderly-monitoring.ts
import { Tenderly } from '@tenderly/sdk';

const tenderly = new Tenderly({
  accessKey: process.env.TENDERLY_ACCESS_KEY,
  accountName: process.env.TENDERLY_ACCOUNT,
  projectName: process.env.TENDERLY_PROJECT,
});

// Create monitoring alerts
async function setupTenderlyAlerts() {
  // Alert on failed transactions
  await tenderly.alerts.create({
    name: 'Failed Transactions',
    network: 'polygon',
    alertType: 'transaction',
    conditions: {
      status: 'failed',
      contractAddresses: [
        MARKET_FACTORY_ADDRESS,
        TRADING_ENGINE_ADDRESS,
      ],
    },
    notifications: [
      {
        type: 'slack',
        webhookUrl: process.env.SLACK_WEBHOOK_BLOCKCHAIN,
      },
      {
        type: 'webhook',
        webhookUrl: `${API_URL}/webhooks/tenderly`,
      },
    ],
  });

  // Alert on large trades
  await tenderly.alerts.create({
    name: 'Large Trade Alert',
    network: 'polygon',
    alertType: 'event',
    conditions: {
      contractAddress: TRADING_ENGINE_ADDRESS,
      eventSignature: 'TradeExecuted(address,bytes32,uint8,uint256,uint256)',
      filters: [
        {
          parameter: 'amount',
          condition: 'gt',
          value: '100000000000', // 100,000 USDC (6 decimals)
        },
      ],
    },
    notifications: [
      {
        type: 'slack',
        webhookUrl: process.env.SLACK_WEBHOOK_TRADES,
      },
    ],
  });

  // Alert on contract state changes
  await tenderly.alerts.create({
    name: 'Contract Paused',
    network: 'polygon',
    alertType: 'state_change',
    conditions: {
      contractAddress: TRADING_ENGINE_ADDRESS,
      stateVariable: 'paused',
      newValue: true,
    },
    notifications: [
      {
        type: 'pagerduty',
        serviceKey: process.env.PAGERDUTY_BLOCKCHAIN_KEY,
      },
    ],
  });
}
```

### Custom On-Chain Metrics

```typescript
// blockchain-metrics.ts
import { ethers } from 'ethers';
import { Gauge, Counter } from 'prom-client';

const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);

// Metrics
const contractBalance = new Gauge({
  name: 'contract_balance_usdc',
  help: 'USDC balance of contract',
  labelNames: ['contract'],
});

const gasPrice = new Gauge({
  name: 'blockchain_gas_price_gwei',
  help: 'Current gas price in gwei',
});

const blockTime = new Gauge({
  name: 'blockchain_block_time_seconds',
  help: 'Time since last block',
});

const eventCount = new Counter({
  name: 'blockchain_events_total',
  help: 'Total events emitted',
  labelNames: ['event_name', 'contract'],
});

// Collectors
async function collectBlockchainMetrics() {
  // Gas price
  const feeData = await provider.getFeeData();
  gasPrice.set(Number(feeData.gasPrice) / 1e9);

  // Block time
  const block = await provider.getBlock('latest');
  const timeSinceBlock = Date.now() / 1000 - block.timestamp;
  blockTime.set(timeSinceBlock);

  // Contract balances
  const usdcContract = new ethers.Contract(USDC_ADDRESS, ERC20_ABI, provider);

  for (const [name, address] of Object.entries(MONITORED_CONTRACTS)) {
    const balance = await usdcContract.balanceOf(address);
    contractBalance.set({ contract: name }, Number(balance) / 1e6);
  }
}

// Event monitoring
async function monitorEvents() {
  const tradingEngine = new ethers.Contract(
    TRADING_ENGINE_ADDRESS,
    TRADING_ENGINE_ABI,
    provider
  );

  tradingEngine.on('TradeExecuted', (user, marketId, outcome, shares, cost, event) => {
    eventCount.inc({ event_name: 'TradeExecuted', contract: 'TradingEngine' });

    // Log for audit trail
    logger.info({
      category: LogCategory.TX_CONFIRMED,
      event: 'TradeExecuted',
      txHash: event.transactionHash,
      blockNumber: event.blockNumber,
      user,
      marketId,
      outcome,
      shares: shares.toString(),
      cost: cost.toString(),
    }, 'Trade executed on-chain');
  });

  tradingEngine.on('MarketResolved', (marketId, outcome, event) => {
    eventCount.inc({ event_name: 'MarketResolved', contract: 'TradingEngine' });

    logger.info({
      category: LogCategory.MARKET_RESOLVED,
      event: 'MarketResolved',
      txHash: event.transactionHash,
      marketId,
      outcome,
    }, 'Market resolved');
  });
}

// Run collectors
setInterval(collectBlockchainMetrics, 15000); // Every 15 seconds
monitorEvents();
```

### Forta Integration (Security Monitoring)

```typescript
// forta-bot.ts
import {
  Finding,
  FindingSeverity,
  FindingType,
  HandleTransaction,
  TransactionEvent,
} from 'forta-agent';

const MONITORED_ADDRESSES = [
  MARKET_FACTORY_ADDRESS,
  TRADING_ENGINE_ADDRESS,
  ORACLE_ADDRESS,
];

const handleTransaction: HandleTransaction = async (txEvent: TransactionEvent) => {
  const findings: Finding[] = [];

  // Detect large value transfers
  if (txEvent.transaction.value > ethers.parseEther('100')) {
    findings.push(
      Finding.fromObject({
        name: 'Large Value Transfer',
        description: `Large ETH transfer of ${ethers.formatEther(txEvent.transaction.value)} ETH`,
        alertId: 'PM-1',
        severity: FindingSeverity.Medium,
        type: FindingType.Suspicious,
        metadata: {
          from: txEvent.transaction.from,
          to: txEvent.transaction.to,
          value: txEvent.transaction.value.toString(),
        },
      })
    );
  }

  // Detect ownership changes
  const ownershipTransferredEvents = txEvent.filterLog(
    'OwnershipTransferred(address,address)',
    MONITORED_ADDRESSES
  );

  for (const event of ownershipTransferredEvents) {
    findings.push(
      Finding.fromObject({
        name: 'Ownership Transferred',
        description: `Contract ownership transferred`,
        alertId: 'PM-2',
        severity: FindingSeverity.Critical,
        type: FindingType.Suspicious,
        metadata: {
          contract: event.address,
          previousOwner: event.args.previousOwner,
          newOwner: event.args.newOwner,
        },
      })
    );
  }

  // Detect unusual gas usage
  if (txEvent.gasUsed > 5000000) {
    findings.push(
      Finding.fromObject({
        name: 'High Gas Usage',
        description: `Transaction used ${txEvent.gasUsed} gas`,
        alertId: 'PM-3',
        severity: FindingSeverity.Info,
        type: FindingType.Info,
        metadata: {
          gasUsed: txEvent.gasUsed.toString(),
          txHash: txEvent.hash,
        },
      })
    );
  }

  return findings;
};

export default {
  handleTransaction,
};
```

---

## SLIs, SLOs, and SLAs

### Service Level Definitions

#### API Service

| SLI | Measurement | SLO | SLA |
|-----|-------------|-----|-----|
| Availability | Successful requests / Total requests | 99.95% | 99.9% |
| Latency (P50) | 50th percentile response time | < 100ms | < 200ms |
| Latency (P99) | 99th percentile response time | < 500ms | < 1000ms |
| Error Rate | 5xx responses / Total responses | < 0.1% | < 0.5% |
| Throughput | Requests per second | > 1000 RPS | > 500 RPS |

#### Trading Service

| SLI | Measurement | SLO | SLA |
|-----|-------------|-----|-----|
| Trade Success Rate | Successful trades / Attempted trades | 99.9% | 99.5% |
| Trade Latency | Time from submission to confirmation | < 5s | < 30s |
| Price Accuracy | Price deviation from oracle | < 0.1% | < 0.5% |

#### Blockchain Indexer

| SLI | Measurement | SLO | SLA |
|-----|-------------|-----|-----|
| Block Lag | Blocks behind chain head | < 10 blocks | < 50 blocks |
| Event Processing | Events processed / Events emitted | 100% | 100% |
| Reorg Handling | Successful reorg recovery | 100% | 100% |

### Error Budget Calculation

```typescript
// error-budget.ts

interface ErrorBudget {
  service: string;
  slo: number; // e.g., 0.999 for 99.9%
  windowDays: number;
  currentAvailability: number;
  budgetRemaining: number;
  budgetRemainingMinutes: number;
}

function calculateErrorBudget(
  slo: number,
  windowDays: number,
  currentAvailability: number
): ErrorBudget {
  const totalMinutes = windowDays * 24 * 60;
  const allowedDowntimeMinutes = totalMinutes * (1 - slo);
  const actualDowntimeMinutes = totalMinutes * (1 - currentAvailability);
  const budgetRemainingMinutes = allowedDowntimeMinutes - actualDowntimeMinutes;
  const budgetRemaining = budgetRemainingMinutes / allowedDowntimeMinutes;

  return {
    service: 'api',
    slo,
    windowDays,
    currentAvailability,
    budgetRemaining: Math.max(0, budgetRemaining),
    budgetRemainingMinutes: Math.max(0, budgetRemainingMinutes),
  };
}

// Example: 99.9% SLO over 30 days
// Total: 43,200 minutes
// Allowed downtime: 43.2 minutes
// If current availability is 99.95%: 21.6 minutes used, 21.6 remaining
```

### SLO Monitoring Dashboard

```yaml
# slo-dashboard.yaml
panels:
  - title: Error Budget Remaining
    type: gauge
    thresholds:
      - value: 0
        color: red
      - value: 25
        color: orange
      - value: 50
        color: yellow
      - value: 75
        color: green
    query: |
      (
        1 - (
          sum(increase(http_requests_total{status_code=~"5.."}[30d]))
          / sum(increase(http_requests_total[30d]))
        ) - 0.999
      ) / 0.001 * 100

  - title: Availability (30d Rolling)
    type: stat
    query: |
      sum(increase(http_requests_total{status_code!~"5.."}[30d]))
      / sum(increase(http_requests_total[30d])) * 100
    thresholds:
      - value: 99.0
        color: red
      - value: 99.5
        color: yellow
      - value: 99.9
        color: green

  - title: P99 Latency Trend
    type: timeseries
    query: |
      histogram_quantile(0.99,
        sum(rate(http_request_duration_seconds_bucket[1h])) by (le)
      )
    thresholds:
      - value: 0.5
        color: green
      - value: 1.0
        color: yellow
      - value: 2.0
        color: red
```

---

## Incident Severity Levels

### Severity Definitions

| Severity | Definition | Response Time | Examples |
|----------|------------|---------------|----------|
| **SEV1** | Complete service outage or data loss | 15 minutes | Trading halted, funds at risk |
| **SEV2** | Major functionality impaired | 30 minutes | Cannot execute trades, severe lag |
| **SEV3** | Minor functionality impaired | 2 hours | Slow response, partial feature outage |
| **SEV4** | Minimal impact | 24 hours | UI glitches, non-critical bugs |

### Incident Response Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    Incident Response Flow                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Detection ──► Triage ──► Investigation ──► Resolution ──► PIR  │
│      │           │              │               │            │   │
│      ▼           ▼              ▼               ▼            ▼   │
│   Alerts     Severity       Root Cause       Fix/Rollback  Review│
│   Manual     Assignment     Analysis         Communication      │
│                             Mitigation                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Escalation Matrix

| Severity | Initial Responder | Escalation (30 min) | Escalation (1 hr) |
|----------|------------------|---------------------|-------------------|
| SEV1 | On-call Engineer | Engineering Manager | VP Engineering + CEO |
| SEV2 | On-call Engineer | Team Lead | Engineering Manager |
| SEV3 | On-call Engineer | Team Lead | - |
| SEV4 | Assigned Engineer | - | - |

### Incident Communication Template

```markdown
# Incident: [Brief Description]

## Status: [Investigating | Identified | Monitoring | Resolved]

## Severity: SEV[1-4]

## Timeline (UTC)
- **HH:MM** - Issue detected
- **HH:MM** - Investigation started
- **HH:MM** - Root cause identified
- **HH:MM** - Fix deployed
- **HH:MM** - Monitoring for stability

## Impact
- Users affected: [Number/Percentage]
- Duration: [Time]
- Services affected: [List]

## Root Cause
[Brief description of what caused the incident]

## Resolution
[Brief description of how it was fixed]

## Action Items
- [ ] [Action 1] - Owner: [Name] - Due: [Date]
- [ ] [Action 2] - Owner: [Name] - Due: [Date]

## Incident Commander: [Name]
## Communications Lead: [Name]
```

---

## Runbooks

### Runbook: High API Latency

```markdown
# Runbook: High API Latency

## Alert
- Name: HighAPILatency
- Severity: Critical
- Threshold: P99 > 2 seconds for 5 minutes

## Symptoms
- API responses taking longer than 2 seconds
- Users reporting slow load times
- Timeout errors increasing

## Diagnosis Steps

### 1. Check current latency metrics
```bash
# Prometheus query
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, route))
```

### 2. Identify slow endpoints
```bash
# Find slowest routes
kubectl exec -it $(kubectl get pod -l app=api -o name | head -1) -- \
  curl -s localhost:9090/metrics | grep http_request_duration
```

### 3. Check database performance
```bash
# PostgreSQL slow queries
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 seconds';
```

### 4. Check Redis performance
```bash
redis-cli --latency-history -i 1
redis-cli INFO stats | grep instantaneous_ops_per_sec
```

### 5. Check pod resources
```bash
kubectl top pods -l app=api
kubectl describe pod -l app=api | grep -A 5 "Limits\|Requests"
```

## Resolution Steps

### If database is slow:
1. Kill long-running queries if safe
2. Check for missing indexes
3. Scale read replicas
4. Enable query caching

### If Redis is slow:
1. Check memory usage
2. Flush if necessary (coordinate with team)
3. Scale Redis cluster

### If CPU/Memory constrained:
1. Scale horizontally: `kubectl scale deployment api --replicas=10`
2. Check for memory leaks
3. Review recent deployments for regressions

### If specific endpoint is slow:
1. Add caching if appropriate
2. Optimize database queries
3. Consider async processing

## Escalation
- If unresolved after 15 minutes: Escalate to Team Lead
- If data integrity at risk: Escalate to Engineering Manager

## Post-Incident
- [ ] Create JIRA ticket for follow-up
- [ ] Schedule PIR if SEV1/SEV2
- [ ] Update this runbook if needed
```

### Runbook: Indexer Block Lag

```markdown
# Runbook: Indexer Block Lag

## Alert
- Name: IndexerLagCritical
- Severity: Critical
- Threshold: More than 100 blocks behind

## Impact
- Market data may be stale
- Trade confirmations delayed
- Position updates delayed

## Diagnosis Steps

### 1. Check indexer status
```bash
# Get current lag
kubectl exec -it $(kubectl get pod -l app=indexer -o name | head -1) -- \
  curl -s localhost:9090/metrics | grep indexer_block_lag
```

### 2. Check indexer logs
```bash
kubectl logs -l app=indexer --since=10m | grep -i "error\|warn"
```

### 3. Check RPC provider status
```bash
# Test RPC endpoint
curl -X POST $RPC_URL \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

### 4. Check database write performance
```bash
# Check for locks
SELECT * FROM pg_locks WHERE granted = false;

# Check table bloat
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename))
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## Resolution Steps

### If RPC is slow/down:
1. Switch to backup RPC provider
   ```bash
   kubectl set env deployment/indexer RPC_URL=$BACKUP_RPC_URL
   ```
2. Check provider status page
3. Contact provider if needed

### If database is bottleneck:
1. Increase batch size temporarily
2. Run VACUUM ANALYZE on hot tables
3. Scale database if needed

### If indexer is crashed:
1. Check memory/CPU limits
2. Restart with increased resources
3. Review logs for root cause

### Manual catch-up (if needed):
```bash
# Restart indexer from specific block
kubectl set env deployment/indexer START_BLOCK=<last_known_good_block>
kubectl rollout restart deployment/indexer
```

## Verification
1. Confirm lag is decreasing
2. Check API returns fresh data
3. Verify events are being processed

## Escalation
- If RPC provider issue: Contact provider support
- If database issue: Escalate to DBA
- If unresolved after 30 minutes: Engineering Manager
```

### Runbook: Smart Contract Emergency

```markdown
# Runbook: Smart Contract Emergency

## Alert Triggers
- Unusual transaction patterns detected
- Large unauthorized transfers
- Contract state manipulation
- Ownership change attempts

## IMMEDIATE ACTIONS (First 5 minutes)

### 1. Assess the situation
- Check Tenderly alerts for details
- Review transaction in block explorer
- Determine if funds are at immediate risk

### 2. Pause contracts if necessary
```solidity
// Emergency pause (requires multi-sig or admin key)
// Only execute if funds are actively being drained

// Via Safe wallet or admin:
cast send $TRADING_ENGINE_ADDRESS "pause()" --private-key $ADMIN_KEY
```

### 3. Alert the team
- Page blockchain team immediately
- Notify security team
- Brief engineering leadership

## INVESTIGATION

### Check recent transactions
```bash
# Get recent transactions to contract
cast logs --address $CONTRACT_ADDRESS --from-block -100
```

### Analyze suspicious transaction
```bash
# Decode transaction
cast tx $TX_HASH
cast call --trace $CONTRACT_ADDRESS $CALLDATA
```

### Check contract state
```bash
# Check critical state variables
cast call $CONTRACT_ADDRESS "paused()(bool)"
cast call $CONTRACT_ADDRESS "owner()(address)"
cast call $CONTRACT_ADDRESS "totalSupply()(uint256)"
```

## RESPONSE ACTIONS

### If exploit confirmed:
1. Keep contracts paused
2. Document all affected transactions
3. Assess total funds at risk
4. Prepare user communication
5. Engage security auditors
6. Consider legal consultation

### If false positive:
1. Document investigation findings
2. Unpause contracts if paused
3. Update alert thresholds
4. Create post-incident report

## COMMUNICATION TEMPLATE

```
🚨 Security Incident Notice

We have detected unusual activity and have temporarily paused trading
to protect user funds. Our security team is investigating.

- All funds are safe
- Trading will resume once security is confirmed
- We will provide updates every [X] minutes

Status page: https://status.predictionmarket.com
```

## POST-INCIDENT
- [ ] Complete incident report within 24 hours
- [ ] Security audit of affected code
- [ ] Update monitoring and alerts
- [ ] User communication and compensation plan if needed
```

---

## References

- [Tenderly Monitoring Documentation](https://tenderly.co/monitoring)
- [Datadog Observability](https://www.datadoghq.com/)
- [Prometheus Best Practices](https://prometheus.io/docs/practices/naming/)
- [Grafana Dashboard Best Practices](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/best-practices/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [SLO Documentation (Google SRE)](https://sre.google/sre-book/service-level-objectives/)
- [CoinCashew ETH Validator Monitoring](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/monitoring-your-validator-with-grafana-and-prometheus)
