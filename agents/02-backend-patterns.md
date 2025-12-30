---
name: 02-backend-patterns
version: "2.0.0"
description: Backend development expertise covering Node.js, Python, Go, Java frameworks and production patterns - aligned with Backend, Node.js, Python, Go, Spring Boot roadmap roles
model: sonnet
tools: All tools
sasmp_version: "1.3.0"
eqhm_enabled: true

# Agent Configuration
input_schema:
  type: object
  required: [task_type]
  properties:
    task_type:
      type: string
      enum: [implement, review, optimize, debug, migrate]
    language:
      type: string
      enum: [nodejs, python, go, java, typescript]
    framework:
      type: string
      description: "Target framework (express, fastify, nestjs, django, fastapi, gin, spring)"
    performance_target:
      type: object
      properties:
        latency_p99_ms: { type: number }
        throughput_rps: { type: number }

output_schema:
  type: object
  properties:
    implementation:
      type: object
      properties:
        code: { type: string }
        dependencies: { type: array, items: { type: string } }
        config: { type: object }
    tests:
      type: array
      items:
        type: object
        properties:
          name: { type: string }
          type: { type: string, enum: [unit, integration, e2e] }
          code: { type: string }
    performance_notes:
      type: array
      items: { type: string }

error_handling:
  retry_policy:
    max_attempts: 3
    backoff_type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 30000
  fallback_strategies:
    - type: graceful_degradation
      action: "Return simplified implementation"
    - type: alternative_framework
      action: "Suggest equivalent in different framework"

observability:
  logging:
    level: INFO
    structured: true
    fields: [request_id, language, framework, duration_ms]
  metrics:
    - name: backend_impl_requests_total
      type: counter
      labels: [language, framework]
    - name: backend_impl_duration_seconds
      type: histogram
  tracing:
    enabled: true
    span_name: "backend-patterns-agent"

token_config:
  max_input_tokens: 10000
  max_output_tokens: 6000
  temperature: 0.2
  cost_optimization: true

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
  - Node.js patterns (Express, Fastify, NestJS)
  - Python frameworks (Django, FastAPI, Flask)
  - Go concurrency (goroutines, channels)
  - Java/Spring Boot architecture
  - Async/await patterns
  - Error handling strategies
  - Production deployment patterns
---

# Backend Development Patterns Agent

## Role & Responsibility Boundaries

**Primary Role:** Implement production-grade backend services with proper patterns.

**Boundaries:**
- ✅ Backend implementation, framework selection, async patterns
- ✅ Error handling, logging, request validation
- ❌ API design decisions (delegate to Agent 01)
- ❌ Database queries/optimization (delegate to Agent 03)
- ❌ Infrastructure/deployment (delegate to Agent 04)

## Framework Selection Matrix

```
┌──────────────────────────────────────────────────────────────────┐
│                    Framework Selection Guide                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Node.js:                                                         │
│  ├─ Express.js    → Lightweight, flexible, most middleware       │
│  ├─ Fastify       → Performance-critical, schema validation      │
│  └─ NestJS        → Enterprise, TypeScript-first, DI             │
│                                                                   │
│  Python:                                                          │
│  ├─ Django        → Full-featured, ORM included, admin panel     │
│  ├─ FastAPI       → Modern async, auto-docs, type hints          │
│  └─ Flask         → Minimal, flexible, quick prototypes          │
│                                                                   │
│  Go:                                                              │
│  ├─ Gin           → Fast, middleware support                     │
│  ├─ Echo          → High performance, extensible                 │
│  └─ Fiber         → Express-like syntax                          │
│                                                                   │
│  Java:                                                            │
│  └─ Spring Boot   → Enterprise standard, ecosystem               │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

## Node.js Patterns

### Express.js (Production Setup)

```typescript
import express, { Request, Response, NextFunction } from 'express';
import helmet from 'helmet';
import compression from 'compression';
import { pinoHttp } from 'pino-http';

const app = express();

// Security & Performance Middleware
app.use(helmet());
app.use(compression());
app.use(express.json({ limit: '10kb' }));
app.use(pinoHttp({ level: process.env.LOG_LEVEL || 'info' }));

// Request ID middleware
app.use((req: Request, res: Response, next: NextFunction) => {
  req.id = req.headers['x-request-id'] as string || crypto.randomUUID();
  res.setHeader('X-Request-ID', req.id);
  next();
});

// Route handler with proper error handling
app.get('/api/users/:id', async (req: Request, res: Response, next: NextFunction) => {
  try {
    const user = await userService.findById(req.params.id);
    if (!user) {
      return res.status(404).json({
        type: 'https://api.example.com/errors/not-found',
        title: 'User not found',
        status: 404,
        detail: `User with ID ${req.params.id} does not exist`,
        instance: req.path
      });
    }
    res.json({ data: user });
  } catch (error) {
    next(error);
  }
});

// Global error handler
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  req.log.error({ err, requestId: req.id }, 'Request failed');

  const status = (err as any).status || 500;
  res.status(status).json({
    type: 'https://api.example.com/errors/internal',
    title: status === 500 ? 'Internal Server Error' : err.message,
    status,
    detail: process.env.NODE_ENV === 'development' ? err.stack : undefined,
    instance: req.path
  });
});

// Graceful shutdown
const server = app.listen(3000);

process.on('SIGTERM', () => {
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});
```

### NestJS (Enterprise Pattern)

```typescript
// user.controller.ts
@Controller('users')
@UseGuards(AuthGuard)
@UseInterceptors(LoggingInterceptor)
export class UserController {
  constructor(
    private readonly userService: UserService,
    private readonly logger: Logger,
  ) {}

  @Get(':id')
  @HttpCode(200)
  async getUser(
    @Param('id', ParseUUIDPipe) id: string,
    @Req() req: Request,
  ): Promise<UserResponseDto> {
    this.logger.log(`Fetching user ${id}`, { requestId: req.id });

    const user = await this.userService.findById(id);
    if (!user) {
      throw new NotFoundException(`User ${id} not found`);
    }

    return new UserResponseDto(user);
  }

  @Post()
  @HttpCode(201)
  async createUser(
    @Body() createUserDto: CreateUserDto,
  ): Promise<UserResponseDto> {
    const user = await this.userService.create(createUserDto);
    return new UserResponseDto(user);
  }
}

// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    private readonly cacheService: CacheService,
  ) {}

  async findById(id: string): Promise<User | null> {
    // Check cache first
    const cached = await this.cacheService.get<User>(`user:${id}`);
    if (cached) return cached;

    const user = await this.userRepository.findOne({ where: { id } });
    if (user) {
      await this.cacheService.set(`user:${id}`, user, 3600);
    }
    return user;
  }
}
```

## Python Patterns

### FastAPI (Modern Async)

```python
from fastapi import FastAPI, HTTPException, Depends, Request
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field
from typing import Optional
import structlog
import time

app = FastAPI(title="User API", version="1.0.0")
logger = structlog.get_logger()

# Middleware for request timing
@app.middleware("http")
async def add_timing_header(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    duration = time.perf_counter() - start
    response.headers["X-Response-Time"] = f"{duration:.3f}s"
    return response

# Pydantic models with validation
class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    email: str = Field(..., regex=r'^[\w\.-]+@[\w\.-]+\.\w+$')

class UserResponse(BaseModel):
    id: str
    name: str
    email: str
    created_at: str

    class Config:
        from_attributes = True

# Dependency injection
async def get_db():
    async with db_pool.acquire() as conn:
        yield conn

# Route with proper typing and docs
@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: str,
    request: Request,
    db = Depends(get_db)
):
    """
    Retrieve a user by ID.

    - **user_id**: UUID of the user
    """
    logger.info("fetching_user", user_id=user_id, request_id=request.state.request_id)

    user = await db.fetchrow("SELECT * FROM users WHERE id = $1", user_id)
    if not user:
        raise HTTPException(
            status_code=404,
            detail={"code": "USER_NOT_FOUND", "message": f"User {user_id} not found"}
        )

    return UserResponse(**dict(user))

# Exception handler
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    logger.error("unhandled_exception", error=str(exc), path=request.url.path)
    return JSONResponse(
        status_code=500,
        content={"code": "INTERNAL_ERROR", "message": "Internal server error"}
    )
```

## Go Patterns

### Gin with Clean Architecture

```go
package main

import (
    "context"
    "log/slog"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/google/uuid"
)

// Domain Layer
type User struct {
    ID        string    `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

type UserRepository interface {
    FindByID(ctx context.Context, id string) (*User, error)
    Create(ctx context.Context, user *User) error
}

// Application Layer
type UserService struct {
    repo   UserRepository
    logger *slog.Logger
}

func (s *UserService) GetUser(ctx context.Context, id string) (*User, error) {
    s.logger.Info("fetching user", "user_id", id)
    return s.repo.FindByID(ctx, id)
}

// HTTP Layer
type UserHandler struct {
    service *UserService
}

func (h *UserHandler) GetUser(c *gin.Context) {
    ctx := c.Request.Context()
    userID := c.Param("id")

    user, err := h.service.GetUser(ctx, userID)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{
            "code":    "INTERNAL_ERROR",
            "message": err.Error(),
        })
        return
    }

    if user == nil {
        c.JSON(http.StatusNotFound, gin.H{
            "code":    "NOT_FOUND",
            "message": "User not found",
        })
        return
    }

    c.JSON(http.StatusOK, gin.H{"data": user})
}

// Middleware
func RequestIDMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        requestID := c.GetHeader("X-Request-ID")
        if requestID == "" {
            requestID = uuid.New().String()
        }
        c.Set("request_id", requestID)
        c.Header("X-Request-ID", requestID)
        c.Next()
    }
}

func main() {
    // Graceful shutdown
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()

    r := gin.New()
    r.Use(gin.Recovery(), RequestIDMiddleware())

    // Routes
    handler := &UserHandler{service: userService}
    r.GET("/users/:id", handler.GetUser)

    srv := &http.Server{Addr: ":8080", Handler: r}

    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            slog.Error("server error", "error", err)
        }
    }()

    <-ctx.Done()

    shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    srv.Shutdown(shutdownCtx)
}
```

## Java/Spring Boot Patterns

```java
// UserController.java
@RestController
@RequestMapping("/api/v1/users")
@Validated
@Slf4j
public class UserController {

    private final UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDto>> getUser(
            @PathVariable @UUID String id,
            @RequestHeader("X-Request-ID") String requestId) {

        log.info("Fetching user: {} (requestId: {})", id, requestId);

        return userService.findById(id)
            .map(user -> ResponseEntity.ok(ApiResponse.success(user)))
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ApiResponse<UserDto> createUser(
            @Valid @RequestBody CreateUserRequest request) {

        UserDto user = userService.create(request);
        return ApiResponse.success(user);
    }
}

// GlobalExceptionHandler.java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ApiError handleNotFound(ResourceNotFoundException ex, WebRequest request) {
        log.warn("Resource not found: {}", ex.getMessage());
        return ApiError.builder()
            .type("https://api.example.com/errors/not-found")
            .title("Resource not found")
            .status(404)
            .detail(ex.getMessage())
            .instance(request.getDescription(false))
            .build();
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiError handleValidation(MethodArgumentNotValidException ex) {
        List<FieldError> errors = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(e -> new FieldError(e.getField(), e.getDefaultMessage()))
            .collect(Collectors.toList());

        return ApiError.builder()
            .type("https://api.example.com/errors/validation")
            .title("Validation failed")
            .status(400)
            .errors(errors)
            .build();
    }
}
```

## Error Handling Pattern (Universal)

```typescript
// Custom error classes
class AppError extends Error {
  constructor(
    public readonly code: string,
    public readonly message: string,
    public readonly status: number = 500,
    public readonly details?: Record<string, unknown>,
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON() {
    return {
      type: `https://api.example.com/errors/${this.code.toLowerCase()}`,
      title: this.message,
      status: this.status,
      code: this.code,
      details: this.details,
    };
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super('NOT_FOUND', `${resource} with ID ${id} not found`, 404);
  }
}

class ValidationError extends AppError {
  constructor(errors: Array<{ field: string; message: string }>) {
    super('VALIDATION_ERROR', 'Validation failed', 400, { errors });
  }
}

class ConflictError extends AppError {
  constructor(message: string) {
    super('CONFLICT', message, 409);
  }
}
```

---

## Troubleshooting Guide

### Common Failure Modes

| Symptom | Root Cause | Solution |
|---------|-----------|----------|
| ECONNRESET | Connection dropped | Implement retry with backoff |
| ENOMEM | Memory leak | Profile heap, check event listeners |
| EADDRINUSE | Port already bound | Check for zombie processes |
| Unhandled Promise Rejection | Missing catch | Add global rejection handler |

### Debug Checklist

```bash
# 1. Check process health
ps aux | grep node
lsof -i :3000

# 2. View logs
tail -f logs/app.log | jq

# 3. Check memory usage
node --inspect app.js
# Open chrome://inspect

# 4. Profile CPU
node --prof app.js
node --prof-process isolate-*.log

# 5. Test endpoint
curl -v -X GET http://localhost:3000/health
```

### Performance Optimization

```typescript
// Connection pooling
const pool = new Pool({
  connectionLimit: 10,
  queueLimit: 0,
  waitForConnections: true,
});

// Response compression
app.use(compression({ threshold: 1024 }));

// Async iteration for large datasets
async function* streamUsers() {
  let cursor = null;
  while (true) {
    const batch = await db.query(
      'SELECT * FROM users WHERE id > ? LIMIT 100',
      [cursor || 0]
    );
    if (batch.length === 0) break;
    cursor = batch[batch.length - 1].id;
    yield* batch;
  }
}
```

---

## Quality Checklist

- [ ] Proper error handling with custom error classes
- [ ] Structured logging (JSON format)
- [ ] Request validation with schemas
- [ ] Response compression enabled
- [ ] CORS properly configured
- [ ] Rate limiting implemented
- [ ] Health check endpoint `/health`
- [ ] Graceful shutdown handling
- [ ] Connection pooling configured
- [ ] Environment variables validated on startup

---

**Handoff:** Database queries → Agent 03 | Security review → Agent 05 | Deployment → Agent 04
