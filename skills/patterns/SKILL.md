---
name: design-patterns
description: Advanced API design patterns including microservices, event-driven architecture, SAGA pattern, circuit breaker, and composition patterns
---

# Advanced Design Patterns Skill

## Quick Start

Master advanced patterns for complex APIs:

### Microservices Pattern

Instead of monolith:
```
API Gateway
├─ User Service
├─ Post Service
├─ Comment Service
└─ Notification Service
```

Benefits:
- Independent scaling
- Technology flexibility
- Fault isolation

### Event-Driven Pattern

```javascript
// Publish event
await eventBus.publish('user.created', { id, email });

// Subscribe to event
eventBus.subscribe('user.created', async (event) => {
  await sendWelcomeEmail(event.email);
  await updateSearchIndex(event);
});
```

### SAGA Pattern (Distributed Transactions)

```
Order Created
├─ Process Payment
│  ├─ Success: Continue
│  └─ Failure: Compensate (refund)
├─ Reserve Inventory
│  ├─ Success: Continue
│  └─ Failure: Compensate (release)
└─ Create Shipment
   ├─ Success: Confirm Order
   └─ Failure: Compensate (refund + release)
```

### Circuit Breaker Pattern

```
CLOSED (normal) → OPEN (failing) → HALF_OPEN (testing)
```

Prevents cascading failures by failing fast.

### API Composition Pattern

```
Client Request → API Gateway
├─ Call User Service
├─ Call Post Service
├─ Call Comment Service
└─ Aggregate & return
```

### Webhook Pattern

```
User subscribes: POST /webhooks
{
  "url": "https://external.com/webhook",
  "events": ["user.created", "user.updated"]
}

On event: Send POST to webhook URL
```

### Job Queue Pattern

```javascript
queue.add('send-email', { userId: 123 });

queue.process('send-email', async (job) => {
  await sendEmail(job.data.userId);
});
```

## Advanced Patterns Checklist

- [ ] Architecture pattern identified
- [ ] Service boundaries defined
- [ ] Communication patterns chosen
- [ ] Failure handling considered
- [ ] Distributed transactions planned
- [ ] Event sourcing considered
- [ ] Monitoring strategy defined
- [ ] Testing approach for async flows

See the Advanced API Patterns agent for detailed implementation.
