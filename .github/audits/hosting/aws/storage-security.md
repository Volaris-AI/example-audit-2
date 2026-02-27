---
genre: hosting
category: storage-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/cloudformation/**"
  keywords:
    - aws_s3_bucket
    - aws_ebs
    - s3_bucket_policy
    - bucket_acl
    - encryption
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Storage Security Audit Template

**Purpose**: Assess AWS storage security including S3, EBS, EFS, and backup configurations for encryption, access controls, and data protection.

**Frameworks**: CIS AWS Foundations Benchmark 2.1-2.3, AWS Well-Architected Security Pillar - Data Protection

---

## Agentic Prompt Examples

### Quick Storage Security Audit
```
@terminal Audit AWS storage security: check all S3 buckets for public access, 
encryption status, versioning, and logging. Review EBS volumes for encryption. 
Check EFS file systems for encryption at rest and in transit. Flag any publicly 
accessible buckets as CRITICAL and unencrypted storage as HIGH severity.
```

### Deep S3 Security Assessment
```
@terminal Perform comprehensive S3 security audit: analyze bucket policies, ACLs, 
public access blocks, encryption settings, versioning, MFA delete, lifecycle policies, 
and access logging for all buckets. Identify data exposure risks, compliance gaps, 
and ransomware vulnerabilities. Generate detailed remediation plan.
```

### Data Protection Audit
```
@terminal Audit AWS data protection: check S3 bucket encryption (SSE-S3, SSE-KMS), 
EBS volume encryption with KMS, RDS encryption status, backup retention policies, 
and cross-region replication. Verify encryption in transit for all storage services. 
Document findings with business impact assessment.
```

---

## AWS CLI Commands

### 1. S3 Bucket Security Audit

#### List All S3 Buckets
```bash
# Get all S3 buckets
aws s3api list-buckets --query 'Buckets[*].[Name,CreationDate]' --output table

# Count buckets
aws s3api list-buckets --query 'length(Buckets)' --output text
```

#### Check Public Access Block Configuration (CIS 2.1.5)
```bash
# Check S3 account-level public access block
echo "🔍 Checking S3 account-level public access block..."
aws s3control get-public-access-block --account-id $(aws sts get-caller-identity --query Account --output text) --output json

# Should have all four settings enabled:
# - BlockPublicAcls: true
# - IgnorePublicAcls: true
# - BlockPublicPolicy: true
# - RestrictPublicBuckets: true

# Check per-bucket public access block
echo "🔍 Checking bucket-level public access blocks..."
for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  public_block=$(aws s3api get-public-access-block --bucket "$bucket" 2>&1)
  
  if echo "$public_block" | grep -q "NoSuchPublicAccessBlockConfiguration"; then
    echo "🚨 CRITICAL: Bucket $bucket has NO public access block configuration"
  elif echo "$public_block" | grep -q "AccessDenied"; then
    echo "ℹ️  INFO: Access denied checking public access block for $bucket"
  else
    # Parse settings
    block_public_acls=$(echo "$public_block" | jq -r '.PublicAccessBlockConfiguration.BlockPublicAcls' 2>/dev/null)
    block_public_policy=$(echo "$public_block" | jq -r '.PublicAccessBlockConfiguration.BlockPublicPolicy' 2>/dev/null)
    
    if [ "$block_public_acls" != "true" ] || [ "$block_public_policy" != "true" ]; then
      echo "⚠️  HIGH: Bucket $bucket public access block not fully enabled"
    fi
  fi
done
```

#### Find Publicly Accessible Buckets (CIS 2.1.5)
```bash
# Check for public buckets via ACL and policy
echo "🔍 Checking for publicly accessible S3 buckets..."

for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  echo "Checking bucket: $bucket"
  
  # Check bucket ACL
  acl=$(aws s3api get-bucket-acl --bucket "$bucket" 2>/dev/null)
  public_acl=$(echo "$acl" | jq -r '.Grants[] | select(.Grantee.URI? // "" | contains("AllUsers") or contains("AuthenticatedUsers")) | .Permission' 2>/dev/null)
  
  if [ -n "$public_acl" ]; then
    echo "🚨 CRITICAL: Bucket $bucket has public ACL permissions: $public_acl"
  fi
  
  # Check bucket policy for public access
  policy=$(aws s3api get-bucket-policy --bucket "$bucket" --query 'Policy' --output text 2>/dev/null)
  if [ -n "$policy" ] && [ "$policy" != "None" ]; then
    public_policy=$(echo "$policy" | jq -r '.Statement[] | select(.Principal == "*" or .Principal.AWS == "*") | select(.Effect == "Allow") | .Action' 2>/dev/null)
    
    if [ -n "$public_policy" ]; then
      echo "🚨 CRITICAL: Bucket $bucket has policy allowing public access: $public_policy"
    fi
  fi
  
  echo "---"
done
```

#### S3 Bucket Encryption Status (CIS 2.1.1)
```bash
# Check encryption for all buckets
echo "🔍 Checking S3 bucket encryption..."

for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  encryption=$(aws s3api get-bucket-encryption --bucket "$bucket" 2>&1)
  
  if echo "$encryption" | grep -q "ServerSideEncryptionConfigurationNotFoundError"; then
    echo "🚨 HIGH: Bucket $bucket has NO default encryption enabled"
  elif echo "$encryption" | grep -q "AccessDenied"; then
    echo "ℹ️  INFO: Access denied checking encryption for $bucket"
  else
    # Parse encryption type
    encryption_type=$(echo "$encryption" | jq -r '.ServerSideEncryptionConfiguration.Rules[0].ApplyServerSideEncryptionByDefault.SSEAlgorithm' 2>/dev/null)
    kms_key=$(echo "$encryption" | jq -r '.ServerSideEncryptionConfiguration.Rules[0].ApplyServerSideEncryptionByDefault.KMSMasterKeyID' 2>/dev/null)
    
    if [ "$encryption_type" = "AES256" ]; then
      echo "✓ Bucket $bucket encrypted with SSE-S3 (AES256)"
    elif [ "$encryption_type" = "aws:kms" ]; then
      echo "✓ Bucket $bucket encrypted with SSE-KMS (Key: $kms_key)"
    fi
  fi
done
```

#### S3 Bucket Versioning Status (CIS 2.1.3)
```bash
# Check versioning for ransomware protection
echo "🔍 Checking S3 bucket versioning..."

for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  versioning=$(aws s3api get-bucket-versioning --bucket "$bucket" --query 'Status' --output text 2>/dev/null)
  mfa_delete=$(aws s3api get-bucket-versioning --bucket "$bucket" --query 'MFADelete' --output text 2>/dev/null)
  
  if [ "$versioning" != "Enabled" ]; then
    echo "⚠️  MEDIUM: Bucket $bucket does NOT have versioning enabled"
    echo "  Risk: No protection against accidental deletion or ransomware"
  else
    if [ "$mfa_delete" != "Enabled" ]; then
      echo "ℹ️  INFO: Bucket $bucket has versioning but MFA Delete is not enabled"
    else
      echo "✓ Bucket $bucket has versioning and MFA Delete enabled"
    fi
  fi
done
```

#### S3 Bucket Logging Status (CIS 2.1.4)
```bash
# Check access logging for compliance
echo "🔍 Checking S3 bucket logging..."

for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  logging=$(aws s3api get-bucket-logging --bucket "$bucket" 2>/dev/null)
  
  if [ "$logging" = "" ] || [ "$logging" = "{}" ]; then
    echo "⚠️  MEDIUM: Bucket $bucket has NO access logging enabled"
    echo "  Impact: No audit trail for bucket access"
  else
    target_bucket=$(echo "$logging" | jq -r '.LoggingEnabled.TargetBucket' 2>/dev/null)
    target_prefix=$(echo "$logging" | jq -r '.LoggingEnabled.TargetPrefix' 2>/dev/null)
    echo "✓ Bucket $bucket logs to $target_bucket with prefix $target_prefix"
  fi
done
```

#### S3 Bucket Lifecycle Policies
```bash
# Check lifecycle policies for cost optimization and data retention
echo "🔍 Checking S3 bucket lifecycle policies..."

for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  lifecycle=$(aws s3api get-bucket-lifecycle-configuration --bucket "$bucket" 2>&1)
  
  if echo "$lifecycle" | grep -q "NoSuchLifecycleConfiguration"; then
    echo "ℹ️  INFO: Bucket $bucket has no lifecycle policy"
    echo "  Consideration: Add lifecycle rules for cost optimization"
  elif echo "$lifecycle" | grep -q "AccessDenied"; then
    echo "ℹ️  INFO: Access denied checking lifecycle for $bucket"
  else
    rule_count=$(echo "$lifecycle" | jq '.Rules | length' 2>/dev/null)
    echo "✓ Bucket $bucket has $rule_count lifecycle rule(s)"
  fi
done
```

#### S3 Bucket Replication Configuration
```bash
# Check cross-region replication for disaster recovery
echo "🔍 Checking S3 bucket replication..."

for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  replication=$(aws s3api get-bucket-replication --bucket "$bucket" 2>&1)
  
  if echo "$replication" | grep -q "ReplicationConfigurationNotFoundError"; then
    echo "ℹ️  INFO: Bucket $bucket has no replication configured"
  elif echo "$replication" | grep -q "AccessDenied"; then
    echo "ℹ️  INFO: Access denied checking replication for $bucket"
  else
    dest_bucket=$(echo "$replication" | jq -r '.ReplicationConfiguration.Rules[0].Destination.Bucket' 2>/dev/null)
    echo "✓ Bucket $bucket replicates to $dest_bucket"
  fi
done
```

#### S3 Bucket CORS Configuration
```bash
# Check CORS for potential security issues
echo "🔍 Checking S3 bucket CORS configuration..."

for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  cors=$(aws s3api get-bucket-cors --bucket "$bucket" 2>&1)
  
  if echo "$cors" | grep -q "NoSuchCORSConfiguration"; then
    echo "✓ Bucket $bucket has no CORS configuration"
  elif echo "$cors" | grep -q "AccessDenied"; then
    echo "ℹ️  INFO: Access denied checking CORS for $bucket"
  else
    # Check for overly permissive CORS
    wildcard_origin=$(echo "$cors" | jq -r '.CORSRules[] | select(.AllowedOrigins[] == "*") | .AllowedMethods' 2>/dev/null)
    
    if [ -n "$wildcard_origin" ]; then
      echo "⚠️  MEDIUM: Bucket $bucket has CORS allowing all origins (*)"
      echo "  Methods: $wildcard_origin"
    fi
  fi
done
```

#### S3 Object Lock (Compliance Mode)
```bash
# Check Object Lock for immutable storage (WORM)
echo "🔍 Checking S3 Object Lock configuration..."

for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  object_lock=$(aws s3api get-object-lock-configuration --bucket "$bucket" 2>&1)
  
  if echo "$object_lock" | grep -q "ObjectLockConfigurationNotFoundError"; then
    echo "ℹ️  INFO: Bucket $bucket does not have Object Lock"
  elif echo "$object_lock" | grep -q "AccessDenied"; then
    echo "ℹ️  INFO: Access denied checking Object Lock for $bucket"
  else
    mode=$(echo "$object_lock" | jq -r '.ObjectLockConfiguration.Rule.DefaultRetention.Mode' 2>/dev/null)
    days=$(echo "$object_lock" | jq -r '.ObjectLockConfiguration.Rule.DefaultRetention.Days' 2>/dev/null)
    echo "✓ Bucket $bucket has Object Lock: Mode=$mode, Days=$days"
  fi
done
```

### 2. EBS Volume Security Audit

#### Check EBS Volume Encryption
```bash
# Find all unencrypted EBS volumes
echo "🔍 Checking EBS volume encryption..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  unencrypted=$(aws ec2 describe-volumes --region "$region" \
    --filters "Name=encrypted,Values=false" \
    --query 'Volumes[*].[VolumeId,Size,State,Attachments[0].InstanceId]' \
    --output text)
  
  if [ -n "$unencrypted" ]; then
    echo "🚨 HIGH: Unencrypted EBS volumes in $region:"
    echo "$unencrypted"
    echo ""
  fi
done
```

#### EBS Encryption by Default Status
```bash
# Check if encryption by default is enabled
echo "🔍 Checking EBS encryption by default..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  encryption_default=$(aws ec2 get-ebs-encryption-by-default --region "$region" --query 'EbsEncryptionByDefault' --output text 2>/dev/null)
  
  if [ "$encryption_default" = "False" ] || [ -z "$encryption_default" ]; then
    echo "⚠️  HIGH: EBS encryption by default is DISABLED in $region"
  else
    default_kms_key=$(aws ec2 get-ebs-default-kms-key-id --region "$region" --query 'KmsKeyId' --output text 2>/dev/null)
    echo "✓ EBS encryption by default enabled in $region (KMS Key: $default_kms_key)"
  fi
done
```

#### EBS Snapshot Encryption and Sharing
```bash
# Check EBS snapshots for encryption and public sharing
echo "🔍 Checking EBS snapshots..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  # Check for unencrypted snapshots
  unencrypted_snapshots=$(aws ec2 describe-snapshots --region "$region" --owner-ids self \
    --filters "Name=encrypted,Values=false" \
    --query 'Snapshots[*].[SnapshotId,VolumeId,VolumeSize,Description]' \
    --output text)
  
  if [ -n "$unencrypted_snapshots" ]; then
    echo "⚠️  HIGH: Unencrypted EBS snapshots in $region:"
    echo "$unencrypted_snapshots"
  fi
  
  # Check for publicly shared snapshots
  public_snapshots=$(aws ec2 describe-snapshots --region "$region" --owner-ids self \
    --restorable-by-user-ids all \
    --query 'Snapshots[*].[SnapshotId,VolumeId,Description]' \
    --output text)
  
  if [ -n "$public_snapshots" ]; then
    echo "🚨 CRITICAL: Publicly shared EBS snapshots in $region:"
    echo "$public_snapshots"
  fi
  
  echo ""
done
```

### 3. EFS (Elastic File System) Security

#### List EFS File Systems
```bash
# Get all EFS file systems
echo "🔍 Checking EFS file systems..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  file_systems=$(aws efs describe-file-systems --region "$region" 2>/dev/null)
  
  if [ -n "$file_systems" ] && [ "$file_systems" != '{"FileSystems":[]}' ]; then
    echo "=== Region: $region ==="
    echo "$file_systems" | jq -r '.FileSystems[] | "\(.FileSystemId) | \(.Name // "Unnamed") | Encrypted: \(.Encrypted)"'
    echo ""
  fi
done
```

#### Check EFS Encryption at Rest
```bash
# Verify EFS encryption
echo "🔍 Checking EFS encryption at rest..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  file_systems=$(aws efs describe-file-systems --region "$region" --query 'FileSystems[*].[FileSystemId,Name,Encrypted,KmsKeyId]' --output text 2>/dev/null)
  
  if [ -n "$file_systems" ]; then
    echo "Region: $region"
    echo "$file_systems" | while read -r fs_id name encrypted kms_key; do
      if [ "$encrypted" = "False" ]; then
        echo "🚨 HIGH: EFS $fs_id ($name) is NOT encrypted at rest"
      else
        echo "✓ EFS $fs_id ($name) encrypted with KMS key: $kms_key"
      fi
    done
    echo ""
  fi
done
```

#### Check EFS Mount Target Security Groups
```bash
# Review security groups attached to EFS mount targets
echo "🔍 Checking EFS mount target security groups..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  file_systems=$(aws efs describe-file-systems --region "$region" --query 'FileSystems[*].FileSystemId' --output text 2>/dev/null)
  
  if [ -n "$file_systems" ]; then
    for fs_id in $file_systems; do
      mount_targets=$(aws efs describe-mount-targets --region "$region" --file-system-id "$fs_id" --query 'MountTargets[*].[MountTargetId,SubnetId]' --output text)
      
      echo "File System: $fs_id"
      echo "$mount_targets" | while read -r mt_id subnet_id; do
        sgs=$(aws efs describe-mount-target-security-groups --region "$region" --mount-target-id "$mt_id" --query 'SecurityGroups[*]' --output text)
        echo "  Mount Target: $mt_id | Subnet: $subnet_id | Security Groups: $sgs"
      done
    done
    echo ""
  fi
done
```

#### Check EFS Access Points
```bash
# Review EFS access points for POSIX user enforcement
echo "🔍 Checking EFS access points..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  file_systems=$(aws efs describe-file-systems --region "$region" --query 'FileSystems[*].FileSystemId' --output text 2>/dev/null)
  
  if [ -n "$file_systems" ]; then
    for fs_id in $file_systems; do
      access_points=$(aws efs describe-access-points --region "$region" --file-system-id "$fs_id" 2>/dev/null)
      
      if [ -n "$access_points" ] && [ "$access_points" != '{"AccessPoints":[]}' ]; then
        echo "File System: $fs_id"
        echo "$access_points" | jq -r '.AccessPoints[] | "  AP: \(.AccessPointId) | POSIX User: \(.PosixUser.Uid // "None"):\(.PosixUser.Gid // "None")"'
        echo ""
      fi
    done
  fi
done
```

### 4. Glacier and Backup Vaults

#### Check AWS Backup Vaults
```bash
# List backup vaults and check encryption
echo "🔍 Checking AWS Backup vaults..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  vaults=$(aws backup list-backup-vaults --region "$region" 2>/dev/null)
  
  if [ -n "$vaults" ] && [ "$vaults" != '{"BackupVaultList":[]}' ]; then
    echo "=== Region: $region ==="
    echo "$vaults" | jq -r '.BackupVaultList[] | "\(.BackupVaultName) | KMS: \(.EncryptionKeyArn // "Not encrypted")"'
    echo ""
  fi
done
```

#### Check Backup Vault Access Policies
```bash
# Review backup vault access policies for security
echo "🔍 Checking AWS Backup vault access policies..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  vaults=$(aws backup list-backup-vaults --region "$region" --query 'BackupVaultList[*].BackupVaultName' --output text 2>/dev/null)
  
  if [ -n "$vaults" ]; then
    for vault in $vaults; do
      policy=$(aws backup get-backup-vault-access-policy --region "$region" --backup-vault-name "$vault" 2>/dev/null)
      
      if [ -n "$policy" ]; then
        echo "Vault: $vault (Region: $region)"
        echo "$policy" | jq '.Policy | fromjson'
        echo "---"
      fi
    done
  fi
done
```

### 5. FSx File Systems

#### List FSx File Systems
```bash
# Check Amazon FSx for Windows/Lustre file systems
echo "🔍 Checking FSx file systems..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  fsx=$(aws fsx describe-file-systems --region "$region" 2>/dev/null)
  
  if [ -n "$fsx" ] && [ "$fsx" != '{"FileSystems":[]}' ]; then
    echo "=== Region: $region ==="
    echo "$fsx" | jq -r '.FileSystems[] | "\(.FileSystemId) | Type: \(.FileSystemType) | Encrypted: \(.KmsKeyId != null)"'
    echo ""
  fi
done
```

---

<!-- analysis: iac -->
## Security Checklist

### Critical (Immediate Action)

- [ ] **No publicly accessible S3 buckets** (CIS 2.1.5)
  - Command: Check bucket ACLs and policies
  - Severity: CRITICAL
  - Finding: Public buckets expose data to internet

- [ ] **No publicly shared EBS snapshots**
  - Command: `aws ec2 describe-snapshots --restorable-by-user-ids all`
  - Severity: CRITICAL
  - Finding: Snapshots may contain sensitive data

### High Priority

- [ ] **All S3 buckets have encryption enabled** (CIS 2.1.1)
  - Command: `aws s3api get-bucket-encryption`
  - Severity: HIGH
  - Finding: Unencrypted data at rest vulnerable to theft

- [ ] **S3 account-level public access block enabled**
  - Command: `aws s3control get-public-access-block`
  - Severity: HIGH
  - Finding: Defense-in-depth control

- [ ] **All EBS volumes encrypted**
  - Command: `aws ec2 describe-volumes --filters "Name=encrypted,Values=false"`
  - Severity: HIGH
  - Finding: Unencrypted volumes expose data

- [ ] **EBS encryption by default enabled**
  - Command: `aws ec2 get-ebs-encryption-by-default`
  - Severity: HIGH
  - Finding: Prevents accidental unencrypted volumes

- [ ] **All EFS file systems encrypted**
  - Command: `aws efs describe-file-systems`
  - Severity: HIGH
  - Finding: Shared file systems often contain sensitive data

### Medium Priority

- [ ] **S3 versioning enabled for critical buckets** (CIS 2.1.3)
  - Command: `aws s3api get-bucket-versioning`
  - Severity: MEDIUM
  - Finding: Protection against ransomware and accidental deletion

- [ ] **S3 access logging enabled** (CIS 2.1.4)
  - Command: `aws s3api get-bucket-logging`
  - Severity: MEDIUM
  - Finding: Audit trail for compliance

- [ ] **S3 lifecycle policies configured**
  - Command: `aws s3api get-bucket-lifecycle-configuration`
  - Severity: LOW
  - Finding: Cost optimization and data retention

### Best Practices

- [ ] **S3 replication for disaster recovery**
  - Command: `aws s3api get-bucket-replication`
  - Severity: INFO
  - Finding: Business continuity

- [ ] **S3 Object Lock for compliance** (WORM)
  - Command: `aws s3api get-object-lock-configuration`
  - Severity: INFO
  - Finding: Regulatory requirements (SEC 17a-4, FINRA)

- [ ] **Backup vaults encrypted with KMS**
  - Command: `aws backup list-backup-vaults`
  - Severity: MEDIUM
  - Finding: Additional layer of protection

---

## Findings Template

**Severity**: CRITICAL | HIGH | MEDIUM | LOW | INFO

**Description**: [Issue details]

**Evidence**:
```bash
[CLI output]
```

**Risk**: [Security/compliance implications]

**Affected Resources**: [List]

**Remediation**:
```bash
[Fix commands]
```

---

## Example Findings

### Example 1: Public S3 Bucket

**Severity**: CRITICAL

**CIS Benchmark**: 2.1.5

**Description**: S3 bucket "company-backups" is publicly accessible via ACL, exposing sensitive backup data to the internet.

**Evidence**:
```bash
Bucket: company-backups
ACL Grantee: http://acs.amazonaws.com/groups/global/AllUsers
Permission: READ
```

**Risk**: 
- Data breach exposure of customer backups
- Regulatory violations (GDPR, HIPAA)
- Potential ransomware encryption of public data

**Affected Resources**:
- Bucket: company-backups
- Estimated size: 500 GB
- Contains: Database backups, application logs

**Remediation**:
```bash
# Remove public ACL
aws s3api put-bucket-acl --bucket company-backups --acl private

# Enable public access block
aws s3api put-public-access-block \
  --bucket company-backups \
  --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Verify
aws s3api get-bucket-acl --bucket company-backups
aws s3api get-public-access-block --bucket company-backups
```

**References**:
- CIS AWS Foundations 2.1.5
- OWASP Cloud Security Top 10 - S1: Accountability and Data Ownership

---

### Example 2: Unencrypted EBS Volume

**Severity**: HIGH

**Description**: Production database server has unencrypted EBS volume, violating data protection requirements.

**Evidence**:
```
Volume: vol-0abc123def456
Size: 100 GB
State: in-use
Instance: i-0db-prod-01
Encrypted: false
```

**Risk**:
- Data at rest not protected
- Compliance violation (PCI DSS 3.4)
- Snapshot copies remain unencrypted

**Affected Resources**:
- Volume: vol-0abc123def456
- Instance: i-0db-prod-01 (PostgreSQL production)

**Remediation**:
```bash
# Create encrypted snapshot
aws ec2 create-snapshot \
  --volume-id vol-0abc123def456 \
  --description "Snapshot before encryption"

# Wait for snapshot completion
aws ec2 wait snapshot-completed --snapshot-ids snap-xxxxx

# Copy snapshot with encryption
aws ec2 copy-snapshot \
  --source-snapshot-id snap-xxxxx \
  --source-region us-east-1 \
  --encrypted \
  --kms-key-id alias/ebs-encryption-key \
  --description "Encrypted snapshot"

# Create encrypted volume from snapshot
aws ec2 create-volume \
  --snapshot-id snap-yyyyy \
  --availability-zone us-east-1a \
  --volume-type gp3 \
  --encrypted

# Attach to instance (requires maintenance window):
# 1. Stop instance
# 2. Detach old volume
# 3. Attach new encrypted volume
# 4. Start instance
```

**References**:
- PCI DSS Requirement 3.4
- CIS AWS Foundations Benchmark

---

## Penetration Testing Perspective

### S3 Bucket Enumeration
```bash
# Common bucket naming patterns
for word in backup backups data prod staging dev test; do
  aws s3 ls s3://company-$word 2>&1
done

# Use tools like bucket_finder, S3Scanner
python3 s3scanner.py --bucket-file common_buckets.txt
```

### S3 Data Exfiltration
```bash
# If bucket allows public LIST
aws s3 ls s3://exposed-bucket --no-sign-request --recursive

# Download all data
aws s3 sync s3://exposed-bucket /tmp/stolen-data --no-sign-request
```

### EBS Snapshot Access
```bash
# Find public snapshots
aws ec2 describe-snapshots --restorable-by-user-ids all --owner-ids <target-account-id>

# Create volume from snapshot
aws ec2 create-volume --snapshot-id snap-xxxxx --availability-zone us-east-1a

# Attach to attacker-controlled instance and mount
aws ec2 attach-volume --volume-id vol-xxxxx --instance-id i-attacker --device /dev/sdf
sudo mount /dev/xvdf /mnt
# Access sensitive data
```

---

## Compliance Mapping

| CIS Control | Requirement | Priority |
|-------------|-------------|----------|
| 2.1.1 | Ensure all S3 buckets employ encryption-at-rest | HIGH |
| 2.1.2 | Ensure S3 bucket policy is set to deny HTTP requests | MEDIUM |
| 2.1.3 | Ensure MFA Delete is enabled on S3 buckets | MEDIUM |
| 2.1.4 | Ensure all data in S3 has been discovered, classified, and secured | INFO |
| 2.1.5 | Ensure that S3 buckets are configured with Block Public Access | CRITICAL |
| 2.2.1 | Ensure EBS volume encryption is enabled | HIGH |
| 2.3.1 | Ensure that encryption is enabled for RDS instances | HIGH |

---

## Summary Report Template

**Audit Date**: [Date]  
**Auditor**: [Name]  
**Scope**: [Account/Regions]

### Findings Summary
| Severity | Count |
|----------|-------|
| CRITICAL | X |
| HIGH | X |
| MEDIUM | X |

### Storage Inventory
- S3 Buckets: X (X encrypted, X with versioning, X with logging)
- EBS Volumes: X (X encrypted)
- EBS Snapshots: X (X public)
- EFS File Systems: X (X encrypted)
- Backup Vaults: X

### Top Recommendations
1. Block public access to all S3 buckets
2. Enable S3 encryption by default
3. Enable EBS encryption by default
4. Enable S3 versioning for critical buckets
5. Implement S3 lifecycle policies

---

**Next Steps**: Remediate critical findings immediately, high findings within 7 days.
