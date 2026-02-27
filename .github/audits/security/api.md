---
genre: security
category: api
analysis-type: static
relevance:
  file-patterns:
    - "**/api/**"
    - "**/routes/**"
    - "**/controllers/**"
    - "**/graphql/**"
    - "**/resolvers/**"
  keywords:
    - "api"
    - "endpoint"
    - "rest"
    - "graphql"
    - "swagger"
    - "openapi"
    - "cors"
    - "rate-limit"
  config-keys:
    - "express"
    - "fastify"
    - "@nestjs/core"
    - "flask"
    - "django-rest-framework"
    - "gin-gonic"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# API Security Assessment

**Assessment Date:** _[YYYY-MM-DD]_
**Auditor:** _[Your name]_
**Application:** _[Application name and version]_
**Status:** _[Draft / In Progress / Complete]_

---

<!-- analysis: static -->

## Executive Summary

**Overall API Security Rating:** [ ] Excellent [ ] Good [ ] Fair [ ] Poor [ ] Critical

**Key Findings:**
- Total Vulnerabilities: _[Number]_
- Critical: _[Number]_ | High: _[Number]_ | Medium: _[Number]_ | Low: _[Number]_

**Most Critical Issue:** _[Brief description]_

---

## Scope

### Components Assessed
- [ ] REST API endpoints and methods
- [ ] GraphQL API schema and queries
- [ ] gRPC services and methods
- [ ] Authentication and authorization mechanisms
- [ ] Rate limiting and throttling
- [ ] Input validation and sanitization
- [ ] API documentation exposure
- [ ] API versioning and deprecation

### API Types
- [ ] REST API
- [ ] GraphQL API
- [ ] gRPC API
- [ ] WebSocket API
- [ ] SOAP API

### Out of Scope
_[List what was not assessed]_

---

## 1. Authentication & Authorization

### 1.1 Authentication Mechanisms

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Strong authentication required for protected endpoints
- [ ] API keys/tokens are securely generated and stored
- [ ] Token expiration is properly implemented
- [ ] Token refresh mechanisms are secure
- [ ] Multi-factor authentication available for sensitive operations
- [ ] Authentication bypass is not possible

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Test Results:**
```
Test: Access protected endpoint without authentication
URL: [URL]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 1.2 Authorization Controls

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Authorization is enforced on all endpoints
- [ ] RBAC/ABAC is properly implemented
- [ ] No Insecure Direct Object References (IDOR)
- [ ] Authorization checks on resource-level operations
- [ ] Bulk operations check authorization per item
- [ ] Scope-based authorization for OAuth tokens

**Issues Found:**

| Endpoint | Method | Severity | Issue | Impact |
|----------|--------|----------|-------|--------|
| | | | | |

**IDOR Test Results:**
```
Test: Access User A's resource as User B
Endpoint: [Endpoint]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 2. Input Validation & Data Security

### 2.1 Input Validation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] All inputs are validated (type, format, length)
- [ ] Whitelist validation is used where possible
- [ ] Special characters are properly handled
- [ ] File uploads are validated (type, size, content)
- [ ] JSON/XML parsing has depth/size limits
- [ ] Query parameters are validated

**Issues Found:**

| Endpoint | Parameter | Severity | Issue | Impact |
|----------|-----------|----------|-------|--------|
| | | | | |

**Test Results:**
```
Test: Inject oversized JSON payload
Endpoint: [Endpoint]
Payload Size: [Size]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 2.2 Injection Prevention

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] SQL injection is prevented
- [ ] NoSQL injection is prevented
- [ ] Command injection is prevented
- [ ] LDAP injection is prevented
- [ ] XPath injection is prevented
- [ ] Server-side template injection is prevented

**Issues Found:**

| Endpoint | Injection Type | Severity | Payload | Impact |
|----------|----------------|----------|---------|--------|
| | | | | |

**Test Results:**
```
Test: SQL Injection
Endpoint: [Endpoint]
Payload: ' OR '1'='1
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 2.3 Output Encoding

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] API responses are properly encoded
- [ ] Content-Type headers are correct
- [ ] XSS prevention in responses
- [ ] Error messages don't expose sensitive data
- [ ] Stack traces are not exposed to clients
- [ ] Response size limits prevent DoS

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 3. Rate Limiting & Abuse Prevention

### 3.1 Rate Limiting

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Rate limiting is implemented per user/IP
- [ ] Rate limits are appropriate for endpoint criticality
- [ ] Rate limit headers are returned (X-RateLimit-*)
- [ ] 429 status code returned when limit exceeded
- [ ] Different limits for authenticated vs unauthenticated
- [ ] Rate limits cannot be bypassed

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Test Results:**
```
Test: Exceed rate limit
Endpoint: [Endpoint]
Requests Sent: [Number]
Result: [Pass/Fail]
Rate Limit Enforced: [Yes/No]
```

**Recommendations:**
- _[Specific recommendation]_

### 3.2 Anti-Automation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] CAPTCHA/challenge for suspicious activity
- [ ] Bot detection mechanisms in place
- [ ] API keys required for automation
- [ ] Abuse patterns are detected and blocked
- [ ] Account lockout after repeated failures
- [ ] Velocity checks on sensitive operations

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 4. API-Specific Security

### 4.1 REST API Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] HTTPS enforced for all endpoints
- [ ] HTTP methods are properly restricted
- [ ] OPTIONS method doesn't leak sensitive info
- [ ] DELETE/PUT methods are protected
- [ ] Idempotency keys used where appropriate
- [ ] HEAD requests don't expose sensitive data

**Issues Found:**

| Endpoint | Method | Severity | Issue | Impact |
|----------|--------|----------|-------|--------|
| | | | | |

**Recommendations:**
- _[Specific recommendation]_

### 4.2 GraphQL Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Query depth limiting is enforced
- [ ] Query complexity limiting is enforced
- [ ] Introspection is disabled in production
- [ ] Field-level authorization is implemented
- [ ] N+1 query problems are mitigated
- [ ] Batching limits prevent abuse

**Issues Found:**

| Query | Severity | Issue | Impact |
|-------|----------|-------|--------|
| | | | |

**Test Results:**
```
Test: Deep nested query
Query Depth: [Number]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 4.3 gRPC Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] TLS is required for all gRPC connections
- [ ] Authentication interceptors are implemented
- [ ] Authorization on all RPC methods
- [ ] Message size limits are enforced
- [ ] Streaming endpoints have timeout protection
- [ ] Error details don't expose internals

**Issues Found:**

| Service | Method | Severity | Issue | Impact |
|---------|--------|----------|-------|--------|
| | | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 5. CORS & Cross-Origin Security

### 5.1 CORS Configuration

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] CORS headers are properly configured
- [ ] Wildcard (*) origins are not used in production
- [ ] Credentials are only allowed for trusted origins
- [ ] Preflight requests are handled correctly
- [ ] CORS policies match actual origin requirements
- [ ] No CORS misconfiguration bypasses

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**CORS Configuration:**
```
Access-Control-Allow-Origin: [Value]
Access-Control-Allow-Credentials: [Value]
Access-Control-Allow-Methods: [Value]
Access-Control-Allow-Headers: [Value]
```

**Recommendations:**
- _[Specific recommendation]_

### 5.2 CSRF Protection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] CSRF tokens required for state-changing operations
- [ ] SameSite cookie attribute is set
- [ ] Origin/Referer headers are validated
- [ ] Custom headers required for API calls
- [ ] Double-submit cookie pattern used (if applicable)

**Issues Found:**

| Endpoint | Method | Severity | Issue | Impact |
|----------|--------|----------|-------|--------|
| | | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 6. API Documentation & Versioning

### 6.1 Documentation Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] API documentation requires authentication
- [ ] Swagger/OpenAPI UI is not publicly exposed
- [ ] Documentation doesn't include secrets/credentials
- [ ] Example requests use placeholder data
- [ ] Internal endpoints are not documented publicly
- [ ] Deprecated endpoints are clearly marked

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Exposed Documentation:**
```
URL: [URL]
Access Level: [Public/Authenticated]
Sensitive Info: [Yes/No]
```

**Recommendations:**
- _[Specific recommendation]_

### 6.2 API Versioning

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Versioning strategy is consistent
- [ ] Old versions have sunset timeline
- [ ] Breaking changes are properly versioned
- [ ] Version in URL/header is validated
- [ ] Deprecated versions show warnings
- [ ] No unauthorized access via old API versions

**Issues Found:**

| Version | Severity | Issue | Impact |
|---------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 7. Monitoring & Logging

### 7.1 API Logging

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] All API requests are logged
- [ ] Authentication failures are logged
- [ ] Authorization failures are logged
- [ ] Suspicious patterns are detected and alerted
- [ ] PII is not logged in plaintext
- [ ] Request/response bodies are sanitized in logs

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 7.2 Security Monitoring

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] API abuse is actively monitored
- [ ] Anomaly detection is in place
- [ ] Real-time alerts for critical events
- [ ] Security dashboards track API health
- [ ] Incident response plan includes API security
- [ ] Regular security reviews of API logs

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
- [ ] Postman/Insomnia
- [ ] OWASP ZAP
- [ ] GraphQL Voyager (for GraphQL)
- [ ] grpcurl (for gRPC)
- [ ] Custom fuzzing scripts

### Test Scenarios Executed
1. **Authentication Bypass:** _[Results]_
2. **Authorization Testing:** _[Results]_
3. **Injection Attacks:** _[Results]_
4. **Rate Limit Testing:** _[Results]_
5. **CORS Misconfiguration:** _[Results]_

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

**API Security Posture:** _[Overall assessment]_

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
