---
genre: security
category: database
analysis-type: static
relevance:
  file-patterns:
    - "**/models/**"
    - "**/migrations/**"
    - "**/db/**"
    - "**/database/**"
    - "**/schema/**"
    - "**/prisma/**"
  keywords:
    - "sql"
    - "query"
    - "orm"
    - "prisma"
    - "sequelize"
    - "mongoose"
    - "typeorm"
    - "knex"
    - "migration"
    - "schema"
  config-keys:
    - "prisma"
    - "sequelize"
    - "mongoose"
    - "typeorm"
    - "knex"
    - "pg"
    - "mysql2"
    - "mongodb"
  always-include: false
severity-scale: "Critical|High|Medium|Low|Info"
---

# Database Security Assessment

**Assessment Date:** _[YYYY-MM-DD]_
**Auditor:** _[Your name]_
**Application:** _[Application name and version]_
**Status:** _[Draft / In Progress / Complete]_

---

<!-- analysis: static -->

## Executive Summary

**Overall Database Security Rating:** [ ] Excellent [ ] Good [ ] Fair [ ] Poor [ ] Critical

**Key Findings:**
- Total Vulnerabilities: _[Number]_
- Critical: _[Number]_ | High: _[Number]_ | Medium: _[Number]_ | Low: _[Number]_

**Most Critical Issue:** _[Brief description]_

---

## Scope

### Components Assessed
- [ ] SQL injection vulnerabilities
- [ ] NoSQL injection vulnerabilities
- [ ] Database access controls and permissions
- [ ] Data encryption (at rest and in transit)
- [ ] Backup security and recovery
- [ ] Database connection security
- [ ] Stored procedures and functions
- [ ] Database auditing and logging

### Database Types
- [ ] SQL (MySQL, PostgreSQL, SQL Server, Oracle)
- [ ] NoSQL (MongoDB, CouchDB, Redis, Cassandra)
- [ ] Graph (Neo4j)
- [ ] Time-series (InfluxDB)

### Out of Scope
_[List what was not assessed]_

---

## 1. Injection Vulnerabilities

### 1.1 SQL Injection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Parameterized queries used throughout application
- [ ] ORM framework used correctly (no raw queries with user input)
- [ ] Input validation on all database-bound parameters
- [ ] Stored procedures use parameterized inputs
- [ ] No dynamic SQL construction with user input
- [ ] WAF/database firewall in place

**Issues Found:**

| Endpoint/Function | Severity | Payload | Impact |
|-------------------|----------|---------|--------|
| | | | |

**Test Results:**
```
Test: SQL Injection via login
Payload: ' OR '1'='1' --
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 1.2 NoSQL Injection

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] NoSQL queries properly sanitized
- [ ] MongoDB operators ($where, $regex) validated
- [ ] Query objects not built from unsanitized input
- [ ] JSON input properly validated before use in queries
- [ ] Type checking enforced on query parameters
- [ ] Query complexity limits enforced

**Issues Found:**

| Query Location | Severity | Payload | Impact |
|----------------|----------|---------|--------|
| | | | |

**Test Results:**
```
Test: NoSQL Injection in MongoDB
Payload: {"$ne": null}
Result: [Pass/Fail]
Details: [Details]
```

**Recommendations:**
- _[Specific recommendation]_

### 1.3 ORM Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] ORM library is up-to-date
- [ ] Raw SQL queries are avoided where possible
- [ ] ORM mass assignment vulnerabilities prevented
- [ ] Query builder uses parameterization
- [ ] No unsafe deserialization of query objects
- [ ] ORM-specific security features enabled

**Issues Found:**

| ORM | Severity | Issue | Impact |
|-----|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 2. Access Controls & Permissions

### 2.1 Database User Permissions

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Application uses dedicated database account
- [ ] Principle of least privilege applied
- [ ] Application account cannot DROP/ALTER tables (production)
- [ ] Admin accounts separated from application accounts
- [ ] No default/shared credentials in use
- [ ] Service accounts have minimal permissions

**Issues Found:**

| Account | Severity | Issue | Permissions |
|---------|----------|-------|-------------|
| | | | |

**Permission Audit:**
```
Application User: [username]
Granted Permissions: [SELECT, INSERT, UPDATE, DELETE, etc.]
Excessive Permissions: [DROP, CREATE, ALTER, etc.]
```

**Recommendations:**
- _[Specific recommendation]_

### 2.2 Network Access Controls

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Database not exposed to public internet
- [ ] Firewall rules restrict database access
- [ ] Only application servers can connect to database
- [ ] Admin access restricted to specific IPs/VPN
- [ ] Default database ports changed (optional)
- [ ] Network segmentation in place

**Issues Found:**

| Severity | Issue | Exposure | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Network Configuration:**
```
Public Access: [Yes/No]
Allowed IPs: [IP ranges]
Firewall: [Enabled/Disabled]
Port: [Port number]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 3. Data Encryption

### 3.1 Encryption at Rest

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Transparent Data Encryption (TDE) enabled
- [ ] Sensitive columns encrypted (application-level)
- [ ] Encryption keys managed securely (KMS)
- [ ] Database files encrypted on disk
- [ ] Backup files encrypted
- [ ] Temporary files encrypted or securely deleted

**Issues Found:**

| Data Type | Severity | Issue | Impact |
|-----------|----------|-------|--------|
| | | | |

**Encryption Configuration:**
```
TDE: [Enabled/Disabled]
Algorithm: [AES-256, etc.]
Key Management: [AWS KMS, etc.]
Encrypted Tables: [List]
```

**Recommendations:**
- _[Specific recommendation]_

### 3.2 Encryption in Transit

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] TLS/SSL enforced for all database connections
- [ ] Strong cipher suites configured
- [ ] Certificate validation enforced
- [ ] No plaintext connections allowed
- [ ] Replication traffic encrypted
- [ ] Admin connections use TLS

**Issues Found:**

| Connection Type | Severity | Issue | Impact |
|-----------------|----------|-------|--------|
| | | | |

**TLS Configuration:**
```
TLS Version: [1.2, 1.3]
Certificate: [Valid/Self-signed]
Cipher Suites: [List]
Enforce TLS: [Yes/No]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 4. Authentication & Connection Security

### 4.1 Authentication Mechanisms

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Strong authentication required (no blank passwords)
- [ ] Password complexity enforced
- [ ] IAM authentication used (cloud environments)
- [ ] Certificate-based authentication available
- [ ] No hardcoded credentials in application code
- [ ] Connection strings stored securely

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Authentication Methods:**
```
Methods: [Password, IAM, Certificate]
Password Policy: [Complexity requirements]
MFA: [Available/Not Available]
```

**Recommendations:**
- _[Specific recommendation]_

### 4.2 Connection Pooling

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Connection pooling implemented correctly
- [ ] Pool size limits prevent resource exhaustion
- [ ] Idle connections timeout appropriately
- [ ] Connection validation before use
- [ ] No connection leaks detected
- [ ] Credentials in pool are protected

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Pool Configuration:**
```
Min Connections: [Number]
Max Connections: [Number]
Idle Timeout: [Seconds]
Validation: [Enabled/Disabled]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 5. Backup & Recovery

### 5.1 Backup Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Backups encrypted at rest
- [ ] Backup files access controlled
- [ ] Backups stored in separate location
- [ ] Backup integrity verified regularly
- [ ] Backup retention policy enforced
- [ ] Backups protected from ransomware

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Backup Configuration:**
```
Frequency: [Daily, Weekly, etc.]
Encryption: [Yes/No]
Location: [S3, tape, etc.]
Retention: [Days]
Last Successful: [Date]
```

**Recommendations:**
- _[Specific recommendation]_

### 5.2 Recovery Procedures

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Recovery procedures documented and tested
- [ ] Recovery time objective (RTO) defined
- [ ] Recovery point objective (RPO) defined
- [ ] Point-in-time recovery available
- [ ] Disaster recovery plan exists
- [ ] Recovery access is controlled and audited

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recovery Metrics:**
```
RTO: [Hours]
RPO: [Hours]
Last Test: [Date]
Test Result: [Success/Failure]
```

**Recommendations:**
- _[Specific recommendation]_

---

## 6. Stored Procedures & Functions

### 6.1 Secure Coding Practices

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Stored procedures use parameterized queries
- [ ] Input validation in stored procedures
- [ ] No dynamic SQL in stored procedures
- [ ] Error messages don't expose sensitive info
- [ ] Stored procedures follow least privilege
- [ ] Code review process for database code

**Issues Found:**

| Procedure/Function | Severity | Issue | Impact |
|--------------------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

### 6.2 Execution Permissions

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] EXECUTE permissions properly restricted
- [ ] Stored procedures run with appropriate privileges
- [ ] No EXECUTE WITH OWNER abuse
- [ ] Dangerous procedures disabled (xp_cmdshell, etc.)
- [ ] User-defined functions reviewed for security
- [ ] Triggers reviewed for security implications

**Issues Found:**

| Object | Severity | Issue | Impact |
|--------|----------|-------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 7. Auditing & Logging

### 7.1 Audit Configuration

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Database auditing enabled
- [ ] Failed login attempts logged
- [ ] Privilege changes logged
- [ ] Schema changes logged
- [ ] Data access logged (sensitive tables)
- [ ] Query logging enabled for suspicious patterns

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Audit Configuration:**
```
Audit Enabled: [Yes/No]
Events Logged: [List]
Log Retention: [Days]
Log Location: [Path/Service]
```

**Recommendations:**
- _[Specific recommendation]_

### 7.2 Log Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Logs stored securely and immutably
- [ ] Sensitive data not logged (passwords, PII)
- [ ] Log access restricted
- [ ] Logs monitored for security events
- [ ] Log tampering detected
- [ ] Logs backed up and retained

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

## 8. Database Hardening

### 8.1 Configuration Security

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Default accounts disabled or removed
- [ ] Sample/test databases removed
- [ ] Unnecessary features disabled
- [ ] Latest security patches applied
- [ ] Strong authentication enforced
- [ ] Resource limits configured (prevent DoS)

**Issues Found:**

| Severity | Issue | Configuration | Impact |
|----------|-------|---------------|--------|
| | | | |

**Hardening Checklist:**
```
Default Accounts: [Disabled/Removed]
Patch Level: [Current version]
Unused Features: [Disabled]
Resource Limits: [Configured]
```

**Recommendations:**
- _[Specific recommendation]_

### 8.2 Monitoring & Alerting

**Finding:** [ ] Pass [ ] Fail [ ] N/A

**Assessment:**
- [ ] Real-time monitoring of database activity
- [ ] Alerts for suspicious queries
- [ ] Performance monitoring includes security metrics
- [ ] Failed login alerts configured
- [ ] Privilege escalation alerts configured
- [ ] Database security dashboard exists

**Issues Found:**

| Severity | Issue | Location | Committed By | Approved By | Impact |
|----------|-------|----------|--------------|-------------|--------|
| | | | |

**Recommendations:**
- _[Specific recommendation]_

---

<!-- analysis: manual -->

## 9. Testing Methodology

### Tools Used
- [ ] SQLMap (SQL injection testing)
- [ ] NoSQLMap (NoSQL injection)
- [ ] Burp Suite Professional
- [ ] Database security scanners
- [ ] Manual query testing
- [ ] Custom injection scripts

### Test Scenarios Executed
1. **SQL Injection:** _[Results]_
2. **NoSQL Injection:** _[Results]_
3. **Access Control Testing:** _[Results]_
4. **Authentication Bypass:** _[Results]_
5. **Backup Security:** _[Results]_

---

## Summary of Findings

### Critical Issues (Immediate Action Required)
1. **[Issue Name]** - _[Brief description]_
2. **[Issue Name]** - _[Brief description]_

### High Priority Issues
1. **[Issue Name]** - _[Brief description]_
2. **[Issue Name]** - _[Brief description]_

### Medium Priority Issues
1. **[Issue Name]** - _[Brief description]_

### Low Priority Issues
1. **[Issue Name]** - _[Brief description]_

---

## Recommendations Summary

### Immediate Actions (0-7 days)
1. _[Action]_
2. _[Action]_

### Short-term Actions (1-4 weeks)
1. _[Action]_
2. _[Action]_

### Long-term Improvements (1-3 months)
1. _[Action]_
2. _[Action]_

---

## Conclusion

**Database Security Posture:** _[Overall assessment]_

**Key Takeaways:**
- _[Key point]_
- _[Key point]_
- _[Key point]_

**Next Steps:**
1. _[Next step]_
2. _[Next step]_

---

**Assessment completed by:** _[Your name]_  
**Date:** _[YYYY-MM-DD]_  
**Review date:** _[YYYY-MM-DD]_
