---
genre: security
category: ui-security
analysis-type: static
relevance:
  file-patterns:
    - "**/components/**"
    - "**/views/**"
    - "**/pages/**"
    - "**/templates/**"
    - "**/public/**"
  keywords:
    - "xss"
    - "csp"
    - "innerHTML"
    - "dangerouslySetInnerHTML"
    - "sanitize"
    - "escape"
    - "script"
    - "iframe"
  config-keys:
    - "dompurify"
    - "xss"
    - "helmet"
    - "csp"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# UI/Frontend Security Assessment

**Assessment Date:** _[YYYY-MM-DD]_
**Auditor:** _[Your name]_
**Application:** _[Application name and version]_
**Status:** _[Draft / In Progress / Complete]_

---

<!-- analysis: static -->

## Executive Summary

**Overall UI Security Rating:** [ ] Excellent [ ] Good [ ] Fair [ ] Poor [ ] Critical

**Key Findings:**
- Total Vulnerabilities: _[Number]_
- Critical: _[Number]_ | High: _[Number]_ | Medium: _[Number]_ | Low: _[Number]_

**Most Critical Issue:** _[Brief description]_

---

## Scope

### Components Assessed
- [ ] Cross-Site Scripting (XSS) vulnerabilities
- [ ] Cross-Site Request Forgery (CSRF) protection
- [ ] Clickjacking protection
- [ ] Content Security Policy (CSP)
- [ ] DOM manipulation security
- [ ] Client-side validation
- [ ] Sensitive data exposure
- [ ] Browser security features

### Technologies
- [ ] React / Vue / Angular
- [ ] jQuery / Vanilla JS
- [ ] WebAssembly
- [ ] Service Workers
- [ ] WebSockets

### Out of Scope
_[List what was not assessed]_

---

## 1. Cross-Site Scripting (XSS)

### 1.1 Stored XSS

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] User input properly escaped before storage
- [ ] Output encoding when displaying stored content
- [ ] Rich text editor sanitizes HTML
- [ ] No unescaped database content in templates
- [ ] File uploads don't contain executable content
- [ ] User profile data escaped on display

**Issues Found:**

| Location | Input Source | Severity | Payload | Impact |
|----------|--------------|----------|---------|--------|
| | | | | |

**Test Results:**
```
Test: Stored XSS via comment field
Payload: <script>alert('XSS')</script>
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 1.2 Reflected XSS

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] URL parameters escaped in output
- [ ] Search queries sanitized before display
- [ ] Error messages don't reflect unescaped input
- [ ] No unencoded data in HTML attributes
- [ ] JavaScript string contexts properly escaped
- [ ] Query parameter pollution prevented

**Issues Found:**

| Endpoint | Parameter | Severity | Payload | Impact |
|----------|-----------|----------|---------|--------|
| | | | | |

**Test Results:**
```
Test: Reflected XSS via search parameter
URL: /search?q=<img src=x onerror=alert(1)>
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 1.3 DOM-based XSS

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] No unsafe use of innerHTML
- [ ] document.write() avoided
- [ ] eval() not used with user input
- [ ] Location properties (hash, search) sanitized
- [ ] postMessage properly validated
- [ ] Safe DOM APIs used (textContent, setAttribute)

**Issues Found:**

| Location | Sink | Severity | Payload | Impact |
|----------|------|----------|---------|--------|
| | | | | |

**Test Results:**
```
Test: DOM XSS via URL fragment
URL: /#<img src=x onerror=alert(1)>
JavaScript: element.innerHTML = location.hash
Result: [Pass/Fail]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 2. Cross-Site Request Forgery (CSRF)

### 2.1 CSRF Protection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] CSRF tokens on all state-changing operations
- [ ] Tokens validated server-side
- [ ] SameSite cookie attribute set
- [ ] Custom headers required for AJAX requests
- [ ] No GET requests for state changes
- [ ] Double-submit cookie pattern (if applicable)

**Issues Found:**

| Endpoint | Method | Severity | Issue | Impact |
|----------|--------|----------|-------|--------|
| | | | | |

**Test Results:**
```
Test: CSRF on password change
Method: POST /change-password
CSRF Token: [Present/Absent]
Result: [Pass/Fail]
```

**Recommendations:**
- _[Specific recommendation]_

### 2.2 Origin Validation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Origin header validated
- [ ] Referer header validated (as secondary check)
- [ ] CORS policies properly configured
- [ ] No wildcard CORS on authenticated endpoints
- [ ] Preflight requests properly handled
- [ ] Credentials only allowed for specific origins

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 3. Clickjacking Protection

### 3.1 Frame Protection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] X-Frame-Options header set (DENY/SAMEORIGIN)
- [ ] CSP frame-ancestors directive configured
- [ ] No sensitive operations in iframeable pages
- [ ] Frame-busting code implemented (if needed)
- [ ] Frameable pages clearly identified
- [ ] User interaction verification for sensitive actions

**Issues Found:**

| Page | Severity | Issue | Header Missing |
|------|----------|-------|----------------|
| | | | |

**Test Results:**
```
Test: Load application in iframe
X-Frame-Options: [Value]
CSP frame-ancestors: [Value]
Result: [Pass/Fail]
```

**Recommendations:**
- _[Specific recommendation]_

### 3.2 UI Redressing

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Transparent overlay attacks prevented
- [ ] Cursor jacking mitigated
- [ ] Drag-and-drop attacks prevented
- [ ] Sensitive buttons have delay/confirmation
- [ ] Visual confirmation of actions
- [ ] Focus management prevents deception

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 4. Content Security Policy (CSP)

### 4.1 CSP Configuration

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] CSP header implemented
- [ ] No unsafe-inline for scripts
- [ ] No unsafe-eval
- [ ] Whitelist approach for sources
- [ ] Nonces or hashes for inline scripts
- [ ] Report-only mode tested before enforcement

**Issues Found:**

| Directive | Severity | Issue | Impact |
|-----------|----------|-------|--------|
| | | | |

**CSP Policy:**
```
Content-Security-Policy: 
  default-src 'self'; 
  script-src 'self' 'nonce-{random}'; 
  style-src 'self' 'unsafe-inline'; 
  img-src 'self' data: https:;
```

**Recommendations:**
- _[Specific recommendation]_

### 4.2 CSP Effectiveness

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] CSP blocks XSS attempts
- [ ] CSP violations logged and monitored
- [ ] No CSP bypass techniques possible
- [ ] Third-party scripts whitelisted appropriately
- [ ] Upgrade-insecure-requests directive used
- [ ] CSP doesn't break application functionality

**Issues Found:**

| Severity | Issue | Bypass Method | Impact |
|----------|-------|---------------|--------|
| | | | |

**Test Results:**
```
Test: Attempt XSS with CSP enabled
Payload: <script>alert(1)</script>
CSP Blocked: [Yes/No]
Violation Report: [Generated/Not Generated]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 5. DOM Manipulation Security

### 5.1 Safe DOM Operations

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] User input sanitized before DOM insertion
- [ ] Template engines auto-escape by default
- [ ] React/Vue/Angular context-aware escaping used
- [ ] jQuery .text() used instead of .html() where possible
- [ ] DOMPurify or similar sanitizer used
- [ ] No direct HTML string concatenation

**Issues Found:**

| Location | Method | Severity | Issue | Impact |
|----------|--------|----------|-------|--------|
| | | | | |

**Code Examples:**
```javascript
// Vulnerable
element.innerHTML = userInput;

// Secure
element.textContent = userInput;
// or
element.innerHTML = DOMPurify.sanitize(userInput);
```

**Recommendations:**
- _[Specific recommendation]_

### 5.2 Event Handlers

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Event handlers not constructed from user input
- [ ] Inline event handlers avoided (onclick, onerror)
- [ ] addEventListener used instead of inline handlers
- [ ] Event delegation properly implemented
- [ ] No eval() in event handling
- [ ] Trusted Types API used (if applicable)

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 6. Client-Side Validation

### 6.1 Input Validation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Client-side validation present for UX
- [ ] Server-side validation enforced (never trust client)
- [ ] Validation can't be bypassed via browser tools
- [ ] Input length limits enforced
- [ ] Data type validation implemented
- [ ] Format validation (email, phone, etc.)

**Issues Found:**

| Field | Severity | Issue | Impact |
|-------|----------|-------|--------|
| | | | |

**Test Results:**
```
Test: Bypass client-side validation
Method: Browser DevTools / Burp Suite
Field: [Field name]
Result: [Server rejected/Server accepted]
```

**Recommendations:**
- _[Specific recommendation]_

### 6.2 Form Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Hidden fields not used for security
- [ ] Form tampering detected server-side
- [ ] File upload validation on client and server
- [ ] Autocomplete disabled for sensitive fields
- [ ] Form submission rate limited
- [ ] Multi-step forms validate each step

**Issues Found:**

| Form | Severity | Issue | Impact |
|------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 7. Sensitive Data Exposure

### 7.1 Client-Side Data Storage

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] No sensitive data in localStorage/sessionStorage
- [ ] Cookies have Secure and HttpOnly flags
- [ ] IndexedDB doesn't store unencrypted sensitive data
- [ ] Client-side cache cleared on logout
- [ ] No PII in browser history
- [ ] Web Storage cleared on sensitive operations

**Issues Found:**

| Storage Type | Data Type | Severity | Issue | Impact |
|--------------|-----------|----------|-------|--------|
| | | | | |

**Test Results:**
```
Test: Check localStorage for sensitive data
Keys Found: [List]
Sensitive Data: [Yes/No]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 7.2 Information Disclosure

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] No sensitive data in JavaScript source
- [ ] API keys not in client-side code
- [ ] No comments with sensitive information
- [ ] Stack traces not exposed to users
- [ ] Error messages generic (not revealing)
- [ ] Source maps disabled in production

**Issues Found:**

| Location | Severity | Data Exposed | Impact |
|----------|----------|--------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 7.3 Browser Autocomplete

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] autocomplete="off" on sensitive fields
- [ ] Password fields properly marked
- [ ] Credit card fields properly marked
- [ ] One-time codes have autocomplete="one-time-code"
- [ ] New password fields have autocomplete="new-password"
- [ ] Browser password managers handled correctly

**Issues Found:**

| Field | Severity | Issue | Impact |
|-------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 8. Browser Security Features

### 8.1 Security Headers

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options properly set
- [ ] Referrer-Policy configured
- [ ] Permissions-Policy configured
- [ ] Strict-Transport-Security (HSTS) enabled
- [ ] Cross-Origin-Embedder-Policy (COEP) set

**Issues Found:**

| Header | Severity | Issue | Impact |
|--------|----------|-------|--------|
| | | | |

**Security Headers:**
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=()
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**Recommendations:**
- _[Specific recommendation]_

### 8.2 Subresource Integrity (SRI)

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] SRI hashes for CDN-hosted resources
- [ ] SRI for JavaScript libraries
- [ ] SRI for CSS files
- [ ] Fallback for SRI failures
- [ ] SRI hashes regularly updated
- [ ] Crossorigin attribute set correctly

**Issues Found:**

| Resource | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Example:**
```html
<script src="https://cdn.example.com/lib.js"
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux..."
        crossorigin="anonymous"></script>
```

**Recommendations:**
- _[Specific recommendation]_

---

<!-- analysis: manual -->

## 9. Testing Methodology

### Tools Used
- [ ] Burp Suite Professional
- [ ] OWASP ZAP
- [ ] Browser Developer Tools
- [ ] XSS Hunter / XSS Strike
- [ ] CSP Evaluator
- [ ] SecurityHeaders.com

### Test Scenarios Executed
1. **XSS Testing (Stored, Reflected, DOM):** _[Results]_
2. **CSRF Testing:** _[Results]_
3. **Clickjacking Test:** _[Results]_
4. **CSP Bypass Attempts:** _[Results]_
5. **Client-Side Validation Bypass:** _[Results]_

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

**UI Security Posture:** _[Overall assessment]_

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
