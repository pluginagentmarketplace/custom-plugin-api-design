---
name: database-patterns
version: "2.0.0"
description: Database design, optimization, and caching strategies for SQL, NoSQL, and Redis
sasmp_version: "1.3.0"
bonded_agent: 03-database-performance
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "Database optimization and caching patterns"
  boundaries:
    includes: [sql_optimization, nosql_patterns, redis_caching, connection_pooling, indexing]
    excludes: [api_design, business_logic, infrastructure]

parameter_validation:
  schema:
    type: object
    properties:
      database_type:
        type: string
        enum: [postgresql, mysql, mongodb, redis]
      operation:
        type: string
        enum: [query, cache, index, pool]
      optimization_goal:
        type: string
        enum: [read_heavy, write_heavy, balanced]

retry_config:
  enabled: true
  max_attempts: 3
  backoff:
    type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 30000
  retryable_errors:
    - ECONNREFUSED
    - ETIMEDOUT
    - connection_pool_exhausted

logging:
  level: INFO
  fields: [database_type, operation, query_time_ms, cache_hit]

dependencies:
  skills: []
  agents: [03-database-performance]
---

# Database Patterns Skill

## Purpose
Optimize database operations with proven patterns for SQL, NoSQL, and caching.

## SQL Optimization

### Indexing Strategy

```sql
-- B-tree index (default, equality + range)
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);

-- Partial index (filter common queries)
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';

-- Covering index (avoid table lookup)
CREATE INDEX idx_orders_covering ON orders(user_id) INCLUDE (total, status);
```

### N+1 Query Prevention

```typescript
// BAD: N+1 queries
const users = await db.query('SELECT * FROM users');
for (const user of users) {
  user.orders = await db.query(
    'SELECT * FROM orders WHERE user_id = ?',
    [user.id]
  );
}

// GOOD: Single query with JOIN
const usersWithOrders = await db.query(`
  SELECT u.*, json_agg(o.*) as orders
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id
`);

// GOOD: DataLoader pattern
const orderLoader = new DataLoader(async (userIds) => {
  const orders = await db.query(
    'SELECT * FROM orders WHERE user_id = ANY($1)',
    [userIds]
  );
  return userIds.map(id => orders.filter(o => o.user_id === id));
});
```

### Connection Pooling

```typescript
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  max: 20,                    // Max connections
  min: 5,                     // Min connections
  idleTimeoutMillis: 30000,   // Close idle after 30s
  connectionTimeoutMillis: 5000,
  acquireTimeoutMillis: 10000,
});

// Monitor pool health
pool.on('error', (err, client) => {
  console.error('Unexpected pool error', err);
});

// Query with automatic release
async function query(sql, params) {
  const client = await pool.connect();
  try {
    return await client.query(sql, params);
  } finally {
    client.release();
  }
}
```

## NoSQL Patterns (MongoDB)

### Document Design

```javascript
// Embedding (1:few, read-heavy)
{
  _id: ObjectId("..."),
  name: "John Doe",
  addresses: [
    { type: "home", city: "NYC" },
    { type: "work", city: "Boston" }
  ]
}

// Referencing (1:many, write-heavy)
{
  _id: ObjectId("..."),
  name: "John Doe",
  order_ids: [ObjectId("..."), ObjectId("...")]
}

// Hybrid (bounded array + overflow)
{
  _id: ObjectId("..."),
  name: "John Doe",
  recent_orders: [...],  // Last 10
  has_more_orders: true
}
```

### Aggregation Pipeline

```javascript
db.orders.aggregate([
  // Match stage (uses indexes)
  { $match: { status: 'completed', created_at: { $gte: lastMonth } } },

  // Group and calculate
  { $group: {
    _id: '$user_id',
    total_spent: { $sum: '$amount' },
    order_count: { $count: {} }
  }},

  // Sort by spending
  { $sort: { total_spent: -1 } },

  // Limit to top 100
  { $limit: 100 },

  // Lookup user details
  { $lookup: {
    from: 'users',
    localField: '_id',
    foreignField: '_id',
    as: 'user'
  }}
]);
```

## Redis Caching

### Cache-Aside Pattern

```typescript
class CacheService {
  constructor(private redis: Redis, private db: Database) {}

  async get<T>(key: string, fetcher: () => Promise<T>, ttl = 3600): Promise<T> {
    // Try cache first
    const cached = await this.redis.get(key);
    if (cached) {
      return JSON.parse(cached);
    }

    // Fetch from database
    const data = await fetcher();

    // Store in cache
    await this.redis.setex(key, ttl, JSON.stringify(data));

    return data;
  }

  async invalidate(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
}

// Usage
const user = await cache.get(
  `user:${userId}`,
  () => db.users.findById(userId),
  3600
);
```

### Write-Through Pattern

```typescript
async function updateUser(userId: string, data: UserUpdate) {
  // Update database
  const user = await db.users.update(userId, data);

  // Immediately update cache
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));

  // Invalidate related caches
  await redis.del(`user:${userId}:orders`);

  return user;
}
```

### Cache Stampede Prevention

```typescript
async function getWithLock<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttl: number
): Promise<T> {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  // Acquire lock
  const lockKey = `lock:${key}`;
  const acquired = await redis.set(lockKey, '1', 'EX', 10, 'NX');

  if (!acquired) {
    // Wait and retry
    await sleep(100);
    return getWithLock(key, fetcher, ttl);
  }

  try {
    const data = await fetcher();
    await redis.setex(key, ttl, JSON.stringify(data));
    return data;
  } finally {
    await redis.del(lockKey);
  }
}
```

---

## Unit Test Template

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { Pool } from 'pg';
import Redis from 'ioredis-mock';

describe('Database Patterns', () => {
  let pool: Pool;
  let redis: Redis;

  beforeEach(() => {
    redis = new Redis();
  });

  describe('N+1 Prevention', () => {
    it('should batch queries with DataLoader', async () => {
      const queries: string[] = [];
      const loader = new DataLoader(async (ids) => {
        queries.push(`SELECT * FROM orders WHERE user_id IN (${ids})`);
        return ids.map(id => ({ user_id: id, total: 100 }));
      });

      await Promise.all([
        loader.load('1'),
        loader.load('2'),
        loader.load('3'),
      ]);

      expect(queries.length).toBe(1); // Single batched query
    });
  });

  describe('Cache-Aside Pattern', () => {
    it('should return cached value on hit', async () => {
      await redis.setex('user:123', 3600, JSON.stringify({ id: '123' }));

      let dbCalled = false;
      const user = await cache.get('user:123', async () => {
        dbCalled = true;
        return { id: '123' };
      });

      expect(dbCalled).toBe(false);
      expect(user.id).toBe('123');
    });

    it('should fetch and cache on miss', async () => {
      const user = await cache.get('user:456', async () => {
        return { id: '456', name: 'Test' };
      });

      const cached = await redis.get('user:456');
      expect(JSON.parse(cached!).name).toBe('Test');
    });
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Slow queries | Missing index | Add composite index for query pattern |
| Connection timeout | Pool exhausted | Increase pool size, add queuing |
| Cache stampede | Many concurrent misses | Use locking or early expiration |
| Stale cache | Missed invalidation | Use write-through or pub/sub |
| Memory pressure | Large cached objects | Compress or use Redis hashes |

---

## Query Optimization Checklist

```
EXPLAIN ANALYZE your_query;

Check for:
- [ ] Seq Scan (should be Index Scan)
- [ ] Nested Loop with high rows
- [ ] Sort with high memory
- [ ] Hash Join memory usage
```

---

## Quality Checklist

- [ ] Proper indexing strategy implemented
- [ ] N+1 queries eliminated
- [ ] Connection pooling configured
- [ ] Caching strategy defined
- [ ] Cache invalidation implemented
- [ ] Query plans analyzed
- [ ] Monitoring and alerting set up
