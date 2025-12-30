---
name: rest
version: "2.0.0"
description: RESTful API design principles and best practices
sasmp_version: "1.3.0"
bonded_agent: 01-api-architect
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "REST API design patterns and conventions"
  boundaries:
    includes: [resource_design, http_methods, status_codes, pagination, filtering]
    excludes: [graphql, grpc, implementation_code]

parameter_validation:
  schema:
    type: object
    properties:
      resource_name:
        type: string
        pattern: "^[a-z][a-z0-9-]*$"
      http_method:
        type: string
        enum: [GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD]
      response_format:
        type: string
        enum: [json, xml, hal, jsonapi]

retry_config:
  enabled: true
  max_attempts: 3
  backoff:
    type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 30000

logging:
  level: INFO
  fields: [resource, method, status_code, duration_ms]

dependencies:
  skills: [api-architecture]
  agents: [01-api-architect]
---

# REST API Design Skill

## Purpose
Design RESTful APIs following industry best practices.

## HTTP Methods

| Method | Action | Idempotent | Safe | Request Body | Response Body |
|--------|--------|------------|------|--------------|---------------|
| GET | Read | Yes | Yes | No | Yes |
| POST | Create | No | No | Yes | Yes |
| PUT | Replace | Yes | No | Yes | Yes |
| PATCH | Update | No | No | Yes | Yes |
| DELETE | Delete | Yes | No | No | No |
| HEAD | Metadata | Yes | Yes | No | No |
| OPTIONS | Capabilities | Yes | Yes | No | Yes |

## Status Codes

### Success (2xx)
```
200 OK           → GET, PUT, PATCH success
201 Created      → POST success (include Location header)
202 Accepted     → Async operation started
204 No Content   → DELETE success
```

### Client Errors (4xx)
```
400 Bad Request  → Validation failed
401 Unauthorized → Authentication required
403 Forbidden    → Permission denied
404 Not Found    → Resource doesn't exist
409 Conflict     → State conflict (duplicate, etc.)
422 Unprocessable → Semantic errors
429 Too Many     → Rate limit exceeded
```

### Server Errors (5xx)
```
500 Internal     → Unexpected error
502 Bad Gateway  → Upstream failed
503 Unavailable  → Temporarily down
504 Timeout      → Upstream timeout
```

## Resource Design

```yaml
# Collection operations
GET /api/v1/users              → List (paginated)
POST /api/v1/users             → Create

# Instance operations
GET /api/v1/users/{id}         → Read
PUT /api/v1/users/{id}         → Replace
PATCH /api/v1/users/{id}       → Update
DELETE /api/v1/users/{id}      → Delete

# Nested resources (max 2 levels)
GET /api/v1/users/{id}/orders  → User's orders
POST /api/v1/users/{id}/orders → Create user's order

# Sub-resource instance
GET /api/v1/users/{id}/orders/{orderId}

# Actions (when CRUD doesn't fit)
POST /api/v1/orders/{id}/cancel
POST /api/v1/users/{id}/verify
POST /api/v1/reports/{id}/generate
```

## Response Envelope

```json
{
  "data": {
    "id": "123",
    "type": "user",
    "attributes": {
      "name": "John Doe",
      "email": "john@example.com"
    }
  },
  "meta": {
    "requestId": "abc-123",
    "timestamp": "2024-12-30T10:00:00Z",
    "version": "1.0.0"
  }
}
```

## Error Response (RFC 7807)

```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Failed",
  "status": 400,
  "detail": "One or more fields have validation errors",
  "instance": "/api/v1/users",
  "errors": [
    { "field": "email", "message": "Invalid email format" },
    { "field": "password", "message": "Minimum 12 characters" }
  ]
}
```

## Pagination

### Offset-based (Simple)
```http
GET /api/v1/users?page=2&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true
  }
}
```

### Cursor-based (Recommended)
```http
GET /api/v1/users?cursor=eyJpZCI6MTAwfQ&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTIwfQ",
    "prevCursor": "eyJpZCI6ODB9",
    "hasNext": true,
    "hasPrev": true
  }
}
```

## Filtering & Sorting

```http
# Filtering
GET /api/v1/users?status=active&role=admin
GET /api/v1/users?created_at[gte]=2024-01-01
GET /api/v1/users?search=john

# Sorting
GET /api/v1/users?sort=created_at:desc
GET /api/v1/users?sort=-created_at,name  # - prefix for desc

# Field selection
GET /api/v1/users?fields=id,name,email

# Embedding related resources
GET /api/v1/users?include=orders,profile
```

## Caching Headers

```http
# Response headers
Cache-Control: public, max-age=3600
ETag: "abc123"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT
Vary: Accept-Encoding, Authorization

# Conditional requests
If-None-Match: "abc123"
If-Modified-Since: Wed, 21 Oct 2024 07:28:00 GMT
```

---

## Unit Test Template

```typescript
import { describe, it, expect } from 'vitest';
import request from 'supertest';
import app from './app';

describe('REST API - Users', () => {
  describe('GET /api/v1/users', () => {
    it('should return paginated users', async () => {
      const res = await request(app)
        .get('/api/v1/users?page=1&limit=10')
        .expect(200);

      expect(res.body).toHaveProperty('data');
      expect(res.body).toHaveProperty('pagination');
      expect(res.body.pagination.page).toBe(1);
    });

    it('should filter by status', async () => {
      const res = await request(app)
        .get('/api/v1/users?status=active')
        .expect(200);

      res.body.data.forEach(user => {
        expect(user.status).toBe('active');
      });
    });
  });

  describe('POST /api/v1/users', () => {
    it('should create user and return 201', async () => {
      const res = await request(app)
        .post('/api/v1/users')
        .send({ email: 'test@example.com', name: 'Test' })
        .expect(201);

      expect(res.headers.location).toMatch(/\/api\/v1\/users\/\w+/);
      expect(res.body.data.id).toBeDefined();
    });

    it('should return 400 for invalid data', async () => {
      const res = await request(app)
        .post('/api/v1/users')
        .send({ email: 'invalid' })
        .expect(400);

      expect(res.body.type).toContain('validation');
    });
  });

  describe('GET /api/v1/users/:id', () => {
    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/api/v1/users/non-existent-id')
        .expect(404);
    });
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| 404 for valid resource | Trailing slash mismatch | Normalize URLs |
| CORS errors | Missing headers | Configure CORS middleware |
| Slow pagination | Large offset | Use cursor pagination |
| Cache not working | Missing Vary header | Add Vary: Authorization |

---

## Quality Checklist

- [ ] Resource names are plural nouns
- [ ] HTTP methods match actions
- [ ] Status codes are correct
- [ ] Error responses follow RFC 7807
- [ ] Pagination implemented for lists
- [ ] Filtering and sorting supported
- [ ] Caching headers configured
- [ ] Rate limiting in place
