# Backend Development Complete Guide

Production patterns for Node.js, Python, Go, and Java backends.

## Framework Comparison

| Framework | Language | Use Case | Performance |
|-----------|----------|----------|-------------|
| Express.js | Node.js | Simple APIs | Good |
| NestJS | Node.js | Enterprise | Good |
| Fastify | Node.js | High perf | Excellent |
| Django | Python | Full-stack | Good |
| FastAPI | Python | Modern APIs | Excellent |
| Gin | Go | High perf | Excellent |
| Spring Boot | Java | Enterprise | Good |

## Node.js Patterns

### Express.js Setup
```javascript
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const compression = require('compression');

const app = express();

// Middleware stack
app.use(helmet());
app.use(cors());
app.use(compression());
app.use(express.json({ limit: '10mb' }));

// Routes
app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'Not found' });
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Error handler (last middleware)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: err.message
    }
  });
});
```

### NestJS Enterprise Pattern
```typescript
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get(':id')
  @UseGuards(AuthGuard)
  async getUser(@Param('id') id: string): Promise<User> {
    return this.userService.findById(id);
  }

  @Post()
  @UsePipes(new ValidationPipe())
  async createUser(@Body() dto: CreateUserDto): Promise<User> {
    return this.userService.create(dto);
  }
}
```

## Python Patterns

### FastAPI Modern API
```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

class User(BaseModel):
    id: int
    name: str
    email: str

@app.get("/users/{user_id}", response_model=User)
async def get_user(user_id: int):
    user = await db.fetch_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.post("/users", response_model=User, status_code=201)
async def create_user(user: User):
    return await db.create_user(user)
```

### Async Operations
```python
import asyncio

async def fetch_user_data(user_id: int):
    # Run multiple fetches concurrently
    user, posts, comments = await asyncio.gather(
        db.fetch_user(user_id),
        db.fetch_user_posts(user_id),
        db.fetch_user_comments(user_id)
    )
    return {
        "user": user,
        "posts": posts,
        "comments": comments
    }
```

## Go Patterns

### HTTP Server
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
    userID := vars["id"]

    user, err := db.GetUser(userID)
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

    users := make([]User, 0)
    for i := 0; i < len(userIDs); i++ {
        users = append(users, <-resultChan)
    }
    return users
}
```

## Error Handling Patterns

### Custom Error Classes
```javascript
class APIError extends Error {
  constructor(message, status = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.status = status;
    this.code = code;
  }
}

class NotFoundError extends APIError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class ValidationError extends APIError {
  constructor(message, details = {}) {
    super(message, 422, 'VALIDATION_ERROR');
    this.details = details;
  }
}
```

### Centralized Error Handler
```javascript
app.use((err, req, res, next) => {
  const status = err.status || 500;
  const code = err.code || 'INTERNAL_ERROR';

  logger.error({
    message: err.message,
    status,
    code,
    stack: err.stack,
    requestId: req.id
  });

  res.status(status).json({
    error: {
      code,
      message: err.message,
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});
```

## Database Patterns

### Connection Pooling
```javascript
const pool = mysql.createPool({
  connectionLimit: 10,
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  waitForConnections: true,
  queueLimit: 0
});
```

### Transaction Handling
```javascript
async function transferMoney(fromId, toId, amount) {
  const connection = await pool.getConnection();
  try {
    await connection.beginTransaction();

    await connection.query(
      'UPDATE accounts SET balance = balance - ? WHERE id = ?',
      [amount, fromId]
    );

    await connection.query(
      'UPDATE accounts SET balance = balance + ? WHERE id = ?',
      [amount, toId]
    );

    await connection.commit();
  } catch (error) {
    await connection.rollback();
    throw error;
  } finally {
    connection.release();
  }
}
```

## Production Checklist

### Before Deployment
- [ ] Environment variables configured
- [ ] Database connection pooling set up
- [ ] Error handling comprehensive
- [ ] Logging configured (structured JSON)
- [ ] Health check endpoints working
- [ ] Graceful shutdown implemented
- [ ] Rate limiting enabled
- [ ] CORS properly configured
- [ ] Security headers set (Helmet)
- [ ] Request validation in place
- [ ] Response compression enabled

### Monitoring
- [ ] APM integration (DataDog, New Relic)
- [ ] Log aggregation (ELK, CloudWatch)
- [ ] Alerting configured
- [ ] Metrics collection (Prometheus)

## Resources

- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Go Web Examples](https://gowebexamples.com/)
- [Spring Boot Guides](https://spring.io/guides)
