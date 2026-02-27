---
genre: security
category: crypto-usage
analysis-type: static
relevance:
  file-patterns:
    - "**/crypto/**"
    - "**/encryption/**"
    - "**/security/**"
  keywords:
    - "encrypt"
    - "decrypt"
    - "hash"
    - "cipher"
    - "aes"
    - "rsa"
    - "hmac"
    - "tls"
    - "ssl"
    - "certificate"
    - "pbkdf2"
  config-keys:
    - "crypto-js"
    - "bcrypt"
    - "node-forge"
    - "tweetnacl"
    - "libsodium"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Cryptography Security Assessment

**Assessment Date:** _[YYYY-MM-DD]_
**Auditor:** _[Your name]_
**Application:** _[Application name and version]_
**Status:** _[Draft / In Progress / Complete]_

---

<!-- analysis: static -->

## Executive Summary

**Overall Cryptography Security Rating:** [ ] Excellent [ ] Good [ ] Fair [ ] Poor [ ] Critical

**Key Findings:**
- Total Vulnerabilities: _[Number]_
- Critical: _[Number]_ | High: _[Number]_ | Medium: _[Number]_ | Low: _[Number]_

**Most Critical Issue:** _[Brief description]_

---

## Scope

### Components Assessed
- [ ] Encryption at rest
- [ ] Encryption in transit
- [ ] Key management and storage
- [ ] Hashing algorithms
- [ ] Random number generation
- [ ] Certificate management
- [ ] Cryptographic protocols
- [ ] Digital signatures

### Out of Scope
_[List what was not assessed]_

---

## 1. Encryption at Rest

### 1.1 Data Encryption

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Sensitive data is encrypted at rest
- [ ] Strong encryption algorithms used (AES-256, ChaCha20)
- [ ] Encryption is transparent to application
- [ ] Full disk encryption enabled on servers
- [ ] Database encryption (TDE) is enabled
- [ ] Backup encryption is enforced

**Issues Found:**

| Data Type | Location | Severity | Issue | Impact |
|-----------|----------|----------|-------|--------|
| | | | | |

**Encryption Configuration:**
```
Algorithm: [AES-256-GCM, etc.]
Key Size: [256 bits, etc.]
Mode: [GCM, CBC, etc.]
Storage: [Database, Filesystem, S3]
```

**Recommendations:**
- _[Specific recommendation]_

### 1.2 File System Encryption

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Uploaded files are encrypted
- [ ] Temporary files are encrypted or securely deleted
- [ ] Log files with sensitive data are encrypted
- [ ] Configuration files with secrets are encrypted
- [ ] Archive files are encrypted
- [ ] Encryption keys are not stored with encrypted data

**Issues Found:**

| File Type | Severity | Issue | Impact |
|-----------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 2. Encryption in Transit

### 2.1 TLS/SSL Configuration

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] TLS 1.2 or higher enforced (TLS 1.3 preferred)
- [ ] Strong cipher suites only (no RC4, 3DES, MD5)
- [ ] Perfect Forward Secrecy (PFS) enabled
- [ ] HTTP Strict Transport Security (HSTS) header present
- [ ] Certificate is valid and not self-signed (production)
- [ ] Certificate chain is complete

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**TLS Configuration:**
```
Protocol Versions: [TLS 1.2, TLS 1.3]
Cipher Suites: [List]
HSTS: [Enabled/Disabled]
Max-Age: [Seconds]
```

**Test Results:**
```
Tool: testssl.sh / SSL Labs
Grade: [A+, A, B, etc.]
Issues: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 2.2 Internal Communications

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Internal service-to-service communication encrypted
- [ ] Database connections use TLS
- [ ] Message queue connections encrypted
- [ ] Cache connections encrypted (Redis TLS)
- [ ] Admin interfaces use TLS
- [ ] No plaintext credentials transmitted

**Issues Found:**

| Connection Type | Severity | Issue | Impact |
|-----------------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 3. Key Management

### 3.1 Key Generation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Keys generated using cryptographically secure methods
- [ ] Key length is sufficient (256 bits for symmetric, 2048+ for RSA)
- [ ] Keys are unique per encryption operation (no key reuse)
- [ ] Key generation uses secure randomness
- [ ] Key derivation functions properly configured (PBKDF2, Argon2)
- [ ] Master keys are protected

**Issues Found:**

| Key Type | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Key Configuration:**
```
Generation Method: [Method]
Key Length: [Bits]
KDF: [PBKDF2, Argon2, etc.]
Iterations: [Number]
```

**Recommendations:**
- _[Specific recommendation]_

### 3.2 Key Storage

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Keys stored in secure key management system (KMS, Vault)
- [ ] Keys never stored in source code
- [ ] Keys not stored in environment variables (production)
- [ ] Keys encrypted at rest
- [ ] Access to keys is audited
- [ ] Keys separated from encrypted data

**Issues Found:**

| Location | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Key Storage:**
```
KMS: [AWS KMS, HashiCorp Vault, etc.]
Access Control: [IAM, RBAC]
Encryption: [Yes/No]
Audit Logging: [Enabled/Disabled]
```

**Recommendations:**
- _[Specific recommendation]_

### 3.3 Key Rotation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Key rotation policy exists and is enforced
- [ ] Keys rotated regularly (90 days recommended)
- [ ] Old keys retained for decryption of existing data
- [ ] Key rotation is automated
- [ ] Emergency key rotation procedure exists
- [ ] Key rotation is logged and auditable

**Issues Found:**

| Key Type | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Rotation Schedule:**
```
Frequency: [Days]
Last Rotation: [Date]
Next Rotation: [Date]
Automated: [Yes/No]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 4. Hashing Algorithms

### 4.1 Password Hashing

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Modern hashing algorithm used (bcrypt, Argon2, scrypt)
- [ ] Work factor/cost is appropriate
- [ ] Salt is unique per password
- [ ] No deprecated algorithms (MD5, SHA1 for passwords)
- [ ] Password hashes are not reversible
- [ ] Hash verification uses constant-time comparison

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Hashing Configuration:**
```
Algorithm: [bcrypt, Argon2id, scrypt]
Cost Factor: [12 for bcrypt, etc.]
Salt: [Unique per password]
```

**Recommendations:**
- _[Specific recommendation]_

### 4.2 Data Integrity Hashing

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Secure hash functions for data integrity (SHA-256, SHA-3)
- [ ] HMAC used for message authentication
- [ ] Hash algorithms appropriate for use case
- [ ] No hash collisions exploitable
- [ ] File integrity checking uses strong hashes
- [ ] Digital signatures use secure algorithms

**Issues Found:**

| Use Case | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 5. Random Number Generation

### 5.1 CSPRNG Usage

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Cryptographically secure random number generator used
- [ ] No predictable random values for security purposes
- [ ] Proper CSPRNG for tokens, keys, IVs
- [ ] Random values have sufficient entropy
- [ ] No seeded random generators for security
- [ ] CSPRNG properly initialized

**Issues Found:**

| Usage | Severity | Issue | Impact |
|-------|----------|-------|--------|
| | | | |

**CSPRNG Implementation:**
```
Library: [/dev/urandom, crypto.randomBytes, etc.]
Entropy Source: [OS, Hardware]
Use Cases: [Session IDs, Tokens, Keys]
```

**Test Results:**
```
Test: Predictability of session tokens
Sample Size: [Number]
Entropy: [Bits]
Predictable: [Yes/No]
```

**Recommendations:**
- _[Specific recommendation]_

### 5.2 Security Token Generation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Session tokens use CSPRNG
- [ ] Password reset tokens use CSPRNG
- [ ] API keys use CSPRNG
- [ ] CSRF tokens use CSPRNG
- [ ] Tokens have sufficient length (128+ bits)
- [ ] Tokens are not predictable or sequential

**Issues Found:**

| Token Type | Severity | Issue | Impact |
|------------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 6. Certificate Management

### 6.1 Certificate Lifecycle

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Certificates from trusted CA (not self-signed in prod)
- [ ] Certificate expiration monitoring in place
- [ ] Certificates renewed before expiration
- [ ] Certificate revocation is possible
- [ ] Certificate pinning implemented (where appropriate)
- [ ] Wildcard certificates usage is minimized

**Issues Found:**

| Certificate | Expiry Date | Severity | Issue | Impact |
|-------------|-------------|----------|-------|--------|
| | | | | |

**Certificate Inventory:**
```
Domain: [Domain]
Issuer: [Let's Encrypt, DigiCert, etc.]
Valid Until: [Date]
Key Size: [2048/4096 bits]
```

**Recommendations:**
- _[Specific recommendation]_

### 6.2 Certificate Validation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Certificate hostname validation enforced
- [ ] Certificate chain validation enforced
- [ ] Revocation checking enabled (OCSP, CRL)
- [ ] Certificate trust store is up-to-date
- [ ] Self-signed certificates rejected (production)
- [ ] Certificate errors are not ignored

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 7. Cryptographic Protocols

### 7.1 Protocol Implementation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Standard cryptographic libraries used (not custom crypto)
- [ ] No deprecated protocols (SSLv3, TLS 1.0, TLS 1.1)
- [ ] Proper padding schemes used (OAEP for RSA)
- [ ] Authenticated encryption used (GCM, CCM)
- [ ] IV/nonce handled correctly (random, not reused)
- [ ] No ECB mode for block ciphers

**Issues Found:**

| Protocol/Mode | Severity | Issue | Impact |
|---------------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 7.2 Implementation Vulnerabilities

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Timing attacks mitigated (constant-time operations)
- [ ] Padding oracle attacks prevented
- [ ] Side-channel attacks considered
- [ ] No cryptographic weaknesses (BEAST, CRIME, etc.)
- [ ] Library versions are current and patched
- [ ] Cryptographic code has been reviewed

**Issues Found:**

| Vulnerability | Severity | Issue | Impact |
|---------------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

<!-- analysis: manual -->

## 8. Testing Methodology

### Tools Used
- [ ] testssl.sh / SSL Labs (TLS testing)
- [ ] Hashcat (hash strength testing)
- [ ] OpenSSL (certificate analysis)
- [ ] Burp Suite (crypto analysis)
- [ ] Custom entropy testing scripts

### Test Scenarios Executed
1. **TLS Configuration:** _[Results]_
2. **Weak Cipher Detection:** _[Results]_
3. **Hash Algorithm Strength:** _[Results]_
4. **Random Number Quality:** _[Results]_
5. **Certificate Validation:** _[Results]_

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

**Cryptography Security Posture:** _[Overall assessment]_

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
