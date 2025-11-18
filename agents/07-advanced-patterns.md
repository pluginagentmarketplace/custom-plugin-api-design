---
description: Advanced API patterns including microservices, event-driven architecture, async operations, webhooks, and enterprise patterns
capabilities: ["Microservices architecture", "Event-driven APIs", "Async operations", "Webhooks", "API composition", "SAGA pattern", "Circuit breaker"]
---

# Advanced API Patterns

## Microservices Architecture

### Traditional Monolithic
```
Client → Monolithic App
           ├─ Auth
           ├─ Users
           ├─ Posts
           ├─ Comments
           └─ Notifications
```
**Problem:** Single point of failure, hard to scale

### Microservices
```
Client → API Gateway
           ├─ Auth Service
           ├─ User Service
           ├─ Post Service
           ├─ Comment Service
           └─ Notification Service
```
**Benefits:**
- Independent scaling
- Technology flexibility
- Fault isolation
- Team autonomy

### Service Communication

#### Synchronous (REST)
```
User Service → GET /api/posts/123
Post Service → 200 OK {post data}
```
❌ Tightly coupled
❌ Cascading failures

#### Asynchronous (Message Queue)
```
User Service → [Publish: UserCreated]
Auth Service → [Listen: UserCreated]
Notification Service → [Listen: UserCreated]
```
✅ Decoupled
✅ Resilient

## Event-Driven Architecture

### Event Sourcing
```
Instead of storing current state:
User table: {id: 1, name: "John", email: "john@example.com"}

Store events:
1. UserCreated: {id: 1, email: "john@example.com"}
2. UserUpdated: {id: 1, name: "John"}
3. UserNameUpdated: {id: 1, name: "Johnny"}
```

**Benefits:**
- Full audit trail
- Time travel debugging
- Rebuild state from events

### Event Publishing
```javascript
// Publish user created event
await eventBus.publish('user.created', {
  id: user.id,
  email: user.email,
  createdAt: new Date()
});

// Subscribe to event
eventBus.subscribe('user.created', async (event) => {
  // Send welcome email
  await sendWelcomeEmail(event.email);

  // Update search index
  await elasticsearch.index('users', event);

  // Send to external webhook
  await axios.post(webhook.url, event);
});
```

## Asynchronous Operations

### Job Queue Pattern
```javascript
// Queue a long-running task
const job = await queue.add('send-email', {
  userId: 123,
  templateId: 'welcome'
}, {
  delay: 5000, // Run after 5 seconds
  attempts: 3, // Retry 3 times
  backoff: { type: 'exponential', delay: 2000 }
});

// Process jobs
queue.process('send-email', async (job) => {
  const { userId, templateId } = job.data;
  await sendEmail(userId, templateId);
});
```

### Webhook Pattern
```javascript
// User subscribes to events
POST /api/webhooks
{
  "url": "https://myapp.com/webhooks/user",
  "events": ["user.created", "user.updated"],
  "secret": "webhook_secret"
}

// When event occurs, notify subscriber
async function notifyWebhook(event, webhook) {
  const signature = createSignature(event, webhook.secret);

  await axios.post(webhook.url, event, {
    headers: {
      'X-Webhook-Signature': signature,
      'X-Webhook-ID': generateId(),
      'X-Webhook-Timestamp': new Date().toISOString()
    }
  });
}

// Subscriber verifies signature
function verifyWebhook(body, signature, secret) {
  const expectedSignature = createSignature(body, secret);
  return constant_time_compare(signature, expectedSignature);
}
```

## SAGA Pattern

### Distributed Transactions
```
Order Service → Initiate Order
  ├─ Payment Service → Process Payment
  │  └─ ❌ Payment Failed
  │     └─ Order Service → Compensate: Cancel Order
  │
  ├─ Inventory Service → Reserve Items
  │  └─ ❌ Out of Stock
  │     └─ Payment Service → Compensate: Refund
  │     └─ Order Service → Compensate: Cancel Order
  │
  └─ Shipping Service → Create Shipment
     └─ ❌ Shipping Failed
        └─ [Compensate all previous steps]
```

### Implementation
```javascript
class OrderSaga {
  async execute(orderData) {
    const order = { id: generateId(), status: 'pending' };

    try {
      // Step 1: Process payment
      order.paymentId = await this.paymentService.process(orderData.payment);

      // Step 2: Reserve inventory
      order.reservationId = await this.inventoryService.reserve(orderData.items);

      // Step 3: Create shipment
      order.shipmentId = await this.shippingService.create(orderData.shipping);

      order.status = 'confirmed';
      return order;

    } catch (error) {
      // Compensate in reverse order
      if (order.shipmentId) {
        await this.shippingService.cancel(order.shipmentId);
      }
      if (order.reservationId) {
        await this.inventoryService.release(order.reservationId);
      }
      if (order.paymentId) {
        await this.paymentService.refund(order.paymentId);
      }

      throw new OrderSagaFailed(error);
    }
  }
}
```

## Circuit Breaker Pattern

### Problem: Cascading Failures
```
Service A calls Service B which calls Service C
Service C goes down → B times out → A times out → Cascade failure
```

### Solution: Circuit Breaker
```
CLOSED: Normal operation
  ↓ (after failures threshold)
OPEN: Stop calling (fail fast)
  ↓ (after timeout)
HALF_OPEN: Try one request
  ├─ Success → CLOSED
  └─ Failure → OPEN
```

### Implementation
```javascript
const CircuitBreaker = require('opossum');

const breakerOptions = {
  timeout: 3000, // 3 seconds
  errorThresholdPercentage: 50, // Open if 50% fail
  resetTimeout: 30000 // Try again after 30s
};

const breaker = new CircuitBreaker(async (url) => {
  return await axios.get(url);
}, breakerOptions);

breaker.fallback(() => {
  return { cached: true, data: [] };
});

// Usage
try {
  const response = await breaker.fire('https://unreliable-service.com/api');
} catch (error) {
  // Circuit is open or request failed
}
```

## Rate Limiting Patterns

### Shared Rate Limits
```
User: 1000 requests/hour
API Key: 10000 requests/hour
IP Address: 5000 requests/hour
```

### Implementation
```javascript
const RateLimiter = require('redis-rate-limiter');

const limiter = new RateLimiter({
  redis: redisClient,
  rules: {
    'user:123': { points: 1000, duration: 3600 },
    'api-key:sk_live_abc': { points: 10000, duration: 3600 }
  }
};

app.use(async (req, res, next) => {
  const key = req.user ? `user:${req.user.id}` : `ip:${req.ip}`;

  try {
    await limiter.consume(key);
    next();
  } catch (error) {
    res.status(429).json({
      error: 'Too Many Requests',
      retryAfter: error.msBeforeNext / 1000
    });
  }
});
```

## API Composition

### Aggregation Pattern
```
Client Request → API Gateway
  ├─ Get User → User Service
  ├─ Get Posts → Post Service
  ├─ Get Comments → Comment Service
  └─ Get Followers → User Service

API Gateway → Aggregate & return combined response
```

### Implementation
```javascript
app.get('/api/profile/:id', async (req, res) => {
  try {
    const [user, posts, followers] = await Promise.all([
      userService.getUser(req.params.id),
      postService.getUserPosts(req.params.id),
      userService.getFollowers(req.params.id)
    ]);

    res.json({
      user,
      posts,
      followers,
      stats: {
        postsCount: posts.length,
        followersCount: followers.length
      }
    });
  } catch (error) {
    res.status(500).json({ error: 'Failed to load profile' });
  }
});
```

## API Versioning

### URL Path Versioning
```
/api/v1/users
/api/v2/users
```
✅ Clear, explicit
❌ Code duplication

### Header Versioning
```
Accept: application/vnd.api+json;version=2
```
✅ Cleaner URLs
❌ Less obvious

### Query Parameter
```
/api/users?version=2
```
❌ Unconventional

### Migration Strategy
```
1. Release v2 alongside v1
2. Announce v1 deprecation
3. Give 6-12 months notice
4. Shutdown v1
5. v2 becomes default (no version in path)
```

## Advanced Patterns Checklist

- [ ] Microservices decomposition identified
- [ ] Service communication patterns chosen
- [ ] Event sourcing considered
- [ ] SAGA pattern for distributed transactions
- [ ] Circuit breaker for resilience
- [ ] Rate limiting strategy defined
- [ ] API gateway configured
- [ ] Webhook support implemented
- [ ] Job queue for async tasks
- [ ] Versioning strategy documented
- [ ] Monitoring for distributed system
- [ ] Testing strategy for async flows

## Next Steps

You now have expertise across all API design aspects:
1. ✅ Fundamentals
2. ✅ REST Architecture
3. ✅ GraphQL
4. ✅ Security
5. ✅ Performance
6. ✅ Documentation & Testing
7. ✅ Advanced Patterns

Use the `/design` command to start building your API!
