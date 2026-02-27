---
genre: hosting
category: compliance
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/cloudformation/**"
  keywords:
    - aws_config
    - compliance
    - cis
    - benchmark
    - scp
    - organization
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Compliance Audit Template

**Purpose**: Assess AWS compliance with CIS Benchmarks, regulatory standards (PCI DSS, HIPAA), and AWS Well-Architected Framework.

---

## Agentic Prompt Examples

### CIS Benchmark Audit
```
@terminal Run automated CIS AWS Foundations Benchmark audit: check all 58 controls 
across IAM, Storage, Logging, Monitoring, and Networking. Generate compliance score 
by section, identify non-compliant controls with severity ratings, and produce 
remediation roadmap prioritized by risk.
```

---

<!-- analysis: iac -->
## CIS AWS Foundations Benchmark v1.5 Controls

### Section 1: IAM (23 controls)
- 1.4: No root account access keys (CRITICAL)
- 1.5: Root account MFA enabled (CRITICAL)
- 1.8-1.11: Password policy requirements (HIGH)
- 1.12: Credentials unused >90 days disabled (HIGH)
- 1.14: Access keys rotated <=90 days (HIGH)

### Section 2: Storage (5 controls)
- 2.1.1: S3 bucket encryption enabled (HIGH)
- 2.1.5: S3 public access block (CRITICAL)
- 2.2.1: EBS encryption by default (HIGH)

### Section 3: Logging (11 controls)
- 3.1: CloudTrail enabled all regions (CRITICAL)
- 3.2: Log file validation enabled (HIGH)
- 3.4: CloudTrail integrated with CloudWatch (HIGH)
- 3.5: AWS Config enabled (HIGH)
- 3.7: CloudTrail logs encrypted with KMS (MEDIUM)

### Section 4: Monitoring (15 controls)
- 4.1-4.15: CloudWatch alarms for security events (MEDIUM)

### Section 5: Networking (4 controls)
- 5.1: Default VPCs not in use (MEDIUM)
- 5.2: No SSH (22) from 0.0.0.0/0 (CRITICAL)
- 5.3: No RDP (3389) from 0.0.0.0/0 (CRITICAL)

---

## Compliance Score Report Template

**Assessment Date**: [Date]  
**Framework**: CIS AWS Foundations Benchmark v1.5  
**Overall Score**: XX%

| Section | Controls | Passed | Failed | Score |
|---------|----------|--------|--------|-------|
| IAM | 23 | X | X | XX% |
| Storage | 5 | X | X | XX% |
| Logging | 11 | X | X | XX% |
| Monitoring | 15 | X | X | XX% |
| Networking | 4 | X | X | XX% |

---

## Regulatory Compliance

### PCI DSS 4.0
- Requirement 1: Network Security → CIS 5.x
- Requirement 3: Data Protection → CIS 2.x
- Requirement 8: Access Control → CIS 1.x
- Requirement 10: Logging → CIS 3.x

### HIPAA
- Access Control → IAM, MFA
- Audit Controls → CloudTrail
- Encryption → S3, EBS, RDS encryption
- Transmission Security → TLS/SSL

### SOC 2
- CC6.1: Logical Access → IAM
- CC6.6: Encryption → KMS
- CC7.2: Monitoring → CloudWatch, GuardDuty
- CC8.1: Change Management → Config

---

## Automated Compliance Tools

### AWS Security Hub
```bash
# Enable CIS Benchmark standard
aws securityhub batch-enable-standards \
  --standards-subscription-requests '[{"StandardsArn":"arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.4.0"}]'
```

### Prowler (Open Source)
```bash
pip install prowler
prowler aws --output-formats json,html,csv
```

---

**Next Steps**: Remediate non-compliant controls by priority.
