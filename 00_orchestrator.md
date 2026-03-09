# COBOL Modernization — Master Orchestrator

## Overview

This is the master prompt that drives the entire COBOL modernization analysis. It assumes your COBOL source code is already checked into a Git repository and uses an LLM coding agent (Claude Code, Cursor, Aider, Copilot, etc.) to systematically analyze the codebase.

## Prerequisites

Before running this process, ensure:
1. The COBOL repository is cloned locally and accessible to your coding agent
2. The repo contains: COBOL source programs, copybooks, JCL files, and optionally BMS maps and DB2 DDL
3. You have identified the output directory where analysis artifacts will be written

## Repository Configuration

Update these variables before starting:

```
REPO_ROOT=<path-to-your-cobol-repo>
OUTPUT_DIR=<path-to-output-directory>
SYSTEM_NAME=<name-of-your-system>
DOMAIN_CONTEXT=<brief-description-of-business-domain>
```

**Example:**
```
REPO_ROOT=/home/user/repos/loan-processing-system
OUTPUT_DIR=/home/user/repos/loan-processing-system/modernization-output
SYSTEM_NAME=LoanProcessingSystem
DOMAIN_CONTEXT=Retail banking loan origination, servicing, and collections system
```

---

## Master Orchestrator Prompt

Copy and paste this entire prompt into your coding agent session. It will scan the repo, build context, and then systematically execute each pass.

---

```
ROLE: You are a COBOL modernization architect. You have access to a Git 
repository containing a complete COBOL codebase. Your job is to 
systematically analyze this codebase and produce modernization-ready 
specifications — both human-readable documents and machine-readable 
contracts.

You will work through 5 passes, each building on the previous. For each 
pass, you will follow the detailed instructions in the corresponding 
phase markdown file. I will reference these files as we go.

REPOSITORY LOCATION: {{REPO_ROOT}}
OUTPUT DIRECTORY: {{OUTPUT_DIR}}
SYSTEM NAME: {{SYSTEM_NAME}}
DOMAIN CONTEXT: {{DOMAIN_CONTEXT}}

====================================================================
STEP 0: REPOSITORY SCAN & PREPARATION
====================================================================

Before any analysis, scan the repository and build a complete picture 
of what we're working with.

TASKS:

0.1 SCAN THE REPOSITORY STRUCTURE
    - List all directories and understand the repo layout
    - Identify where COBOL source programs are stored (common patterns: 
      src/, cobol/, source/, programs/, or flat in root)
    - Identify where copybooks are stored (common patterns: copy/, 
      copybook/, cpy/, include/)
    - Identify where JCL files are stored (common patterns: jcl/, 
      jobs/, proc/)
    - Identify where BMS maps are stored (if any)
    - Identify any DB2 DDL files
    - Note any README or existing documentation

0.2 BUILD THE FILE INVENTORY
    Create a file at {{OUTPUT_DIR}}/00_repo_scan/file_inventory.yaml:
    
    ```yaml
    repository:
      root: "{{REPO_ROOT}}"
      scanned_date: "<today's date>"
      
    source_locations:
      cobol_programs:
        path: "<detected path>"
        extensions: [".cbl", ".cob", ".cobol", ".CBL"]
        count: <number found>
        files:
          - name: "<filename>"
            path: "<relative path>"
            size_lines: <line count>
      
      copybooks:
        path: "<detected path>"
        extensions: [".cpy", ".copy", ".CPY", ".cbk"]
        count: <number found>
        files:
          - name: "<filename>"
            path: "<relative path>"
      
      jcl:
        path: "<detected path>"
        extensions: [".jcl", ".JCL", ".proc", ".PROC"]
        count: <number found>
        files:
          - name: "<filename>"
            path: "<relative path>"
      
      bms_maps:
        path: "<detected path or 'not found'>"
        count: <number found>
      
      db2_ddl:
        path: "<detected path or 'not found'>"
        count: <number found>
      
      other:
        - type: "<any other relevant file types>"
          path: "<path>"
          count: <number>
    ```

0.3 BUILD THE COPYBOOK DEPENDENCY MAP
    For each COBOL program, scan for COPY statements and map which 
    copybooks each program uses. Create:
    {{OUTPUT_DIR}}/00_repo_scan/copybook_dependencies.yaml
    
    ```yaml
    dependencies:
      programs:
        - program: "<program filename>"
          copybooks_used:
            - name: "<copybook name from COPY statement>"
              resolved_path: "<actual file path in repo, or 'UNRESOLVED'>"
      
      unresolved_copybooks:
        - name: "<copybook name>"
          referenced_by: ["<program1>", "<program2>"]
          note: "Could not find this copybook in the repository"
      
      copybook_usage:
        - copybook: "<copybook name>"
          used_by: ["<program1>", "<program2>", ...]
          usage_count: <number of programs using it>
    ```

0.4 BUILD THE CALL GRAPH (QUICK SCAN)
    For each COBOL program, scan for CALL statements (both static 
    and dynamic). Create:
    {{OUTPUT_DIR}}/00_repo_scan/call_graph.yaml
    
    ```yaml
    call_graph:
      programs:
        - program: "<program name>"
          calls:
            - target: "<called program>"
              type: "<STATIC | DYNAMIC>"
              location: "<approximate line or section>"
          called_by: []  # Will be populated by cross-referencing
      
      entry_points:
        - "<programs that are never called by other programs>"
      
      leaf_programs:
        - "<programs that don't call any other programs>"
      
      clusters:
        - name: "<suggested domain/cluster name>"
          programs: ["<list of tightly connected programs>"]
          rationale: "<why these belong together>"
    ```

0.5 CLASSIFY PROGRAMS BY TIER
    Based on the quick scan, classify each program:
    {{OUTPUT_DIR}}/00_repo_scan/program_tiers.yaml
    
    ```yaml
    tiers:
      tier_1_critical:
        description: "Complex programs with financial calculations, 
                      heavy DB2, many dependencies, or regulatory logic"
        programs:
          - name: "<program>"
            reason: "<why tier 1>"
      
      tier_2_important:
        description: "Moderate complexity business logic programs"
        programs:
          - name: "<program>"
            reason: "<why tier 2>"
      
      tier_3_commodity:
        description: "Simple CRUD, utilities, reports, data movement"
        programs:
          - name: "<program>"
            reason: "<why tier 3>"
    
    processing_order:
      description: "Recommended order to analyze programs"
      order:
        - phase: 1
          programs: ["<list — start with most-called subroutines and 
                      shared copybooks so context is available for callers>"]
        - phase: 2
          programs: ["<next batch>"]
    ```

0.6 CREATE OUTPUT DIRECTORY STRUCTURE
    Create the following directory structure under {{OUTPUT_DIR}}:
    
    ```
    {{OUTPUT_DIR}}/
    ├── 00_repo_scan/
    │   ├── file_inventory.yaml
    │   ├── copybook_dependencies.yaml
    │   ├── call_graph.yaml
    │   └── program_tiers.yaml
    ├── 01_discovery/
    │   ├── prose/
    │   └── specs/
    │       ├── inventory/
    │       └── domain-model/
    ├── 02_business/
    │   ├── prose/
    │   └── specs/
    │       ├── workflows/
    │       └── rules/
    ├── 02b_product_stories/
    │   ├── prose/
    │   └── specs/
    ├── 03_functional/
    │   ├── prose/
    │   └── specs/
    │       ├── api-specs/
    │       ├── interfaces/
    │       └── events/
    ├── 04_technical/
    │   ├── prose/
    │   └── specs/
    │       ├── features/
    │       └── migration/
    ├── 05_security/
    │   ├── prose/
    │   └── specs/
    ├── 06_data_quality/
    │   ├── prose/
    │   ├── specs/
    │   └── scripts/
    ├── 07_operations/
    │   ├── prose/
    │   └── specs/
    ├── 08_governance/
    │   ├── prose/
    │   └── specs/
    └── logs/
        └── processing_log.yaml
    ```

0.7 INITIALIZE PROCESSING LOG
    Create {{OUTPUT_DIR}}/logs/processing_log.yaml to track progress:
    
    ```yaml
    processing_log:
      system: "{{SYSTEM_NAME}}"
      started: "<timestamp>"
      status: "IN_PROGRESS"
      
      pass_1_discovery:
        status: "NOT_STARTED"
        programs_completed: []
        programs_remaining: ["<all programs>"]
      
      pass_2_business:
        status: "NOT_STARTED"
        programs_completed: []
        programs_remaining: []
      
      pass_3_functional:
        status: "NOT_STARTED"
        programs_completed: []
        programs_remaining: []
      
      pass_4_technical:
        status: "NOT_STARTED"
        programs_completed: []
        programs_remaining: []
      
      pass_5_security:
        status: "NOT_STARTED"
        domains_completed: []
        domains_remaining: []
      
      pass_6_data_quality:
        status: "NOT_STARTED"
        entities_completed: []
        entities_remaining: []
      
      pass_7_operations:
        status: "NOT_STARTED"
        interview_template_generated: false
        transition_plan_generated: false
      
      pass_8_governance:
        status: "NOT_STARTED"
    ```

After completing Step 0, report:
- Total programs found
- Total copybooks found
- Total JCL files found
- Number of unresolved copybook references (these are a risk)
- Suggested domain clusters
- Recommended processing order
- Any concerns or anomalies discovered

Then we will proceed to Pass 1 using the instructions in 
01_discovery_inventory.md.

====================================================================
STEP 1: PASS 1 — DISCOVERY & INVENTORY
====================================================================

Follow the instructions in: 01_discovery_inventory.md

For each COBOL program (in the processing order from Step 0):

1. Read the program source from the repository
2. Read all its copybooks (resolved in Step 0.3)
3. Read its JCL (if identifiable from Step 0)
4. Apply the prompts from 01_discovery_inventory.md
5. Write human-readable output to: {{OUTPUT_DIR}}/01_discovery/prose/
6. Write machine-readable output to: {{OUTPUT_DIR}}/01_discovery/specs/
7. Update the processing log

QUALITY GATE after Pass 1:
- Every program has an inventory entry
- Every copybook dependency is resolved or flagged
- Domain model schemas are valid YAML
- Update call_graph.yaml with "called_by" fields (cross-reference)

====================================================================
STEP 2: PASS 2 — BUSINESS SPECIFICATIONS
====================================================================

Follow the instructions in: 02_business_specifications.md

Process programs grouped by domain cluster (from Step 0.5).
Feed all programs in a domain together when possible.

1. For each domain cluster, read all programs in the cluster
2. Include Pass 1 outputs as context
3. Apply the prompts from 02_business_specifications.md
4. Write outputs to: {{OUTPUT_DIR}}/02_business/
5. After all domains: run the Business Rules Aggregation prompt

QUALITY GATE after Pass 2:
- Every business process has a document
- Business rule IDs are unique (no duplicates)
- All rules have category and confidence rating
- SME review flag: list rules needing validation

====================================================================
STEP 2B: PASS 2B — PRODUCT STORIES & ACCEPTANCE CRITERIA
====================================================================

Follow the instructions in: 02b_product_stories.md

This is the PRODUCT MANAGER GATE. Nothing proceeds to functional 
or technical specs without PM review and approval of these stories.

1. For each domain cluster, generate user stories with Given-When-Then 
   acceptance criteria in plain business English
2. Generate PM review worksheets and story map
3. Write outputs to: {{OUTPUT_DIR}}/02b_product_stories/
4. **STOP AND WAIT FOR PM REVIEW**

PMs mark each story as:
- KEEP AS-IS → Proceeds to Pass 3 unchanged
- MODIFY → Proceeds to Pass 3 with PM's changes incorporated
- ELIMINATE → Skipped in Pass 3 (no functional spec generated)
- ENHANCE → Proceeds to Pass 3 with new requirements added

QUALITY GATE after Pass 2B:
- Every business process has corresponding user stories
- Every business rule is referenced by at least one story
- Stories are in plain English (no technical jargon)
- PM review worksheets are generated
- Story map shows end-to-end user journeys
- **PM MUST REVIEW AND MARK UP STORIES BEFORE PROCEEDING**

====================================================================
STEP 3: PASS 3 — FUNCTIONAL SPECIFICATIONS
====================================================================

Follow the instructions in: 03_functional_specifications.md

Process ONLY stories that PMs marked as KEEP, MODIFY, or ENHANCE.
Skip stories marked ELIMINATE.
Incorporate PM notes from MODIFY and ENHANCE stories.

Process programs by domain, including Pass 1, Pass 2, and Pass 2B 
outputs as context.

1. For each program/module (where stories are approved), read source + copybooks
2. Include inventory, business process docs, rules, AND PM-approved stories as context
3. For MODIFY stories, incorporate PM's change notes into the functional spec
4. For ENHANCE stories, add the new requirements alongside existing functionality
5. Apply the prompts from 03_functional_specifications.md
6. Write outputs to: {{OUTPUT_DIR}}/03_functional/
7. Include UI mapping for CICS programs, report specs for batch

QUALITY GATE after Pass 3:
- Every program has a functional spec
- OpenAPI specs validate successfully
- All BR-xxx references resolve to existing rules
- Integration points are documented

====================================================================
STEP 4: PASS 4 — TECHNICAL SPECIFICATIONS
====================================================================

Follow the instructions in: 04_technical_specifications.md

Process programs individually, with full context from all prior passes.

1. For each program, read source + copybooks
2. Include ALL prior outputs as context
3. Apply the prompts from 04_technical_specifications.md
4. Write outputs to: {{OUTPUT_DIR}}/04_technical/
5. Generate Gherkin features alongside tech specs

QUALITY GATE after Pass 4:
- Every program has a technical spec
- Gherkin features parse without errors
- Every business rule has at least one test scenario
- Migration mappings cover all data stores
- Precision warnings are explicit for all financial fields

====================================================================
STEP 5: PASS 5 — SECURITY & ACCESS CONTROL
====================================================================

Follow the instructions in: 05_security_access_control.md

Analyze security patterns in the code and flag what must be gathered 
from the mainframe security team.

1. Scan for security-related files in the repo (CSD, RACF, GRANT scripts)
2. For each domain, extract authentication, authorization, and data 
   protection patterns from the COBOL source
3. Classify all data fields by sensitivity
4. Write outputs to: {{OUTPUT_DIR}}/05_security/
5. Generate the operations interview questions for security gaps

QUALITY GATE after Pass 5:
- Every API endpoint has an authorization rule
- All PII/PCI/PHI fields are classified
- Gaps requiring mainframe security team input are listed

====================================================================
STEP 6: PASS 6 — DATA QUALITY & MIGRATION READINESS
====================================================================

Follow the instructions in: 06_data_quality_migration.md

Go beyond structure to analyze actual data quality risks and produce 
the definitive migration plan.

1. For each entity, analyze COBOL data definitions for quality risks
2. Generate validation SQL scripts for the DBA to run against production
3. Define migration sequence respecting referential integrity
4. Produce the comprehensive migration plan with rollback procedures
5. Write outputs to: {{OUTPUT_DIR}}/06_data_quality/

QUALITY GATE after Pass 6:
- Every entity has a data quality risk assessment
- Validation scripts are generated
- Migration sequence is defined
- Financial reconciliation checks have zero tolerance
- Rollback plan exists for every stage

====================================================================
STEP 7: PASS 7 — OPERATIONS & TRANSITION PLANNING
====================================================================

Follow the instructions in: 07_operations_transition.md

Capture operational knowledge and design the production cutover.

1. Infer operational characteristics from JCL and code patterns
2. Generate the operations team interview template
3. Design the transition plan (parallel run, canary, cutover sequence)
4. Define SLAs and monitoring for the Java system
5. Write outputs to: {{OUTPUT_DIR}}/07_operations/

QUALITY GATE after Pass 7:
- Operations interview template is complete and specific
- Transition plan has rollback for every phase
- Parallel run criteria defined
- Service cutover sequence aligns with service decomposition

====================================================================
STEP 8: PASS 8 — GOVERNANCE & CROSS-CUTTING
====================================================================

Follow the instructions in: 08_governance_crosscutting.md

This pass aggregates across ALL prior outputs (Passes 1-7).

1. Collect all business rules, specs, open items, and risks from all passes
2. Apply the governance prompts from 08_governance_crosscutting.md
3. Write outputs to: {{OUTPUT_DIR}}/08_governance/
4. Generate final traceability matrix, risk register, and executive summary

QUALITY GATE after Pass 8:
- Traceability matrix shows >95% coverage
- All risks have severity, owner, and recommended action
- Executive summary is coherent
- Processing log shows all programs completed through all passes

====================================================================
FINAL: COMPLETION REPORT
====================================================================

After all passes, generate:
{{OUTPUT_DIR}}/COMPLETION_REPORT.md

Contents:
- Total programs analyzed
- Total business rules extracted
- Total API endpoints defined
- Total test scenarios generated
- Coverage percentage
- Top 10 risks
- Unresolved items requiring human review
- Recommended next steps for development team
```

---

## How to Use This Orchestrator

### With Claude Code (Terminal)

```bash
cd /path/to/your/cobol-repo
claude

# Then paste or reference the orchestrator prompt, updating variables:
# REPO_ROOT=/path/to/your/cobol-repo
# OUTPUT_DIR=/path/to/your/cobol-repo/modernization-output
# SYSTEM_NAME=YourSystemName
# DOMAIN_CONTEXT=Your business domain description
```

### With Cursor / Windsurf / Copilot

Open your COBOL repository in the IDE, then paste the orchestrator prompt into the AI chat. The agent will have access to the file system to read source files and write outputs.

### With API Pipeline

Use the orchestrator logic as a script driver. For each step, assemble the prompt programmatically by reading the appropriate phase markdown, injecting the source code and context, calling the API, and storing the output.

### Processing Large Codebases

For codebases with 100+ programs, the orchestrator will hit context window limits if you try to process everything in one session. Instead:

1. Run Step 0 (repo scan) in one session — this produces the processing plan
2. For each subsequent pass, process programs in batches of 5-10 per session
3. Always load the processing log at the start of each session to know where you left off
4. Feed prior outputs as context (summaries, not full text, to save context window)

### Resuming After Interruption

The processing log (`logs/processing_log.yaml`) tracks which programs have been completed in each pass. To resume:

```
I'm resuming the COBOL modernization analysis.

Repository: {{REPO_ROOT}}
Output directory: {{OUTPUT_DIR}}

Please read the processing log at {{OUTPUT_DIR}}/logs/processing_log.yaml 
and continue from where we left off. The phase markdown files are at:
[path to phase files]
```
