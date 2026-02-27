---
genre: hosting
category: database-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/cloudformation/**"
  keywords:
    - aws_rds
    - aws_dynamodb
    - aws_elasticache
    - aws_redshift
    - rds_cluster
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Database Security Audit Template

**Purpose**: Assess AWS database security including RDS, DynamoDB, Redshift, DocumentDB, and ElastiCache configurations for encryption, access controls, and data protection.

**Frameworks**: CIS AWS Foundations Benchmark 2.3.x, AWS Well-Architected Security Pillar - Data Protection

---

## Agentic Prompt Examples

### Quick Database Security Audit
```
@terminal Audit AWS database security: check all RDS instances for encryption, 
public accessibility, backup retention, multi-AZ configuration, and automatic 
minor version upgrades. Review DynamoDB tables for encryption and point-in-time 
recovery. Flag any publicly accessible databases as CRITICAL.
```

### Deep Database Assessment
```
@terminal Perform comprehensive database security audit: analyze RDS encryption 
at rest and in transit, audit database parameter groups for SSL enforcement, 
check IAM database authentication, review security group rules, verify backup 
configurations, and assess Redshift cluster security. Document compliance gaps 
and generate remediation plan.
```

### Database Access Control Audit
```
@terminal Review database access controls: check RDS security groups for overly 
permissive rules, verify no databases allow 0.0.0.0/0 access, audit database 
user permissions, review IAM policies granting database access, and check for 
proper network segmentation of data tier resources.
```

---

## AWS CLI Commands

### 1. RDS (Relational Database Service) Audit

#### List All RDS Instances
```bash
# Get all RDS instances across regions
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "=== Region: $region ==="
  aws rds describe-db-instances --region "$region" \
    --query 'DBInstances[*].[DBInstanceIdentifier,Engine,EngineVersion,DBInstanceClass,DBInstanceStatus,MultiAZ,PubliclyAccessible,StorageEncrypted]' \
    --output table
  echo ""
done
```

#### Check RDS Public Accessibility (CRITICAL)
```bash
# Find publicly accessible RDS instances
echo "🔍 Checking for publicly accessible RDS instances..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  public_dbs=$(aws rds describe-db-instances --region "$region" \
    --query 'DBInstances[?PubliclyAccessible==`true`].[DBInstanceIdentifier,Engine,Endpoint.Address,DBSubnetGroup.VpcId]' \
    --output text)
  
  if [ -n "$public_dbs" ]; then
    echo "🚨 CRITICAL: Publicly accessible RDS instances in $region:"
    echo "$public_dbs"
    echo ""
  fi
done
```

#### Check RDS Encryption at Rest (CIS 2.3.1)
```bash
# Find unencrypted RDS instances
echo "🔍 Checking RDS encryption at rest..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  unencrypted_dbs=$(aws rds describe-db-instances --region "$region" \
    --query 'DBInstances[?StorageEncrypted==`false`].[DBInstanceIdentifier,Engine,DBInstanceClass,AllocatedStorage]' \
    --output text)
  
  if [ -n "$unencrypted_dbs" ]; then
    echo "🚨 HIGH: Unencrypted RDS instances in $region:"
    echo "$unencrypted_dbs"
    echo ""
  fi
done
```

#### Check RDS Backup Retention
```bash
# Verify backup retention periods (should be >= 7 days)
echo "🔍 Checking RDS backup retention..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  instances=$(aws rds describe-db-instances --region "$region" \
    --query 'DBInstances[*].[DBInstanceIdentifier,BackupRetentionPeriod,Engine]' \
    --output text)
  
  if [ -n "$instances" ]; then
    echo "Region: $region"
    echo "$instances" | while read -r db_id retention engine; do
      if [ "$retention" -lt 7 ]; then
        echo "⚠️  MEDIUM: RDS instance $db_id ($engine) has backup retention < 7 days: $retention"
      else
        echo "✓ RDS instance $db_id has $retention days backup retention"
      fi
    done
    echo ""
  fi
done
```

#### Check RDS Multi-AZ Configuration
```bash
# Verify high availability configuration
echo "🔍 Checking RDS Multi-AZ configuration..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  single_az=$(aws rds describe-db-instances --region "$region" \
    --query 'DBInstances[?MultiAZ==`false`].[DBInstanceIdentifier,Engine,DBInstanceClass]' \
    --output text)
  
  if [ -n "$single_az" ]; then
    echo "⚠️  MEDIUM: Single-AZ RDS instances in $region (no HA):"
    echo "$single_az"
    echo ""
  fi
done
```

#### Check RDS Auto Minor Version Upgrade
```bash
# Verify automatic patching is enabled
echo "🔍 Checking RDS auto minor version upgrade..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  instances=$(aws rds describe-db-instances --region "$region" \
    --query 'DBInstances[?AutoMinorVersionUpgrade==`false`].[DBInstanceIdentifier,Engine,EngineVersion]' \
    --output text)
  
  if [ -n "$instances" ]; then
    echo "⚠️  MEDIUM: RDS instances without auto minor version upgrade in $region:"
    echo "$instances"
    echo "  Risk: Missing security patches"
    echo ""
  fi
done
```

#### Check RDS Parameter Groups (SSL/TLS Enforcement)
```bash
# Check if SSL/TLS is enforced for database connections
echo "🔍 Checking RDS SSL/TLS enforcement..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  instances=$(aws rds describe-db-instances --region "$region" --query 'DBInstances[*].[DBInstanceIdentifier,Engine,DBParameterGroups[0].DBParameterGroupName]' --output text)
  
  if [ -n "$instances" ]; then
    echo "Region: $region"
    echo "$instances" | while read -r db_id engine param_group; do
      # Check parameter based on engine type
      case $engine in
        postgres*)
          ssl_param=$(aws rds describe-db-parameters --region "$region" --db-parameter-group-name "$param_group" --query "Parameters[?ParameterName=='rds.force_ssl'].ParameterValue" --output text)
          if [ "$ssl_param" != "1" ]; then
            echo "⚠️  HIGH: PostgreSQL instance $db_id does NOT enforce SSL (rds.force_ssl != 1)"
          fi
          ;;
        mysql*|mariadb*)
          ssl_param=$(aws rds describe-db-parameters --region "$region" --db-parameter-group-name "$param_group" --query "Parameters[?ParameterName=='require_secure_transport'].ParameterValue" --output text)
          if [ "$ssl_param" != "1" ] && [ "$ssl_param" != "ON" ]; then
            echo "⚠️  HIGH: MySQL/MariaDB instance $db_id does NOT enforce SSL (require_secure_transport != 1)"
          fi
          ;;
      esac
    done
    echo ""
  fi
done
```

#### Check RDS IAM Database Authentication
```bash
# Check if IAM database authentication is enabled
echo "🔍 Checking RDS IAM database authentication..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  instances=$(aws rds describe-db-instances --region "$region" \
    --query 'DBInstances[*].[DBInstanceIdentifier,Engine,IAMDatabaseAuthenticationEnabled]' \
    --output text)
  
  if [ -n "$instances" ]; then
    echo "Region: $region"
    echo "$instances" | while read -r db_id engine iam_auth; do
      if [ "$iam_auth" = "False" ]; then
        echo "ℹ️  INFO: RDS instance $db_id ($engine) does not use IAM database authentication"
        echo "  Benefit: Centralized access control, no hardcoded passwords"
      fi
    done
    echo ""
  fi
done
```

#### Check RDS Enhanced Monitoring
```bash
# Verify enhanced monitoring is enabled
echo "🔍 Checking RDS enhanced monitoring..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  instances=$(aws rds describe-db-instances --region "$region" \
    --query 'DBInstances[?MonitoringInterval==`0` || MonitoringInterval==`null`].[DBInstanceIdentifier,Engine]' \
    --output text)
  
  if [ -n "$instances" ]; then
    echo "ℹ️  INFO: RDS instances without enhanced monitoring in $region:"
    echo "$instances"
    echo ""
  fi
done
```

#### Check RDS Deletion Protection
```bash
# Check if deletion protection is enabled
echo "🔍 Checking RDS deletion protection..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  instances=$(aws rds describe-db-instances --region "$region" \
    --query 'DBInstances[?DeletionProtection==`false`].[DBInstanceIdentifier,Engine,DBInstanceClass]' \
    --output text)
  
  if [ -n "$instances" ]; then
    echo "⚠️  MEDIUM: RDS instances without deletion protection in $region:"
    echo "$instances"
    echo "  Risk: Accidental deletion possible"
    echo ""
  fi
done
```

### 2. RDS Snapshots Audit

#### Check RDS Snapshot Encryption
```bash
# Find unencrypted RDS snapshots
echo "🔍 Checking RDS snapshot encryption..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  unencrypted_snapshots=$(aws rds describe-db-snapshots --region "$region" \
    --snapshot-type manual \
    --query 'DBSnapshots[?Encrypted==`false`].[DBSnapshotIdentifier,DBInstanceIdentifier,SnapshotCreateTime]' \
    --output text)
  
  if [ -n "$unencrypted_snapshots" ]; then
    echo "⚠️  HIGH: Unencrypted RDS snapshots in $region:"
    echo "$unencrypted_snapshots"
    echo ""
  fi
done
```

#### Check RDS Snapshot Public Access
```bash
# Find publicly shared RDS snapshots (CRITICAL)
echo "🔍 Checking for publicly accessible RDS snapshots..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  public_snapshots=$(aws rds describe-db-snapshot-attributes --region "$region" \
    --db-snapshot-identifier <snapshot-id> \
    --query 'DBSnapshotAttributesResult.DBSnapshotAttributes[?AttributeName==`restore`].AttributeValues[]' \
    --output text 2>/dev/null | grep -i "all")
  
  # This requires iterating through all snapshots - use script for full audit
  # Public snapshots have "all" in restore attribute
done
```

### 3. DynamoDB Audit

#### List All DynamoDB Tables
```bash
# Get all DynamoDB tables across regions
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  tables=$(aws dynamodb list-tables --region "$region" --query 'TableNames[*]' --output text)
  
  if [ -n "$tables" ]; then
    echo "=== Region: $region ==="
    echo "$tables"
    echo ""
  fi
done
```

#### Check DynamoDB Encryption at Rest
```bash
# Verify DynamoDB table encryption
echo "🔍 Checking DynamoDB encryption at rest..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  tables=$(aws dynamodb list-tables --region "$region" --query 'TableNames[*]' --output text)
  
  if [ -n "$tables" ]; then
    echo "Region: $region"
    for table in $tables; do
      encryption=$(aws dynamodb describe-table --region "$region" --table-name "$table" --query 'Table.SSEDescription.Status' --output text)
      kms_key=$(aws dynamodb describe-table --region "$region" --table-name "$table" --query 'Table.SSEDescription.KMSMasterKeyArn' --output text)
      
      if [ "$encryption" != "ENABLED" ]; then
        echo "⚠️  MEDIUM: DynamoDB table $table is NOT encrypted at rest"
      else
        echo "✓ DynamoDB table $table encrypted (KMS: ${kms_key:-AWS managed})"
      fi
    done
    echo ""
  fi
done
```

#### Check DynamoDB Point-in-Time Recovery
```bash
# Verify PITR for backup protection
echo "🔍 Checking DynamoDB point-in-time recovery..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  tables=$(aws dynamodb list-tables --region "$region" --query 'TableNames[*]' --output text)
  
  if [ -n "$tables" ]; then
    echo "Region: $region"
    for table in $tables; do
      pitr=$(aws dynamodb describe-continuous-backups --region "$region" --table-name "$table" --query 'ContinuousBackupsDescription.PointInTimeRecoveryDescription.PointInTimeRecoveryStatus' --output text)
      
      if [ "$pitr" != "ENABLED" ]; then
        echo "⚠️  MEDIUM: DynamoDB table $table does NOT have point-in-time recovery"
        echo "  Risk: No protection against accidental data deletion/corruption"
      else
        echo "✓ DynamoDB table $table has PITR enabled"
      fi
    done
    echo ""
  fi
done
```

#### Check DynamoDB VPC Endpoints
```bash
# Verify private access via VPC endpoints
echo "🔍 Checking DynamoDB VPC endpoints..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  endpoints=$(aws ec2 describe-vpc-endpoints --region "$region" \
    --filters "Name=service-name,Values=com.amazonaws.$region.dynamodb" \
    --query 'VpcEndpoints[*].[VpcEndpointId,VpcId,State]' \
    --output text)
  
  if [ -z "$endpoints" ]; then
    echo "ℹ️  INFO: No DynamoDB VPC endpoints in $region"
    echo "  Benefit: Private access without internet gateway"
  else
    echo "✓ DynamoDB VPC endpoints in $region:"
    echo "$endpoints"
  fi
  echo ""
done
```

### 4. Redshift Audit

#### List Redshift Clusters
```bash
# Get all Redshift clusters
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  clusters=$(aws redshift describe-clusters --region "$region" 2>/dev/null)
  
  if [ -n "$clusters" ] && [ "$clusters" != '{"Clusters":[]}' ]; then
    echo "=== Region: $region ==="
    echo "$clusters" | jq -r '.Clusters[] | "\(.ClusterIdentifier) | \(.NodeType) | Public: \(.PubliclyAccessible) | Encrypted: \(.Encrypted)"'
    echo ""
  fi
done
```

#### Check Redshift Public Accessibility
```bash
# Find publicly accessible Redshift clusters (CRITICAL)
echo "🔍 Checking for publicly accessible Redshift clusters..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  public_clusters=$(aws redshift describe-clusters --region "$region" \
    --query 'Clusters[?PubliclyAccessible==`true`].[ClusterIdentifier,Endpoint.Address,VpcId]' \
    --output text)
  
  if [ -n "$public_clusters" ]; then
    echo "🚨 CRITICAL: Publicly accessible Redshift clusters in $region:"
    echo "$public_clusters"
    echo "  Risk: Data warehouse exposed to internet"
    echo ""
  fi
done
```

#### Check Redshift Encryption
```bash
# Verify Redshift cluster encryption
echo "🔍 Checking Redshift encryption..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  unencrypted=$(aws redshift describe-clusters --region "$region" \
    --query 'Clusters[?Encrypted==`false`].[ClusterIdentifier,NodeType,NumberOfNodes]' \
    --output text)
  
  if [ -n "$unencrypted" ]; then
    echo "🚨 HIGH: Unencrypted Redshift clusters in $region:"
    echo "$unencrypted"
    echo ""
  fi
done
```

#### Check Redshift Audit Logging
```bash
# Verify audit logging is enabled
echo "🔍 Checking Redshift audit logging..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  clusters=$(aws redshift describe-clusters --region "$region" --query 'Clusters[*].ClusterIdentifier' --output text)
  
  if [ -n "$clusters" ]; then
    for cluster in $clusters; do
      logging=$(aws redshift describe-logging-status --region "$region" --cluster-identifier "$cluster" --query 'LoggingEnabled' --output text)
      
      if [ "$logging" != "True" ]; then
        echo "⚠️  MEDIUM: Redshift cluster $cluster (region: $region) does NOT have audit logging enabled"
      else
        bucket=$(aws redshift describe-logging-status --region "$region" --cluster-identifier "$cluster" --query 'BucketName' --output text)
        echo "✓ Redshift cluster $cluster logs to S3 bucket: $bucket"
      fi
    done
  fi
done
```

### 5. ElastiCache Audit

#### List ElastiCache Clusters
```bash
# Get Redis and Memcached clusters
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "=== Region: $region ==="
  
  # Redis clusters
  redis=$(aws elasticache describe-replication-groups --region "$region" 2>/dev/null)
  if [ -n "$redis" ] && [ "$redis" != '{"ReplicationGroups":[]}' ]; then
    echo "Redis Clusters:"
    echo "$redis" | jq -r '.ReplicationGroups[] | "\(.ReplicationGroupId) | \(.Status) | Transit Encryption: \(.TransitEncryptionEnabled) | At-Rest Encryption: \(.AtRestEncryptionEnabled)"'
  fi
  
  # Memcached clusters
  memcached=$(aws elasticache describe-cache-clusters --region "$region" --query 'CacheClusters[?Engine==`memcached`].[CacheClusterId,CacheNodeType,Engine]' --output text)
  if [ -n "$memcached" ]; then
    echo "Memcached Clusters:"
    echo "$memcached"
  fi
  
  echo ""
done
```

#### Check ElastiCache Encryption
```bash
# Verify Redis encryption (at-rest and in-transit)
echo "🔍 Checking ElastiCache encryption..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  clusters=$(aws elasticache describe-replication-groups --region "$region" 2>/dev/null)
  
  if [ -n "$clusters" ] && [ "$clusters" != '{"ReplicationGroups":[]}' ]; then
    echo "Region: $region"
    echo "$clusters" | jq -r '.ReplicationGroups[] | select(.TransitEncryptionEnabled == false or .AtRestEncryptionEnabled == false) | "\(.ReplicationGroupId) - Transit: \(.TransitEncryptionEnabled), At-Rest: \(.AtRestEncryptionEnabled)"' | while read line; do
      echo "⚠️  HIGH: ElastiCache Redis cluster with encryption disabled: $line"
    done
    echo ""
  fi
done
```

### 6. DocumentDB and Neptune

#### List DocumentDB Clusters
```bash
# Check DocumentDB (MongoDB-compatible) clusters
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  docdb=$(aws docdb describe-db-clusters --region "$region" 2>/dev/null)
  
  if [ -n "$docdb" ] && [ "$docdb" != '{"DBClusters":[]}' ]; then
    echo "=== Region: $region - DocumentDB ==="
    echo "$docdb" | jq -r '.DBClusters[] | "\(.DBClusterIdentifier) | Encrypted: \(.StorageEncrypted) | Backup Retention: \(.BackupRetentionPeriod) days"'
    echo ""
  fi
done
```

---

<!-- analysis: iac -->
## Security Checklist

### Critical

- [ ] **No publicly accessible RDS instances**
  - Severity: CRITICAL
  - Risk: Direct database access from internet

- [ ] **No publicly accessible Redshift clusters**
  - Severity: CRITICAL
  - Risk: Data warehouse exposure

- [ ] **No publicly shared RDS/Redshift snapshots**
  - Severity: CRITICAL
  - Risk: Data breach via snapshot access

### High Priority

- [ ] **All RDS instances encrypted** (CIS 2.3.1)
  - Severity: HIGH
  - Risk: Unencrypted data at rest

- [ ] **All Redshift clusters encrypted**
  - Severity: HIGH
  - Risk: Data warehouse without encryption

- [ ] **SSL/TLS enforced for database connections**
  - Severity: HIGH
  - Risk: Credentials/data transmitted in clear text

- [ ] **RDS backup retention >= 7 days**
  - Severity: MEDIUM
  - Risk: Insufficient disaster recovery capability

### Medium Priority

- [ ] **RDS Multi-AZ enabled for production**
  - Severity: MEDIUM
  - Risk: Single point of failure

- [ ] **DynamoDB point-in-time recovery enabled**
  - Severity: MEDIUM
  - Risk: No protection against data corruption

- [ ] **ElastiCache encryption enabled**
  - Severity: HIGH
  - Risk: Cached sensitive data unencrypted

- [ ] **RDS deletion protection enabled**
  - Severity: MEDIUM
  - Risk: Accidental deletion

### Best Practices

- [ ] **IAM database authentication used**
  - Severity: INFO
  - Benefit: No hardcoded passwords

- [ ] **RDS enhanced monitoring enabled**
  - Severity: INFO
  - Benefit: Better performance insights

- [ ] **Redshift audit logging enabled**
  - Severity: MEDIUM
  - Benefit: Compliance and forensics

---

## Summary Report Template

**Audit Date**: [Date]  
**Auditor**: [Name]

### Findings
| Severity | Count |
|----------|-------|
| CRITICAL | X |
| HIGH | X |
| MEDIUM | X |

### Database Inventory
- RDS Instances: X (X public, X unencrypted)
- DynamoDB Tables: X
- Redshift Clusters: X
- ElastiCache Clusters: X

### Top Recommendations
1. Remove public accessibility from all databases
2. Enable encryption for all database services
3. Enforce SSL/TLS for connections
4. Enable backup/PITR for all databases

---

**Next Steps**: Immediate remediation of critical findings.
