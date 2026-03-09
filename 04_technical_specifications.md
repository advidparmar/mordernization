# Pass 4 — Technical Specifications

## Purpose

Produce specifications detailed enough that a Java developer can implement equivalent functionality without seeing the COBOL source. Also produce executable Gherkin test features and data migration mappings.

## Inputs Required

- COBOL source + copybooks (from repo)
- ALL prior pass outputs (inventory, domain models, business processes, rules, functional specs, API contracts)

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Technical Spec | `04_technical/prose/{program}_tech_spec.md` | — |
| Behavior Tests | — | `04_technical/specs/features/{domain}/{feature}.feature` |
| Migration Map | — | `04_technical/specs/migration/{entity}-migration.mapping.yaml` |
| Data Model Design | `04_technical/prose/DATA_MODEL_DESIGN.md` | (domain-model schemas from Pass 1 serve this role) |
| Service Decomposition | `04_technical/prose/SERVICE_DECOMPOSITION.md` | — |
| Error Strategy | `04_technical/prose/ERROR_HANDLING_STRATEGY.md` | — |
| Batch Design | `04_technical/prose/BATCH_PROCESSING_DESIGN.md` | — |

## Processing Instructions

```
1. For each program (with full context from all prior passes):
   a. READ program source + copybooks from repo
   b. LOAD all prior outputs as context
   c. APPLY Prompt 4.1 (Technical Spec + Gherkin + Migration)
   d. WRITE outputs
   e. UPDATE processing log

2. After all programs processed:
   a. APPLY Prompt 4.2 (Data Model Design — aggregation)
   b. APPLY Prompt 4.3 (Service Decomposition — aggregation)
   c. APPLY Prompt 4.4 (Error Handling Strategy — aggregation)
   d. APPLY Prompt 4.5 (Batch Processing Design — aggregation)
```

---

## Prompt 4.1 — Technical Spec, Gherkin Tests & Migration Mapping

For each program, produce detailed technical spec with executable tests.

```
ROLE: You are BOTH a senior technical architect writing Java implementation 
specs AND a BDD test engineer writing executable Gherkin features. Be 
precise about data types, precision, edge cases, and processing sequences.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

PRIOR CONTEXT (read these files):
- Inventory: {{OUTPUT_DIR}}/01_discovery/specs/inventory/{{PROGRAM_NAME}}.inventory.yaml
- Domain Model: {{OUTPUT_DIR}}/01_discovery/specs/domain-model/*.schema.yaml (relevant ones)
- Business Rules: {{OUTPUT_DIR}}/02_business/specs/rules/{{DOMAIN}}/{{CLUSTER}}.rules.yaml
- Workflow: {{OUTPUT_DIR}}/02_business/specs/workflows/{{PROCESS}}.workflow.yaml
- Functional Spec: {{OUTPUT_DIR}}/03_functional/prose/{{PROGRAM_NAME}}_functional_spec.md
- API Contract: {{OUTPUT_DIR}}/03_functional/specs/api-specs/{{SERVICE}}.openapi.yaml

TASK: Read the program from the repository and produce a technical 
specification, executable Gherkin tests, and migration mapping.

Program: {{REPO_ROOT}}/{{PROGRAM_PATH}}
Copybooks: {{COPYBOOK_PATHS}}

===== PRODUCE THREE FILES =====

FILE 1: {{OUTPUT_DIR}}/04_technical/prose/{{PROGRAM_NAME}}_tech_spec.md

## Technical Specification: {{PROGRAM_NAME}}
**Tech Spec ID:** TS-{{PROGRAM_NAME}}
**Related:** FS-{{SPEC_ID}}, {{SERVICE}}.openapi.yaml

### 1. Architecture Context
[Where this fits in target architecture. Service, layer, runtime dependencies.]

### 2. Data Structure Mapping — COBOL to Java

**[Structure] → Java Class: [Name]**
| COBOL Field | PIC Clause | Java Type | Java Field | Validation | Notes |
|---|---|---|---|---|---|
| [field] | [PIC] | [type] | [camelCase] | [annotations] | [critical notes] |

⚠️ **Critical Precision Rules:**
[List EVERY field where COBOL arithmetic differs from Java defaults.]
| Field | COBOL Behavior | Java Requirement | What Breaks If Wrong |
|---|---|---|---|
| [field] | [behavior] | [BigDecimal approach] | [consequence] |

### 3. Processing Logic — Pseudocode

Map COBOL logic to Java methods. Preserve exact processing sequence.

```pseudocode
/**
 * [Description]
 * Source: {{PROGRAM_NAME}}, paragraph [name]
 * Rules: BR-xxx, BR-yyy
 * API: [endpoint reference]
 */
METHOD [methodName]([Type] [param]): [ReturnType]

  // BR-V-001: [Rule description]
  IF param.accountNumber IS NULL OR BLANK:
    THROW ValidationException("ERR-ACCT-001", "Invalid account number")

  // BR-C-001: Daily interest = balance × (rate / 360)
  // CRITICAL: BigDecimal with HALF_UP, scale=2
  // CRITICAL: 360-day year convention — NOT 365
  VAR dailyRate = annualRate.divide(BigDecimal.valueOf(360), 10, HALF_UP)
  VAR dailyInterest = balance.multiply(dailyRate).setScale(2, HALF_UP)

  // 4000-WRITE: Update and persist
  account.setAccruedInterest(
    account.getAccruedInterest().add(dailyInterest)
  )
  repository.save(account)

  RETURN InterestResult(dailyInterest, account.getAccruedInterest())
```

### 4. Error Handling Mapping
| COBOL Pattern | Specific Example | Java Implementation |
|---|---|---|
| FILE STATUS | "35" = not found | FileNotFoundException |
| SQLCODE | +100 = not found | Optional.empty() |
| SQLCODE | -803 = duplicate | DuplicateKeyException → HTTP 409 |
| SQLCODE | < 0 = error | DataAccessException → HTTP 500 |
| CICS RESP | NOTFND | NotFoundException → HTTP 404 |
| ABEND | U1001 | SystemException("U1001") |
| RETURN-CODE | 0/4/8/12/16 | ResultCode enum |
| ON SIZE ERROR | overflow | ArithmeticException / check bounds |

### 5. Concurrency & Locking
[File locking, DB2 isolation, CICS enqueue — what Java must replicate]

### 6. Performance & Volume
- Current: [N records/transactions]
- Pattern: [Sequential scan / Random / Cursor-based]
- Java notes: [Batch insert, streaming, caching, connection pooling]

### 7. Batch Design (if applicable)
[Spring Batch job/step structure, reader/processor/writer, chunk size, 
restart strategy, error threshold, idempotency approach]

### 8. Full Traceability
| Business Rule | Func Spec | API Endpoint | COBOL Source | Java Component | Test |
|---|---|---|---|---|---|
| BR-xxx | FS-xxx §N | [endpoint] | [program, para] | [Class.method()] | TC-xxx |

---

FILE 2: {{OUTPUT_DIR}}/04_technical/specs/features/{{DOMAIN}}/{{FEATURE_NAME}}.feature

```gherkin
# features/{{DOMAIN}}/{{FEATURE_NAME}}.feature

@domain:{{DOMAIN}} @source:{{PROGRAM_NAME}} @priority:P1
@human-doc:TS-{{PROGRAM_NAME}} @api-doc:{{SERVICE}}.openapi.yaml
Feature: [Feature Name]
  As a [actor]
  I need to [capability]
  So that [business value]

  Background:
    Given [common setup for all scenarios]

  # Source: {{PROGRAM_NAME}}, paragraph [name]
  # Business Rule: BR-xxx
  # Human Spec: TS-{{PROGRAM_NAME}}, Section 3, Step [N]
  @rule:BR-xxx @precision-critical
  Scenario Outline: [Description]
    Given [precondition with "<variable>"]
    And [additional context]
    When [action]
    Then [expected outcome with "<expected>"]

    Examples:
      | variable     | expected      |
      | [normal]     | [result]      |
      | [boundary]   | [result]      |
      | [max_value]  | [result]      |
      | [zero]       | [result]      |

  @rule:BR-xxx @negative @error
  Scenario: [Validation failure scenario]
    Given [precondition triggering validation error]
    When [action]
    Then the system should respond with status 400
    And the error code should be "ERR-xxx"
    And the error message should indicate [description]

  @rule:BR-xxx @boundary @precision-critical
  Scenario: [COBOL-specific precision test]
    Given [precondition testing numeric precision]
    When [calculation is performed]
    Then the result should be exactly [value to 2 decimal places]
    # Note: This tests COBOL COMPUTE ROUNDED equivalence

  @parallel-run
  Scenario: [Parallel run equivalence test]
    Given a batch input matching COBOL production data:
      | field1 | field2 | field3 |
      | [val]  | [val]  | [val]  |
    When the [process] runs
    Then the output should match COBOL exactly:
      | output1  | output2  |
      | [val]    | [val]    |
    And the return code should be [0|4|8]

  @batch @empty-input
  Scenario: Empty input handling
    Given an empty input [file/dataset]
    When the [process] runs
    Then it should complete with return code 0
    And the summary should show 0 records processed

  @batch @error-threshold
  Scenario: Mixed valid and invalid records
    Given an input with [N] valid and [M] invalid records
    When the [process] runs
    Then [N] records should be processed successfully
    And [M] records should be in the error file
    And the return code should be [4]
```

---

FILE 3: {{OUTPUT_DIR}}/04_technical/specs/migration/{{ENTITY}}-migration.mapping.yaml

Only produce this if the program reads/writes persistent data stores.

```yaml
migration:
  id: "MIG-{{ENTITY}}"
  x-human-doc: "04_technical/prose/{{PROGRAM_NAME}}_tech_spec.md, Section 2"
  source:
    type: "[DB2 | VSAM | SEQUENTIAL_FILE]"
    name: "[table/file name]"
    x-cobol-source:
      copybook: "[copybook name]"
      source_path: "[path in repo]"
    estimated_rows: [N if known]
  target:
    type: "[PostgreSQL | MySQL | etc.]"
    table: "[schema.table_name]"
    schema_ref: "[domain-model schema path]"

  field_mappings:
    - source:
        column: "[COBOL/DB2 field]"
        type: "[COBOL type]"
        pic: "[PIC clause]"
      target:
        column: "[new column]"
        type: "[SQL type]"
        java_type: "[Java type]"
      transform: "[DIRECT | TRIM | MAP | DATE_PARSE | DECIMAL_CONVERT | CUSTOM]"
      mapping:  # for MAP transform
        "[source_val]": "[target_val]"
        " ": null
      format: "[YYYYMMDD for DATE_PARSE]"
      null_representations: ["00000000", "99999999", "        "]
      precision:
        source_scale: [N]
        target_scale: [N]
        rounding: "[mode]"
      on_failure: "[REJECT_RECORD | SET_NULL_AND_LOG | DEFAULT_VALUE | HALT]"
      notes: "[human-readable concern]"

  validation_checks:
    pre_migration:
      - check: "ROW_COUNT"
        description: "Count source rows before migration"
      - check: "SUM"
        field: "[balance field]"
        description: "Sum of financial fields — must match post-migration"
        tolerance: "0.00"
      - check: "NULL_ANALYSIS"
        description: "Count nulls/spaces per column"
    post_migration:
      - check: "ROW_COUNT_MATCH"
      - check: "SUM_MATCH"
        tolerance: "0.00"
      - check: "SAMPLE_COMPARE"
        sample_size: "1%"
      - check: "STATUS_DISTRIBUTION"
        description: "Verify enum distributions match"
```
```

---

## Prompt 4.2 — Data Model Design (Aggregation)

After all programs processed, aggregate into a unified data model.

```
TASK: Read all domain model schemas from:
{{OUTPUT_DIR}}/01_discovery/specs/domain-model/

And all migration mappings from:
{{OUTPUT_DIR}}/04_technical/specs/migration/

Produce: {{OUTPUT_DIR}}/04_technical/prose/DATA_MODEL_DESIGN.md

[Unified data model showing all entities, relationships, type translations, 
eliminated redundancies, new structures needed, and migration complexity.]
```

---

## Prompt 4.3 — Service Decomposition (Aggregation)

```
TASK: Read all inventory, dependency graph, business processes, and API specs.

Produce: {{OUTPUT_DIR}}/04_technical/prose/SERVICE_DECOMPOSITION.md

[Bounded contexts, service definitions with APIs and data ownership, 
interaction map, migration sequence, strangler fig opportunities.]
```

---

## Prompt 4.4 — Error Handling Strategy (Aggregation)

```
TASK: Read all error handling sections from technical specs.

Produce: {{OUTPUT_DIR}}/04_technical/prose/ERROR_HANDLING_STRATEGY.md

[Comprehensive COBOL-to-Java error pattern mapping, exception hierarchy, 
logging strategy, monitoring/alerting, batch error handling patterns.]
```

---

## Prompt 4.5 — Batch Processing Design (Aggregation)

```
TASK: Read all JCL analyses and batch program specs.

Produce: {{OUTPUT_DIR}}/04_technical/prose/BATCH_PROCESSING_DESIGN.md

[Technology recommendation, job-to-Spring-Batch mapping, data flow redesign, 
restart/recovery, scalability, processing window analysis.]
```

---

## Quality Gate Checklist

Before proceeding to Pass 5, verify:

- [ ] Every program has a technical specification in `04_technical/prose/`
- [ ] Every business rule has at least one Gherkin scenario (verify by grepping `@rule:BR-xxx` tags)
- [ ] All Gherkin feature files parse without syntax errors
- [ ] Every data store has a migration mapping YAML
- [ ] All migration YAMLs have pre and post validation checks
- [ ] Precision warnings are explicit for ALL financial/numeric fields
- [ ] Aggregation documents (data model, service decomposition, error handling, batch) are complete
- [ ] Processing log is updated
- [ ] **ARCHITECTURE REVIEW:** Java architect reviews data model and service decomposition for feasibility
