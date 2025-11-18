---
description: Advanced plugin patterns including middleware, filters, plugin composition, async plugins, and enterprise-scale patterns
capabilities: ["Middleware patterns", "Plugin composition", "Async operations", "Enterprise patterns", "Performance optimization", "Plugin orchestration"]
---

# Advanced Plugin Patterns

## Middleware Pattern

### What is Middleware?
Sequential processors that can modify data/requests

```
Request → Middleware 1 → Middleware 2 → Middleware 3 → Handler
```

### Implementation
```javascript
class MiddlewareRegistry {
  private middlewares = [];

  use(handler) {
    this.middlewares.push(handler);
  }

  async execute(data) {
    const middlewares = [...this.middlewares];

    const next = async (index = 0) => {
      if (index >= middlewares.length) {
        return data;
      }

      const middleware = middlewares[index];
      return middleware(data, () => next(index + 1));
    };

    return next();
  }
}
```

### Example: Authentication Middleware
```javascript
// Plugin A: Authentication
context.middleware.use(async (req, next) => {
  const token = req.headers.authorization;

  if (!token) {
    throw new Error('No authorization');
  }

  req.user = await verifyToken(token);
  return next();
});

// Plugin B: Logging
context.middleware.use(async (req, next) => {
  console.log(`${req.method} ${req.path}`);
  return next();
});

// Request flows: Auth → Logging → Handler
```

## Plugin Composition

### Composite Pattern
```
Plugin A
├─ Sub-plugin 1
├─ Sub-plugin 2
└─ Sub-plugin 3

One plugin can contain/manage others
```

```javascript
class CompositePlugin {
  constructor() {
    this.children = [];
  }

  addPlugin(plugin) {
    this.children.push(plugin);
  }

  async activate(context) {
    // Activate children
    for (const child of this.children) {
      await child.activate(context);
    }
  }

  async deactivate() {
    // Deactivate children
    for (const child of this.children) {
      await child.deactivate();
    }
  }
}
```

### Plugin Chains
```
Plugin A → Plugin B → Plugin C

Each handles request and passes to next
```

```javascript
async function executeChain(plugins, request) {
  for (const plugin of plugins) {
    request = await plugin.process(request);
  }

  return request;
}
```

## Async Plugins

### Background Tasks
```javascript
// Plugin executes async work
context.eventBus.subscribe('document:saved', async (event) => {
  // Run in background (don't wait)
  setTimeout(() => {
    this.indexDocument(event.document);
  }, 0);
});
```

### Job Queue
```javascript
// Plugin A enqueues job
await context.jobs.enqueue('process-image', {
  imageId: 123,
  format: 'webp'
});

// Plugin B processes
context.jobs.process('process-image', async (job) => {
  const image = await db.get(job.imageId);
  const processed = await compress(image, job.format);
  await db.save(processed);
});
```

### Streaming
```javascript
// Plugin A sends stream
async function* getDataStream(pluginId) {
  for (let i = 0; i < 1000; i++) {
    yield await db.getPage(i);
  }
}

// Plugin B consumes stream
for await (const page of getDataStream()) {
  await processPage(page);
}
```

## Plugin Dependencies

### Dependency Graph
```
Plugin A (requires B, C)
├─ Plugin B (requires D)
└─ Plugin C (requires D)
        ↓
Plugin D (no dependencies)
```

### Dependency Injection
```javascript
class PluginLoader {
  async load(pluginId) {
    const metadata = registry.get(pluginId);
    const deps = metadata.dependencies || {};

    // Load dependencies first
    const resolved = {};
    for (const [id, version] of Object.entries(deps)) {
      resolved[id] = await this.load(id);
    }

    // Pass dependencies to plugin
    const plugin = new PluginClass();
    plugin.setDependencies(resolved);

    return plugin;
  }
}
```

## Plugin Discovery & Registry Advanced Patterns

### Plugin Marketplace
```
Centralized registry of plugins
├─ Search
├─ Download
├─ Version management
└─ Review/ratings
```

### Auto-discovery
```javascript
class DiscoveryService {
  async discover() {
    const sources = [
      this.fileSystem,
      this.marketplace,
      this.npmRegistry,
      this.githubReleases
    ];

    const plugins = [];
    for (const source of sources) {
      plugins.push(...await source.discover());
    }

    return deduplicateAndSort(plugins);
  }
}
```

## Enterprise Patterns

### Multi-tenancy
```javascript
// Each tenant has isolated plugins
class TenantPluginManager {
  getPluginsFor(tenantId) {
    return registry.filter(p => p.tenantId === tenantId);
  }

  async activateForTenant(tenantId, pluginId) {
    const context = {
      ...baseContext,
      tenantId: tenantId,
      storage: new TenantStorage(tenantId)
    };

    return plugins.activate(pluginId, context);
  }
}
```

### Versioning
```
Multiple versions of same plugin coexist
├─ Plugin v1.0.0 (active)
├─ Plugin v2.0.0 (beta)
└─ Plugin v1.5.0 (deprecated)
```

### Hot Reloading
```javascript
class HotReloadManager {
  async reload(pluginId) {
    // Save state
    const state = plugins.get(pluginId).getState();

    // Unload
    await plugins.unload(pluginId);

    // Reload
    await plugins.load(pluginId);

    // Restore state
    plugins.get(pluginId).restoreState(state);
  }
}
```

## Performance Optimization

### Lazy Loading
```javascript
class LazyPlugin {
  constructor() {
    this.loaded = false;
    this.module = null;
  }

  async getModule() {
    if (!this.loaded) {
      this.module = await import('./plugin.js');
      this.loaded = true;
    }

    return this.module;
  }
}
```

### Caching
```javascript
class CachingPluginManager {
  private cache = new Map();

  async execute(pluginId, method, args) {
    const cacheKey = `${pluginId}:${method}:${JSON.stringify(args)}`;

    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }

    const result = await plugins.get(pluginId)[method](...args);
    this.cache.set(cacheKey, result);

    return result;
  }
}
```

## Monitoring & Observability

### Plugin Metrics
```javascript
class PluginMetrics {
  track(pluginId, event, duration) {
    metrics.histogram(`plugin.${event}.duration`, duration, {
      tags: { plugin: pluginId }
    });

    metrics.counter(`plugin.${event}.total`, 1, {
      tags: { plugin: pluginId }
    });
  }
}
```

### Error Tracking
```javascript
class PluginErrorHandler {
  async execute(plugin, fn) {
    try {
      return await fn();
    } catch (error) {
      errorTracking.captureException(error, {
        pluginId: plugin.id,
        pluginVersion: plugin.version
      });

      throw error;
    }
  }
}
```

## Advanced Patterns Checklist

- [ ] Middleware system designed
- [ ] Plugin composition supported
- [ ] Dependency injection implemented
- [ ] Async operations handled
- [ ] Multi-tenancy (if needed)
- [ ] Versioning strategy defined
- [ ] Hot reloading (if needed)
- [ ] Lazy loading for performance
- [ ] Caching strategy
- [ ] Metrics collection
- [ ] Error handling comprehensive
- [ ] Monitoring in place

## Summary

**Level 1: Basics**
- Plugin fundamentals ✓
- Simple API ✓
- Basic lifecycle ✓

**Level 2: Production**
- Security model ✓
- Event system ✓
- Command pattern ✓

**Level 3: Enterprise**
- Middleware ✓
- Composition ✓
- Multi-tenancy ✓
- Hot reload ✓

You now have expertise to design enterprise-grade plugin systems!
