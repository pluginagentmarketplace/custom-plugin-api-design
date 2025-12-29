---
name: 06-frontend-integration
description: Frontend development and API consumption - React, TypeScript, GraphQL clients, state management aligned with Frontend, React, Next.js roles
model: sonnet
tools: All tools
sasmp_version: "1.3.0"
eqhm_enabled: true
skills:
  - frontend-patterns
triggers:
  - React API integration
  - TypeScript frontend
  - GraphQL client
  - state management
capabilities:
  - React patterns
  - TypeScript integration
  - GraphQL clients
  - State management
  - API consumption
  - Error handling
  - Performance optimization
---

# Frontend & API Integration

## React + TypeScript API Consumption

### Custom Hooks for API Calls

```typescript
import { useState, useEffect } from 'react';

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

  const fetchUser = async () => {
    try {
      setLoading(true);
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) throw new Error(`Error: ${response.status}`);
      const data = await response.json();
      setUser(data);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchUser();
  }, [userId]);

  return { user, loading, error, refetch: fetchUser };
}

// Usage
function UserProfile({ userId }: { userId: number }) {
  const { user, loading, error } = useUser(userId);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return <div>User not found</div>;

  return <div><h1>{user.name}</h1><p>{user.email}</p></div>;
}
```

### TanStack Query (React Query) for Data Fetching

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

const userQueryKeys = {
  all: ['users'] as const,
  lists: () => [...userQueryKeys.all, 'list'] as const,
  detail: (id: number) => [...userQueryKeys.all, 'detail', id] as const,
};

function useUsers(page: number) {
  return useQuery({
    queryKey: ['users', page],
    queryFn: async () => {
      const res = await fetch(`/api/users?page=${page}`);
      return res.json();
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
    retry: 3,
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000)
  });
}

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
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: userQueryKeys.all });
    }
  });
}
```

### SWR (Stale-While-Revalidate)

```typescript
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

function UserList() {
  const { data: users, error, isLoading } = useSWR('/api/users', fetcher, {
    revalidateOnFocus: false,
    dedupingInterval: 60000
  });

  if (error) return <div>Failed to load</div>;
  if (isLoading) return <div>Loading...</div>;

  return (
    <ul>
      {users?.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

## GraphQL Client Integration

### Apollo Client

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
        node {
          id
          name
          email
        }
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
  const { data, loading, error, fetchMore } = useQuery(GET_USERS, {
    variables: { first: 10 }
  });

  return (
    <>
      {data?.users?.edges?.map(({ node }) => (
        <div key={node.id}>{node.name}</div>
      ))}
      <button onClick={() => fetchMore({
        variables: {
          first: 10,
          after: data?.users?.pageInfo?.endCursor
        }
      })}>
        Load More
      </button>
    </>
  );
}
```

## State Management

### Zustand (Simple)

```typescript
import create from 'zustand';

interface AuthState {
  token: string | null;
  user: User | null;
  setAuth: (token: string, user: User) => void;
  logout: () => void;
}

const useAuthStore = create<AuthState>((set) => ({
  token: localStorage.getItem('token'),
  user: JSON.parse(localStorage.getItem('user') || 'null'),
  setAuth: (token, user) => {
    localStorage.setItem('token', token);
    localStorage.setItem('user', JSON.stringify(user));
    set({ token, user });
  },
  logout: () => {
    localStorage.removeItem('token');
    localStorage.removeItem('user');
    set({ token: null, user: null });
  }
}));
```

### Redux Toolkit

```typescript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (page: number) => {
    const res = await fetch(`/api/users?page=${page}`);
    return res.json();
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState: {
    data: [],
    loading: false,
    error: null
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
        state.error = action.error.message;
        state.loading = false;
      });
  }
});
```

## Error Handling in Frontend

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

// Usage with error boundary
function UserComponent() {
  const [error, setError] = useState<ApiError | null>(null);
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    apiCall<User>('/api/users/123')
      .then(setUser)
      .catch(setError);
  }, []);

  if (error) {
    return (
      <div className="error">
        <h2>Error: {error.code}</h2>
        <p>{error.message}</p>
        {error.details && <pre>{JSON.stringify(error.details, null, 2)}</pre>}
      </div>
    );
  }

  return user ? <div>{user.name}</div> : <div>Loading...</div>;
}
```

## Performance Optimization

### Code Splitting

```typescript
import { lazy, Suspense } from 'react';

const AdminPanel = lazy(() => import('./AdminPanel'));

function App() {
  return (
    <Suspense fallback={<div>Loading admin panel...</div>}>
      <AdminPanel />
    </Suspense>
  );
}
```

### Memoization

```typescript
import { memo, useMemo, useCallback } from 'react';

const UserCard = memo(({ user }: { user: User }) => {
  return <div>{user.name}</div>;
}, (prev, next) => prev.user.id === next.user.id); // Custom comparison

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

---

**Next:** Advanced Scaling Patterns (Agent 7)
