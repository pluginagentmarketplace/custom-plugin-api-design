---
name: scaling-patterns
description: Enterprise scaling patterns for microservices, event-driven architecture, and distributed systems
sasmp_version: "2.0.0"
bonded_agent: 07-scaling-patterns
bond_type: PRIMARY_BOND

# Production-Grade Metadata
parameters:
  - name: pattern_type
    type: string
    required: true
    validation: "^(microservices|event-driven|saga|cqrs|circuit-breaker)$"
    description: Scaling pattern type
  - name: consistency
    type: string
    required: false
    validation: "^(strong|eventual)$"
    description: Consistency requirement

validation_rules:
  - Service boundaries must be clearly defined
  - Circuit breakers must have fallbacks
  - SAGAs must have compensation logic

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [service_latency, circuit_breaker_state, saga_completion_rate]
  distributed_tracing: true
---

# Scaling Patterns Skill

## Quick Start

Advanced patterns for building scalable, distributed systems.

### Microservices
- Service decomposition
- Communication patterns
- Service discovery

### Event-Driven Architecture
- Event sourcing
- CQRS pattern
- Message queues

### Distributed Patterns
- SAGA for transactions
- Circuit breaker
- Bulkhead isolation

### AI Agent Integration
- LLM API patterns
- Agentic workflows
- Tool use integration

## Key Patterns

### Circuit Breaker
```javascript
class CircuitBreaker {
  async call(...args) {
    if (this.state === 'OPEN') {
      throw new Error('Circuit breaker is OPEN');
    }
    try {
      const result = await this.fn(...args);
      this.failureCount = 0;
      return result;
    } catch (error) {
      this.failureCount++;
      if (this.failureCount >= this.threshold) {
        this.state = 'OPEN';
      }
      throw error;
    }
  }
}
```

### SAGA Orchestration
```javascript
class OrderSaga {
  async execute(order) {
    try {
      await this.paymentService.charge(order.amount);
      await this.inventoryService.reserve(order.items);
      return { status: 'completed' };
    } catch (error) {
      await this.compensate();
      throw error;
    }
  }
}
```

## Scaling Checklist

- [ ] Service boundaries defined
- [ ] Communication patterns chosen
- [ ] SAGA implemented for transactions
- [ ] Circuit breakers in place
- [ ] Monitoring configured

## Unit Test Template

```javascript
describe('scaling-patterns skill', () => {
  test('circuit breaker opens after threshold', async () => {
    const cb = new CircuitBreaker(failingFn, { threshold: 3 });
    for (let i = 0; i < 3; i++) {
      await expect(cb.call()).rejects.toThrow();
    }
    expect(cb.state).toBe('OPEN');
  });

  test('saga compensates on failure', async () => {
    const saga = new OrderSaga();
    await expect(saga.execute(invalidOrder)).rejects.toThrow();
    expect(mockCompensate).toHaveBeenCalled();
  });
});
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Circuit open | Service down | Wait for recovery, use fallback |
| SAGA stuck | Compensation failed | Manual intervention |
| Queue backlog | Consumer too slow | Scale consumers |

See Agent 7: Scaling Patterns for detailed guidance.
