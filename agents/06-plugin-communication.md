---
description: Inter-plugin communication, event systems, command patterns, data sharing, and host integration patterns
capabilities: ["Event systems", "Command execution", "Data sharing", "Message passing", "Pub/Sub patterns", "Integration patterns"]
---

# Plugin Communication & Integration

## Communication Models

### 1. Event-Driven (Pub/Sub)
```
Plugin A emits event
    ↓
Event Bus
    ↓
Plugin B, C, D subscribe and handle
```

**Pros:** ✅ Decoupled, scalable
**Cons:** ❌ Hard to debug, no direct return

### 2. Command Pattern
```
Plugin A → Command Bus → Call Plugin B method
         ← Return result
```

**Pros:** ✅ Direct, request-response
**Cons:** ❌ Tight coupling possible

### 3. Request-Reply
```
Plugin A: Send request
    ↓
Plugin B: Process
    ↓
Plugin A: Receive reply
```

## Event System

### Event Definition
```typescript
interface PluginEvent {
  type: string;
  source: string;    // Plugin ID that emitted
  timestamp: number;
  data: any;
}
```

### Publishing Events
```javascript
class EventBus {
  private listeners = new Map();

  subscribe(event, handler) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(handler);
  }

  async emit(event, data) {
    const handlers = this.listeners.get(event) || [];

    return Promise.all(
      handlers.map(h => this.safeCall(h, data))
    );
  }

  private async safeCall(handler, data) {
    try {
      return await handler(data);
    } catch (error) {
      logger.error('Event handler error:', error);
    }
  }
}
```

### Example: Plugin Lifecycle Events
```javascript
// Plugin A emits
await eventBus.emit('plugin:activated', {
  pluginId: 'plugin-a',
  version: '1.0.0'
});

// Plugin B listens
eventBus.subscribe('plugin:activated', (event) => {
  console.log(`${event.pluginId} activated`);
});
```

### Common Events
```javascript
// System events
'plugin:loaded'
'plugin:activated'
'plugin:deactivated'
'plugin:error'

// Domain-specific (your app defines)
'data:changed'
'user:loggedIn'
'document:saved'
```

## Command Pattern

### Command Registration
```javascript
class CommandRegistry {
  private commands = new Map();

  register(pluginId, command, handler) {
    const fullName = `${pluginId}:${command}`;
    this.commands.set(fullName, handler);
  }

  async execute(command, args) {
    const handler = this.commands.get(command);

    if (!handler) {
      throw new Error(`Command not found: ${command}`);
    }

    return handler(args);
  }
}
```

### Plugin Commands
```javascript
// Plugin A registers command
context.commands.register('get-data', async (query) => {
  return database.query(query);
});

// Plugin B calls command
const data = await context.commands.execute(
  'plugin-a:get-data',
  { table: 'users' }
);
```

## Data Sharing

### Shared Storage
```javascript
// Plugin A writes
await context.storage.set('shared-key', { data: 'value' });

// Plugin B reads
const data = await context.storage.get('shared-key');
```

### Scoped Storage
```javascript
// Plugin can only access its own data
class PluginStorage {
  constructor(pluginId) {
    this.scope = `plugin:${pluginId}`;
  }

  set(key, value) {
    return db.set(`${this.scope}:${key}`, value);
  }

  get(key) {
    return db.get(`${this.scope}:${key}`);
  }
}
```

## Message Passing

### Direct Messaging
```javascript
// Plugin A sends message
await context.messaging.sendTo('plugin-b', {
  type: 'request',
  action: 'processData',
  payload: { items: [...] }
});

// Plugin B receives
context.messaging.on('message', (msg) => {
  if (msg.action === 'processData') {
    // Process and reply
    msg.reply({ status: 'done' });
  }
});
```

### RPC-style Communication
```javascript
class PluginRPC {
  async call(pluginId, method, ...args) {
    // Send request
    const requestId = generateId();
    const promise = this.waitForResponse(requestId);

    await this.sendMessage(pluginId, {
      type: 'rpc-request',
      id: requestId,
      method,
      args
    });

    // Wait for response
    return promise;
  }

  onMessage(msg) {
    if (msg.type === 'rpc-response') {
      this.resolveResponse(msg.id, msg.result);
    }
  }
}
```

## Plugin Integration Patterns

### Provider Pattern
```javascript
// Plugin A provides data
context.providers.register('users', {
  async get(query) {
    return database.query('users', query);
  }
});

// Plugin B consumes
const users = await context.providers.call('users', 'get', {});
```

### Middleware Pattern
```javascript
// Plugins can inject middleware
context.middleware.use(async (req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  await next();
  console.log(`${res.status}`);
});
```

### Hook Pattern
```javascript
// Plugin A hooks into process
context.hooks.register('before:save', async (data) => {
  console.log('About to save:', data);
  return data; // Modified data passed along
});

// System calls hooks
async function save(data) {
  const modified = await eventBus.emit('before:save', data);
  await database.save(modified);
  await eventBus.emit('after:save', modified);
}
```

## Host Integration

### Context API
```typescript
interface PluginContext {
  // Communication
  eventBus: EventBus;
  commands: CommandRegistry;
  messaging: Messaging;

  // Data
  storage: PluginStorage;
  database: Database;      // If allowed

  // Extension
  hooks: HookRegistry;
  middleware: MiddlewareRegistry;
  providers: ProviderRegistry;

  // Utilities
  logger: Logger;
  config: ConfigService;
}
```

### Lifecycle Integration
```javascript
// Plugin A listens for other plugins
context.eventBus.subscribe('plugin:activated', (event) => {
  if (event.pluginId === 'required-plugin') {
    this.onRequiredPluginLoaded();
  }
});
```

## Communication Checklist

- [ ] Event system designed
- [ ] Event types documented
- [ ] Command registry implemented
- [ ] Data sharing scoped properly
- [ ] Message passing secure
- [ ] RPC pattern (if needed)
- [ ] Provider pattern (if needed)
- [ ] Middleware pattern (if needed)
- [ ] Hook system robust
- [ ] Context API complete
- [ ] Error handling for failures
- [ ] Logging for debugging

Next: Master advanced patterns with the Advanced Plugin Patterns agent.
