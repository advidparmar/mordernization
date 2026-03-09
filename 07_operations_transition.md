# Pass 7 — Operations & Transition Planning

## Purpose

Capture operational knowledge that lives OUTSIDE the code — how the system is run, monitored, and supported in production — and produce the transition/cutover plan for moving from COBOL to Java.

## Why This Pass Exists

Code analysis tells you what the system *does*. This pass captures how the system *lives*:
- What happens at month-end, quarter-end, year-end?
- What do operators do when batch jobs fail?
- What SLAs exist? What gets paged at 2am?
- What manual interventions happen routinely?
- How do you actually cut over from COBOL to Java without a business outage?

Most of this information cannot be extracted from code. This pass generates the questions and templates that MUST be filled in by the operations team, along with whatever CAN be inferred from JCL, CICS definitions, and code patterns.

## Inputs Required

- JCL analyses (Pass 1)
- Batch processing design (Pass 4)
- Service decomposition (Pass 4)
- All prior pass outputs for context
- Operations team interviews (this pass generates the interview templates)

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Operations Runbook | `07_operations/prose/OPERATIONS_RUNBOOK.md` | `07_operations/specs/operations.yaml` |
| SLA & Monitoring Spec | `07_operations/prose/SLA_MONITORING.md` | `07_operations/specs/sla_monitoring.yaml` |
| Transition Plan | `07_operations/prose/TRANSITION_PLAN.md` | `07_operations/specs/transition_plan.yaml` |
| Interview Templates | `07_operations/prose/OPERATIONS_INTERVIEW_TEMPLATE.md` | — |

---

## Prompt 7.1 — Operations Runbook (Code-Inferred + Interview Template)

```
ROLE: You are a site reliability engineer documenting the operational 
characteristics of a legacy COBOL system. You can infer SOME operational 
information from the code and JCL. For everything else, you generate 
specific interview questions for the operations team.

CONTEXT:
- System: {{SYSTEM_NAME}}

PRIOR CONTEXT:
- JCL analyses: {{OUTPUT_DIR}}/01_discovery/prose/*_jcl_analysis.md
- Batch design: {{OUTPUT_DIR}}/04_technical/prose/BATCH_PROCESSING_DESIGN.md
- Error handling: {{OUTPUT_DIR}}/04_technical/prose/ERROR_HANDLING_STRATEGY.md
- Service decomposition: {{OUTPUT_DIR}}/04_technical/prose/SERVICE_DECOMPOSITION.md

TASK: Produce an operations runbook combining what can be inferred from 
code with gaps that need operations team input.

===== PRODUCE THREE FILES =====

FILE 1: {{OUTPUT_DIR}}/07_operations/prose/OPERATIONS_RUNBOOK.md

## Operations Runbook — {{SYSTEM_NAME}}

### System Overview (for Operations)
[Brief description written for an operations engineer, not a developer.
What does this system do? Who uses it? When is it busiest?]

### Batch Job Schedule (Inferred from JCL)

| Job Name | Programs | Schedule (inferred) | Dependencies | Typical Duration | ⚠️ Verified by Ops? |
|---|---|---|---|---|---|
| [job] | [programs] | [inferred from name/comments] | [job dependencies] | [unknown — ops must fill in] | [ ] |

**Calendar-Driven Processing:**
[Infer from JCL/code any month-end, quarter-end, year-end, or 
holiday-specific processing. Look for date comparisons, symbolic 
parameters, or conditional JCL.]

⚠️ **OPERATIONS TEAM MUST VERIFY:**
- [ ] Actual job schedule (times, frequencies, dependencies)
- [ ] Month-end / quarter-end / year-end special processing
- [ ] Holiday schedule adjustments
- [ ] Seasonal volume variations

### Error Recovery Procedures (Inferred from Code)

For each batch job, infer from the COBOL code and JCL:

**[Job Name] — Recovery**
| Error Type | COBOL Behavior (from code) | Current Recovery (NEED FROM OPS) |
|---|---|---|
| [e.g., "DB2 deadlock"] | [SQLCODE -911, program retries 3 times] | ⚠️ What does the operator do if retries exhaust? |
| [e.g., "Input file missing"] | [FILE STATUS 35, ABEND U1001] | ⚠️ Who is contacted? Where does the file come from? |
| [e.g., "Record validation failure"] | [Writes to error file, continues] | ⚠️ Who reviews the error file? What's the threshold? |

### Online System Operations (if CICS)
[CICS transaction availability, region startup/shutdown, peak hours]

⚠️ **OPERATIONS TEAM MUST PROVIDE:**
- [ ] CICS region availability windows
- [ ] Planned maintenance windows
- [ ] Peak usage hours and expected transaction volumes
- [ ] Current monitoring tools and dashboards
- [ ] Alerting thresholds and escalation paths

### Monitoring & Alerting (Current State)

⚠️ **THIS SECTION CANNOT BE INFERRED FROM CODE. Operations team must 
provide:**

- [ ] What monitoring tools are used? (OMEGAMON, Zabbix, BMC, etc.)
- [ ] What metrics are tracked?
- [ ] What thresholds trigger alerts?
- [ ] Who gets alerted? (Pager/email/Slack)
- [ ] What's the escalation path?
- [ ] What SLAs exist?
- [ ] What's the incident response process?

### Data Interfaces — Operational View
[For each external interface from Pass 3, document operational aspects]

| Interface | Source/Target | Transfer Method | Schedule | ⚠️ Contact |
|---|---|---|---|---|
| [name] | [system] | [FTP/MQ/NDM/etc.] | [when] | ⚠️ Ops must provide |

### Backup & Recovery

⚠️ **OPERATIONS TEAM MUST PROVIDE:**
- [ ] Database backup schedule and retention
- [ ] File backup procedures
- [ ] Disaster recovery plan and RPO/RTO targets
- [ ] Last DR test date and results

---

FILE 2: {{OUTPUT_DIR}}/07_operations/prose/OPERATIONS_INTERVIEW_TEMPLATE.md

## Operations Team Interview Template — {{SYSTEM_NAME}}

**Instructions:** This template contains specific questions that MUST be 
answered by the mainframe operations team before the Java system can go 
to production. Each question is tagged with priority and impact.

### Section 1: Batch Operations (Priority: CRITICAL)

1. What is the exact schedule for each batch job? (Include day, time, 
   timezone, and any conditional scheduling)
   | Job | Schedule | Timezone | Conditional? |
   |---|---|---|---|
   | [pre-filled from JCL analysis] | _____ | _____ | _____ |

2. What are the job dependencies? (Which jobs must complete before 
   others can start?)

3. What is the typical batch window? When must all overnight batch 
   complete by?

4. What happens if batch doesn't complete by [time]? Who is contacted? 
   What's the business impact?

5. Are there month-end, quarter-end, or year-end special runs? Describe 
   each.

6. What manual steps does the operator perform between or during jobs?

7. What restart/recovery procedures exist for each job? Is there 
   documentation?

8. What are the typical volumes for each job? How have they grown 
   year-over-year?

### Section 2: Online Operations (Priority: HIGH)

9. What are the system availability hours? Is there planned downtime?

10. What's the peak transaction volume? When does it occur?

11. What monitoring dashboards exist for online transaction health?

12. What response time SLAs exist? Who measures them?

### Section 3: Incident Management (Priority: HIGH)

13. What is the escalation path when something goes wrong?
    - Level 1: _____
    - Level 2: _____
    - Level 3: _____

14. What are the most common production incidents? (Top 5)

15. What's the mean time to recovery (MTTR) for typical incidents?

16. Are there runbook documents for common issues? Where are they stored?

### Section 4: External Interfaces (Priority: HIGH)

17. For each external file/data interface:
    | Interface | External Contact | Support Hours | SLA |
    |---|---|---|---|
    | [pre-filled] | _____ | _____ | _____ |

18. What happens if an expected file doesn't arrive? Who investigates?

19. Are there any manual data corrections that happen regularly?

### Section 5: Security & Compliance (Priority: MEDIUM)

20. How are user access requests processed?
21. How often are access reviews conducted?
22. Are there any compliance audits? When? What's reviewed?

### Section 6: Environment (Priority: MEDIUM)

23. What are the differences between DEV, QA, and PROD environments?
24. How is code promoted between environments?
25. What change management process exists?

---

FILE 3: {{OUTPUT_DIR}}/07_operations/specs/operations.yaml

```yaml
operations:
  system: "{{SYSTEM_NAME}}"
  
  batch_schedule:
    jobs:
      - name: "[job name]"
        programs: ["list"]
        schedule_inferred: "[from JCL analysis]"
        schedule_verified: null  # Ops team fills in
        dependencies: ["other jobs"]
        typical_duration: null  # Ops team fills in
        volume: null  # Ops team fills in
        recovery_from_code:
          - error: "[error type]"
            code_behavior: "[what COBOL does]"
            operator_action: null  # Ops team fills in
    
    calendar_processing:
      month_end: null  # Ops team fills in
      quarter_end: null
      year_end: null
      holidays: null

  online:
    availability_hours: null  # Ops team fills in
    peak_hours: null
    transaction_volume: null
    response_time_sla: null

  monitoring:
    tools: null  # Ops team fills in
    dashboards: null
    alert_thresholds: null
    escalation_path: null
    on_call_rotation: null

  interfaces_operational:
    - interface_id: "[from Pass 3]"
      external_contact: null  # Ops team fills in
      support_hours: null
      sla: null
      failure_procedure: null

  disaster_recovery:
    rpo: null  # Ops team fills in
    rto: null
    backup_schedule: null
    last_dr_test: null

  status: "NEEDS_OPERATIONS_INPUT"
```
```

---

## Prompt 7.2 — Transition & Cutover Plan

```
ROLE: You are a modernization program manager designing the transition 
from COBOL to Java in production.

CONTEXT:
- System: {{SYSTEM_NAME}}

PRIOR CONTEXT:
- Service decomposition: {{OUTPUT_DIR}}/04_technical/prose/SERVICE_DECOMPOSITION.md
- Batch design: {{OUTPUT_DIR}}/04_technical/prose/BATCH_PROCESSING_DESIGN.md
- Migration plan: {{OUTPUT_DIR}}/06_data_quality/prose/MIGRATION_PLAN.md
- Operations: {{OUTPUT_DIR}}/07_operations/specs/operations.yaml

TASK: Design the transition plan for moving from COBOL to Java.

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/07_operations/prose/TRANSITION_PLAN.md

## Transition Plan — {{SYSTEM_NAME}}

### Transition Strategy
[Strangler fig / Big bang / Parallel run — justify based on system 
characteristics discovered in prior passes]

### Phase Plan

**Phase 0: Foundation** (Duration: [estimate])
- Deploy Java infrastructure alongside mainframe
- Implement anti-corruption layer / API gateway
- Set up parallel-run comparison tooling
- Deploy monitoring for Java services
- No traffic to Java yet

**Phase 1: Shadow Mode** (Duration: [estimate])
- Route production traffic to BOTH COBOL and Java
- Java results are computed but NOT used
- Compare outputs continuously
- Fix discrepancies
- Success criteria: [N] days with <[X]% discrepancy

**Phase 2: Canary** (Duration: [estimate])
- Route [X]% of traffic to Java for real
- Monitor closely
- Automatic fallback to COBOL on error
- Gradually increase percentage
- Success criteria: [X]% traffic for [N] days

**Phase 3: Majority Migration** (Duration: [estimate])
- Flip to Java as primary, COBOL as fallback
- [X]% traffic to Java
- COBOL runs in shadow for comparison
- Success criteria: [N] days stable

**Phase 4: COBOL Decommission** (Duration: [estimate])
- All traffic on Java
- COBOL maintained but not receiving traffic
- Final reconciliation
- Archive COBOL system
- Decommission mainframe resources

### Service-by-Service Cutover Sequence
[Based on service decomposition, which services cut over first?]

| Phase | Service(s) | Rationale | Dependencies | Rollback Approach |
|---|---|---|---|---|
| 1 | [service] | [why first — least risk] | [none] | [revert routing] |
| 2 | [service] | [why next] | [phase 1 complete] | [revert routing] |

### Parallel Run Design

**For Batch Processing:**
- Run COBOL and Java batch jobs with same input
- Compare output files field-by-field
- Financial totals must match to the penny
- Record-count must match exactly
- Exception report for any discrepancy

**For Online Processing:**
- API gateway routes to both systems
- Compare response payloads
- Log discrepancies without affecting users
- Dashboard showing match rate in real-time

### Rollback Procedures

For each phase, document:
| Trigger | Action | Duration | Data Impact |
|---|---|---|---|
| [what triggers rollback] | [specific steps] | [how long] | [any data reconciliation needed] |

### Communication Plan
| Audience | When | What to Communicate |
|---|---|---|
| Business users | Before Phase 2 | System will be transitioning, expected behavior |
| Operations | Before each phase | New runbooks, new escalation paths |
| External partners | Before interfaces change | New endpoints, format changes, testing |

### Success Metrics
| Metric | Target | Measurement |
|---|---|---|
| Transaction accuracy | 100% match with COBOL | Parallel run comparison |
| Response time | Within [X]ms of COBOL | APM monitoring |
| Batch completion | Same window as COBOL | Job scheduler |
| Error rate | ≤ COBOL error rate | Error log comparison |
| Availability | [X]% | Uptime monitoring |

---

FILE 2: {{OUTPUT_DIR}}/07_operations/specs/transition_plan.yaml

```yaml
transition:
  system: "{{SYSTEM_NAME}}"
  strategy: "[STRANGLER_FIG | BIG_BANG | PARALLEL_RUN]"
  
  phases:
    - phase: 0
      name: "Foundation"
      duration_estimate: "[weeks]"
      description: "[what happens]"
      success_criteria: "[measurable criteria]"
      services_affected: []
      rollback: "[approach]"
    
    - phase: 1
      name: "Shadow Mode"
      duration_estimate: "[weeks]"
      traffic_split: { cobol: 100, java: 0, comparison: true }
      success_criteria: "< [X]% discrepancy for [N] days"
      services_affected: ["list"]
    
    - phase: 2
      name: "Canary"
      traffic_split: { cobol: 90, java: 10 }
      success_criteria: "[criteria]"
      rollback: "Automatic on error rate > [X]%"
    
    - phase: 3
      name: "Majority"
      traffic_split: { cobol: 10, java: 90 }
      success_criteria: "[criteria]"
    
    - phase: 4
      name: "Decommission"
      traffic_split: { cobol: 0, java: 100 }
      cobol_status: "Archive"

  service_cutover_sequence:
    - order: 1
      service: "[name]"
      rationale: "[why first]"
      dependencies: []
      estimated_duration: "[weeks]"
    - order: 2
      service: "[name]"
      dependencies: ["phase 1 service"]

  parallel_run:
    batch:
      comparison_method: "Field-by-field output diff"
      financial_tolerance: "0.00"
      record_count_tolerance: "0"
    online:
      comparison_method: "Response payload diff at API gateway"
      routing: "Duplicate traffic via gateway"
      dashboard: true

  rollback_procedures:
    - phase: [N]
      trigger: "[condition]"
      steps: ["step 1", "step 2"]
      duration: "[minutes/hours]"
      data_reconciliation: "[needed / not needed]"
```
```

---

## Prompt 7.3 — SLA & Monitoring Specification

```
ROLE: You are an SRE designing the monitoring and observability strategy 
for the modernized system.

TASK: Based on what can be inferred from the code and what the operations 
team will need to provide, create the SLA and monitoring spec.

===== PRODUCE =====

FILE: {{OUTPUT_DIR}}/07_operations/prose/SLA_MONITORING.md

[Define proposed SLAs, monitoring metrics, alerting thresholds, 
dashboards, and logging strategy for the Java system. Reference 
error handling strategy from Pass 4.]
```

---

## Quality Gate Checklist

- [ ] Operations interview template is complete and specific
- [ ] Batch schedule has entries for every JCL job from Pass 1
- [ ] Transition plan has rollback procedures for every phase
- [ ] Parallel run comparison criteria are defined (including financial tolerances)
- [ ] Service cutover sequence aligns with service decomposition from Pass 4
- [ ] **OPERATIONS TEAM CHECKPOINT:** Interview template must be completed by operations team before development begins
