---
name: plugin-fundamentals
description: Plugin system fundamentals, types, architecture patterns, and core design principles for extensible systems
---

# Plugin Fundamentals Skill

## Quick Start

Understand plugin architecture:

### What is a Plugin System?
- **Third-party code extension** - Without modifying core
- **Loosely coupled** - Plugins don't depend on internals
- **Discoverable** - System finds and loads automatically
- **Isolated** - Plugins don't interfere

### Plugin Types

**Library-based:** Simple, fast, no isolation
```
App → Plugin Library → Executes in-process
```

**Process-based:** Complete isolation
```
App ←→ IPC ←→ Plugin Process
```

**Sandboxed:** Secure, controlled access
```
App ←→ Sandbox VM ←→ Plugin (limited access)
```

**Embedded Language:** Flexible but slower
```
App ←→ Interpreter (Lua/Python) ←→ Plugin Script
```

### Core Concepts

**Metadata:** Plugin information
```json
{
  "id": "plugin-id",
  "name": "Plugin Name",
  "version": "1.0.0",
  "main": "index.js"
}
```

**Interface Contract:** What all plugins must implement
```typescript
interface Plugin {
  activate(context): Promise<void>;
  deactivate(): Promise<void>;
}
```

**Extension Points:** Where plugins can extend
- Hooks (events)
- Commands (functions)
- Providers (data)
- UI Components

**Lifecycle:** Stages from discovery to unload
```
Discovered → Loaded → Initialized → Active → Unloaded
```

### Design Principles

1. **Interface-First** - Define APIs before plugins
2. **Dependency Injection** - Pass needed services
3. **Convention** - Predictable method names
4. **Minimal Coupling** - Loose, stable interfaces

See Agent 1 for detailed fundamentals.
