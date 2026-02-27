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
- **System Name**: jQuery JavaScript Library
- **Audit Date**: 2026-02-27
- **Auditor**: Infrastructure Auditor (automated static analysis)

> **Context**: jQuery is a client-side DOM manipulation library. UI security is assessed from two angles: (1) security properties of jQuery's own internal DOM operations, and (2) the security risks jQuery exposes to consuming applications. jQuery does not serve HTTP responses or set security headers — those are deployment concerns. The assessment focuses on jQuery's code patterns and what attack surface it presents or mitigates.

<!-- analysis: static -->

## Maturity Level Assessment

| Level | Description | XSS Prevention | CSRF Protection | Security Headers | Content Policy |
|-------|-------------|----------------|-----------------|------------------|----------------|
| **5** | Defense in depth | Full CSP, DOMPurify, auto-escaping | Token + SameSite, double submit | All headers, subresource integrity | Strict CSP, trusted types |
| **4** | Strong protection | Context-aware escaping, CSP | Token-based, SameSite cookies | CSP, XFO, HSTS, referrer policy | CSP report-only → enforced |
| **3** | Basic protection | Template auto-escaping | CSRF tokens | Some headers (XFO, HSTS) | Basic CSP |
| **2** | Minimal | Manual escaping, inconsistent | No CSRF protection | Minimal headers | No CSP |
| **1** | Vulnerable | No escaping, innerHTML with user input | None | None | None |

### Current Maturity Score: 2 / 5

**Justification**:
jQuery's DOM manipulation APIs (`$.html()`, `$.append()`, `$.prepend()`, `$.before()`, `$.after()`) pass HTML strings directly to `innerHTML` without sanitization. This is by design — jQuery is a general-purpose library that intentionally allows consumers to insert arbitrary HTML. The `htmlPrefilter` hook in `buildFragment.js` provides a basic sanitization point but only applies regex normalization of self-closing tags, not XSS prevention. `jQuery.globalEval()` uses `eval` indirectly. No CSP guidance is bundled. No DOMPurify or sanitization helper is provided. There is no CSRF protection layer (library-level, not applicable) and no security header injection (deployment-level). Scoring at Level 2 reflects that the library's design inherently requires callers to handle sanitization, with minimal built-in protection.

**Evidence**:
- **File:** `src/manipulation/buildFragment.js` line 42 — `tmp.innerHTML = wrap[1] + jQuery.htmlPrefilter(elem) + wrap[2]`
- **File:** `src/core.js` lines 270–293 — `jQuery.globalEval()` uses indirect `eval`
- **File:** `src/manipulation.js` lines 28–39 — Regex patterns for script tag detection, not full sanitization

---

## Assessment Areas

### 1. XSS (Cross-Site Scripting) Prevention
- [ ] **Template engine auto-escaping** — jQuery is not a template engine; no auto-escaping
- [ ] **Context-aware encoding** — Not provided; callers must encode
- [ ] **Avoid innerHTML with user input** — jQuery uses `innerHTML` internally (`buildFragment.js` line 42); consumers frequently pass user-supplied HTML to jQuery methods
- [ ] **DOMPurify or sanitization library** — Not bundled or recommended in library
- [ ] **Content-Security-Policy (CSP)** — Not set by library (deployment concern)
- [ ] **Trusted Types (Chrome)** — Not implemented; jQuery's `innerHTML` use would conflict with Trusted Types enforcement without adaptation
- [x] **Input validation on API parameters** — Type guards for API parameters (`jQuery.isFunction`, `jQuery.type()`)
- [ ] **HTTPOnly cookies** — N/A (library does not manage cookies)

**Evidence:**
- **File:** `src/manipulation/buildFragment.js` line 42 — `tmp.innerHTML = wrap[1] + jQuery.htmlPrefilter(elem) + wrap[2]` — innerHTML assignment with only regex preprocessing
- **File:** `src/manipulation.js` line 29 — `rxhtmlTag = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:-]+)[^>]*)\/>/gi` — regex for self-closing tags only
- jQuery 3.5.0 release notes (2020) document XSS-related fixes to `htmlPrefilter`; this 2.2.5-pre branch does not include those fixes

**Risk**: Applications passing unsanitized user input to `$(userInput).appendTo(...)` or `$(elem).html(userInput)` are vulnerable to DOM XSS.

### 2. CSRF (Cross-Site Request Forgery) Protection
- [ ] **CSRF tokens** — Not provided; library-level AJAX does not add CSRF tokens by default
- [ ] **SameSite cookies** — N/A (library does not set cookies)
- [ ] **Double-submit cookie** pattern — Not implemented
- [x] **Custom request headers** — `$.ajax` allows setting custom headers (`X-Requested-With` was historically common; not set by default in 2.x)
- [x] **Referer/Origin validation** — N/A (server responsibility)
- [ ] **State-changing actions require POST** — Callers' responsibility
- [ ] **Re-authentication** — N/A

**Evidence:**
- **File:** `src/ajax.js` lines 633–637 — Custom headers can be set via `s.headers` option; no CSRF token automatically injected
- jQuery 1.x used to set `X-Requested-With: XMLHttpRequest` by default; this version no longer does so unconditionally

### 3. Security Headers
- [ ] **Content-Security-Policy** — N/A (not an HTTP server)
- [ ] **X-Frame-Options** — N/A
- [ ] **X-Content-Type-Options** — N/A
- [ ] **X-XSS-Protection** — N/A
- [ ] **Strict-Transport-Security** — N/A
- [ ] **Referrer-Policy** — N/A
- [ ] **Permissions-Policy** — N/A
- [ ] **Clear-Site-Data** — N/A

> **Note**: jQuery is a client-side library. All HTTP security headers are the responsibility of the server delivering the application. jQuery cannot and does not set HTTP response headers.

### 4. Content Security Policy (CSP)
- [ ] **CSP policy defined** — Not set; deployment concern
- [ ] **Default-src restrictive** — N/A
- [ ] **Script-src no unsafe-inline** — jQuery's `globalEval` uses `eval`/dynamic `<script>` tag injection which requires `unsafe-eval` or `unsafe-inline` in CSP (`src/core.js` lines 280–292)
- [ ] **Style-src restrictive** — N/A
- [ ] **Img-src limited** — N/A
- [ ] **Connect-src** — N/A
- [ ] **Frame-ancestors** — N/A
- [ ] **Report-URI** — N/A
- [ ] **Nonce/hash-based** — Not implemented

**Critical Finding:**
- **File:** `src/core.js` lines 280–292 — `jQuery.globalEval()` creates `<script>` elements and appends/removes them from `document.head`. This pattern is incompatible with strict CSP (no `unsafe-inline` / no `unsafe-eval`). Applications using jQuery that want to enforce strict CSP cannot use `$.globalEval()` or `$.getScript()`.

### 5. Clickjacking Prevention
- [ ] **X-Frame-Options** — N/A (library)
- [ ] **CSP frame-ancestors directive** — N/A
- [ ] **Framebusting JavaScript** — Not provided

### 6. Open Redirects
- [x] **Redirect validation** — N/A (jQuery does not perform redirects)
- [x] **URL handling** — `$.ajax` validates URLs via anchor-based origin parsing (`src/ajax.js` lines 48–49, 535–553)

**Evidence:**
- **File:** `src/ajax.js` lines 535–553 — URL cross-domain detection uses `document.createElement("a")` to compare origins; safe approach

### 7. DOM-Based Vulnerabilities
- [ ] **Avoid eval()** — `jQuery.globalEval()` uses indirect eval (`src/core.js` line 291: `indirect(code)`)
- [ ] **Avoid document.write()** — Not used in jQuery source (positive)
- [ ] **Sanitize DOM manipulation** — `htmlPrefilter` provides minimal normalization only, not XSS sanitization
- [ ] **Trusted Types enforcement** — Not implemented; jQuery is not Trusted Types compatible without a polyfill
- [ ] **Safe navigation** — Callers bear responsibility for URL safety

**Evidence:**
- **File:** `src/core.js` line 272 — `var indirect = eval;` — indirect eval stored for globalEval
- **File:** `src/core.js` line 291 — `indirect(code)` — eval executed indirectly (browser security concern)
- jQuery 3.x introduced `jQuery.htmlPrefilter` as a no-op to allow consumers to plug in sanitizers — this 2.x version has a regex-only implementation

### 8. Third-Party Integration
- [ ] **Subresource Integrity (SRI)** — Not enforced for the library itself (consumers' responsibility); no SRI documentation provided
- [x] **Vet third-party libraries** — Sizzle (CSS selector engine) is a jQuery Foundation project
- [ ] **Sandboxed iframes** — N/A
- [ ] **Permissions-Policy** — N/A
- [x] **External scripts** — `$.getScript()` fetches and executes remote scripts; no integrity checking

**Evidence:**
- **File:** `src/ajax.js` lines 818–820 — `jQuery.getScript()` fetches remote JS with no integrity verification; executing untrusted remote scripts is a supply chain risk

### 9. Client-Side Storage
- [ ] **Sensitive data not in localStorage** — N/A; jQuery does not manage localStorage
- [ ] **HttpOnly cookies** — N/A
- [ ] **Secure flag on cookies** — N/A; jQuery does not set cookies
- [ ] **SameSite flag** — N/A
- [ ] **Encryption** — N/A

### 10. User Input Handling
- [ ] **Input validation client-side** — Not provided by library (callers' responsibility)
- [x] **Type checking** — jQuery API parameters are type-checked (`isFunction`, `isArray`, `type()`)
- [ ] **File upload validation** — Not provided
- [ ] **Dangerous file types** — N/A
- [ ] **Virus scanning** — N/A

## Recommendations

**Level 2→3**:
- Add a recommended security pattern to documentation for sanitizing HTML before passing to jQuery DOM methods
- Provide an `htmlPrefilter` hook that defaults to DOMPurify sanitization (jQuery 3.5+ approach)
- Document Trusted Types compatibility requirements and provide migration guidance

**Level 3→4**:
- Integrate DOMPurify or a compatible sanitization library as an optional plugin
- Document CSRF protection patterns for `$.ajax` consumers (adding CSRF token to `$.ajaxSetup`)
- Provide `$.getScript()` with an integrity option

## Success Criteria
- `jQuery.htmlPrefilter` defaults to safe behavior (no raw HTML injection without opt-in)
- `$.globalEval` usage documented as CSP-incompatible
- Documentation page on secure usage of jQuery DOM methods
- Trusted Types adapter provided

---
**Document Version**: 1.0
**Last Updated**: 2026-02-27
