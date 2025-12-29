---
name: 07-scaling-patterns
description: Enterprise patterns for scaling - microservices, async operations, AI agents, event-driven architecture aligned with Software Design & Architecture, AI Agents, System Design roles
model: sonnet
tools: All tools
sasmp_version: "1.3.0"
eqhm_enabled: true
skills:
  - scaling-patterns
triggers:
  - microservices
  - event-driven
  - SAGA pattern
  - distributed systems
  - AI agent integration
capabilities:
  - Microservices
  - Event-driven architecture
  - Async patterns
  - AI agent integration
  - SAGA pattern
  - Event sourcing
  - Distributed systems
---

# Advanced Scaling Patterns & Enterprise Design

## Microservices Architecture

### Service Decomposition

```
Monolithic Application
├─ User Management (20% of code)
├─ Order Processing (30% of code)
├─ Payment (15% of code)
├─ Inventory (20% of code)
└─ Notifications (15% of code)

Becomes:

User Service (Node.js)
├─ Authentication
├─ Profile Management
└─ Authorization

Order Service (Python)
├─ Order Creation
├─ Order Management
└─ Status Tracking

Payment Service (Go)
├─ Payment Processing
├─ Refunds
└─ Reconciliation

Inventory Service (Java)
├─ Stock Management
├─ Reservations
└─ Allocations

Notification Service (Node.js)
├─ Email
├─ SMS
└─ Push Notifications
```

### Service Communication

#### Synchronous (REST/gRPC)

```typescript
// Order Service calls Payment Service
async function processPayment(orderId: number, amount: number) {
  const response = await fetch('http://payment-service/charge', {
    method: 'POST',
    body: JSON.stringify({ amount, orderId })
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
  timestamp: new Date()
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
      timestamp: new Date()
    });
  }

  addItem(item) {
    this.applyEvent({
      type: 'ItemAdded',
      orderId: this.id,
      item,
      timestamp: new Date()
    });
  }

  applyEvent(event) {
    this.events.push(event);

    // Update state
    if (event.type === 'OrderCreated') {
      this.state.status = 'created';
      this.state.items = event.items;
    }
  }

  getUncommittedEvents() {
    return this.events;
  }
}
```

### CQRS (Command Query Responsibility Segregation)

```javascript
// Command Handler
async function handleCreateOrder(command) {
  const order = new Order(command.id);
  order.createOrder(command.items);

  // Store events
  await eventStore.append(order.id, order.getUncommittedEvents());

  // Publish for read model update
  await eventBus.publish('order.created', command);

  return order.id;
}

// Query Handler (reads from optimized read model)
async function getOrderDetails(orderId) {
  const order = await orderReadModel.findById(orderId);
  return order;
}

// Update read model from events
eventBus.subscribe('order.*', async (event) => {
  await orderReadModel.update(event);
});
```

## SAGA Pattern for Distributed Transactions

### Choreography (Event-driven)

```javascript
// Order Service
app.post('/orders', async (req, res) => {
  const order = createOrder(req.body);
  await eventBus.publish('order.created', order);
  res.json(order);
});

// Payment Service (subscribes to order.created)
eventBus.subscribe('order.created', async (order) => {
  try {
    const payment = await processPayment(order.amount);
    await eventBus.publish('payment.completed', { orderId: order.id, payment });
  } catch (error) {
    await eventBus.publish('payment.failed', { orderId: order.id, error });
  }
});

// Inventory Service (subscribes to payment.completed)
eventBus.subscribe('payment.completed', async (event) => {
  try {
    await reserveInventory(event.orderId);
    await eventBus.publish('inventory.reserved', { orderId: event.orderId });
  } catch (error) {
    await eventBus.publish('inventory.failed', { orderId: event.orderId });
    // Trigger compensation: refund payment
    await eventBus.publish('refund.requested', { orderId: event.orderId });
  }
});

// Compensate payment failure
eventBus.subscribe('inventory.failed', async (event) => {
  await refundPayment(event.orderId);
});
```

### Orchestration (Centralized)

```javascript
class OrderSaga {
  async executeOrder(order) {
    try {
      // Step 1: Process Payment
      const paymentResult = await this.paymentService.charge(order.amount);
      console.log('Payment succeeded');

      // Step 2: Reserve Inventory
      const inventoryResult = await this.inventoryService.reserve(order.items);
      console.log('Inventory reserved');

      // Step 3: Create Shipment
      const shipmentResult = await this.shipmentService.create(order);
      console.log('Shipment created');

      return { status: 'completed', paymentResult, inventoryResult, shipmentResult };

    } catch (error) {
      // Compensate in reverse order
      await this.compensate(order);
      throw error;
    }
  }

  async compensate(order) {
    try {
      await this.paymentService.refund(order.paymentId);
      await this.inventoryService.release(order.inventoryId);
      await this.shipmentService.cancel(order.shipmentId);
    } catch (error) {
      console.error('Compensation failed:', error);
    }
  }
}
```

## Async Operations with Job Queues

### Bull Queue (Redis-backed)

```javascript
const Queue = require('bull');

// Create queue
const emailQueue = new Queue('emails', {
  redis: { host: 'localhost', port: 6379 }
});

// Enqueue job
app.post('/send-email', async (req, res) => {
  const job = await emailQueue.add(
    {
      to: req.body.email,
      template: 'welcome',
      data: req.body
    },
    {
      attempts: 3,
      backoff: { type: 'exponential', delay: 2000 },
      removeOnComplete: true
    }
  );

  res.json({ jobId: job.id });
});

// Process jobs
emailQueue.process(async (job) => {
  const { to, template, data } = job.data;
  await sendEmail(to, template, data);
});

// Handle failures
emailQueue.on('failed', (job, error) => {
  console.error(`Job ${job.id} failed:`, error);
});
```

## AI Agent Integration

### LLM API Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

async function processOrderWithAI(orderData: Record<string, unknown>) {
  const message = await client.messages.create({
    model: "claude-3-5-sonnet-20241022",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: `Analyze this order and suggest optimizations:\n${JSON.stringify(orderData)}`
      }
    ]
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Use in API
app.post('/orders/analyze', async (req, res) => {
  const analysis = await processOrderWithAI(req.body);
  res.json({ analysis });
});
```

### Agentic Workflows

```typescript
interface Agent {
  name: string;
  goal: string;
  tools: Tool[];
}

interface Tool {
  name: string;
  description: string;
  execute: (args: unknown) => Promise<unknown>;
}

class AutonomousAgent {
  private model = "claude-3-5-sonnet-20241022";
  private tools: Tool[] = [];

  addTool(tool: Tool) {
    this.tools.push(tool);
  }

  async run(task: string) {
    let messages: Array<{ role: string; content: unknown }> = [
      { role: "user", content: task }
    ];

    while (true) {
      const response = await client.messages.create({
        model: this.model,
        max_tokens: 1024,
        tools: this.tools.map(t => ({
          name: t.name,
          description: t.description
        })),
        messages
      });

      // Check if agent wants to use tools
      const toolUse = response.content.find(c => c.type === 'tool_use');

      if (!toolUse) {
        // Agent is done
        return response.content.find(c => c.type === 'text')?.text;
      }

      // Execute tool
      const result = await this.executeTool(toolUse.name, toolUse.input);

      // Continue conversation
      messages = [
        ...messages,
        { role: "assistant", content: response.content },
        { role: "user", content: [{ type: "tool_result", tool_use_id: toolUse.id, content: JSON.stringify(result) }] }
      ];
    }
  }

  private async executeTool(name: string, input: unknown) {
    const tool = this.tools.find(t => t.name === name);
    if (!tool) throw new Error(`Tool not found: ${name}`);
    return tool.execute(input);
  }
}
```

## Circuit Breaker Pattern

```javascript
class CircuitBreaker {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
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

try {
  const data = await externalAPI.call('https://api.example.com/data');
} catch (error) {
  console.error('API call failed:', error);
  // Fallback to cached data
}
```

## Scaling Patterns Checklist

- [ ] Service boundaries identified
- [ ] Communication patterns chosen (sync/async)
- [ ] Event sourcing considered
- [ ] SAGA pattern for transactions
- [ ] Job queues for async work
- [ ] Circuit breaker for resilience
- [ ] Monitoring and observability
- [ ] Error handling and recovery
- [ ] Data consistency strategy
- [ ] Disaster recovery plan

---

**You've reached expertise across all 7 domains!**

API Architecture → Backend Patterns → Database & Performance → DevOps → Security → Frontend → Scaling Patterns

---

*Use the /architect command to design your complete plugin system leveraging all these patterns.*
