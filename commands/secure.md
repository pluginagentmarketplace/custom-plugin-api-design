---
name: secure
description: Security & Compliance Guide
allowed-tools: Read, Glob, Grep
sasmp_version: "2.0.0"

# Production-Grade Metadata
input_validation:
  required: [auth_type]
  optional: [compliance, data_sensitivity]
  schema:
    auth_type:
      type: string
      enum: [oauth, jwt, apikey, session]
    compliance:
      type: array
      items: [GDPR, HIPAA, SOC2, PCI-DSS]
    data_sensitivity:
      type: string
      enum: [low, medium, high, critical]

exit_codes:
  0: Success - secure
  1: Warnings found
  2: Critical vulnerabilities
  3: Compliance violation
---

# /secure - Security & Compliance Guide

Security audit and hardening recommendations.

## What to Share

- Your current authentication method
- Authorization approach
- Data sensitivity level
- Compliance requirements (GDPR, HIPAA, etc.)
- Current security measures

## Usage Examples

```
/secure oauth compliance=GDPR,SOC2
/secure jwt data_sensitivity=high
/secure apikey compliance=PCI-DSS
```

## What You'll Get

- **Vulnerability Assessment** - Known security gaps
- **Security Hardening Guide** - Step-by-step improvements
- **Compliance Checklist** - Regulatory requirements
- **Implementation Examples** - Code snippets
- **Testing Strategy** - Security testing approach

## Output Structure

```
1. Security Assessment
   └─ Current state analysis
   └─ Risk level: [LOW/MEDIUM/HIGH/CRITICAL]
   └─ Vulnerabilities found

2. Recommendations
   └─ Authentication hardening
   └─ Authorization improvements
   └─ Data encryption

3. Compliance Status
   └─ GDPR: [PASS/FAIL/N/A]
   └─ HIPAA: [PASS/FAIL/N/A]
   └─ SOC2: [PASS/FAIL/N/A]

4. Action Items
   └─ Critical (fix now)
   └─ High (fix this week)
   └─ Medium (plan for next sprint)
```
