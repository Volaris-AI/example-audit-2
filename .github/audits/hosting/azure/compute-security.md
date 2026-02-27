---
genre: hosting
category: compute-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/*.bicep"
  keywords:
    - azurerm_virtual_machine
    - azurerm_kubernetes_cluster
    - azurerm_function_app
    - aks
    - app_service
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Azure Compute Security Audit

## Overview

Audit security configurations for Azure Virtual Machines (VMs), Azure Kubernetes Service (AKS), Container Instances, App Service, Function Apps, and related compute resources.

## Compliance References

- **CIS Azure Foundations v2.0**: Section 7 (Virtual Machines), Section 9 (App Service)
- **Azure Security Benchmark**: ES (Endpoint Security), PV (Posture and Vulnerability Management)
- **NIST**: SI (System and Information Integrity)

## Discovery Commands

### List Virtual Machines

```bash
# List all VMs
az vm list --output table

# Get detailed VM information
az vm list --query "[].{Name:name, ResourceGroup:resourceGroup, Location:location, OSType:storageProfile.osDisk.osType, Size:hardwareProfile.vmSize}" --output table

# List VM power states
az vm list --show-details --query "[].{Name:name, PowerState:powerState, PublicIP:publicIps, PrivateIP:privateIps}" --output table

# Export VM inventory
az vm list --output json > vms.json
```

### List VM Extensions

```bash
# List extensions for all VMs
az vm extension list --vm-name <vm-name> --resource-group <rg-name> --output table

# Check for security extensions (Microsoft Monitoring Agent, Azure Security Center)
az vm extension list --vm-name <vm-name> --resource-group <rg-name> --query "[].{Name:name, Publisher:publisher, Type:typeHandlerVersion}" --output table

# Script to check all VMs for security extensions
az vm list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  echo "VM: $name"
  az vm extension list --vm-name "$name" --resource-group "$rg" --query "[?contains(name, 'MicrosoftMonitoringAgent') || contains(name, 'AzureSecurityCenter')].name" --output tsv
done
```

### List VM Disks

```bash
# List all managed disks
az disk list --output table

# Check disk encryption status
az disk list --query "[].{Name:name, ResourceGroup:resourceGroup, Encryption:encryptionSettingsCollection.enabled}" --output table

# List VM OS disks encryption status
az vm list --query "[].{Name:name, RG:resourceGroup, OSDiskEncryption:storageProfile.osDisk.encryptionSettings.enabled}" --output table
```

### Azure Kubernetes Service (AKS)

```bash
# List all AKS clusters
az aks list --output table

# Get AKS cluster details
az aks show --name <aks-name> --resource-group <rg-name> --output json

# Check AKS RBAC status
az aks list --query "[].{Name:name, ResourceGroup:resourceGroup, RBACEnabled:enableRbac, AADEnabled:aadProfile.managed}" --output table

# Check AKS network policy
az aks list --query "[].{Name:name, NetworkPolicy:networkProfile.networkPolicy, NetworkPlugin:networkProfile.networkPlugin}" --output table

# List AKS node pools
az aks nodepool list --cluster-name <aks-name> --resource-group <rg-name> --output table

# Get AKS credentials and check cluster configuration
az aks get-credentials --name <aks-name> --resource-group <rg-name> --overwrite-existing
kubectl cluster-info
kubectl get nodes
kubectl get namespaces
```

### Container Instances

```bash
# List all container instances
az container list --output table

# Get container instance details
az container show --name <container-name> --resource-group <rg-name> --output json

# Check for exposed ports
az container list --query "[].{Name:name, ResourceGroup:resourceGroup, Ports:ipAddress.ports[].port}" --output table
```

### App Service and Function Apps

```bash
# List all App Services
az webapp list --output table

# Get App Service authentication settings
az webapp auth show --name <app-name> --resource-group <rg-name> --output json

# Check HTTPS-only setting
az webapp list --query "[].{Name:name, ResourceGroup:resourceGroup, HTTPSOnly:httpsOnly}" --output table

# List Function Apps
az functionapp list --output table

# Check App Service TLS version
az webapp config show --name <app-name> --resource-group <rg-name> --query "minTlsVersion" --output tsv

# Check managed identity status
az webapp identity show --name <app-name> --resource-group <rg-name> --output json
```

### VM Scale Sets

```bash
# List VM Scale Sets
az vmss list --output table

# Get VM Scale Set details
az vmss show --name <vmss-name> --resource-group <rg-name> --output json

# List instances in scale set
az vmss list-instances --name <vmss-name> --resource-group <rg-name> --output table

# Check automatic OS upgrades
az vmss list --query "[].{Name:name, AutoOSUpgrade:upgradePolicy.automaticOSUpgradePolicy.enableAutomaticOSUpgrade}" --output table
```

<!-- analysis: iac -->
## Security Checks

### 1. VM Disk Encryption

**CIS Control 7.1**: Ensure that 'OS and Data' disks are encrypted with Customer Managed Key (CMK)

```bash
# Check if Azure Disk Encryption is enabled
az vm encryption show --name <vm-name> --resource-group <rg-name> --output json

# List VMs without disk encryption
az vm list --query "[?storageProfile.osDisk.encryptionSettings.enabled==\`false\` || storageProfile.osDisk.encryptionSettings==null].{Name:name, ResourceGroup:resourceGroup}" --output table

# Check for Customer Managed Keys
az disk list --query "[?encryption.type=='EncryptionAtRestWithCustomerKey'].{Name:name, ResourceGroup:resourceGroup, KeyVaultKeyUrl:encryption.diskEncryptionSetId}" --output table
```

#### Agentic Prompt Example

```
Audit Azure VM disk encryption:

1. List all VMs and their disk encryption status
2. Identify VMs without Azure Disk Encryption (ADE) enabled
3. Check if encrypted disks use Customer Managed Keys (CMK) or Platform Managed Keys (PMK)
4. For each unencrypted VM, assess:
   - Data sensitivity (based on naming, tags, resource group)
   - Compliance requirements
   - Risk level
5. Provide remediation commands to enable encryption

Output format:
| VM Name | Resource Group | OS Disk Encrypted | Data Disks Encrypted | Key Type | Risk | Remediation |
```

### 2. VM Security Extensions

**CIS Control 7.4**: Ensure that endpoint protection for all Virtual Machines is installed

```bash
# Check for Microsoft Monitoring Agent (Log Analytics)
az vm extension list --vm-name <vm-name> --resource-group <rg-name> --query "[?contains(name, 'MicrosoftMonitoringAgent') || contains(name, 'OmsAgentForLinux')].{Name:name, Status:provisioningState}" --output table

# Check for Azure Security Center agent
az vm extension list --vm-name <vm-name> --resource-group <rg-name> --query "[?contains(name, 'AzureSecurityCenter')].{Name:name, Status:provisioningState}" --output table

# Check for antimalware extension (Windows VMs)
az vm extension list --vm-name <vm-name> --resource-group <rg-name> --query "[?contains(name, 'IaaSAntimalware')].{Name:name, Status:provisioningState}" --output table

# Script to audit all VMs
az vm list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  echo "=== VM: $name ==="
  echo "Extensions:"
  az vm extension list --vm-name "$name" --resource-group "$rg" --query "[].{Name:name, Publisher:publisher}" --output table
  echo ""
done
```

### 3. VM Security Posture

**Azure Security Benchmark ES-2**: Use modern anti-malware software

```bash
# Check if VMs are registered with Microsoft Defender for Cloud
# This requires checking Defender for Cloud directly

# Check VM boot diagnostics (useful for troubleshooting and forensics)
az vm list --query "[].{Name:name, ResourceGroup:resourceGroup, BootDiagnostics:diagnosticsProfile.bootDiagnostics.enabled}" --output table

# Check for VMs with public IP addresses (security risk)
az vm list-ip-addresses --query "[?virtualMachine.network.publicIpAddresses[0] != null].{VM:virtualMachine.name, PublicIP:virtualMachine.network.publicIpAddresses[0].ipAddress}" --output table

# Check VM update management
az vm get-instance-view --name <vm-name> --resource-group <rg-name> --query "instanceView.extensions[?name=='UpdateManagement'].{Status:statuses}" --output json
```

### 4. AKS Security Configuration

**CIS Control (AKS)**: Ensure Kubernetes API server is not publicly accessible

```bash
# Check if AKS has authorized IP ranges configured
az aks show --name <aks-name> --resource-group <rg-name> --query "apiServerAccessProfile.authorizedIpRanges" --output tsv

# Check if AKS uses private cluster
az aks show --name <aks-name> --resource-group <rg-name> --query "apiServerAccessProfile.enablePrivateCluster" --output tsv

# Verify AKS RBAC is enabled
az aks show --name <aks-name> --resource-group <rg-name> --query "enableRbac" --output tsv

# Check Azure AD integration
az aks show --name <aks-name> --resource-group <rg-name> --query "aadProfile" --output json

# Check if network policy is enabled
az aks show --name <aks-name> --resource-group <rg-name> --query "networkProfile.networkPolicy" --output tsv

# Check pod security policy (deprecated) or pod security standards
kubectl get psp
kubectl get all --all-namespaces | grep -i "security"
```

**AKS Pod Security**

```bash
# Get AKS credentials
az aks get-credentials --name <aks-name> --resource-group <rg-name> --overwrite-existing

# Check for privileged containers
kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.containers[].securityContext.privileged == true) | .metadata.name'

# Check for containers running as root
kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.containers[].securityContext.runAsUser == 0 or .spec.containers[].securityContext.runAsUser == null) | "\(.metadata.namespace)/\(.metadata.name)"'

# List pods with host network access
kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.hostNetwork == true) | "\(.metadata.namespace)/\(.metadata.name)"'

# Check for pods mounting sensitive host paths
kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.volumes[]?.hostPath.path == "/" or .spec.volumes[]?.hostPath.path == "/etc" or .spec.volumes[]?.hostPath.path == "/var/run/docker.sock") | "\(.metadata.namespace)/\(.metadata.name)"'

# Check RBAC bindings
kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.subjects[].name == "system:unauthenticated" or .subjects[].name == "system:anonymous") | .metadata.name'
```

### 5. App Service Security

**CIS Control 9.1**: Ensure App Service Authentication is set up for apps

```bash
# Check if authentication is enabled
az webapp auth show --name <app-name> --resource-group <rg-name> --query "enabled" --output tsv

# Check HTTPS-only enforcement
az webapp show --name <app-name> --resource-group <rg-name> --query "httpsOnly" --output tsv

# Check minimum TLS version (should be 1.2 or higher)
az webapp config show --name <app-name> --resource-group <rg-name> --query "minTlsVersion" --output tsv

# Check for managed identity
az webapp identity show --name <app-name> --resource-group <rg-name> --query "type" --output tsv

# Check if remote debugging is disabled
az webapp config show --name <app-name> --resource-group <rg-name> --query "remoteDebuggingEnabled" --output tsv

# Check FTP state (should be disabled or FTPS only)
az webapp config show --name <app-name> --resource-group <rg-name> --query "ftpsState" --output tsv

# List App Services with issues
az webapp list --query "[?httpsOnly==\`false\` || contains(ftpsState, 'All')].{Name:name, ResourceGroup:resourceGroup, HTTPSOnly:httpsOnly, FTP:ftpsState}" --output table
```

**CIS Control 9.2**: Ensure web app redirects all HTTP traffic to HTTPS

```bash
# Script to check all App Services
az webapp list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  https_only=$(az webapp show --name "$name" --resource-group "$rg" --query "httpsOnly" --output tsv)
  tls_version=$(az webapp config show --name "$name" --resource-group "$rg" --query "minTlsVersion" --output tsv)
  auth_enabled=$(az webapp auth show --name "$name" --resource-group "$rg" --query "enabled" --output tsv)
  
  echo "$name | HTTPS: $https_only | TLS: $tls_version | Auth: $auth_enabled"
done
```

### 6. Container Security

**Azure Security Benchmark ES-1**: Use endpoint detection and response (EDR)

```bash
# Check Azure Container Registry vulnerability scanning
az acr list --output table

# Check if Container Registry has Microsoft Defender enabled
# This is checked in Microsoft Defender for Cloud

# List container images
az acr repository list --name <acr-name> --output table

# Check container instance network profile
az container show --name <container-name> --resource-group <rg-name> --query "networkProfile" --output json

# Verify container instances don't have public IPs unless necessary
az container list --query "[?ipAddress.type=='Public'].{Name:name, ResourceGroup:resourceGroup, IP:ipAddress.ip, Ports:ipAddress.ports}" --output table
```

### 7. VM Auto-Shutdown and Update Management

```bash
# Check auto-shutdown schedules
az vm show --name <vm-name> --resource-group <rg-name> --query "tags" --output json | jq 'select(.AutoShutdownSchedule)'

# Check VM update assessment
az vm assess-patches --name <vm-name> --resource-group <rg-name> --output json

# List VMs with missing patches
az vm list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  az vm assess-patches --name "$name" --resource-group "$rg" --query "availablePatchCountByClassification" --output json
done
```

## Penetration Testing Perspective

### Attack Vectors to Check

1. **VM Compromise**
   - VMs with public IPs and open management ports
   - VMs without antimalware or security agents
   - Unpatched VMs with known vulnerabilities
   - VMs with local admin accounts

2. **Container Escape**
   - Privileged containers
   - Containers running as root
   - Host path mounts (especially Docker socket)
   - Containers with host network access

3. **AKS Cluster Takeover**
   - Publicly accessible API server
   - Weak RBAC configuration
   - No network policies
   - Service accounts with cluster-admin role

4. **App Service Exploitation**
   - HTTP-only applications (MitM attacks)
   - Weak TLS versions (TLS 1.0, 1.1)
   - Debug endpoints exposed
   - FTP access enabled

### Red Team Commands

```bash
# Enumerate compute resources with internet exposure
az vm list-ip-addresses --query "[?virtualMachine.network.publicIpAddresses[0]].{VM:virtualMachine.name, PublicIP:virtualMachine.network.publicIpAddresses[0].ipAddress, PrivateIP:virtualMachine.network.privateIpAddresses[0]}" --output table

# Find VMs in insecure resource groups
az vm list --query "[].{Name:name, ResourceGroup:resourceGroup, Location:location}" --output table

# Check for VM extensions that might be exploitable
az vm extension list --vm-name <vm-name> --resource-group <rg-name> --query "[].{Name:name, Publisher:publisher, Version:typeHandlerVersion}" --output table

# Enumerate AKS service accounts with elevated permissions
kubectl get clusterrolebinding -o json | jq -r '.items[] | select(.roleRef.name == "cluster-admin") | {name: .metadata.name, subjects: .subjects}'

# Check for App Service environment variables (potential credential exposure)
az webapp config appsettings list --name <app-name> --resource-group <rg-name> --output table
```

## DevSecOps Checks

### CI/CD Pipeline Security

```bash
# Check for hardcoded secrets in VM custom script extensions
az vm extension list --vm-name <vm-name> --resource-group <rg-name> --query "[?contains(name, 'CustomScript')].settings" --output json

# Verify App Service deployment slots for safe rollback
az webapp deployment slot list --name <app-name> --resource-group <rg-name> --output table

# Check container image provenance
az acr repository show-tags --name <acr-name> --repository <repo-name> --output table
```

### Infrastructure as Code Security

```bash
# Export VM configuration for IaC comparison
az vm show --name <vm-name> --resource-group <rg-name> --output json > vm-config.json

# Check for VMs created manually vs via IaC
# Compare against Terraform state or ARM template deployments

# Verify App Service configuration matches IaC
az webapp config show --name <app-name> --resource-group <rg-name> --output json > webapp-config.json
```

## Agentic Audit Workflow

### Complete Compute Security Audit Prompt

```
You are a DevSecOps engineer performing an Azure compute security audit.

Subscription: <subscription-id>
Compliance Framework: CIS Azure Foundations Benchmark v2.0, Azure Security Benchmark

Execute comprehensive compute security audit:

Phase 1 - Inventory:
1. List all VMs with their configurations
2. Enumerate AKS clusters and node pools
3. List App Services and Function Apps
4. Identify Container Instances and Container Registries
5. List VM Scale Sets

Phase 2 - VM Security Analysis:

A. Disk Encryption:
- Check Azure Disk Encryption status for OS and data disks
- Verify Customer Managed Keys vs Platform Managed Keys
- Identify unencrypted VMs

B. Security Extensions:
- Check for Microsoft Monitoring Agent
- Verify antimalware/endpoint protection
- Check Azure Security Center agent

C. Network Exposure:
- Find VMs with public IP addresses
- Check associated NSG rules
- Identify management ports (RDP/SSH) exposed to internet

D. Patch Management:
- Assess patch compliance status
- Identify VMs with critical missing patches
- Check automatic update configuration

E. Boot Diagnostics and Monitoring:
- Verify boot diagnostics enabled
- Check diagnostic logs configuration

Phase 3 - AKS Security Analysis:

A. Cluster Configuration:
- Check if API server is private or has authorized IP ranges
- Verify Azure AD integration and RBAC
- Check network policy (Calico, Azure, None)
- Verify Azure Policy for AKS is enabled

B. Pod Security:
- Identify privileged containers
- Find pods running as root
- Check for host network/path mounts
- Review pod security policies/standards

C. RBAC Analysis:
- Check for overly permissive ClusterRoleBindings
- Identify service accounts with cluster-admin
- Verify namespace isolation

D. Container Image Security:
- Check if images are scanned for vulnerabilities
- Verify images come from trusted registries
- Check image pull policies

Phase 4 - App Service Security:

A. Authentication:
- Check if App Service Authentication is enabled
- Verify identity provider configuration
- Check managed identity usage

B. Transport Security:
- Verify HTTPS-only enforcement
- Check minimum TLS version (1.2+)
- Verify custom domain SSL/TLS certificates

C. Access Controls:
- Check if remote debugging is disabled
- Verify FTP is disabled or FTPS-only
- Check IP restrictions

D. Configuration:
- Verify always-on setting for production apps
- Check client certificate authentication
- Review CORS configuration

Phase 5 - Risk Assessment:

Assign risk levels:
- Critical: Unencrypted VMs with sensitive data, AKS API server public with no IP restrictions
- High: VMs with public IPs and open RDP/SSH, privileged containers, App Service over HTTP
- Medium: Missing security extensions, weak TLS versions, FTP enabled
- Low: Missing boot diagnostics, no auto-shutdown configured

Phase 6 - Penetration Testing Perspective:

For each critical/high finding:
1. Describe the attack scenario
2. Document the attack path
3. Estimate potential impact
4. Provide detection methods

Phase 7 - Reporting:

Generate detailed markdown report:

## Compute Security Audit Report

### Executive Summary
[Overview of compute security posture, critical findings count]

### Compute Inventory
| Resource Type | Count | Public Exposure | Encryption Status |
|---------------|-------|-----------------|-------------------|

### Findings by Severity
#### Critical Findings
[List with evidence and immediate remediation]

#### High Findings
[List with evidence and remediation timeline]

#### Medium Findings
[List with remediation recommendations]

### Detailed Findings

#### [Finding Title]
- **Resource**: [VM/AKS/App Service name]
- **Risk Level**: Critical/High/Medium/Low
- **Issue**: [What is misconfigured]
- **Evidence**: [CLI command output]
- **Attack Scenario**: [How this could be exploited]
- **Impact**: [Data breach, service disruption, etc.]
- **CIS/Azure Security Benchmark Control**: [Control number and title]
- **Remediation**: [Step-by-step fix with Azure CLI commands]

### Compliance Summary
| Control | Requirement | Status | Notes |
|---------|-------------|--------|-------|

### Recommendations
1. **Immediate Actions** (0-7 days)
2. **Short-term Improvements** (1-3 months)
3. **Long-term Strategy** (3-12 months)

Use Azure CLI and kubectl commands throughout. Be thorough and provide actionable recommendations.
```

## Remediation Examples

### Enable Azure Disk Encryption

```bash
# Create Key Vault for encryption keys
az keyvault create --name <keyvault-name> --resource-group <rg-name> --location <location> --enabled-for-disk-encryption true

# Enable Azure Disk Encryption on VM
az vm encryption enable \
  --resource-group <rg-name> \
  --name <vm-name> \
  --disk-encryption-keyvault <keyvault-name> \
  --volume-type ALL

# Verify encryption status
az vm encryption show --name <vm-name> --resource-group <rg-name>
```

### Install Security Extensions on VM

```bash
# Install Microsoft Monitoring Agent (Linux)
az vm extension set \
  --publisher Microsoft.EnterpriseCloud.Monitoring \
  --name OmsAgentForLinux \
  --resource-group <rg-name> \
  --vm-name <vm-name> \
  --settings '{"workspaceId":"<workspace-id>"}' \
  --protected-settings '{"workspaceKey":"<workspace-key>"}'

# Install Microsoft Monitoring Agent (Windows)
az vm extension set \
  --publisher Microsoft.EnterpriseCloud.Monitoring \
  --name MicrosoftMonitoringAgent \
  --resource-group <rg-name> \
  --vm-name <vm-name> \
  --settings '{"workspaceId":"<workspace-id>"}' \
  --protected-settings '{"workspaceKey":"<workspace-key>"}'

# Install Antimalware extension (Windows)
az vm extension set \
  --publisher Microsoft.Azure.Security \
  --name IaaSAntimalware \
  --resource-group <rg-name> \
  --vm-name <vm-name> \
  --settings '{"AntimalwareEnabled": true, "RealtimeProtectionEnabled": true}'
```

### Secure App Service

```bash
# Enable HTTPS-only
az webapp update --name <app-name> --resource-group <rg-name> --https-only true

# Set minimum TLS version to 1.2
az webapp config set --name <app-name> --resource-group <rg-name> --min-tls-version 1.2

# Disable FTP
az webapp config set --name <app-name> --resource-group <rg-name> --ftps-state Disabled

# Disable remote debugging
az webapp config set --name <app-name> --resource-group <rg-name> --remote-debugging-enabled false

# Enable managed identity
az webapp identity assign --name <app-name> --resource-group <rg-name>

# Enable App Service Authentication (Azure AD)
az webapp auth update \
  --name <app-name> \
  --resource-group <rg-name> \
  --enabled true \
  --action LoginWithAzureActiveDirectory \
  --aad-client-id <client-id> \
  --aad-client-secret <client-secret> \
  --aad-token-issuer-url "https://sts.windows.net/<tenant-id>/"
```

### Secure AKS Cluster

```bash
# Enable authorized IP ranges for API server
az aks update \
  --name <aks-name> \
  --resource-group <rg-name> \
  --api-server-authorized-ip-ranges <ip-range>

# Enable Azure Policy for AKS
az aks enable-addons \
  --addons azure-policy \
  --name <aks-name> \
  --resource-group <rg-name>

# Enable network policy (requires cluster recreation or new node pool)
az aks update \
  --name <aks-name> \
  --resource-group <rg-name> \
  --network-policy azure

# Apply pod security policy (example)
kubectl apply -f - <<EOF
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: false
EOF
```

## Output Checklist

- [ ] All VMs inventoried with encryption status
- [ ] Security extensions verified on all VMs
- [ ] VMs with public IPs identified and justified
- [ ] Patch compliance assessed
- [ ] AKS clusters reviewed for RBAC and network policies
- [ ] AKS pod security evaluated
- [ ] Container images scanned for vulnerabilities
- [ ] App Services configured with HTTPS-only and TLS 1.2+
- [ ] App Service authentication reviewed
- [ ] Remote debugging disabled on all App Services
- [ ] FTP access restricted or disabled
- [ ] Managed identities enabled where applicable
- [ ] Boot diagnostics enabled for forensics

## References

- [CIS Azure Foundations Benchmark - Virtual Machines](https://www.cisecurity.org/benchmark/azure)
- [CIS Azure Foundations Benchmark - App Service](https://www.cisecurity.org/benchmark/azure)
- [Azure Security Benchmark - Endpoint Security](https://docs.microsoft.com/security/benchmark/azure/mcsb-endpoint-security)
- [Azure VM Security Best Practices](https://docs.microsoft.com/azure/security/fundamentals/iaas)
- [AKS Security Best Practices](https://docs.microsoft.com/azure/aks/operator-best-practices-cluster-security)
- [App Service Security](https://docs.microsoft.com/azure/app-service/overview-security)
- [Azure Disk Encryption](https://docs.microsoft.com/azure/virtual-machines/disk-encryption-overview)
