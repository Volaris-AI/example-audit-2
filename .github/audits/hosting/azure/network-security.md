---
genre: hosting
category: network-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/*.bicep"
  keywords:
    - azurerm_network_security_group
    - azurerm_virtual_network
    - azurerm_subnet
    - nsg
    - vnet
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Azure Network Security Audit

## Overview

Audit Azure Virtual Networks (VNets), Network Security Groups (NSGs), Azure Firewall, Application Gateway, Web Application Firewall (WAF), DDoS Protection, and network segmentation.

## Compliance References

- **CIS Azure Foundations v2.0**: Section 6 (Networking)
- **Azure Security Benchmark**: NS (Network Security)
- **NIST**: SC (System and Communications Protection)

## Discovery Commands

### List Virtual Networks

```bash
# List all VNets
az network vnet list --output table

# Get detailed VNet information
az network vnet list --query "[].{Name:name, ResourceGroup:resourceGroup, AddressSpace:addressSpace.addressPrefixes, Location:location}" --output table

# List subnets in a VNet
az network vnet subnet list --vnet-name <vnet-name> --resource-group <rg-name> --output table

# Export VNet configuration
az network vnet list --output json > vnets.json
```

### List Network Security Groups (NSGs)

```bash
# List all NSGs
az network nsg list --output table

# Get NSG rules
az network nsg show --name <nsg-name> --resource-group <rg-name> --output json | jq '.securityRules'

# List all NSG rules across all NSGs
az network nsg list --query "[].{Name:name, ResourceGroup:resourceGroup}" --output tsv | while read name rg; do
  echo "NSG: $name"
  az network nsg rule list --nsg-name "$name" --resource-group "$rg" --output table
done

# Export all NSGs and rules
az network nsg list --output json > nsgs.json
```

### List Public IP Addresses

```bash
# List all public IPs
az network public-ip list --output table

# List VMs with public IPs
az vm list-ip-addresses --output table

# Find resources with public internet exposure
az network public-ip list --query "[].{Name:name, IPAddress:ipAddress, AssociatedResource:ipConfiguration.id}" --output table
```

### List Network Interfaces

```bash
# List all network interfaces
az network nic list --output table

# Get network interface details with NSG associations
az network nic list --query "[].{Name:name, ResourceGroup:resourceGroup, NSG:networkSecurityGroup.id, PrivateIP:ipConfigurations[0].privateIPAddress}" --output table
```

### Azure Firewall

```bash
# List Azure Firewalls
az network firewall list --output table

# Get firewall rules
az network firewall show --name <firewall-name> --resource-group <rg-name> --output json | jq '.applicationRuleCollections, .natRuleCollections, .networkRuleCollections'

# Check firewall threat intelligence mode
az network firewall show --name <firewall-name> --resource-group <rg-name> --query "threatIntelMode" --output tsv
```

### Application Gateway and WAF

```bash
# List Application Gateways
az network application-gateway list --output table

# Check WAF configuration
az network application-gateway waf-config show --gateway-name <gateway-name> --resource-group <rg-name> --output json

# Get WAF policy
az network application-gateway waf-policy list --output table

# Check WAF mode (Detection vs Prevention)
az network application-gateway waf-config show --gateway-name <gateway-name> --resource-group <rg-name> --query "firewallMode" --output tsv
```

### DDoS Protection

```bash
# List DDoS protection plans
az network ddos-protection list --output table

# Check if VNet has DDoS protection enabled
az network vnet list --query "[].{Name:name, DDoSProtection:enableDdosProtection}" --output table
```

### Private Endpoints and Service Endpoints

```bash
# List all private endpoints
az network private-endpoint list --output table

# List service endpoints enabled on subnets
az network vnet subnet list --vnet-name <vnet-name> --resource-group <rg-name> --query "[].{Name:name, ServiceEndpoints:serviceEndpoints}" --output table

# Check Private Link services
az network private-link-service list --output table
```

### Network Watcher

```bash
# List Network Watchers
az network watcher list --output table

# Check if Network Watcher is enabled in each region
az network watcher list --query "[].{Name:name, Location:location, ProvisioningState:provisioningState}" --output table

# Get flow logs configuration
az network watcher flow-log list --location <location> --output table
```

<!-- analysis: iac -->
## Security Checks

### 1. Overly Permissive NSG Rules

**CIS Control 6.1**: Ensure that RDP access is restricted from the internet

```bash
# Find NSG rules allowing RDP (3389) from internet
az network nsg list --output json | jq -r '.[] | select(.securityRules[]? | select(.destinationPortRange == "3389" or .destinationPortRange == "*") | select(.sourceAddressPrefix == "*" or .sourceAddressPrefix == "Internet" or .sourceAddressPrefix == "0.0.0.0/0") | select(.access == "Allow")) | {name, resourceGroup, rule: .securityRules[]}'

# Script to find all permissive rules
az network nsg list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  echo "Checking NSG: $name"
  az network nsg rule list --nsg-name "$name" --resource-group "$rg" --query "[?sourceAddressPrefix=='*' || sourceAddressPrefix=='Internet' || sourceAddressPrefix=='0.0.0.0/0'].{Rule:name, Direction:direction, Access:access, Protocol:protocol, DestPort:destinationPortRange, Priority:priority}" --output table
done
```

**CIS Control 6.2**: Ensure that SSH access is restricted from the internet

```bash
# Find NSG rules allowing SSH (22) from internet
az network nsg list --output json | jq -r '.[] | select(.securityRules[]? | select(.destinationPortRange == "22" or .destinationPortRange == "*") | select(.sourceAddressPrefix == "*" or .sourceAddressPrefix == "Internet" or .sourceAddressPrefix == "0.0.0.0/0") | select(.access == "Allow"))'
```

#### Agentic Prompt Example

```
Audit all Azure Network Security Groups for overly permissive rules:

1. List all NSGs and their associated resources
2. For each NSG, analyze rules that:
   - Allow inbound traffic from 0.0.0.0/0 or Internet
   - Allow RDP (3389), SSH (22), or other management ports
   - Use protocol "Any" or "*"
   - Have low priority (allow before deny rules)
3. Generate a risk assessment for each finding:
   - Critical: Management ports open to internet
   - High: Database ports (1433, 3306, 5432) exposed
   - Medium: Application ports with no source restriction
4. Provide remediation commands to restrict access

Output in markdown table:
| NSG | Resource Group | Rule | Risk | Issue | Fix Command |
```

### 2. Network Segmentation

**Azure Security Benchmark NS-1**: Implement network segmentation

```bash
# Check subnet isolation and peering
az network vnet peering list --vnet-name <vnet-name> --resource-group <rg-name> --output table

# Verify hub-spoke topology
az network vnet list --query "[].{Name:name, Subnets:subnets[].name, Peerings:virtualNetworkPeerings[].name}" --output json

# Check for flat network design (security risk)
az network vnet list --query "[].addressSpace.addressPrefixes[]" --output tsv | wc -l
```

#### Security Questions
- Are production and non-production environments isolated?
- Is there proper subnet segmentation (web, app, data tiers)?
- Are management interfaces on separate subnets?
- Is hub-spoke or mesh topology implemented?

### 3. Private Endpoints for PaaS Services

**CIS Control 6.5**: Ensure that Network Watcher is 'Enabled'

```bash
# Check if storage accounts use private endpoints
az storage account list --query "[].{Name:name, RG:resourceGroup, PublicAccess:publicNetworkAccess, PrivateEndpoints:privateEndpointConnections}" --output table

# Check if SQL databases use private endpoints
az sql server list --query "[].{Name:name, RG:resourceGroup, PublicAccess:publicNetworkAccess}" --output table

# List PaaS resources exposed to internet
az resource list --query "[?contains(type, 'Microsoft.Sql') || contains(type, 'Microsoft.Storage')].{Name:name, Type:type, RG:resourceGroup}" --output table
```

### 4. Azure Firewall Configuration

**Azure Security Benchmark NS-4**: Protect applications and services from external network attacks

```bash
# Verify Azure Firewall threat intelligence is enabled
az network firewall list --query "[].{Name:name, RG:resourceGroup, ThreatIntelMode:threatIntelMode}" --output table

# Check firewall rules for overly broad allow rules
az network firewall show --name <firewall-name> --resource-group <rg-name> --output json | jq '.networkRuleCollections[] | select(.rules[].sourceAddresses[] == "*")'

# Verify forced tunneling is configured
az network firewall show --name <firewall-name> --resource-group <rg-name> --query "ipConfigurations[].subnet.id" --output tsv
```

### 5. WAF Configuration

**CIS Control 6.6**: Ensure that 'HTTP(S) access from the Internet' is evaluated for necessary web apps

```bash
# Check WAF mode (should be Prevention, not Detection)
az network application-gateway waf-config show --gateway-name <gateway-name> --resource-group <rg-name> --query "{Mode:firewallMode, Enabled:enabled, RuleSetType:ruleSetType, RuleSetVersion:ruleSetVersion}" --output table

# Check for disabled WAF rules
az network application-gateway waf-config show --gateway-name <gateway-name> --resource-group <rg-name> --query "disabledRuleGroups" --output json

# Verify OWASP rule set version (should be 3.2 or higher)
az network application-gateway waf-config show --gateway-name <gateway-name> --resource-group <rg-name> --query "ruleSetVersion" --output tsv
```

### 6. DDoS Protection

**Azure Security Benchmark NS-5**: Deploy DDoS protection

```bash
# Check DDoS protection status for all VNets
az network vnet list --query "[].{Name:name, DDoSProtectionEnabled:enableDdosProtection, DDoSProtectionPlan:ddosProtectionPlan.id}" --output table

# Verify DDoS protection plan exists
az network ddos-protection list --output table

# Check if critical VNets have DDoS Standard enabled
# Note: DDoS Basic is free but limited; Standard is recommended for production
```

### 7. Network Flow Logs

**CIS Control 6.5**: Ensure that Network Watcher is 'Enabled'

```bash
# Check if NSG flow logs are enabled
az network watcher flow-log list --location <location> --output table

# Verify flow logs configuration
az network watcher flow-log show --location <location> --name <flow-log-name> --output json | jq '{enabled, storageAccount, retentionDays, trafficAnalytics}'

# Check for regions without Network Watcher
az account list-locations --query "[].name" --output tsv | while read location; do
  echo "Checking $location"
  az network watcher list --query "[?location=='$location'].name" --output tsv || echo "No Network Watcher in $location"
done
```

### 8. Service Endpoints vs Private Endpoints

```bash
# List subnets with service endpoints
az network vnet list --output json | jq -r '.[] | .name as $vnet | .subnets[] | {vnet: $vnet, subnet: .name, serviceEndpoints: .serviceEndpoints}'

# Verify critical services use private endpoints instead of service endpoints
# Private endpoints > Service endpoints for security

# List all private endpoints and their target resources
az network private-endpoint list --query "[].{Name:name, ResourceGroup:resourceGroup, TargetResource:privateLinkServiceConnections[0].privateLinkServiceId}" --output table
```

## Penetration Testing Perspective

### Attack Vectors to Check

1. **Internet-Facing Resources**
   - Public IP addresses with open management ports
   - NSG rules allowing broad internet access
   - Load balancers with public IPs

2. **Network Pivoting**
   - VNet peerings that allow lateral movement
   - Lack of microsegmentation
   - No network isolation between environments

3. **DDoS Exposure**
   - VNets without DDoS protection
   - Single points of failure
   - No rate limiting on Application Gateways

4. **Data Exfiltration**
   - Unrestricted outbound traffic
   - No Azure Firewall or NVA inspection
   - Missing NSG rules for outbound control

### Red Team Commands

```bash
# Enumerate all internet-facing resources
az network public-ip list --query "[].{IP:ipAddress, Name:name, Associated:ipConfiguration.id}" --output table

# Find NSG rules that might allow C2 traffic
az network nsg list --output json | jq -r '.[] | .securityRules[] | select(.direction == "Outbound" and .access == "Allow" and (.destinationAddressPrefix == "*" or .destinationAddressPrefix == "Internet"))'

# Check for VNet peering that allows lateral movement
az network vnet peering list --vnet-name <vnet-name> --resource-group <rg-name> --query "[].{Name:name, AllowForwardedTraffic:allowForwardedTraffic, AllowGatewayTransit:allowGatewayTransit}" --output table

# Identify potential pivot points
az vm list-ip-addresses --query "[].{VM:virtualMachine.name, PrivateIP:virtualMachine.network.privateIpAddresses[0], PublicIP:virtualMachine.network.publicIpAddresses[0].ipAddress}" --output table
```

## DevSecOps Checks

### Infrastructure as Code (IaC) Validation

```bash
# Export NSG rules for code review
az network nsg list --output json | jq '[.[] | {name, location, resourceGroup, securityRules}]' > nsg-config.json

# Check for NSG rules created manually (not via IaC)
# Compare against Terraform/Bicep state
```

### CI/CD Pipeline Security

```bash
# Verify network policies in deployment templates
# Check for hardcoded IP addresses in NSG rules
grep -r "0.0.0.0/0" *.json *.tf *.bicep

# Validate that deployments use NSG variables, not hardcoded values
```

### Network Security Automation

```bash
# Script to automatically check for permissive NSG rules
#!/bin/bash
CRITICAL_PORTS=(22 3389 1433 3306 5432 27017)

az network nsg list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  for port in "${CRITICAL_PORTS[@]}"; do
    az network nsg rule list --nsg-name "$name" --resource-group "$rg" --query "[?destinationPortRange=='$port' && (sourceAddressPrefix=='*' || sourceAddressPrefix=='Internet') && access=='Allow'].{Rule:name, Port:destinationPortRange, Source:sourceAddressPrefix}" --output table
  done
done
```

## Agentic Audit Workflow

### Complete Network Security Audit Prompt

```
You are a network security engineer auditing Azure network infrastructure.

Subscription: <subscription-id>
Compliance Framework: CIS Azure Foundations Benchmark v2.0, Azure Security Benchmark

Execute the following network security audit:

Phase 1 - Discovery and Mapping:
1. List all VNets with their address spaces and subnets
2. Map network topology (peerings, gateways, connections)
3. Identify all NSGs and their associated resources
4. Find all public IP addresses and internet-facing resources
5. List Azure Firewalls, Application Gateways, and WAF policies
6. Enumerate private endpoints and service endpoints

Phase 2 - Security Analysis:

A. NSG Rule Analysis:
- Find rules allowing RDP (3389) from internet
- Find rules allowing SSH (22) from internet
- Identify rules with source 0.0.0.0/0 or *
- Check for overly broad port ranges (e.g., 0-65535)
- Verify deny rules have higher priority than allow rules

B. Network Segmentation:
- Assess VNet design (hub-spoke, flat, mesh)
- Check subnet isolation (prod vs non-prod)
- Verify application tier segmentation (web, app, data)
- Review VNet peering configurations

C. PaaS Network Security:
- Check if storage accounts use private endpoints
- Verify SQL databases are not publicly accessible
- Confirm Key Vaults use private endpoints
- Check App Services VNet integration

D. Perimeter Protection:
- Verify DDoS protection is enabled on critical VNets
- Check Azure Firewall threat intelligence mode
- Verify WAF is in Prevention mode with OWASP 3.2+
- Check Application Gateway configurations

E. Monitoring and Visibility:
- Confirm Network Watcher is enabled in all regions
- Verify NSG flow logs are configured
- Check flow log retention periods (90+ days)
- Verify Traffic Analytics is enabled

Phase 3 - Risk Assessment:
Assign risk levels:
- Critical: Management ports (22, 3389) open to internet
- High: Database ports publicly accessible, no DDoS protection
- Medium: Service endpoints instead of private endpoints
- Low: Flow logs not enabled, detection-only WAF mode

Phase 4 - Penetration Test Perspective:
For each finding, document:
- How an attacker could exploit this
- Potential impact (data breach, service disruption)
- Attack path (reconnaissance → access → lateral movement)

Phase 5 - Reporting:
Generate markdown report with:

## Network Security Audit Report

### Executive Summary
[Overall network security posture, critical issues]

### Network Topology
[Diagram or description of VNet architecture]

### Findings Table
| Risk | Category | Finding | Evidence | CIS Control | Remediation |
|------|----------|---------|----------|-------------|-------------|

### Detailed Analysis
[For each finding:]
#### [Finding Title]
- **Risk Level**: Critical/High/Medium/Low
- **Description**: What is the issue?
- **Evidence**: Azure CLI command output
- **Attack Scenario**: How could this be exploited?
- **Compliance Impact**: Which controls are violated?
- **Remediation**: Step-by-step fix with commands

### Quick Wins
[Low-effort, high-impact fixes with commands]

### Long-Term Recommendations
[Architectural changes, process improvements]

### Compliance Summary
| Framework | Control | Status | Notes |
|-----------|---------|--------|-------|

Use Azure CLI commands throughout. Be thorough and security-focused.
```

## Remediation Examples

### Restrict NSG Rule to Specific IP

```bash
# Update NSG rule to allow only from specific IP
az network nsg rule update \
  --nsg-name <nsg-name> \
  --resource-group <rg-name> \
  --name <rule-name> \
  --source-address-prefixes <your-ip>/32

# Example: Restrict RDP to corporate VPN IP
az network nsg rule update \
  --nsg-name myNSG \
  --resource-group myRG \
  --name AllowRDP \
  --source-address-prefixes 203.0.113.0/24 \
  --priority 100
```

### Enable DDoS Protection

```bash
# Create DDoS protection plan
az network ddos-protection create \
  --resource-group <rg-name> \
  --name <ddos-plan-name> \
  --location <location>

# Enable DDoS protection on VNet
az network vnet update \
  --resource-group <rg-name> \
  --name <vnet-name> \
  --ddos-protection-plan <ddos-plan-id> \
  --ddos-protection true
```

### Enable NSG Flow Logs

```bash
# Create storage account for flow logs
az storage account create \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --location <location> \
  --sku Standard_LRS

# Enable flow logs for NSG
az network watcher flow-log create \
  --location <location> \
  --name <flow-log-name> \
  --nsg <nsg-name> \
  --resource-group <rg-name> \
  --storage-account <storage-account-id> \
  --enabled true \
  --retention 90 \
  --traffic-analytics true \
  --workspace <log-analytics-workspace-id>
```

### Create Private Endpoint for Storage Account

```bash
# Disable public network access on storage account
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --public-network-access Disabled

# Create private endpoint
az network private-endpoint create \
  --name <endpoint-name> \
  --resource-group <rg-name> \
  --vnet-name <vnet-name> \
  --subnet <subnet-name> \
  --private-connection-resource-id <storage-account-id> \
  --group-id blob \
  --connection-name <connection-name>
```

### Update WAF to Prevention Mode

```bash
# Update WAF configuration to prevention mode
az network application-gateway waf-config set \
  --gateway-name <gateway-name> \
  --resource-group <rg-name> \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-type OWASP \
  --rule-set-version 3.2
```

## Output Checklist

- [ ] All VNets and subnets documented
- [ ] Network topology mapped (peerings, gateways)
- [ ] All NSG rules reviewed for permissiveness
- [ ] Internet-facing resources identified
- [ ] Management port exposure assessed
- [ ] Network segmentation evaluated
- [ ] Private endpoints implemented for PaaS
- [ ] DDoS protection status verified
- [ ] Azure Firewall/NVA configuration reviewed
- [ ] WAF policies in prevention mode
- [ ] NSG flow logs enabled with adequate retention
- [ ] Network Watcher enabled in all regions
- [ ] Outbound traffic control assessed

## References

- [CIS Azure Foundations Benchmark - Networking](https://www.cisecurity.org/benchmark/azure)
- [Azure Security Benchmark - Network Security](https://docs.microsoft.com/security/benchmark/azure/mcsb-network-security)
- [Azure Network Security Best Practices](https://docs.microsoft.com/azure/security/fundamentals/network-best-practices)
- [NSG Security Rules](https://docs.microsoft.com/azure/virtual-network/network-security-groups-overview)
- [Azure Firewall Documentation](https://docs.microsoft.com/azure/firewall/)
- [Private Endpoints](https://docs.microsoft.com/azure/private-link/private-endpoint-overview)
