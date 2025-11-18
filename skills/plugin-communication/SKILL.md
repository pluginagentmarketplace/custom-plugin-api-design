---
name: plugin-communication
description: Inter-plugin communication, event systems, command execution, data sharing, and host integration patterns
---

# Plugin Communication Skill

## Quick Start

Enable plugin communication:

### Event System

**Emit:**
```javascript
eventBus.emit('data:changed', { id: 123, value: 'new' });
```

**Subscribe:**
```javascript
eventBus.subscribe('data:changed', (event) => {
  console.log('Data changed:', event);
});
```

**Common Events:**
```
plugin:loaded
plugin:activated
plugin:error
data:changed
user:loggedIn
```

### Command Pattern

**Register:**
```javascript
context.commands.register('get-data', async (query) => {
  return db.query(query);
});
```

**Execute:**
```javascript
const data = await context.commands.execute(
  'plugin-a:get-data',
  { table: 'users' }
);
```

### Data Sharing

**Scoped Storage:**
```javascript
// Isolated per plugin
await context.storage.set('key', value);
const value = await context.storage.get('key');
```

### Message Passing

**Send:**
```javascript
await context.messaging.sendTo('plugin-b', {
  type: 'request',
  action: 'process'
});
```

**Receive:**
```javascript
context.messaging.on('message', (msg) => {
  msg.reply({ status: 'done' });
});
```

### Patterns

**Provider:** Data source
```javascript
context.providers.register('users', {
  get: (query) => db.query('users', query)
});
```

**Middleware:** Sequential processing
```javascript
context.middleware.use(async (req, next) => {
  await validate(req);
  return next();
});
```

**Hooks:** Lifecycle events
```javascript
context.hooks.register('before:save', (data) => {
  return validateAndModify(data);
});
```

See Agent 6 for communication details.
