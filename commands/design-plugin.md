# Design Your Plugin System

Start designing your custom plugin system from scratch.

## What Would You Like to Build?

Tell Claude about your plugin system:
- **Domain:** What's your application? (e.g., editor, IDE, CMS, framework)
- **Scale:** How many plugins? Which users create them? (developers, users)
- **Plugin Type:** Library-based, process-based, sandboxed, embedded language?
- **Key Features:** What can plugins extend? (UI, commands, data, behavior)
- **Constraints:** Performance, security, ease of use?

## Design Process

Claude will guide you through:

### 1. Architecture Decision
- ✅ Choose plugin type (library, process, sandbox, embedded)
- ✅ Define isolation strategy
- ✅ Design lifecycle stages
- ✅ Plan discovery mechanism

### 2. API Design
- ✅ Define core plugin interface
- ✅ List host services to provide
- ✅ Identify extension points
- ✅ Plan versioning strategy

### 3. Security Model
- ✅ Authentication approach
- ✅ Permission system
- ✅ Sandbox requirements
- ✅ Trust model

### 4. Communication
- ✅ Event system
- ✅ Command pattern
- ✅ Data sharing approach
- ✅ Plugin-to-plugin communication

### 5. Implementation Plan
- ✅ Technology stack
- ✅ Development order
- ✅ Testing strategy
- ✅ Performance considerations

## Output

You'll get a complete design including:

1. **Architecture Diagram** - How plugins integrate
2. **API Specification** - Plugin interface and contracts
3. **Lifecycle Flowchart** - From discovery to unload
4. **Security Model** - Authentication, permissions, isolation
5. **Implementation Roadmap** - Step-by-step development
6. **Code Examples** - Starter template for plugins
7. **Best Practices** - Design patterns to follow

## Example Questions to Ask

"I'm building a browser extension system..."
"I want to let users extend my editor..."
"I need plugin support for my game engine..."
"My SaaS needs third-party integrations..."

## Next Steps

1. **Answer design questions** - Provide context about your system
2. **Get recommendations** - Claude designs your system
3. **Validate design** - Use `/review-plugin` to check
4. **Plan security** - Use `/security-audit` for security model
5. **Integration guide** - Use `/integration-guide` for details

Start by describing your plugin system needs!
