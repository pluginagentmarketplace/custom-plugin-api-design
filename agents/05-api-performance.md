---
description: API performance optimization including caching strategies, pagination, compression, monitoring, and scaling patterns
capabilities: ["Caching strategies", "Compression", "Pagination optimization", "Database optimization", "Monitoring", "Load testing", "Scaling patterns"]
---

# API Performance & Optimization

## Performance Metrics

### Response Time Targets
- **Fast:** < 100ms
- **Good:** 100-500ms
- **Acceptable:** 500ms-1s
- **Slow:** > 1s

### Key Metrics
- **TTFB** - Time to first byte
- **P95/P99** - 95th/99th percentile latency
- **Throughput** - Requests per second
- **Error Rate** - % of failed requests
- **CPU/Memory** - Resource utilization

## Caching Strategies

### 1. Client-Side Caching

#### HTTP Cache Headers
```
Cache-Control: public, max-age=3600
ETag: "abc123def456"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT
```

#### Browser Implementation
```javascript
// Automatic with headers
// But also manual caching
const cache = new Map();

async function fetchUser(id) {
  if (cache.has(id)) {
    return cache.get(id);
  }

  const response = await fetch(`/api/users/${id}`);
  const user = await response.json();
  cache.set(id, user);
  return user;
}
```

### 2. Server-Side Caching

#### In-Memory Cache (Redis)
```javascript
const redis = require('redis');
const client = redis.createClient();

async function getUser(id) {
  // Try cache first
  const cached = await client.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  // Fetch from database
  const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);

  // Cache for 1 hour
  await client.setex(`user:${id}`, 3600, JSON.stringify(user));

  return user;
}
```

#### Cache Invalidation
```javascript
// Invalidate on update
app.put('/users/:id', async (req, res) => {
  const user = await db.updateUser(req.params.id, req.body);

  // Invalidate cache
  await client.del(`user:${req.params.id}`);

  res.json(user);
});

// Time-based invalidation
setInterval(() => {
  client.del('expensive:query:key');
}, 3600000); // Every hour
```

#### Cache Warming
```javascript
async function warmCache() {
  const users = await db.query('SELECT id FROM users');

  for (const user of users) {
    const userData = await db.query('SELECT * FROM users WHERE id = ?', [user.id]);
    await client.setex(`user:${user.id}`, 3600, JSON.stringify(userData));
  }
}
```

### 3. Database Query Caching
```javascript
// ❌ Without caching
SELECT * FROM orders WHERE user_id = 123 -- Runs every time

// ✅ With caching
const cacheKey = `user:123:orders`;
let orders = await cache.get(cacheKey);

if (!orders) {
  orders = await db.query('SELECT * FROM orders WHERE user_id = 123');
  await cache.set(cacheKey, orders, 3600);
}
```

## Pagination

### Offset-based (Simple but slow)
```
GET /api/posts?limit=10&offset=0
-- O(offset) scan time
-- Inefficient with large offsets
```

### Cursor-based (Fast)
```
GET /api/posts?limit=10&cursor=next_page_token_here
-- O(1) to start at cursor position
-- Much faster with large datasets
```

### Keyset Pagination (Optimal)
```
GET /api/posts?limit=10&after=2024-01-01&sort=created_at:desc
-- Uses index efficiently
-- No offset calculation
```

## Compression

### Response Compression
```
Accept-Encoding: gzip, deflate, br
Content-Encoding: gzip
Content-Length: 1234
```

### Implementation
```javascript
const compression = require('compression');
app.use(compression());
```

### Compression Comparison
- **None:** 100KB response
- **Gzip:** 25KB (75% reduction)
- **Brotli:** 20KB (80% reduction)

## Database Optimization

### 1. Indexing
```sql
-- ✅ Fast
CREATE INDEX idx_user_email ON users(email);
SELECT * FROM users WHERE email = 'john@example.com';

-- ❌ Slow
SELECT * FROM users WHERE UPPER(email) = 'JOHN@EXAMPLE.COM';
```

### 2. Query Optimization
```sql
-- ❌ N+1 queries
SELECT * FROM users;
-- Then for each user:
SELECT * FROM posts WHERE user_id = ?;

-- ✅ Join
SELECT u.*, p.*
FROM users u
LEFT JOIN posts p ON u.id = p.user_id;
```

### 3. Connection Pooling
```javascript
const pool = mysql.createPool({
  connectionLimit: 10,
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'myapp'
});

// Reuse connections
pool.query('SELECT * FROM users', callback);
```

## Load Testing

### Tools
- Apache JMeter
- Locust
- k6
- Gatling

### Example with k6
```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 100,
  duration: '30s',
};

export default function() {
  const response = http.get('https://api.example.com/users');
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
}
```

## Monitoring & Observability

### Metrics to Track
```javascript
// Response time histogram
histogram('http.request.duration', duration, {
  tags: { endpoint: '/api/users', method: 'GET' }
});

// Request counter
counter('http.requests', 1, {
  tags: { status: response.status }
});

// Custom metrics
gauge('active.connections', activeConnections);
```

### APM Tools
- New Relic
- Datadog
- Elastic APM
- Prometheus + Grafana

## Scaling Patterns

### Vertical Scaling
- More CPU/RAM
- ✅ Simple
- ❌ Limited by hardware

### Horizontal Scaling
- Multiple instances
- ✅ Unlimited scaling
- ❌ Complexity

### Load Balancing
```
       Client Requests
              ↓
        Load Balancer
         /  |  |  \
    Server1 Server2 Server3 Server4
```

### Database Sharding
```
User 1-1000  → Shard 1
User 1001-2000 → Shard 2
User 2001-3000 → Shard 3
```

## Performance Checklist

- [ ] Response times measured and acceptable
- [ ] Caching strategy implemented
- [ ] HTTP cache headers configured
- [ ] Pagination optimized
- [ ] Database indexes created
- [ ] N+1 queries eliminated
- [ ] Gzip/Brotli compression enabled
- [ ] Connection pooling used
- [ ] Load testing performed
- [ ] Monitoring in place
- [ ] Scaling strategy defined
- [ ] CDN configured for static assets

Next: Document and test your API with the API Documentation & Testing agent.
