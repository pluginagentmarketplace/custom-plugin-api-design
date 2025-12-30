---
name: secure
version: "2.0.0"
description: Security & Compliance Guide
sasmp_version: "1.3.0"
allowed-tools: Read

# Command Configuration
input_schema:
  type: object
  properties:
    current_auth:
      type: string
      description: "Current authentication mechanism"
    authorization:
      type: string
      description: "Current authorization approach"
    data_sensitivity:
      type: string
      enum: [public, internal, confidential, restricted]
    compliance:
      type: array
      items:
        type: string
        enum: [gdpr, hipaa, pci_dss, soc2, iso27001, ccpa]
    current_measures:
      type: array
      items:
        type: string

output_schema:
  type: object
  properties:
    vulnerability_assessment:
      type: array
    hardening_guide:
      type: object
    compliance_checklist:
      type: object
    implementation_examples:
      type: object
    testing_strategy:
      type: object

orchestration:
  agents:
    - 05-security-compliance  # Primary
    - 01-api-architect        # Secure design patterns
  execution: sequential
---

# /secure - Security & Compliance Guide

Comprehensive security audit and hardening recommendations for your API.

## What You'll Get

```
┌─────────────────────────────────────────────────────────────┐
│                 Security Assessment                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Risk Level: MEDIUM                                          │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ OWASP Top 10 Coverage                               │    │
│  ├─────────────────────────────────────────────────────┤    │
│  │ A01 Broken Access Control      ⚠️ Partial          │    │
│  │ A02 Cryptographic Failures     ✅ Addressed        │    │
│  │ A03 Injection                  ⚠️ Needs Review     │    │
│  │ A04 Insecure Design            ✅ Addressed        │    │
│  │ A05 Security Misconfiguration  ❌ Missing          │    │
│  │ A06 Vulnerable Components      ⚠️ Partial          │    │
│  │ A07 Auth Failures              ✅ Addressed        │    │
│  │ A08 Data Integrity Failures    ⚠️ Partial          │    │
│  │ A09 Logging Failures           ❌ Missing          │    │
│  │ A10 SSRF                       ✅ Addressed        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Compliance Status:                                          │
│  • GDPR: 70% Ready                                          │
│  • SOC2: 55% Ready                                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## How to Use

Share your current security posture:

```yaml
Authentication:
  type: "JWT with refresh tokens"
  provider: "Auth0 / Custom / Firebase"

Authorization:
  model: "RBAC with 3 roles (admin, user, guest)"
  enforcement: "Middleware on each route"

Data Sensitivity:
  level: "confidential"
  pii_types: [email, phone, address]
  payment_data: true

Compliance Requirements:
  - GDPR (EU users)
  - PCI-DSS (payment processing)

Current Measures:
  - HTTPS enforced
  - Rate limiting (100 req/min)
  - Input validation (Zod)
  - SQL parameterized queries
```

## Security Assessment Areas

### 1. Authentication Security

```typescript
// Assessment checklist
const authChecklist = {
  token_security: {
    jwt_algorithm: 'RS256 recommended over HS256',
    token_expiry: '15 minutes for access, 7 days for refresh',
    token_storage: 'httpOnly cookies, not localStorage',
    rotation: 'Refresh token rotation enabled',
  },

  password_policy: {
    min_length: 12,
    complexity: 'upper, lower, number, special',
    hashing: 'Argon2id or bcrypt (cost 12+)',
    breach_check: 'HaveIBeenPwned integration',
  },

  mfa: {
    available: true,
    methods: ['TOTP', 'WebAuthn', 'SMS (not recommended)'],
    enforcement: 'Required for admin roles',
  },
};
```

### 2. Authorization Security

```typescript
// RBAC/ABAC implementation review
const authzChecklist = {
  model: 'RBAC with resource-level permissions',

  enforcement_points: [
    'API Gateway (coarse-grained)',
    'Service layer (fine-grained)',
    'Database (row-level security)',
  ],

  common_issues: [
    'IDOR vulnerabilities',
    'Privilege escalation paths',
    'Missing resource ownership checks',
  ],
};
```

### 3. Data Protection

```typescript
// Encryption and data handling
const dataProtection = {
  in_transit: {
    tls_version: '1.3 minimum',
    cipher_suites: 'AEAD ciphers only',
    hsts: 'Enabled with preload',
  },

  at_rest: {
    database: 'AES-256 encryption',
    files: 'Server-side encryption',
    backups: 'Encrypted with separate keys',
  },

  pii_handling: {
    minimization: 'Collect only necessary data',
    anonymization: 'For analytics and logs',
    deletion: 'Automated after retention period',
  },
};
```

## Compliance Checklists

### GDPR Compliance

```yaml
GDPR Requirements:
  consent:
    - [ ] Explicit consent collection
    - [ ] Consent withdrawal mechanism
    - [ ] Consent audit trail

  data_subject_rights:
    - [ ] Access request (Article 15)
    - [ ] Rectification (Article 16)
    - [ ] Erasure/Right to be forgotten (Article 17)
    - [ ] Data portability (Article 20)

  technical_measures:
    - [ ] Encryption at rest and in transit
    - [ ] Access logging
    - [ ] Data breach notification (72 hours)

  documentation:
    - [ ] Privacy policy
    - [ ] Data processing agreements
    - [ ] Records of processing activities
```

### PCI-DSS Compliance

```yaml
PCI-DSS Requirements:
  network_security:
    - [ ] Firewall configuration
    - [ ] No vendor defaults
    - [ ] Network segmentation

  cardholder_data:
    - [ ] No full PAN storage (use tokens)
    - [ ] Encryption for transmission
    - [ ] Secure key management

  access_control:
    - [ ] Unique user IDs
    - [ ] Physical access restrictions
    - [ ] Access logging

  monitoring:
    - [ ] Audit trails
    - [ ] Intrusion detection
    - [ ] Regular security testing
```

## Implementation Examples

### Secure JWT Implementation

```typescript
import jwt from 'jsonwebtoken';
import { randomBytes } from 'crypto';

class SecureJWTService {
  // Use RS256 with public/private key pair
  private readonly privateKey = process.env.JWT_PRIVATE_KEY!;
  private readonly publicKey = process.env.JWT_PUBLIC_KEY!;

  generateTokenPair(user: User): TokenPair {
    const jti = randomBytes(16).toString('hex');

    const accessToken = jwt.sign(
      {
        sub: user.id,
        roles: user.roles,
        jti,
        type: 'access',
      },
      this.privateKey,
      {
        algorithm: 'RS256',
        expiresIn: '15m',
        issuer: 'api.example.com',
        audience: 'api.example.com',
      }
    );

    const refreshToken = jwt.sign(
      { sub: user.id, jti, type: 'refresh' },
      this.privateKey,
      { algorithm: 'RS256', expiresIn: '7d' }
    );

    // Store jti for revocation
    await redis.setex(`token:${jti}`, 7 * 24 * 60 * 60, 'valid');

    return { accessToken, refreshToken };
  }

  async verify(token: string): Promise<TokenPayload> {
    const payload = jwt.verify(token, this.publicKey, {
      algorithms: ['RS256'],
      issuer: 'api.example.com',
    }) as TokenPayload;

    // Check if revoked
    const status = await redis.get(`token:${payload.jti}`);
    if (status !== 'valid') {
      throw new TokenRevokedError();
    }

    return payload;
  }
}
```

### Input Validation

```typescript
import { z } from 'zod';
import xss from 'xss';

// Sanitizing schema
const UserInputSchema = z.object({
  email: z.string()
    .email()
    .max(255)
    .toLowerCase()
    .trim(),

  name: z.string()
    .min(1)
    .max(100)
    .transform(val => xss(val, {
      whiteList: {},  // No HTML allowed
      stripIgnoreTag: true
    })),

  bio: z.string()
    .max(1000)
    .optional()
    .transform(val => val ? xss(val) : val),
});

// Validation middleware
function validate(schema: z.ZodSchema) {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      req.body = await schema.parseAsync(req.body);
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        return res.status(400).json({
          type: 'https://api.example.com/errors/validation',
          title: 'Validation Failed',
          status: 400,
          errors: error.errors.map(e => ({
            field: e.path.join('.'),
            message: e.message,
          })),
        });
      }
      next(error);
    }
  };
}
```

## Testing Strategy

```yaml
Security Testing:
  static_analysis:
    tools: [ESLint security plugin, Semgrep, SonarQube]
    frequency: Every commit

  dependency_scanning:
    tools: [npm audit, Snyk, Dependabot]
    frequency: Daily

  dynamic_testing:
    tools: [OWASP ZAP, Burp Suite]
    frequency: Weekly / Before release

  penetration_testing:
    scope: Full application
    frequency: Annually / After major changes
    provider: External security firm
```

---

> **Important:** Security is an ongoing process. Regular audits, updates, and monitoring are essential to maintain a secure API.
