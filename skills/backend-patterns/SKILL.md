---
name: backend-patterns
description: Production-grade backend patterns for Node.js, Python, Go, and Java/Spring frameworks
sasmp_version: "1.3.0"
bonded_agent: 02-backend-patterns
bond_type: PRIMARY_BOND
---

# Backend Patterns Skill

## Quick Start

Expert guidance for building production-ready backends across major ecosystems.

### Node.js Ecosystem
- **Express.js**: Lightweight, flexible routing
- **NestJS**: Enterprise-grade, TypeScript-first
- **Fastify**: Performance-optimized

### Python Ecosystem
- **Django**: Full-featured, batteries-included
- **FastAPI**: Modern, async, type-safe
- **Flask**: Microframework, flexible

### Go Ecosystem
- **Gorilla Mux**: HTTP routing
- **Gin**: Web framework
- Goroutines for concurrency

### Java/Spring
- **Spring Boot**: Enterprise standard
- **Spring MVC**: REST APIs
- Dependency injection

## Key Patterns

### Error Handling
```javascript
class APIError extends Error {
  constructor(message, status = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.status = status;
    this.code = code;
  }
}
```

### Async Operations
```javascript
async function fetchData(id) {
  try {
    const result = await db.query('SELECT * FROM items WHERE id = ?', [id]);
    return result;
  } catch (error) {
    logger.error('Fetch failed', error);
    throw new APIError('Data not found', 404);
  }
}
```

## Production Checklist

- [ ] Error handling implemented
- [ ] Logging configured
- [ ] Request validation in place
- [ ] Response compression enabled
- [ ] CORS properly configured
- [ ] Rate limiting implemented
- [ ] Health check endpoint
- [ ] Graceful shutdown handling

See Agent 2: Backend Patterns for detailed guidance.
