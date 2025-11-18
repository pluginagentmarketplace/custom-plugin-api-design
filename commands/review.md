# Review Your API Design

Validate your API design against best practices and catch issues early.

## What to Review

Provide Claude Code with:
1. **Your API specification** (OpenAPI, GraphQL schema, or description)
2. **Endpoints/Queries** - List of available endpoints or queries
3. **Data models** - How data is structured
4. **Authentication/Authorization** - Security approach
5. **Performance considerations** - Caching, pagination, etc.

## Review Checks

Claude Code will check:

### ✅ Design Quality
- [ ] Resources properly modeled
- [ ] HTTP methods used correctly (if REST)
- [ ] Naming conventions consistent
- [ ] Relationships handled properly

### ✅ Functionality
- [ ] All requirements covered
- [ ] Edge cases considered
- [ ] Error handling complete
- [ ] Response formats consistent

### ✅ API Maturity
- [ ] Versioning strategy
- [ ] Deprecation plan
- [ ] Backward compatibility
- [ ] Migration path

### ✅ Developer Experience
- [ ] Endpoint naming clear
- [ ] Query parameters intuitive
- [ ] Error messages helpful
- [ ] Documentation complete

### ✅ Performance
- [ ] Pagination strategy
- [ ] Caching opportunities
- [ ] Database queries optimized
- [ ] Response times acceptable

### ✅ Security
- [ ] Authentication required
- [ ] Authorization checks
- [ ] Input validation
- [ ] Sensitive data protected

## How to Share Your Design

### Option 1: OpenAPI/Swagger
```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
paths:
  /users: ...
```

### Option 2: GraphQL Schema
```graphql
type User {
  id: ID!
  name: String!
}
type Query {
  user(id: ID!): User
}
```

### Option 3: Description
Just describe your API endpoints and data model.

## Get Feedback

After review, you'll get:
- Issues found (with severity)
- Suggestions for improvement
- Examples of better patterns
- Links to relevant documentation

## Next Steps

1. Fix issues from the review
2. Use `/security-check` for security audit
3. Use `/performance-guide` for optimization
