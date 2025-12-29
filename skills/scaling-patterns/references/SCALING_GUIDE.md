# Scaling Patterns Complete Guide

Enterprise patterns for microservices, event-driven architecture, and AI agents.

## Microservices Architecture

### Service Decomposition
```
Monolithic Application
├── User Management (20%)
├── Order Processing (30%)
├── Payment (15%)
├── Inventory (20%)
└── Notifications (15%)

Becomes Microservices:

User Service (Node.js)
├── Authentication
├── Profile Management
└── Authorization

Order Service (Python)
├── Order Creation
├── Order Management
└── Status Tracking

Payment Service (Go)
├── Payment Processing
├── Refunds
└── Reconciliation
```

### Communication Patterns

**Synchronous (REST/gRPC)**
```typescript
async function processPayment(orderId: number, amount: number) {
  const response = await fetch('http://payment-service/charge', {
    method: 'POST',
    body: JSON.stringify({ amount, orderId })
  });

  if (!response.ok) throw new Error('Payment failed');
  return response.json();
}
```

**Asynchronous (Message Queue)**
```javascript
// Publisher
await messageQueue.publish('order.created', {
  orderId: 123,
  amount: 99.99,
  timestamp: new Date()
});

// Subscriber
messageQueue.subscribe('order.created', async (event) => {
  await chargePayment(event.amount, event.orderId);
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
      timestamp: new Date()
    });
  }

  applyEvent(event) {
    this.events.push(event);

    switch (event.type) {
      case 'OrderCreated':
        this.state.status = 'created';
        this.state.items = event.items;
        break;
      case 'OrderCompleted':
        this.state.status = 'completed';
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
// Command Handler (Write)
async function handleCreateOrder(command) {
  const order = new OrderAggregate(command.id);
  order.createOrder(command.items);

  // Store events
  await eventStore.append(order.id, order.getUncommittedEvents());

  // Publish for read model
  await eventBus.publish('order.created', command);

  return order.id;
}

// Query Handler (Read)
async function getOrderDetails(orderId) {
  return orderReadModel.findById(orderId);
}

// Update read model from events
eventBus.subscribe('order.*', async (event) => {
  await orderReadModel.update(event);
});
```

## SAGA Pattern

### Choreography (Event-driven)
```javascript
// Order Service
app.post('/orders', async (req, res) => {
  const order = createOrder(req.body);
  await eventBus.publish('order.created', order);
  res.json(order);
});

// Payment Service subscribes
eventBus.subscribe('order.created', async (order) => {
  try {
    const payment = await processPayment(order.amount);
    await eventBus.publish('payment.completed', { orderId: order.id });
  } catch (error) {
    await eventBus.publish('payment.failed', { orderId: order.id });
  }
});

// Compensation on failure
eventBus.subscribe('inventory.failed', async (event) => {
  await refundPayment(event.orderId);
});
```

### Orchestration (Centralized)
```javascript
class OrderSaga {
  async executeOrder(order) {
    try {
      // Step 1: Payment
      const payment = await this.paymentService.charge(order.amount);

      // Step 2: Inventory
      const inventory = await this.inventoryService.reserve(order.items);

      // Step 3: Shipment
      const shipment = await this.shipmentService.create(order);

      return { status: 'completed', payment, inventory, shipment };

    } catch (error) {
      await this.compensate(order);
      throw error;
    }
  }

  async compensate(order) {
    await this.paymentService.refund(order.paymentId);
    await this.inventoryService.release(order.inventoryId);
    await this.shipmentService.cancel(order.shipmentId);
  }
}
```

## Resilience Patterns

### Circuit Breaker
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
const externalAPI = new CircuitBreaker(
  async (url) => fetch(url).then(r => r.json()),
  { failureThreshold: 5, timeout: 60000 }
);
```

### Bulkhead Pattern
```javascript
class Bulkhead {
  constructor(maxConcurrent) {
    this.maxConcurrent = maxConcurrent;
    this.currentCount = 0;
    this.queue = [];
  }

  async execute(fn) {
    if (this.currentCount >= this.maxConcurrent) {
      return new Promise((resolve, reject) => {
        this.queue.push({ fn, resolve, reject });
      });
    }

    this.currentCount++;

    try {
      return await fn();
    } finally {
      this.currentCount--;
      this.processQueue();
    }
  }

  processQueue() {
    if (this.queue.length > 0 && this.currentCount < this.maxConcurrent) {
      const { fn, resolve, reject } = this.queue.shift();
      this.execute(fn).then(resolve).catch(reject);
    }
  }
}
```

## AI Agent Integration

### LLM API Pattern
```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

async function processWithAI(data: unknown) {
  const message = await client.messages.create({
    model: "claude-3-5-sonnet-20241022",
    max_tokens: 1024,
    messages: [{
      role: "user",
      content: `Analyze: ${JSON.stringify(data)}`
    }]
  });

  return message.content[0].type === 'text'
    ? message.content[0].text
    : '';
}
```

### Agentic Workflow
```typescript
class AutonomousAgent {
  private tools: Tool[] = [];

  addTool(tool: Tool) {
    this.tools.push(tool);
  }

  async run(task: string) {
    let messages = [{ role: "user", content: task }];

    while (true) {
      const response = await client.messages.create({
        model: "claude-3-5-sonnet-20241022",
        max_tokens: 1024,
        tools: this.tools.map(t => ({
          name: t.name,
          description: t.description
        })),
        messages
      });

      const toolUse = response.content.find(c => c.type === 'tool_use');

      if (!toolUse) {
        return response.content.find(c => c.type === 'text')?.text;
      }

      const result = await this.executeTool(toolUse.name, toolUse.input);

      messages = [
        ...messages,
        { role: "assistant", content: response.content },
        { role: "user", content: [{
          type: "tool_result",
          tool_use_id: toolUse.id,
          content: JSON.stringify(result)
        }]}
      ];
    }
  }
}
```

## Scaling Checklist

### Architecture
- [ ] Service boundaries defined
- [ ] Communication patterns chosen
- [ ] Event schema versioned
- [ ] API contracts documented

### Resilience
- [ ] Circuit breakers implemented
- [ ] Bulkheads configured
- [ ] Retry policies defined
- [ ] Fallbacks in place

### Transactions
- [ ] SAGA pattern for distributed transactions
- [ ] Compensation logic tested
- [ ] Idempotency ensured

### Monitoring
- [ ] Distributed tracing enabled
- [ ] Metrics collection
- [ ] Alerting configured
- [ ] Log aggregation

### Disaster Recovery
- [ ] Backup strategy defined
- [ ] Failover tested
- [ ] RTO/RPO documented

## Resources

- [Microservices Patterns](https://microservices.io/patterns/)
- [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [CQRS](https://martinfowler.com/bliki/CQRS.html)
- [Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)
