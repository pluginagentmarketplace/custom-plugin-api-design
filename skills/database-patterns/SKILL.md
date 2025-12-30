---
name: database-patterns
description: Database design, optimization, and caching strategies for SQL, NoSQL, and Redis
sasmp_version: "2.0.0"
bonded_agent: 03-database-performance
bond_type: PRIMARY_BOND

# Production-Grade Metadata
parameters:
  - name: database_type
    type: string
    required: true
    validation: "^(postgresql|mysql|mongodb|redis|elasticsearch)$"
    description: Database type
  - name: operation
    type: string
    required: false
    validation: "^(design|optimize|debug|migrate)$"
    description: Operation type

validation_rules:
  - All queries must use indexes for large tables
  - N+1 queries must be avoided
  - Connection pooling must be configured

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [query_duration, cache_hit_rate, connection_pool_usage]
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
- Sharding and replication

### Caching (Redis)
- Cache-aside pattern
- Write-through caching
- TTL management

## Key Patterns

### N+1 Query Prevention
```javascript
// BAD: N+1 queries
for (const user of users) {
  user.teams = await db.query('SELECT * FROM teams WHERE user_id = ?', [user.id]);
}

// GOOD: Single JOIN
const users = await db.query(`
  SELECT u.*, t.name as team_name
  FROM users u LEFT JOIN teams t ON u.id = t.user_id
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

## Unit Test Template

```javascript
describe('database-patterns skill', () => {
  test('cache returns data on hit', async () => {
    await redis.set('user:123', JSON.stringify({ id: '123' }));
    const result = await getUser('123');
    expect(result.id).toBe('123');
  });

  test('cache miss fetches from DB', async () => {
    const result = await getUser('456');
    expect(result).toBeDefined();
  });
});
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Slow queries | Missing index | Add appropriate index |
| Connection timeout | Pool exhausted | Increase pool size |
| Lock timeout | Long transaction | Reduce transaction scope |

See Agent 3: Database Performance for detailed guidance.
