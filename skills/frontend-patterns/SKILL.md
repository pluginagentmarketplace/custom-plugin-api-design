---
name: frontend-patterns
description: Frontend development and API integration patterns for React, TypeScript, and state management
sasmp_version: "1.3.0"
bonded_agent: 06-frontend-integration
bond_type: PRIMARY_BOND
---

# Frontend Patterns Skill

## Quick Start

Modern frontend patterns for API consumption and state management.

### Data Fetching
- TanStack Query (React Query)
- SWR (Stale-While-Revalidate)
- Custom hooks

### State Management
- Zustand (Simple)
- Redux Toolkit (Enterprise)
- Context API (Component)

### GraphQL Clients
- Apollo Client
- urql

### TypeScript Integration
- Type-safe API calls
- Generated types from OpenAPI/GraphQL

## Key Patterns

### TanStack Query
```typescript
function useUsers(page: number) {
  return useQuery({
    queryKey: ['users', page],
    queryFn: async () => {
      const res = await fetch(`/api/users?page=${page}`);
      return res.json();
    },
    staleTime: 5 * 60 * 1000
  });
}
```

### Zustand Store
```typescript
const useAuthStore = create<AuthState>((set) => ({
  token: null,
  user: null,
  setAuth: (token, user) => set({ token, user }),
  logout: () => set({ token: null, user: null })
}));
```

## Performance Checklist

- [ ] Data fetching optimized
- [ ] Caching configured
- [ ] Error boundaries set
- [ ] Loading states handled
- [ ] Code splitting implemented
- [ ] Memoization applied

See Agent 6: Frontend Integration for detailed guidance.
