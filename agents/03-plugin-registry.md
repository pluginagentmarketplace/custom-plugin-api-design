---
description: Plugin registry, discovery mechanisms, metadata management, and plugin versioning systems
capabilities: ["Plugin discovery", "Registry design", "Metadata management", "Plugin search", "Version management"]
---

# Plugin Registry & Discovery

## What is a Plugin Registry?

**Registry** = Central system for discovering, managing, and loading plugins

```
File System / Marketplace
           ↓
    Plugin Registry
    ├─ Metadata
    ├─ Versions
    ├─ Dependencies
    └─ Location
           ↓
    Plugin Loader
           ↓
    Running Plugin
```

## Plugin Metadata

### Standard Metadata Format
```json
{
  "id": "com.example.myplugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "Does amazing things",
  "author": "John Doe",
  "license": "MIT",

  "main": "dist/index.js",
  "minHostVersion": "1.0.0",
  "requiredFeatures": ["events", "storage"],

  "dependencies": {
    "other-plugin": "^1.0.0"
  },

  "capabilities": {
    "commands": ["cmd1", "cmd2"],
    "events": ["event1", "event2"],
    "providers": ["provider1"]
  },

  "permissions": {
    "storage": "read-write",
    "network": "read-only",
    "filesystem": "none"
  }
}
```

## Discovery Mechanisms

### 1. File System Discovery
```
plugins/
├── my-plugin/
│   ├── plugin.json
│   └── index.js
├── other-plugin/
│   ├── plugin.json
│   └── index.js
```

**How it works:**
```javascript
class PluginRegistry {
  discover() {
    const pluginsDir = './plugins';
    return fs.readdirSync(pluginsDir)
      .filter(f => fs.existsSync(`./plugins/${f}/plugin.json`))
      .map(f => ({
        path: `./plugins/${f}`,
        metadata: require(`./plugins/${f}/plugin.json`)
      }));
  }
}
```

### 2. Marketplace/Registry Discovery
```
Central Registry (NPM-like)
        ↓
Search & Download
        ↓
Install locally
        ↓
Load from local cache
```

```javascript
async function searchRegistry(query) {
  const response = await fetch(
    `https://registry.plugins.com/search?q=${query}`
  );
  return response.json(); // [{id, name, version}, ...]
}
```

### 3. Dynamic Discovery
```
Scan directories → Load metadata → Validate → Register
     ↓
Scan package manager (npm) → Filter by plugin → Load
```

## Registry Design

### Option 1: In-Memory Registry
```javascript
class PluginRegistry {
  private plugins = new Map();

  register(metadata, plugin) {
    this.plugins.set(metadata.id, {
      metadata,
      plugin,
      loaded: false
    });
  }

  get(id) {
    return this.plugins.get(id);
  }

  findByCapability(capability) {
    return Array.from(this.plugins.values())
      .filter(p => p.metadata.capabilities[capability]);
  }
}
```

### Option 2: File-Based Registry
```
registry.json:
{
  "plugins": {
    "com.example.plugin1": {
      "version": "1.0.0",
      "path": "./plugins/plugin1",
      "status": "active"
    }
  }
}
```

### Option 3: Database Registry
```
TABLE plugins:
  id, name, version, path, status, metadata

Query: SELECT * FROM plugins WHERE status = 'active'
```

## Plugin Resolution

### Version Resolution
```javascript
class VersionResolver {
  // Plugin requires: other-plugin ^1.0.0
  // Available: 1.0.0, 1.1.0, 1.2.0, 2.0.0
  // Resolution: 1.2.0 (highest compatible)

  resolve(requirement) {
    const available = this.getAvailableVersions(requirement.id);
    return semver.maxSatisfying(available, requirement.version);
  }
}
```

### Dependency Resolution
```
Plugin A (requires B, C)
├─ Plugin B (requires D)
├─ Plugin C (requires D)
└─ Plugin D (no requirements)

Resolved order: D → B,C → A
```

```javascript
class DependencyResolver {
  resolve(plugin) {
    const deps = plugin.dependencies || {};
    const resolved = [];

    for (const [id, version] of Object.entries(deps)) {
      const dep = this.registry.find(id, version);
      resolved.push(dep);
      resolved.push(...this.resolve(dep)); // Recursive
    }

    return [...new Set(resolved)]; // Deduplicate
  }
}
```

## Plugin Search

### Search Capabilities
```javascript
interface PluginRegistry {
  // By ID
  findById(id: string): Plugin | null;

  // By capability
  findByCapability(capability: string): Plugin[];

  // By keyword
  search(keyword: string): Plugin[];

  // By version requirement
  findVersion(id: string, versionRange: string): Plugin | null;
}
```

### Search Example
```javascript
// Find all plugins that provide "storage"
const storagePlugins = registry.findByCapability('storage');

// Find specific version
const plugin = registry.findVersion('db-plugin', '^2.0.0');

// Search
const results = registry.search('database');
```

## Registry State Management

### Active Plugins
```json
{
  "registry": {
    "plugins": [
      {
        "id": "plugin1",
        "version": "1.0.0",
        "status": "active",
        "loadedAt": "2024-01-01T00:00:00Z"
      }
    ]
  }
}
```

### Persistence
```javascript
class PluginRegistry {
  save() {
    const state = {
      plugins: Array.from(this.plugins.values())
        .map(p => ({
          id: p.metadata.id,
          version: p.metadata.version,
          status: p.loaded ? 'active' : 'inactive'
        }))
    };

    fs.writeFileSync('registry.json', JSON.stringify(state, null, 2));
  }

  load() {
    const state = JSON.parse(fs.readFileSync('registry.json', 'utf8'));
    // Restore plugins
  }
}
```

## Registry & Discovery Checklist

- [ ] Metadata format standardized
- [ ] Discovery mechanism(s) implemented
- [ ] Registry structure designed
- [ ] Version resolution strategy
- [ ] Dependency resolution implemented
- [ ] Search capabilities defined
- [ ] State management planned
- [ ] Persistence strategy chosen
- [ ] Conflict detection for duplicates
- [ ] Plugin status tracking

Next: Secure your plugin system with the Plugin Security Model agent.
