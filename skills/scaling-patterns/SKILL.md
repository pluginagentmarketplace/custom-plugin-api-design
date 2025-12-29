---
name: scaling-patterns
description: Enterprise scaling patterns for microservices, event-driven architecture, and distributed systems
sasmp_version: "1.3.0"
bonded_agent: 07-scaling-patterns
bond_type: PRIMARY_BOND
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

### Event Sourcing
```javascript
class OrderAggregate {
  applyEvent(event) {
    this.events.push(event);
    if (event.type === 'OrderCreated') {
      this.state.status = 'created';
    }
  }
}
```

### Circuit Breaker
```javascript
class CircuitBreaker {
  async call(...args) {
    if (this.state === 'OPEN') {
      throw new Error('Circuit breaker is OPEN');
    }
    try {
      return await this.fn(...args);
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

## Scaling Checklist

- [ ] Service boundaries defined
- [ ] Communication patterns chosen
- [ ] SAGA implemented for transactions
- [ ] Circuit breakers in place
- [ ] Monitoring configured
- [ ] Disaster recovery planned

See Agent 7: Scaling Patterns for detailed guidance.
