# Pass 2B — Product Stories & Acceptance Criteria

## Purpose

Generate product-manager-readable user stories with Given-When-Then acceptance criteria from the COBOL codebase. These are NOT the technical Gherkin tests from Pass 4 — those are for developers and Cucumber. These are written for product managers, business stakeholders, and scrum teams to:

1. **Understand** what the existing system does in business terms
2. **Review and amend** — mark stories as "keep as-is," "modify," or "eliminate"
3. **Prioritize** — decide what goes into which sprint/release
4. **Accept** — serve as the definition of done for the Java implementation
5. **Identify gaps** — spot missing capabilities the new system should add

## The Two Kinds of Gherkin in This Library

| | Pass 2B Stories (This File) | Pass 4 Technical Tests |
|---|---|---|
| **Audience** | Product managers, business stakeholders | Java developers, QA engineers |
| **Language** | Business English, no technical terms | Technical, precise values |
| **Purpose** | Describe WHAT the system should do | Verify HOW the code behaves |
| **Granularity** | One story per user-visible capability | One scenario per business rule |
| **Amendable** | Yes — PMs mark up with changes | No — must match COBOL exactly |
| **Example** | "When a customer makes a payment, their balance decreases" | "Given balance 50000.00 and payment 1000.00, then new balance is 49000.00 with precision HALF_UP" |
| **Tool** | JIRA, Azure DevOps, product backlog | Cucumber test runner |

## Inputs Required

- Business process documents (Pass 2)
- Business rules (Pass 2)
- UI mapping (Pass 3, if available)
- Report specifications (Pass 3, if available)
- Inventory (Pass 1) — for understanding what the system does

## Outputs Produced

| Output | Location |
|--------|----------|
| Product Stories (per domain) | `02b_product_stories/prose/{domain}_user_stories.md` |
| Machine-readable backlog | `02b_product_stories/specs/{domain}_backlog.yaml` |
| PM Review Worksheet | `02b_product_stories/prose/{domain}_pm_review_worksheet.md` |
| Story Map | `02b_product_stories/prose/STORY_MAP.md` |

## Processing Instructions

Run this AFTER Pass 2 (business specifications) and BEFORE Pass 3. The product stories inform the functional specs — if a PM eliminates a story, the functional spec for that capability can be skipped.

```
1. For each domain cluster (from Pass 2):
   a. READ the business process document
   b. READ the business rules
   c. READ the COBOL source for context
   d. APPLY Prompt 2B.1 (User Story Generation)
   e. WRITE outputs

2. After all domains:
   a. APPLY Prompt 2B.2 (Story Map aggregation)
   b. APPLY Prompt 2B.3 (PM Review Worksheet generation)
```

---

## Prompt 2B.1 — User Story Generation

```
ROLE: You are a product manager reverse-engineering a legacy COBOL system 
into a modern product backlog. You write user stories that a non-technical 
product owner can read, understand, and make decisions about.

Your stories must be:
- Written in PLAIN BUSINESS ENGLISH — no COBOL terms, no Java terms, 
  no technical jargon
- Organized by USER PERSONA and BUSINESS CAPABILITY — not by program
- Actionable — a PM reading this should be able to say "keep this," 
  "change this," or "we don't need this anymore"
- Each story must have Given-When-Then acceptance criteria that describe 
  OBSERVABLE BUSINESS BEHAVIOR — what a user sees or what business 
  outcome occurs

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Domain Cluster: {{CLUSTER_NAME}}

PRIOR CONTEXT (read these files):
- Business Process: {{OUTPUT_DIR}}/02_business/prose/{{CLUSTER}}_business_process.md
- Business Rules: {{OUTPUT_DIR}}/02_business/specs/rules/{{DOMAIN}}/{{CLUSTER}}.rules.yaml
- Inventory: relevant entries from {{OUTPUT_DIR}}/01_discovery/specs/inventory/

COBOL Programs (read from repo for context):
{{PROGRAM_PATHS}}

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/02b_product_stories/prose/{{DOMAIN}}_user_stories.md

## Product Stories: [Business Domain Name]

### About This Document

This document describes what the current {{SYSTEM_NAME}} system does, 
expressed as user stories that a product team can review and amend. 
Each story represents a capability that EXISTS in the current system.

**Your job as Product Manager is to mark each story as:**
- ✅ **KEEP AS-IS** — Carry this capability forward unchanged
- ✏️ **MODIFY** — Carry forward but change the behavior (add your notes)
- ❌ **ELIMINATE** — We no longer need this capability
- ➕ **ENHANCE** — Keep it but add new capabilities on top

Stories marked ELIMINATE will be excluded from development. Stories 
marked MODIFY will be updated in the functional specs before development.

---

### Personas

[Identify the user personas from the COBOL system. These come from 
analyzing who uses CICS transactions, who receives reports, who 
triggers batch jobs.]

| Persona | Description | How They Interact |
|---|---|---|
| [e.g., "Loan Officer"] | [who they are] | [screens they use, reports they read] |
| [e.g., "Operations Manager"] | [who they are] | [reports, batch monitoring] |
| [e.g., "System (Automated)"] | [scheduled processes] | [batch jobs, automated calculations] |

---

### Epic: [Business Capability Name]

**Business Value:** [Why this capability exists — what business problem 
it solves]

**Current COBOL Implementation:** [Brief note for context — which 
programs, but written for a PM: "This is currently handled by the 
nightly interest calculation batch job"]

---

#### Story [DOMAIN]-001: [Story Title]

**As a** [persona],
**I want to** [action/capability],
**So that** [business value/outcome].

**Acceptance Criteria:**

```gherkin
Given [business context in plain English]
When [user action or business event]
Then [observable business outcome]
```

```gherkin
Given [another scenario - perhaps an edge case]
When [action]
Then [different outcome]
```

```gherkin
Given [error/exception scenario]
When [something goes wrong]
Then [how the system should respond from user's perspective]
```

**Business Rules Applied:** BR-V-001, BR-C-001 [references for traceability]

**Current Behavior Notes:**
[Any nuance a PM should know — e.g., "The current system uses a 360-day 
year for interest calculation, which is a banking industry convention. 
If we move to a 365-day year, calculated interest amounts will change 
slightly. This decision needs product owner input."]

**PM Decision:** [ ] KEEP AS-IS  [ ] MODIFY  [ ] ELIMINATE  [ ] ENHANCE

**PM Notes:** _________________________________

---

#### Story [DOMAIN]-002: [Next Story Title]

[Continue for all capabilities discovered in this domain...]

---

### Epic: [Next Business Capability]

[Continue for all epics/capabilities...]

---

### Non-Functional Stories

These aren't user-visible features but represent system behaviors the 
product team should be aware of:

#### Story [DOMAIN]-NF-001: [Title]

**As a** [persona — often "operations team" or "the business"],
**I want** [non-functional capability],
**So that** [business reason].

```gherkin
Given the system processes [volume] transactions per day
When the nightly batch runs
Then all accounts should be processed within [time window]
And a summary report should be available by [time]
```

**PM Decision:** [ ] KEEP AS-IS  [ ] MODIFY  [ ] ELIMINATE  [ ] ENHANCE

---

### Stories for Reports

Each legacy report becomes a story. PMs often discover that many 
reports are no longer needed.

#### Story [DOMAIN]-RPT-001: [Report Name]

**As a** [who reads this report],
**I want to** receive the [report name] [frequency],
**So that** I can [business purpose].

**What's in the report:**
[Plain English description of contents — not field layouts]

**Current Delivery:** [Printed / Emailed / On mainframe spool]

```gherkin
Given it is the end of [business day / month / quarter]
When the [report name] is generated
Then it should include [key content]
And it should be sorted by [sort order]
And it should show totals for [what's totaled]
```

**PM Decision:** [ ] KEEP AS-IS  [ ] MODIFY  [ ] ELIMINATE  [ ] ENHANCE
**Suggested Modern Alternative:** [Dashboard / API / Scheduled email / Self-service query]

**PM Notes:** _________________________________

---

### Summary Statistics

| Category | Count |
|----------|-------|
| Total Stories | [N] |
| Functional Stories | [N] |
| Non-Functional Stories | [N] |
| Report Stories | [N] |
| Business Rules Covered | [N] of [total] |

---

FILE 2: {{OUTPUT_DIR}}/02b_product_stories/specs/{{DOMAIN}}_backlog.yaml

```yaml
backlog:
  domain: "{{DOMAIN}}"
  system: "{{SYSTEM_NAME}}"
  generated_from: "COBOL analysis Pass 2B"

  personas:
    - id: "PERSONA-001"
      name: "[persona name]"
      description: "[who they are]"
      interaction: "[how they use the system]"

  epics:
    - id: "EPIC-{{DOMAIN}}-001"
      title: "[Business Capability Name]"
      business_value: "[why it exists]"
      cobol_programs: ["list of programs implementing this"]
      stories:
        - id: "{{DOMAIN}}-001"
          title: "[story title]"
          persona: "PERSONA-001"
          as_a: "[persona]"
          i_want_to: "[action]"
          so_that: "[value]"
          type: "[FUNCTIONAL | NON_FUNCTIONAL | REPORT]"
          priority_suggestion: "[MUST_HAVE | SHOULD_HAVE | COULD_HAVE | WONT_HAVE]"
          acceptance_criteria:
            - given: "[context]"
              when: "[action]"
              then: "[outcome]"
            - given: "[edge case]"
              when: "[action]"
              then: "[outcome]"
            - given: "[error scenario]"
              when: "[problem]"
              then: "[response]"
          business_rules: ["BR-xxx", "BR-yyy"]
          cobol_source:
            programs: ["program names"]
            paragraphs: ["key paragraphs"]
          pm_decision: null  # PM fills in: KEEP | MODIFY | ELIMINATE | ENHANCE
          pm_notes: null     # PM fills in
          modern_alternative: "[suggestion if applicable]"

  reports:
    - id: "{{DOMAIN}}-RPT-001"
      title: "[report name]"
      persona: "PERSONA-002"
      frequency: "[Daily | Weekly | Monthly | On-Demand]"
      current_delivery: "[Print | Email | Spool]"
      suggested_alternative: "[Dashboard | API | Email | Self-Service]"
      acceptance_criteria:
        - given: "[timing]"
          when: "[trigger]"
          then: "[content and format]"
      pm_decision: null
      pm_notes: null

  summary:
    total_stories: [N]
    functional: [N]
    non_functional: [N]
    reports: [N]
    business_rules_covered: [N]
    business_rules_total: [N]
```
```

---

## Prompt 2B.2 — Story Map (Aggregation)

After all domains, create a visual story map.

```
ROLE: You are a product manager creating a story map across the entire 
system — organizing stories by user journey (horizontal) and priority 
(vertical).

TASK: Read all story backlogs from:
{{OUTPUT_DIR}}/02b_product_stories/specs/*_backlog.yaml

Produce: {{OUTPUT_DIR}}/02b_product_stories/prose/STORY_MAP.md

## Story Map — {{SYSTEM_NAME}}

### User Journey (Left to Right)

Organize all stories into the end-to-end user journeys:

**Journey 1: [e.g., "Loan Application to Funding"]**

| Step | Activity | Stories |
|------|----------|---------|
| 1 | [e.g., "Customer applies"] | [DOMAIN]-001, [DOMAIN]-002 |
| 2 | [e.g., "Credit check"] | [DOMAIN]-003 |
| 3 | [e.g., "Approval decision"] | [DOMAIN]-004, [DOMAIN]-005 |
| 4 | [e.g., "Document generation"] | [DOMAIN]-006 |
| 5 | [e.g., "Funding"] | [DOMAIN]-007 |

**Journey 2: [e.g., "Account Servicing"]**
[Continue...]

**Journey 3: [e.g., "Month-End Processing"]**
[Continue...]

### Priority Matrix

| Priority | Stories | Rationale |
|----------|---------|-----------|
| **MUST HAVE** (MVP) | [list] | [These are core to the business] |
| **SHOULD HAVE** (Release 1) | [list] | [Important but can wait for post-MVP] |
| **COULD HAVE** (Future) | [list] | [Nice to have] |
| **ELIMINATE CANDIDATES** | [list] | [These appear to be dead functionality, workarounds, or no longer needed] |

### Modernization Opportunities

Stories where the modern system can do BETTER than the COBOL system:

| Story | Current Limitation | Modern Opportunity |
|---|---|---|
| [story ID] | [what's limited today] | [what could be better in Java/web] |

### Cross-Domain Dependencies

Stories that span multiple domains and need coordination:

| Story A | Story B | Dependency | Impact |
|---|---|---|---|
| [story] | [story] | [what connects them] | [why it matters for sequencing] |
```

---

## Prompt 2B.3 — PM Review Worksheet

Generate a streamlined worksheet PMs can print or import into JIRA.

```
ROLE: You are a product manager creating a concise review worksheet 
that can be used in a story review meeting.

TASK: For each domain, create a worksheet optimized for PM review.

Produce: {{OUTPUT_DIR}}/02b_product_stories/prose/{{DOMAIN}}_pm_review_worksheet.md

## PM Review Worksheet: [Domain Name]

**Instructions:** For each story, circle your decision and add notes. 
Bring this to the story review meeting.

| # | Story | As a... I want to... | Decision | Notes |
|---|-------|---------------------|----------|-------|
| [DOMAIN]-001 | [title] | [abbreviated story] | KEEP / MODIFY / ELIMINATE / ENHANCE | |
| [DOMAIN]-002 | [title] | [abbreviated story] | KEEP / MODIFY / ELIMINATE / ENHANCE | |
| [DOMAIN]-RPT-001 | [report title] | Receive [report] [frequency] | KEEP / MODIFY / ELIMINATE / ENHANCE | |

### Key Questions for PM Review Meeting

[List the most important decisions PMs need to make — questions that 
surfaced from business rules flagged for SME review, calculations with 
conventions that might change, reports that might be obsolete, etc.]

1. [Question — e.g., "Should we continue using the 360-day interest 
   convention or switch to actual/365? This affects every interest 
   calculation in the system."]

2. [Question — e.g., "The monthly balance report is generated for 
   15 different department codes. Do all 15 departments still need it?"]

3. [Continue for all key decisions...]

### Impact of PM Decisions on Development

| If PM decides... | Impact on scope | Impact on timeline |
|---|---|---|
| Eliminate all reports | -[N] stories, ~[X]% reduction | -[N] weeks |
| Modify interest convention | +[N] stories for recalculation | +[N] weeks |
| Keep everything as-is | Baseline scope | Baseline timeline |
```

---

## How This Connects to Other Passes

```
Pass 2: Business Specs (rules, processes)
    │
    ▼
Pass 2B: Product Stories  ◄── PM REVIEWS AND AMENDS HERE
    │
    │   PM marks stories as KEEP / MODIFY / ELIMINATE / ENHANCE
    │   PM adds notes for modifications
    │   PM identifies new requirements
    │
    ▼
Pass 3: Functional Specs  ← Only for stories marked KEEP or MODIFY
    │                       Stories marked ELIMINATE are skipped
    │                       Stories marked MODIFY incorporate PM notes
    │                       Stories marked ENHANCE get new requirements added
    ▼
Pass 4: Technical Specs   ← Gherkin tests aligned with PM-approved stories
```

The product stories become the **gate** between analysis and development. 
Nothing proceeds to functional/technical specs without PM approval.

---

## Quality Gate Checklist

- [ ] Every business process from Pass 2 has corresponding user stories
- [ ] Every business rule is referenced by at least one story's acceptance criteria
- [ ] Stories are written in plain English (no COBOL or Java terminology)
- [ ] Every story has at least one Given-When-Then acceptance criterion
- [ ] Every story has a PM Decision field (even if blank — PM fills in)
- [ ] Report stories include suggested modern alternatives
- [ ] Story map shows end-to-end user journeys
- [ ] PM review worksheet is generated for each domain
- [ ] Eliminate candidates are identified with justification
- [ ] **PM REVIEW CHECKPOINT:** Product managers MUST review and mark up stories before Pass 3 begins
