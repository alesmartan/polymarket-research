# DevOps and CI/CD Pipeline

## Web3 DevOps Best Practices for Prediction Market Platform

This document outlines the comprehensive DevOps strategy and CI/CD pipeline architecture for a blockchain-based prediction market platform.

---

## Table of Contents

1. [Git Workflow](#git-workflow)
2. [Branch Naming Conventions](#branch-naming-conventions)
3. [PR Review Process](#pr-review-process)
4. [CI Pipeline Stages](#ci-pipeline-stages)
5. [CD Pipeline](#cd-pipeline)
6. [Infrastructure as Code](#infrastructure-as-code)
7. [Container Orchestration](#container-orchestration)
8. [Smart Contract Deployment Pipeline](#smart-contract-deployment-pipeline)
9. [Environment Management](#environment-management)
10. [Secrets Management](#secrets-management)
11. [GitHub Actions Examples](#github-actions-examples)

---

## Git Workflow

### Trunk-Based Development (Recommended)

For a fast-moving blockchain project, we recommend **trunk-based development** with short-lived feature branches.

```
main (trunk)
  │
  ├── feature/add-market-creation (1-3 days max)
  │     └── Merged via PR with required reviews
  │
  ├── feature/oracle-integration (1-3 days max)
  │     └── Merged via PR with required reviews
  │
  └── release/v1.0.0 (created from main for production)
        └── Hotfixes cherry-picked back to main
```

#### Why Trunk-Based for Web3?

1. **Fast iteration**: Smart contract development requires rapid feedback loops
2. **Continuous integration**: Catch integration issues early
3. **Reduced merge conflicts**: Short-lived branches minimize drift
4. **Simpler pipeline**: One main branch to deploy from

### GitFlow (Alternative for Regulated Environments)

```
main (production)
  │
  └── develop
        │
        ├── feature/market-creation
        │     └── PR to develop
        │
        ├── release/v1.0.0
        │     └── RC testing, then merge to main
        │
        └── hotfix/critical-bug
              └── Merge to both main and develop
```

### Branch Protection Rules

```yaml
# .github/branch-protection.yml (documentation)
main:
  required_reviews: 2
  require_code_owners: true
  require_signed_commits: true
  require_status_checks:
    - lint
    - test
    - security-scan
    - smart-contract-test
  restrict_push:
    - release-managers
  allow_force_push: false
  allow_deletions: false

develop:
  required_reviews: 1
  require_status_checks:
    - lint
    - test
  allow_force_push: false
```

---

## Branch Naming Conventions

### Standard Prefixes

| Prefix | Purpose | Example |
|--------|---------|---------|
| `feature/` | New features | `feature/multi-outcome-markets` |
| `fix/` | Bug fixes | `fix/trading-calculation-error` |
| `hotfix/` | Production emergencies | `hotfix/critical-oracle-bug` |
| `refactor/` | Code improvements | `refactor/optimize-gas-usage` |
| `docs/` | Documentation | `docs/api-reference` |
| `test/` | Test additions | `test/e2e-trading-flow` |
| `chore/` | Maintenance | `chore/upgrade-dependencies` |
| `contract/` | Smart contract changes | `contract/upgrade-amm-v2` |

### Naming Rules

```
<type>/<ticket-id>-<short-description>

Examples:
- feature/PM-123-add-market-categories
- fix/PM-456-incorrect-share-calculation
- contract/PM-789-gas-optimization
- hotfix/critical-withdrawal-bug
```

### Commit Message Convention

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]

Examples:
- feat(contracts): add multi-outcome market support
- fix(api): correct share price calculation
- docs(readme): update deployment instructions
- chore(deps): upgrade ethers to v6
- BREAKING CHANGE: oracle interface updated

Types: feat, fix, docs, style, refactor, perf, test, chore, ci
Scopes: contracts, api, frontend, infra, docs
```

---

## PR Review Process

### PR Template

```markdown
<!-- .github/pull_request_template.md -->

## Description
<!-- Describe your changes -->

## Type of Change
- [ ] Feature (new functionality)
- [ ] Bug fix (non-breaking fix)
- [ ] Breaking change (fix or feature causing existing functionality to change)
- [ ] Smart contract change (requires audit review)
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed
- [ ] Gas usage verified (for contract changes)

## Smart Contract Checklist (if applicable)
- [ ] No new external calls without reentrancy protection
- [ ] Access control verified
- [ ] Event emissions added
- [ ] Upgrade compatibility checked
- [ ] Gas optimization considered

## Security Considerations
<!-- Describe any security implications -->

## Screenshots (if applicable)
<!-- Add screenshots for UI changes -->

## Related Issues
<!-- Link related issues: Fixes #123 -->
```

### Review Requirements

```yaml
# CODEOWNERS
# Smart contracts require security team review
/contracts/ @security-team @smart-contract-leads

# Infrastructure requires DevOps review
/infrastructure/ @devops-team
/.github/ @devops-team

# API changes require backend lead review
/api/ @backend-leads

# Frontend changes
/frontend/ @frontend-leads
```

### Review Checklist

#### For All PRs
- [ ] Code follows style guidelines
- [ ] Tests pass locally and in CI
- [ ] Documentation updated
- [ ] No sensitive data committed
- [ ] Commit messages follow convention

#### For Smart Contract PRs
- [ ] Static analysis passed (Slither, Mythril)
- [ ] Gas report reviewed
- [ ] Upgrade safety verified
- [ ] Events emitted for state changes
- [ ] Access control implemented

#### For Security-Critical PRs
- [ ] Threat model reviewed
- [ ] Input validation complete
- [ ] Error handling appropriate
- [ ] Audit trail maintained

---

## CI Pipeline Stages

### Pipeline Overview

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│  Lint   │ → │  Test   │ → │  Build  │ → │ Security│ → │ Deploy  │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
     │             │             │             │             │
     ▼             ▼             ▼             ▼             ▼
  ESLint       Unit Tests    Docker       Slither      Staging
  Prettier     Contract     Build       Mythril      Production
  Solhint      Integration   Artifacts   Audit        Preview
```

### Stage 1: Lint

```yaml
# lint.yml
name: Lint
on: [push, pull_request]

jobs:
  lint-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm format:check
      - run: pnpm typecheck

  lint-contracts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: foundry-rs/foundry-toolchain@v1
        with:
          version: nightly
      - run: forge fmt --check
      - name: Run Solhint
        run: npx solhint 'contracts/**/*.sol'

  lint-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: cd api && pnpm install --frozen-lockfile
      - run: cd api && pnpm lint
      - run: cd api && pnpm typecheck
```

### Stage 2: Test

```yaml
# test.yml
name: Test
on: [push, pull_request]

jobs:
  test-contracts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive
      - uses: foundry-rs/foundry-toolchain@v1
        with:
          version: nightly

      - name: Run Forge Tests
        run: |
          forge test -vvv --gas-report
        env:
          FOUNDRY_PROFILE: ci

      - name: Run Coverage
        run: forge coverage --report lcov

      - name: Upload Coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./lcov.info
          flags: contracts

  test-api:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: timescale/timescaledb:latest-pg15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: prediction_market_test
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
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - run: cd api && pnpm install --frozen-lockfile
      - run: cd api && pnpm db:migrate:test
      - run: cd api && pnpm test:coverage
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/prediction_market_test
          REDIS_URL: redis://localhost:6379

      - name: Upload Coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./api/coverage/lcov.info
          flags: api

  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - run: cd frontend && pnpm install --frozen-lockfile
      - run: cd frontend && pnpm test:coverage

      - name: Upload Coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./frontend/coverage/lcov.info
          flags: frontend

  test-e2e:
    runs-on: ubuntu-latest
    needs: [test-contracts, test-api, test-frontend]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install Playwright
        run: cd frontend && pnpm exec playwright install --with-deps

      - name: Start Services
        run: docker-compose -f docker-compose.test.yml up -d

      - name: Run E2E Tests
        run: cd frontend && pnpm test:e2e

      - name: Upload Artifacts
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: frontend/playwright-report/
```

### Stage 3: Build

```yaml
# build.yml
name: Build
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - run: cd frontend && pnpm install --frozen-lockfile
      - run: cd frontend && pnpm build

      - name: Upload Build
        uses: actions/upload-artifact@v4
        with:
          name: frontend-build
          path: frontend/.next/

  build-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - run: cd api && pnpm install --frozen-lockfile
      - run: cd api && pnpm build

      - name: Upload Build
        uses: actions/upload-artifact@v4
        with:
          name: api-build
          path: api/dist/

  build-docker:
    runs-on: ubuntu-latest
    needs: [build-frontend, build-api]
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and Push API
        uses: docker/build-push-action@v5
        with:
          context: ./api
          push: ${{ github.event_name != 'pull_request' }}
          tags: |
            ghcr.io/${{ github.repository }}/api:${{ github.sha }}
            ghcr.io/${{ github.repository }}/api:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Build and Push Frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: ${{ github.event_name != 'pull_request' }}
          tags: |
            ghcr.io/${{ github.repository }}/frontend:${{ github.sha }}
            ghcr.io/${{ github.repository }}/frontend:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Stage 4: Security

```yaml
# security.yml
name: Security
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'  # Daily

jobs:
  slither:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: crytic/slither-action@v0.4.0
        with:
          target: 'contracts/'
          slither-args: '--checklist --markdown-root ${{ github.server_url }}/${{ github.repository }}/blob/${{ github.sha }}/'
          fail-on: 'medium'
          sarif: 'slither.sarif'

      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: slither.sarif

  mythril:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Mythril
        uses: docker://mythril/myth:latest
        with:
          args: analyze contracts/MarketFactory.sol --solc-json mythril.config.json

  dependency-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: pnpm audit --audit-level=high
      - run: cd api && pnpm audit --audit-level=high
      - run: cd frontend && pnpm audit --audit-level=high

  codeql:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript, typescript
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
```

---

## CD Pipeline

### Deployment Environments

```
┌─────────────────────────────────────────────────────────────────┐
│                        Deployment Flow                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   PR Merge    ──►   Development   ──►   Staging   ──►  Prod    │
│   to main          (auto-deploy)     (auto-deploy)   (manual)  │
│                                                                 │
│   Testnet          Sepolia/         Polygon          Polygon    │
│   Contracts        Mumbai           Mainnet Fork     Mainnet    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Continuous Deployment Workflow

```yaml
# deploy.yml
name: Deploy
on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy to'
        required: true
        type: choice
        options:
          - development
          - staging
          - production

jobs:
  deploy-development:
    if: github.event_name == 'push' || github.event.inputs.environment == 'development'
    runs-on: ubuntu-latest
    environment: development
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to EKS
        run: |
          aws eks update-kubeconfig --name prediction-market-dev
          helm upgrade --install prediction-market ./helm/prediction-market \
            --namespace development \
            --set image.tag=${{ github.sha }} \
            --set environment=development \
            -f ./helm/values-development.yaml

      - name: Verify Deployment
        run: |
          kubectl rollout status deployment/api -n development
          kubectl rollout status deployment/frontend -n development

      - name: Run Smoke Tests
        run: |
          ./scripts/smoke-test.sh https://dev.predictionmarket.com

  deploy-staging:
    needs: deploy-development
    if: github.event_name == 'push' || github.event.inputs.environment == 'staging'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to EKS
        run: |
          aws eks update-kubeconfig --name prediction-market-staging
          helm upgrade --install prediction-market ./helm/prediction-market \
            --namespace staging \
            --set image.tag=${{ github.sha }} \
            --set environment=staging \
            -f ./helm/values-staging.yaml

      - name: Run Integration Tests
        run: |
          ./scripts/integration-test.sh https://staging.predictionmarket.com

      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          channel: '#deployments'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  deploy-production:
    needs: deploy-staging
    if: github.event.inputs.environment == 'production'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://predictionmarket.com
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Blue/Green Deploy
        run: |
          aws eks update-kubeconfig --name prediction-market-prod

          # Deploy to green environment
          helm upgrade --install prediction-market-green ./helm/prediction-market \
            --namespace production \
            --set image.tag=${{ github.sha }} \
            --set environment=production \
            --set deployment.color=green \
            -f ./helm/values-production.yaml

          # Wait for green to be healthy
          kubectl rollout status deployment/api-green -n production

          # Run canary tests
          ./scripts/canary-test.sh https://green.predictionmarket.com

          # Switch traffic
          kubectl patch service api -n production \
            -p '{"spec":{"selector":{"color":"green"}}}'

      - name: Post-Deploy Verification
        run: |
          ./scripts/post-deploy-verify.sh

      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ github.run_number }}
          release_name: Release v${{ github.run_number }}
          body: |
            Deployed commit: ${{ github.sha }}
```

---

## Infrastructure as Code

### Terraform Project Structure

```
infrastructure/
├── terraform/
│   ├── modules/
│   │   ├── vpc/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf
│   │   ├── eks/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf
│   │   ├── rds/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf
│   │   └── redis/
│   │       ├── main.tf
│   │       ├── variables.tf
│   │       └── outputs.tf
│   ├── environments/
│   │   ├── development/
│   │   │   ├── main.tf
│   │   │   ├── terraform.tfvars
│   │   │   └── backend.tf
│   │   ├── staging/
│   │   │   ├── main.tf
│   │   │   ├── terraform.tfvars
│   │   │   └── backend.tf
│   │   └── production/
│   │       ├── main.tf
│   │       ├── terraform.tfvars
│   │       └── backend.tf
│   └── shared/
│       ├── providers.tf
│       └── versions.tf
└── pulumi/  # Alternative IaC
    └── ...
```

### EKS Cluster Module

```hcl
# modules/eks/main.tf

resource "aws_eks_cluster" "main" {
  name     = var.cluster_name
  role_arn = aws_iam_role.cluster.arn
  version  = var.kubernetes_version

  vpc_config {
    subnet_ids              = var.subnet_ids
    endpoint_private_access = true
    endpoint_public_access  = var.public_access
    security_group_ids      = [aws_security_group.cluster.id]
  }

  encryption_config {
    provider {
      key_arn = aws_kms_key.eks.arn
    }
    resources = ["secrets"]
  }

  enabled_cluster_log_types = [
    "api",
    "audit",
    "authenticator",
    "controllerManager",
    "scheduler"
  ]

  tags = var.tags
}

resource "aws_eks_node_group" "main" {
  cluster_name    = aws_eks_cluster.main.name
  node_group_name = "${var.cluster_name}-workers"
  node_role_arn   = aws_iam_role.node.arn
  subnet_ids      = var.private_subnet_ids

  scaling_config {
    desired_size = var.desired_capacity
    max_size     = var.max_capacity
    min_size     = var.min_capacity
  }

  instance_types = var.instance_types

  labels = {
    role = "worker"
  }

  tags = var.tags
}

# Cluster Autoscaler
resource "helm_release" "cluster_autoscaler" {
  name       = "cluster-autoscaler"
  repository = "https://kubernetes.github.io/autoscaler"
  chart      = "cluster-autoscaler"
  namespace  = "kube-system"

  set {
    name  = "autoDiscovery.clusterName"
    value = aws_eks_cluster.main.name
  }

  set {
    name  = "awsRegion"
    value = var.region
  }
}
```

### RDS Module with TimescaleDB

```hcl
# modules/rds/main.tf

resource "aws_db_instance" "main" {
  identifier = var.identifier

  engine         = "postgres"
  engine_version = "15.4"
  instance_class = var.instance_class

  allocated_storage     = var.allocated_storage
  max_allocated_storage = var.max_allocated_storage
  storage_type          = "gp3"
  storage_encrypted     = true
  kms_key_id           = aws_kms_key.rds.arn

  db_name  = var.database_name
  username = var.master_username
  password = random_password.master.result

  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name

  multi_az               = var.multi_az
  publicly_accessible    = false

  backup_retention_period = var.backup_retention_days
  backup_window          = "03:00-04:00"
  maintenance_window     = "Mon:04:00-Mon:05:00"

  performance_insights_enabled = true
  monitoring_interval         = 60
  monitoring_role_arn         = aws_iam_role.rds_monitoring.arn

  enabled_cloudwatch_logs_exports = [
    "postgresql",
    "upgrade"
  ]

  deletion_protection = var.environment == "production"

  tags = var.tags
}

# Read Replica
resource "aws_db_instance" "replica" {
  count = var.create_replica ? 1 : 0

  identifier = "${var.identifier}-replica"

  replicate_source_db = aws_db_instance.main.id
  instance_class      = var.replica_instance_class

  vpc_security_group_ids = [aws_security_group.rds.id]

  multi_az            = false
  publicly_accessible = false

  tags = var.tags
}
```

---

## Container Orchestration

### Kubernetes Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Kubernetes Cluster                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │   Ingress    │  │   Ingress    │  │      Ingress         │  │
│  │  (Frontend)  │  │    (API)     │  │   (WebSocket)        │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘  │
│         │                 │                      │              │
│  ┌──────▼───────┐  ┌──────▼───────┐  ┌──────────▼───────────┐  │
│  │   Frontend   │  │     API      │  │      WS Gateway      │  │
│  │   Service    │  │   Service    │  │       Service        │  │
│  │   (3 pods)   │  │   (5 pods)   │  │       (3 pods)       │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │   Indexer    │  │    Worker    │  │    Blockchain        │  │
│  │   Service    │  │   Service    │  │       Node           │  │
│  │   (2 pods)   │  │   (3 pods)   │  │      (1 pod)         │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Helm Chart Structure

```
helm/
├── prediction-market/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── values-development.yaml
│   ├── values-staging.yaml
│   ├── values-production.yaml
│   └── templates/
│       ├── _helpers.tpl
│       ├── api-deployment.yaml
│       ├── api-service.yaml
│       ├── api-hpa.yaml
│       ├── frontend-deployment.yaml
│       ├── frontend-service.yaml
│       ├── ingress.yaml
│       ├── indexer-deployment.yaml
│       ├── worker-deployment.yaml
│       ├── configmap.yaml
│       ├── secrets.yaml
│       └── pdb.yaml
```

### API Deployment

```yaml
# templates/api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "prediction-market.fullname" . }}-api
  labels:
    {{- include "prediction-market.labels" . | nindent 4 }}
    component: api
spec:
  replicas: {{ .Values.api.replicas }}
  selector:
    matchLabels:
      {{- include "prediction-market.selectorLabels" . | nindent 6 }}
      component: api
  template:
    metadata:
      labels:
        {{- include "prediction-market.selectorLabels" . | nindent 8 }}
        component: api
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
    spec:
      serviceAccountName: {{ include "prediction-market.serviceAccountName" . }}
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
      containers:
        - name: api
          image: "{{ .Values.api.image.repository }}:{{ .Values.api.image.tag }}"
          imagePullPolicy: {{ .Values.api.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 3000
              protocol: TCP
            - name: metrics
              containerPort: 9090
              protocol: TCP
          env:
            - name: NODE_ENV
              value: {{ .Values.environment }}
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: {{ include "prediction-market.fullname" . }}-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: {{ include "prediction-market.fullname" . }}-secrets
                  key: redis-url
            - name: RPC_URL
              valueFrom:
                secretKeyRef:
                  name: {{ include "prediction-market.fullname" . }}-secrets
                  key: rpc-url
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            {{- toYaml .Values.api.resources | nindent 12 }}
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels:
              component: api
```

### Horizontal Pod Autoscaler

```yaml
# templates/api-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "prediction-market.fullname" . }}-api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "prediction-market.fullname" . }}-api
  minReplicas: {{ .Values.api.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.api.autoscaling.maxReplicas }}
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.api.autoscaling.targetCPUUtilization }}
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: {{ .Values.api.autoscaling.targetMemoryUtilization }}
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: {{ .Values.api.autoscaling.targetRPS }}
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max
```

---

## Smart Contract Deployment Pipeline

### Contract Deployment Workflow

```yaml
# deploy-contracts.yml
name: Deploy Smart Contracts
on:
  workflow_dispatch:
    inputs:
      network:
        description: 'Network to deploy to'
        required: true
        type: choice
        options:
          - sepolia
          - polygon-amoy
          - polygon-mainnet
      contracts:
        description: 'Contracts to deploy (comma-separated or "all")'
        required: true
        default: 'all'
      verify:
        description: 'Verify on block explorer'
        type: boolean
        default: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.network }}
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - uses: foundry-rs/foundry-toolchain@v1
        with:
          version: nightly

      - name: Run Pre-Deploy Checks
        run: |
          # Run all tests
          forge test -vvv

          # Run Slither
          pip install slither-analyzer
          slither contracts/ --fail-on high

          # Check gas usage
          forge test --gas-report > gas-report.txt
          cat gas-report.txt

      - name: Deploy Contracts
        id: deploy
        run: |
          # Set up environment
          export DEPLOYER_PRIVATE_KEY="${{ secrets.DEPLOYER_PRIVATE_KEY }}"
          export RPC_URL="${{ secrets[format('RPC_URL_{0}', github.event.inputs.network)] }}"

          # Run deployment script
          forge script script/Deploy.s.sol:DeployScript \
            --rpc-url $RPC_URL \
            --broadcast \
            --verify \
            --etherscan-api-key ${{ secrets.ETHERSCAN_API_KEY }} \
            -vvvv

          # Save deployment artifacts
          echo "deployment_file=broadcast/Deploy.s.sol/$CHAIN_ID/run-latest.json" >> $GITHUB_OUTPUT

      - name: Upload Deployment Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: deployment-${{ github.event.inputs.network }}-${{ github.run_number }}
          path: |
            broadcast/
            out/

      - name: Update Deployment Registry
        run: |
          # Parse deployment addresses from broadcast
          node scripts/update-deployment-registry.js \
            --network ${{ github.event.inputs.network }} \
            --file ${{ steps.deploy.outputs.deployment_file }}

      - name: Verify on Tenderly
        if: github.event.inputs.verify == 'true'
        run: |
          tenderly verify \
            --network ${{ github.event.inputs.network }} \
            $(cat ${{ steps.deploy.outputs.deployment_file }} | jq -r '.transactions[].contractAddress')

      - name: Post-Deploy Verification
        run: |
          # Run integration tests against deployed contracts
          forge test --fork-url $RPC_URL -vvv --match-contract Integration

      - name: Notify Team
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          fields: repo,message,commit,author,workflow
          text: |
            Smart contracts deployed to ${{ github.event.inputs.network }}
            Deployment: ${{ steps.deploy.outputs.deployment_file }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Upgrade Contracts Workflow

```yaml
# upgrade-contracts.yml
name: Upgrade Smart Contracts
on:
  workflow_dispatch:
    inputs:
      network:
        description: 'Network'
        required: true
        type: choice
        options:
          - polygon-amoy
          - polygon-mainnet
      contract:
        description: 'Contract to upgrade'
        required: true
        type: choice
        options:
          - MarketFactory
          - TradingEngine
          - Oracle
      dry_run:
        description: 'Dry run (simulate only)'
        type: boolean
        default: true

jobs:
  upgrade:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.network }}
    steps:
      - uses: actions/checkout@v4

      - uses: foundry-rs/foundry-toolchain@v1

      - name: Check Upgrade Safety
        run: |
          # Compare storage layouts
          forge inspect ${{ github.event.inputs.contract }} storage-layout > new-layout.json

          # Fetch current implementation layout
          node scripts/fetch-current-layout.js \
            --network ${{ github.event.inputs.network }} \
            --contract ${{ github.event.inputs.contract }} \
            > current-layout.json

          # Compare layouts
          node scripts/compare-storage-layouts.js \
            --old current-layout.json \
            --new new-layout.json

      - name: Simulate Upgrade
        run: |
          forge script script/Upgrade${{ github.event.inputs.contract }}.s.sol \
            --rpc-url ${{ secrets[format('RPC_URL_{0}', github.event.inputs.network)] }} \
            -vvvv

      - name: Execute Upgrade
        if: github.event.inputs.dry_run == 'false'
        run: |
          forge script script/Upgrade${{ github.event.inputs.contract }}.s.sol \
            --rpc-url ${{ secrets[format('RPC_URL_{0}', github.event.inputs.network)] }} \
            --broadcast \
            --verify \
            -vvvv
```

---

## Environment Management

### Environment Configuration Matrix

| Setting | Development | Staging | Production |
|---------|-------------|---------|------------|
| Chain | Sepolia | Polygon Amoy | Polygon Mainnet |
| Chain ID | 11155111 | 80002 | 137 |
| RPC Provider | Alchemy | Alchemy | Multi (Alchemy + Infura) |
| Database | Single instance | Single + replica | Multi-AZ + replica |
| Redis | Single node | Sentinel | Cluster |
| Replicas | 1-2 | 2-3 | 3-10 |
| Auto-scaling | Disabled | Enabled | Enabled |
| Monitoring | Basic | Full | Full + APM |
| Alerts | Slack only | Slack + Email | PagerDuty |

### Environment Variables

```yaml
# values-production.yaml
environment: production

api:
  replicas: 5
  image:
    repository: ghcr.io/org/prediction-market-api
    tag: latest
  resources:
    requests:
      cpu: 500m
      memory: 1Gi
    limits:
      cpu: 2000m
      memory: 4Gi
  autoscaling:
    enabled: true
    minReplicas: 5
    maxReplicas: 20
    targetCPUUtilization: 70
    targetMemoryUtilization: 80

frontend:
  replicas: 3
  resources:
    requests:
      cpu: 200m
      memory: 512Mi
    limits:
      cpu: 1000m
      memory: 2Gi

indexer:
  replicas: 2
  resources:
    requests:
      cpu: 1000m
      memory: 2Gi

config:
  logLevel: info
  rpcUrls:
    primary: https://polygon-mainnet.g.alchemy.com/v2/
    fallback: https://polygon-mainnet.infura.io/v3/
  features:
    rateLimit: true
    caching: true
    analytics: true
```

---

## Secrets Management

### Secrets Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Secrets Management                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐                    ┌──────────────────────┐   │
│  │   AWS KMS    │◄──────────────────►│  AWS Secrets Manager │   │
│  │  (Encryption)│                    │   (Storage)          │   │
│  └──────────────┘                    └──────────┬───────────┘   │
│                                                  │               │
│                                      ┌───────────▼───────────┐  │
│                                      │  External Secrets     │  │
│                                      │  Operator (K8s)       │  │
│                                      └───────────┬───────────┘  │
│                                                  │               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────▼───────────┐  │
│  │   API Pod    │  │ Frontend Pod │  │   Kubernetes         │  │
│  │              │  │              │  │   Secrets            │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### External Secrets Configuration

```yaml
# external-secrets.yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: prediction-market-secrets
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    kind: ClusterSecretStore
    name: aws-secrets-manager
  target:
    name: prediction-market-secrets
    creationPolicy: Owner
  data:
    - secretKey: database-url
      remoteRef:
        key: production/prediction-market/database
        property: url
    - secretKey: redis-url
      remoteRef:
        key: production/prediction-market/redis
        property: url
    - secretKey: rpc-url
      remoteRef:
        key: production/prediction-market/blockchain
        property: rpc_url
    - secretKey: deployer-key
      remoteRef:
        key: production/prediction-market/blockchain
        property: deployer_private_key
```

### GitHub Secrets Structure

```
Repository Secrets:
├── AWS_ACCESS_KEY_ID
├── AWS_SECRET_ACCESS_KEY
├── GITHUB_TOKEN (auto)
├── SLACK_WEBHOOK
└── CODECOV_TOKEN

Environment: development
├── RPC_URL_sepolia
├── DEPLOYER_PRIVATE_KEY
└── ETHERSCAN_API_KEY

Environment: staging
├── RPC_URL_polygon-amoy
├── DEPLOYER_PRIVATE_KEY
└── POLYGONSCAN_API_KEY

Environment: production
├── RPC_URL_polygon-mainnet
├── DEPLOYER_PRIVATE_KEY (multi-sig)
├── POLYGONSCAN_API_KEY
└── TENDERLY_ACCESS_KEY
```

### Secret Rotation Policy

```yaml
# secrets-rotation-policy.yaml
secrets:
  - name: database-credentials
    rotation_interval: 30d
    notification: 7d_before

  - name: api-keys
    rotation_interval: 90d
    notification: 14d_before

  - name: deployer-keys
    rotation_interval: never  # Multi-sig, manual rotation
    notification: on_use

  - name: jwt-signing-key
    rotation_interval: 7d
    notification: 1d_before
    supports_overlap: true  # Grace period for old key
```

---

## GitHub Actions Examples

### Complete CI/CD Pipeline

```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # Stage 1: Lint all code
  lint:
    runs-on: ubuntu-latest
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
      - run: pnpm lint
      - run: pnpm format:check

  # Stage 2: Test all components
  test-contracts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive
      - uses: foundry-rs/foundry-toolchain@v1
      - run: forge test -vvv --gas-report
      - run: forge coverage --report lcov
      - uses: codecov/codecov-action@v4
        with:
          files: ./lcov.info
          flags: contracts

  test-api:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: timescale/timescaledb:latest-pg15
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: cd api && pnpm install
      - run: cd api && pnpm test:coverage
      - uses: codecov/codecov-action@v4

  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: cd frontend && pnpm install
      - run: cd frontend && pnpm test:coverage

  # Stage 3: Security scans
  security:
    runs-on: ubuntu-latest
    needs: [lint]
    steps:
      - uses: actions/checkout@v4
      - uses: crytic/slither-action@v0.4.0
        with:
          fail-on: medium
      - run: pnpm audit --audit-level=high

  # Stage 4: Build artifacts
  build:
    runs-on: ubuntu-latest
    needs: [test-contracts, test-api, test-frontend, security]
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          context: ./api
          push: ${{ github.ref == 'refs/heads/main' }}
          tags: ghcr.io/${{ github.repository }}/api:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Stage 5: Deploy
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - run: |
          aws eks update-kubeconfig --name staging
          helm upgrade --install prediction-market ./helm \
            --set image.tag=${{ github.sha }} \
            -f ./helm/values-staging.yaml
```

### Reusable Workflow

```yaml
# .github/workflows/reusable-deploy.yml
name: Reusable Deploy

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      image_tag:
        required: true
        type: string
    secrets:
      AWS_ACCESS_KEY_ID:
        required: true
      AWS_SECRET_ACCESS_KEY:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v4

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to Kubernetes
        run: |
          aws eks update-kubeconfig --name ${{ inputs.environment }}
          helm upgrade --install prediction-market ./helm \
            --namespace ${{ inputs.environment }} \
            --set image.tag=${{ inputs.image_tag }} \
            -f ./helm/values-${{ inputs.environment }}.yaml

      - name: Verify Deployment
        run: kubectl rollout status deployment/api -n ${{ inputs.environment }}
```

---

## References

- [Effortless Smart Contract Deployments](https://dev.to/grossiwm/effortless-smart-contract-deployments-2lj8)
- [How to Set up Multichain CD for Smart Contracts](https://blog.tenderly.co/how-to-set-up-continuous-deployment-for-smart-contracts/)
- [Blockchain DevOps Best Practices](https://www.hackquest.io/articles/blockchain-devops-best-practices-building-robust-web3-deployment-pipelines)
- [AWS CI/CD Pipeline for Ethereum](https://aws.amazon.com/blogs/web3/implement-a-ci-cd-pipeline-for-ethereum-smart-contract-development-on-aws-part-1/)
- [Operating Staking Nodes on Kubernetes](https://www.coinbase.com/blog/operating-staking-nodes-on-kubernetes)
- [Foundry Toolchain GitHub Action](https://github.com/foundry-rs/foundry-toolchain)
