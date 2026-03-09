# Pass 2B — Product Stories & Acceptance Criteria

## Purpose

Generate product-manager-readable user stories with Given-When-Then acceptance criteria from the COBOL codebase. These are the product manager's primary artifact for deciding what to keep, change, or eliminate.

**CRITICAL:** These stories must read as if a product manager wrote them after talking to users — NOT as if they were extracted from code. A PM should be able to hand these to a stakeholder and say "here's what our system does" without anyone suspecting they came from COBOL analysis.

## The Two Kinds of Given-When-Then in This Library

| | Pass 2B (This File) | Pass 4 Technical Tests |
|---|---|---|
| **Audience** | Product managers, business stakeholders | Java developers, QA engineers |
| **Language** | Business English, user perspective | Technical, precise computed values |
| **Granularity** | One story per user-visible capability | One scenario per business rule |
| **Purpose** | Decide what to build | Verify code correctness |
| **Amendable** | YES — PMs mark up with changes | No — must match COBOL exactly |

### What Good Stories Look Like vs. Bad

**BAD (code-derived):**
> "As a user, I want the system to execute paragraph 3000-CALC-INTEREST 
> which reads WORKING-STORAGE fields WS-BALANCE and WS-RATE and computes 
> daily interest using COMP-3 packed decimal arithmetic."
>
> Given WS-ACCT-STATUS = 'A'
> When PERFORM 3000-CALC-INTEREST is executed
> Then WS-DAILY-INT = WS-CURR-BALANCE * (WS-INT-RATE / 360) ROUNDED

**GOOD (user-perspective):**
> "As a loan servicing manager, I want interest to accrue automatically 
> on all active loans each business day, so that our accounting records 
> stay current and customers are billed accurately."
>
> Given a customer has an active loan with a $50,000 balance at 5.5% annual interest
> When the daily interest process runs
> Then $7.64 should be added to their accrued interest balance
> And the customer's statement should reflect the updated interest total

**BAD acceptance criteria:**
> Given FILE-STATUS = '00' after reading LOAN-MASTER
> When SQLCODE = 0 on SELECT from ACCT_TABLE
> Then RETURN-CODE should be set to 0

**GOOD acceptance criteria:**
> Given a customer's loan account is in good standing
> When I look up their account details
> Then I should see their current balance, interest rate, and payment history
> And the information should be current as of today

### Banned Terms (Same as Pass 2 + Additional)

All terms banned in Pass 2 are banned here, PLUS:
- NEVER reference program names, file names, or table names
- NEVER use database/file terminology (SELECT, INSERT, READ, WRITE)
- NEVER reference return codes, status codes, or ABEND codes
- NEVER describe internal processing steps — only user-visible outcomes
- NEVER say "the system performs" or "the process executes" — say what the USER sees or what BUSINESS OUTCOME occurs
- Acceptance criteria must describe what someone can OBSERVE, not what code does internally

---

## Inputs Required

- Business process documents (Pass 2)
- Business rules (Pass 2)
- Inventory (Pass 1)

## Outputs Produced

| Output | Location |
|--------|----------|
| Product Stories | `02b_product_stories/prose/{domain}_user_stories.md` |
| Machine-readable backlog | `02b_product_stories/specs/{domain}_backlog.yaml` |
| PM Review Worksheet | `02b_product_stories/prose/{domain}_pm_review_worksheet.md` |
| Story Map | `02b_product_stories/prose/STORY_MAP.md` |

## Processing Instructions

Run AFTER Pass 2, BEFORE Pass 3. Product stories gate all downstream work.

```
1. For each domain:
   a. READ business process document and rules
   b. READ COBOL source for context (but don't let it leak into output)
   c. APPLY Prompt 2B.1
   d. WRITE outputs
2. After all domains: APPLY Prompt 2B.2 (Story Map)
3. Generate PM Review Worksheets
4. **STOP AND WAIT FOR PM REVIEW**
```

---

## Prompt 2B.1 — User Story Generation

```
ROLE: You are a product manager who has been observing how users 
interact with the {{SYSTEM_NAME}} system for the past month. You've 
watched loan officers use their screens, you've seen the reports that 
managers receive, you've understood the nightly processes that keep 
the business running. Now you're writing user stories to capture what 
the system does — from the USER'S perspective, in the USER'S language.

You have NEVER seen source code. You don't know what COBOL is. You 
describe capabilities in terms of what USERS DO, what they SEE, and 
what BUSINESS OUTCOMES occur.

CRITICAL RULES:
1. Every story must be from a USER'S perspective — what they experience
2. Acceptance criteria describe OBSERVABLE OUTCOMES, not internal processing
3. Use real-world language: "customer," "balance," "payment" — not 
   field names or technical terms
4. Include realistic dollar amounts, dates, and scenarios in examples
5. Flag capabilities that look outdated or unnecessary (helps PM scope)
6. For batch/automated processes, the "user" is the business itself 
   or the team that monitors the results
7. NEVER reference any COBOL construct — if a PM can tell this came 
   from code analysis, you've failed

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

PRIOR CONTEXT:
- Business Process: {{OUTPUT_DIR}}/02_business/prose/{{CLUSTER}}_business_process.md
- Business Rules: {{OUTPUT_DIR}}/02_business/specs/rules/{{DOMAIN}}/{{CLUSTER}}.rules.yaml

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/02b_product_stories/prose/{{DOMAIN}}_user_stories.md

## Product Stories: [Business Domain Name]

### About This Document

This describes what {{SYSTEM_NAME}} does today, expressed as user 
stories. Each story represents a capability that EXISTS today.

**Product Manager: Please mark each story:**
- ✅ **KEEP AS-IS** — Build this exactly as described
- ✏️ **MODIFY** — Build this but change the behavior (add your notes)
- ❌ **ELIMINATE** — We no longer need this
- ➕ **ENHANCE** — Keep it and add new capabilities

---

### People Who Use This System

| Person | What They Do | How They Interact |
|---|---|---|
| [e.g., "Loan Officer"] | [daily responsibility] | [what they see and do] |
| [e.g., "Branch Manager"] | [responsibility] | [reports they review] |
| [e.g., "Customer"] | [relationship] | [what they experience] |

---

### Epic: [Business Capability — e.g., "Loan Interest Management"]

**Why this matters to the business:** [1-2 sentences on business value]

---

#### Story [ID]-001: [Title in user language]

**As a** [person/role],
**I want** [what they want to do or have happen],
**So that** [business benefit or outcome].

**How it works today:**
[1-2 paragraphs in plain English describing the current behavior from 
the user's perspective. What do they see? What happens automatically? 
What do they need to check or verify?]

**Acceptance Criteria:**

```
Given [a real-world business situation described naturally]
  And [additional context if needed]
When [something happens — a user action, a time trigger, a business event]
Then [what the user should see or what business outcome should occur]
  And [additional outcomes]
```

```
Given [an edge case or unusual situation]
When [the same action occurs]
Then [how the system should handle it — from user's perspective]
```

```
Given [something goes wrong — bad data, missing information, etc.]
When [the user tries to proceed]
Then [what the user should see — an error message, a warning, a fallback]
  And [what happens to their work — is it saved? lost? queued?]
```

**Things the product team should know:**
[Important context, like "This uses a 360-day year convention which 
means customers pay slightly more than under a 365-day calculation. 
Is this still the right approach?" or "This report is generated for 
15 departments but we should check if all 15 still need it."]

**PM Decision:** [ ] KEEP AS-IS  [ ] MODIFY  [ ] ELIMINATE  [ ] ENHANCE
**PM Notes:** _________________________________

---

#### Story [ID]-002: [Next capability]

[Continue for all user-visible capabilities...]

---

### Automated Process Stories

[For batch/nightly processes, the "user" is the business or ops team]

#### Story [ID]-AUTO-001: [Process title in business terms]

**As a** [business function or ops team],
**I need** [what the automated process does],
**So that** [business outcome — not "so that the file is processed"].

**How it works today:**
[Describe as a business person would: "Every night, the system 
automatically calculates the day's interest for all active loans. 
By 6am, the finance team can see the updated totals in their morning 
report. If any accounts couldn't be processed, they appear on an 
exception list that the operations team reviews first thing."]

**Acceptance Criteria:**

```
Given it is the end of a business day
  And there are 50,000 active loan accounts
When the daily interest process completes
Then every active account should have today's interest added
  And a summary should be available showing total interest charged
  And any accounts that couldn't be processed should be listed separately
```

```
Given no accounts are eligible for processing (e.g., a holiday)
When the process runs
Then it should complete without errors
  And the summary should clearly show zero accounts were processed
  And this should NOT trigger an alert (it's expected on holidays)
```

**PM Decision:** [ ] KEEP AS-IS  [ ] MODIFY  [ ] ELIMINATE  [ ] ENHANCE

---

### Report Stories

#### Story [ID]-RPT-001: [Report name in business terms]

**As a** [who reads this],
**I want to** receive [what report] [how often],
**So that** I can [what business decision or action it supports].

**What's in it today:**
[Describe contents in business terms: "Shows each department's total 
loan portfolio, broken down by product type, with month-over-month 
comparison. The bottom line shows the total outstanding balance and 
weighted average interest rate."]

```
Given it is the first business day of the month
When the monthly portfolio report is generated
Then it should show each department's total loans by product type
  And include a comparison to the previous month
  And be available by 8am for the management meeting
```

**Modern alternative suggestion:** [e.g., "This could become a 
self-service dashboard where managers can filter by department and 
date range instead of waiting for a monthly report."]

**PM Decision:** [ ] KEEP AS-IS  [ ] MODIFY  [ ] ELIMINATE  [ ] ENHANCE

---

### Summary
| Category | Stories |
|----------|---------|
| User-facing capabilities | [N] |
| Automated processes | [N] |
| Reports | [N] |
| **Total** | **[N]** |

---

FILE 2: {{OUTPUT_DIR}}/02b_product_stories/specs/{{DOMAIN}}_backlog.yaml

```yaml
backlog:
  domain: "{{DOMAIN}}"
  system: "{{SYSTEM_NAME}}"

  personas:
    - id: "PERSONA-001"
      name: "[name]"
      description: "[who they are]"

  epics:
    - id: "EPIC-{{DOMAIN}}-001"
      title: "[Business Capability]"
      business_value: "[why it matters]"
      stories:
        - id: "{{DOMAIN}}-001"
          title: "[story title]"
          persona: "PERSONA-001"
          as_a: "[role]"
          i_want: "[capability]"
          so_that: "[value]"
          type: "[FUNCTIONAL | AUTOMATED | REPORT]"
          acceptance_criteria:
            - given: "[situation]"
              when: "[trigger]"
              then: "[outcome]"
          business_rules: ["BR-xxx"]
          x-cobol-source:
            programs: ["list — for traceability only, never shown to PM"]
          pm_decision: null
          pm_notes: null
```
```

---

## Prompt 2B.2 — Story Map

```
ROLE: Product manager creating a story map showing end-to-end user 
journeys. Organize stories by the journey a user takes through the 
system, not by technical module.

Read all backlogs from: {{OUTPUT_DIR}}/02b_product_stories/specs/

Produce: {{OUTPUT_DIR}}/02b_product_stories/prose/STORY_MAP.md

Include:
- User journeys (horizontal flow of activities)
- Priority matrix (MUST HAVE / SHOULD HAVE / COULD HAVE / ELIMINATE)
- Modernization opportunities (where modern tech enables better UX)
- Cross-domain dependencies

Also produce PM Review Worksheets:
{{OUTPUT_DIR}}/02b_product_stories/prose/{{DOMAIN}}_pm_review_worksheet.md

[Concise table: Story ID | Title | As a... I want... | Decision column]
[Key questions for the PM review meeting]
[Impact analysis: "If you eliminate X stories, scope reduces by Y%"]
```

---

## How Stories Gate Downstream Passes

```
Pass 2B Output → PM Reviews → PM Decisions Flow Downstream:

KEEP AS-IS   → Pass 3 generates functional spec as-is
MODIFY       → Pass 3 incorporates PM's change notes
ELIMINATE    → Pass 3 SKIPS this capability entirely
ENHANCE      → Pass 3 adds new requirements alongside existing
```

---

## Quality Gate Checklist

- [ ] **LANGUAGE CHECK:** Every story reads as if written by a PM, not generated from code
- [ ] **USER PERSPECTIVE CHECK:** Every acceptance criterion describes something observable
- [ ] No COBOL terms anywhere in any prose document
- [ ] Every story has at least 2-3 Given-When-Then criteria (happy path + edge case + error)
- [ ] Report stories include modern alternative suggestions
- [ ] Story map shows complete user journeys
- [ ] PM review worksheets generated
- [ ] **PM REVIEW GATE:** PMs MUST review before Pass 3 begins
