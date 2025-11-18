# Review Your Plugin System Design

Validate and improve your plugin system design.

## What to Share

Provide Claude with:

1. **Your Plugin Architecture**
   - Current design/implementation
   - Plugin interface definition
   - How plugins are discovered and loaded

2. **API Specification**
   - Host services provided to plugins
   - Extension points available
   - Current versioning approach

3. **Plugin Examples**
   - How plugins currently look
   - What they can and cannot do
   - Real plugin you've built

4. **Known Issues** (if any)
   - Performance problems
   - Security concerns
   - Usability challenges

## Review Checks

Claude will validate:

### ✅ Architecture Quality
- [ ] Plugin type appropriate
- [ ] Isolation strategy sufficient
- [ ] Lifecycle well-defined
- [ ] Discovery mechanism effective

### ✅ API Design
- [ ] Interface clear and consistent
- [ ] Host services well-scoped
- [ ] Extension points comprehensive
- [ ] Versioning strategy sustainable

### ✅ Security
- [ ] Plugin authentication present
- [ ] Permissions properly enforced
- [ ] Isolation adequate
- [ ] Communication secure

### ✅ Performance
- [ ] Plugin loading efficient
- [ ] Runtime overhead acceptable
- [ ] Resource limits enforced
- [ ] No memory leaks

### ✅ Developer Experience
- [ ] API easy to understand
- [ ] Documentation clear
- [ ] Error messages helpful
- [ ] Examples provided

### ✅ Maintainability
- [ ] Code organization clean
- [ ] Testing strategy present
- [ ] Error handling robust
- [ ] Monitoring adequate

## Feedback You'll Get

- Issues found with severity levels
- Specific suggestions for improvement
- Examples of better implementations
- Links to relevant documentation
- Code examples for fixes

## Common Issues to Fix

### Plugin Discovery
- ❌ Hard-coded paths
- ✅ Dynamic directory scanning
- ✅ Registry-based discovery

### API Stability
- ❌ Changing interfaces often
- ✅ Semantic versioning
- ✅ Deprecation process

### Error Handling
- ❌ Plugin crashes kill app
- ✅ Process isolation
- ✅ Error recovery

### Performance
- ❌ Blocking plugin loads
- ✅ Parallel loading
- ✅ Lazy loading

## How to Share

### Option 1: Code
```javascript
class MyPlugin {
  activate(context) { ... }
  deactivate() { ... }
}
```

### Option 2: Architecture
```
MyApp
├─ Plugin Registry
├─ Plugin Loader
├─ Event Bus
└─ Plugin Context
```

### Option 3: Description
Just describe how your system currently works.

## Next Steps

1. **Share your design** - Code, diagram, or description
2. **Get feedback** - Claude reviews and suggests improvements
3. **Security audit** - Use `/security-audit` for security review
4. **Integration guide** - Use `/integration-guide` for details

Ready to review your plugin system?
