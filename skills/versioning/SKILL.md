---
name: versioning
version: "2.0.0"
description: API versioning strategies and backward compatibility
sasmp_version: "1.3.0"
bonded_agent: 01-api-architect
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "API versioning and deprecation management"
  boundaries:
    includes: [url_versioning, header_versioning, deprecation, migration]
    excludes: [api_design, implementation]

parameter_validation:
  schema:
    type: object
    properties:
      strategy:
        type: string
        enum: [url, header, query]
      version_format:
        type: string
        pattern: "^v?[0-9]+$"

retry_config:
  enabled: false

logging:
  level: INFO
  fields: [version, strategy, deprecated]

dependencies:
  skills: [api-architecture]
  agents: [01-api-architect]
---

# API Versioning Skill

## Purpose
Choose and implement API versioning strategies.

## Versioning Strategies

| Strategy | Format | Pros | Cons |
|----------|--------|------|------|
| URL path | `/api/v1/users` | Clear, cacheable | URL pollution |
| Header | `Accept: application/vnd.api.v1+json` | Clean URLs | Hidden |
| Query param | `/api/users?version=1` | Flexible | Unconventional |

## URL Versioning (Recommended)

```yaml
# Version in path
/api/v1/users        # Current stable
/api/v2/users        # New version
/api/v3-beta/users   # Pre-release

# Version-specific routing
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);
```

## Header Versioning

```http
# Request
Accept: application/vnd.myapi.v1+json

# Response
Content-Type: application/vnd.myapi.v1+json
```

## Deprecation Policy

```yaml
# Sunset header
Sunset: Sat, 31 Dec 2025 23:59:59 GMT
Deprecation: true
Link: </api/v2/users>; rel="successor-version"

# Response body warning
{
  "data": {...},
  "warnings": [
    {
      "type": "deprecation",
      "message": "This endpoint is deprecated. Use /api/v2/users instead.",
      "sunset": "2025-12-31"
    }
  ]
}
```

## Breaking vs Non-Breaking Changes

### Non-Breaking (Safe)
- Adding new endpoints
- Adding optional fields
- Adding new enum values
- Relaxing validation

### Breaking (Requires New Version)
- Removing endpoints
- Removing/renaming fields
- Changing field types
- Tightening validation
- Changing response structure

## Migration Guide Template

```markdown
# Migration from v1 to v2

## Breaking Changes

### User endpoint
- `GET /api/v1/users/{id}` → `GET /api/v2/users/{id}`
- Response field `fullName` renamed to `name`

### Before (v1)
{
  "id": "123",
  "fullName": "John Doe"
}

### After (v2)
{
  "id": "123",
  "name": "John Doe"
}

## Timeline
- v2 available: January 2025
- v1 deprecated: March 2025
- v1 removed: December 2025
```

---

## Unit Test Template

```typescript
describe('API Versioning', () => {
  it('should route to v1 handler', async () => {
    await request(app)
      .get('/api/v1/users')
      .expect(200);
  });

  it('should include deprecation headers for old version', async () => {
    const res = await request(app)
      .get('/api/v1/users')
      .expect(200);

    expect(res.headers.deprecation).toBe('true');
    expect(res.headers.sunset).toBeDefined();
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Clients use wrong version | No redirect | Add version negotiation |
| Breaking change missed | No detection | Use schema diff tools |
| Sunset too short | Client migration time | Minimum 6 month notice |

---

## Quality Checklist

- [ ] Versioning strategy documented
- [ ] Deprecation policy defined
- [ ] Sunset headers implemented
- [ ] Migration guides published
- [ ] Version changelog maintained
