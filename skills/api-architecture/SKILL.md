---
name: api-architecture
description: REST, GraphQL, and hybrid API architecture patterns for building scalable and maintainable APIs
sasmp_version: "2.0.0"
bonded_agent: 01-api-architect
bond_type: PRIMARY_BOND

# Production-Grade Metadata
parameters:
  - name: api_style
    type: string
    required: true
    validation: "^(REST|GraphQL|gRPC|hybrid)$"
    description: API architecture style
  - name: scale_tier
    type: string
    required: false
    validation: "^(startup|growth|enterprise)$"
    description: Expected scale tier

validation_rules:
  - All endpoints must have defined schemas
  - Error responses must follow standard format
  - Versioning strategy must be documented

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [duration, success_rate, api_style_usage]
---

# API Architecture Skill

## Quick Start

Master API design across three major paradigms.

### REST API (Traditional)
- Resource-oriented design
- HTTP status codes and methods
- Stateless communication
- Best for: Simple CRUD, mobile apps, public APIs

### GraphQL (Modern)
- Query language for APIs
- Client specifies exact data
- Single endpoint
- Best for: Complex queries, frontend-heavy apps

### Hybrid (Recommended)
- REST for public/simple operations
- GraphQL for internal/complex operations

## Core Patterns

### Resource Modeling
```http
GET /api/v1/resources              → List
POST /api/v1/resources             → Create
GET /api/v1/resources/{id}         → Read
PUT /api/v1/resources/{id}         → Replace
PATCH /api/v1/resources/{id}       → Update
DELETE /api/v1/resources/{id}      → Delete
```

### Response Format
```json
{
  "data": { },
  "meta": { "timestamp": "..." },
  "pagination": { "cursor": "...", "hasNext": true }
}
```

## API Design Checklist

- [ ] API style chosen
- [ ] Resource models defined
- [ ] Versioning strategy set
- [ ] Error format standardized
- [ ] Rate limiting configured

## Unit Test Template

```javascript
describe('api-architecture skill', () => {
  test('validates REST endpoint naming', () => {
    const endpoint = '/api/v1/users';
    expect(isValidRESTEndpoint(endpoint)).toBe(true);
  });

  test('rejects verb-based URLs', () => {
    const endpoint = '/api/getUsers';
    expect(isValidRESTEndpoint(endpoint)).toBe(false);
  });
});
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| 400 errors | Malformed request | Validate against schema |
| Slow responses | Missing pagination | Implement cursor pagination |
| Version conflicts | No versioning | Add URL path versioning |

See Agent 1: API Design Architect for detailed guidance.
