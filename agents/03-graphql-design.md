---
description: GraphQL API design, schema creation, query optimization, and advanced patterns including federation and subscriptions
capabilities: ["Schema design", "Query optimization", "Subscription patterns", "Federation", "Caching strategies", "Performance tuning"]
---

# GraphQL Design

## What is GraphQL?

**Query Language for APIs** - Client specifies exact data needed

```graphql
query GetUserPosts {
  user(id: 123) {
    name
    email
    posts {
      id
      title
      createdAt
    }
  }
}
```

vs REST:
```
GET /api/users/123              # Get user
GET /api/users/123/posts        # Get posts
GET /api/posts/1 through 10     # Individual posts
```

## Schema Design Basics

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
  createdAt: DateTime!
}

type Comment {
  id: ID!
  text: String!
  author: User!
  post: Post!
}

type Query {
  user(id: ID!): User
  posts(limit: Int, offset: Int): [Post!]!
  search(query: String!): [SearchResult!]!
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
  updatePost(id: ID!, input: UpdatePostInput!): Post!
  deletePost(id: ID!): Boolean!
}

input CreatePostInput {
  title: String!
  content: String!
}

input UpdatePostInput {
  title: String
  content: String
}

union SearchResult = User | Post | Comment
```

## Query Examples

### Basic Query
```graphql
query {
  user(id: 123) {
    name
    email
  }
}
```

### With Arguments
```graphql
query {
  posts(limit: 10, offset: 0) {
    id
    title
    author {
      name
    }
  }
}
```

### Aliases
```graphql
query {
  recentPosts: posts(limit: 5, sort: "recent") {
    title
  }
  popularPosts: posts(limit: 5, sort: "popular") {
    title
  }
}
```

### Fragments
```graphql
fragment PostFields on Post {
  id
  title
  createdAt
  author {
    name
  }
}

query {
  post1: post(id: 1) {
    ...PostFields
  }
  post2: post(id: 2) {
    ...PostFields
  }
}
```

## Mutations

```graphql
mutation CreatePost($title: String!, $content: String!) {
  createPost(input: {
    title: $title
    content: $content
  }) {
    id
    title
    createdAt
  }
}
```

## Subscriptions

```graphql
subscription OnPostCreated {
  postCreated {
    id
    title
    author {
      name
    }
  }
}
```

## Error Handling

```json
{
  "data": {
    "user": null
  },
  "errors": [
    {
      "message": "User not found",
      "extensions": {
        "code": "NOT_FOUND",
        "path": ["user"]
      }
    }
  ]
}
```

## Federation Pattern

### Subgraph
```graphql
type User @key(fields: "id") {
  id: ID!
  name: String!
}

type Post {
  id: ID!
  title: String!
  author: User!
}
```

### Apollo Federation
```graphql
# Gateway
type Query {
  user(id: ID!): User
  post(id: ID!): Post
}

# Routes to User Service or Post Service
```

## Performance Optimization

### 1. Complexity Analysis
```graphql
# Heavy query (avoid)
query {
  users {
    posts {
      comments {
        author {
          posts {
            comments { ... }
          }
        }
      }
    }
  }
}
```

### 2. Depth Limiting
```
Max query depth: 10
Max query complexity: 1000
```

### 3. Batching & Caching
```javascript
const DataLoader = require('dataloader');

const userLoader = new DataLoader(async (userIds) => {
  return Promise.all(userIds.map(id => fetchUser(id)));
});
```

### 4. Query Timeout
```
Max execution time: 30 seconds
```

## N+1 Query Problem

### ❌ Without Batching
```
Query users → 100 users
For each user, query posts → 100 database queries
```

### ✅ With Batching
```
Query users → 100 users
Batch query posts for all users → 1 database query
```

## Caching Strategies

### HTTP Caching
```
Cache-Control: public, max-age=300
ETag: "abc123"
```

### Query Caching
```javascript
const cache = new Map();
const key = hashQuery(query, variables);
if (cache.has(key)) return cache.get(key);
```

### Server-side Caching
```graphql
directive @cached(ttl: Int) on FIELD_DEFINITION

type Query {
  user(id: ID!): User @cached(ttl: 3600)
}
```

## Best Practices

1. **Use TypeScript** for type safety
2. **Implement Authorization** at field level
3. **Monitor Query Complexity** to prevent abuse
4. **Batch Data Requests** to avoid N+1
5. **Cache Aggressively** with proper invalidation
6. **Use Persisted Queries** for security and performance
7. **Monitor Performance** with APM tools
8. **Document Schema** thoroughly

## Design Checklist

- [ ] Clear schema structure
- [ ] Proper type definitions
- [ ] Authorization implemented
- [ ] Complexity limits set
- [ ] Batching implemented
- [ ] Caching strategy defined
- [ ] Error handling complete
- [ ] Performance tested
- [ ] Documentation complete

Next: Secure your API with the API Security & Authentication agent.
