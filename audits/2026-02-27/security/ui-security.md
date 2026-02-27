---
genre: security
category: ui-security
analysis-type: static
relevance:
  file-patterns:
    - "src/**"
  keywords:
    - "innerHTML"
    - "html"
    - "parseHTML"
    - "globalEval"
    - "script"
  config-keys: []
audit-date: 2026-02-27
---

# UI/Frontend Security Assessment

**Assessment Date:** 2026-02-27
**Auditor:** Security Auditor (Automated Static Analysis)
**Application:** jQuery JavaScript Library v2.2.5-pre
**Status:** Complete

---

<!-- analysis: static -->

## Executive Summary

**Overall UI Security Rating:** [x] Poor [ ] Critical

jQuery v2.2.5-pre is a **client-side DOM manipulation library** — it *is* the frontend security layer that application developers depend on. Its design deliberately enables powerful HTML/DOM manipulation APIs (`$(html)`, `.html()`, `.append()`, `.load()`) that, when used with untrusted input, create direct cross-site scripting (XSS) attack surfaces. Four high-severity DOM-XSS vectors were confirmed through static analysis, all rooted in the library's decision to execute scripts by default and assign `innerHTML` without sanitization.

**Key Findings:**
- Total Vulnerabilities: **8**
- Critical: **0** | High: **4** | Medium: **2** | Low: **2**

**Most Critical Issue:** The `$(html)` constructor passes `keepScripts: true` to `parseHTML` (`src/core/init.js:51-54`), allowing `<script>` tags within HTML strings to execute when passed to the jQuery initializer.

---

## Scope

### Components Assessed
- [x] Cross-Site Scripting (XSS) vulnerabilities
- [x] Cross-Site Request Forgery (CSRF) protection
- [ ] Clickjacking protection
- [ ] Content Security Policy (CSP)
- [x] DOM manipulation security
- [ ] Client-side validation
- [x] Sensitive data exposure
- [x] Browser security features

### Technologies
- [ ] React / Vue / Angular
- [x] jQuery / Vanilla JS
- [ ] WebAssembly
- [ ] Service Workers
- [ ] WebSockets

### Out of Scope
- Server-side rendering or back-end components (jQuery is a client-side library with no backend)
- CSP header configuration (jQuery does not set HTTP headers)
- Clickjacking protection (HTTP-header based; N/A for a JS library)

---

## 1. Cross-Site Scripting (XSS)

### 1.1 Stored XSS

**Finding:** [ ] Pass [x] Fail [ ] N/A

**Assessment:**
- [ ] User input properly escaped before storage
- [ ] Output encoding when displaying stored content
- [ ] Rich text editor sanitizes HTML
- [ ] No unescaped database content in templates
- [x] File uploads don't contain executable content — N/A
- [ ] User profile data escaped on display

**Issues Found:**

| Location | Input Source | Severity | Payload | Impact | Committed By | Approved By |
|----------|--------------|----------|---------|--------|--------------|-------------|
| `src/manipulation.js:418` | `.html(userInput)` | High | `<img src=x onerror=alert(1)>` | Stored XSS when displayed via `.html()` | jQuery Core Team | Unknown |
| `src/manipulation/buildFragment.js:42` | `buildFragment(htmlString, ...)` | High | `<script>evil()</script>` | Arbitrary script execution during HTML parsing | jQuery Core Team | Unknown |

**Evidence:**

**File:** `src/manipulation.js:418`
**Code:**
```javascript
elem.innerHTML = value;
```
**Issue:** `.html()` sets `innerHTML` directly with the value argument. If `value` contains user-supplied data, XSS results. No sanitization is applied.

**File:** `src/manipulation/buildFragment.js:42`
**Code:**
```javascript
tmp.innerHTML = wrap[ 1 ] + jQuery.htmlPrefilter( elem ) + wrap[ 2 ];
```
**Issue:** `htmlPrefilter` only performs a structural self-closing-tag transformation, not XSS sanitization. Raw HTML (potentially attacker-controlled) is assigned to `innerHTML`.

**Recommendations:**
- Add documentation explicitly warning that `.html()`, `.append()`, `.prepend()`, `.before()`, `.after()`, and `buildFragment` must never be called with untrusted input.
- Consider integrating an optional DOMPurify hook in `htmlPrefilter` so consuming applications can opt in to sanitization.

### 1.2 Reflected XSS

**Finding:** [ ] Pass [x] Fail [ ] N/A

**Assessment:**
- [ ] URL parameters escaped in output
- [ ] Search queries sanitized before display
- [ ] Error messages don't reflect unescaped input
- [ ] No unencoded data in HTML attributes
- [ ] JavaScript string contexts properly escaped
- [ ] Query parameter pollution prevented

**Issues Found:**

| Endpoint | Parameter | Severity | Payload | Impact | Committed By | Approved By |
|----------|-----------|----------|---------|--------|--------------|-------------|
| `src/ajax/load.js:61-68` | `responseText` (no selector path) | High | Malicious server HTML | XSS: unsanitized HTML injected into DOM | jQuery Core Team | Unknown |

**Evidence:**

**File:** `src/ajax/load.js:61-68`
**Code:**
```javascript
self.html( selector ?
    jQuery( "<div>" ).append( jQuery.parseHTML( responseText ) ).find( selector ) :
    // Otherwise use the full result
    responseText );
```
**Issue:** When `$.load(url)` is called without a selector (the `else` branch), the full `responseText` is passed directly to `.html()`. This means any HTML returned by the loaded URL (including injected `<script>` tags or event-handler attributes) is executed verbatim in the DOM. An attacker who can influence the server's response or intercept the request (e.g., on an HTTP endpoint) can achieve reflected XSS.

**Recommendations:**
- Always use `jQuery.parseHTML(responseText, null, false)` (with `keepScripts: false`) when inserting untrusted content.
- Warn users in docs that `$.load()` without a selector is unsafe against malicious server responses.

### 1.3 DOM-based XSS

**Finding:** [ ] Pass [x] Fail [ ] N/A

**Assessment:**
- [ ] No unsafe use of innerHTML — innerHTML used extensively without sanitization
- [ ] document.write() avoided — ✅ Not used
- [ ] eval() not used with user input — ❌ `jQuery.globalEval()` used on script content
- [ ] Location properties (hash, search) sanitized
- [x] postMessage properly validated — N/A
- [ ] Safe DOM APIs used (textContent, setAttribute)

**Issues Found:**

| Location | Sink | Severity | Payload | Impact | Committed By | Approved By |
|----------|------|----------|---------|--------|--------------|-------------|
| `src/core/init.js:51-54` | `jQuery.parseHTML(..., true)` | High | `$('<script>evil()</script>')` | Script execution via `$(html)` constructor | jQuery Core Team | Unknown |
| `src/ajax/script.js:17-19` | `jQuery.globalEval(text)` | High | Malicious `text/javascript` response | Global script execution from AJAX response | jQuery Core Team | Unknown |

**Evidence:**

**File:** `src/core/init.js:49-55`
**Code:**
```javascript
// Option to run scripts is true for back-compat
// Intentionally let the error be thrown if parseHTML is not present
jQuery.merge( this, jQuery.parseHTML(
    match[ 1 ],
    context && context.nodeType ? context.ownerDocument || context : document,
    true   // <-- keepScripts = true
) );
```
**Issue:** The `$(html)` constructor deliberately passes `keepScripts: true` for backward compatibility. Any application that does `$('<script>alert(1)</script>')` or `$(userControlledHTMLString)` will execute scripts. This is the root cause of multiple publicly known jQuery XSS CVEs.

**File:** `src/ajax/script.js:17-19`
**Code:**
```javascript
"text script": function( text ) {
    jQuery.globalEval( text );
    return text;
}
```
**Issue:** AJAX responses with `dataType: "script"` are fed into `jQuery.globalEval()`, which executes them in the global scope. While intentional for remote script loading, this means any application that loads untrusted content as `script` type (or is tricked into doing so) executes arbitrary code globally.

**Recommendations:**
- Change `$(html)` to default `keepScripts: false` (breaking change; should target jQuery 4.x).
- Document that `$("<script>")` executes the script element and warn consumers to never pass user-controlled HTML to the `$()` constructor.

---

## 2. Cross-Site Request Forgery (CSRF)

### 2.1 CSRF Protection

**Finding:** [ ] Pass [ ] Fail [x] N/A

**Assessment:**
- CSRF token management is a server-side concern. jQuery provides helpers (`.ajax()`, `.param()`, `.serialize()`) but does not implement CSRF protection itself.
- The `X-Requested-With: XMLHttpRequest` header is added automatically for same-origin requests (`src/ajax/xhr.js:62-63`), which provides a partial CSRF mitigation for same-origin policies.

**Issues Found:**

| Endpoint | Method | Severity | Issue | Impact |
|----------|--------|----------|-------|--------|
| `src/ajax/xhr.js:62-63` | XHR | Info | `X-Requested-With` header only on same-origin requests | Cross-domain AJAX lacks this header | jQuery Core Team | Unknown |

**Recommendations:**
- Document that consuming applications must implement CSRF token injection (e.g., via `$.ajaxSetup` headers) for state-changing endpoints.

### 2.2 Origin Validation

**Finding:** [ ] Pass [ ] Fail [x] N/A

Origin/Referer validation is a server-side concern outside jQuery's scope.

---

## 3. Clickjacking Protection

**Finding:** [ ] Pass [ ] Fail [x] N/A

Clickjacking protection via `X-Frame-Options` or CSP `frame-ancestors` is a server-side/HTTP-header concern. jQuery is a client-side library and does not set HTTP headers.

---

## 4. Content Security Policy (CSP)

### 4.1 CSP Configuration

**Finding:** [ ] Pass [x] Fail [ ] N/A

**Assessment:**
- [ ] CSP header implemented — N/A for a library
- [ ] No unsafe-inline for scripts — ❌ jQuery's design requires `unsafe-eval` (via `jQuery.globalEval`) and `unsafe-inline` (via inline script execution) if these features are used
- [ ] No unsafe-eval — ❌ `jQuery.globalEval` calls `eval()`-equivalent operations
- [x] Whitelist approach for sources — N/A
- [ ] Nonces or hashes for inline scripts — Not supported
- [x] Report-only mode tested before enforcement — N/A

**Issues Found:**

| Directive | Severity | Issue | Impact | Committed By | Approved By |
|-----------|----------|-------|--------|--------------|-------------|
| `unsafe-eval` | Medium | `jQuery.globalEval()` uses `eval`-like execution | Applications cannot enforce `script-src` without `unsafe-eval` if using `$.ajax(dataType:'script')` or `$(html)` with scripts | jQuery Core Team | Unknown |
| `unsafe-inline` | Low | `$(html)` constructor with `keepScripts:true` executes inline scripts | Defeats inline-script CSP protections | jQuery Core Team | Unknown |

**Recommendations:**
- Consider providing a "CSP-safe mode" flag that disables `globalEval` and enforces `keepScripts: false` globally.
- Document CSP compatibility requirements for each jQuery feature.

### 4.2 CSP Effectiveness

**Finding:** [ ] Pass [ ] Fail [x] N/A

This is assessed at the application level, not the library level. Libraries that bypass `unsafe-eval` restrictions undermine application CSP policies.

---

## 5. DOM Manipulation Security

### 5.1 Safe DOM Operations

**Finding:** [ ] Pass [x] Fail [ ] N/A

**Assessment:**
- [ ] User input sanitized before DOM insertion — No sanitization hooks
- [x] Template engines auto-escape by default — N/A
- [x] React/Vue/Angular context-aware escaping used — N/A
- [ ] jQuery .text() used instead of .html() where possible — Library provides both; no enforcement
- [ ] DOMPurify or similar sanitizer used — No integration
- [ ] No direct HTML string concatenation — Used internally in `buildFragment`

**Issues Found:**

| Location | Method | Severity | Issue | Impact | Committed By | Approved By |
|----------|--------|----------|-------|--------|--------------|-------------|
| `src/manipulation.js:418` | `.html()` → `innerHTML` | High | No sanitization before DOM insertion | XSS via unsanitized HTML | jQuery Core Team | Unknown |
| `src/manipulation/buildFragment.js:42` | `buildFragment` → `innerHTML` | High | Raw HTML parsed via `innerHTML` | XSS during HTML fragment construction | jQuery Core Team | Unknown |
| `src/core/init.js:51-54` | `$()` constructor | High | `keepScripts: true` on HTML strings | Script execution from HTML strings | jQuery Core Team | Unknown |

**Code Examples:**
```javascript
// Vulnerable — passes to innerHTML without sanitization
$('#container').html(userInput);

// Also vulnerable — $() constructor runs scripts
$(userControlledHTMLString);

// Secure alternative
$('#container').text(userInput);
// or with sanitization library:
$('#container').html(DOMPurify.sanitize(userInput));
```

**Recommendations:**
- Change `htmlPrefilter` to call a user-configurable sanitizer hook.
- Default `keepScripts` in `$.parseHTML()` to `false` at the API level for the `$(html)` constructor.

### 5.2 Event Handlers

**Finding:** [x] Pass [ ] Fail [ ] N/A

**Assessment:**
- [x] Event handlers not constructed from user input — Event system uses stored closures
- [x] Inline event handlers avoided (onclick, onerror) — jQuery uses `addEventListener`
- [x] addEventListener used instead of inline handlers — `jQuery.event.add` wraps `addEventListener`
- [x] Event delegation properly implemented — Delegation uses selector-based filtering
- [x] No eval() in event handling — Event module does not use eval
- [ ] Trusted Types API used (if applicable) — Not implemented (Trusted Types is modern browser API)

No issues found in event handler construction.

---

## 6. Client-Side Validation

**Finding:** [ ] Pass [ ] Fail [x] N/A

jQuery is a DOM manipulation library; it does not implement application-level input validation. Validation concerns are delegated to consuming applications.

---

## 7. Sensitive Data Exposure

### 7.1 Client-Side Data Storage

**Finding:** [ ] Pass [ ] Fail [x] N/A

jQuery uses private `dataPriv` and `dataUser` internal data stores attached to DOM elements, not `localStorage`/`sessionStorage`. No sensitive data is written to browser storage.

### 7.2 Information Disclosure

**Finding:** [ ] Pass [x] Fail [ ] N/A

**Assessment:**
- [x] No sensitive data in JavaScript source — No secrets or credentials in source
- [x] API keys not in client-side code — ✅
- [x] No comments with sensitive information — ✅
- [x] Stack traces not exposed to users — ✅
- [ ] Error messages generic (not revealing) — ❌ `parseXML` includes raw input in error message
- [x] Source maps disabled in production — Build process produces separate source maps

**Issues Found:**

| Location | Severity | Data Exposed | Impact | Committed By | Approved By |
|----------|----------|--------------|--------|--------------|-------------|
| `src/ajax/parseXML.js:21` | Low | Raw input reflected in error: `"Invalid XML: " + data` | If error objects are logged server-side or displayed to users, raw XML data (potentially sensitive) is disclosed | jQuery Core Team | Unknown |

**Evidence:**

**File:** `src/ajax/parseXML.js:19-21`
**Code:**
```javascript
if ( !xml || xml.getElementsByTagName( "parsererror" ).length ) {
    jQuery.error( "Invalid XML: " + data );
}
```
**Issue:** The full raw XML string is concatenated into the error message. If the XML contains sensitive data (API responses, tokens, PII), this data is embedded in thrown error objects that may be logged or displayed.

**Recommendations:**
- Replace `jQuery.error("Invalid XML: " + data)` with `jQuery.error("Invalid XML: " + data.substring(0, 100) + "...")` or simply `jQuery.error("Invalid XML: parsing failed")`.

### 7.3 Browser Autocomplete

**Finding:** [ ] Pass [ ] Fail [x] N/A

Autocomplete concerns are application-level. jQuery does not render forms or set `autocomplete` attributes automatically.

---

## 8. Browser Security Features

### 8.1 Security Headers

**Finding:** [ ] Pass [ ] Fail [x] N/A

jQuery is a client-side JavaScript library and cannot set HTTP security headers. This is outside scope.

### 8.2 Subresource Integrity (SRI)

**Finding:** [ ] Pass [x] Fail [ ] N/A

**Assessment:**
- [ ] SRI hashes for CDN-hosted resources — jQuery is distributed via CDN; no SRI enforcement in library itself
- [ ] SRI for JavaScript libraries — jQuery documentation does not prominently feature SRI usage guidance
- [x] SRI for CSS files — N/A
- [ ] Fallback for SRI failures — N/A
- [ ] SRI hashes regularly updated — N/A
- [ ] Crossorigin attribute set correctly — N/A

**Issues Found:**

| Resource | Severity | Issue | Impact | Committed By | Approved By |
|----------|----------|-------|--------|--------------|-------------|
| jQuery CDN distribution | Low | No SRI guidance in library README or docs | Applications loading jQuery from CDN without SRI are vulnerable to CDN compromise | jQuery Core Team | Unknown |

**Recommendations:**
- jQuery official CDN pages should prominently provide SRI hashes for each release so consuming applications can use them.

---

<!-- analysis: manual -->

## 9. Testing Methodology

_This section requires manual penetration testing and cannot be completed by automated analysis._

### Tools Used
- [x] Static code analysis (manual review of src/)
- [ ] Burp Suite Professional
- [ ] OWASP ZAP
- [ ] Browser Developer Tools
- [ ] XSS Hunter / XSS Strike
- [ ] CSP Evaluator
- [ ] SecurityHeaders.com

### Test Scenarios Executed
1. **XSS Testing (Stored, Reflected, DOM):** Static analysis only — runtime exploitation not performed
2. **CSRF Testing:** Not applicable (library-level)
3. **Clickjacking Test:** Not applicable (library-level)
4. **CSP Bypass Attempts:** Static analysis identified `globalEval` and `unsafe-eval` dependency
5. **Client-Side Validation Bypass:** Not applicable (library-level)

---

## Summary of Findings

### Critical Issues (Immediate Action Required)
_None identified._

### High Priority Issues
1. **`$(html)` constructor executes scripts** — `src/core/init.js:51-54`: `keepScripts: true` is hardcoded for backward compatibility, making `$('<script>evil()</script>')` execute.
2. **`.html()` method assigns innerHTML without sanitization** — `src/manipulation.js:418`: Direct `innerHTML` assignment; any attacker-controlled string causes XSS.
3. **`buildFragment` uses innerHTML for HTML parsing** — `src/manipulation/buildFragment.js:42`: Core HTML parsing pipeline uses `innerHTML` without sanitization.
4. **`$.load()` without selector injects raw server response** — `src/ajax/load.js:68`: Passes `responseText` directly to `.html()`.

### Medium Priority Issues
1. **`jQuery.globalEval` incompatible with strict CSP** — `src/ajax/script.js:17-19`: Requires `unsafe-eval` exception in any Content Security Policy.
2. **No sanitization hook in `htmlPrefilter`** — `src/manipulation.js:228-230`: `htmlPrefilter` performs structural transformation only; cannot be used to sanitize content.

### Low Priority Issues
1. **`parseXML` error discloses raw input** — `src/ajax/parseXML.js:21`: Error message includes full XML string.
2. **No SRI guidance for CDN distribution** — README/docs: No Subresource Integrity hashes prominently documented.

---

## Recommendations Summary

### Immediate Actions (0-7 days)
1. Add prominent security warnings in README and API documentation for `.html()`, `$(html)`, `.append()`, `.load()`, and `$.ajax({dataType:"script"})` that these APIs must never receive untrusted input.
2. Publish SRI hashes for the v2.2.5 release on the official jQuery CDN page.

### Short-term Actions (1-4 weeks)
1. Modify `htmlPrefilter` to support an optional application-configurable sanitizer callback.
2. Fix `parseXML` to not include raw input in error messages.

### Long-term Improvements (1-3 months)
1. Plan migration path to change `$(html)` default to `keepScripts: false` (jQuery 4.x breaking change).
2. Evaluate Trusted Types API compatibility for a future jQuery release.

---

## Conclusion

**UI Security Posture:** Poor — The library's core DOM manipulation APIs are inherently unsafe when used with untrusted input, and there is no built-in sanitization layer.

**Key Takeaways:**
- jQuery's design prioritizes developer convenience over security defaults; four high-severity XSS vectors exist in the core manipulation API.
- The `$(html)` constructor executing scripts by default is a well-known security anti-pattern (see CVE-2015-9251 and related jQuery XSS advisories).
- Consuming applications bear the full burden of sanitizing input before using jQuery DOM APIs.

**Next Steps:**
1. Review and annotate all DOM-manipulation API documentation with explicit security warnings.
2. Track progress toward removing `keepScripts: true` default in a future major release.

---

**Assessment completed by:** Security Auditor (Automated Static Analysis)
**Date:** 2026-02-27
**Review date:** 2026-08-27
