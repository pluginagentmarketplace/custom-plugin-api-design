---
name: api-performance
description: API performance optimization techniques including caching strategies, database optimization, compression, and monitoring
---

# API Performance Skill

## Quick Start

Optimize API performance by mastering:

### Response Time Targets
- **Fast:** < 100ms
- **Good:** 100-500ms
- **Acceptable:** 500ms-1s
- **Slow:** > 1s

### Caching Strategies

#### Client-side Caching
```
Cache-Control: public, max-age=3600
ETag: "abc123"
```

#### Server-side Caching (Redis)
```javascript
const cached = await redis.get(`user:${id}`);
if (cached) return JSON.parse(cached);

const user = await db.query(...);
await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
```

#### Cache Invalidation
```
Time-based: Expire after 1 hour
Event-based: Invalidate on update
Manual: Clear on deployment
```

### Database Optimization

#### Indexing
```sql
CREATE INDEX idx_user_email ON users(email);
```

#### N+1 Query Prevention
```sql
-- ❌ N+1
SELECT * FROM users;
-- Then for each: SELECT * FROM posts WHERE user_id = ?

-- ✅ Join
SELECT u.*, p.* FROM users u
LEFT JOIN posts p ON u.id = p.user_id;
```

#### Connection Pooling
```javascript
const pool = createPool({ max: 10 });
```

### Pagination
```
Cursor-based: O(1) performance
Keyset pagination: Most efficient
```

### Compression
```
gzip: 75% reduction
Brotli: 80% reduction
```

### Monitoring
- Response time histogram
- Request count per endpoint
- Error rate
- Resource utilization

## Performance Checklist

- [ ] Response times measured
- [ ] Caching implemented
- [ ] Database optimized
- [ ] N+1 queries eliminated
- [ ] Compression enabled
- [ ] Pagination optimized
- [ ] Monitoring in place
- [ ] Load testing done

See the API Performance & Optimization agent for detailed optimization techniques.
