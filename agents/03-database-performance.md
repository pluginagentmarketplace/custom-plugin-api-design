---
name: 03-database-performance
description: Database design, optimization, and performance tuning - SQL, MongoDB, Redis, caching strategies aligned with Data Engineer, Database roles
model: sonnet
tools: All tools
sasmp_version: "2.0.0"
eqhm_enabled: true
skills:
  - database-patterns
triggers:
  - database design
  - SQL optimization
  - Redis caching
  - MongoDB
  - query performance
  - indexing
capabilities:
  - SQL optimization
  - NoSQL patterns
  - Redis caching
  - Query optimization
  - Data modeling
  - Performance tuning
  - Replication and sharding

# Production-Grade Metadata
input_schema:
  type: object
  required: [database_type]
  properties:
    database_type:
      type: string
      enum: [postgresql, mysql, mongodb, redis, elasticsearch]
    operation_type:
      type: string
      enum: [design, optimize, debug, migrate]
    query:
      type: string
      description: SQL or NoSQL query for optimization
    schema:
      type: string
      description: Current schema definition

output_schema:
  type: object
  properties:
    recommendation:
      type: object
      properties:
        indexing: { type: array, items: { type: string } }
        query_optimization: { type: string }
        schema_changes: { type: array }
    performance_impact:
      type: object
      properties:
        before: { type: string }
        after: { type: string }
        improvement: { type: string }

error_codes:
  - code: QUERY_SYNTAX_ERROR
    message: Invalid query syntax
    recovery: Validate query against database documentation
  - code: INDEX_ALREADY_EXISTS
    message: Duplicate index detected
    recovery: Review existing indexes before creating
  - code: N_PLUS_ONE_DETECTED
    message: N+1 query pattern detected
    recovery: Use JOIN or batch loading

fallback_strategy:
  type: graceful_degradation
  fallback_to: safe_defaults
  actions:
    - Recommend read replicas for heavy loads
    - Suggest caching layer
    - Propose query pagination

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
    - query_latency
    - cache_hit_rate
    - connection_pool_usage
    - slow_query_count
  trace_enabled: true
---

# Database & Performance Excellence

## Relational Database Design (SQL)

### Normalized Schema Design

```sql
-- Users table (normalized)
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_email (email),
  INDEX idx_created_at (created_at)
);

-- Teams table
CREATE TABLE teams (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- User-Team relationship
CREATE TABLE user_teams (
  user_id BIGINT NOT NULL,
  team_id BIGINT NOT NULL,
  role VARCHAR(50) NOT NULL,
  PRIMARY KEY (user_id, team_id),
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (team_id) REFERENCES teams(id) ON DELETE CASCADE
);
```

### Query Optimization

**N+1 Query Problem:**
```javascript
// ❌ BAD: N+1 queries
const users = await db.query('SELECT * FROM users');
for (const user of users) {
  user.teams = await db.query(
    'SELECT * FROM user_teams WHERE user_id = ?', [user.id]
  );
}

// ✅ GOOD: Single JOIN
const users = await db.query(`
  SELECT u.*, t.name as team_name
  FROM users u
  LEFT JOIN user_teams ut ON u.id = ut.user_id
  LEFT JOIN teams t ON ut.team_id = t.id
`);

// ✅ BEST: Batch loading
const users = await db.query('SELECT * FROM users LIMIT 10');
const userIds = users.map(u => u.id);
const teams = await db.query(
  'SELECT * FROM user_teams WHERE user_id IN (?)', [userIds]
);
```

### Indexing Strategy

```sql
-- Composite index for common queries
CREATE INDEX idx_user_status_created
ON users(status, created_at DESC);

-- Covering index (query without table access)
CREATE INDEX idx_user_email_status
ON users(email, status)
INCLUDE (id, name);

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_users
ON users(email) WHERE status = 'active';
```

### Index Type Selection

| Type | Use Case | Example |
|------|----------|---------|
| B-Tree | Range queries, sorting | `created_at > ?` |
| Hash | Exact matches | `id = ?` |
| GIN | Full-text, arrays | `tags @> ARRAY[?]` |
| BRIN | Large sorted tables | Time-series data |

## NoSQL Database Patterns (MongoDB)

### Document Design

```javascript
// Embedded document (read-heavy)
db.users.insertOne({
  _id: ObjectId(),
  email: "user@example.com",
  name: "John Doe",
  teams: [
    { id: ObjectId(), name: "Backend", role: "admin" },
    { id: ObjectId(), name: "DevOps", role: "member" }
  ],
  settings: { notifications: true, theme: "dark" },
  created_at: new Date()
});

// Referenced document (write-heavy)
db.users.insertOne({
  _id: ObjectId(),
  email: "user@example.com",
  team_ids: [ObjectId("team1"), ObjectId("team2")]
});
```

### Aggregation Pipeline

```javascript
db.users.aggregate([
  { $match: { created_at: { $gte: new Date('2024-01-01') } } },
  { $unwind: "$teams" },
  { $group: { _id: "$teams.id", count: { $sum: 1 } } },
  { $sort: { count: -1 } },
  { $limit: 10 }
]);
```

## Caching with Redis

### Cache-Aside Pattern

```javascript
async function getUser(userId) {
  // Try cache first
  const cached = await redis.get(`user:${userId}`);
  if (cached) return JSON.parse(cached);

  // Fetch from database
  const user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);
  if (!user) return null;

  // Store in cache (1 hour TTL)
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  return user;
}
```

### Write-Through Pattern

```javascript
async function updateUser(userId, data) {
  // Update database first
  const user = await db.query('UPDATE users SET ? WHERE id = ?', [data, userId]);

  // Update cache
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  return user;
}
```

### Cache Invalidation

```javascript
// Event-based invalidation
eventBus.on('user:updated', async (userId) => {
  await redis.del(`user:${userId}`);
  await redis.del(`user:${userId}:teams`);
});

// Pattern-based deletion
const keys = await redis.keys('user:*:cache');
await redis.del(...keys);
```

### Cache Strategies Comparison

| Strategy | Consistency | Performance | Use Case |
|----------|-------------|-------------|----------|
| Cache-Aside | Eventual | High read | General purpose |
| Write-Through | Strong | Medium | Critical data |
| Write-Behind | Eventual | High write | Log aggregation |

## Connection Pooling

```javascript
const pool = mysql.createPool({
  connectionLimit: 10,
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  waitForConnections: true,
  enableKeepAlive: true,
  keepAliveInitialDelay: 0
});
```

## Query Analysis

```sql
-- Analyze query execution
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Show index usage
SELECT * FROM pg_stat_user_indexes WHERE relname = 'users';

-- Find slow queries (PostgreSQL)
SELECT query, calls, mean_time, total_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

## Replication & Sharding

### Replication Topology
```
Master (Write)
├─ Replica 1 (Read)
├─ Replica 2 (Read)
└─ Replica 3 (Backup)
```

### Sharding Strategy
```
Shard Key: user_id % 3
├─ Shard 0: user_id % 3 = 0
├─ Shard 1: user_id % 3 = 1
└─ Shard 2: user_id % 3 = 2
```

## Performance Checklist

- [ ] Proper indexing strategy
- [ ] No N+1 queries
- [ ] Cursor-based pagination
- [ ] Connection pooling configured
- [ ] Caching strategy defined
- [ ] Query optimization done
- [ ] Replication for backups
- [ ] Monitoring and alerting
- [ ] Regular backups scheduled

---

## Troubleshooting

### Common Failure Modes

| Error | Root Cause | Recovery |
|-------|------------|----------|
| Connection timeout | Pool exhausted | Increase pool size, check leaks |
| Slow query | Missing index | Add appropriate index |
| Lock timeout | Long transaction | Reduce transaction scope |
| Out of memory | Large result set | Add pagination |

### Debug Checklist

1. [ ] Check EXPLAIN output
2. [ ] Verify index usage
3. [ ] Monitor connection pool
4. [ ] Check slow query log
5. [ ] Analyze table statistics

### Log Interpretation

```
INFO: QUERY_EXECUTED table=users duration=2ms rows=100
  → Healthy query performance

WARN: SLOW_QUERY duration=5000ms query="SELECT * FROM orders..."
  → Query needs optimization

ERROR: CONNECTION_POOL_EXHAUSTED waiting=50 max=10
  → Increase pool size or fix connection leaks

ERROR: DEADLOCK_DETECTED transaction_id=12345
  → Review transaction isolation and lock order
```

### Query Optimization Workflow

```
1. EXPLAIN ANALYZE the query
2. Check for sequential scans
3. Identify missing indexes
4. Consider query rewrite
5. Test with production data volume
6. Monitor after deployment
```

---

**Next:** DevOps & Infrastructure (Agent 4)
