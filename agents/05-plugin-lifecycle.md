---
description: Plugin lifecycle management, loading, initialization, configuration, hooks, and unloading strategies
capabilities: ["Plugin loading", "Initialization", "Configuration", "Lifecycle hooks", "Error handling", "Unloading"]
---

# Plugin Lifecycle Management

## Lifecycle Stages

```
Discovered → Downloaded → Validated → Loaded → Initialized → Active → Disabled → Unloaded
```

## Stage 1: Discovery
```
Find plugin in registry or file system
├─ Check if plugin exists
├─ Load metadata
└─ Validate manifest
```

```javascript
async function discover() {
  const pluginDirs = fs.readdirSync('plugins');

  return pluginDirs
    .filter(dir => fs.existsSync(`plugins/${dir}/plugin.json`))
    .map(dir => ({
      id: require(`plugins/${dir}/plugin.json`).id,
      path: `plugins/${dir}`
    }));
}
```

## Stage 2: Download
```
Get plugin code from source
├─ From registry/marketplace
├─ From file system
└─ From git repository
```

```javascript
async function download(plugin) {
  if (isLocal(plugin)) {
    return plugin.path;
  }

  const downloaded = await marketplace.download(plugin.id);
  const extracted = await extract(downloaded);
  return extracted;
}
```

## Stage 3: Validation
```
Verify plugin is safe to load
├─ Check signature/certificate
├─ Validate manifest
├─ Check dependencies
└─ Verify permissions
```

```javascript
async function validate(plugin) {
  // Check signature
  const signature = fs.readFileSync(`${plugin.path}/plugin.sig`);
  if (!verifySignature(plugin.path, signature)) {
    throw new Error('Invalid signature');
  }

  // Check manifest
  const manifest = require(`${plugin.path}/plugin.json`);
  if (!manifest.id || !manifest.version) {
    throw new Error('Invalid manifest');
  }

  // Check dependencies
  const deps = Object.keys(manifest.dependencies || {});
  for (const dep of deps) {
    if (!registry.has(dep)) {
      throw new Error(`Dependency not found: ${dep}`);
    }
  }

  return true;
}
```

## Stage 4: Loading
```
Load plugin code into memory
├─ Require/import module
├─ Instantiate plugin
└─ Load dependencies
```

```javascript
async function load(plugin) {
  // Load dependencies first
  const deps = plugin.manifest.dependencies || {};
  for (const [id, version] of Object.entries(deps)) {
    await load(registry.find(id, version));
  }

  // Load plugin
  const PluginClass = require(plugin.path);
  plugin.instance = new PluginClass();

  return plugin;
}
```

## Stage 5: Initialization
```
Initialize plugin and prepare for use
├─ Call activate() hook
├─ Load configuration
├─ Set up event listeners
└─ Initialize UI
```

```javascript
async function initialize(plugin) {
  // Create context
  const context = {
    id: plugin.id,
    storage: new PluginStorage(plugin.id),
    eventBus: eventBus,
    logger: logger.forPlugin(plugin.id)
  };

  // Call activate hook
  await plugin.instance.activate(context);

  // Load configuration
  const config = loadConfig(plugin.id);
  await plugin.instance.configure(config);

  plugin.state = 'active';
  plugin.context = context;

  return plugin;
}
```

## Stage 6: Active
```
Plugin is running and usable
├─ Respond to events
├─ Handle commands
└─ Provide services
```

```javascript
async function executeCommand(pluginId, command, args) {
  const plugin = registry.get(pluginId);

  if (plugin.state !== 'active') {
    throw new Error('Plugin not active');
  }

  return plugin.instance[command](...args);
}
```

## Stage 7: Disable
```
Temporarily stop plugin
├─ Suspend event handling
├─ Stop accepting commands
└─ Preserve state
```

```javascript
async function disable(plugin) {
  plugin.state = 'disabled';

  // Stop processing
  eventBus.unsubscribe(plugin.id);

  // Save state
  plugin.savedState = plugin.instance.getState?.();
}
```

## Stage 8: Unload
```
Remove plugin from memory
├─ Call deactivate() hook
├─ Clean up resources
├─ Clear from registry
└─ Remove from file system
```

```javascript
async function unload(plugin) {
  // Call deactivate
  if (plugin.instance.deactivate) {
    await plugin.instance.deactivate();
  }

  // Clean up
  plugin.context.storage.clear();
  plugin.context.eventBus.unsubscribe(plugin.id);

  // Remove
  registry.delete(plugin.id);
  plugin = null;
}
```

## Lifecycle Hooks

### Plugin Hooks
```typescript
interface PluginLifecycleHooks {
  // Required
  async activate(context: PluginContext): Promise<void>;
  async deactivate(): Promise<void>;

  // Optional
  async configure?(config: PluginConfig): Promise<void>;
  async getState?(): Promise<PluginState>;
  async restoreState?(state: PluginState): Promise<void>;

  // Inspection
  getCapabilities?(): PluginCapability[];
  getMetadata?(): PluginMetadata;
}
```

### Host Hooks
```typescript
interface HostLifecycleHooks {
  // Notify plugin of events
  onPluginLoaded?(plugin: Plugin): void;
  onPluginUnloaded?(plugin: Plugin): void;
  onPluginError?(plugin: Plugin, error: Error): void;

  // Request changes
  requestPluginDisable?(pluginId: string): Promise<boolean>;
  requestPluginUnload?(pluginId: string): Promise<boolean>;
}
```

## Error Handling

### Loading Errors
```javascript
async function safeLoad(plugin) {
  try {
    return await load(plugin);
  } catch (error) {
    plugin.state = 'error';
    plugin.error = error;
    logger.error(`Failed to load ${plugin.id}:`, error);

    // Allow host to handle
    hostHooks.onPluginError?.(plugin, error);

    throw error;
  }
}
```

### Crash Recovery
```javascript
class PluginMonitor {
  async monitor(plugin) {
    try {
      await plugin.instance.execute();
    } catch (error) {
      logger.error(`Plugin ${plugin.id} crashed:`, error);

      // Attempt recovery
      if (this.canRecover(error)) {
        await plugin.instance.recover();
      } else {
        // Mark as failed
        plugin.state = 'failed';
      }
    }
  }
}
```

## Configuration

### Plugin Configuration Format
```javascript
// plugin-config.json
{
  "enabled": true,
  "settings": {
    "option1": "value1",
    "option2": 123
  },
  "permissions": {
    "overrides": {
      "storage": { "scope": "plugin-only" }
    }
  }
}
```

### Configuration Loading
```javascript
class PluginConfig {
  load(pluginId) {
    const configPath = `config/${pluginId}.json`;

    if (!fs.existsSync(configPath)) {
      return this.getDefaults(pluginId);
    }

    return JSON.parse(fs.readFileSync(configPath, 'utf8'));
  }

  save(pluginId, config) {
    fs.writeFileSync(
      `config/${pluginId}.json`,
      JSON.stringify(config, null, 2)
    );
  }
}
```

## Lifecycle Checklist

- [ ] All stages defined
- [ ] Validation process comprehensive
- [ ] Hooks clearly documented
- [ ] Error handling robust
- [ ] Resource cleanup guaranteed
- [ ] State persistence (if needed)
- [ ] Configuration system designed
- [ ] Monitoring in place
- [ ] Logging at each stage
- [ ] Recovery strategy defined

Next: Enable plugin communication with the Plugin Communication & Integration agent.
