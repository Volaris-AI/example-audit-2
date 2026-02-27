---
genre: security
category: mobile
analysis-type: static
relevance:
  file-patterns:
    - "**/ios/**"
    - "**/android/**"
    - "**/mobile/**"
    - "**/*.swift"
    - "**/*.kt"
    - "**/ReactNative/**"
  keywords:
    - "mobile"
    - "ios"
    - "android"
    - "swift"
    - "kotlin"
    - "react-native"
    - "flutter"
    - "cordova"
    - "capacitor"
  config-keys:
    - "react-native"
    - "expo"
    - "@capacitor/core"
    - "cordova"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Mobile Security Assessment

**Assessment Date:** _[YYYY-MM-DD]_
**Auditor:** _[Your name]_
**Application:** _[Application name and version]_
**Status:** _[Draft / In Progress / Complete]_

---

<!-- analysis: static -->

## Executive Summary

**Overall Mobile Security Rating:** [ ] Excellent [ ] Good [ ] Fair [ ] Poor [ ] Critical

**Key Findings:**
- Total Vulnerabilities: _[Number]_
- Critical: _[Number]_ | High: _[Number]_ | Medium: _[Number]_ | Low: _[Number]_

**Most Critical Issue:** _[Brief description]_

---

## Scope

### Components Assessed
- [ ] iOS application security
- [ ] Android application security
- [ ] Local data storage
- [ ] Certificate pinning and network security
- [ ] Deep links and URL schemes
- [ ] Permissions and entitlements
- [ ] Reverse engineering protection
- [ ] Third-party SDK security

### Platforms
- [ ] iOS (Version: _[Version]_)
- [ ] Android (Version: _[Version]_)

### Out of Scope
_[List what was not assessed]_

---

## 1. Data Storage Security

### 1.1 Local Storage

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Sensitive data encrypted in local storage
- [ ] iOS Keychain used for credentials/tokens
- [ ] Android Keystore used for sensitive data
- [ ] No sensitive data in NSUserDefaults/SharedPreferences
- [ ] SQLite databases encrypted
- [ ] File-based storage has appropriate permissions

**Issues Found:**

| Data Type | Storage Location | Severity | Issue | Impact |
|-----------|------------------|----------|-------|--------|
| | | | | |

**Test Results:**
```
Test: Extract local storage
Location: [Path]
Sensitive Data Found: [Yes/No]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 1.2 Cache & Temporary Files

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Sensitive data not cached inappropriately
- [ ] Screenshot prevention for sensitive screens
- [ ] Clipboard cleared for sensitive data
- [ ] Temporary files securely deleted
- [ ] No sensitive data in app logs
- [ ] Background task snapshots protected

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 1.3 Backup Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Sensitive data excluded from cloud backups (iOS)
- [ ] Android Auto Backup configured properly
- [ ] Keychain items marked as non-synchronizing
- [ ] Database encryption maintained in backups
- [ ] No sensitive data in iTunes/iCloud backups
- [ ] Backup files encrypted

**Issues Found:**

| Severity | Issue | Backup Type | Impact |
|----------|-------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 2. Network Communication Security

### 2.1 Certificate Pinning

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Certificate pinning implemented
- [ ] Pin includes backup certificates
- [ ] Certificate validation cannot be bypassed
- [ ] Pinning works on all API endpoints
- [ ] Certificate rotation process documented
- [ ] Pinning failures handled gracefully

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Test Results:**
```
Test: MITM with proxy
Tool: Burp Suite / Charles Proxy
Result: [Pass/Fail]
Pinning Bypassed: [Yes/No]
```

**Recommendations:**
- _[Specific recommendation]_

### 2.2 Network Security Configuration

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] HTTPS enforced for all connections
- [ ] No cleartext traffic allowed (NSAppTransportSecurity, cleartextTrafficPermitted)
- [ ] Strong TLS configuration
- [ ] No insecure protocol handlers (HTTP, FTP)
- [ ] WebView secure communication only
- [ ] API tokens/keys protected in transit

**Issues Found:**

| Severity | Issue | Configuration | Impact |
|----------|-------|---------------|--------|
| | | | |

**Configuration:**
```
iOS ATS: [Configured/Disabled]
Android Network Security Config: [Present/Absent]
Cleartext Allowed: [Yes/No]
TLS Version: [1.2, 1.3]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 3. Authentication & Session Management

### 3.1 Biometric Authentication

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Biometric authentication properly implemented
- [ ] Face ID/Touch ID/Fingerprint fallback secure
- [ ] Biometric templates not stored locally
- [ ] LAPolicy configured securely (iOS)
- [ ] BiometricPrompt used correctly (Android)
- [ ] Biometric failure handling secure

**Issues Found:**

| Platform | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 3.2 Session Handling

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Session tokens stored securely (Keychain/Keystore)
- [ ] Session timeout implemented
- [ ] Sessions invalidated on logout
- [ ] No persistent sessions without re-authentication
- [ ] Token refresh handled securely
- [ ] Multi-device session management

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 4. Deep Links & URL Schemes

### 4.1 Deep Link Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Deep links validated before processing
- [ ] Universal Links/App Links configured (not custom schemes)
- [ ] No sensitive actions via deep links
- [ ] Deep link parameters sanitized
- [ ] URL scheme hijacking prevented
- [ ] Intent filters properly configured (Android)

**Issues Found:**

| URL Scheme | Severity | Issue | Impact |
|------------|----------|-------|--------|
| | | | |

**Test Results:**
```
Test: Deep link injection
URL: [Malicious URL]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 4.2 WebView Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] WebView JavaScript interface secured
- [ ] File access disabled in WebView
- [ ] WebView content validated
- [ ] No XSS vulnerabilities in WebView content
- [ ] WebView SSL errors not ignored
- [ ] Universal Links prevent WebView phishing

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 5. Platform-Specific Security

### 5.1 iOS Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Code signing certificate valid
- [ ] Entitlements follow least privilege
- [ ] No jailbreak detection bypass
- [ ] Pasteboard security configured
- [ ] No sensitive data in app snapshots
- [ ] Secure Enclave used where appropriate

**Issues Found:**

| Severity | Issue | Configuration | Impact |
|----------|-------|---------------|--------|
| | | | |

**iOS Configuration:**
```
Code Signing: [Valid/Invalid]
Entitlements: [List]
Jailbreak Detection: [Yes/No]
App Transport Security: [Configured]
```

**Recommendations:**
- _[Specific recommendation]_

### 5.2 Android Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] App signing v2/v3 scheme used
- [ ] ProGuard/R8 obfuscation enabled
- [ ] Root detection implemented
- [ ] SafetyNet Attestation API used
- [ ] No exported components without protection
- [ ] Manifest permissions minimized

**Issues Found:**

| Severity | Issue | Configuration | Impact |
|----------|-------|---------------|--------|
| | | | |

**Android Configuration:**
```
Signing Scheme: [v1/v2/v3]
Obfuscation: [Enabled/Disabled]
Root Detection: [Yes/No]
Debuggable: [true/false]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 6. Permissions & Entitlements

### 6.1 Permission Requests

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Minimum necessary permissions requested
- [ ] Permission requests justified to user
- [ ] Runtime permission requests implemented
- [ ] Permissions checked before use
- [ ] Graceful degradation when permissions denied
- [ ] No dangerous permissions unnecessary

**Issues Found:**

| Permission | Platform | Severity | Justification | Impact |
|------------|----------|----------|---------------|--------|
| | | | | |

**Permissions Declared:**
- iOS: _[List]_
- Android: _[List]_

**Recommendations:**
- _[Specific recommendation]_

### 6.2 Sensitive Permissions

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Camera/microphone access justified
- [ ] Location access appropriate (fine vs coarse)
- [ ] Contacts/calendar access necessary
- [ ] Storage access minimized (Android)
- [ ] Background location justified
- [ ] Bluetooth/NFC permissions needed

**Issues Found:**

| Permission | Severity | Issue | Impact |
|------------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 7. Reverse Engineering Protection

### 7.1 Code Obfuscation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Code obfuscation enabled (ProGuard/R8 for Android)
- [ ] String encryption for sensitive strings
- [ ] Control flow obfuscation
- [ ] Debug symbols stripped
- [ ] Anti-tampering checks implemented
- [ ] Obfuscation mapping files secured

**Issues Found:**

| Platform | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Obfuscation Status:**
```
iOS: [BitCode, Strip Symbols]
Android: [ProGuard/R8 enabled]
String Encryption: [Yes/No]
```

**Recommendations:**
- _[Specific recommendation]_

### 7.2 Runtime Protection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Jailbreak/root detection
- [ ] Debugger detection
- [ ] Emulator detection
- [ ] Hooking framework detection (Frida, Xposed)
- [ ] Integrity checks (checksum verification)
- [ ] Response to tampering (app shutdown, alert)

**Issues Found:**

| Detection Type | Severity | Issue | Bypass Possible |
|----------------|----------|-------|-----------------|
| | | | |

**Test Results:**
```
Test: Run on jailbroken/rooted device
Result: [Detected/Not Detected]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 8. Third-Party SDKs & Libraries

### 8.1 SDK Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Third-party SDKs from trusted sources
- [ ] SDK vulnerability scanning performed
- [ ] SDK permissions reviewed
- [ ] SDKs kept up-to-date
- [ ] Minimal SDK usage (only necessary)
- [ ] SDK security policies reviewed

**Issues Found:**

| SDK | Version | Severity | Vulnerability | Impact |
|-----|---------|----------|---------------|--------|
| | | | | |

**SDK Inventory:**
```
Total SDKs: [Number]
Outdated: [Number]
Known Vulnerabilities: [Number]
```

**Recommendations:**
- _[Specific recommendation]_

### 8.2 SDK Data Collection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] SDK data collection disclosed to users
- [ ] Analytics SDKs configured for privacy
- [ ] Ad SDKs follow privacy policies
- [ ] Crash reporting doesn't include sensitive data
- [ ] SDK data sharing reviewed
- [ ] User consent obtained where required

**Issues Found:**

| SDK | Severity | Issue | Data Collected |
|-----|----------|-------|----------------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

<!-- analysis: manual -->

## 9. Testing Methodology

### Tools Used
- [ ] MobSF (Mobile Security Framework)
- [ ] Frida (dynamic instrumentation)
- [ ] objection (runtime mobile exploration)
- [ ] Burp Suite Mobile Assistant
- [ ] APKTool / Hopper (reverse engineering)
- [ ] Xcode / Android Studio debugger

### Test Scenarios Executed
1. **Local Storage Extraction:** _[Results]_
2. **SSL Pinning Bypass:** _[Results]_
3. **Deep Link Exploitation:** _[Results]_
4. **Root/Jailbreak Detection:** _[Results]_
5. **Runtime Tampering:** _[Results]_

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

**Mobile Security Posture:** _[Overall assessment]_

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
