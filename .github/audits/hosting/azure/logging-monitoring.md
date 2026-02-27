---
genre: hosting
category: logging-monitoring
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/*.bicep"
  keywords:
    - azurerm_monitor
    - azurerm_log_analytics
    - azurerm_application_insights
    - diagnostic_setting
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Azure Logging and Monitoring Audit

## Overview

Audit Azure Monitor, Log Analytics, diagnostic settings, Microsoft Defender for Cloud, Azure Sentinel, activity logs, and alerting configurations.

## Compliance References

- **CIS Azure Foundations v2.0**: Section 5 (Logging and Monitoring)
- **Azure Security Benchmark**: LT (Logging and Threat Detection), IR (Incident Response)
- **NIST**: AU (Audit and Accountability), SI (System and Information Integrity)

## Discovery Commands

### Log Analytics Workspaces

```bash
# List all Log Analytics workspaces
az monitor log-analytics workspace list --output table

# Get workspace details
az monitor log-analytics workspace list --query "[].{Name:name, ResourceGroup:resourceGroup, Location:location, SKU:sku.name, RetentionDays:retentionInDays}" --output table

# Get workspace ID
az monitor log-analytics workspace show --resource-group <rg-name> --workspace-name <workspace-name> --query "customerId" --output tsv
```

### Activity Logs

```bash
# Get activity log events (last 7 days)
az monitor activity-log list --start-time $(date -u -d '7 days ago' '+%Y-%m-%dT%H:%M:%SZ') --output table

# Query specific operations
az monitor activity-log list --caller <email@domain.com> --output table

# Export activity log for analysis
az monitor activity-log list --start-time $(date -u -d '30 days ago' '+%Y-%m-%dT%H:%M:%SZ') --output json > activity-logs.json

# Check activity log retention
az monitor log-profiles list --output json
```

### Diagnostic Settings

```bash
# List diagnostic settings for a resource
az monitor diagnostic-settings list --resource <resource-id> --output table

# Check diagnostic settings for all VMs
az vm list --query "[].{Name:name, RG:resourceGroup, ID:id}" --output tsv | while read name rg id; do
  echo "VM: $name"
  az monitor diagnostic-settings list --resource "$id" --output table 2>/dev/null
  echo ""
done

# Check diagnostic settings for storage accounts
az storage account list --query "[].{Name:name, RG:resourceGroup, ID:id}" --output tsv | while read name rg id; do
  echo "Storage Account: $name"
  az monitor diagnostic-settings list --resource "$id" --output table 2>/dev/null
  echo ""
done
```

### Metric Alerts

```bash
# List all metric alerts
az monitor metrics alert list --output table

# Get alert details
az monitor metrics alert show --resource-group <rg-name> --name <alert-name> --output json

# List alerts by severity
az monitor metrics alert list --query "[].{Name:name, Severity:severity, Enabled:enabled}" --output table
```

### Action Groups

```bash
# List all action groups
az monitor action-group list --output table

# Get action group details
az monitor action-group show --resource-group <rg-name> --name <action-group-name> --output json

# Check configured receivers
az monitor action-group show --resource-group <rg-name> --name <action-group-name> --query "{EmailReceivers:emailReceivers, SMSReceivers:smsReceivers, WebhookReceivers:webhookReceivers}" --output json
```

### Microsoft Defender for Cloud

```bash
# Get Defender for Cloud pricing tier
az security pricing list --output table

# Check Defender plans
az security pricing show --name VirtualMachines --output json
az security pricing show --name SqlServers --output json
az security pricing show --name AppServices --output json
az security pricing show --name StorageAccounts --output json
az security pricing show --name KubernetesService --output json

# Get security contacts
az security contact list --output table

# Get security assessments
az security assessment list --output table

# Get security alerts
az security alert list --output table
```

### Azure Sentinel

```bash
# Check if Sentinel is enabled (Sentinel is a solution on Log Analytics workspace)
az sentinel list --resource-group <rg-name> --workspace-name <workspace-name> --output table 2>/dev/null || echo "Sentinel not enabled or CLI extension not installed"

# List Sentinel incidents (requires Azure Sentinel extension)
# az sentinel incident list --resource-group <rg-name> --workspace-name <workspace-name> --output table
```

<!-- analysis: iac -->
## Security Checks

### 1. Activity Log Retention

**CIS Control 5.1.1**: Ensure that a 'Diagnostic Setting' exists

```bash
# Check if activity log is sent to Log Analytics
az monitor log-profiles list --output json

# Verify diagnostic settings for subscription
subscription_id=$(az account show --query id --output tsv)
az monitor diagnostic-settings subscription list --output table

# Check if activity log is exported
az monitor log-profiles list --query "[].{Name:name, RetentionDays:retentionPolicy.days, StorageAccount:storageAccountId, LogAnalytics:serviceBusRuleId}" --output table
```

#### Agentic Prompt Example

```
Audit Azure logging and monitoring configuration:

1. Check if activity logs are exported to Log Analytics or Storage Account
2. Verify activity log retention is 365+ days
3. Check diagnostic settings for:
   - All VMs
   - All storage accounts
   - All SQL databases
   - All App Services
   - All network resources (NSGs, Application Gateways)
4. For each resource type, identify:
   - Resources without diagnostic settings
   - Resources with insufficient log retention
   - Resources not sending logs to Log Analytics
5. Check alert configuration:
   - Are there alerts for security events?
   - Are action groups properly configured?
   - Is anyone notified of alerts?

Output format:
| Resource | Type | Diagnostic Settings | Log Destination | Retention | Alerts | Risk |
```

**CIS Control 5.1.2**: Ensure Activity Log Retention is set to 365 days or greater

```bash
# Check retention policy
az monitor log-profiles list --query "[].retentionPolicy.{Enabled:enabled, Days:days}" --output table

# Find profiles with insufficient retention
az monitor log-profiles list --query "[?retentionPolicy.days < \`365\`].{Name:name, RetentionDays:retentionPolicy.days}" --output table
```

### 2. Diagnostic Settings for Resources

**CIS Control 5.1.3**: Ensure that diagnostic logs are enabled for all services which support it

```bash
# Function to check diagnostic settings for resource types
check_diagnostics() {
  resource_type=$1
  echo "=== Checking $resource_type ==="
  
  az resource list --resource-type "$resource_type" --query "[].{Name:name, RG:resourceGroup, ID:id}" --output tsv | while read name rg id; do
    diag=$(az monitor diagnostic-settings list --resource "$id" --output tsv 2>/dev/null)
    if [ -z "$diag" ]; then
      echo "NO DIAGNOSTICS: $name"
    fi
  done
  echo ""
}

# Check common resource types
check_diagnostics "Microsoft.Compute/virtualMachines"
check_diagnostics "Microsoft.Network/networkSecurityGroups"
check_diagnostics "Microsoft.Sql/servers/databases"
check_diagnostics "Microsoft.Storage/storageAccounts"
check_diagnostics "Microsoft.Web/sites"
check_diagnostics "Microsoft.KeyVault/vaults"
check_diagnostics "Microsoft.Network/applicationGateways"
check_diagnostics "Microsoft.ContainerService/managedClusters"
```

### 3. Log Analytics Workspace Configuration

**Azure Security Benchmark LT-1**: Enable threat detection and response

```bash
# Check workspace retention
az monitor log-analytics workspace list --query "[].{Name:name, ResourceGroup:resourceGroup, RetentionDays:retentionInDays}" --output table

# Find workspaces with insufficient retention
az monitor log-analytics workspace list --query "[?retentionInDays < \`90\`].{Name:name, RetentionDays:retentionInDays}" --output table

# Check workspace SKU (PerGB2018 is recommended)
az monitor log-analytics workspace list --query "[].{Name:name, SKU:sku.name}" --output table

# List solutions installed on workspace
az monitor log-analytics workspace list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  echo "Workspace: $name"
  # Note: Solutions are listed via different API, not directly via CLI
  resource_id=$(az monitor log-analytics workspace show --resource-group "$rg" --workspace-name "$name" --query "id" --output tsv)
  az resource list --resource-group "$rg" --query "[?contains(id, 'solutions')].name" --output tsv
  echo ""
done
```

### 4. Security Monitoring (Microsoft Defender for Cloud)

**CIS Control 2.1**: Ensure that Microsoft Defender for Cloud is set to On

```bash
# Check Defender for Cloud pricing (Free vs Standard)
az security pricing list --query "[].{Name:name, Tier:pricingTier}" --output table

# Find services on Free tier (should be Standard for production)
az security pricing list --query "[?pricingTier=='Free'].{Name:name, Tier:pricingTier}" --output table

# Check Defender for specific services
for service in VirtualMachines SqlServers AppServices StorageAccounts KubernetesService ContainerRegistry KeyVaults; do
  echo -n "$service: "
  az security pricing show --name "$service" --query "pricingTier" --output tsv 2>/dev/null || echo "Not available"
done
```

**CIS Control 2.2**: Ensure that 'Automatic provisioning of monitoring agent' is set to 'On'

```bash
# Check auto-provisioning settings
az security auto-provisioning-setting list --output table

# Verify monitoring agent auto-provisioning is enabled
az security auto-provisioning-setting show --name default --query "autoProvision" --output tsv
```

**CIS Control 2.3**: Ensure that 'System updates' is set to 'On' for Azure Security Center

```bash
# Get security recommendations
az security assessment list --query "[?contains(displayName, 'System updates')].{Name:displayName, Status:status.code}" --output table

# Get all assessments with Unhealthy status
az security assessment list --query "[?status.code=='Unhealthy'].{Name:displayName, Description:description, Severity:metadata.severity}" --output table --output table
```

### 5. Alert Rules

**CIS Control 5.2**: Ensure that Activity Log Alert exists for various critical operations

```bash
# List activity log alerts
az monitor activity-log alert list --output table

# Check for specific critical alerts (CIS requirements)
CRITICAL_OPERATIONS=(
  "Microsoft.Authorization/policyAssignments/write"
  "Microsoft.Network/networkSecurityGroups/write"
  "Microsoft.Network/networkSecurityGroups/delete"
  "Microsoft.Sql/servers/firewallRules/write"
  "Microsoft.Security/securitySolutions/write"
  "Microsoft.Security/securitySolutions/delete"
)

echo "Checking for activity log alerts..."
for operation in "${CRITICAL_OPERATIONS[@]}"; do
  alert=$(az monitor activity-log alert list --query "[?condition.allOf[?field=='operationName' && equals=='$operation']].name" --output tsv)
  if [ -z "$alert" ]; then
    echo "MISSING ALERT: $operation"
  else
    echo "FOUND: $operation -> $alert"
  fi
done
```

**CIS Control 5.2.1**: Ensure Activity Log Alert exists for Create Policy Assignment

```bash
# Check for policy assignment alert
az monitor activity-log alert list --query "[?condition.allOf[?field=='operationName' && equals=='Microsoft.Authorization/policyAssignments/write']].{Name:name, Enabled:enabled}" --output table
```

**CIS Control 5.2.2**: Ensure Activity Log Alert exists for Create or Update Network Security Group

```bash
# Check for NSG modification alert
az monitor activity-log alert list --query "[?condition.allOf[?field=='operationName' && (equals=='Microsoft.Network/networkSecurityGroups/write' || equals=='Microsoft.Network/networkSecurityGroups/delete')]].{Name:name, Enabled:enabled}" --output table
```

**CIS Control 5.2.3**: Ensure Activity Log Alert exists for Delete Network Security Group

```bash
# Check for NSG deletion alert
az monitor activity-log alert list --query "[?condition.allOf[?field=='operationName' && equals=='Microsoft.Network/networkSecurityGroups/delete']].{Name:name, Enabled:enabled}" --output table
```

**CIS Control 5.2.7**: Ensure Activity Log Alert exists for Create or Update SQL Server Firewall Rule

```bash
# Check for SQL firewall rule alert
az monitor activity-log alert list --query "[?condition.allOf[?field=='operationName' && equals=='Microsoft.Sql/servers/firewallRules/write']].{Name:name, Enabled:enabled}" --output table
```

### 6. Security Alerts and Incidents

```bash
# Get active security alerts
az security alert list --query "[].{Name:alertDisplayName, Severity:reportedSeverity, State:state, TimeGenerated:timeGeneratedUtc}" --output table

# Get high severity alerts
az security alert list --query "[?reportedSeverity=='High' || reportedSeverity=='Critical'].{Name:alertDisplayName, Severity:reportedSeverity, Description:description}" --output table

# Get security recommendations
az security assessment list --query "[].{Name:displayName, Status:status.code, Severity:metadata.severity}" --output table | head -20
```

### 7. Key Vault Logging

**CIS Control 5.1.5**: Ensure that logging for Azure Key Vault is 'Enabled'

```bash
# Check diagnostic settings for all Key Vaults
az keyvault list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  echo "Key Vault: $name"
  vault_id=$(az keyvault show --name "$name" --resource-group "$rg" --query "id" --output tsv)
  az monitor diagnostic-settings list --resource "$vault_id" --output table 2>/dev/null
  echo ""
done

# Find Key Vaults without diagnostic logging
az keyvault list --query "[].{Name:name, RG:resourceGroup, ID:id}" --output tsv | while read name rg id; do
  diag=$(az monitor diagnostic-settings list --resource "$id" --output tsv 2>/dev/null)
  if [ -z "$diag" ]; then
    echo "NO LOGGING: $name"
  fi
done
```

### 8. Network Security Group Flow Logs

**CIS Control 6.5**: Ensure that Network Watcher is 'Enabled'

```bash
# Check if Network Watcher is enabled in each region
az network watcher list --query "[].{Name:name, Location:location, State:provisioningState}" --output table

# Check NSG flow logs
az network watcher flow-log list --location <location> --output table

# Find NSGs without flow logs
az network nsg list --query "[].{Name:name, RG:resourceGroup, Location:location}" --output tsv | while read name rg location; do
  flow_log=$(az network watcher flow-log list --location "$location" --query "[?targetResourceId==nsg_id].name" --output tsv 2>/dev/null)
  if [ -z "$flow_log" ]; then
    echo "NO FLOW LOG: $name (Region: $location)"
  fi
done
```

## Penetration Testing Perspective

### Attack Vectors to Check

1. **Blind Spot Exploitation**
   - Resources without diagnostic logging
   - No activity log retention (evidence destruction)
   - Missing security alerts
   - No monitoring agent on VMs

2. **Alert Fatigue**
   - Too many low-severity alerts
   - No action groups configured
   - Alerts not reaching security team
   - No alert response procedures

3. **Incident Response Gaps**
   - No Sentinel or SIEM integration
   - Insufficient log retention for forensics
   - Missing key security logs (NSG flow logs, Key Vault access)
   - No automated response playbooks

4. **Compliance Evasion**
   - Administrative actions without alerts
   - Critical configuration changes unmonitored
   - No audit trail for privileged operations

### Red Team Commands

```bash
# Check for logging gaps
az vm list --query "[?storageProfile.osDisk.osType=='Linux'].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  extensions=$(az vm extension list --vm-name "$name" --resource-group "$rg" --query "[?contains(name, 'OmsAgent')].name" --output tsv)
  if [ -z "$extensions" ]; then
    echo "NO MONITORING: $name"
  fi
done

# Look for resources without diagnostic settings
az resource list --query "[].{Type:type, Name:name, ID:id}" --output tsv | while read type name id; do
  diag=$(az monitor diagnostic-settings list --resource "$id" --output tsv 2>/dev/null)
  if [ -z "$diag" ]; then
    echo "NO DIAGNOSTICS: $type - $name"
  fi
done | head -20

# Check if security alerts are reaching anyone
az monitor action-group list --query "[].{Name:name, EmailReceivers:emailReceivers[].emailAddress, WebhookReceivers:webhookReceivers[].serviceUri}" --output json
```

## DevSecOps Checks

### CI/CD Pipeline Monitoring

```bash
# Check for pipeline activity monitoring
# Look for alerts on failed deployments, unauthorized changes

# Verify deployment logs are retained
# Check Azure DevOps/GitHub Actions integration with Log Analytics
```

### Infrastructure as Code Monitoring

```bash
# Check for alerts on infrastructure changes
az monitor activity-log alert list --query "[?contains(condition.allOf[].field, 'resourceProvider')].{Name:name, Condition:condition}" --output json

# Verify alerts exist for:
# - Resource group deletions
# - Policy changes
# - Network configuration changes
# - Security setting modifications
```

## Agentic Audit Workflow

### Complete Logging and Monitoring Audit Prompt

```
You are a security operations engineer auditing Azure logging and monitoring.

Subscription: <subscription-id>
Compliance Framework: CIS Azure Foundations Benchmark v2.0, Azure Security Benchmark

Execute comprehensive logging and monitoring audit:

Phase 1 - Infrastructure Discovery:
1. List all Log Analytics workspaces
2. Enumerate diagnostic settings across all resource types
3. List activity log profiles
4. Identify metric alerts and action groups
5. Check Microsoft Defender for Cloud configuration
6. Verify Network Watcher status

Phase 2 - Activity Log Analysis:

A. Retention and Export:
- Check activity log retention (365+ days)
- Verify activity logs are exported to Log Analytics or Storage
- Confirm diagnostic settings at subscription level

B. Critical Operation Alerts:
Check for alerts on:
- Create/Update/Delete Policy Assignment
- Create/Update/Delete Network Security Group
- Create/Update/Delete Network Security Group Rule
- Create/Update/Delete Security Solution
- Create/Update SQL Server Firewall Rule
- Create/Update/Delete Public IP Address

Phase 3 - Resource Diagnostic Settings:

Check diagnostic settings for:
- Virtual Machines
- Network Security Groups
- Storage Accounts
- SQL Databases
- App Services
- Key Vaults
- Application Gateways
- AKS Clusters
- Azure Firewalls

For each resource type, identify:
- Resources without diagnostic settings
- Resources with insufficient log categories enabled
- Resources not sending logs to Log Analytics
- Log retention periods

Phase 4 - Microsoft Defender for Cloud:

A. Coverage:
- Check pricing tier for all services (Standard vs Free)
- Verify auto-provisioning of monitoring agent
- Check security contacts configuration

B. Assessments:
- List all unhealthy security assessments
- Identify high/critical severity recommendations
- Check secure score
- Review security alerts

C. Monitoring Agents:
- Verify Log Analytics agent on all VMs
- Check for missing agents
- Verify workspace configuration

Phase 5 - Alerting Configuration:

A. Alert Rules:
- List all metric alerts
- List all activity log alerts
- Check alert coverage for critical operations
- Verify alert severity configuration

B. Action Groups:
- List all action groups
- Check if email/SMS receivers are configured
- Verify webhook integrations (SIEM, ticketing)
- Check if anyone is notified

C. Alert Response:
- Verify alerts are enabled (not disabled)
- Check for alert suppression rules
- Verify action groups are attached to alerts

Phase 6 - Advanced Monitoring:

A. Network Flow Logs:
- Verify Network Watcher is enabled in all regions
- Check NSG flow logs configuration
- Verify Traffic Analytics is enabled
- Check flow log retention (30+ days)

B. Key Vault Logging:
- Check diagnostic settings for all Key Vaults
- Verify AuditEvent logging is enabled
- Check log retention

C. Storage Analytics:
- Check Storage Analytics logging
- Verify metrics collection

Phase 7 - Risk Assessment:

Assign risk levels:
- Critical: No activity log retention, Defender for Cloud on Free tier, no alerts
- High: Missing diagnostic settings on critical resources, no security monitoring, insufficient retention
- Medium: Missing alerts for some operations, no action groups, monitoring agents not deployed
- Low: Suboptimal log categories, missing Traffic Analytics

Phase 8 - SIEM Integration:

- Check if Sentinel is deployed
- Verify data connectors configuration
- Check for automated response playbooks
- Verify incident creation rules

Phase 9 - Reporting:

Generate detailed markdown report:

## Azure Logging and Monitoring Audit Report

### Executive Summary
[Overall monitoring posture, critical gaps, detection coverage %]

### Monitoring Infrastructure
| Component | Status | Configuration | Coverage | Issues |
|-----------|--------|---------------|----------|--------|

### Activity Log
- **Retention**: [Days]
- **Export Destination**: [Log Analytics/Storage/Both/None]
- **Critical Operation Alerts**: [X/Y configured]

### Diagnostic Settings Coverage
| Resource Type | Total Resources | With Diagnostics | Coverage % | Missing Resources |
|---------------|-----------------|------------------|------------|-------------------|

### Microsoft Defender for Cloud
- **Overall Status**: [Standard/Free/Partial]
- **Services Protected**: [List]
- **Secure Score**: [Score/100]
- **Critical Recommendations**: [Count]
- **Active Alerts**: [Count]

### Critical Findings

#### [Finding Title]
- **Category**: [Activity Log/Diagnostic Settings/Alerting/Defender]
- **Risk Level**: Critical/High/Medium/Low
- **Issue**: [Description]
- **Evidence**:
  ```bash
  [Azure CLI command and output]
  ```
- **Impact**: [What attacker activity would go undetected]
- **Compliance**: [CIS control violated]
- **Remediation**:
  ```bash
  [Step-by-step fix commands]
  ```

### Alert Coverage Analysis
| Operation Category | Required Alerts | Configured | Missing |
|--------------------|-----------------|------------|---------|

### Detection Blind Spots
[List resources/operations without monitoring]

### Remediation Roadmap

#### Immediate Actions (0-7 days)
1. [Enable Defender for Cloud Standard tier]
2. [Configure activity log retention]
3. [Create critical operation alerts]

#### Short-term (1-3 months)
1. [Enable diagnostic settings on all resources]
2. [Deploy monitoring agents]
3. [Configure action groups]

#### Long-term (3-12 months)
1. [Deploy Azure Sentinel]
2. [Implement automated response]
3. [Integrate with SOC/SIEM]

### Compliance Summary
| Framework | Control | Requirement | Status | Notes |
|-----------|---------|-------------|--------|-------|

Use Azure CLI commands for all checks. Provide actionable remediation steps.
```

## Remediation Examples

### Enable Activity Log Export

```bash
# Create Log Analytics workspace if needed
az monitor log-analytics workspace create \
  --resource-group <rg-name> \
  --workspace-name <workspace-name> \
  --location <location>

# Create diagnostic settings for subscription
subscription_id=$(az account show --query id --output tsv)

az monitor diagnostic-settings subscription create \
  --name "ActivityLogToLogAnalytics" \
  --location <location> \
  --workspace "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>" \
  --logs '[
    {
      "category": "Administrative",
      "enabled": true
    },
    {
      "category": "Security",
      "enabled": true
    },
    {
      "category": "ServiceHealth",
      "enabled": true
    },
    {
      "category": "Alert",
      "enabled": true
    },
    {
      "category": "Recommendation",
      "enabled": true
    },
    {
      "category": "Policy",
      "enabled": true
    },
    {
      "category": "Autoscale",
      "enabled": true
    },
    {
      "category": "ResourceHealth",
      "enabled": true
    }
  ]'
```

### Enable Diagnostic Settings for VM

```bash
# Enable diagnostic settings for VM
az monitor diagnostic-settings create \
  --resource "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Compute/virtualMachines/<vm-name>" \
  --name "VMDiagnostics" \
  --workspace "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>" \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'

# Install Log Analytics agent on VM
az vm extension set \
  --publisher Microsoft.EnterpriseCloud.Monitoring \
  --name OmsAgentForLinux \
  --resource-group <rg-name> \
  --vm-name <vm-name> \
  --settings '{"workspaceId":"<workspace-id>"}' \
  --protected-settings '{"workspaceKey":"<workspace-key>"}'
```

### Create Activity Log Alert

```bash
# Create action group first
az monitor action-group create \
  --resource-group <rg-name> \
  --name SecurityTeamAlerts \
  --short-name SecTeam \
  --email-receiver name=SecurityTeam email=security@example.com

# Create alert for NSG modifications
az monitor activity-log alert create \
  --resource-group <rg-name> \
  --name "Alert-NSG-Modification" \
  --description "Alert when NSG is created, updated or deleted" \
  --condition category=Administrative and operationName=Microsoft.Network/networkSecurityGroups/write \
  --action-group /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Insights/actionGroups/SecurityTeamAlerts \
  --scope /subscriptions/<subscription-id>

# Create alert for policy assignment changes
az monitor activity-log alert create \
  --resource-group <rg-name> \
  --name "Alert-Policy-Assignment" \
  --description "Alert when policy assignment is created or updated" \
  --condition category=Policy and operationName=Microsoft.Authorization/policyAssignments/write \
  --action-group /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Insights/actionGroups/SecurityTeamAlerts \
  --scope /subscriptions/<subscription-id>
```

### Enable Microsoft Defender for Cloud

```bash
# Enable Defender for VMs
az security pricing create \
  --name VirtualMachines \
  --tier Standard

# Enable Defender for SQL
az security pricing create \
  --name SqlServers \
  --tier Standard

# Enable Defender for App Services
az security pricing create \
  --name AppServices \
  --tier Standard

# Enable Defender for Storage
az security pricing create \
  --name StorageAccounts \
  --tier Standard

# Enable Defender for Kubernetes
az security pricing create \
  --name KubernetesService \
  --tier Standard

# Enable auto-provisioning of monitoring agent
az security auto-provisioning-setting update \
  --name default \
  --auto-provision On

# Set security contact
az security contact create \
  --name default1 \
  --email security@example.com \
  --phone '+1-555-0123' \
  --alert-notifications On \
  --alerts-admins On
```

### Enable NSG Flow Logs

```bash
# Ensure Network Watcher exists
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations <location> \
  --enabled true

# Create storage account for flow logs
az storage account create \
  --name flowlogsstorage \
  --resource-group <rg-name> \
  --location <location> \
  --sku Standard_LRS

# Enable flow logs for NSG
az network watcher flow-log create \
  --location <location> \
  --name <flow-log-name> \
  --nsg /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Network/networkSecurityGroups/<nsg-name> \
  --storage-account /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Storage/storageAccounts/flowlogsstorage \
  --enabled true \
  --retention 90 \
  --traffic-analytics true \
  --workspace /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>
```

### Enable Key Vault Logging

```bash
# Enable diagnostic settings for Key Vault
az monitor diagnostic-settings create \
  --resource "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>" \
  --name "KeyVaultAuditLogs" \
  --workspace "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>" \
  --logs '[{"category": "AuditEvent", "enabled": true, "retentionPolicy": {"enabled": true, "days": 365}}]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'
```

## Output Checklist

- [ ] Activity log retention verified (365+ days)
- [ ] Activity logs exported to Log Analytics or Storage
- [ ] Diagnostic settings reviewed for all resource types
- [ ] Critical operation alerts configured
- [ ] Action groups created and tested
- [ ] Microsoft Defender for Cloud enabled (Standard tier)
- [ ] Auto-provisioning of monitoring agent enabled
- [ ] Security contacts configured
- [ ] Log Analytics workspaces documented
- [ ] Workspace retention periods adequate
- [ ] Network Watcher enabled in all regions
- [ ] NSG flow logs enabled
- [ ] Key Vault logging enabled
- [ ] Security alerts and recommendations reviewed

## References

- [CIS Azure Foundations Benchmark - Logging and Monitoring](https://www.cisecurity.org/benchmark/azure)
- [Azure Security Benchmark - Logging and Threat Detection](https://docs.microsoft.com/security/benchmark/azure/mcsb-logging-threat-detection)
- [Azure Monitor Documentation](https://docs.microsoft.com/azure/azure-monitor/)
- [Microsoft Defender for Cloud](https://docs.microsoft.com/azure/defender-for-cloud/)
- [Azure Sentinel](https://docs.microsoft.com/azure/sentinel/)
- [Diagnostic Settings](https://docs.microsoft.com/azure/azure-monitor/essentials/diagnostic-settings)
