---
name: rest
description: RESTful API design principles and best practices
sasmp_version: "2.0.0"
bonded_agent: 01-api-architect
bond_type: SECONDARY_BOND

# Production-Grade Metadata
parameters:
  - name: maturity_level
    type: integer
    required: false
    validation: "^[0-3]$"
    description: Richardson Maturity Model level (0-3)
  - name: resource
    type: string
    required: true
    description: Resource name to design

validation_rules:
  - URLs must use nouns, not verbs
  - HTTP methods must match CRUD operations
  - Status codes must be semantically correct

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [endpoint_usage, status_code_distribution]
---

# REST API Design Skill

## Topics Covered
- Resource naming, HTTP methods, status codes
- HATEOAS, pagination, filtering
- Request/response design, error handling

## HTTP Methods

| Method | Action | Idempotent |
|--------|--------|------------|
| GET | Read | Yes |
| POST | Create | No |
| PUT | Replace | Yes |
| PATCH | Update | Yes |
| DELETE | Delete | Yes |

## Status Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET/PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Malformed request |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable | Validation failed |

## URL Patterns

```http
GET /api/v1/users              # List
POST /api/v1/users             # Create
GET /api/v1/users/{id}         # Read
PUT /api/v1/users/{id}         # Replace
DELETE /api/v1/users/{id}      # Delete
GET /api/v1/users/{id}/orders  # Nested resource
```

## Unit Test Template

```javascript
describe('rest skill', () => {
  test('GET returns 200 with data', async () => {
    const res = await request(app).get('/api/v1/users');
    expect(res.status).toBe(200);
    expect(res.body.data).toBeDefined();
  });

  test('POST returns 201 on creation', async () => {
    const res = await request(app)
      .post('/api/v1/users')
      .send({ name: 'Test', email: 'test@example.com' });
    expect(res.status).toBe(201);
  });
});
```

## Learning Outcomes
- Design RESTful endpoints
- Handle errors properly
- Implement pagination

See Agent 1: API Architect for detailed guidance.
