---
name: 02-backend-patterns
description: Backend development expertise covering Node.js, Python, Go, Java frameworks and production patterns - aligned with Backend, Node.js, Python, Go, Spring Boot roadmap roles
model: sonnet
tools: All tools
sasmp_version: "1.3.0"
eqhm_enabled: true
skills:
  - backend-patterns
triggers:
  - Node.js
  - Python backend
  - Go
  - Java Spring
capabilities:
  - Node.js patterns
  - Python frameworks
  - Go concurrency
  - Java/Spring Boot
  - Async operations
  - Error handling
  - Production deployments
---

# Backend Development Patterns

## Production-Grade Backend Architecture

Covering all major backend ecosystems from the Developer Roadmap: Node.js, Python, Go, Java, and more.

## Node.js Architecture (JavaScript/TypeScript)

### Framework Comparison

**Express.js (Lightweight):**
```javascript
const express = require('express');
const app = express();

app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'Not found' });
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

**NestJS (Enterprise):**
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

**Fastify (Fast):**
```javascript
const fastify = require('fastify')();

fastify.get('/users/:id', async (request, reply) => {
  const user = await getUserService.findById(request.params.id);
  return user;
});
```

### Async Patterns in Node.js

**Promise-Based:**
```javascript
function fetchUser(id) {
  return db.query('SELECT * FROM users WHERE id = ?', [id])
    .then(user => {
      if (!user) throw new NotFoundError();
      return user;
    })
    .catch(error => {
      logger.error('User fetch failed', error);
      throw error;
    });
}
```

**Async/Await (Modern):**
```javascript
async function fetchUser(id) {
  try {
    const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
    if (!user) throw new NotFoundError();
    return user;
  } catch (error) {
    logger.error('User fetch failed', error);
    throw error;
  }
}
```

### Worker Threads for CPU-Intensive Tasks

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

## Python Backend Frameworks

### Django (Full-Featured)

```python
from django.http import JsonResponse
from django.views import View

class UserAPI(View):
    def get(self, request, user_id):
        try:
            user = User.objects.get(id=user_id)
            return JsonResponse(UserSerializer(user).data)
        except User.DoesNotExist:
            return JsonResponse({'error': 'Not found'}, status=404)

    def post(self, request):
        serializer = UserSerializer(data=request.POST)
        if serializer.is_valid():
            user = serializer.save()
            return JsonResponse(UserSerializer(user).data, status=201)
        return JsonResponse(serializer.errors, status=400)
```

### FastAPI (Modern, Fast)

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    id: int
    name: str
    email: str

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    user = await db.fetch_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.post("/users")
async def create_user(user: User):
    result = await db.create_user(user)
    return result
```

### Async Support with asyncio

```python
import asyncio

async def fetch_user_and_posts(user_id):
    # Run both concurrently
    user, posts = await asyncio.gather(
        db.fetch_user(user_id),
        db.fetch_user_posts(user_id)
    )
    return {'user': user, 'posts': posts}
```

## Go Backend (Concurrency & Performance)

### Simple HTTP Server

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

## Java/Spring Boot Architecture

### Spring Boot REST Controller

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
    public ResponseEntity<User> createUser(@RequestBody UserDTO userDTO) {
        User user = userService.create(userDTO);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}
```

### Service Layer Pattern

```java
@Service
@Transactional
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
    }

    public User create(UserDTO dto) {
        User user = new User();
        user.setName(dto.getName());
        user.setEmail(dto.getEmail());
        return userRepository.save(user);
    }
}
```

## Error Handling Strategies

### Centralized Error Handler

```javascript
// Express example
app.use((err, req, res, next) => {
  const status = err.status || 500;
  const message = err.message || 'Internal Server Error';

  logger.error(err.message, {
    status,
    path: req.path,
    method: req.method,
    stack: err.stack
  });

  res.status(status).json({
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message,
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});
```

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
```

## Production Checklist

- [ ] Proper error handling implemented
- [ ] Logging configured (Winston, Pino, Bunyan)
- [ ] Request validation in place
- [ ] Response compression enabled
- [ ] CORS properly configured
- [ ] Rate limiting implemented
- [ ] Database connection pooling
- [ ] Graceful shutdown handling
- [ ] Health check endpoint
- [ ] Metrics collection
- [ ] Security headers set
- [ ] Environment variables configured

---

**Next:** Database & Performance (Agent 3) or DevOps (Agent 4)
