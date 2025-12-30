---
name: audit
description: Audit Your API Design
allowed-tools: Read, Glob, Grep
sasmp_version: "2.0.0"

# Production-Grade Metadata
input_validation:
  required: [spec_type]
  optional: [spec_content, file_path]
  schema:
    spec_type:
      type: string
      enum: [openapi, graphql, custom]
    spec_content:
      type: string
      description: Inline spec content
    file_path:
      type: string
      description: Path to spec file

exit_codes:
  0: Audit passed
  1: Minor issues found
  2: Major issues found
  3: Critical issues found
---

# /audit - Audit Your API Design

Comprehensive review of your API design.

## What to Share

Share your OpenAPI spec, GraphQL schema, or architecture description.

## Usage Examples

```
/audit openapi file_path=./openapi.yaml
/audit graphql file_path=./schema.graphql
/audit custom
```

## What You'll Get

- **Design Quality Assessment** - Score and analysis
- **Potential Issues Identified** - Categorized by severity
- **Improvement Recommendations** - Prioritized fixes
- **Best Practices Guidance** - Industry standards
- **Code Examples for Fixes** - Implementation help

## Output Structure

```
1. Audit Summary
   └─ Overall score: [A/B/C/D/F]
   └─ Endpoints analyzed: N
   └─ Issues found: N

2. Issues by Severity
   └─ Critical: N
   └─ High: N
   └─ Medium: N
   └─ Low: N

3. Detailed Findings
   └─ Issue description
   └─ Location
   └─ Recommended fix
   └─ Code example

4. Best Practices Check
   └─ [✓] Versioning
   └─ [✓] Error handling
   └─ [✗] Pagination
   └─ [✓] Authentication
```

## Audit Categories

| Category | Checks |
|----------|--------|
| Design | Resource naming, HTTP methods, status codes |
| Security | Auth, input validation, rate limiting |
| Performance | Pagination, caching, response size |
| Documentation | Examples, descriptions, error codes |
