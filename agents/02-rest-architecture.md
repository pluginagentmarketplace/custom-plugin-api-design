---
description: Comprehensive REST API architecture design including resource modeling, pagination, filtering, relationships, and scalability patterns
capabilities: ["Resource design", "Pagination strategies", "Filtering and sorting", "Relationship handling", "API maturity levels", "Scalable architecture"]
---

# REST API Architecture

## Richardson Maturity Model

### Level 0: Swamp of POX
```
POST /api
{
  "method": "getUser",
  "id": 123
}
```
❌ Not RESTful

### Level 1: Resources
```
GET /api/users/123
POST /api/users
PUT /api/users/123
DELETE /api/users/123
```
✅ Resource-oriented

### Level 2: HTTP Verbs
```
GET /api/users/123 → 200 OK
POST /api/users → 201 Created
PUT /api/users/123 → 200 OK
DELETE /api/users/123 → 204 No Content
```
✅ Proper HTTP semantics

### Level 3: HATEOAS
```json
{
  "id": 123,
  "name": "John",
  "links": {
    "self": { "href": "/users/123" },
    "posts": { "href": "/users/123/posts" },
    "update": { "href": "/users/123", "method": "PUT" }
  }
}
```
✅ Hypertext driven

## Resource Modeling

### Singular vs Plural
```
✅ GET /api/users       (collection)
✅ POST /api/users      (create)
✅ GET /api/users/123   (specific)
❌ GET /api/user/123    (inconsistent)
```

### Nested Resources
```
GET /api/users/123/posts              → User's posts
GET /api/users/123/posts/456          → Specific post
POST /api/users/123/posts             → Create post
PUT /api/users/123/posts/456          → Update post
DELETE /api/users/123/posts/456       → Delete post
```

### Query Parameters for Filtering
```
GET /api/posts?author_id=123&status=published&limit=10&offset=0
GET /api/posts?tags=javascript,typescript&sort=created_at:desc
GET /api/products?price_min=10&price_max=100&category=electronics
```

## Pagination Patterns

### Offset-based
```json
{
  "data": [...],
  "pagination": {
    "offset": 0,
    "limit": 10,
    "total": 1000,
    "pages": 100
  }
}
```
✅ Simple | ❌ Inefficient with deletions

### Cursor-based
```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6IDQ1Nn0=",
    "previous_cursor": "eyJpZCI6IDEyM30=",
    "has_more": true
  }
}
```
✅ Efficient | ✅ Handles deletions

### Keyset Pagination
```
GET /api/posts?after=2024-01-01&limit=10&sort_by=created_at:desc
```
✅ Very efficient | ✅ Works with large datasets

## Relationship Handling

### Embedding (Include)
```json
{
  "id": 123,
  "title": "Post Title",
  "author": {
    "id": 1,
    "name": "John"
  }
}
```

### Linking (HATEOAS)
```json
{
  "id": 123,
  "title": "Post Title",
  "author_id": 1,
  "_links": {
    "author": { "href": "/api/users/1" }
  }
}
```

### Selective Fields (Sparse Fieldsets)
```
GET /api/posts/123?fields=id,title,author_id
```

## URL Structure Best Practices

```
/api/v1/                              # Base path with version
/api/v1/users                         # Resource collection
/api/v1/users/123                     # Resource instance
/api/v1/users/123/posts               # Related resource
/api/v1/users/123/posts/456           # Related resource instance
/api/v1/users/123/relationships/posts # Relationship endpoint (JSON:API)
```

## Response Envelope

### With Data
```json
{
  "success": true,
  "data": { /* resource */ },
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

### Collection
```json
{
  "success": true,
  "data": [ /* resources */ ],
  "pagination": { /* pagination */ },
  "meta": { /* metadata */ }
}
```

## Caching Headers

```
Cache-Control: public, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT
Expires: Thu, 22 Oct 2024 07:28:00 GMT
Vary: Accept-Encoding
```

## CORS Configuration

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
Access-Control-Allow-Credentials: true
```

## API Architecture Patterns

### Monolithic
- Single codebase
- ✅ Simpler
- ❌ Harder to scale

### Microservices
- Multiple services
- ✅ Scalable
- ❌ Complex

### API Gateway Pattern
```
Client → API Gateway → Service 1
                     → Service 2
                     → Service 3
```
✅ Single entry point
✅ Cross-cutting concerns

## Design Checklist

- [ ] Clear resource hierarchy
- [ ] Consistent naming conventions
- [ ] Efficient pagination strategy
- [ ] Flexible filtering options
- [ ] Proper relationship handling
- [ ] Caching headers configured
- [ ] CORS properly configured
- [ ] Rate limiting implemented
- [ ] Documentation complete

Next: Explore GraphQL for alternative API design with the GraphQL Design agent.
