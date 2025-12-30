---
name: 06-frontend-integration
description: Frontend development and API consumption - React, TypeScript, GraphQL clients, state management aligned with Frontend, React, Next.js roles
model: sonnet
tools: All tools
sasmp_version: "2.0.0"
eqhm_enabled: true
skills:
  - frontend-patterns
triggers:
  - React API integration
  - TypeScript frontend
  - GraphQL client
  - state management
  - Next.js
  - TanStack Query
capabilities:
  - React patterns
  - TypeScript integration
  - GraphQL clients
  - State management
  - API consumption
  - Error handling
  - Performance optimization

# Production-Grade Metadata
input_schema:
  type: object
  required: [framework]
  properties:
    framework:
      type: string
      enum: [react, nextjs, vue, angular, svelte]
    api_type:
      type: string
      enum: [REST, GraphQL, both]
    state_management:
      type: string
      enum: [tanstack-query, swr, redux, zustand, context]
    typescript:
      type: boolean
      default: true

output_schema:
  type: object
  properties:
    implementation:
      type: object
      properties:
        components: { type: array }
        hooks: { type: array }
        types: { type: array }
    dependencies:
      type: array
      items: { type: string }
    patterns:
      type: array
      items: { type: string }

error_codes:
  - code: API_FETCH_FAILED
    message: API request failed
    recovery: Check network, verify endpoint, handle error state
  - code: TYPE_MISMATCH
    message: API response doesn't match expected type
    recovery: Update types or validate response
  - code: STATE_SYNC_ERROR
    message: State synchronization failed
    recovery: Invalidate cache, refetch data

fallback_strategy:
  type: graceful_degradation
  fallback_to: error_ui
  actions:
    - Show error boundary
    - Display cached data if available
    - Provide retry action

token_budget:
  max_input: 8000
  max_output: 16000
  context_window: 100000

cost_tier: standard

retry_config:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

observability:
  log_level: info
  metrics:
    - component_render_time
    - api_latency
    - cache_hit_rate
    - error_rate
  trace_enabled: true
---

# Frontend & API Integration

## React + TypeScript API Consumption

### Custom Hooks Pattern

```typescript
import { useState, useEffect, useCallback } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UseUserResult {
  user: User | null;
  loading: boolean;
  error: Error | null;
  refetch: () => Promise<void>;
}

function useUser(userId: number): UseUserResult {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchUser = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) throw new Error(`Error: ${response.status}`);
      setUser(await response.json());
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  }, [userId]);

  useEffect(() => {
    fetchUser();
  }, [fetchUser]);

  return { user, loading, error, refetch: fetchUser };
}
```

### TanStack Query (React Query)

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Query keys factory
const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: string) => [...userKeys.lists(), { filters }] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: number) => [...userKeys.details(), id] as const,
};

// Fetch hook with proper typing
function useUsers(page: number) {
  return useQuery({
    queryKey: userKeys.list(`page=${page}`),
    queryFn: async () => {
      const res = await fetch(`/api/users?page=${page}`);
      if (!res.ok) throw new Error('Failed to fetch users');
      return res.json() as Promise<User[]>;
    },
    staleTime: 5 * 60 * 1000,
    retry: 3,
    retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30000),
  });
}

// Mutation with cache invalidation
function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (user: User) => {
      const res = await fetch(`/api/users/${user.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(user),
      });
      if (!res.ok) throw new Error('Failed to update user');
      return res.json();
    },
    onSuccess: (data, variables) => {
      queryClient.setQueryData(userKeys.detail(variables.id), data);
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
    },
  });
}
```

### SWR Pattern

```typescript
import useSWR from 'swr';

const fetcher = async (url: string) => {
  const res = await fetch(url);
  if (!res.ok) throw new Error('Fetch failed');
  return res.json();
};

function UserList() {
  const { data: users, error, isLoading, mutate } = useSWR<User[]>(
    '/api/users',
    fetcher,
    {
      revalidateOnFocus: false,
      dedupingInterval: 60000,
      errorRetryCount: 3,
    }
  );

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {users?.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

## GraphQL Client Integration

### Apollo Client Setup

```typescript
import { ApolloClient, InMemoryCache, gql, useQuery } from '@apollo/client';

const client = new ApolloClient({
  uri: 'https://api.example.com/graphql',
  cache: new InMemoryCache(),
  defaultOptions: {
    watchQuery: { fetchPolicy: 'cache-and-network' },
  },
});

const GET_USERS = gql`
  query GetUsers($first: Int!, $after: String) {
    users(first: $first, after: $after) {
      edges {
        node { id name email }
        cursor
      }
      pageInfo { hasNextPage endCursor }
    }
  }
`;

function UserList() {
  const { data, loading, error, fetchMore } = useQuery(GET_USERS, {
    variables: { first: 10 },
  });

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <>
      {data?.users?.edges?.map(({ node }) => (
        <div key={node.id}>{node.name}</div>
      ))}
      {data?.users?.pageInfo?.hasNextPage && (
        <button onClick={() => fetchMore({
          variables: { after: data.users.pageInfo.endCursor }
        })}>
          Load More
        </button>
      )}
    </>
  );
}
```

## State Management

### Zustand (Lightweight)

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  token: string | null;
  user: User | null;
  setAuth: (token: string, user: User) => void;
  logout: () => void;
}

const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      user: null,
      setAuth: (token, user) => set({ token, user }),
      logout: () => set({ token: null, user: null }),
    }),
    { name: 'auth-storage' }
  )
);
```

### Redux Toolkit

```typescript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (page: number, { rejectWithValue }) => {
    try {
      const res = await fetch(`/api/users?page=${page}`);
      if (!res.ok) throw new Error('Failed to fetch');
      return res.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState: { data: [], loading: false, error: null as string | null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.data = action.payload;
        state.loading = false;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.error = action.payload as string;
        state.loading = false;
      });
  },
});
```

## Error Handling

### Type-Safe API Client

```typescript
interface ApiError {
  code: string;
  message: string;
  status: number;
  details?: Record<string, unknown>;
}

async function apiCall<T>(url: string, options?: RequestInit): Promise<T> {
  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${localStorage.getItem('token')}`,
      ...options?.headers,
    },
  });

  const data = await response.json();

  if (!response.ok) {
    const error: ApiError = {
      code: data.errors?.[0]?.code || 'UNKNOWN_ERROR',
      message: data.errors?.[0]?.message || 'Unknown error',
      status: response.status,
      details: data.errors?.[0]?.details,
    };
    throw error;
  }

  return data;
}
```

### Error Boundary

```typescript
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <div>Something went wrong</div>;
    }
    return this.props.children;
  }
}
```

## Performance Optimization

### Code Splitting

```typescript
import { lazy, Suspense } from 'react';

const AdminPanel = lazy(() => import('./AdminPanel'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <AdminPanel />
    </Suspense>
  );
}
```

### Memoization

```typescript
import { memo, useMemo, useCallback } from 'react';

const UserCard = memo(function UserCard({ user }: { user: User }) {
  return <div>{user.name}</div>;
});

function UserList({ users }: { users: User[] }) {
  const sortedUsers = useMemo(
    () => [...users].sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  );

  const handleSelect = useCallback((id: number) => {
    console.log('Selected:', id);
  }, []);

  return (
    <>
      {sortedUsers.map(u => (
        <UserCard key={u.id} user={u} />
      ))}
    </>
  );
}
```

## Frontend Checklist

- [ ] Type-safe API client
- [ ] Data fetching library configured
- [ ] Error boundaries set up
- [ ] Loading states handled
- [ ] Code splitting implemented
- [ ] Memoization applied where needed
- [ ] Form validation in place
- [ ] Accessibility (a11y) checked

---

## Troubleshooting

### Common Failure Modes

| Error | Root Cause | Recovery |
|-------|------------|----------|
| Network error | API unreachable | Check connectivity, show offline UI |
| 401 response | Token expired | Redirect to login, refresh token |
| Stale data | Cache not invalidated | Force refetch, check cache keys |
| Hydration mismatch | SSR/client mismatch | Check server vs client rendering |

### Debug Checklist

1. [ ] Check network tab in DevTools
2. [ ] Verify API response format
3. [ ] Inspect React Query DevTools
4. [ ] Check component re-renders
5. [ ] Profile with React DevTools

### Log Interpretation

```
WARN: QUERY_STALE queryKey=["users","list"] staleTime=0
  → Query needs refetch

ERROR: MUTATION_FAILED mutation=updateUser error=NetworkError
  → API call failed, check connectivity

INFO: CACHE_HIT queryKey=["users","detail",123]
  → Data served from cache, no network request
```

---

**Next:** Advanced Scaling Patterns (Agent 7)
