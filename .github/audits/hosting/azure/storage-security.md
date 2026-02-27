---
genre: hosting
category: storage-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/*.bicep"
  keywords:
    - azurerm_storage_account
    - azurerm_storage_container
    - blob
    - storage_account
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Azure Storage Security Audit

## Overview

Audit Azure Storage Accounts, Blob Storage, Azure Files, Data Lake Storage, encryption configurations, access controls, and network restrictions.

## Compliance References

- **CIS Azure Foundations v2.0**: Section 3 (Storage Accounts)
- **Azure Security Benchmark**: DP (Data Protection)
- **NIST**: SC (System and Communications Protection), MP (Media Protection)

## Discovery Commands

### List Storage Accounts

```bash
# List all storage accounts
az storage account list --output table

# Get detailed storage account information
az storage account list --query "[].{Name:name, ResourceGroup:resourceGroup, Location:location, SKU:sku.name, Kind:kind, AccessTier:accessTier}" --output table

# Export storage account inventory
az storage account list --output json > storage-accounts.json
```

### Storage Account Properties

```bash
# Get storage account properties
az storage account show --name <storage-account-name> --resource-group <rg-name> --output json

# Check public access configuration
az storage account list --query "[].{Name:name, ResourceGroup:resourceGroup, AllowBlobPublicAccess:allowBlobPublicAccess, PublicNetworkAccess:publicNetworkAccess}" --output table

# Check HTTPS-only enforcement
az storage account list --query "[].{Name:name, HTTPSOnly:enableHttpsTrafficOnly}" --output table

# Check minimum TLS version
az storage account list --query "[].{Name:name, MinTLSVersion:minimumTlsVersion}" --output table
```

### List Blob Containers

```bash
# List containers in a storage account
az storage container list --account-name <storage-account-name> --output table

# Check container public access level
az storage container list --account-name <storage-account-name> --query "[].{Name:name, PublicAccess:properties.publicAccess}" --output table

# List all containers with public access across all storage accounts
az storage account list --query "[].name" --output tsv | while read account; do
  echo "Storage Account: $account"
  az storage container list --account-name "$account" --auth-mode login --query "[?properties.publicAccess != null && properties.publicAccess != 'None'].{Container:name, PublicAccess:properties.publicAccess}" --output table 2>/dev/null
done
```

### List File Shares

```bash
# List Azure File shares
az storage share list --account-name <storage-account-name> --output table

# Check file share properties
az storage share show --name <share-name> --account-name <storage-account-name> --output json

# List file share snapshots
az storage share snapshot list --share-name <share-name> --account-name <storage-account-name> --output table
```

### Data Lake Storage

```bash
# List Data Lake Storage Gen2 accounts
az storage account list --query "[?isHnsEnabled].{Name:name, ResourceGroup:resourceGroup, HNS:isHnsEnabled}" --output table

# List file systems in Data Lake
az storage fs list --account-name <storage-account-name> --auth-mode login --output table
```

### Storage Account Keys and SAS Tokens

```bash
# List storage account keys (highly sensitive)
az storage account keys list --account-name <storage-account-name> --resource-group <rg-name> --output table

# Check key regeneration date (requires custom tracking)
# Note: Azure doesn't provide last regeneration date via CLI

# List account SAS tokens (not directly available via CLI)
# SAS tokens are generated, not listed
```

<!-- analysis: iac -->
## Security Checks

### 1. Blob Public Access

**CIS Control 3.6**: Ensure that 'Public access level' is set to Private for blob containers

```bash
# Find storage accounts allowing blob public access
az storage account list --query "[?allowBlobPublicAccess==\`true\`].{Name:name, ResourceGroup:resourceGroup, AllowPublicAccess:allowBlobPublicAccess}" --output table

# Check for publicly accessible containers
az storage account list --query "[].name" --output tsv | while read account; do
  rg=$(az storage account show --name "$account" --query "resourceGroup" --output tsv)
  echo "=== Storage Account: $account ==="
  az storage container list --account-name "$account" --auth-mode login --query "[].{Name:name, PublicAccess:properties.publicAccess}" --output table 2>/dev/null || echo "Unable to access (check permissions)"
  echo ""
done
```

#### Agentic Prompt Example

```
Audit Azure Storage public access configuration:

1. List all storage accounts and their public access settings
2. For each storage account:
   - Check if allowBlobPublicAccess is enabled
   - List all blob containers with public access
   - Identify containers with 'Blob' or 'Container' public access level
3. Assess risk:
   - Critical: Sensitive data in public containers
   - High: Production storage accounts with public access enabled
   - Medium: Dev/test storage with public access
4. Provide remediation commands to:
   - Disable public access at storage account level
   - Set container access level to Private

Output format:
| Storage Account | Resource Group | Public Access Allowed | Public Containers | Risk | Remediation |
```

### 2. Encryption at Rest

**CIS Control 3.1**: Ensure that 'Secure transfer required' is set to 'Enabled'

```bash
# Check if HTTPS-only is enforced
az storage account list --query "[?enableHttpsTrafficOnly==\`false\`].{Name:name, ResourceGroup:resourceGroup, HTTPSOnly:enableHttpsTrafficOnly}" --output table

# Check encryption configuration
az storage account list --query "[].{Name:name, EncryptionBlob:encryption.services.blob.enabled, EncryptionFile:encryption.services.file.enabled}" --output table

# Check for Customer Managed Keys (CMK)
az storage account list --query "[].{Name:name, ResourceGroup:resourceGroup, KeySource:encryption.keySource, KeyVaultKeyUri:encryption.keyvaultproperties.keyname}" --output table
```

**CIS Control 3.2**: Ensure that 'Storage service encryption' is set to Enabled for Blob Service

```bash
# Verify blob encryption (enabled by default)
az storage account show --name <storage-account-name> --resource-group <rg-name> --query "encryption.services.blob" --output json

# Verify file encryption
az storage account show --name <storage-account-name> --resource-group <rg-name> --query "encryption.services.file" --output json

# Check infrastructure encryption
az storage account list --query "[].{Name:name, InfraEncryption:encryption.requireInfrastructureEncryption}" --output table
```

### 3. Network Access Restrictions

**CIS Control 3.7**: Ensure default network access rule for Storage Accounts is set to deny

```bash
# Check default network action
az storage account list --query "[].{Name:name, ResourceGroup:resourceGroup, DefaultAction:networkRuleSet.defaultAction}" --output table

# Find storage accounts with default action 'Allow'
az storage account list --query "[?networkRuleSet.defaultAction=='Allow'].{Name:name, ResourceGroup:resourceGroup}" --output table

# Check firewall rules
az storage account show --name <storage-account-name> --resource-group <rg-name> --query "networkRuleSet" --output json

# List storage accounts without network restrictions
az storage account list --query "[?networkRuleSet.defaultAction=='Allow' || networkRuleSet==null].{Name:name, ResourceGroup:resourceGroup, NetworkAccess:publicNetworkAccess}" --output table
```

### 4. Private Endpoints

**Azure Security Benchmark DP-4**: Enable data at rest encryption by default

```bash
# List storage accounts with private endpoints
az storage account list --query "[].{Name:name, ResourceGroup:resourceGroup, PrivateEndpoints:privateEndpointConnections[].id}" --output table

# Check if storage accounts use private endpoints
az storage account show --name <storage-account-name> --resource-group <rg-name> --query "privateEndpointConnections" --output json

# Find storage accounts without private endpoints
az storage account list --query "[?privateEndpointConnections==null || length(privateEndpointConnections)==\`0\`].{Name:name, ResourceGroup:resourceGroup, PublicNetworkAccess:publicNetworkAccess}" --output table
```

### 5. Soft Delete and Backup

**CIS Control 3.8**: Ensure soft delete is enabled for Azure Storage

```bash
# Check blob soft delete status
az storage account blob-service-properties show --account-name <storage-account-name> --query "deleteRetentionPolicy" --output json

# Check container soft delete
az storage account blob-service-properties show --account-name <storage-account-name> --query "containerDeleteRetentionPolicy" --output json

# Check file share soft delete
az storage account file-service-properties show --account-name <storage-account-name> --query "shareDeleteRetentionPolicy" --output json

# Script to check all storage accounts
az storage account list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  echo "Storage Account: $name"
  echo "Blob soft delete:"
  az storage account blob-service-properties show --account-name "$name" --query "deleteRetentionPolicy.enabled" --output tsv 2>/dev/null || echo "N/A"
  echo "Container soft delete:"
  az storage account blob-service-properties show --account-name "$name" --query "containerDeleteRetentionPolicy.enabled" --output tsv 2>/dev/null || echo "N/A"
  echo ""
done
```

### 6. Immutable Storage

**Azure Security Benchmark DP-5**: Use customer-managed keys for data protection

```bash
# Check for immutable blob storage (WORM)
az storage account blob-service-properties show --account-name <storage-account-name> --resource-group <rg-name> --output json | jq '.immutableStorageWithVersioning'

# Check blob versioning
az storage account blob-service-properties show --account-name <storage-account-name> --query "isVersioningEnabled" --output tsv

# Check blob legal hold and time-based retention
az storage container immutability-policy show --account-name <storage-account-name> --container-name <container-name> --output json
```

### 7. Access Keys and SAS Tokens

**Azure Security Benchmark DP-6**: Use secure key management processes

```bash
# Check if shared key access is disabled (recommended)
az storage account show --name <storage-account-name> --resource-group <rg-name> --query "allowSharedKeyAccess" --output tsv

# List storage accounts with shared key access enabled
az storage account list --query "[?allowSharedKeyAccess==\`true\` || allowSharedKeyAccess==null].{Name:name, ResourceGroup:resourceGroup, SharedKeyAccess:allowSharedKeyAccess}" --output table

# Check for stored access policies (used for SAS)
az storage container policy list --account-name <storage-account-name> --container-name <container-name> --output table
```

### 8. Azure AD Authentication

**CIS Control 3.10**: Ensure storage for critical data are encrypted with Customer Managed Key

```bash
# Check if Azure AD authentication is required
# This is enforced by disabling shared key access and using RBAC

# List RBAC assignments for storage account
az role assignment list --scope /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name> --output table

# Check for overly permissive roles
az role assignment list --scope /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name> --query "[?roleDefinitionName=='Storage Blob Data Owner' || roleDefinitionName=='Storage Account Contributor'].{Principal:principalName, Role:roleDefinitionName}" --output table
```

### 9. Logging and Monitoring

**CIS Control 3.3**: Ensure Storage logging is enabled for Blob service for read, write and delete requests

```bash
# Check diagnostic settings
az monitor diagnostic-settings list --resource /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name> --output table

# Check blob service logging
az storage logging show --services b --account-name <storage-account-name> --output json

# Check metrics configuration
az storage metrics show --services b --account-name <storage-account-name> --output json

# Script to audit logging across all storage accounts
az storage account list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read name rg; do
  echo "=== Storage Account: $name ==="
  resource_id="/subscriptions/$(az account show --query id -o tsv)/resourceGroups/$rg/providers/Microsoft.Storage/storageAccounts/$name"
  az monitor diagnostic-settings list --resource "$resource_id" --output table
  echo ""
done
```

## Penetration Testing Perspective

### Attack Vectors to Check

1. **Data Exposure**
   - Publicly accessible blob containers
   - SAS tokens with excessive permissions
   - Storage accounts with no network restrictions
   - Misconfigured CORS policies

2. **Credential Theft**
   - Storage account keys in code repositories
   - SAS tokens in logs or URLs
   - Weak or default access policies
   - No key rotation policy

3. **Data Exfiltration**
   - Unrestricted outbound access
   - No logging or monitoring
   - Missing soft delete (data destruction)
   - Shared key access enabled

4. **Ransomware/Data Destruction**
   - No immutable storage for critical data
   - Missing backups or recovery plans
   - Insufficient retention periods
   - No legal holds for compliance data

### Red Team Commands

```bash
# Enumerate publicly accessible storage accounts
az storage account list --query "[].{Name:name, PublicAccess:allowBlobPublicAccess, NetworkAccess:publicNetworkAccess}" --output table

# Try to list containers without authentication
# Note: This attempts unauthenticated access
curl "https://<storage-account-name>.blob.core.windows.net/?comp=list"

# Check for anonymous access to containers
az storage account list --query "[].name" --output tsv | while read account; do
  echo "Trying $account..."
  curl -s "https://$account.blob.core.windows.net/?comp=list" | grep -q "ContainerName" && echo "PUBLIC: $account" || echo "Protected: $account"
done

# Look for storage account keys in common locations
# Check environment variables, config files, scripts
env | grep -i "storage\|account"
find . -type f -name "*.config" -o -name "*.json" -o -name ".env" | xargs grep -i "AccountKey\|ConnectionString" 2>/dev/null
```

## DevSecOps Checks

### Infrastructure as Code Security

```bash
# Export storage account configuration for IaC validation
az storage account show --name <storage-account-name> --resource-group <rg-name> --output json > storage-config.json

# Check for hardcoded storage keys in IaC
grep -r "AccountKey\|DefaultEndpointsProtocol" *.tf *.json *.yml *.yaml

# Verify storage account creation uses secure defaults
# Check Terraform/Bicep templates for:
# - enable_https_traffic_only = true
# - allow_blob_public_access = false
# - min_tls_version = "TLS1_2"
```

### CI/CD Pipeline Security

```bash
# Check for storage account keys in pipeline variables
# This is environment-specific and requires pipeline access

# Verify managed identities are used for pipeline storage access
az storage account show --name <storage-account-name> --resource-group <rg-name> --query "identity" --output json
```

## Agentic Audit Workflow

### Complete Storage Security Audit Prompt

```
You are a cloud security engineer auditing Azure Storage security.

Subscription: <subscription-id>
Compliance Framework: CIS Azure Foundations Benchmark v2.0, Azure Security Benchmark

Execute comprehensive storage security audit:

Phase 1 - Discovery and Inventory:
1. List all storage accounts with their configurations
2. Enumerate blob containers, file shares, and data lakes
3. Map storage account network configuration
4. Identify private endpoints and service endpoints

Phase 2 - Security Configuration Analysis:

A. Public Access:
- Check if allowBlobPublicAccess is disabled at account level
- List all containers with public access (Container or Blob level)
- Identify storage accounts accessible from internet

B. Encryption:
- Verify HTTPS-only enforcement (Secure transfer required)
- Check encryption at rest for blobs and files
- Identify if Customer Managed Keys (CMK) are used
- Verify infrastructure encryption is enabled

C. Network Security:
- Check default network action (should be Deny)
- Review firewall rules and virtual network rules
- Verify private endpoint usage for critical storage
- Check if public network access is disabled

D. Access Control:
- Check if shared key access is disabled
- Review RBAC assignments (look for overly permissive roles)
- Identify SAS token usage patterns
- Verify Azure AD authentication is primary method

E. Data Protection:
- Check soft delete configuration for blobs and containers
- Verify soft delete for file shares
- Check blob versioning status
- Identify immutable storage usage (WORM)
- Verify backup/recovery mechanisms

F. Logging and Monitoring:
- Check diagnostic settings configuration
- Verify Storage Analytics logging for blob operations
- Check metrics collection
- Review log retention periods

Phase 3 - Risk Assessment:

Assign risk levels:
- Critical: Public containers with sensitive data, no encryption at rest
- High: Shared key access enabled, no network restrictions, HTTPS not enforced
- Medium: Missing soft delete, no CMK, insufficient logging
- Low: Suboptimal TLS version, missing versioning

Phase 4 - Compliance Mapping:

For each finding, map to:
- CIS Azure Foundations Benchmark controls
- Azure Security Benchmark controls
- Regulatory requirements (GDPR, HIPAA, PCI-DSS)

Phase 5 - Attack Scenario Analysis:

For critical/high findings:
1. **Data Exposure Scenario**: How could data be accessed?
2. **Credential Compromise**: What happens if keys/SAS tokens leak?
3. **Data Exfiltration**: How could attacker extract data?
4. **Ransomware/Destruction**: How could data be destroyed or encrypted?

Phase 6 - Reporting:

Generate detailed markdown report:

## Azure Storage Security Audit Report

### Executive Summary
[Overall storage security posture, statistics, critical issues]

### Storage Inventory
| Storage Account | Type | Location | Public Access | Encryption | Network Security | Risk Level |
|-----------------|------|----------|---------------|------------|------------------|------------|

### Critical Findings

#### [Finding Title]
- **Storage Account**: [name]
- **Risk Level**: Critical
- **Issue**: [Description]
- **Evidence**: 
  ```bash
  [Azure CLI command and output]
  ```
- **Attack Scenario**: [How this could be exploited]
- **Data at Risk**: [Type and sensitivity of data]
- **Compliance Impact**: [Which controls violated]
- **Remediation**:
  ```bash
  [Step-by-step fix commands]
  ```

### Findings by Category

#### Public Access Issues
[List all public access findings]

#### Encryption Gaps
[List encryption-related findings]

#### Network Security Issues
[List network restriction findings]

#### Access Control Weaknesses
[List authentication/authorization findings]

#### Data Protection Gaps
[List backup/recovery/soft delete findings]

#### Monitoring Deficiencies
[List logging and monitoring findings]

### Quick Wins
[High-impact, low-effort fixes with exact commands]

### Compliance Summary
| Framework | Control | Requirement | Status | Notes |
|-----------|---------|-------------|--------|-------|

### Remediation Roadmap

#### Immediate Actions (0-7 days)
1. [Action with priority and effort estimate]

#### Short-term (1-3 months)
1. [Action with priority and effort estimate]

#### Long-term (3-12 months)
1. [Strategic improvements]

### Appendix: Best Practices
[Azure storage security best practices summary]

Use Azure CLI commands for all evidence. Be thorough and provide actionable remediation steps.
```

## Remediation Examples

### Disable Public Blob Access

```bash
# Disable public access at storage account level
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --allow-blob-public-access false

# Set container to private
az storage container set-permission \
  --name <container-name> \
  --account-name <storage-account-name> \
  --public-access off
```

### Enable Secure Transfer (HTTPS-only)

```bash
# Enforce HTTPS-only traffic
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --https-only true

# Set minimum TLS version to 1.2
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --min-tls-version TLS1_2
```

### Configure Network Restrictions

```bash
# Set default action to Deny
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --default-action Deny

# Add allowed IP range
az storage account network-rule add \
  --account-name <storage-account-name> \
  --resource-group <rg-name> \
  --ip-address <ip-address-or-range>

# Add allowed VNet
az storage account network-rule add \
  --account-name <storage-account-name> \
  --resource-group <rg-name> \
  --vnet-name <vnet-name> \
  --subnet <subnet-name>

# Disable public network access completely
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --public-network-access Disabled
```

### Create Private Endpoint

```bash
# Create private endpoint for storage account
az network private-endpoint create \
  --name <endpoint-name> \
  --resource-group <rg-name> \
  --vnet-name <vnet-name> \
  --subnet <subnet-name> \
  --private-connection-resource-id "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name>" \
  --group-id blob \
  --connection-name <connection-name>

# Create private DNS zone integration
az network private-dns zone create \
  --resource-group <rg-name> \
  --name privatelink.blob.core.windows.net

az network private-dns link vnet create \
  --resource-group <rg-name> \
  --zone-name privatelink.blob.core.windows.net \
  --name <dns-link-name> \
  --virtual-network <vnet-name> \
  --registration-enabled false
```

### Enable Soft Delete

```bash
# Enable blob soft delete (retention in days)
az storage account blob-service-properties update \
  --account-name <storage-account-name> \
  --resource-group <rg-name> \
  --enable-delete-retention true \
  --delete-retention-days 30

# Enable container soft delete
az storage account blob-service-properties update \
  --account-name <storage-account-name> \
  --resource-group <rg-name> \
  --enable-container-delete-retention true \
  --container-delete-retention-days 30

# Enable file share soft delete
az storage account file-service-properties update \
  --account-name <storage-account-name> \
  --resource-group <rg-name> \
  --enable-delete-retention true \
  --delete-retention-days 30
```

### Disable Shared Key Access

```bash
# Disable shared key authorization (use Azure AD only)
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --allow-shared-key-access false

# Grant Storage Blob Data Contributor role to user/managed identity
az role assignment create \
  --role "Storage Blob Data Contributor" \
  --assignee <user-or-managed-identity-id> \
  --scope "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name>"
```

### Enable Customer Managed Keys

```bash
# Create Key Vault
az keyvault create \
  --name <keyvault-name> \
  --resource-group <rg-name> \
  --location <location> \
  --enable-purge-protection true

# Create encryption key
az keyvault key create \
  --vault-name <keyvault-name> \
  --name <key-name> \
  --protection software

# Enable system-assigned managed identity for storage account
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --assign-identity

# Grant storage account access to Key Vault
storage_identity=$(az storage account show --name <storage-account-name> --resource-group <rg-name> --query identity.principalId --output tsv)

az keyvault set-policy \
  --name <keyvault-name> \
  --object-id $storage_identity \
  --key-permissions get unwrapKey wrapKey

# Configure storage account to use CMK
az storage account update \
  --name <storage-account-name> \
  --resource-group <rg-name> \
  --encryption-key-name <key-name> \
  --encryption-key-source Microsoft.Keyvault \
  --encryption-key-vault "https://<keyvault-name>.vault.azure.net"
```

### Enable Diagnostic Logging

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group <rg-name> \
  --workspace-name <workspace-name>

# Enable diagnostic settings for storage account
az monitor diagnostic-settings create \
  --name <diagnostic-setting-name> \
  --resource /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name> \
  --logs '[{"category": "StorageRead", "enabled": true}, {"category": "StorageWrite", "enabled": true}, {"category": "StorageDelete", "enabled": true}]' \
  --metrics '[{"category": "Transaction", "enabled": true}]' \
  --workspace /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>
```

## Output Checklist

- [ ] All storage accounts inventoried
- [ ] Public access status documented for all storage accounts
- [ ] Blob containers reviewed for public access
- [ ] HTTPS-only enforcement verified
- [ ] Encryption at rest confirmed (with key type: PMK vs CMK)
- [ ] Network access restrictions reviewed
- [ ] Private endpoint usage documented
- [ ] Shared key access status checked
- [ ] RBAC assignments reviewed
- [ ] Soft delete configuration verified
- [ ] Blob versioning status checked
- [ ] Immutable storage usage identified
- [ ] Diagnostic logging configuration reviewed
- [ ] Log retention periods documented

## References

- [CIS Azure Foundations Benchmark - Storage Accounts](https://www.cisecurity.org/benchmark/azure)
- [Azure Security Benchmark - Data Protection](https://docs.microsoft.com/security/benchmark/azure/mcsb-data-protection)
- [Azure Storage Security Guide](https://docs.microsoft.com/azure/storage/blobs/security-recommendations)
- [Storage Account Encryption](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)
- [Private Endpoints for Storage](https://docs.microsoft.com/azure/storage/common/storage-private-endpoints)
- [Azure Storage Monitoring](https://docs.microsoft.com/azure/storage/blobs/monitor-blob-storage)
