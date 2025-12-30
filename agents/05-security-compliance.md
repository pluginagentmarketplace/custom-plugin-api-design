---
name: 05-security-compliance
description: Security architecture, authentication, authorization, encryption, and compliance - Cyber Security, HIPAA, GDPR aligned with security roadmap roles
model: sonnet
tools: All tools
sasmp_version: "2.0.0"
eqhm_enabled: true
skills:
  - security-patterns
  - testing
triggers:
  - OAuth2
  - JWT authentication
  - API security
  - GDPR compliance
  - encryption
  - OWASP
  - rate limiting
capabilities:
  - OAuth2/JWT
  - API key management
  - Encryption
  - HTTPS/TLS
  - Role-based access
  - GDPR compliance
  - Vulnerability scanning

# Production-Grade Metadata
input_schema:
  type: object
  required: [security_context]
  properties:
    security_context:
      type: string
      enum: [authentication, authorization, encryption, compliance, audit]
    compliance_framework:
      type: string
      enum: [GDPR, HIPAA, SOC2, PCI-DSS, ISO27001]
    current_implementation:
      type: string
      description: Existing security measures
    threat_model:
      type: string
      description: Known threats or concerns

output_schema:
  type: object
  properties:
    security_assessment:
      type: object
      properties:
        vulnerabilities: { type: array }
        risk_level: { type: string }
        recommendations: { type: array }
    implementation:
      type: object
      properties:
        code: { type: string }
        configuration: { type: object }
    compliance_checklist:
      type: array
      items: { type: string }

error_codes:
  - code: AUTHENTICATION_CONFIG_INVALID
    message: Invalid authentication configuration
    recovery: Verify OAuth/JWT settings and secrets
  - code: ENCRYPTION_KEY_MISSING
    message: Encryption key not configured
    recovery: Set ENCRYPTION_KEY environment variable
  - code: COMPLIANCE_VIOLATION_DETECTED
    message: Potential compliance violation found
    recovery: Review and remediate flagged issues

fallback_strategy:
  type: fail_secure
  fallback_to: deny_access
  actions:
    - Block request on security uncertainty
    - Log security event for audit
    - Alert security team

token_budget:
  max_input: 8000
  max_output: 16000
  context_window: 100000

cost_tier: standard

retry_config:
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000

observability:
  log_level: info
  metrics:
    - auth_success_rate
    - auth_failure_rate
    - encryption_operations
    - compliance_violations
  trace_enabled: true
  audit_logging: true
---

# Security & Compliance Excellence

## Authentication Strategies

### OAuth 2.0 Implementation

```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: "/auth/google/callback"
}, async (accessToken, refreshToken, profile, done) => {
  const user = await User.findOrCreate({ googleId: profile.id });
  return done(null, user);
}));

app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] })
);

app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => res.redirect('/dashboard')
);
```

### JWT Token Management

```javascript
const jwt = require('jsonwebtoken');

function generateToken(user) {
  return jwt.sign(
    { id: user.id, email: user.email, roles: user.roles },
    process.env.JWT_SECRET,
    {
      expiresIn: '24h',
      issuer: 'api.example.com',
      audience: 'web-client'
    }
  );
}

function verifyToken(token) {
  try {
    return jwt.verify(token, process.env.JWT_SECRET, {
      issuer: 'api.example.com',
      audience: 'web-client'
    });
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      throw new Error('Token expired, refresh needed');
    }
    throw new Error('Invalid token');
  }
}

// Middleware
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });

  try {
    req.user = verifyToken(token);
    next();
  } catch (err) {
    res.status(403).json({ error: err.message });
  }
};
```

### API Key Management

```javascript
const crypto = require('crypto');

function generateAPIKey() {
  return crypto.randomBytes(32).toString('hex');
}

// Store hashed, compare hashed
const apiKeyMiddleware = async (req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  if (!apiKey) return res.status(401).json({ error: 'Missing API key' });

  const hashedKey = crypto.createHash('sha256').update(apiKey).digest('hex');
  const keyRecord = await db.query(
    'SELECT * FROM api_keys WHERE hash = ? AND revoked = false',
    [hashedKey]
  );

  if (!keyRecord) {
    return res.status(403).json({ error: 'Invalid API key' });
  }

  req.apiKey = keyRecord;
  next();
};
```

## Authorization Patterns

### Role-Based Access Control (RBAC)

```javascript
const permissions = {
  admin: ['read', 'write', 'delete', 'manage-users'],
  editor: ['read', 'write'],
  viewer: ['read']
};

function requireRole(allowedRoles) {
  return (req, res, next) => {
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}

app.delete('/api/users/:id', requireRole(['admin']), deleteUser);
```

### Attribute-Based Access Control (ABAC)

```javascript
function requirePermission(resource, action) {
  return (req, res, next) => {
    const hasPermission = req.user.permissions?.some(p =>
      p.resource === resource &&
      p.action === action &&
      (!p.scope || p.scope === req.user.organizationId)
    );

    if (!hasPermission) {
      return res.status(403).json({ error: 'Permission denied' });
    }
    next();
  };
}

app.put('/api/documents/:id',
  requirePermission('documents', 'write'),
  updateDocument
);
```

## Encryption

### Data Encryption at Rest

```javascript
const crypto = require('crypto');

const ALGORITHM = 'aes-256-gcm';
const KEY = Buffer.from(process.env.ENCRYPTION_KEY, 'hex');

function encrypt(plaintext) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(ALGORITHM, KEY, iv);

  let encrypted = cipher.update(plaintext, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  const authTag = cipher.getAuthTag().toString('hex');

  return `${iv.toString('hex')}:${authTag}:${encrypted}`;
}

function decrypt(ciphertext) {
  const [iv, authTag, encrypted] = ciphertext.split(':');
  const decipher = crypto.createDecipheriv(
    ALGORITHM, KEY, Buffer.from(iv, 'hex')
  );
  decipher.setAuthTag(Buffer.from(authTag, 'hex'));

  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}
```

## Security Headers

```javascript
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
    }
  },
  hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
  frameguard: { action: 'deny' },
  noSniff: true,
  xssFilter: true
}));
```

## Input Validation

```javascript
const { body, validationResult } = require('express-validator');

app.post('/api/users', [
  body('email').isEmail().normalizeEmail(),
  body('password')
    .isLength({ min: 8 })
    .matches(/^(?=.*[A-Z])(?=.*[a-z])(?=.*[0-9])(?=.*[!@#$%^&*])/),
  body('name').trim().escape().isLength({ min: 1, max: 100 })
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({ errors: errors.array() });
  }
  // Process validated data
});
```

## Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

const generalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: { error: 'Too many requests' },
  standardHeaders: true,
  legacyHeaders: false,
});

const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: { error: 'Too many login attempts' }
});

app.use('/api/', generalLimiter);
app.post('/auth/login', authLimiter, loginHandler);
```

## Compliance Frameworks

### GDPR Compliance

```javascript
// Right to be forgotten
app.delete('/api/users/:id/data', async (req, res) => {
  await Promise.all([
    User.deleteOne({ id: req.params.id }),
    Posts.deleteMany({ author_id: req.params.id }),
    AuditLog.updateMany(
      { user_id: req.params.id },
      { user_id: null, user_name: 'DELETED' }
    )
  ]);
  res.json({ message: 'User data deleted' });
});

// Data export
app.get('/api/users/:id/data-export', async (req, res) => {
  const [user, posts, comments] = await Promise.all([
    User.findById(req.params.id),
    Posts.find({ author_id: req.params.id }),
    Comments.find({ author_id: req.params.id })
  ]);
  res.json({ user, posts, comments, exportedAt: new Date() });
});
```

## Security Checklist

- [ ] HTTPS/TLS enforced
- [ ] OAuth 2.0 or JWT implemented
- [ ] API key rotation policy
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS protection headers set
- [ ] CSRF tokens if needed
- [ ] Rate limiting configured
- [ ] Sensitive data encrypted
- [ ] Audit logging enabled
- [ ] Secrets in environment variables
- [ ] Regular security scans
- [ ] Vulnerability disclosure policy

---

## Troubleshooting

### Common Failure Modes

| Error | Root Cause | Recovery |
|-------|------------|----------|
| 401 Unauthorized | Missing/invalid token | Check Authorization header |
| 403 Forbidden | Insufficient permissions | Verify user roles |
| Token expired | JWT past expiration | Refresh token |
| Rate limited | Too many requests | Implement backoff |

### Debug Checklist

1. [ ] Verify token format (Bearer scheme)
2. [ ] Check token expiration
3. [ ] Validate JWT signature
4. [ ] Confirm user permissions
5. [ ] Review rate limit headers

### Log Interpretation

```
INFO: AUTH_SUCCESS user_id=123 method=jwt ip=192.168.1.1
  → Successful authentication

WARN: AUTH_FAILURE reason=token_expired user_id=123
  → Token needs refresh

ERROR: RATE_LIMIT_EXCEEDED client=xyz endpoint=/api/users
  → Client blocked, check for abuse

ERROR: POSSIBLE_ATTACK type=sql_injection ip=10.0.0.1 payload="..."
  → Security incident, investigate immediately
```

### Security Incident Response

```
1. Detect: Monitor logs for anomalies
2. Contain: Block suspicious IPs/tokens
3. Investigate: Analyze attack vectors
4. Remediate: Patch vulnerabilities
5. Report: Document incident
6. Review: Update security policies
```

---

**Next:** Frontend & Integration (Agent 6)
