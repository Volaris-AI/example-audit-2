---
genre: hosting
category: secrets-management
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/*.bicep"
  keywords:
    - azurerm_key_vault
    - key_vault_secret
    - key_vault_key
    - managed_identity
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Azure Secrets Management Audit

## Overview

Audit Azure Key Vault configurations, managed identities, certificate management, key rotation policies, and secrets exposure risks.

## Compliance References

- **CIS Azure Foundations v2.0**: Section 8 (Key Vault)
- **Azure Security Benchmark**: IM (Identity Management), DP (Data Protection)
- **NIST**: IA (Identification and Authentication), SC (System and Communications Protection)

## Discovery Commands

### List Key Vaults

```bash
# List all Key Vaults
az keyvault list --output table

# Get Key Vault details
az keyvault list --query "[].{Name:name, ResourceGroup:resourceGroup, Location:location, SKU:properties.sku.name}" --output table

# Export Key Vault inventory
az keyvault list --output json > keyvaults.json
```

### Key Vault Properties

```bash
# Get Key Vault properties
az keyvault show --name <vault-name> --output json

# Check soft delete and purge protection
az keyvault list --query "[].{Name:name, SoftDelete:properties.enableSoftDelete, PurgeProtection:properties.enablePurgeProtection}" --output table

# Check network access
az keyvault show --name <vault-name> --query "{Name:name, NetworkAcls:properties.networkAcls, PublicAccess:properties.publicNetworkAccess}" --output json
```

### List Secrets, Keys, and Certificates

```bash
# List secrets in a Key Vault
az keyvault secret list --vault-name <vault-name> --output table

# List keys
az keyvault key list --vault-name <vault-name> --output table

# List certificates
az keyvault certificate list --vault-name <vault-name> --output table

# Get secret metadata (not value)
az keyvault secret show --vault-name <vault-name> --name <secret-name> --query "{Name:name, Enabled:attributes.enabled, Expires:attributes.expires, Created:attributes.created}" --output json
```

### Access Policies

```bash
# Show access policies
az keyvault show --name <vault-name> --query "properties.accessPolicies" --output json

# List access policies with permissions
az keyvault show --name <vault-name> --output json | jq '.properties.accessPolicies[] | {objectId, permissions}'

# Check for overly permissive policies (all permissions)
az keyvault show --name <vault-name> --output json | jq '.properties.accessPolicies[] | select(.permissions.keys == ["all"] or .permissions.secrets == ["all"] or .permissions.certificates == ["all"])'
```

### RBAC on Key Vault

```bash
# List RBAC role assignments for Key Vault
az role assignment list --scope /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name> --output table

# Check for Key Vault RBAC mode (RBAC vs Access Policies)
az keyvault show --name <vault-name> --query "properties.enableRbacAuthorization" --output tsv

# List Key Vaults using RBAC
az keyvault list --query "[?properties.enableRbacAuthorization].{Name:name, RBACEnabled:properties.enableRbacAuthorization}" --output table
```

### Managed Identities

```bash
# List all managed identities
az identity list --output table

# List system-assigned managed identities on VMs
az vm list --query "[?identity.type=='SystemAssigned'].{Name:name, ResourceGroup:resourceGroup, PrincipalId:identity.principalId}" --output table

# List App Services with managed identity
az webapp list --query "[?identity.type=='SystemAssigned' || identity.type=='UserAssigned'].{Name:name, ResourceGroup:resourceGroup, Identity:identity}" --output table

# List AKS clusters with managed identity
az aks list --query "[].{Name:name, IdentityType:identity.type, PrincipalId:identityProfile.kubeletidentity.objectId}" --output table

# Get role assignments for a managed identity
az role assignment list --assignee <principal-id> --all --output table
```

### Private Endpoints for Key Vault

```bash
# Check if Key Vault has private endpoints
az keyvault show --name <vault-name> --query "properties.privateEndpointConnections" --output json

# List Key Vaults without private endpoints
az keyvault list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  private_endpoints=$(az keyvault show --name "$name" --query "properties.privateEndpointConnections" --output tsv 2>/dev/null)
  public_access=$(az keyvault show --name "$name" --query "properties.publicNetworkAccess" --output tsv 2>/dev/null)
  
  if [ -z "$private_endpoints" ] && [ "$public_access" == "Enabled" ]; then
    echo "PUBLIC: $name"
  fi
done
```

<!-- analysis: iac -->
## Security Checks

### 1. Key Vault Soft Delete and Purge Protection

**CIS Control 8.1**: Ensure that the expiration date is set on all keys

```bash
# Check soft delete and purge protection
az keyvault list --query "[].{Name:name, SoftDelete:properties.enableSoftDelete, PurgeProtection:properties.enablePurgeProtection}" --output table

# Find Key Vaults without soft delete
az keyvault list --query "[?properties.enableSoftDelete!=\`true\`].{Name:name, ResourceGroup:resourceGroup}" --output table

# Find Key Vaults without purge protection
az keyvault list --query "[?properties.enablePurgeProtection!=\`true\`].{Name:name, ResourceGroup:resourceGroup}" --output table
```

#### Agentic Prompt Example

```
Audit Azure Key Vault and secrets management:

1. List all Key Vaults and their configurations
2. For each Key Vault, check:
   - Soft delete enabled
   - Purge protection enabled
   - Network access restrictions
   - Private endpoint usage
   - Diagnostic logging enabled
   - RBAC vs Access Policies model
3. For secrets, keys, and certificates:
   - Identify items without expiration dates
   - Find expired items still enabled
   - Check for secrets not rotated in 90+ days
4. Access control audit:
   - List all access policies and RBAC assignments
   - Identify overly permissive policies (all permissions)
   - Check for service principals with Key Vault access
5. Managed identities:
   - List all managed identities
   - Map managed identities to resources
   - Check role assignments for managed identities
6. Secrets exposure risks:
   - Check for hardcoded secrets in code repositories
   - Look for secrets in environment variables
   - Verify managed identities used instead of connection strings

Output format:
| Key Vault | Soft Delete | Purge Protection | Network Security | Secrets without Expiry | Access Control Risk | Overall Risk |
```

**CIS Control 8.2**: Ensure that the expiration date is set on all secrets

```bash
# List secrets without expiration date
az keyvault list --query "[].name" --output tsv | while read vault; do
  echo "Key Vault: $vault"
  az keyvault secret list --vault-name "$vault" --query "[?attributes.expires==null].{Name:name, Enabled:attributes.enabled}" --output table 2>/dev/null
  echo ""
done

# Find expired secrets still enabled
az keyvault list --query "[].name" --output tsv | while read vault; do
  echo "Checking $vault for expired secrets..."
  current_date=$(date +%s)
  az keyvault secret list --vault-name "$vault" --output json 2>/dev/null | jq -r --arg now "$current_date" '.[] | select(.attributes.enabled == true and .attributes.expires != null) | select((.attributes.expires | fromdateiso8601) < ($now | tonumber)) | "\(.name) expired on \(.attributes.expires)"'
done
```

**CIS Control 8.3**: Ensure that the expiration date is set on all certificates

```bash
# List certificates without expiration date
az keyvault list --query "[].name" --output tsv | while read vault; do
  echo "Key Vault: $vault"
  az keyvault certificate list --vault-name "$vault" --query "[?attributes.expires==null].{Name:name, Enabled:attributes.enabled}" --output table 2>/dev/null
  echo ""
done

# Find certificates expiring soon (30 days)
az keyvault list --query "[].name" --output tsv | while read vault; do
  echo "Checking $vault for expiring certificates..."
  expiry_threshold=$(date -d "+30 days" +%s)
  az keyvault certificate list --vault-name "$vault" --output json 2>/dev/null | jq -r --arg threshold "$expiry_threshold" '.[] | select(.attributes.expires != null) | select((.attributes.expires | fromdateiso8601) < ($threshold | tonumber)) | "\(.name) expires on \(.attributes.expires)"'
done
```

### 2. Key Vault Network Security

**CIS Control 8.4**: Ensure the key vault is recoverable

```bash
# Check network ACLs for Key Vault
az keyvault show --name <vault-name> --query "properties.networkAcls" --output json

# Find Key Vaults with default action Allow (publicly accessible)
az keyvault list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  default_action=$(az keyvault show --name "$name" --query "properties.networkAcls.defaultAction" --output tsv 2>/dev/null)
  if [ "$default_action" == "Allow" ]; then
    echo "PUBLIC ACCESS: $name"
  fi
done

# Check for Key Vaults without network restrictions
az keyvault list --output json | jq -r '.[] | select(.properties.networkAcls.defaultAction == "Allow" or .properties.networkAcls == null) | .name'
```

**CIS Control 8.5**: Enable role-based access control (RBAC) within Azure Key Vault

```bash
# Check if RBAC is enabled on Key Vault
az keyvault list --query "[].{Name:name, RBACEnabled:properties.enableRbacAuthorization}" --output table

# Find Key Vaults using legacy access policies (not RBAC)
az keyvault list --query "[?properties.enableRbacAuthorization!=\`true\`].{Name:name, ResourceGroup:resourceGroup}" --output table

# List access policies (legacy model)
az keyvault list --query "[].name" --output tsv | while read vault; do
  rbac_enabled=$(az keyvault show --name "$vault" --query "properties.enableRbacAuthorization" --output tsv)
  if [ "$rbac_enabled" != "true" ]; then
    echo "=== $vault (Access Policies Mode) ==="
    az keyvault show --name "$vault" --query "properties.accessPolicies[].{ObjectId:objectId, KeyPerms:permissions.keys, SecretPerms:permissions.secrets, CertPerms:permissions.certificates}" --output table
  fi
done
```

### 3. Key Vault Logging

**CIS Control 8.6**: Ensure logging for Azure Key Vault is 'Enabled'

```bash
# Check diagnostic settings for Key Vaults
az keyvault list --query "[].{Name:name, RG:resourceGroup, ID:id}" --output tsv | while read name rg id; do
  echo "Key Vault: $name"
  az monitor diagnostic-settings list --resource "$id" --output table 2>/dev/null
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

### 4. Managed Identity Usage

**Azure Security Benchmark IM-3**: Manage application identities securely

```bash
# Find resources NOT using managed identity
echo "=== VMs without managed identity ==="
az vm list --query "[?identity==null].{Name:name, ResourceGroup:resourceGroup}" --output table

echo "=== App Services without managed identity ==="
az webapp list --query "[?identity==null].{Name:name, ResourceGroup:resourceGroup}" --output table

echo "=== Function Apps without managed identity ==="
az functionapp list --query "[?identity==null].{Name:name, ResourceGroup:resourceGroup}" --output table

# Check for service principals with Key Vault access (should use managed identity)
az keyvault list --query "[].name" --output tsv | while read vault; do
  echo "=== Key Vault: $vault ==="
  rbac_enabled=$(az keyvault show --name "$vault" --query "properties.enableRbacAuthorization" --output tsv)
  
  if [ "$rbac_enabled" == "true" ]; then
    # RBAC mode
    vault_id=$(az keyvault show --name "$vault" --query "id" --output tsv)
    az role assignment list --scope "$vault_id" --query "[?principalType=='ServicePrincipal'].{Principal:principalName, Role:roleDefinitionName}" --output table
  else
    # Access Policies mode
    az keyvault show --name "$vault" --query "properties.accessPolicies[].objectId" --output tsv | while read object_id; do
      sp_check=$(az ad sp show --id "$object_id" --query "appId" --output tsv 2>/dev/null)
      if [ -n "$sp_check" ]; then
        echo "Service Principal: $object_id has access"
      fi
    done
  fi
  echo ""
done
```

### 5. Key Rotation Policies

**Azure Security Benchmark DP-6**: Use a secure key management process

```bash
# Check for keys without rotation policy
az keyvault list --query "[].name" --output tsv | while read vault; do
  echo "=== Key Vault: $vault ==="
  az keyvault key list --vault-name "$vault" --query "[].{Name:name, KeyType:keyType, Created:attributes.created, Updated:attributes.updated}" --output table 2>/dev/null
  echo ""
done

# Find secrets older than 90 days without update
az keyvault list --query "[].name" --output tsv | while read vault; do
  echo "Checking $vault for old secrets..."
  cutoff_date=$(date -d "90 days ago" +%s)
  az keyvault secret list --vault-name "$vault" --output json 2>/dev/null | jq -r --arg cutoff "$cutoff_date" '.[] | select(.attributes.updated != null) | select((.attributes.updated | fromdateiso8601) < ($cutoff | tonumber)) | "\(.name) last updated \(.attributes.updated)"'
done
```

### 6. Secrets in Code and Configuration

**Manual checks required:**

```bash
# Check for connection strings in environment variables
env | grep -i "connection\|secret\|password\|key" | grep -v "KUBECONFIG\|SSH"

# Search code repositories for secrets (example patterns)
# Note: This should be done in the code repository, not Azure
grep -r "DefaultEndpointsProtocol\|AccountKey\|ConnectionString" . --include="*.cs" --include="*.js" --include="*.py" --include="*.json" --include="*.config" 2>/dev/null | head -20

# Check for hardcoded Key Vault names or secret names
grep -r "vault.azure.net\|keyvault" . --include="*.cs" --include="*.js" --include="*.py" --include="*.json" 2>/dev/null | head -20
```

## Penetration Testing Perspective

### Attack Vectors to Check

1. **Credential Theft**
   - Secrets hardcoded in code
   - Connection strings in config files
   - Service principal credentials in environment variables
   - Secrets in source control history

2. **Key Vault Access**
   - Overly permissive access policies (all permissions)
   - Key Vault accessible from public internet
   - No network restrictions
   - Missing private endpoints

3. **Privilege Escalation**
   - Service principals with Key Vault Administrator role
   - Managed identities with excessive permissions
   - Users with ability to create new access policies

4. **Data Exfiltration**
   - No logging on Key Vault access
   - Secrets without expiration (unlimited validity)
   - No monitoring of secret retrieval

5. **Persistence**
   - Creating new secrets for backdoor access
   - Adding access policies for compromised accounts
   - Rotating keys to lock out legitimate users

### Red Team Commands

```bash
# Enumerate Key Vaults accessible to current identity
az keyvault list --query "[].{Name:name, ResourceGroup:resourceGroup}" --output table

# Try to list secrets (requires permissions)
az keyvault secret list --vault-name <vault-name> --output table

# Try to retrieve secret value (high-value target)
az keyvault secret show --vault-name <vault-name> --name <secret-name> --query "value" --output tsv

# Check if you can modify access policies
az keyvault set-policy --name <vault-name> --object-id <your-object-id> --secret-permissions get list

# Look for Key Vault references in code
grep -r "@Microsoft.KeyVault" . --include="*.json" --include="*.config" 2>/dev/null
```

## DevSecOps Checks

### CI/CD Pipeline Secrets

```bash
# Check for Key Vault integration in pipelines
# Azure DevOps: Verify variable groups linked to Key Vault
# GitHub Actions: Verify secrets stored in GitHub, not hardcoded

# Check for service principals used in pipelines
az ad sp list --show-mine --query "[].{DisplayName:displayName, AppId:appId}" --output table

# Verify service principals have minimal Key Vault permissions
az keyvault show --name <vault-name> --query "properties.accessPolicies[].{ObjectId:objectId, SecretPerms:permissions.secrets}" --output json
```

### Infrastructure as Code

```bash
# Check for hardcoded secrets in IaC
grep -r "secureString\|secureObject" *.json *.tf *.bicep | grep -v "parameters\|variables"

# Verify Key Vault references in IaC
grep -r "@Microsoft.KeyVault\|keyvault" *.json *.tf *.bicep

# Check for parameterization of secrets
grep -r "password\|connectionString\|apiKey" *.json *.tf *.bicep | grep -v "param\|var"
```

## Agentic Audit Workflow

### Complete Secrets Management Audit Prompt

```
You are a security engineer auditing Azure secrets management.

Subscription: <subscription-id>
Compliance Framework: CIS Azure Foundations Benchmark v2.0, Azure Security Benchmark

Execute comprehensive secrets management audit:

Phase 1 - Key Vault Infrastructure:

A. Discovery:
- List all Key Vaults
- Map Key Vaults to resource groups and owners
- Check Key Vault SKU (Standard vs Premium)

B. Configuration Security:
- Verify soft delete is enabled
- Check purge protection status
- Review network ACL configuration
- Verify private endpoint usage
- Check public network access setting

C. Logging and Monitoring:
- Verify diagnostic settings enabled
- Check log retention periods
- Confirm AuditEvent logging

D. Access Control Model:
- Identify if RBAC or Access Policies are used
- Review access policies for overly permissive settings
- Check RBAC role assignments
- Identify accounts with administrative access

Phase 2 - Secrets, Keys, and Certificates Audit:

A. Expiration Management:
- List secrets without expiration dates
- List keys without expiration dates
- List certificates without expiration dates
- Identify expired items still enabled
- Find items expiring in next 30 days

B. Rotation Analysis:
- Identify secrets not updated in 90+ days
- Check for key rotation policies
- Verify certificate auto-renewal settings

C. Usage Analysis:
- Count total secrets per vault
- Identify unused secrets (never accessed)
- Check for duplicate secrets across vaults

Phase 3 - Managed Identity Assessment:

A. Current Usage:
- List all managed identities (system and user-assigned)
- Map managed identities to resources
- Check managed identity role assignments

B. Gap Analysis:
- Identify VMs without managed identity
- Find App Services not using managed identity
- Check for service principals that should use managed identity

C. Permission Review:
- Review managed identity permissions
- Identify managed identities with excessive permissions
- Check Key Vault access by managed identities

Phase 4 - Secrets Exposure Analysis:

A. Code Repository Scan:
- Look for connection strings in code
- Search for API keys and passwords
- Check for Key Vault secret names in code
- Review git history for exposed secrets

B. Configuration Files:
- Check web.config, appsettings.json for secrets
- Review environment files (.env)
- Check Docker files and compose files

C. Environment Variables:
- Check for secrets in environment variables
- Verify use of Key Vault references instead

Phase 5 - Access Control Audit:

A. Access Policies (if not using RBAC):
- List all access policies
- Identify policies with "all" permissions
- Check for unnecessary secret/key/certificate permissions
- Review application access policies

B. RBAC (if enabled):
- List all RBAC role assignments
- Identify Key Vault Administrator assignments
- Check for custom roles
- Review principle of least privilege

Phase 6 - Risk Assessment:

Assign risk levels:
- Critical: Hardcoded secrets in code, Key Vault publicly accessible with no restrictions, no soft delete
- High: Secrets without expiration, overly permissive access policies, no logging
- Medium: Service principals instead of managed identities, missing purge protection
- Low: Secrets not rotated recently, missing network restrictions

Phase 7 - Compliance Mapping:

For each finding, document:
- CIS Azure Foundations control
- Azure Security Benchmark control
- Regulatory requirement (PCI-DSS, HIPAA)

Phase 8 - Reporting:

Generate detailed markdown report:

## Azure Secrets Management Audit Report

### Executive Summary
- **Key Vaults**: [Count]
- **Total Secrets**: [Count]
- **Secrets Without Expiration**: [Count]
- **Managed Identities**: [Count]
- **Critical Findings**: [Count]

### Key Vault Inventory
| Key Vault | Resource Group | Soft Delete | Purge Protection | Network Security | Logging | Risk Level |
|-----------|----------------|-------------|------------------|------------------|---------|------------|

### Critical Findings

#### [Finding Title]
- **Key Vault**: [name]
- **Risk Level**: Critical/High/Medium/Low
- **Issue**: [Description]
- **Evidence**:
  ```bash
  [Azure CLI command and output]
  ```
- **Attack Scenario**: [How attacker could exploit]
- **Impact**: [Potential damage]
- **Compliance**: [CIS/Azure Security Benchmark control]
- **Remediation**:
  ```bash
  [Step-by-step fix commands]
  ```

### Secrets Management Analysis

#### Secrets Without Expiration
| Key Vault | Secret Name | Created | Last Updated | Risk |
|-----------|-------------|---------|--------------|------|

#### Expired Items Still Enabled
| Key Vault | Item Type | Name | Expired Date | Risk |
|-----------|-----------|------|--------------|------|

#### Items Expiring Soon (30 days)
| Key Vault | Item Type | Name | Expiry Date | Action Needed |
|-----------|-----------|------|-------------|---------------|

### Managed Identity Assessment

#### Resources Without Managed Identity
| Resource Type | Name | Resource Group | Current Auth Method | Risk |
|---------------|------|----------------|---------------------|------|

#### Managed Identities with Excessive Permissions
| Managed Identity | Resource | Permissions | Risk | Remediation |
|------------------|----------|-------------|------|-------------|

### Access Control Findings

#### Overly Permissive Access Policies
| Key Vault | Principal | Permissions | Risk | Remediation |
|-----------|-----------|-------------|------|-------------|

#### Service Principals with Key Vault Access
| Service Principal | Key Vault | Permissions | Should Use Managed Identity? |
|-------------------|-----------|-------------|------------------------------|

### Secrets Exposure Risks
[Document any hardcoded secrets, connection strings in code, etc.]

### Remediation Roadmap

#### Immediate Actions (0-7 days)
1. [Remove hardcoded secrets from code]
2. [Enable soft delete and purge protection]
3. [Set expiration on all secrets]

#### Short-term (1-3 months)
1. [Migrate to managed identities]
2. [Implement network restrictions]
3. [Enable diagnostic logging]

#### Long-term (3-12 months)
1. [Migrate to RBAC model]
2. [Implement automated key rotation]
3. [Deploy private endpoints]

### Compliance Summary
| Framework | Control | Requirement | Status | Notes |
|-----------|---------|-------------|--------|-------|

Use Azure CLI commands for all evidence. Provide actionable remediation steps.
```

## Remediation Examples

### Enable Soft Delete and Purge Protection

```bash
# Enable soft delete and purge protection on existing Key Vault
az keyvault update \
  --name <vault-name> \
  --resource-group <rg-name> \
  --enable-soft-delete true \
  --enable-purge-protection true

# Create new Key Vault with soft delete and purge protection
az keyvault create \
  --name <vault-name> \
  --resource-group <rg-name> \
  --location <location> \
  --enable-soft-delete true \
  --enable-purge-protection true \
  --retention-days 90
```

### Configure Network Restrictions

```bash
# Set default action to Deny
az keyvault update \
  --name <vault-name> \
  --resource-group <rg-name> \
  --default-action Deny

# Add allowed IP range
az keyvault network-rule add \
  --name <vault-name> \
  --resource-group <rg-name> \
  --ip-address <ip-address-or-range>

# Add virtual network rule
az keyvault network-rule add \
  --name <vault-name> \
  --resource-group <rg-name> \
  --vnet-name <vnet-name> \
  --subnet <subnet-name>

# Disable public network access
az keyvault update \
  --name <vault-name> \
  --resource-group <rg-name> \
  --public-network-access Disabled
```

### Create Private Endpoint for Key Vault

```bash
# Create private endpoint
az network private-endpoint create \
  --name <endpoint-name> \
  --resource-group <rg-name> \
  --vnet-name <vnet-name> \
  --subnet <subnet-name> \
  --private-connection-resource-id "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>" \
  --group-id vault \
  --connection-name <connection-name>

# Create private DNS zone
az network private-dns zone create \
  --resource-group <rg-name> \
  --name privatelink.vaultcore.azure.net

# Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group <rg-name> \
  --zone-name privatelink.vaultcore.azure.net \
  --name <dns-link-name> \
  --virtual-network <vnet-name> \
  --registration-enabled false
```

### Enable Diagnostic Logging

```bash
# Enable diagnostic settings for Key Vault
az monitor diagnostic-settings create \
  --resource "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>" \
  --name "KeyVaultAuditLogs" \
  --workspace "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>" \
  --logs '[
    {
      "category": "AuditEvent",
      "enabled": true,
      "retentionPolicy": {
        "enabled": true,
        "days": 365
      }
    }
  ]' \
  --metrics '[
    {
      "category": "AllMetrics",
      "enabled": true
    }
  ]'
```

### Migrate to RBAC Model

```bash
# Enable RBAC authorization on Key Vault
az keyvault update \
  --name <vault-name> \
  --resource-group <rg-name> \
  --enable-rbac-authorization true

# Grant Key Vault Secrets User role to a user
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <user-or-sp-object-id> \
  --scope "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"

# Grant Key Vault Secrets Officer role (for secret management)
az role assignment create \
  --role "Key Vault Secrets Officer" \
  --assignee <user-object-id> \
  --scope "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"
```

### Set Expiration on Secrets

```bash
# Update secret with expiration date (90 days from now)
expiration_date=$(date -u -d "90 days" +"%Y-%m-%dT%H:%M:%SZ")

az keyvault secret set \
  --vault-name <vault-name> \
  --name <secret-name> \
  --value <secret-value> \
  --expires "$expiration_date"

# Update existing secret's expiration without changing value
# Note: You must provide the value again
current_value=$(az keyvault secret show --vault-name <vault-name> --name <secret-name> --query "value" --output tsv)
az keyvault secret set \
  --vault-name <vault-name> \
  --name <secret-name> \
  --value "$current_value" \
  --expires "$expiration_date"
```

### Enable Managed Identity and Grant Key Vault Access

```bash
# Enable system-assigned managed identity on VM
az vm identity assign \
  --name <vm-name> \
  --resource-group <rg-name>

# Get the principal ID
principal_id=$(az vm show --name <vm-name> --resource-group <rg-name> --query "identity.principalId" --output tsv)

# Grant Key Vault access using RBAC
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee "$principal_id" \
  --scope "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"

# Or using access policy (if not using RBAC)
az keyvault set-policy \
  --name <vault-name> \
  --object-id "$principal_id" \
  --secret-permissions get list
```

### Remove Hardcoded Secrets from Code

```bash
# Example: Replace connection string with Key Vault reference

# Before (in appsettings.json):
# "ConnectionStrings": {
#   "DefaultConnection": "Server=..."
# }

# After:
# "ConnectionStrings": {
#   "DefaultConnection": "@Microsoft.KeyVault(SecretUri=https://<vault-name>.vault.azure.net/secrets/<secret-name>/)"
# }

# Store the connection string in Key Vault
az keyvault secret set \
  --vault-name <vault-name> \
  --name "DefaultConnection" \
  --value "Server=..."

# Verify App Service can access Key Vault (managed identity required)
az webapp identity show --name <app-name> --resource-group <rg-name>
```

## Output Checklist

- [ ] All Key Vaults inventoried
- [ ] Soft delete and purge protection verified
- [ ] Network access restrictions reviewed
- [ ] Private endpoint usage documented
- [ ] Diagnostic logging enabled with adequate retention
- [ ] Access control model identified (RBAC vs Access Policies)
- [ ] Overly permissive access policies identified
- [ ] Secrets without expiration documented
- [ ] Expired items identified
- [ ] Key rotation policies reviewed
- [ ] Managed identity usage assessed
- [ ] Service principals with Key Vault access documented
- [ ] Hardcoded secrets in code identified
- [ ] Secrets exposure risks documented

## References

- [CIS Azure Foundations Benchmark - Key Vault](https://www.cisecurity.org/benchmark/azure)
- [Azure Security Benchmark - Data Protection](https://docs.microsoft.com/security/benchmark/azure/mcsb-data-protection)
- [Azure Key Vault Best Practices](https://docs.microsoft.com/azure/key-vault/general/best-practices)
- [Managed Identities for Azure Resources](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/)
- [Key Vault RBAC Guide](https://docs.microsoft.com/azure/key-vault/general/rbac-guide)
- [Key Vault Security](https://docs.microsoft.com/azure/key-vault/general/security-features)
