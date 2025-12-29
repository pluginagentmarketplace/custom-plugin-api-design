---
name: security-patterns
description: Security architecture, authentication, authorization, and compliance patterns
sasmp_version: "1.3.0"
bonded_agent: 05-security-compliance
bond_type: PRIMARY_BOND
---

# Security Patterns Skill

## Quick Start

Comprehensive security patterns for production APIs.

### Authentication
- OAuth 2.0 / OpenID Connect
- JWT token management
- API key handling
- Session management

### Authorization
- Role-Based Access Control (RBAC)
- Attribute-Based Access Control (ABAC)
- Policy-based permissions

### Data Protection
- Encryption at rest and in transit
- HTTPS/TLS configuration
- Secret management

### Compliance
- GDPR requirements
- HIPAA considerations
- PCI-DSS guidelines

## Key Patterns

### JWT Authentication
```javascript
function generateToken(user) {
  return jwt.sign(
    { id: user.id, roles: user.roles },
    process.env.JWT_SECRET,
    { expiresIn: '24h', issuer: 'api.example.com' }
  );
}
```

### Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // 100 requests
  message: 'Too many requests'
});
```

## Security Checklist

- [ ] HTTPS/TLS enforced
- [ ] Authentication implemented
- [ ] Input validation in place
- [ ] Rate limiting enabled
- [ ] Security headers set
- [ ] Secrets in environment
- [ ] Audit logging enabled

See Agent 5: Security Compliance for detailed guidance.
