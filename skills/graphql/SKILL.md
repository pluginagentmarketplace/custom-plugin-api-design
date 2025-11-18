---
name: graphql-patterns
description: GraphQL schema design, query optimization, resolver patterns, and advanced features like subscriptions and federation
---

# GraphQL Patterns Skill

## Quick Start

Master GraphQL by learning:

### Schema Definition
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  author: User!
}

type Query {
  user(id: ID!): User
  posts(limit: Int): [Post!]!
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
}

input CreatePostInput {
  title: String!
  content: String!
}
```

### Query Examples
```graphql
# Basic query
query {
  user(id: 123) {
    name
    email
    posts { title }
  }
}

# With variables
query GetUser($id: ID!) {
  user(id: $id) {
    name
  }
}
```

### Performance Tips
1. **Use batching** to avoid N+1
2. **Limit query depth** (max 10 levels)
3. **Limit query complexity** (max 1000 points)
4. **Cache responses** appropriately
5. **Use DataLoader** for batch operations

### Subscriptions
```graphql
subscription OnPostCreated {
  postCreated {
    id
    title
  }
}
```

### Errors
```json
{
  "data": null,
  "errors": [
    {
      "message": "User not found",
      "extensions": {
        "code": "NOT_FOUND"
      }
    }
  ]
}
```

## GraphQL Best Practices

- [ ] Schema well-documented
- [ ] Query complexity limited
- [ ] N+1 queries prevented with batching
- [ ] Timeout configured
- [ ] Authorization at field level
- [ ] Introspection controlled
- [ ] Errors clearly communicated
- [ ] Subscriptions scalable

See the GraphQL Design agent for detailed patterns.
