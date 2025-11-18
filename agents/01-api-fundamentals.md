---
description: Expert guidance on RESTful API principles, HTTP protocols, status codes, and foundational API design concepts
capabilities: ["HTTP methods and semantics", "Status codes and error handling", "Resource modeling", "API versioning basics", "Request/response patterns"]
---

# API Design Fundamentals

## Quick Start

Every API starts with understanding the fundamentals. This agent guides you through:

### HTTP Methods
- **GET** - Retrieve resources (idempotent, safe)
- **POST** - Create new resources
- **PUT** - Full replacement of resources
- **PATCH** - Partial updates
- **DELETE** - Remove resources
- **HEAD** - Like GET, but no body (metadata only)
- **OPTIONS** - Describe available methods

### Status Code Categories
- **2xx Success** - 200 OK, 201 Created, 204 No Content
- **3xx Redirection** - 301 Moved Permanently, 304 Not Modified
- **4xx Client Error** - 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found
- **5xx Server Error** - 500 Internal Server Error, 503 Service Unavailable

### Core Principles
1. **Resource-Oriented** - Model your API around resources, not actions
2. **Stateless** - Each request contains all info needed
3. **Cacheable** - Design responses to be cacheable when appropriate
4. **Layered** - Clients don't know if connected directly to end server
5. **Uniform Interface** - Consistent request/response format

## Key Concepts

### Resource Identification
```
/users/123          ✅ Resource
/users/123/posts    ✅ Nested resource
/users/list         ❌ Action-oriented
/getUser            ❌ RPC-style
```

### Request/Response Structure
```json
// Request
{
  "method": "POST",
  "path": "/api/v1/users",
  "body": {
    "name": "John",
    "email": "john@example.com"
  }
}

// Response
{
  "status": 201,
  "data": {
    "id": 1,
    "name": "John",
    "email": "john@example.com",
    "created_at": "2024-01-01T00:00:00Z"
  }
}
```

## When to Use Each HTTP Method

| Method | Purpose | Idempotent | Safe | Body |
|--------|---------|-----------|------|------|
| GET | Retrieve | Yes | Yes | No |
| POST | Create | No | No | Yes |
| PUT | Replace | Yes | No | Yes |
| PATCH | Update | No | No | Yes |
| DELETE | Remove | Yes | No | No |

## Common Status Codes

**Success Codes:**
- 200 OK - Request succeeded
- 201 Created - Resource created successfully
- 202 Accepted - Request accepted for processing
- 204 No Content - Success, no content to return

**Client Error Codes:**
- 400 Bad Request - Invalid request syntax
- 401 Unauthorized - Authentication required
- 403 Forbidden - Authenticated but not authorized
- 404 Not Found - Resource doesn't exist
- 409 Conflict - Request conflicts with current state
- 422 Unprocessable Entity - Validation failed

**Server Error Codes:**
- 500 Internal Server Error - Server error
- 502 Bad Gateway - Gateway error
- 503 Service Unavailable - Service temporarily down

## Content Negotiation

```
Accept: application/json
Accept: application/xml
Accept: application/hal+json
Content-Type: application/json; charset=utf-8
```

## Versioning Strategy

**URL-based:** `/api/v1/users` ✅ Clear and simple
**Header-based:** `Accept: application/vnd.api+json;version=1` More flexible
**Query-based:** `/api/users?version=1` Less common

## Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

## Design Checklist

- [ ] Clear resource naming conventions
- [ ] Consistent HTTP method usage
- [ ] Proper status codes in all responses
- [ ] Error responses with helpful messages
- [ ] API versioning strategy defined
- [ ] Content negotiation considered
- [ ] Request/response examples documented

Next: Explore advanced REST patterns with the REST API Architecture agent.
