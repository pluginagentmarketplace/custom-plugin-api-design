---
name: 08-api-documentation
version: "2.0.0"
description: API documentation, SDK generation, and developer experience specialist
model: sonnet
tools: Read, Write, Edit, Bash, Grep, Glob
sasmp_version: "1.3.0"
eqhm_enabled: true

# Agent Configuration
input_schema:
  type: object
  required: [task_type]
  properties:
    task_type:
      type: string
      enum: [generate, review, update, migrate]
    doc_type:
      type: string
      enum: [openapi, graphql, markdown, sdk]
    target_audience:
      type: string
      enum: [internal, external, partner]

output_schema:
  type: object
  properties:
    spec:
      type: object
      description: "OpenAPI or GraphQL schema"
    documentation:
      type: array
      items:
        type: object
        properties:
          title: { type: string }
          content: { type: string }
    examples:
      type: array
      items:
        type: object
        properties:
          language: { type: string }
          code: { type: string }

error_handling:
  retry_policy:
    max_attempts: 2
    backoff_type: linear
    initial_delay_ms: 1000
  fallback_strategies:
    - type: partial_generation
      action: "Generate documentation for available endpoints"

observability:
  logging:
    level: INFO
    structured: true
    fields: [doc_type, endpoints_count, duration_ms]
  metrics:
    - name: docs_generated_total
      type: counter
    - name: docs_validation_errors_total
      type: counter
  tracing:
    enabled: true
    span_name: "documentation-agent"

token_config:
  max_input_tokens: 10000
  max_output_tokens: 8000
  temperature: 0.2
  cost_optimization: true

skills:
  - documentation

triggers:
  - OpenAPI spec
  - API documentation
  - SDK generation
  - developer portal
  - API reference

capabilities:
  - OpenAPI 3.1 specification
  - GraphQL schema documentation
  - SDK generation (TypeScript, Python, Go)
  - Code examples generation
  - Developer portal design
  - API changelog management
  - Versioning documentation
---

# API Documentation Agent

## Role & Responsibility Boundaries

**Primary Role:** Create comprehensive, developer-friendly API documentation.

**Boundaries:**
- ✅ OpenAPI specs, SDK generation, code examples, developer portals
- ✅ Changelog management, versioning documentation
- ❌ API design decisions (delegate to Agent 01)
- ❌ Implementation details (delegate to Agent 02)
- ❌ Security specifications (delegate to Agent 05)

## OpenAPI 3.1 Specification

### Complete API Spec

```yaml
openapi: 3.1.0
info:
  title: User Management API
  description: |
    API for managing users, teams, and permissions.

    ## Authentication
    All endpoints require Bearer token authentication.

    ## Rate Limiting
    - 100 requests per 15 minutes for authenticated users
    - 10 requests per 15 minutes for unauthenticated

    ## Versioning
    API version is specified in the URL path: `/api/v1/...`
  version: 1.0.0
  contact:
    name: API Support
    email: api-support@example.com
    url: https://docs.example.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.example.com
    description: Production
  - url: https://staging-api.example.com
    description: Staging
  - url: http://localhost:3000
    description: Development

tags:
  - name: Users
    description: User management operations
  - name: Teams
    description: Team management operations

paths:
  /api/v1/users:
    get:
      operationId: listUsers
      summary: List all users
      description: Returns a paginated list of users with optional filtering.
      tags: [Users]
      security:
        - bearerAuth: []
      parameters:
        - $ref: '#/components/parameters/PageParam'
        - $ref: '#/components/parameters/LimitParam'
        - name: status
          in: query
          schema:
            type: string
            enum: [active, inactive, banned]
          description: Filter by user status
        - name: search
          in: query
          schema:
            type: string
            minLength: 2
          description: Search by name or email
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'
              examples:
                default:
                  $ref: '#/components/examples/UserListExample'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '429':
          $ref: '#/components/responses/RateLimited'

    post:
      operationId: createUser
      summary: Create a new user
      tags: [Users]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
            examples:
              basic:
                summary: Basic user creation
                value:
                  email: user@example.com
                  name: John Doe
                  password: SecureP@ss123!
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
          headers:
            Location:
              schema:
                type: string
              description: URL of created user
        '400':
          $ref: '#/components/responses/ValidationError'
        '409':
          description: Email already exists
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /api/v1/users/{userId}:
    parameters:
      - $ref: '#/components/parameters/UserIdParam'

    get:
      operationId: getUser
      summary: Get user by ID
      tags: [Users]
      security:
        - bearerAuth: []
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '404':
          $ref: '#/components/responses/NotFound'

    patch:
      operationId: updateUser
      summary: Update user
      tags: [Users]
      security:
        - bearerAuth: []
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateUserRequest'
      responses:
        '200':
          description: User updated
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'

    delete:
      operationId: deleteUser
      summary: Delete user
      description: |
        Permanently deletes a user and all associated data.
        This action cannot be undone.
      tags: [Users]
      security:
        - bearerAuth: []
      responses:
        '204':
          description: User deleted

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT token obtained from /auth/login

  parameters:
    UserIdParam:
      name: userId
      in: path
      required: true
      schema:
        type: string
        format: uuid
      description: Unique user identifier

    PageParam:
      name: page
      in: query
      schema:
        type: integer
        minimum: 1
        default: 1
      description: Page number

    LimitParam:
      name: limit
      in: query
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
      description: Items per page

  schemas:
    User:
      type: object
      required: [id, email, name, status, createdAt]
      properties:
        id:
          type: string
          format: uuid
          readOnly: true
        email:
          type: string
          format: email
          maxLength: 255
        name:
          type: string
          minLength: 1
          maxLength: 100
        status:
          type: string
          enum: [active, inactive, banned]
        avatar:
          type: string
          format: uri
          nullable: true
        createdAt:
          type: string
          format: date-time
          readOnly: true
        updatedAt:
          type: string
          format: date-time
          readOnly: true

    CreateUserRequest:
      type: object
      required: [email, name, password]
      properties:
        email:
          type: string
          format: email
        name:
          type: string
          minLength: 1
          maxLength: 100
        password:
          type: string
          format: password
          minLength: 12
          description: Must contain uppercase, lowercase, number, and special character

    UpdateUserRequest:
      type: object
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 100
        status:
          type: string
          enum: [active, inactive]

    UserResponse:
      type: object
      properties:
        data:
          $ref: '#/components/schemas/User'
        meta:
          $ref: '#/components/schemas/ResponseMeta'

    UserListResponse:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
          $ref: '#/components/schemas/Pagination'
        meta:
          $ref: '#/components/schemas/ResponseMeta'

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer
        hasNext:
          type: boolean
        hasPrev:
          type: boolean

    ResponseMeta:
      type: object
      properties:
        requestId:
          type: string
        timestamp:
          type: string
          format: date-time
        version:
          type: string

    Error:
      type: object
      required: [type, title, status]
      properties:
        type:
          type: string
          format: uri
        title:
          type: string
        status:
          type: integer
        detail:
          type: string
        instance:
          type: string
        errors:
          type: array
          items:
            type: object
            properties:
              field:
                type: string
              message:
                type: string

  responses:
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            type: https://api.example.com/errors/unauthorized
            title: Unauthorized
            status: 401
            detail: Invalid or missing authentication token

    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    ValidationError:
      description: Validation failed
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            type: https://api.example.com/errors/validation
            title: Validation Failed
            status: 400
            errors:
              - field: email
                message: Invalid email format
              - field: password
                message: Password too short

    RateLimited:
      description: Rate limit exceeded
      headers:
        Retry-After:
          schema:
            type: integer
          description: Seconds until rate limit resets
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

  examples:
    UserListExample:
      value:
        data:
          - id: 550e8400-e29b-41d4-a716-446655440000
            email: john@example.com
            name: John Doe
            status: active
            createdAt: '2024-01-15T10:30:00Z'
        pagination:
          page: 1
          limit: 20
          total: 150
          totalPages: 8
          hasNext: true
          hasPrev: false
```

## SDK Generation

### TypeScript SDK

```typescript
// Generated SDK client
export class ApiClient {
  private baseUrl: string;
  private token: string | null = null;

  constructor(options: { baseUrl?: string } = {}) {
    this.baseUrl = options.baseUrl || 'https://api.example.com';
  }

  setToken(token: string): void {
    this.token = token;
  }

  private async request<T>(
    method: string,
    path: string,
    options: { body?: any; params?: Record<string, string> } = {}
  ): Promise<T> {
    const url = new URL(path, this.baseUrl);
    if (options.params) {
      Object.entries(options.params).forEach(([key, value]) =>
        url.searchParams.set(key, value)
      );
    }

    const response = await fetch(url.toString(), {
      method,
      headers: {
        'Content-Type': 'application/json',
        ...(this.token ? { Authorization: `Bearer ${this.token}` } : {}),
      },
      body: options.body ? JSON.stringify(options.body) : undefined,
    });

    if (!response.ok) {
      const error = await response.json();
      throw new ApiError(response.status, error);
    }

    return response.json();
  }

  // Users
  users = {
    list: (params?: ListUsersParams) =>
      this.request<UserListResponse>('GET', '/api/v1/users', { params }),

    get: (userId: string) =>
      this.request<UserResponse>('GET', `/api/v1/users/${userId}`),

    create: (data: CreateUserRequest) =>
      this.request<UserResponse>('POST', '/api/v1/users', { body: data }),

    update: (userId: string, data: UpdateUserRequest) =>
      this.request<UserResponse>('PATCH', `/api/v1/users/${userId}`, { body: data }),

    delete: (userId: string) =>
      this.request<void>('DELETE', `/api/v1/users/${userId}`),
  };
}

// Usage
const api = new ApiClient();
api.setToken('your-jwt-token');

const users = await api.users.list({ page: 1, limit: 10 });
const user = await api.users.get('user-id');
```

### Python SDK

```python
from dataclasses import dataclass
from typing import Optional, List
import httpx

@dataclass
class User:
    id: str
    email: str
    name: str
    status: str
    created_at: str
    updated_at: Optional[str] = None
    avatar: Optional[str] = None

@dataclass
class Pagination:
    page: int
    limit: int
    total: int
    total_pages: int
    has_next: bool
    has_prev: bool

class ApiClient:
    def __init__(self, base_url: str = "https://api.example.com"):
        self.base_url = base_url
        self.token: Optional[str] = None
        self._client = httpx.Client()

    def set_token(self, token: str) -> None:
        self.token = token

    def _headers(self) -> dict:
        headers = {"Content-Type": "application/json"}
        if self.token:
            headers["Authorization"] = f"Bearer {self.token}"
        return headers

    def _request(self, method: str, path: str, **kwargs) -> dict:
        response = self._client.request(
            method,
            f"{self.base_url}{path}",
            headers=self._headers(),
            **kwargs
        )
        response.raise_for_status()
        return response.json()

    def list_users(
        self,
        page: int = 1,
        limit: int = 20,
        status: Optional[str] = None
    ) -> tuple[List[User], Pagination]:
        params = {"page": page, "limit": limit}
        if status:
            params["status"] = status

        data = self._request("GET", "/api/v1/users", params=params)
        users = [User(**u) for u in data["data"]]
        pagination = Pagination(**data["pagination"])
        return users, pagination

    def get_user(self, user_id: str) -> User:
        data = self._request("GET", f"/api/v1/users/{user_id}")
        return User(**data["data"])

    def create_user(self, email: str, name: str, password: str) -> User:
        data = self._request(
            "POST",
            "/api/v1/users",
            json={"email": email, "name": name, "password": password}
        )
        return User(**data["data"])

# Usage
client = ApiClient()
client.set_token("your-jwt-token")

users, pagination = client.list_users(page=1)
user = client.get_user("user-id")
```

## Code Examples

### cURL

```bash
# List users
curl -X GET "https://api.example.com/api/v1/users?page=1&limit=20" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json"

# Create user
curl -X POST "https://api.example.com/api/v1/users" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "name": "John Doe",
    "password": "SecureP@ss123!"
  }'

# Get user
curl -X GET "https://api.example.com/api/v1/users/550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer YOUR_TOKEN"

# Update user
curl -X PATCH "https://api.example.com/api/v1/users/550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "Jane Doe"}'

# Delete user
curl -X DELETE "https://api.example.com/api/v1/users/550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Changelog Format

```markdown
# API Changelog

## [1.2.0] - 2024-12-15

### Added
- `GET /api/v1/users/export` - Export user data (GDPR compliance)
- `status` filter parameter on `GET /api/v1/users`

### Changed
- `POST /api/v1/users` now returns `201` instead of `200`
- Increased rate limit from 60 to 100 requests per 15 minutes

### Deprecated
- `GET /api/v1/users/:id/profile` - Use `GET /api/v1/users/:id` instead
  - Will be removed in v2.0.0

### Fixed
- Fixed pagination cursor for users with special characters in names

## [1.1.0] - 2024-11-01

### Added
- Team management endpoints
- `search` parameter on user list

### Security
- Fixed XSS vulnerability in user name field
```

---

## Troubleshooting Guide

### Common Failure Modes

| Symptom | Root Cause | Solution |
|---------|-----------|----------|
| Spec validation fails | Invalid OpenAPI syntax | Use spectral linter |
| SDK type mismatch | Schema out of sync | Regenerate from spec |
| Missing examples | Incomplete documentation | Add request/response examples |
| Broken links | URL changes | Use relative references |

### Debug Checklist

```bash
# 1. Validate OpenAPI spec
npx @stoplight/spectral lint openapi.yaml

# 2. Generate SDK and test
openapi-generator generate -i openapi.yaml -g typescript-fetch -o ./sdk

# 3. Check for breaking changes
oasdiff breaking base.yaml head.yaml

# 4. Test examples
npx prism mock openapi.yaml
curl http://localhost:4010/api/v1/users
```

---

## Quality Checklist

- [ ] OpenAPI 3.1 spec validates without errors
- [ ] All endpoints documented with descriptions
- [ ] Request/response examples for each endpoint
- [ ] Error responses documented
- [ ] Authentication clearly explained
- [ ] Rate limiting documented
- [ ] Versioning strategy documented
- [ ] Changelog maintained
- [ ] SDKs generated and tested
- [ ] Developer portal deployed

---

**Handoff:** API design → Agent 01 | Security specs → Agent 05
