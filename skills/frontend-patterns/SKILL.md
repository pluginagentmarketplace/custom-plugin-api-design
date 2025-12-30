---
name: frontend-patterns
description: Frontend development and API integration patterns for React, TypeScript, and state management
sasmp_version: "2.0.0"
bonded_agent: 06-frontend-integration
bond_type: PRIMARY_BOND

# Production-Grade Metadata
parameters:
  - name: framework
    type: string
    required: true
    validation: "^(react|nextjs|vue|angular|svelte)$"
    description: Frontend framework
  - name: api_type
    type: string
    required: false
    validation: "^(REST|GraphQL|both)$"
    description: API type

validation_rules:
  - API calls must have error handling
  - Loading states must be implemented
  - Type safety must be enforced

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [api_latency, cache_hit_rate, error_rate]
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

## Unit Test Template

```typescript
describe('frontend-patterns skill', () => {
  test('useUsers fetches and caches data', async () => {
    const { result } = renderHook(() => useUsers(1));
    await waitFor(() => expect(result.current.data).toBeDefined());
  });

  test('handles API errors gracefully', async () => {
    server.use(rest.get('/api/users', (req, res, ctx) => res(ctx.status(500))));
    const { result } = renderHook(() => useUsers(1));
    await waitFor(() => expect(result.current.error).toBeDefined());
  });
});
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Network error | API unreachable | Check connectivity |
| Stale data | Cache not invalidated | Force refetch |
| Hydration mismatch | SSR/client mismatch | Check rendering |

See Agent 6: Frontend Integration for detailed guidance.
