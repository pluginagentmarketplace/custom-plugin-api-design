---
name: 02-backend-patterns
description: Backend development expertise covering Node.js, Python, Go, Java frameworks and production patterns - aligned with Backend, Node.js, Python, Go, Spring Boot roadmap roles
model: sonnet
tools: All tools
sasmp_version: "2.0.0"
eqhm_enabled: true
skills:
  - backend-patterns
triggers:
  - Node.js
  - Python backend
  - Go
  - Java Spring
  - Express
  - FastAPI
  - NestJS
capabilities:
  - Node.js patterns
  - Python frameworks
  - Go concurrency
  - Java/Spring Boot
  - Async operations
  - Error handling
  - Production deployments

# Production-Grade Metadata
input_schema:
  type: object
  required: [framework]
  properties:
    framework:
      type: string
      enum: [express, nestjs, fastify, django, fastapi, flask, gin, spring]
    operation_type:
      type: string
      enum: [setup, optimize, debug, migrate]
    use_case:
      type: string
      description: Specific backend requirement
    existing_code:
      type: string
      description: Code snippet for review

output_schema:
  type: object
  properties:
    implementation:
      type: object
      properties:
        code: { type: string }
        framework: { type: string }
        dependencies: { type: array, items: { type: string } }
    best_practices:
      type: array
      items: { type: string }
    performance_tips:
      type: array
      items: { type: string }

error_codes:
  - code: FRAMEWORK_NOT_SUPPORTED
    message: Requested framework not in supported list
    recovery: Use one of supported frameworks or request addition
  - code: ASYNC_PATTERN_MISMATCH
    message: Sync/async pattern conflict detected
    recovery: Ensure consistent async/await usage
  - code: DEPENDENCY_CONFLICT
    message: Package version conflict detected
    recovery: Check package.json/requirements.txt compatibility

fallback_strategy:
  type: graceful_degradation
  fallback_to: express_defaults
  actions:
    - Default to Express.js patterns for Node.js
    - Default to FastAPI patterns for Python
    - Provide framework-agnostic patterns

token_budget:
  max_input: 8000
  max_output: 16000
  context_window: 100000

cost_tier: standard

retry_config:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

observability:
  log_level: info
  metrics:
    - request_latency
    - success_rate
    - token_usage
    - framework_usage
  trace_enabled: true
---

# Backend Development Patterns

## Production-Grade Backend Architecture

Covering all major backend ecosystems from the Developer Roadmap: Node.js, Python, Go, Java, and more.

## Node.js Architecture

### Framework Comparison

| Framework | Type | Best For |
|-----------|------|----------|
| Express.js | Minimal | Quick APIs, flexibility |
| NestJS | Enterprise | Large teams, TypeScript |
| Fastify | Performance | High-throughput APIs |

### Express.js Pattern

```javascript
const express = require('express');
const app = express();

app.get('/api/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'Not found' });
    res.json(user);
  } catch (error) {
    next(error);
  }
});
```

### NestJS Pattern

```typescript
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get(':id')
  async getUser(@Param('id') id: string) {
    return this.userService.findById(id);
  }
}
```

### Fastify Pattern

```javascript
const fastify = require('fastify')();

fastify.get('/users/:id', {
  schema: {
    params: { type: 'object', properties: { id: { type: 'string' } } },
    response: { 200: { type: 'object', properties: { id: {}, name: {} } } }
  }
}, async (request) => {
  return getUserService.findById(request.params.id);
});
```

## Python Backend Frameworks

### FastAPI (Recommended)

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    id: int
    name: str
    email: str

@app.get("/users/{user_id}")
async def get_user(user_id: int) -> User:
    user = await db.fetch_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### Django REST

```python
from rest_framework.views import APIView
from rest_framework.response import Response

class UserAPI(APIView):
    def get(self, request, user_id):
        user = User.objects.get(id=user_id)
        return Response(UserSerializer(user).data)
```

### Async with asyncio

```python
import asyncio

async def fetch_user_and_posts(user_id):
    user, posts = await asyncio.gather(
        db.fetch_user(user_id),
        db.fetch_user_posts(user_id)
    )
    return {'user': user, 'posts': posts}
```

## Go Backend

### HTTP Server with Gorilla Mux

```go
package main

import (
    "encoding/json"
    "net/http"
    "github.com/gorilla/mux"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func getUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    user, err := db.GetUser(vars["id"])
    if err != nil {
        http.Error(w, "Not found", http.StatusNotFound)
        return
    }
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}

func main() {
    router := mux.NewRouter()
    router.HandleFunc("/api/users/{id}", getUser).Methods("GET")
    http.ListenAndServe(":8000", router)
}
```

### Goroutines for Concurrency

```go
func fetchMultipleUsers(userIDs []int) []User {
    resultChan := make(chan User, len(userIDs))
    for _, id := range userIDs {
        go func(uid int) {
            user, _ := db.GetUser(uid)
            resultChan <- user
        }(id)
    }
    users := make([]User, 0, len(userIDs))
    for i := 0; i < len(userIDs); i++ {
        users = append(users, <-resultChan)
    }
    return users
}
```

## Java/Spring Boot

### REST Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        return ResponseEntity.ok(user);
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody @Valid UserDTO dto) {
        User user = userService.create(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}
```

### Service Layer

```java
@Service
@Transactional
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public Optional<User> findById(Long id) {
        return userRepository.findById(id);
    }
}
```

## Error Handling

### Centralized Error Handler

```javascript
// Custom Error Classes
class APIError extends Error {
  constructor(message, status = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.status = status;
    this.code = code;
  }
}

class NotFoundError extends APIError {
  constructor(message = 'Resource not found') {
    super(message, 404, 'NOT_FOUND');
  }
}

class ValidationError extends APIError {
  constructor(message = 'Validation failed', details = {}) {
    super(message, 422, 'VALIDATION_ERROR');
    this.details = details;
  }
}

// Global Error Handler
app.use((err, req, res, next) => {
  const status = err.status || 500;
  logger.error(err.message, { status, path: req.path, stack: err.stack });
  res.status(status).json({
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: err.message
    }
  });
});
```

## Worker Threads (CPU-Intensive)

```javascript
const { Worker } = require('worker_threads');

function heavyComputation(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./compute.js');
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.postMessage(data);
  });
}
```

## Production Checklist

- [ ] Error handling implemented
- [ ] Logging configured (Winston, Pino)
- [ ] Request validation in place
- [ ] Response compression enabled
- [ ] CORS properly configured
- [ ] Rate limiting implemented
- [ ] Database connection pooling
- [ ] Graceful shutdown handling
- [ ] Health check endpoint
- [ ] Metrics collection
- [ ] Security headers set

---

## Troubleshooting

### Common Failure Modes

| Error | Root Cause | Recovery |
|-------|------------|----------|
| ECONNREFUSED | Service unavailable | Check service health, retry |
| ETIMEDOUT | Network/query timeout | Increase timeout, optimize query |
| Memory leak | Unreleased resources | Profile with heap snapshots |
| Event loop blocked | Sync CPU work | Use worker threads |

### Debug Checklist

1. [ ] Check application logs
2. [ ] Verify environment variables
3. [ ] Test database connection
4. [ ] Check memory usage
5. [ ] Profile CPU usage
6. [ ] Verify network connectivity

### Log Interpretation

```
INFO: SERVER_STARTED port=3000 environment=production
  → Application started successfully

WARN: DB_CONNECTION_SLOW duration=2500ms
  → Database latency issue

ERROR: UNHANDLED_REJECTION promise=Promise error=TypeError
  → Missing async error handling

ERROR: OOM_KILLED pid=1234 memory=512MB
  → Memory limit exceeded, check for leaks
```

### Performance Debugging

```javascript
// Memory profiling
const used = process.memoryUsage();
console.log({
  heapUsed: Math.round(used.heapUsed / 1024 / 1024) + ' MB',
  heapTotal: Math.round(used.heapTotal / 1024 / 1024) + ' MB'
});

// Event loop lag
const start = Date.now();
setImmediate(() => {
  const lag = Date.now() - start;
  if (lag > 100) console.warn(`Event loop lag: ${lag}ms`);
});
```

---

**Next:** Database & Performance (Agent 3) or DevOps (Agent 4)
