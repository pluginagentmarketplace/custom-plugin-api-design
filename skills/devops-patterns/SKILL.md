---
name: devops-patterns
description: Infrastructure, deployment, and operations patterns for Docker, Kubernetes, and CI/CD
sasmp_version: "1.3.0"
bonded_agent: 04-devops-infrastructure
bond_type: PRIMARY_BOND
---

# DevOps Patterns Skill

## Quick Start

Infrastructure and deployment patterns for scalable applications.

### Containerization
- Docker multi-stage builds
- Container optimization
- Health checks

### Orchestration
- Kubernetes deployments
- Service discovery
- Auto-scaling (HPA)

### Infrastructure as Code
- Terraform modules
- AWS/GCP/Azure patterns
- GitOps workflows

### CI/CD
- GitHub Actions pipelines
- Deployment strategies
- Rollback procedures

## Key Patterns

### Multi-Stage Dockerfile
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
CMD ["node", "dist/index.js"]
```

### Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: api
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
```

## DevOps Checklist

- [ ] Docker images optimized
- [ ] Kubernetes manifests validated
- [ ] CI/CD pipeline automated
- [ ] Monitoring configured
- [ ] Auto-scaling enabled
- [ ] Disaster recovery plan

See Agent 4: DevOps Infrastructure for detailed guidance.
