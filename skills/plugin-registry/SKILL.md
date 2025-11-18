---
name: plugin-registry
description: Plugin discovery, registry design, metadata management, version resolution, and search patterns
---

# Plugin Registry Skill

## Quick Start

Implement plugin discovery:

### Metadata Format
```json
{
  "id": "com.example.plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "main": "dist/index.js",
  "minHostVersion": "1.0.0",
  "dependencies": {
    "other-plugin": "^1.0.0"
  }
}
```

### Discovery Methods

**File System:**
```
plugins/
├── plugin1/plugin.json
├── plugin2/plugin.json
```

**Marketplace:**
```
Central registry → Search → Download → Install
```

**Dynamic:**
```
Scan → Validate → Register
```

### Registry Structure
```javascript
// In-memory
const registry = new Map();
registry.set('plugin-id', { metadata, instance });

// File-based
registry.json

// Database
SELECT * FROM plugins
```

### Version Resolution
```
Require: plugin ^1.0.0
Available: 1.0.0, 1.1.0, 1.2.0, 2.0.0
Resolved: 1.2.0 (highest compatible)
```

### Dependency Resolution
```
Plugin A (requires B, C)
├─ Plugin B (requires D)
├─ Plugin C (requires D)
└─ Plugin D

Load order: D → B,C → A
```

### Search
```javascript
// By ID
registry.findById('plugin-id');

// By capability
registry.findByCapability('storage');

// Search
registry.search('database');
```

See Agent 3 for registry details.
