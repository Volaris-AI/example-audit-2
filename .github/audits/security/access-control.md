---
genre: security
category: access-control
analysis-type: static
relevance:
  file-patterns:
    - "**/rbac/**"
    - "**/roles/**"
    - "**/permissions/**"
    - "**/middleware/auth*"
    - "**/policies/**"
  keywords:
    - "rbac"
    - "role"
    - "permission"
    - "authorize"
    - "guard"
    - "policy"
    - "acl"
    - "casl"
  config-keys:
    - "casl"
    - "accesscontrol"
    - "casbin"
    - "@casl/ability"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Access Control & Authorization Security Assessment

**Assessment Date:** _[YYYY-MM-DD]_
**Auditor:** _[Your name]_
**Application:** _[Application name and version]_
**Status:** _[Draft / In Progress / Complete]_

---

<!-- analysis: static -->

## Executive Summary

**Overall Access Control Security Rating:** [ ] Excellent [ ] Good [ ] Fair [ ] Poor [ ] Critical

**Key Findings:**
- Total Vulnerabilities: _[Number]_
- Critical: _[Number]_ | High: _[Number]_ | Medium: _[Number]_ | Low: _[Number]_

**Most Critical Issue:** _[Brief description]_

---

## Scope

### Components Assessed
- [ ] User role definitions and RBAC implementation
- [ ] Permission systems and access control lists (ACLs)
- [ ] Administrative interfaces and privilege escalation paths
- [ ] Resource-level authorization checks
- [ ] API endpoint authorization
- [ ] File and directory access controls
- [ ] Database-level access controls

### Out of Scope
_[List what was not assessed]_

---

## 1. Role-Based Access Control (RBAC)

### 1.1 Role Definition & Management

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Roles are clearly defined with documented responsibilities
- [ ] Roles follow principle of least privilege
- [ ] Role hierarchies are properly implemented
- [ ] Default roles are appropriately restrictive
- [ ] Role assignments are auditable

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 1.2 Permission Granularity

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Permissions are granular enough to enforce least privilege
- [ ] Permissions are not overly granular (maintainability)
- [ ] Permission inheritance is logical and secure
- [ ] Permissions are consistently applied across the application

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 2. Authorization Checks

### 2.1 Endpoint Authorization

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] All protected endpoints have authorization checks
- [ ] Authorization is checked on server-side (not client-only)
- [ ] Authorization checks occur before business logic execution
- [ ] Authorization failures return appropriate HTTP status codes
- [ ] No authorization bypass through parameter manipulation

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Code Examples:**
```
[Code snippet showing vulnerable authorization]
```

**Recommendations:**
- _[Specific recommendation]_

### 2.2 Resource-Level Authorization

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Users can only access their own resources (no IDOR)
- [ ] Object-level authorization is consistently enforced
- [ ] Authorization checks use secure identifiers (not guessable IDs)
- [ ] Pagination respects authorization boundaries
- [ ] Bulk operations check authorization for each item

**Issues Found:**

| Resource Type | Severity | Issue | Test Case |
|---------------|----------|-------|-----------|
| | | | |

**IDOR Test Results:**
```
Test: Access User A's resource as User B
URL: [URL]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 2.3 Function-Level Authorization

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Administrative functions require admin role
- [ ] Privileged operations are protected
- [ ] No hidden admin endpoints accessible to regular users
- [ ] API versioning doesn't expose unauthorized functionality

**Issues Found:**

| Function | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 3. Privilege Escalation

### 3.1 Vertical Privilege Escalation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Regular users cannot access admin functions
- [ ] Role changes are properly validated and logged
- [ ] No parameter manipulation allows privilege escalation
- [ ] JWT/token claims cannot be manipulated for higher privileges

**Test Cases:**

| Test | Method | Result | Details |
|------|--------|--------|---------|
| Admin panel access | Direct URL | [Pass/Fail] | [Details] |
| Role parameter tampering | POST data modification | [Pass/Fail] | [Details] |
| Token manipulation | JWT modification | [Pass/Fail] | [Details] |

**Issues Found:**

| Severity | Issue | Exploitation Method | Impact |
|----------|-------|---------------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 3.2 Horizontal Privilege Escalation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Users cannot access other users' data at the same privilege level
- [ ] User IDs cannot be enumerated or guessed
- [ ] Session tokens are user-specific and validated
- [ ] Resource ownership is properly validated

**Test Cases:**

| Test | Method | Result | Details |
|------|--------|--------|---------|
| Access another user's profile | ID manipulation | [Pass/Fail] | [Details] |
| View another user's orders | IDOR | [Pass/Fail] | [Details] |
| Modify another user's data | PUT/PATCH | [Pass/Fail] | [Details] |

**Issues Found:**

| Severity | Issue | Exploitation Method | Impact |
|----------|-------|---------------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 4. Access Control Implementation

### 4.1 Centralized vs Decentralized

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Access control logic is centralized (not scattered)
- [ ] Consistent authorization mechanism across application
- [ ] Authorization middleware/decorators used consistently
- [ ] Access control decisions are auditable

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 4.2 Access Control Lists (ACLs)

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] ACLs are properly defined and enforced
- [ ] ACL bypass is not possible through path traversal
- [ ] ACL modifications are logged
- [ ] Default deny policy is in place

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 5. Special Cases

### 5.1 Administrative Interfaces

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Admin interfaces require authentication and authorization
- [ ] Admin interfaces are not publicly discoverable
- [ ] Admin interfaces use additional security controls (IP whitelisting, MFA)
- [ ] Admin actions are logged with user attribution

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 5.2 API Authorization

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] API keys/tokens include authorization information
- [ ] API rate limiting respects user permissions
- [ ] GraphQL queries enforce field-level authorization
- [ ] REST API endpoints use consistent authorization

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 5.3 File System Access

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] File uploads respect user permissions
- [ ] File downloads verify user authorization
- [ ] Directory traversal is prevented
- [ ] File paths are not user-controllable

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

<!-- analysis: manual -->

## 6. Testing Methodology

### Tools Used
- [ ] Burp Suite (authorization testing)
- [ ] Postman (API authorization)
- [ ] Custom scripts (IDOR testing)
- [ ] Browser developer tools (client-side bypass)

### Test Scenarios Executed
1. **IDOR Testing:** _[Results]_
2. **Privilege Escalation:** _[Results]_
3. **Forced Browsing:** _[Results]_
4. **Parameter Tampering:** _[Results]_
5. **Token Manipulation:** _[Results]_

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

**Access Control Security Posture:** _[Overall assessment]_

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
