---
description: API documentation with OpenAPI/Swagger, API testing strategies, integration testing, and comprehensive documentation practices
capabilities: ["OpenAPI/Swagger", "Testing strategies", "Documentation generation", "API testing tools", "Integration testing", "Documentation best practices"]
---

# API Documentation & Testing

## OpenAPI/Swagger

### What is OpenAPI?
Machine-readable API specification in JSON or YAML

### OpenAPI 3.0 Example

```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: API for managing users
servers:
  - url: https://api.example.com/v1
paths:
  /users:
    get:
      summary: List all users
      parameters:
        - name: limit
          in: query
          required: false
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
        '401':
          description: Unauthorized

    post:
      summary: Create a new user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserInput'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'

  /users/{id}:
    get:
      summary: Get a specific user
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found

components:
  schemas:
    User:
      type: object
      required:
        - id
        - name
        - email
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        email:
          type: string
          format: email
        createdAt:
          type: string
          format: date-time

    CreateUserInput:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
        email:
          type: string
          format: email

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

## Testing Strategies

### Unit Tests
```javascript
describe('User API', () => {
  describe('createUser', () => {
    it('should create a user with valid input', async () => {
      const user = await createUser({
        name: 'John Doe',
        email: 'john@example.com'
      });

      expect(user.id).toBeDefined();
      expect(user.name).toBe('John Doe');
      expect(user.email).toBe('john@example.com');
    });

    it('should throw error with invalid email', async () => {
      await expect(createUser({
        name: 'John',
        email: 'invalid-email'
      })).rejects.toThrow('Invalid email');
    });
  });
});
```

### Integration Tests
```javascript
describe('User API Integration', () => {
  let server;

  beforeAll(() => {
    server = app.listen(3000);
  });

  afterAll(() => {
    server.close();
  });

  it('should create and retrieve user via HTTP', async () => {
    // Create
    const createRes = await request(app)
      .post('/api/users')
      .send({
        name: 'John',
        email: 'john@example.com'
      });

    expect(createRes.status).toBe(201);
    const userId = createRes.body.id;

    // Retrieve
    const getRes = await request(app)
      .get(`/api/users/${userId}`);

    expect(getRes.status).toBe(200);
    expect(getRes.body.name).toBe('John');
  });
});
```

### E2E Tests
```javascript
describe('User Flow E2E', () => {
  it('complete user signup flow', async () => {
    // 1. Register
    const signupRes = await request(app)
      .post('/api/auth/signup')
      .send({
        name: 'John',
        email: 'john@example.com',
        password: 'secure123'
      });

    expect(signupRes.status).toBe(201);
    const token = signupRes.body.token;

    // 2. Get profile
    const profileRes = await request(app)
      .get('/api/users/me')
      .set('Authorization', `Bearer ${token}`);

    expect(profileRes.status).toBe(200);
    expect(profileRes.body.email).toBe('john@example.com');

    // 3. Update profile
    const updateRes = await request(app)
      .put('/api/users/me')
      .set('Authorization', `Bearer ${token}`)
      .send({ name: 'John Updated' });

    expect(updateRes.status).toBe(200);
  });
});
```

## Testing Tools

### Postman
```javascript
// Test script
pm.test('Status is 200', () => {
  pm.response.to.have.status(200);
});

pm.test('Response time < 500ms', () => {
  pm.expect(pm.response.responseTime).to.be.below(500);
});

pm.test('User has required fields', () => {
  const json = pm.response.json();
  pm.expect(json).to.have.property('id');
  pm.expect(json).to.have.property('email');
});
```

### curl Testing
```bash
# Create user
curl -X POST https://api.example.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}'

# Get user
curl -X GET https://api.example.com/api/users/123

# With authentication
curl -X GET https://api.example.com/api/users/me \
  -H "Authorization: Bearer token123"
```

### REST Client (VS Code)
```http
### Create User
POST https://api.example.com/api/users
Content-Type: application/json

{
  "name": "John",
  "email": "john@example.com"
}

### Get User
GET https://api.example.com/api/users/123
Authorization: Bearer token123
```

## Documentation Best Practices

### 1. Clear Endpoint Description
```markdown
## Create Post

Creates a new blog post.

### Request
- **Method:** POST
- **Path:** `/api/posts`
- **Authentication:** Required (Bearer token)

### Request Body
```json
{
  "title": "Post Title",
  "content": "Post content",
  "tags": ["tag1", "tag2"]
}
```

### Response (201 Created)
```json
{
  "id": 1,
  "title": "Post Title",
  "content": "Post content",
  "author": { "id": 123, "name": "John" },
  "createdAt": "2024-01-01T00:00:00Z"
}
```

### Errors
- **400 Bad Request** - Invalid request body
- **401 Unauthorized** - Authentication required
- **422 Unprocessable Entity** - Validation failed
```

### 2. Include Examples
```markdown
### Example: Paginated Response

Request:
```
GET /api/posts?limit=10&offset=0
```

Response:
```json
{
  "data": [
    { "id": 1, "title": "Post 1" },
    { "id": 2, "title": "Post 2" }
  ],
  "pagination": {
    "limit": 10,
    "offset": 0,
    "total": 100
  }
}
```
```

### 3. Version Documentation
```markdown
## API Versions

### v1 (Current)
- Latest version
- Recommended for new integrations
- Base URL: https://api.example.com/v1

### v0 (Deprecated)
- Deprecated as of 2024-01-01
- Will be removed 2024-12-31
- Migrate to v1

## Migration Guide: v0 → v1
1. Change base URL
2. Update authentication (now uses JWT)
3. Change response format (now uses data envelope)
```

## Documentation Generation

### Swagger UI
```javascript
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const swaggerSpec = swaggerJsdoc({
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'User API',
      version: '1.0.0',
    },
  },
  apis: ['./routes/*.js'],
});

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

### API Reference Template
```markdown
# API Reference

## Base URL
```
https://api.example.com/v1
```

## Authentication
All requests require a Bearer token:
```
Authorization: Bearer YOUR_TOKEN_HERE
```

## Rate Limiting
- 1000 requests per hour
- Headers: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset

## Error Response Format
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message"
  }
}
```
```

## Testing Checklist

- [ ] Unit tests for all endpoints
- [ ] Integration tests for workflows
- [ ] Edge case testing
- [ ] Error response testing
- [ ] Authentication testing
- [ ] Rate limiting testing
- [ ] Load testing performed
- [ ] Security testing completed

## Documentation Checklist

- [ ] OpenAPI/Swagger specification created
- [ ] All endpoints documented
- [ ] Request/response examples provided
- [ ] Error codes documented
- [ ] Authentication explained
- [ ] Rate limits documented
- [ ] Migration guides for versions
- [ ] Code examples in multiple languages

Next: Implement advanced patterns with the Advanced API Patterns agent.
