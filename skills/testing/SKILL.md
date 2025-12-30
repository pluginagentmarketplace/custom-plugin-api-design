---
name: testing
version: "2.0.0"
description: API testing strategies and contract testing
sasmp_version: "1.3.0"
bonded_agent: 05-security-compliance
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "API testing patterns and automation"
  boundaries:
    includes: [unit_testing, integration_testing, contract_testing, load_testing]
    excludes: [implementation_code, deployment, monitoring]

parameter_validation:
  schema:
    type: object
    properties:
      test_type:
        type: string
        enum: [unit, integration, contract, e2e, load, security]
      coverage_target:
        type: number
        minimum: 0
        maximum: 100

retry_config:
  enabled: true
  max_attempts: 3
  backoff:
    type: linear
    initial_delay_ms: 1000

logging:
  level: INFO
  fields: [test_type, passed, failed, duration_ms]

dependencies:
  skills: [security-patterns]
  agents: [05-security-compliance]
---

# API Testing Skill

## Purpose
Implement comprehensive API testing strategies.

## Testing Pyramid

```
         ╱╲
        ╱E2E╲        Few, slow, expensive
       ╱──────╲
      ╱ Contract╲    Consumer-driven contracts
     ╱────────────╲
    ╱ Integration  ╲  API + Database
   ╱────────────────╲
  ╱     Unit Tests   ╲ Fast, many, cheap
 ╱────────────────────╲
```

## Unit Testing

### Controller/Handler Tests

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { UserController } from './user.controller';
import { UserService } from './user.service';

describe('UserController', () => {
  let controller: UserController;
  let mockUserService: jest.Mocked<UserService>;

  beforeEach(() => {
    mockUserService = {
      findById: vi.fn(),
      create: vi.fn(),
      update: vi.fn(),
      delete: vi.fn(),
    };
    controller = new UserController(mockUserService);
  });

  describe('getUser', () => {
    it('should return user when found', async () => {
      const mockUser = { id: '123', name: 'John', email: 'john@test.com' };
      mockUserService.findById.mockResolvedValue(mockUser);

      const result = await controller.getUser('123');

      expect(result).toEqual({ data: mockUser });
      expect(mockUserService.findById).toHaveBeenCalledWith('123');
    });

    it('should throw NotFoundException when user not found', async () => {
      mockUserService.findById.mockResolvedValue(null);

      await expect(controller.getUser('999')).rejects.toThrow('User not found');
    });
  });

  describe('createUser', () => {
    it('should create and return new user', async () => {
      const input = { email: 'new@test.com', name: 'New User' };
      const created = { id: '456', ...input };
      mockUserService.create.mockResolvedValue(created);

      const result = await controller.createUser(input);

      expect(result.data.id).toBe('456');
    });

    it('should validate input', async () => {
      const invalidInput = { email: 'invalid', name: '' };

      await expect(controller.createUser(invalidInput)).rejects.toThrow('Validation');
    });
  });
});
```

### Service/Business Logic Tests

```typescript
describe('UserService', () => {
  let service: UserService;
  let mockRepository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    mockRepository = {
      findById: vi.fn(),
      save: vi.fn(),
      delete: vi.fn(),
    };
    service = new UserService(mockRepository);
  });

  describe('create', () => {
    it('should hash password before saving', async () => {
      const input = { email: 'test@test.com', name: 'Test', password: 'secret123' };

      await service.create(input);

      expect(mockRepository.save).toHaveBeenCalledWith(
        expect.objectContaining({
          email: 'test@test.com',
          passwordHash: expect.not.stringContaining('secret123'),
        })
      );
    });

    it('should throw ConflictError for duplicate email', async () => {
      mockRepository.save.mockRejectedValue({ code: '23505' }); // Unique violation

      await expect(service.create({ email: 'existing@test.com' }))
        .rejects.toThrow('Email already exists');
    });
  });
});
```

## Integration Testing

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import request from 'supertest';
import { app } from './app';
import { db } from './database';

describe('Users API Integration', () => {
  beforeAll(async () => {
    await db.migrate.latest();
    await db.seed.run();
  });

  afterAll(async () => {
    await db.destroy();
  });

  describe('GET /api/v1/users', () => {
    it('should return paginated list', async () => {
      const res = await request(app)
        .get('/api/v1/users?page=1&limit=10')
        .set('Authorization', `Bearer ${testToken}`)
        .expect(200);

      expect(res.body.data).toBeInstanceOf(Array);
      expect(res.body.pagination).toMatchObject({
        page: 1,
        limit: 10,
        total: expect.any(Number),
      });
    });
  });

  describe('POST /api/v1/users', () => {
    it('should persist user to database', async () => {
      const userData = {
        email: 'integration@test.com',
        name: 'Integration Test',
        password: 'SecurePass123!',
      };

      const res = await request(app)
        .post('/api/v1/users')
        .set('Authorization', `Bearer ${adminToken}`)
        .send(userData)
        .expect(201);

      // Verify in database
      const dbUser = await db('users')
        .where('id', res.body.data.id)
        .first();

      expect(dbUser).toBeDefined();
      expect(dbUser.email).toBe(userData.email);
    });
  });
});
```

## Contract Testing (Pact)

### Consumer Side

```typescript
import { Pact } from '@pact-foundation/pact';
import { UserApiClient } from './user-api-client';

describe('User API Contract', () => {
  const provider = new Pact({
    consumer: 'WebApp',
    provider: 'UserService',
    port: 1234,
  });

  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());
  afterEach(() => provider.verify());

  describe('get user by id', () => {
    it('should return user when exists', async () => {
      await provider.addInteraction({
        state: 'user with id 123 exists',
        uponReceiving: 'a request for user 123',
        withRequest: {
          method: 'GET',
          path: '/api/v1/users/123',
          headers: { Accept: 'application/json' },
        },
        willRespondWith: {
          status: 200,
          headers: { 'Content-Type': 'application/json' },
          body: {
            data: {
              id: '123',
              name: like('John Doe'),
              email: like('john@example.com'),
            },
          },
        },
      });

      const client = new UserApiClient('http://localhost:1234');
      const user = await client.getUser('123');

      expect(user.id).toBe('123');
    });
  });
});
```

### Provider Side

```typescript
import { Verifier } from '@pact-foundation/pact';

describe('Pact Verification', () => {
  it('should validate consumer contracts', async () => {
    const verifier = new Verifier({
      providerBaseUrl: 'http://localhost:3000',
      pactBrokerUrl: process.env.PACT_BROKER_URL,
      provider: 'UserService',
      publishVerificationResult: true,
      stateHandlers: {
        'user with id 123 exists': async () => {
          await db('users').insert({ id: '123', name: 'John', email: 'john@test.com' });
        },
      },
    });

    await verifier.verifyProvider();
  });
});
```

## Load Testing (k6)

```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 50 },   // Ramp up
    { duration: '3m', target: 50 },   // Steady state
    { duration: '1m', target: 100 },  // Peak load
    { duration: '1m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% under 500ms
    http_req_failed: ['rate<0.01'],    // <1% error rate
  },
};

export default function () {
  const token = __ENV.API_TOKEN;

  // List users
  const listRes = http.get('http://api.example.com/api/v1/users?limit=20', {
    headers: { Authorization: `Bearer ${token}` },
  });

  check(listRes, {
    'list status 200': (r) => r.status === 200,
    'list has data': (r) => JSON.parse(r.body).data.length > 0,
  });

  sleep(1);

  // Get single user
  const getRes = http.get('http://api.example.com/api/v1/users/123', {
    headers: { Authorization: `Bearer ${token}` },
  });

  check(getRes, {
    'get status 200': (r) => r.status === 200,
  });

  sleep(1);
}
```

Run: `k6 run --env API_TOKEN=xxx load-test.js`

## Security Testing

```typescript
describe('Security Tests', () => {
  describe('Authentication', () => {
    it('should reject requests without token', async () => {
      await request(app)
        .get('/api/v1/users')
        .expect(401);
    });

    it('should reject expired tokens', async () => {
      await request(app)
        .get('/api/v1/users')
        .set('Authorization', `Bearer ${expiredToken}`)
        .expect(401);
    });
  });

  describe('Authorization', () => {
    it('should prevent accessing other users data', async () => {
      await request(app)
        .get('/api/v1/users/other-user-id')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(403);
    });
  });

  describe('Input Validation', () => {
    it('should reject SQL injection attempts', async () => {
      await request(app)
        .get(`/api/v1/users?search='; DROP TABLE users; --`)
        .set('Authorization', `Bearer ${token}`)
        .expect(400);
    });

    it('should sanitize XSS in input', async () => {
      const res = await request(app)
        .post('/api/v1/users')
        .set('Authorization', `Bearer ${adminToken}`)
        .send({ name: '<script>alert("xss")</script>' })
        .expect(201);

      expect(res.body.data.name).not.toContain('<script>');
    });
  });

  describe('Rate Limiting', () => {
    it('should block after exceeding limit', async () => {
      // Make many requests
      for (let i = 0; i < 100; i++) {
        await request(app).get('/api/v1/users');
      }

      // Should be rate limited
      await request(app)
        .get('/api/v1/users')
        .expect(429);
    });
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Flaky tests | Shared state | Isolate test data |
| Slow integration tests | Database setup | Use transactions |
| Contract mismatches | Schema changes | Version contracts |
| Load test failures | Connection limits | Check pool size |

---

## Quality Checklist

- [ ] Unit test coverage > 80%
- [ ] Integration tests for critical paths
- [ ] Contract tests with consumers
- [ ] Load tests before releases
- [ ] Security tests automated
- [ ] Tests run in CI/CD
- [ ] Test data isolated
- [ ] Mock external services
