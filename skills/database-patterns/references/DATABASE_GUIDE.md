# Database Performance Complete Guide

Optimization patterns for SQL, NoSQL, and caching.

## Indexing Strategy

### Index Types
| Type | Use Case | Example |
|------|----------|---------|
| B-Tree | Range queries, sorting | `created_at > '2024-01-01'` |
| Hash | Equality lookups | `id = 123` |
| GIN | Full-text search, arrays | `tags @> '{javascript}'` |
| GiST | Geometric, range types | `location <-> point(0,0)` |

### Composite Index Design
```sql
-- Query pattern
SELECT * FROM orders WHERE user_id = ? AND status = ? ORDER BY created_at DESC;

-- Optimal index (left-to-right rule)
CREATE INDEX idx_orders_user_status_created
ON orders(user_id, status, created_at DESC);
```

### Covering Index
```sql
-- Avoid table lookup for common queries
CREATE INDEX idx_users_email_covering
ON users(email)
INCLUDE (id, name, status);

-- Query uses only index
SELECT id, name, status FROM users WHERE email = 'user@example.com';
```

## Query Optimization

### EXPLAIN ANALYZE
```sql
EXPLAIN ANALYZE
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.status = 'active'
GROUP BY u.id;

-- Look for:
-- - Seq Scan (bad for large tables)
-- - Index Scan (good)
-- - Index Only Scan (best)
-- - Nested Loop with high rows (potential N+1)
```

### N+1 Prevention
```javascript
// BAD: N+1 queries (1 + N database calls)
const users = await db.query('SELECT * FROM users LIMIT 10');
for (const user of users) {
  user.orders = await db.query(
    'SELECT * FROM orders WHERE user_id = ?',
    [user.id]
  );
}

// GOOD: Two queries with IN clause
const users = await db.query('SELECT * FROM users LIMIT 10');
const userIds = users.map(u => u.id);
const orders = await db.query(
  'SELECT * FROM orders WHERE user_id = ANY($1)',
  [userIds]
);
// Map orders to users in memory

// BEST: Single JOIN query
const result = await db.query(`
  SELECT u.*, json_agg(o.*) as orders
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id
  LIMIT 10
`);
```

## Pagination Strategies

### Offset Pagination (Simple, Not Scalable)
```sql
-- Page 1
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 0;
-- Page 1000 (SLOW - scans 20,000 rows)
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 19980;
```

### Cursor Pagination (Recommended)
```sql
-- First page
SELECT * FROM users ORDER BY id LIMIT 20;
-- Next page (fast - uses index)
SELECT * FROM users WHERE id > 20 ORDER BY id LIMIT 20;
```

### Keyset Pagination (Complex, Fastest)
```sql
-- Multi-column cursor
SELECT * FROM users
WHERE (created_at, id) > ('2024-01-01', 100)
ORDER BY created_at, id
LIMIT 20;
```

## Redis Caching Patterns

### Cache-Aside (Lazy Loading)
```javascript
async function getUser(userId) {
  // 1. Check cache
  const cached = await redis.get(`user:${userId}`);
  if (cached) return JSON.parse(cached);

  // 2. Fetch from database
  const user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);

  // 3. Store in cache with TTL
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));

  return user;
}
```

### Write-Through
```javascript
async function updateUser(userId, data) {
  // 1. Update database
  const user = await db.query(
    'UPDATE users SET name = ? WHERE id = ? RETURNING *',
    [data.name, userId]
  );

  // 2. Update cache immediately
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));

  return user;
}
```

### Cache Invalidation
```javascript
// Event-based invalidation
eventBus.on('user:updated', async (userId) => {
  await redis.del(`user:${userId}`);
  await redis.del(`user:${userId}:profile`);
  // Pattern delete for related caches
  const keys = await redis.keys(`org:*:users`);
  if (keys.length) await redis.del(keys);
});
```

## MongoDB Patterns

### Document Design
```javascript
// Embedded (denormalized) - for 1:few relationships
{
  _id: ObjectId(),
  name: "John Doe",
  addresses: [
    { type: "home", city: "NYC" },
    { type: "work", city: "Boston" }
  ]
}

// Referenced (normalized) - for 1:many relationships
{
  _id: ObjectId(),
  name: "John Doe",
  team_ids: [ObjectId("..."), ObjectId("...")]
}
```

### Aggregation Pipeline
```javascript
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: {
      _id: "$user_id",
      total: { $sum: "$amount" },
      count: { $sum: 1 }
    }
  },
  { $sort: { total: -1 } },
  { $limit: 10 }
]);
```

## Connection Pooling

### PostgreSQL
```javascript
const pool = new Pool({
  max: 20,                    // Max connections
  min: 2,                     // Min connections
  idleTimeoutMillis: 30000,   // Close idle after 30s
  connectionTimeoutMillis: 5000
});
```

### Best Practices
- Pool size = (CPU cores * 2) + effective_spindle_count
- For SSD: 10-20 connections per service instance
- Monitor pool exhaustion
- Use connection timeout

## Performance Checklist

- [ ] Indexes on all foreign keys
- [ ] Indexes on frequently queried columns
- [ ] No sequential scans on large tables
- [ ] N+1 queries eliminated
- [ ] Cursor pagination for large datasets
- [ ] Connection pooling configured
- [ ] Query caching implemented
- [ ] Slow query logging enabled
- [ ] Regular VACUUM/ANALYZE scheduled
- [ ] Replication for read scaling

## Resources

- [Use The Index, Luke](https://use-the-index-luke.com/)
- [PostgreSQL Performance Tips](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [Redis Best Practices](https://redis.io/docs/management/optimization/)
- [MongoDB Performance](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/)
