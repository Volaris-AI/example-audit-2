---
genre: hosting
category: identity-security
analysis-type: iac
relevance:
  file-patterns:
    - "**/terraform/**"
    - "**/*.bicep"
    - "**/arm-templates/**"
  keywords:
    - azurerm_role_assignment
    - azurerm_user_assigned_identity
    - azure_ad
    - rbac
    - managed_identity
  config-keys: []
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Azure Identity and Access Security Audit

## Overview

Audit Azure Active Directory (Azure AD/Entra ID), Role-Based Access Control (RBAC), Multi-Factor Authentication (MFA), Conditional Access, and Privileged Identity Management (PIM) configurations.

## Compliance References

- **CIS Azure Foundations v2.0**: Section 1 (Identity and Access Management)
- **Azure Security Benchmark**: IM (Identity Management), PA (Privileged Access)
- **NIST**: AC (Access Control)

## Discovery Commands

### List All Azure AD Users

```bash
# List all users
az ad user list --output table

# Get detailed user information
az ad user list --query "[].{UPN:userPrincipalName, DisplayName:displayName, AccountEnabled:accountEnabled, UserType:userType}" --output table

# Export to JSON for analysis
az ad user list --output json > users.json
```

### List Azure AD Groups

```bash
# List all groups
az ad group list --output table

# Get group members
az ad group member list --group <group-name> --output table

# List groups with owner information
az ad group list --query "[].{Name:displayName, Description:description}" --output table
```

### List Service Principals and Applications

```bash
# List all service principals
az ad sp list --all --output table

# List applications
az ad app list --output table

# Show service principal credentials expiration
az ad sp list --all --query "[].{DisplayName:displayName, AppId:appId}" --output table
```

### RBAC Role Assignments

```bash
# List all role assignments in subscription
az role assignment list --all --output table

# List role assignments for a specific scope
az role assignment list --resource-group <rg-name> --output table

# List role assignments for a specific user
az role assignment list --assignee <user-upn> --all --output table

# List custom roles
az role definition list --custom-role-only true --output table

# Get details of privileged roles
az role assignment list --role Owner --all --output table
az role assignment list --role Contributor --all --output table
az role assignment list --role "User Access Administrator" --all --output table
```

### Managed Identities

```bash
# List all managed identities
az identity list --output table

# List VMs with system-assigned managed identity
az vm list --query "[?identity.type=='SystemAssigned'].{Name:name, ResourceGroup:resourceGroup, PrincipalId:identity.principalId}" --output table

# List App Services with managed identity
az webapp list --query "[?identity.type=='SystemAssigned'].{Name:name, ResourceGroup:resourceGroup}" --output table
```

<!-- analysis: iac -->
## Security Checks

### 1. Multi-Factor Authentication (MFA)

**CIS Control 1.1**: Ensure that multi-factor authentication is enabled for all privileged users

```bash
# Note: MFA status requires Microsoft Graph API or Azure AD PowerShell
# Using Azure CLI with Graph API

# Get MFA status for all users (requires appropriate permissions)
az rest --method GET --uri "https://graph.microsoft.com/v1.0/reports/credentialUserRegistrationDetails" --output json

# Alternative: Check Conditional Access policies requiring MFA
az rest --method GET --uri "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" --output json
```

**Manual Check Required**: Azure Portal > Azure AD > Users > Per-user MFA

#### Agentic Prompt Example

```
Analyze MFA configuration for Azure AD:

1. List all users with privileged roles (Owner, Contributor, User Access Administrator)
2. Check if Conditional Access policies enforce MFA for these users
3. Identify any privileged accounts without MFA requirement
4. Provide remediation steps for any findings

Output findings in a table with: Username, Roles, MFA Status, Risk Level
```

### 2. Guest User Access

**CIS Control 1.2**: Ensure that 'Guest users permissions are limited' is set to 'Yes'

```bash
# List all guest users
az ad user list --filter "userType eq 'Guest'" --output table

# Check guest user settings
az rest --method GET --uri "https://graph.microsoft.com/v1.0/policies/authorizationPolicy" --output json | jq '.guestUserRoleId'

# List external identities
az ad user list --query "[?userType=='Guest'].{UPN:userPrincipalAccount, DisplayName:displayName, CreatedDateTime:createdDateTime}" --output table
```

#### Security Questions
- Are guest accounts still active and needed?
- Do guests have excessive permissions?
- Is guest access regularly reviewed?

### 3. Privileged Role Assignments

**CIS Control 1.3**: Ensure that there are no guest users with privileged roles

```bash
# Check for guest users with Owner role
az role assignment list --role Owner --all --query "[].{Assignee:principalName, Scope:scope, PrincipalType:principalType}" --output table

# Check for guest users with Contributor role
az role assignment list --role Contributor --all --query "[].{Assignee:principalName, Scope:scope, PrincipalType:principalType}" --output table

# Script to identify guest users with privileged roles
for role in "Owner" "Contributor" "User Access Administrator"; do
  echo "Checking role: $role"
  az role assignment list --role "$role" --all --output json | jq -r '.[] | select(.principalType == "User") | "\(.principalName) - \(.roleDefinitionName)"'
done
```

### 4. Service Principal Credential Expiration

**Azure Security Benchmark IM-2**: Manage application identities securely and automatically

```bash
# List service principals with credentials
az ad sp list --all --query "[].{DisplayName:displayName, AppId:appId}" --output table

# Check credential expiration (requires Graph API)
az rest --method GET --uri "https://graph.microsoft.com/v1.0/applications" --output json | jq '.value[] | {displayName, appId, passwordCredentials, keyCredentials}'

# Find service principals with expired credentials
# Note: This requires custom scripting or Graph API access
```

#### Agentic Prompt Example

```
Audit service principal security:

1. List all service principals in the subscription
2. For each service principal, check:
   - Credential expiration dates
   - Credentials expiring in next 30 days
   - Expired credentials
   - Role assignments and permissions
3. Identify service principals with:
   - Owner or Contributor roles
   - No credential expiration set
   - Credentials older than 1 year
4. Generate a report with remediation recommendations
```

### 5. Conditional Access Policies

**Azure Security Benchmark IM-7**: Eliminate unintended credential exposure

```bash
# List all Conditional Access policies
az rest --method GET --uri "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" --output json | jq '.value[] | {displayName, state, conditions, grantControls}'

# Check for policies requiring MFA
az rest --method GET --uri "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" --output json | jq '.value[] | select(.grantControls.builtInControls[] == "mfa") | {displayName, state}'

# Check for policies blocking legacy authentication
az rest --method GET --uri "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" --output json | jq '.value[] | select(.conditions.clientAppTypes[] == "exchangeActiveSync") | {displayName, grantControls}'
```

#### Security Checks
- ✅ MFA required for all users
- ✅ Legacy authentication blocked
- ✅ Device compliance required for sensitive resources
- ✅ Impossible travel conditions configured
- ✅ Sign-in risk policies enabled

### 6. Privileged Identity Management (PIM)

**CIS Control 1.23**: Ensure that 'Privileged Identity Management' is used to manage roles

```bash
# Check if PIM is enabled (requires Azure AD Premium P2)
# List role assignments eligible for PIM
az rest --method GET --uri "https://graph.microsoft.com/v1.0/privilegedAccess/azureResources" --output json

# List active PIM assignments
az rest --method GET --uri "https://graph.microsoft.com/beta/privilegedAccess/azureResources/roleAssignments?$filter=assignmentState eq 'Active'" --output json
```

**Manual Check Required**: Azure Portal > Azure AD > Privileged Identity Management

### 7. Security Defaults

```bash
# Check if security defaults are enabled
az rest --method GET --uri "https://graph.microsoft.com/v1.0/policies/identitySecurityDefaultsEnforcementPolicy" --output json | jq '{isEnabled}'
```

### 8. Password Policies

```bash
# Get password policy settings
az rest --method GET --uri "https://graph.microsoft.com/v1.0/domains" --output json | jq '.value[] | {id, passwordValidityPeriodInDays, passwordNotificationWindowInDays}'

# Check for password expiration settings
az rest --method GET --uri "https://graph.microsoft.com/v1.0/organization" --output json | jq '.[].passwordPolicies'
```

### 9. Sign-in Logs and Risk Detection

```bash
# Get recent sign-in logs (last 7 days)
az rest --method GET --uri "https://graph.microsoft.com/v1.0/auditLogs/signIns?\$top=100" --output json

# Get risky users
az rest --method GET --uri "https://graph.microsoft.com/v1.0/identityProtection/riskyUsers" --output json

# Get risk detections
az rest --method GET --uri "https://graph.microsoft.com/v1.0/identityProtection/riskDetections" --output json | jq '.value[] | {userDisplayName, riskType, riskLevel, detectedDateTime}'
```

## Penetration Testing Perspective

### Attack Vectors to Check

1. **Privilege Escalation**
   - Users with "User Access Administrator" role
   - Service principals with excessive permissions
   - Guest users with privileged access

2. **Credential Theft**
   - Service principals with permanent credentials
   - Applications storing credentials in code
   - No MFA on privileged accounts

3. **Persistence Mechanisms**
   - Unused or orphaned service principals
   - External user accounts not reviewed
   - Custom roles with excessive permissions

4. **Lateral Movement**
   - Overly permissive RBAC assignments
   - Service principals with access across resource groups
   - Managed identities with broad permissions

### Red Team Commands

```bash
# Enumerate current user's permissions
az role assignment list --assignee $(az ad signed-in-user show --query id -o tsv) --all

# Find resources you have access to
az resource list --query "[].{Name:name, Type:type, ResourceGroup:resourceGroup}" --output table

# Check for role assignments at subscription scope (broadest access)
az role assignment list --scope /subscriptions/$(az account show --query id -o tsv) --output table

# Identify potential privilege escalation paths
az role definition list --query "[?contains(permissions[0].actions[], '*')].{RoleName:roleName, Actions:permissions[0].actions}" --output table
```

## DevSecOps Checks

### CI/CD Pipeline Security

```bash
# List service principals used in pipelines
az ad sp list --show-mine --output table

# Check for service principals with Owner role (should use least privilege)
az role assignment list --role Owner --assignee <sp-app-id> --output table

# Verify managed identity usage in Azure DevOps / GitHub Actions
az webapp identity show --name <app-name> --resource-group <rg-name>
```

### Automation Recommendations

1. **Rotate service principal credentials** every 90 days
2. **Use managed identities** instead of service principals where possible
3. **Implement PIM** for just-in-time access
4. **Enable Conditional Access** for all users
5. **Monitor sign-in logs** for anomalies

## Agentic Audit Workflow

### Complete Identity Audit Prompt

```
You are a security engineer auditing Azure AD identity and access controls.

Subscription: <subscription-id>
Compliance Framework: CIS Azure Foundations Benchmark v2.0

Execute the following audit:

Phase 1 - Discovery:
1. List all Azure AD users (regular and guest)
2. List all service principals and applications
3. Enumerate all RBAC role assignments
4. Identify managed identities in use

Phase 2 - Security Analysis:
1. Check MFA enforcement status
2. Identify privileged role assignments (Owner, Contributor, UAA)
3. Find guest users with elevated permissions
4. Check service principal credential expiration
5. Review Conditional Access policies
6. Verify PIM configuration

Phase 3 - Risk Assessment:
For each finding, assign risk level (Critical, High, Medium, Low):
- Critical: No MFA on privileged accounts
- High: Guest users with Owner role
- Medium: Service principals with credentials expiring soon
- Low: Unused guest accounts

Phase 4 - Reporting:
Generate a markdown report with:
- Executive summary of identity security posture
- Table of findings with risk levels
- Remediation recommendations with Azure CLI commands
- CIS control compliance status

Output Format:
# Identity Security Audit Report

## Executive Summary
[Overall assessment]

## Findings
| Risk | Finding | Evidence | CIS Control | Remediation |
|------|---------|----------|-------------|-------------|
| ... | ... | ... | ... | ... |

## Detailed Analysis
[For each finding, provide details and commands to fix]

## Compliance Summary
[CIS controls pass/fail status]
```

## Remediation Examples

### Enable MFA for a User

```bash
# MFA must be enabled through Azure Portal or Graph API
# Conditional Access is the recommended approach

# Example: Create Conditional Access policy requiring MFA
az rest --method POST \
  --uri "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" \
  --body '{
    "displayName": "Require MFA for all users",
    "state": "enabled",
    "conditions": {
      "users": {
        "includeUsers": ["All"]
      },
      "applications": {
        "includeApplications": ["All"]
      }
    },
    "grantControls": {
      "operator": "OR",
      "builtInControls": ["mfa"]
    }
  }'
```

### Remove Excessive Role Assignment

```bash
# Remove a role assignment
az role assignment delete --assignee <user-or-sp-id> --role <role-name> --scope <scope>

# Example: Remove Owner role from a user at resource group scope
az role assignment delete \
  --assignee user@domain.com \
  --role Owner \
  --resource-group <rg-name>
```

### Rotate Service Principal Credentials

```bash
# Reset service principal credentials
az ad sp credential reset --id <app-id>

# Create a new credential with expiration
az ad sp credential reset --id <app-id> --years 1
```

### Remove Guest User

```bash
# Delete a guest user
az ad user delete --id <user-upn>

# Alternatively, revoke all sessions and disable account
az ad user update --id <user-upn> --account-enabled false
```

## Output Checklist

- [ ] All users listed with privilege levels
- [ ] MFA enforcement status documented
- [ ] Guest users identified and reviewed
- [ ] Service principal credentials audited
- [ ] RBAC assignments mapped to least privilege
- [ ] Conditional Access policies reviewed
- [ ] PIM status checked for privileged roles
- [ ] Managed identities documented
- [ ] Custom roles analyzed for excessive permissions
- [ ] Sign-in logs reviewed for anomalies

## References

- [CIS Azure Foundations Benchmark](https://www.cisecurity.org/benchmark/azure)
- [Azure Security Benchmark - Identity Management](https://docs.microsoft.com/security/benchmark/azure/mcsb-identity-management)
- [Azure AD Best Practices](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-deployment-checklist-p2)
- [Conditional Access Documentation](https://docs.microsoft.com/azure/active-directory/conditional-access/)
- [Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/)
