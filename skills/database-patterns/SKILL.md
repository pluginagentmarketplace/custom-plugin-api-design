---
name: database-patterns
description: Database design, optimization, and caching strategies for SQL, NoSQL, and Redis
sasmp_version: "1.3.0"
bonded_agent: 03-database-performance
bond_type: PRIMARY_BOND
---

# Database Patterns Skill

## Quick Start

Master database patterns for high-performance applications.

### SQL Databases
- Schema design and normalization
- Indexing strategies
- Query optimization
- Connection pooling

### NoSQL (MongoDB)
- Document design patterns
- Aggregation pipelines
- Denormalization strategies
- Sharding and replication

### Caching (Redis)
- Cache-aside pattern
- Write-through caching
- TTL management
- Invalidation strategies

## Key Patterns

### N+1 Query Prevention
```javascript
// BAD: N+1 queries
const users = await db.query('SELECT * FROM users');
for (const user of users) {
  user.teams = await db.query('SELECT * FROM user_teams WHERE user_id = ?', [user.id]);
}

// GOOD: Single query with JOIN
const users = await db.query(`
  SELECT u.*, t.name as team_name
  FROM users u
  LEFT JOIN user_teams ut ON u.id = ut.user_id
  LEFT JOIN teams t ON ut.team_id = t.id
`);
```

### Cache-Aside Pattern
```javascript
async function getUser(userId) {
  const cached = await redis.get(`user:${userId}`);
  if (cached) return JSON.parse(cached);

  const user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  return user;
}
```

## Performance Checklist

- [ ] Proper indexing strategy
- [ ] No N+1 queries
- [ ] Pagination implemented
- [ ] Connection pooling configured
- [ ] Caching strategy defined
- [ ] Query optimization done
- [ ] Monitoring in place

See Agent 3: Database Performance for detailed guidance.
