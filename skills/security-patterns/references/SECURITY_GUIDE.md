# Security & Compliance Complete Guide

Production security patterns for authentication, authorization, and compliance.

## Authentication Strategies

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
      throw new Error('Token expired');
    }
    throw new Error('Invalid token');
  }
}
```

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
```

### API Key Management
```javascript
const crypto = require('crypto');

function generateAPIKey() {
  return crypto.randomBytes(32).toString('hex');
}

function hashAPIKey(key) {
  return crypto.createHash('sha256').update(key).digest('hex');
}

// Middleware
async function validateAPIKey(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  if (!apiKey) {
    return res.status(401).json({ error: 'Missing API key' });
  }

  const hashedKey = hashAPIKey(apiKey);
  const keyRecord = await db.query(
    'SELECT * FROM api_keys WHERE hash = ? AND NOT revoked',
    [hashedKey]
  );

  if (!keyRecord) {
    return res.status(403).json({ error: 'Invalid API key' });
  }

  req.apiKey = keyRecord;
  next();
}
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

// Usage
app.delete('/api/users/:id', requireRole(['admin']), deleteUser);
```

### Attribute-Based Access Control (ABAC)
```javascript
function requirePermission(resource, action) {
  return (req, res, next) => {
    const hasPermission = req.user.permissions.some(p =>
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
```

## Security Headers
```javascript
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"]
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

## Input Validation
```javascript
const { body, validationResult } = require('express-validator');

app.post('/api/users', [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).escape(),
  body('name').trim().escape(),
  body('age').optional().isInt({ min: 0, max: 120 })
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

// General limiter
const generalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,
  message: { error: 'Too many requests' }
});

// Strict limiter for auth
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: { error: 'Too many login attempts' }
});

app.use('/api/', generalLimiter);
app.post('/auth/login', authLimiter, loginHandler);
```

## Encryption

### Data at Rest
```javascript
const crypto = require('crypto');

function encrypt(data) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(
    'aes-256-gcm',
    Buffer.from(process.env.ENCRYPTION_KEY, 'hex'),
    iv
  );

  let encrypted = cipher.update(data, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  const authTag = cipher.getAuthTag();

  return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
}

function decrypt(encrypted) {
  const [ivHex, authTagHex, data] = encrypted.split(':');
  const iv = Buffer.from(ivHex, 'hex');
  const authTag = Buffer.from(authTagHex, 'hex');

  const decipher = crypto.createDecipheriv(
    'aes-256-gcm',
    Buffer.from(process.env.ENCRYPTION_KEY, 'hex'),
    iv
  );
  decipher.setAuthTag(authTag);

  let decrypted = decipher.update(data, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}
```

## GDPR Compliance

### Right to Erasure
```javascript
app.delete('/api/users/:id', async (req, res) => {
  // Delete user data
  await User.deleteOne({ id: req.params.id });

  // Delete related data
  await Posts.deleteMany({ author_id: req.params.id });
  await Comments.deleteMany({ author_id: req.params.id });

  // Anonymize audit logs
  await AuditLog.updateMany(
    { user_id: req.params.id },
    { user_id: null, user_name: '[DELETED]' }
  );

  res.json({ message: 'User data deleted' });
});
```

### Data Export
```javascript
app.get('/api/users/:id/export', async (req, res) => {
  const data = {
    user: await User.findById(req.params.id),
    posts: await Posts.find({ author_id: req.params.id }),
    comments: await Comments.find({ author_id: req.params.id }),
    exportedAt: new Date()
  };

  res.json(data);
});
```

## Security Checklist

### Authentication
- [ ] JWT with expiration
- [ ] Refresh token rotation
- [ ] API key hashing
- [ ] OAuth 2.0 for third-party

### Authorization
- [ ] RBAC or ABAC implemented
- [ ] Least privilege principle
- [ ] Resource-level permissions

### Data Protection
- [ ] HTTPS enforced
- [ ] Sensitive data encrypted
- [ ] Secrets in environment variables
- [ ] No credentials in code

### Input/Output
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention
- [ ] XSS protection
- [ ] Rate limiting enabled

### Monitoring
- [ ] Audit logging enabled
- [ ] Failed login tracking
- [ ] Anomaly detection
- [ ] Security alerts configured

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [GDPR Guidelines](https://gdpr.eu/)
- [JWT Best Practices](https://auth0.com/blog/a-look-at-the-latest-draft-for-jwt-bcp/)
- [Node.js Security Checklist](https://github.com/goldbergyoni/nodebestpractices#6-security-best-practices)
