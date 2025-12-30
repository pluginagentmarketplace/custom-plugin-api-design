---
name: architect
description: Design Your Plugin API System
allowed-tools: Read, Glob, Grep
sasmp_version: "2.0.0"

# Production-Grade Metadata
input_validation:
  required: [domain]
  optional: [scale, constraints]
  schema:
    domain:
      type: string
      description: Primary business domain
    scale:
      type: string
      enum: [startup, growth, enterprise]
    constraints:
      type: array
      description: Technical constraints

exit_codes:
  0: Success
  1: Invalid input
  2: Incomplete requirements
  3: Conflict detected
---

# /architect - Design Your Plugin API System

Design a complete plugin system architecture.

## What You'll Get

- **Architecture Recommendation** - Best approach for your use case
- **API Design** - Plugin interfaces and contracts
- **Implementation Roadmap** - Step-by-step development plan
- **Security Model** - Authentication and authorization
- **Scalability Plan** - Growth from 10 to 10,000 users

## How to Use

Share information about your system:
- What's your primary domain? (e.g., payments, ecommerce, content management)
- Expected scale? (users, requests/sec)
- Primary plugins' responsibilities?
- Any existing constraints? (tech stack, timeline)

## Usage Examples

```
/architect payments startup
/architect ecommerce enterprise constraints=microservices,k8s
/architect content-management growth
```

## Example Input

> "I'm building a SaaS product with 5 user-created plugins per customer, 1000 daily active users, focus on data processing"

## Output Structure

```
1. Architecture Overview
   └─ Recommended pattern
   └─ Component diagram

2. API Design
   └─ Endpoint definitions
   └─ Data schemas
   └─ Authentication method

3. Implementation Steps
   └─ Phase 1: Core API
   └─ Phase 2: Plugin system
   └─ Phase 3: Scale optimization

4. Security Considerations
   └─ Auth strategy
   └─ Rate limiting
   └─ Data encryption
```

→ Get complete architecture design with all 8 agent perspectives
