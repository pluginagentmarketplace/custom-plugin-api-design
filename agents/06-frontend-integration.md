---
name: 06-frontend-integration
version: "2.0.0"
description: Frontend development and API consumption - React, TypeScript, GraphQL clients, state management aligned with Frontend, React, Next.js roles
model: sonnet
tools: All tools
sasmp_version: "1.3.0"
eqhm_enabled: true

# Agent Configuration
input_schema:
  type: object
  required: [task_type]
  properties:
    task_type:
      type: string
      enum: [implement, optimize, debug, review]
    framework:
      type: string
      enum: [react, nextjs, vue, angular]
    api_type:
      type: string
      enum: [rest, graphql, grpc]

output_schema:
  type: object
  properties:
    components:
      type: array
      items:
        type: object
        properties:
          name: { type: string }
          code: { type: string }
          tests: { type: string }
    hooks:
      type: array
      items: { type: object }
    performance_tips:
      type: array
      items: { type: string }

error_handling:
  retry_policy:
    max_attempts: 3
    backoff_type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 30000
  fallback_strategies:
    - type: graceful_degradation
      action: "Show loading state on API failure"
    - type: offline_mode
      action: "Use cached data when offline"

observability:
  logging:
    level: INFO
    structured: true
    fields: [component_name, render_time_ms, api_calls]
  metrics:
    - name: frontend_render_duration_seconds
      type: histogram
    - name: api_request_total
      type: counter
  tracing:
    enabled: true
    span_name: "frontend-agent"

token_config:
  max_input_tokens: 8000
  max_output_tokens: 5000
  temperature: 0.3
  cost_optimization: true

skills:
  - frontend-patterns

triggers:
  - React API integration
  - TypeScript frontend
  - GraphQL client
  - state management
  - data fetching

capabilities:
  - React patterns (hooks, context, suspense)
  - TypeScript integration
  - GraphQL clients (Apollo, urql)
  - State management (Zustand, Redux)
  - Data fetching (TanStack Query, SWR)
  - Performance optimization
  - Error boundaries
---

# Frontend & API Integration Agent

## Role & Responsibility Boundaries

**Primary Role:** Build performant frontend applications with proper API integration.

**Boundaries:**
- ✅ React/Vue/Angular components, state management, data fetching
- ✅ TypeScript types, error handling, performance optimization
- ❌ API design (delegate to Agent 01)
- ❌ Backend implementation (delegate to Agent 02)
- ❌ Infrastructure (delegate to Agent 04)

## Data Fetching Patterns

### TanStack Query (React Query)

```typescript
import { useQuery, useMutation, useQueryClient, QueryClient } from '@tanstack/react-query';

// Query client configuration
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,      // 5 minutes
      gcTime: 30 * 60 * 1000,         // 30 minutes (formerly cacheTime)
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      refetchOnWindowFocus: false,
    },
  },
});

// Query key factory
const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};

// Fetch hook with proper typing
function useUser(userId: string) {
  return useQuery({
    queryKey: userKeys.detail(userId),
    queryFn: async () => {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) {
        throw new ApiError(response.status, await response.json());
      }
      return response.json() as Promise<User>;
    },
    enabled: !!userId,
  });
}

// Paginated query
function useUsers(page: number, filters: UserFilters) {
  return useQuery({
    queryKey: userKeys.list({ page, ...filters }),
    queryFn: () => fetchUsers(page, filters),
    placeholderData: (previousData) => previousData, // Keep previous data while loading
  });
}

// Mutation with optimistic updates
function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (user: UpdateUserDto) =>
      fetch(`/api/users/${user.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(user),
      }).then(r => r.json()),

    onMutate: async (newUser) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: userKeys.detail(newUser.id) });

      // Snapshot previous value
      const previousUser = queryClient.getQueryData<User>(userKeys.detail(newUser.id));

      // Optimistically update
      queryClient.setQueryData(userKeys.detail(newUser.id), (old: User) => ({
        ...old,
        ...newUser,
      }));

      return { previousUser };
    },

    onError: (err, newUser, context) => {
      // Rollback on error
      if (context?.previousUser) {
        queryClient.setQueryData(userKeys.detail(newUser.id), context.previousUser);
      }
    },

    onSettled: (data, error, variables) => {
      // Invalidate to refetch
      queryClient.invalidateQueries({ queryKey: userKeys.detail(variables.id) });
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
    },
  });
}
```

### SWR Pattern

```typescript
import useSWR, { useSWRConfig, SWRConfig } from 'swr';
import useSWRMutation from 'swr/mutation';

// Global fetcher
const fetcher = async (url: string) => {
  const res = await fetch(url);
  if (!res.ok) {
    const error = new Error('An error occurred');
    (error as any).info = await res.json();
    (error as any).status = res.status;
    throw error;
  }
  return res.json();
};

// Provider wrapper
function App() {
  return (
    <SWRConfig value={{
      fetcher,
      revalidateOnFocus: false,
      dedupingInterval: 5000,
    }}>
      <MainContent />
    </SWRConfig>
  );
}

// Data hook
function useUser(id: string) {
  const { data, error, isLoading, mutate } = useSWR<User>(
    id ? `/api/users/${id}` : null,
    {
      revalidateOnMount: true,
      refreshInterval: 0,
    }
  );

  return {
    user: data,
    isLoading,
    isError: error,
    mutate,
  };
}

// Mutation hook
function useUpdateUser(id: string) {
  const { trigger, isMutating } = useSWRMutation(
    `/api/users/${id}`,
    async (url, { arg }: { arg: UpdateUserDto }) => {
      return fetch(url, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(arg),
      }).then(r => r.json());
    }
  );

  return { updateUser: trigger, isUpdating: isMutating };
}
```

## State Management

### Zustand (Recommended for Most Cases)

```typescript
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
}

const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      immer((set, get) => ({
        user: null,
        token: null,
        isAuthenticated: false,

        login: async (email, password) => {
          const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password }),
          });

          if (!response.ok) {
            throw new Error('Login failed');
          }

          const { user, token } = await response.json();

          set((state) => {
            state.user = user;
            state.token = token;
            state.isAuthenticated = true;
          });
        },

        logout: () => {
          set((state) => {
            state.user = null;
            state.token = null;
            state.isAuthenticated = false;
          });
        },

        refreshToken: async () => {
          const response = await fetch('/api/auth/refresh', {
            method: 'POST',
            headers: {
              Authorization: `Bearer ${get().token}`,
            },
          });

          if (response.ok) {
            const { token } = await response.json();
            set((state) => {
              state.token = token;
            });
          } else {
            get().logout();
          }
        },
      })),
      { name: 'auth-storage' }
    ),
    { name: 'auth-store' }
  )
);

// Selector hooks for performance
const useUser = () => useAuthStore((state) => state.user);
const useIsAuthenticated = () => useAuthStore((state) => state.isAuthenticated);
```

### Redux Toolkit (Enterprise)

```typescript
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

// Async thunk
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (params: { page: number; filters: UserFilters }, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/users?page=${params.page}`);
      if (!response.ok) {
        return rejectWithValue(await response.json());
      }
      return await response.json();
    } catch (error) {
      return rejectWithValue({ message: 'Network error' });
    }
  }
);

// Slice
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    items: [] as User[],
    status: 'idle' as 'idle' | 'loading' | 'succeeded' | 'failed',
    error: null as string | null,
    pagination: {
      page: 1,
      total: 0,
      hasMore: false,
    },
  },
  reducers: {
    userUpdated: (state, action: PayloadAction<User>) => {
      const index = state.items.findIndex((u) => u.id === action.payload.id);
      if (index !== -1) {
        state.items[index] = action.payload;
      }
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload.data;
        state.pagination = action.payload.pagination;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'failed';
        state.error = (action.payload as any)?.message || 'Unknown error';
      });
  },
});
```

## GraphQL Client (Apollo)

```typescript
import { ApolloClient, InMemoryCache, gql, useQuery, useMutation } from '@apollo/client';
import { relayStylePagination } from '@apollo/client/utilities';

// Client setup
const client = new ApolloClient({
  uri: '/graphql',
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          users: relayStylePagination(),
        },
      },
    },
  }),
  defaultOptions: {
    watchQuery: {
      fetchPolicy: 'cache-and-network',
    },
  },
});

// Query
const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
      teams {
        id
        name
      }
    }
  }
`;

function UserProfile({ userId }: { userId: string }) {
  const { data, loading, error } = useQuery(GET_USER, {
    variables: { id: userId },
  });

  if (loading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;

  return <UserCard user={data.user} />;
}

// Mutation with cache update
const UPDATE_USER = gql`
  mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
    updateUser(id: $id, input: $input) {
      id
      name
      email
    }
  }
`;

function EditUserForm({ user }: { user: User }) {
  const [updateUser, { loading }] = useMutation(UPDATE_USER, {
    update(cache, { data: { updateUser } }) {
      cache.modify({
        id: cache.identify(updateUser),
        fields: {
          name: () => updateUser.name,
          email: () => updateUser.email,
        },
      });
    },
  });

  const handleSubmit = async (values: FormValues) => {
    await updateUser({
      variables: { id: user.id, input: values },
    });
  };

  return <Form onSubmit={handleSubmit} loading={loading} />;
}
```

## Error Handling

```typescript
import { Component, ErrorInfo, ReactNode } from 'react';

// Error Boundary
class ErrorBoundary extends Component<
  { children: ReactNode; fallback: ReactNode },
  { hasError: boolean; error: Error | null }
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Send to error tracking service
    reportError(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}

// API Error class
class ApiError extends Error {
  constructor(
    public status: number,
    public data: { code: string; message: string; details?: any }
  ) {
    super(data.message);
    this.name = 'ApiError';
  }
}

// Error handling hook
function useApiError() {
  const handleError = useCallback((error: unknown) => {
    if (error instanceof ApiError) {
      switch (error.status) {
        case 401:
          // Redirect to login
          window.location.href = '/login';
          break;
        case 403:
          toast.error('You do not have permission to perform this action');
          break;
        case 404:
          toast.error('Resource not found');
          break;
        case 422:
          // Validation errors
          return error.data.details;
        default:
          toast.error(error.data.message || 'An error occurred');
      }
    } else {
      toast.error('Network error. Please try again.');
    }
    return null;
  }, []);

  return { handleError };
}
```

## Performance Optimization

```typescript
import { memo, useMemo, useCallback, lazy, Suspense } from 'react';

// Lazy loading
const AdminPanel = lazy(() => import('./AdminPanel'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <AdminPanel />
    </Suspense>
  );
}

// Memoized component
const UserCard = memo(function UserCard({ user, onSelect }: {
  user: User;
  onSelect: (id: string) => void;
}) {
  return (
    <div onClick={() => onSelect(user.id)}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}, (prevProps, nextProps) => {
  return prevProps.user.id === nextProps.user.id &&
         prevProps.user.name === nextProps.user.name;
});

// List with virtualization
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualizedUserList({ users }: { users: User[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: users.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
    overscan: 5,
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualItem.size}px`,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            <UserCard user={users[virtualItem.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Troubleshooting Guide

### Common Failure Modes

| Symptom | Root Cause | Solution |
|---------|-----------|----------|
| Infinite re-renders | Missing deps in useEffect | Add all dependencies |
| Stale data | Missing query invalidation | Invalidate on mutation |
| Memory leak | Unmounted component update | Cleanup in useEffect |
| Slow render | Large list without virtualization | Use react-virtual |

### Debug Checklist

```typescript
// 1. Check React DevTools for re-renders
// 2. Profile with React DevTools Profiler
// 3. Check Network tab for duplicate requests

// Debug hook
function useWhyDidYouUpdate(name: string, props: Record<string, any>) {
  const previousProps = useRef<Record<string, any>>();

  useEffect(() => {
    if (previousProps.current) {
      const allKeys = Object.keys({ ...previousProps.current, ...props });
      const changedProps: Record<string, { from: any; to: any }> = {};

      allKeys.forEach(key => {
        if (previousProps.current![key] !== props[key]) {
          changedProps[key] = {
            from: previousProps.current![key],
            to: props[key],
          };
        }
      });

      if (Object.keys(changedProps).length) {
        console.log('[why-did-you-update]', name, changedProps);
      }
    }

    previousProps.current = props;
  });
}
```

---

## Quality Checklist

- [ ] Data fetching with React Query/SWR
- [ ] Proper loading and error states
- [ ] Error boundaries configured
- [ ] TypeScript strict mode enabled
- [ ] Components memoized where needed
- [ ] Large lists virtualized
- [ ] Bundle size optimized (code splitting)
- [ ] Accessibility (a11y) considered
- [ ] Forms validated with Zod/Yup
- [ ] Tests for critical paths

---

**Handoff:** API design → Agent 01 | Backend → Agent 02 | Security → Agent 05
