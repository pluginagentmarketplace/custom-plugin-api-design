---
name: architect
version: "2.0.0"
description: Design Your Plugin API System
sasmp_version: "1.3.0"
allowed-tools: Read

# Command Configuration
input_schema:
  type: object
  required: [domain]
  properties:
    domain:
      type: string
      description: "Primary domain (e.g., payments, ecommerce, content)"
    scale:
      type: object
      properties:
        users: { type: number }
        requests_per_second: { type: number }
    plugins:
      type: array
      items:
        type: string
      description: "List of plugin responsibilities"
    constraints:
      type: object
      properties:
        tech_stack: { type: array, items: { type: string } }
        timeline_weeks: { type: number }
        budget_tier: { type: string, enum: [startup, growth, enterprise] }

output_schema:
  type: object
  properties:
    architecture:
      type: object
      properties:
        style: { type: string, enum: [rest, graphql, hybrid] }
        pattern: { type: string }
        diagram: { type: string }
    api_design:
      type: object
      properties:
        endpoints: { type: array }
        schemas: { type: object }
    security_model:
      type: object
    scalability_plan:
      type: object
    implementation_roadmap:
      type: array

orchestration:
  agents:
    - 01-api-architect      # API style selection
    - 05-security-compliance # Security model
    - 07-scaling-patterns    # Scalability
    - 04-devops-infrastructure # Deployment
  execution: parallel_then_merge

error_handling:
  on_missing_input: prompt_user
  on_agent_failure: fallback_to_defaults
  timeout_ms: 60000
---

# /architect - Design Your Plugin API System

Design a complete, production-ready plugin system architecture with multi-agent analysis.

## What You'll Get

```
┌─────────────────────────────────────────────────────────────┐
│                    Architecture Output                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Architecture Recommendation                              │
│     └─ REST vs GraphQL vs Hybrid decision                   │
│     └─ Microservices vs Monolith analysis                   │
│     └─ Component diagram                                     │
│                                                              │
│  2. API Design                                               │
│     └─ Plugin interface contracts                           │
│     └─ Event system design                                  │
│     └─ Versioning strategy                                  │
│                                                              │
│  3. Security Model                                           │
│     └─ Authentication (OAuth2/JWT)                          │
│     └─ Authorization (RBAC/ABAC)                            │
│     └─ Plugin sandboxing                                    │
│                                                              │
│  4. Scalability Plan                                         │
│     └─ 10 → 100 → 1000 → 10000 users                       │
│     └─ Horizontal scaling strategy                          │
│     └─ Caching & performance                                │
│                                                              │
│  5. Implementation Roadmap                                   │
│     └─ Phase-by-phase breakdown                             │
│     └─ Critical path identification                         │
│     └─ Risk assessment                                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## How to Use

Provide information about your system:

```yaml
Domain: "Your primary domain (payments, ecommerce, content, analytics)"
Scale:
  current_users: 100
  target_users: 10000
  requests_per_second: 500
Plugins:
  - "What plugins will do"
  - "Another plugin responsibility"
Constraints:
  tech_stack: [Node.js, PostgreSQL, Redis]
  timeline: "3 months"
  team_size: 4
```

## Example Input

```
I'm building a SaaS analytics platform:
- Domain: Business intelligence
- Current: 50 users, targeting 5000 in 12 months
- Plugins: Custom data connectors, visualization widgets, export formats
- Stack: TypeScript, Next.js, PostgreSQL
- Team: 3 engineers, 2 months to MVP
```

## Example Output

```yaml
Architecture:
  style: hybrid
  pattern: "REST for CRUD, GraphQL for complex queries"
  reasoning: "Data exploration needs flexible querying"

API Design:
  plugin_interface:
    registration: "POST /api/v1/plugins"
    lifecycle: [install, activate, deactivate, uninstall]
    hooks: [data.query, data.transform, viz.render]

  openapi_spec: |
    openapi: 3.1.0
    paths:
      /api/v1/plugins:
        post:
          summary: Register new plugin
          requestBody:
            content:
              application/json:
                schema:
                  $ref: '#/components/schemas/PluginManifest'

Security:
  authentication: "OAuth 2.0 + API Keys for plugins"
  authorization: "RBAC with plugin-specific scopes"
  sandboxing: "V8 Isolates for custom code execution"

Scalability:
  phase_1: "Single instance, connection pooling"
  phase_2: "Read replicas, Redis caching"
  phase_3: "Kubernetes HPA, sharded databases"

Roadmap:
  - week_1_2: "Core plugin loader, basic API"
  - week_3_4: "Security layer, first plugin type"
  - week_5_6: "Scaling infrastructure, monitoring"
  - week_7_8: "Beta testing, documentation"
```

## Multi-Agent Analysis

This command orchestrates multiple specialized agents:

| Agent | Contribution |
|-------|-------------|
| API Architect | Architecture style, API design |
| Security | Auth model, sandboxing, compliance |
| Scaling | Performance, caching, infrastructure |
| DevOps | Deployment, CI/CD, monitoring |

Each agent provides domain-specific recommendations that are merged into a cohesive architecture.

---

> **Tip:** The more context you provide, the more tailored the architecture will be. Include any existing code, constraints, or preferences.
