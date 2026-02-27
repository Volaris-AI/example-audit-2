# Azure Hosting Security Audit Templates

Comprehensive Azure infrastructure security audit templates designed for agentic coding tools (GitHub Copilot CLI/IDE) with Azure CLI access.

## Overview

These templates enable AI coding agents to perform systematic security audits of Azure infrastructure by executing Azure CLI commands and analyzing results against security benchmarks.

## Prerequisites

### Azure CLI Setup
```bash
# Verify Azure CLI installation
az --version

# Login to Azure
az login

# Set subscription
az account set --subscription "<subscription-id>"

# Verify permissions
az role assignment list --assignee $(az ad signed-in-user show --query id -o tsv)
```

### Required Permissions
- **Reader** role minimum (for read-only audits)
- **Security Reader** role (for Defender for Cloud)
- **Monitoring Reader** role (for Log Analytics)
- Additional permissions for specific resource types as needed

### Tool Requirements
- Azure CLI 2.50.0 or higher
- jq (JSON processor)
- GitHub Copilot CLI or IDE with terminal access
- Access to subscription(s) to audit

## Audit Workflow

### 1. Planning Phase
- Define audit scope (subscriptions, resource groups, resources)
- Identify compliance requirements (CIS, Azure Security Benchmark)
- Set up output directory for results

### 2. Discovery Phase
```bash
# List all subscriptions
az account list --output table

# List all resource groups
az group list --output table

# Get resource inventory
az resource list --output table
```

### 3. Audit Execution
Use agentic tools to execute templates systematically:
- Identity and Access Management
- Network Security
- Compute Security
- Storage Security
- Database Security
- Logging and Monitoring
- Compliance
- Secrets Management

### 4. Analysis Phase
- Parse CLI outputs
- Compare against security baselines
- Identify gaps and vulnerabilities
- Prioritize findings by severity

### 5. Reporting Phase
- Generate executive summary
- Document findings with evidence
- Provide remediation recommendations
- Track remediation progress

## Templates

### Core Security Templates

1. **[Identity Security](templates/identity-security.md)**
   - Azure AD users, groups, and roles
   - Multi-factor authentication (MFA)
   - Conditional Access policies
   - Privileged Identity Management (PIM)
   - Service principals and managed identities

2. **[Network Security](templates/network-security.md)**
   - Virtual Networks (VNets) and subnets
   - Network Security Groups (NSGs)
   - Azure Firewall and DDoS Protection
   - Application Gateway and WAF
   - Private endpoints and service endpoints

3. **[Compute Security](templates/compute-security.md)**
   - Virtual Machines (VMs)
   - Azure Kubernetes Service (AKS)
   - Container Instances
   - App Service and Function Apps
   - VM extensions and security agents

4. **[Storage Security](templates/storage-security.md)**
   - Storage accounts and access controls
   - Blob storage, Azure Files, Data Lake
   - Encryption at rest and in transit
   - Network access restrictions
   - Immutable storage and soft delete

5. **[Database Security](templates/database-security.md)**
   - Azure SQL Database
   - Cosmos DB
   - PostgreSQL, MySQL, MariaDB
   - Transparent Data Encryption (TDE)
   - Private endpoints and firewall rules

### Monitoring & Compliance Templates

6. **[Logging & Monitoring](templates/logging-monitoring.md)**
   - Azure Monitor and Log Analytics
   - Diagnostic settings
   - Microsoft Defender for Cloud
   - Azure Sentinel
   - Activity logs and alerts

7. **[Compliance](templates/compliance.md)**
   - CIS Azure Foundations Benchmark
   - Azure Security Benchmark
   - Azure Policy compliance
   - Microsoft Defender secure score
   - Regulatory compliance (GDPR, HIPAA, PCI-DSS)

8. **[Secrets Management](templates/secrets-management.md)**
   - Azure Key Vault configuration
   - Managed identities
   - Key and certificate rotation
   - Access policies and RBAC
   - Secrets exposure risks

### Reporting Template

9. **[Executive Summary](templates/executive-summary.md)**
   - Overall security posture
   - Critical findings
   - Risk assessment
   - Remediation roadmap
   - Compliance status

## Using with Agentic Tools

### GitHub Copilot CLI Example

```bash
# Ask Copilot to audit NSG rules
gh copilot suggest "List all Azure Network Security Groups and identify overly permissive rules allowing 0.0.0.0/0"

# Execute suggested command
az network nsg list --query "[].{Name:name, ResourceGroup:resourceGroup}" -o table

# Ask for analysis
gh copilot explain "Analyze these NSG rules for security risks"
```

### GitHub Copilot in IDE

Use inline prompts:
```
@workspace Audit all Azure storage accounts for public access
@terminal Run Azure CLI commands to check blob container public access settings
```

### Agentic Audit Prompt Pattern

```
You are a DevSecOps engineer performing an Azure security audit.

Task: Audit [RESOURCE_TYPE] for security misconfigurations.

Context:
- Subscription: [SUBSCRIPTION_ID]
- Resource Group: [RESOURCE_GROUP]
- Compliance: [CIS/Azure Security Benchmark]

Steps:
1. Run discovery commands to list resources
2. For each resource, check security configurations
3. Compare findings against benchmark
4. Identify risks and provide remediation steps
5. Output structured results in markdown

Use Azure CLI commands. Be thorough and security-focused.
```

## Security Benchmarks

### CIS Azure Foundations Benchmark v2.0
- Identity and Access Management (IAM)
- Microsoft Defender for Cloud
- Storage Accounts
- Database Services
- Logging and Monitoring
- Networking
- Virtual Machines
- Key Vault
- App Service
- Security Center

### Azure Security Benchmark v3
- Network Security (NS)
- Identity Management (IM)
- Privileged Access (PA)
- Data Protection (DP)
- Asset Management (AM)
- Logging and Threat Detection (LT)
- Incident Response (IR)
- Posture and Vulnerability Management (PV)
- Endpoint Security (ES)
- Backup and Recovery (BR)

## Best Practices

### Audit Execution
1. **Run in read-only mode** - Use viewer roles to prevent accidental changes
2. **Document everything** - Save all command outputs for evidence
3. **Use structured output** - JSON/table formats for parsing
4. **Automate where possible** - Script repetitive checks
5. **Verify findings** - Cross-reference multiple data sources

### Security Considerations
- **Never expose credentials** - Use managed identities and Key Vault
- **Limit scope** - Audit only what's necessary
- **Encrypt audit data** - Store results securely
- **Time-bound access** - Use temporary elevated permissions
- **Audit the auditor** - Log all audit activities

### Agentic Tool Usage
- **Be specific in prompts** - Clear instructions yield better results
- **Validate outputs** - AI can hallucinate, verify commands
- **Iterate on findings** - Ask follow-up questions for deeper analysis
- **Combine templates** - Use multiple templates for comprehensive audits
- **Document methodology** - Track what the agent discovered and how

## Output Structure

Recommended directory structure for audit results:

```
azure-audit-YYYY-MM-DD/
├── executive-summary.md
├── raw-data/
│   ├── identity/
│   ├── network/
│   ├── compute/
│   ├── storage/
│   ├── database/
│   ├── logging/
│   └── compliance/
├── findings/
│   ├── critical/
│   ├── high/
│   ├── medium/
│   └── low/
└── remediation/
    ├── immediate-actions.md
    ├── short-term-plan.md
    └── long-term-strategy.md
```

## Common Issues and Solutions

### Authentication Issues
```bash
# Clear cached credentials
az logout
az login

# Use service principal
az login --service-principal -u <app-id> -p <password> --tenant <tenant-id>
```

### Permission Errors
```bash
# Check current permissions
az role assignment list --assignee $(az ad signed-in-user show --query id -o tsv) --all

# Request required roles from admin
```

### Large Environments
```bash
# Use filters to reduce output
az resource list --resource-group <rg-name>

# Export to files for analysis
az resource list --output json > resources.json
```

## Additional Resources

- [Azure CLI Documentation](https://docs.microsoft.com/cli/azure/)
- [CIS Azure Foundations Benchmark](https://www.cisecurity.org/benchmark/azure)
- [Azure Security Benchmark](https://docs.microsoft.com/security/benchmark/azure/)
- [Microsoft Defender for Cloud](https://docs.microsoft.com/azure/defender-for-cloud/)
- [Azure Security Best Practices](https://docs.microsoft.com/azure/security/fundamentals/best-practices-and-patterns)

## Contributing

When creating or updating templates:
- Follow markdown formatting standards
- Include working Azure CLI commands
- Provide agentic prompt examples
- Reference relevant benchmark controls
- Test commands before committing
- Document any prerequisites

## License

These templates are provided as-is for security audit purposes. Always follow your organization's security policies and obtain proper authorization before conducting audits.
