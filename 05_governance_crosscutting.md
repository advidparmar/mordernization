# Pass 5 — Governance & Cross-Cutting

## Purpose

Ensure nothing is lost in translation. Produce the traceability matrix, risk register, and executive summary that prove completeness and highlight remaining gaps.

## Inputs Required

- ALL outputs from Passes 1-4 (both human-readable and machine-readable)
- Processing log showing completion status

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Executive Summary | `05_governance/prose/EXECUTIVE_SUMMARY.md` | — |
| Traceability Report | `05_governance/prose/TRACEABILITY_REPORT.md` | `05_governance/specs/traceability-matrix.yaml` |
| Risk Register | `05_governance/prose/RISK_REGISTER.md` | `05_governance/specs/risk-register.yaml` |
| Completion Report | `COMPLETION_REPORT.md` (root of output dir) | — |

## Processing Instructions

```
1. COLLECT all open items, assumptions, and flags from all prior passes
2. APPLY Prompt 5.1 (Traceability Matrix)
3. APPLY Prompt 5.2 (Risk Register)
4. APPLY Prompt 5.3 (Executive Summary & Completion Report)
5. FINALIZE the processing log
```

---

## Prompt 5.1 — Traceability Matrix

```
ROLE: You are a quality assurance architect ensuring completeness and 
traceability across the entire modernization analysis.

CONTEXT:
- System: {{SYSTEM_NAME}}

TASK: Read ALL artifacts produced in Passes 1-4 and create a complete 
traceability matrix linking every business rule from discovery through 
to implementation specification and test coverage.

Read from:
- All rules: {{OUTPUT_DIR}}/02_business/specs/rules/**/*.rules.yaml
- All functional specs: {{OUTPUT_DIR}}/03_functional/prose/*_functional_spec.md
- All API specs: {{OUTPUT_DIR}}/03_functional/specs/api-specs/*.openapi.yaml
- All tech specs: {{OUTPUT_DIR}}/04_technical/prose/*_tech_spec.md
- All Gherkin features: {{OUTPUT_DIR}}/04_technical/specs/features/**/*.feature
- All migration mappings: {{OUTPUT_DIR}}/04_technical/specs/migration/*.yaml
- Master rules catalog: {{OUTPUT_DIR}}/02_business/specs/rules/master_rules_catalog.yaml
- Processing log: {{OUTPUT_DIR}}/logs/processing_log.yaml

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/05_governance/prose/TRACEABILITY_REPORT.md

## Traceability Report — {{SYSTEM_NAME}}

### Coverage Summary

| Metric | Count | Percentage |
|--------|-------|-----------|
| Total Business Rules | [N] | — |
| Rules with Full Traceability (rule→spec→code→test) | [N] | [%] |
| Rules with Functional Spec | [N] | [%] |
| Rules with Technical Spec | [N] | [%] |
| Rules with Test Coverage (Gherkin) | [N] | [%] |
| Rules with Migration Mapping | [N/A or N] | [%] |

| Metric | Count |
|--------|-------|
| Total Programs Analyzed | [N] |
| API Endpoints Defined | [N] |
| Gherkin Scenarios Written | [N] |
| Domain Model Entities | [N] |
| Migration Mappings | [N] |
| Interface Specs | [N] |

### Gaps Identified

#### Rules Missing Functional Spec Coverage
| Rule ID | Description | Source Program | Action Needed |
|---|---|---|---|
| BR-xxx | [description] | [program] | [what to do] |

#### Rules Missing Technical Spec Coverage
| Rule ID | Description | Action Needed |
|---|---|---|
| BR-xxx | [description] | [what to do] |

#### Rules Missing Test Coverage
| Rule ID | Description | Risk Level | Action Needed |
|---|---|---|---|
| BR-xxx | [description] | [H/M/L] | [what to do] |

#### Orphaned Specs (No Business Rule)
[Technical specs that don't trace back to any business rule — these 
might be pure technical concerns, or they might be missed rules]
| Spec Section | Program | Description | Assessment |
|---|---|---|---|
| [spec reference] | [program] | [what it covers] | [Technical concern / Missed rule] |

### Programs Not Fully Analyzed
[Programs that were partially analyzed or skipped, with reasons]
| Program | Pass Completed Through | Reason | Risk |
|---|---|---|---|
| [program] | [pass N] | [reason] | [impact] |

---

FILE 2: {{OUTPUT_DIR}}/05_governance/specs/traceability-matrix.yaml

```yaml
traceability:
  system: "{{SYSTEM_NAME}}"
  generated_date: "[date]"
  
  coverage:
    total_business_rules: [N]
    rules_full_trace: [N]
    rules_with_functional_spec: [N]
    rules_with_technical_spec: [N]
    rules_with_test_coverage: [N]
    coverage_percentage: [N]
    total_programs: [N]
    total_api_endpoints: [N]
    total_gherkin_scenarios: [N]
    total_entities: [N]
    total_migration_mappings: [N]

  entries:
    - trace_id: "TRC-001"
      business_rule: "BR-V-001"
      rule_description: "[description]"
      rule_source_file: "[rules YAML path]"
      business_process_doc: "[human doc path]"
      workflow_spec: "[workflow YAML path]"
      functional_spec:
        id: "FS-[id]"
        file: "[prose path]"
        section: "[section number]"
      api_spec:
        file: "[OpenAPI YAML path]"
        endpoint: "[path and method]"
      technical_spec:
        id: "TS-[program]"
        file: "[prose path]"
        section: "[section number]"
      cobol_source:
        program: "[program name]"
        source_path: "[path in repo]"
        paragraph: "[paragraph/section]"
      domain_model:
        schema_file: "[schema YAML path]"
        fields: ["affected fields"]
      target:
        service: "[service name]"
        java_component: "[Class.method()]"
      test_coverage:
        gherkin_feature: "[feature file path]"
        scenarios: ["TC-xxx", "TC-yyy"]
        tags: ["@rule:BR-V-001"]
      migration:
        mapping_file: "[migration YAML path]"
      status: "[COMPLETE | PARTIAL | MISSING_FUNC_SPEC | MISSING_TECH_SPEC | MISSING_TEST | MISSING_MULTIPLE]"
      gaps: ["description of each gap"]

  gap_summary:
    missing_functional_spec:
      - rule_id: "BR-xxx"
        description: "[description]"
    missing_technical_spec:
      - rule_id: "BR-xxx"
        description: "[description]"
    missing_test_coverage:
      - rule_id: "BR-xxx"
        description: "[description]"
        risk: "[HIGH | MEDIUM | LOW]"
    orphaned_specs:
      - spec_reference: "[reference]"
        program: "[program]"
        assessment: "[Technical concern | Missed rule]"
```
```

---

## Prompt 5.2 — Risk Register

```
ROLE: You are a modernization risk analyst consolidating all identified 
risks, open questions, and assumptions from the entire analysis.

CONTEXT:
- System: {{SYSTEM_NAME}}

TASK: Read ALL "Open Items," "Flags," "Assumptions," and "SME Review" 
sections from every artifact in Passes 1-4. Consolidate into a single 
prioritized risk register.

Scan these directories for open items:
- {{OUTPUT_DIR}}/01_discovery/prose/*.md (look for "Flags" sections)
- {{OUTPUT_DIR}}/01_discovery/specs/inventory/*.yaml (look for "flags:" entries)
- {{OUTPUT_DIR}}/02_business/prose/*.md (look for "Open Questions" sections)
- {{OUTPUT_DIR}}/02_business/specs/rules/**/*.yaml (look for "x-sme-review:" entries)
- {{OUTPUT_DIR}}/03_functional/prose/*.md (look for "Open Items" sections)
- {{OUTPUT_DIR}}/04_technical/prose/*.md (look for traceability gaps)
- {{OUTPUT_DIR}}/05_governance/specs/traceability-matrix.yaml (look for gaps)

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/05_governance/prose/RISK_REGISTER.md

## Risk Register — {{SYSTEM_NAME}}

### Summary Dashboard
| Severity | Count |
|----------|-------|
| CRITICAL | [N] |
| HIGH | [N] |
| MEDIUM | [N] |
| LOW | [N] |
| **TOTAL** | **[N]** |

### Top 10 Risks Requiring Immediate Attention

**RISK-001: [Title]** — Severity: CRITICAL
- **Description:** [Detailed narrative — what, where, why it matters]
- **Discovered In:** [Which artifact/pass flagged this]
- **Affected:** Programs: [list], Services: [list], Rules: [list]
- **Impact If Unresolved:** [Specific consequence]
- **Recommended Action:** [Concrete mitigation step]
- **Owner:** [Role responsible — SME / Architect / Developer / PM]

[Continue for top 10...]

### Complete Risk Register

| ID | Category | Title | Severity | Likelihood | Source | Components | Action | Owner | Status |
|---|---|---|---|---|---|---|---|---|---|
| RISK-001 | [category] | [title] | [sev] | [H/M/L] | [artifact] | [affected] | [action] | [role] | OPEN |

Categories: AMBIGUOUS_LOGIC, DEAD_CODE, MISSING_KNOWLEDGE, DATA_QUALITY, 
PRECISION, INTEGRATION, PERFORMANCE, COMPLIANCE, MISSING_COPYBOOK, OTHER

### Assumptions Log

Every assumption that could invalidate specifications if wrong:

| ID | Assumption | Impact If Wrong | Validation Method | Status |
|---|---|---|---|---|
| ASMP-001 | [what was assumed] | [what breaks] | [how to verify] | UNVALIDATED |

### SME Review Queue

All items specifically flagged for SME review, sorted by priority:

| Rule/Item | Question for SME | Why Uncertain | Priority | Status |
|---|---|---|---|---|
| BR-xxx | [specific question] | [reason] | [H/M/L] | PENDING |

---

FILE 2: {{OUTPUT_DIR}}/05_governance/specs/risk-register.yaml

```yaml
risk_register:
  system: "{{SYSTEM_NAME}}"
  generated_date: "[date]"

  summary:
    total: [N]
    critical: [N]
    high: [N]
    medium: [N]
    low: [N]

  risks:
    - id: "RISK-001"
      title: "[title]"
      category: "[AMBIGUOUS_LOGIC | DEAD_CODE | MISSING_KNOWLEDGE | DATA_QUALITY | PRECISION | INTEGRATION | PERFORMANCE | COMPLIANCE | MISSING_COPYBOOK | OTHER]"
      severity: "[CRITICAL | HIGH | MEDIUM | LOW]"
      likelihood: "[HIGH | MEDIUM | LOW]"
      description: "[detailed description]"
      source:
        artifact: "[which output flagged this]"
        pass: [1-5]
        file: "[file path]"
      affected:
        programs: ["list"]
        services: ["list"]
        business_rules: ["BR-xxx"]
        api_endpoints: ["list"]
      impact: "[consequence if unresolved]"
      recommended_action: "[specific mitigation]"
      owner_role: "[SME | Architect | Developer | PM | Business Analyst]"
      status: "[OPEN | IN_PROGRESS | RESOLVED | ACCEPTED]"

  assumptions:
    - id: "ASMP-001"
      description: "[assumption]"
      impact_if_wrong: "[consequence]"
      validation_method: "[how to verify]"
      source_artifact: "[where this was assumed]"
      status: "[UNVALIDATED | VALIDATED | INVALIDATED]"

  sme_review_queue:
    - item_id: "BR-xxx"
      question: "[question]"
      reason: "[why uncertain]"
      priority: "[HIGH | MEDIUM | LOW]"
      status: "[PENDING | REVIEWED | RESOLVED]"
      resolution: "[answer, if resolved]"
```
```

---

## Prompt 5.3 — Executive Summary & Completion Report

```
ROLE: You are a modernization program director producing the final 
summary of the COBOL analysis effort.

CONTEXT:
- System: {{SYSTEM_NAME}}

TASK: Read the traceability matrix and risk register just produced, 
plus the processing log, and generate the executive summary and 
completion report.

Read:
- {{OUTPUT_DIR}}/05_governance/specs/traceability-matrix.yaml
- {{OUTPUT_DIR}}/05_governance/specs/risk-register.yaml
- {{OUTPUT_DIR}}/logs/processing_log.yaml
- {{OUTPUT_DIR}}/04_technical/prose/SERVICE_DECOMPOSITION.md

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/05_governance/prose/EXECUTIVE_SUMMARY.md

## Executive Summary — {{SYSTEM_NAME}} Modernization Analysis

### Scope & Completion
[2-3 paragraphs: What was analyzed, how many programs, what the system 
does, and the overall completion status. Written for a CTO or VP.]

### Key Findings
[3-5 most important discoveries about the system — not technical 
minutiae, but things leadership needs to know. E.g., "The interest 
calculation uses a 360-day convention that may need regulatory review" 
or "23% of the codebase appears to be dead code."]

### Modernization Readiness Assessment
| Dimension | Rating | Notes |
|---|---|---|
| Specification Completeness | [1-5 stars] | [brief] |
| Business Rule Coverage | [%] | [brief] |
| Test Scenario Coverage | [%] | [brief] |
| Data Migration Readiness | [1-5 stars] | [brief] |
| Risk Profile | [LOW/MEDIUM/HIGH] | [brief] |

### Recommended Architecture
[Brief summary of the proposed service decomposition and why]

### Critical Risks
[Top 3 risks that leadership must be aware of, in plain language]

### Recommended Next Steps
1. [Most important next action]
2. [Second action]
3. [Third action]

### Effort Estimate for Development Phase
[Based on the analysis, rough estimate of development effort by tier]

---

FILE 2: {{OUTPUT_DIR}}/COMPLETION_REPORT.md

## Modernization Analysis — Completion Report

### Analysis Statistics
| Metric | Value |
|--------|-------|
| COBOL Programs in Repository | [N] |
| Programs Fully Analyzed (all 5 passes) | [N] |
| Programs Partially Analyzed | [N] |
| Programs Not Analyzed | [N] |
| Copybooks Analyzed | [N] |
| Copybooks Unresolved | [N] |
| JCL Jobs Analyzed | [N] |
| Business Rules Extracted | [N] |
| API Endpoints Defined | [N] |
| Gherkin Test Scenarios | [N] |
| Domain Model Entities | [N] |
| Migration Mappings | [N] |
| Interface Specs | [N] |
| Risks Identified | [N] (Critical: [N], High: [N]) |
| Assumptions Requiring Validation | [N] |
| Traceability Coverage | [%] |

### Artifacts Produced
[List all output directories with file counts]

```
{{OUTPUT_DIR}}/
├── 00_repo_scan/         [N files]
├── 01_discovery/
│   ├── prose/            [N files]
│   └── specs/            [N files]
├── 02_business/
│   ├── prose/            [N files]
│   └── specs/            [N files]
├── 03_functional/
│   ├── prose/            [N files]
│   └── specs/            [N files]
├── 04_technical/
│   ├── prose/            [N files]
│   └── specs/            [N files]
├── 05_governance/
│   ├── prose/            [N files]
│   └── specs/            [N files]
└── logs/
    └── processing_log.yaml
```

### Items Requiring Human Action Before Development
[Prioritized list of things that MUST be resolved by humans before 
Java development begins]

1. **SME Reviews:** [N] business rules need validation. [Top 3 listed.]
2. **Architectural Decisions:** [Any unresolved architecture choices]
3. **Risk Mitigations:** [Critical risks needing immediate attention]
4. **Missing Information:** [Unresolved copybooks, undocumented interfaces]
5. **Regulatory Confirmation:** [Any compliance-sensitive rules flagged]

### How to Use These Artifacts

**For Business Stakeholders:**
Read the prose documents in each pass's `prose/` directory, starting 
with the Executive Summary and Business Process documents.

**For Java Developers:**
Start with the Technical Specs (`04_technical/prose/`) and API Contracts 
(`03_functional/specs/api-specs/`). Use domain model schemas for entity 
generation and Gherkin features for TDD.

**For QA/Test Team:**
Use Gherkin features in `04_technical/specs/features/` as the acceptance 
test suite. Business rules with embedded test cases in 
`02_business/specs/rules/` provide unit test data.

**For DevOps/Migration Team:**
Use migration mappings in `04_technical/specs/migration/` to build 
ETL pipelines. Batch design in `04_technical/prose/BATCH_PROCESSING_DESIGN.md` 
guides job scheduling.

**For CI/CD Pipeline:**
- OpenAPI specs → openapi-generator for server stubs
- Gherkin features → Cucumber for acceptance tests
- Domain model schemas → Bean Validation + DB migration
- Rules YAML → Rule engine configuration or validation code generation
```
```

---

## Quality Gate Checklist (Final)

- [ ] Traceability matrix shows >95% coverage
- [ ] Every CRITICAL and HIGH risk has an owner and recommended action
- [ ] Executive summary is coherent and factually accurate
- [ ] Completion report statistics match actual file counts
- [ ] Processing log shows all programs completed through all passes
- [ ] All YAML files across all passes are syntactically valid
- [ ] All cross-references resolve (BR-xxx, FS-xxx, TS-xxx, etc.)
- [ ] **PROGRAM DIRECTOR REVIEW:** Leadership reviews executive summary and risk register
