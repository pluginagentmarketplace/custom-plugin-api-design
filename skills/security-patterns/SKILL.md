---
name: security-patterns
version: "2.0.0"
description: Security architecture, authentication, authorization, and compliance patterns
sasmp_version: "1.3.0"
bonded_agent: 05-security-compliance
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "API security patterns and compliance"
  boundaries:
    includes: [authentication, authorization, encryption, rate_limiting, input_validation]
    excludes: [infrastructure_security, network_security, physical_security]

parameter_validation:
  schema:
    type: object
    properties:
      auth_type:
        type: string
        enum: [jwt, oauth2, api_key, session]
      access_control:
        type: string
        enum: [rbac, abac, policy]
      compliance:
        type: array
        items:
          type: string
          enum: [gdpr, hipaa, pci_dss, soc2]

retry_config:
  enabled: false  # Security operations should not auto-retry

logging:
  level: INFO
  fields: [auth_type, user_id, action, resource, result]
  sensitive_fields_redacted: [password, token, api_key]

dependencies:
  skills: []
  agents: [05-security-compliance]
---

# Security Patterns Skill

## Purpose
Implement comprehensive security for APIs following OWASP guidelines.

## Authentication Patterns

### JWT Implementation

```typescript
import jwt from 'jsonwebtoken';

interface TokenPayload {
  sub: string;        // User ID
  roles: string[];
  iat: number;
  exp: number;
  jti: string;        // Token ID for revocation
}

class JWTService {
  private readonly secret = process.env.JWT_SECRET!;
  private readonly accessTTL = '15m';
  private readonly refreshTTL = '7d';

  generateTokenPair(user: User) {
    const jti = crypto.randomUUID();

    const accessToken = jwt.sign(
      { sub: user.id, roles: user.roles, jti },
      this.secret,
      { expiresIn: this.accessTTL, issuer: 'api.example.com' }
    );

    const refreshToken = jwt.sign(
      { sub: user.id, jti, type: 'refresh' },
      this.secret,
      { expiresIn: this.refreshTTL }
    );

    return { accessToken, refreshToken, jti };
  }

  verify(token: string): TokenPayload {
    return jwt.verify(token, this.secret, {
      issuer: 'api.example.com',
    }) as TokenPayload;
  }

  async isRevoked(jti: string): Promise<boolean> {
    return await redis.exists(`revoked:${jti}`);
  }
}
```

### OAuth 2.0 + PKCE Flow

```typescript
// 1. Generate PKCE challenge
function generatePKCE() {
  const verifier = crypto.randomBytes(32).toString('base64url');
  const challenge = crypto
    .createHash('sha256')
    .update(verifier)
    .digest('base64url');

  return { verifier, challenge };
}

// 2. Authorization request
const authUrl = new URL('https://auth.example.com/authorize');
authUrl.searchParams.set('client_id', CLIENT_ID);
authUrl.searchParams.set('redirect_uri', REDIRECT_URI);
authUrl.searchParams.set('response_type', 'code');
authUrl.searchParams.set('scope', 'openid profile email');
authUrl.searchParams.set('code_challenge', challenge);
authUrl.searchParams.set('code_challenge_method', 'S256');
authUrl.searchParams.set('state', state);

// 3. Token exchange
async function exchangeCode(code: string, verifier: string) {
  const response = await fetch('https://auth.example.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      client_id: CLIENT_ID,
      code,
      redirect_uri: REDIRECT_URI,
      code_verifier: verifier,
    }),
  });

  return response.json();
}
```

## Authorization Patterns

### RBAC Middleware

```typescript
type Permission = 'read' | 'write' | 'delete' | 'admin';
type Resource = 'users' | 'orders' | 'products';

const rolePermissions: Record<string, Record<Resource, Permission[]>> = {
  admin: {
    users: ['read', 'write', 'delete', 'admin'],
    orders: ['read', 'write', 'delete', 'admin'],
    products: ['read', 'write', 'delete', 'admin'],
  },
  manager: {
    users: ['read'],
    orders: ['read', 'write'],
    products: ['read', 'write'],
  },
  user: {
    users: ['read'],
    orders: ['read'],
    products: ['read'],
  },
};

function requirePermission(resource: Resource, permission: Permission) {
  return (req: Request, res: Response, next: NextFunction) => {
    const userRoles = req.user?.roles || [];

    const hasPermission = userRoles.some(role => {
      const perms = rolePermissions[role]?.[resource] || [];
      return perms.includes(permission);
    });

    if (!hasPermission) {
      return res.status(403).json({
        type: 'https://api.example.com/errors/forbidden',
        title: 'Forbidden',
        status: 403,
        detail: `Missing permission: ${permission} on ${resource}`,
      });
    }

    next();
  };
}

// Usage
app.delete('/users/:id',
  authenticate,
  requirePermission('users', 'delete'),
  deleteUser
);
```

### ABAC Policy Engine

```typescript
interface Policy {
  effect: 'allow' | 'deny';
  actions: string[];
  resources: string[];
  conditions?: Condition[];
}

interface Condition {
  field: string;
  operator: 'eq' | 'neq' | 'in' | 'contains';
  value: any;
}

class PolicyEngine {
  private policies: Policy[] = [];

  evaluate(context: {
    user: User;
    action: string;
    resource: string;
    environment: Record<string, any>;
  }): boolean {
    for (const policy of this.policies) {
      if (!policy.actions.includes(context.action)) continue;
      if (!this.matchResource(policy.resources, context.resource)) continue;
      if (!this.evaluateConditions(policy.conditions, context)) continue;

      return policy.effect === 'allow';
    }

    return false; // Default deny
  }

  private evaluateConditions(conditions: Condition[] | undefined, context: any): boolean {
    if (!conditions) return true;

    return conditions.every(cond => {
      const value = this.getNestedValue(context, cond.field);
      switch (cond.operator) {
        case 'eq': return value === cond.value;
        case 'neq': return value !== cond.value;
        case 'in': return cond.value.includes(value);
        case 'contains': return value?.includes(cond.value);
        default: return false;
      }
    });
  }
}
```

## Input Validation & Sanitization

```typescript
import { z } from 'zod';
import xss from 'xss';
import SqlString from 'sqlstring';

// Schema validation
const CreateUserSchema = z.object({
  email: z.string().email().max(255),
  name: z.string().min(1).max(100).transform(val => xss(val)),
  password: z.string()
    .min(12)
    .regex(/[A-Z]/, 'Must contain uppercase')
    .regex(/[a-z]/, 'Must contain lowercase')
    .regex(/[0-9]/, 'Must contain number')
    .regex(/[^A-Za-z0-9]/, 'Must contain special character'),
});

// SQL Injection prevention
function safeQuery(template: TemplateStringsArray, ...values: any[]) {
  return template.reduce((acc, str, i) => {
    return acc + str + (values[i] !== undefined ? SqlString.escape(values[i]) : '');
  }, '');
}

// Usage
const sql = safeQuery`SELECT * FROM users WHERE id = ${userId}`;
```

## Rate Limiting

```typescript
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

// Tiered rate limiting
const rateLimits = {
  anonymous: rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    standardHeaders: true,
    legacyHeaders: false,
    store: new RedisStore({ prefix: 'rl:anon:' }),
  }),

  authenticated: rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 1000,
    keyGenerator: (req) => req.user?.id || req.ip,
    store: new RedisStore({ prefix: 'rl:auth:' }),
  }),

  sensitive: rateLimit({
    windowMs: 60 * 60 * 1000,  // 1 hour
    max: 10,                    // 10 attempts
    keyGenerator: (req) => req.ip,
    store: new RedisStore({ prefix: 'rl:sens:' }),
  }),
};

// Apply to routes
app.post('/login', rateLimits.sensitive, loginHandler);
app.use('/api', authenticate, rateLimits.authenticated);
```

## Security Headers

```typescript
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.example.com"],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
}));

// Additional headers
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('Permissions-Policy', 'geolocation=(), microphone=()');
  next();
});
```

---

## Unit Test Template

```typescript
import { describe, it, expect } from 'vitest';
import request from 'supertest';
import app from './app';

describe('Security Patterns', () => {
  describe('Authentication', () => {
    it('should reject invalid JWT', async () => {
      await request(app)
        .get('/api/users')
        .set('Authorization', 'Bearer invalid-token')
        .expect(401);
    });

    it('should accept valid JWT', async () => {
      const token = generateTestToken({ sub: '123', roles: ['user'] });

      await request(app)
        .get('/api/users')
        .set('Authorization', `Bearer ${token}`)
        .expect(200);
    });
  });

  describe('Authorization', () => {
    it('should enforce RBAC permissions', async () => {
      const userToken = generateTestToken({ sub: '123', roles: ['user'] });

      await request(app)
        .delete('/api/users/456')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(403);
    });
  });

  describe('Input Validation', () => {
    it('should reject XSS attempts', async () => {
      const res = await request(app)
        .post('/api/users')
        .send({ name: '<script>alert("xss")</script>', email: 'test@test.com' });

      expect(res.body.data?.name).not.toContain('<script>');
    });

    it('should reject SQL injection', async () => {
      await request(app)
        .get('/api/users?id=1; DROP TABLE users;--')
        .expect(400);
    });
  });

  describe('Rate Limiting', () => {
    it('should block after threshold', async () => {
      for (let i = 0; i < 10; i++) {
        await request(app).post('/login').send({ email: 'test@test.com', password: 'wrong' });
      }

      const res = await request(app)
        .post('/login')
        .send({ email: 'test@test.com', password: 'wrong' });

      expect(res.status).toBe(429);
    });
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| JWT expired prematurely | Clock skew | Add leeway (30s) to verification |
| CORS blocked | Missing headers | Configure allowed origins properly |
| Rate limit too aggressive | Shared IP (NAT) | Use user ID for authenticated requests |
| Token leaked | Stored insecurely | Use httpOnly cookies, short TTL |
| XSS vulnerability | Unsanitized output | Always escape user content |

---

## OWASP Top 10 Checklist

- [ ] A01: Broken Access Control → RBAC/ABAC implemented
- [ ] A02: Cryptographic Failures → TLS, secure secrets
- [ ] A03: Injection → Input validation, parameterized queries
- [ ] A04: Insecure Design → Threat modeling done
- [ ] A05: Security Misconfiguration → Headers, defaults
- [ ] A06: Vulnerable Components → Dependencies updated
- [ ] A07: Auth Failures → Strong password, MFA
- [ ] A08: Data Integrity → Signed tokens, CSRF protection
- [ ] A09: Logging Failures → Audit logging enabled
- [ ] A10: SSRF → URL validation, allowlists

---

## Quality Checklist

- [ ] Authentication implemented (JWT/OAuth2)
- [ ] Authorization enforced (RBAC/ABAC)
- [ ] Input validation on all endpoints
- [ ] Rate limiting configured
- [ ] Security headers set
- [ ] Secrets management secure
- [ ] Audit logging enabled
- [ ] Compliance requirements met
