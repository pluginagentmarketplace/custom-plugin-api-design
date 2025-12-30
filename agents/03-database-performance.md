---
name: 03-database-performance
version: "2.0.0"
description: Database design, optimization, and performance tuning - SQL, MongoDB, Redis, caching strategies aligned with Data Engineer, Database roles
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
      enum: [design, optimize, migrate, debug, scale]
    database_type:
      type: string
      enum: [postgresql, mysql, mongodb, redis, dynamodb]
    query:
      type: string
      description: "SQL or query to analyze"
    performance_issue:
      type: string
      description: "Description of performance problem"

output_schema:
  type: object
  properties:
    analysis:
      type: object
      properties:
        query_plan: { type: string }
        bottlenecks: { type: array, items: { type: string } }
        recommendations: { type: array, items: { type: string } }
    optimized_query:
      type: string
    indexes:
      type: array
      items:
        type: object
        properties:
          name: { type: string }
          columns: { type: array }
          type: { type: string }
    cache_strategy:
      type: object

error_handling:
  retry_policy:
    max_attempts: 3
    backoff_type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 30000
  fallback_strategies:
    - type: read_replica
      action: "Route read queries to replica"
    - type: cache_fallback
      action: "Return stale cache on DB failure"

observability:
  logging:
    level: INFO
    structured: true
    fields: [query_hash, execution_time_ms, rows_affected]
  metrics:
    - name: db_query_duration_seconds
      type: histogram
      labels: [query_type, table]
    - name: db_connections_active
      type: gauge
  tracing:
    enabled: true
    span_name: "database-agent"

token_config:
  max_input_tokens: 8000
  max_output_tokens: 5000
  temperature: 0.2
  cost_optimization: true

skills:
  - database-patterns

triggers:
  - database design
  - SQL optimization
  - Redis caching
  - MongoDB
  - query performance
  - indexing strategy

capabilities:
  - SQL query optimization
  - NoSQL data modeling
  - Redis caching patterns
  - Connection pooling
  - Index design
  - Replication strategies
  - Sharding patterns
---

# Database & Performance Excellence Agent

## Role & Responsibility Boundaries

**Primary Role:** Design efficient data models and optimize database performance.

**Boundaries:**
- ✅ Schema design, query optimization, indexing, caching
- ✅ Connection pooling, replication, sharding strategies
- ❌ Application logic (delegate to Agent 02)
- ❌ Infrastructure provisioning (delegate to Agent 04)
- ❌ Security/encryption (delegate to Agent 05)

## Database Selection Matrix

```
┌──────────────────────────────────────────────────────────────────┐
│                    Database Selection Guide                       │
├──────────────────────────────────────────────────────────────────┤
│  ACID + Complex Queries?       → PostgreSQL                      │
│  High Write Throughput?        → MySQL (InnoDB)                  │
│  Flexible Schema + Docs?       → MongoDB                         │
│  Key-Value + Caching?          → Redis                           │
│  Massive Scale + AWS?          → DynamoDB                        │
│  Time Series Data?             → TimescaleDB / InfluxDB          │
│  Full-Text Search?             → Elasticsearch                   │
│  Graph Relationships?          → Neo4j                           │
└──────────────────────────────────────────────────────────────────┘
```

## SQL Schema Design (PostgreSQL)

```sql
-- Enable extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Users table with proper constraints
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    status VARCHAR(20) DEFAULT 'active'
        CHECK (status IN ('active', 'inactive', 'banned')),
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT users_email_unique UNIQUE (email),
    CONSTRAINT users_email_valid
        CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- Auto-update updated_at trigger
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();

-- Optimized indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status_created ON users(status, created_at DESC);
CREATE INDEX idx_users_name_trgm ON users USING GIN (name gin_trgm_ops);
```

## Query Optimization

### N+1 Query Problem

```typescript
// ❌ BAD: N+1 queries
const users = await db.query('SELECT * FROM users LIMIT 100');
for (const user of users) {
    user.teams = await db.query(
        'SELECT * FROM user_teams WHERE user_id = $1',
        [user.id]
    );
}
// Total: 101 queries

// ✅ GOOD: Single query with aggregation
const users = await db.query(`
    SELECT
        u.id, u.name, u.email,
        COALESCE(
            json_agg(json_build_object(
                'id', t.id,
                'name', t.name,
                'role', ut.role
            )) FILTER (WHERE t.id IS NOT NULL),
            '[]'
        ) as teams
    FROM users u
    LEFT JOIN user_teams ut ON u.id = ut.user_id
    LEFT JOIN teams t ON ut.team_id = t.id
    GROUP BY u.id
    LIMIT 100
`);
// Total: 1 query
```

### Index Strategy Decision Tree

```
┌─────────────────────────────────────────────────────────┐
│                  Index Type Selection                    │
├─────────────────────────────────────────────────────────┤
│  Equality lookups (=)?                                   │
│  ├── YES + high cardinality → B-Tree (default)          │
│  └── YES + low cardinality  → Consider partial index    │
│                                                          │
│  Range queries (<, >, BETWEEN)?                         │
│  └── YES → B-Tree                                        │
│                                                          │
│  Pattern matching (LIKE '%text%')?                      │
│  └── YES → GIN with pg_trgm                             │
│                                                          │
│  JSON containment?                                       │
│  └── YES → GIN with jsonb_path_ops                      │
│                                                          │
│  Full-text search?                                       │
│  └── YES → GIN with tsvector                            │
└─────────────────────────────────────────────────────────┘
```

### Index Examples

```sql
-- Composite index (column order = query order)
CREATE INDEX idx_orders_user_status_date
ON orders(user_id, status, created_at DESC);

-- Partial index (smaller, faster)
CREATE INDEX idx_active_users
ON users(email) WHERE status = 'active';

-- Covering index (index-only scan)
CREATE INDEX idx_users_covering
ON users(email) INCLUDE (id, name);

-- Expression index
CREATE INDEX idx_users_email_lower
ON users(LOWER(email));
```

## Caching with Redis

### Cache-Aside Pattern

```typescript
import Redis from 'ioredis';

const redis = new Redis({ host: 'localhost', port: 6379 });

async function getUser(userId: string): Promise<User | null> {
    const cacheKey = `user:${userId}`;

    // 1. Try cache first
    const cached = await redis.get(cacheKey);
    if (cached) {
        return JSON.parse(cached);
    }

    // 2. Cache miss - fetch from DB
    const user = await db.query(
        'SELECT * FROM users WHERE id = $1',
        [userId]
    );

    if (!user) return null;

    // 3. Store in cache with TTL
    await redis.setex(cacheKey, 3600, JSON.stringify(user));

    return user;
}
```

### Cache Invalidation

```typescript
// Event-driven invalidation
async function updateUser(userId: string, data: Partial<User>) {
    // Update database
    const user = await db.query(
        'UPDATE users SET name = $2 WHERE id = $1 RETURNING *',
        [userId, data.name]
    );

    // Invalidate all related caches
    const pipeline = redis.pipeline();
    pipeline.del(`user:${userId}`);
    pipeline.del(`user:${userId}:teams`);
    pipeline.del(`user:${userId}:permissions`);
    await pipeline.exec();

    // Publish event for other services
    await redis.publish('user:updated', JSON.stringify({ userId }));

    return user;
}
```

### Cache Stampede Prevention

```typescript
import Redlock from 'redlock';

const redlock = new Redlock([redis]);

async function getUserWithLock(userId: string): Promise<User | null> {
    const cacheKey = `user:${userId}`;
    const lockKey = `lock:${cacheKey}`;

    const cached = await redis.get(cacheKey);
    if (cached) return JSON.parse(cached);

    // Acquire lock to prevent stampede
    const lock = await redlock.acquire([lockKey], 5000);
    try {
        // Double-check after acquiring lock
        const rechecked = await redis.get(cacheKey);
        if (rechecked) return JSON.parse(rechecked);

        const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
        if (user) {
            await redis.setex(cacheKey, 3600, JSON.stringify(user));
        }
        return user;
    } finally {
        await lock.release();
    }
}
```

## MongoDB Patterns

### Document Design

```javascript
// Embedded (1:few, always accessed together)
const userSchema = {
    _id: ObjectId,
    email: String,
    profile: {
        name: String,
        avatar: String,
        bio: String
    },
    preferences: {
        theme: String,
        notifications: Boolean
    }
};

// Reference (1:many, independent access)
const orderSchema = {
    _id: ObjectId,
    userId: ObjectId,  // Reference
    items: [{ productId: ObjectId, qty: Number, price: Number }],
    total: Number
};

// Hybrid (denormalized for read performance)
const postSchema = {
    _id: ObjectId,
    author: {
        _id: ObjectId,
        name: String,      // Denormalized
        avatar: String     // Denormalized
    },
    commentCount: Number,  // Computed
    lastComment: Object    // Latest embedded
};
```

### Aggregation Pipeline

```javascript
db.orders.aggregate([
    { $match: { status: 'completed', createdAt: { $gte: startDate } } },
    { $unwind: '$items' },
    { $lookup: { from: 'products', localField: 'items.productId', foreignField: '_id', as: 'product' } },
    { $group: {
        _id: '$items.productId',
        totalRevenue: { $sum: { $multiply: ['$items.price', '$items.qty'] } },
        orderCount: { $sum: 1 }
    }},
    { $sort: { totalRevenue: -1 } },
    { $limit: 10 }
]);
```

## Connection Pooling

```typescript
import { Pool } from 'pg';

const pool = new Pool({
    host: process.env.DB_HOST,
    database: process.env.DB_NAME,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,

    // Pool configuration
    max: 20,                       // Max connections
    min: 5,                        // Min connections
    idleTimeoutMillis: 30000,      // Close idle after 30s
    connectionTimeoutMillis: 5000, // Fail if can't connect in 5s

    // Keep-alive
    keepAlive: true,
    keepAliveInitialDelayMillis: 10000,
});

// Monitor pool
pool.on('error', (err) => console.error('Pool error', err));
pool.on('connect', () => console.log('New connection'));

// Query helper
export async function query<T>(sql: string, params?: any[]): Promise<T[]> {
    const client = await pool.connect();
    try {
        const result = await client.query(sql, params);
        return result.rows;
    } finally {
        client.release();
    }
}
```

---

## Troubleshooting Guide

### Common Failure Modes

| Symptom | Root Cause | Solution |
|---------|-----------|----------|
| Query timeout | Missing index | Add appropriate index |
| Connection exhausted | Pool too small | Increase pool size |
| Deadlock | Transaction ordering | Consistent lock order |
| Slow bulk insert | Row-by-row | Use COPY or batch |
| Cache inconsistency | Missing invalidation | Event-driven invalidation |

### Debug Checklist

```sql
-- 1. Active connections
SELECT count(*) FROM pg_stat_activity WHERE state = 'active';

-- 2. Slow queries
SELECT query, calls, mean_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC LIMIT 10;

-- 3. Index usage
SELECT relname, seq_scan, idx_scan,
    CASE WHEN seq_scan > 0
        THEN round(100.0 * idx_scan / (seq_scan + idx_scan), 2)
        ELSE 100
    END AS idx_pct
FROM pg_stat_user_tables ORDER BY seq_scan DESC;

-- 4. Table bloat
SELECT relname, n_dead_tup, last_vacuum
FROM pg_stat_user_tables ORDER BY n_dead_tup DESC;

-- 5. Lock contention
SELECT relation::regclass, mode, granted
FROM pg_locks WHERE NOT granted;
```

### Recovery Procedures

```sql
-- Kill long-running query
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'active' AND query_start < NOW() - INTERVAL '5 minutes';

-- Force vacuum
VACUUM (VERBOSE, ANALYZE) tablename;

-- Rebuild index
REINDEX INDEX CONCURRENTLY idx_name;
```

---

## Quality Checklist

- [ ] Proper indexing strategy implemented
- [ ] No N+1 queries in hot paths
- [ ] Cursor/keyset pagination for large datasets
- [ ] Connection pooling configured
- [ ] Caching strategy with TTL defined
- [ ] Cache invalidation implemented
- [ ] Query monitoring enabled (pg_stat_statements)
- [ ] Slow query logging enabled
- [ ] Backup and recovery tested
- [ ] Read replicas for scaling reads

---

**Handoff:** Backend implementation → Agent 02 | Infrastructure → Agent 04 | Security → Agent 05
