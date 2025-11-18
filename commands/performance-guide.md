# Performance Optimization Guide

Analyze your API and get specific recommendations for performance improvements.

## What's Analyzed

### 📊 Response Times
- [ ] Current latency metrics
- [ ] Target response times
- [ ] P95/P99 percentiles
- [ ] Bottleneck identification

### 💾 Caching Strategy
- [ ] HTTP cache headers
- [ ] Server-side caching (Redis)
- [ ] Database query caching
- [ ] Cache invalidation
- [ ] Cache warming strategy

### 🗄️ Database Optimization
- [ ] Index creation
- [ ] Query optimization
- [ ] N+1 query prevention
- [ ] Connection pooling
- [ ] Sharding strategy

### 📦 Payload Optimization
- [ ] Response compression (gzip/brotli)
- [ ] Field selection/sparse fieldsets
- [ ] Pagination strategy
- [ ] Payload size reduction

### ⚡ Load & Scalability
- [ ] Current throughput
- [ ] Expected growth
- [ ] Horizontal scaling readiness
- [ ] Load balancing strategy
- [ ] CDN usage

### 📈 Monitoring
- [ ] Response time tracking
- [ ] Error rate monitoring
- [ ] Resource utilization
- [ ] Database performance
- [ ] APM tools configured

## Performance Checklist

### Quick Wins (Easy, High Impact)
- ✅ Enable gzip compression
- ✅ Add cache headers
- ✅ Optimize database queries
- ✅ Add Redis caching
- ✅ Implement pagination

### Medium Effort (Moderate Impact)
- ✅ Implement cursor-based pagination
- ✅ Add database indexes
- ✅ Use connection pooling
- ✅ Implement rate limiting
- ✅ Add CDN for static files

### Advanced (Complex, High Impact)
- ✅ Implement caching layer
- ✅ Database sharding
- ✅ Microservices separation
- ✅ API composition pattern
- ✅ Event-driven architecture

## Response Time Targets

| Target | Type | Example |
|--------|------|---------|
| < 100ms | Fast | Cache hit, simple query |
| 100-500ms | Good | Database query, light processing |
| 500ms-1s | Acceptable | Complex query, some processing |
| > 1s | Slow | Needs optimization |

## How to Optimize

Provide your current metrics:
1. **Response times** - Current latency distribution
2. **Endpoints** - Which are slow?
3. **Data** - Volume and size?
4. **Traffic** - Requests per second?
5. **Infrastructure** - Server specs?

## Optimization Recommendations

After analysis, you'll get:
- Response time improvement potential
- Specific optimization techniques
- Implementation examples
- Tools and libraries to use
- Monitoring setup guide

## Tools for Monitoring

### Application Performance Monitoring (APM)
- New Relic
- Datadog
- Elastic APM
- Prometheus + Grafana

### Load Testing
- Apache JMeter
- Locust
- k6
- Gatling

### Profiling
- Chrome DevTools
- Node.js Inspector
- Python cProfile
- Go pprof

## Next Steps

1. **Baseline** - Measure current performance
2. **Identify** - Find bottlenecks
3. **Optimize** - Implement improvements
4. **Test** - Verify performance gains
5. **Monitor** - Track improvements over time

## Example Optimizations

### Before
```
GET /api/users/123/posts
└─ Takes 2 seconds
   ├─ User query: 100ms
   ├─ For each post: Comments query (100ms × 20 posts)
   └─ Total: 100 + 2000 = 2100ms
```

### After
```
GET /api/users/123/posts?include=comments
└─ Takes 200ms
   ├─ User + Posts query (batch): 50ms
   ├─ Comments query (single batch): 50ms
   └─ Total: 100ms

Plus:
- Cache: 10ms (90% of requests)
```

## Quick Performance Wins

### 1. Add Pagination
```
GET /api/posts?limit=10
```
Reduce payload by 90%

### 2. Enable Compression
```
Gzip: 75% smaller
Brotli: 80% smaller
```

### 3. Add Redis Cache
```
Cache common queries
99% hit rate possible
```

### 4. Create Indexes
```
Database query: 10x faster
```

### 5. Fix N+1 Queries
```
From 100 queries to 2 queries
90% faster
```

## Related Topics

- Use `/design` for architecture recommendations
- Use `/review` for general design feedback
- Use `/security-check` for security audit
