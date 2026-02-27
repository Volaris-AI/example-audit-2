---
genre: security
category: audit-checklist
analysis-type: static
relevance:
  file-patterns: []
  keywords: []
  config-keys: []
  always-include: true
severity-scale: "Critical|High|Medium|Low|Info"
---

# Security Audit Checklist

**Project Name:** _[Enter project name]_
**Audit Date:** _[YYYY-MM-DD]_
**Auditor:** _[Your name]_
**Scope:** _[Describe what's being audited]_

---

<!-- analysis: static -->

## 1. Preparation

- [ ] Read project README.md
- [ ] Review architecture documentation
- [ ] Identify key components and data flows
- [ ] Review dependencies and third-party integrations
- [ ] Set up test environment

## 2. Authentication & Authorization

### Authentication
- [ ] Password policies implemented (length, complexity)
- [ ] Secure password storage (hashing with salt)
- [ ] Rate limiting on login attempts
- [ ] Multi-factor authentication available (if required)
- [ ] Session management secure (secure cookies, timeouts)
- [ ] Password reset flows secure (token expiration, rate limiting)
- [ ] No hardcoded credentials in code

### Authorization
- [ ] Access control checks on all protected resources
- [ ] Principle of least privilege enforced
- [ ] Role-based access control (RBAC) properly implemented
- [ ] No privilege escalation vulnerabilities
- [ ] Authorization checks on API endpoints
- [ ] Direct object references protected (IDOR prevention)

## 3. Input Validation & Sanitization

- [ ] All user inputs validated on server-side
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (output encoding)
- [ ] Command injection prevention
- [ ] Path traversal prevention
- [ ] File upload validation (type, size, content)
- [ ] JSON/XML parsing secure (prevent XXE)
- [ ] Regular expressions safe (no ReDoS vulnerabilities)

## 4. Data Protection

### Data at Rest
- [ ] Sensitive data encrypted in database
- [ ] Encryption keys properly managed
- [ ] Backups encrypted
- [ ] PII handled according to regulations (GDPR, CCPA)

### Data in Transit
- [ ] HTTPS enforced for all connections
- [ ] TLS 1.2 or higher used
- [ ] Strong cipher suites configured
- [ ] Certificate validation proper
- [ ] No mixed content warnings

### Sensitive Data Handling
- [ ] No secrets in version control
- [ ] No secrets in logs
- [ ] Environment variables used for configuration
- [ ] API keys rotated regularly
- [ ] Sensitive data minimized and purpose-limited

## 5. Dependencies & Third-Party Code

- [ ] All dependencies up to date
- [ ] Known vulnerabilities identified (`npm audit`, `snyk test`)
- [ ] Dependency sources verified (no typosquatting)
- [ ] Unused dependencies removed
- [ ] Third-party scripts from trusted CDNs only
- [ ] Subresource Integrity (SRI) used for external scripts
- [ ] License compliance checked

## 6. API Security

- [ ] Authentication required on protected endpoints
- [ ] Rate limiting implemented
- [ ] CORS configured properly
- [ ] API versioning in place
- [ ] Input validation on all parameters
- [ ] Error messages don't leak sensitive information
- [ ] API documentation reviewed for security issues

## 7. Session Management

- [ ] Secure session tokens (random, unpredictable)
- [ ] HttpOnly flag set on session cookies
- [ ] Secure flag set on session cookies
- [ ] SameSite attribute configured
- [ ] Session timeout implemented
- [ ] Session invalidation on logout
- [ ] Concurrent session handling proper

## 8. Error Handling & Logging

- [ ] Generic error messages to users
- [ ] Detailed errors logged securely
- [ ] Stack traces not exposed to users
- [ ] Sensitive data not logged
- [ ] Logging sufficient for security monitoring
- [ ] Log injection prevented
- [ ] Centralized logging implemented

## 9. Security Headers

- [ ] Content-Security-Policy (CSP) configured
- [ ] X-Frame-Options set (clickjacking prevention)
- [ ] X-Content-Type-Options set (MIME sniffing prevention)
- [ ] Strict-Transport-Security (HSTS) configured
- [ ] X-XSS-Protection set (legacy browsers)
- [ ] Referrer-Policy configured
- [ ] Permissions-Policy configured

## 10. Configuration & Deployment

- [ ] Debug mode disabled in production
- [ ] Default passwords changed
- [ ] Unnecessary services disabled
- [ ] Directory listing disabled
- [ ] Source maps not exposed in production
- [ ] Admin interfaces protected
- [ ] Environment-specific configurations separate

## 11. Business Logic

- [ ] Workflows can't be bypassed
- [ ] Race conditions handled
- [ ] Transaction integrity maintained
- [ ] Proper state management
- [ ] Business rules enforced server-side
- [ ] No logic flaws in critical operations

## 12. Code Quality & Practices

- [ ] Security linters integrated (ESLint security rules, Bandit, etc.)
- [ ] Static analysis tools configured
- [ ] Code review process includes security checks
- [ ] Security training for developers
- [ ] Secure coding guidelines documented
- [ ] Security issues tracked and prioritized

## 13. Infrastructure & DevOps

- [ ] Secrets management solution used (not in code)
- [ ] Container images scanned for vulnerabilities
- [ ] Infrastructure as Code reviewed for security
- [ ] CI/CD pipeline secured
- [ ] Deployment process documented
- [ ] Rollback procedures tested

---

## Summary of Findings

### Critical Issues
_List any critical vulnerabilities found_

### High Severity Issues
_List high severity vulnerabilities_

### Medium Severity Issues
_List medium severity vulnerabilities_

### Low Severity Issues
_List low severity vulnerabilities_

### Recommendations
_Overall security recommendations and next steps_

---

**Completed by:** _[Your name]_  
**Date:** _[YYYY-MM-DD]_
