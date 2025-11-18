---
name: plugin-lifecycle
description: Plugin lifecycle management, loading, initialization, configuration, hooks, and lifecycle error handling
---

# Plugin Lifecycle Skill

## Quick Start

Manage plugin lifecycle:

### Lifecycle Stages
```
Discovered → Validated → Loaded → Initialized → Active → Disabled → Unloaded
```

### Stage Details

**Discovery:** Find plugins
```javascript
const plugins = fs.readdirSync('plugins');
```

**Validation:** Check safety
```javascript
// Verify signature
// Validate manifest
// Check dependencies
```

**Loading:** Load code
```javascript
const Plugin = require(path);
```

**Initialization:** Prepare for use
```javascript
await plugin.activate(context);
await plugin.configure(config);
```

**Active:** Running and usable
```javascript
await plugin.execute(command);
```

**Unload:** Clean up
```javascript
await plugin.deactivate();
```

### Lifecycle Hooks
```typescript
interface Plugin {
  activate(context): Promise<void>;
  deactivate(): Promise<void>;
  configure?(config): Promise<void>;
  getState?(): Promise<State>;
  restoreState?(state): Promise<void>;
}
```

### Configuration
```json
{
  "enabled": true,
  "settings": {
    "option1": "value1"
  }
}
```

### Error Handling
```javascript
try {
  await load(plugin);
} catch (error) {
  plugin.state = 'error';
  hostHooks.onPluginError?.(plugin, error);
}
```

### Recovery
```javascript
if (plugin.crashed) {
  if (canRecover(error)) {
    await plugin.recover();
  } else {
    plugin.state = 'failed';
  }
}
```

See Agent 5 for lifecycle details.
