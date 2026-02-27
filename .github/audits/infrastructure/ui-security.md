---
genre: infrastructure
category: ui-security
analysis-type: static
relevance:
  file-patterns:
    - "**/components/**"
    - "**/views/**"
    - "**/pages/**"
  keywords:
    - "xss"
    - "csp"
    - "innerHTML"
    - "dangerouslySetInnerHTML"
    - "sanitize"
  config-keys:
    - "dompurify"
    - "helmet"
    - "csp"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# UI Security Infrastructure Audit

## System Information
- **System Name**: 
- **Audit Date**: 
- **Auditor**: 

<!-- analysis: static -->

## Maturity Level Assessment

| Level | Description | XSS Prevention | CSRF Protection | Security Headers | Content Policy |
|-------|-------------|----------------|-----------------|------------------|----------------|
| **5** | Defense in depth | Full CSP, DOMPurify, auto-escaping | Token + SameSite, double submit | All headers, subresource integrity | Strict CSP, trusted types |
| **4** | Strong protection | Context-aware escaping, CSP | Token-based, SameSite cookies | CSP, XFO, HSTS, referrer policy | CSP report-only → enforced |
| **3** | Basic protection | Template auto-escaping | CSRF tokens | Some headers (XFO, HSTS) | Basic CSP |
| **2** | Minimal | Manual escaping, inconsistent | No CSRF protection | Minimal headers | No CSP |
| **1** | Vulnerable | No escaping, innerHTML with user input | None | None | None |

### Current Maturity Score: [ ] / 5

## Assessment Areas

### 1. XSS (Cross-Site Scripting) Prevention
- [ ] **Template engine** auto-escaping (React, Vue, Angular)
- [ ] **Context-aware** encoding (HTML, JS, URL, CSS)
- [ ] **Avoid innerHTML** with user input
- [ ] **DOMPurify** or sanitization library
- [ ] **Content-Security-Policy** (CSP)
- [ ] **Trusted Types** (Chrome)
- [ ] **Input validation** on client and server
- [ ] **HTTPOnly cookies** for sensitive data

### 2. CSRF (Cross-Site Request Forgery) Protection
- [ ] **CSRF tokens** (synchronizer token)
- [ ] **SameSite cookies**: Strict / Lax / None
- [ ] **Double-submit cookie** pattern
- [ ] **Custom request headers** (X-Requested-With)
- [ ] **Referer/Origin** validation
- [ ] **State-changing actions** require POST
- [ ] **Re-authentication** for sensitive actions

### 3. Security Headers
- [ ] **Content-Security-Policy** (CSP): Configured / Report-Only / None
- [ ] **X-Frame-Options**: DENY / SAMEORIGIN
- [ ] **X-Content-Type-Options**: nosniff
- [ ] **X-XSS-Protection**: 1; mode=block
- [ ] **Strict-Transport-Security** (HSTS): max-age, includeSubDomains
- [ ] **Referrer-Policy**: no-referrer / strict-origin-when-cross-origin
- [ ] **Permissions-Policy**: Restrictive
- [ ] **Clear-Site-Data** (on logout)

### 4. Content Security Policy (CSP)
- [ ] **CSP policy** defined
- [ ] **Default-src** restrictive ('self')
- [ ] **Script-src** no 'unsafe-inline' or 'unsafe-eval'
- [ ] **Style-src** restrictive
- [ ] **Img-src** limited
- [ ] **Connect-src** API endpoints only
- [ ] **Frame-ancestors** restricted
- [ ] **Report-URI** or report-to configured
- [ ] **Nonce** or hash-based for inline scripts

### 5. Clickjacking Prevention
- [ ] **X-Frame-Options** set
- [ ] **CSP frame-ancestors** directive
- [ ] **Framebusting** JavaScript (legacy fallback)

### 6. Open Redirects
- [ ] **Whitelist** for redirect URLs
- [ ] **Validate** redirect parameters
- [ ] **Relative URLs** preferred
- [ ] **User confirmation** for external redirects

### 7. DOM-Based Vulnerabilities
- [ ] **Avoid eval()** and Function()
- [ ] **Avoid document.write()**
- [ ] **Sanitize** DOM manipulation
- [ ] **Trusted Types** enforcement
- [ ] **Safe navigation** (no window.location from user input)

### 8. Third-Party Integration
- [ ] **Subresource Integrity** (SRI) for CDN
- [ ] **Vet third-party** libraries
- [ ] **Sandboxed iframes** for untrusted content
- [ ] **Permissions-Policy** to restrict third-party features
- [ ] **Content-Security-Policy** for scripts

### 9. Client-Side Storage
- [ ] **Sensitive data** not in localStorage (use sessionStorage or cookies with security flags)
- [ ] **HttpOnly cookies** for auth tokens
- [ ] **Secure flag** on cookies (HTTPS only)
- [ ] **SameSite flag** on cookies
- [ ] **Encryption** for sensitive data in storage

### 10. User Input Handling
- [ ] **Input validation** on client (UX) and server (security)
- [ ] **Type checking** for user inputs
- [ ] **File upload** validation (type, size, content)
- [ ] **Dangerous file types** blocked
- [ ] **Virus scanning** for uploads

## Recommendations
**Level 1→2**: Template auto-escaping, basic input validation, HTTPS
**Level 2→3**: CSRF tokens, security headers (XFO, HSTS), basic CSP
**Level 3→4**: Strong CSP, SameSite cookies, all security headers, DOMPurify
**Level 4→5**: Trusted Types, strict CSP, SRI, comprehensive defense in depth

## Success Criteria
- Zero XSS vulnerabilities
- CSRF protection on all state-changing actions
- All security headers configured
- CSP enforced (not report-only)
- Third-party scripts with SRI

---
**Document Version**: 1.0
