---
name: plugin-patterns
description: Advanced plugin patterns including middleware, composition, async operations, and enterprise-scale plugin systems
---

# Advanced Plugin Patterns Skill

## Quick Start

Master advanced patterns:

### Middleware Pattern
```
Request → Middleware 1 → Middleware 2 → Handler
```

```javascript
context.middleware.use(async (req, next) => {
  console.log('Before');
  const result = await next();
  console.log('After');
  return result;
});
```

### Plugin Composition
```
Parent Plugin
├─ Sub-plugin 1
├─ Sub-plugin 2
└─ Sub-plugin 3
```

```javascript
class CompositePlugin {
  addPlugin(plugin) { this.children.push(plugin); }

  async activate(context) {
    for (const child of this.children) {
      await child.activate(context);
    }
  }
}
```

### Plugin Chains
```
Plugin A → Plugin B → Plugin C → Result
```

```javascript
async function executeChain(plugins, request) {
  for (const plugin of plugins) {
    request = await plugin.process(request);
  }
  return request;
}
```

### Async Operations

**Job Queue:**
```javascript
await context.jobs.enqueue('process', { data });

context.jobs.process('process', async (job) => {
  // Handle background job
});
```

**Streaming:**
```javascript
async function* getDataStream() {
  for (let i = 0; i < 1000; i++) {
    yield await db.getPage(i);
  }
}

for await (const page of getDataStream()) {
  // Process each page
}
```

### Dependency Injection
```javascript
// Pass dependencies to plugin
plugin.setDependencies({
  database: dbService,
  cache: cacheService
});
```

### Enterprise Patterns

**Multi-tenancy:**
```javascript
getPluginsFor(tenantId);
context.storage = new TenantStorage(tenantId);
```

**Hot Reload:**
```javascript
const state = plugin.getState();
await reload(pluginId);
plugin.restoreState(state);
```

**Lazy Loading:**
```javascript
async getModule() {
  if (!this.loaded) {
    this.module = await import('./plugin.js');
  }
  return this.module;
}
```

**Caching:**
```javascript
const result = cache.get(key) || await plugin.execute();
cache.set(key, result);
```

### Monitoring
```javascript
metrics.histogram('plugin.execute.duration', duration);
metrics.counter('plugin.error.total', 1);
```

See Agent 7 for advanced patterns.
