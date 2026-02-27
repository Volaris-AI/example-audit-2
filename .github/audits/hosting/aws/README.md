# AWS Hosting Security Audit

Comprehensive AWS infrastructure security audit templates designed for agentic coding tools (GitHub Copilot CLI/IDE) with AWS CLI access.

## Overview

These templates enable AI coding agents to perform thorough security audits of AWS infrastructure by running AWS CLI commands, analyzing configurations, and identifying security vulnerabilities.

**Target Audience:** DevSecOps engineers, security auditors, and penetration testers using AI-assisted tools.

## Prerequisites

### Required Tools
- **AWS CLI v2**: Install from [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- **jq**: JSON processor for parsing AWS CLI output (`apt-get install jq` or `brew install jq`)
- **GitHub Copilot CLI/IDE**: With terminal/shell access

### AWS Configuration
```bash
# Verify AWS CLI is configured
aws sts get-caller-identity

# Expected output shows your account ID, user/role ARN
{
  "UserId": "AIDAI...",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/auditor"
}
```

### Required IAM Permissions

The auditor account needs **read-only** access to perform security assessments:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:Get*",
        "iam:List*",
        "ec2:Describe*",
        "s3:GetBucket*",
        "s3:ListAllMyBuckets",
        "rds:Describe*",
        "lambda:List*",
        "lambda:Get*",
        "cloudtrail:Describe*",
        "cloudtrail:Get*",
        "cloudtrail:LookupEvents",
        "cloudwatch:Describe*",
        "cloudwatch:Get*",
        "cloudwatch:List*",
        "config:Describe*",
        "config:Get*",
        "guardduty:Get*",
        "guardduty:List*",
        "securityhub:Get*",
        "securityhub:Describe*",
        "securityhub:List*",
        "kms:Describe*",
        "kms:Get*",
        "kms:List*",
        "secretsmanager:Describe*",
        "secretsmanager:List*",
        "access-analyzer:List*",
        "access-analyzer:Get*"
      ],
      "Resource": "*"
    }
  ]
}
```

Attach the AWS managed policies:
- `SecurityAudit` (comprehensive read-only access for security auditing)
- `ViewOnlyAccess` (additional read access)

## Audit Workflow

### 1. **Preparation Phase**
```bash
# Set your AWS region
export AWS_REGION=us-east-1

# Verify access
aws sts get-caller-identity

# Get account details
aws iam get-account-summary
```

### 2. **Discovery Phase**
Run discovery commands to map your AWS environment:

```bash
# List all regions in use
aws ec2 describe-regions --query 'Regions[*].RegionName' --output text

# List resources across key services
aws resourcegroupstaggingapi get-resources --output json > aws-resources.json
```

### 3. **Audit Execution**
Work through each template systematically. Use the provided AWS CLI commands and let your AI agent analyze the output.

### 4. **Findings Documentation**
Document findings using the severity ratings:
- **CRITICAL**: Immediate risk (e.g., exposed credentials, public databases)
- **HIGH**: Significant risk (e.g., missing encryption, overly permissive IAM)
- **MEDIUM**: Notable security gap (e.g., missing MFA on some users)
- **LOW**: Best practice deviation (e.g., missing tags, incomplete logging)
- **INFO**: Observations for future consideration

### 5. **Remediation Planning**
Prioritize findings and create remediation tasks with:
- Finding description
- Business impact
- Remediation steps (with AWS CLI commands or CloudFormation/Terraform)
- Estimated effort
- Owner assignment

## Using Agentic Tools for AWS Audits

### Example Prompts for GitHub Copilot CLI

**IAM Security Audit:**
```
@terminal Audit AWS IAM for security issues: check for root account MFA, 
list users without MFA, find overly permissive policies, identify unused 
credentials older than 90 days, and check password policy compliance. 
Document findings with severity ratings.
```

**S3 Security Audit:**
```
@terminal Audit all S3 buckets for security: check for public access, 
encryption status, versioning, logging, and lifecycle policies. Flag any 
bucket with public access or missing encryption as CRITICAL.
```

**Network Security Audit:**
```
@terminal Review all VPC security groups for overly permissive rules. 
Find any rules allowing 0.0.0.0/0 on ports other than 80/443. Check for 
security groups with no inbound rules (unused). Document risky configurations.
```

**Comprehensive Multi-Service Audit:**
```
@terminal Perform a comprehensive AWS security audit covering IAM, S3, 
EC2 security groups, RDS public accessibility, CloudTrail status, and 
GuardDuty findings. Generate an executive summary with critical and high 
severity findings prioritized for immediate remediation.
```

### Agent Capabilities

When using GitHub Copilot or similar agentic tools:

1. **Command Execution**: Agent can run AWS CLI commands and parse JSON output
2. **Pattern Recognition**: Agent identifies security anti-patterns from configuration data
3. **Cross-Service Analysis**: Agent correlates findings across multiple AWS services
4. **Compliance Mapping**: Agent maps findings to CIS benchmarks and compliance frameworks
5. **Remediation Guidance**: Agent suggests specific AWS CLI commands or IaC changes

### Agent Workflow Pattern

```
1. Agent reads audit template
2. Agent executes AWS CLI commands
3. Agent parses JSON/text output
4. Agent applies security heuristics
5. Agent identifies violations
6. Agent documents findings with severity
7. Agent suggests remediation steps
```

## Audit Templates

### Core Security Domains

| Template | Focus Area | Key Risks |
|----------|-----------|-----------|
| [IAM Security](templates/iam-security.md) | Identity & Access Management | Privilege escalation, exposed credentials |
| [Network Security](templates/network-security.md) | VPC, Security Groups, Network Firewall | Unauthorized access, lateral movement |
| [Compute Security](templates/compute-security.md) | EC2, Lambda, ECS, EKS | Instance compromise, container escapes |
| [Storage Security](templates/storage-security.md) | S3, EBS, EFS | Data exfiltration, unauthorized access |
| [Database Security](templates/database-security.md) | RDS, DynamoDB, Redshift | Data breaches, SQL injection |
| [Logging & Monitoring](templates/logging-monitoring.md) | CloudTrail, CloudWatch, GuardDuty | Blind spots, delayed detection |
| [Compliance](templates/compliance.md) | CIS Benchmarks, Well-Architected | Regulatory violations |
| [Secrets Management](templates/secrets-management.md) | Secrets Manager, KMS | Credential exposure, crypto failures |

### Reporting

| Template | Purpose |
|----------|---------|
| [Executive Summary](templates/executive-summary.md) | High-level security posture overview |

## Multi-Region Audits

Many AWS resources are regional. Audit across all active regions:

```bash
# Loop through all regions
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "=== Auditing Region: $region ==="
  export AWS_REGION=$region
  
  # Run your audit commands here
  aws ec2 describe-security-groups --output json
  
done
```

## Output Management

### Organize Your Audit Data

```bash
# Create audit directory structure
mkdir -p audit-$(date +%Y%m%d)/{iam,network,compute,storage,database,logging,compliance,secrets}

# Save command outputs
aws iam get-account-summary > audit-$(date +%Y%m%d)/iam/account-summary.json
aws s3api list-buckets > audit-$(date +%Y%m%d)/storage/s3-buckets.json
```

### Generate Reports

Your agent can analyze the collected data and generate:
- Markdown reports for documentation
- CSV files for spreadsheet analysis
- JSON for integration with SIEM/security tools
- HTML dashboards for stakeholders

## Security Frameworks

These templates align with:

- **CIS AWS Foundations Benchmark v1.5**
- **AWS Well-Architected Framework - Security Pillar**
- **OWASP Cloud Security Top 10**
- **NIST Cybersecurity Framework**
- **ISO 27001 Controls**
- **PCI DSS 4.0** (for payment card environments)
- **HIPAA** (for healthcare environments)
- **SOC 2 Type II** (for SaaS providers)

## Common Pitfalls

### ❌ Don't Do This
- Run audits from production accounts (use dedicated auditor account)
- Make changes during audits (read-only assessment only)
- Skip multi-region checks (attackers exploit forgotten regions)
- Ignore INFO findings (they provide context)
- Audit once and forget (continuous monitoring required)

### ✅ Best Practices
- Use dedicated auditor IAM role with MFA
- Export audit data to S3 for retention
- Schedule regular audits (monthly minimum)
- Automate repetitive checks with Lambda/EventBridge
- Track remediation progress in ticketing system
- Re-audit after major changes

## Remediation Tracking

After documenting findings:

1. **Triage**: Assign severity and priority
2. **Assign**: Designate owner for each finding
3. **Track**: Use Jira/GitHub Issues for remediation tasks
4. **Verify**: Re-run audit commands to confirm fixes
5. **Document**: Update runbooks and security policies

## Advanced Techniques

### Using AWS Config for Continuous Compliance

```bash
# Check Config status
aws configservice describe-configuration-recorders

# Get compliance summary
aws configservice describe-compliance-by-config-rule
```

### Querying AWS Security Hub

```bash
# Get Security Hub findings
aws securityhub get-findings \
  --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}]}' \
  --output json
```

### Automated Scanning with Prowler

Consider supplementing manual audits with [Prowler](https://github.com/prowler-cloud/prowler):

```bash
# Install Prowler
pip install prowler

# Run comprehensive scan
prowler aws --output-formats json,html,csv
```

## Questions Before Starting

1. **Scope**: Which AWS accounts and regions are in scope?
2. **Compliance**: Which compliance frameworks apply?
3. **Risk Appetite**: What's the organization's risk tolerance?
4. **Timeline**: When is the audit due?
5. **Access**: Do you have SecurityAudit policy attached?
6. **Remediation Authority**: Can you fix issues or just report?

## Next Steps

1. Verify AWS CLI access and IAM permissions
2. Start with [IAM Security Audit](templates/iam-security.md) (foundation)
3. Proceed through remaining templates systematically
4. Generate [Executive Summary](templates/executive-summary.md)
5. Present findings to stakeholders
6. Create remediation plan
7. Schedule follow-up audit

## Support Resources

- **AWS Security Documentation**: https://docs.aws.amazon.com/security/
- **CIS AWS Benchmark**: https://www.cisecurity.org/benchmark/amazon_web_services
- **AWS Well-Architected Labs**: https://wellarchitectedlabs.com/security/
- **AWS Security Blog**: https://aws.amazon.com/blogs/security/

---

**Remember**: This is a security audit using read-only access. Document everything, but make no changes without proper approval and change management process.
