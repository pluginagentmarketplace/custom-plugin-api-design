---
description: Database design, optimization, and performance tuning - SQL, MongoDB, Redis, caching strategies aligned with Data Engineer, Database roles
capabilities: ["SQL optimization", "NoSQL patterns", "Redis caching", "Query optimization", "Data modeling", "Performance tuning", "Replication and sharding"]
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
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_created_at (created_at)
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
  user.teams = await db.query('SELECT * FROM user_teams WHERE user_id = ?', [user.id]);
  // 1 + N queries
}

// ✅ GOOD: Single join
const users = await db.query(`
  SELECT u.*, t.name as team_name
  FROM users u
  LEFT JOIN user_teams ut ON u.id = ut.user_id
  LEFT JOIN teams t ON ut.team_id = t.id
`);

// ✅ BEST: Separate optimized queries
const users = await db.query('SELECT * FROM users LIMIT 10');
const userIds = users.map(u => u.id);
const teamsByUser = await db.query(
  'SELECT user_id, team_id FROM user_teams WHERE user_id IN (?)',
  [userIds]
);
```

### Indexing Strategy

```sql
-- Multi-column index for common queries
CREATE INDEX idx_user_team_role
ON user_teams(user_id, team_id, role);

-- Covering index (query without table access)
CREATE INDEX idx_user_email_status
ON users(email, status)
INCLUDE (id, name);

-- Composite index for sorting
CREATE INDEX idx_created_updated
ON users(created_at DESC, updated_at DESC);
```

## NoSQL Database Patterns (MongoDB)

### Document Design

```javascript
// User document with embedded data
db.users.insertOne({
  _id: ObjectId(),
  email: "user@example.com",
  name: "John Doe",
  teams: [
    { id: ObjectId(), name: "Backend", role: "admin" },
    { id: ObjectId(), name: "DevOps", role: "member" }
  ],
  settings: {
    notifications: true,
    theme: "dark"
  },
  created_at: new Date(),
  updated_at: new Date()
});
```

### Optimization Patterns

```javascript
// Denormalization for read performance
db.users.updateOne(
  { _id: userId },
  {
    $set: {
      teams: [
        { id: teamId, name: "Team Name", memberCount: 15 }
      ]
    }
  }
);

// Aggregation pipeline for complex queries
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
  const user = await db.query(
    'UPDATE users SET ? WHERE id = ?',
    [data, userId]
  );

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
  await redis.del(`org:*:users`); // Pattern delete
});

// Time-based invalidation (already handled by TTL)
await redis.setex(key, 3600, value); // 1 hour TTL
```

## Performance Monitoring

### Query Performance

```sql
-- Analyze query execution
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Show indexes usage
SELECT * FROM information_schema.STATISTICS
WHERE table_name = 'users';

-- Slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 0.5;
```

### Connection Pooling

```javascript
const pool = mysql.createPool({
  connectionLimit: 10,
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  waitForConnections: true,
  enableKeepAlive: true,
  keepAliveInitialDelayMs: 0
});

// Use pool in queries
pool.query('SELECT * FROM users', (err, results) => {
  // Handle results
});
```

## Data Replication & Sharding

### Replication Setup

```
Master (Write)
├─ Slave 1 (Read replica)
├─ Slave 2 (Read replica)
└─ Slave 3 (Backup)
```

### Horizontal Sharding

```
User ID 1-1000000    → Shard 1 (DB1)
User ID 1000001-2000000 → Shard 2 (DB2)
User ID 2000001+     → Shard 3 (DB3)

Shard Key: user_id % 3
```

## Database Performance Checklist

- [ ] Proper indexing strategy
- [ ] No N+1 queries
- [ ] Pagination implemented (cursor-based for large sets)
- [ ] Connection pooling configured
- [ ] Caching strategy defined
- [ ] Query optimization done
- [ ] Replication for backup/read distribution
- [ ] Monitoring and alerting in place
- [ ] Regular backups scheduled
- [ ] Disaster recovery plan

---

**Next:** DevOps & Infrastructure (Agent 4)
