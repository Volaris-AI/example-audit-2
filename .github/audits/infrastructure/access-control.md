---
genre: infrastructure
category: access-control
analysis-type: static
relevance:
  file-patterns:
    - "**/rbac/**"
    - "**/roles/**"
    - "**/permissions/**"
  keywords:
    - "rbac"
    - "role"
    - "permission"
    - "authorize"
    - "guard"
    - "policy"
  config-keys:
    - "casl"
    - "accesscontrol"
    - "casbin"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Access Control Infrastructure Audit

## System Information
- **System Name**: 
- **Audit Date**: 
- **Auditor**: 

<!-- analysis: static -->

## Maturity Level Assessment

| Level | Description | Model | Enforcement | Management |
|-------|-------------|-------|-------------|------------|
| **5** | Zero-trust, dynamic | ABAC, Policy-Based, Dynamic | Fine-grained, context-aware, realtime | Automated, IaC, audited |
| **4** | Modern, attribute-based | RBAC + Attributes, Claims | Centralized, API gateway | Managed service, version controlled |
| **3** | Role-based, adequate | RBAC | Middleware, guards | Manual but documented |
| **2** | Basic roles | Simple roles | Application-level | Ad-hoc |
| **1** | No access control | None or hardcoded | None | None |

### Current Maturity Score: [ ] / 5

## Assessment Areas

### 1. Access Control Model
- [ ] **Model type**: None / ACL / RBAC / ABAC / ReBAC / Policy-based
- [ ] **Granularity**: Coarse / Medium / Fine-grained
- [ ] **Dynamic policies** based on context
- [ ] **Least privilege** principle enforced
- [ ] **Separation of duties**

### 2. Authorization Enforcement
- [ ] **Enforcement point**: Application / API Gateway / Service Mesh / WAF
- [ ] **Centralized** authorization service
- [ ] **Policy decision point** (PDP)
- [ ] **Policy enforcement point** (PEP)
- [ ] **Consistent** across all services

### 3. Permission Management
- [ ] **Permission assignment** process
- [ ] **Role management** documented
- [ ] **Permission review** regular (quarterly/annual)
- [ ] **Orphaned accounts** removal
- [ ] **Temporary access** management

### 4. API & Resource Authorization
- [ ] **OAuth scopes** for APIs
- [ ] **Resource-level** permissions
- [ ] **Field-level** security (if needed)
- [ ] **Rate limiting** per user/role
- [ ] **Audit logging** of access decisions

### 5. Administrative Access
- [ ] **Just-in-time** (JIT) admin access
- [ ] **Privileged Access Management** (PAM)
- [ ] **Admin actions** logged
- [ ] **Approval workflow** for sensitive access
- [ ] **Break-glass** procedures

## Recommendations
**Level 1→2**: Implement basic RBAC, document roles
**Level 2→3**: Centralize authorization, add middleware enforcement
**Level 3→4**: Implement ABAC, policy-based access control, automated management
**Level 4→5**: Zero-trust model, dynamic policies, continuous authorization

## Success Criteria
- All users assigned appropriate roles
- Least privilege enforced
- Regular access reviews completed
- Zero unauthorized access incidents

---
**Document Version**: 1.0
