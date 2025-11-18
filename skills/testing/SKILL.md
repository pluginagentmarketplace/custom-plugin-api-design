---
name: api-testing
description: API testing strategies including unit tests, integration tests, end-to-end tests, and testing tools
---

# API Testing Skill

## Quick Start

Ensure API quality through comprehensive testing:

### Testing Pyramid

```
    /\     E2E Tests (10%)
   /  \
  /____\   Integration Tests (30%)
 /      \
/________\ Unit Tests (60%)
```

### Unit Tests
```javascript
test('createUser validates email', async () => {
  await expect(createUser({ email: 'invalid' }))
    .rejects.toThrow('Invalid email');
});
```

### Integration Tests
```javascript
test('POST /users creates user', async () => {
  const res = await request(app)
    .post('/api/users')
    .send({ name: 'John', email: 'john@example.com' });

  expect(res.status).toBe(201);
  expect(res.body.id).toBeDefined();
});
```

### E2E Tests
```javascript
test('complete signup flow', async () => {
  // Sign up
  const signup = await request(app)
    .post('/auth/signup')
    .send({ email: 'john@example.com', password: 'secure' });

  // Login
  const login = await request(app)
    .post('/auth/login')
    .send({ email: 'john@example.com', password: 'secure' });

  // Verify authenticated
  const profile = await request(app)
    .get('/users/me')
    .set('Authorization', `Bearer ${login.body.token}`);

  expect(profile.status).toBe(200);
});
```

### Testing Tools
- **Jest** - Unit & integration tests
- **Supertest** - HTTP assertions
- **Postman** - Manual API testing
- **k6** - Load testing
- **Cypress** - E2E testing

### Response Testing
```javascript
test('returns proper error format', async () => {
  const res = await request(app)
    .post('/users')
    .send({ email: 'invalid' });

  expect(res.status).toBe(400);
  expect(res.body).toHaveProperty('error.code');
  expect(res.body).toHaveProperty('error.message');
});
```

## Testing Checklist

- [ ] Unit tests for functions
- [ ] Integration tests for endpoints
- [ ] E2E tests for workflows
- [ ] Error cases covered
- [ ] Authentication tested
- [ ] Rate limits tested
- [ ] Load testing done
- [ ] Test coverage > 80%

See the API Documentation & Testing agent for complete testing strategies.
