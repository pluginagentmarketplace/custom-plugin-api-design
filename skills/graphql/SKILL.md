---
name: graphql
version: "2.0.0"
description: GraphQL API design and schema development
sasmp_version: "1.3.0"
bonded_agent: 01-api-architect
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "GraphQL schema design and resolver patterns"
  boundaries:
    includes: [schema_design, resolvers, subscriptions, n_plus_one, caching]
    excludes: [rest_design, database_queries, frontend_code]

parameter_validation:
  schema:
    type: object
    properties:
      operation_type:
        type: string
        enum: [query, mutation, subscription]
      complexity_limit:
        type: number
        minimum: 0
        maximum: 1000

retry_config:
  enabled: true
  max_attempts: 3
  backoff:
    type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 30000

logging:
  level: INFO
  fields: [operation_name, complexity, duration_ms]

dependencies:
  skills: [api-architecture]
  agents: [01-api-architect]
---

# GraphQL API Design Skill

## Purpose
Design efficient GraphQL APIs with proper schema patterns.

## Schema Design

### Types

```graphql
# Scalar types
scalar DateTime
scalar UUID
scalar Email

# Object type
type User {
  id: ID!
  email: Email!
  name: String!
  status: UserStatus!
  profile: Profile
  teams: [Team!]!
  createdAt: DateTime!
  updatedAt: DateTime
}

# Enum
enum UserStatus {
  ACTIVE
  INACTIVE
  BANNED
}

# Interface
interface Node {
  id: ID!
}

# Union
union SearchResult = User | Team | Post
```

### Queries

```graphql
type Query {
  # Single resource
  user(id: ID!): User

  # Paginated list (Relay-style)
  users(
    first: Int
    after: String
    last: Int
    before: String
    filter: UserFilter
  ): UserConnection!

  # Search
  search(query: String!, types: [SearchType!]): [SearchResult!]!
}

# Filter input
input UserFilter {
  status: UserStatus
  role: String
  createdAfter: DateTime
}
```

### Mutations

```graphql
type Mutation {
  # Create
  createUser(input: CreateUserInput!): CreateUserPayload!

  # Update
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!

  # Delete
  deleteUser(id: ID!): DeleteUserPayload!

  # Action
  verifyUser(id: ID!): VerifyUserPayload!
}

# Input types
input CreateUserInput {
  email: Email!
  name: String!
  password: String!
}

input UpdateUserInput {
  name: String
  status: UserStatus
}

# Payload types (with errors)
type CreateUserPayload {
  user: User
  errors: [UserError!]!
}

type UserError {
  field: String
  message: String!
  code: ErrorCode!
}
```

### Subscriptions

```graphql
type Subscription {
  # Real-time updates
  userCreated: User!
  userUpdated(id: ID): User!

  # Filtered subscription
  orderStatusChanged(orderId: ID!): Order!
}
```

## Connection Pattern (Relay)

```graphql
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

## Resolver Patterns

### Basic Resolver

```typescript
const resolvers = {
  Query: {
    user: async (_, { id }, context) => {
      return context.dataSources.users.findById(id);
    },

    users: async (_, { first, after, filter }, context) => {
      return context.dataSources.users.findMany({
        first,
        after,
        filter,
      });
    },
  },

  User: {
    // Field resolver
    teams: async (user, _, context) => {
      return context.dataSources.teams.findByUserId(user.id);
    },
  },
};
```

### DataLoader (N+1 Solution)

```typescript
import DataLoader from 'dataloader';

// Create loader
const userLoader = new DataLoader(async (ids: string[]) => {
  const users = await db.query(
    'SELECT * FROM users WHERE id = ANY($1)',
    [ids]
  );

  // Return in same order as input ids
  const userMap = new Map(users.map(u => [u.id, u]));
  return ids.map(id => userMap.get(id) || null);
});

// Use in resolver
const resolvers = {
  Post: {
    author: (post, _, context) => {
      return context.loaders.user.load(post.authorId);
    },
  },
};
```

### Context Setup

```typescript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => ({
    user: req.user,
    loaders: {
      user: new DataLoader(batchUsers),
      team: new DataLoader(batchTeams),
    },
    dataSources: {
      users: new UserDataSource(db),
      teams: new TeamDataSource(db),
    },
  }),
});
```

## Error Handling

```typescript
import { GraphQLError } from 'graphql';

// Custom error
throw new GraphQLError('User not found', {
  extensions: {
    code: 'NOT_FOUND',
    field: 'userId',
  },
});

// Error formatting
const server = new ApolloServer({
  formatError: (error) => {
    // Log internal errors
    if (error.extensions?.code === 'INTERNAL_SERVER_ERROR') {
      logger.error(error);
      return { message: 'Internal server error' };
    }
    return error;
  },
});
```

## Security

### Query Complexity

```typescript
import { createComplexityRule } from 'graphql-query-complexity';

const complexityRule = createComplexityRule({
  maximumComplexity: 1000,
  estimators: [
    fieldExtensionsEstimator(),
    simpleEstimator({ defaultComplexity: 1 }),
  ],
  onComplete: (complexity) => {
    console.log('Query complexity:', complexity);
  },
});

const server = new ApolloServer({
  validationRules: [complexityRule],
});
```

### Depth Limiting

```typescript
import depthLimit from 'graphql-depth-limit';

const server = new ApolloServer({
  validationRules: [depthLimit(10)],
});
```

---

## Unit Test Template

```typescript
import { describe, it, expect } from 'vitest';
import { ApolloServer } from '@apollo/server';
import { typeDefs, resolvers } from './schema';

describe('GraphQL API', () => {
  const server = new ApolloServer({ typeDefs, resolvers });

  describe('Query.user', () => {
    it('should return user by id', async () => {
      const result = await server.executeOperation({
        query: `
          query GetUser($id: ID!) {
            user(id: $id) {
              id
              name
              email
            }
          }
        `,
        variables: { id: 'user-123' },
      });

      expect(result.body.singleResult.data?.user).toEqual({
        id: 'user-123',
        name: 'John Doe',
        email: 'john@example.com',
      });
    });

    it('should return null for non-existent user', async () => {
      const result = await server.executeOperation({
        query: `query { user(id: "invalid") { id } }`,
      });

      expect(result.body.singleResult.data?.user).toBeNull();
    });
  });

  describe('Mutation.createUser', () => {
    it('should create user and return payload', async () => {
      const result = await server.executeOperation({
        query: `
          mutation CreateUser($input: CreateUserInput!) {
            createUser(input: $input) {
              user { id name }
              errors { field message }
            }
          }
        `,
        variables: {
          input: { email: 'new@example.com', name: 'New User', password: 'Secret123!' },
        },
      });

      expect(result.body.singleResult.data?.createUser.user).toBeDefined();
      expect(result.body.singleResult.data?.createUser.errors).toEqual([]);
    });
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| N+1 queries | Field-level resolvers | Use DataLoader |
| Slow queries | High complexity | Add complexity limits |
| Memory issues | Large result sets | Implement pagination |
| Introspection leak | Enabled in production | Disable in prod |

---

## Quality Checklist

- [ ] Schema follows naming conventions
- [ ] Relay connection pattern for lists
- [ ] Input/Payload types for mutations
- [ ] DataLoader for relationships
- [ ] Query complexity limits set
- [ ] Depth limiting enabled
- [ ] Introspection disabled in production
- [ ] Error handling standardized
