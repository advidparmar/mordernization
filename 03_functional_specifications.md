# Pass 3 — Functional Specifications

## Purpose

Create platform-independent functional specifications for Java developers. These must describe WHAT the system does algorithmically — without COBOL flavoring in the prose, and with pseudocode written in modern programming concepts.

**CRITICAL:** The functional spec has TWO audiences with different needs:
1. **Human-readable prose** — for dev leads reviewing scope and approach. Must use business language for the "what" and clean pseudocode for the "how."
2. **Machine-readable OpenAPI** — for code generators and contract testing. This CAN reference COBOL source in x- extension fields.

## The Functional Spec Language Standard

### Processing Logic: Modern Pseudocode, Not COBOL Translation

The biggest failure mode in functional specs is writing pseudocode that mirrors COBOL structure. COBOL processes sequentially with PERFORMs, GO TOs, and paragraph fall-throughs. Java developers think in objects, methods, streams, and exception handling.

**BAD (COBOL-flavored pseudocode):**
```
OPEN INPUT LOAN-FILE
READ LOAN-FILE INTO WS-LOAN-RECORD
PERFORM UNTIL EOF
  IF WS-ACCT-STATUS = 'A'
    PERFORM 3000-CALC-INTEREST
  END-IF
  IF WS-ERROR-FLAG = 'Y'
    PERFORM 9000-ERROR-RTN
  END-IF
  WRITE OUTPUT-RECORD FROM WS-LOAN-RECORD
  READ LOAN-FILE INTO WS-LOAN-RECORD
END-PERFORM
CLOSE LOAN-FILE
```

**GOOD (modern pseudocode):**
```
function calculateDailyInterest(processingDate):
    accounts = accountRepository.findAllActive()
    results = new ProcessingResults()
    
    for each account in accounts:
        try:
            if not account.isEligibleForInterest():
                results.skip(account, "Not eligible")
                continue
            
            dailyInterest = interestCalculator.calculate(
                balance = account.currentBalance,
                annualRate = account.interestRate,
                convention = DayCount.THIRTY_360
            )
            
            account.addAccruedInterest(dailyInterest)
            accountRepository.save(account)
            results.recordSuccess(account, dailyInterest)
            
        catch ValidationException as e:
            results.recordError(account, e.message)
    
    return results.toSummary()
```

### Key Differences in Pseudocode Style

| COBOL Pattern | Write It As |
|---|---|
| PERFORM paragraph | Call a method with a descriptive name |
| READ file / WRITE file | Repository.find() / Repository.save() |
| Working storage flags (WS-ERROR-FLAG) | Exceptions or result objects |
| EVALUATE / WHEN | switch/case or strategy pattern |
| Nested IF with GO TO | Early return or guard clauses |
| Sequential file processing | Stream/iterator with for-each |
| COPY copybook | Reference the domain model class |
| REDEFINES | Polymorphism or type discriminator |
| 88-level conditions | Enum values or boolean methods |
| RETURN-CODE 0/4/8 | Return result object with status enum |
| ON SIZE ERROR | Try/catch ArithmeticException or validation |

### Banned Terms in Prose (Same as Pass 2)

All COBOL terms from the Pass 2 banned list apply here too. Additionally:
- NEVER describe processing as "the program reads record X then writes record Y" — describe the business operation
- NEVER use COBOL paragraph names as section headers — use business action names
- Field names in the spec should be the DOMAIN MODEL names (camelCase Java style) from Pass 1 schemas, not COBOL field names

---

## Inputs Required

- COBOL source + copybooks (from repo)
- Pass 1: inventory, domain models
- Pass 2: business processes, rules
- Pass 2B: PM-approved stories (**only build specs for KEEP/MODIFY/ENHANCE stories**)

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Functional Spec | `03_functional/prose/{program}_functional_spec.md` | `03_functional/specs/api-specs/{service}.openapi.yaml` |
| UI Mapping | `03_functional/prose/{transaction}_ui_mapping.md` | (embedded in OpenAPI) |
| Report Spec | `03_functional/prose/{report}_report_spec.md` | — |
| Interface Spec | `03_functional/prose/{interface}_interface_spec.md` | `03_functional/specs/interfaces/{name}.interface.yaml` |

## Processing Instructions

```
1. CHECK Pass 2B PM decisions: only process stories marked KEEP, MODIFY, or ENHANCE
2. For each approved story/module:
   a. READ program source + copybooks
   b. LOAD all prior outputs as context
   c. APPLY Prompt 3.1
   d. WRITE outputs
3. UPDATE processing log
```

---

## Prompt 3.1 — Functional Specification & API Contract

```
ROLE: You are a senior systems analyst writing specifications for a 
Java team. You think in terms of services, APIs, domain objects, and 
modern architecture patterns. When you describe processing logic, you 
write pseudocode that a Java developer would naturally implement — 
using methods, objects, repositories, exception handling, and streams.

You translate COBOL's procedural, sequential, file-oriented processing 
into equivalent logic expressed through modern patterns. You NEVER 
write pseudocode that looks like COBOL with different syntax.

CRITICAL RULES:
1. Processing logic uses MODERN pseudocode (methods, objects, try/catch,
   streams) — NOT COBOL-flavored (PERFORM, READ, WRITE, paragraph names)
2. Field names use domain model names from Pass 1 schemas (camelCase),
   NOT COBOL field names (WS-ACCT-NUM)
3. Prose describes business operations, not file I/O operations
4. Error handling uses exceptions and result objects, not return codes
5. Data access uses repository/service patterns, not file open/read/close
6. Conditional logic uses guard clauses and early returns, not nested IFs
7. Collections use streams and filters, not sequential READ loops
8. The spec must be implementable by a Java developer who has never 
   seen COBOL and doesn't know what a copybook is

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

PRIOR CONTEXT:
- Inventory: {{OUTPUT_DIR}}/01_discovery/specs/inventory/{{PROGRAM_NAME}}.inventory.yaml
- Domain Model: {{OUTPUT_DIR}}/01_discovery/specs/domain-model/*.schema.yaml
- Business Process: {{OUTPUT_DIR}}/02_business/prose/{{CLUSTER}}_business_process.md
- Business Rules: {{OUTPUT_DIR}}/02_business/specs/rules/{{DOMAIN}}/{{CLUSTER}}.rules.yaml
- PM-Approved Stories: {{OUTPUT_DIR}}/02b_product_stories/specs/{{DOMAIN}}_backlog.yaml

TASK: Read the program from the repository. Produce a functional 
specification that a Java developer can implement from.

Program: {{REPO_ROOT}}/{{PROGRAM_PATH}}
Copybooks: {{COPYBOOK_PATHS}}

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/03_functional/prose/{{PROGRAM_NAME}}_functional_spec.md

## Functional Specification: [Business Function Name — NOT COBOL program name]
**Spec ID:** FS-{{SPEC_ID}}  |  **Stories:** [PM-approved story IDs]  |  **Status:** Draft

### 1. What This Service Does
[2-3 sentences describing the business capability. Written for a Java 
developer who needs to understand the business context, not the COBOL 
implementation.]

### 2. When It Runs
[Trigger: user action, API call, scheduled job, upstream event. 
Describe in business terms.]

### 3. What It Receives (Inputs)

**[Input Name]** — from [business source, not "file" or "CICS screen"]

| Field | Business Meaning | Type | Required | Constraints | Example |
|---|---|---|---|---|---|
| accountId | Unique loan account identifier | String, 10 chars | Yes | Alphanumeric only | "LN00045821" |
| currentBalance | Outstanding loan amount | Decimal, 2 places | Yes | Must be >= 0 | 50000.00 |
| interestRate | Annual interest rate | Decimal, 6 places | Yes | 0-100% range | 0.055000 |

[Use domain model field names from Pass 1 schemas. Include realistic 
example values. Describe constraints in business terms.]

### 4. What It Produces (Outputs)

**[Output Name]** — delivered to [business destination]

| Field | Business Meaning | Type | How It's Determined | Example |
|---|---|---|---|---|
| dailyInterest | Interest charged for today | Decimal, 2 places | Balance × (Rate ÷ 360) | 7.64 |
| newAccruedTotal | Running interest total | Decimal, 2 places | Previous total + today's interest | 107.64 |

### 5. How It Works (Processing Logic)

[Write in modern pseudocode. Organize by BUSINESS OPERATION, not by 
COBOL paragraph. Use domain model field names.]

**5.1 Validate the Request**

```pseudocode
function validateAccount(account: LoanAccount): ValidationResult
    // Policy BR-V-001: Account must exist and be identifiable
    if account is null:
        return ValidationResult.error("ACCOUNT_NOT_FOUND", 
            "No account found with the provided identifier")
    
    // Policy BR-V-003: Account must be in processable status
    if account.status not in [ACTIVE, DORMANT]:
        return ValidationResult.error("INVALID_STATUS",
            "Account status {account.status} is not eligible for this operation")
    
    return ValidationResult.success()
```

**5.2 Calculate Daily Interest**

```pseudocode
function calculateDailyInterest(account: LoanAccount): Money
    // Policy BR-C-001: Daily interest using 30/360 convention
    // Formula: Balance × (Annual Rate ÷ 360)
    // IMPORTANT: Must use exact decimal arithmetic (BigDecimal), 
    //   never floating-point. Round to 2 decimal places using HALF_UP.
    //   The 360-day divisor is an industry convention, not a bug.
    
    dailyRate = account.interestRate / 360    // precision: 10 decimal places
    dailyInterest = account.currentBalance × dailyRate
    
    return dailyInterest.roundTo(2, HALF_UP)
    
    // Example walkthrough:
    // Balance: $50,000.00, Rate: 5.5% (0.055000)
    // Daily rate: 0.055000 / 360 = 0.0001527778
    // Daily interest: 50000.00 × 0.0001527778 = 7.638889
    // Rounded: $7.64
```

**5.3 Apply Interest and Update Account**

```pseudocode
function applyInterest(account: LoanAccount, interest: Money): InterestResult
    // Policy BR-E-001: Only active accounts accrue interest
    if account.status != ACTIVE:
        return InterestResult.skipped(account, "Not active")
    
    // Policy BR-C-001: Accrued interest is cumulative
    account.accruedInterest += interest
    account.lastInterestDate = today()
    
    accountRepository.save(account)
    
    return InterestResult.success(account, interest)
```

**5.4 Handle Errors and Generate Summary**

```pseudocode
function processAllAccounts(): ProcessingSummary
    accounts = accountRepository.findAllActiveAccounts()
    summary = new ProcessingSummary()
    
    for each account in accounts:
        try:
            validation = validateAccount(account)
            if validation.failed():
                summary.addSkipped(account, validation.reason)
                continue
            
            interest = calculateDailyInterest(account)
            result = applyInterest(account, interest)
            summary.addSuccess(result)
            
        catch (exception):
            // Individual account failures don't stop processing
            summary.addError(account, exception.message)
            log.error("Failed to process account {}", account.id, exception)
    
    // Policy: Process completes even if some accounts fail
    // Warning if error rate > 5%, failure if > 20%
    summary.determineOverallStatus()
    return summary
```

### 6. Validation Rules

| Policy | What's Checked | If It Fails | HTTP Status | Error Code |
|---|---|---|---|---|
| BR-V-001 | Account exists | "Account not found" response | 404 | ACCOUNT_NOT_FOUND |
| BR-V-003 | Account status is processable | "Invalid status" response | 422 | INVALID_STATUS |

### 7. Business Calculations

| Policy | Formula | Example | Precision | Rounding | Day Count |
|---|---|---|---|---|---|
| BR-C-001 | Balance × (Rate ÷ 360) | $50K × (5.5% ÷ 360) = $7.64 | 2 decimal places | HALF_UP | 30/360 |

⚠️ **Precision is critical.** These calculations involve money. Using 
floating-point (float/double) WILL produce different results than the 
legacy system. All monetary arithmetic MUST use exact decimal types 
(BigDecimal in Java, Decimal in databases).

### 8. How Errors Are Handled

| What Goes Wrong | What the User Sees | What Happens Internally | Severity |
|---|---|---|---|
| Account not found | "Account not found" message | Return 404 response | Normal — expected for invalid IDs |
| Database unavailable | "Service temporarily unavailable" | Retry 3 times, then return 503 | Critical — alert operations |
| Calculation overflow | N/A (batch) — account flagged | Skip account, add to error list | Warning — manual review needed |
| No eligible accounts | Summary shows zero processed | Complete normally, no alert | Normal — expected on holidays |

### 9. Connections to Other Services

| Service | Why | Direction | What's Exchanged | If It's Down |
|---|---|---|---|---|
| [e.g., Account Service] | [e.g., Look up account details] | Inbound | [Account data] | [This service can't function] |
| [e.g., Notification Service] | [e.g., Send error alerts] | Outbound | [Alert payload] | [Processing continues, alerts queued] |

### 10. Performance Expectations
- **Volume:** [N] accounts processed per run
- **Time:** Must complete within [X] hours
- **Concurrency:** [Can multiple instances run? Locking?]
- **Safe to re-run:** [Yes/No — and why]

### 11. Traceability
| Business Policy | Spec Section | PM Story |
|---|---|---|
| BR-C-001 | §5.2, §7 | [DOMAIN]-001 |
| BR-V-001 | §5.1, §6 | [DOMAIN]-001 |

### 12. Open Items
| Item | Type | Description | What Breaks If Wrong |
|---|---|---|---|
| [item] | [ASSUMPTION / QUESTION / RISK] | [detail] | [consequence] |

---

FILE 2: {{OUTPUT_DIR}}/03_functional/specs/api-specs/{{SERVICE_NAME}}.openapi.yaml

[OpenAPI 3.1 spec — CAN contain x-cobol-source fields for traceability.
Same format as previously defined. Reference domain model schemas from
Pass 1 via $ref. Map COBOL error patterns to HTTP status codes.]
```

---

## Prompt 3.2 — Screen/UI Mapping (CICS Only)

```
ROLE: You are a UX analyst describing screens from the USER'S perspective.
Describe what users SEE and DO — not BMS map field definitions.

BAD: "Field ACCTNO at row 5 col 10, PIC X(10), UNPROT BRT, MDT"
GOOD: "Account Number — a text field where the user types the 10-character 
account ID. The cursor starts here when the screen loads."

Document every screen with: what the user sees, what they can type, 
what each button/key does, where each action takes them, and what 
error messages they might encounter.

Include a "Modern UI Recommendation" — how this should look as a web page.
```

---

## Prompt 3.3 — Report Specification

```
ROLE: You are a reporting analyst describing reports from the READER'S 
perspective. Describe what the report TELLS the reader and what 
DECISIONS it supports — not the print layout mechanics.

BAD: "WRITE REPORT-LINE FROM DETAIL-LINE AFTER ADVANCING 1 LINE"
GOOD: "Each row shows one loan account with its current balance, rate, 
and accrued interest. Accounts are grouped by branch, with subtotals 
showing each branch's total portfolio value."

Include recommendation: dashboard, self-service query, scheduled email, 
API, PDF, or eliminate entirely.
```

---

## Prompt 3.4 — Interface Specification

```
ROLE: You are an integration architect describing data exchanges in 
business terms.

BAD: "Fixed-width file, LRECL=200, FB, with 15 fields starting at position 1"
GOOD: "Every evening, the partner bank sends us a file containing the day's 
payment transactions. Each record represents one payment, including the 
account number, payment amount, payment date, and payment method."

Include: what business process drives this exchange, what happens if 
it doesn't arrive, and what the modern equivalent should be (API, event 
stream, etc.)
```

---

## Quality Gate Checklist

- [ ] **LANGUAGE CHECK:** No COBOL terms in prose. No COBOL-flavored pseudocode.
- [ ] **PSEUDOCODE CHECK:** All pseudocode uses modern patterns (methods, objects, exceptions, repositories)
- [ ] **FIELD NAME CHECK:** All field names are domain model names (camelCase), not COBOL names (WS-XXX)
- [ ] Only PM-approved stories (KEEP/MODIFY/ENHANCE) have functional specs
- [ ] Eliminated stories are not in scope
- [ ] Modified stories incorporate PM notes
- [ ] OpenAPI specs are syntactically valid
- [ ] All $ref references resolve
- [ ] **TECH REVIEW:** COBOL developer validates 10-15% against source
