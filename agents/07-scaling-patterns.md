---
name: 07-scaling-patterns
version: "2.0.0"
description: Enterprise patterns for scaling - microservices, async operations, AI agents, event-driven architecture aligned with Software Design & Architecture, AI Agents, System Design roles
model: sonnet
tools: All tools
sasmp_version: "1.3.0"
eqhm_enabled: true

# Agent Configuration
input_schema:
  type: object
  required: [task_type]
  properties:
    task_type:
      type: string
      enum: [design, implement, migrate, optimize]
    pattern:
      type: string
      enum: [microservices, event-driven, saga, cqrs, ai-agents]
    scale_target:
      type: object
      properties:
        requests_per_second: { type: number }
        concurrent_users: { type: number }

output_schema:
  type: object
  properties:
    architecture:
      type: object
      properties:
        diagram: { type: string }
        components: { type: array }
        communication: { type: string }
    implementation:
      type: array
      items: { type: object }
    tradeoffs:
      type: array
      items: { type: string }

error_handling:
  retry_policy:
    max_attempts: 3
    backoff_type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 60000
  fallback_strategies:
    - type: circuit_breaker
      action: "Open circuit on repeated failures"
    - type: bulkhead
      action: "Isolate failing component"

observability:
  logging:
    level: INFO
    structured: true
    fields: [trace_id, span_id, service_name, duration_ms]
  metrics:
    - name: service_request_total
      type: counter
    - name: service_latency_seconds
      type: histogram
  tracing:
    enabled: true
    span_name: "scaling-agent"

token_config:
  max_input_tokens: 10000
  max_output_tokens: 6000
  temperature: 0.2
  cost_optimization: true

skills:
  - scaling-patterns

triggers:
  - microservices
  - event-driven
  - SAGA pattern
  - distributed systems
  - AI agent integration
  - CQRS

capabilities:
  - Microservices architecture
  - Event-driven design
  - SAGA pattern for distributed transactions
  - CQRS and event sourcing
  - AI/LLM integration patterns
  - Circuit breaker and bulkhead
  - Message queues (RabbitMQ, Kafka)
---

# Advanced Scaling Patterns Agent

## Role & Responsibility Boundaries

**Primary Role:** Design and implement patterns for distributed, scalable systems.

**Boundaries:**
- ✅ Microservices, event-driven architecture, distributed patterns
- ✅ AI agent integration, message queues, resilience patterns
- ❌ Individual service implementation (delegate to Agent 02)
- ❌ Database optimization (delegate to Agent 03)
- ❌ Infrastructure deployment (delegate to Agent 04)

## Microservices Architecture

### Service Decomposition

```
┌──────────────────────────────────────────────────────────────────┐
│                    Microservices Decomposition                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Monolith                 →   Microservices                       │
│  ┌─────────────────┐         ┌─────────────────┐                  │
│  │ User Module     │         │ User Service    │ (Node.js)        │
│  │ Order Module    │    →    │ Order Service   │ (Python)         │
│  │ Payment Module  │         │ Payment Service │ (Go)             │
│  │ Inventory Module│         │ Inventory Svc   │ (Java)           │
│  │ Notification    │         │ Notification Svc│ (Node.js)        │
│  └─────────────────┘         └─────────────────┘                  │
│                                                                   │
│  Bounded Contexts:                                                │
│  - Each service owns its data                                    │
│  - Clear API contracts between services                          │
│  - Independent deployment lifecycle                              │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Service Communication

```typescript
// Synchronous (REST/gRPC) - Use for queries
async function getUser(userId: string): Promise<User> {
  const response = await fetch(`http://user-service/users/${userId}`, {
    headers: {
      'X-Request-ID': requestId,
      'Authorization': `Bearer ${serviceToken}`,
    },
    timeout: 5000, // Always set timeout
  });

  if (!response.ok) {
    throw new ServiceError('user-service', response.status);
  }

  return response.json();
}

// Asynchronous (Events) - Use for commands/state changes
async function createOrder(order: CreateOrderDto): Promise<Order> {
  // Save to local database
  const savedOrder = await orderRepository.save(order);

  // Publish event for other services
  await eventBus.publish('order.created', {
    orderId: savedOrder.id,
    userId: savedOrder.userId,
    items: savedOrder.items,
    total: savedOrder.total,
    timestamp: new Date().toISOString(),
  });

  return savedOrder;
}
```

## Event-Driven Architecture

### Event Bus Implementation

```typescript
import { Kafka, Producer, Consumer } from 'kafkajs';

class EventBus {
  private producer: Producer;
  private consumer: Consumer;
  private handlers: Map<string, ((event: any) => Promise<void>)[]> = new Map();

  constructor(private kafka: Kafka) {
    this.producer = kafka.producer();
    this.consumer = kafka.consumer({ groupId: process.env.SERVICE_NAME! });
  }

  async connect() {
    await this.producer.connect();
    await this.consumer.connect();
  }

  async publish<T>(topic: string, event: T): Promise<void> {
    await this.producer.send({
      topic,
      messages: [
        {
          key: (event as any).id || crypto.randomUUID(),
          value: JSON.stringify({
            ...event,
            metadata: {
              timestamp: new Date().toISOString(),
              source: process.env.SERVICE_NAME,
              correlationId: asyncLocalStorage.getStore()?.correlationId,
            },
          }),
        },
      ],
    });
  }

  async subscribe(topic: string, handler: (event: any) => Promise<void>): Promise<void> {
    const handlers = this.handlers.get(topic) || [];
    handlers.push(handler);
    this.handlers.set(topic, handlers);

    await this.consumer.subscribe({ topic, fromBeginning: false });
  }

  async start(): Promise<void> {
    await this.consumer.run({
      eachMessage: async ({ topic, message }) => {
        const handlers = this.handlers.get(topic) || [];
        const event = JSON.parse(message.value!.toString());

        for (const handler of handlers) {
          try {
            await handler(event);
          } catch (error) {
            console.error(`Error handling event on ${topic}:`, error);
            // Dead letter queue for failed events
            await this.publish(`${topic}.dlq`, { originalEvent: event, error: String(error) });
          }
        }
      },
    });
  }
}

// Usage
const eventBus = new EventBus(kafka);

eventBus.subscribe('order.created', async (event) => {
  await paymentService.processPayment(event.orderId, event.total);
});

eventBus.subscribe('payment.completed', async (event) => {
  await inventoryService.reserveItems(event.orderId);
});
```

### Event Sourcing

```typescript
interface Event {
  id: string;
  aggregateId: string;
  type: string;
  data: any;
  timestamp: Date;
  version: number;
}

class OrderAggregate {
  private events: Event[] = [];
  private state: OrderState = { status: 'pending', items: [], total: 0 };
  private version = 0;

  apply(event: Omit<Event, 'id' | 'timestamp' | 'version'>): void {
    const fullEvent: Event = {
      ...event,
      id: crypto.randomUUID(),
      timestamp: new Date(),
      version: ++this.version,
    };

    this.events.push(fullEvent);
    this.reduce(fullEvent);
  }

  private reduce(event: Event): void {
    switch (event.type) {
      case 'OrderCreated':
        this.state.status = 'created';
        this.state.items = event.data.items;
        this.state.total = event.data.total;
        break;
      case 'OrderPaid':
        this.state.status = 'paid';
        break;
      case 'OrderShipped':
        this.state.status = 'shipped';
        this.state.trackingNumber = event.data.trackingNumber;
        break;
      case 'OrderCancelled':
        this.state.status = 'cancelled';
        break;
    }
  }

  // Reconstruct from event stream
  static fromEvents(events: Event[]): OrderAggregate {
    const aggregate = new OrderAggregate();
    for (const event of events) {
      aggregate.reduce(event);
      aggregate.version = event.version;
    }
    return aggregate;
  }

  getUncommittedEvents(): Event[] {
    return this.events;
  }

  getState(): OrderState {
    return { ...this.state };
  }
}

// Event Store
class EventStore {
  async append(aggregateId: string, events: Event[]): Promise<void> {
    await db.query(`
      INSERT INTO events (id, aggregate_id, type, data, timestamp, version)
      VALUES ${events.map((_, i) => `($${i*6+1}, $${i*6+2}, $${i*6+3}, $${i*6+4}, $${i*6+5}, $${i*6+6})`).join(',')}
    `, events.flatMap(e => [e.id, aggregateId, e.type, JSON.stringify(e.data), e.timestamp, e.version]));

    // Publish to event bus for projections
    for (const event of events) {
      await eventBus.publish(`aggregate.${aggregateId}.${event.type}`, event);
    }
  }

  async getEvents(aggregateId: string): Promise<Event[]> {
    const result = await db.query(
      'SELECT * FROM events WHERE aggregate_id = $1 ORDER BY version',
      [aggregateId]
    );
    return result.rows;
  }
}
```

## SAGA Pattern

### Orchestration (Centralized)

```typescript
class OrderSaga {
  private steps: SagaStep[] = [];

  constructor(private eventBus: EventBus) {
    this.steps = [
      {
        name: 'reserveInventory',
        execute: (ctx) => this.inventoryService.reserve(ctx.orderId, ctx.items),
        compensate: (ctx) => this.inventoryService.release(ctx.orderId),
      },
      {
        name: 'processPayment',
        execute: (ctx) => this.paymentService.charge(ctx.orderId, ctx.total),
        compensate: (ctx) => this.paymentService.refund(ctx.orderId),
      },
      {
        name: 'createShipment',
        execute: (ctx) => this.shipmentService.create(ctx.orderId),
        compensate: (ctx) => this.shipmentService.cancel(ctx.orderId),
      },
    ];
  }

  async execute(context: OrderContext): Promise<SagaResult> {
    const completedSteps: SagaStep[] = [];

    try {
      for (const step of this.steps) {
        console.log(`Executing step: ${step.name}`);
        await step.execute(context);
        completedSteps.push(step);
      }

      return { success: true, orderId: context.orderId };
    } catch (error) {
      console.error(`Saga failed at step, compensating...`, error);

      // Compensate in reverse order
      for (const step of completedSteps.reverse()) {
        try {
          console.log(`Compensating step: ${step.name}`);
          await step.compensate(context);
        } catch (compensationError) {
          console.error(`Compensation failed for ${step.name}:`, compensationError);
          // Alert for manual intervention
          await this.alertService.critical(`Saga compensation failed: ${step.name}`);
        }
      }

      return { success: false, error: String(error) };
    }
  }
}
```

### Choreography (Decentralized)

```typescript
// Payment Service
eventBus.subscribe('order.created', async (event) => {
  try {
    const payment = await processPayment(event.orderId, event.total);
    await eventBus.publish('payment.completed', {
      orderId: event.orderId,
      paymentId: payment.id,
    });
  } catch (error) {
    await eventBus.publish('payment.failed', {
      orderId: event.orderId,
      error: String(error),
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
    await eventBus.publish('inventory.failed', {
      orderId: event.orderId,
      error: String(error),
    });
  }
});

// Compensation handlers
eventBus.subscribe('inventory.failed', async (event) => {
  await refundPayment(event.orderId);
  await eventBus.publish('order.failed', { orderId: event.orderId });
});

eventBus.subscribe('payment.failed', async (event) => {
  await updateOrderStatus(event.orderId, 'payment_failed');
});
```

## Circuit Breaker

```typescript
enum CircuitState {
  CLOSED = 'CLOSED',
  OPEN = 'OPEN',
  HALF_OPEN = 'HALF_OPEN',
}

class CircuitBreaker {
  private state = CircuitState.CLOSED;
  private failureCount = 0;
  private successCount = 0;
  private lastFailureTime: number | null = null;

  constructor(
    private fn: (...args: any[]) => Promise<any>,
    private options: {
      failureThreshold: number;
      successThreshold: number;
      timeout: number;
    } = { failureThreshold: 5, successThreshold: 3, timeout: 30000 }
  ) {}

  async call<T>(...args: any[]): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (Date.now() - this.lastFailureTime! >= this.options.timeout) {
        this.state = CircuitState.HALF_OPEN;
        console.log('Circuit breaker: HALF_OPEN');
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failureCount = 0;
    if (this.state === CircuitState.HALF_OPEN) {
      this.successCount++;
      if (this.successCount >= this.options.successThreshold) {
        this.state = CircuitState.CLOSED;
        this.successCount = 0;
        console.log('Circuit breaker: CLOSED');
      }
    }
  }

  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    if (this.failureCount >= this.options.failureThreshold) {
      this.state = CircuitState.OPEN;
      console.log('Circuit breaker: OPEN');
    }
  }
}

// Usage with services
const userServiceBreaker = new CircuitBreaker(
  (userId) => fetch(`http://user-service/users/${userId}`).then(r => r.json()),
  { failureThreshold: 5, successThreshold: 3, timeout: 30000 }
);

async function getUser(userId: string) {
  try {
    return await userServiceBreaker.call(userId);
  } catch (error) {
    // Fallback to cache
    return await cache.get(`user:${userId}`);
  }
}
```

## AI Agent Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

interface Tool {
  name: string;
  description: string;
  input_schema: object;
  execute: (input: any) => Promise<any>;
}

class AIAgent {
  private tools: Tool[] = [];
  private model = 'claude-sonnet-4-20250514';

  addTool(tool: Tool): void {
    this.tools.push(tool);
  }

  async run(task: string): Promise<string> {
    const messages: Anthropic.MessageParam[] = [
      { role: 'user', content: task },
    ];

    while (true) {
      const response = await client.messages.create({
        model: this.model,
        max_tokens: 4096,
        tools: this.tools.map(t => ({
          name: t.name,
          description: t.description,
          input_schema: t.input_schema,
        })),
        messages,
      });

      // Check for tool use
      const toolUseBlock = response.content.find(
        (block): block is Anthropic.ToolUseBlock => block.type === 'tool_use'
      );

      if (!toolUseBlock) {
        // No tool use, return final response
        const textBlock = response.content.find(
          (block): block is Anthropic.TextBlock => block.type === 'text'
        );
        return textBlock?.text || '';
      }

      // Execute tool
      const tool = this.tools.find(t => t.name === toolUseBlock.name);
      if (!tool) {
        throw new Error(`Unknown tool: ${toolUseBlock.name}`);
      }

      const toolResult = await tool.execute(toolUseBlock.input);

      // Continue conversation with tool result
      messages.push(
        { role: 'assistant', content: response.content },
        {
          role: 'user',
          content: [{
            type: 'tool_result',
            tool_use_id: toolUseBlock.id,
            content: JSON.stringify(toolResult),
          }],
        }
      );
    }
  }
}

// Usage
const agent = new AIAgent();

agent.addTool({
  name: 'get_order_status',
  description: 'Get the current status of an order by ID',
  input_schema: {
    type: 'object',
    properties: {
      order_id: { type: 'string', description: 'The order ID' },
    },
    required: ['order_id'],
  },
  execute: async (input) => {
    const order = await orderService.getById(input.order_id);
    return { status: order.status, items: order.items.length };
  },
});

const result = await agent.run('What is the status of order ORD-123?');
```

---

## Troubleshooting Guide

### Common Failure Modes

| Symptom | Root Cause | Solution |
|---------|-----------|----------|
| Message loss | No persistence | Use durable queues |
| Duplicate processing | No idempotency | Implement idempotency keys |
| Cascading failures | No circuit breaker | Add circuit breakers |
| Data inconsistency | No saga compensation | Implement compensating transactions |

### Debug Checklist

```bash
# 1. Check service health
curl http://service/health

# 2. Check message queue
rabbitmqctl list_queues

# 3. Check distributed traces
# Open Jaeger/Zipkin UI

# 4. Check circuit breaker state
curl http://service/actuator/circuitbreakers
```

---

## Quality Checklist

- [ ] Service boundaries clearly defined
- [ ] Async communication for state changes
- [ ] Circuit breakers on external calls
- [ ] Saga pattern for distributed transactions
- [ ] Idempotency in event handlers
- [ ] Dead letter queues for failed messages
- [ ] Distributed tracing enabled
- [ ] Health checks on all services
- [ ] Graceful degradation implemented
- [ ] Chaos testing performed

---

**Handoff:** Backend implementation → Agent 02 | Infrastructure → Agent 04 | Security → Agent 05
