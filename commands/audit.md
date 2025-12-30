---
name: audit
version: "2.0.0"
description: Audit Your API Design
sasmp_version: "1.3.0"
allowed-tools: Read

# Command Configuration
input_schema:
  type: object
  properties:
    spec_type:
      type: string
      enum: [openapi, graphql, grpc, architecture]
    spec_content:
      type: string
      description: "OpenAPI YAML/JSON, GraphQL SDL, or description"
    focus_areas:
      type: array
      items:
        type: string
        enum: [security, performance, design, compatibility, documentation]

output_schema:
  type: object
  properties:
    score:
      type: object
      properties:
        overall: { type: number, minimum: 0, maximum: 100 }
        security: { type: number }
        design: { type: number }
        performance: { type: number }
        documentation: { type: number }
    issues:
      type: array
      items:
        type: object
        properties:
          severity: { type: string, enum: [critical, high, medium, low] }
          category: { type: string }
          description: { type: string }
          location: { type: string }
          fix: { type: string }
    recommendations:
      type: array

orchestration:
  agents:
    - 01-api-architect      # Design patterns
    - 05-security-compliance # Security review
    - 08-api-documentation   # Documentation quality
  execution: parallel

scoring:
  weights:
    security: 0.30
    design: 0.25
    performance: 0.20
    documentation: 0.15
    compatibility: 0.10
---

# /audit - Comprehensive API Design Audit

Get a thorough review of your API design with actionable improvements.

## What You'll Get

```
┌─────────────────────────────────────────────────────────────┐
│                    Audit Report                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Overall Score: 72/100                                       │
│  ├── Security:      68/100  ⚠️                               │
│  ├── Design:        85/100  ✅                               │
│  ├── Performance:   70/100  ⚠️                               │
│  ├── Documentation: 65/100  ⚠️                               │
│  └── Compatibility: 80/100  ✅                               │
│                                                              │
│  Critical Issues: 2                                          │
│  High Issues: 5                                              │
│  Medium Issues: 12                                           │
│  Low Issues: 8                                               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## How to Use

Share your API specification:

**Option 1: OpenAPI/Swagger**
```yaml
openapi: 3.1.0
info:
  title: My API
  version: 1.0.0
paths:
  /users:
    get:
      summary: List users
      # ... rest of spec
```

**Option 2: GraphQL Schema**
```graphql
type Query {
  users(first: Int, after: String): UserConnection!
  user(id: ID!): User
}

type User {
  id: ID!
  email: String!
  # ...
}
```

**Option 3: Architecture Description**
```
Our API uses REST with JWT authentication.
Endpoints: /users, /orders, /products
Database: PostgreSQL with Redis caching
```

## Audit Categories

### Security Audit
- [ ] Authentication mechanism review
- [ ] Authorization patterns
- [ ] Input validation coverage
- [ ] Rate limiting configuration
- [ ] OWASP Top 10 compliance
- [ ] Sensitive data exposure risks

### Design Audit
- [ ] RESTful conventions adherence
- [ ] Resource naming consistency
- [ ] HTTP method correctness
- [ ] Status code usage
- [ ] Error response format (RFC 7807)
- [ ] Pagination implementation

### Performance Audit
- [ ] N+1 query risks
- [ ] Caching opportunities
- [ ] Payload size optimization
- [ ] Compression headers
- [ ] Connection pooling

### Documentation Audit
- [ ] Endpoint descriptions
- [ ] Request/response examples
- [ ] Error documentation
- [ ] Authentication guide
- [ ] SDK generation readiness

## Example Output

```yaml
Audit Results:
  overall_score: 72

  critical_issues:
    - id: SEC-001
      category: Security
      severity: critical
      title: "Missing authentication on admin endpoints"
      location: "GET /api/admin/users"
      description: "Admin endpoints accessible without authentication"
      fix: |
        Add authentication middleware:
        ```typescript
        app.use('/api/admin/*', authenticate, requireRole('admin'));
        ```

    - id: SEC-002
      category: Security
      severity: critical
      title: "SQL injection vulnerability"
      location: "GET /api/users?search="
      description: "Search parameter not properly sanitized"
      fix: |
        Use parameterized queries:
        ```typescript
        db.query('SELECT * FROM users WHERE name LIKE $1', [`%${search}%`]);
        ```

  high_issues:
    - id: DES-001
      category: Design
      severity: high
      title: "Inconsistent resource naming"
      location: "GET /api/getUsers vs GET /api/products"
      description: "Mix of verb-based and noun-based endpoints"
      fix: "Use noun-based: GET /api/users"

  recommendations:
    - category: Performance
      title: "Add cursor-based pagination"
      benefit: "Better performance for large datasets"
      effort: medium

    - category: Documentation
      title: "Add response examples"
      benefit: "Improved developer experience, better SDK generation"
      effort: low
```

## Severity Levels

| Level | Description | Action Required |
|-------|-------------|-----------------|
| Critical | Security vulnerability, data loss risk | Immediate fix |
| High | Significant design flaw | Fix before release |
| Medium | Best practice violation | Plan to fix |
| Low | Minor improvement | Nice to have |

---

> **Tip:** Include your full OpenAPI spec for the most comprehensive audit. Partial specs will receive partial analysis.
