---
name: plugin-security
description: Plugin security model, authentication, authorization, permissions, sandboxing, and isolation strategies
---

# Plugin Security Skill

## Quick Start

Secure your plugin system:

### Authentication
```javascript
// Sign plugins
const signature = sign(plugin, privateKey);

// Verify on load
if (!verify(plugin, signature, publicKey)) {
  throw new Error('Invalid signature');
}
```

### Permissions
```json
{
  "permissions": {
    "storage": { "read": true, "write": true },
    "network": { "fetch": true },
    "filesystem": "none"
  }
}
```

### Permission Checking
```javascript
class PermissionManager {
  check(pluginId, resource, action) {
    const perms = registry.get(pluginId).permissions;
    return perms[resource][action] !== false;
  }
}
```

### Isolation Approaches

**Process-based:** Separate processes
```
Host ←→ Plugin 1 Process
    ←→ Plugin 2 Process
```

**VM/Sandbox:** Controlled environment
```javascript
const sandbox = { console, fetch: restrictedFetch };
vm.runInNewContext(code, sandbox);
```

**Capability-based:** Provide only what's needed
```javascript
class Context {
  get storage() {
    if (!permissions.storage.read) return null;
    return new RestrictedStorage();
  }
}
```

### Resource Limits
```javascript
// CPU timeout
Promise.race([fn(), timeout(5000)]);

// Memory limit
resourceLimits: { maxOldGenerationSizeMb: 512 }

// Network domains
const allowed = ['api.example.com'];
```

### Secure Communication
```javascript
interface SecureMessage {
  nonce: string;        // Unique ID
  timestamp: number;    // Prevent old messages
  signature: string;    // Verify authenticity
  payload: encrypted;   // Encrypted data
}
```

See Agent 4 for security details.
