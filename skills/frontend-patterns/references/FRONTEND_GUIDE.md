# Frontend API Integration Complete Guide

Modern patterns for React, TypeScript, and state management.

## Data Fetching Libraries

### TanStack Query (React Query)
```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Query keys factory
const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  detail: (id: number) => [...userKeys.all, 'detail', id] as const,
};

// Custom hook
function useUsers(page: number) {
  return useQuery({
    queryKey: userKeys.lists(),
    queryFn: async () => {
      const res = await fetch(`/api/users?page=${page}`);
      if (!res.ok) throw new Error('Failed to fetch');
      return res.json();
    },
    staleTime: 5 * 60 * 1000,  // 5 minutes
    retry: 3,
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000)
  });
}

// Mutation with cache invalidation
function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (user: User) => {
      const res = await fetch(`/api/users/${user.id}`, {
        method: 'PUT',
        body: JSON.stringify(user)
      });
      return res.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: userKeys.all });
    }
  });
}
```

### SWR
```typescript
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

function UserList() {
  const { data, error, isLoading, mutate } = useSWR('/api/users', fetcher, {
    revalidateOnFocus: false,
    dedupingInterval: 60000,
    errorRetryCount: 3
  });

  if (error) return <div>Failed to load</div>;
  if (isLoading) return <div>Loading...</div>;

  return (
    <ul>
      {data?.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

## State Management

### Zustand (Recommended for Simplicity)
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
      logout: () => set({ token: null, user: null })
    }),
    { name: 'auth-storage' }
  )
);

// Usage
function Profile() {
  const { user, logout } = useAuthStore();
  return (
    <div>
      <span>{user?.name}</span>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

### Redux Toolkit
```typescript
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (page: number) => {
    const response = await fetch(`/api/users?page=${page}`);
    return response.json();
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState: {
    data: [] as User[],
    loading: false,
    error: null as string | null
  },
  reducers: {
    addUser: (state, action: PayloadAction<User>) => {
      state.data.push(action.payload);
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.data = action.payload;
        state.loading = false;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.error = action.error.message ?? 'Failed';
        state.loading = false;
      });
  }
});
```

## Type-Safe API Client

### Custom API Wrapper
```typescript
interface ApiError {
  code: string;
  message: string;
  status: number;
  details?: Record<string, unknown>;
}

async function apiCall<T>(
  url: string,
  options?: RequestInit
): Promise<T> {
  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${localStorage.getItem('token')}`,
      ...options?.headers
    }
  });

  const data = await response.json();

  if (!response.ok) {
    const error: ApiError = {
      code: data.errors?.[0]?.code || 'UNKNOWN_ERROR',
      message: data.errors?.[0]?.message || 'Unknown error',
      status: response.status,
      details: data.errors?.[0]?.details
    };
    throw error;
  }

  return data;
}

// Usage
const user = await apiCall<User>('/api/users/123');
```

## GraphQL with Apollo

### Setup
```typescript
import { ApolloClient, InMemoryCache, gql, useQuery } from '@apollo/client';

const client = new ApolloClient({
  uri: 'https://api.example.com/graphql',
  cache: new InMemoryCache(),
  headers: {
    authorization: localStorage.getItem('token') || ''
  }
});

const GET_USERS = gql`
  query GetUsers($first: Int!, $after: String) {
    users(first: $first, after: $after) {
      edges {
        node { id name email }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

function UserList() {
  const { data, loading, fetchMore } = useQuery(GET_USERS, {
    variables: { first: 10 }
  });

  if (loading) return <div>Loading...</div>;

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

const UserCard = memo(({ user, onSelect }: Props) => {
  return <div onClick={() => onSelect(user.id)}>{user.name}</div>;
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
        <UserCard key={u.id} user={u} onSelect={handleSelect} />
      ))}
    </>
  );
}
```

## Error Handling

### Error Boundary
```typescript
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <h2>Something went wrong</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <UserList />
    </ErrorBoundary>
  );
}
```

## Frontend Checklist

### Data Fetching
- [ ] Caching strategy configured
- [ ] Error handling implemented
- [ ] Loading states handled
- [ ] Pagination working

### State Management
- [ ] Global state organized
- [ ] Persistence configured
- [ ] DevTools enabled

### Performance
- [ ] Code splitting implemented
- [ ] Memoization applied
- [ ] Bundle size optimized
- [ ] Lazy loading enabled

### TypeScript
- [ ] API types generated
- [ ] Strict mode enabled
- [ ] No `any` types

## Resources

- [TanStack Query Docs](https://tanstack.com/query/latest)
- [Zustand GitHub](https://github.com/pmndrs/zustand)
- [Apollo Client](https://www.apollographql.com/docs/react/)
- [React Performance](https://react.dev/learn/render-and-commit)
