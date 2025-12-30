---
name: versioning
description: API versioning strategies and backward compatibility
sasmp_version: "2.0.0"
bonded_agent: 01-api-architect
bond_type: SECONDARY_BOND

# Production-Grade Metadata
parameters:
  - name: strategy
    type: string
    required: true
    validation: "^(url|header|query)$"
    description: Versioning strategy
  - name: current_version
    type: string
    required: false
    description: Current API version

validation_rules:
  - Version format must follow semver
  - Breaking changes require major version bump
  - Deprecation notices must be provided

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [version_distribution, deprecation_warnings]
---

# API Versioning Skill

## Topics Covered
- URL versioning, header versioning
- Semantic versioning, deprecation policies
- Breaking changes, migration guides

## Versioning Strategies

| Strategy | Example | Use Case |
|----------|---------|----------|
| URL Path | `/api/v1/users` | Public APIs |
| Header | `Accept: application/vnd.api+json;v=2` | Internal APIs |
| Query | `/api/users?version=2` | Legacy support |

## Breaking vs Non-Breaking Changes

### Non-Breaking (Minor Version)
- Add optional field with default
- Add new endpoint
- Make validation more lenient

### Breaking (Major Version)
- Remove or rename field
- Change field type
- Change response structure

## Deprecation Timeline

```
v1.0 (Current)
  ↓
v1.5 (Deprecation announced, sunset date set)
  ↓
v2.0 (New version released)
  ↓
v1.x (Sunset after 12 months)
```

## Implementation

```javascript
// URL versioning
app.get('/api/v1/users', v1UsersHandler);
app.get('/api/v2/users', v2UsersHandler);

// Header versioning
app.get('/api/users', (req, res) => {
  const version = req.headers['accept-version'] || 'v1';
  if (version === 'v2') return v2UsersHandler(req, res);
  return v1UsersHandler(req, res);
});
```

## Unit Test Template

```javascript
describe('versioning skill', () => {
  test('v1 endpoint returns v1 format', async () => {
    const res = await request(app).get('/api/v1/users');
    expect(res.body).toHaveProperty('users');
  });

  test('v2 endpoint returns v2 format', async () => {
    const res = await request(app).get('/api/v2/users');
    expect(res.body).toHaveProperty('data');
  });
});
```

## Learning Outcomes
- Choose versioning strategy
- Handle deprecation
- Maintain compatibility

See Agent 1: API Architect for detailed guidance.
