# Pass 5 — Security & Access Control

## Purpose

Capture all security, authorization, and data protection patterns from the legacy system. This information typically lives OUTSIDE the COBOL source code — in RACF/ACF2/TopSecret definitions, CICS resource definitions, DB2 grants, and operational procedures. The coding agent should document what it CAN find in the code and flag what needs to be gathered from mainframe security administrators.

## Why This Pass Exists

In every COBOL modernization where this was skipped, the result was a Java system that:
- Had no authorization model (anyone could do anything)
- Exposed sensitive data that was previously restricted
- Failed security audits that the COBOL system passed
- Required emergency post-deployment security patches

## Inputs Required

- COBOL source + copybooks (from repo)
- CICS CSD definitions (if available in repo)
- DB2 DDL / GRANT statements (if available)
- Any security configuration files in the repo
- All prior pass outputs for context

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Security Spec | `05_security/prose/{domain}_security_spec.md` | `05_security/specs/{domain}_security.yaml` |
| Data Classification | `05_security/prose/DATA_CLASSIFICATION.md` | `05_security/specs/data_classification.yaml` |

## Processing Instructions

```
1. SCAN the repo for any security-related files:
   - CICS CSD definitions, RACF commands, DB2 GRANT scripts,
     security exit programs, encryption routines
2. For each COBOL program, SCAN for security-relevant patterns
3. APPLY Prompt 5.1 for each domain cluster
4. APPLY Prompt 5.2 for data classification
5. WRITE outputs and UPDATE processing log
```

---

## Prompt 5.1 — Security & Authorization Specification

```
ROLE: You are a security architect analyzing legacy COBOL programs for 
security patterns, access control logic, and data protection mechanisms.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

PRIOR CONTEXT:
- API specs: {{OUTPUT_DIR}}/03_functional/specs/api-specs/ (for mapping 
  security to endpoints)
- Business rules: {{OUTPUT_DIR}}/02_business/specs/rules/ (for 
  authorization rules already captured)

TASK: Read the following programs from the repository and extract all 
security-related patterns.

Programs: {{PROGRAM_PATHS}}
Copybooks: {{COPYBOOK_PATHS}}
CICS CSD (if available): {{CSD_PATH}}
DB2 GRANT scripts (if available): {{GRANT_PATH}}

LOOK FOR in the COBOL source:
- EXEC CICS VERIFY / SIGNON / ASSIGN USERID
- CICS transaction security levels
- EIBTRMID, EIBUSRID checks (terminal/user verification)
- Authorization IF statements (checking user ID, role, permission)
- Data masking or redaction logic (e.g., showing only last 4 digits of SSN)
- Encryption/decryption CALL statements
- Audit logging (writing to security log files or tables)
- Password handling
- Session management patterns
- Field-level security (different users see different fields)

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/05_security/prose/{{DOMAIN}}_security_spec.md

## Security Specification: [Domain Name]

### Authentication Patterns Found
[How users are identified in the COBOL system — CICS signon, 
user ID passed in COMMAREA, external security manager, etc.]

| Pattern | Source Program | Details | Java Equivalent |
|---|---|---|---|
| [pattern] | [program] | [how it works] | [OAuth/JWT/RBAC approach] |

### Authorization Rules
[Who can do what — extracted from code logic and CICS definitions]

| Action | Required Permission | Source | How Enforced | API Endpoint |
|---|---|---|---|---|
| [e.g., "View account"] | [e.g., "Any authenticated user"] | [program, paragraph] | [CICS security / code check] | [GET /accounts/{id}] |
| [e.g., "Close account"] | [e.g., "Supervisor role only"] | [program, paragraph] | [IF WS-USER-LEVEL > 3] | [POST /accounts/{id}/close] |

### Data Protection Patterns
| Data Element | Protection in COBOL | Recommended Java Approach |
|---|---|---|
| [e.g., "SSN"] | [e.g., "Displayed as XXX-XX-1234 on screen"] | [Field-level encryption + masking] |
| [e.g., "Account balance"] | [e.g., "Visible only to account owner and supervisor"] | [Row-level security + API scope] |

### Audit Trail Requirements
[What the COBOL system logs for security/compliance]
| Event | What's Logged | Where Logged | Retention |
|---|---|---|---|
| [event] | [data captured] | [file/table] | [duration if known] |

### INFORMATION NOT IN CODE — MUST GATHER FROM MAINFRAME TEAM
[Flag everything that can't be determined from source code alone]

⚠️ **The following security information is NOT in the COBOL source and 
must be obtained from the mainframe security team:**

- [ ] RACF/ACF2/TopSecret resource profiles for this application
- [ ] CICS transaction security definitions (XTRANID, XCMD settings)
- [ ] DB2 GRANT/REVOKE for application tables
- [ ] Network security (SNA LU definitions, IP restrictions)
- [ ] Encryption key management procedures
- [ ] User provisioning/deprovisioning process
- [ ] Security incident response procedures
- [ ] Compliance audit requirements and schedule

---

FILE 2: {{OUTPUT_DIR}}/05_security/specs/{{DOMAIN}}_security.yaml

```yaml
security:
  domain: "{{DOMAIN}}"
  x-cobol-source:
    programs: ["list"]

  authentication:
    patterns:
      - type: "[CICS_SIGNON | COMMAREA_USERID | EXTERNAL_SECURITY | NONE_FOUND]"
        source_program: "[program]"
        source_location: "[paragraph]"
        details: "[how it works]"
        proposed_java: "[OAuth2 | JWT | Spring Security | etc.]"

  authorization:
    rules:
      - action: "[what operation]"
        resource: "[what resource]"
        required_permission: "[permission/role needed]"
        source_program: "[program]"
        source_location: "[paragraph]"
        enforcement: "[how enforced in COBOL]"
        api_endpoint: "[mapped API endpoint]"
        proposed_java: "[Spring Security role | @PreAuthorize | etc.]"

  data_protection:
    sensitive_fields:
      - field: "[field name]"
        classification: "[PII | PCI | PHI | FINANCIAL | CONFIDENTIAL]"
        current_protection: "[masking | encryption | access_control | none]"
        source_program: "[program]"
        proposed_protection: "[encryption_at_rest | field_masking | tokenization]"
        domain_model_ref: "[schema.yaml#/properties/field]"

  audit:
    events:
      - event: "[what triggers audit]"
        data_captured: ["list of fields logged"]
        destination: "[log file/table]"
        source_program: "[program]"
        retention: "[if known]"

  gaps_requiring_mainframe_team:
    - item: "[what information is missing]"
      why_needed: "[impact on Java security design]"
      suggested_source: "[who to ask — security admin, DBA, etc.]"
      priority: "[HIGH | MEDIUM | LOW]"
```
```

---

## Prompt 5.2 — Data Classification

```
ROLE: You are a data protection specialist classifying all data in the 
system by sensitivity level.

TASK: Read ALL domain model schemas from Pass 1 and classify every field.

Read: {{OUTPUT_DIR}}/01_discovery/specs/domain-model/*.schema.yaml

Produce: {{OUTPUT_DIR}}/05_security/specs/data_classification.yaml

```yaml
data_classification:
  system: "{{SYSTEM_NAME}}"
  
  classifications:
    - schema: "[schema file]"
      entity: "[entity name]"
      fields:
        - field: "[field name]"
          classification: "[PUBLIC | INTERNAL | CONFIDENTIAL | PII | PCI | PHI | RESTRICTED]"
          sub_category: "[SSN | DOB | ACCOUNT_NUMBER | BALANCE | ADDRESS | PHONE | EMAIL | MEDICAL | etc.]"
          regulations: ["GDPR", "PCI-DSS", "HIPAA", "SOX", "CCPA"]
          required_controls:
            at_rest: "[ENCRYPT | MASK | NONE]"
            in_transit: "[TLS | ENCRYPT | NONE]"
            in_display: "[MASK | REDACT | FULL | NONE]"
            in_logs: "[EXCLUDE | MASK | HASH | NONE]"
          retention: "[policy if known]"
          notes: "[special handling requirements]"
  
  summary:
    total_fields: [N]
    public: [N]
    internal: [N]
    confidential: [N]
    pii: [N]
    pci: [N]
    phi: [N]
    restricted: [N]
```
```

---

## Quality Gate Checklist

- [ ] Every domain has a security specification
- [ ] Every API endpoint has an authorization rule (even if "any authenticated user")
- [ ] All PII/PCI/PHI fields are identified in data classification
- [ ] Gaps requiring mainframe team input are explicitly listed
- [ ] Security patterns found in code match the authorization rules in business rules from Pass 2
