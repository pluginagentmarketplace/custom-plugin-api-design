# DevOps & Infrastructure Complete Guide

Production patterns for containerization, orchestration, and CI/CD.

## Docker Best Practices

### Multi-Stage Build
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

ENV NODE_ENV=production
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health')"

CMD ["node", "dist/index.js"]
```

### Image Optimization
| Technique | Impact |
|-----------|--------|
| Alpine base | 70% size reduction |
| Multi-stage | 50% size reduction |
| .dockerignore | Faster builds |
| Layer caching | 80% build time reduction |

### Docker Compose (Development)
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/app
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: pass
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
```

## Kubernetes Deployment

### Deployment Manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    spec:
      containers:
      - name: api
        image: myregistry/api:latest
        ports:
        - containerPort: 3000
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health/live
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Horizontal Pod Autoscaler
```yaml
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
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## Infrastructure as Code (Terraform)

### AWS Infrastructure
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
}

resource "aws_db_instance" "main" {
  identifier        = "app-db"
  engine            = "postgres"
  engine_version    = "15.3"
  instance_class    = "db.t3.micro"
  allocated_storage = 20

  multi_az                = true
  backup_retention_period = 30
  skip_final_snapshot     = false
}

resource "aws_elasticache_cluster" "redis" {
  cluster_id      = "app-cache"
  engine          = "redis"
  node_type       = "cache.t3.micro"
  num_cache_nodes = 1
}
```

## CI/CD Pipeline (GitHub Actions)

### Complete Pipeline
```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: docker build -t myregistry/api:${{ github.sha }} .
      - run: docker push myregistry/api:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: azure/k8s-set-context@v3
      - run: |
          kubectl set image deployment/api api=myregistry/api:${{ github.sha }}
          kubectl rollout status deployment/api --timeout=5m
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

app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path, res.statusCode)
      .observe(duration);
  });
  next();
});
```

## Deployment Strategies

| Strategy | Downtime | Rollback | Use Case |
|----------|----------|----------|----------|
| Rolling | None | Fast | Most apps |
| Blue-Green | None | Instant | Critical apps |
| Canary | None | Fast | High-risk changes |
| Recreate | Yes | Slow | Stateful apps |

## DevOps Checklist

### Container
- [ ] Multi-stage Dockerfile
- [ ] .dockerignore configured
- [ ] Health checks defined
- [ ] Non-root user
- [ ] Minimal base image

### Kubernetes
- [ ] Resource limits set
- [ ] Liveness/readiness probes
- [ ] HPA configured
- [ ] PDB defined
- [ ] Network policies

### CI/CD
- [ ] Automated testing
- [ ] Security scanning
- [ ] Image vulnerability check
- [ ] Deployment automation
- [ ] Rollback procedure

### Monitoring
- [ ] Metrics collection
- [ ] Log aggregation
- [ ] Alerting configured
- [ ] Dashboards created

## Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Terraform Registry](https://registry.terraform.io/)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [GitHub Actions](https://docs.github.com/en/actions)
