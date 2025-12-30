---
name: backend-patterns
description: Production-grade backend patterns for Node.js, Python, Go, and Java/Spring frameworks
sasmp_version: "2.0.0"
bonded_agent: 02-backend-patterns
bond_type: PRIMARY_BOND

# Production-Grade Metadata
parameters:
  - name: framework
    type: string
    required: true
    validation: "^(express|nestjs|fastify|django|fastapi|flask|gin|spring)$"
    description: Backend framework
  - name: operation
    type: string
    required: false
    validation: "^(setup|optimize|debug|migrate)$"
    description: Operation type

validation_rules:
  - All async operations must have error handling
  - Database queries must use parameterized statements
  - Middleware must have proper ordering

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [duration, success_rate, framework_usage]
---

# Backend Patterns Skill

## Quick Start

Expert guidance for building production-ready backends.

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
    return await db.query('SELECT * FROM items WHERE id = ?', [id]);
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
- [ ] Rate limiting implemented
- [ ] Health check endpoint

## Unit Test Template

```javascript
describe('backend-patterns skill', () => {
  test('handles errors gracefully', async () => {
    await expect(fetchData('invalid')).rejects.toThrow(APIError);
  });

  test('returns data for valid id', async () => {
    const result = await fetchData('123');
    expect(result).toBeDefined();
  });
});
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| ECONNREFUSED | Service down | Check service health |
| Memory leak | Unreleased resources | Profile with heap snapshots |
| Event loop blocked | Sync CPU work | Use worker threads |

See Agent 2: Backend Patterns for detailed guidance.
