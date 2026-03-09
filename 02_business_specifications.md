# Pass 2 — Business Specifications

## Purpose

Extract business-level meaning from the code. The audience is business analysts, product owners, and stakeholders who do NOT read COBOL.

**CRITICAL WRITING PRINCIPLE:** The human-readable output from this pass must read as if it were written by a business analyst who interviewed subject matter experts — NOT by someone reading source code. If a business person can tell these specs were generated from COBOL, the output has failed.

## The Business Language Standard

### What Good Looks Like vs. What Bad Looks Like

**BAD (COBOL-oriented):**
> "In paragraph 3000-CALC-INTEREST, the program reads WS-CURR-BALANCE (PIC S9(7)V99 COMP-3) and WS-INT-RATE (PIC S9(3)V9(6)), then executes COMPUTE WS-DAILY-INT = WS-CURR-BALANCE * (WS-INT-RATE / 360) ROUNDED."

**GOOD (Business-oriented):**
> "Each business day, the system calculates interest for every active loan account. The daily interest charge equals the outstanding balance multiplied by the daily rate (the annual rate divided by 360 days — this uses the banking industry's '30/360' convention, not actual calendar days). The result is rounded to the nearest cent."

### Banned Terms in Human-Readable Output

These terms must NEVER appear in prose business documents:

| BANNED Term | Use Instead |
|-------------|------------|
| paragraph, section (COBOL) | "processing step" or "business rule" |
| WORKING-STORAGE, LINKAGE | "internal data" or "input parameters" |
| copybook, COPY member | "shared data definition" or "standard record format" |
| PIC, PICTURE clause | describe the business meaning (e.g., "dollar amount with two decimal places") |
| COMP-3, packed decimal, zoned | "number" or "amount" |
| FD, file description | "data source" or "input file" |
| PERFORM, GO TO | describe what happens, not how control flows |
| ABEND | "the system stops and flags an error" |
| return code, RETURN-CODE | "the process reports success, warning, or failure" |
| SQLCODE | "the database query succeeds or fails" |
| FILE STATUS | "the file operation succeeds or fails" |
| CICS, BMS, MAP | "the user screen" or "the interface" |
| EXEC SQL, EXEC CICS | describe the business action ("retrieves the account") |
| JCL, DD name, DSN | "scheduled job" or "input data" |
| EBCDIC, binary | never mention encoding in business docs |
| COMMAREA, TWA | "data passed between steps" |
| cursor, FETCH | "the system retrieves records" |

### Business Rule Writing Standard

**BAD:** "BR-V-001: IF WS-ACCT-STATUS NOT = 'A' AND NOT = 'D', PERFORM 9000-ERROR-RTN."

**GOOD:** "BR-V-001: Every account must have a valid status (Active or Dormant). If an account has any other status, the transaction is rejected with a notification to the operations team."

### Calculation Writing Standard

**BAD:** "COMPUTE WS-DAILY-INT = WS-CURR-BALANCE * (WS-INT-RATE / 360) ROUNDED"

**GOOD:**
> "Daily Interest = Outstanding Balance × (Annual Interest Rate ÷ 360)
> Example: $50,000 at 5.5% = $50,000 × 0.055 ÷ 360 = **$7.64 per day**
> Note: Uses the banking industry's 30/360 day-count convention."

### Process Flow Writing Standard

**BAD:** "1. Program opens input file LOAN-MASTER. 2. Program reads first record. 3. Program performs 2000-VALIDATE. 4. If FILE-STATUS = '00', performs 3000-CALCULATE."

**GOOD:** "1. The system retrieves all active loan accounts scheduled for interest processing today. 2. For each account, it verifies the account is eligible (must be Active with a positive balance). 3. The system calculates the day's interest charge. 4. If any account fails validation, it is set aside for manual review without affecting other accounts."

---

## Inputs Required

- COBOL program source + copybooks (from repo)
- Pass 1 outputs: inventory entries, domain model schemas, dependency graph, domain clusters

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Business Process | `02_business/prose/{domain}_business_process.md` | `02_business/specs/workflows/{process}.workflow.yaml` |
| Business Rules | (embedded in process doc) | `02_business/specs/rules/{domain}/{ruleset}.rules.yaml` |
| Rules Catalog | `02_business/prose/MASTER_BUSINESS_RULES_CATALOG.md` | `02_business/specs/rules/master_rules_catalog.yaml` |

## Processing Instructions

Process programs grouped by domain cluster:

```
1. For each domain cluster:
   a. READ all programs + copybooks
   b. LOAD Pass 1 inventory entries as context
   c. APPLY Prompt 2.1
   d. WRITE outputs

2. After all domains: APPLY Prompt 2.2 (aggregation)
3. UPDATE the processing log
```

---

## Prompt 2.1 — Business Process, Workflow & Rules

```
ROLE: You are a senior business analyst who just spent two weeks 
interviewing subject matter experts. You are writing up your findings.
You have NEVER seen the source code. You describe business processes 
the way a management consultant would — in terms of business outcomes, 
decisions, policies, and customer impact.

You also produce machine-readable workflow and rules YAML (these CAN 
reference COBOL source for traceability, but the human docs MUST NOT).

CRITICAL RULES FOR HUMAN-READABLE OUTPUT:
1. NEVER use any COBOL terminology (see Banned Terms list)
2. NEVER reference program names, paragraph names, or line numbers
3. NEVER describe processing in terms of file reads/writes
4. NEVER say "the program does X" — describe the business action
5. Write as if you learned this from interviewing business people
6. Every calculation must include a worked example with real numbers
7. Every rule must be understandable by a non-technical person
8. Describe error handling as what the USER experiences, not code behavior

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Domain Cluster: {{CLUSTER_NAME}}

PRIOR CONTEXT:
- Inventory: {{OUTPUT_DIR}}/01_discovery/specs/inventory/
- Domain models: {{OUTPUT_DIR}}/01_discovery/specs/domain-model/

Programs to read: {{PROGRAM_PATHS}}
Copybooks to read: {{COPYBOOK_PATHS}}

===== PRODUCE THREE FILES =====

FILE 1: {{OUTPUT_DIR}}/02_business/prose/{{CLUSTER_NAME}}_business_process.md

## [Business Process Name — NOT the COBOL program name]

### Why This Process Exists
[2-3 sentences for an executive. What business need? What if it stopped?]

### Where This Fits in the Business
[Value chain context. What's upstream? Downstream? Who's involved?]

### Who's Involved
| Role | What They Do | What They Need From This Process |
|---|---|---|
| [business role] | [responsibility] | [outcome they depend on] |

### How It Works

[Tell the story as a narrative. Every step describes a BUSINESS ACTION 
and its BUSINESS PURPOSE.]

**Step 1: [Business action name]**
[What happens, why, business outcome, decisions made at this step.]

**Step 2: [Next action]**
[Continue. Decision points described as business decisions, not IF 
statements: "If the account is dormant, interest accrual pauses 
because dormant accounts are not actively serviced."]

[Continue for all steps...]

### Business Policies & Rules

#### Eligibility Policies
| Policy ID | Policy | Rationale | Review Needed? |
|---|---|---|---|
| BR-E-001 | [business policy statement] | [why] | [Yes/No + question] |

#### Calculation Policies
| Policy ID | What's Calculated | Formula | Worked Example | Convention |
|---|---|---|---|---|
| BR-C-001 | [description] | [plain math] | [$50K at 5.5% = $7.64/day] | [30/360 etc.] |

#### Validation Policies
| Policy ID | What's Checked | If It Fails | Who's Notified |
|---|---|---|---|
| BR-V-001 | [plain English] | [business response] | [business role] |

#### Threshold Policies
| Policy ID | Policy | Current Value | Needs PM Review? |
|---|---|---|---|
| BR-T-001 | [policy] | [value] | [YES — may be outdated] |

### What Can Go Wrong
| Scenario | Business Impact | How It's Handled |
|---|---|---|
| [described from user perspective] | [business consequence] | [business response] |

### Reports & Outputs
| Output | Who Reads It | What It Tells Them | Still Needed? |
|---|---|---|---|
| [name] | [role] | [business value] | [PM should confirm] |

### Questions for Business Review
1. **[Specific question]** — [Full context and why it matters]
2. [Continue...]

---

FILE 2: {{OUTPUT_DIR}}/02_business/specs/workflows/{{PROCESS_NAME}}.workflow.yaml
[Machine-readable — CAN contain x-cobol-source references. Same YAML format as before.]

FILE 3: {{OUTPUT_DIR}}/02_business/specs/rules/{{DOMAIN}}/{{CLUSTER_NAME}}.rules.yaml
[Machine-readable — CAN contain x-cobol-source. Same YAML format with conditions, actions, test cases.]
```

---

## Prompt 2.2 — Business Rules Aggregation

```
ROLE: You are consolidating business policies across the organization.
Write the catalog as a POLICY REFERENCE DOCUMENT for business reviewers.
Use the same business language standard — no technical terms.

TASK: Read ALL rules YAML from {{OUTPUT_DIR}}/02_business/specs/rules/
Produce master catalog organized by BUSINESS CATEGORY, not by program.

FILE 1: {{OUTPUT_DIR}}/02_business/prose/MASTER_BUSINESS_RULES_CATALOG.md

## Business Policy Catalog — {{SYSTEM_NAME}}

### How to Read This Document
This lists every business policy enforced by the system. Organized by 
business function. Your job: confirm each is still correct.

[Organize by: Eligibility, Calculations, Validations, Thresholds, Workflow]
[Flag inconsistencies where the same rule is implemented differently]
[Include specific questions for business reviewers]

FILE 2: {{OUTPUT_DIR}}/02_business/specs/rules/master_rules_catalog.yaml
```

---

## Quality Gate Checklist

- [ ] **LANGUAGE CHECK:** Read every prose doc aloud. If ANY COBOL term appears, rewrite.
- [ ] Every calculation has a worked example with realistic dollar amounts
- [ ] Rules are written as business policies, not code translations
- [ ] Inconsistencies across the system are flagged
- [ ] Questions for business review are specific and answerable
- [ ] Machine-readable specs are valid YAML
- [ ] **SME REVIEW:** Business SMEs validate 15-20% of documents
