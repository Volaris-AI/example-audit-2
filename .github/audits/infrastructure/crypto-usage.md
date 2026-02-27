---
genre: infrastructure
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
  config-keys:
    - "crypto-js"
    - "bcrypt"
    - "node-forge"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Cryptography Usage Infrastructure Audit

## System Information
- **System Name**: 
- **Audit Date**: 
- **Auditor**: 

<!-- analysis: static -->

## Maturity Level Assessment

| Level | Description | Algorithms | Key Management | Implementation |
|-------|-------------|------------|----------------|----------------|
| **5** | Modern, HSM-backed | Modern algos (AES-256-GCM, ChaCha20), quantum-ready | HSM, automated rotation, centralized | Crypto library, audited, no custom crypto |
| **4** | Strong cryptography | Current standards (AES-256, RSA-2048+) | Key vault, rotation policy | Vetted libraries, proper implementation |
| **3** | Adequate crypto | Standard algos (AES-128, RSA-2048) | Secure storage, manual rotation | Standard libraries |
| **2** | Weak or outdated | Weak algos (DES, MD5, SHA-1) | Hardcoded or insecure storage | Mixed implementation |
| **1** | No crypto or broken | None, plaintext, or broken (ECB mode) | No key management | Custom crypto, insecure |

### Current Maturity Score: [ ] / 5

## Assessment Areas

### 1. Encryption at Rest
- [ ] **Database encryption**: TDE / Column-level / Application-level
- [ ] **File storage encryption**: Yes / No
- [ ] **Backup encryption**: Yes / No
- [ ] **Algorithm**: AES-256-GCM / AES-256-CBC / Other: ______
- [ ] **Key rotation** policy and implementation

### 2. Encryption in Transit
- [ ] **TLS version**: TLS 1.3 / TLS 1.2 / TLS 1.1 / TLS 1.0 / SSL
- [ ] **Certificate management**: Let's Encrypt / Commercial CA / Self-signed
- [ ] **Certificate expiration** monitoring
- [ ] **Perfect Forward Secrecy** (PFS) enabled
- [ ] **HSTS** header configured
- [ ] **Internal traffic** encrypted (service-to-service)

### 3. Hashing & Signing
- [ ] **Password hashing**: Argon2 / bcrypt / PBKDF2 / scrypt / Other: ______
- [ ] **Message integrity**: HMAC-SHA256 / Digital signatures
- [ ] **JWT signing**: RS256 / ES256 / HS256
- [ ] **File integrity**: SHA-256 / SHA-512
- [ ] **Avoid weak hashes**: No MD5, SHA-1

### 4. Key Management
- [ ] **Key storage**: HSM / Cloud KMS / Vault / Env vars / Hardcoded
- [ ] **Key rotation** automated and regular
- [ ] **Key lifecycle** management (creation, rotation, revocation)
- [ ] **Separation of duties** for key access
- [ ] **Key backup** and recovery
- [ ] **Envelope encryption** for data keys

### 5. Random Number Generation
- [ ] **CSPRNG** used (cryptographically secure)
- [ ] **Avoid weak** RNG (Math.random, etc.)
- [ ] **Token generation** secure
- [ ] **Session ID** generation secure
- [ ] **IV/Nonce** generation proper

### 6. Cryptographic Implementation
- [ ] **Vetted libraries**: OpenSSL / BoringSSL / libsodium / NaCl / Other: ______
- [ ] **No custom crypto** implemented
- [ ] **Proper modes**: GCM / CBC with HMAC / Never ECB
- [ ] **Authenticated encryption** (AEAD)
- [ ] **Constant-time** comparisons for secrets

### 7. Certificate Management
- [ ] **Certificate inventory** maintained
- [ ] **Expiration monitoring** and alerting
- [ ] **Automated renewal** (ACME, Let's Encrypt)
- [ ] **Certificate pinning** (mobile apps, if applicable)
- [ ] **Revocation** checking (OCSP, CRL)

### 8. Compliance & Standards
- [ ] **FIPS 140-2** compliance (if required)
- [ ] **PCI DSS** crypto requirements (if applicable)
- [ ] **NIST guidelines** followed
- [ ] **Crypto agility** (easy to update algorithms)
- [ ] **Post-quantum** readiness planning

## Recommendations
**Level 1→2**: Remove weak algos, use standard libraries, secure key storage
**Level 2→3**: Implement TLS 1.2+, proper password hashing, key rotation
**Level 3→4**: Use key vault (AWS KMS, Azure Key Vault), TLS 1.3, automated cert renewal
**Level 4→5**: HSM, automated key rotation, quantum-ready algorithms

## Success Criteria
- All data encrypted at rest and in transit
- Modern algorithms (AES-256-GCM, TLS 1.3)
- Keys stored in vault/HSM
- Automated key rotation
- No custom cryptography

---
**Document Version**: 1.0
