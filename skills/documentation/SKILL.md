---
name: documentation
description: API documentation with OpenAPI and developer portals
sasmp_version: "2.0.0"
bonded_agent: 08-api-documentation
bond_type: PRIMARY_BOND

# Production-Grade Metadata
parameters:
  - name: doc_type
    type: string
    required: true
    validation: "^(openapi|reference|guide|changelog|sdk)$"
    description: Documentation type
  - name: format
    type: string
    required: false
    validation: "^(yaml|json|markdown)$"
    description: Output format

validation_rules:
  - All endpoints must have examples
  - Error codes must be documented
  - Authentication must be explained

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [documentation_coverage, example_count]
---

# API Documentation Skill

## Topics Covered
- OpenAPI/Swagger specs
- API reference generation
- SDK documentation
- Developer portals, code samples

## OpenAPI Structure

```yaml
openapi: 3.1.0
info:
  title: API Name
  version: 1.0.0

paths:
  /users:
    get:
      summary: List users
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'

components:
  schemas:
    User:
      type: object
      properties:
        id: { type: string }
        name: { type: string }
```

## Endpoint Documentation Template

```markdown
## Create User

Creates a new user account.

**Endpoint:** `POST /users`

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | User's name |
| email | string | Yes | Email address |

### Response

**201 Created**
```json
{ "id": "123", "name": "John" }
```
```

## SDK Generation

```typescript
// Generated TypeScript client
class UsersAPI {
  async listUsers(page = 1): Promise<User[]> {
    const res = await fetch(`${this.baseUrl}/users?page=${page}`);
    return res.json();
  }
}
```

## Changelog Format

```markdown
## [1.2.0] - 2025-01-15

### Added
- GET /users/{id}/teams endpoint
- role field to User object

### Deprecated
- GET /users/{id}/groups (use /teams instead)
```

## Unit Test Template

```javascript
describe('documentation skill', () => {
  test('OpenAPI spec is valid', async () => {
    const spec = await loadSpec('openapi.yaml');
    const result = await validateOpenAPI(spec);
    expect(result.errors).toHaveLength(0);
  });

  test('all endpoints have examples', () => {
    const endpoints = parseEndpoints(spec);
    endpoints.forEach(endpoint => {
      expect(endpoint.examples).toBeDefined();
    });
  });
});
```

## Learning Outcomes
- Write OpenAPI specs
- Generate documentation
- Create code examples

See Agent 8: API Documentation for detailed guidance.
