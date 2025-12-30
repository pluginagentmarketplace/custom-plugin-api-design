---
name: graphql
description: GraphQL API design and schema development
sasmp_version: "2.0.0"
bonded_agent: 01-api-architect
bond_type: SECONDARY_BOND

# Production-Grade Metadata
parameters:
  - name: operation_type
    type: string
    required: true
    validation: "^(query|mutation|subscription)$"
    description: GraphQL operation type
  - name: schema
    type: string
    required: false
    description: Existing schema to extend

validation_rules:
  - Types must have clear naming conventions
  - Mutations must return affected objects
  - N+1 must be prevented with DataLoader

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [query_complexity, resolver_duration]
---

# GraphQL API Design Skill

## Topics Covered
- Schema design, types, resolvers
- Queries, mutations, subscriptions
- N+1 problem, DataLoader, caching

## Schema Design

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Query {
  user(id: ID!): User
  users(first: Int, after: String): UserConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
}

type Subscription {
  userCreated: User!
}
```

## Resolver Pattern

```javascript
const resolvers = {
  Query: {
    user: (_, { id }) => db.users.findById(id),
    users: (_, { first, after }) => db.users.paginate({ first, after })
  },
  User: {
    posts: (user, _, { loaders }) => loaders.posts.load(user.id)
  }
};
```

## DataLoader (N+1 Prevention)

```javascript
const postsLoader = new DataLoader(async (userIds) => {
  const posts = await db.posts.findByUserIds(userIds);
  return userIds.map(id => posts.filter(p => p.userId === id));
});
```

## Unit Test Template

```javascript
describe('graphql skill', () => {
  test('query returns user', async () => {
    const result = await graphql({
      schema,
      source: `query { user(id: "1") { id name } }`
    });
    expect(result.data.user).toBeDefined();
  });

  test('mutation creates user', async () => {
    const result = await graphql({
      schema,
      source: `mutation { createUser(input: {name: "Test"}) { user { id } } }`
    });
    expect(result.data.createUser.user.id).toBeDefined();
  });
});
```

## Learning Outcomes
- Design GraphQL schemas
- Optimize resolvers
- Implement subscriptions

See Agent 1: API Architect for detailed guidance.
