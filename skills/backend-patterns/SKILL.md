---
name: backend-patterns
version: "2.0.0"
description: Production-grade backend patterns for Node.js, Python, Go, and Java/Spring frameworks
sasmp_version: "1.3.0"
bonded_agent: 02-backend-patterns
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "Backend implementation patterns and frameworks"
  boundaries:
    includes: [nodejs, python, go, java, error_handling, middleware, async]
    excludes: [api_design, database_queries, infrastructure]

parameter_validation:
  schema:
    type: object
    properties:
      language:
        type: string
        enum: [nodejs, python, go, java]
      framework:
        type: string
      pattern:
        type: string
        enum: [middleware, error_handling, validation, logging]

retry_config:
  enabled: true
  max_attempts: 3
  backoff:
    type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 30000

logging:
  level: INFO
  fields: [language, framework, pattern, duration_ms]

dependencies:
  skills: []
  agents: [02-backend-patterns]
---

# Backend Patterns Skill

## Purpose
Implement production-ready backend services with proper patterns.

## Framework Selection

| Language | Framework | Best For |
|----------|-----------|----------|
| Node.js | Express | Simple APIs, quick prototypes |
| Node.js | NestJS | Enterprise, TypeScript |
| Node.js | Fastify | High performance |
| Python | FastAPI | Modern async, auto-docs |
| Python | Django | Full-featured, admin |
| Go | Gin | Fast, middleware |
| Java | Spring Boot | Enterprise, ecosystem |

## Error Handling Pattern

```typescript
// Error hierarchy
class AppError extends Error {
  constructor(
    public code: string,
    message: string,
    public status: number = 500,
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super('NOT_FOUND', `${resource} ${id} not found`, 404);
  }
}

class ValidationError extends AppError {
  constructor(public errors: { field: string; message: string }[]) {
    super('VALIDATION_ERROR', 'Validation failed', 400);
  }
}

// Global handler
app.use((err, req, res, next) => {
  if (err instanceof AppError) {
    return res.status(err.status).json({
      type: `https://api.example.com/errors/${err.code.toLowerCase()}`,
      title: err.message,
      status: err.status,
    });
  }
  logger.error(err);
  res.status(500).json({ title: 'Internal error', status: 500 });
});
```

## Middleware Pattern

```typescript
// Request ID middleware
const requestId = (req, res, next) => {
  req.id = req.headers['x-request-id'] || crypto.randomUUID();
  res.setHeader('X-Request-ID', req.id);
  next();
};

// Logging middleware
const requestLogger = (req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    logger.info({
      method: req.method,
      path: req.path,
      status: res.statusCode,
      duration: Date.now() - start,
      requestId: req.id,
    });
  });
  next();
};

// Error boundary
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};
```

## Validation Pattern

```typescript
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  password: z.string().min(12),
});

function validate(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      throw new ValidationError(
        result.error.issues.map(i => ({
          field: i.path.join('.'),
          message: i.message,
        }))
      );
    }
    req.body = result.data;
    next();
  };
}

app.post('/users', validate(CreateUserSchema), createUser);
```

## Graceful Shutdown

```typescript
const server = app.listen(3000);

async function shutdown() {
  console.log('Shutting down...');

  // Stop accepting new connections
  server.close();

  // Close database connections
  await db.destroy();

  // Close Redis
  await redis.quit();

  process.exit(0);
}

process.on('SIGTERM', shutdown);
process.on('SIGINT', shutdown);
```

---

## Unit Test Template

```typescript
describe('Backend Pattern: Error Handling', () => {
  it('should return proper error response', async () => {
    const res = await request(app)
      .get('/users/invalid-id')
      .expect(404);

    expect(res.body).toMatchObject({
      type: expect.stringContaining('not-found'),
      status: 404,
    });
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Memory leak | Event listeners | Remove on cleanup |
| Connection timeout | Pool exhausted | Increase pool size |
| Unhandled rejection | Missing catch | Add async handler |

---

## Quality Checklist

- [ ] Error handling standardized
- [ ] Request logging enabled
- [ ] Input validation implemented
- [ ] Graceful shutdown configured
- [ ] Health check endpoint
