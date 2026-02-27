---
genre: hosting
category: secrets-management
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/cloudformation/**"
  keywords:
    - aws_secretsmanager
    - aws_ssm_parameter
    - aws_kms
    - kms_key
    - secret
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Secrets Management Audit Template

**Purpose**: Audit AWS secrets management including Secrets Manager, Parameter Store, KMS, and credential hygiene.

---

## Agentic Prompt Examples

### Secrets Security Audit
```
@terminal Audit AWS secrets management: check Lambda functions for hardcoded 
credentials in environment variables, verify KMS key rotation is enabled, review 
Secrets Manager rotation configuration, scan Parameter Store for non-SecureString 
parameters, check EC2 user data for secrets, and identify IAM policies with 
overly broad secretsmanager:GetSecretValue permissions.
```

---

## AWS CLI Commands

### 1. Secrets Manager Audit

#### List All Secrets
```bash
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  aws secretsmanager list-secrets --region "$region" \
    --query 'SecretList[*].[Name,RotationEnabled,LastRotatedDate]' --output table
done
```

#### Check Rotation Status
```bash
# Find secrets without automatic rotation
aws secretsmanager list-secrets \
  --query 'SecretList[?RotationEnabled==`false`].[Name]' --output text
```

#### Check Encryption
```bash
# Verify secrets use KMS encryption
for secret in $(aws secretsmanager list-secrets --query 'SecretList[*].Name' --output text); do
  kms_key=$(aws secretsmanager describe-secret --secret-id "$secret" --query 'KmsKeyId' --output text)
  echo "Secret: $secret | KMS: $kms_key"
done
```

### 2. Parameter Store Audit

#### List Parameters
```bash
aws ssm describe-parameters --query 'Parameters[*].[Name,Type,LastModifiedDate]' --output table
```

#### Check for Non-Encrypted Parameters
```bash
# Find String (non-SecureString) parameters
aws ssm describe-parameters \
  --query 'Parameters[?Type==`String`].[Name]' --output text
```

### 3. KMS Key Management

#### Check Key Rotation
```bash
for key in $(aws kms list-keys --query 'Keys[*].KeyId' --output text); do
  key_manager=$(aws kms describe-key --key-id "$key" --query 'KeyMetadata.KeyManager' --output text)
  
  if [ "$key_manager" = "CUSTOMER" ]; then
    rotation=$(aws kms get-key-rotation-status --key-id "$key" --query 'KeyRotationEnabled' --output text)
    
    if [ "$rotation" != "True" ]; then
      echo "⚠️  Key $key: Rotation NOT enabled"
    fi
  fi
done
```

### 4. Lambda Environment Variables

#### Scan for Hardcoded Secrets
```bash
for func in $(aws lambda list-functions --query 'Functions[*].FunctionName' --output text); do
  env_vars=$(aws lambda get-function-configuration --function-name "$func" --query 'Environment.Variables' --output json)
  
  # Check for suspicious variable names
  if echo "$env_vars" | jq -e 'keys[] | select(. | test("PASSWORD|SECRET|KEY|TOKEN"; "i"))' >/dev/null 2>&1; then
    echo "⚠️  Function $func has suspicious environment variables"
  fi
done
```

---

<!-- analysis: iac -->
## Security Checklist

### Critical
- [ ] No plaintext secrets in Lambda environment variables
- [ ] No secrets in EC2 user data
- [ ] No hardcoded credentials in code repositories

### High Priority
- [ ] KMS key rotation enabled for customer managed keys
- [ ] Secrets Manager rotation configured
- [ ] Parameter Store uses SecureString for sensitive data
- [ ] IAM policies follow least privilege for secret access

### Best Practices
- [ ] Customer managed KMS keys for encryption
- [ ] Secrets Manager preferred over Parameter Store
- [ ] Automatic rotation for database credentials
- [ ] Audit logging enabled for secret access

---

## Summary Report Template

**Audit Date**: [Date]

### Findings
| Severity | Count |
|----------|-------|
| CRITICAL | X |
| HIGH | X |
| MEDIUM | X |

### Secrets Inventory
- Secrets Manager: X secrets (X with rotation)
- Parameter Store: X parameters (X SecureString)
- KMS Keys: X (X with rotation)

### Top Recommendations
1. Migrate hardcoded secrets to Secrets Manager
2. Enable KMS key rotation
3. Configure automatic secret rotation
4. Implement least privilege IAM policies

---

**Next Steps**: Immediate remediation of hardcoded secrets.
