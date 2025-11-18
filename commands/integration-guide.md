# Plugin Integration Guide

Complete guide for integrating plugins into your application.

## Integration Steps

### Step 1: Design Plugin Interface
Define what all plugins must implement

```typescript
interface Plugin {
  id: string;
  version: string;
  activate(context: PluginContext): Promise<void>;
  deactivate(): Promise<void>;
}
```

### Step 2: Create Plugin Context
Define host services available to plugins

```typescript
interface PluginContext {
  storage: StorageService;
  eventBus: EventBus;
  commands: CommandRegistry;
  logger: Logger;
}
```

### Step 3: Build Plugin Discovery
Find and load plugins from file system or registry

```javascript
async discover() {
  const pluginDirs = fs.readdirSync('plugins');
  // Load plugin.json from each
  // Validate and register
}
```

### Step 4: Implement Plugin Loader
Load plugins in dependency order

```javascript
async load(pluginId) {
  // Load dependencies first
  // Validate plugin
  // Create instance
  // Call activate()
}
```

### Step 5: Create Plugin Registry
Manage available and active plugins

```javascript
class PluginRegistry {
  register(metadata, instance)
  get(id)
  getAll()
}
```

### Step 6: Set Up Communication
Enable plugins to communicate

```javascript
// Event system
eventBus.subscribe('event', handler)

// Commands
commands.register('cmd', handler)

// Data sharing
storage.set(key, value)
```

## Common Integration Patterns

### Pattern 1: Event-Driven
```
Plugin A emits event
    ↓
Event Bus
    ↓
Plugin B, C handle
```

**Use when:** Loose coupling needed

### Pattern 2: Command-Based
```
Plugin A calls Plugin B:execute()
    ↓
Plugin B processes
    ↓
Plugin A gets result
```

**Use when:** Direct request-response needed

### Pattern 3: Middleware Chain
```
Plugin A → Plugin B → Plugin C → Result
```

**Use when:** Sequential processing needed

## Plugin Metadata Template

```json
{
  "id": "com.example.myplugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "Does something cool",
  "main": "dist/index.js",
  "minHostVersion": "1.0.0",
  "requiredFeatures": ["events", "storage"],
  "permissions": {
    "storage": { "read": true, "write": true },
    "network": { "fetch": true }
  },
  "dependencies": {}
}
```

## Basic Plugin Template

```javascript
export class MyPlugin {
  constructor() {
    this.id = 'my-plugin';
    this.name = 'My Plugin';
    this.version = '1.0.0';
  }

  async activate(context) {
    this.context = context;

    // Register commands
    context.commands.register('my-command', () => {
      return 'Result';
    });

    // Listen to events
    context.eventBus.subscribe('app:ready', () => {
      this.onAppReady();
    });

    this.context.logger.info('Plugin activated');
  }

  async deactivate() {
    // Clean up resources
    this.context.logger.info('Plugin deactivated');
  }

  onAppReady() {
    // Do something when app is ready
  }
}
```

## Testing Plugins

### Unit Test
```javascript
test('plugin execute command', async () => {
  const plugin = new MyPlugin();
  const context = createMockContext();

  await plugin.activate(context);

  const result = await context.commands.execute('my-plugin:my-command');
  expect(result).toBe('Expected');

  await plugin.deactivate();
});
```

### Integration Test
```javascript
test('plugin integrates with app', async () => {
  const app = createApp();
  const plugin = await app.loadPlugin('./my-plugin');

  // Emit event
  app.eventBus.emit('event', data);

  // Check plugin responded
  expect(plugin.wasNotified).toBe(true);
});
```

## Debugging Plugins

### Logging
```javascript
context.logger.info('Message');
context.logger.error('Error:', error);
```

### State Inspection
```javascript
const state = plugin.getState?.();
console.log('Plugin state:', state);
```

### Communication Tracing
```javascript
eventBus.on('*', (event) => {
  console.log('Event:', event);
});

commands.onCall((cmd, args) => {
  console.log('Command:', cmd, args);
});
```

## Performance Optimization

### Lazy Loading
```javascript
// Load plugins only when needed
if (userNeedsFeature) {
  await loadPlugin('feature-plugin');
}
```

### Parallel Loading
```javascript
// Load multiple plugins in parallel
await Promise.all([
  loader.load('plugin-1'),
  loader.load('plugin-2'),
  loader.load('plugin-3')
]);
```

### Caching
```javascript
// Cache plugin results
const cached = cache.get(key);
if (cached) return cached;

const result = await plugin.execute();
cache.set(key, result);
return result;
```

## Versioning Strategy

### Semantic Versioning
- 1.0.0 - Breaking changes
- 1.1.0 - New features
- 1.1.1 - Bug fixes

### Forward Compatibility
```javascript
// Check host version
if (semver.gte(hostVersion, '2.0.0')) {
  // Use new API
} else {
  // Use old API
}
```

### Deprecation
```
1.0.0: Initial release
1.1.0: Introduce newMethod, oldMethod deprecated
2.0.0: Remove oldMethod
```

## Troubleshooting

### Plugin Won't Load
- ✅ Check plugin.json exists and valid
- ✅ Verify dependencies installed
- ✅ Check file permissions
- ✅ Review error logs

### Plugin Crashes
- ✅ Add try-catch in activate()
- ✅ Check context passed correctly
- ✅ Verify permissions granted
- ✅ Use process isolation

### Performance Issues
- ✅ Profile plugin code
- ✅ Use lazy loading
- ✅ Implement caching
- ✅ Optimize database queries

## Best Practices

1. **Keep interfaces minimal** - Only expose needed APIs
2. **Document thoroughly** - Clear examples and guides
3. **Version carefully** - Think about compatibility
4. **Test well** - Unit, integration, and E2E tests
5. **Monitor closely** - Log and track plugin performance
6. **Plan updates** - Beta testing and gradual rollout
7. **Secure properly** - Validate, sandbox, limit resources

## Resources

- See Agent 1-7 for detailed guidance
- Skills provide quick references
- Use `/design-plugin` to start new system
- Use `/review-plugin` to validate design
- Use `/security-audit` for security review

Ready to integrate plugins?
