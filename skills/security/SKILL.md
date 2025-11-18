---
name: api-security
description: API security implementation including authentication methods, authorization patterns, encryption, and security best practices
---

# API Security Skill

## Quick Start

Secure your API by implementing:

### Authentication Methods

#### API Keys
```
Authorization: Bearer sk_live_abc123xyz789
```
Simple but less secure

#### JWT (JSON Web Tokens)
```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI1MzUyMjUxIn0...
```
Stateless and scalable

#### OAuth 2.0
```
Authorization code flow for third-party integrations
```
Industry standard

### JWT Implementation
```javascript
const token = jwt.sign(
  { id: user.id, roles: ['user'] },
  secret,
  { expiresIn: '24h' }
);

// Verify
const decoded = jwt.verify(token, secret);
```

### Rate Limiting
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1609459200
```

### HTTPS
```
✅ Always use HTTPS
✅ Minimum TLS 1.2
✅ Strong certificates
```

### Input Validation
```javascript
// Validate all inputs
const email = validator.isEmail(input.email);
const password = input.password.length >= 8;
```

### SQL Injection Prevention
```javascript
// ✅ Parameterized query
db.query('SELECT * FROM users WHERE email = ?', [email]);

// ❌ Vulnerable
db.query(`SELECT * FROM users WHERE email = '${email}'`);
```

### Security Headers
```
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'
```

## Security Checklist

- [ ] Authentication implemented
- [ ] Authorization configured
- [ ] HTTPS enforced
- [ ] Secrets in environment variables
- [ ] Input validated
- [ ] Rate limiting applied
- [ ] Security headers set
- [ ] Encryption at rest
- [ ] Audit logging active
- [ ] Regular security testing

See the API Security & Authentication agent for complete security guidelines.
