---
name: roadmap
description: Get Implementation Roadmap
allowed-tools: Read, Glob, Grep
sasmp_version: "2.0.0"

# Production-Grade Metadata
input_validation:
  required: [architecture]
  optional: [team_size, timeline, current_progress]
  schema:
    architecture:
      type: string
      description: Architecture type or description
    team_size:
      type: integer
      description: Number of developers
    timeline:
      type: string
      description: Target completion
    current_progress:
      type: string
      description: What's already built

exit_codes:
  0: Roadmap generated
  1: Insufficient information
  2: Conflicting requirements
---

# /roadmap - Get Implementation Roadmap

Get a step-by-step implementation plan.

## What to Share

- Your architecture (from /architect or existing)
- Current progress (if any)
- Team size and capabilities
- Timeline and constraints

## Usage Examples

```
/roadmap microservices team_size=3
/roadmap monolith timeline=3months
/roadmap existing current_progress=50%
```

## What You'll Get

- **Prioritized Tasks** - Ordered by importance
- **Team Assignments** - Role-based distribution
- **Risk Mitigation** - Potential blockers
- **Testing Strategy** - Quality assurance plan

## Output Structure

```
1. Executive Summary
   └─ Total phases: N
   └─ Critical path items
   └─ Key milestones

2. Implementation Phases

   Phase 1: Foundation
   └─ Task 1.1: Setup project structure
   └─ Task 1.2: Configure CI/CD
   └─ Task 1.3: Implement core API

   Phase 2: Core Features
   └─ Task 2.1: User authentication
   └─ Task 2.2: Resource endpoints
   └─ Task 2.3: Database layer

   Phase 3: Advanced Features
   └─ Task 3.1: Plugin system
   └─ Task 3.2: Webhooks
   └─ Task 3.3: Rate limiting

   Phase 4: Production Ready
   └─ Task 4.1: Load testing
   └─ Task 4.2: Security audit
   └─ Task 4.3: Documentation

3. Risk Register
   └─ Risk: Scope creep
   └─ Mitigation: Sprint boundaries
   └─ Contingency: Feature prioritization

4. Success Metrics
   └─ API latency < 200ms
   └─ Uptime > 99.9%
   └─ Test coverage > 80%
```
