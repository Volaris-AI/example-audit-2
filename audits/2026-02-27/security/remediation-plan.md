---
genre: security
category: remediation-plan
analysis-type: static
relevance:
  file-patterns: []
  keywords: []
  config-keys: []
  always-include: true
audit-date: 2026-02-27
---

# Security Remediation Plan

**Project Name:** jQuery JavaScript Library v2.2.5-pre
**Plan Date:** 2026-02-27
**Plan Owner:** jQuery Core Team
**Status:** Draft

---

<!-- analysis: static -->

## Overview

This remediation plan outlines the actions required to address 12 security vulnerabilities identified during the static security audit of jQuery v2.2.5-pre. Findings are prioritized by severity and business impact. jQuery is a client-side JavaScript library — findings are scoped to the library source code, build toolchain, and documentation.

**Important context:** The jQuery 2.x release line reached end-of-life in 2021. Consuming applications should migrate to jQuery 3.x or 4.x. Remediations in this plan are applicable to any active maintenance of the codebase and are also guidance for backport considerations.

---

## Summary of Vulnerabilities

| ID | Vulnerability | Severity | Status | Owner | Target Date |
|----|---------------|----------|--------|-------|-------------|
| VULN-2026-001 | DOM XSS via `$(html)` constructor (keepScripts:true) | High | 🔴 Open | @jquery-core | 2026-03-06 |
| VULN-2026-002 | DOM XSS via `.html()` method (unsanitized innerHTML) | High | 🔴 Open | @jquery-core | 2026-03-06 |
| VULN-2026-003 | DOM XSS via `buildFragment` innerHTML | High | 🔴 Open | @jquery-core | 2026-03-06 |
| VULN-2026-004 | DOM XSS via `$.load()` raw response injection | High | 🔴 Open | @jquery-core | 2026-03-06 |
| VULN-2026-005 | Arbitrary script execution via JSONP | Medium | 🔴 Open | @jquery-core | 2026-03-27 |
| VULN-2026-006 | Cross-origin script injection (dataType:"script") | Medium | 🔴 Open | @jquery-core | 2026-03-27 |
| VULN-2026-007 | Remote script execution via `_evalUrl` | Medium | 🔴 Open | @jquery-core | 2026-03-27 |
| VULN-2026-008 | Predictable JSONP callback name | Low | 🔴 Open | @jquery-core | 2026-04-28 |
| VULN-2026-009 | Raw XML input in error message | Low | 🔴 Open | @jquery-core | 2026-03-13 |
| VULN-2026-010 | No `package-lock.json` | Low | 🔴 Open | @jquery-devops | 2026-03-06 |
| VULN-2026-011 | Severely outdated dev dependencies | Medium | 🔴 Open | @jquery-devops | 2026-03-27 |
| VULN-2026-012 | No automated dependency scanning in CI | Medium | 🔴 Open | @jquery-devops | 2026-03-06 |

---

## Phase 1: Critical/High Issues — Security Documentation & Immediate Mitigations (Target: 0-7 days)

> **Rationale:** The four high-severity XSS vulnerabilities (VULN-2026-001 through 004) are architectural — fully fixing them requires breaking API changes or a new major version. The immediate actions are (a) publish authoritative security advisories, (b) add API documentation warnings, and (c) apply the one quick code fix available (`$.load()` sanitization).

---

### VULN-2026-001: DOM XSS via `$(html)` Constructor

**Severity:** High
**Owner:** @jquery-core
**Target Completion:** 2026-03-06
**Status:** 🔴 Not Started

**Description:**
`jQuery.fn.init()` calls `jQuery.parseHTML(match[1], ..., true)` — the `true` argument enables script execution from HTML strings passed to the `$()` constructor, enabling DOM XSS.

**Remediation Steps:**
1. **Immediate (documentation):** Publish a security advisory at https://jquery.com/upgrade-guide/4.0/ and in the GitHub repository's `SECURITY.md` that explicitly names this pattern as a XSS vector.
2. **Immediate (documentation):** Add an `@security` JSDoc annotation to `jQuery.fn.init()` warning against passing untrusted strings.
3. **Short-term (code):** Add a sanitizer hook in `htmlPrefilter` so applications can opt in to DOMPurify sanitization:
   ```javascript
   // src/manipulation.js
   jQuery.htmlSanitizer = null; // Consumers can set: jQuery.htmlSanitizer = DOMPurify.sanitize;
   jQuery.htmlPrefilter = function( html ) {
       if ( jQuery.htmlSanitizer ) {
           html = jQuery.htmlSanitizer( html );
       }
       return html.replace( rxhtmlTag, "<$1></$2>" );
   };
   ```
4. **Long-term (breaking change — jQuery 4.x):** Change `keepScripts` default to `false` in `jQuery.fn.init()`.

**Acceptance Criteria:**
- [x] Security advisory published (immediate)
- [ ] `SECURITY.md` updated with XSS guidance
- [ ] `htmlPrefilter` sanitizer hook implemented
- [ ] Unit tests added for sanitizer hook
- [ ] jQuery 4.x migration guide updated

**Dependencies:** Requires jQuery 4.x roadmap decision for breaking change.

**Estimated Effort:**
- Documentation: 2 hours
- Sanitizer hook: 4 hours + testing
- Breaking change (4.x): 1 developer-day

---

### VULN-2026-002: DOM XSS via `.html()` Method

**Severity:** High
**Owner:** @jquery-core
**Target Completion:** 2026-03-06
**Status:** 🔴 Not Started

**Description:**
`jQuery.fn.html()` sets `elem.innerHTML = value` without sanitization.

**Remediation Steps:**
1. **Immediate:** Add security warning to `.html()` API documentation at api.jquery.com.
2. **Short-term:** Apply `jQuery.htmlPrefilter` (with optional sanitizer hook from VULN-2026-001 fix) to the `.html()` setter path.
3. **Documentation:** Add prominent "Security" section to `.html()` docs with DOMPurify example.

**Acceptance Criteria:**
- [ ] API docs updated with security warning
- [ ] Sanitizer hook applied to `.html()` setter
- [ ] Regression tests pass
- [ ] Security example added to docs

**Estimated Effort:** 2 hours (docs) + 2 hours (code) + 2 hours (tests) = 6 hours

---

### VULN-2026-003: DOM XSS via `buildFragment` innerHTML

**Severity:** High
**Owner:** @jquery-core
**Target Completion:** 2026-03-06
**Status:** 🔴 Not Started

**Description:**
`buildFragment()` uses `tmp.innerHTML = wrap[1] + jQuery.htmlPrefilter(elem) + wrap[2]`. The sanitizer hook fix from VULN-2026-001 will also address this — `htmlPrefilter` is the centralized fix point.

**Remediation Steps:**
1. Same sanitizer hook in `htmlPrefilter` as VULN-2026-001 (shared fix).
2. Document that `.append()`, `.prepend()`, `.after()`, `.before()`, `.replaceWith()` all go through `buildFragment` and share the same XSS risk.

**Acceptance Criteria:**
- [ ] `htmlPrefilter` sanitizer hook applied (shared with VULN-2026-001 fix)
- [ ] Docs for `.append()`, `.prepend()`, etc. include security warnings

**Estimated Effort:** Covered by VULN-2026-001 fix. Documentation: 1 hour.

---

### VULN-2026-004: DOM XSS via `$.load()` Raw Response Injection

**Severity:** High
**Owner:** @jquery-core
**Target Completion:** 2026-03-06
**Status:** 🔴 Not Started

**Description:**
`$.fn.load()` passes raw `responseText` to `.html()` in the no-selector path.

**Remediation Steps:**
1. **Code fix (immediate):** Change `self.html(responseText)` to `self.html(jQuery.parseHTML(responseText, null, false))`:
   ```javascript
   // src/ajax/load.js:61-68 — BEFORE
   self.html( selector ?
       jQuery( "<div>" ).append( jQuery.parseHTML( responseText ) ).find( selector ) :
       responseText );

   // src/ajax/load.js — AFTER
   self.html( selector ?
       jQuery( "<div>" ).append( jQuery.parseHTML( responseText ) ).find( selector ) :
       jQuery( "<div>" ).append( jQuery.parseHTML( responseText, null, false ) ) );
   ```
2. **Note:** This change filters scripts from responses in the no-selector path, which is a behavioral change but a necessary security improvement.
3. **Testing:** Verify that HTML content (non-script) from `$.load()` still renders correctly.
4. **Documentation:** Update `$.load()` docs to explicitly warn about HTTP security and server trust requirements.

**Acceptance Criteria:**
- [ ] Code change implemented in `src/ajax/load.js`
- [ ] Unit tests confirm scripts in no-selector `$.load()` response are not executed
- [ ] Unit tests confirm non-script HTML still renders
- [ ] Documentation updated

**Dependencies:** None — this is a standalone fix.

**Estimated Effort:** 2 hours (code) + 2 hours (tests) + 1 hour (docs) = 5 hours

---

### VULN-2026-010: No `package-lock.json`

**Severity:** Low
**Owner:** @jquery-devops
**Target Completion:** 2026-03-06
**Status:** 🔴 Not Started

**Description:** No lock file committed; non-reproducible builds.

**Remediation Steps:**
1. Run `npm install` to generate `package-lock.json`.
2. Commit `package-lock.json` to the repository.
3. Update CI to use `npm ci` instead of `npm install`.

**Acceptance Criteria:**
- [ ] `package-lock.json` committed
- [ ] CI uses `npm ci`
- [ ] Build verified with locked dependencies

**Estimated Effort:** 30 minutes

---

### VULN-2026-012: No Automated Dependency Scanning in CI

**Severity:** Medium
**Owner:** @jquery-devops
**Target Completion:** 2026-03-06
**Status:** 🔴 Not Started

**Description:** No `npm audit` in CI pipeline.

**Remediation Steps:**
1. Add `npm audit --audit-level=high` step to `.travis.yml` after `npm ci`.
2. Enable GitHub Dependabot by adding `.github/dependabot.yml`:
   ```yaml
   version: 2
   updates:
     - package-ecosystem: "npm"
       directory: "/"
       schedule:
         interval: "weekly"
       open-pull-requests-limit: 5
   ```
3. Review initial audit output and triage findings.

**Acceptance Criteria:**
- [ ] `npm audit --audit-level=high` in `.travis.yml`
- [ ] `.github/dependabot.yml` created
- [ ] Initial audit report reviewed

**Estimated Effort:** 1 hour

---

## Phase 2: Medium Issues — Code & Process (Target: 1-4 weeks)

### VULN-2026-005: JSONP Security

**Severity:** Medium
**Owner:** @jquery-core
**Target Completion:** 2026-03-27
**Status:** 🔴 Not Started

**Description:** JSONP executes arbitrary third-party scripts with no validation.

**Remediation Steps:**
1. Add a `jsonpAllowedOrigins` configuration option to restrict JSONP to trusted origins.
2. Add deprecation notice to `$.ajax({dataType:"jsonp"})` documenting security risks.
3. Create upgrade guide section: "Replacing JSONP with CORS."
4. Long-term: Mark JSONP as deprecated in jQuery 3.x / remove in 4.x.

**Acceptance Criteria:**
- [ ] Deprecation notice in JSONP code comments
- [ ] API docs updated with CORS migration guide
- [ ] Optional origin allowlist implemented

**Estimated Effort:** 4 hours (code) + 2 hours (docs)

---

### VULN-2026-006: Cross-Origin Script Injection

**Severity:** Medium
**Owner:** @jquery-core
**Target Completion:** 2026-03-27
**Status:** 🔴 Not Started

**Description:** `<script>` tag injection for cross-domain requests lacks SRI support.

**Remediation Steps:**
1. Add an `integrity` option to `$.ajax()` for `dataType:"script"` requests.
2. When `integrity` is provided, set the `integrity` attribute on the injected `<script>` tag.
3. Document SRI usage in AJAX API docs.

**Acceptance Criteria:**
- [ ] `integrity` option supported for script transport
- [ ] `integrity` attribute applied to injected `<script>` elements
- [ ] Tests verify SRI enforcement

**Estimated Effort:** 4 hours (code) + 2 hours (tests) + 1 hour (docs)

---

### VULN-2026-007: Remote Script Execution via `_evalUrl`

**Severity:** Medium
**Owner:** @jquery-core
**Target Completion:** 2026-03-27
**Status:** 🔴 Not Started

**Description:** `_evalUrl()` fetches and executes remote scripts without URL validation.

**Remediation Steps:**
1. Add same-origin check before executing `_evalUrl`:
   ```javascript
   jQuery._evalUrl = function( url ) {
       // Only eval same-origin or explicitly allowlisted URLs
       var anchor = document.createElement( "a" );
       anchor.href = url;
       if ( anchor.origin !== window.location.origin ) {
           jQuery.error( "jQuery._evalUrl: cross-origin script evaluation blocked. " +
               "Use $.ajax({ dataType: 'script' }) with explicit crossDomain: true." );
           return;
       }
       return jQuery.ajax( { url: url, type: "GET", dataType: "script",
           async: false, global: false, "throws": true } );
   };
   ```
2. Document the security implications of `_evalUrl` as an internal API.
3. Consider deprecating `_evalUrl` in favor of module-based script loading.

**Acceptance Criteria:**
- [ ] Same-origin check added to `_evalUrl`
- [ ] Tests verify cross-origin URLs are blocked
- [ ] Internal API documented with security warning

**Estimated Effort:** 2 hours (code) + 2 hours (tests)

---

### VULN-2026-011: Outdated Dev Dependencies

**Severity:** Medium
**Owner:** @jquery-devops
**Target Completion:** 2026-03-27
**Status:** 🔴 Not Started

**Description:** jsdom 5.6.1, sinon 1.10.3, grunt 0.4.5 are severely outdated.

**Remediation Steps:**
1. Update `jsdom` from 5.6.1 to latest 25.x: `npm install --save-dev jsdom@25`.
2. Update `sinon` from 1.10.3 to latest 17.x: `npm install --save-dev sinon@17`.
3. Update `grunt` and all `grunt-*` plugins to current versions.
4. Update `qunitjs` from 1.17.1 to 2.x.
5. Run full test suite after each update and fix any breaking changes.

**Acceptance Criteria:**
- [ ] jsdom updated to 25.x
- [ ] sinon updated to 17.x
- [ ] grunt updated to 1.x
- [ ] All tests passing with updated dependencies
- [ ] `npm audit` shows no high/critical issues in dev deps

**Dependencies:** VULN-2026-010 (lock file) must be committed first.

**Estimated Effort:** 2-3 developer-days (dependency update and test fixes)

---

## Phase 3: Low Severity Issues (Target: 1-2 months)

### VULN-2026-008: Predictable JSONP Callback Name

**Severity:** Low
**Owner:** @jquery-core
**Target Completion:** 2026-04-28
**Status:** 🔴 Not Started

**Description:** JSONP nonce uses `Date.now()` counter, not CSPRNG.

**Remediation Steps:**
1. Update `src/ajax/var/nonce.js` to use `crypto.getRandomValues()` when available:
   ```javascript
   define( function() {
       if ( window.crypto && window.crypto.getRandomValues ) {
           var arr = new Uint32Array( 1 );
           window.crypto.getRandomValues( arr );
           return arr[ 0 ];
       }
       return jQuery.now();
   } );
   ```
2. Note: This is moot if JSONP is deprecated (VULN-2026-005 long-term fix).

**Acceptance Criteria:**
- [ ] CSPRNG used for nonce when Web Crypto API available
- [ ] Fallback to `Date.now()` for environments without Web Crypto

**Estimated Effort:** 1 hour

---

### VULN-2026-009: Raw XML in Error Message

**Severity:** Low
**Owner:** @jquery-core
**Target Completion:** 2026-03-13
**Status:** 🔴 Not Started

**Description:** `parseXML` includes full raw XML in thrown error message.

**Remediation Steps:**
1. Change `src/ajax/parseXML.js:21`:
   ```javascript
   // BEFORE:
   jQuery.error( "Invalid XML: " + data );

   // AFTER:
   jQuery.error( "Invalid XML: parsing failed" );
   ```
2. Add test case verifying the error message does not include input data.

**Acceptance Criteria:**
- [ ] Error message no longer includes raw input
- [ ] Unit test verifies sanitized error message

**Estimated Effort:** 30 minutes (code) + 30 minutes (tests)

---

## Phase 4: Process Improvements

### Long-term Security Enhancements

1. **Security Testing in CI/CD**
   - **Owner:** @jquery-devops
   - **Target:** 2026-06-27
   - **Actions:**
     - [x] Add `npm audit` to CI (Phase 1 — VULN-2026-012)
     - [ ] Integrate CodeQL or ESLint security plugin for static analysis
     - [ ] Configure pre-commit hook for secrets detection
     - [ ] Set up automated security regression tests

2. **Developer Security Documentation**
   - **Owner:** @jquery-core
   - **Target:** 2026-05-27
   - **Actions:**
     - [ ] Create `SECURITY.md` with responsible disclosure process
     - [ ] Write "Secure jQuery Usage" guide on jquery.com
     - [ ] Add DOMPurify integration example to official docs
     - [ ] Document CSP compatibility guidance

3. **Security Review Process**
   - **Owner:** @jquery-core
   - **Target:** 2026-06-27
   - **Actions:**
     - [ ] Add security checklist to PR template for DOM manipulation changes
     - [ ] Require security review for changes to `manipulation.js`, `ajax.js`, `core/init.js`
     - [ ] Establish quarterly security audit schedule
     - [ ] Create security-specific changelog entries

4. **Dependency Modernization**
   - **Owner:** @jquery-devops
   - **Target:** 2026-06-27
   - **Actions:**
     - [ ] Complete dev dependency updates from Phase 2 (VULN-2026-011)
     - [ ] Evaluate migrating from Grunt to a modern build tool (Rollup, esbuild)
     - [ ] Establish automated dependency update PR workflow (Dependabot)
     - [ ] Document dependency governance policy

---

## Resource Requirements

### Team Members Required
- **Core Developer (DOM/API):** 20 hours (Phases 1-2)
- **DevOps/Build Engineer:** 8 hours (Phases 1-2)
- **Documentation Writer:** 6 hours (across all phases)

### Tools & Services
- [ ] GitHub Dependabot (free for public repos)
- [ ] npm audit (built-in, no cost)
- [ ] CodeQL (free for public repos via GitHub Actions)

### Budget Impact
- **Phase 1 (Emergency docs + quick fixes):** ~8 developer-hours
- **Phase 2 (Code hardening + dep updates):** ~24 developer-hours
- **Phase 3 (Low severity):** ~4 developer-hours
- **Phase 4 (Process):** ~16 developer-hours
- **Total:** ~52 developer-hours

---

## Risk Management

### Temporary Mitigations

While fixes are in development, the following temporary mitigations are recommended for **consuming applications**:

1. **VULN-2026-001 through 004 (XSS):** Wrap jQuery with DOMPurify:
   ```javascript
   // Application-level mitigation
   var originalHtml = $.fn.html;
   $.fn.html = function(value) {
       if (typeof value === 'string') {
           value = DOMPurify.sanitize(value);
       }
       return originalHtml.call(this, value);
   };
   ```
2. **VULN-2026-005 (JSONP):** Replace all `$.ajax({dataType:"jsonp"})` with CORS-based requests.
3. **VULN-2026-011 (outdated deps):** Run `npm audit` manually until CI scanning is in place.

### Rollback Plan

For all code changes in Phase 1-2:
1. All fixes are in separate commits; individual commits can be reverted.
2. Tests must pass before merging; test regressions trigger immediate revert.
3. `git revert <commit-sha>` for any breaking change discovered post-merge.

---

## Communication Plan

### Stakeholder Updates
- **Frequency:** Weekly during Phases 1-2, bi-weekly thereafter
- **Format:** GitHub issue tracker + mailing list announcement
- **Audience:** jQuery Foundation, consuming application developers, security community

### Escalation Path
Issue owner → jQuery Core Team lead → jQuery Foundation board

---

## Verification & Validation

### Testing Strategy
- [x] Unit tests for each code fix
- [ ] Integration tests for AJAX pipeline changes
- [ ] Security regression test suite for XSS vectors
- [ ] Penetration testing for critical fixes (optional — library-level)
- [x] User acceptance testing — not applicable (library, not application)

### Verification Checklist
- [ ] Code reviewed by second jQuery core team member
- [ ] Automated tests pass in CI
- [ ] Manual security testing for XSS fixes
- [ ] Documentation updated
- [ ] Deployment to npm successful
- [ ] Post-release verification: no regression reports in 72 hours

---

## Timeline Overview

```
Week 1 (2026-03-06):   VULN-004 code fix, VULN-010 lock file, VULN-012 CI scanning
                        Security advisory publication
                        Documentation warnings for VULN-001, 002, 003
Week 2-3 (2026-03-13): VULN-009 error message fix
Week 3-4 (2026-03-27): VULN-005, 006, 007 code hardening
                        VULN-011 dependency updates
Month 2 (2026-04-28):  VULN-008 CSPRNG nonce
Month 3-4:             Process improvements (SECURITY.md, docs, CI/CD hardening)
```

### Key Milestones
- **2026-03-06:** All high-severity documentation mitigations complete; `$.load()` code fix deployed
- **2026-03-13:** `parseXML` error message fix deployed; `package-lock.json` committed
- **2026-03-27:** All medium-severity code fixes deployed; dev dependencies updated
- **2026-04-28:** All low-severity fixes deployed; CSPRNG nonce in place
- **2026-06-27:** Process improvements fully implemented

---

## Success Metrics

### Quantitative Metrics
- **Vulnerabilities Resolved:** 0 / 12 (Target: 100% within 90 days)
- **Mean Time to Remediation:**
  - High: < 14 days (documentation + code for `$.load()`)
  - Medium: < 30 days
  - Low: < 60 days
- **Regression Rate:** < 5%

### Qualitative Metrics
- Security posture improved from "Poor" (current) to "Fair/Good" after Phase 1-2
- API documentation includes comprehensive security warnings
- Consuming applications have clear migration guidance

---

## Follow-up Actions

### Post-Remediation Audit
- **Scheduled Date:** 2026-05-27 (90 days post-audit)
- **Scope:** Verify all fixes, check for new vulnerabilities in updated deps
- **Team:** Independent security review or same auditor

### Continuous Improvement
- [ ] Schedule quarterly security reviews
- [ ] Maintain security backlog in GitHub Issues with `security` label
- [ ] Track new CVEs via Dependabot and npm advisory feeds
- [ ] Update security documentation with each major jQuery release

---

## Sign-off

### Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Security Lead | _Pending_ | | |
| Engineering Manager | _Pending_ | | |
| Product Owner | _Pending_ | | |

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-27 | Security Auditor | Initial plan based on static analysis |

---

**Plan maintained by:** jQuery Core Team
**Last updated:** 2026-02-27
**Next review:** 2026-03-13
