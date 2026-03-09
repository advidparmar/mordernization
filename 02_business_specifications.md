# Pass 2 — Business Specifications

## Purpose

Extract business-level meaning from the code. The audience is business analysts, product owners, and stakeholders who do NOT read COBOL. This pass also produces machine-readable workflow definitions and business rules with embedded test cases.

## Inputs Required

- COBOL program source + copybooks (from repo)
- Pass 1 outputs: inventory entries, domain model schemas, dependency graph, domain clusters
- Programs should be processed by domain cluster (not individually) when possible

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Business Process | `02_business/prose/{domain}_business_process.md` | `02_business/specs/workflows/{process}.workflow.yaml` |
| Business Rules | (embedded in process doc) | `02_business/specs/rules/{domain}/{ruleset}.rules.yaml` |
| Rules Catalog | `02_business/prose/MASTER_BUSINESS_RULES_CATALOG.md` | `02_business/specs/rules/master_rules_catalog.yaml` |

## Processing Instructions

Process programs grouped by domain cluster from `00_repo_scan/program_tiers.yaml`:

```
1. For each domain cluster:
   a. READ all programs in the cluster from the repository
   b. READ all their copybooks
   c. LOAD their Pass 1 inventory entries as context
   d. APPLY Prompt 2.1 (Business Process + Workflow + Rules)
   e. WRITE outputs to the appropriate directories

2. After all domains are processed:
   a. APPLY Prompt 2.2 (Business Rules Aggregation)
   b. WRITE the master catalog

3. UPDATE the processing log
```

---

## Prompt 2.1 — Business Process, Workflow & Rules

For each domain cluster, produce dual-format business documentation.

```
ROLE: You are BOTH a senior business analyst (writing for a business 
audience — no technical jargon) AND a workflow architect (producing 
machine-readable state machines and rules).

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Domain Cluster: {{CLUSTER_NAME}}
- Programs in this cluster: {{PROGRAM_LIST}}

PRIOR CONTEXT (read these files for context):
- Inventory entries: {{OUTPUT_DIR}}/01_discovery/specs/inventory/{{PROGRAM_NAME}}.inventory.yaml
  (for each program in the cluster)
- Domain models: {{OUTPUT_DIR}}/01_discovery/specs/domain-model/*.schema.yaml
  (relevant schemas)
- Dependency graph: {{OUTPUT_DIR}}/01_discovery/specs/dependency_graph.yaml

TASK: Read ALL programs in this domain cluster from the repository.
Analyze them TOGETHER to extract the complete business process, all 
business rules, and the workflow definition.

Programs to read:
{{PROGRAM_PATHS}}  (list of file paths in repo)

Copybooks to read:
{{COPYBOOK_PATHS}}  (all copybooks used by these programs)

===== PRODUCE THREE FILES =====

FILE 1: {{OUTPUT_DIR}}/02_business/prose/{{CLUSTER_NAME}}_business_process.md

## Business Process: [Business-Friendly Process Name]

### Business Objective
[2-3 sentences for an executive. What business goal? Why does it exist?]

### Business Context
[Where this fits in the larger business. What's upstream? Downstream? 
What breaks if this stops running?]

### Actors & Stakeholders
| Who | Role in Process | What They Care About |
|---|---|---|
| [actor] | [role] | [interest] |

### End-to-End Process Flow
[Tell the story as a narrative. Plain English. Numbered steps.]

1. [First business step — e.g., "Each business day, the system reviews 
   all active loan accounts due for interest calculation."]
2. [Next step]
3. [Continue for ALL steps across ALL programs in this cluster]

### Business Rules

| Rule ID | Plain English Description | Category | Source Program | Confidence | SME Question |
|---|---|---|---|---|---|
| BR-V-001 | [description a business person can validate] | Validation | [program] | HIGH | None |
| BR-C-001 | [description] | Calculation | [program] | HIGH | [question] |
| BR-T-001 | [description with specific values] | Threshold | [program] | MEDIUM | "Are the $100 threshold and $25 fee still current?" |

### Business Calculations
| Calc ID | What It Calculates | Formula | Worked Example | Convention |
|---|---|---|---|---|
| BR-C-001 | Daily interest | Balance × (Rate ÷ 360) | $50,000 × (5.5% ÷ 360) = $7.64 | 360-day year |

### Exception Handling (Business View)
| What Goes Wrong | Business Response | Impact |
|---|---|---|
| [scenario] | [what happens] | [business consequence] |

### Reports & Outputs
| Output | Audience | Contents | Frequency |
|---|---|---|---|
| [name] | [who] | [what] | [when] |

### Compliance & Regulatory Notes
[Flag anything regulatory — be specific about which rules]

### Open Questions for Business SME Review
[Specific questions — not vague. Each should be answerable by a 
knowledgeable business person.]

---

FILE 2: {{OUTPUT_DIR}}/02_business/specs/workflows/{{PROCESS_NAME}}.workflow.yaml

```yaml
workflow:
  id: "WF-{{PROCESS_ID}}"
  version: "1.0.0"
  name: "[Business process name]"
  description: "[Business description]"
  x-cobol-source:
    programs: ["list of all programs in this cluster"]
    jcl_jobs: ["associated JCL jobs"]
    source_paths:
      - program: "[name]"
        path: "[path in repo]"
  x-human-doc: "02_business/prose/{{CLUSTER_NAME}}_business_process.md"

  trigger:
    type: "[SCHEDULED | EVENT | USER_ACTION | UPSTREAM_PROCESS]"
    schedule: "[cron or description]"
    event: "[event name if triggered]"

  states:
    [STATE_NAME]:
      description: "[business meaning]"
      x-cobol-source:
        program: "[program name]"
        paragraph: "[section/paragraph]"
      on_entry:
        - action: "[action name]"
          rules: ["BR-xxx"]
      on_exit:
        - action: "[action name]"
      scheduled_actions:
        - action: "[action name]"
          schedule: "[daily | weekly | monthly | on-demand]"
          rules: ["BR-xxx"]
          x-cobol-source:
            program: "[program]"
            jcl_job: "[job name]"
      transitions:
        - event: "[EVENT_NAME]"
          target: "[TARGET_STATE]"
          guard:
            rules: ["BR-xxx"]
            condition: "[human-readable condition]"
          action: "[action during transition]"
          x-cobol-source:
            program: "[program]"
            paragraph: "[where transition logic is]"
```

---

FILE 3: {{OUTPUT_DIR}}/02_business/specs/rules/{{DOMAIN}}/{{CLUSTER_NAME}}.rules.yaml

```yaml
rule_set:
  id: "RS-{{CLUSTER_ID}}"
  version: "1.0.0"
  domain: "{{DOMAIN}}"
  description: "[What rules this set covers]"
  x-cobol-source:
    programs: ["list"]
  x-human-doc: "02_business/prose/{{CLUSTER_NAME}}_business_process.md"

rules:
  - id: "BR-[V|C|E|T|R|S]-[number]"
    name: "[Human-readable name]"
    category: "[VALIDATION | CALCULATION | ELIGIBILITY | THRESHOLD | STATE_TRANSITION | ROUTING]"
    priority: [1-10, 1=highest]
    description: "[Plain English — same as in human doc]"
    x-cobol-source:
      program: "[program name]"
      source_path: "[path in repo]"
      paragraph: "[paragraph/section]"
      line_range: "[approximate lines]"
    x-human-doc:
      file: "02_business/prose/{{CLUSTER_NAME}}_business_process.md"
      section: "Business Rules"
    condition:
      all:  # or any:
        - field: "[field from domain model schema]"
          operator: "[equals | not_equals | greater_than | less_than | in | not_in | is_null | is_blank | matches | not_matches | between]"
          value: "[value]"
    action:
      type: "[validate | calculate | reject | skip | apply_fee | route | transform]"
      # For reject/validate:
      error_code: "[ERR-xxx]"
      error_message: "[message]"
      http_status: [code]
      # For calculate:
      formula: "[mathematical formula]"
      target_field: "[output field]"
      precision:
        scale: [decimal places]
        rounding_mode: "[HALF_UP | HALF_DOWN | TRUNCATE | CEILING | FLOOR]"
        day_count_convention: "[30/360 | actual/365 | actual/actual]"
      # For apply_fee:
      fee_amount: "[amount]"
      fee_code: "[code]"
    test_cases:
      - description: "[happy path]"
        input: { field1: "value1", field2: "value2" }
        expected: { result_field: "expected_value" }
      - description: "[boundary case]"
        input: { field1: "boundary_value" }
        expected: { result_field: "expected_value" }
      - description: "[negative case]"
        input: { field1: "invalid_value" }
        expected: { valid: false, error_code: "ERR-xxx" }
    x-sme-review:
      needed: [true | false]
      question: "[specific question]"
      confidence: "[HIGH | MEDIUM | LOW]"
```
```

---

## Prompt 2.2 — Business Rules Aggregation

After ALL domain clusters have been processed, run this aggregation.

```
ROLE: You are a senior business analyst consolidating business rules 
across the entire system into a master catalog.

CONTEXT:
- System: {{SYSTEM_NAME}}

TASK: Read ALL rules YAML files from:
{{OUTPUT_DIR}}/02_business/specs/rules/

Consolidate into a single deduplicated, categorized catalog. Identify 
rules in multiple programs, flag contradictions, group related rules.

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/02_business/prose/MASTER_BUSINESS_RULES_CATALOG.md

## Master Business Rules Catalog — {{SYSTEM_NAME}}

### Summary
- Total Unique Rules: [N]
- By Category: Validation: [N], Calculation: [N], Eligibility: [N], 
  Threshold: [N], State Transition: [N], Routing: [N]
- Rules in Multiple Programs: [N]
- Potential Contradictions: [N]
- Rules Needing SME Validation: [N]

### Rules by Category

#### Validation Rules
| ID | Description | Programs | Priority | Confidence |
|---|---|---|---|---|
| BR-V-001 | [description] | [programs] | [priority] | [confidence] |

[Repeat for each category...]

### Cross-Program Duplicates
| Rule ID | Found In | Implementations Consistent? | Notes |
|---|---|---|---|
| [ID] | [programs] | [YES / NO / PARTIAL] | [differences] |

### Contradictions Found
| Rule A | Rule B | Programs | Conflict Description |
|---|---|---|---|
| [ID] | [ID] | [programs] | [nature of contradiction] |

### Rules Requiring SME Validation
| Rule ID | Question for SME | Why Uncertain |
|---|---|---|
| [ID] | [specific question] | [reason] |

---

FILE 2: {{OUTPUT_DIR}}/02_business/specs/rules/master_rules_catalog.yaml

```yaml
master_catalog:
  system: "{{SYSTEM_NAME}}"
  generated_date: "[date]"
  
  summary:
    total_unique_rules: [N]
    by_category:
      validation: [N]
      calculation: [N]
      eligibility: [N]
      threshold: [N]
      state_transition: [N]
      routing: [N]
    cross_program_rules: [N]
    contradictions: [N]
    needing_sme_review: [N]

  all_rules:
    - id: "BR-xxx"
      name: "[name]"
      category: "[category]"
      description: "[description]"
      found_in_programs: ["program1", "program2"]
      source_rule_files: ["path1.rules.yaml", "path2.rules.yaml"]
      consistent_across_programs: [true | false]
      inconsistency_notes: "[if inconsistent, describe difference]"
      sme_review_needed: [true | false]
      sme_question: "[question]"

  contradictions:
    - rule_a: "BR-xxx"
      rule_b: "BR-yyy"
      programs: ["prog1", "prog2"]
      description: "[conflict]"
      resolution: "[PENDING_SME | RESOLVED | ACCEPTED]"
```
```

---

## Quality Gate Checklist

Before proceeding to Pass 3, verify:

- [ ] Every domain cluster has a business process document
- [ ] Every domain has a workflow YAML
- [ ] Every domain has a rules YAML
- [ ] Business rule IDs are unique across the entire system (no duplicates)
- [ ] Every rule has at least 3 test cases (happy, boundary, negative)
- [ ] Master rules catalog is generated with cross-program analysis
- [ ] Contradictions are flagged for SME review
- [ ] All YAML files are syntactically valid
- [ ] Processing log is updated
- [ ] **SME REVIEW CHECKPOINT:** Business SMEs should validate 15-20% of business process documents before proceeding
