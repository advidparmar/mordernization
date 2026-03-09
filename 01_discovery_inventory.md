# Pass 1 — Discovery & Inventory

## Purpose

Build a complete structural map of the codebase. This pass produces an inventory entry, dependency extract, and data dictionary for every program — both human-readable and machine-readable.

## Inputs Required

- COBOL program source (read from repo)
- All copybooks referenced by the program (resolved from `00_repo_scan/copybook_dependencies.yaml`)
- JCL files associated with the program (if identifiable)
- Repository scan outputs from Step 0

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Inventory Entry | `01_discovery/prose/{program}_inventory.md` | `01_discovery/specs/inventory/{program}.inventory.yaml` |
| Data Dictionary | `01_discovery/prose/{program}_data_dictionary.md` | `01_discovery/specs/domain-model/{entity}.schema.yaml` |
| JCL Analysis | `01_discovery/prose/{job}_jcl_analysis.md` | (embedded in inventory YAML) |

## Processing Instructions

For each COBOL program in the processing order defined in `00_repo_scan/program_tiers.yaml`:

```
1. READ the program source file from the repository
2. READ all copybooks listed in copybook_dependencies.yaml for this program
3. READ associated JCL if identifiable
4. APPLY Prompt 1.1 (Inventory + Dependencies) below
5. APPLY Prompt 1.2 (Data Dictionary + Domain Model) below
6. If JCL exists, APPLY Prompt 1.3 (JCL Analysis) below
7. WRITE outputs to the appropriate directories
8. UPDATE the processing log
```

---

## Prompt 1.1 — Inventory & Dependency Extraction

For each COBOL program, produce dual-format inventory and dependency analysis.

```
ROLE: You are a COBOL mainframe analyst performing an inventory assessment.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Repository scan results are available in {{OUTPUT_DIR}}/00_repo_scan/

TASK: Read the following COBOL program from the repository and produce 
a dual-format inventory entry. Be exhaustive on dependencies — a missed 
dependency means a gap in the entire modernization.

Read the program source from: {{REPO_ROOT}}/{{PROGRAM_PATH}}

Read its copybooks from:
{{COPYBOOK_PATHS}}  (list from copybook_dependencies.yaml)

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/01_discovery/prose/{{PROGRAM_NAME}}_inventory.md

## Program Summary: {{PROGRAM_NAME}}

**What This Program Does:**
[2-3 sentences in plain business English. No COBOL jargon.]

**Business Process:** [Which process this belongs to]
**Program Type:** [Batch / Online / Subroutine / Utility]
**Complexity:** [Low / Medium / High / Very High] — [rationale]

**What It Depends On:**
[Narrative describing programs called, data accessed, external systems]

**What Depends On It:**
[Cross-reference from call_graph.yaml — which programs call this one]

**Key Business Rules Found:**
[List important rules with preliminary BR-xxx IDs]

**Flags for Human Review:**
[Dead code, commented sections, hardcoded values, possible bugs, 
unclear logic, compliance-sensitive sections]

**Data Stores Touched:**
| Data Store | Type | Access | Brief Purpose |
|---|---|---|---|
| [name] | [DB2/VSAM/Seq/MQ] | [Read/Write/Both] | [purpose] |

**External Dependencies:**
| Type | Name | Direction | Details |
|---|---|---|---|
| Program Call | [name] | [Calls / Called By] | [STATIC/DYNAMIC, parameters] |
| Copybook | [name] | Used | [DATA/PROCEDURE division] |
| File | [FD name / DD name] | [Read/Write] | [Sequential/VSAM/etc.] |
| Database | [table name] | [Select/Insert/Update/Delete] | [key columns] |
| CICS Resource | [name] | [Send/Receive/etc.] | [MAP/TDQ/TSQ] |
| MQ Queue | [name] | [Put/Get] | [message format] |

---

FILE 2: {{OUTPUT_DIR}}/01_discovery/specs/inventory/{{PROGRAM_NAME}}.inventory.yaml

```yaml
program:
  id: "[PROGRAM-ID from source]"
  source_file: "{{PROGRAM_PATH}}"
  type: "[BATCH | ONLINE_CICS | SUBROUTINE | UTILITY]"
  lines_of_code: [count]
  complexity: "[LOW | MEDIUM | HIGH | VERY_HIGH]"
  complexity_rationale: "[reason]"
  summary: "[one-line description]"
  business_process: "[process name]"
  domain: "[bounded context / business domain]"

  divisions:
    identification:
      program_id: "[value]"
      author: "[if present]"
      date_written: "[if present]"
    environment:
      file_assignments:
        - select_name: "[SELECT name]"
          assign_to: "[DD name or file]"
    data:
      fd_count: [N]
      working_storage_01_count: [N]
      linkage_section_items: [N]
      copybooks_in_data:
        - "[copybook name]"
    procedure:
      section_count: [N]
      paragraph_count: [N]
      entry_point: "[main section/paragraph]"
      exit_point: "[STOP RUN / GOBACK / CICS RETURN]"

  dependencies:
    calls_programs:
      - program: "[called program name]"
        call_type: "[STATIC | DYNAMIC | CICS_LINK | CICS_XCTL]"
        parameters:
          - name: "[USING parameter name]"
            type: "[data type]"
            direction: "[IN | OUT | IN-OUT]"
        location: "[paragraph/section where called]"
        purpose: "[why it calls this program]"

    called_by:
      - program: "[calling program name]"
        source: "call_graph.yaml cross-reference"

    copybooks:
      - name: "[copybook name]"
        resolved_path: "[path in repo]"
        division: "[DATA | PROCEDURE | BOTH]"
        purpose: "[what this copybook provides]"

    files:
      - fd_name: "[FD name]"
        dd_name: "[DD/assignment name]"
        access_mode: "[SEQUENTIAL | RANDOM | DYNAMIC]"
        record_format: "[FB | VB | etc.]"
        record_length: [bytes]
        operations: ["READ", "WRITE", "REWRITE", "DELETE"]
        purpose: "[what this file contains]"

    database:
      - table: "[table/view name]"
        operation: "[SELECT | INSERT | UPDATE | DELETE]"
        sql_type: "[STATIC | DYNAMIC]"
        cursor_name: "[if applicable]"
        key_columns: ["col1", "col2"]
        purpose: "[what this query does]"

    cics_resources:
      - type: "[MAP | TDQ | TSQ | FILE | PROGRAM | QUEUE]"
        name: "[resource name]"
        operation: "[SEND | RECEIVE | WRITEQ | READQ | etc.]"

    mq_queues:
      - queue: "[queue name]"
        operation: "[PUT | GET]"
        message_format: "[copybook or format]"

  data_flow:
    inputs:
      - source: "[description]"
        type: "[FILE | DB2 | SCREEN | MQ | LINKAGE]"
        format: "[copybook or layout name]"
    outputs:
      - target: "[description]"
        type: "[FILE | DB2 | SCREEN | MQ | REPORT]"
        format: "[copybook or layout name]"

  preliminary_business_rules:
    - id: "BR-[category]-[number]"
      summary: "[brief rule description]"
      location: "[paragraph/section]"
      confidence: "[HIGH | MEDIUM | LOW]"

  flags:
    - type: "[DEAD_CODE | UNCLEAR_LOGIC | HARDCODED_VALUE | POSSIBLE_BUG | COMPLIANCE | OTHER]"
      description: "[what was found]"
      location: "[where in the code]"
      action_needed: "[INVESTIGATE | SME_REVIEW | DOCUMENT | IGNORE]"
```
```

---

## Prompt 1.2 — Data Dictionary & Domain Model

For each COBOL program, extract all data structures into dual format.

```
ROLE: You are a COBOL data analyst cataloging all data structures.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Inventory entry just produced: {{OUTPUT_DIR}}/01_discovery/specs/inventory/{{PROGRAM_NAME}}.inventory.yaml

TASK: Read the COBOL program and its copybooks from the repository. 
Extract a complete data dictionary.

Program: {{REPO_ROOT}}/{{PROGRAM_PATH}}
Copybooks: {{COPYBOOK_PATHS}}

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/01_discovery/prose/{{PROGRAM_NAME}}_data_dictionary.md

## Data Dictionary: {{PROGRAM_NAME}}

**Overview:**
[2-3 sentences describing the major data structures and business purpose]

### Record Layouts

For each major record (FD or significant 01-level group):

**[Record Name] — [Business Purpose]**
[One paragraph explaining what this record represents]

| Field | Business Name | Description | Type | Example Values | Notes |
|---|---|---|---|---|---|
| [COBOL name] | [human name] | [meaning] | [readable type] | [examples] | [gotchas] |

### Key Data Relationships
[How structures relate — foreign keys, linked records, shared fields]

### COBOL-Specific Patterns
[REDEFINES, packed decimals, 88-levels, OCCURS, sign handling — 
explained for non-COBOL audience]

### Data Quality Observations
[Spaces vs nulls, sentinel dates, potential overflow, invalid values]

### Condition Names (88-levels)
| Parent Field | Condition Name | Value(s) | Business Meaning |
|---|---|---|---|
| [parent] | [88-level] | [VALUE clause] | [what it represents] |

### Constants & Magic Numbers
| Location | Value | Usage Context | Likely Meaning | SME Review Needed |
|---|---|---|---|---|
| [paragraph] | [value] | [how used] | [interpretation] | [Y/N] |

---

FILE 2: {{OUTPUT_DIR}}/01_discovery/specs/domain-model/{{ENTITY_NAME}}.schema.yaml

Produce one schema file per major entity/record. If a copybook defines 
a single logical entity, name the file after the entity.

```yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
$id: "https://{{SYSTEM_NAME}}/schemas/{{ENTITY_NAME}}.yaml"
title: "[EntityName]"
description: "[Business description of this entity]"

x-cobol-source:
  copybook: "[copybook name]"
  programs: ["list of programs using this structure"]
  source_path: "[path in repo]"

x-database:
  table_name: "[proposed table name]"
  schema: "[proposed schema name]"

type: object
required: [list of required fields]

properties:
  [fieldName]:
    type: "[string | number | integer | boolean]"
    format: "[date | decimal | etc. if applicable]"
    maxLength: [if string]
    pattern: "[regex if applicable]"
    enum: [if fixed values]
    description: "[business description]"
    x-cobol-field:
      name: "[COBOL field name]"
      pic: "[PIC clause]"
      storage: "[alphanumeric | zoned-decimal | packed-decimal | binary]"
      bytes: [storage size]
      level: "[01 | 05 | 10 | etc.]"
      parent: "[parent group field]"
      condition_names:
        "[value]": "[88-level condition name]"
      redefines: "[field it redefines, if applicable]"
    x-database:
      column: "[proposed column name]"
      type: "[SQL data type]"
      constraints: ["PK", "FK:other_table.column", "UNIQUE", "NOT NULL"]
    x-java:
      type: "[Java type]"
      validation: "[Bean Validation annotations]"
    x-precision:
      integer_digits: [N]
      decimal_digits: [N]
      rounding: "[HALF_UP | TRUNCATE]"
      signed: [true | false]
      warning: "[any precision concern]"
    x-migration:
      transform: "[DIRECT | TRIM | MAP | DATE_PARSE | DECIMAL_CONVERT]"
      null_representations: ["00000000", "        "]
      notes: "[migration concern]"
    x-business-rules: ["BR-xxx references"]
```
```

---

## Prompt 1.3 — JCL Job Stream Analysis

For each JCL file, document the batch job structure.

```
ROLE: You are a mainframe operations analyst documenting batch job streams.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Inventory entries for programs referenced in this JCL are available in:
  {{OUTPUT_DIR}}/01_discovery/specs/inventory/

TASK: Read the following JCL from the repository and document the 
complete job stream.

JCL file: {{REPO_ROOT}}/{{JCL_PATH}}

===== PRODUCE ONE FILE =====

FILE: {{OUTPUT_DIR}}/01_discovery/prose/{{JOB_NAME}}_jcl_analysis.md

## Job Stream Analysis — [Job Name]

### Job Overview
- **Job Name:** [JOB card name]
- **Job Class:** [CLASS parameter]
- **Description:** [What this job does end-to-end]
- **Typical Schedule:** [If determinable from comments or naming]
- **Programs Executed:** [List all PGM= values with links to inventory]

### Step-by-Step Breakdown

**Step [N]: [Step Name]**
- **Program:** [PGM= value] (see inventory: [link to inventory entry])
- **Condition:** [COND= — what conditions skip this step]
- **Purpose:** [What this step accomplishes]
- **Input Files:**
  | DD Name | DSN | DISP | Description |
  |---|---|---|---|
  | [DD] | [dataset] | [DISP] | [purpose] |
- **Output Files:**
  | DD Name | DSN | DISP | Space | Description |
  |---|---|---|---|---|
  | [DD] | [dataset] | [DISP] | [SPACE] | [purpose] |
- **SYSIN:** [Inline parameters or control cards]

[Repeat for all steps...]

### Data Flow Between Steps
[Which step's output feeds the next step's input]

### Restart/Recovery
- **Safe Restart Points:** [Steps that can be restarted independently]
- **Manual Intervention:** [Steps needing operator action]

### Condition Code Logic
| Step | Expected RC | On Failure | Downstream Impact |
|---|---|---|---|
| [step] | [RC] | [action] | [which steps skip] |

### Modernization Notes
[Should this become a Spring Batch job, a workflow, or event-driven? Why?]
```

---

## Post-Pass 1 Aggregation

After all programs have been processed through Pass 1:

```
TASK: Generate an aggregated dependency graph visualization.

Read all inventory YAML files from:
{{OUTPUT_DIR}}/01_discovery/specs/inventory/

Produce: {{OUTPUT_DIR}}/01_discovery/prose/FULL_DEPENDENCY_MAP.md

Contents:
- Complete program-to-program call graph (text-based)
- Programs grouped by domain cluster
- Shared copybooks and their usage frequency
- Data stores and which programs access them
- Programs with no callers (entry points / potential dead code)
- Programs with most dependencies (highest risk)
- Suggested bounded contexts / service boundaries based on clustering

Also produce: {{OUTPUT_DIR}}/01_discovery/specs/dependency_graph.yaml
with the complete machine-readable graph.
```

---

## Quality Gate Checklist

Before proceeding to Pass 2, verify:

- [ ] Every COBOL program has an inventory YAML in `01_discovery/specs/inventory/`
- [ ] Every COBOL program has a prose inventory in `01_discovery/prose/`
- [ ] Every major copybook has a domain model schema in `01_discovery/specs/domain-model/`
- [ ] All inventory YAML files are syntactically valid
- [ ] All domain model schemas conform to JSON Schema
- [ ] `called_by` fields are populated via cross-reference
- [ ] Unresolved copybooks are flagged in the processing log
- [ ] Domain clusters are identified for Pass 2 grouping
- [ ] Processing log is updated with all completed programs
