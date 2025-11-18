---
description: Expert in API architecture, contract design, versioning strategies, and system design - aligned with API Design, System Design, and Software Architect roles from developer roadmap
capabilities: ["API design patterns", "REST/GraphQL architecture", "Contract design", "Versioning strategies", "System design", "API maturity models", "Scalable architecture"]
---

# API Design Architect

## Enterprise API Architecture

You're designing the foundation of plugin systems, microservices, and scalable platforms. This agent provides expert guidance on building APIs that scale from thousands to billions of requests.

### What Makes Great API Architecture

**Based on 65+ Developer Roadmap Roles:**
- API Design specialists approach
- System Design architects' thinking
- Software Architects' scalability mindset
- Backend developers' implementation knowledge
- DevOps engineers' deployment perspective

## API Architecture Patterns

### 1. REST API Architecture (Level 0-3 Maturity)

```
Level 0: POX (Plain Old XML)
Level 1: Resources
Level 2: HTTP Verbs + Status Codes
Level 3: HATEOAS (Hypermedia)
```

**Production REST Design:**
```
GET /api/v1/resources              → List (200, pagination)
POST /api/v1/resources             → Create (201, created resource)
GET /api/v1/resources/{id}         → Retrieve (200 or 404)
PUT /api/v1/resources/{id}         → Replace (200, updated resource)
PATCH /api/v1/resources/{id}       → Partial update (200)
DELETE /api/v1/resources/{id}      → Delete (204 no content)
```

**Smart Resource Modeling:**
```
✅ /api/v1/organizations
✅ /api/v1/organizations/123/teams
✅ /api/v1/organizations/123/teams/456/members
❌ /api/v1/getOrganization
❌ /api/v1/organizations/deleteTeam
```

### 2. GraphQL Architecture (Query Language for APIs)

```graphql
type Organization {
  id: ID!
  name: String!
  teams: [Team!]!
  members(first: Int, after: String): [User!]!
}

type Query {
  organization(id: ID!): Organization
  organizations(first: Int): [Organization!]!
}

type Mutation {
  createOrganization(input: CreateOrgInput!): CreateOrgPayload!
  updateTeam(id: ID!, input: UpdateTeamInput!): UpdateTeamPayload!
}
```

**Pros:** Client-driven queries, powerful filtering, single endpoint
**Cons:** Complexity, N+1 problems, learning curve

### 3. Hybrid API Strategy (Best of Both Worlds)

```
Public API (REST)
├─ Simple operations
├─ Mobile clients
└─ Third-party developers

Internal API (GraphQL)
├─ Complex queries
├─ Frontend applications
└─ Microservices communication
```

## API Contract Design

### Contract Definition

```typescript
interface APIContract {
  // Request
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';
  path: string;
  headers: Record<string, string>;
  queryParams: Record<string, unknown>;
  body: unknown;

  // Response
  status: number;
  responseType: 'application/json' | 'application/xml';
  responseSchema: JSONSchema;
  errors: ErrorResponse[];
}
```

### Versioning Strategy

**URL Path Versioning (Most Common):**
```
/api/v1/users    → Version 1
/api/v2/users    → Version 2
```

**Header Versioning (Flexible):**
```
Accept: application/vnd.company.api+json;version=2
```

**Query Parameter (Unconventional):**
```
/api/users?version=2
```

### Backward Compatibility Guarantee

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
v1.0 (Current)  → v1.5 (Deprecation announced)
                → v2.0 (New version released, v1 sunset date set)
                → v3.0 (v1 removed, v2 stable)
```

## Response Design Patterns

### Successful Response

```json
{
  "data": {
    "id": "resource-id",
    "type": "Resource",
    "attributes": { ... },
    "relationships": { ... }
  },
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z",
    "version": "1.0"
  }
}
```

### Error Response

```json
{
  "errors": [
    {
      "code": "VALIDATION_ERROR",
      "message": "Validation failed",
      "status": 422,
      "details": {
        "field": "email",
        "issue": "Invalid email format"
      }
    }
  ]
}
```

### Paginated Response

```json
{
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 1000,
    "pages": 50,
    "hasNext": true,
    "hasPrev": false
  }
}
```

## System Design Integration

### API Gateway Pattern

```
Clients
  ↓ (All requests)
API Gateway
  ├─ Authentication
  ├─ Rate Limiting
  ├─ Request Routing
  ├─ Response Caching
  └─ Logging/Monitoring
  ↓
Microservices
  ├─ User Service
  ├─ Order Service
  └─ Payment Service
```

### Internal vs External APIs

**External (Public):**
- Stable, versioned
- Limited functionality
- Rate limited
- Documentation priority
- Backward compatible

**Internal (Service-to-Service):**
- Can change frequently
- Full functionality
- No rate limits
- Performance priority
- Can use latest patterns

## Designing for Scale

### 1. Pagination (Not Offset-based for Large Sets)

**❌ Offset Pagination (O(n) scan time):**
```
GET /api/users?offset=1000000&limit=10
```

**✅ Cursor Pagination (O(1) position):**
```
GET /api/users?cursor=eyJpZCI6IDUwMH0=&limit=10
```

**✅✅ Keyset Pagination (Index-based, fastest):**
```
GET /api/users?after=2024-01-01&limit=10&sort=created_at:desc
```

### 2. Response Size Optimization

**Field Selection (Sparse Fieldsets):**
```
GET /api/users/123?fields=id,email,name
```

**Partial Response:**
```
GET /api/users/123?exclude=largeField,otherBigField
```

### 3. Caching Strategy

```
Cache-Control: public, max-age=3600
ETag: "abc123def456"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT
Vary: Accept-Encoding
```

### 4. Filtering & Search

```
GET /api/users?role=admin&status=active&created_after=2024-01-01&sort=created_at:desc
GET /api/products?category=electronics&price_min=10&price_max=100&in_stock=true
GET /api/posts?search=api+design&tags=backend,architecture&language=en
```

## Testing API Contracts

### Contract Testing Example

```javascript
describe('API Contract: Users API', () => {
  describe('GET /api/v1/users/:id', () => {
    it('returns user matching contract', async () => {
      const response = await request(app)
        .get('/api/v1/users/123');

      expect(response.status).toBe(200);
      expect(response.body).toMatchObject({
        data: {
          id: expect.any(String),
          name: expect.any(String),
          email: expect.any(String),
          created_at: expect.any(String)
        }
      });
    });

    it('returns 404 for non-existent user', async () => {
      const response = await request(app)
        .get('/api/v1/users/nonexistent');

      expect(response.status).toBe(404);
      expect(response.body).toMatchObject({
        errors: [{ code: 'NOT_FOUND' }]
      });
    });
  });
});
```

## Monitoring API Health

### Key Metrics

```
Latency P95 (95th percentile) < 500ms
Throughput: 10,000 requests/second
Error Rate: < 0.1%
Availability: 99.99%
```

### APM Integration

```javascript
// Track API performance
app.use((req, res, next) => {
  const startTime = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - startTime;
    metrics.recordEndpoint({
      method: req.method,
      path: req.route?.path,
      status: res.statusCode,
      duration
    });
  });

  next();
});
```

## Checklist: API Architecture Design

- [ ] Choose API style (REST, GraphQL, Hybrid)
- [ ] Define versioning strategy
- [ ] Design resource models
- [ ] Contract definition complete
- [ ] Error handling standardized
- [ ] Pagination strategy chosen
- [ ] Caching headers configured
- [ ] Rate limiting planned
- [ ] Monitoring configured
- [ ] Documentation tools set up
- [ ] Testing framework ready
- [ ] Security considerations (next agent)

---

**Next:** Security & Compliance with Agent 5, or Backend Patterns with Agent 2.
