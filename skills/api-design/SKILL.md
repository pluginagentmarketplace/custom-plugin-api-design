---
name: api-design
description: Core API design principles, resource modeling, HTTP semantics, and foundational architecture patterns for building robust APIs
---

# API Design Skill

## Quick Start

Master the fundamentals of API design by understanding:

### REST Principles
- **Resource-Oriented** - Design around nouns (resources)
- **Stateless** - Each request is complete
- **Client-Server** - Clear separation
- **Cacheable** - Responses cached when appropriate
- **Uniform Interface** - Consistent across API

### HTTP Methods
```
GET     → Retrieve resources (safe, idempotent)
POST    → Create new resources
PUT     → Replace entire resource (idempotent)
PATCH   → Partial update
DELETE  → Remove resources (idempotent)
HEAD    → Like GET without response body
OPTIONS → Describe available methods
```

### Resource Design

Good:
```
GET /api/users              ✅ Collection
GET /api/users/123          ✅ Resource
GET /api/users/123/posts    ✅ Related resource
```

Bad:
```
GET /api/getUsers           ❌ RPC-style
GET /api/user/123           ❌ Singular collection
POST /api/createUser        ❌ Verb in URL
```

### Status Codes

#### Success (2xx)
- **200 OK** - Request succeeded
- **201 Created** - Resource created
- **204 No Content** - Success, no response body

#### Redirection (3xx)
- **301 Moved Permanently**
- **304 Not Modified** - Use cache

#### Client Error (4xx)
- **400 Bad Request** - Invalid syntax
- **401 Unauthorized** - Auth required
- **403 Forbidden** - Not authorized
- **404 Not Found** - Resource missing
- **409 Conflict** - State conflict

#### Server Error (5xx)
- **500 Internal Server Error**
- **503 Service Unavailable**

## API Design Checklist

- [ ] Resources clearly defined
- [ ] HTTP methods used correctly
- [ ] Status codes appropriate
- [ ] Error responses consistent
- [ ] Request/response formats documented
- [ ] Versioning strategy defined
- [ ] Rate limiting considered
- [ ] Security requirements identified

See the API Design Fundamentals agent for more details.
