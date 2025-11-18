---
name: plugin-api-design
description: Plugin API interface design, contracts, versioning strategies, and stable API evolution patterns
---

# Plugin API Design Skill

## Quick Start

Design stable plugin APIs:

### Two-Way Contract
```
Host provides:     Services, Events, Data
                        ↓
                   Plugin API
                        ↓
Plugin implements: Hooks, Commands, Providers
```

### Core Plugin Interface
```typescript
interface Plugin {
  readonly id: string;
  readonly version: string;

  activate(context: PluginContext): Promise<void>;
  deactivate(): Promise<void>;
}
```

### Host Services
```typescript
interface PluginContext {
  storage: StorageService;
  eventBus: EventBus;
  commandRegistry: CommandRegistry;
  logger: Logger;
}
```

### Extension Points
```typescript
// Commands (callable)
interface Command {
  execute(...args): Promise<any>;
}

// Event Handlers
interface EventHandler {
  handle(data): Promise<void>;
}

// Providers
interface DataProvider {
  getData(query): Promise<any[]>;
}
```

### Versioning

**Semantic Versioning:** MAJOR.MINOR.PATCH
- 1.0.0 - Breaking changes
- 1.1.0 - Features (compatible)
- 1.1.1 - Bug fixes (compatible)

**Safe Changes:**
✅ Add optional parameter
✅ Add new method
✅ Add new event

**Breaking Changes:**
❌ Remove method
❌ Change signature
❌ Change behavior

### Deprecation
```
1.1.0: Old method works but deprecated
2.0.0: Old method removed
```

See Agent 2 for API design details.
