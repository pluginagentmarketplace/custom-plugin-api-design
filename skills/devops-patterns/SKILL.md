---
name: devops-patterns
description: Infrastructure, deployment, and operations patterns for Docker, Kubernetes, and CI/CD
sasmp_version: "2.0.0"
bonded_agent: 04-devops-infrastructure
bond_type: PRIMARY_BOND

# Production-Grade Metadata
parameters:
  - name: infrastructure_type
    type: string
    required: true
    validation: "^(docker|kubernetes|terraform|cicd|monitoring)$"
    description: Infrastructure type
  - name: cloud_provider
    type: string
    required: false
    validation: "^(aws|gcp|azure|self-hosted)$"
    description: Cloud provider

validation_rules:
  - Docker images must use multi-stage builds
  - K8s deployments must have resource limits
  - Secrets must not be in plain text

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [deployment_duration, rollback_rate, container_health]
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
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: api
        resources:
          requests: { memory: "256Mi", cpu: "250m" }
          limits: { memory: "512Mi", cpu: "500m" }
        livenessProbe:
          httpGet: { path: /health, port: 3000 }
```

## DevOps Checklist

- [ ] Docker images optimized
- [ ] K8s manifests validated
- [ ] CI/CD pipeline automated
- [ ] Monitoring configured
- [ ] Auto-scaling enabled

## Unit Test Template

```javascript
describe('devops-patterns skill', () => {
  test('validates Dockerfile has health check', () => {
    const dockerfile = readFileSync('Dockerfile');
    expect(dockerfile).toContain('HEALTHCHECK');
  });

  test('K8s deployment has resource limits', () => {
    const deployment = parseYAML('deployment.yaml');
    expect(deployment.spec.template.spec.containers[0].resources.limits).toBeDefined();
  });
});
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| ImagePullBackOff | Registry auth failed | Check credentials |
| CrashLoopBackOff | App crashes on start | Check container logs |
| OOMKilled | Memory limit exceeded | Increase limits |

See Agent 4: DevOps Infrastructure for detailed guidance.
