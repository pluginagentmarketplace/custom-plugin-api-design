---
name: roadmap
version: "2.0.0"
description: Get Implementation Roadmap
sasmp_version: "1.3.0"
allowed-tools: Read

# Command Configuration
input_schema:
  type: object
  properties:
    architecture:
      type: string
      description: "Current or planned architecture"
    current_progress:
      type: string
      description: "What's already built"
    timeline:
      type: object
      properties:
        deadline: { type: string, format: date }
        flexibility: { type: string, enum: [fixed, flexible, no_deadline] }
    team:
      type: object
      properties:
        size: { type: number }
        skills: { type: array, items: { type: string } }
        availability: { type: string, enum: [full_time, part_time, mixed] }
    constraints:
      type: array
      items:
        type: string

output_schema:
  type: object
  properties:
    phases:
      type: array
      items:
        type: object
        properties:
          name: { type: string }
          duration: { type: string }
          tasks: { type: array }
          deliverables: { type: array }
          risks: { type: array }
    critical_path:
      type: array
    milestones:
      type: array
    resource_allocation:
      type: object

orchestration:
  agents:
    - 01-api-architect       # Architecture tasks
    - 02-backend-patterns    # Implementation patterns
    - 04-devops-infrastructure # Infrastructure tasks
    - 05-security-compliance  # Security milestones
  execution: parallel_then_merge
---

# /roadmap - Implementation Roadmap Generator

Get a detailed, actionable implementation plan tailored to your team and timeline.

## What You'll Get

```
┌─────────────────────────────────────────────────────────────┐
│                Implementation Roadmap                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Timeline: 8 weeks │ Team: 4 engineers                      │
│                                                              │
│  Week 1-2: Foundation ████████░░░░░░░░░░░░░░░░░░ 25%        │
│  Week 3-4: Core API  ░░░░░░░░████████░░░░░░░░░░░░ 25%        │
│  Week 5-6: Security  ░░░░░░░░░░░░░░░░████████░░░░ 25%        │
│  Week 7-8: Launch    ░░░░░░░░░░░░░░░░░░░░░░░░████ 25%        │
│                                                              │
│  Critical Path: Database → API → Auth → Deploy              │
│                                                              │
│  Risk Level: MEDIUM                                          │
│  • Team capacity: 85%                                        │
│  • Technical complexity: Medium                              │
│  • External dependencies: 2                                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## How to Use

Share your project context:

```yaml
Architecture:
  style: "REST API with PostgreSQL"
  components: [API Gateway, User Service, Order Service]

Current Progress:
  completed:
    - Database schema design
    - Basic project setup
  in_progress:
    - User authentication
  not_started:
    - Order management
    - Payment integration

Timeline:
  deadline: "2025-03-15"
  flexibility: "1-2 weeks buffer possible"

Team:
  size: 4
  skills:
    - 2 backend (Node.js, Python)
    - 1 frontend (React)
    - 1 fullstack/devops
  availability: "Full-time"

Constraints:
  - Must integrate with existing payment provider
  - Need to maintain backward compatibility with v1 API
  - Limited DevOps experience on team
```

## Roadmap Output Example

```yaml
Roadmap:
  overview:
    total_duration: "8 weeks"
    team_utilization: "85%"
    risk_level: "medium"

  phases:
    - name: "Phase 1: Foundation"
      duration: "Week 1-2"
      capacity: "4 engineers"

      tasks:
        - id: F1
          title: "Set up development environment"
          assignee: "DevOps Engineer"
          effort: "2 days"
          dependencies: []
          deliverables:
            - Docker Compose for local dev
            - CI/CD pipeline skeleton
            - Environment configuration

        - id: F2
          title: "Database schema implementation"
          assignee: "Backend Engineer 1"
          effort: "3 days"
          dependencies: [F1]
          deliverables:
            - PostgreSQL schema
            - Migrations setup
            - Seed data scripts

        - id: F3
          title: "API project structure"
          assignee: "Backend Engineer 2"
          effort: "2 days"
          dependencies: [F1]
          deliverables:
            - Express/NestJS setup
            - Error handling middleware
            - Logging configuration

      milestone: "Development environment ready, database deployed"
      risks:
        - risk: "Database design changes"
          mitigation: "Early stakeholder review"
          impact: "medium"

    - name: "Phase 2: Core API Development"
      duration: "Week 3-4"
      capacity: "4 engineers"

      tasks:
        - id: C1
          title: "User management API"
          assignee: "Backend Engineer 1"
          effort: "4 days"
          dependencies: [F2, F3]
          deliverables:
            - CRUD endpoints for users
            - Input validation
            - Unit tests (>80% coverage)

        - id: C2
          title: "Order management API"
          assignee: "Backend Engineer 2"
          effort: "5 days"
          dependencies: [F2, F3]
          deliverables:
            - Order lifecycle endpoints
            - State machine implementation
            - Integration tests

        - id: C3
          title: "Payment integration"
          assignee: "Backend Engineer 1"
          effort: "4 days"
          dependencies: [C2]
          deliverables:
            - Payment provider SDK integration
            - Webhook handlers
            - Retry logic

      milestone: "Core API endpoints functional"
      risks:
        - risk: "Payment provider API issues"
          mitigation: "Use sandbox early, have fallback"
          impact: "high"

    - name: "Phase 3: Security & Polish"
      duration: "Week 5-6"
      capacity: "4 engineers"

      tasks:
        - id: S1
          title: "Authentication system"
          assignee: "Backend Engineer 1"
          effort: "4 days"
          dependencies: [C1]
          deliverables:
            - JWT implementation
            - Refresh token rotation
            - Password reset flow

        - id: S2
          title: "Authorization & RBAC"
          assignee: "Backend Engineer 2"
          effort: "3 days"
          dependencies: [S1]
          deliverables:
            - Role-based permissions
            - Resource-level access control
            - Admin endpoints

        - id: S3
          title: "API documentation"
          assignee: "Fullstack Engineer"
          effort: "3 days"
          dependencies: [C1, C2]
          deliverables:
            - OpenAPI 3.1 spec
            - Interactive API docs
            - SDK generation

      milestone: "Security review complete, documentation published"

    - name: "Phase 4: Launch Preparation"
      duration: "Week 7-8"
      capacity: "4 engineers"

      tasks:
        - id: L1
          title: "Production infrastructure"
          assignee: "DevOps Engineer"
          effort: "4 days"
          dependencies: [S2]
          deliverables:
            - Kubernetes deployment
            - Auto-scaling configuration
            - Monitoring dashboards

        - id: L2
          title: "Load testing"
          assignee: "Backend Engineer 2"
          effort: "3 days"
          dependencies: [L1]
          deliverables:
            - k6 load test scripts
            - Performance baseline
            - Bottleneck identification

        - id: L3
          title: "Security audit"
          assignee: "Backend Engineer 1"
          effort: "2 days"
          dependencies: [S2]
          deliverables:
            - Dependency audit
            - OWASP checklist review
            - Penetration test (if external)

        - id: L4
          title: "Launch checklist"
          assignee: "All"
          effort: "2 days"
          dependencies: [L1, L2, L3]
          deliverables:
            - Runbook documentation
            - Rollback procedures
            - On-call rotation

      milestone: "Production launch"

  critical_path:
    - F1 → F2 → C1 → S1 → S2 → L1 → L4

  milestones:
    - week: 2
      name: "Development Ready"
      criteria: ["CI/CD working", "Database deployed", "API skeleton"]

    - week: 4
      name: "Core API Complete"
      criteria: ["All CRUD endpoints", ">80% test coverage", "Integration tests pass"]

    - week: 6
      name: "Security Complete"
      criteria: ["Auth working", "RBAC enforced", "Docs published"]

    - week: 8
      name: "Production Launch"
      criteria: ["Load tested", "Security audited", "Monitoring active"]

  resource_allocation:
    "Backend Engineer 1":
      utilization: "90%"
      focus: ["User API", "Auth", "Security"]
    "Backend Engineer 2":
      utilization: "85%"
      focus: ["Order API", "Payments", "Load testing"]
    "Fullstack Engineer":
      utilization: "80%"
      focus: ["Documentation", "Frontend integration"]
    "DevOps Engineer":
      utilization: "75%"
      focus: ["Infrastructure", "CI/CD", "Monitoring"]

  risks:
    - id: R1
      description: "Payment integration delays"
      probability: "medium"
      impact: "high"
      mitigation: "Start integration early, have mock service"

    - id: R2
      description: "Team member unavailability"
      probability: "low"
      impact: "medium"
      mitigation: "Cross-training, documentation"

    - id: R3
      description: "Scope creep"
      probability: "high"
      impact: "medium"
      mitigation: "Strict change control, MVP focus"
```

## Gantt Chart View

```
Week    │ 1   │ 2   │ 3   │ 4   │ 5   │ 6   │ 7   │ 8   │
────────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
Dev Env │████ │     │     │     │     │     │     │     │
Database│ ████│     │     │     │     │     │     │     │
User API│     │     │████ │     │     │     │     │     │
Orders  │     │     │████ │████ │     │     │     │     │
Payments│     │     │     │████ │     │     │     │     │
Auth    │     │     │     │     │████ │     │     │     │
RBAC    │     │     │     │     │ ████│     │     │     │
Docs    │     │     │     │     │████ │████ │     │     │
Infra   │     │     │     │     │     │     │████ │     │
Testing │     │     │     │     │     │     │████ │     │
Launch  │     │     │     │     │     │     │     │████ │
```

---

> **Tips:**
> - Focus on the critical path items first
> - Build buffer time for external dependencies
> - Regular check-ins to catch delays early
> - Document decisions for future reference
