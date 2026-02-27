---
genre: security
category: ai
analysis-type: static
relevance:
  file-patterns:
    - "**/ai/**"
    - "**/ml/**"
    - "**/models/**"
    - "**/llm/**"
    - "**/prompts/**"
  keywords:
    - "openai"
    - "llm"
    - "model"
    - "prompt"
    - "embedding"
    - "inference"
    - "training"
    - "tensorflow"
    - "pytorch"
    - "langchain"
  config-keys:
    - "openai"
    - "langchain"
    - "@langchain/core"
    - "transformers"
    - "tensorflow"
    - "torch"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# AI/ML Security Assessment

**Assessment Date:** _[YYYY-MM-DD]_
**Auditor:** _[Your name]_
**Application:** _[Application name and version]_
**Status:** _[Draft / In Progress / Complete]_

---

<!-- analysis: static -->

## Executive Summary

**Overall AI/ML Security Rating:** [ ] Excellent [ ] Good [ ] Fair [ ] Poor [ ] Critical

**Key Findings:**
- Total Vulnerabilities: _[Number]_
- Critical: _[Number]_ | High: _[Number]_ | Medium: _[Number]_ | Low: _[Number]_

**Most Critical Issue:** _[Brief description]_

---

## Scope

### Components Assessed
- [ ] ML model security and integrity
- [ ] Training data security and governance
- [ ] Model deployment and serving infrastructure
- [ ] Data poisoning attack surface
- [ ] Adversarial attack resilience
- [ ] Model bias and fairness
- [ ] ML pipeline security
- [ ] Model versioning and governance

### Out of Scope
_[List what was not assessed]_

---

## 1. Model Security & Integrity

### 1.1 Model Protection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Models are encrypted at rest
- [ ] Model files have integrity checks (checksums, signatures)
- [ ] Model theft/extraction attacks are mitigated
- [ ] Model inversion attacks are prevented
- [ ] Model access is restricted to authorized services
- [ ] Model weights cannot be directly downloaded by users

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 1.2 Model Versioning & Provenance

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Model versions are tracked and documented
- [ ] Model lineage (training data, parameters) is recorded
- [ ] Model changes require approval and review
- [ ] Rollback procedures exist for problematic models
- [ ] Model metadata includes security properties
- [ ] Training provenance is auditable

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 2. Data Poisoning & Training Security

### 2.1 Training Data Integrity

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Training data sources are validated and trusted
- [ ] Data poisoning attacks are detected/prevented
- [ ] Input validation on training data exists
- [ ] Malicious data injection is monitored
- [ ] Training data is sanitized before use
- [ ] Data provenance is tracked

**Issues Found:**

| Data Source | Severity | Issue | Impact |
|-------------|----------|-------|--------|
| | | | |

**Test Results:**
```
Test: Inject malicious training samples
Method: [Method]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 2.2 Training Pipeline Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Training infrastructure is isolated and secured
- [ ] Training jobs have access controls
- [ ] Training logs don't contain sensitive data
- [ ] Training artifacts are stored securely
- [ ] Training code is reviewed for vulnerabilities
- [ ] Dependencies in training environment are secured

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 3. Adversarial Attack Resilience

### 3.1 Input Validation & Sanitization

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Input data is validated before inference
- [ ] Adversarial examples are detected
- [ ] Input bounds and constraints are enforced
- [ ] Unusual inputs trigger alerts
- [ ] Input preprocessing is consistent
- [ ] Evasion attacks are monitored

**Issues Found:**

| Attack Type | Severity | Issue | Test Case |
|-------------|----------|-------|-----------|
| | | | |

**Adversarial Test Results:**
```
Test: FGSM attack on classifier
Success Rate: [Percentage]
Detection Rate: [Percentage]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 3.2 Model Robustness

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Model has been tested against adversarial examples
- [ ] Adversarial training is used
- [ ] Model confidence thresholds are set appropriately
- [ ] Ensemble methods improve robustness
- [ ] Out-of-distribution detection is implemented
- [ ] Model degrades gracefully under attack

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 3.3 Prompt Injection (LLMs)

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Prompt injection attacks are prevented
- [ ] System prompts are protected from user manipulation
- [ ] Context separation between system and user inputs
- [ ] Output filtering prevents leaked instructions
- [ ] Role-based prompt restrictions are enforced
- [ ] Jailbreak attempts are detected and blocked

**Issues Found:**

| Attack Vector | Severity | Issue | Impact |
|---------------|----------|-------|--------|
| | | | |

**Test Results:**
```
Test: Attempt to override system instructions
Prompt: [Malicious prompt]
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 4. Model Bias & Fairness

### 4.1 Bias Detection & Mitigation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Bias metrics are measured and tracked
- [ ] Protected attributes are identified
- [ ] Disparate impact is assessed
- [ ] Training data is balanced across groups
- [ ] Bias mitigation techniques are applied
- [ ] Regular bias audits are conducted

**Issues Found:**

| Protected Class | Severity | Bias Metric | Impact |
|-----------------|----------|-------------|--------|
| | | | |

**Fairness Metrics:**
```
Demographic Parity: [Score]
Equal Opportunity: [Score]
Disparate Impact: [Score]
```

**Recommendations:**
- _[Specific recommendation]_

### 4.2 Fairness Governance

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Fairness requirements are documented
- [ ] Model cards include fairness assessments
- [ ] Bias findings are reported to stakeholders
- [ ] Ethical review process exists
- [ ] Fairness testing is part of CI/CD
- [ ] Incident response for bias issues exists

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 5. ML Pipeline Security

### 5.1 MLOps Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] ML pipelines have access controls
- [ ] Pipeline artifacts are signed and verified
- [ ] Secrets in pipelines are managed securely
- [ ] Pipeline logs are monitored for anomalies
- [ ] CI/CD for ML includes security checks
- [ ] Model deployment requires approval

**Issues Found:**

| Pipeline Stage | Severity | Issue | Impact |
|----------------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 5.2 Model Serving Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Inference APIs have authentication/authorization
- [ ] Rate limiting prevents abuse
- [ ] Model endpoints are not publicly exposed
- [ ] Inference requests are logged and monitored
- [ ] Model serving infrastructure is hardened
- [ ] Rollback capabilities exist for serving

**Issues Found:**

| Endpoint | Severity | Issue | Impact |
|----------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 6. Model Governance & Compliance

### 6.1 Model Documentation

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Model cards document intended use and limitations
- [ ] Training data characteristics are documented
- [ ] Model performance metrics are tracked
- [ ] Known vulnerabilities are documented
- [ ] Model updates are communicated
- [ ] Ethical considerations are documented

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 6.2 Privacy & Data Protection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Training data includes only necessary information
- [ ] PII is removed or anonymized from training data
- [ ] Model outputs don't leak training data
- [ ] Membership inference attacks are mitigated
- [ ] GDPR/privacy compliance is maintained
- [ ] Data retention policies are enforced

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Test Results:**
```
Test: Membership inference attack
Success Rate: [Percentage]
Data Leaked: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

---

<!-- analysis: manual -->

## 7. Testing Methodology

### Tools Used
- [ ] Adversarial Robustness Toolbox (ART)
- [ ] Foolbox (adversarial attacks)
- [ ] AI Fairness 360 (bias detection)
- [ ] TensorFlow Privacy (privacy testing)
- [ ] CleverHans (adversarial examples)
- [ ] Custom security scripts

### Test Scenarios Executed
1. **Adversarial Examples:** _[Results]_
2. **Data Poisoning:** _[Results]_
3. **Model Extraction:** _[Results]_
4. **Bias Testing:** _[Results]_
5. **Prompt Injection (if LLM):** _[Results]_

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

**AI/ML Security Posture:** _[Overall assessment]_

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
