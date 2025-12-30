---
name: 04-devops-infrastructure
description: Infrastructure, deployment, and operations - Docker, Kubernetes, AWS, Terraform, CI/CD aligned with DevOps and Infrastructure roles
model: sonnet
tools: All tools
sasmp_version: "2.0.0"
eqhm_enabled: true
skills:
  - devops-patterns
triggers:
  - Docker
  - Kubernetes
  - Terraform
  - CI/CD
  - AWS infrastructure
  - deployment
  - containerization
capabilities:
  - Containerization
  - Kubernetes orchestration
  - Cloud infrastructure
  - CI/CD pipelines
  - Infrastructure as Code
  - Monitoring
  - Scaling strategies

# Production-Grade Metadata
input_schema:
  type: object
  required: [infrastructure_type]
  properties:
    infrastructure_type:
      type: string
      enum: [docker, kubernetes, terraform, cicd, monitoring]
    cloud_provider:
      type: string
      enum: [aws, gcp, azure, self-hosted]
    operation:
      type: string
      enum: [setup, optimize, debug, migrate]
    existing_config:
      type: string
      description: Current infrastructure configuration

output_schema:
  type: object
  properties:
    configuration:
      type: object
      properties:
        files: { type: array }
        commands: { type: array }
    deployment_steps:
      type: array
      items: { type: string }
    rollback_procedure:
      type: string

error_codes:
  - code: CONTAINER_BUILD_FAILED
    message: Docker build failed
    recovery: Check Dockerfile syntax and base image availability
  - code: K8S_DEPLOYMENT_FAILED
    message: Kubernetes deployment failed
    recovery: Check pod logs and resource limits
  - code: TERRAFORM_PLAN_FAILED
    message: Terraform plan failed
    recovery: Validate HCL syntax and provider credentials

fallback_strategy:
  type: graceful_degradation
  fallback_to: rollback
  actions:
    - Automatic rollback to previous version
    - Scale down gracefully
    - Preserve logs for debugging

token_budget:
  max_input: 8000
  max_output: 16000
  context_window: 100000

cost_tier: standard

retry_config:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

observability:
  log_level: info
  metrics:
    - deployment_duration
    - rollback_rate
    - container_health
    - resource_utilization
  trace_enabled: true
---

# DevOps & Infrastructure at Scale

## Docker Containerization

### Production Dockerfile

```dockerfile
# Multi-stage build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine
WORKDIR /app

# Security: non-root user
RUN addgroup -g 1001 appgroup && adduser -u 1001 -G appgroup -s /bin/sh -D appuser
USER appuser

COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --chown=appuser:appgroup . .

ENV NODE_ENV=production
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

CMD ["node", "dist/index.js"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/myapp
      REDIS_URL: redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_healthy
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s

volumes:
  postgres_data:
```

## Kubernetes Deployment

### Deployment with Best Practices

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
  labels:
    app: api
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      serviceAccountName: api-service
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
      containers:
      - name: api
        image: myregistry/api:v1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-service
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## Infrastructure as Code (Terraform)

### AWS Infrastructure

```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" {
  region = var.aws_region
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  tags = { Name = "${var.project}-vpc" }
}

# RDS Database
resource "aws_db_instance" "main" {
  identifier           = "${var.project}-db"
  engine               = "postgres"
  engine_version       = "15"
  instance_class       = "db.t3.medium"
  allocated_storage    = 20

  db_name              = var.db_name
  username             = var.db_username
  password             = var.db_password

  multi_az             = true
  backup_retention_period = 30
  deletion_protection  = true

  skip_final_snapshot  = false
  final_snapshot_identifier = "${var.project}-db-final"
}

# ElastiCache Redis
resource "aws_elasticache_cluster" "redis" {
  cluster_id           = "${var.project}-cache"
  engine               = "redis"
  node_type            = "cache.t3.micro"
  num_cache_nodes      = 1
  port                 = 6379
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "${var.project}-cluster"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
```

## CI/CD Pipeline (GitHub Actions)

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v4

      - name: Log in to registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/api-service \
            api=${{ needs.build-and-push.outputs.image-tag }} \
            -n production
          kubectl rollout status deployment/api-service \
            -n production --timeout=5m
```

## Monitoring & Observability

### Prometheus Metrics

```javascript
const prometheus = require('prom-client');

const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path || 'unknown', res.statusCode)
      .observe(duration);
    httpRequestTotal
      .labels(req.method, req.route?.path || 'unknown', res.statusCode)
      .inc();
  });
  next();
});
```

## DevOps Checklist

- [ ] Docker images optimized and scanned
- [ ] Kubernetes manifests validated
- [ ] Infrastructure as Code reviewed
- [ ] CI/CD pipeline automated
- [ ] Secrets management configured
- [ ] Monitoring and alerting in place
- [ ] Auto-scaling policies set
- [ ] Disaster recovery plan tested
- [ ] Rollback procedure documented

---

## Troubleshooting

### Common Failure Modes

| Error | Root Cause | Recovery |
|-------|------------|----------|
| ImagePullBackOff | Registry auth or image missing | Check credentials, verify image exists |
| CrashLoopBackOff | App crashes on start | Check container logs |
| OOMKilled | Memory limit exceeded | Increase limits or fix leak |
| Pending pod | Insufficient resources | Scale cluster or reduce requests |

### Debug Checklist

1. [ ] Check pod status: `kubectl get pods`
2. [ ] View pod logs: `kubectl logs <pod>`
3. [ ] Describe pod: `kubectl describe pod <pod>`
4. [ ] Check events: `kubectl get events`
5. [ ] Exec into pod: `kubectl exec -it <pod> -- sh`

### Log Interpretation

```
INFO: DEPLOYMENT_STARTED version=v1.2.0 replicas=3
  → Deployment initiated

WARN: POD_RESTART count=3 reason=OOMKilled
  → Memory issues, increase limits

ERROR: HEALTHCHECK_FAILED endpoint=/health status=503
  → Application unhealthy, check dependencies

ERROR: ROLLBACK_TRIGGERED reason=deployment_timeout
  → Automatic rollback executed
```

### Deployment Rollback

```bash
# Check rollout history
kubectl rollout history deployment/api-service

# Rollback to previous
kubectl rollout undo deployment/api-service

# Rollback to specific revision
kubectl rollout undo deployment/api-service --to-revision=2
```

---

**Next:** Security & Compliance (Agent 5)
