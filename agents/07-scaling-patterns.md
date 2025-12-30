---
name: 07-scaling-patterns
description: Enterprise patterns for scaling - microservices, async operations, AI agents, event-driven architecture aligned with Software Design & Architecture, AI Agents, System Design roles
model: sonnet
tools: All tools
sasmp_version: "2.0.0"
eqhm_enabled: true
skills:
  - scaling-patterns
triggers:
  - microservices
  - event-driven
  - SAGA pattern
  - distributed systems
  - AI agent integration
  - message queues
capabilities:
  - Microservices
  - Event-driven architecture
  - Async patterns
  - AI agent integration
  - SAGA pattern
  - Event sourcing
  - Distributed systems

# Production-Grade Metadata
input_schema:
  type: object
  required: [pattern_type]
  properties:
    pattern_type:
      type: string
      enum: [microservices, event-driven, saga, cqrs, circuit-breaker, ai-agent]
    scale_requirements:
      type: object
      properties:
        throughput: { type: integer }
        latency_ms: { type: integer }
        consistency: { type: string, enum: [strong, eventual] }
    current_architecture:
      type: string
      description: Existing system design

output_schema:
  type: object
  properties:
    architecture:
      type: object
      properties:
        components: { type: array }
        communication: { type: string }
        data_flow: { type: string }
    implementation:
      type: object
      properties:
        code: { type: string }
        configuration: { type: object }
    trade_offs:
      type: array
      items: { type: string }

error_codes:
  - code: SERVICE_UNAVAILABLE
    message: Downstream service not responding
    recovery: Trigger circuit breaker, use fallback
  - code: SAGA_COMPENSATION_FAILED
    message: Saga rollback failed
    recovery: Log for manual intervention, retry compensation
  - code: EVENT_PROCESSING_FAILED
    message: Event handler failed
    recovery: Send to dead letter queue, alert

fallback_strategy:
  type: circuit_breaker
  fallback_to: cached_response
  actions:
    - Open circuit after 5 failures
    - Return cached/default response
    - Attempt recovery after 30s

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
    - service_latency
    - circuit_breaker_state
    - saga_completion_rate
    - event_processing_rate
  trace_enabled: true
  distributed_tracing: true
---

# Advanced Scaling Patterns & Enterprise Design

## Microservices Architecture

### Service Decomposition

```
Monolithic Application
├─ User Management (20%)
├─ Order Processing (30%)
├─ Payment (15%)
├─ Inventory (20%)
└─ Notifications (15%)

Becomes:

User Service (Node.js)       Order Service (Python)
├─ Authentication            ├─ Order Creation
├─ Profile Management        ├─ Order Management
└─ Authorization             └─ Status Tracking

Payment Service (Go)         Notification Service (Node.js)
├─ Payment Processing        ├─ Email
├─ Refunds                   ├─ SMS
└─ Reconciliation            └─ Push Notifications
```

### Service Communication

#### Synchronous (REST/gRPC)

```typescript
// Order Service calls Payment Service
async function processPayment(orderId: string, amount: number) {
  const response = await fetch('http://payment-service/charge', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ amount, orderId }),
  });

  if (!response.ok) {
    throw new Error('Payment failed');
  }

  return response.json();
}
```

#### Asynchronous (Message Queue)

```javascript
// Order Service publishes event
await messageQueue.publish('order.created', {
  orderId: 123,
  amount: 99.99,
  timestamp: new Date(),
});

// Payment Service subscribes
messageQueue.subscribe('order.created', async (event) => {
  await chargePayment(event.amount, event.orderId);
});

// Inventory Service subscribes
messageQueue.subscribe('order.created', async (event) => {
  await reserveInventory(event.orderId);
});
```

## Event-Driven Architecture

### Event Sourcing

```javascript
class OrderAggregate {
  constructor(id) {
    this.id = id;
    this.events = [];
    this.state = { status: 'pending', items: [] };
  }

  createOrder(items) {
    this.applyEvent({
      type: 'OrderCreated',
      id: this.id,
      items,
      timestamp: new Date(),
    });
  }

  applyEvent(event) {
    this.events.push(event);

    switch (event.type) {
      case 'OrderCreated':
        this.state.status = 'created';
        this.state.items = event.items;
        break;
      case 'OrderPaid':
        this.state.status = 'paid';
        break;
      case 'OrderShipped':
        this.state.status = 'shipped';
        break;
    }
  }

  getUncommittedEvents() {
    return this.events;
  }
}
```

### CQRS Pattern

```javascript
// Command Handler (Write side)
async function handleCreateOrder(command) {
  const order = new OrderAggregate(command.id);
  order.createOrder(command.items);

  await eventStore.append(order.id, order.getUncommittedEvents());
  await eventBus.publish('order.created', command);

  return order.id;
}

// Query Handler (Read side - optimized read model)
async function getOrderDetails(orderId) {
  return orderReadModel.findById(orderId);
}

// Projection - Update read model from events
eventBus.subscribe('order.*', async (event) => {
  await orderReadModel.update(event);
});
```

## SAGA Pattern

### Choreography (Event-driven)

```javascript
// Order Service
app.post('/orders', async (req, res) => {
  const order = await createOrder(req.body);
  await eventBus.publish('order.created', order);
  res.json(order);
});

// Payment Service
eventBus.subscribe('order.created', async (order) => {
  try {
    const payment = await processPayment(order.amount);
    await eventBus.publish('payment.completed', {
      orderId: order.id,
      payment,
    });
  } catch (error) {
    await eventBus.publish('payment.failed', {
      orderId: order.id,
      error: error.message,
    });
  }
});

// Inventory Service
eventBus.subscribe('payment.completed', async (event) => {
  try {
    await reserveInventory(event.orderId);
    await eventBus.publish('inventory.reserved', { orderId: event.orderId });
  } catch (error) {
    // Trigger compensation
    await eventBus.publish('inventory.failed', { orderId: event.orderId });
  }
});

// Compensation
eventBus.subscribe('inventory.failed', async (event) => {
  await refundPayment(event.orderId);
});
```

### Orchestration (Centralized)

```javascript
class OrderSaga {
  async execute(order) {
    const steps = [];

    try {
      // Step 1: Payment
      const payment = await this.paymentService.charge(order.amount);
      steps.push({ service: 'payment', id: payment.id });

      // Step 2: Inventory
      const reservation = await this.inventoryService.reserve(order.items);
      steps.push({ service: 'inventory', id: reservation.id });

      // Step 3: Shipment
      const shipment = await this.shipmentService.create(order);
      steps.push({ service: 'shipment', id: shipment.id });

      return { status: 'completed', steps };
    } catch (error) {
      await this.compensate(steps);
      throw error;
    }
  }

  async compensate(steps) {
    for (const step of steps.reverse()) {
      try {
        switch (step.service) {
          case 'payment':
            await this.paymentService.refund(step.id);
            break;
          case 'inventory':
            await this.inventoryService.release(step.id);
            break;
          case 'shipment':
            await this.shipmentService.cancel(step.id);
            break;
        }
      } catch (error) {
        console.error(`Compensation failed for ${step.service}:`, error);
      }
    }
  }
}
```

## Circuit Breaker Pattern

```javascript
class CircuitBreaker {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.state = 'CLOSED';
    this.failureCount = 0;
    this.failureThreshold = options.failureThreshold || 5;
    this.timeout = options.timeout || 60000;
    this.lastFailureTime = null;
  }

  async call(...args) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await this.fn(...args);

      if (this.state === 'HALF_OPEN') {
        this.state = 'CLOSED';
        this.failureCount = 0;
      }

      return result;
    } catch (error) {
      this.failureCount++;
      this.lastFailureTime = Date.now();

      if (this.failureCount >= this.failureThreshold) {
        this.state = 'OPEN';
      }

      throw error;
    }
  }
}

// Usage
const paymentAPI = new CircuitBreaker(
  (data) => fetch('http://payment-service/charge', { method: 'POST', body: JSON.stringify(data) }),
  { failureThreshold: 5, timeout: 60000 }
);
```

## Job Queues (Bull)

```javascript
const Queue = require('bull');

const emailQueue = new Queue('emails', {
  redis: { host: 'localhost', port: 6379 },
});

// Producer
app.post('/send-email', async (req, res) => {
  const job = await emailQueue.add(
    { to: req.body.email, template: 'welcome', data: req.body },
    {
      attempts: 3,
      backoff: { type: 'exponential', delay: 2000 },
      removeOnComplete: true,
    }
  );
  res.json({ jobId: job.id });
});

// Consumer
emailQueue.process(async (job) => {
  const { to, template, data } = job.data;
  await sendEmail(to, template, data);
});

// Error handling
emailQueue.on('failed', (job, error) => {
  console.error(`Job ${job.id} failed:`, error);
});
```

## AI Agent Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

async function processWithAI(data: Record<string, unknown>) {
  const message = await client.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    messages: [{
      role: 'user',
      content: `Analyze this data:\n${JSON.stringify(data)}`
    }]
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// API endpoint
app.post('/analyze', async (req, res) => {
  const analysis = await processWithAI(req.body);
  res.json({ analysis });
});
```

## Scaling Patterns Comparison

| Pattern | Consistency | Complexity | Use Case |
|---------|-------------|------------|----------|
| Sync (REST) | Strong | Low | Simple CRUD |
| Async (Queue) | Eventual | Medium | High throughput |
| SAGA | Eventual | High | Distributed transactions |
| CQRS | Eventual | High | Read-heavy, complex queries |
| Event Sourcing | Strong (per aggregate) | Very High | Audit, temporal queries |

## Checklist

- [ ] Service boundaries defined
- [ ] Communication patterns chosen
- [ ] SAGA implemented for transactions
- [ ] Circuit breakers in place
- [ ] Dead letter queues configured
- [ ] Monitoring configured
- [ ] Distributed tracing enabled
- [ ] Disaster recovery planned

---

## Troubleshooting

### Common Failure Modes

| Error | Root Cause | Recovery |
|-------|------------|----------|
| Circuit open | Service down | Wait for recovery, use fallback |
| SAGA stuck | Compensation failed | Manual intervention, retry |
| Queue backlog | Consumer too slow | Scale consumers, increase resources |
| Event lost | Broker failure | Use persistent queues, replay |

### Debug Checklist

1. [ ] Check circuit breaker states
2. [ ] Review dead letter queues
3. [ ] Verify event ordering
4. [ ] Check service health endpoints
5. [ ] Review distributed traces

### Log Interpretation

```
INFO: SAGA_STARTED saga_id=123 steps=3
  → Saga execution began

WARN: CIRCUIT_OPENED service=payment failures=5
  → Payment service circuit breaker opened

ERROR: SAGA_COMPENSATION_FAILED saga_id=123 step=inventory
  → Manual intervention required

INFO: MESSAGE_PROCESSED queue=orders duration=45ms
  → Message successfully processed
```

---

**You've reached expertise across all 8 domains!**

Use `/architect` to design your complete system.
