---
name: scaling-patterns
version: "2.0.0"
description: Enterprise scaling patterns for microservices, event-driven architecture, and distributed systems
sasmp_version: "1.3.0"
bonded_agent: 07-scaling-patterns
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "Distributed systems and scaling architecture"
  boundaries:
    includes: [microservices, event_driven, saga, circuit_breaker, cqrs, ai_agents]
    excludes: [infrastructure, database_optimization, security]

parameter_validation:
  schema:
    type: object
    properties:
      pattern:
        type: string
        enum: [saga, circuit_breaker, cqrs, event_sourcing, bulkhead]
      scale:
        type: string
        enum: [startup, growth, enterprise]
      communication:
        type: string
        enum: [sync, async, hybrid]

retry_config:
  enabled: true
  max_attempts: 5
  backoff:
    type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 60000
    jitter: true

logging:
  level: INFO
  fields: [pattern, service, correlation_id, duration_ms]

dependencies:
  skills: []
  agents: [07-scaling-patterns]
---

# Scaling Patterns Skill

## Purpose
Design and implement scalable distributed systems with proven patterns.

## Microservices Architecture

### Service Decomposition

```
┌─────────────────────────────────────────────────────────────────┐
│                        API Gateway                               │
│  (Authentication, Rate Limiting, Routing, Load Balancing)       │
└─────────────────────────┬───────────────────────────────────────┘
                          │
         ┌────────────────┼────────────────┐
         │                │                │
    ┌────▼────┐     ┌────▼────┐     ┌────▼────┐
    │  User   │     │  Order  │     │ Product │
    │ Service │     │ Service │     │ Service │
    └────┬────┘     └────┬────┘     └────┬────┘
         │               │                │
    ┌────▼────┐     ┌────▼────┐     ┌────▼────┐
    │ User DB │     │Order DB │     │Prod. DB │
    └─────────┘     └─────────┘     └─────────┘
```

### Service Communication

```typescript
// Synchronous (REST/gRPC)
class OrderService {
  constructor(
    private userClient: UserServiceClient,
    private productClient: ProductServiceClient,
  ) {}

  async createOrder(userId: string, productId: string) {
    // Validate user exists
    const user = await this.userClient.getUser(userId);
    if (!user) throw new NotFoundError('User not found');

    // Check product availability
    const product = await this.productClient.getProduct(productId);
    if (!product.available) throw new ConflictError('Product unavailable');

    return this.orderRepository.create({ userId, productId });
  }
}

// Asynchronous (Event-driven)
class OrderService {
  async createOrder(userId: string, productId: string) {
    const order = await this.orderRepository.create({
      userId,
      productId,
      status: 'pending',
    });

    // Emit event instead of sync call
    await this.eventBus.publish('order.created', {
      orderId: order.id,
      userId,
      productId,
      timestamp: new Date(),
    });

    return order;
  }
}
```

## SAGA Pattern

### Choreography-based SAGA

```typescript
// Event handlers in each service

// Order Service
class OrderEventHandler {
  @EventHandler('order.created')
  async onOrderCreated(event: OrderCreatedEvent) {
    // Emit next event in chain
    await this.eventBus.publish('payment.requested', {
      orderId: event.orderId,
      amount: event.total,
    });
  }

  @EventHandler('payment.failed')
  async onPaymentFailed(event: PaymentFailedEvent) {
    // Compensating transaction
    await this.orderRepository.update(event.orderId, { status: 'cancelled' });
    await this.eventBus.publish('order.cancelled', { orderId: event.orderId });
  }
}

// Payment Service
class PaymentEventHandler {
  @EventHandler('payment.requested')
  async onPaymentRequested(event: PaymentRequestedEvent) {
    try {
      await this.paymentGateway.charge(event.amount);
      await this.eventBus.publish('payment.completed', {
        orderId: event.orderId,
      });
    } catch (error) {
      await this.eventBus.publish('payment.failed', {
        orderId: event.orderId,
        reason: error.message,
      });
    }
  }
}
```

### Orchestration-based SAGA

```typescript
class OrderSaga {
  private steps: SagaStep[] = [
    {
      name: 'reserveInventory',
      execute: (ctx) => this.inventoryService.reserve(ctx.productId, ctx.quantity),
      compensate: (ctx) => this.inventoryService.release(ctx.productId, ctx.quantity),
    },
    {
      name: 'processPayment',
      execute: (ctx) => this.paymentService.charge(ctx.userId, ctx.amount),
      compensate: (ctx) => this.paymentService.refund(ctx.paymentId),
    },
    {
      name: 'createShipment',
      execute: (ctx) => this.shippingService.create(ctx.orderId, ctx.address),
      compensate: (ctx) => this.shippingService.cancel(ctx.shipmentId),
    },
  ];

  async execute(context: OrderContext) {
    const completedSteps: SagaStep[] = [];

    for (const step of this.steps) {
      try {
        const result = await step.execute(context);
        context = { ...context, ...result };
        completedSteps.push(step);
      } catch (error) {
        // Rollback in reverse order
        for (const completedStep of completedSteps.reverse()) {
          await completedStep.compensate(context);
        }
        throw new SagaExecutionError(step.name, error);
      }
    }

    return context;
  }
}
```

## Circuit Breaker Pattern

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
  private lastFailureTime?: Date;

  constructor(
    private readonly options: {
      failureThreshold: number;      // Failures before opening
      successThreshold: number;      // Successes in half-open to close
      timeout: number;               // Time in open state before half-open
    }
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (this.shouldAttemptReset()) {
        this.state = CircuitState.HALF_OPEN;
      } else {
        throw new CircuitBreakerOpenError();
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failureCount = 0;
    if (this.state === CircuitState.HALF_OPEN) {
      this.successCount++;
      if (this.successCount >= this.options.successThreshold) {
        this.state = CircuitState.CLOSED;
        this.successCount = 0;
      }
    }
  }

  private onFailure() {
    this.failureCount++;
    this.lastFailureTime = new Date();
    if (this.failureCount >= this.options.failureThreshold) {
      this.state = CircuitState.OPEN;
    }
  }

  private shouldAttemptReset(): boolean {
    if (!this.lastFailureTime) return false;
    const elapsed = Date.now() - this.lastFailureTime.getTime();
    return elapsed >= this.options.timeout;
  }
}

// Usage
const userServiceBreaker = new CircuitBreaker({
  failureThreshold: 5,
  successThreshold: 3,
  timeout: 30000,
});

async function getUser(id: string) {
  return userServiceBreaker.execute(() => userService.get(id));
}
```

## Event Sourcing & CQRS

```typescript
// Event Store
interface DomainEvent {
  id: string;
  aggregateId: string;
  aggregateType: string;
  eventType: string;
  payload: unknown;
  timestamp: Date;
  version: number;
}

class EventStore {
  async append(events: DomainEvent[]): Promise<void> {
    await this.db.transaction(async (tx) => {
      for (const event of events) {
        await tx.insert('events', event);
      }
    });

    // Publish to message bus
    for (const event of events) {
      await this.messageBus.publish(event.eventType, event);
    }
  }

  async getEvents(aggregateId: string, fromVersion = 0): Promise<DomainEvent[]> {
    return this.db.query(
      'SELECT * FROM events WHERE aggregate_id = ? AND version > ? ORDER BY version',
      [aggregateId, fromVersion]
    );
  }
}

// Aggregate
class OrderAggregate {
  private events: DomainEvent[] = [];
  private state: OrderState = { status: 'draft', items: [] };

  static async load(eventStore: EventStore, id: string): Promise<OrderAggregate> {
    const aggregate = new OrderAggregate();
    const events = await eventStore.getEvents(id);
    events.forEach((e) => aggregate.apply(e, false));
    return aggregate;
  }

  addItem(productId: string, quantity: number): void {
    this.apply({
      eventType: 'ItemAdded',
      payload: { productId, quantity },
    });
  }

  submit(): void {
    if (this.state.items.length === 0) {
      throw new Error('Cannot submit empty order');
    }
    this.apply({ eventType: 'OrderSubmitted', payload: {} });
  }

  private apply(event: Partial<DomainEvent>, isNew = true): void {
    // Update state based on event
    switch (event.eventType) {
      case 'ItemAdded':
        this.state.items.push(event.payload);
        break;
      case 'OrderSubmitted':
        this.state.status = 'submitted';
        break;
    }

    if (isNew) {
      this.events.push(event as DomainEvent);
    }
  }

  getUncommittedEvents(): DomainEvent[] {
    return this.events;
  }
}

// CQRS: Separate read model
class OrderReadModel {
  async project(event: DomainEvent): Promise<void> {
    switch (event.eventType) {
      case 'OrderSubmitted':
        await this.db.query(
          'UPDATE orders_view SET status = ?, submitted_at = ? WHERE id = ?',
          ['submitted', event.timestamp, event.aggregateId]
        );
        break;
    }
  }
}
```

## AI Agent Integration Patterns

```typescript
// Tool-use pattern for AI agents
interface Tool {
  name: string;
  description: string;
  parameters: JSONSchema;
  execute: (params: unknown) => Promise<unknown>;
}

class AIAgentService {
  private tools: Map<string, Tool> = new Map();

  registerTool(tool: Tool): void {
    this.tools.set(tool.name, tool);
  }

  async executeWorkflow(prompt: string): Promise<AgentResponse> {
    const messages: Message[] = [{ role: 'user', content: prompt }];

    while (true) {
      const response = await this.llm.chat({
        messages,
        tools: Array.from(this.tools.values()),
      });

      if (response.stopReason === 'end_turn') {
        return { result: response.content, steps: messages };
      }

      if (response.stopReason === 'tool_use') {
        const toolCalls = response.toolCalls;
        const results = await Promise.all(
          toolCalls.map(async (call) => {
            const tool = this.tools.get(call.name);
            if (!tool) throw new Error(`Unknown tool: ${call.name}`);
            return {
              toolCallId: call.id,
              result: await tool.execute(call.parameters),
            };
          })
        );

        messages.push(
          { role: 'assistant', content: response.content, toolCalls },
          { role: 'tool', toolResults: results }
        );
      }
    }
  }
}

// Register tools
agentService.registerTool({
  name: 'search_orders',
  description: 'Search orders by customer or status',
  parameters: {
    type: 'object',
    properties: {
      customerId: { type: 'string' },
      status: { type: 'string', enum: ['pending', 'shipped', 'delivered'] },
    },
  },
  execute: async (params) => orderService.search(params),
});
```

---

## Unit Test Template

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

describe('Scaling Patterns', () => {
  describe('Circuit Breaker', () => {
    let breaker: CircuitBreaker;

    beforeEach(() => {
      breaker = new CircuitBreaker({
        failureThreshold: 3,
        successThreshold: 2,
        timeout: 1000,
      });
    });

    it('should open after failure threshold', async () => {
      const failingFn = vi.fn().mockRejectedValue(new Error('fail'));

      for (let i = 0; i < 3; i++) {
        await expect(breaker.execute(failingFn)).rejects.toThrow();
      }

      await expect(breaker.execute(failingFn)).rejects.toThrow(CircuitBreakerOpenError);
    });

    it('should close after success threshold in half-open', async () => {
      // Open the circuit
      const failingFn = vi.fn().mockRejectedValue(new Error('fail'));
      for (let i = 0; i < 3; i++) {
        await expect(breaker.execute(failingFn)).rejects.toThrow();
      }

      // Wait for timeout
      await new Promise((r) => setTimeout(r, 1100));

      // Succeed twice to close
      const successFn = vi.fn().mockResolvedValue('ok');
      await breaker.execute(successFn);
      await breaker.execute(successFn);

      expect(breaker.state).toBe(CircuitState.CLOSED);
    });
  });

  describe('SAGA', () => {
    it('should rollback on failure', async () => {
      const saga = new OrderSaga();
      const compensateSpy = vi.fn();

      saga.steps[1].compensate = compensateSpy;
      saga.steps[2].execute = vi.fn().mockRejectedValue(new Error('fail'));

      await expect(saga.execute({})).rejects.toThrow(SagaExecutionError);
      expect(compensateSpy).toHaveBeenCalled();
    });
  });

  describe('Event Sourcing', () => {
    it('should rebuild state from events', async () => {
      const events = [
        { eventType: 'ItemAdded', payload: { productId: '1', quantity: 2 } },
        { eventType: 'ItemAdded', payload: { productId: '2', quantity: 1 } },
        { eventType: 'OrderSubmitted', payload: {} },
      ];

      const aggregate = OrderAggregate.fromEvents(events);

      expect(aggregate.state.items).toHaveLength(2);
      expect(aggregate.state.status).toBe('submitted');
    });
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Cascading failures | No circuit breaker | Implement circuit breaker pattern |
| Data inconsistency | Distributed transaction | Use SAGA with compensation |
| Event ordering issues | Async processing | Use sequence numbers, idempotency |
| Slow event replay | Too many events | Use snapshots |
| Lost messages | No retry mechanism | Use durable queues, dead-letter |

---

## Quality Checklist

- [ ] Service boundaries clearly defined
- [ ] Communication patterns chosen (sync/async)
- [ ] SAGA implemented for distributed transactions
- [ ] Circuit breakers in place
- [ ] Event sourcing with snapshots (if needed)
- [ ] Idempotency for all handlers
- [ ] Distributed tracing configured
- [ ] Dead-letter queues set up
- [ ] Monitoring and alerting
- [ ] Chaos engineering tests
