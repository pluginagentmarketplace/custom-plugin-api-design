---
description: Plugin API design, interface contracts, versioning strategies, and stable API evolution
capabilities: ["API interface design", "Contract definition", "Versioning strategies", "API stability", "Backward compatibility"]
---

# Plugin API Architecture

## Plugin API Design

**Plugin API** = The interface that plugins implement and the services the host provides

### Two-Way Contract

```
Host provides → Services, Events, Data
               ↓
            Plugin API
               ↓
Plugin implements → Hooks, Commands, Providers
```

## Interface Design

### 1. Core Plugin Interface
```typescript
// Every plugin must implement this
interface Plugin {
  // Identification
  readonly id: string;
  readonly name: string;
  readonly version: string;

  // Lifecycle
  activate(context: PluginContext): Promise<void>;
  deactivate(): Promise<void>;

  // Capabilities
  getCapabilities(): PluginCapability[];
}
```

### 2. Host Services (What Host Provides)
```typescript
interface PluginContext {
  // Data access
  storage: StorageService;
  database: DatabaseService;

  // Communication
  eventBus: EventBus;
  commandRegistry: CommandRegistry;

  // UI (if applicable)
  uiRegistry: UIRegistry;

  // Utilities
  logger: Logger;
  config: ConfigService;
}
```

### 3. Extension Points (What Plugin Can Extend)
```typescript
// Commands: Callable from elsewhere
interface Command {
  id: string;
  title: string;
  execute(...args: any[]): Promise<any>;
}

// Event Handlers: Respond to events
interface EventHandler {
  event: string;
  handle(data: any): Promise<void>;
}

// Data Providers: Provide data
interface DataProvider {
  type: string;
  getData(query: any): Promise<any[]>;
}

// UI Components (if applicable)
interface UIComponent {
  id: string;
  render(): HTMLElement;
}
```

## API Versioning Strategies

### Strategy 1: Semantic Versioning
```
MAJOR.MINOR.PATCH (1.2.3)

1.0.0 - Breaking changes
1.1.0 - New features (backward compatible)
1.1.1 - Bug fixes (backward compatible)
```

### Strategy 2: API Version in Host
```javascript
// Host provides versioning info
context.hostVersion = "2.0.0";

// Plugin checks compatibility
if (semver.gte(context.hostVersion, "2.0.0")) {
  // Use new API
} else {
  // Use old API
}
```

### Strategy 3: Capability Negotiation
```javascript
// Plugin declares what it needs
const capabilities = {
  events: true,
  storage: true,
  networkAccess: false
};

// Host validates
if (!host.supports(capabilities)) {
  throw new Error("Host doesn't support required capabilities");
}
```

## API Stability Guarantees

### Stable vs Unstable APIs
```typescript
// ✅ Stable API (version 1.0+)
interface StableAPI {
  getData(): Promise<Data>;
}

// ⚠️ Experimental API (preview)
interface ExperimentalAPI {
  /** @unstable This API may change */
  getDataV2(): Promise<DataV2>;
}
```

### Deprecation Process
```javascript
// Version 1.0 - New API introduced
interface EventBus {
  subscribe(event, handler): void;
}

// Version 1.1 - Start deprecating old API
interface EventBus {
  subscribe(event, handler): void;  // ✅ Preferred
  on(event, handler): void;         // ⚠️ Deprecated, use subscribe()
}

// Version 2.0 - Remove deprecated API
interface EventBus {
  subscribe(event, handler): void;  // ✅ Only way
}
```

### Migration Path Example
```javascript
// 1.0.0
const event = await getEvent(id);

// 1.1.0 - Add new API
const event = await eventService.getEvent(id);      // ✅ New
const event = await getEvent(id);                   // ⚠️ Deprecated

// 2.0.0 - Remove old
const event = await eventService.getEvent(id);      // ✅ Only option
```

## Backward Compatibility Rules

### ✅ Safe Changes
- Add new optional parameter (with default)
- Add new method
- Add new event
- Make strict validation lenient

### ❌ Breaking Changes
- Remove method
- Change method signature
- Change event data structure
- Change default behavior

### Handling Changes
```typescript
// Version upgrade check
class PluginManager {
  async loadPlugin(plugin: Plugin, hostVersion: string) {
    const pluginRequiredVersion = plugin.minHostVersion;

    if (semver.lt(hostVersion, pluginRequiredVersion)) {
      throw new Error(
        `Plugin requires host ${pluginRequiredVersion}, ` +
        `but got ${hostVersion}`
      );
    }
  }
}
```

## API Documentation

### Plugin Interface Documentation
```typescript
/**
 * Plugin interface all plugins must implement
 * @version 2.0.0
 * @stability stable
 */
interface Plugin {
  /**
   * Unique identifier for the plugin
   * @example "com.example.myplugin"
   */
  readonly id: string;

  /**
   * Called when plugin is activated
   * @param context - Host-provided context
   * @throws ActivationError if activation fails
   */
  activate(context: PluginContext): Promise<void>;
}
```

## API Evolution Example

### Initial API (1.0.0)
```typescript
interface Plugin {
  name: string;
  activate(): void;
  deactivate(): void;
}
```

### Add Features (1.1.0)
```typescript
interface Plugin {
  name: string;
  activate(): void;
  deactivate(): void;

  // NEW: Async activation
  activateAsync?(): Promise<void>;

  // NEW: Get metadata
  getMetadata?(): PluginMetadata;
}
```

### Modernize (2.0.0)
```typescript
interface Plugin {
  readonly id: string;
  readonly name: string;
  readonly version: string;

  activate(context: PluginContext): Promise<void>;
  deactivate(): Promise<void>;

  // Removed old methods - breaking change
}
```

## API Design Checklist

- [ ] Core plugin interface defined
- [ ] Host services documented
- [ ] Extension points clear
- [ ] Versioning strategy chosen
- [ ] Backward compatibility rules established
- [ ] Deprecation process defined
- [ ] API documentation complete
- [ ] Migration guides provided
- [ ] Version checking implemented
- [ ] Breaking changes planned in advance

Next: Learn about plugin discovery with the Plugin Registry & Discovery agent.
