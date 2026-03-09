# Pass 3 — Functional Specifications

## Purpose

Create platform-independent functional specifications detailed enough for a Java development team to implement from, without ever seeing the COBOL source. Also produce machine-readable API contracts (OpenAPI), interface specs, and event specs.

## Inputs Required

- COBOL source + copybooks (from repo)
- Pass 1: inventory entries, domain models, dependency graph
- Pass 2: business process docs, workflow YAMLs, rules YAMLs

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Functional Spec | `03_functional/prose/{program}_functional_spec.md` | `03_functional/specs/api-specs/{service}.openapi.yaml` |
| UI Mapping | `03_functional/prose/{transaction}_ui_mapping.md` | (embedded in OpenAPI) |
| Report Spec | `03_functional/prose/{report}_report_spec.md` | — |
| Interface Spec | `03_functional/prose/{interface}_interface_spec.md` | `03_functional/specs/interfaces/{name}.interface.yaml` |
| Event Spec | — | `03_functional/specs/events/{domain}-events.asyncapi.yaml` |

## Processing Instructions

```
1. For each program (by domain, in processing order):
   a. READ program source + copybooks from repo
   b. LOAD Pass 1 inventory and domain model as context
   c. LOAD Pass 2 business process and rules as context
   d. APPLY Prompt 3.1 (Functional Spec + API Contract)
   e. If CICS online: APPLY Prompt 3.2 (UI Mapping)
   f. If generates reports: APPLY Prompt 3.3 (Report Spec)
   g. If external interfaces: APPLY Prompt 3.4 (Interface Spec)
   h. WRITE outputs
   i. UPDATE processing log
```

---

## Prompt 3.1 — Functional Specification & API Contract

```
ROLE: You are BOTH a senior systems analyst (writing for Java developers 
who've NEVER seen COBOL) AND an API architect (producing OpenAPI 3.1 specs).

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

PRIOR CONTEXT (read these files):
- Inventory: {{OUTPUT_DIR}}/01_discovery/specs/inventory/{{PROGRAM_NAME}}.inventory.yaml
- Domain Model: {{OUTPUT_DIR}}/01_discovery/specs/domain-model/{{ENTITY}}.schema.yaml
- Business Process: {{OUTPUT_DIR}}/02_business/prose/{{CLUSTER}}_business_process.md
- Business Rules: {{OUTPUT_DIR}}/02_business/specs/rules/{{DOMAIN}}/{{CLUSTER}}.rules.yaml
- Workflow: {{OUTPUT_DIR}}/02_business/specs/workflows/{{PROCESS}}.workflow.yaml

TASK: Read the program from the repository and produce a dual-format 
functional specification.

Program: {{REPO_ROOT}}/{{PROGRAM_PATH}}
Copybooks: {{COPYBOOK_PATHS}}

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/03_functional/prose/{{PROGRAM_NAME}}_functional_spec.md

## Functional Specification: [Module/Function Name]
**Spec ID:** FS-{{SPEC_ID}}  |  **Source:** {{PROGRAM_NAME}}  |  **Status:** Draft

### 1. Purpose
[2-3 sentences. What does this function do? A Java developer must understand 
the full scope without seeing COBOL.]

### 2. Trigger
[What initiates this function — user action, schedule, event, upstream process]

### 3. Inputs
**[Input Name]** — from [source]
| Field | Type | Size/Precision | Required | Validation Rules | Description |
|---|---|---|---|---|---|
| [field] | [String/BigDecimal/LocalDate/etc.] | [details] | [Y/N] | [BR-xxx refs] | [meaning] |

### 4. Outputs
**[Output Name]** — to [destination]
| Field | Type | Size/Precision | Derivation | Description |
|---|---|---|---|---|
| [field] | [type] | [details] | [how calculated/derived] | [meaning] |

### 5. Processing Logic
Describe completely in structured English. Numbered steps. No COBOL terms.

**5.1 [Block Name]** (Source: paragraph [name], rules: BR-xxx)
1. [Step in clear English]
   - If [condition]: [action]
   - Else: [alternative]
2. [Next step]

**5.2 [Next Block]**
[Continue for all processing blocks...]

### 6. Validation Rules (Detail)
| Rule ID | Field(s) | Condition | Error Response | Error Code | HTTP Status |
|---|---|---|---|---|---|
| BR-V-xxx | [fields] | [exact condition] | [message] | [ERR-xxx] | [4xx] |

### 7. Calculations (Detail)
| Calc ID | Formula | Worked Example | Precision | Rounding | COBOL Source |
|---|---|---|---|---|---|
| BR-C-xxx | [formula] | [example with real numbers] | [decimal places] | [mode] | [program, paragraph] |

### 8. State Management
| Current State | Event/Condition | New State | Side Effects |
|---|---|---|---|
| [state] | [trigger] | [new state] | [what else happens] |

### 9. Error Handling
| Condition | Detection | Response | Recovery | Notification |
|---|---|---|---|---|
| [what can go wrong] | [how detected] | [system response] | [auto-recovery?] | [who's notified] |

### 10. Integration Points
| System/Service | Direction | Protocol | Format | Error Handling |
|---|---|---|---|---|
| [system] | [In/Out] | [REST/MQ/File] | [JSON/XML/Fixed] | [retry, fallback] |

### 11. Non-Functional
- **Volume:** [N transactions/day]
- **Performance:** [target response/throughput]
- **Concurrency:** [locking needs]
- **Idempotent:** [Y/N — safe to re-run?]
- **Audit:** [what needs logging]

### 12. Traceability
| Business Rule | Spec Section | COBOL Source |
|---|---|---|
| BR-xxx | §N | [program, paragraph] |

### 13. Open Items
| Item | Type | Description | Impact if Wrong |
|---|---|---|---|
| [item] | [ASSUMPTION / QUESTION / RISK] | [detail] | [consequence] |

---

FILE 2: {{OUTPUT_DIR}}/03_functional/specs/api-specs/{{SERVICE_NAME}}.openapi.yaml

If a service OpenAPI file already exists for this domain, APPEND to it.
Otherwise create a new one.

```yaml
openapi: "3.1.0"
info:
  title: "[Service Name]"
  version: "1.0.0"
  description: "[Service description]"
  x-cobol-source:
    programs: ["list"]
    cics_transactions: ["if applicable"]
  x-human-doc: "03_functional/prose/{{PROGRAM_NAME}}_functional_spec.md"
  x-service:
    bounded_context: "[domain]"

paths:
  /[resource-path]:
    [method]:
      operationId: "[operationName]"
      summary: "[description]"
      x-cobol-source:
        program: "{{PROGRAM_NAME}}"
        source_path: "{{PROGRAM_PATH}}"
        paragraph: "[paragraph]"
        cics_transaction: "[if applicable]"
      x-business-rules: ["BR-xxx list"]
      x-human-doc:
        spec: "FS-{{SPEC_ID}}"
        section: "[section number]"
      x-precision-critical:
        - field: "[field name]"
          calculation: "[formula]"
          scale: [N]
          rounding: "[mode]"
      parameters: [...]
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/[Schema]"
      responses:
        "200":
          description: "[success]"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/[Response]"
        "400":
          description: "Validation failure"
          x-business-rules: ["BR-V-xxx"]
        "404":
          description: "Not found"
          x-cobol-source:
            condition: "SQLCODE = +100 | FILE STATUS = '23' | RESP(NOTFND)"
        "409":
          description: "Conflict"
          x-cobol-source:
            condition: "SQLCODE = -803 | RESP(DUPRC)"
        "422":
          description: "Business rule violation"
          x-business-rules: ["BR-xxx"]
        "500":
          description: "Internal error"

components:
  schemas:
    [SchemaName]:
      type: object
      required: [list]
      properties:
        [field]:
          $ref: "[domain-model-schema.yaml reference]"
```
```

---

## Prompt 3.2 — UI/Screen Mapping (CICS Online Programs Only)

```
ROLE: You are a UI/UX analyst documenting legacy mainframe screens.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Functional Spec: {{OUTPUT_DIR}}/03_functional/prose/{{PROGRAM_NAME}}_functional_spec.md

TASK: Read the BMS map and COBOL program from the repository.

BMS Map: {{REPO_ROOT}}/{{BMS_PATH}}
Program: {{REPO_ROOT}}/{{PROGRAM_PATH}}
Copybooks: {{COPYBOOK_PATHS}}

===== PRODUCE ONE FILE =====

FILE: {{OUTPUT_DIR}}/03_functional/prose/{{TRANSACTION_ID}}_ui_mapping.md

[Document every screen field, PF key action, navigation path, validation 
trigger, and message. Include modern UI recommendations. Full template 
same as Pass 3 Prompt 3.2 in the master guide.]
```

---

## Prompt 3.3 — Report Specification (Batch Report Programs Only)

```
ROLE: You are a reporting analyst documenting legacy report specifications.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Functional Spec: {{OUTPUT_DIR}}/03_functional/prose/{{PROGRAM_NAME}}_functional_spec.md

TASK: Read the report-generating program from the repository.

Program: {{REPO_ROOT}}/{{PROGRAM_PATH}}
Copybooks: {{COPYBOOK_PATHS}}

===== PRODUCE ONE FILE =====

FILE: {{OUTPUT_DIR}}/03_functional/prose/{{REPORT_NAME}}_report_spec.md

[Document selection criteria, sort order, layout (headers, detail lines, 
group breaks, totals), calculations, and modernization recommendation 
(dashboard/API/PDF/eliminate). Full template same as master guide.]
```

---

## Prompt 3.4 — Interface & Integration Specification

```
ROLE: You are an integration architect documenting external interfaces.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Functional Spec: {{OUTPUT_DIR}}/03_functional/prose/{{PROGRAM_NAME}}_functional_spec.md

TASK: Read the program and JCL to document external interfaces.

Program: {{REPO_ROOT}}/{{PROGRAM_PATH}}
JCL: {{REPO_ROOT}}/{{JCL_PATH}}
Copybooks: {{COPYBOOK_PATHS}}

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/03_functional/prose/{{INTERFACE_NAME}}_interface_spec.md

[Document direction, protocol, format, record layout, validation, error 
handling, volume, and modernization recommendation.]

---

FILE 2: {{OUTPUT_DIR}}/03_functional/specs/interfaces/{{INTERFACE_NAME}}.interface.yaml

```yaml
interface:
  id: "INT-[N]"
  name: "[Interface Name]"
  direction: "[INBOUND | OUTBOUND | BIDIRECTIONAL]"
  external_system: "[name if known]"
  x-cobol-source:
    program: "{{PROGRAM_NAME}}"
    source_path: "{{PROGRAM_PATH}}"
    jcl_job: "[job name]"
  x-human-doc: "03_functional/prose/{{INTERFACE_NAME}}_interface_spec.md"

  current:
    protocol: "[FILE_TRANSFER | MQ | CICS_LINK | DB2_SHARED]"
    format: "[FIXED_WIDTH | DELIMITED | XML | JSON]"
    encoding: "[EBCDIC | ASCII | UTF-8]"
    record_length: [bytes]

  proposed:
    protocol: "[REST_API | EVENT_STREAM | BATCH_FILE | DATABASE]"
    format: "[JSON | CSV | AVRO]"
    rationale: "[why]"

  record_layout:
    - field: "[name]"
      position: "[start-end]"
      length: [N]
      type: "[Alpha | Numeric | PackedDecimal]"
      required: [true | false]
      x-cobol-field: "[COBOL field name]"
      x-domain-model-ref: "[schema.yaml#/properties/field]"

  validation:
    - check: "[what's validated]"
      on_failure: "[REJECT_RECORD | REJECT_FILE | DEFAULT | LOG]"

  volume:
    typical: [N]
    peak: [N]
    processing_window: "[SLA]"
```

If async/event interactions are found, also produce:
FILE 3: {{OUTPUT_DIR}}/03_functional/specs/events/{{DOMAIN}}-events.asyncapi.yaml
(AsyncAPI 3.0 format — append if file exists)
```

---

## Quality Gate Checklist

Before proceeding to Pass 4, verify:

- [ ] Every program has a functional specification in `03_functional/prose/`
- [ ] API specs exist for each proposed service in `03_functional/specs/api-specs/`
- [ ] All OpenAPI specs are syntactically valid (test with openapi-generator validate)
- [ ] All `$ref` references resolve to existing schemas
- [ ] All `BR-xxx` references in `x-business-rules` exist in the rules YAML from Pass 2
- [ ] All `x-cobol-source` references point to real files in the repo
- [ ] CICS programs have UI mapping documents
- [ ] Report programs have report specifications
- [ ] External interfaces have interface specs
- [ ] Processing log is updated
- [ ] **TECHNICAL REVIEW CHECKPOINT:** COBOL developer validates 10-15% of functional specs against actual source code
