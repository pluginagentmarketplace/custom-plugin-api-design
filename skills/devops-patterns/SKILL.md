---
name: devops-patterns
version: "2.0.0"
description: Infrastructure, deployment, and operations patterns for Docker, Kubernetes, and CI/CD
sasmp_version: "1.3.0"
bonded_agent: 04-devops-infrastructure
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "Infrastructure and deployment automation"
  boundaries:
    includes: [docker, kubernetes, terraform, ci_cd, monitoring]
    excludes: [application_code, database_queries, security_policies]

parameter_validation:
  schema:
    type: object
    properties:
      platform:
        type: string
        enum: [docker, kubernetes, terraform, github_actions]
      environment:
        type: string
        enum: [development, staging, production]
      cloud_provider:
        type: string
        enum: [aws, gcp, azure, self_hosted]

retry_config:
  enabled: true
  max_attempts: 3
  backoff:
    type: exponential
    initial_delay_ms: 5000
    max_delay_ms: 60000

logging:
  level: INFO
  fields: [platform, environment, deployment_id, duration_ms]

dependencies:
  skills: []
  agents: [04-devops-infrastructure]
---

# DevOps Patterns Skill

## Purpose
Deploy and operate production-grade infrastructure with modern DevOps practices.

## Docker Patterns

### Multi-Stage Build (Optimized)

```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Stage 2: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Production
FROM node:20-alpine AS runner
WORKDIR /app

# Security: non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 appuser

# Copy only necessary files
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

# Security hardening
USER appuser
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/index.js"]
```

### Docker Compose for Development

```yaml
version: '3.8'

services:
  api:
    build:
      context: .
      target: builder
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://user:pass@db:5432/app
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  db:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Kubernetes Patterns

### Production Deployment

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
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
    spec:
      serviceAccountName: api-service
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
      containers:
      - name: api
        image: myregistry/api:v1.2.3
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
          name: http
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: database-url
        livenessProbe:
          httpGet:
            path: /health/live
            port: http
          initialDelaySeconds: 10
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health/ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
          failureThreshold: 3
        startupProbe:
          httpGet:
            path: /health/startup
            port: http
          failureThreshold: 30
          periodSeconds: 10
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: api
              topologyKey: kubernetes.io/hostname
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
  ports:
  - port: 80
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
  maxReplicas: 10
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

## Terraform Patterns

### AWS EKS Module

```hcl
# main.tf
terraform {
  required_version = ">= 1.5.0"

  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "eks/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.25"
    }
  }
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = var.cluster_name
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  cluster_endpoint_public_access = true

  eks_managed_node_groups = {
    general = {
      desired_size = 3
      min_size     = 2
      max_size     = 10

      instance_types = ["t3.medium"]
      capacity_type  = "ON_DEMAND"

      labels = {
        role = "general"
      }
    }
  }

  tags = local.tags
}

# outputs.tf
output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}

output "cluster_name" {
  value = module.eks.cluster_name
}
```

## CI/CD Pipeline

### GitHub Actions

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
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

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test -- --coverage

    - name: Upload coverage
      uses: codecov/codecov-action@v4

  build:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    outputs:
      image_tag: ${{ steps.meta.outputs.tags }}
    steps:
    - uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Login to Container Registry
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
          type=ref,event=branch
          type=semver,pattern={{version}}

    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    steps:
    - uses: actions/checkout@v4

    - name: Deploy to Staging
      uses: azure/k8s-deploy@v4
      with:
        manifests: k8s/staging/
        images: ${{ needs.build.outputs.image_tag }}

  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
    - uses: actions/checkout@v4

    - name: Deploy to Production
      uses: azure/k8s-deploy@v4
      with:
        manifests: k8s/production/
        images: ${{ needs.build.outputs.image_tag }}
        strategy: canary
        percentage: 20
```

---

## Unit Test Template

```typescript
import { describe, it, expect } from 'vitest';
import { execSync } from 'child_process';

describe('DevOps Patterns', () => {
  describe('Dockerfile', () => {
    it('should build successfully', () => {
      expect(() => {
        execSync('docker build -t test-app .', { stdio: 'pipe' });
      }).not.toThrow();
    });

    it('should run as non-root user', () => {
      const output = execSync(
        'docker run --rm test-app whoami',
        { encoding: 'utf-8' }
      );
      expect(output.trim()).not.toBe('root');
    });

    it('should expose health endpoint', async () => {
      execSync('docker run -d --name test-container -p 3001:3000 test-app');
      await new Promise(r => setTimeout(r, 5000));

      const response = await fetch('http://localhost:3001/health');
      expect(response.ok).toBe(true);

      execSync('docker rm -f test-container');
    });
  });

  describe('Kubernetes manifests', () => {
    it('should pass kubeval validation', () => {
      expect(() => {
        execSync('kubeval k8s/*.yaml', { stdio: 'pipe' });
      }).not.toThrow();
    });

    it('should have resource limits', () => {
      const manifest = execSync('cat k8s/deployment.yaml', { encoding: 'utf-8' });
      expect(manifest).toContain('resources:');
      expect(manifest).toContain('limits:');
      expect(manifest).toContain('requests:');
    });
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| OOMKilled pods | Memory limit too low | Increase limits, profile memory |
| ImagePullBackOff | Registry auth failed | Check imagePullSecrets |
| CrashLoopBackOff | App startup fails | Check logs, increase startupProbe |
| Pending pods | Insufficient resources | Add nodes or reduce requests |
| Slow deployments | Large images | Use multi-stage builds, slim base |

---

## Quality Checklist

- [ ] Docker images optimized (multi-stage, slim base)
- [ ] Non-root user in containers
- [ ] Health checks configured
- [ ] Kubernetes manifests validated
- [ ] Resource limits set
- [ ] HPA configured
- [ ] CI/CD pipeline automated
- [ ] Secrets managed securely
- [ ] Monitoring and alerting configured
- [ ] Disaster recovery plan documented
