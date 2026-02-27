---
genre: hosting
category: logging-monitoring
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/cloudformation/**"
  keywords:
    - aws_cloudtrail
    - aws_cloudwatch
    - aws_config
    - aws_guardduty
    - flow_log
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Logging and Monitoring Audit Template

**Purpose**: Assess AWS logging, monitoring, and threat detection capabilities including CloudTrail, CloudWatch, GuardDuty, Security Hub, and Config.

**Frameworks**: CIS AWS Foundations Benchmark 3.x, AWS Well-Architected Security Pillar - Detective Controls

---

## Agentic Prompt Examples

### Quick Logging Audit
```
@terminal Audit AWS logging and monitoring: check CloudTrail status across all 
regions, verify CloudWatch log group retention, check GuardDuty enablement, review 
Security Hub findings, and verify AWS Config is recording. Flag any region without 
CloudTrail as CRITICAL.
```

### Deep Monitoring Assessment
```
@terminal Perform comprehensive logging assessment: analyze CloudTrail configuration 
including S3 bucket encryption and log file validation, review CloudWatch alarms for 
security events, check GuardDuty findings severity distribution, audit Security Hub 
compliance status, and verify Config rules are active. Generate compliance report.
```

---

## AWS CLI Commands

### 1. CloudTrail Audit (CIS 3.1-3.11)

#### Check CloudTrail Status Per Region (CIS 3.1)
```bash
# Verify CloudTrail enabled in all regions
echo "🔍 Checking CloudTrail status across all regions..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  trails=$(aws cloudtrail describe-trails --region "$region" --query 'trailList[*].[Name,IsMultiRegionTrail,IsOrganizationTrail]' --output text)
  
  if [ -z "$trails" ]; then
    echo "🚨 CRITICAL: No CloudTrail trails in region $region"
  else
    echo "✓ Region $region has trails: $trails"
  fi
done
```

#### Check CloudTrail Log File Validation (CIS 3.2)
```bash
# Verify log file integrity validation is enabled
echo "🔍 Checking CloudTrail log file validation..."

aws cloudtrail describe-trails --query 'trailList[*].[Name,LogFileValidationEnabled,S3BucketName]' --output table
```

#### Check CloudTrail S3 Bucket Access Logging (CIS 3.3)
```bash
# Verify S3 bucket logging for CloudTrail bucket
echo "🔍 Checking CloudTrail S3 bucket logging..."

for trail in $(aws cloudtrail describe-trails --query 'trailList[*].S3BucketName' --output text | sort -u); do
  logging=$(aws s3api get-bucket-logging --bucket "$trail" 2>/dev/null)
  
  if [ "$logging" = "" ] || [ "$logging" = "{}" ]; then
    echo "⚠️  MEDIUM: CloudTrail S3 bucket $trail does NOT have access logging"
  else
    echo "✓ CloudTrail bucket $trail has logging enabled"
  fi
done
```

#### Check CloudTrail S3 Bucket Encryption (CIS 3.7)
```bash
# Verify CloudTrail logs are encrypted at rest
echo "🔍 Checking CloudTrail log encryption..."

for trail in $(aws cloudtrail describe-trails --query 'trailList[*].[Name,KmsKeyId]' --output text); do
  trail_name=$(echo "$trail" | awk '{print $1}')
  kms_key=$(echo "$trail" | awk '{print $2}')
  
  if [ "$kms_key" = "None" ] || [ -z "$kms_key" ]; then
    echo "⚠️  MEDIUM: Trail $trail_name does NOT use KMS encryption"
  else
    echo "✓ Trail $trail_name encrypted with KMS: $kms_key"
  fi
done
```

#### Check CloudTrail Integration with CloudWatch Logs (CIS 3.4)
```bash
# Verify CloudTrail sends logs to CloudWatch
echo "🔍 Checking CloudTrail CloudWatch Logs integration..."

aws cloudtrail describe-trails --query 'trailList[*].[Name,CloudWatchLogsLogGroupArn,CloudWatchLogsRoleArn]' --output table
```

### 2. CloudWatch Logs Audit

#### Check CloudWatch Log Group Retention (CIS 3.3)
```bash
# Verify log retention is set (not indefinite)
echo "🔍 Checking CloudWatch log group retention..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  log_groups=$(aws logs describe-log-groups --region "$region" --query 'logGroups[?retentionInDays==null].[logGroupName]' --output text)
  
  if [ -n "$log_groups" ]; then
    echo "⚠️  MEDIUM: Log groups with no retention policy in $region:"
    echo "$log_groups"
    echo ""
  fi
done
```

#### Check CloudWatch Log Group Encryption
```bash
# Verify log groups are encrypted with KMS
echo "🔍 Checking CloudWatch log group encryption..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  unencrypted=$(aws logs describe-log-groups --region "$region" --query 'logGroups[?kmsKeyId==null].[logGroupName]' --output text)
  
  if [ -n "$unencrypted" ]; then
    echo "⚠️  MEDIUM: Unencrypted log groups in $region:"
    echo "$unencrypted"
    echo ""
  fi
done
```

### 3. CloudWatch Alarms for CIS Metrics (CIS 4.1-4.15)

#### Check Security-Related CloudWatch Alarms
```bash
# List all CloudWatch alarms
echo "🔍 Checking CloudWatch alarms..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  alarms=$(aws cloudwatch describe-alarms --region "$region" --query 'MetricAlarms[*].[AlarmName,StateValue,ActionsEnabled]' --output text)
  
  if [ -n "$alarms" ]; then
    echo "Region: $region"
    echo "$alarms"
    echo ""
  fi
done

# Required CIS metric filters (should have alarms):
# 4.1 - Unauthorized API calls
# 4.2 - Console sign-in without MFA
# 4.3 - Root account usage
# 4.4 - IAM policy changes
# 4.5 - CloudTrail configuration changes
# 4.6 - Console authentication failures
# 4.7 - CMK deletion or disablement
# 4.8 - S3 bucket policy changes
# 4.9 - AWS Config changes
# 4.10 - Security group changes
# 4.11 - NACL changes
# 4.12 - Network gateway changes
# 4.13 - Route table changes
# 4.14 - VPC changes
# 4.15 - Organizations changes
```

### 4. GuardDuty Audit

#### Check GuardDuty Status
```bash
# Verify GuardDuty is enabled in all regions
echo "🔍 Checking GuardDuty status..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  detector=$(aws guardduty list-detectors --region "$region" --query 'DetectorIds[0]' --output text 2>/dev/null)
  
  if [ -z "$detector" ] || [ "$detector" = "None" ]; then
    echo "⚠️  HIGH: GuardDuty NOT enabled in $region"
  else
    status=$(aws guardduty get-detector --region "$region" --detector-id "$detector" --query 'Status' --output text)
    echo "✓ GuardDuty enabled in $region (Status: $status, Detector: $detector)"
  fi
done
```

#### Get GuardDuty Findings
```bash
# Retrieve active GuardDuty findings
echo "🔍 Retrieving GuardDuty findings..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  detector=$(aws guardduty list-detectors --region "$region" --query 'DetectorIds[0]' --output text 2>/dev/null)
  
  if [ -n "$detector" ] && [ "$detector" != "None" ]; then
    findings=$(aws guardduty list-findings --region "$region" --detector-id "$detector" --finding-criteria '{"Criterion":{"service.archived":{"Eq":["false"]}}}' --query 'FindingIds[*]' --output text)
    
    if [ -n "$findings" ]; then
      count=$(echo "$findings" | wc -w)
      echo "🚨 Region $region has $count active GuardDuty findings"
      
      # Get finding details
      for finding_id in $findings; do
        aws guardduty get-findings --region "$region" --detector-id "$detector" --finding-ids "$finding_id" --query 'Findings[0].[Title,Severity,Type,Description]' --output table
      done
    else
      echo "✓ No active GuardDuty findings in $region"
    fi
  fi
done
```

### 5. AWS Security Hub Audit

#### Check Security Hub Status
```bash
# Verify Security Hub is enabled
echo "🔍 Checking Security Hub status..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  hub_arn=$(aws securityhub describe-hub --region "$region" --query 'HubArn' --output text 2>/dev/null)
  
  if [ -z "$hub_arn" ] || echo "$hub_arn" | grep -q "not subscribed"; then
    echo "⚠️  MEDIUM: Security Hub NOT enabled in $region"
  else
    echo "✓ Security Hub enabled in $region"
  fi
done
```

#### Get Security Hub Findings
```bash
# Retrieve Security Hub findings by severity
echo "🔍 Retrieving Security Hub findings..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  # Check critical findings
  critical=$(aws securityhub get-findings --region "$region" \
    --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}],"RecordState":[{"Value":"ACTIVE","Comparison":"EQUALS"}]}' \
    --query 'length(Findings)' --output text 2>/dev/null)
  
  high=$(aws securityhub get-findings --region "$region" \
    --filters '{"SeverityLabel":[{"Value":"HIGH","Comparison":"EQUALS"}],"RecordState":[{"Value":"ACTIVE","Comparison":"EQUALS"}]}' \
    --query 'length(Findings)' --output text 2>/dev/null)
  
  if [ "$critical" -gt 0 ] || [ "$high" -gt 0 ]; then
    echo "🚨 Region $region: CRITICAL=$critical, HIGH=$high findings"
  fi
done
```

#### Check Security Hub Standards
```bash
# List enabled security standards
echo "🔍 Checking Security Hub standards..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  standards=$(aws securityhub get-enabled-standards --region "$region" 2>/dev/null)
  
  if [ -n "$standards" ] && [ "$standards" != '{"StandardsSubscriptions":[]}' ]; then
    echo "Region: $region"
    echo "$standards" | jq -r '.StandardsSubscriptions[] | "\(.StandardsArn) | Status: \(.StandardsStatus)"'
    echo ""
  fi
done
```

### 6. AWS Config Audit (CIS 3.5)

#### Check AWS Config Status
```bash
# Verify Config is recording in all regions
echo "🔍 Checking AWS Config status..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  recorders=$(aws configservice describe-configuration-recorders --region "$region" 2>/dev/null)
  
  if [ "$recorders" = '{"ConfigurationRecorders":[]}' ]; then
    echo "🚨 HIGH: AWS Config NOT configured in $region"
  else
    recording_status=$(aws configservice describe-configuration-recorder-status --region "$region" --query 'ConfigurationRecordersStatus[0].recording' --output text)
    
    if [ "$recording_status" = "True" ]; then
      echo "✓ AWS Config recording in $region"
    else
      echo "⚠️  HIGH: AWS Config NOT recording in $region"
    fi
  fi
done
```

#### Check Config Rules Compliance
```bash
# Get Config rule compliance status
echo "🔍 Checking AWS Config rule compliance..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  rules=$(aws configservice describe-compliance-by-config-rule --region "$region" 2>/dev/null)
  
  if [ -n "$rules" ] && [ "$rules" != '{"ComplianceByConfigRules":[]}' ]; then
    echo "Region: $region"
    non_compliant=$(echo "$rules" | jq -r '.ComplianceByConfigRules[] | select(.Compliance.ComplianceType != "COMPLIANT") | "\(.ConfigRuleName): \(.Compliance.ComplianceType)"')
    
    if [ -n "$non_compliant" ]; then
      echo "⚠️  Non-compliant Config rules:"
      echo "$non_compliant"
    fi
    echo ""
  fi
done
```

### 7. VPC Flow Logs (CIS 2.9)

#### Check VPC Flow Logs Status
```bash
# Verify flow logs enabled for all VPCs
echo "🔍 Checking VPC Flow Logs..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  vpcs=$(aws ec2 describe-vpcs --region "$region" --query 'Vpcs[*].VpcId' --output text)
  
  if [ -n "$vpcs" ]; then
    for vpc in $vpcs; do
      flow_logs=$(aws ec2 describe-flow-logs --region "$region" --filter "Name=resource-id,Values=$vpc" --query 'FlowLogs[*].FlowLogId' --output text)
      
      if [ -z "$flow_logs" ]; then
        echo "🚨 HIGH: VPC $vpc in $region has NO flow logs"
      fi
    done
  fi
done
```

### 8. CloudWatch Logs Insights Queries

#### Sample Security Queries
```bash
# Query CloudTrail logs for unauthorized API calls
LOG_GROUP="/aws/cloudtrail/my-trail"
QUERY='fields @timestamp, userIdentity.principalId, eventName, errorCode
| filter errorCode = "UnauthorizedOperation" or errorCode = "AccessDenied"
| sort @timestamp desc
| limit 20'

aws logs start-query \
  --log-group-name "$LOG_GROUP" \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string "$QUERY"
```

---

<!-- analysis: iac -->
## Security Checklist

### Critical

- [ ] **CloudTrail enabled in all regions** (CIS 3.1)
  - Severity: CRITICAL
  - Finding: No audit trail for API activity

- [ ] **GuardDuty enabled in all regions**
  - Severity: HIGH
  - Finding: No threat detection

### High Priority

- [ ] **CloudTrail log file validation enabled** (CIS 3.2)
  - Severity: HIGH
  - Finding: Log integrity cannot be verified

- [ ] **CloudTrail integrated with CloudWatch Logs** (CIS 3.4)
  - Severity: HIGH
  - Finding: No real-time log analysis

- [ ] **AWS Config recording in all regions** (CIS 3.5)
  - Severity: HIGH
  - Finding: No configuration change tracking

- [ ] **VPC Flow Logs enabled for all VPCs** (CIS 2.9)
  - Severity: HIGH
  - Finding: No network traffic visibility

### Medium Priority

- [ ] **CloudWatch log groups have retention policies**
  - Severity: MEDIUM
  - Finding: Uncontrolled log storage costs

- [ ] **Security Hub enabled**
  - Severity: MEDIUM
  - Finding: No centralized security findings

- [ ] **CloudWatch alarms for security events** (CIS 4.x)
  - Severity: MEDIUM
  - Finding: No alerting on security events

---

## Summary Report Template

**Audit Date**: [Date]  
**Auditor**: [Name]

### Findings Summary
| Severity | Count |
|----------|-------|
| CRITICAL | X |
| HIGH | X |
| MEDIUM | X |

### Logging Status
- CloudTrail: Enabled in X/X regions
- GuardDuty: Enabled in X/X regions
- Security Hub: Enabled in X/X regions
- AWS Config: Recording in X/X regions
- VPC Flow Logs: X/X VPCs covered

### GuardDuty Findings
- Critical: X
- High: X
- Medium: X

### Top Recommendations
1. Enable CloudTrail in all regions
2. Enable GuardDuty for threat detection
3. Enable VPC Flow Logs
4. Configure CloudWatch alarms for security events
5. Enable AWS Config recording

---

**Next Steps**: Enable missing detective controls within 7 days.
