# API Design Complete Guide

Production-ready patterns for REST, GraphQL, and hybrid APIs.

## Richardson Maturity Model

```
Level 0: POX (Plain Old XML/JSON)
├── Single endpoint for all operations
├── POST /api with operation in body
└── Example: SOAP-style RPC

Level 1: Resources
├── Multiple endpoints for resources
├── POST /users, POST /orders
└── Still uses single method

Level 2: HTTP Verbs + Status Codes
├── GET /users (list), POST /users (create)
├── PUT /users/123 (update), DELETE /users/123
└── Proper HTTP status codes

Level 3: HATEOAS (Hypermedia)
├── Links in responses for navigation
├── Clients discover actions dynamically
└── Self-documenting API
```

## Resource Design

### URL Patterns
```
# Good - Nouns, plural, hierarchical
GET  /api/v1/organizations
GET  /api/v1/organizations/123
GET  /api/v1/organizations/123/teams
GET  /api/v1/organizations/123/teams/456/members

# Bad - Verbs, actions in URL
GET  /api/v1/getOrganization
POST /api/v1/createUser
GET  /api/v1/organizations/deleteTeam
```

### Naming Conventions
| Type | Convention | Example |
|------|-----------|---------|
| Resources | Plural nouns | /users, /orders |
| Actions | POST to resource | POST /orders |
| Search | Query params | /users?role=admin |
| Nested | Parent/child | /users/123/posts |

## Versioning Strategies

### Path Versioning (Recommended)
```
/api/v1/users
/api/v2/users

Pros:
- Clear and visible
- Easy to route
- Cache-friendly

Cons:
- URL changes
- More endpoints to maintain
```

### Header Versioning
```
Accept: application/vnd.company.v2+json

Pros:
- Clean URLs
- Flexible

Cons:
- Harder to test
- Less visible
```

### Backward Compatibility

**Safe Changes (Non-Breaking):**
- Add optional field with default
- Add new endpoint
- Add new optional parameter
- Make validation more lenient

**Breaking Changes (Major Version):**
- Remove field
- Rename field
- Change type of field
- Change status code meanings

**Deprecation Timeline:**
```
v1.0 → v1.5 (Deprecation announced)
     → v2.0 (New version, v1 sunset set)
     → v3.0 (v1 removed, v2 stable)
```

## Response Design

### Success Response
```json
{
  "data": {
    "id": "user-123",
    "type": "user",
    "attributes": {
      "name": "John Doe",
      "email": "john@example.com"
    },
    "relationships": {
      "organization": {
        "id": "org-456"
      }
    }
  },
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z",
    "version": "v1"
  },
  "links": {
    "self": "/api/v1/users/user-123",
    "organization": "/api/v1/organizations/org-456"
  }
}
```

### Error Response
```json
{
  "errors": [
    {
      "code": "VALIDATION_ERROR",
      "message": "Email format is invalid",
      "status": 422,
      "source": {
        "pointer": "/data/attributes/email"
      },
      "details": {
        "field": "email",
        "value": "invalid-email",
        "constraint": "email_format"
      }
    }
  ]
}
```

### Pagination Response
```json
{
  "data": [...],
  "pagination": {
    "cursor": "eyJpZCI6IDEwMH0=",
    "has_next": true,
    "has_prev": false
  },
  "links": {
    "next": "/api/v1/users?cursor=eyJpZCI6IDEwMH0=",
    "prev": null
  }
}
```

## Pagination Strategies

### Offset Pagination
```
GET /users?offset=0&limit=20
GET /users?offset=20&limit=20

Pros:
- Simple to implement
- Jump to any page

Cons:
- O(n) database scan
- Inconsistent with real-time data
- Max offset limits needed
```

### Cursor Pagination (Recommended)
```
GET /users?cursor=abc123&limit=20

Pros:
- O(1) positioning
- Consistent with changes
- Works at any scale

Cons:
- No page jumping
- More complex cursors
```

### Keyset Pagination (Best Performance)
```
GET /users?after=2024-01-01&sort=created_at:desc&limit=20

Pros:
- Uses indexes efficiently
- Fastest for sorted data
- Scales infinitely

Cons:
- Requires sortable field
- No arbitrary jumping
```

## GraphQL Patterns

### Schema Design
```graphql
type Query {
  user(id: ID!): User
  users(first: Int, after: String): UserConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
}

type User {
  id: ID!
  name: String!
  email: String!
  posts(first: Int): [Post!]!
}

type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}
```

### N+1 Prevention
```javascript
// Use DataLoader for batching
const userLoader = new DataLoader(async (userIds) => {
  const users = await db.query(
    'SELECT * FROM users WHERE id = ANY($1)',
    [userIds]
  );
  return userIds.map(id => users.find(u => u.id === id));
});

// In resolver
resolve: (parent) => userLoader.load(parent.userId)
```

## API Gateway Pattern

```
Clients (Web, Mobile, IoT)
         ↓
┌─────────────────────────────┐
│       API Gateway           │
├─────────────────────────────┤
│ • Authentication            │
│ • Rate Limiting             │
│ • Request Routing           │
│ • Response Caching          │
│ • Logging & Monitoring      │
│ • SSL Termination           │
│ • Request/Response Transform│
└─────────────────────────────┘
         ↓
┌─────────────────────────────┐
│     Microservices           │
├──────┬──────┬──────┬────────┤
│User  │Order │Payment│Notify  │
│Svc   │Svc   │Svc    │Svc     │
└──────┴──────┴──────┴────────┘
```

## API Design Checklist

### Design Phase
- [ ] API style chosen (REST/GraphQL/Hybrid)
- [ ] Resource models defined
- [ ] URL patterns established
- [ ] Versioning strategy set
- [ ] Error format standardized
- [ ] Response envelope designed

### Security
- [ ] Authentication method selected
- [ ] Authorization rules defined
- [ ] Rate limiting configured
- [ ] Input validation planned
- [ ] CORS policy set

### Documentation
- [ ] OpenAPI/GraphQL schema written
- [ ] Examples included
- [ ] Error codes documented
- [ ] Changelog maintained

### Operations
- [ ] Health check endpoints
- [ ] Metrics collection
- [ ] Logging strategy
- [ ] Caching headers

## Resources

- [REST API Design Guidelines](https://restfulapi.net/)
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)
- [OpenAPI Specification](https://swagger.io/specification/)
- [JSON:API Specification](https://jsonapi.org/)
