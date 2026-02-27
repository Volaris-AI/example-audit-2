---
genre: infrastructure
category: secure-coding
analysis-type: static
relevance:
  file-patterns:
    - "**/src/**"
    - "**/lib/**"
    - "**/app/**"
  keywords:
    - "lint"
    - "eslint"
    - "sonarqube"
    - "static-analysis"
    - "code-review"
  config-keys:
    - "eslint"
    - "prettier"
    - "@typescript-eslint"
    - "sonarqube"
  always-include: true
severity-scale: "Critical|High|Medium|Low|Info"
---

# Secure Coding Infrastructure Audit

## System Information
- **System Name**: 
- **Audit Date**: 
- **Auditor**: 

<!-- analysis: static -->

## Maturity Level Assessment

| Level | Description | Practices | Tooling | Training |
|-------|-------------|-----------|---------|----------|
| **5** | Security champions | Secure by design, threat modeling, SAST/DAST integrated | Full suite, automated, gates, SCA | Regular, certifications, security champions |
| **4** | Proactive security | Code review for security, linting, standards | SAST, SCA, pre-commit hooks | Annual, secure coding guide |
| **3** | Reactive security | Some security awareness, ad-hoc reviews | Basic linting, occasional scans | Some awareness |
| **2** | Minimal security | No security focus in development | No tooling | No training |
| **1** | No security practices | No awareness, vulnerable code | None | None |

### Current Maturity Score: [ ] / 5

## Assessment Areas

### 1. Input Validation
- [ ] **All inputs validated** (whitelist approach)
- [ ] **Type checking** and casting
- [ ] **Length limits** enforced
- [ ] **Encoding** proper (UTF-8)
- [ ] **Parameterized queries** (SQL injection prevention)
- [ ] **Command injection** prevention
- [ ] **Path traversal** prevention

### 2. Output Encoding
- [ ] **HTML encoding** for XSS prevention
- [ ] **JavaScript encoding** in dynamic JS
- [ ] **URL encoding** for URLs
- [ ] **SQL encoding** (or use ORM)
- [ ] **Content-Type** headers correct
- [ ] **X-Content-Type-Options: nosniff**

### 3. Error Handling
- [ ] **Generic error messages** to users
- [ ] **Detailed logs** for developers (not exposed)
- [ ] **Stack traces** not in production responses
- [ ] **Exception handling** comprehensive
- [ ] **Fail securely** (fail closed, not open)

### 4. Security Headers
- [ ] **Content-Security-Policy** (CSP)
- [ ] **X-Frame-Options** (clickjacking)
- [ ] **X-XSS-Protection**
- [ ] **Strict-Transport-Security** (HSTS)
- [ ] **Referrer-Policy**
- [ ] **Permissions-Policy**

### 5. Dependency Management
- [ ] **Dependency scanning** (Snyk, Dependabot)
- [ ] **SCA** (Software Composition Analysis)
- [ ] **No known vulnerabilities** in prod
- [ ] **Automated updates** for security patches
- [ ] **SBOM** maintained

### 6. Code Quality
- [ ] **Code reviews** mandatory
- [ ] **Linting** enforced (ESLint, Pylint, etc.)
- [ ] **Code complexity** monitored
- [ ] **Technical debt** tracked and addressed
- [ ] **Test coverage** >70%
- [ ] **SAST** (Static Application Security Testing)
- [ ] **DAST** (Dynamic Application Security Testing)

### 7. Secrets Management
- [ ] **No secrets in code** or config files
- [ ] **Secrets in vault** (not env vars in prod)
- [ ] **Secret scanning** in commits (GitGuardian, TruffleHog)
- [ ] **Secrets rotation** policy
- [ ] **.gitignore** properly configured

### 8. Development Process
- [ ] **Security requirements** in user stories
- [ ] **Threat modeling** for new features
- [ ] **Security checklist** for developers
- [ ] **Pre-commit hooks** for security checks
- [ ] **CI/CD security gates**
- [ ] **Penetration testing** regular
- [ ] **Bug bounty program**

### 9. Training & Awareness
- [ ] **Secure coding training** annually
- [ ] **OWASP Top 10** awareness
- [ ] **Security champions** program
- [ ] **Internal security guidelines** documented
- [ ] **Security newsletters** or updates

## Recommendations
**Level 1→2**: Basic input validation, error handling, initial training
**Level 2→3**: Code reviews, linting, security awareness program
**Level 3→4**: SAST/DAST integration, secure coding standards, annual training
**Level 4→5**: Security champions, threat modeling, comprehensive security culture

## Success Criteria
- All code reviewed for security
- SAST/DAST in CI/CD
- Zero high-severity vulnerabilities in production
- Team trained on secure coding
- Security headers on all responses

---
**Document Version**: 1.0
