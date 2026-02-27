---
genre: hosting
category: database-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/*.bicep"
  keywords:
    - azurerm_sql_server
    - azurerm_cosmosdb
    - azurerm_postgresql
    - azurerm_mysql
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Azure Database Security Audit

## Overview

Audit security configurations for Azure SQL Database, Azure Cosmos DB, Azure Database for PostgreSQL, MySQL, MariaDB, and other database services.

## Compliance References

- **CIS Azure Foundations v2.0**: Section 4 (Database Services)
- **Azure Security Benchmark**: DP (Data Protection), IM (Identity Management)
- **NIST**: SC (System and Communications Protection), IA (Identification and Authentication)

## Discovery Commands

### Azure SQL Database

```bash
# List all SQL servers
az sql server list --output table

# Get SQL server details
az sql server list --query "[].{Name:name, ResourceGroup:resourceGroup, Location:location, AdminUser:administratorLogin, MinTLSVersion:minimalTlsVersion}" --output table

# List all SQL databases
az sql db list --server <server-name> --resource-group <rg-name> --output table

# Get database details across all servers
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  echo "Server: $server"
  az sql db list --server "$server" --resource-group "$rg" --output table
done
```

### Azure Cosmos DB

```bash
# List all Cosmos DB accounts
az cosmosdb list --output table

# Get Cosmos DB account details
az cosmosdb list --query "[].{Name:name, ResourceGroup:resourceGroup, Location:location, PublicNetworkAccess:publicNetworkAccess, Kind:kind}" --output table

# List Cosmos DB databases
az cosmosdb sql database list --account-name <cosmos-account-name> --resource-group <rg-name> --output table

# List containers
az cosmosdb sql container list --account-name <cosmos-account-name> --resource-group <rg-name> --database-name <database-name> --output table
```

### Azure Database for PostgreSQL

```bash
# List PostgreSQL servers
az postgres server list --output table

# Get PostgreSQL server details
az postgres server list --query "[].{Name:name, ResourceGroup:resourceGroup, AdminUser:administratorLogin, SSLEnforcement:sslEnforcement, MinTLSVersion:minimalTlsVersion}" --output table

# List PostgreSQL flexible servers
az postgres flexible-server list --output table

# List databases
az postgres db list --server-name <server-name> --resource-group <rg-name> --output table
```

### Azure Database for MySQL

```bash
# List MySQL servers
az mysql server list --output table

# Get MySQL server details
az mysql server list --query "[].{Name:name, ResourceGroup:resourceGroup, AdminUser:administratorLogin, SSLEnforcement:sslEnforcement, MinTLSVersion:minimalTlsVersion}" --output table

# List MySQL flexible servers
az mysql flexible-server list --output table

# List databases
az mysql db list --server-name <server-name> --resource-group <rg-name> --output table
```

### Azure Database for MariaDB

```bash
# List MariaDB servers
az mariadb server list --output table

# Get MariaDB server details
az mariadb server list --query "[].{Name:name, ResourceGroup:resourceGroup, AdminUser:administratorLogin, SSLEnforcement:sslEnforcement}" --output table

# List databases
az mariadb db list --server-name <server-name> --resource-group <rg-name> --output table
```

<!-- analysis: iac -->
## Security Checks

### 1. Azure SQL Database - Advanced Data Security

**CIS Control 4.1.1**: Ensure that 'Auditing' is set to 'On'

```bash
# Check server-level auditing
az sql server audit-policy show --server <server-name> --resource-group <rg-name> --output json

# Check database-level auditing
az sql db audit-policy show --server <server-name> --database <database-name> --resource-group <rg-name> --output json

# Script to audit all SQL servers
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  echo "=== SQL Server: $server ==="
  audit=$(az sql server audit-policy show --server "$server" --resource-group "$rg" --query "state" --output tsv)
  echo "Auditing: $audit"
  echo ""
done
```

**CIS Control 4.1.2**: Ensure that 'Data encryption' is set to 'On' on a SQL Database

```bash
# Check Transparent Data Encryption (TDE) status
az sql db tde show --server <server-name> --database <database-name> --resource-group <rg-name> --output json

# Check TDE for all databases
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  echo "SQL Server: $server"
  az sql db list --server "$server" --resource-group "$rg" --query "[].name" --output tsv | while read db; do
    if [ "$db" != "master" ]; then
      tde=$(az sql db tde show --server "$server" --database "$db" --resource-group "$rg" --query "state" --output tsv 2>/dev/null)
      echo "  Database: $db - TDE: $tde"
    fi
  done
  echo ""
done
```

**CIS Control 4.1.3**: Ensure that 'Auditing' Retention is 'greater than 90 days'

```bash
# Check auditing retention period
az sql server audit-policy show --server <server-name> --resource-group <rg-name> --query "retentionDays" --output tsv

# Find servers with insufficient retention
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  retention=$(az sql server audit-policy show --server "$server" --resource-group "$rg" --query "retentionDays" --output tsv 2>/dev/null)
  if [ -n "$retention" ] && [ "$retention" -lt 90 ]; then
    echo "ISSUE: $server has retention of only $retention days"
  fi
done
```

#### Agentic Prompt Example

```
Audit Azure SQL Database security:

1. List all SQL servers and databases
2. For each SQL server, check:
   - Auditing status (On/Off)
   - Auditing retention period (should be 90+ days)
   - Transparent Data Encryption (TDE) status
   - Advanced Data Security (Microsoft Defender for SQL)
   - Firewall rules (identify 0.0.0.0-255.255.255.255 rules)
   - Azure AD authentication configuration
   - Minimum TLS version (should be 1.2+)
3. For each database, verify:
   - TDE enabled
   - Diagnostic logging configured
   - Private endpoint usage
4. Risk assessment:
   - Critical: TDE disabled, auditing off, public internet access
   - High: Weak firewall rules, no Azure AD auth, retention < 90 days
   - Medium: Suboptimal TLS version, missing advanced threat protection
5. Provide remediation commands

Output format:
| Server | Database | TDE | Auditing | Retention | Public Access | Risk | Remediation |
```

### 2. SQL Server Firewall Rules

**CIS Control 4.1.4**: Ensure that Azure Active Directory Admin is configured

```bash
# Check Azure AD administrator
az sql server ad-admin list --server <server-name> --resource-group <rg-name> --output table

# List SQL servers without Azure AD admin
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  ad_admin=$(az sql server ad-admin list --server "$server" --resource-group "$rg" --output tsv 2>/dev/null)
  if [ -z "$ad_admin" ]; then
    echo "NO AD ADMIN: $server"
  fi
done
```

**CIS Control 4.2.1**: Ensure that the server firewall rules are not open to the Internet

```bash
# List firewall rules for SQL server
az sql server firewall-rule list --server <server-name> --resource-group <rg-name> --output table

# Find overly permissive firewall rules
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  echo "SQL Server: $server"
  az sql server firewall-rule list --server "$server" --resource-group "$rg" --query "[].{Name:name, StartIP:startIpAddress, EndIP:endIpAddress}" --output table
  
  # Check for rules allowing all IPs (0.0.0.0 to 255.255.255.255)
  az sql server firewall-rule list --server "$server" --resource-group "$rg" --query "[?startIpAddress=='0.0.0.0' && endIpAddress=='255.255.255.255'].name" --output tsv | while read rule; do
    echo "  CRITICAL: Rule '$rule' allows all internet IPs"
  done
  echo ""
done
```

### 3. Private Endpoints for SQL

**Azure Security Benchmark DP-3**: Monitor for unauthorized transfer of sensitive data

```bash
# Check if SQL server has private endpoints
az sql server show --name <server-name> --resource-group <rg-name> --query "privateEndpointConnections" --output json

# List SQL servers without private endpoints
az sql server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  private_endpoints=$(az sql server show --name "$server" --resource-group "$rg" --query "privateEndpointConnections" --output tsv 2>/dev/null)
  public_access=$(az sql server show --name "$server" --resource-group "$rg" --query "publicNetworkAccess" --output tsv 2>/dev/null)
  
  if [ -z "$private_endpoints" ]; then
    echo "No private endpoint: $server (Public Access: $public_access)"
  fi
done
```

### 4. Cosmos DB Security

**Azure Security Benchmark**: Cosmos DB encryption and access control

```bash
# Check Cosmos DB public network access
az cosmosdb list --query "[].{Name:name, ResourceGroup:resourceGroup, PublicNetworkAccess:publicNetworkAccess}" --output table

# Check Cosmos DB firewall rules
az cosmosdb show --name <cosmos-account-name> --resource-group <rg-name> --query "ipRules" --output json

# Check if Cosmos DB uses virtual network rules
az cosmosdb show --name <cosmos-account-name> --resource-group <rg-name> --query "virtualNetworkRules" --output json

# Check Cosmos DB private endpoints
az cosmosdb show --name <cosmos-account-name> --resource-group <rg-name> --query "privateEndpointConnections" --output json

# Check Cosmos DB key-based authentication
az cosmosdb show --name <cosmos-account-name> --resource-group <rg-name> --query "disableKeyBasedMetadataWriteAccess" --output tsv

# Check automatic failover
az cosmosdb show --name <cosmos-account-name> --resource-group <rg-name> --query "enableAutomaticFailover" --output tsv

# List Cosmos DB accounts with issues
az cosmosdb list --query "[?publicNetworkAccess=='Enabled' && (ipRules==null || length(ipRules)==\`0\`)].{Name:name, ResourceGroup:resourceGroup}" --output table
```

### 5. PostgreSQL/MySQL Security

**CIS Control 4.3.1**: Ensure 'Enforce SSL connection' is set to 'ENABLED' for PostgreSQL Database Server

```bash
# Check SSL enforcement for PostgreSQL
az postgres server show --name <server-name> --resource-group <rg-name> --query "sslEnforcement" --output tsv

# Check minimum TLS version
az postgres server show --name <server-name> --resource-group <rg-name> --query "minimalTlsVersion" --output tsv

# Check all PostgreSQL servers
az postgres server list --query "[].{Name:name, ResourceGroup:resourceGroup, SSL:sslEnforcement, MinTLS:minimalTlsVersion}" --output table

# Check flexible servers
az postgres flexible-server list --query "[].{Name:name, ResourceGroup:resourceGroup}" --output table
```

**Check MySQL SSL enforcement**

```bash
# Check SSL enforcement for MySQL
az mysql server show --name <server-name> --resource-group <rg-name> --query "sslEnforcement" --output tsv

# Check minimum TLS version
az mysql server show --name <server-name> --resource-group <rg-name> --query "minimalTlsVersion" --output tsv

# List all MySQL servers
az mysql server list --query "[].{Name:name, ResourceGroup:resourceGroup, SSL:sslEnforcement, MinTLS:minimalTlsVersion}" --output table
```

**CIS Control 4.3.2**: Ensure Server Parameter 'log_checkpoints' is set to 'ON' for PostgreSQL Database Server

```bash
# Check log_checkpoints parameter
az postgres server configuration show --server-name <server-name> --resource-group <rg-name> --name log_checkpoints --output tsv

# Check connection throttling
az postgres server configuration show --server-name <server-name> --resource-group <rg-name> --name connection_throttling --output tsv

# Script to check critical parameters
CRITICAL_PARAMS=("log_checkpoints" "log_connections" "log_disconnections" "connection_throttling" "log_retention_days")

az postgres server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  echo "=== PostgreSQL Server: $server ==="
  for param in "${CRITICAL_PARAMS[@]}"; do
    value=$(az postgres server configuration show --server-name "$server" --resource-group "$rg" --name "$param" --query "value" --output tsv 2>/dev/null)
    echo "$param: $value"
  done
  echo ""
done
```

**CIS Control 4.3.8**: Ensure 'Allow access to Azure services' for PostgreSQL Database Server is disabled

```bash
# Check for firewall rule allowing Azure services
az postgres server firewall-rule list --server-name <server-name> --resource-group <rg-name> --query "[?name=='AllowAllWindowsAzureIps'].{Name:name, StartIP:startIpAddress, EndIP:endIpAddress}" --output table

# Find PostgreSQL servers with Azure services access enabled
az postgres server list --query "[].{Name:name, RG:resourceGroup}" --output tsv | while read server rg; do
  rule=$(az postgres server firewall-rule list --server-name "$server" --resource-group "$rg" --query "[?startIpAddress=='0.0.0.0' && endIpAddress=='0.0.0.0'].name" --output tsv 2>/dev/null)
  if [ -n "$rule" ]; then
    echo "ISSUE: $server allows Azure services access (rule: $rule)"
  fi
done
```

### 6. Database Private Endpoints

```bash
# Check SQL server private endpoints
az sql server show --name <server-name> --resource-group <rg-name> --query "privateEndpointConnections[].{ID:id, Status:privateLinkServiceConnectionState.status}" --output table

# Check PostgreSQL private endpoints
az postgres server show --name <server-name> --resource-group <rg-name> --query "privateEndpointConnections" --output json

# Check MySQL private endpoints
az mysql server show --name <server-name> --resource-group <rg-name> --query "privateEndpointConnections" --output json

# Check Cosmos DB private endpoints
az cosmosdb show --name <cosmos-account-name> --resource-group <rg-name> --query "privateEndpointConnections[].{ID:id, Status:privateLinkServiceConnectionState.status}" --output table
```

### 7. Database Backup and Recovery

```bash
# Check SQL database backup retention
az sql db show --server <server-name> --database <database-name> --resource-group <rg-name> --query "retentionDays" --output tsv

# Check long-term retention policy
az sql db ltr-policy show --server <server-name> --database <database-name> --resource-group <rg-name> --output json

# Check PostgreSQL backup retention
az postgres server show --name <server-name> --resource-group <rg-name> --query "{Name:name, BackupRetention:storageProfile.backupRetentionDays, GeoRedundant:storageProfile.geoRedundantBackup}" --output json

# Check MySQL backup retention
az mysql server show --name <server-name> --resource-group <rg-name> --query "{Name:name, BackupRetention:storageProfile.backupRetentionDays, GeoRedundant:storageProfile.geoRedundantBackup}" --output json

# Check Cosmos DB backup policy
az cosmosdb show --name <cosmos-account-name> --resource-group <rg-name> --query "backupPolicy" --output json
```

### 8. Microsoft Defender for SQL

**CIS Control 4.1.5**: Ensure that Advanced Data Security on a SQL server is set to 'On'

```bash
# Check Microsoft Defender for SQL (formerly Advanced Data Security)
az security setting show --name MCAS --query "enabled" --output tsv

# Check threat detection policy
az sql server threat-policy show --server <server-name> --resource-group <rg-name> --output json

# Check vulnerability assessment
az sql server va-scan list --server <server-name> --resource-group <rg-name> --database <database-name> --output table
```

## Penetration Testing Perspective

### Attack Vectors to Check

1. **SQL Injection Points**
   - Application connections using SQL authentication
   - Dynamic SQL queries without parameterization
   - Stored procedures with dynamic SQL

2. **Credential Theft**
   - SQL authentication usernames/passwords in connection strings
   - Hardcoded credentials in code
   - Connection strings in configuration files
   - Credentials in environment variables

3. **Network Access**
   - Public-facing database servers
   - Overly permissive firewall rules
   - Missing private endpoint configuration
   - Database accessible from internet

4. **Data Exfiltration**
   - No auditing or monitoring
   - Missing encryption in transit (SSL/TLS)
   - Missing encryption at rest (TDE)
   - Weak access controls

5. **Privilege Escalation**
   - SQL users with sysadmin/server-level permissions
   - Database users with db_owner role
   - Azure AD users with excessive RBAC permissions

### Red Team Commands

```bash
# Enumerate database servers
az sql server list --query "[].{Name:name, FQDN:fullyQualifiedDomainName, AdminUser:administratorLogin}" --output table
az postgres server list --query "[].{Name:name, FQDN:fullyQualifiedDomainName, AdminUser:administratorLogin}" --output table
az mysql server list --query "[].{Name:name, FQDN:fullyQualifiedDomainName, AdminUser:administratorLogin}" --output table

# Check for publicly accessible databases
az sql server firewall-rule list --server <server-name> --resource-group <rg-name> --query "[?startIpAddress=='0.0.0.0'].{Name:name, StartIP:startIpAddress, EndIP:endIpAddress}" --output table

# Attempt connection to database (requires credentials)
psql "host=<server-name>.postgres.database.azure.com port=5432 dbname=<database-name> user=<username> sslmode=require"
mysql -h <server-name>.mysql.database.azure.com -u <username> -p

# Check for SQL Server on non-standard ports
nmap -p 1433,1434 <server-fqdn>
```

## DevSecOps Checks

### Infrastructure as Code Security

```bash
# Export database configuration for IaC validation
az sql server show --name <server-name> --resource-group <rg-name> --output json > sql-config.json
az postgres server show --name <server-name> --resource-group <rg-name> --output json > postgres-config.json

# Check for hardcoded credentials in IaC
grep -r "administratorLoginPassword\|password\|Password" *.tf *.json *.yml *.yaml | grep -v "null"

# Verify secure defaults in Terraform/Bicep
# - sslEnforcement = Enabled
# - minimalTlsVersion = TLS1_2
# - publicNetworkAccess = Disabled
# - firewallRules should be specific, not 0.0.0.0-255.255.255.255
```

### CI/CD Pipeline Security

```bash
# Check for database credentials in pipeline variables
# This is environment-specific

# Verify managed identity usage for database access
az sql server show --name <server-name> --resource-group <rg-name> --query "identity" --output json
```

## Agentic Audit Workflow

### Complete Database Security Audit Prompt

```
You are a database security engineer auditing Azure database services.

Subscription: <subscription-id>
Compliance Framework: CIS Azure Foundations Benchmark v2.0, Azure Security Benchmark

Execute comprehensive database security audit:

Phase 1 - Discovery and Inventory:
1. List all SQL servers and databases
2. Enumerate Cosmos DB accounts
3. List PostgreSQL servers
4. List MySQL servers
5. List MariaDB servers

Phase 2 - Azure SQL Security Analysis:

A. Encryption:
- Verify Transparent Data Encryption (TDE) is enabled
- Check if Customer Managed Keys are used
- Verify encryption in transit (TLS 1.2+)

B. Auditing and Monitoring:
- Check if auditing is enabled
- Verify retention period (90+ days)
- Check diagnostic settings
- Verify Microsoft Defender for SQL is enabled

C. Authentication:
- Check if Azure AD authentication is configured
- Verify SQL authentication is disabled or restricted
- Check for service principal/managed identity usage

D. Network Security:
- Review firewall rules (identify 0.0.0.0-255.255.255.255 rules)
- Check public network access setting
- Verify private endpoint usage
- Check virtual network rules

E. Backup and Recovery:
- Verify backup retention period
- Check long-term retention policy
- Verify geo-redundant backup configuration
- Check point-in-time restore capability

Phase 3 - Cosmos DB Security Analysis:

A. Network Security:
- Check public network access
- Review IP firewall rules
- Verify virtual network rules
- Check private endpoint usage

B. Access Control:
- Check if key-based metadata write access is disabled
- Review RBAC assignments
- Verify managed identity usage

C. Availability:
- Check automatic failover configuration
- Verify multi-region replication
- Check backup policy

Phase 4 - PostgreSQL/MySQL Security Analysis:

A. Encryption:
- Verify SSL enforcement
- Check minimum TLS version
- Check encryption at rest

B. Server Parameters:
- Check log_checkpoints (PostgreSQL)
- Verify log_connections
- Check connection_throttling
- Verify log retention days

C. Network Security:
- Review firewall rules
- Check if "Allow access to Azure services" is disabled
- Verify private endpoint usage

D. Backup:
- Check backup retention days
- Verify geo-redundant backup

Phase 5 - Risk Assessment:

Assign risk levels:
- Critical: TDE disabled, public internet access with no firewall rules, auditing off
- High: SQL authentication only (no Azure AD), firewall rule 0.0.0.0-255.255.255.255, SSL not enforced
- Medium: Insufficient retention periods, no private endpoints, weak TLS version
- Low: Missing monitoring, no automatic failover

Phase 6 - Compliance Mapping:

For each finding, document:
- CIS Azure Foundations control
- Azure Security Benchmark control
- Regulatory requirement (GDPR, HIPAA, PCI-DSS)

Phase 7 - Attack Scenario Analysis:

For critical/high findings:
1. **SQL Injection**: Risk and impact
2. **Credential Compromise**: What access would attacker gain?
3. **Data Exfiltration**: How could data be stolen?
4. **Data Destruction**: Ransomware/deletion risk

Phase 8 - Reporting:

Generate detailed markdown report:

## Azure Database Security Audit Report

### Executive Summary
[Overall database security posture, critical statistics]

### Database Inventory
| Type | Count | Encryption | Network Security | Auditing | Backup | Risk Level |
|------|-------|------------|------------------|----------|--------|------------|

### Critical Findings

#### [Finding Title]
- **Database**: [SQL Server/Cosmos DB/etc.]
- **Resource**: [Resource name]
- **Risk Level**: Critical
- **Issue**: [What is misconfigured]
- **Evidence**:
  ```bash
  [Azure CLI command and output]
  ```
- **Attack Scenario**: [How attacker could exploit this]
- **Data at Risk**: [Type and sensitivity]
- **Compliance Impact**: [Which controls violated]
- **Remediation**:
  ```bash
  [Step-by-step fix commands]
  ```

### Findings by Database Service

#### Azure SQL Database
[SQL-specific findings]

#### Cosmos DB
[Cosmos-specific findings]

#### PostgreSQL
[PostgreSQL-specific findings]

#### MySQL
[MySQL-specific findings]

### Compliance Summary
| Framework | Control | Requirement | Status | Notes |
|-----------|---------|-------------|--------|-------|

### Remediation Roadmap

#### Immediate Actions (0-7 days)
- [Critical fixes with commands]

#### Short-term (1-3 months)
- [Important improvements]

#### Long-term (3-12 months)
- [Strategic enhancements]

### Best Practices Recommendations
[Database security best practices summary]

Use Azure CLI commands for all checks. Provide detailed, actionable remediation steps.
```

## Remediation Examples

### Enable Azure SQL Auditing

```bash
# Enable server-level auditing
az sql server audit-policy update \
  --resource-group <rg-name> \
  --server <server-name> \
  --state Enabled \
  --log-analytics-workspace-resource-id <workspace-resource-id> \
  --retention-days 90

# Enable database-level auditing
az sql db audit-policy update \
  --resource-group <rg-name> \
  --server <server-name> \
  --name <database-name> \
  --state Enabled \
  --log-analytics-workspace-resource-id <workspace-resource-id>
```

### Enable Transparent Data Encryption (TDE)

```bash
# Enable TDE on SQL database (enabled by default for new databases)
az sql db tde set \
  --resource-group <rg-name> \
  --server <server-name> \
  --database <database-name> \
  --status Enabled

# Check TDE status
az sql db tde show \
  --resource-group <rg-name> \
  --server <server-name> \
  --database <database-name>
```

### Configure Azure AD Administrator for SQL

```bash
# Set Azure AD administrator
az sql server ad-admin create \
  --resource-group <rg-name> \
  --server <server-name> \
  --display-name <admin-display-name> \
  --object-id <admin-object-id>

# Verify Azure AD admin
az sql server ad-admin list \
  --resource-group <rg-name> \
  --server <server-name>
```

### Remove Overly Permissive Firewall Rules

```bash
# Delete firewall rule allowing all IPs
az sql server firewall-rule delete \
  --resource-group <rg-name> \
  --server <server-name> \
  --name <rule-name>

# Add specific IP range
az sql server firewall-rule create \
  --resource-group <rg-name> \
  --server <server-name> \
  --name AllowCorporateNetwork \
  --start-ip-address <start-ip> \
  --end-ip-address <end-ip>

# Disable public network access
az sql server update \
  --resource-group <rg-name> \
  --name <server-name> \
  --public-network-access Disabled
```

### Create Private Endpoint for SQL Server

```bash
# Create private endpoint
az network private-endpoint create \
  --name <endpoint-name> \
  --resource-group <rg-name> \
  --vnet-name <vnet-name> \
  --subnet <subnet-name> \
  --private-connection-resource-id "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Sql/servers/<server-name>" \
  --group-id sqlServer \
  --connection-name <connection-name>

# Create private DNS zone
az network private-dns zone create \
  --resource-group <rg-name> \
  --name privatelink.database.windows.net

# Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group <rg-name> \
  --zone-name privatelink.database.windows.net \
  --name <dns-link-name> \
  --virtual-network <vnet-name> \
  --registration-enabled false
```

### Enable SSL Enforcement for PostgreSQL

```bash
# Enable SSL enforcement
az postgres server update \
  --resource-group <rg-name> \
  --name <server-name> \
  --ssl-enforcement Enabled \
  --minimal-tls-version TLS1_2

# Verify SSL enforcement
az postgres server show \
  --resource-group <rg-name> \
  --name <server-name> \
  --query "{Name:name, SSL:sslEnforcement, MinTLS:minimalTlsVersion}"
```

### Configure PostgreSQL Server Parameters

```bash
# Enable log_checkpoints
az postgres server configuration set \
  --resource-group <rg-name> \
  --server-name <server-name> \
  --name log_checkpoints \
  --value ON

# Enable connection throttling
az postgres server configuration set \
  --resource-group <rg-name> \
  --server-name <server-name> \
  --name connection_throttling \
  --value ON

# Set log retention
az postgres server configuration set \
  --resource-group <rg-name> \
  --server-name <server-name> \
  --name log_retention_days \
  --value 7
```

### Secure Cosmos DB

```bash
# Disable public network access
az cosmosdb update \
  --resource-group <rg-name> \
  --name <cosmos-account-name> \
  --public-network-access Disabled

# Enable virtual network filter
az cosmosdb update \
  --resource-group <rg-name> \
  --name <cosmos-account-name> \
  --enable-virtual-network true

# Add IP firewall rule
az cosmosdb update \
  --resource-group <rg-name> \
  --name <cosmos-account-name> \
  --ip-range-filter <ip-range>

# Disable key-based metadata write access
az cosmosdb update \
  --resource-group <rg-name> \
  --name <cosmos-account-name> \
  --disable-key-based-metadata-write-access true

# Enable automatic failover
az cosmosdb update \
  --resource-group <rg-name> \
  --name <cosmos-account-name> \
  --enable-automatic-failover true
```

## Output Checklist

- [ ] All database servers inventoried by type
- [ ] Encryption at rest verified (TDE for SQL)
- [ ] Encryption in transit verified (SSL/TLS enforced)
- [ ] Auditing enabled with adequate retention
- [ ] Azure AD authentication configured
- [ ] Firewall rules reviewed for overly permissive rules
- [ ] Public network access status documented
- [ ] Private endpoint usage verified
- [ ] Microsoft Defender for SQL status checked
- [ ] Backup retention periods verified
- [ ] Geo-redundant backup configuration checked
- [ ] Server parameters reviewed (PostgreSQL/MySQL)
- [ ] Diagnostic logging configured

## References

- [CIS Azure Foundations Benchmark - Database Services](https://www.cisecurity.org/benchmark/azure)
- [Azure Security Benchmark - Data Protection](https://docs.microsoft.com/security/benchmark/azure/mcsb-data-protection)
- [Azure SQL Database Security](https://docs.microsoft.com/azure/azure-sql/database/security-overview)
- [Cosmos DB Security](https://docs.microsoft.com/azure/cosmos-db/database-security)
- [Azure Database for PostgreSQL Security](https://docs.microsoft.com/azure/postgresql/concepts-security)
- [Transparent Data Encryption](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-tde-overview)
