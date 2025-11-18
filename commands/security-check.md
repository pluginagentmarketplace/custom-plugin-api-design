# Security Audit

Comprehensive security audit for your API to identify vulnerabilities and ensure compliance.

## What's Checked

### 🔐 Authentication & Authorization
- [ ] Authentication method (OAuth2, JWT, API keys)
- [ ] Token expiration and refresh
- [ ] Authorization level (field-level, endpoint, resource)
- [ ] Role-based access control (RBAC)
- [ ] Multi-factor authentication considered

### 🔒 Data Protection
- [ ] HTTPS enforced
- [ ] TLS 1.2 minimum
- [ ] Data encrypted at rest
- [ ] Sensitive fields masked in logs
- [ ] PII handled properly

### 🛡️ Input Validation
- [ ] All inputs validated
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS protection (output encoding)
- [ ] Rate limiting configured
- [ ] Request size limits set

### 🚫 API Security
- [ ] CORS properly configured
- [ ] Security headers set
- [ ] API keys rotated regularly
- [ ] Secrets not in code
- [ ] Environment variables used

### 📋 Compliance
- [ ] GDPR considerations
- [ ] HIPAA if applicable
- [ ] SOC2 compliance path
- [ ] Audit logging enabled
- [ ] Data retention policy

### 🔍 Advanced Security
- [ ] GraphQL query complexity limits
- [ ] Rate limiting per user/IP
- [ ] Circuit breaker for dependencies
- [ ] Security testing performed
- [ ] Penetration testing planned

## Security Checklist

### Required
- ✅ HTTPS only
- ✅ Strong authentication
- ✅ Input validation
- ✅ Rate limiting
- ✅ Error handling (no leaks)

### Recommended
- ✅ Security headers
- ✅ API versioning
- ✅ Audit logging
- ✅ Monitoring & alerting
- ✅ Security testing

### Best Practice
- ✅ OAuth2/OpenID Connect
- ✅ Field-level authorization
- ✅ Encryption at rest
- ✅ API gateway
- ✅ Regular security audits

## Common Vulnerabilities

### OWASP Top 10 API Risks
1. **Broken Authentication** - Use modern auth (OAuth2, JWT)
2. **Broken Authorization** - Check permissions always
3. **Injection** - Parameterized queries, input validation
4. **Insecure Data Transfer** - HTTPS only
5. **Broken Access Control** - Role-based access
6. **Security Misconfiguration** - Follow best practices
7. **Injection** - Sanitize inputs
8. **Cross-Site Scripting** - Set CSP headers
9. **Insecure Deserialization** - Validate input
10. **Insufficient Logging** - Log security events

## How to Audit

Provide:
1. **Authentication method** - How users authenticate
2. **Authorization logic** - How permissions are checked
3. **Data handling** - How sensitive data is stored/transmitted
4. **Error messages** - What errors return to clients
5. **Rate limiting** - How throttling is implemented
6. **Dependencies** - External services called

## Get Security Report

After audit, you'll receive:
- Critical vulnerabilities (must fix)
- High-risk issues (fix soon)
- Medium-risk issues (plan fix)
- Low-risk issues (nice to have)
- Compliance recommendations

## Next Steps

1. Fix critical/high vulnerabilities
2. Plan medium-risk improvements
3. Use `/performance-guide` for optimization
4. Schedule regular security reviews
