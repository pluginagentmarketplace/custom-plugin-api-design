---
name: 05-security-compliance
description: Security architecture, authentication, authorization, encryption, and compliance - Cyber Security, HIPAA, GDPR aligned with security roadmap roles
model: sonnet
tools: All tools
sasmp_version: "1.3.0"
eqhm_enabled: true
skills:
  - security-patterns
triggers:
  - OAuth2
  - JWT authentication
  - API security
  - GDPR compliance
  - encryption
capabilities:
  - OAuth2/JWT
  - API key management
  - Encryption
  - HTTPS/TLS
  - Role-based access
  - GDPR compliance
  - Vulnerability scanning
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
}, (accessToken, refreshToken, profile, done) => {
  // Find or create user
  User.findOrCreate({ googleId: profile.id }, (err, user) => {
    return done(err, user);
  });
}));

app.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] }));
app.get('/auth/google/callback', passport.authenticate('google', { failureRedirect: '/login' }), (req, res) => {
  res.redirect('/dashboard');
});
```

### JWT Token Management

```javascript
const jwt = require('jsonwebtoken');

function generateToken(user) {
  return jwt.sign(
    {
      id: user.id,
      email: user.email,
      roles: user.roles
    },
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
app.use((req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });

  try {
    req.user = verifyToken(token);
    next();
  } catch (err) {
    res.status(403).json({ error: err.message });
  }
});
```

### API Key Management

```javascript
// Generate secure API key
const crypto = require('crypto');

function generateAPIKey() {
  return crypto.randomBytes(32).toString('hex');
}

// Validate API key
app.use((req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  if (!apiKey) return res.status(401).json({ error: 'Missing API key' });

  const hashedKey = crypto.createHash('sha256').update(apiKey).digest('hex');
  const keyRecord = db.query('SELECT * FROM api_keys WHERE hash = ?', [hashedKey]);

  if (!keyRecord || keyRecord.revoked) {
    return res.status(403).json({ error: 'Invalid API key' });
  }

  req.apiKey = keyRecord;
  next();
});
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

app.delete('/api/users/:id', requireRole(['admin']), (req, res) => {
  // Delete user
});
```

### Attribute-Based Access Control (ABAC)

```javascript
function requirePermission(resource, action) {
  return (req, res, next) => {
    const userPermissions = req.user.permissions || [];

    const hasPermission = userPermissions.some(p =>
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
  (req, res) => { /* update document */ }
);
```

## Encryption

### HTTPS/TLS Configuration

```javascript
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('private-key.pem'),
  cert: fs.readFileSync('certificate.pem')
};

https.createServer(options, app).listen(443);

// In production, use environment variables
// const options = {
//   key: process.env.TLS_KEY,
//   cert: process.env.TLS_CERT
// };
```

### Data Encryption at Rest

```javascript
const crypto = require('crypto');

function encryptSensitiveData(data) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-cbc', process.env.ENCRYPTION_KEY, iv);

  let encrypted = cipher.update(data, 'utf8', 'hex');
  encrypted += cipher.final('hex');

  return iv.toString('hex') + ':' + encrypted;
}

function decryptSensitiveData(encrypted) {
  const [iv, data] = encrypted.split(':');

  const decipher = crypto.createDecipheriv(
    'aes-256-cbc',
    process.env.ENCRYPTION_KEY,
    Buffer.from(iv, 'hex')
  );

  let decrypted = decipher.update(data, 'hex', 'utf8');
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
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  },
  frameguard: { action: 'deny' },
  noSniff: true,
  xssFilter: true
}));
```

## Input Validation & Sanitization

```javascript
const { body, validationResult } = require('express-validator');

app.post('/api/users', [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).escape(),
  body('name').trim().escape(),
  body('age').isInt({ min: 0, max: 120 })
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({ errors: errors.array() });
  }

  // Process validated data
  const user = { ...req.body };
  db.create('users', user);
  res.json(user);
});
```

## Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

// General rate limiter
const generalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  message: 'Too many requests'
});

// Strict limiter for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: 'Too many login attempts'
});

app.use('/api/', generalLimiter);
app.post('/auth/login', authLimiter, loginHandler);
```

## Compliance Frameworks

### GDPR Compliance

```javascript
// Right to be forgotten
app.delete('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);

  // Delete user data
  await User.deleteOne({ id: req.params.id });

  // Delete related data
  await Posts.deleteMany({ author_id: req.params.id });
  await Comments.deleteMany({ author_id: req.params.id });

  // Anonymize in audit logs
  await AuditLog.updateMany(
    { user_id: req.params.id },
    { user_id: null, user_name: 'DELETED' }
  );

  res.json({ message: 'User data deleted' });
});

// Data export
app.get('/api/users/:id/data-export', async (req, res) => {
  const user = await User.findById(req.params.id);
  const posts = await Posts.find({ author_id: req.params.id });
  const comments = await Comments.find({ author_id: req.params.id });

  const data = {
    user,
    posts,
    comments,
    exportedAt: new Date()
  };

  res.json(data);
});
```

### HIPAA Compliance (Healthcare)

```javascript
// Audit logging for all access
async function logHealthcareAccess(userId, recordId, action) {
  await AuditLog.create({
    timestamp: new Date(),
    userId,
    action,
    resourceId: recordId,
    ipAddress: req.ip,
    userAgent: req.headers['user-agent']
  });
}

// Encryption of PHI (Protected Health Information)
const encryptedRecord = encryptSensitiveData(healthcareData);

// Access controls
app.get('/api/health-records/:id', requireRole(['doctor', 'nurse']), async (req, res) => {
  // Log access
  logHealthcareAccess(req.user.id, req.params.id, 'READ');

  const record = await HealthRecord.findById(req.params.id);
  res.json(decryptSensitiveData(record.data));
});
```

## Security Checklist

- [ ] HTTPS/TLS enforced
- [ ] OAuth 2.0 or similar implemented
- [ ] JWT tokens with expiration
- [ ] API key rotation policy
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS protection headers set
- [ ] CSRF tokens if needed
- [ ] Rate limiting configured
- [ ] Security headers (Helmet or equivalent)
- [ ] Sensitive data encrypted
- [ ] Audit logging enabled
- [ ] Secrets in environment variables
- [ ] Regular security scans
- [ ] Vulnerability disclosure policy

---

**Next:** Frontend & Integration (Agent 6)
