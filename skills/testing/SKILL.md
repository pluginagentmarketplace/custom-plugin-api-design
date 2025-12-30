---
name: testing
description: API testing strategies and contract testing
sasmp_version: "2.0.0"
bonded_agent: 05-security-compliance
bond_type: SECONDARY_BOND

# Production-Grade Metadata
parameters:
  - name: test_type
    type: string
    required: true
    validation: "^(unit|integration|contract|load|security)$"
    description: Type of testing
  - name: framework
    type: string
    required: false
    validation: "^(jest|mocha|pytest|pact)$"
    description: Testing framework

validation_rules:
  - All endpoints must have test coverage
  - Contract tests must run in CI
  - Security tests must include OWASP checks

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [test_coverage, test_duration, failure_rate]
---

# API Testing Skill

## Topics Covered
- Unit/integration testing
- Contract testing (Pact)
- Load testing, security testing
- Mock servers, test automation

## Testing Pyramid

```
       /\
      /  \     E2E Tests (Few)
     /----\
    /      \   Integration Tests
   /--------\
  /          \  Unit Tests (Many)
 /------------\
```

## Unit Testing

```javascript
describe('UserService', () => {
  test('creates user with valid data', async () => {
    const user = await userService.create({
      name: 'Test',
      email: 'test@example.com'
    });
    expect(user.id).toBeDefined();
    expect(user.name).toBe('Test');
  });

  test('throws on invalid email', async () => {
    await expect(userService.create({
      name: 'Test',
      email: 'invalid'
    })).rejects.toThrow('Invalid email');
  });
});
```

## Integration Testing

```javascript
describe('POST /api/users', () => {
  test('creates user and returns 201', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Test', email: 'test@example.com' });

    expect(response.status).toBe(201);
    expect(response.body.id).toBeDefined();
  });
});
```

## Contract Testing (Pact)

```javascript
describe('Pact Consumer', () => {
  test('gets user by id', async () => {
    await provider.addInteraction({
      state: 'user exists',
      uponReceiving: 'a request for user',
      withRequest: { method: 'GET', path: '/api/users/1' },
      willRespondWith: {
        status: 200,
        body: { id: '1', name: 'Test' }
      }
    });

    const user = await client.getUser('1');
    expect(user.name).toBe('Test');
  });
});
```

## Load Testing

```javascript
// k6 load test
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 100,
  duration: '30s',
};

export default function () {
  const res = http.get('http://api.example.com/users');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
}
```

## Unit Test Template

```javascript
describe('testing skill', () => {
  test('validates test coverage report', () => {
    const coverage = getCoverageReport();
    expect(coverage.lines).toBeGreaterThan(80);
    expect(coverage.branches).toBeGreaterThan(70);
  });

  test('contract tests pass', async () => {
    await expect(runContractTests()).resolves.toBe(true);
  });
});
```

## Learning Outcomes
- Test APIs thoroughly
- Implement contract tests
- Automate API testing

See Agent 5: Security Compliance for detailed guidance.
