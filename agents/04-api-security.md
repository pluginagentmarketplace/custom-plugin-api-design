---
description: API security implementation including OAuth2, JWT, API keys, rate limiting, encryption, and compliance strategies
capabilities: ["OAuth2 flows", "JWT implementation", "API key management", "Rate limiting", "Encryption", "Input validation", "Security testing"]
---

# API Security & Authentication

## Authentication Methods

### 1. API Keys
**Simple but less secure**
```
curl -H "Authorization: Bearer sk_live_abc123xyz789" https://api.example.com/users
```

**Pros:** Simple, stateless
**Cons:** No expiration, hard to revoke, no scope

### 2. Bearer Token (JWT)
```
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." https://api.example.com/users
```

JWT Structure:
```
Header.Payload.Signature
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI1MzUyMjUxIn0.uSvf9Py3Wp8QvjfQnt6at8xZc6ZUrzckxIreP6IWgEE
```

**Pros:** Stateless, scalable, can include claims
**Cons:** Can't be revoked easily

### 3. OAuth 2.0
**Authorization Framework**

#### Authorization Code Flow
```
1. User clicks "Login with Google"
2. Redirect to Google's auth endpoint
3. User approves access
4. Google redirects with authorization code
5. Exchange code for access token
6. Use token to access API
```

#### Implicit Flow (Deprecated)
```
Directly return token in URL fragment
❌ Not secure for mobile/SPAs anymore
```

#### Client Credentials Flow
```
Machine-to-machine authentication
POST /oauth/token
{
  "grant_type": "client_credentials",
  "client_id": "...",
  "client_secret": "..."
}
```

## JWT Implementation

```javascript
// Creating JWT
const jwt = require('jsonwebtoken');

const token = jwt.sign({
  id: user.id,
  email: user.email,
  roles: ['user']
}, process.env.JWT_SECRET, {
  expiresIn: '24h',
  issuer: 'api.example.com'
});

// Verifying JWT
const decoded = jwt.verify(token, process.env.JWT_SECRET);
```

## Rate Limiting

### 1. Token Bucket
```
- 100 requests per hour
- Refills 100/3600 tokens per second
- Burst up to 10 requests
```

### 2. Fixed Window
```
Resets every hour at :00
```

### 3. Sliding Window
```
Last 3600 seconds of requests
More accurate than fixed window
```

### Implementation
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit to 100 requests per windowMs
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', limiter);
```

### Rate Limit Headers
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 43
X-RateLimit-Reset: 1609459200
Retry-After: 3600
```

## Encryption

### In Transit (TLS/SSL)
```
Always use HTTPS
Minimum TLS 1.2
```

### At Rest
```javascript
const crypto = require('crypto');

function encryptData(data, key) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-cbc', key, iv);

  let encrypted = cipher.update(data, 'utf8', 'hex');
  encrypted += cipher.final('hex');

  return iv.toString('hex') + ':' + encrypted;
}

function decryptData(encrypted, key) {
  const [iv, data] = encrypted.split(':');
  const decipher = crypto.createDecipheriv(
    'aes-256-cbc',
    key,
    Buffer.from(iv, 'hex')
  );

  let decrypted = decipher.update(data, 'hex', 'utf8');
  decrypted += decipher.final('utf8');

  return decrypted;
}
```

## Input Validation

```javascript
// ❌ Unsafe
app.post('/users', (req, res) => {
  const user = { email: req.body.email };
  db.insert('users', user);
});

// ✅ Safe
const { body, validationResult } = require('express-validator');

app.post('/users',
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    const user = { email: req.body.email };
    db.insert('users', user);
  }
);
```

## SQL Injection Prevention

```javascript
// ❌ Vulnerable
const query = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ Safe - Parameterized
const query = 'SELECT * FROM users WHERE email = ?';
db.query(query, [email]);
```

## CORS Security

```javascript
const cors = require('cors');

// ❌ Allow all
app.use(cors());

// ✅ Whitelist origins
app.use(cors({
  origin: ['https://example.com', 'https://app.example.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

## Security Headers

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
```

## API Security Checklist

- [ ] All endpoints require authentication
- [ ] HTTPS enforced
- [ ] JWT tokens have expiration
- [ ] Rate limiting implemented
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS protection headers set
- [ ] CORS properly configured
- [ ] Secrets managed in environment variables
- [ ] Security headers configured
- [ ] Sensitive data encrypted at rest
- [ ] API keys rotated regularly
- [ ] Audit logging implemented
- [ ] Regular security testing

## OWASP Top 10 API Security

1. **Broken Authentication** - Use OAuth2/JWT properly
2. **Broken Authorization** - Check permissions at every endpoint
3. **Injection** - Parameterized queries, input validation
4. **Insecure Data Transfer** - Use HTTPS only
5. **Broken Access Control** - Implement role-based access
6. **Security Misconfiguration** - Follow security best practices
7. **Injection** - Sanitize all inputs
8. **Cross-Site Scripting** - Set CSP headers
9. **Insecure Deserialization** - Validate all input
10. **Insufficient Logging** - Log all security events

Next: Optimize your API performance with the API Performance & Optimization agent.
