---
genre: security
category: authentication
analysis-type: static
relevance:
  file-patterns:
    - "**/auth/**"
    - "**/login/**"
    - "**/middleware/auth*"
    - "**/passport*"
  keywords:
    - "jwt"
    - "oauth"
    - "session"
    - "passport"
    - "bcrypt"
    - "argon2"
    - "saml"
    - "mfa"
    - "totp"
  config-keys:
    - "passport"
    - "jsonwebtoken"
    - "@auth0"
    - "bcrypt"
    - "argon2"
    - "express-session"
    - "next-auth"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Authentication Security Assessment

**Assessment Date:** _[YYYY-MM-DD]_
**Auditor:** _[Your name]_
**Application:** _[Application name and version]_
**Status:** _[Draft / In Progress / Complete]_

---

<!-- analysis: static -->

## Executive Summary

**Overall Authentication Security Rating:** [ ] Excellent [ ] Good [ ] Fair [ ] Poor [ ] Critical

**Key Findings:**
- Total Vulnerabilities: _[Number]_
- Critical: _[Number]_ | High: _[Number]_ | Medium: _[Number]_ | Low: _[Number]_

**Most Critical Issue:** _[Brief description]_

---

## Scope

### Components Assessed
- [ ] Password policies and storage
- [ ] Multi-factor authentication (MFA)
- [ ] Session management
- [ ] OAuth 2.0 / OpenID Connect implementation
- [ ] SAML authentication
- [ ] Brute force protection
- [ ] Password reset flows
- [ ] Account lockout mechanisms
- [ ] Single Sign-On (SSO) integration

### Out of Scope
_[List what was not assessed]_

---

## 1. Password Security

### 1.1 Password Policies

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Minimum password length enforced (12+ characters)
- [ ] Password complexity requirements are reasonable
- [ ] Password history prevents reuse (5+ previous passwords)
- [ ] Maximum password age is enforced (90 days recommended)
- [ ] Common/breached passwords are blocked
- [ ] No maximum password length restriction (<64 chars)

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Current Policy:**
```
Minimum Length: [Number]
Complexity: [Requirements]
History: [Number of passwords]
Max Age: [Days]
```

**Recommendations:**
- _[Specific recommendation]_

### 1.2 Password Storage

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Passwords are hashed with modern algorithm (bcrypt, Argon2, PBKDF2)
- [ ] Salt is unique per password
- [ ] Work factor/iterations are appropriate (bcrypt: 12+, PBKDF2: 310,000+)
- [ ] Passwords are never logged or stored in plaintext
- [ ] Password hashes are not exposed in API responses
- [ ] Legacy passwords are migrated to secure hashing

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Hashing Configuration:**
```
Algorithm: [bcrypt/Argon2/PBKDF2]
Cost Factor: [Number]
Salt Method: [Per-password/Global]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 2. Multi-Factor Authentication (MFA)

### 2.1 MFA Implementation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] MFA is available for all users
- [ ] MFA is enforced for privileged accounts
- [ ] Multiple MFA methods supported (TOTP, SMS, hardware tokens)
- [ ] MFA enrollment process is secure
- [ ] MFA cannot be bypassed
- [ ] Backup codes are provided and stored securely

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**MFA Methods Supported:**
- [ ] TOTP (Google Authenticator, Authy)
- [ ] SMS (not recommended as primary)
- [ ] Email codes
- [ ] Hardware tokens (YubiKey, FIDO2)
- [ ] Biometric
- [ ] Push notifications

**Recommendations:**
- _[Specific recommendation]_

### 2.2 MFA Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] TOTP secrets are generated securely
- [ ] Backup codes are one-time use
- [ ] MFA codes have expiration (30-60 seconds)
- [ ] Rate limiting on MFA verification attempts
- [ ] MFA changes require re-authentication
- [ ] SMS-based MFA warns about security limitations

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Test Results:**
```
Test: Bypass MFA via direct endpoint access
Method: [Method]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 3. Session Management

### 3.1 Session Creation & Storage

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Session IDs are cryptographically random
- [ ] Session IDs are long enough (128+ bits)
- [ ] New session ID generated after login
- [ ] Session data stored server-side (not in cookie)
- [ ] Session cookies have Secure, HttpOnly, SameSite flags
- [ ] Session storage is encrypted

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Session Configuration:**
```
Session ID Length: [Bits]
Storage: [Redis/Database/Memory]
Cookie Flags: [Secure, HttpOnly, SameSite]
```

**Recommendations:**
- _[Specific recommendation]_

### 3.2 Session Lifecycle

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Absolute session timeout is enforced (8-24 hours)
- [ ] Idle timeout is enforced (15-30 minutes)
- [ ] Logout invalidates session server-side
- [ ] Concurrent session limits are enforced
- [ ] Session fixation is prevented
- [ ] Active sessions can be viewed and revoked by user

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Timeout Configuration:**
```
Absolute Timeout: [Hours]
Idle Timeout: [Minutes]
Max Concurrent: [Number]
```

**Test Results:**
```
Test: Session fixation attack
Method: [Method]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 4. OAuth 2.0 / OpenID Connect

### 4.1 OAuth Implementation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] PKCE is used for authorization code flow
- [ ] State parameter is used and validated
- [ ] Redirect URIs are strictly validated (no wildcards)
- [ ] Access tokens have short lifetimes (1 hour or less)
- [ ] Refresh tokens are rotated on use
- [ ] Client secrets are stored securely

**Issues Found:**

| Flow | Severity | Issue | Impact |
|------|----------|-------|--------|
| | | | |

**OAuth Configuration:**
```
Flows Supported: [Authorization Code, Implicit, etc.]
PKCE: [Enabled/Disabled]
Token Lifetime: [Minutes]
Refresh Token Rotation: [Yes/No]
```

**Recommendations:**
- _[Specific recommendation]_

### 4.2 Token Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] JWT tokens are properly signed (RS256/ES256)
- [ ] JWT signatures are validated
- [ ] Token expiration (exp) is checked
- [ ] Tokens are not logged
- [ ] Token revocation is supported
- [ ] Sensitive data not stored in JWT payload

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**JWT Configuration:**
```
Signing Algorithm: [RS256/HS256/ES256]
Key Rotation: [Yes/No]
Revocation: [Supported/Not Supported]
```

**Test Results:**
```
Test: None/weak algorithm attack
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 5. Brute Force Protection

### 5.1 Account Lockout

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Account lockout after failed attempts (5-10 attempts)
- [ ] Lockout duration is appropriate (15-30 minutes)
- [ ] Lockout applies to all authentication methods
- [ ] Rate limiting on login endpoint
- [ ] CAPTCHA after multiple failures
- [ ] Legitimate users have unlock mechanism

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Lockout Configuration:**
```
Failed Attempts: [Number]
Lockout Duration: [Minutes]
Rate Limit: [Requests per minute]
CAPTCHA Threshold: [Number]
```

**Test Results:**
```
Test: Brute force login
Attempts: [Number]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 5.2 Credential Stuffing Protection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Monitoring for credential stuffing attacks
- [ ] Device fingerprinting to detect automation
- [ ] Anomaly detection for login patterns
- [ ] Compromised password checking (HaveIBeenPwned)
- [ ] Geographic anomaly detection
- [ ] User notification of suspicious login attempts

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 6. Password Reset & Recovery

### 6.1 Password Reset Flow

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Reset tokens are cryptographically random
- [ ] Reset tokens expire (15-60 minutes)
- [ ] Reset tokens are single-use
- [ ] Reset links sent via secure channel (email)
- [ ] User identity verified before reset
- [ ] No username enumeration via reset flow

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Reset Configuration:**
```
Token Length: [Bits]
Token Expiration: [Minutes]
Delivery Method: [Email/SMS]
Identity Verification: [Method]
```

**Test Results:**
```
Test: Token reuse after reset
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 6.2 Account Recovery

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Security questions are not used (if possible)
- [ ] Recovery options are secure (not SMS-only)
- [ ] Recovery process requires multiple factors
- [ ] Recovery changes are logged and notified
- [ ] Recovery bypass is not possible
- [ ] Backup codes are available

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 7. Single Sign-On (SSO) & SAML

### 7.1 SAML Implementation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] SAML assertions are signed and validated
- [ ] XML signature wrapping attacks are prevented
- [ ] Assertions have short validity period
- [ ] Replay attacks are prevented (NotOnOrAfter, NotBefore)
- [ ] Audience restriction is enforced
- [ ] IdP certificate validation is enforced

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**SAML Configuration:**
```
Assertion Signed: [Yes/No]
Response Signed: [Yes/No]
Validity Period: [Minutes]
Signature Algorithm: [Algorithm]
```

**Recommendations:**
- _[Specific recommendation]_

### 7.2 SSO Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] SSO logout properly terminates all sessions
- [ ] IdP metadata is validated and up-to-date
- [ ] SSO bypasses are not possible
- [ ] Just-in-time (JIT) provisioning is secure
- [ ] Attribute mapping is validated
- [ ] SSO errors don't leak sensitive information

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

<!-- analysis: manual -->

## 8. Testing Methodology

### Tools Used
- [ ] Burp Suite Professional
- [ ] Hydra (brute force testing)
- [ ] JWT.io (token analysis)
- [ ] OWASP ZAP
- [ ] Custom authentication scripts
- [ ] HaveIBeenPwned API

### Test Scenarios Executed
1. **Password Policy Bypass:** _[Results]_
2. **Brute Force Attack:** _[Results]_
3. **Session Fixation:** _[Results]_
4. **OAuth Flow Manipulation:** _[Results]_
5. **MFA Bypass:** _[Results]_

---

## Summary of Findings

### Critical Issues (Immediate Action Required)
1. **[Issue Name]** - _[Brief description]_
2. **[Issue Name]** - _[Brief description]_

### High Priority Issues
1. **[Issue Name]** - _[Brief description]_
2. **[Issue Name]** - _[Brief description]_

### Medium Priority Issues
1. **[Issue Name]** - _[Brief description]_

### Low Priority Issues
1. **[Issue Name]** - _[Brief description]_

---

## Recommendations Summary

### Immediate Actions (0-7 days)
1. _[Action]_
2. _[Action]_

### Short-term Actions (1-4 weeks)
1. _[Action]_
2. _[Action]_

### Long-term Improvements (1-3 months)
1. _[Action]_
2. _[Action]_

---

## Conclusion

**Authentication Security Posture:** _[Overall assessment]_

**Key Takeaways:**
- _[Key point]_
- _[Key point]_
- _[Key point]_

**Next Steps:**
1. _[Next step]_
2. _[Next step]_

---

**Assessment completed by:** _[Your name]_  
**Date:** _[YYYY-MM-DD]_  
**Review date:** _[YYYY-MM-DD]_
