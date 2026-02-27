---
genre: hosting
category: compliance
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/*.bicep"
  keywords:
    - azurerm_policy
    - azure_policy
    - compliance
    - security_center
    - defender
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Azure Compliance Audit

## Overview

Audit compliance with CIS Azure Foundations Benchmark, Azure Security Benchmark, and regulatory frameworks (GDPR, HIPAA, PCI-DSS). Includes Azure Policy evaluation and Microsoft Defender for Cloud secure score.

## Compliance References

- **CIS Azure Foundations Benchmark v2.0**: All sections
- **Azure Security Benchmark v3**: All control families
- **Regulatory**: GDPR, HIPAA, PCI-DSS, ISO 27001, SOC 2, FedRAMP

## Discovery Commands

### Azure Policy

```bash
# List all policy assignments
az policy assignment list --output table

# Get policy compliance state
az policy state list --output table

# Get policy compliance summary
az policy state summarize --output json

# List non-compliant resources
az policy state list --filter "complianceState eq 'NonCompliant'" --output table

# Get policy definitions
az policy definition list --output table

# List custom policy definitions
az policy definition list --query "[?policyType=='Custom'].{Name:displayName, Description:description}" --output table
```

### Azure Policy Initiatives (Policy Sets)

```bash
# List policy set definitions
az policy set-definition list --output table

# Get built-in compliance initiatives
az policy set-definition list --query "[?policyType=='BuiltIn' && contains(displayName, 'CIS')].{Name:displayName, Description:description}" --output table

# Check for Azure Security Benchmark assignment
az policy assignment list --query "[?displayName=='Azure Security Benchmark'].{Name:displayName, Scope:scope}" --output table

# Check for CIS benchmark assignment
az policy assignment list --query "[?contains(displayName, 'CIS')].{Name:displayName, Scope:scope}" --output table
```

### Microsoft Defender for Cloud Secure Score

```bash
# Get secure score
az security secure-scores list --output table

# Get secure score controls
az security secure-score-controls list --output table

# Get secure score control details
az security secure-score-control-definitions list --output table
```

### Regulatory Compliance

```bash
# List compliance standards in Defender for Cloud
az security regulatory-compliance-standards list --output table

# Get compliance assessments for a standard
az security regulatory-compliance-assessments list --standard-name <standard-name> --output table

# Get compliance for specific control
az security regulatory-compliance-controls list --standard-name <standard-name> --output table
```

### Compliance Manager (via Azure Portal API)

```bash
# Note: Azure CLI doesn't have direct Compliance Manager commands
# Access via REST API or Azure Portal

# Get compliance score via REST API
az rest --method GET \
  --uri "https://management.azure.com/subscriptions/<subscription-id>/providers/Microsoft.Security/complianceResults?api-version=2017-08-01-preview" \
  --output json
```

<!-- analysis: iac -->
## CIS Azure Foundations Benchmark Checks

### Section 1: Identity and Access Management

```bash
# CIS 1.1: MFA for privileged users
# Check Conditional Access policies (see identity-security.md template)

# CIS 1.2: Guest users permissions limited
az rest --method GET \
  --uri "https://graph.microsoft.com/v1.0/policies/authorizationPolicy" \
  --output json | jq '.guestUserRoleId'

# CIS 1.3: No guest users with privileged roles
az role assignment list --role Owner --all --query "[].{Principal:principalName, Type:principalType}" --output table

# CIS 1.23: PIM used to manage roles
# Check via Azure Portal or Graph API
```

### Section 2: Microsoft Defender for Cloud

```bash
# CIS 2.1: Defender for Cloud set to On
az security pricing list --query "[].{Name:name, Tier:pricingTier}" --output table

# CIS 2.2: Automatic provisioning of monitoring agent On
az security auto-provisioning-setting show --name default --query "autoProvision" --output tsv

# CIS 2.3-2.14: Various security policies enabled
az security assessment list --query "[].{Name:displayName, Status:status.code}" --output table
```

### Section 3: Storage Accounts

```bash
# CIS 3.1: Secure transfer required
az storage account list --query "[?enableHttpsTrafficOnly==\`false\`].{Name:name, RG:resourceGroup}" --output table

# CIS 3.2: Blob service encryption enabled
az storage account list --query "[].{Name:name, BlobEncryption:encryption.services.blob.enabled}" --output table

# CIS 3.6: Public access level Private for blob containers
az storage account list --query "[?allowBlobPublicAccess==\`true\`].{Name:name, RG:resourceGroup}" --output table

# CIS 3.7: Default network access Deny
az storage account list --query "[?networkRuleSet.defaultAction=='Allow'].{Name:name, RG:resourceGroup}" --output table

# CIS 3.8: Soft delete enabled
az storage account list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  soft_delete=$(az storage account blob-service-properties show --account-name "$name" --query "deleteRetentionPolicy.enabled" --output tsv 2>/dev/null)
  [ "$soft_delete" != "true" ] && echo "NO SOFT DELETE: $name"
done
```

### Section 4: Database Services

```bash
# CIS 4.1.1: SQL Server auditing On
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  audit=$(az sql server audit-policy show --server "$server" --resource-group "$rg" --query "state" --output tsv)
  [ "$audit" != "Enabled" ] && echo "AUDITING OFF: $server"
done

# CIS 4.1.2: SQL Database TDE enabled
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  az sql db list --server "$server" --resource-group "$rg" --query "[].name" --output tsv | while read db; do
    if [ "$db" != "master" ]; then
      tde=$(az sql db tde show --server "$server" --database "$db" --resource-group "$rg" --query "state" --output tsv 2>/dev/null)
      [ "$tde" != "Enabled" ] && echo "TDE OFF: $server/$db"
    fi
  done
done

# CIS 4.1.3: Auditing retention >= 90 days
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  retention=$(az sql server audit-policy show --server "$server" --resource-group "$rg" --query "retentionDays" --output tsv)
  [ "$retention" -lt 90 ] 2>/dev/null && echo "LOW RETENTION: $server ($retention days)"
done

# CIS 4.1.4: Azure AD admin configured
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  ad_admin=$(az sql server ad-admin list --server "$server" --resource-group "$rg" --output tsv 2>/dev/null)
  [ -z "$ad_admin" ] && echo "NO AD ADMIN: $server"
done
```

### Section 5: Logging and Monitoring

```bash
# CIS 5.1.1: Diagnostic setting exists
subscription_id=$(az account show --query id --output tsv)
diag=$(az monitor diagnostic-settings subscription list --output tsv)
[ -z "$diag" ] && echo "NO SUBSCRIPTION DIAGNOSTIC SETTINGS"

# CIS 5.1.2: Activity log retention >= 365 days
az monitor log-profiles list --query "[?retentionPolicy.days < \`365\`].{Name:name, Retention:retentionPolicy.days}" --output table

# CIS 5.1.5: Key Vault logging enabled
az keyvault list --query "[].{Name:name, RG:resourceGroup, ID:id}" --output tsv | while read name rg id; do
  diag=$(az monitor diagnostic-settings list --resource "$id" --output tsv 2>/dev/null)
  [ -z "$diag" ] && echo "NO KV LOGGING: $name"
done

# CIS 5.2.x: Activity log alerts exist
CRITICAL_OPERATIONS=(
  "Microsoft.Authorization/policyAssignments/write"
  "Microsoft.Network/networkSecurityGroups/write"
  "Microsoft.Network/networkSecurityGroups/delete"
  "Microsoft.Sql/servers/firewallRules/write"
)

for operation in "${CRITICAL_OPERATIONS[@]}"; do
  alert=$(az monitor activity-log alert list --query "[?condition.allOf[?field=='operationName' && equals=='$operation']].name" --output tsv)
  [ -z "$alert" ] && echo "MISSING ALERT: $operation"
done
```

### Section 6: Networking

```bash
# CIS 6.1: RDP restricted from internet
az network nsg list --output json | jq -r '.[] | select(.securityRules[]? | select(.destinationPortRange == "3389" or .destinationPortRange == "*") | select(.sourceAddressPrefix == "*" or .sourceAddressPrefix == "Internet" or .sourceAddressPrefix == "0.0.0.0/0") | select(.access == "Allow"))' | jq -r '.name' | sort -u

# CIS 6.2: SSH restricted from internet
az network nsg list --output json | jq -r '.[] | select(.securityRules[]? | select(.destinationPortRange == "22" or .destinationPortRange == "*") | select(.sourceAddressPrefix == "*" or .sourceAddressPrefix == "Internet" or .sourceAddressPrefix == "0.0.0.0/0") | select(.access == "Allow"))' | jq -r '.name' | sort -u

# CIS 6.5: Network Watcher enabled
az network watcher list --query "[].{Name:name, Location:location}" --output table

# CIS 6.6: UDP restricted from internet
az network nsg list --output json | jq -r '.[] | select(.securityRules[]? | select(.protocol == "UDP" or .protocol == "*") | select(.sourceAddressPrefix == "*" or .sourceAddressPrefix == "Internet") | select(.access == "Allow"))'
```

### Section 7: Virtual Machines

```bash
# CIS 7.1: OS and data disks encrypted
az vm list --query "[?storageProfile.osDisk.encryptionSettings.enabled!=\`true\`].{Name:name, RG:resourceGroup}" --output table

# CIS 7.4: Endpoint protection installed
az vm list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  extensions=$(az vm extension list --vm-name "$name" --resource-group "$rg" --query "[?contains(name, 'IaaSAntimalware') || contains(name, 'MicrosoftMonitoringAgent')].name" --output tsv)
  [ -z "$extensions" ] && echo "NO ENDPOINT PROTECTION: $name"
done
```

### Section 9: App Service

```bash
# CIS 9.1: App Service authentication enabled
az webapp list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  auth=$(az webapp auth show --name "$name" --resource-group "$rg" --query "enabled" --output tsv)
  [ "$auth" != "true" ] && echo "AUTH OFF: $name"
done

# CIS 9.2: HTTPS-only enforced
az webapp list --query "[?httpsOnly==\`false\`].{Name:name, RG:resourceGroup}" --output table

# CIS 9.3: Minimum TLS version 1.2
az webapp list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  tls=$(az webapp config show --name "$name" --resource-group "$rg" --query "minTlsVersion" --output tsv)
  [ "$tls" != "1.2" ] && echo "WEAK TLS: $name ($tls)"
done

# CIS 9.5: Remote debugging disabled
az webapp list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  debug=$(az webapp config show --name "$name" --resource-group "$rg" --query "remoteDebuggingEnabled" --output tsv)
  [ "$debug" == "true" ] && echo "DEBUG ENABLED: $name"
done
```

## Azure Security Benchmark Checks

### Control Families

#### NS (Network Security)
```bash
# NS-1: Network segmentation
az network vnet list --query "[].{Name:name, Subnets:subnets[].name}" --output json

# NS-2: Secure network connectivity
az network nsg list --output table
az network private-endpoint list --output table

# NS-4: DDoS protection
az network vnet list --query "[].{Name:name, DDoS:enableDdosProtection}" --output table

# NS-6: Deploy WAF
az network application-gateway list --output table
```

#### IM (Identity Management)
```bash
# IM-1: Use centralized identity
az ad user list --query "[?userType=='Member'].{UPN:userPrincipalName}" --output table

# IM-3: Manage application identities
az ad sp list --all --output table

# IM-7: Restrict credential exposure
# Check for service principals with credentials
```

#### PA (Privileged Access)
```bash
# PA-1: Separate and limit privileged users
az role assignment list --role Owner --all --output table
az role assignment list --role Contributor --all --output table

# PA-2: Avoid standing access
# Check PIM configuration

# PA-7: Follow just-in-time access
# Verify PIM is used for privileged roles
```

#### DP (Data Protection)
```bash
# DP-1: Discover and classify sensitive data
# Manual review required

# DP-2: Monitor anomalies and threats
az security alert list --output table

# DP-3: Encrypt sensitive data in transit
az storage account list --query "[?enableHttpsTrafficOnly==\`false\`].name" --output table

# DP-4: Enable encryption at rest by default
az sql db list --server <server> --resource-group <rg> --query "[].name" --output tsv | while read db; do
  az sql db tde show --server <server> --database "$db" --resource-group <rg> --query "state" --output tsv
done

# DP-5: Use customer-managed keys
az storage account list --query "[?encryption.keySource=='Microsoft.Keyvault'].{Name:name, KeyVault:encryption.keyvaultproperties}" --output table
```

#### LT (Logging and Threat Detection)
```bash
# LT-1: Enable threat detection
az security pricing list --output table

# LT-3: Enable logging for security investigation
az monitor diagnostic-settings subscription list --output table

# LT-4: Enable network logging
az network watcher flow-log list --location <location> --output table

# LT-6: Configure log storage retention
az monitor log-analytics workspace list --query "[].{Name:name, Retention:retentionInDays}" --output table
```

#### PV (Posture and Vulnerability Management)
```bash
# PV-1: Define and establish secure configurations
az policy assignment list --output table

# PV-2: Audit and enforce secure configurations
az policy state list --filter "complianceState eq 'NonCompliant'" --output table

# PV-6: Rapidly remediate vulnerabilities
az security assessment list --query "[?status.code=='Unhealthy' && metadata.severity=='High'].{Name:displayName, Description:description}" --output table
```

## Agentic Audit Workflow

### Complete Compliance Audit Prompt

```
You are a compliance auditor assessing Azure infrastructure against security frameworks.

Subscription: <subscription-id>
Frameworks: CIS Azure Foundations Benchmark v2.0, Azure Security Benchmark v3

Execute comprehensive compliance audit:

Phase 1 - Policy and Governance:

A. Azure Policy:
- List all policy assignments
- Check for CIS Azure Foundations Benchmark assignment
- Check for Azure Security Benchmark assignment
- Identify non-compliant resources
- Review custom policies

B. Policy Compliance:
- Get overall compliance percentage
- List non-compliant resources by policy
- Identify repeated violations
- Check for policy exemptions

Phase 2 - CIS Azure Foundations Benchmark:

Systematically check all CIS controls:

Section 1 - Identity and Access Management:
- 1.1: MFA for privileged users
- 1.2: Guest user permissions limited
- 1.3: No guest users with privileged roles
- 1.23: PIM used for roles
[Continue for all Section 1 controls]

Section 2 - Microsoft Defender for Cloud:
- 2.1: Defender set to On
- 2.2: Auto-provisioning enabled
- 2.3-2.14: Security policies enabled
[Continue for all Section 2 controls]

Section 3 - Storage Accounts:
- 3.1: Secure transfer required
- 3.2: Blob encryption enabled
- 3.6: Public access Private
- 3.7: Default network Deny
- 3.8: Soft delete enabled
[Continue for all Section 3 controls]

Section 4 - Database Services:
- 4.1.1: SQL auditing On
- 4.1.2: TDE enabled
- 4.1.3: Audit retention >= 90 days
- 4.1.4: Azure AD admin configured
[Continue for all Section 4 controls]

Section 5 - Logging and Monitoring:
- 5.1.1: Diagnostic setting exists
- 5.1.2: Activity log retention >= 365 days
- 5.1.5: Key Vault logging enabled
- 5.2.x: Activity log alerts configured
[Continue for all Section 5 controls]

Section 6 - Networking:
- 6.1: RDP restricted from internet
- 6.2: SSH restricted from internet
- 6.5: Network Watcher enabled
[Continue for all Section 6 controls]

Section 7 - Virtual Machines:
- 7.1: Disks encrypted
- 7.4: Endpoint protection installed
[Continue for all Section 7 controls]

Section 9 - App Service:
- 9.1: Authentication enabled
- 9.2: HTTPS-only enforced
- 9.3: Minimum TLS 1.2
- 9.5: Remote debugging disabled
[Continue for all Section 9 controls]

Phase 3 - Azure Security Benchmark:

Check all control families:
- NS (Network Security)
- IM (Identity Management)
- PA (Privileged Access)
- DP (Data Protection)
- AM (Asset Management)
- LT (Logging and Threat Detection)
- IR (Incident Response)
- PV (Posture and Vulnerability Management)
- ES (Endpoint Security)
- BR (Backup and Recovery)

For each control family:
1. List applicable controls
2. Check compliance status
3. Document evidence
4. Identify gaps

Phase 4 - Regulatory Compliance:

If applicable, check:
- GDPR compliance
- HIPAA compliance
- PCI-DSS compliance
- ISO 27001 alignment
- SOC 2 Type II requirements

Phase 5 - Microsoft Defender for Cloud:

A. Secure Score:
- Get overall secure score
- List all control scores
- Identify low-scoring controls
- Review recommendations

B. Compliance Dashboard:
- Check regulatory compliance standards
- Review compliance percentages
- Identify failed assessments

Phase 6 - Gap Analysis:

For each non-compliant control:
1. Document current state
2. Document required state
3. Assess risk of non-compliance
4. Estimate effort to remediate
5. Assign priority (Critical/High/Medium/Low)

Phase 7 - Reporting:

Generate comprehensive markdown report:

## Azure Compliance Audit Report

### Executive Summary
- **Overall Compliance Score**: [X%]
- **CIS Benchmark Compliance**: [X/Y controls passed]
- **Azure Security Benchmark Compliance**: [X/Y controls passed]
- **Critical Findings**: [Count]
- **Total Non-Compliant Resources**: [Count]

### Compliance Dashboard
| Framework | Controls Checked | Passed | Failed | Compliance % |
|-----------|------------------|--------|--------|--------------|
| CIS Azure Foundations v2.0 | X | Y | Z | % |
| Azure Security Benchmark v3 | X | Y | Z | % |

### CIS Azure Foundations Benchmark Detailed Results

#### Section 1: Identity and Access Management
| Control | Requirement | Status | Evidence | Remediation |
|---------|-------------|--------|----------|-------------|
| 1.1 | MFA for privileged users | ❌ FAIL | [Evidence] | [Steps] |
| 1.2 | Guest permissions limited | ✅ PASS | [Evidence] | N/A |
[Continue for all controls]

#### Section 2: Microsoft Defender for Cloud
[Similar table]

[Continue for all sections]

### Azure Security Benchmark Detailed Results

#### NS (Network Security)
| Control | Requirement | Status | Evidence | Remediation |
|---------|-------------|--------|----------|-------------|
[All NS controls]

#### IM (Identity Management)
[All IM controls]

[Continue for all control families]

### Policy Compliance

#### Non-Compliant Resources
| Resource | Policy | Compliance State | Effect | Remediation |
|----------|--------|------------------|--------|-------------|

#### Policy Gaps
[List missing policy assignments]

### Microsoft Defender Secure Score
- **Current Score**: [X/100]
- **Industry Average**: [Y/100]
- **Potential Score Increase**: [Z points]

**Top Recommendations**:
1. [Recommendation with score impact]
2. [Recommendation with score impact]
3. [Recommendation with score impact]

### Critical Compliance Gaps

#### [Gap Title]
- **Framework**: CIS/Azure Security Benchmark/Regulatory
- **Control**: [Control ID and description]
- **Current State**: [What is currently configured]
- **Required State**: [What should be configured]
- **Risk**: [What is the risk of non-compliance]
- **Business Impact**: [Potential consequences]
- **Remediation**:
  ```bash
  [Step-by-step Azure CLI commands]
  ```
- **Effort**: [Low/Medium/High]
- **Priority**: [Critical/High/Medium/Low]

### Remediation Roadmap

#### Phase 1: Critical Fixes (0-30 days)
1. [High-priority, high-impact remediations]

#### Phase 2: Important Improvements (1-3 months)
1. [Medium-priority remediations]

#### Phase 3: Long-term Enhancements (3-12 months)
1. [Strategic improvements]

### Compliance Certification Readiness

#### GDPR
- **Status**: [Ready/Not Ready/Partially Ready]
- **Gaps**: [List key gaps]

#### HIPAA
- **Status**: [Ready/Not Ready/Partially Ready]
- **Gaps**: [List key gaps]

#### PCI-DSS
- **Status**: [Ready/Not Ready/Partially Ready]
- **Gaps**: [List key gaps]

### Recommendations
1. [Strategic recommendation]
2. [Process recommendation]
3. [Technical recommendation]

Use Azure CLI commands for all evidence. Be thorough and provide actionable remediation steps with exact commands.
```

## Remediation Examples

### Assign Azure Policy Initiative

```bash
# Assign CIS Azure Foundations Benchmark
az policy assignment create \
  --name "CIS-Azure-Foundations-1.3.0" \
  --display-name "CIS Microsoft Azure Foundations Benchmark v1.3.0" \
  --policy-set-definition "/providers/Microsoft.Authorization/policySetDefinitions/612b5213-9160-4969-8578-1518bd2a000c" \
  --scope "/subscriptions/<subscription-id>"

# Assign Azure Security Benchmark
az policy assignment create \
  --name "Azure-Security-Benchmark" \
  --display-name "Azure Security Benchmark" \
  --policy-set-definition "/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8" \
  --scope "/subscriptions/<subscription-id>"
```

### Enable Microsoft Defender for Cloud (All Services)

```bash
# Enable Defender for all resource types
for service in VirtualMachines SqlServers AppServices StorageAccounts KubernetesService ContainerRegistry KeyVaults Arm OpenSourceRelationalDatabases; do
  echo "Enabling Defender for $service..."
  az security pricing create --name "$service" --tier Standard
done

# Verify all services are on Standard tier
az security pricing list --query "[].{Service:name, Tier:pricingTier}" --output table
```

### Remediate Policy Non-Compliance

```bash
# Get non-compliant resources for a specific policy
az policy state list \
  --filter "policyAssignmentName eq '<assignment-name>' and complianceState eq 'NonCompliant'" \
  --query "[].{Resource:resourceId, Policy:policyDefinitionName}" \
  --output table

# Trigger policy compliance evaluation
az policy state trigger-scan --subscription <subscription-id>

# Get remediation tasks
az policy remediation list --output table

# Create remediation task
az policy remediation create \
  --name <remediation-name> \
  --policy-assignment <assignment-id> \
  --resource-discovery-mode ReEvaluateCompliance
```

### Fix Specific CIS Controls

```bash
# CIS 3.1: Enable secure transfer for storage account
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --https-only true

# CIS 3.6: Disable public blob access
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --allow-blob-public-access false

# CIS 3.7: Set default network action to Deny
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --default-action Deny

# CIS 4.1.1: Enable SQL Server auditing
az sql server audit-policy update \
  --resource-group <rg-name> \
  --server <server-name> \
  --state Enabled \
  --log-analytics-workspace-resource-id <workspace-id> \
  --retention-days 90

# CIS 9.2: Enable HTTPS-only for App Service
az webapp update \
  --name <app-name> \
  --resource-group <rg-name> \
  --https-only true
```

## Output Checklist

- [ ] Azure Policy assignments documented
- [ ] CIS Azure Foundations Benchmark compliance assessed
- [ ] Azure Security Benchmark compliance assessed
- [ ] Policy compliance state reviewed
- [ ] Non-compliant resources identified
- [ ] Microsoft Defender secure score documented
- [ ] Regulatory compliance status checked
- [ ] Critical compliance gaps identified
- [ ] Remediation priorities assigned
- [ ] Remediation roadmap created
- [ ] GDPR/HIPAA/PCI-DSS readiness assessed
- [ ] Policy exemptions reviewed and justified

## References

- [CIS Azure Foundations Benchmark](https://www.cisecurity.org/benchmark/azure)
- [Azure Security Benchmark](https://docs.microsoft.com/security/benchmark/azure/)
- [Azure Policy Documentation](https://docs.microsoft.com/azure/governance/policy/)
- [Microsoft Defender for Cloud](https://docs.microsoft.com/azure/defender-for-cloud/)
- [Azure Compliance Documentation](https://docs.microsoft.com/azure/compliance/)
- [GDPR on Azure](https://docs.microsoft.com/azure/compliance/offerings/offering-gdpr)
- [HIPAA on Azure](https://docs.microsoft.com/azure/compliance/offerings/offering-hipaa-us)
- [PCI-DSS on Azure](https://docs.microsoft.com/azure/compliance/offerings/offering-pci-dss)
