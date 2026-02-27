---
genre: hosting
category: compute-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/cloudformation/**"
    - "**/ecs/**"
    - "**/lambda/**"
  keywords:
    - aws_instance
    - aws_ecs
    - aws_lambda
    - aws_autoscaling
    - ec2
    - fargate
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Compute Security Audit Template

**Purpose**: Assess security of AWS compute services including EC2, Lambda, ECS, EKS, and related configurations.

**Frameworks**: CIS AWS Foundations Benchmark, AWS Well-Architected Security Pillar - Compute Protection

---

## Agentic Prompt Examples

### Quick Compute Security Audit
```
@terminal Audit AWS compute security: check EC2 instances for IMDSv2 enforcement, 
review Lambda function permissions and VPC configuration, assess ECS task security 
settings, verify no instances have public IPs in private subnets, and check for 
unencrypted EBS volumes. Document findings with severity ratings.
```

### Deep Compute Assessment
```
@terminal Perform comprehensive compute security assessment: analyze all EC2 instances 
for security group misconfigurations, AMI vulnerabilities, and patch compliance. Review 
Lambda execution roles for over-permissive policies. Audit ECS/EKS clusters for container 
security best practices. Check for exposed instance metadata service and privileged 
container configurations.
```

### Container Security Audit
```
@terminal Audit container workloads: review ECS task definitions for privileged mode, 
check EKS pod security policies, analyze container image sources, verify secrets 
management practices, and assess network policies. Flag any containers running as root 
or with elevated privileges.
```

---

## AWS CLI Commands

### 1. EC2 Instances Audit

#### List All EC2 Instances
```bash
# Get all EC2 instances across regions
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "=== Region: $region ==="
  aws ec2 describe-instances --region "$region" \
    --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name,PublicIpAddress,PrivateIpAddress,Tags[?Key==`Name`].Value|[0]]' \
    --output table
  echo ""
done
```

#### Check Instance Metadata Service Version (IMDSv2)
```bash
# IMDSv2 should be required for security (CIS benchmark)
echo "🔍 Checking Instance Metadata Service configuration..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  instances=$(aws ec2 describe-instances --region "$region" \
    --filters "Name=instance-state-name,Values=running" \
    --query 'Reservations[*].Instances[*].[InstanceId,MetadataOptions.HttpTokens,MetadataOptions.HttpPutResponseHopLimit,Tags[?Key==`Name`].Value|[0]]' \
    --output text)
  
  if [ -n "$instances" ]; then
    echo "Region: $region"
    echo "$instances" | while read -r instance_id http_tokens hop_limit name; do
      if [ "$http_tokens" != "required" ]; then
        echo "🚨 HIGH: Instance $instance_id ($name) allows IMDSv1 (HttpTokens: $http_tokens)"
        echo "  Risk: IMDSv1 vulnerable to SSRF attacks (CVE-2019-5736)"
        echo "  Remediation: aws ec2 modify-instance-metadata-options --instance-id $instance_id --http-tokens required --http-endpoint enabled"
      fi
      
      if [ "$hop_limit" -gt 1 ]; then
        echo "⚠️  MEDIUM: Instance $instance_id has HttpPutResponseHopLimit > 1 ($hop_limit)"
        echo "  Risk: Metadata accessible from containers (potential container escape)"
      fi
    done
    echo ""
  fi
done
```

#### Public EC2 Instances in Private Subnets
```bash
# Find instances with public IPs that shouldn't have them
echo "🔍 Checking for instances with unexpected public IPs..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  instances_with_public_ip=$(aws ec2 describe-instances --region "$region" \
    --filters "Name=instance-state-name,Values=running" \
    --query 'Reservations[*].Instances[?PublicIpAddress!=null].[InstanceId,PublicIpAddress,SubnetId,Tags[?Key==`Name`].Value|[0]]' \
    --output text)
  
  if [ -n "$instances_with_public_ip" ]; then
    echo "Region: $region - Instances with public IPs:"
    echo "$instances_with_public_ip" | while read -r instance_id public_ip subnet_id name; do
      # Check if subnet name indicates it should be private
      subnet_name=$(aws ec2 describe-subnets --region "$region" --subnet-ids "$subnet_id" --query 'Subnets[0].Tags[?Key==`Name`].Value|[0]' --output text)
      
      if [[ "$subnet_name" =~ [Pp]rivate ]]; then
        echo "⚠️  MEDIUM: Instance $instance_id ($name) has public IP $public_ip in private subnet $subnet_id"
      fi
    done
    echo ""
  fi
done
```

#### EBS Volume Encryption Status
```bash
# Check for unencrypted EBS volumes
echo "🔍 Checking EBS volume encryption status..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  unencrypted_volumes=$(aws ec2 describe-volumes --region "$region" \
    --filters "Name=encrypted,Values=false" \
    --query 'Volumes[*].[VolumeId,Size,State,Attachments[0].InstanceId,Tags[?Key==`Name`].Value|[0]]' \
    --output text)
  
  if [ -n "$unencrypted_volumes" ]; then
    echo "🚨 HIGH: Unencrypted EBS volumes in $region:"
    echo "$unencrypted_volumes"
    echo ""
  fi
done
```

#### EBS Encryption by Default
```bash
# Check if EBS encryption is enabled by default per region
echo "🔍 Checking EBS encryption by default setting..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  ebs_encryption_default=$(aws ec2 get-ebs-encryption-by-default --region "$region" --query 'EbsEncryptionByDefault' --output text 2>/dev/null)
  
  if [ "$ebs_encryption_default" = "False" ] || [ -z "$ebs_encryption_default" ]; then
    echo "⚠️  MEDIUM: EBS encryption by default is DISABLED in region $region"
    echo "  Remediation: aws ec2 enable-ebs-encryption-by-default --region $region"
  else
    echo "✓ EBS encryption by default is enabled in $region"
  fi
done
```

#### EC2 AMI Age and Patch Status
```bash
# Find instances running old AMIs (potential security patches missing)
echo "🔍 Checking EC2 AMI age..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  instances=$(aws ec2 describe-instances --region "$region" \
    --filters "Name=instance-state-name,Values=running" \
    --query 'Reservations[*].Instances[*].[InstanceId,ImageId,LaunchTime,Tags[?Key==`Name`].Value|[0]]' \
    --output text)
  
  if [ -n "$instances" ]; then
    echo "Region: $region"
    echo "$instances" | while read -r instance_id ami_id launch_time name; do
      # Get AMI creation date
      ami_creation=$(aws ec2 describe-images --region "$region" --image-ids "$ami_id" --query 'Images[0].CreationDate' --output text 2>/dev/null)
      
      if [ -n "$ami_creation" ]; then
        ami_age_days=$(( ( $(date +%s) - $(date -d "$ami_creation" +%s) ) / 86400 ))
        
        if [ "$ami_age_days" -gt 90 ]; then
          echo "⚠️  MEDIUM: Instance $instance_id ($name) uses AMI >90 days old (AMI: $ami_id, Age: $ami_age_days days)"
        fi
      fi
    done
    echo ""
  fi
done
```

#### EC2 Systems Manager Compliance
```bash
# Check if instances are managed by Systems Manager
echo "🔍 Checking Systems Manager agent status..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  # Get running instances
  running_instances=$(aws ec2 describe-instances --region "$region" \
    --filters "Name=instance-state-name,Values=running" \
    --query 'Reservations[*].Instances[*].InstanceId' \
    --output text)
  
  if [ -n "$running_instances" ]; then
    # Get managed instances
    managed_instances=$(aws ssm describe-instance-information --region "$region" \
      --query 'InstanceInformationList[*].InstanceId' \
      --output text 2>/dev/null)
    
    echo "Region: $region"
    echo "Running Instances: $(echo $running_instances | wc -w)"
    echo "Managed by SSM: $(echo $managed_instances | wc -w)"
    
    # Find unmanaged instances
    for instance in $running_instances; do
      if ! echo "$managed_instances" | grep -q "$instance"; then
        instance_name=$(aws ec2 describe-instances --region "$region" --instance-ids "$instance" --query 'Reservations[0].Instances[0].Tags[?Key==`Name`].Value|[0]' --output text)
        echo "⚠️  MEDIUM: Instance $instance ($instance_name) not managed by Systems Manager"
        echo "  Impact: No automated patching, compliance scanning, or secure shell access"
      fi
    done
    echo ""
  fi
done
```

### 2. Lambda Functions Audit

#### List All Lambda Functions
```bash
# Get all Lambda functions across regions
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "=== Region: $region ==="
  aws lambda list-functions --region "$region" \
    --query 'Functions[*].[FunctionName,Runtime,Role,VpcConfig.VpcId]' \
    --output table
  echo ""
done
```

#### Lambda Functions Not in VPC
```bash
# Find Lambda functions without VPC configuration (may have unrestricted internet access)
echo "🔍 Checking Lambda VPC configuration..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  functions_no_vpc=$(aws lambda list-functions --region "$region" \
    --query 'Functions[?VpcConfig==null || VpcConfig.VpcId==null].[FunctionName,Runtime]' \
    --output text)
  
  if [ -n "$functions_no_vpc" ]; then
    echo "ℹ️  INFO: Lambda functions NOT in VPC (region $region):"
    echo "$functions_no_vpc"
    echo "  Note: Functions outside VPC have internet access by default"
    echo "  Consideration: Place in VPC if accessing internal resources"
    echo ""
  fi
done
```

#### Lambda Function Permissions (Resource-Based Policies)
```bash
# Check Lambda function policies for overly permissive access
echo "🔍 Checking Lambda function permissions..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  functions=$(aws lambda list-functions --region "$region" --query 'Functions[*].FunctionName' --output text)
  
  if [ -n "$functions" ]; then
    echo "Region: $region"
    for func in $functions; do
      policy=$(aws lambda get-policy --region "$region" --function-name "$func" --query 'Policy' --output text 2>/dev/null)
      
      if [ -n "$policy" ] && [ "$policy" != "None" ]; then
        # Check for wildcard principals
        public_access=$(echo "$policy" | jq -r '.Statement[] | select(.Principal == "*" or .Principal.AWS == "*" or .Principal.Service == "*") | .Effect' 2>/dev/null)
        
        if [ "$public_access" = "Allow" ]; then
          echo "🚨 CRITICAL: Function $func allows invocation from any AWS account (*)"
          echo "  Policy: $policy"
        fi
      fi
    done
    echo ""
  fi
done
```

#### Lambda Execution Role Permissions
```bash
# Check Lambda execution roles for over-permissive policies
echo "🔍 Checking Lambda execution role permissions..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  functions=$(aws lambda list-functions --region "$region" --query 'Functions[*].[FunctionName,Role]' --output text)
  
  if [ -n "$functions" ]; then
    echo "Region: $region"
    echo "$functions" | while read -r func_name role_arn; do
      role_name=$(echo "$role_arn" | awk -F'/' '{print $NF}')
      
      # Get attached policies
      admin_policy=$(aws iam list-attached-role-policies --role-name "$role_name" \
        --query 'AttachedPolicies[?PolicyName==`AdministratorAccess`]' --output text 2>/dev/null)
      
      if [ -n "$admin_policy" ]; then
        echo "🚨 HIGH: Lambda function $func_name has AdministratorAccess via role $role_name"
        echo "  Violates least privilege principle"
      fi
      
      # Check for wildcard inline policies
      inline_policies=$(aws iam list-role-policies --role-name "$role_name" --query 'PolicyNames[*]' --output text 2>/dev/null)
      for inline_policy in $inline_policies; do
        policy_doc=$(aws iam get-role-policy --role-name "$role_name" --policy-name "$inline_policy" --query 'PolicyDocument' --output json 2>/dev/null)
        wildcards=$(echo "$policy_doc" | jq -r '.Statement[] | select(.Effect=="Allow" and (.Action=="*" or .Resource=="*"))' 2>/dev/null)
        
        if [ -n "$wildcards" ]; then
          echo "⚠️  HIGH: Lambda function $func_name has wildcard permissions in inline policy $inline_policy"
        fi
      done
    done
    echo ""
  fi
done
```

#### Lambda Runtime Versions (Deprecated Runtimes)
```bash
# Find Lambda functions using deprecated or EOL runtimes
echo "🔍 Checking Lambda runtime versions..."

deprecated_runtimes=("python2.7" "python3.6" "nodejs10.x" "nodejs12.x" "dotnetcore2.1" "ruby2.5" "go1.x")

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  for runtime in "${deprecated_runtimes[@]}"; do
    functions=$(aws lambda list-functions --region "$region" \
      --query "Functions[?Runtime=='$runtime'].[FunctionName,Runtime]" \
      --output text)
    
    if [ -n "$functions" ]; then
      echo "⚠️  HIGH: Functions using deprecated runtime $runtime in $region:"
      echo "$functions"
      echo ""
    fi
  done
done
```

#### Lambda Environment Variables (Secrets Check)
```bash
# Warn about environment variables (secrets should be in Secrets Manager)
echo "🔍 Checking Lambda environment variables..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  functions=$(aws lambda list-functions --region "$region" --query 'Functions[*].FunctionName' --output text)
  
  if [ -n "$functions" ]; then
    for func in $functions; do
      env_vars=$(aws lambda get-function-configuration --region "$region" --function-name "$func" --query 'Environment.Variables' --output json 2>/dev/null)
      
      if [ "$env_vars" != "null" ] && [ "$env_vars" != "{}" ]; then
        # Check for suspicious keywords in variable names
        suspicious=$(echo "$env_vars" | jq -r 'keys[] | select(. | test("PASSWORD|SECRET|KEY|TOKEN"; "i"))' 2>/dev/null)
        
        if [ -n "$suspicious" ]; then
          echo "⚠️  MEDIUM: Function $func (region: $region) has suspicious environment variable names: $suspicious"
          echo "  Recommendation: Use AWS Secrets Manager or Parameter Store instead"
        fi
      fi
    done
  fi
done
```

### 3. ECS (Elastic Container Service) Audit

#### List ECS Clusters
```bash
# Get all ECS clusters
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  clusters=$(aws ecs list-clusters --region "$region" --query 'clusterArns[*]' --output text)
  
  if [ -n "$clusters" ]; then
    echo "=== Region: $region ==="
    for cluster_arn in $clusters; do
      cluster_name=$(echo "$cluster_arn" | awk -F'/' '{print $NF}')
      echo "Cluster: $cluster_name"
      
      # Get cluster details
      aws ecs describe-clusters --region "$region" --clusters "$cluster_arn" \
        --query 'clusters[0].[clusterName,status,runningTasksCount,activeServicesCount]' \
        --output table
    done
    echo ""
  fi
done
```

#### ECS Task Definitions - Privileged Containers
```bash
# Find task definitions with privileged containers (container escape risk)
echo "🔍 Checking ECS task definitions for privileged containers..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  task_def_arns=$(aws ecs list-task-definitions --region "$region" --status ACTIVE --query 'taskDefinitionArns[*]' --output text)
  
  if [ -n "$task_def_arns" ]; then
    echo "Region: $region"
    for task_def_arn in $task_def_arns; do
      privileged=$(aws ecs describe-task-definition --region "$region" --task-definition "$task_def_arn" \
        --query 'taskDefinition.containerDefinitions[?privileged==`true`].name' --output text)
      
      if [ -n "$privileged" ]; then
        echo "🚨 HIGH: Task definition $task_def_arn has privileged container: $privileged"
        echo "  Risk: Privileged containers can escape to host system"
      fi
    done
    echo ""
  fi
done
```

#### ECS Task Definitions - Root User
```bash
# Find containers running as root user
echo "🔍 Checking ECS containers running as root..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  task_def_arns=$(aws ecs list-task-definitions --region "$region" --status ACTIVE --query 'taskDefinitionArns[*]' --output text)
  
  if [ -n "$task_def_arns" ]; then
    echo "Region: $region"
    for task_def_arn in $task_def_arns; do
      root_containers=$(aws ecs describe-task-definition --region "$region" --task-definition "$task_def_arn" \
        --query 'taskDefinition.containerDefinitions[?user==`null` || user==`root` || user==`0`].name' --output text)
      
      if [ -n "$root_containers" ]; then
        echo "⚠️  MEDIUM: Task definition $task_def_arn has containers running as root: $root_containers"
        echo "  Recommendation: Specify non-root user in container definition"
      fi
    done
    echo ""
  fi
done
```

#### ECS Task Execution Role Permissions
```bash
# Check task execution roles for least privilege
echo "🔍 Checking ECS task execution role permissions..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  task_def_arns=$(aws ecs list-task-definitions --region "$region" --status ACTIVE --query 'taskDefinitionArns[*]' --output text)
  
  if [ -n "$task_def_arns" ]; then
    for task_def_arn in $task_def_arns; do
      task_role_arn=$(aws ecs describe-task-definition --region "$region" --task-definition "$task_def_arn" --query 'taskDefinition.taskRoleArn' --output text)
      
      if [ -n "$task_role_arn" ] && [ "$task_role_arn" != "None" ]; then
        role_name=$(echo "$task_role_arn" | awk -F'/' '{print $NF}')
        
        # Check for AdministratorAccess
        admin=$(aws iam list-attached-role-policies --role-name "$role_name" --query 'AttachedPolicies[?PolicyName==`AdministratorAccess`]' --output text 2>/dev/null)
        
        if [ -n "$admin" ]; then
          echo "🚨 HIGH: Task definition $task_def_arn has AdministratorAccess via role $role_name"
        fi
      fi
    done
  fi
done
```

### 4. EKS (Elastic Kubernetes Service) Audit

#### List EKS Clusters
```bash
# Get all EKS clusters
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  clusters=$(aws eks list-clusters --region "$region" --query 'clusters[*]' --output text)
  
  if [ -n "$clusters" ]; then
    echo "=== Region: $region ==="
    for cluster in $clusters; do
      echo "EKS Cluster: $cluster"
      aws eks describe-cluster --region "$region" --name "$cluster" \
        --query 'cluster.[name,status,version,endpoint,resourcesVpcConfig.endpointPublicAccess,resourcesVpcConfig.endpointPrivateAccess]' \
        --output table
    done
    echo ""
  fi
done
```

#### EKS Public Endpoint Access
```bash
# Check if EKS API endpoint is publicly accessible
echo "🔍 Checking EKS public endpoint access..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  clusters=$(aws eks list-clusters --region "$region" --query 'clusters[*]' --output text)
  
  if [ -n "$clusters" ]; then
    echo "Region: $region"
    for cluster in $clusters; do
      public_access=$(aws eks describe-cluster --region "$region" --name "$cluster" --query 'cluster.resourcesVpcConfig.endpointPublicAccess' --output text)
      private_access=$(aws eks describe-cluster --region "$region" --name "$cluster" --query 'cluster.resourcesVpcConfig.endpointPrivateAccess' --output text)
      public_access_cidrs=$(aws eks describe-cluster --region "$region" --name "$cluster" --query 'cluster.resourcesVpcConfig.publicAccessCidrs[*]' --output text)
      
      if [ "$public_access" = "True" ]; then
        if [ "$public_access_cidrs" = "0.0.0.0/0" ]; then
          echo "⚠️  MEDIUM: EKS cluster $cluster has unrestricted public API access (0.0.0.0/0)"
          echo "  Recommendation: Restrict to corporate IP ranges or disable public access"
        else
          echo "✓ EKS cluster $cluster has restricted public access: $public_access_cidrs"
        fi
      fi
      
      if [ "$private_access" = "False" ]; then
        echo "ℹ️  INFO: EKS cluster $cluster has private endpoint access disabled"
        echo "  Recommendation: Enable private access for VPC-based management"
      fi
    done
    echo ""
  fi
done
```

#### EKS Secrets Encryption
```bash
# Check if EKS secrets are encrypted with KMS
echo "🔍 Checking EKS secrets encryption..."

for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  clusters=$(aws eks list-clusters --region "$region" --query 'clusters[*]' --output text)
  
  if [ -n "$clusters" ]; then
    for cluster in $clusters; do
      encryption=$(aws eks describe-cluster --region "$region" --name "$cluster" --query 'cluster.encryptionConfig[*].resources' --output text)
      
      if [ -z "$encryption" ] || [ "$encryption" = "None" ]; then
        echo "⚠️  HIGH: EKS cluster $cluster (region: $region) does NOT encrypt secrets with KMS"
        echo "  Risk: Kubernetes secrets stored unencrypted in etcd"
      else
        echo "✓ EKS cluster $cluster has secrets encryption enabled"
      fi
    done
  fi
done
```

### 5. Elastic Beanstalk Security

#### List Elastic Beanstalk Applications
```bash
# Check Elastic Beanstalk environments
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  apps=$(aws elasticbeanstalk describe-applications --region "$region" --query 'Applications[*].ApplicationName' --output text)
  
  if [ -n "$apps" ]; then
    echo "=== Region: $region ==="
    for app in $apps; do
      echo "Application: $app"
      aws elasticbeanstalk describe-environments --region "$region" --application-name "$app" \
        --query 'Environments[*].[EnvironmentName,Status,Health,CNAME]' \
        --output table
    done
    echo ""
  fi
done
```

---

<!-- analysis: iac -->
## Security Checklist

### Critical (Immediate Action)

- [ ] **IMDSv2 required on all EC2 instances**
  - Command: Check `HttpTokens` in instance metadata options
  - Severity: HIGH
  - Finding: IMDSv1 vulnerable to SSRF attacks

- [ ] **No privileged containers in ECS**
  - Command: Check task definitions for `privileged: true`
  - Severity: HIGH
  - Finding: Privileged containers can escape to host

- [ ] **No Lambda functions with wildcard permissions**
  - Command: Check execution role policies
  - Severity: CRITICAL
  - Finding: Violates least privilege

### High Priority

- [ ] **All EBS volumes encrypted**
  - Command: `aws ec2 describe-volumes --filters "Name=encrypted,Values=false"`
  - Severity: HIGH
  - Finding: Unencrypted data at rest vulnerable to physical theft

- [ ] **EBS encryption by default enabled**
  - Command: `aws ec2 get-ebs-encryption-by-default`
  - Severity: HIGH
  - Finding: Prevents accidental unencrypted volume creation

- [ ] **EKS secrets encrypted with KMS**
  - Command: Check `encryptionConfig` in EKS cluster
  - Severity: HIGH
  - Finding: Kubernetes secrets exposed in etcd

- [ ] **No deprecated Lambda runtimes**
  - Command: `aws lambda list-functions --query 'Functions[*].Runtime'`
  - Severity: HIGH
  - Finding: EOL runtimes lack security patches

### Medium Priority

- [ ] **EC2 instances managed by Systems Manager**
  - Command: `aws ssm describe-instance-information`
  - Severity: MEDIUM
  - Finding: Enables automated patching and compliance

- [ ] **No containers running as root**
  - Command: Check ECS task definitions for `user` field
  - Severity: MEDIUM
  - Finding: Root containers increase container escape risk

- [ ] **Lambda functions use latest runtime versions**
  - Command: Check runtime versions
  - Severity: MEDIUM
  - Finding: Older runtimes may have known vulnerabilities

- [ ] **EKS public endpoint access restricted**
  - Command: Check `publicAccessCidrs`
  - Severity: MEDIUM
  - Finding: Unrestricted access increases attack surface

### Best Practices

- [ ] **AMIs less than 90 days old**
  - Command: Compare AMI creation date to current date
  - Severity: LOW
  - Finding: Old AMIs likely missing security patches

- [ ] **Lambda functions in VPC when accessing internal resources**
  - Command: Check `VpcConfig`
  - Severity: INFO
  - Finding: Improves network security posture

- [ ] **Secrets in Secrets Manager, not environment variables**
  - Command: Check Lambda environment variables
  - Severity: LOW
  - Finding: Environment variables visible in console/logs

---

## Findings Template

**Severity**: CRITICAL | HIGH | MEDIUM | LOW | INFO

**Description**: [Issue details]

**Evidence**:
```bash
[CLI output]
```

**Risk**: [Security implications]

**Affected Resources**: [List]

**Remediation**:
```bash
[Fix commands]
```

---

## Example Findings

### Example 1: IMDSv1 Enabled

**Severity**: HIGH

**Description**: EC2 instance i-0abc123 allows IMDSv1, vulnerable to SSRF attacks that can steal instance credentials.

**Evidence**:
```
Instance: i-0abc123def456 (web-server-prod)
HttpTokens: optional
HttpPutResponseHopLimit: 1
```

**Risk**: Attacker exploiting SSRF vulnerability in application can retrieve IAM credentials from metadata service.

**Affected Resources**: 
- Instance: i-0abc123def456
- Role: prod-web-server-role (has S3 and DynamoDB access)

**Remediation**:
```bash
# Require IMDSv2
aws ec2 modify-instance-metadata-options \
  --instance-id i-0abc123def456 \
  --http-tokens required \
  --http-endpoint enabled
```

**References**:
- AWS Security Best Practices for IMDSv2
- MITRE ATT&CK T1552.005 (Cloud Instance Metadata API)

---

### Example 2: Privileged ECS Container

**Severity**: HIGH

**Description**: ECS task definition runs container in privileged mode, allowing container escape.

**Evidence**:
```
Task Definition: webapp-task:12
Container: nginx
Privileged: true
```

**Risk**: Compromised container can access host system, pivot to other containers, and potentially access instance IAM role.

**Affected Resources**:
- Task Definition: webapp-task:12
- Service: webapp-service
- Cluster: prod-cluster

**Remediation**:
```json
{
  "containerDefinitions": [{
    "name": "nginx",
    "privileged": false,
    "user": "nginx:nginx"
  }]
}
```

**References**:
- CIS Docker Benchmark 5.4
- OWASP Docker Top 10

---

## Penetration Testing Perspective

### IMDS Exploitation
```bash
# From compromised EC2 (SSRF or direct access)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/role-name

# Retrieve temporary credentials
# Use aws-cli with --profile to test permissions
```

### Container Escape
```bash
# From privileged container
docker run --privileged -it ubuntu bash
mount /dev/sda1 /mnt
chroot /mnt
# Now on host system
```

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
| LOW | X |

### Key Metrics
- EC2 Instances: X (X with IMDSv2 required)
- EBS Volumes: X (X encrypted)
- Lambda Functions: X
- ECS Clusters: X
- EKS Clusters: X

### Top Recommendations
1. Enforce IMDSv2 on all EC2 instances
2. Encrypt all EBS volumes
3. Remove privileged containers
4. Update deprecated Lambda runtimes

---

**Next Steps**: Remediate critical/high findings within 7 days.
