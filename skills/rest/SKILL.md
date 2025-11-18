---
name: rest-patterns
description: REST API implementation patterns including pagination, filtering, relationships, nesting, and architectural best practices
---

# REST API Patterns Skill

## Quick Start

Build scalable REST APIs by mastering:

### Resource Nesting
```
/api/users                  ✅ Collection
/api/users/123              ✅ Specific user
/api/users/123/posts        ✅ Nested collection
/api/users/123/posts/456    ✅ Nested specific
```

### Pagination
```
GET /api/posts?limit=10&offset=0
GET /api/posts?limit=10&cursor=abc123
GET /api/posts?limit=10&after=2024-01-01
```

### Filtering & Sorting
```
GET /api/posts?author_id=123&status=published
GET /api/posts?tags=javascript,typescript&sort=created_at:desc
GET /api/products?price_min=10&price_max=100&category=electronics
```

### Query Parameter Best Practices
- Use lowercase
- Use snake_case
- Provide defaults
- Document all parameters
- Validate on server

### API Maturity

**Level 0:** POX (Plain Old XML/JSON)
**Level 1:** Resources
**Level 2:** HTTP Verbs
**Level 3:** HATEOAS (Hypermedia)

## REST Response Patterns

### Single Resource
```json
{
  "id": 123,
  "name": "John",
  "email": "john@example.com"
}
```

### Collection with Pagination
```json
{
  "data": [ /* items */ ],
  "pagination": {
    "limit": 10,
    "offset": 0,
    "total": 1000
  }
}
```

### Error Response
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": []
  }
}
```

## REST Design Checklist

- [ ] Consistent resource naming
- [ ] Proper HTTP verbs used
- [ ] Relationships handled correctly
- [ ] Pagination implemented
- [ ] Filtering options available
- [ ] Sorting supported
- [ ] Error responses consistent
- [ ] Documentation complete

See the REST API Architecture agent for advanced patterns.
