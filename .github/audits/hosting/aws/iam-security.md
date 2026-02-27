---
genre: hosting
category: iam-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/cloudformation/**"
    - "**/cdk/**"
    - "serverless.yml"
  keywords:
    - aws_iam
    - iam_role
    - iam_policy
    - iam_user
    - assume_role
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# IAM Security Audit Template

**Purpose**: Assess AWS Identity and Access Management (IAM) security posture to identify privilege escalation risks, credential exposure, and policy misconfigurations.

**Frameworks**: CIS AWS Foundations Benchmark 1.1-1.22, AWS Well-Architected Security Pillar

---

## Agentic Prompt Examples

### Quick IAM Audit
```
@terminal Audit AWS IAM security: check root account MFA status, list all 
users without MFA enabled, identify users with unused credentials older than 
90 days, check password policy compliance, and find overly permissive inline 
policies. Output findings with severity ratings.
```

### Deep IAM Assessment
```
@terminal Perform comprehensive IAM security assessment: analyze all IAM users, 
roles, and policies for least privilege violations. Check for privilege escalation 
paths, wildcard permissions, cross-account access risks, and service control policy 
gaps. Generate detailed findings report with CIS benchmark mapping.
```

### Credential Hygiene Check
```
@terminal Review IAM credential hygiene: list all access keys with age and last 
used date, identify stale credentials (>90 days unused), check for keys belonging 
to deleted users, find users with multiple active access keys, and verify key 
rotation practices.
```

---

## AWS CLI Commands

### 1. Account-Level Security

#### Root Account MFA Status (CIS 1.5)
```bash
# Check if root account has MFA enabled
aws iam get-account-summary --query 'SummaryMap.AccountMFAEnabled' --output text

# CRITICAL: Must return "1" (enabled)
# If returns "0", root account lacks MFA protection
```

#### Account Password Policy (CIS 1.8-1.11)
```bash
# Get password policy
aws iam get-account-password-policy --output json

# Verify requirements:
# - MinimumPasswordLength >= 14
# - RequireSymbols = true
# - RequireNumbers = true
# - RequireUppercaseCharacters = true
# - RequireLowercaseCharacters = true
# - AllowUsersToChangePassword = true
# - ExpirePasswords = true
# - MaxPasswordAge <= 90
# - PasswordReusePrevention >= 24
```

#### Account Summary
```bash
# Overview of IAM resources
aws iam get-account-summary --output json

# Check for:
# - AccountAccessKeysPresent (should be 0 - no root keys)
# - Users (minimize human users, prefer roles)
# - MFADevices vs UsersQuota (MFA coverage)
```

### 2. IAM Users Audit

#### List All IAM Users
```bash
# Get all IAM users
aws iam list-users --output json > iam-users.json

# Extract usernames
aws iam list-users --query 'Users[*].UserName' --output text
```

#### MFA Status for Users (CIS 1.2, 1.3)
```bash
# Check MFA for all users
for user in $(aws iam list-users --query 'Users[*].UserName' --output text); do
  mfa_devices=$(aws iam list-mfa-devices --user-name "$user" --query 'MFADevices[*].SerialNumber' --output text)
  if [ -z "$mfa_devices" ]; then
    echo "⚠️  HIGH: User $user has NO MFA enabled"
  else
    echo "✓ User $user has MFA: $mfa_devices"
  fi
done
```

#### Console Access Without MFA (CIS 1.2)
```bash
# Find users with console access but no MFA
for user in $(aws iam list-users --query 'Users[*].UserName' --output text); do
  # Check if user has console password
  aws iam get-login-profile --user-name "$user" &>/dev/null
  if [ $? -eq 0 ]; then
    # User has console access, check MFA
    mfa_count=$(aws iam list-mfa-devices --user-name "$user" --query 'length(MFADevices)' --output text)
    if [ "$mfa_count" -eq 0 ]; then
      echo "🚨 CRITICAL: User $user has console access WITHOUT MFA"
    fi
  fi
done
```

### 3. Access Keys Audit

#### List All Access Keys (CIS 1.4, 1.12)
```bash
# Get all access keys with details
for user in $(aws iam list-users --query 'Users[*].UserName' --output text); do
  keys=$(aws iam list-access-keys --user-name "$user" --query 'AccessKeyMetadata[*].[AccessKeyId,Status,CreateDate]' --output text)
  if [ -n "$keys" ]; then
    echo "User: $user"
    echo "$keys"
    echo "---"
  fi
done
```

#### Access Key Age and Last Used
```bash
# Check access key age and usage (flag keys >90 days unused)
for user in $(aws iam list-users --query 'Users[*].UserName' --output text); do
  for key in $(aws iam list-access-keys --user-name "$user" --query 'AccessKeyMetadata[*].AccessKeyId' --output text); do
    last_used=$(aws iam get-access-key-last-used --access-key-id "$key" --query 'AccessKeyLastUsed.LastUsedDate' --output text)
    create_date=$(aws iam list-access-keys --user-name "$user" --query "AccessKeyMetadata[?AccessKeyId=='$key'].CreateDate" --output text)
    
    echo "User: $user | Key: $key | Created: $create_date | Last Used: $last_used"
    
    # Calculate days since creation
    if [ "$last_used" = "None" ] || [ -z "$last_used" ]; then
      echo "⚠️  MEDIUM: Key never used - consider deleting"
    fi
  done
done
```

#### Root Account Access Keys (CIS 1.4)
```bash
# Check for root account access keys (CRITICAL if any exist)
root_keys=$(aws iam get-account-summary --query 'SummaryMap.AccountAccessKeysPresent' --output text)

if [ "$root_keys" -eq 1 ]; then
  echo "🚨 CRITICAL: Root account has access keys - DELETE IMMEDIATELY"
else
  echo "✓ Root account has no access keys"
fi
```

#### Inactive Access Keys
```bash
# Find access keys not used in 90+ days
aws iam generate-credential-report
sleep 10  # Wait for report generation

aws iam get-credential-report --query 'Content' --output text | base64 -d > credential-report.csv

# Parse credential report for unused keys
awk -F',' 'NR>1 {
  if ($11 == "true" && $15 != "N/A") {
    cmd = "date -d " $15 " +%s"
    cmd | getline last_used_epoch
    close(cmd)
    
    "date +%s" | getline now_epoch
    close("date +%s")
    
    days_unused = (now_epoch - last_used_epoch) / 86400
    
    if (days_unused > 90) {
      print "⚠️  HIGH: User " $1 " has access key unused for " int(days_unused) " days"
    }
  }
}' credential-report.csv
```

### 4. IAM Policies Audit

#### Policies with Broad Permissions
```bash
# List customer-managed policies
aws iam list-policies --scope Local --output json > custom-policies.json

# Check for overly permissive policies (wildcard actions/resources)
for policy_arn in $(aws iam list-policies --scope Local --query 'Policies[*].Arn' --output text); do
  echo "Analyzing policy: $policy_arn"
  
  # Get default policy version
  version=$(aws iam list-policy-versions --policy-arn "$policy_arn" --query 'Versions[?IsDefaultVersion==`true`].VersionId' --output text)
  
  # Get policy document
  policy_doc=$(aws iam get-policy-version --policy-arn "$policy_arn" --version-id "$version" --query 'PolicyVersion.Document' --output json)
  
  # Check for wildcard permissions
  wildcards=$(echo "$policy_doc" | jq -r '.Statement[] | select(.Effect=="Allow") | select(.Action=="*" or .Resource=="*") | "Action: \(.Action) | Resource: \(.Resource)"')
  
  if [ -n "$wildcards" ]; then
    echo "🚨 HIGH: Policy has wildcard permissions:"
    echo "$wildcards"
    echo "---"
  fi
done
```

#### Inline Policies (CIS 1.16)
```bash
# List all users with inline policies (should be none - use managed policies)
for user in $(aws iam list-users --query 'Users[*].UserName' --output text); do
  inline_policies=$(aws iam list-user-policies --user-name "$user" --query 'PolicyNames[*]' --output text)
  if [ -n "$inline_policies" ]; then
    echo "⚠️  MEDIUM: User $user has inline policies: $inline_policies"
    echo "Recommendation: Convert to managed policies"
  fi
done

# List all roles with inline policies
for role in $(aws iam list-roles --query 'Roles[*].RoleName' --output text); do
  inline_policies=$(aws iam list-role-policies --role-name "$role" --query 'PolicyNames[*]' --output text)
  if [ -n "$inline_policies" ]; then
    echo "⚠️  MEDIUM: Role $role has inline policies: $inline_policies"
  fi
done
```

#### Administrative Policies Attached to Users (CIS 1.22)
```bash
# Find users with AdministratorAccess
for user in $(aws iam list-users --query 'Users[*].UserName' --output text); do
  admin_attached=$(aws iam list-attached-user-policies --user-name "$user" --query 'AttachedPolicies[?PolicyName==`AdministratorAccess`]' --output text)
  
  if [ -n "$admin_attached" ]; then
    echo "🚨 HIGH: User $user has AdministratorAccess attached directly"
    echo "Recommendation: Remove direct attachment, use groups or roles"
  fi
done
```

### 5. IAM Roles Audit

#### List All IAM Roles
```bash
# Get all IAM roles
aws iam list-roles --output json > iam-roles.json

# Roles with assume role policies
aws iam list-roles --query 'Roles[*].[RoleName,Arn,Description]' --output table
```

#### Roles with Wildcard Trust Policies
```bash
# Check for overly permissive assume role policies
for role in $(aws iam list-roles --query 'Roles[*].RoleName' --output text); do
  trust_policy=$(aws iam get-role --role-name "$role" --query 'Role.AssumeRolePolicyDocument' --output json)
  
  # Check for wildcards in principal
  wildcard=$(echo "$trust_policy" | jq -r '.Statement[] | select(.Principal.AWS? // "" | contains("*")) | .Principal.AWS')
  
  if [ -n "$wildcard" ] && [ "$wildcard" != "null" ]; then
    echo "🚨 HIGH: Role $role has wildcard in trust policy: $wildcard"
  fi
done
```

#### Roles Not Used in 90 Days
```bash
# Find unused roles (requires tracking usage externally or via CloudTrail)
for role in $(aws iam list-roles --query 'Roles[*].RoleName' --output text); do
  last_used=$(aws iam get-role --role-name "$role" --query 'Role.RoleLastUsed.LastUsedDate' --output text)
  
  if [ "$last_used" = "None" ] || [ -z "$last_used" ]; then
    echo "⚠️  LOW: Role $role has never been used"
  else
    echo "Role $role last used: $last_used"
  fi
done
```

### 6. Service-Linked Roles
```bash
# List AWS service-linked roles (managed by AWS, safe to ignore for security issues)
aws iam list-roles --query 'Roles[?contains(Arn, `service-role`)].[RoleName,Arn]' --output table
```

### 7. IAM Groups Audit
```bash
# List all IAM groups
aws iam list-groups --output json > iam-groups.json

# Check group membership
for group in $(aws iam list-groups --query 'Groups[*].GroupName' --output text); do
  echo "Group: $group"
  aws iam get-group --group-name "$group" --query 'Users[*].UserName' --output text
  echo "Attached policies:"
  aws iam list-attached-group-policies --group-name "$group" --query 'AttachedPolicies[*].PolicyName' --output text
  echo "---"
done
```

### 8. Cross-Account Access Audit
```bash
# Find roles that allow cross-account access
for role in $(aws iam list-roles --query 'Roles[*].RoleName' --output text); do
  trust_policy=$(aws iam get-role --role-name "$role" --query 'Role.AssumeRolePolicyDocument' --output json)
  
  # Extract external account IDs
  external_accounts=$(echo "$trust_policy" | jq -r '.Statement[] | select(.Principal.AWS?) | .Principal.AWS' | grep -v "$(aws sts get-caller-identity --query Account --output text)")
  
  if [ -n "$external_accounts" ]; then
    echo "ℹ️  INFO: Role $role allows cross-account access:"
    echo "$external_accounts"
    echo "Verify these external accounts are authorized"
    echo "---"
  fi
done
```

---

<!-- analysis: iac -->
## Security Checklist

### Critical (Must Fix Immediately)

- [ ] **Root account has MFA enabled** (CIS 1.5)
  - Command: `aws iam get-account-summary`
  - Severity: CRITICAL
  - Finding: Root account without MFA is vulnerable to credential compromise

- [ ] **Root account has NO access keys** (CIS 1.4)
  - Command: `aws iam get-account-summary`
  - Severity: CRITICAL
  - Finding: Root account access keys can't be restricted and pose maximum risk

- [ ] **No users with console access lack MFA** (CIS 1.2)
  - Command: Check login profile + MFA devices
  - Severity: CRITICAL
  - Finding: Console access without MFA vulnerable to password compromise

- [ ] **No users have AdministratorAccess attached directly** (CIS 1.22)
  - Command: `aws iam list-attached-user-policies`
  - Severity: HIGH
  - Finding: Admin permissions should be role-based, not user-attached

### High Priority

- [ ] **Password policy meets CIS requirements** (CIS 1.8-1.11)
  - Minimum 14 characters, complexity requirements, expiration <= 90 days
  - Severity: HIGH
  - Finding: Weak password policy increases brute force risk

- [ ] **No access keys older than 90 days** (CIS 1.14)
  - Command: `aws iam list-access-keys` + calculate age
  - Severity: HIGH
  - Finding: Old access keys increase exposure window if compromised

- [ ] **No unused access keys (90+ days inactive)** (CIS 1.12)
  - Command: `aws iam get-access-key-last-used`
  - Severity: HIGH
  - Finding: Unused keys provide unnecessary attack surface

- [ ] **No wildcard permissions in customer-managed policies**
  - Command: Check policy documents for `"Action": "*"` or `"Resource": "*"`
  - Severity: HIGH
  - Finding: Violates least privilege principle

### Medium Priority

- [ ] **Minimize use of inline policies** (CIS 1.16)
  - Command: `aws iam list-user-policies`, `aws iam list-role-policies`
  - Severity: MEDIUM
  - Finding: Inline policies harder to audit and manage centrally

- [ ] **All users belong to at least one group**
  - Command: Check user group membership
  - Severity: MEDIUM
  - Finding: Direct user policy attachments harder to manage at scale

- [ ] **Credential report generated regularly**
  - Command: `aws iam generate-credential-report`
  - Severity: MEDIUM
  - Finding: Regular credential audits prevent security drift

### Low Priority / Best Practices

- [ ] **Unused IAM roles identified and removed**
  - Command: Check `RoleLastUsed` field
  - Severity: LOW
  - Finding: Reduces attack surface and management overhead

- [ ] **All IAM entities have descriptions/tags**
  - Command: Check `Description` field in roles/policies
  - Severity: LOW
  - Finding: Improves auditability and compliance documentation

- [ ] **Service control policies (SCPs) in use** (AWS Organizations)
  - Command: `aws organizations list-policies --filter SERVICE_CONTROL_POLICY`
  - Severity: INFO
  - Finding: SCPs provide defense-in-depth at organization level

---

## Findings Template

### Finding: [Title]

**Severity**: CRITICAL | HIGH | MEDIUM | LOW | INFO

**CIS Benchmark**: [e.g., 1.5 - Ensure MFA is enabled for root account]

**Description**: 
[Detailed description of the security issue]

**Evidence**:
```bash
# AWS CLI command output showing the issue
[paste command output]
```

**Risk**:
- **Confidentiality**: [Impact on data confidentiality]
- **Integrity**: [Impact on data integrity]
- **Availability**: [Impact on system availability]

**Business Impact**:
[Explain the business consequences if exploited]

**Affected Resources**:
- Resource 1: [ARN or identifier]
- Resource 2: [ARN or identifier]

**Remediation Steps**:

1. [Step-by-step remediation instructions]
2. [Include AWS CLI commands or Console steps]
3. [Verification step to confirm fix]

**AWS CLI Remediation** (if applicable):
```bash
# Example remediation commands
aws iam enable-mfa-device --user-name example-user --serial-number arn:aws:iam::123456789012:mfa/example-user --authentication-code1 123456 --authentication-code2 789012
```

**CloudFormation/Terraform Remediation** (if applicable):
```hcl
# Example IaC fix
resource "aws_iam_account_password_policy" "strict" {
  minimum_password_length        = 14
  require_symbols                = true
  require_numbers                = true
  require_uppercase_characters   = true
  require_lowercase_characters   = true
  allow_users_to_change_password = true
  max_password_age              = 90
  password_reuse_prevention     = 24
}
```

**Verification**:
```bash
# Command to verify the fix
[verification command]
```

**References**:
- CIS AWS Foundations Benchmark v1.5: [link]
- AWS Documentation: [link]
- MITRE ATT&CK: [relevant technique, e.g., T1078]

---

## Example Findings

### Example 1: Root Account Without MFA

**Severity**: CRITICAL

**CIS Benchmark**: 1.5 - Ensure MFA is enabled for root account

**Description**: 
The AWS root account does not have Multi-Factor Authentication (MFA) enabled. The root account has unrestricted access to all resources and services in the AWS account.

**Evidence**:
```bash
$ aws iam get-account-summary --query 'SummaryMap.AccountMFAEnabled' --output text
0
```

**Risk**:
- **Confidentiality**: CRITICAL - Full account compromise possible
- **Integrity**: CRITICAL - Attacker can modify all resources
- **Availability**: CRITICAL - Attacker can delete entire infrastructure

**Business Impact**:
Complete AWS account takeover. Attacker could exfiltrate all data, deploy cryptocurrency miners, delete backups, and hold infrastructure for ransom. Estimated impact: $500K+ in recovery costs plus reputation damage.

**Affected Resources**:
- Root account: 123456789012

**Remediation Steps**:

1. Sign in to AWS Console as root user
2. Navigate to "My Security Credentials"
3. Click "Activate MFA" on your root account
4. Choose "Virtual MFA device"
5. Use Google Authenticator or similar TOTP app
6. Scan QR code and enter two consecutive MFA codes
7. Store recovery codes in secure password manager

**Verification**:
```bash
$ aws iam get-account-summary --query 'SummaryMap.AccountMFAEnabled' --output text
1
```

**References**:
- CIS AWS Foundations Benchmark 1.5
- MITRE ATT&CK T1078.004 (Valid Accounts: Cloud Accounts)

---

### Example 2: Users with Console Access Lacking MFA

**Severity**: CRITICAL

**CIS Benchmark**: 1.2 - Ensure MFA is enabled for all IAM users that have console password

**Description**: 
Multiple IAM users have console access enabled but do not have MFA configured.

**Evidence**:
```bash
User: john.doe - Console Access: Yes, MFA: No
User: jane.smith - Console Access: Yes, MFA: No
User: bob.johnson - Console Access: Yes, MFA: No
```

**Risk**:
- **Confidentiality**: HIGH - Compromised passwords grant full console access
- **Integrity**: HIGH - Attacker can modify resources based on user permissions
- **Availability**: MEDIUM - Attacker could disrupt services

**Business Impact**:
Phishing or credential stuffing attacks could compromise these accounts. Each user has varying levels of access that could be exploited for data theft or infrastructure disruption.

**Affected Resources**:
- IAM User: john.doe (arn:aws:iam::123456789012:user/john.doe)
- IAM User: jane.smith (arn:aws:iam::123456789012:user/jane.smith)
- IAM User: bob.johnson (arn:aws:iam::123456789012:user/bob.johnson)

**Remediation Steps**:

1. Notify each user to enable MFA
2. Provide MFA setup instructions
3. Set deadline for compliance (e.g., 7 days)
4. Consider IAM policy to deny all actions until MFA enabled:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllExceptListedIfNoMFA",
      "Effect": "Deny",
      "NotAction": [
        "iam:CreateVirtualMFADevice",
        "iam:EnableMFADevice",
        "iam:GetUser",
        "iam:ListMFADevices",
        "iam:ListVirtualMFADevices",
        "iam:ResyncMFADevice",
        "sts:GetSessionToken"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

**Verification**:
```bash
for user in john.doe jane.smith bob.johnson; do
  aws iam list-mfa-devices --user-name "$user" --query 'MFADevices[*].SerialNumber' --output text
done
```

**References**:
- CIS AWS Foundations Benchmark 1.2
- AWS Security Best Practices

---

## Penetration Testing Perspective

### Privilege Escalation Paths

Check for IAM privilege escalation vectors:

```bash
# Find users with iam:PutUserPolicy (can grant themselves admin)
# Find roles with iam:PassRole + lambda:CreateFunction (can create privileged Lambda)
# Find users with iam:AttachUserPolicy (can attach AdministratorAccess)
# Find users with iam:CreateAccessKey for other users
```

Use tools like [Pacu](https://github.com/RhinoSecurityLabs/pacu) for automated privilege escalation checks:

```bash
# Install Pacu
git clone https://github.com/RhinoSecurityLabs/pacu.git
cd pacu && bash install.sh

# Run IAM privilege escalation module
run iam__privesc_scan
```

### Attack Scenarios

1. **Credential Stuffing**: Test password policy strength against known breached credentials
2. **MFA Bypass**: Check for backup authentication methods or MFA reset procedures
3. **Role Assumption**: Attempt to assume roles with weak trust policies
4. **API Key Leakage**: Search GitHub/GitLab for exposed AWS access keys for this account
5. **Confused Deputy**: Test cross-account role assumptions with crafted external IDs

---

## Compliance Mapping

| CIS Benchmark | Control | Priority |
|---------------|---------|----------|
| 1.4 | Ensure no root account access key exists | CRITICAL |
| 1.5 | Ensure MFA enabled for root account | CRITICAL |
| 1.2 | Ensure MFA enabled for all IAM users with console password | CRITICAL |
| 1.8-1.11 | Password policy requirements | HIGH |
| 1.12 | Ensure credentials unused for 90 days or greater are disabled | HIGH |
| 1.14 | Ensure access keys are rotated every 90 days | HIGH |
| 1.16 | Ensure IAM policies attached only to groups or roles | MEDIUM |
| 1.22 | Ensure IAM policies that allow full "*:*" admin privileges not created | HIGH |

---

## Summary Report Template

### IAM Security Audit Summary

**Audit Date**: [YYYY-MM-DD]  
**Auditor**: [Name]  
**Account ID**: [123456789012]  
**Account Name**: [Production/Staging/etc]

#### Executive Summary

[2-3 sentences on overall IAM security posture]

#### Findings by Severity

| Severity | Count | % of Total |
|----------|-------|------------|
| CRITICAL | X | XX% |
| HIGH | X | XX% |
| MEDIUM | X | XX% |
| LOW | X | XX% |
| INFO | X | XX% |
| **TOTAL** | **X** | **100%** |

#### Top 3 Critical Findings

1. [Finding title and brief description]
2. [Finding title and brief description]
3. [Finding title and brief description]

#### Recommendations Priority

1. **Immediate (0-7 days)**: [List critical findings]
2. **Short-term (7-30 days)**: [List high findings]
3. **Medium-term (30-90 days)**: [List medium findings]
4. **Long-term (90+ days)**: [List low/info findings]

#### Metrics

- Total IAM Users: X
- Users with MFA: X (XX%)
- Users without MFA: X (XX%)
- Total Access Keys: X
- Access Keys >90 days old: X
- Roles with Admin Access: X
- Customer-Managed Policies: X
- Policies with Wildcards: X

---

**Next Steps**: Schedule remediation planning meeting with security and DevOps teams.
