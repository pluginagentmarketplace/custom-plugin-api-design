---
name: security-patterns
description: Security architecture, authentication, authorization, and compliance patterns
sasmp_version: "2.0.0"
bonded_agent: 05-security-compliance
bond_type: PRIMARY_BOND

# Production-Grade Metadata
parameters:
  - name: security_context
    type: string
    required: true
    validation: "^(authentication|authorization|encryption|compliance|audit)$"
    description: Security context
  - name: compliance_framework
    type: string
    required: false
    validation: "^(GDPR|HIPAA|SOC2|PCI-DSS|ISO27001)$"
    description: Compliance framework

validation_rules:
  - All endpoints must have authentication
  - Sensitive data must be encrypted
  - Secrets must be in environment variables

retry_logic:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

logging_hooks:
  on_start: true
  on_success: true
  on_failure: true
  metrics: [auth_success_rate, auth_failure_rate, encryption_operations]
  audit_logging: true
---

# Security Patterns Skill

## Quick Start

Comprehensive security patterns for production APIs.

### Authentication
- OAuth 2.0 / OpenID Connect
- JWT token management
- API key handling

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
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: 'Too many requests'
});
```

## Security Checklist

- [ ] HTTPS/TLS enforced
- [ ] Authentication implemented
- [ ] Input validation in place
- [ ] Rate limiting enabled
- [ ] Security headers set
- [ ] Audit logging enabled

## Unit Test Template

```javascript
describe('security-patterns skill', () => {
  test('rejects expired tokens', () => {
    const expiredToken = createExpiredToken();
    expect(() => verifyToken(expiredToken)).toThrow('Token expired');
  });

  test('validates input against injection', () => {
    const maliciousInput = "'; DROP TABLE users;--";
    expect(sanitizeInput(maliciousInput)).not.toContain('DROP');
  });
});
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Missing/invalid token | Check Authorization header |
| 403 Forbidden | Insufficient permissions | Verify user roles |
| Rate limited | Too many requests | Implement backoff |

See Agent 5: Security Compliance for detailed guidance.
